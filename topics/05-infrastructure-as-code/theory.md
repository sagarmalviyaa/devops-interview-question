# Infrastructure as Code (IaC) — Theory Questions

---

## Q1: What is Infrastructure as Code (IaC)?

**Answer:**
**Infrastructure as Code** means managing and provisioning infrastructure (servers, networks, databases, load balancers) through code files instead of manual processes (clicking in a web console or running ad-hoc commands).

**Think of it this way:** Instead of logging into AWS and clicking "Create Server," you write a file that says "I need a server with these specs" and a tool creates it for you.

**Benefits:**
- **Consistency** — Same code produces same infrastructure every time
- **Version control** — Track infrastructure changes in Git
- **Repeatability** — Create identical environments (dev, staging, production)
- **Speed** — Spin up entire environments in minutes
- **Documentation** — The code IS the documentation of your infrastructure
- **Collaboration** — Team members can review infrastructure changes via pull requests

---

## Q2: What is the difference between declarative and imperative IaC?

**Answer:**

**Declarative (What):** You describe the desired end state, and the tool figures out how to get there.
```hcl
# Terraform (declarative) — "I want 3 servers"
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-12345"
  instance_type = "t3.medium"
}
```

**Imperative (How):** You write step-by-step instructions for what to do.
```python
# Script (imperative) — "Create 3 servers one by one"
for i in range(3):
    create_server(ami="ami-12345", type="t3.medium")
```

| Feature | Declarative | Imperative |
|---------|------------|-----------|
| You define | Desired state | Steps to execute |
| Examples | Terraform, CloudFormation, Kubernetes YAML | Bash scripts, AWS CLI, Pulumi (can be) |
| Idempotent? | Yes (by design) | You must handle it yourself |
| Easier to | Understand current state | Handle complex logic |

**Most modern IaC tools are declarative** because it's safer and more predictable.

---

## Q3: What is state in IaC? Why is it important?

**Answer:**
**State** is a record of what infrastructure currently exists. IaC tools use state to know what's already created, so they can determine what needs to change.

**Terraform example:**
- Terraform stores state in a `terraform.tfstate` file
- When you run `terraform plan`, it compares your code to the state to determine what to create, update, or delete

**Why state matters:**
- Without state, the tool doesn't know what already exists
- It prevents duplicate resource creation
- It enables accurate planning (showing what will change)
- It tracks resource metadata (IDs, IPs, etc.)

**State management best practices:**
- **Never store state locally** in production — use remote state (S3, Terraform Cloud)
- **Enable state locking** — prevent two people from modifying state simultaneously
- **Encrypt state** — it may contain sensitive data (passwords, IPs)
- **Never edit state manually** — use `terraform state` commands

---

## Q4: What is idempotency in IaC?

**Answer:**
**Idempotency** means running the same IaC code multiple times produces the same result as running it once. No matter how many times you apply, you get the same infrastructure.

**Example:**
```hcl
# Running "terraform apply" with this code:
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t3.medium"
}

# First run: Creates 1 server
# Second run: No changes (server already exists)
# Third run: No changes (still the same)
```

**Why it matters:**
- Safe to re-run after failures (just run again)
- CI/CD pipelines can apply IaC on every deployment
- No risk of creating duplicate resources

---

## Q5: What is drift in IaC? How do you detect and fix it?

**Answer:**
**Drift** happens when the actual infrastructure no longer matches what's defined in your IaC code. Someone made a manual change (clicked something in the console) that the code doesn't know about.

**Example:** Your Terraform code says the server should be `t3.medium`, but someone manually changed it to `t3.large` in the AWS console.

**How to detect drift:**
```bash
# Terraform
terraform plan
# Shows differences between code and actual infrastructure

# AWS CloudFormation
aws cloudformation detect-stack-drift --stack-name mystack
```

**How to fix drift:**
1. **Option A:** Update the code to match reality (if the manual change was intentional)
2. **Option B:** Re-apply the code to revert the manual change
   ```bash
   terraform apply  # Reverts the server back to t3.medium
   ```

**How to prevent drift:**
- Never make manual changes — always change the code
- Set up drift detection on a schedule
- Use read-only access for most team members (only CI/CD can modify)
- Enable alerts when drift is detected

---

## Q6: What are the most popular IaC tools and when would you use each?

**Answer:**

| Tool | Type | Best For | Language |
|------|------|----------|----------|
| **Terraform** | Provisioning | Multi-cloud infrastructure | HCL |
| **AWS CloudFormation** | Provisioning | AWS-only infrastructure | JSON/YAML |
| **Pulumi** | Provisioning | Teams who prefer real programming languages | Python, TypeScript, Go |
| **Ansible** | Configuration | Configuring servers after creation | YAML |
| **AWS CDK** | Provisioning | AWS infrastructure with programming languages | TypeScript, Python |
| **OpenTofu** | Provisioning | Open-source Terraform alternative | HCL |
| **Crossplane** | Provisioning | Kubernetes-native infrastructure management | YAML |

**Common combination:** Terraform (create servers) + Ansible (configure servers)

---

## Q7: What is the difference between provisioning and configuration management?

**Answer:**

**Provisioning** — Creating the infrastructure itself:
- Servers, networks, databases, load balancers, DNS records
- Tools: Terraform, CloudFormation, Pulumi
- Analogy: Building a house (foundation, walls, roof)

**Configuration Management** — Setting up software on existing infrastructure:
- Installing packages, configuring services, managing files
- Tools: Ansible, Chef, Puppet, Salt
- Analogy: Furnishing a house (furniture, appliances, decorations)

**Example workflow:**
1. Terraform creates 3 EC2 instances
2. Ansible installs Nginx, configures firewall, deploys application code on those instances

---

## Q8: What are IaC modules and why are they important?

**Answer:**
**Modules** are reusable, self-contained packages of IaC code. Instead of copying the same code everywhere, you create a module once and use it many times.

**Terraform module example:**
```hcl
# modules/web-server/main.tf
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = var.instance_type
  tags = {
    Name = var.name
  }
}

# Using the module
module "web_prod" {
  source        = "./modules/web-server"
  ami           = "ami-12345"
  instance_type = "t3.large"
  name          = "production-web"
}

module "web_staging" {
  source        = "./modules/web-server"
  ami           = "ami-12345"
  instance_type = "t3.small"
  name          = "staging-web"
}
```

**Benefits:**
- **DRY (Don't Repeat Yourself)** — Write once, use everywhere
- **Consistency** — All environments use the same patterns
- **Easier maintenance** — Update the module, all users get the update
- **Abstraction** — Hide complexity behind a simple interface