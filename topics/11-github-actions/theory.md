# GitHub Actions — Theory Questions

---

## Q1: What is GitHub Actions?

**Answer:**
**GitHub Actions** is a CI/CD platform built into GitHub. It lets you automate workflows — building, testing, and deploying code — directly from your GitHub repository.

**Key advantages:**
- **Native GitHub integration** — No external CI server needed
- **YAML-based** — Workflows defined in `.github/workflows/` directory
- **Marketplace** — Thousands of pre-built actions to reuse
- **Free tier** — 2,000 minutes/month for public repos (unlimited), private repos have limits
- **Matrix builds** — Test across multiple OS/language versions simultaneously

---

## Q2: What are the key components of a GitHub Actions workflow?

**Answer:**

```yaml
# .github/workflows/ci.yml
name: CI Pipeline              # Workflow name

on:                            # Triggers
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:                           # Global environment variables
  NODE_ENV: production

jobs:                          # Jobs (run in parallel by default)
  build:
    runs-on: ubuntu-latest     # Runner (machine type)
    
    steps:                     # Steps (run sequentially)
      - uses: actions/checkout@v4    # Action (reusable step)
      - name: Install deps           # Step name
        run: npm install             # Shell command
      - name: Run tests
        run: npm test
```

**Hierarchy:** Workflow → Jobs → Steps

| Component | Description |
|-----------|-------------|
| **Workflow** | The entire automation file (triggered by events) |
| **Job** | A set of steps that run on the same runner |
| **Step** | A single task (either a shell command or an action) |
| **Action** | A reusable unit of code (from marketplace or custom) |
| **Runner** | The machine that executes the job |

---

## Q3: What are GitHub Actions runners?

**Answer:**
A **runner** is the machine that executes your workflow jobs.

**GitHub-hosted runners:**
- Managed by GitHub (you don't maintain them)
- Fresh VM for every job (clean environment)
- Available OS: Ubuntu, Windows, macOS
- Pre-installed tools: Node.js, Python, Docker, etc.
- Limited: 6 hours max per job, limited concurrency

**Self-hosted runners:**
- Your own machines (physical or virtual)
- You maintain and secure them
- Use when: you need specific hardware, private network access, or more power
- Persistent: state carries over between jobs (be careful!)

```yaml
jobs:
  build:
    runs-on: ubuntu-latest          # GitHub-hosted
  
  deploy:
    runs-on: self-hosted            # Self-hosted
    # Or with labels:
    runs-on: [self-hosted, linux, gpu]
```

---

## Q4: What are GitHub Actions secrets and how do you use them?

**Answer:**
**Secrets** are encrypted environment variables stored in GitHub. They're used for sensitive data like API keys, passwords, and tokens.

**Setting secrets:**
- Repository Settings → Secrets and variables → Actions → New repository secret

**Using secrets in workflows:**
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to production
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: aws s3 sync ./dist s3://my-bucket
```

**Secret scopes:**
- **Repository secrets** — Available to workflows in that repo
- **Environment secrets** — Available only in specific environments (staging, production)
- **Organization secrets** — Shared across repos in an organization

**Security notes:**
- Secrets are masked in logs (shown as `***`)
- Secrets are not available in forked repos (prevents leaking via PRs)
- Use OIDC (OpenID Connect) for cloud providers instead of long-lived keys

---

## Q5: What is a matrix strategy in GitHub Actions?

**Answer:**
A **matrix** lets you run the same job with different configurations (OS, language version, etc.) in parallel.

```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20, 22]
      fail-fast: false    # Don't cancel other jobs if one fails
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```

This creates **9 jobs** (3 OS × 3 Node versions), all running in parallel.

---

## Q6: What are reusable workflows and composite actions?

**Answer:**

**Reusable Workflows** — Call an entire workflow from another workflow:
```yaml
# .github/workflows/reusable-deploy.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string
    secrets:
      deploy_key:
        required: true

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ inputs.environment }}"
```

```yaml
# .github/workflows/main.yml
jobs:
  deploy-staging:
    uses: ./.github/workflows/reusable-deploy.yml
    with:
      environment: staging
    secrets:
      deploy_key: ${{ secrets.STAGING_KEY }}
```

**Composite Actions** — Bundle multiple steps into a single reusable action:
```yaml
# .github/actions/setup-and-test/action.yml
name: Setup and Test
runs:
  using: composite
  steps:
    - run: npm ci
      shell: bash
    - run: npm test
      shell: bash
```

**When to use which:**
- **Reusable workflows** — For complete pipelines shared across repos
- **Composite actions** — For reusable groups of steps within a workflow

---

## Q7: What are GitHub Actions environments and deployment protection rules?

**Answer:**
**Environments** represent deployment targets (staging, production) with their own secrets and protection rules.

```yaml
jobs:
  deploy-production:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.com
    steps:
      - run: echo "Deploying to production"
```

**Protection rules:**
- **Required reviewers** — Someone must approve before the job runs
- **Wait timer** — Delay before deployment (e.g., 30 minutes)
- **Branch restrictions** — Only deploy from specific branches
- **Custom rules** — Call external APIs for approval

This enables manual approval gates for production deployments.

---

## Q8: How do you cache dependencies in GitHub Actions?

**Answer:**

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Cache node modules
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.os }}-node-${{ hashFiles('package-lock.json') }}
          restore-keys: |
            ${{ runner.os }}-node-
      
      - name: Install dependencies
        run: npm ci
```

**How it works:**
1. First run: No cache exists → installs everything → saves cache
2. Next run: Cache key matches → restores from cache → skips install (fast!)
3. When `package-lock.json` changes → new cache key → fresh install

**What to cache:**
- `node_modules` (Node.js)
- `~/.cache/pip` (Python)
- `~/.m2/repository` (Maven/Java)
- Docker layers