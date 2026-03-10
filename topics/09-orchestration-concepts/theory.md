# Orchestration Concepts — Theory Questions

---

## Q1: What is container orchestration?

**Answer:**
Container orchestration is the automated management of containerized applications across a cluster of machines. It handles deployment, scaling, networking, and availability of containers.

**What an orchestrator does:**
- **Scheduling** — Decides which machine (node) runs each container
- **Scaling** — Adds or removes container instances based on demand
- **Self-healing** — Restarts failed containers automatically
- **Load balancing** — Distributes traffic across healthy containers
- **Service discovery** — Lets containers find each other by name
- **Rolling updates** — Deploys new versions without downtime
- **Resource management** — Ensures containers get the CPU and memory they need

---

## Q2: Why can't you just use Docker alone for production?

**Answer:**
Docker runs containers on a single machine. In production, you need:

| Need | Docker Alone | With Orchestrator |
|------|-------------|-------------------|
| Run on multiple machines | Manual setup | Automatic scheduling |
| Handle container crashes | Manual restart | Auto-restart (self-healing) |
| Scale up/down | Manual | Automatic based on load |
| Load balancing | Manual (external LB) | Built-in |
| Rolling updates | Manual, risky | Automated, zero-downtime |
| Service discovery | Manual DNS/config | Automatic |
| Secret management | Environment variables | Encrypted secret stores |

**Docker Compose** helps for development (multiple containers on one machine), but it's not designed for production-scale multi-machine deployments.

---

## Q3: What are the main container orchestration tools?

**Answer:**

| Tool | Maintained By | Best For |
|------|--------------|----------|
| **Kubernetes (K8s)** | CNCF (Cloud Native Computing Foundation) | Industry standard, any scale |
| **Docker Swarm** | Docker Inc. | Simple setups, small teams |
| **Amazon ECS** | AWS | AWS-native, simpler than K8s |
| **Amazon EKS** | AWS | Managed Kubernetes on AWS |
| **Google GKE** | Google | Managed Kubernetes on GCP |
| **Azure AKS** | Microsoft | Managed Kubernetes on Azure |
| **Nomad** | HashiCorp | Multi-workload (containers + VMs + batch) |

**Kubernetes is the dominant choice** in 2026. Most companies either run Kubernetes directly or use a managed version (EKS, GKE, AKS).

---

## Q4: What is a cluster in orchestration?

**Answer:**
A **cluster** is a group of machines (nodes) that work together to run containerized applications.

**Components:**
- **Control Plane (Master)** — The brain that makes decisions:
  - API Server — Entry point for all commands
  - Scheduler — Decides where to place containers
  - Controller Manager — Ensures desired state is maintained
  - etcd — Database storing cluster state

- **Worker Nodes** — The machines that actually run containers:
  - Kubelet — Agent that manages containers on the node
  - Container Runtime — Software that runs containers (containerd, CRI-O)
  - Kube-proxy — Handles networking

```
Cluster
├── Control Plane (Master Node)
│   ├── API Server
│   ├── Scheduler
│   ├── Controller Manager
│   └── etcd
├── Worker Node 1
│   ├── Kubelet
│   ├── Container Runtime
│   └── Pods (containers)
├── Worker Node 2
│   └── ...
└── Worker Node 3
    └── ...
```

---

## Q5: What is the difference between horizontal and vertical scaling?

**Answer:**

**Vertical scaling (scale up):**
- Add more resources (CPU, RAM) to an existing machine
- Like upgrading your computer
- Limit: There's a maximum machine size
- Downtime: Usually requires restart

**Horizontal scaling (scale out):**
- Add more machines/containers
- Like adding more computers
- Limit: Virtually unlimited
- Downtime: None (new instances added alongside existing ones)

**In container orchestration:**
```bash
# Horizontal scaling (preferred)
kubectl scale deployment myapp --replicas=10
# Now 10 copies of your app are running

# Vertical scaling
# Change resource requests in the deployment spec
resources:
  requests:
    cpu: "500m"    → "1000m"
    memory: "256Mi" → "512Mi"
```

**Best practice:** Design applications for horizontal scaling (stateless, shared-nothing). It's more resilient and cost-effective.