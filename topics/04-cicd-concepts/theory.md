# CI/CD (Continuous Integration & Continuous Delivery) — Theory Questions

---

## Q1: What is CI/CD? Explain each part.

**Answer:**

**CI (Continuous Integration):**
- Developers frequently merge their code changes into a shared branch (usually `main`)
- Every merge triggers an automated process: build the code, run tests, check for errors
- Goal: Catch bugs early, before they pile up

**CD has two meanings:**

**Continuous Delivery:**
- After CI passes, the code is automatically prepared for release
- A human still clicks a button to deploy to production
- Goal: Code is always in a deployable state

**Continuous Deployment:**
- After CI passes, the code is automatically deployed to production — no human approval needed
- Goal: Get changes to users as fast as possible

```
Developer pushes code
    → CI: Build → Test → Lint → Scan
        → CD (Delivery): Package → Stage → Ready for manual deploy
        → CD (Deployment): Package → Stage → Auto-deploy to production
```

---

## Q2: What are the stages of a typical CI/CD pipeline?

**Answer:**

| Stage | What Happens | Tools |
|-------|-------------|-------|
| **Source** | Code is pushed/PR is created | Git, GitHub, GitLab |
| **Build** | Code is compiled/packaged | Maven, npm, Docker |
| **Test** | Automated tests run | Jest, pytest, Selenium |
| **Lint/Scan** | Code quality and security checks | ESLint, SonarQube, Trivy |
| **Package** | Create deployable artifact | Docker image, JAR, zip |
| **Deploy to Staging** | Deploy to a test environment | Kubernetes, Terraform |
| **Integration Tests** | Test in staging environment | Postman, Cypress |
| **Deploy to Production** | Deploy to live environment | ArgoCD, Spinnaker |
| **Monitor** | Watch for issues after deploy | Prometheus, Grafana |

---

## Q3: What are the different deployment strategies?

**Answer:**

**1. Rolling Update:**
- Replace old instances one at a time with new ones
- Zero downtime, but both versions run simultaneously during the update
- Risk: If the new version has a bug, some users are affected before you notice

**2. Blue-Green Deployment:**
- Run two identical environments: Blue (current) and Green (new)
- Deploy to Green, test it, then switch traffic from Blue to Green
- Instant rollback: just switch back to Blue
- Cost: You need double the infrastructure

**3. Canary Deployment:**
- Deploy the new version to a small percentage of users (e.g., 5%)
- Monitor for errors, then gradually increase to 100%
- Low risk: only a few users are affected if something goes wrong

**4. A/B Testing:**
- Similar to canary, but the goal is to compare features (not just test stability)
- Route specific user groups to different versions

**5. Recreate (Big Bang):**
- Stop all old instances, then start all new instances
- Simple but causes downtime
- Only acceptable for non-critical applications

| Strategy | Downtime | Risk | Rollback Speed | Cost |
|----------|----------|------|---------------|------|
| Rolling | None | Medium | Slow | Low |
| Blue-Green | None | Low | Instant | High |
| Canary | None | Low | Fast | Medium |
| Recreate | Yes | High | Slow | Low |

---

## Q4: What is a build artifact? Why is artifact management important?

**Answer:**
A **build artifact** is the output of a build process — the thing you actually deploy. Examples: Docker image, JAR file, npm package, compiled binary, zip file.

**Why artifact management matters:**
- **Reproducibility** — Deploy the exact same artifact to staging and production
- **Versioning** — Track which version is deployed where
- **Speed** — Build once, deploy many times (don't rebuild for each environment)
- **Audit trail** — Know what was deployed and when
- **Rollback** — Quickly redeploy a previous version

**Artifact repositories:** JFrog Artifactory, Nexus, AWS ECR, GitHub Packages, Docker Hub

---

## Q5: What is "shift left" in the context of CI/CD?

**Answer:**
**Shift left** means moving testing, security, and quality checks earlier in the development process. Instead of finding bugs in production, find them during development.

**Traditional:** `Code → Build → Deploy → Test → Find bugs (expensive!)`
**Shift left:** `Code → Lint → Unit Test → Security Scan → Build → Deploy → Monitor`

**Examples:**
- Run linters in the IDE (before even committing)
- Run unit tests in pre-commit hooks
- Security scanning in the CI pipeline (not after deployment)
- Code review before merging
- Infrastructure validation before applying (`terraform plan`)

---

## Q6: What is "pipeline as code"?

**Answer:**
**Pipeline as code** means defining your CI/CD pipeline in a file (like `Jenkinsfile`, `.github/workflows/ci.yml`) that lives in your Git repository alongside your application code.

**Benefits over UI-configured pipelines:**
- **Version controlled** — Track changes to the pipeline over time
- **Code review** — Pipeline changes go through pull requests
- **Reproducible** — Anyone can recreate the pipeline from the file
- **Portable** — Move between environments easily
- **Disaster recovery** — Pipeline definition is backed up with your code

---

## Q7: What is feature flagging and how does it relate to CI/CD?

**Answer:**
**Feature flags** (feature toggles) are switches in your code that let you turn features on or off without deploying new code.

```javascript
if (featureFlags.isEnabled('new-checkout')) {
    showNewCheckout();
} else {
    showOldCheckout();
}
```

**How it relates to CI/CD:**
- Enables **trunk-based development** — merge incomplete features to main (hidden behind a flag)
- Enables **canary releases** — enable the feature for a percentage of users
- Enables **instant rollback** — turn off the flag instead of redeploying
- Separates **deployment** from **release** — deploy code anytime, release features when ready

**Tools:** LaunchDarkly, Unleash, Flagsmith, ConfigCat, AWS AppConfig

---

## Q8: What are the different types of testing in CI/CD?

**Answer:**

| Test Type | What It Tests | Speed | Scope |
|-----------|--------------|-------|-------|
| **Unit Tests** | Individual functions/methods | Very fast | Smallest |
| **Integration Tests** | How components work together | Medium | Medium |
| **End-to-End (E2E) Tests** | Full user workflows | Slow | Largest |
| **Smoke Tests** | Basic functionality after deploy | Fast | Critical paths |
| **Performance Tests** | Speed and load handling | Slow | System-wide |
| **Security Tests** | Vulnerabilities and compliance | Medium | System-wide |

**Testing pyramid:**
```
        /  E2E  \        ← Few, slow, expensive
       / Integration \    ← Some, medium speed
      /   Unit Tests   \  ← Many, fast, cheap
```

**Best practice:** Have many unit tests, fewer integration tests, and even fewer E2E tests. Run fast tests first in the pipeline to get quick feedback.

---

## Q9: What is the difference between a monorepo and polyrepo CI/CD approach?

**Answer:**

**Monorepo** — All services in one repository:
- CI/CD must detect which services changed and only build those
- Tools: Nx, Turborepo, Bazel
- Pros: Easy code sharing, atomic cross-service changes
- Cons: Complex CI/CD, longer build times

**Polyrepo** — Each service has its own repository:
- Each repo has its own independent CI/CD pipeline
- Pros: Independent deployments, simpler pipelines
- Cons: Code duplication, harder cross-service changes

---

## Q10: What is environment promotion in CI/CD?

**Answer:**
**Environment promotion** is the process of moving a build artifact through increasingly production-like environments before it reaches production.

```
Build → Dev → QA → Staging → Production
```

**Key principles:**
- **Same artifact** moves through all environments (never rebuild)
- **Configuration** changes per environment (database URLs, API keys)
- **Automated gates** — tests must pass before promotion
- **Manual gates** — human approval for production (in Continuous Delivery)

**Example with Docker:**
```bash
# Build once, tag with commit hash
docker build -t myapp:abc123 .

# Promote through environments by retagging
docker tag myapp:abc123 myapp:staging
docker tag myapp:abc123 myapp:production
```

---

## Q11: What is idempotency and why does it matter in CI/CD?

**Answer:**
**Idempotency** means running the same operation multiple times produces the same result as running it once. In other words, it's safe to repeat.

**Why it matters:**
- Pipelines may retry failed steps — they should be safe to re-run
- Deployments should produce the same result whether run once or ten times
- Infrastructure provisioning should not create duplicate resources

**Examples:**
- ✅ Idempotent: `kubectl apply -f deployment.yaml` (applies desired state)
- ❌ Not idempotent: `kubectl create -f deployment.yaml` (fails if already exists)
- ✅ Idempotent: `terraform apply` (converges to desired state)
- ❌ Not idempotent: A script that appends to a file on every run

---

## Q12: What is a rollback and how should it be implemented?

**Answer:**
A **rollback** is reverting to a previous working version when a deployment causes problems.

**Rollback strategies:**

1. **Redeploy previous version:**
   ```bash
   # Kubernetes
   kubectl rollout undo deployment/myapp
   
   # Docker
   docker run myapp:previous-version
   ```

2. **Blue-Green switch:**
   - Just route traffic back to the Blue (old) environment

3. **Feature flag:**
   - Disable the problematic feature flag

4. **Database rollback:**
   - This is the hardest part — database migrations may not be reversible
   - Use **backward-compatible migrations** (add columns, don't remove them)
   - Keep the old code compatible with the new schema

**Best practices:**
- Always have a rollback plan before deploying
- Test rollback procedures regularly
- Keep previous artifacts available
- Use database migration tools that support rollbacks (Flyway, Alembic)
- Monitor closely after deployment to catch issues early