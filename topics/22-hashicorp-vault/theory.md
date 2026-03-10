# HashiCorp Vault — Theory Questions

---

## Q1: What is HashiCorp Vault?

**Answer:**
**Vault** is a tool for securely managing secrets (passwords, API keys, certificates, tokens). It provides a central place to store, access, and audit secrets.

**Key features:**
- **Secret storage** — Encrypted storage for any secret
- **Dynamic secrets** — Generate short-lived credentials on demand
- **Encryption as a service** — Encrypt/decrypt data without managing keys
- **Leasing and renewal** — Secrets have TTLs (time-to-live)
- **Audit logging** — Every secret access is logged
- **Access control** — Fine-grained policies

---

## Q2: What are Vault secret engines?

**Answer:**
**Secret engines** are components that store, generate, or encrypt data.

| Engine | Purpose | Example |
|--------|---------|---------|
| **KV (Key-Value)** | Store static secrets | API keys, passwords |
| **AWS** | Generate temporary AWS credentials | Dynamic IAM access keys |
| **Database** | Generate temporary database credentials | Short-lived DB passwords |
| **PKI** | Issue TLS certificates | Auto-generated SSL certs |
| **Transit** | Encryption as a service | Encrypt data without storing keys |
| **SSH** | Manage SSH access | Signed SSH certificates |

```bash
# Store a secret
vault kv put secret/myapp db_password=s3cur3P@ss api_key=abc123

# Read a secret
vault kv get secret/myapp

# Generate dynamic AWS credentials
vault read aws/creds/my-role
# Returns temporary access key and secret key (auto-expires)
```

---

## Q3: What are dynamic secrets and why are they better?

**Answer:**
**Dynamic secrets** are generated on-demand and automatically expire. Unlike static secrets (which exist forever until rotated), dynamic secrets are created when requested and revoked after a TTL.

**Example — Dynamic database credentials:**
```bash
vault read database/creds/my-role
# Returns:
# username: v-token-my-role-abc123
# password: A1B2C3D4E5F6
# lease_duration: 1h
# After 1 hour, these credentials are automatically revoked
```

**Why dynamic secrets are better:**
- **No shared passwords** — Each application gets unique credentials
- **Auto-expiration** — Credentials don't live forever
- **Reduced blast radius** — If leaked, they expire quickly
- **Audit trail** — Know exactly who accessed what and when
- **No rotation needed** — New credentials are generated each time

---

## Q4: What are Vault policies?

**Answer:**
**Policies** define what secrets a user or application can access.

```hcl
# policy: app-read-only
path "secret/data/myapp/*" {
  capabilities = ["read", "list"]
}

path "database/creds/myapp-role" {
  capabilities = ["read"]
}

# Deny access to admin secrets
path "secret/data/admin/*" {
  capabilities = ["deny"]
}
```

```bash
# Create the policy
vault policy write app-read-only app-policy.hcl

# Assign to a token
vault token create -policy=app-read-only
```

---

## Q5: What are Vault authentication methods?

**Answer:**
Auth methods verify the identity of users and applications.

| Method | Use Case |
|--------|----------|
| **Token** | Direct token authentication (default) |
| **AppRole** | Applications and CI/CD pipelines |
| **Kubernetes** | Pods in Kubernetes clusters |
| **AWS IAM** | AWS services and EC2 instances |
| **LDAP/AD** | Enterprise directory integration |
| **OIDC** | SSO with identity providers (Okta, Google) |
| **GitHub** | Authenticate with GitHub tokens |

**Kubernetes auth example:**
```bash
# Configure Kubernetes auth
vault auth enable kubernetes
vault write auth/kubernetes/config \
  kubernetes_host="https://kubernetes.default.svc"

# Create a role for a service account
vault write auth/kubernetes/role/myapp \
  bound_service_account_names=myapp-sa \
  bound_service_account_namespaces=production \
  policies=app-read-only \
  ttl=1h
```

Now pods with the `myapp-sa` service account can authenticate to Vault and read secrets.