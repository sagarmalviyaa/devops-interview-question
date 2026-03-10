# Prometheus & Grafana — Scenario-Based Questions

---

## S1: You need to set up monitoring for a Kubernetes-based microservices application. Walk through your approach.

**Answer:**

1. **Deploy Prometheus stack** (using Helm):
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
   ```
   This installs: Prometheus, Grafana, AlertManager, Node Exporter, kube-state-metrics.

2. **Instrument your applications:**
   ```javascript
   // Node.js example with prom-client
   const client = require('prom-client');
   const httpRequestDuration = new client.Histogram({
     name: 'http_request_duration_seconds',
     help: 'Duration of HTTP requests',
     labelNames: ['method', 'route', 'status'],
     buckets: [0.01, 0.05, 0.1, 0.5, 1, 5]
   });
   ```

3. **Add Prometheus annotations to pods:**
   ```yaml
   metadata:
     annotations:
       prometheus.io/scrape: "true"
       prometheus.io/port: "8080"
       prometheus.io/path: "/metrics"
   ```

4. **Create Grafana dashboards:**
   - Import community dashboards (IDs: 315 for K8s, 1860 for Node Exporter)
   - Create custom dashboards for your services

5. **Set up alerts:**
   - High error rate (> 5% for 5 minutes)
   - High latency (p99 > 2 seconds)
   - Pod restarts (> 3 in 15 minutes)
   - Node disk usage (> 85%)
   - Memory usage (> 90%)

---

## S2: Prometheus is consuming too much disk space. How do you manage it?

**Answer:**

1. **Check current storage usage:**
   ```bash
   du -sh /prometheus/data/
   ```

2. **Adjust retention settings:**
   ```yaml
   # prometheus.yml or command-line flags
   --storage.tsdb.retention.time=15d      # Keep data for 15 days
   --storage.tsdb.retention.size=50GB     # Or limit by size
   ```

3. **Reduce cardinality:**
   - Remove unnecessary labels (high-cardinality labels like user IDs)
   - Drop unused metrics with relabeling:
     ```yaml
     metric_relabel_configs:
       - source_labels: [__name__]
         regex: 'go_.*'
         action: drop
     ```

4. **Increase scrape interval** for less critical targets:
   ```yaml
   - job_name: 'less-critical'
     scrape_interval: 60s    # Instead of default 15s
   ```

5. **Use long-term storage** for historical data:
   - Thanos or Cortex for long-term metric storage
   - Keep only recent data in Prometheus

---

## S3: A Grafana dashboard shows a sudden spike in error rates. Walk through your investigation.

**Answer:**

1. **Identify the scope** on the dashboard:
   - Which service(s) are affected?
   - When did the spike start?
   - What's the error rate now vs. normal?

2. **Drill down with PromQL:**
   ```promql
   # Error rate by endpoint
   sum(rate(http_requests_total{status=~"5.."}[5m])) by (route)
   
   # Error rate by pod
   sum(rate(http_requests_total{status=~"5.."}[5m])) by (pod)
   
   # Check if it correlates with a deployment
   kube_deployment_status_observed_generation
   ```

3. **Check for correlating events:**
   - Recent deployments (check deployment timestamps)
   - Infrastructure changes (node issues, scaling events)
   - External dependency failures

4. **Check logs** (if using Loki with Grafana):
   ```
   {app="myservice"} |= "error" | json | status >= 500
   ```

5. **Check resource metrics:**
   - CPU/memory spikes on affected pods
   - Network issues between services
   - Database connection issues

6. **Take action:**
   - If caused by a deployment → rollback
   - If caused by load → scale up
   - If caused by dependency → check dependency health