# Kubernetes Environment Variables & ConfigMaps Lab 

# 1. Check Existing Pods

## List Pods

```bash id="xv4f2m"
kubectl get pods
```

Example output:

```text id="mx1mqt"
NAME             READY   STATUS    RESTARTS   AGE
webapp-color     1/1     Running   0          2m
```

---

# 2. Inspect Environment Variables in Pod

## Describe Pod

```bash id="gpn4oe"
kubectl describe pod webapp-color
```

Look under:

```text id="o1ul7d"
Environment:
  APP_COLOR: pink
```

---

# 3. Verify Environment Variable

| Variable    | Value  |
| ----------- | ------ |
| `APP_COLOR` | `pink` |

---

# 4. Update Environment Variable

## Edit Pod

```bash id="1j2w7y"
kubectl edit pod webapp-color
```

Change:

```yaml id="djlwm2"
- name: APP_COLOR
  value: pink
```

to:

```yaml id="48jsw9"
- name: APP_COLOR
  value: green
```

---

# Important Kubernetes Concept

Pods are mostly immutable.

Direct updates to many Pod fields are forbidden.

Typical workflow:

1. Edit manifest
2. Delete old Pod
3. Recreate Pod

---

# Replace Pod with Updated Configuration

```bash id="q8z7hy"
kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml
```

This:

* Deletes old Pod
* Creates new Pod

---

# 5. Verify Application

Application background changes from:

* Pink → Green

---

# Working with ConfigMaps

# 6. List ConfigMaps

```bash id="wz5v3c"
kubectl get configmaps
```

Short form:

```bash id="8vn3dw"
kubectl get cm
```

---

# 7. Inspect ConfigMap

```bash id="2f0r1a"
kubectl describe cm db-config
```

Example:

```text id="npxk6h"
DB_HOST=sql01.example.com
DB_PORT=3306
DB_NAME=mydatabase
```

---

# 8. Create ConfigMap

## Using Imperative Command

```bash id="5l2nva"
kubectl create configmap webapp-config-map \
  --from-literal=APP_COLOR=darkblue
```

---

# Verify ConfigMap

```bash id="mjlwmk"
kubectl describe cm webapp-config-map
```

---

# 9. Use ConfigMap in Pod

## Old Method (Direct Variable)

```yaml id="dfclmg"
env:
  - name: APP_COLOR
    value: green
```

---

## New Method (ConfigMap)

```yaml id="d4wy0z"
envFrom:
  - configMapRef:
      name: webapp-config-map
```

---

# Full Example Pod YAML

```yaml id="jlwmj3"
apiVersion: v1
kind: Pod
metadata:
  name: webapp-color
spec:
  containers:
    - name: webapp-color
      image: simple-webapp-color
      envFrom:
        - configMapRef:
            name: webapp-config-map
```

---

# Recreate Pod

```bash id="2pqq4j"
kubectl replace --force -f pod.yaml
```

---

# Verify Environment Variables

```bash id="bqfd2f"
kubectl describe pod webapp-color
```

Look for:

```text id="qev4ij"
Environment Variables from:
  webapp-config-map
```

---

# Final Result

Application background changes to:

* Dark Blue

because value comes from:

```text id="nyy2fi"
APP_COLOR=darkblue
```

inside the ConfigMap.

---

# Key Commands Summary

| Purpose            | Command                                |
| ------------------ | -------------------------------------- |
| List Pods          | `kubectl get pods`                     |
| Describe Pod       | `kubectl describe pod <pod-name>`      |
| Edit Pod           | `kubectl edit pod <pod-name>`          |
| Replace Pod        | `kubectl replace --force -f file.yaml` |
| List ConfigMaps    | `kubectl get cm`                       |
| Describe ConfigMap | `kubectl describe cm <name>`           |
| Create ConfigMap   | `kubectl create configmap`             |

---

# Important Exam Tips (CKA / CKAD)

## Shortcuts

| Resource  | Short Form |
| --------- | ---------- |
| Pod       | `po`       |
| ConfigMap | `cm`       |

Example:

```bash id="paznsm"
kubectl get po
kubectl get cm
```

---

# Common ConfigMap Injection Methods

| Method            | Usage                  |
| ----------------- | ---------------------- |
| `envFrom`         | Import all variables   |
| `configMapKeyRef` | Import single variable |
| Volume Mount      | Mount as files         |

---

# Core Concepts Learned

* Environment variables in Pods
* Editing Pod configuration
* Pod immutability
* Creating ConfigMaps
* Injecting ConfigMaps into Pods
* Using `envFrom`
* Troubleshooting Kubernetes configuration
