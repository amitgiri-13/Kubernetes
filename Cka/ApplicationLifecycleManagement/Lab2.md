# Kubernetes Commands and Arguments 

## Docker vs Kubernetes Mapping

In Docker:

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["5"]
```

Equivalent Docker run:

```bash
docker run ubuntu-sleeper 10
```

Final executed command:

```bash
sleep 10
```

---

# Kubernetes Mapping

| Docker Concept | Kubernetes Field |
| -------------- | ---------------- |
| ENTRYPOINT     | `command`        |
| CMD            | `args`           |

This is the most important concept.

---

# 1. Using `args`

If the image already contains:

```dockerfile
ENTRYPOINT ["sleep"]
CMD ["5"]
```

You can override only the CMD value using `args`.

## Pod Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  containers:
    - name: ubuntu
      image: ubuntu-sleeper
      args: ["10"]
```

Final command executed:

```bash
sleep 10
```

---

# 2. Using `command`

`command` overrides the Docker `ENTRYPOINT`.

## Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
spec:
  containers:
    - name: ubuntu
      image: ubuntu-sleeper
      command: ["sleep2.0"]
      args: ["10"]
```

Final command executed:

```bash
sleep2.0 10
```

---

# Important Rule

| Kubernetes Field | Overrides         |
| ---------------- | ----------------- |
| `command`        | Docker ENTRYPOINT |
| `args`           | Docker CMD        |

---

# Correct Array Syntax

Commands and args must be arrays of strings.

## Correct

```yaml
command: ["sleep", "5000"]
```

OR

```yaml
command: ["sleep"]
args: ["5000"]
```

---

## Wrong

```yaml
command: ["sleep 5000"]
```

This becomes a single string item.

---

## Wrong Type

```yaml
command: ["sleep", 5000]
```

`5000` must be a string.

Correct:

```yaml
command: ["sleep", "5000"]
```

---

# Inspecting Commands

## View pod details

```bash
kubectl describe pod ubuntu-sleeper
```

Look under:

```text
Command:
Args:
```

---

# Pod Fields That Cannot Be Edited

You cannot directly edit container command/args in a running Pod.

This fails:

```bash
kubectl edit pod mypod
```

for immutable fields.

---

# Correct Way to Update

## Option 1 — Delete and recreate

```bash
kubectl delete pod mypod
kubectl apply -f pod.yaml
```

---

## Option 2 — Force replace

```bash
kubectl replace --force -f pod.yaml
```

This:

1. Deletes pod
2. Recreates pod

---

# Dockerfile Examples

## Example 1

```dockerfile
ENTRYPOINT ["python", "app.py"]
```

Final command:

```bash
python app.py
```

---

## Example 2

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]
```

Final command:

```bash
python app.py --color red
```

---

# Kubernetes Override Behavior

## Dockerfile

```dockerfile
ENTRYPOINT ["python", "app.py"]
CMD ["--color", "red"]
```

## Pod

```yaml
containers:
  - image: webapp-color
    args: ["--color", "green"]
```

Final command:

```bash
python app.py --color green
```

Only CMD is overridden.

---

# Overriding ENTRYPOINT Completely

## Pod

```yaml
containers:
  - image: webapp-color
    command: ["echo"]
```

Final command:

```bash
echo
```

Original ENTRYPOINT is ignored.

---

# `kubectl run` with Arguments

## Pass args to container

```bash
kubectl run webapp-green \
  --image=kodekloud/webapp-color \
  -- --color green
```

---

# Why Double Dash `--`?

It separates:

* kubectl options
* container application arguments

## Example

```bash
kubectl run myapp --image=nginx -- --port 8080
```

### Before `--`

Handled by kubectl.

### After `--`

Passed into container.

---

# Override Command Using kubectl

```bash
kubectl run myapp \
  --image=ubuntu \
  --command -- sleep 1000
```

Equivalent YAML:

```yaml
command: ["sleep", "1000"]
```

---

# Fast Exam Tips

## Override CMD only

Use:

```yaml
args:
```

---

## Override ENTRYPOINT

Use:

```yaml
command:
```

---

## Always use string arrays

Correct:

```yaml
args: ["1000"]
```

Not:

```yaml
args: [1000]
```

---

# Quick Memory Trick

```text
command  -> ENTRYPOINT
args     -> CMD
```

Or:

```text
command replaces executable
args replaces parameters
```
