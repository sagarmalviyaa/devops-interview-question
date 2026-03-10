# HashiCorp Vault — Scenario-Based Questions

---

## S1: Your application needs database credentials. How do you integrate it with Vault?

**Answer:**

**Option 1: Vault Agent Sidecar (Kubernetes):**
```yaml
# Pod with Vault agent annotations
metadata:
  annotations:
    vault.hashicorp.com/agent-inject: "true"
    vault.hashicorp.com/role: "myapp"
    vault.hashicorp.com/agent-inject-secret-db: "database/creds/myapp-role"
    vault.hashicorp.com/agent-inject-template-db: |
      {{- with secret "database/creds/myapp-role" -}}
      DB_USER={{ .Data.username }}
      DB_PASS={{ .Data.password }}
      {{- end }}
```

The Vault agent sidecar automatically:
1. Authenticates to Vault using the pod's service account
2. Fetches the database credentials
3. Writes them to a file in the pod
4. Renews them before they expire

**Option 2: Application directly calls Vault API:**
```python
import hvac
client = hvac.Client(url='https://vault.example.com')
client.auth.kubernetes.login(role='myapp', jwt=service_account_token)
creds = client.secrets.database.generate_credentials('myapp-role')
db_user = creds['data']['username']
db_pass = creds['data']['password']
```

**Option 3: External Secrets Operator (Kubernetes):**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: vault-backend
    kind: ClusterSecretStore
  target:
    name: myapp-secrets
  data:
  - secretKey: DB_PASSWORD
    remoteRef:
      key: secret/data/myapp
      property: db_password
```

---

## S2: A secret was accidentally exposed. What's your incident response?

**Answer:**

1. **Revoke the secret immediately:**
   ```bash
   # Revoke a specific lease
   vault lease revoke database/creds/myapp-role/abc123
   
   # Revoke all leases for a path
   vault lease revoke -prefix database/creds/myapp-role/
   
   # Rotate the root credential
   vault write -force database/rotate-root/mydb
   ```

2. **Assess the impact:**
   - Check Vault audit logs: When was the secret accessed? By whom?
   - Was the secret used maliciously?
   - What systems does this secret protect?

3. **Generate new credentials:**
   - Applications using dynamic secrets automatically get new ones
   - For static secrets, update them in Vault and restart affected services

4. **Investigate the root cause:**
   - How was the secret exposed? (logs, code, screenshot)
   - Fix the exposure vector

5. **Prevention:**
   - Use dynamic secrets (auto-expire)
   - Enable audit logging
   - Use short TTLs
   - Scan code and logs for secrets (gitleaks, trufflehog)
   - Train team on secret handling