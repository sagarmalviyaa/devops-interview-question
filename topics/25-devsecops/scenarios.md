# DevSecOps — Scenario-Based Questions

---

## S1: You're tasked with implementing a security pipeline for a new project. What does it look like?

**Answer:**

```yaml
# .github/workflows/security-pipeline.yml
name: Security Pipeline

on: [push, pull_request]

jobs:
  secret-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Scan for secrets
        uses: gitleaks/gitleaks-action@v2

  sast:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: SonarQube Scan
        uses: SonarSource/sonarqube-scan-action@master
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

  dependency-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - name: Snyk dependency scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

  image-scan:
    needs: [secret-scan, sast, dependency-scan]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build image
        run: docker build -t myapp:${{ github.sha }} .
      - name: Trivy image scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: myapp:${{ github.sha }}
          exit-code: '1'
          severity: 'CRITICAL,HIGH'

  iac-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Checkov IaC scan
        uses: bridgecrewio/checkov-action@master
        with:
          directory: terraform/

  dast:
    needs: image-scan
    runs-on: ubuntu-latest
    steps:
      - name: OWASP ZAP scan
        uses: zaproxy/action-full-scan@v0.10.0
        with:
          target: 'https://staging.myapp.com'
```

**Pipeline flow:**
1. **Secret scanning** — Catch leaked credentials
2. **SAST** — Find code vulnerabilities
3. **Dependency scanning** — Find vulnerable libraries
4. **Image scanning** — Find OS/package vulnerabilities
5. **IaC scanning** — Find infrastructure misconfigurations
6. **DAST** — Find runtime vulnerabilities

---

## S2: A developer committed an AWS access key to a public GitHub repository. What do you do?

**Answer:**

**Immediate (within minutes):**
1. **Revoke the AWS key immediately:**
   ```bash
   aws iam delete-access-key --user-name developer --access-key-id AKIAEXAMPLE
   ```
2. **Check CloudTrail** for any unauthorized usage of the key
3. **Notify the security team**

**Investigation:**
4. **Check what the key had access to** (IAM policies attached)
5. **Review CloudTrail logs** for suspicious activity:
   - New resources created?
   - Data accessed or exfiltrated?
   - New IAM users or keys created?
6. **Check for cryptocurrency mining** instances (common attack)

**Remediation:**
7. **Remove the commit from Git history:**
   ```bash
   # Use BFG Repo-Cleaner
   bfg --replace-text passwords.txt repo.git
   git push --force
   ```
8. **Generate new credentials** and distribute securely

**Prevention:**
9. **Pre-commit hooks** with gitleaks:
   ```bash
   # .pre-commit-config.yaml
   repos:
   - repo: https://github.com/gitleaks/gitleaks
     hooks:
     - id: gitleaks
   ```
10. **CI pipeline scanning** with gitleaks or trufflehog
11. **Use IAM roles** instead of access keys
12. **Use short-lived credentials** from Vault
13. **Enable GitHub secret scanning** (alerts on pushed secrets)
14. **Security training** for the team