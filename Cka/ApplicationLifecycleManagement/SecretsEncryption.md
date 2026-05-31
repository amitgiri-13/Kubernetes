# Kubernetes Secrets Encryption at Rest

## What is the Problem?

[Reference](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)

By default, Kubernetes Secrets are:

* **Base64 encoded**, not encrypted.
* Stored in **etcd** in plaintext.
* Anyone with access to etcd can view secret values.

Example:

```bash
kubectl create secret generic my-secret \
  --from-literal=key1=super-secret
```

View secret:

```bash
kubectl get secret my-secret -o yaml
```

Decode:

```bash
echo "c3VwZXItc2VjcmV0" | base64 -d
```

Output:

```bash
super-secret
```

> Base64 encoding ≠ Encryption

---

# Verify Secret Storage in etcd

Install etcd client:

```bash
apt install etcd-client
```

Retrieve secret directly from etcd:

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secret
```

Result:

* Secret value is visible.
* Indicates data is stored unencrypted.

---

# Check if Encryption at Rest is Enabled

Check kube-apiserver process:

```bash
ps aux | grep kube-apiserver
```

Look for:

```bash
--encryption-provider-config=
```

If missing:

```text
Encryption at Rest is NOT enabled
```

For kubeadm clusters:

```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```

Verify the flag exists.

---

# Create Encryption Configuration

Generate a 32-byte key:

```bash
head -c 32 /dev/urandom | base64
```

Create:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration

resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE64-ENCODED-KEY>
      - identity: {}
```

Save as:

```bash
enc.yaml
```

---

# Provider Order Matters

Example:

```yaml
providers:
  - aescbc:
  - identity:
```

### Encryption

The **first provider** is used for encryption.

### Decryption

Kubernetes tries all providers in order.

### Bad Example

```yaml
providers:
  - identity:
  - aescbc:
```

Result:

```text
No encryption occurs
```

because `identity` means:

```text
Store data as-is
```

---

# Make Configuration Available to kube-apiserver

Create directory:

```bash
mkdir -p /etc/kubernetes/enc
```

Move file:

```bash
mv enc.yaml /etc/kubernetes/enc/
```

---

# Update kube-apiserver Manifest

File:

```bash
/etc/kubernetes/manifests/kube-apiserver.yaml
```

Add argument:

```yaml
- --encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

Add volume mount:

```yaml
volumeMounts:
- mountPath: /etc/kubernetes/enc
  name: enc
  readOnly: true
```

Add volume:

```yaml
volumes:
- name: enc
  hostPath:
    path: /etc/kubernetes/enc
    type: DirectoryOrCreate
```

---

# Restart kube-apiserver

After saving the manifest:

```text
kube-apiserver automatically restarts
```

Verify:

```bash
ps aux | grep kube-apiserver
```

Look for:

```bash
--encryption-provider-config
```

---

# Test Encryption

Create a new secret:

```bash
kubectl create secret generic my-secret2 \
  --from-literal=key2=top-secret
```

Check in etcd:

```bash
ETCDCTL_API=3 etcdctl ...
```

Result:

```text
Secret value is NOT visible
```

This confirms:

```text
Encryption at Rest is working
```

---

# Important Note

Secrets created **before enabling encryption** remain unencrypted.

Example:

```text
my-secret     -> Unencrypted
my-secret2    -> Encrypted
```

---

# Re-encrypt Existing Secrets

Run:

```bash
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

This updates all existing Secrets.

Result:

```text
All Secrets are rewritten and encrypted in etcd.
```

---

# Exam Tips (CKA/CKS)

### Check Encryption Status

```bash
ps aux | grep encryption-provider-config
```

### Generate Key

```bash
head -c 32 /dev/urandom | base64
```

### Encryption Config Path

```bash
--encryption-provider-config=/etc/kubernetes/enc/enc.yaml
```

### Verify in etcd

```bash
ETCDCTL_API=3 etcdctl get /registry/secrets/default/<secret-name>
```

### Re-encrypt Existing Secrets

```bash
kubectl get secrets --all-namespaces -o json | kubectl replace -f -
```

---

# Key Takeaways

* Kubernetes Secrets are **Base64 encoded**, not encrypted.
* Secret data is stored in **etcd**.
* Enable **Encryption at Rest** using an `EncryptionConfiguration`.
* `aescbc`, `aesgcm`, and `secretbox` are supported encryption providers.
* The **first provider encrypts**, all providers can decrypt.
* Existing Secrets are **not automatically re-encrypted**.
* Use `kubectl replace` to re-encrypt old Secrets.
* Always verify encryption directly from **etcd**.
