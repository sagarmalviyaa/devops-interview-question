# Infrastructure as Code — Scenario-Based Questions

---

## S1: Two team members ran `terraform apply` at the same time and now the infrastructure is in a broken state. What happened and how do you prevent this?

**Answer:**

**What happened:**
- Both ran `terraform plan` at the same time and got the same plan
- Both tried to apply changes simultaneously
- This caused conflicting changes, resource conflicts, or a corrupted state file

**How to fix it:**
1. **Don't panic.** Check the current state:
   ```bash
   terraform state list    # See what resources exist
   terraform plan          # See what Terraform thinks needs to change
   ```
2. If state is corrupted, restore from a backup:
   ```bash
   # If using S3 backend with versioning
   aws s3api list-object-versions --bucket my-tf-state --prefix terraform.tfstate
   # Restore the previous version
   ```
3. Manually reconcile if needed:
   ```bash
   terraform import aws_instance.web i-1234567890  # Import existing resources
   terraform state rm aws_instance.duplicate        # Remove duplicates from state
   ```

**How to prevent this:**
1. **Enable state locking:**
   ```hcl
   terraform {
     backend "s3" {
       bucket         = "my-tf-state"
       key            = "prod/terraform.tfstate"
       region         = "us-east-1"
       dynamodb_table = "terraform-locks"  # This enables locking!
       encrypt        = true
     }
   }
   ```
2. **Use CI/CD for Terraform** — only the pipeline runs `terraform apply`, not individuals
3. **Use Terraform Cloud or Spacelift** — built-in locking and collaboration
4. **Enable state file versioning** in S3 for backup

---

## S2: Your company has 50 AWS accounts and you need to manage infrastructure consistently across all of them. How?

**Answer:**

1. **Use Terraform modules for standardization:**
   ```
   modules/
   ├── vpc/              # Standard VPC setup
   ├── security/         # Standard security groups
   ├── monitoring/       # Standard CloudWatch setup
   └── iam/              # Standard IAM roles
   ```

2. **Use a module registry** (Terraform Cloud or private registry) so all teams use approved modules

3. **Structure by account:**
   ```
   infrastructure/
   ├── modules/          # Shared modules
   ├── accounts/
   │   ├── prod-account-1/
   │   │   ├── main.tf
   │   │   └── terraform.tfvars
   │   ├── staging-account/
   │   └── dev-account/
   └── global/           # Cross-account resources
   ```

4. **Use Terraform workspaces or Terragrunt** for managing multiple environments:
   ```hcl
   # terragrunt.hcl
   terraform {
     source = "../../modules/vpc"
   }
   inputs = {
     cidr_block = "10.1.0.0/16"
     environment = "production"
   }
   ```

5. **Automate with CI/CD:**
   - Pull request → `terraform plan` (review changes)
   - Merge → `terraform apply` (apply changes)
   - Use policy-as-code (OPA, Sentinel) to enforce standards

6. **Use AWS Organizations** with Service Control Policies (SCPs) for guardrails

---

## S3: You need to migrate existing manually-created infrastructure to Terraform. How do you approach this?

**Answer:**

1. **Inventory existing resources:**
   ```bash
   # List all resources in AWS
   aws resourcegroupstaggingapi get-resources --region us-east-1
   ```

2. **Import resources into Terraform state:**
   ```bash
   # Write the Terraform code for the resource first
   # Then import the existing resource
   terraform import aws_instance.web i-1234567890
   terraform import aws_vpc.main vpc-abcdef123
   terraform import aws_s3_bucket.data my-bucket-name
   ```

3. **Use automated import tools:**
   ```bash
   # Terraformer — auto-generates Terraform code from existing infrastructure
   terraformer import aws --resources=ec2,vpc,s3 --regions=us-east-1
   ```

4. **Verify the import:**
   ```bash
   terraform plan
   # Should show "No changes" if the code matches reality
   # If it shows changes, adjust the code to match
   ```

5. **Migrate incrementally:**
   - Start with non-critical resources (dev environment)
   - Import one service at a time
   - Test thoroughly before moving to production
   - Don't try to import everything at once

6. **Best practices:**
   - Create a migration plan and timeline
   - Tag resources as "managed by Terraform" after import
   - Set up CI/CD for Terraform before migrating production
   - Keep a rollback plan (you can always remove from state without destroying)

---

## S4: Your Terraform state file is lost. What do you do?

**Answer:**

**Impact:** Terraform doesn't know what infrastructure exists. Running `terraform apply` would try to create everything from scratch (duplicating resources).

**Recovery steps:**

1. **Check for backups:**
   ```bash
   # If using S3 with versioning
   aws s3api list-object-versions --bucket my-tf-state --prefix terraform.tfstate
   
   # Restore previous version
   aws s3api get-object --bucket my-tf-state --key terraform.tfstate \
     --version-id "version-id-here" terraform.tfstate
   ```

2. **If no backup exists, re-import resources:**
   ```bash
   # For each resource in your Terraform code
   terraform import aws_instance.web i-1234567890
   terraform import aws_rds_instance.db mydb
   # ... repeat for all resources
   ```

3. **Verify after import:**
   ```bash
   terraform plan
   # Should show "No changes" or minimal changes
   ```

**Prevention:**
- **Always use remote state** (S3, Terraform Cloud, Azure Blob)
- **Enable versioning** on the state storage
- **Enable state locking** (DynamoDB for S3 backend)
- **Back up state regularly**
- **Never store state in Git** (it may contain secrets)