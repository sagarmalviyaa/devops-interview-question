# Chaos Engineering — Theory Questions

---

## Q1: What is Chaos Engineering?

**Answer:**
**Chaos Engineering** is the discipline of experimenting on a system to build confidence in its ability to withstand turbulent conditions in production. You intentionally inject failures to find weaknesses before they cause real outages.

**Core idea:** "Let's break things on purpose, in a controlled way, to find problems before our users do."

**Principles (from Netflix):**
1. **Build a hypothesis** around steady-state behavior
2. **Vary real-world events** (server crashes, network issues, disk full)
3. **Run experiments in production** (or production-like environments)
4. **Automate experiments** to run continuously
5. **Minimize blast radius** — Start small, expand gradually

---

## Q2: What are common chaos experiments?

**Answer:**

| Experiment | What It Tests | Tools |
|-----------|---------------|-------|
| **Kill a pod/container** | Does the system self-heal? | Chaos Monkey, LitmusChaos |
| **Kill a node** | Does the workload migrate? | kube-monkey, Gremlin |
| **Network latency** | Does the app handle slow responses? | tc (traffic control), Toxiproxy |
| **Network partition** | What happens when services can't communicate? | Pumba, Chaos Mesh |
| **CPU/Memory stress** | Does the app degrade gracefully under resource pressure? | stress-ng, Chaos Mesh |
| **Disk full** | Does the app handle disk exhaustion? | LitmusChaos |
| **DNS failure** | What happens when DNS resolution fails? | Chaos Mesh |
| **Dependency failure** | What if the database/cache goes down? | Gremlin, Toxiproxy |

---

## Q3: What are popular Chaos Engineering tools?

**Answer:**

| Tool | Platform | Key Feature |
|------|----------|-------------|
| **Chaos Monkey** (Netflix) | AWS | Randomly kills instances |
| **LitmusChaos** | Kubernetes | Cloud-native, large experiment hub |
| **Chaos Mesh** | Kubernetes | Rich fault injection, web UI |
| **Gremlin** | Any | Commercial, easy to use, safe |
| **Toxiproxy** | Any | Simulates network conditions |
| **AWS Fault Injection Simulator** | AWS | AWS-native chaos tool |

---

## Q4: What is the difference between Chaos Engineering and testing?

**Answer:**

| Feature | Traditional Testing | Chaos Engineering |
|---------|-------------------|-------------------|
| **Environment** | Test/staging | Production (ideally) |
| **Known outcomes** | Tests have expected results | Experiments explore unknowns |
| **Scope** | Individual components | System-wide behavior |
| **Goal** | Verify functionality | Discover weaknesses |
| **Frequency** | During development | Continuously |

**Chaos Engineering complements testing** — testing verifies known behaviors, chaos engineering discovers unknown failure modes.

---

## Q5: How do you implement Chaos Engineering safely?

**Answer:**

1. **Start in non-production:**
   - Run experiments in staging first
   - Only move to production when confident

2. **Start small:**
   - Kill one pod, not all pods
   - Add 100ms latency, not 10 seconds
   - Affect 1% of traffic, not 100%

3. **Have a hypothesis:**
   - "If we kill one pod, the service should remain available because we have 3 replicas"
   - Measure: error rate, latency, availability

4. **Have a rollback plan:**
   - Know how to stop the experiment immediately
   - Have runbooks ready for unexpected outcomes

5. **Monitor during experiments:**
   - Watch dashboards in real-time
   - Set up alerts for anomalies
   - Have the team ready to respond

6. **Communicate:**
   - Inform the team before running experiments
   - Schedule during business hours (when people are available to respond)
   - Document results and share learnings

7. **Automate gradually:**
   - Start with manual experiments
   - Automate as confidence grows
   - Run automated chaos experiments in CI/CD