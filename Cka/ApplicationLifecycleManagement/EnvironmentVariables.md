# Kubernetes Environment Variables — Notes

## Setting Environment Variables in Kubernetes

In Kubernetes, environment variables for a container are defined inside the Pod specification using the `env` field.

The `env` field is an array, so each environment variable entry starts with `-`.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
spec:
  containers:
    - name: simple-webapp
      image: simple-webapp-color
      env:
        - name: APP_COLOR
          value: blue
```

## Explanation

* `env`: List of environment variables
* `name`: Environment variable name
* `value`: Value assigned to the variable

Inside the container:

```bash
echo $APP_COLOR
```

Output:

```bash
blue
```

---

# Different Ways to Set Environment Variables

## 1. Direct Key-Value Pair

```yaml
env:
  - name: APP_COLOR
    value: blue
```

Used for simple static values.

---

## 2. Using ConfigMaps

Instead of hardcoding values, Kubernetes can load them from a ConfigMap.

```yaml
env:
  - name: APP_COLOR
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_COLOR
```

Useful for:

* Centralized configuration
* Reusable settings
* Easier updates

---

## 3. Using Secrets

Sensitive data such as passwords or API keys should use Secrets.

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: password
```

Useful for:

* Passwords
* Tokens
* API keys
* Credentials

---

# Key Difference

Direct value:

```yaml
value: blue
```

External source:

```yaml
valueFrom:
```

---

# Important Concepts

| Method            | Use Case                    |
| ----------------- | --------------------------- |
| `value`           | Static simple values        |
| `configMapKeyRef` | Non-sensitive configuration |
| `secretKeyRef`    | Sensitive information       |

---

# Quick Summary

* Kubernetes uses `env` to define environment variables.
* `env` is an array of objects.
* Each object contains:

  * `name`
  * `value` or `valueFrom`
* ConfigMaps and Secrets help externalize configuration from Pod definitions.
