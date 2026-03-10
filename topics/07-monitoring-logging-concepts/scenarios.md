# Monitoring & Logging — Scenario-Based Questions

---

## S1: You're setting up monitoring for a new microservices application. What's your approach?

**Answer:**

1. **Define what to monitor at each layer:**

   **Infrastructure level:**
   - CPU, memory, disk usage per host
   - Network throughput and errors
   - Container resource usage

   **Application level (per service):**
   - Request rate (requests/second)
   - Error rate (% of 5xx responses)
   - Response time (p50, p95, p99 latency)
   - Active connections

   **Business level:**
   - User sign-ups per hour
   - Orders completed per minute
   - Payment success rate

2. **Set up the monitoring stack:**
   - **Prometheus** for metrics collection
   - **Grafana** for dashboards
   - **Loki or ELK** for log aggregation
   - **Jaeger** for distributed tracing
   - **AlertManager** for alerting

3. **Create dashboards:**
   - Overview dashboard (all services at a glance)
   - Per-service dashboard (detailed metrics)
   - Infrastructure dashboard (hosts and containers)

4. **Set up alerts:**
   ```yaml
   # High error rate
   - alert: HighErrorRate
     expr: rate(http_requests_total{status=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
     for: 5m
     labels:
       severity: critical
     annotations:
       summary: "Error rate above 5% for {{ $labels.service }}"
   ```

5. **Set up on-call rotation** with PagerDuty or Opsgenie

---

## S2: Your monitoring shows a gradual increase in response time over the past week. How do you investigate?

**Answer:**

1. **Identify the scope:**
   - Is it all endpoints or specific ones?
   - Is it all users or specific regions?
   - When did it start?

2. **Check for correlating events:**
   - Was there a deployment around that time?
   - Did traffic increase?
   - Were there infrastructure changes?

3. **Check resource utilization:**
   ```
   CPU trending up? → Might need more compute
   Memory trending up? → Possible memory leak
   Disk I/O increasing? → Database growing, need optimization
   Network latency up? → Network issue or increased payload size
   ```

4. **Check the database:**
   - Slow query logs — are queries taking longer?
   - Table sizes — has data grown significantly?
   - Missing indexes — new queries without proper indexes?

5. **Check for memory leaks:**
   - Is memory usage growing without releasing?
   - Restart a service and see if latency drops (confirms leak)

6. **Check external dependencies:**
   - Are third-party APIs slower?
   - Is the CDN performing well?

7. **Common causes of gradual slowdown:**
   - Database growing without index optimization
   - Memory leak in the application
   - Increasing traffic without scaling
   - Log files filling up disk space
   - Connection pool exhaustion

---

## S3: You receive 500 alerts in one night and the on-call engineer ignored most of them. How do you fix the alerting system?

**Answer:**

This is **alert fatigue** — too many alerts cause people to stop paying attention.

1. **Audit all existing alerts:**
   - Categorize each alert: Critical / Warning / Informational / Noise
   - Delete alerts that never require action
   - Combine related alerts into single, meaningful alerts

2. **Apply alerting best practices:**
   - **Alert on symptoms, not causes** — "Error rate > 5%" instead of "CPU > 80%"
   - **Alert on user impact** — "Users seeing errors" matters more than "one pod restarted"
   - **Set proper thresholds** — Use historical data to set realistic thresholds
   - **Add proper duration** — Don't alert on 1-second spikes; require sustained issues

3. **Create alert tiers:**
   ```
   Critical → PagerDuty (wake someone up)
   Warning  → Slack channel (fix during business hours)
   Info     → Dashboard only (no notification)
   ```

4. **Add runbooks to every alert:**
   - What does this alert mean?
   - What should I check first?
   - How do I fix it?
   - Who to escalate to?

5. **Review alerts regularly:**
   - Weekly: Review all alerts that fired
   - Monthly: Remove or tune alerts that are too noisy
   - After incidents: Add alerts that would have caught the issue earlier