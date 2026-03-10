# DevSecOps — Theory Questions

---

## Q1: What is DevSecOps?

**Answer:**
**DevSecOps** integrates security practices into every phase of the DevOps lifecycle, rather than treating security as a separate phase at the end. The goal is to make security everyone's responsibility and automate security checks.

**Traditional approach:**
```
Develop → Test → Deploy → Security Review (too late, expensive to fix)
```

**DevSecOps approach:**
```
Plan (threat modeling) → Code (SAST, secrets scanning) → Build (dependency scanning)
→ Test (DAST, penetration testing) → Deploy (image scanning, compliance)
→ Operate (runtime security, monitoring) → Monitor (incident response)
```

---

## Q2: What are the key security practices in each phase?

**Answer:**

| Phase | Security Practice | Tools |
|-------|------------------|-------|
| **Plan** | Threat modeling, security requirements | STRIDE, OWASP |
| **Code** | Static Application Security Testing (SAST) | SonarQube, Semgrep, CodeQL |
| **Code** | Secret scanning | gitleaks, trufflehog, git-secrets |
| **Code** | Pre-commit hooks | Husky + lint-staged |
| **Build** | Dependency scanning (SCA) | Snyk, Dependabot, npm audit |
| **Build** | Container image scanning | Trivy, Grype, Snyk Container |
| **Build** | License compliance | FOSSA, Black Duck |
| **Test** | Dynamic Application Security Testing (DAST) | OWASP ZAP, Burp Suite |
| **Test** | Infrastructure as Code scanning | Checkov, tfsec, Trivy |
| **Deploy** | Admission control | OPA Gatekeeper, Kyverno |
| **Deploy** | Image signing | Cosign, Notary |
| **Operate** | Runtime security | Falco, Aqua, Sysdig |
| **Monitor** | Security monitoring | SIEM, CloudTrail, audit logs |

---

## Q3: What is SAST vs. DAST?

**Answer:**

| Feature | SAST (Static) | DAST (Dynamic) |
|---------|--------------|----------------|
| **When** | During development (code review) | During testing (running application) |
| **What** | Analyzes source code | Tests the running application |
| **Finds** | Code-level vulnerabilities | Runtime vulnerabilities |
| **Examples** | SQL injection patterns, hardcoded secrets | XSS, authentication bypasses |
| **Speed** | Fast (no running app needed) | Slower (needs running app) |
| **False positives** | Higher | Lower |
| **Tools** | SonarQube, Semgrep, CodeQL | OWASP ZAP, Burp Suite |

**Best practice:** Use both. SAST catches issues early (shift left), DAST catches issues that only appear at runtime.

---

## Q4: What is Software Composition Analysis (SCA)?

**Answer:**
**SCA** scans your application's dependencies (third-party libraries) for known vulnerabilities.

**Why it matters:** Most applications are 80-90% third-party code. A vulnerability in a popular library (like Log4j) can affect millions of applications.

**Tools:**
- **Snyk** — Scans dependencies and suggests fixes
- **Dependabot** — GitHub's built-in dependency scanner
- **npm audit** — Built into npm
- **OWASP Dependency-Check** — Open-source SCA tool
- **JFrog Xray** — Scans artifacts in Artifactory

```bash
# npm audit
npm audit
npm audit fix

# Snyk
snyk test
snyk monitor    # Continuous monitoring
```

---

## Q5: What is the principle of "Shift Left" in security?

**Answer:**
**Shift Left** means moving security testing earlier in the development lifecycle (to the "left" of the timeline).

```
Traditional:  Code → Build → Test → Deploy → [Security]
Shift Left:   [Security] → Code → Build → Test → Deploy
                              ↑       ↑      ↑       ↑
                            SAST    SCA    DAST   Image scan
```

**Benefits:**
- **Cheaper** — Fixing a bug in development costs 6x less than in production
- **Faster** — Developers get immediate feedback
- **Better** — Security becomes part of the culture, not an afterthought

**How to implement:**
- IDE plugins that flag security issues as you code
- Pre-commit hooks that scan for secrets
- CI pipeline gates that block insecure code
- Security training for developers

---

## Q6: What is the OWASP Top 10?

**Answer:**
The **OWASP Top 10** is a list of the most critical web application security risks, updated periodically.

**2021 OWASP Top 10:**

| # | Risk | Description |
|---|------|-------------|
| 1 | **Broken Access Control** | Users accessing unauthorized resources |
| 2 | **Cryptographic Failures** | Weak encryption, exposed sensitive data |
| 3 | **Injection** | SQL injection, command injection, XSS |
| 4 | **Insecure Design** | Missing security controls in architecture |
| 5 | **Security Misconfiguration** | Default configs, unnecessary features enabled |
| 6 | **Vulnerable Components** | Using libraries with known vulnerabilities |
| 7 | **Authentication Failures** | Weak passwords, missing MFA |
| 8 | **Software Integrity Failures** | Untrusted updates, compromised CI/CD |
| 9 | **Logging Failures** | Insufficient logging and monitoring |
| 10 | **SSRF** | Server-Side Request Forgery |

---

## Q7: What is Zero Trust security?

**Answer:**
**Zero Trust** is a security model that assumes no user, device, or network should be trusted by default — even if they're inside the corporate network.

**Principles:**
- **Never trust, always verify** — Authenticate and authorize every request
- **Least privilege** — Give minimum access needed
- **Assume breach** — Design systems assuming attackers are already inside
- **Micro-segmentation** — Isolate workloads, don't rely on network perimeter

**Implementation in DevOps:**
- Use mutual TLS (mTLS) between services (service mesh)
- Implement RBAC in Kubernetes
- Use short-lived credentials (Vault dynamic secrets)
- Network policies to restrict pod-to-pod communication
- Continuous monitoring and anomaly detection