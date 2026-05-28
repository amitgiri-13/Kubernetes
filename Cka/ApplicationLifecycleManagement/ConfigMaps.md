# Kubernetes ConfigMaps — Notes

## What is a ConfigMap?

A ConfigMap in Kubernetes is used to store configuration data as **key-value pairs**.

Instead of hardcoding environment variables inside Pod YAML files, ConfigMaps allow centralized configuration management.

This improves:

* Maintainability
* Reusability
* Separation of configuration from application code

---

# Why Use ConfigMaps?

Without ConfigMaps:

```yaml id="gozqff"
env:
  - name: APP_COLOR
    value: blue
```

If many Pods use the same configuration, updating values becomes difficult.

With ConfigMaps:

* Store configuration once
* Reuse across multiple Pods
* Update configurations more easily

---

# Two Steps in Using ConfigMaps

## 1. Create the ConfigMap

## 2. Inject it into the Pod

---

# Methods to Create ConfigMaps

## 1. Imperative Method

Create directly from command line.

### Single Key-Value Pair

```bash id="s4j1s0"
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue
```

---

### Multiple Key-Value Pairs

```bash id="ndvw8u"
kubectl create configmap app-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_MODE=prod
```

---

## 2. Create from File

Example file:

```text id="6igj9f"
app.properties
```

Contents:

```text id="bhm8kg"
APP_COLOR=blue
APP_MODE=prod
```

Create ConfigMap:

```bash id="d4yzcw"
kubectl create configmap app-config \
  --from-file=app.properties
```

---

# Declarative Method (Recommended)

Create YAML definition file.

## ConfigMap YAML

```yaml id="k1f1qd"
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

Create it:

```bash id="f8ng9p"
kubectl apply -f config-map.yaml
```

---

# View ConfigMaps

## List ConfigMaps

```bash id="79u5t4"
kubectl get configmaps
```

or

```bash id="4f3n9m"
kubectl get cm
```

---

## Describe ConfigMap

```bash id="a5g3km"
kubectl describe configmap app-config
```

---

# Inject ConfigMap into Pod

## Using `envFrom`

Imports all key-value pairs as environment variables.

```yaml id="98kqpi"
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp-color
      envFrom:
        - configMapRef:
            name: app-config
```

Result inside container:

```bash id="m8fq0d"
echo $APP_COLOR
```

Output:

```bash id="c4eh5i"
blue
```

---

# Inject Single ConfigMap Value

Instead of importing everything:

```yaml id="n12f8o"
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

---

# ConfigMap as Volume

ConfigMaps can also be mounted as files.

```yaml id="1q5z7x"
volumes:
  - name: config-volume
    configMap:
      name: app-config
```

Useful when applications expect configuration files instead of environment variables.

---

# Important Concepts

| Concept           | Description                        |
| ----------------- | ---------------------------------- |
| `ConfigMap`       | Stores non-sensitive configuration |
| `envFrom`         | Inject all variables               |
| `configMapKeyRef` | Inject single variable             |
| `--from-literal`  | Create using CLI values            |
| `--from-file`     | Create from file                   |
| `data:`           | Stores key-value pairs             |

---

# ConfigMap vs Secret

| Feature        | ConfigMap  | Secret            |
| -------------- | ---------- | ----------------- |
| Sensitive Data | No         | Yes               |
| Storage        | Plain text | Base64 encoded    |
| Use Cases      | App config | Passwords, tokens |

---

# Quick Summary

* ConfigMaps centralize application configuration.
* They store configuration as key-value pairs.
* Can be created:

  * Imperatively
  * Declaratively
* Can be injected into Pods:

  * As environment variables
  * As mounted files
* Best used for non-sensitive application configuration.
