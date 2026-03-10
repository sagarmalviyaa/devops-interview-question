# Terraform — Theory Questions

---

## Q1: What is Terraform?

**Answer:**
**Terraform** is an open-source Infrastructure as Code (IaC) tool by HashiCorp. It lets you define cloud infrastructure in code files and create/manage it automatically.

**Key features:**
- **Declarative** — You describe what you want, Terraform figures out how
- **Multi-cloud** — Works with AWS, Azure, GCP, and 3,000+ providers
- **State management** — Tracks what infrastructure exists
- **Plan before apply** — Preview changes before making them
- **Modular** — Reusable components (modules)

---

## Q2: What is HCL? Show the basic syntax.

**Answer:**
**HCL (HashiCorp Configuration Language)** is Terraform's configuration language.

```hcl
# Provider — which cloud to use
provider "aws" {
  region = "us-east-1"
}

# Resource — what to create
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.medium"
  
  tags = {
    Name        = "web-server"
    Environment = "production"
  }
}

# Variable — input parameter
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t3.medium"
}

# Output — display values after apply
output "server_ip" {
  value = aws_instance.web.public_ip
}

# Data source — read existing resources
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-*-amd64-server-*"]
  }
}

# Local values — computed values
locals {
  common_tags = {
    Project     = "myapp"
    ManagedBy   = "terraform"
  }
}
```

---

## Q3: What is the Terraform workflow?

**Answer:**

```bash
# 1. Initialize — download providers and modules
terraform init

# 2. Plan — preview what will change
terraform plan
# Shows: + create, ~ update, - destroy

# 3. Apply — make the changes
terraform apply
# Type "yes" to confirm

# 4. Destroy — remove all resources (when done)
terraform destroy
```

**In CI/CD:**
```bash
terraform init
terraform plan -out=tfplan     # Save the plan
terraform apply tfplan          # Apply the exact plan (no prompt)
```

---

## Q4: What is Terraform state?

**Answer:**
**State** is a JSON file (`terraform.tfstate`) that maps your Terraform code to real infrastructure. It's Terraform's memory of what exists.

**Why state matters:**
- Knows what resources exist (to avoid duplicates)
- Tracks resource metadata (IDs, IPs)
- Enables accurate planning (what needs to change)
- Improves performance (doesn't query cloud for every plan)

**Remote state (required for teams):**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**Best practices:**
- Never store state locally in production
- Enable state locking (DynamoDB for S3)
- Enable encryption
- Enable versioning for rollback
- Never edit state manually

---

## Q5: What are Terraform modules?

**Answer:**
**Modules** are reusable packages of Terraform code. They let you create infrastructure patterns once and use them many times.

```hcl
# modules/vpc/main.tf
variable "cidr_block" {}
variable "environment" {}

resource "aws_vpc" "main" {
  cidr_block = var.cidr_block
  tags = { Name = "${var.environment}-vpc" }
}

output "vpc_id" {
  value = aws_vpc.main.id
}
```

```hcl
# Using the module
module "prod_vpc" {
  source      = "./modules/vpc"
  cidr_block  = "10.0.0.0/16"
  environment = "production"
}

module "staging_vpc" {
  source      = "./modules/vpc"
  cidr_block  = "10.1.0.0/16"
  environment = "staging"
}
```

**Module sources:** Local paths, Git repos, Terraform Registry, S3 buckets.

---

## Q6: What are Terraform providers?

**Answer:**
**Providers** are plugins that let Terraform interact with cloud platforms and services.

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

**Popular providers:** AWS, Azure, GCP, Kubernetes, Docker, GitHub, Datadog, Cloudflare, Helm

---

## Q7: What are `terraform plan` output symbols?

**Answer:**

| Symbol | Meaning |
|--------|---------|
| `+` | Resource will be **created** |
| `-` | Resource will be **destroyed** |
| `~` | Resource will be **updated in-place** |
| `-/+` | Resource will be **destroyed and recreated** |
| `<=` | Data source will be **read** |

```
# aws_instance.web will be created
+ resource "aws_instance" "web" {
    + ami           = "ami-12345"
    + instance_type = "t3.medium"
  }

# aws_instance.old will be destroyed
- resource "aws_instance" "old" {
    - ami           = "ami-old"
  }
```

---

## Q8: What is `terraform import`?

**Answer:**
`terraform import` brings existing infrastructure (created manually or by other tools) under Terraform management.

```bash
# Step 1: Write the Terraform code for the resource
resource "aws_instance" "web" {
  # Configuration will be filled after import
}

# Step 2: Import the existing resource
terraform import aws_instance.web i-1234567890abcdef0

# Step 3: Run terraform plan to see if code matches reality
terraform plan
# Adjust code until plan shows no changes
```

**Use cases:** Migrating from manual management to IaC, adopting Terraform for existing infrastructure.

---

## Q9: What are Terraform workspaces?

**Answer:**
**Workspaces** let you manage multiple environments with the same code but separate state files.

```bash
terraform workspace new staging
terraform workspace new production
terraform workspace list
terraform workspace select staging
```

```hcl
# Use workspace name in configuration
resource "aws_instance" "web" {
  instance_type = terraform.workspace == "production" ? "t3.large" : "t3.small"
  tags = {
    Environment = terraform.workspace
  }
}
```

**Alternative:** Many teams prefer separate directories or Terragrunt instead of workspaces for better isolation.

---

## Q10: What is the difference between `count` and `for_each`?

**Answer:**

**`count`** — Create multiple copies by number:
```hcl
resource "aws_instance" "web" {
  count         = 3
  ami           = "ami-12345"
  instance_type = "t3.medium"
  tags = { Name = "web-${count.index}" }
}
# Creates: web-0, web-1, web-2
```

**`for_each`** — Create copies from a map or set:
```hcl
resource "aws_instance" "web" {
  for_each      = toset(["web1", "web2", "api1"])
  ami           = "ami-12345"
  instance_type = "t3.medium"
  tags = { Name = each.key }
}
```

**`for_each` is preferred** because removing an item from the middle doesn't affect others. With `count`, removing item 1 of 3 causes items 2 and 3 to shift.

---

## Q11: What is a Terraform data source?

**Answer:**
A **data source** reads information from existing infrastructure without creating anything.

```hcl
# Read the latest Ubuntu AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-22.04-amd64-server-*"]
  }
}

# Use it in a resource
resource "aws_instance" "web" {
  ami = data.aws_ami.ubuntu.id
}
```

**Use cases:** Look up AMI IDs, VPC IDs, availability zones, existing resources.

---

## Q12: What are Terraform provisioners and why should you avoid them?

**Answer:**
**Provisioners** run scripts on resources after creation (like installing software).

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t3.medium"

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
    ]
  }
}
```

**Why avoid them:**
- They break the declarative model (Terraform can't track what they did)
- They're not idempotent (running twice might cause issues)
- They make `terraform plan` unreliable
- If they fail, the resource is "tainted"

**Better alternatives:**
- Use **Ansible** for configuration management
- Use **Packer** to build pre-configured images
- Use **cloud-init** / **user_data** for initial setup
- Use **Docker** containers (pre-built images)