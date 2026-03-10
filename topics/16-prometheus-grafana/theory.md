# Prometheus & Grafana — Theory Questions

---

## Q1: What is Prometheus?

**Answer:**
**Prometheus** is an open-source monitoring and alerting system. It collects numerical metrics from your applications and infrastructure, stores them as time-series data, and lets you query and alert on them.

**Key features:**
- **Pull-based** — Prometheus scrapes (pulls) metrics from targets
- **Time-series database** — Stores metrics with timestamps
- **PromQL** — Powerful query language for metrics
- **Service discovery** — Automatically finds targets to monitor
- **Alerting** — Built-in alert rules with AlertManager integration

---

## Q2: How does Prometheus collect metrics?

**Answer:**
Prometheus uses a **pull model**: it periodically sends HTTP requests to targets and scrapes their `/metrics` endpoint.

```
Application → exposes /metrics endpoint
Prometheus → scrapes /metrics every 15s (configurable)
```

**prometheus.yml configuration:**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'myapp'
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
  
  - job_name: 'node'
    static_configs:
      - targets: ['node1:9100', 'node2:9100']
  
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
```

**Exporters** expose metrics for services that don't natively support Prometheus:
- **Node Exporter** — Linux system metrics (CPU, memory, disk)
- **MySQL Exporter** — Database metrics
- **Nginx Exporter** — Web server metrics
- **Blackbox Exporter** — Probe endpoints (HTTP, TCP, ICMP)

---

## Q3: What are the Prometheus metric types?

**Answer:**

| Type | Description | Example |
|------|-------------|---------|
| **Counter** | Only goes up (resets on restart) | Total requests, total errors |
| **Gauge** | Can go up or down | Temperature, memory usage, active connections |
| **Histogram** | Counts observations in buckets | Request duration distribution |
| **Summary** | Like histogram but calculates percentiles client-side | Request duration percentiles |

```
# Counter
http_requests_total{method="GET", status="200"} 1234

# Gauge
node_memory_available_bytes 4294967296

# Histogram
http_request_duration_seconds_bucket{le="0.1"} 500
http_request_duration_seconds_bucket{le="0.5"} 800
http_request_duration_seconds_bucket{le="1.0"} 950
```

---

## Q4: What is PromQL? Show common queries.

**Answer:**
**PromQL (Prometheus Query Language)** is used to query and aggregate metrics.

```promql
# Instant query — current value
http_requests_total

# Filter by labels
http_requests_total{method="GET", status="200"}

# Rate — requests per second over 5 minutes
rate(http_requests_total[5m])

# Error rate percentage
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# 95th percentile response time
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# CPU usage percentage
100 - (avg by(instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory usage percentage
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# Top 5 pods by CPU usage
topk(5, sum by(pod) (rate(container_cpu_usage_seconds_total[5m])))

# Aggregation
sum(http_requests_total) by (method)
avg(node_cpu_seconds_total) by (instance)
max(node_memory_MemTotal_bytes) by (instance)
```

---

## Q5: What is AlertManager?

**Answer:**
**AlertManager** handles alerts sent by Prometheus. It manages deduplication, grouping, routing, and notification.

**Alert rule (in Prometheus):**
```yaml
groups:
- name: app-alerts
  rules:
  - alert: HighErrorRate
    expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "High error rate on {{ $labels.instance }}"
      description: "Error rate is {{ $value | humanizePercentage }}"

  - alert: HighMemoryUsage
    expr: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) > 0.9
    for: 10m
    labels:
      severity: warning
```

**AlertManager routing:**
```yaml
route:
  receiver: 'slack-default'
  routes:
  - match:
      severity: critical
    receiver: 'pagerduty'
  - match:
      severity: warning
    receiver: 'slack-warnings'

receivers:
- name: 'pagerduty'
  pagerduty_configs:
  - service_key: '<key>'
- name: 'slack-default'
  slack_configs:
  - channel: '#alerts'
    api_url: 'https://hooks.slack.com/...'
```

---

## Q6: What is Grafana?

**Answer:**
**Grafana** is an open-source visualization and dashboarding platform. It connects to data sources (Prometheus, Elasticsearch, CloudWatch, etc.) and creates beautiful, interactive dashboards.

**Key features:**
- **Multiple data sources** — Prometheus, Loki, Elasticsearch, MySQL, CloudWatch, etc.
- **Rich visualizations** — Graphs, gauges, tables, heatmaps, logs
- **Alerting** — Can also send alerts based on dashboard queries
- **Templating** — Variables for dynamic dashboards
- **Sharing** — Export dashboards, embed panels, share links
- **Plugins** — Extend with community plugins

**Common dashboard panels:**
- Time series graph (request rate over time)
- Gauge (current CPU usage)
- Stat (total requests today)
- Table (top endpoints by latency)
- Heatmap (request distribution)
- Logs panel (with Loki)

---

## Q7: What is the difference between Prometheus and Grafana?

**Answer:**

| Feature | Prometheus | Grafana |
|---------|-----------|---------|
| **Purpose** | Collect and store metrics | Visualize and dashboard metrics |
| **Data** | Time-series database | Visualization layer (no storage) |
| **Query** | PromQL | Supports multiple query languages |
| **Alerting** | Rule-based alerts → AlertManager | Visual alert configuration |
| **UI** | Basic expression browser | Rich, customizable dashboards |

**They work together:** Prometheus collects and stores data → Grafana visualizes it.

---

## Q8: What is Prometheus service discovery?

**Answer:**
**Service discovery** automatically finds targets to scrape, instead of manually listing them.

**Kubernetes service discovery:**
```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```

**Other discovery methods:**
- **Consul** — Service registry
- **AWS EC2** — Discover instances by tags
- **DNS** — Discover via DNS SRV records
- **File-based** — Read targets from a JSON/YAML file (auto-reloads)

This means when you add a new pod with the right annotations, Prometheus automatically starts monitoring it.