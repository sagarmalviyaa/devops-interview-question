# AWS — Scenario-Based Questions

---

## S1: You need to design a highly available web application on AWS. What's your architecture?

**Answer:**

```
Users → Route 53 (DNS) → CloudFront (CDN)
    → Application Load Balancer
        → Auto Scaling Group
            → EC2 instances (or ECS/EKS)
                AZ-1: App Server 1, App Server 2
                AZ-2: App Server 3, App Server 4
        → RDS (Multi-AZ)
            Primary (AZ-1) ↔ Standby (AZ-2)
        → ElastiCache (Redis)
```

**Key design decisions:**
1. **Multi-AZ** — Deploy across at least 2 Availability Zones
2. **Auto Scaling** — Automatically add/remove instances based on load
3. **RDS Multi-AZ** — Automatic failover for the database
4. **CloudFront** — Cache static content at edge locations
5. **ALB** — Distribute traffic across healthy instances
6. **S3** — Store static assets, backups
7. **CloudWatch** — Monitor everything, set up alarms

**For even higher availability:**
- Multi-region deployment with Route 53 failover routing
- Read replicas for the database
- DynamoDB Global Tables for multi-region data

---

## S2: Your AWS bill has increased by 40% this month. How do you investigate and reduce costs?

**Answer:**

1. **Investigate:**
   ```
   AWS Cost Explorer → Group by Service → Identify top spenders
   AWS Cost Explorer → Group by Usage Type → Find specific cost drivers
   ```

2. **Common cost culprits:**
   - Unused EC2 instances (dev/test left running)
   - Oversized instances (t3.xlarge when t3.medium would suffice)
   - Unattached EBS volumes
   - Old snapshots and AMIs
   - Data transfer costs
   - Unused Elastic IPs
   - NAT Gateway data processing

3. **Quick wins:**
   ```bash
   # Find unused resources
   aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" \
     --query "Reservations[].Instances[].[InstanceId,InstanceType,LaunchTime]"
   
   # Find unattached EBS volumes
   aws ec2 describe-volumes --filters "Name=status,Values=available"
   
   # Find old snapshots
   aws ec2 describe-snapshots --owner-ids self \
     --query "Snapshots[?StartTime<'2025-01-01']"
   ```

4. **Long-term savings:**
   - Use Reserved Instances or Savings Plans for steady workloads
   - Use Spot Instances for fault-tolerant workloads
   - Right-size instances (use AWS Compute Optimizer)
   - Use S3 lifecycle policies to move old data to cheaper storage
   - Set up billing alerts and budgets
   - Tag all resources for cost allocation

---

## S3: An EC2 instance is unreachable via SSH. How do you troubleshoot?

**Answer:**

1. **Check instance status:**
   ```bash
   aws ec2 describe-instance-status --instance-ids i-1234567890
   # Check: instance state, system status, instance status
   ```

2. **Check Security Group:**
   - Is port 22 (SSH) allowed from your IP?
   - Did your IP change? (home/office IP rotation)

3. **Check Network ACL:**
   - Is inbound port 22 allowed?
   - Is outbound traffic allowed? (NACLs are stateless)

4. **Check Route Table:**
   - Does the subnet have a route to the Internet Gateway?
   - Is the instance in a public subnet with a public IP?

5. **Check the instance itself:**
   - View system log: `aws ec2 get-console-output --instance-id i-1234567890`
   - Is the instance running? (not stopped or terminated)
   - Is the SSH key correct?

6. **If all else fails:**
   - Use **EC2 Instance Connect** or **SSM Session Manager** (doesn't need port 22)
   - Stop the instance, detach the root volume, attach to another instance, check logs
   - Check `/var/log/auth.log` and `/var/log/syslog`

**Prevention:**
- Use SSM Session Manager instead of SSH (no need to open port 22)
- Use a bastion host in a public subnet
- Keep security groups restrictive