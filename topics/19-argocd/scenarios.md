# ArgoCD — Scenario-Based Questions

---

## S1: An ArgoCD application shows "OutOfSync" status. How do you investigate?

**Answer:**

1. **Check the diff:**
   ```bash
   argocd app diff myapp
   # Or use the ArgoCD web UI — it shows exactly what's different
   ```

2. **Common causes:**
   - Someone made a manual change to the cluster (kubectl edit/apply)
   - Git repo was updated but sync hasn't happened yet
   - A resource has fields that are auto-populated by Kubernetes (annotations, status)

3. **Fix based on cause:**
   - **Manual change:** If `selfHeal` is enabled, ArgoCD will revert it. If not, sync manually:
     ```bash
     argocd app sync myapp
     ```
   - **Expected difference:** Add ignore rules for auto-populated fields:
     ```yaml
     spec:
       ignoreDifferences:
       - group: apps
         kind: Deployment
         jsonPointers:
         - /spec/replicas    # Ignore if HPA manages replicas
     ```

4. **Enable self-healing** to prevent future drift:
   ```yaml
   syncPolicy:
     automated:
       selfHeal: true
   ```

---

## S2: You need to set up ArgoCD for multiple environments (dev, staging, production). How?

**Answer:**

**Option 1: Directory-based (recommended):**
```
myapp-k8s/
├── base/                    # Shared manifests
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   └── kustomization.yaml    # Dev-specific patches
│   ├── staging/
│   │   └── kustomization.yaml
│   └── production/
│       └── kustomization.yaml
```

**Create an ArgoCD Application per environment:**
```yaml
# dev-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-dev
spec:
  source:
    repoURL: https://github.com/myorg/myapp-k8s.git
    path: overlays/dev
  destination:
    namespace: dev
---
# production-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-production
spec:
  source:
    repoURL: https://github.com/myorg/myapp-k8s.git
    path: overlays/production
  destination:
    namespace: production
  syncPolicy:
    automated:
      prune: false       # Don't auto-delete in production
      selfHeal: true
```

**Option 2: Use ApplicationSets** for dynamic generation:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: myapp
spec:
  generators:
  - list:
      elements:
      - env: dev
        namespace: dev
      - env: staging
        namespace: staging
      - env: production
        namespace: production
  template:
    metadata:
      name: 'myapp-{{env}}'
    spec:
      source:
        repoURL: https://github.com/myorg/myapp-k8s.git
        path: 'overlays/{{env}}'
      destination:
        namespace: '{{namespace}}'
```