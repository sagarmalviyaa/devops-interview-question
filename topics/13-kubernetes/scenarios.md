# Kubernetes — Scenario-Based Questions

---

## S1: A pod is stuck in CrashLoopBackOff. How do you debug it?

**Answer:**

**CrashLoopBackOff** means the container keeps crashing and Kubernetes keeps restarting it, with increasing delays between restarts.

1. **Check pod status and events:**
   ```bash
   kubectl describe pod myapp-xyz
   # Look at the "Events" section at the bottom
   # Look at "Last State" for exit code and reason
   ```

2. **Check container logs:**
   ```bash
   kubectl logs myapp-xyz
   kubectl logs myapp-xyz --previous    # Logs from the last crashed container
   ```

3. **Common causes:**

   | Exit Code | Meaning | Fix |
   |-----------|---------|-----|
   | 0 | Container exited successfully | Command finished — ensure it runs continuously |
   | 1 | Application error | Check logs for the error |
   | 137 | OOM Killed | Increase memory limits |
   | 139 | Segmentation fault | Application bug |

4. **Debug interactively:**
   ```bash
   # Override the command to keep the container running
   kubectl run debug --image=myapp:1.0 --command -- sleep 3600
   kubectl exec -it debug -- /bin/sh
   # Now investigate inside the container
   ```

5. **Check configuration:**
   - Are environment variables set correctly?
   - Are ConfigMaps and Secrets mounted?
   - Can the pod reach its dependencies (database, other services)?
   - Are resource limits too low?

---

## S2: You need to perform a zero-downtime deployment. How do you ensure it?

**Answer:**

1. **Use a Deployment with rolling update strategy:**
   ```yaml
   spec:
     strategy:
       type: RollingUpdate
       rollingUpdate:
         maxSurge: 1          # 1 extra pod during update
         maxUnavailable: 0    # Never have fewer than desired replicas
   ```

2. **Configure readiness probes:**
   ```yaml
   readinessProbe:
     httpGet:
       path: /health
       port: 3000
     initialDelaySeconds: 10
     periodSeconds: 5
   ```
   This ensures traffic only goes to pods that are ready.

3. **Handle graceful shutdown:**
   ```yaml
   spec:
     terminationGracePeriodSeconds: 60
     containers:
     - name: myapp
       lifecycle:
         preStop:
           exec:
             command: ["/bin/sh", "-c", "sleep 10"]
   ```
   This gives the pod time to finish in-flight requests.

4. **Use Pod Disruption Budgets:**
   ```yaml
   apiVersion: policy/v1
   kind: PodDisruptionBudget
   metadata:
     name: myapp-pdb
   spec:
     minAvailable: 2
     selector:
       matchLabels:
         app: myapp
   ```

5. **Run multiple replicas** (at least 2, preferably 3+)

6. **Test the deployment:**
   ```bash
   kubectl rollout status deployment/myapp
   # Watch the rollout progress
   
   kubectl rollout undo deployment/myapp
   # Rollback if something goes wrong
   ```

---

## S3: A Kubernetes node is running out of disk space. What happens and how do you fix it?

**Answer:**

**What happens:**
- Kubelet starts evicting pods (DiskPressure condition)
- New pods can't be scheduled on this node
- Existing pods may be terminated

**Diagnosis:**
```bash
kubectl describe node node-name
# Look for "Conditions" — DiskPressure: True

# SSH into the node
df -h                                    # Check disk usage
du -sh /var/lib/docker/* | sort -rh      # Docker storage
du -sh /var/log/* | sort -rh             # Logs
crictl images | sort -k3 -rh             # Container images
```

**Fix:**
```bash
# Clean unused Docker/containerd resources
crictl rmi --prune
docker system prune -af    # If using Docker

# Clean old logs
journalctl --vacuum-size=500M
find /var/log -name "*.log" -mtime +7 -delete

# Remove unused images
kubectl get pods --all-namespaces -o jsonpath='{.items[*].spec.containers[*].image}' | tr ' ' '\n' | sort -u
# Compare with all images on the node and remove unused ones
```

**Prevention:**
- Set up monitoring alerts for disk usage > 80%
- Configure image garbage collection in kubelet
- Use ephemeral storage limits on pods
- Set up log rotation

---

## S4: You need to troubleshoot networking issues between two pods. Walk through your process.

**Answer:**

1. **Verify pods are running:**
   ```bash
   kubectl get pods -o wide
   # Note the pod IPs and which nodes they're on
   ```

2. **Test DNS resolution:**
   ```bash
   kubectl exec pod-a -- nslookup service-b
   kubectl exec pod-a -- nslookup service-b.namespace.svc.cluster.local
   ```

3. **Test connectivity:**
   ```bash
   kubectl exec pod-a -- curl -v http://service-b:8080/health
   kubectl exec pod-a -- nc -zv service-b 8080
   kubectl exec pod-a -- ping <pod-b-ip>
   ```

4. **Check Services:**
   ```bash
   kubectl get svc service-b
   kubectl describe svc service-b
   # Check if Endpoints list includes pod-b's IP
   kubectl get endpoints service-b
   ```

5. **Check NetworkPolicies:**
   ```bash
   kubectl get networkpolicies -n namespace
   # A NetworkPolicy might be blocking traffic
   ```

6. **Check CoreDNS:**
   ```bash
   kubectl get pods -n kube-system -l k8s-app=kube-dns
   kubectl logs -n kube-system -l k8s-app=kube-dns
   ```

7. **Use a debug pod:**
   ```bash
   kubectl run netdebug --image=nicolaka/netshoot -it --rm -- /bin/bash
   # This image has all networking tools pre-installed
   ```

---

## S5: Your Kubernetes cluster needs to handle sensitive workloads. How do you secure it?

**Answer:**

1. **RBAC (Role-Based Access Control):**
   - Create specific roles with minimum permissions
   - Don't use cluster-admin for regular users
   - Use service accounts for applications

2. **Network Policies:**
   - Default deny all traffic
   - Explicitly allow only needed communication
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: default-deny-all
   spec:
     podSelector: {}
     policyTypes:
     - Ingress
     - Egress
   ```

3. **Pod Security:**
   ```yaml
   securityContext:
     runAsNonRoot: true
     runAsUser: 1000
     readOnlyRootFilesystem: true
     allowPrivilegeEscalation: false
     capabilities:
       drop: ["ALL"]
   ```

4. **Secrets Management:**
   - Use external secret managers (Vault, AWS Secrets Manager)
   - Enable encryption at rest for etcd
   - Use Sealed Secrets or External Secrets Operator

5. **Image Security:**
   - Scan images with Trivy before deployment
   - Use trusted base images only
   - Enforce image signing and verification
   - Use private registries

6. **Audit Logging:**
   - Enable Kubernetes audit logs
   - Monitor for suspicious API calls

7. **Keep cluster updated** — Apply security patches promptly