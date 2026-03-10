# Service Mesh — Theory Questions

---

## Q1: What is a Service Mesh?

**Answer:**
A **service mesh** is a dedicated infrastructure layer for managing service-to-service communication in a microservices architecture. It handles networking concerns (security, observability, traffic management) without changing application code.

**What it provides:**
- **mTLS (mutual TLS)** — Encrypted communication between all services
- **Traffic management** — Canary deployments, A/B testing, retries, timeouts
- **Observability** — Distributed tracing, metrics, access logs
- **Security** — Authentication, authorization between services
- **Resilience** — Circuit breaking, retries, fault injection

---

## Q2: How does a service mesh work?

**Answer:**
A service mesh uses the **sidecar proxy pattern**. A lightweight proxy (usually Envoy) is deployed alongside each service instance. All traffic goes through this proxy.

```
Service A → [Sidecar Proxy A] → [Sidecar Proxy B] → Service B
                    ↕                      ↕
              Control Plane (Istiod)
```

**Components:**
- **Data Plane** — The sidecar proxies that handle actual traffic
- **Control Plane** — Manages and configures the proxies (policies, routing rules)

---

## Q3: What is Istio?

**Answer:**
**Istio** is the most popular service mesh for Kubernetes. Its control plane is called **Istiod**.

**Key features:**
- **Traffic management:**
  ```yaml
  apiVersion: networking.istio.io/v1beta1
  kind: VirtualService
  metadata:
    name: myapp
  spec:
    hosts:
    - myapp
    http:
    - route:
      - destination:
          host: myapp
          subset: v2
        weight: 10        # 10% to v2 (canary)
      - destination:
          host: myapp
          subset: v1
        weight: 90        # 90% to v1
  ```

- **mTLS (automatic encryption):**
  ```yaml
  apiVersion: security.istio.io/v1beta1
  kind: PeerAuthentication
  metadata:
    name: default
  spec:
    mtls:
      mode: STRICT    # All traffic must be mTLS
  ```

- **Authorization:**
  ```yaml
  apiVersion: security.istio.io/v1beta1
  kind: AuthorizationPolicy
  metadata:
    name: api-policy
  spec:
    selector:
      matchLabels:
        app: api
    rules:
    - from:
      - source:
          principals: ["cluster.local/ns/default/sa/frontend"]
      to:
      - operation:
          methods: ["GET", "POST"]
  ```

---

## Q4: What are alternatives to Istio?

**Answer:**

| Service Mesh | Key Feature | Complexity |
|-------------|-------------|-----------|
| **Istio** | Most feature-rich, largest community | High |
| **Linkerd** | Lightweight, simple, fast | Low |
| **Consul Connect** | HashiCorp ecosystem, multi-platform | Medium |
| **Cilium** | eBPF-based (no sidecar needed), high performance | Medium |

**Choose Istio when:** You need advanced traffic management, large-scale deployment.
**Choose Linkerd when:** You want simplicity and low resource overhead.
**Choose Cilium when:** You want high performance without sidecar overhead.

---

## Q5: When do you need a service mesh?

**Answer:**

**You likely need a service mesh when:**
- You have 10+ microservices communicating with each other
- You need mTLS between all services (zero trust)
- You want canary deployments or traffic splitting
- You need distributed tracing across services
- You have compliance requirements for encrypted communication

**You probably don't need a service mesh when:**
- You have fewer than 5 services
- Your services don't communicate much with each other
- You're using a monolithic architecture
- The added complexity outweighs the benefits

**Important:** A service mesh adds complexity and resource overhead. Don't add it unless you have a clear need.