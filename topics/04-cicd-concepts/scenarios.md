# CI/CD — Scenario-Based Questions

---

## S1: Your CI/CD pipeline takes 45 minutes to complete. The team is complaining it's too slow. How do you speed it up?

**Answer:**

1. **Analyze where time is spent:**
   - Look at pipeline stage durations
   - Identify the slowest stages

2. **Quick wins:**
   - **Cache dependencies:** Don't download `node_modules` or Maven dependencies every time
     ```yaml
     # GitHub Actions example
     - uses: actions/cache@v4
       with:
         path: node_modules
         key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
     ```
   - **Parallelize tests:** Split tests across multiple runners
   - **Use faster runners:** Larger CI machines with more CPU/RAM
   - **Docker layer caching:** Reuse unchanged Docker layers

3. **Structural improvements:**
   - **Run tests in parallel:** Split test suites across multiple jobs
   - **Only build what changed:** In a monorepo, skip unchanged services
   - **Move slow tests to a separate pipeline:** Run E2E tests on a schedule (nightly) instead of every push
   - **Use incremental builds:** Only recompile changed files

4. **Optimize Docker builds:**
   ```dockerfile
   # Bad: copies everything, invalidates cache on any change
   COPY . .
   RUN npm install
   
   # Good: copy package.json first, install, then copy code
   COPY package*.json ./
   RUN npm install
   COPY . .
   ```

5. **Optimize test execution:**
   - Use test impact analysis (only run tests affected by changes)
   - Remove flaky tests that cause retries
   - Use test splitting tools (e.g., Jest `--shard`, CircleCI test splitting)

**Target:** Most CI pipelines should complete in under 10–15 minutes.

---

## S2: A deployment to production failed and users are seeing errors. Walk through your incident response.

**Answer:**

**Phase 1: Detect and Assess (0–5 minutes)**
1. Confirm the issue — check monitoring dashboards, error rates, user reports
2. Assess severity — how many users are affected? Is it a complete outage or partial?
3. Alert the team — notify on-call engineers and relevant stakeholders

**Phase 2: Mitigate (5–15 minutes)**
1. **Rollback immediately** — don't debug in production
   ```bash
   # Kubernetes
   kubectl rollout undo deployment/myapp -n production
   
   # Or deploy the last known good version
   kubectl set image deployment/myapp myapp=myapp:v1.4.9
   ```
2. Verify the rollback worked — check error rates dropping
3. Communicate status to stakeholders

**Phase 3: Investigate (after mitigation)**
1. Check what changed in the deployment:
   ```bash
   git log --oneline v1.4.9..v1.5.0
   ```
2. Review application logs from the failed deployment
3. Check if it was a code issue, config issue, or infrastructure issue
4. Try to reproduce in staging

**Phase 4: Fix and Prevent**
1. Fix the root cause
2. Add tests that would have caught this issue
3. Deploy the fix to staging first, verify, then production
4. Write a **post-mortem** document:
   - What happened?
   - Timeline of events
   - Root cause
   - What we did to fix it
   - Action items to prevent recurrence

---

## S3: You need to set up a CI/CD pipeline for a new microservice. What does it look like?

**Answer:**

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  IMAGE_NAME: myapp
  REGISTRY: ghcr.io/${{ github.repository_owner }}

jobs:
  # Stage 1: Lint and Test
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run test -- --coverage
      - uses: actions/upload-artifact@v4
        with:
          name: coverage
          path: coverage/

  # Stage 2: Security Scan
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          severity: 'HIGH,CRITICAL'

  # Stage 3: Build and Push Docker Image
  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Build and push Docker image
        run: |
          docker build -t $REGISTRY/$IMAGE_NAME:${{ github.sha }} .
          docker push $REGISTRY/$IMAGE_NAME:${{ github.sha }}

  # Stage 4: Deploy to Staging
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy to staging
        run: |
          kubectl set image deployment/$IMAGE_NAME \
            $IMAGE_NAME=$REGISTRY/$IMAGE_NAME:${{ github.sha }} \
            -n staging

  # Stage 5: Deploy to Production (manual approval)
  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production  # Requires manual approval
    steps:
      - name: Deploy to production
        run: |
          kubectl set image deployment/$IMAGE_NAME \
            $IMAGE_NAME=$REGISTRY/$IMAGE_NAME:${{ github.sha }} \
            -n production
```

---

## S4: Your team wants to implement canary deployments. How would you set this up?

**Answer:**

**Using Kubernetes with Nginx Ingress:**

1. **Deploy the stable version (90% traffic):**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-stable
   spec:
     replicas: 9
     template:
       spec:
         containers:
         - name: myapp
           image: myapp:v1.0
   ```

2. **Deploy the canary version (10% traffic):**
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: myapp-canary
   spec:
     replicas: 1
     template:
       spec:
         containers:
         - name: myapp
           image: myapp:v1.1
   ```

3. **Both share the same Service** (traffic splits based on replica count)

4. **Monitor canary metrics:**
   - Error rate comparison (canary vs. stable)
   - Response time comparison
   - Business metrics (conversion rate, etc.)

5. **Progressive rollout with Argo Rollouts:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Rollout
   metadata:
     name: myapp
   spec:
     strategy:
       canary:
         steps:
         - setWeight: 5
         - pause: {duration: 10m}
         - setWeight: 20
         - pause: {duration: 10m}
         - setWeight: 50
         - pause: {duration: 10m}
         - setWeight: 100
         analysis:
           templates:
           - templateName: success-rate
   ```

**Automated canary analysis:**
- If error rate increases > 1% compared to stable → automatic rollback
- If latency increases > 200ms → automatic rollback
- If all metrics are healthy → continue promotion

---

## S5: A CI pipeline keeps failing intermittently with "flaky" tests. How do you handle this?

**Answer:**

1. **Identify flaky tests:**
   ```bash
   # Run tests multiple times to find flaky ones
   for i in {1..10}; do npm test 2>&1 | tee "run-$i.log"; done
   
   # Or use a test framework's retry feature
   jest --forceExit --detectOpenHandles
   ```

2. **Common causes of flaky tests:**
   - **Timing issues** — Tests depend on specific timing (use mocks/stubs)
   - **Shared state** — Tests affect each other (isolate test data)
   - **External dependencies** — Tests call real APIs (use mocks)
   - **Resource contention** — Tests compete for ports/files (use random ports)
   - **Order dependency** — Tests only pass in a specific order (make them independent)

3. **Short-term fix:**
   - Quarantine flaky tests (mark them and run separately)
   - Add automatic retries for known flaky tests
   ```yaml
   # GitHub Actions retry
   - uses: nick-fields/retry@v2
     with:
       max_attempts: 3
       command: npm test
   ```

4. **Long-term fix:**
   - Fix the root cause of each flaky test
   - Track flaky test metrics over time
   - Set a policy: flaky tests must be fixed within X days or deleted
   - Use test isolation (each test sets up and tears down its own data)
   - Replace real external calls with mocks/stubs