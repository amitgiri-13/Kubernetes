# Kubernetes Commands and Arguments in Pods

# 1. Introduction

In Kubernetes, you can override:

* Docker `CMD`
* Docker `ENTRYPOINT`

using Pod definition fields.

This is important for:

* Custom startup behavior
* Running scripts
* Passing runtime arguments
* Debugging containers

---

# 2. Docker Recap

## Dockerfile Example

```dockerfile id="e0m4qt"
FROM ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"]
```

---

# Behavior

## Default Run

```bash id="i5r1lu"
docker run ubuntu-sleeper
```

Runs:

```text id="m8x9pv"
sleep 5
```

---

## Override CMD

```bash id="l1q7wo"
docker run ubuntu-sleeper 10
```

Runs:

```text id="y7t4zs"
sleep 10
```

---

# 3. Kubernetes Pod Example

## Basic Pod Definition

```yaml id="z4v6kt"
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod

spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
```

---

# Behavior

Container uses:

* Docker ENTRYPOINT
* Docker CMD

Result:

```text id="k9m2fw"
sleep 5
```

---

# 4. Override CMD Using args

## Pod Definition

```yaml id="t8c1qs"
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod

spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      args: ["10"]
```

---

# Result

Kubernetes appends:

```text id="r2j6pn"
10
```

to ENTRYPOINT.

Final command:

```text id="x6p0da"
sleep 10
```

---

# Important Mapping

| Kubernetes Field | Docker Equivalent |
| ---------------- | ----------------- |
| `args`           | `CMD`             |
| `command`        | `ENTRYPOINT`      |

---

# 5. args Field

## Purpose

`args` overrides:

```text id="v0o8fh"
Docker CMD
```

---

# Example

Dockerfile:

```dockerfile id="u9m7zk"
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Pod:

```yaml id="s1q5xl"
args: ["20"]
```

Final execution:

```text id="d7v4wa"
sleep 20
```

---

# 6. Override ENTRYPOINT Using command

## Pod Definition

```yaml id="w8c2mv"
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod

spec:
  containers:
    - name: ubuntu-sleeper
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]
```

---

# Result

Final executed command:

```text id="f4x8ny"
sleep2.0 10
```

---

# 7. command Field

## Purpose

`command` overrides:

```text id="n6k5bp"
Docker ENTRYPOINT
```

---

# Important Clarification

Many people confuse this:

```text id="z3w8uh"
command DOES NOT override Docker CMD
```

Instead:

```text id="v2l1fm"
command overrides ENTRYPOINT
args overrides CMD
```

---

# 8. Full Mapping Diagram

## Dockerfile

```dockerfile id="k7o9dx"
ENTRYPOINT ["sleep"]

CMD ["5"]
```

---

## Kubernetes Pod

```yaml id="u0f7jq"
command: ["sleep2.0"]
args: ["10"]
```

---

# Final Executed Command

```text id="g5c9ev"
sleep2.0 10
```

---

# 9. Execution Flow

| Docker Component | Kubernetes Override |
| ---------------- | ------------------- |
| ENTRYPOINT       | `command`           |
| CMD              | `args`              |

---

# 10. Real Example

## Dockerfile

```dockerfile id="r6t3na"
FROM python:3.11

ENTRYPOINT ["python"]

CMD ["app.py"]
```

---

## Kubernetes Override

```yaml id="x4w8ju"
containers:
  - name: python-app
    image: my-python-app
    args: ["test.py"]
```

---

# Final Command

```text id="b0p7ze"
python test.py
```

---

# 11. Override Both ENTRYPOINT and CMD

## Pod Definition

```yaml id="f7m2yl"
containers:
  - name: app
    image: my-image
    command: ["node"]
    args: ["server.js"]
```

---

# Final Execution

```text id="k8q1sa"
node server.js
```

---

# 12. Kubernetes vs Docker Syntax

| Docker Command                              | Kubernetes Equivalent                    |
| ------------------------------------------- | ---------------------------------------- |
| `docker run image 10`                       | `args: ["10"]`                           |
| `docker run --entrypoint sleep2.0 image 10` | `command: ["sleep2.0"]` + `args: ["10"]` |

---

# 13. Common Mistakes

## Mistake 1

Thinking:

```text id="j9o6vr"
command overrides CMD
```

Wrong.

---

## Correct

```text id="y2x4tc"
command → ENTRYPOINT
args → CMD
```

---

# Mistake 2

Using string instead of array:

Wrong:

```yaml id="u3v8mf"
args: "10"
```

Correct:

```yaml id="e5j7dp"
args: ["10"]
```

---

# Mistake 3

Combining command and arguments together:

Wrong:

```yaml id="p1x9tr"
command: ["sleep 10"]
```

Correct:

```yaml id="d8w0lf"
command: ["sleep"]
args: ["10"]
```

---

# 14. Complete Example

## Pod YAML

```yaml id="m7k4za"
apiVersion: v1
kind: Pod

metadata:
  name: sleeper-pod

spec:
  containers:
    - name: sleeper
      image: ubuntu-sleeper
      command: ["sleep"]
      args: ["15"]
```

---

# Execution

Container runs:

```text id="r4y6cx"
sleep 15
```

---

# 15. Best Practices

## Use args For

* Runtime parameters
* Environment-specific values
* Configurable startup options

---

## Use command For

* Replacing executable entirely
* Debugging
* Running alternate startup processes

---

## Prefer Array Syntax

Correct:

```yaml id="n1w8lu"
command: ["python"]
args: ["app.py"]
```

Avoid:

```yaml id="g3r5ox"
command: "python app.py"
```

---

# Interview Questions

## What does args override?

```text id="j7s4kw"
Docker CMD
```

---

## What does command override?

```text id="z0m6ev"
Docker ENTRYPOINT
```

---

## Final Command Formula

```text id="v9t2ha"
command + args
```

maps to:

```text id="a7p5nf"
ENTRYPOINT + CMD
```

---

# Quick Summary

| Kubernetes | Docker     |
| ---------- | ---------- |
| `command`  | ENTRYPOINT |
| `args`     | CMD        |

---

# Key Takeaway

## Kubernetes Translation Rule

```text id="s5u8yn"
command → ENTRYPOINT
args → CMD
```

This is one of the most important concepts for:

* CKAD
* CKA
* Kubernetes troubleshooting
* Container debugging
