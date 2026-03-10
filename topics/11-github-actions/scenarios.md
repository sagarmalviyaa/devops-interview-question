# GitHub Actions — Scenario-Based Questions

---

## S1: You need to set up a workflow that deploys to staging on every push to main, but requires manual approval for production. How?

**Answer:**

```yaml
name: Deploy Pipeline

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          kubectl apply -f k8s/ --namespace staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production    # Has required reviewers configured
      url: https://myapp.com
    steps:
      - uses: actions/checkout@v4
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          kubectl apply -f k8s/ --namespace production
```

**Setup in GitHub:**
1. Go to Repository Settings → Environments
2. Create "production" environment
3. Add required reviewers (team leads, senior engineers)
4. Optionally add a wait timer (e.g., 15 minutes)

---

## S2: Your GitHub Actions workflow is slow (20+ minutes). How do you optimize it?

**Answer:**

1. **Cache dependencies:**
   ```yaml
   - uses: actions/cache@v4
     with:
       path: node_modules
       key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
   ```

2. **Run jobs in parallel:**
   ```yaml
   jobs:
     lint:
       runs-on: ubuntu-latest
       steps: [...]
     
     test:
       runs-on: ubuntu-latest    # Runs at the same time as lint
       steps: [...]
     
     build:
       needs: [lint, test]       # Waits for both to finish
       steps: [...]
   ```

3. **Use path filters** (skip unnecessary runs):
   ```yaml
   on:
     push:
       paths:
         - 'src/**'
         - 'package.json'
       paths-ignore:
         - '**.md'
         - 'docs/**'
   ```

4. **Use larger runners** (GitHub-hosted or self-hosted with more CPU/RAM)

5. **Split test suites:**
   ```yaml
   strategy:
     matrix:
       shard: [1, 2, 3, 4]
   steps:
     - run: npm test -- --shard=${{ matrix.shard }}/4
   ```

6. **Use Docker layer caching** for Docker builds

---

## S3: A secret was accidentally printed in the GitHub Actions logs. What do you do?

**Answer:**

1. **Rotate the secret immediately** — Change the API key/password/token at the source
2. **Update the secret in GitHub** — Repository Settings → Secrets → Update
3. **Delete the workflow run logs:**
   - Go to the Actions tab → find the run → click "..." → Delete logs
4. **Check for exposure:**
   - Was the log public? (public repos have public logs)
   - Check if anyone accessed the logs
5. **Prevent recurrence:**
   - GitHub automatically masks secrets referenced via `${{ secrets.* }}`
   - Don't echo secrets: `echo ${{ secrets.MY_KEY }}` will print `***`
   - But if a secret is used in a command that outputs it differently, it may leak
   - Use `add-mask` for dynamic secrets:
     ```yaml
     - run: |
         TOKEN=$(get-token)
         echo "::add-mask::$TOKEN"
         echo "TOKEN=$TOKEN" >> $GITHUB_ENV
     ```