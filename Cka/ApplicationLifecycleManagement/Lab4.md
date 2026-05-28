# Kubernetes Secrets Lab 

---

# 1. Inspect Existing Secrets

## List Secrets (default namespace)

```bash id="9k2v1a"
kubectl get secrets
```

Example result:

* `default-token-xxxxx` (auto-created service account secret)

👉 Answer: **1 secret exists**

---

# 2. Inspect Default Token Secret

## Describe Secret

```bash id="c8xq3m"
kubectl describe secret default-token-xxxxx
```

### Data section typically contains:

* `ca.crt`
* `namespace`
* `token`

👉 Total keys: **3**

---

# 3. Secret Type

From describe output:

```text id="v2n8qp"
Type: kubernetes.io/service-account-token
```

👉 This is a **Service Account Token Secret**

---

# 4. Identify Non-Secret Field

From data:

* ca.crt
* namespace
* token

👉 `type` is **NOT part of data**

---

# 5. Application Failure Scenario

## Observed issue:

Web application cannot connect to MySQL:

* DB host missing
* DB user missing
* DB password missing

👉 Root cause: **Secret not created**

---

# 6. Create Secret (Imperative Approach)

## Command

```bash id="m9xk2d"
kubectl create secret generic db-secret \
  --from-literal=DB_HOST=sql01 \
  --from-literal=DB_USER=root \
  --from-literal=DB_PASSWORD=password123
```

---

## Verify Secret

```bash id="p4q8lm"
kubectl describe secret db-secret
```

Output confirms:

* DB_HOST
* DB_USER
* DB_PASSWORD

---

# 7. Key Concept — Case Sensitivity

Secret keys are case-sensitive:

```text id="a1d9pw"
DB_HOST ≠ db_host
```

---

# 8. Configure Pod to Use Secret

## Method: envFrom (recommended for labs)

```yaml id="s7n2kc"
envFrom:
  - secretRef:
      name: db-secret
```

---

## Alternative: Single Variable Injection

```yaml id="x3m8qv"
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

---

# 9. Replace Pod After Edit

Since Pods are immutable:

```bash id="k2v9zd"
kubectl replace --force -f pod.yaml
```

This will:

* Delete existing Pod
* Recreate new Pod

---

# 10. Verify Injection

```bash id="w1n7qp"
kubectl describe pod webapp
```

Look for:

```text id="c9m2xz"
Environment Variables From:
  db-secret
```

---

# 11. Validate Application

After correct secret injection:

* Web app connects to MySQL successfully
* Status changes from **FAIL → SUCCESS**

---

# 12. Important Security Insight

## Kubernetes Secrets are NOT truly encrypted by default

* Stored in **etcd in Base64 format**
* Base64 ≠ encryption

Risk:

* Anyone with etcd or API access can decode secrets

---

# 13. Better Security Practices

For production-grade setups:

### Enable:

* etcd encryption at rest
* RBAC restrictions

### Prefer external systems:

* AWS Secrets Manager
* HashiCorp Vault
* Azure Key Vault
* GCP Secret Manager

---

# 14. Secrets vs ConfigMaps (Final View)

| Feature        | ConfigMap     | Secret                            |
| -------------- | ------------- | --------------------------------- |
| Data type      | Non-sensitive | Sensitive                         |
| Encoding       | Plain text    | Base64                            |
| Use case       | Config        | Credentials                       |
| Security level | Low           | Medium (not encrypted by default) |

---

# 15. Core Exam Takeaways (CKA/CKAD)

### Must-know commands:

```bash id="t6p3lm"
kubectl get secrets
kubectl describe secret <name>
kubectl create secret generic
kubectl get secret -o yaml
```

---

# 16. Core Concept Summary

* Secrets store sensitive data in Kubernetes
* Created imperatively or declaratively
* Injected into Pods via:

  * `envFrom`
  * `secretKeyRef`
  * Volumes
* Must use Base64 encoding in YAML
* Require RBAC + encryption for production security

---
