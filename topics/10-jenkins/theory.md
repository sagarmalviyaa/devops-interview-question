# Jenkins — Theory Questions

---

## Q1: What is Jenkins?

**Answer:**
**Jenkins** is an open-source automation server used primarily for CI/CD (Continuous Integration and Continuous Delivery). It automates building, testing, and deploying software.

**Key features:**
- **Pipeline as code** — Define pipelines in a `Jenkinsfile`
- **Plugins** — Over 1,800 plugins for integrating with virtually any tool
- **Distributed builds** — Run builds on multiple machines (agents)
- **Extensible** — Highly customizable through plugins and scripts
- **Free and open-source**

---

## Q2: What is a Jenkinsfile? What are the two types of pipeline syntax?

**Answer:**
A **Jenkinsfile** is a text file that defines a Jenkins pipeline. It lives in your Git repository alongside your code.

**Declarative Pipeline (recommended — simpler):**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        stage('Deploy') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
    post {
        failure {
            mail to: 'team@example.com', subject: 'Build Failed'
        }
    }
}
```

**Scripted Pipeline (more flexible — uses Groovy):**
```groovy
node {
    stage('Build') {
        sh 'npm install'
        sh 'npm run build'
    }
    stage('Test') {
        sh 'npm test'
    }
}
```

**Declarative is preferred** because it's easier to read, has built-in error handling, and enforces structure.

---

## Q3: What is a Jenkins agent (node)?

**Answer:**
An **agent** (also called a node or slave) is a machine that Jenkins uses to run builds. The main Jenkins server is called the **controller** (or master).

**Why use agents?**
- **Distribute load** — Don't overload the controller
- **Different environments** — Run builds on Linux, Windows, or Mac
- **Parallel builds** — Run multiple builds simultaneously
- **Security** — Keep build tools off the controller

**Agent types:**
- **Permanent agents** — Always-on machines connected to Jenkins
- **Cloud agents** — Spun up on demand (Docker, Kubernetes, AWS EC2)
- **Docker agents** — Run each build in a fresh Docker container

```groovy
pipeline {
    agent {
        docker {
            image 'node:20'   // Run in a Node.js Docker container
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'node --version'
                sh 'npm install'
            }
        }
    }
}
```

---

## Q4: What are Jenkins plugins? Name some essential ones.

**Answer:**
**Plugins** extend Jenkins' functionality. They integrate Jenkins with other tools and add features.

**Essential plugins:**

| Plugin | Purpose |
|--------|---------|
| **Pipeline** | Enables pipeline-as-code (Jenkinsfile) |
| **Git** | Integrates with Git repositories |
| **Docker Pipeline** | Build and use Docker containers in pipelines |
| **Kubernetes** | Run agents as Kubernetes pods |
| **Blue Ocean** | Modern UI for Jenkins pipelines |
| **Credentials** | Securely store passwords, tokens, SSH keys |
| **Email Extension** | Send build notifications |
| **SonarQube Scanner** | Code quality analysis |
| **Slack Notification** | Send alerts to Slack |
| **Role-Based Access** | Fine-grained user permissions |

---

## Q5: What are Jenkins triggers?

**Answer:**
Triggers define when a pipeline should run.

| Trigger | Description | Configuration |
|---------|-------------|---------------|
| **SCM Polling** | Check Git for changes periodically | `pollSCM('H/5 * * * *')` |
| **Webhook** | Git pushes trigger the build instantly | GitHub/GitLab webhook → Jenkins URL |
| **Cron** | Run on a schedule | `cron('0 2 * * *')` — daily at 2 AM |
| **Upstream** | Run after another job completes | `upstream('build-job')` |
| **Manual** | Human clicks "Build Now" | Default behavior |

```groovy
pipeline {
    triggers {
        pollSCM('H/5 * * * *')    // Check Git every 5 minutes
        cron('0 2 * * *')          // Also run daily at 2 AM
    }
    // ...
}
```

**Best practice:** Use webhooks instead of polling — they're instant and don't waste resources.

---

## Q6: What is a Jenkins shared library?

**Answer:**
A **shared library** is reusable pipeline code stored in a separate Git repository. Instead of copying the same pipeline steps across many Jenkinsfiles, you write them once in a library.

**Structure:**
```
shared-library/
├── vars/
│   ├── buildApp.groovy       # Global functions
│   └── deployToK8s.groovy
├── src/
│   └── com/company/Utils.groovy  # Helper classes
└── resources/
    └── templates/             # Config templates
```

**Using the library:**
```groovy
@Library('my-shared-library') _

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                buildApp()          // Calls vars/buildApp.groovy
            }
        }
        stage('Deploy') {
            steps {
                deployToK8s('production')
            }
        }
    }
}
```

**Benefits:** DRY code, consistent pipelines, centralized updates.

---

## Q7: How does Jenkins handle credentials and secrets?

**Answer:**
Jenkins has a built-in **Credentials Store** that securely stores sensitive data.

**Credential types:**
- Username and password
- SSH private key
- Secret text (API tokens)
- Secret file
- Certificate

**Using credentials in a pipeline:**
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-hub',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                    sh 'docker push myapp:latest'
                }
            }
        }
    }
}
```

**Best practices:**
- Never hardcode secrets in Jenkinsfiles
- Use credential scoping (folder-level or global)
- Rotate credentials regularly
- Integrate with external secret managers (Vault, AWS Secrets Manager)
- Mask secrets in console output

---

## Q8: What is the difference between Jenkins freestyle and pipeline jobs?

**Answer:**

| Feature | Freestyle Job | Pipeline Job |
|---------|--------------|-------------|
| Configuration | UI-based (click and configure) | Code-based (Jenkinsfile) |
| Version control | Not easily versioned | Stored in Git with your code |
| Complexity | Simple, linear | Supports complex workflows |
| Reusability | Limited | Shared libraries |
| Parallel stages | Not supported | Supported |
| Code review | Not possible | Via pull requests |

**Pipeline jobs are strongly preferred** because they follow "pipeline as code" principles.