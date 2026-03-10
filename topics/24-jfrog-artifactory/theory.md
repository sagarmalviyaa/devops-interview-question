# JFrog Artifactory — Theory Questions

---

## Q1: What is JFrog Artifactory?

**Answer:**
**JFrog Artifactory** is a universal artifact repository manager. It stores, manages, and distributes build artifacts (Docker images, npm packages, JAR files, Python packages, etc.) used in your CI/CD pipeline.

**Why you need it:**
- **Single source of truth** for all artifacts
- **Caching** — Proxy and cache remote packages (npm, Maven Central) for faster builds
- **Security** — Scan artifacts for vulnerabilities
- **Promotion** — Move artifacts through environments (dev → staging → prod)
- **Cleanup** — Automated policies to remove old artifacts

---

## Q2: What are the repository types in Artifactory?

**Answer:**

| Type | Description | Example |
|------|-------------|---------|
| **Local** | Store your own artifacts | Your Docker images, your npm packages |
| **Remote** | Proxy and cache external repositories | Cache from Docker Hub, npm registry |
| **Virtual** | Combine local and remote repos into one URL | Single URL for all npm packages |

```
Developer → Virtual Repo (npm-all)
              ├── Local Repo (npm-internal)    → Your packages
              └── Remote Repo (npm-remote)     → Cached from npmjs.org
```

**Benefits of this setup:**
- Developers use one URL for all packages
- External packages are cached (faster, available even if npm is down)
- Internal packages are served from the same URL

---

## Q3: What is artifact promotion?

**Answer:**
**Promotion** is moving an artifact through stages (dev → staging → production) without rebuilding it.

```
Build → npm-dev-local → (promote) → npm-staging-local → (promote) → npm-prod-local
```

**Why promote instead of rebuild:**
- **Consistency** — Exact same artifact in all environments
- **Speed** — No need to rebuild
- **Traceability** — Track which build is in which environment
- **Security** — Only tested artifacts reach production

---

## Q4: How do you integrate Artifactory with CI/CD?

**Answer:**

**Docker example:**
```bash
# Login to Artifactory Docker registry
docker login mycompany.jfrog.io

# Build and push
docker build -t mycompany.jfrog.io/docker-local/myapp:1.0 .
docker push mycompany.jfrog.io/docker-local/myapp:1.0

# Pull
docker pull mycompany.jfrog.io/docker-local/myapp:1.0
```

**npm example:**
```bash
# Configure npm to use Artifactory
npm config set registry https://mycompany.jfrog.io/artifactory/api/npm/npm-virtual/

# Publish your package
npm publish --registry https://mycompany.jfrog.io/artifactory/api/npm/npm-local/
```

---

## Q5: What is the difference between Artifactory and Docker Hub / ECR?

**Answer:**

| Feature | Artifactory | Docker Hub / ECR |
|---------|------------|-----------------|
| **Package types** | Universal (Docker, npm, Maven, PyPI, etc.) | Docker images only |
| **Caching** | Proxy and cache any remote registry | No (or limited) |
| **Promotion** | Built-in artifact promotion | Manual tagging |
| **Security scanning** | Integrated (with Xray) | Basic (or separate tool) |
| **Metadata** | Rich metadata and properties | Limited |
| **Cleanup policies** | Advanced automated cleanup | Basic lifecycle policies |
| **Cost** | Paid (free tier available) | Free tier + paid |

**Choose Artifactory when:** You have multiple package types, need caching, want artifact promotion, or need advanced security scanning.
**Choose Docker Hub/ECR when:** You only need Docker images and want simplicity.