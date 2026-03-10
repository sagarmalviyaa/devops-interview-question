# Service Mesh — Scenario-Based Questions

---

## S1: You need to implement a canary deployment for a critical service. How do you use Istio?

**Answer:**

1. **Deploy both versions:**
   ```bash
   # v1 is already running
   kubectl apply -f deployment-v2.yaml    # Deploy v2 alongside v1
   ```

2. **Create a VirtualService for traffic splitting:**
   ```yaml
   apiVersion: networking.istio.io/v1beta1
   kind: VirtualService
   metadata:
     name: payment-service
   spec:
     hosts:
     - payment-service
     http:
     - route:
       - destination:
           host: payment-service
           subset: v1
         weight: 95
       - destination:
           host: payment-service
           subset: v2
         weight: 5     # Start with 5% to v2
   ```

3. **Monitor v2 closely:**
   - Check error rates in Grafana/Kiali
   - Compare latency between v1 and v2
   - Check business metrics

4. **Gradually increase traffic:**
   - 5% → 10% → 25% → 50% → 100%
   - At each step, verify metrics are healthy

5. **If issues arise, rollback instantly:**
   ```yaml
   # Set v2 weight back to 0
   weight: 0
   ```

---

## S2: Services in your mesh are experiencing intermittent failures. How do you add resilience?

**Answer:**

**Retries:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
spec:
  http:
  - route:
    - destination:
        host: payment-service
    retries:
      attempts: 3
      perTryTimeout: 2s
      retryOn: 5xx,reset,connect-failure
```

**Circuit Breaking:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
spec:
  host: payment-service
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        h2UpgradePolicy: DEFAULT
        http1MaxPendingRequests: 100
        http2MaxRequests: 1000
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
```

**Timeouts:**
```yaml
http:
- timeout: 10s
  route:
  - destination:
      host: payment-service
```

This combination ensures: failed requests are retried, unhealthy instances are removed from the pool, and slow services don't block callers indefinitely.