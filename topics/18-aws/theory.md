# AWS (Amazon Web Services) — Theory Questions

---

## Q1: What is AWS? Name the most important services for DevOps.

**Answer:**
**AWS** is the world's largest cloud computing platform, offering 200+ services. Key services for DevOps:

| Category | Service | Purpose |
|----------|---------|---------|
| **Compute** | EC2 | Virtual servers |
| | Lambda | Serverless functions |
| | ECS/EKS | Container orchestration |
| **Storage** | S3 | Object storage (files, backups, static sites) |
| | EBS | Block storage (disk volumes for EC2) |
| **Database** | RDS | Managed relational databases |
| | DynamoDB | Managed NoSQL database |
| **Networking** | VPC | Virtual private network |
| | Route 53 | DNS service |
| | CloudFront | CDN |
| | ALB/NLB | Load balancers |
| **Security** | IAM | Identity and access management |
| | Secrets Manager | Secret storage |
| | KMS | Encryption key management |
| **CI/CD** | CodePipeline | CI/CD pipeline |
| | CodeBuild | Build service |
| | ECR | Docker image registry |
| **Monitoring** | CloudWatch | Metrics, logs, alarms |
| | CloudTrail | API audit logging |
| **IaC** | CloudFormation | AWS-native IaC |

---

## Q2: What is IAM? Explain users, groups, roles, and policies.

**Answer:**
**IAM (Identity and Access Management)** controls who can access what in AWS.

- **User** — An individual person or application with credentials
- **Group** — A collection of users (e.g., "developers", "admins")
- **Role** — An identity that can be assumed by users, services, or applications (no permanent credentials)
- **Policy** — A JSON document that defines permissions

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    },
    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }
  ]
}
```

**Best practices:**
- Follow **least privilege** — only grant permissions that are needed
- Use **roles** instead of long-lived access keys
- Enable **MFA** for all human users
- Use **groups** to manage permissions (not individual users)
- Regularly audit and rotate credentials

---

## Q3: What is a VPC? Explain its components.

**Answer:**
A **VPC (Virtual Private Cloud)** is your own isolated network in AWS.

**Components:**

| Component | Purpose |
|-----------|---------|
| **VPC** | The overall network (e.g., 10.0.0.0/16) |
| **Subnet** | A segment of the VPC (public or private) |
| **Internet Gateway** | Connects VPC to the internet |
| **NAT Gateway** | Lets private subnets access the internet (outbound only) |
| **Route Table** | Rules for directing traffic |
| **Security Group** | Firewall for instances (stateful) |
| **Network ACL** | Firewall for subnets (stateless) |
| **VPC Peering** | Connect two VPCs |
| **VPN Gateway** | Connect VPC to on-premises network |

**Typical architecture:**
```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24) → Internet Gateway
│   ├── Load Balancer
│   └── NAT Gateway
└── Private Subnet (10.0.2.0/24) → NAT Gateway (for outbound)
    ├── Application Servers
    └── Database Servers
```

---

## Q4: What is the difference between Security Groups and NACLs?

**Answer:**

| Feature | Security Group | Network ACL |
|---------|---------------|-------------|
| Level | Instance (ENI) level | Subnet level |
| Stateful? | Yes (return traffic auto-allowed) | No (must allow both inbound AND outbound) |
| Rules | Allow rules only | Allow AND deny rules |
| Evaluation | All rules evaluated | Rules evaluated in order (by number) |
| Default | Deny all inbound, allow all outbound | Allow all inbound and outbound |

**Best practice:** Use Security Groups as your primary firewall. Use NACLs as an additional layer for subnet-level control.

---

## Q5: What is S3? What are the storage classes?

**Answer:**
**S3 (Simple Storage Service)** is object storage for any type of file. It's highly durable (99.999999999% — "eleven nines") and scalable.

**Storage classes (by cost and access frequency):**

| Class | Use Case | Cost |
|-------|----------|------|
| **S3 Standard** | Frequently accessed data | Highest |
| **S3 Intelligent-Tiering** | Unknown access patterns | Auto-optimizes |
| **S3 Standard-IA** | Infrequently accessed | Lower storage, higher retrieval |
| **S3 One Zone-IA** | Infrequent, non-critical | Lowest IA cost |
| **S3 Glacier Instant** | Archive with instant access | Low |
| **S3 Glacier Flexible** | Archive (minutes to hours retrieval) | Very low |
| **S3 Glacier Deep Archive** | Long-term archive (12+ hours retrieval) | Lowest |

**DevOps use cases:** Terraform state, build artifacts, backups, static website hosting, log storage.

---

## Q6: What is EC2? What are the instance types?

**Answer:**
**EC2 (Elastic Compute Cloud)** provides virtual servers in the cloud.

**Instance type families:**

| Family | Optimized For | Example | Use Case |
|--------|-------------|---------|----------|
| **T** | General purpose (burstable) | t3.medium | Web servers, dev environments |
| **M** | General purpose (steady) | m6i.xlarge | Application servers |
| **C** | Compute | c6i.2xlarge | Batch processing, CI/CD builds |
| **R** | Memory | r6i.xlarge | Databases, caching |
| **I** | Storage I/O | i3.xlarge | Databases, data warehouses |
| **G/P** | GPU | g5.xlarge | Machine learning, video processing |

**Purchasing options:**
- **On-Demand** — Pay by the hour, no commitment
- **Reserved** — 1-3 year commitment, up to 72% discount
- **Spot** — Bid for unused capacity, up to 90% discount (can be interrupted)
- **Savings Plans** — Flexible commitment, up to 72% discount

---

## Q7: What is Lambda?

**Answer:**
**AWS Lambda** is a serverless compute service. You upload code, and AWS runs it in response to events. You pay only for the time your code runs.

**Key features:**
- No servers to manage
- Automatic scaling (from 0 to thousands of concurrent executions)
- Pay per invocation and duration
- Supports: Node.js, Python, Java, Go, .NET, Ruby, custom runtimes
- Max execution time: 15 minutes

**Common triggers:** API Gateway, S3 events, SQS messages, CloudWatch Events, DynamoDB streams.

**DevOps use cases:** Automated remediation, log processing, CI/CD webhooks, scheduled tasks, infrastructure automation.

---

## Q8: What is the difference between ECS and EKS?

**Answer:**

| Feature | ECS | EKS |
|---------|-----|-----|
| **Full name** | Elastic Container Service | Elastic Kubernetes Service |
| **Orchestrator** | AWS-proprietary | Kubernetes |
| **Complexity** | Simpler | More complex |
| **Portability** | AWS-only | Multi-cloud (standard K8s) |
| **Learning curve** | Lower | Higher |
| **Ecosystem** | AWS-native tools | Entire K8s ecosystem |
| **Launch types** | Fargate or EC2 | Fargate or EC2 |

**Choose ECS when:** You're all-in on AWS, want simplicity, smaller team.
**Choose EKS when:** You need Kubernetes features, multi-cloud strategy, large ecosystem.

---

## Q9: What is CloudWatch?

**Answer:**
**CloudWatch** is AWS's monitoring and observability service.

**Components:**
- **Metrics** — Collect and track metrics (CPU, network, custom)
- **Logs** — Collect, monitor, and analyze log files
- **Alarms** — Alert when metrics cross thresholds
- **Dashboards** — Visualize metrics
- **Events/EventBridge** — React to AWS resource changes

```bash
# Create an alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "HighCPU" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456:alerts
```

---

## Q10: What is CloudFormation?

**Answer:**
**CloudFormation** is AWS's native Infrastructure as Code service. It uses JSON or YAML templates to define AWS resources.

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345678
      InstanceType: t3.medium
      SecurityGroupIds:
        - !Ref WebSecurityGroup
  
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

Outputs:
  ServerIP:
    Value: !GetAtt WebServer.PublicIp
```

**CloudFormation vs. Terraform:**
- CloudFormation: AWS-only, deeper AWS integration, free
- Terraform: Multi-cloud, larger community, HCL syntax