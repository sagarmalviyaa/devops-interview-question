# Helm — Theory Questions

---

## Q1: What is Helm?

**Answer:**
**Helm** is a package manager for Kubernetes. It packages Kubernetes manifests into reusable, versioned units called **charts**. Think of it as "apt/yum for Kubernetes."

**Why Helm?**
- **Templating** — Use variables instead of hardcoding values
- **Packaging** — Bundle all K8s manifests into one chart
- **Versioning** — Track and rollback releases
- **Reusability** — Share charts across teams and projects
- **Dependency management** — Charts can depend on other charts

---

## Q2: What is a Helm chart?

**Answer:**
A **chart** is a collection of files that describe a set of Kubernetes resources.

```
mychart/
├── Chart.yaml          # Chart metadata (name, version, description)
├── values.yaml         # Default configuration values
├── templates/          # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── _helpers.tpl    # Template helper functions
│   └── NOTES.txt       # Post-install instructions
├── charts/             # Dependency charts
└── .helmignore         # Files to ignore when packaging
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: myapp
description: My application Helm chart
version: 1.2.0          # Chart version
appVersion: "2.1.0"      # Application version
```

---

## Q3: What are Helm values and how do you override them?

**Answer:**
**Values** are configuration parameters that customize a chart.

**values.yaml (defaults):**
```yaml
replicaCount: 3
image:
  repository: myapp
  tag: "1.0"
  pullPolicy: IfNotPresent
service:
  type: ClusterIP
  port: 80
resources:
  limits:
    cpu: 500m
    memory: 256Mi
```

**Template using values:**
```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-app
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: app
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

**Override values:**
```bash
# Using --set flag
helm install myapp ./mychart --set replicaCount=5

# Using a values file
helm install myapp ./mychart -f production-values.yaml

# Multiple overrides (later takes precedence)
helm install myapp ./mychart -f base.yaml -f production.yaml --set image.tag=2.0
```

---

## Q4: What are the key Helm commands?

**Answer:**

```bash
# Install a chart
helm install myrelease ./mychart
helm install myrelease bitnami/nginx

# Upgrade a release
helm upgrade myrelease ./mychart --set image.tag=2.0

# Rollback to a previous version
helm rollback myrelease 1

# Uninstall a release
helm uninstall myrelease

# List releases
helm list

# Show release history
helm history myrelease

# Preview what would be installed (dry run)
helm install myrelease ./mychart --dry-run --debug

# Template rendering (see generated YAML)
helm template myrelease ./mychart

# Add a chart repository
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

# Search for charts
helm search repo nginx
helm search hub prometheus
```

---

## Q5: What is a Helm release?

**Answer:**
A **release** is a running instance of a chart. You can install the same chart multiple times with different release names and configurations.

```bash
# Same chart, different releases
helm install frontend-prod ./webapp -f prod-values.yaml
helm install frontend-staging ./webapp -f staging-values.yaml
```

Each release has its own history, making rollbacks independent.

---

## Q6: What are Helm hooks?

**Answer:**
**Hooks** let you run actions at specific points in a release lifecycle.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  annotations:
    "helm.sh/hook": pre-upgrade
    "helm.sh/hook-weight": "1"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: myapp:1.0
        command: ["python", "manage.py", "migrate"]
      restartPolicy: Never
```

**Hook types:** `pre-install`, `post-install`, `pre-upgrade`, `post-upgrade`, `pre-delete`, `post-delete`, `pre-rollback`, `post-rollback`

**Use cases:** Database migrations, sending notifications, running tests, cleaning up resources.