# Orchestration Concepts — Scenario-Based Questions

---

## S1: Your application needs to handle a sudden 10x traffic spike (like a flash sale). How do you prepare?

**Answer:**

1. **Auto-scaling:**
   - Configure Horizontal Pod Autoscaler (HPA) based on CPU/memory or custom metrics
   - Set appropriate min/max replica counts
   - Pre-scale before the event if you know it's coming

2. **Infrastructure readiness:**
   - Ensure the cluster has enough nodes (or enable cluster autoscaler)
   - Pre-warm load balancers
   - Increase database connection pool limits
   - Scale database read replicas

3. **Application readiness:**
   - Ensure the app is stateless (can scale horizontally)
   - Use caching (Redis) to reduce database load
   - Implement rate limiting to protect backend services
   - Use a CDN for static assets

4. **Testing:**
   - Load test before the event to find bottlenecks
   - Test auto-scaling behavior
   - Have a runbook for manual scaling if auto-scaling isn't fast enough

5. **During the event:**
   - Monitor dashboards closely
   - Have engineers on standby
   - Be ready to manually scale if needed

---

## S2: A node in your cluster goes down. What happens to the containers running on it?

**Answer:**

**With an orchestrator (Kubernetes):**
1. The control plane detects the node is unhealthy (no heartbeat)
2. After a timeout (default ~5 minutes), it marks the node as `NotReady`
3. The scheduler automatically reschedules the containers (pods) to other healthy nodes
4. If using Deployments, the desired replica count is maintained
5. Load balancer stops sending traffic to the dead node

**What you should have in place:**
- **Multiple replicas** — so losing one pod doesn't cause downtime
- **Pod Disruption Budgets** — ensure minimum available pods during disruptions
- **Anti-affinity rules** — spread replicas across different nodes
- **Cluster autoscaler** — add new nodes if capacity is insufficient
- **Persistent volume replication** — so data isn't lost with the node

**Without an orchestrator:**
- Containers on the dead node are gone
- You manually restart them on another machine
- Downtime until you notice and act

This is a key reason why orchestration is essential for production.

---

## S3: You need to choose between running your own Kubernetes cluster or using a managed service (EKS/GKE/AKS). What factors do you consider?

**Answer:**

| Factor | Self-Managed | Managed (EKS/GKE/AKS) |
|--------|-------------|----------------------|
| **Control plane management** | You maintain it | Cloud provider maintains it |
| **Upgrades** | You plan and execute | Provider handles (with your approval) |
| **Cost** | Lower compute cost, higher ops cost | Higher compute cost, lower ops cost |
| **Customization** | Full control | Some limitations |
| **Team expertise needed** | High (need K8s experts) | Medium |
| **Time to set up** | Days to weeks | Hours |
| **SLA** | You define it | Provider guarantees (99.95%+) |

**Choose managed when:**
- Your team is small or lacks deep Kubernetes expertise
- You want to focus on applications, not infrastructure
- You need quick setup and reliable operations
- Budget allows for managed service costs

**Choose self-managed when:**
- You need full control over the cluster configuration
- You have a dedicated platform team with K8s expertise
- You're running on-premises or in a non-supported cloud
- You have specific compliance requirements

**Recommendation for most teams:** Start with managed Kubernetes. You can always migrate to self-managed later if needed.