# Configuration Management — Theory Questions

---

## Q1: What is Configuration Management?

**Answer:**
**Configuration Management** is the practice of keeping all your servers and systems configured consistently and automatically. Instead of manually logging into each server to install software and change settings, you write code that does it for you.

**Why it matters:**
- **Consistency** — All servers are configured the same way (no "snowflake servers")
- **Automation** — Configure 100 servers as easily as 1
- **Reproducibility** — Rebuild a server from scratch in minutes
- **Audit trail** — Track every configuration change in version control
- **Drift prevention** — Automatically fix servers that deviate from the desired state

---

## Q2: What is the difference between push and pull configuration management?

**Answer:**

**Push model:**
- A central server pushes configurations to target machines
- You initiate the process manually or via CI/CD
- Example: **Ansible** — you run a playbook and it connects to servers via SSH
- Pros: Simple, no agent needed on target machines
- Cons: Central server must be able to reach all targets

**Pull model:**
- Target machines periodically pull their configuration from a central server
- An agent runs on each target machine
- Example: **Puppet**, **Chef** — agents check in every 30 minutes
- Pros: Scales well, machines self-heal
- Cons: Requires agent installation, more complex setup

---

## Q3: What is "desired state" configuration?

**Answer:**
**Desired state** means you describe what the system should look like, not the steps to get there. The tool compares the current state to the desired state and makes only the necessary changes.

**Example (Ansible):**
```yaml
- name: Ensure Nginx is installed and running
  hosts: webservers
  tasks:
    - name: Install Nginx
      apt:
        name: nginx
        state: present    # Desired state: nginx should be installed

    - name: Start Nginx
      service:
        name: nginx
        state: started    # Desired state: nginx should be running
        enabled: yes      # Desired state: nginx should start on boot
```

**What happens:**
- First run: Installs Nginx and starts it
- Second run: Checks that Nginx is installed and running — no changes needed
- If someone stops Nginx manually: Next run starts it again (self-healing)

---

## Q4: What are the main configuration management tools and how do they compare?

**Answer:**

| Feature | Ansible | Puppet | Chef | Salt |
|---------|---------|--------|------|------|
| Language | YAML (playbooks) | Puppet DSL (Ruby-based) | Ruby | YAML/Python |
| Architecture | Agentless (SSH) | Agent-based | Agent-based | Agent or agentless |
| Push/Pull | Push | Pull | Pull | Both |
| Learning curve | Easy | Medium | Hard | Medium |
| Idempotent | Yes | Yes | Yes | Yes |
| Community | Very large | Large | Medium | Medium |

**Ansible is the most popular** in 2026 because:
- No agent needed (just SSH access)
- Easy to learn (YAML syntax)
- Large module library
- Works for both configuration management and orchestration

---

## Q5: What is the difference between Configuration Management and Infrastructure as Code?

**Answer:**

| Aspect | Configuration Management | Infrastructure as Code |
|--------|------------------------|----------------------|
| **What it does** | Configures existing servers | Creates/destroys infrastructure |
| **Scope** | Software, settings, files on servers | Servers, networks, databases, cloud resources |
| **Tools** | Ansible, Puppet, Chef | Terraform, CloudFormation, Pulumi |
| **Analogy** | Furnishing a house | Building a house |
| **Example** | Install Nginx, configure firewall | Create an EC2 instance, set up a VPC |

**They work together:**
1. Terraform creates 3 servers
2. Ansible installs and configures software on those servers

---

## Q6: What is immutable vs. mutable infrastructure?

**Answer:**

**Mutable infrastructure:**
- Servers are updated in place (install updates, change configs)
- The same server evolves over time
- Risk: Configuration drift, "works on my server" problems
- Tools: Ansible, Puppet, Chef

**Immutable infrastructure:**
- Servers are never modified after creation
- To make changes, you create a new server with the new configuration and destroy the old one
- Like replacing a light bulb instead of fixing it
- Tools: Docker, Packer, Kubernetes

**Example:**
```
Mutable:    Server v1 → update → Server v1.1 → update → Server v1.2
Immutable:  Server v1 → destroy → Server v2 (brand new)
```

**Modern trend:** Immutable infrastructure (using containers) is preferred because it's more predictable and eliminates drift.