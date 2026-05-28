# Kubernetes Secrets 

# What are Secrets?

Kubernetes Secrets are used to store:

* Passwords
* API keys
* Tokens
* Certificates
* Sensitive configuration

Secrets are similar to ConfigMaps, but designed for confidential data.

---

# Problem with Hardcoding Credentials

Example application code:

```python id="6x9p1w"
DB_HOST = "mysql"
DB_USER = "root"
DB_PASSWORD = "mypassword"
```

Problems:

* Credentials exposed in source code
* Difficult to rotate passwords
* Security risk
* Unsafe for Git repositories

---

# Why Not Use ConfigMaps?

ConfigMaps store data as plain text.

Suitable for:

* Application settings
* URLs
* Ports
* Feature flags

Not suitable for:

* Passwords
* Tokens
* Secret keys

---

# Secret Workflow

## Two Steps

1. Create Secret
2. Inject Secret into Pod

---

# Methods to Create Secrets

# 1. Imperative Method

## Create Secret from Literal Values

```bash id="kzcj4t"
kubectl create secret generic app-secret \
  --from-literal=DB_HOST=mysql \
  --from-literal=DB_USER=root \
  --from-literal=DB_PASSWORD=password123
```

---

# Create Secret from File

Example file:

```text id="6x2v9p"
db-secret.properties
```

Contents:

```text id="eahplv"
DB_HOST=mysql
DB_USER=root
DB_PASSWORD=password123
```

Create secret:

```bash id="svjlwm"
kubectl create secret generic app-secret \
  --from-file=db-secret.properties
```

---

# 2. Declarative Method

Create YAML file.

---

# Secret YAML Example

```yaml id="kax9eo"
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_HOST: bXlzcWw=
  DB_USER: cm9vdA==
  DB_PASSWORD: cGFzc3dvcmQxMjM=
```

---

# Important: Values Must Be Base64 Encoded

Kubernetes Secret `data:` field expects Base64 encoded values.

---

# Encode Values

## Linux Command

```bash id="6jjlwm"
echo -n 'mysql' | base64
```

Output:

```text id="ff0wqz"
bXlzcWw=
```

---

# Decode Values

```bash id="zk3c9w"
echo -n 'bXlzcWw=' | base64 --decode
```

Output:

```text id="1mn67r"
mysql
```

---

# Create Secret from YAML

```bash id="2okgzm"
kubectl apply -f secret.yaml
```

---

# View Secrets

## List Secrets

```bash id="mjlwm8"
kubectl get secrets
```

---

## Describe Secret

```bash id="h5olb0"
kubectl describe secret app-secret
```

Shows:

* Metadata
* Keys
* Size

Does NOT show values.

---

## View Secret YAML

```bash id="u5rjso"
kubectl get secret app-secret -o yaml
```

Displays Base64 encoded values.

---

# Inject Secrets into Pods

# Method 1 — Environment Variables

## Using `envFrom`

```yaml id="59fjlwm"
apiVersion: v1
kind: Pod
metadata:
  name: webapp-pod
spec:
  containers:
    - name: webapp
      image: webapp-image
      envFrom:
        - secretRef:
            name: app-secret
```

All secret keys become environment variables.

---

# Method 2 — Single Secret Variable

```yaml id="jlwmc9"
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: DB_PASSWORD
```

---

# Method 3 — Mount as Volume

## Pod Example

```yaml id="90ux8k"
volumes:
  - name: app-secret-volume
    secret:
      secretName: app-secret
```

---

# Secret Volume Behavior

If Secret contains:

```text id="3uxo4g"
DB_HOST
DB_USER
DB_PASSWORD
```

Kubernetes creates files:

```text id="9bqjlwm"
/etc/secrets/DB_HOST
/etc/secrets/DB_USER
/etc/secrets/DB_PASSWORD
```

Each file contains the actual secret value.

---

# Example File Content

```bash id="iy7kvc"
cat /etc/secrets/DB_PASSWORD
```

Output:

```text id="dy2b6m"
password123
```

---

# Secret Types

| Type                             | Purpose                     |
| -------------------------------- | --------------------------- |
| `Opaque`                         | Generic secret              |
| `kubernetes.io/dockerconfigjson` | Docker registry credentials |
| `kubernetes.io/tls`              | TLS certificates            |
| `kubernetes.io/basic-auth`       | Username/password           |

Most common:

* `Opaque`

---

# ConfigMap vs Secret

| Feature        | ConfigMap         | Secret            |
| -------------- | ----------------- | ----------------- |
| Sensitive Data | No                | Yes               |
| Encoding       | Plain text        | Base64 encoded    |
| Purpose        | App configuration | Confidential data |
| Storage        | Non-sensitive     | Sensitive         |

---

# Important Security Note

Base64 is:

* Encoding
* NOT encryption

Anyone can decode it easily.

---

# Better Secret Security Practices

For production:

* Enable etcd encryption
* Use RBAC
* Use external secret managers:

  * HashiCorp Vault
  * AWS Secrets Manager
  * Azure Key Vault
  * Google Secret Manager

---

# Common Commands Summary

| Purpose         | Command                         |
| --------------- | ------------------------------- |
| Create Secret   | `kubectl create secret generic` |
| List Secrets    | `kubectl get secrets`           |
| Describe Secret | `kubectl describe secret`       |
| View YAML       | `kubectl get secret -o yaml`    |
| Encode Value    | `echo -n 'text' \| base64`      |
| Decode Value    | `base64 --decode`               |

---

# Quick Summary

* Secrets store sensitive data in Kubernetes.
* Similar to ConfigMaps but intended for confidential information.
* Secret values are Base64 encoded.
* Secrets can be injected:

  * As environment variables
  * As files via volumes
* Base64 encoding is not strong security by itself.
