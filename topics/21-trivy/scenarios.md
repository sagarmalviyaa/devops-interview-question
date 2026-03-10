# Trivy — Scenario-Based Questions

---

## S1: Trivy found 50 HIGH vulnerabilities in your production Docker image. How do you prioritize and fix them?

**Answer:**

1. **Triage the results:**
   ```bash
   trivy image --severity HIGH,CRITICAL --format json myapp:latest | \
     jq '.Results[].Vulnerabilities[] | {id: .VulnerabilityID, pkg: .PkgName, fix: .FixedVersion}'
   ```

2. **Prioritize by:**
   - **CRITICAL first** — These have known exploits
   - **Fixable vulnerabilities** — Focus on ones with available patches
   - **Exploitability** — Is the vulnerable code actually reachable?
   - **Network exposure** — Is the vulnerable component internet-facing?

3. **Fix strategies:**
   - **Update base image:** `FROM node:20.11-alpine` (latest patched version)
   - **Update packages:** `RUN apt-get update && apt-get upgrade -y`
   - **Update dependencies:** `npm audit fix` or `pip install --upgrade`
   - **Use multi-stage builds** to remove build-time dependencies from the final image

4. **For unfixable vulnerabilities:**
   - Accept the risk if the component isn't exposed
   - Add to `.trivyignore` with justification:
     ```
     # CVE-2024-XXXX: Not exploitable in our context, no fix available
     CVE-2024-XXXX
     ```

5. **Prevent future issues:**
   - Add Trivy to CI/CD pipeline as a gate
   - Set up automated base image updates
   - Regularly scan running containers in production