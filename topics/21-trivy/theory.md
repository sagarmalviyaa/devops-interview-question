# Trivy — Theory Questions

---

## Q1: What is Trivy?

**Answer:**
**Trivy** is an open-source security scanner by Aqua Security. It finds vulnerabilities, misconfigurations, and secrets in containers, code, and infrastructure.

**What Trivy scans:**
- **Container images** — OS packages and application dependencies
- **Filesystem** — Source code and config files
- **Git repositories** — Remote repos
- **Kubernetes** — Cluster misconfigurations
- **IaC files** — Terraform, CloudFormation, Dockerfiles
- **SBOM** — Software Bill of Materials

---

## Q2: How do you scan a Docker image with Trivy?

**Answer:**

```bash
# Scan a local image
trivy image myapp:latest

# Scan a remote image
trivy image nginx:1.25

# Only show HIGH and CRITICAL vulnerabilities
trivy image --severity HIGH,CRITICAL myapp:latest

# Output as JSON (for CI/CD processing)
trivy image --format json --output results.json myapp:latest

# Fail the build if critical vulnerabilities found (CI/CD gate)
trivy image --exit-code 1 --severity CRITICAL myapp:latest

# Scan and ignore unfixed vulnerabilities
trivy image --ignore-unfixed myapp:latest

# Scan filesystem (source code)
trivy fs --security-checks vuln,secret,config .

# Scan Kubernetes cluster
trivy k8s --report summary cluster

# Scan Terraform files
trivy config ./terraform/
```

---

## Q3: How do you integrate Trivy into a CI/CD pipeline?

**Answer:**

**GitHub Actions:**
```yaml
- name: Run Trivy vulnerability scanner
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: 'myapp:${{ github.sha }}'
    format: 'sarif'
    output: 'trivy-results.sarif'
    severity: 'HIGH,CRITICAL'
    exit-code: '1'

- name: Upload Trivy scan results
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: 'trivy-results.sarif'
```

**Jenkins:**
```groovy
stage('Security Scan') {
    steps {
        sh 'trivy image --exit-code 1 --severity HIGH,CRITICAL myapp:latest'
    }
}
```

---

## Q4: What is the difference between Trivy and other scanning tools?

**Answer:**

| Feature | Trivy | Snyk | Grype | Clair |
|---------|-------|------|-------|-------|
| **Scope** | Images, FS, K8s, IaC | Images, code, deps | Images, FS | Images only |
| **Speed** | Very fast | Medium | Fast | Slow |
| **Cost** | Free (open-source) | Freemium | Free | Free |
| **Ease of use** | Very easy (single binary) | Easy (SaaS) | Easy | Complex |
| **IaC scanning** | Yes | Yes | No | No |
| **Secret detection** | Yes | No | No | No |

**Trivy is popular because** it's free, fast, easy to install, and covers many scan types in one tool.