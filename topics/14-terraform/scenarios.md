# Kubernetes — Theory Questions

---

## Q1: What is Kubernetes?

**Answer:**
**Kubernetes** (K8s) is an open-source container orchestration platform. It automates deploying, scaling, and managing containerized applications across a cluster of machines. Originally developed by Google, now maintained by the CNCF.

**What Kubernetes does:**
- Runs containers across multiple machines
- Automatically restarts failed containers (self-healing)
- Scales applications up/down based on demand
- Manages networking between containers
- Handles rolling updates and rollbacks
- Manages configuration and secrets

---

## Q2: What are the main Kubernetes components?

**Answer:**

**Control Plane (manages the cluster):**

| Component | Role |
|-----------|------|
| **API Server** | Front door for all operations. All commands go through it |
| **etcd** | Key-value database storing all cluster data and state |
| **Scheduler** | Decides which node should run each new pod |
| **Controller Manager** | Runs controllers that maintain desired state |
| **Cloud Controller Manager** | Integrates with cloud provider APIs |

**Worker Node (runs the workloads):**

| Component | Role |
|-----------|------|
| **Kubelet** | Agent on each node that manages pods |
| **Kube-proxy** | Handles networking rules for pod communication |
| **Container Runtime** | Actually runs containers (containerd, CRI-O) |

---

## Q3: What is a Pod?

**Answer:**
A **Pod** is the smallest deployable unit in Kubernetes. It wraps one or more containers that share the same network and storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 3000
```

**Key points:**
- Containers in the same pod share the same IP address
- They can communicate via `localhost`
- Pods are ephemeral — they can be destroyed at any time
- You rarely create pods directly; use Deployments instead

---

## Q4: What is a Deployment?

**Answer:**
A **Deployment** manages a set of identical pods. It ensures the desired number of pods are running and handles updates.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        resources:
          requests:
            cpu: "250m"
            memory: "128Mi"
          limits:
            cpu: "500m"
            memory: "256Mi"
```

**What a Deployment provides:**
- **Desired state** — "I want 3 replicas running at all times"
- **Rolling updates** — Update pods one at a time, zero downtime
- **Rollback** — `kubectl rollout undo deployment/myapp`
- **Self-healing** — If a pod dies, a new one is created

---

## Q5: What is a Service in Kubernetes?

**Answer:**
A **Service** provides a stable network endpoint to access a group of pods.

| Type | Description | Use Case |
|------|-------------|----------|
| **ClusterIP** (default) | Internal IP only | Service-to-service communication |
| **NodePort** | Exposes on each node's IP at a static port | Development, testing |
| **LoadBalancer** | Creates an external load balancer | Production external access |
| **ExternalName** | Maps to an external DNS name | Accessing external services |

```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 3000
```

---

## Q6: What is an Ingress?

**Answer:**
An **Ingress** manages external HTTP/HTTPS access to services. It provides URL-based routing, SSL termination, and virtual hosting.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

**Ingress Controllers:** Nginx Ingress, Traefik, HAProxy, AWS ALB Ingress Controller

---

## Q7: What are ConfigMaps and Secrets?

**Answer:**

**ConfigMap** — Stores non-sensitive configuration:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  DATABASE_HOST: "db.example.com"
  LOG_LEVEL: "info"
```

**Secret** — Stores sensitive data (base64 encoded):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQxMjM=
```

**Using them in pods:**
```yaml
env:
- name: DB_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: DATABASE_HOST
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: app-secrets
      key: DB_PASSWORD
```

**Important:** K8s Secrets are base64 encoded, NOT encrypted by default. Use Vault or Sealed Secrets for real security.

---

## Q8: What are Namespaces?

**Answer:**
**Namespaces** are virtual clusters within a Kubernetes cluster for isolation and organization.

```bash
kubectl get namespaces
# default, kube-system, kube-public

kubectl create namespace staging
kubectl apply -f deployment.yaml -n staging
```

**Use cases:** Separate environments, teams, or projects. Apply resource quotas and network policies per namespace.

---

## Q9: What are liveness, readiness, and startup probes?

**Answer:**

| Probe | Purpose | On Failure |
|-------|---------|-----------|
| **Liveness** | Is the container alive? | Container is restarted |
| **Readiness** | Is it ready for traffic? | Removed from Service endpoints |
| **Startup** | Has it finished starting? | Liveness/readiness probes delayed |

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 3000
  initialDelaySeconds: 15
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 3000
  initialDelaySeconds: 5
  periodSeconds: 5

startupProbe:
  httpGet:
    path: /healthz
    port: 3000
  failureThreshold: 30
  periodSeconds: 10
```

---

## Q10: What is a StatefulSet?

**Answer:**

| Feature | Deployment | StatefulSet |
|---------|-----------|-------------|
| Pod identity | Random names | Stable names (myapp-0, myapp-1) |
| Storage | Shared or none | Each pod gets its own volume |
| Scaling order | All at once | One at a time, in order |
| Use case | Stateless apps | Databases, message queues |

---

## Q11: What is a DaemonSet?

**Answer:**
A **DaemonSet** ensures a copy of a pod runs on every node. Used for log collectors (Fluentd), monitoring agents (Node Exporter), and network plugins.

---

## Q12: What is Horizontal Pod Autoscaler (HPA)?

**Answer:**
HPA automatically scales pod replicas based on metrics.

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## Q13: What are Resource Requests and Limits?

**Answer:**
- **Requests** — Minimum resources needed (used for scheduling)
- **Limits** — Maximum resources allowed (enforced at runtime)

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

CPU limit exceeded → throttled. Memory limit exceeded → OOMKilled.

---

## Q14: What is RBAC in Kubernetes?

**Answer:**
**RBAC (Role-Based Access Control)** controls who can do what in the cluster.

**Components:**
- **Role** — Permissions within a namespace
- **ClusterRole** — Permissions cluster-wide
- **RoleBinding** — Assigns a Role to a user/group
- **ClusterRoleBinding** — Assigns a ClusterRole cluster-wide

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: staging
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: staging
subjects:
- kind: User
  name: developer@example.com
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

---

## Q15: What is a NetworkPolicy?

**Answer:**
**NetworkPolicy** controls traffic flow between pods. By default, all pods can communicate with each other. NetworkPolicies restrict this.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - port: 5432
```

This policy says: The `api` pods can only receive traffic from `frontend` pods on port 8080, and can only send traffic to `database` pods on port 5432.

---

## Q16: What are Persistent Volumes (PV) and Persistent Volume Claims (PVC)?

**Answer:**

- **PersistentVolume (PV)** — A piece of storage provisioned by an admin or dynamically
- **PersistentVolumeClaim (PVC)** — A request for storage by a user/pod
- **StorageClass** — Defines the type of storage (SSD, HDD, etc.)

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: gp3
  resources:
    requests:
      storage: 20Gi
---
# Using in a pod
spec:
  containers:
  - name: app
    volumeMounts:
    - name: data
      mountPath: /data
  volumes:
  - name: data
    persistentVolumeClaim:
      claimName: data-pvc
```

**Access modes:**
- `ReadWriteOnce` (RWO) — One node can read/write
- `ReadOnlyMany` (ROX) — Many nodes can read
- `ReadWriteMany` (RWX) — Many nodes can read/write

---

## Q17: What are Init Containers?

**Answer:**
**Init containers** run before the main containers start. They're used for setup tasks.

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'until nc -z db-service 5432; do sleep 2; done']
  - name: run-migrations
    image: myapp:1.0
    command: ['python', 'manage.py', 'migrate']
  containers:
  - name: myapp
    image: myapp:1.0
```

**Use cases:**
- Wait for a dependency to be ready
- Run database migrations
- Download configuration files
- Set up permissions on shared volumes

---

## Q18: What are Taints and Tolerations?

**Answer:**
**Taints** are applied to nodes to repel pods. **Tolerations** are applied to pods to allow them on tainted nodes.

```bash
# Taint a node
kubectl taint nodes node1 dedicated=gpu:NoSchedule
```

```yaml
# Pod with toleration (can run on the tainted node)
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

**Effects:**
- `NoSchedule` — Don't schedule new pods (existing pods stay)
- `PreferNoSchedule` — Try to avoid, but allow if necessary
- `NoExecute` — Evict existing pods and don't schedule new ones

**Use cases:** Dedicate nodes for specific workloads (GPU, high-memory), keep system pods on master nodes.

---

## Q19: What are Labels and Selectors?

**Answer:**
**Labels** are key-value pairs attached to Kubernetes objects. **Selectors** filter objects by their labels.

```yaml
# Labels on a pod
metadata:
  labels:
    app: myapp
    environment: production
    team: backend
    version: v2.1
```

```bash
# Select by label
kubectl get pods -l app=myapp
kubectl get pods -l "environment=production,team=backend"
kubectl get pods -l "environment in (staging, production)"
```

**Used by:** Services (to find pods), Deployments (to manage pods), NetworkPolicies (to target pods), Node selectors (to place pods).

---

## Q20: What is the difference between `kubectl apply` and `kubectl create`?

**Answer:**

| Command | Behavior | Idempotent? |
|---------|----------|-------------|
| `kubectl create` | Creates a new resource. Fails if it already exists | No |
| `kubectl apply` | Creates if new, updates if exists. Declarative | Yes |

```bash
kubectl create -f deployment.yaml    # First time: OK. Second time: ERROR
kubectl apply -f deployment.yaml     # First time: Created. Second time: Updated (if changed)
```

**Best practice:** Always use `kubectl apply` — it's declarative and idempotent.