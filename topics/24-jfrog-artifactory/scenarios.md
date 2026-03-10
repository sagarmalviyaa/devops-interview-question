# JFrog Artifactory — Scenario-Based Questions

---

## S1: Your CI/CD builds are slow because they download dependencies from the internet every time. How do you fix this with Artifactory?

**Answer:**

1. **Set up remote repositories** to proxy and cache external registries:
   - `npm-remote` → proxies npmjs.org
   - `docker-remote` → proxies Docker Hub
   - `maven-remote` → proxies Maven Central

2. **Create virtual repositories** that combine local and remote:
   - `npm-virtual` = `npm-local` + `npm-remote`

3. **Configure build tools to use Artifactory:**
   ```bash
   # npm
   npm config set registry https://artifactory.example.com/api/npm/npm-virtual/
   
   # Docker
   docker pull artifactory.example.com/docker-virtual/nginx:latest
   
   # Maven (settings.xml)
   <mirror>
     <id>artifactory</id>
     <url>https://artifactory.example.com/maven-virtual/</url>
     <mirrorOf>*</mirrorOf>
   </mirror>
   ```

4. **Results:**
   - First build: Downloads from internet, caches in Artifactory
   - Subsequent builds: Served from Artifactory cache (much faster)
   - If npm/Docker Hub goes down, builds still work from cache

---

## S2: You need to ensure only approved artifacts are deployed to production. How?

**Answer:**

1. **Use separate repositories per environment:**
   - `docker-dev` — All builds go here
   - `docker-staging` — Promoted after testing
   - `docker-prod` — Promoted after staging approval

2. **Integrate JFrog Xray** for security scanning:
   - Scan all artifacts for vulnerabilities
   - Block promotion if critical vulnerabilities are found
   - Create policies that define acceptable risk levels

3. **Set up promotion pipelines:**
   ```
   Build → Push to dev repo → Run tests → Xray scan
     → If pass: Promote to staging
     → Manual approval → Promote to production
   ```

4. **Use Artifactory permissions:**
   - Developers can push to dev repos only
   - CI/CD can promote to staging
   - Only approved pipelines (with manual gate) can promote to production

5. **Enable immutable artifacts** in production repos:
   - Once an artifact is in production, it can't be overwritten
   - Ensures what was tested is exactly what runs in production