# Kubernetes Multi-Container Pod Design Patterns


# 1. Co-Located Containers (Traditional Multi-Container Pod)

This is the simplest form of a multi-container pod.

```yaml
spec:
  containers:
  - name: web-server
    image: nginx

  - name: app
    image: myapp
```

### Characteristics

* Both containers start when the pod starts.
* Both containers run throughout the pod lifecycle.
* Both containers stop when the pod is deleted.
* Containers share:

  * Network namespace (same IP)
  * Storage volumes
  * Lifecycle

### Important Limitation

There is **no startup order guarantee**.

Kubernetes may start:

```text
web-server → app
```

or

```text
app → web-server
```

The order is not controlled.

### Use Case

When two containers need to run together but neither depends on the other starting first.

Example:

```text
Application Container
+
Monitoring Agent
```

---

# 2. Init Containers

Init containers run **before** the main application starts.

```yaml
spec:
  initContainers:
  - name: wait-for-db
    image: busybox

  containers:
  - name: app
    image: myapp
```

### Lifecycle

```text
Pod Start
     │
     ▼
Init Container Runs
     │
     ▼
Init Container Exits
     │
     ▼
Main Application Starts
```

### Characteristics

* Run only once.
* Must complete successfully.
* Main application waits until all init containers finish.
* Execute sequentially.

---

### Multiple Init Containers

```yaml
initContainers:
- name: wait-for-db
- name: wait-for-api

containers:
- name: app
```

Execution order:

```text
wait-for-db
      ↓
wait-for-api
      ↓
app
```

If any init container fails:

```text
Pod = Init:CrashLoopBackOff
```

Application never starts.

---

### Common Use Cases

#### Wait for Database

```bash
until nc -z mysql 3306
do
  sleep 2
done
```

#### Download Configuration

```bash
wget config-file
```

#### Database Migration

```bash
run-migrations.sh
```

---

# 3. Sidecar Containers

A sidecar is a helper container that runs alongside the main application.

### Purpose

The sidecar:

1. Starts before or with the application.
2. Continues running while the application runs.
3. Stops when the application stops.

### Lifecycle

```text
Pod Start
     │
     ▼
Sidecar Starts
     │
     ▼
Main App Starts
     │
     ▼
Both Run Together
     │
     ▼
Pod Stops
```

---

## Example: Log Shipping

```text
Pod
│
├── Application
│
└── Filebeat Sidecar
```

Application writes logs.

```text
/app/logs/app.log
```

Filebeat reads those logs and forwards them to:

* Elasticsearch
* Logstash
* Splunk
* Kafka

---

## Why Sidecar?

Without a sidecar:

```text
Application crashes
↓
Startup logs lost
↓
Termination logs lost
```

With a sidecar:

```text
Application starts
↓
Filebeat captures startup logs

Application runs
↓
Filebeat captures runtime logs

Application crashes
↓
Filebeat captures termination logs
```

This makes troubleshooting much easier.

---

# Comparison Table

| Feature                    | Co-Located Container | Init Container | Sidecar Container          |
| -------------------------- | -------------------- | -------------- | -------------------------- |
| Runs before app            | No                   | Yes            | Yes                        |
| Runs with app              | Yes                  | No             | Yes                        |
| Runs after app starts      | Yes                  | No             | Yes                        |
| Startup order control      | No                   | Yes            | Yes                        |
| Stops after initialization | No                   | Yes            | No                         |
| Typical use                | Companion service    | Setup tasks    | Logging, monitoring, proxy |

---

# Visual Summary

```text
1. Co-Located Containers

Pod
├── App
└── Web Server

(Both start together)
```

```text
2. Init Containers

Pod
├── Init Container
└── App

(Init finishes → App starts)
```

```text
3. Sidecar Containers

Pod
├── Sidecar
└── App

(Sidecar starts first and stays running)
```

---

# DevOps Interview Question

**What is the difference between an Init Container and a Sidecar Container?**

| Init Container          | Sidecar Container              |
| ----------------------- | ------------------------------ |
| Runs before application | Starts before/with application |
| Finishes and exits      | Continues running              |
| Used for setup tasks    | Used for supporting tasks      |
| Sequential execution    | Parallel execution with app    |

**Examples:**

* Init Container → Wait for database, download config, run migrations.
* Sidecar Container → Log collector, monitoring agent, service mesh proxy (e.g., Envoy Proxy), metrics exporter.

### Exam Tip (CKA/CKAD)

Remember:

```text
Init Container
= Start → Complete → Exit

Sidecar Container
= Start → Keep Running → Stop with Pod
```

That distinction is the key concept tested in Kubernetes certification exams.
