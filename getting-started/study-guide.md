# 📘 Study Guide — Recommended Learning Path

This guide tells you **what to study first, what comes next, and why** — so you build knowledge in the right order.

---

## 🗺️ The Learning Path (Follow This Order)

### Phase 1: Build the Foundation 

**1. Linux Basics & Shell Scripting**
- **Why first?** Almost everything in DevOps runs on Linux. You need to be comfortable with the command line, file system, permissions, and writing simple scripts before touching any tool.
- **What to focus on:** File navigation, permissions, process management, Bash scripting basics, cron jobs.

**2. Networking**
- **Why next?** Every DevOps tool communicates over a network. You need to understand how computers talk to each other — IP addresses, ports, DNS, HTTP, firewalls.
- **What to focus on:** TCP/IP basics, DNS, HTTP/HTTPS, ports, load balancing concepts, SSH.

**3. Git (Version Control)**
- **Why here?** Git is how teams manage code. Every CI/CD pipeline starts with Git. You'll use it daily.
- **What to focus on:** Cloning, branching, merging, pull requests, resolving conflicts, Git workflows.

---

### Phase 2: Core DevOps Concepts 

**4. CI/CD Concepts**
- **Why now?** CI/CD is the backbone of DevOps. Before learning specific tools (Jenkins, GitHub Actions), understand the *idea* — automatically building, testing, and deploying code.
- **What to focus on:** What CI and CD mean, pipeline stages, automated testing, deployment strategies.

**5. Infrastructure as Code (IaC) Concepts**
- **Why?** Instead of manually setting up servers, you write code to do it. This concept is essential before learning Terraform.
- **What to focus on:** Why IaC matters, declarative vs. imperative, state management, idempotency (running the same thing twice gives the same result).

**6. Configuration Management Concepts**
- **Why?** Once servers exist, you need to configure them consistently. This prepares you for Ansible.
- **What to focus on:** Why manual config is risky, desired state, push vs. pull models.

**7. Monitoring & Logging Concepts**
- **Why?** You can't fix what you can't see. Understanding observability prepares you for Prometheus, Grafana, and ELK.
- **What to focus on:** Metrics vs. logs vs. traces, alerting, dashboards, the "three pillars of observability."

**8. Containerization Concepts**
- **Why?** Containers changed how we deploy software. Understand the concept before diving into Docker.
- **What to focus on:** What containers are (vs. virtual machines), images, isolation, portability.

**9. Orchestration Concepts**
- **Why?** When you have many containers, you need something to manage them. This prepares you for Kubernetes.
- **What to focus on:** Why orchestration is needed, scaling, self-healing, service discovery.

---

### Phase 3: Essential Tools 

**10. Jenkins**
- **Why?** One of the most widely used CI/CD tools. Many companies still use it. Learning Jenkins teaches you pipeline thinking.
- **What to focus on:** Jenkinsfile, pipeline stages, plugins, agents, triggers.

**11. GitHub Actions**
- **Why?** The modern, cloud-native CI/CD tool. Increasingly popular and tightly integrated with GitHub.
- **What to focus on:** Workflows, jobs, steps, actions marketplace, secrets, matrix builds.

**12. Docker**
- **Why?** The standard for containerization. You'll use Docker every day in DevOps.
- **What to focus on:** Dockerfile, images, containers, volumes, networking, Docker Compose, multi-stage builds.

**13. Kubernetes**
- **Why?** The industry standard for container orchestration. It's complex but essential.
- **What to focus on:** Pods, Deployments, Services, ConfigMaps, Secrets, Ingress, namespaces, kubectl commands.

**14. Terraform**
- **Why?** The most popular IaC tool. Used to create and manage cloud infrastructure.
- **What to focus on:** HCL syntax, providers, resources, state files, modules, plan/apply workflow.

**15. Ansible**
- **Why?** The go-to tool for configuration management. Agentless (no software needed on target machines) and easy to learn.
- **What to focus on:** Playbooks, inventory, modules, roles, idempotency, Ansible Vault.

**16. Prometheus & Grafana**
- **Why?** The standard monitoring stack. Prometheus collects metrics; Grafana visualizes them.
- **What to focus on:** PromQL, exporters, alerting rules, Grafana dashboards, data sources.

**17. ELK Stack**
- **Why?** The standard logging stack. Elasticsearch stores logs, Logstash processes them, Kibana visualizes them.
- **What to focus on:** Log pipelines, index patterns, Kibana queries, Filebeat.

**18. AWS**
- **Why?** The largest cloud provider. Most DevOps roles require AWS knowledge.
- **What to focus on:** EC2, S3, IAM, VPC, RDS, Lambda, CloudFormation, ECS/EKS, CloudWatch.

---

### Phase 4: Advanced Tools 

**19. ArgoCD (GitOps)**
- **Why?** GitOps is the modern way to deploy to Kubernetes — your Git repo is the single source of truth.
- **What to focus on:** GitOps principles, Application CRDs, sync policies, rollbacks.

**20. Helm**
- **Why?** Kubernetes manifests get repetitive. Helm packages them into reusable charts.
- **What to focus on:** Charts, values files, templates, repositories, upgrades, rollbacks.

**21. Trivy**
- **Why?** Security scanning is now part of every pipeline. Trivy scans containers and code for vulnerabilities.
- **What to focus on:** Image scanning, filesystem scanning, CI integration, severity levels.

**22. HashiCorp Vault**
- **Why?** Secrets (passwords, API keys) need to be managed securely. Vault is the industry standard.
- **What to focus on:** Secret engines, policies, authentication methods, dynamic secrets, encryption as a service.

**23. SonarQube**
- **Why?** Code quality matters. SonarQube finds bugs, vulnerabilities, and code smells automatically.
- **What to focus on:** Quality gates, rules, profiles, CI integration, technical debt.

**24. JFrog Artifactory**
- **Why?** You need a place to store build artifacts (Docker images, JARs, npm packages). Artifactory is a universal repository manager.
- **What to focus on:** Repository types, artifact promotion, CI integration, cleanup policies.

---

### Phase 5: Advanced Topics 

**25. DevSecOps & Security**
- Integrating security into every stage of the DevOps pipeline.

**26. Service Mesh**
- Managing communication between microservices (Istio, Linkerd).

**27. Chaos Engineering**
- Intentionally breaking things to build more resilient systems.

---

## 💡 Tips for Following This Path

| Tip | Why It Helps |
|-----|-------------|
| Don't skip the foundations | Tools change; concepts don't. |
| Practice with real projects | Reading isn't enough — build something. |
| Learn one tool at a time | Trying to learn everything at once leads to confusion. |
| Revisit earlier topics | You'll understand Linux better after learning Docker. |
| Join communities | Reddit r/devops, DevOps Discord servers, and LinkedIn groups. |

---

## ⏱️ Realistic Timeline

| Level | Time Needed | What You'll Know |
|-------|-------------|-----------------|
| **Beginner** | 6–8 weeks | Linux, Git, CI/CD basics, Docker basics |
| **Intermediate** | 12–16 weeks | Jenkins/GH Actions, Kubernetes, Terraform, Ansible |
| **Advanced** | 20+ weeks | Full monitoring stack, security tools, GitOps, cloud |

> **Remember:** It's better to deeply understand 5 tools than to superficially know 15. Interviewers can tell the difference.