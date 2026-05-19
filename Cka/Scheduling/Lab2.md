# Kubernetes Labels and Selectors Lab

# View Existing Pods

## List Pods
```bash
kubectl get pods
```

* Pods contain labels such as:

  * `env`
  * `bu`
  * `tier`

---

# Find Pods in Dev Environment

## Using Selectors

```bash
kubectl get pods --selector env=dev
```

## Count Pods

```bash
kubectl get pods --selector env=dev --no-headers | wc -l
```

### Result

```text
7
```

---

# Find Pods in Finance Business Unit

## Filter by `bu`

```bash
kubectl get pods --selector bu=finance --no-headers | wc -l
```

### Result

```text
6
```

---

# Find All Objects in Prod Environment

## List All Resources

```bash
kubectl get all --selector env=prod
```

## Count Resources

```bash
kubectl get all --selector env=prod --no-headers | wc -l
```

### Result

```text
7
```

---

# Filter Using Multiple Labels

## Requirement

Find object with:

* `env=prod`
* `bu=finance`
* `tier=frontend`

## Command

```bash
kubectl get pods --selector env=prod,bu=finance,tier=frontend
```

---

# ReplicaSet Label Issue

## Error While Creating ReplicaSet

```bash
kubectl create -f replicaset-definition.yaml
```

### Error

```text
selector does not match template labels
```

---

# Cause of Error

## ReplicaSet Selector

```yaml
selector:
  matchLabels:
    app: nginx
```

## Pod Template Labels

```yaml
template:
  metadata:
    labels:
      app: web
```

* Selector labels and Pod labels do not match
* ReplicaSet cannot manage Pods

---

# Fix the Issue

## Make Labels Match

```yaml
selector:
  matchLabels:
    app: web
```

or

```yaml
template:
  metadata:
    labels:
      app: nginx
```

---

# Recreate ReplicaSet

## Apply Configuration

```bash
kubectl create -f replicaset-definition.yaml
```

## Verify

```bash
kubectl get rs
kubectl get pods
```

---

# Important Commands

## List Labels

```bash
kubectl get pods --show-labels
```

## Selector Syntax

```bash
kubectl get pods --selector key=value
```

## Multiple Selectors

```bash
kubectl get pods --selector key1=value1,key2=value2
```

---

# Key Points

* Labels organize Kubernetes objects
* Selectors filter objects using labels
* `--selector` is used with `kubectl`
* `--no-headers` helps with accurate counting
* ReplicaSet selectors must match Pod labels


