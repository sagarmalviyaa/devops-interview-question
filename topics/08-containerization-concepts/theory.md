# Containerization Concepts — Theory Questions

---

## Q1: What is a container? How is it different from a virtual machine?

**Answer:**
A **container** is a lightweight, isolated package that includes an application and everything it needs to run (code, libraries, settings). It shares the host operating system's kernel but runs in its own isolated space.

| Feature | Container | Virtual Machine |
|---------|-----------|----------------|
| **Size** | Megabytes | Gigabytes |
| **Startup time** | Seconds | Minutes |
| **OS** | Shares host OS kernel | Has its own full OS |
| **Isolation** | Process-level | Hardware-level |
| **Resource usage** | Lightweight | Heavy |
| **Portability** | Very portable | Less portable |
| **Use case** | Microservices, CI/CD | Legacy apps, different OS needs |

**Analogy:**
- VM = A whole house (with its own foundation, plumbing, electricity)
- Container = An apartment in a building (shares the building's infrastructure)

---

## Q2: What is a container image vs. a container?

**Answer:**
- **Image** — A read-only template that contains the application code, libraries, and configuration. Think of it as a recipe or blueprint.
- **Container** — A running instance of an image. Think of it as the actual dish made from the recipe.

```
Image (blueprint) → Container (running instance)
                  → Container (another running instance)
                  → Container (yet another instance)
```

You can create many containers from one image, just like you can bake many cakes from one recipe.

---

## Q3: What is a container registry?

**Answer:**
A **container registry** is a storage service for container images. It's like a library where you store and retrieve images.

**Popular registries:**
| Registry | Type | Notes |
|----------|------|-------|
| Docker Hub | Public/Private | Default registry, largest public collection |
| AWS ECR | Private | Integrated with AWS services |
| Google GCR / Artifact Registry | Private | Integrated with GCP |
| Azure ACR | Private | Integrated with Azure |
| GitHub Container Registry (ghcr.io) | Public/Private | Integrated with GitHub Actions |
| Harbor | Self-hosted | Open-source, enterprise features |

---

## Q4: What are the benefits of containerization?

**Answer:**

1. **Consistency** — "Works on my machine" becomes "works everywhere"
2. **Portability** — Run the same container on your laptop, in CI/CD, or in production
3. **Isolation** — Each container is independent; one crashing doesn't affect others
4. **Efficiency** — Containers share the OS kernel, using less resources than VMs
5. **Speed** — Start in seconds, not minutes
6. **Scalability** — Easy to run multiple copies for handling more traffic
7. **Microservices** — Each service can have its own container with its own dependencies
8. **Immutability** — Containers don't change after creation; deploy a new one instead

---

## Q5: What are namespaces and cgroups in Linux? Why do containers need them?

**Answer:**
Containers are built on two Linux kernel features:

**Namespaces** — Provide isolation (what a container can see):
| Namespace | Isolates |
|-----------|----------|
| PID | Process IDs (container sees only its own processes) |
| NET | Network interfaces, IPs, ports |
| MNT | Filesystem mount points |
| UTS | Hostname |
| IPC | Inter-process communication |
| USER | User and group IDs |

**Cgroups (Control Groups)** — Limit resources (what a container can use):
- CPU: "This container can use at most 2 CPU cores"
- Memory: "This container can use at most 512MB RAM"
- Disk I/O: "This container can read/write at most 100MB/s"
- Network: Bandwidth limits

**Together:** Namespaces make the container think it's alone on the system. Cgroups prevent it from using too many resources.

---

## Q6: What is container orchestration and why is it needed?

**Answer:**
**Container orchestration** is the automated management of many containers across multiple machines. When you have hundreds or thousands of containers, you need a system to:

- **Schedule** — Decide which machine runs which container
- **Scale** — Add or remove containers based on demand
- **Self-heal** — Restart containers that crash
- **Load balance** — Distribute traffic across containers
- **Network** — Let containers find and talk to each other
- **Update** — Roll out new versions without downtime
- **Store** — Manage persistent data for containers

**Tools:** Kubernetes (most popular), Docker Swarm, Amazon ECS, Nomad