# ELK Stack — Theory Questions

---

## Q1: What is the ELK Stack?

**Answer:**
The **ELK Stack** is a collection of three open-source tools for centralized logging:

- **E — Elasticsearch** — A search and analytics engine that stores and indexes logs
- **L — Logstash** — A data processing pipeline that collects, transforms, and sends logs
- **K — Kibana** — A web interface for searching, visualizing, and analyzing logs

**Modern addition:** **Beats** (lightweight data shippers), making it the "Elastic Stack":
- **Filebeat** — Ships log files
- **Metricbeat** — Ships system metrics
- **Packetbeat** — Ships network data
- **Heartbeat** — Ships uptime monitoring data

```
Applications → Filebeat → Logstash → Elasticsearch → Kibana
                (collect)   (process)    (store/index)   (visualize)
```

---

## Q2: What is Elasticsearch?

**Answer:**
**Elasticsearch** is a distributed search and analytics engine built on Apache Lucene. It stores data as JSON documents and provides near real-time search.

**Key concepts:**
- **Index** — A collection of documents (like a database table)
- **Document** — A single log entry or data record (JSON)
- **Shard** — A piece of an index (for distribution across nodes)
- **Replica** — A copy of a shard (for redundancy)
- **Node** — A single Elasticsearch server
- **Cluster** — A group of nodes working together

```json
// Example document (log entry)
{
  "@timestamp": "2026-03-09T10:15:23Z",
  "level": "ERROR",
  "service": "payment-service",
  "message": "Payment processing failed",
  "error_code": "TIMEOUT",
  "user_id": "12345",
  "trace_id": "abc-123"
}
```

---

## Q3: What is Logstash?

**Answer:**
**Logstash** is a data processing pipeline with three stages: input, filter, output.

```ruby
# logstash.conf
input {
  beats {
    port => 5044
  }
  file {
    path => "/var/log/application.log"
  }
}

filter {
  # Parse JSON logs
  json {
    source => "message"
  }
  
  # Parse timestamps
  date {
    match => ["timestamp", "ISO8601"]
  }
  
  # Add geographic info from IP
  geoip {
    source => "client_ip"
  }
  
  # Remove sensitive fields
  mutate {
    remove_field => ["password", "credit_card"]
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

**Common filters:** `grok` (parse unstructured logs), `json`, `mutate`, `date`, `geoip`, `drop`

---

## Q4: What is Filebeat and why use it instead of Logstash?

**Answer:**
**Filebeat** is a lightweight log shipper. It's much lighter than Logstash and is designed to run on every server.

| Feature | Filebeat | Logstash |
|---------|----------|----------|
| Resource usage | Very light (~10MB RAM) | Heavy (~1GB RAM) |
| Purpose | Ship logs | Process and transform logs |
| Filtering | Basic | Advanced (grok, mutate, etc.) |
| Use case | On every server | Central processing |

**Common architecture:**
```
Server 1: App → Filebeat ─┐
Server 2: App → Filebeat ──┤→ Logstash → Elasticsearch → Kibana
Server 3: App → Filebeat ─┘
```

Filebeat is lightweight enough to run on every server. Logstash handles the heavy processing centrally.

---

## Q5: What is Kibana?

**Answer:**
**Kibana** is the visualization layer of the ELK stack. It provides a web interface for:

- **Discover** — Search and browse log data
- **Visualize** — Create charts, graphs, and tables
- **Dashboard** — Combine visualizations into dashboards
- **Lens** — Drag-and-drop visualization builder
- **Alerts** — Set up alerts based on log patterns
- **Dev Tools** — Run Elasticsearch queries directly

**Common Kibana queries (KQL):**
```
level: "ERROR"
service: "payment-service" and level: "ERROR"
message: "timeout" and @timestamp > "2026-03-09"
status >= 500
```

---

## Q6: How do you manage Elasticsearch index lifecycle?

**Answer:**
**Index Lifecycle Management (ILM)** automates the management of indices as they age.

**Phases:**
1. **Hot** — Actively written to and queried (SSD storage)
2. **Warm** — No longer written to, still queried (cheaper storage)
3. **Cold** — Rarely queried (cheapest storage)
4. **Delete** — Removed after retention period

```json
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": { "number_of_shards": 1 },
          "forcemerge": { "max_num_segments": 1 }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

---

## Q7: What is the difference between ELK and Loki?

**Answer:**

| Feature | ELK Stack | Grafana Loki |
|---------|-----------|-------------|
| **Indexing** | Full-text indexing (indexes log content) | Only indexes labels/metadata |
| **Storage cost** | Higher (indexes everything) | Lower (stores raw logs) |
| **Query speed** | Fast for complex queries | Fast for label-based, slower for full-text |
| **Resource usage** | Heavy | Lightweight |
| **Query language** | KQL / Lucene | LogQL |
| **Best for** | Complex log analysis, compliance | Cost-effective logging, Kubernetes |
| **Visualization** | Kibana | Grafana |

**Choose ELK when:** You need powerful full-text search, compliance requirements, complex log analysis.
**Choose Loki when:** You want cost-effective logging, already use Grafana, Kubernetes-native.