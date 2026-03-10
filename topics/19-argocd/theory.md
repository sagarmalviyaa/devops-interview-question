# ArgoCD (GitOps) — Theory Questions

---

## Q1: What is GitOps?

**Answer:**
**GitOps** is a way of managing infrastructure and application deployments where Git is the single source of truth. All changes go through Git, and an automated system ensures the live environment matches what's in Git.

**Core principles:**
1. **Declarative** — The desired state is described in Git (YAML/JSON)
2. **Versioned** — All changes are tracked in Git history
3. **Automated** — Changes in Git are automatically applied
4. **Self-healing** — If someone changes something manually, the system reverts it to match Git

---

## Q2: What is ArgoCD?

**Answer:**
**ArgoCD** is a declarative GitOps continuous delivery tool for Kubernetes. It watches a Git repository and automatically syncs the Kubernetes cluster to match the desired state in Git.

```
Developer → Push to Git → ArgoCD detects change → Syncs to Kubernetes
```

**Key features:**
- Automatic sync from Git to Kubernetes
- Web UI for visualizing application state
- Rollback to any previous Git commit
- Multi-cluster support
- SSO integration
- Health status monitoring
- Diff visualization (what changed)

---

## Q3: What is an ArgoCD Application?

**Answer:**
An **Application** is ArgoCD's core resource. It defines the connection between a Git repo and a Kubernetes cluster.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myapp-k8s.git
    targetRevision: main
    path: k8s/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true          # Delete resources removed from Git
      selfHeal: true       # Revert manual changes
    syncOptions:
    - CreateNamespace=true
```

---

## Q4: What is the difference between ArgoCD sync policies?

**Answer:**

**Manual sync:** You click "Sync" in the UI or run `argocd app sync myapp`. ArgoCD shows the diff but doesn't apply automatically.

**Automated sync:** ArgoCD automatically applies changes when it detects a difference between Git and the cluster.

**Sync options:**
- `prune: true` — Delete resources that were removed from Git
- `selfHeal: true` — Revert any manual changes made directly to the cluster
- `retry` — Automatically retry failed syncs

**Sync strategies:**
- `Apply` — Standard `kubectl apply`
- `Hook` — Run pre/post sync hooks (jobs, scripts)

---

## Q5: How does ArgoCD compare to traditional CI/CD for deployments?

**Answer:**

| Feature | Traditional CI/CD (Push) | ArgoCD (Pull/GitOps) |
|---------|------------------------|---------------------|
| **Model** | CI pipeline pushes to cluster | ArgoCD pulls from Git |
| **Credentials** | CI needs cluster credentials | Only ArgoCD needs cluster access |
| **Source of truth** | Pipeline configuration | Git repository |
| **Drift detection** | None | Continuous monitoring |
| **Rollback** | Redeploy previous version | `git revert` |
| **Audit trail** | Pipeline logs | Git history |
| **Security** | Cluster creds in CI system | Cluster creds only in ArgoCD |

**Best practice:** Use CI for building and testing, ArgoCD for deploying.
```
CI (GitHub Actions/Jenkins): Code → Build → Test → Push image → Update Git manifests
CD (ArgoCD): Detect Git change → Sync to Kubernetes
```