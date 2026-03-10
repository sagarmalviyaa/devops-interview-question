# ELK Stack — Scenario-Based Questions

---

## S1: Your Elasticsearch cluster is running out of disk space. How do you handle it?

**Answer:**

1. **Check current usage:**
   ```bash
   curl -s localhost:9200/_cat/indices?v&s=store.size:desc | head -20
   curl -s localhost:9200/_cat/allocation?v
   ```

2. **Immediate fixes:**
   ```bash
   # Delete old indices
   curl -X DELETE localhost:9200/logs-2026.01.*
   
   # Force merge to reclaim space
   curl -X POST localhost:9200/logs-2026.02/_forcemerge?max_num_segments=1
   ```

3. **Set up ILM (Index Lifecycle Management):**
   - Automatically delete indices older than 30 days
   - Move old indices to cheaper storage
   - Roll over indices when they reach a size limit

4. **Reduce data volume:**
   - Filter out unnecessary logs at the Filebeat/Logstash level
   - Reduce log verbosity (don't ship DEBUG logs to production)
   - Remove unnecessary fields before indexing

5. **Scale the cluster:**
   - Add more data nodes
   - Use hot-warm-cold architecture

---

## S2: Developers need to search production logs quickly during an incident. How do you set this up?

**Answer:**

1. **Create a dedicated Kibana dashboard for incidents:**
   - Pre-built searches for common error patterns
   - Filters by service, severity, and time range
   - Saved queries for frequent investigations

2. **Set up structured logging** in all applications:
   ```json
   {
     "timestamp": "2026-03-09T10:15:23Z",
     "level": "ERROR",
     "service": "order-service",
     "trace_id": "abc-123",
     "user_id": "12345",
     "message": "Order processing failed",
     "error": "DatabaseConnectionTimeout"
   }
   ```

3. **Create useful Kibana saved searches:**
   ```
   # All errors in the last hour
   level: "ERROR" and @timestamp > now-1h
   
   # Trace a specific request
   trace_id: "abc-123"
   
   # All errors for a specific service
   service: "order-service" and level: ("ERROR" or "FATAL")
   ```

4. **Set up RBAC** — Developers can read logs but not modify the cluster

5. **Add correlation IDs** — Every request gets a unique trace_id that flows through all services, making it easy to trace a request end-to-end

---

## S3: Log ingestion is falling behind — Logstash can't keep up with the volume. How do you fix it?

**Answer:**

1. **Add a buffer (Kafka/Redis) between Filebeat and Logstash:**
   ```
   Filebeat → Kafka → Logstash → Elasticsearch
   ```
   Kafka absorbs traffic spikes and lets Logstash process at its own pace.

2. **Scale Logstash horizontally:**
   - Run multiple Logstash instances
   - Each reads from different Kafka partitions

3. **Optimize Logstash pipeline:**
   ```ruby
   # Increase batch size and workers
   pipeline.batch.size: 500
   pipeline.workers: 8     # Match CPU cores
   ```

4. **Simplify filters:**
   - Remove unnecessary grok patterns
   - Use Filebeat processors for simple transformations
   - Move complex processing to Elasticsearch ingest pipelines

5. **Ship directly to Elasticsearch** (skip Logstash if processing isn't needed):
   ```yaml
   # filebeat.yml
   output.elasticsearch:
     hosts: ["elasticsearch:9200"]
     pipeline: "my-ingest-pipeline"
   ```

6. **Reduce log volume at the source:**
   - Don't log DEBUG in production
   - Sample high-volume logs (log 1 in 10 requests)
   - Filter out health check logs