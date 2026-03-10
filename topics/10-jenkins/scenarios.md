# Jenkins — Scenario-Based Questions

---

## S1: Your Jenkins server is running out of disk space and builds are failing. How do you fix it?

**Answer:**

1. **Identify what's using space:**
   ```bash
   du -sh /var/lib/jenkins/*  | sort -rh | head -10
   ```

2. **Common space consumers:**
   - **Old build artifacts and logs** — `/var/lib/jenkins/jobs/*/builds/`
   - **Workspace files** — `/var/lib/jenkins/workspace/`
   - **Docker images** (if building Docker on Jenkins)

3. **Immediate fixes:**
   ```bash
   # Clean old workspaces
   # In Jenkins UI: Manage Jenkins → Script Console
   Jenkins.instance.getAllItems(Job).each { job ->
       job.builds.each { build ->
           if (build.number < job.lastBuild.number - 10) {
               build.delete()
           }
       }
   }
   
   # Clean Docker (if applicable)
   docker system prune -af
   ```

4. **Long-term prevention:**
   - **Set build retention policies** in each job:
     ```groovy
     options {
         buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '30'))
     }
     ```
   - **Clean workspace after build:**
     ```groovy
     post {
         always {
             cleanWs()
         }
     }
     ```
   - **Store artifacts externally** (S3, Artifactory) instead of on Jenkins
   - **Monitor disk space** with alerts

---

## S2: A Jenkins pipeline works locally but fails in production with "permission denied" errors. How do you debug?

**Answer:**

1. **Check which user Jenkins runs as:**
   ```bash
   # On the Jenkins agent
   whoami    # Usually 'jenkins'
   id        # Shows user ID and groups
   ```

2. **Common permission issues:**
   - Jenkins user can't access Docker socket:
     ```bash
     sudo usermod -aG docker jenkins
     sudo systemctl restart jenkins
     ```
   - Jenkins user can't write to a directory:
     ```bash
     sudo chown -R jenkins:jenkins /path/to/directory
     ```
   - SSH key permissions are wrong:
     ```bash
     chmod 600 /var/lib/jenkins/.ssh/id_rsa
     chmod 700 /var/lib/jenkins/.ssh/
     ```

3. **Check credentials:**
   - Are the credentials configured in Jenkins' credential store?
   - Is the credential ID correct in the Jenkinsfile?
   - Has the credential expired?

4. **Check agent vs. controller:**
   - The pipeline might be running on a different agent than expected
   - Use `agent { label 'specific-agent' }` to ensure the right agent

---

## S3: You need to migrate from Jenkins to GitHub Actions. What's your approach?

**Answer:**

1. **Inventory current Jenkins setup:**
   - List all jobs and their configurations
   - Document plugins used and their purposes
   - Map out credential usage
   - Identify shared libraries

2. **Map Jenkins concepts to GitHub Actions:**

   | Jenkins | GitHub Actions |
   |---------|---------------|
   | Jenkinsfile | `.github/workflows/*.yml` |
   | Pipeline stages | Jobs and steps |
   | Agents | Runners (GitHub-hosted or self-hosted) |
   | Plugins | Actions (from marketplace) |
   | Shared libraries | Reusable workflows, composite actions |
   | Credentials | GitHub Secrets |
   | Build triggers | Workflow triggers (`on:`) |

3. **Migrate incrementally:**
   - Start with the simplest pipelines
   - Run both systems in parallel during migration
   - Migrate one team/project at a time
   - Test thoroughly before decommissioning Jenkins

4. **Example conversion:**
   ```groovy
   // Jenkins
   pipeline {
       agent { docker { image 'node:20' } }
       stages {
           stage('Test') {
               steps { sh 'npm test' }
           }
       }
   }
   ```
   ```yaml
   # GitHub Actions equivalent
   name: CI
   on: [push, pull_request]
   jobs:
     test:
       runs-on: ubuntu-latest
       container: node:20
       steps:
         - uses: actions/checkout@v4
         - run: npm test
   ```