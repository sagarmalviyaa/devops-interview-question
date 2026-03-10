# Chaos Engineering — Scenario-Based Questions

---

## S1: Your team wants to start Chaos Engineering. How do you introduce it?

**Answer:**

**Phase 1: Foundation (Week 1-2)**
1. **Assess readiness:**
   - Do you have monitoring and alerting? (Required!)
   - Do you have auto-scaling and self-healing?
   - Do you have runbooks for common failures?

2. **Get buy-in:**
   - Present the business case (prevent outages, improve reliability)
   - Start with a "Game Day" — a planned failure exercise

**Phase 2: First Experiments (Week 3-4)**
3. **Start in staging:**
   ```yaml
   # LitmusChaos experiment: Kill a pod
   apiVersion: litmuschaos.io/v1alpha1
   kind: ChaosEngine
   metadata:
     name: pod-kill-test
   spec:
     appinfo:
       appns: staging
       applabel: app=myservice
     chaosServiceAccount: litmus-admin
     experiments:
     - name: pod-delete
       spec:
         components:
           env:
           - name: TOTAL_CHAOS_DURATION
             value: '30'
           - name: CHAOS_INTERVAL
             value: '10'
   ```

4. **Define steady state:**
   - Error rate < 0.1%
   - p99 latency < 500ms
   - All health checks passing

5. **Run the experiment and observe:**
   - Did the system self-heal?
   - Was there any user impact?
   - How long did recovery take?

**Phase 3: Expand (Month 2+)**
6. **Increase complexity:**
   - Network latency injection
   - Node failures
   - Dependency failures

7. **Move to production** (carefully):
   - Start with low-impact experiments
   - Run during business hours
   - Have the team on standby

8. **Automate:**
   - Schedule regular chaos experiments
   - Integrate into CI/CD pipeline

---

## S2: During a chaos experiment, you killed a database pod and the application went completely down instead of failing gracefully. What do you do?

**Answer:**

**Immediate:**
1. **Stop the experiment** — Restore the database pod
2. **Verify recovery** — Ensure the application is back to normal
3. **Document what happened** — Timeline, impact, recovery steps

**Root cause analysis:**
4. **Why did the app go completely down?**
   - No connection pooling with retry logic?
   - No circuit breaker for database calls?
   - Only one database replica?
   - No graceful degradation (showing cached data)?

**Fixes:**
5. **Add resilience patterns:**
   - **Connection retry with backoff:**
     ```javascript
     const pool = new Pool({
       connectionTimeoutMillis: 5000,
       max: 20,
       retryDelay: 1000,
     });
     ```
   - **Circuit breaker:**
     ```javascript
     const breaker = new CircuitBreaker(dbQuery, {
       timeout: 3000,
       errorThresholdPercentage: 50,
       resetTimeout: 30000,
     });
     ```
   - **Graceful degradation:**
     - Return cached data when DB is unavailable
     - Show a friendly error message instead of crashing
   
6. **Infrastructure improvements:**
   - Run database with replicas (RDS Multi-AZ, PostgreSQL streaming replication)
   - Use Pod Disruption Budgets
   - Add readiness probes that check database connectivity

7. **Re-run the experiment** to verify the fixes work

**Key takeaway:** The chaos experiment succeeded — it found a real weakness before it caused an unplanned outage. This is exactly what Chaos Engineering is for.