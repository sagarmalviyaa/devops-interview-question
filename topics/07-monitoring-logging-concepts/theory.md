# Monitoring & Logging Concepts — Theory Questions

---

## Q1: What are the "Three Pillars of Observability"?

**Answer:**
Observability is the ability to understand what's happening inside your system by looking at its outputs. The three pillars are:

**1. Metrics** — Numerical measurements over time
- CPU usage: 75%
- Request count: 1,000/sec
- Error rate: 0.5%
- Response time: 200ms (average)
- Tools: Prometheus, Datadog, CloudWatch

**2. Logs** — Detailed text records of events
- "2026-03-09 10:15:23 ERROR: Database connection failed"
- Structured (JSON) or unstructured (plain text)
- Tools: ELK Stack, Loki, CloudWatch Logs

**3. Traces** — The journey of a request through multiple services
- User request → API Gateway → Auth Service → Database → Response
- Shows where time is spent and where failures occur
- Tools: Jaeger, Zipkin, AWS X-Ray, OpenTelemetry

**How they work together:**
- **Metrics** tell you something is wrong (high error rate)
- **Logs** tell you what went wrong (specific error message)
- **Traces** tell you where it went wrong (which service in the chain)

---

## Q2: What is the difference between monitoring and observability?

**Answer:**

**Monitoring:**
- Watching for known problems
- "Alert me when CPU > 90%"
- You define what to watch in advance
- Answers: "Is the system working?"

**Observability:**
- Understanding the system well enough to diagnose unknown problems
- "Why is this specific user's request slow?"
- The system provides enough data to investigate any question
- Answers: "Why is the system not working?"

**Analogy:**
- Monitoring = Dashboard warning lights in a car (tells you something is wrong)
- Observability = Full diagnostic computer (tells you exactly what's wrong and why)

---

## Q3: What types of metrics should you monitor?

**Answer:**

**The RED Method (for request-driven services):**
- **R**ate — How many requests per second?
- **E**rrors — How many requests are failing?
- **D**uration — How long do requests take?

**The USE Method (for infrastructure resources):**
- **U**tilization — How busy is the resource? (CPU at 80%)
- **S**aturation — How much work is queued? (10 requests waiting)
- **E**rrors — How many errors occurred? (5 disk errors)

**The Four Golden Signals (from Google's SRE book):**
1. **Latency** — Time to serve a request
2. **Traffic** — How much demand is on the system
3. **Errors** — Rate of failed requests
4. **Saturation** — How "full" the system is

**What to monitor at each level:**

| Level | Metrics |
|-------|---------|
| **Infrastructure** | CPU, memory, disk, network |
| **Application** | Request rate, error rate, response time |
| **Business** | Orders per minute, sign-ups, revenue |

---

## Q4: What is alerting? What makes a good alert?

**Answer:**
**Alerting** is automatically notifying the right people when something goes wrong or is about to go wrong.

**A good alert:**
- **Actionable** — Someone needs to do something about it
- **Timely** — Fires before users are affected (or immediately after)
- **Clear** — Describes what's wrong and where
- **Not noisy** — Doesn't fire for non-issues (alert fatigue is real)

**Alert severity levels:**

| Level | Meaning | Response |
|-------|---------|----------|
| **Critical (P1)** | Service is down, users affected | Wake someone up, fix immediately |
| **Warning (P2)** | Something is degraded or approaching a limit | Fix during business hours |
| **Info (P3)** | Something to be aware of | Review when convenient |

**Common alerting mistakes:**
- Too many alerts → alert fatigue → people ignore them
- Alerting on symptoms instead of causes
- No runbook (instructions for what to do when the alert fires)
- Alerting on things that self-heal

**Best practice:** Every alert should have a runbook that answers: "What does this alert mean? What should I check? How do I fix it?"

---

## Q5: What is structured logging and why is it better than plain text logs?

**Answer:**

**Plain text log:**
```
2026-03-09 10:15:23 ERROR Failed to process order 12345 for user john@example.com
```

**Structured log (JSON):**
```json
{
  "timestamp": "2026-03-09T10:15:23Z",
  "level": "ERROR",
  "message": "Failed to process order",
  "order_id": "12345",
  "user_email": "john@example.com",
  "service": "order-service",
  "trace_id": "abc-123-def",
  "error": "PaymentGatewayTimeout"
}
```

**Why structured is better:**
- **Searchable** — Query by any field: "Show me all errors for user john"
- **Parseable** — Machines can process it automatically
- **Consistent** — Same format across all services
- **Aggregatable** — Count errors by type, service, or user
- **Correlatable** — Use `trace_id` to follow a request across services

---

## Q6: What is a dashboard? What should a good dashboard include?

**Answer:**
A **dashboard** is a visual display of key metrics and data, usually in real-time.

**Good dashboard principles:**
- **Purpose-driven** — Each dashboard answers specific questions
- **Glanceable** — Understand the status in 5 seconds
- **Layered** — Overview dashboard → drill down to details
- **Not cluttered** — 5–8 panels maximum per dashboard

**Common dashboard types:**

| Dashboard | Audience | Key Panels |
|-----------|----------|------------|
| **Service Health** | On-call engineer | Error rate, latency, throughput, uptime |
| **Infrastructure** | Platform team | CPU, memory, disk, network per host |
| **Business** | Product/management | Orders, revenue, active users |
| **Deployment** | DevOps team | Deploy frequency, lead time, failure rate |

---

## Q7: What are SLIs, SLOs, and SLAs?

**Answer:**

**SLI (Service Level Indicator)** — A measurement of service performance.
- Example: "99.2% of requests completed in under 200ms"

**SLO (Service Level Objective)** — A target for an SLI.
- Example: "99.9% of requests should complete in under 200ms"

**SLA (Service Level Agreement)** — A contract with consequences if the SLO is not met.
- Example: "If uptime drops below 99.9%, the customer gets a 10% credit"

```
SLI (what you measure) → SLO (what you aim for) → SLA (what you promise)
```

**Error budget:**
- If your SLO is 99.9% uptime, you have a 0.1% error budget
- That's about 43 minutes of downtime per month
- If you've used your error budget, slow down deployments and focus on reliability