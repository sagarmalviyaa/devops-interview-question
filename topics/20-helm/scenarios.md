# Helm — Scenario-Based Questions

---

## S1: A Helm upgrade failed and the application is down. How do you recover?

**Answer:**

1. **Check the release status:**
   ```bash
   helm list
   helm history myrelease
   # Find the last successful revision number
   ```

2. **Rollback immediately:**
   ```bash
   helm rollback myrelease 3    # Rollback to revision 3
   ```

3. **Investigate the failure:**
   ```bash
   # See what changed
   helm diff upgrade myrelease ./mychart    # Requires helm-diff plugin
   
   # Check Kubernetes events
   kubectl get events --sort-by=.lastTimestamp
   
   # Check pod logs
   kubectl logs -l app=myrelease --previous
   ```

4. **Fix and retry:**
   - Fix the chart or values
   - Test with `helm upgrade --dry-run`
   - Apply the fix

---

## S2: You need to manage Helm charts for 20 microservices. How do you avoid duplication?

**Answer:**

**Create a shared "library chart":**

```
charts/
├── common-library/          # Shared templates
│   ├── Chart.yaml
│   └── templates/
│       ├── _deployment.tpl
│       ├── _service.tpl
│       └── _helpers.tpl
├── service-a/
│   ├── Chart.yaml           # Depends on common-library
│   └── values.yaml          # Service-specific values only
├── service-b/
│   ├── Chart.yaml
│   └── values.yaml
```

**Each service's Chart.yaml:**
```yaml
dependencies:
- name: common-library
  version: "1.0.0"
  repository: "file://../common-library"
```

**Each service's values.yaml:**
```yaml
# Only service-specific values
replicaCount: 3
image:
  repository: service-a
  tag: "1.5.0"
port: 8080
```

**Benefits:**
- Update the library chart once, all services benefit
- Each service only defines what's unique to it
- Consistent patterns across all services