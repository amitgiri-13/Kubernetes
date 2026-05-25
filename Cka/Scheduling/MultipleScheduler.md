# Multiple Schedulers in Kubernetes

## What is a Scheduler?

In Kubernetes, the scheduler is responsible for:

* Watching for newly created Pods
* Selecting the best node for each Pod
* Assigning the Pod to that node

Default scheduler:

```bash id="t8s7oe"
kube-scheduler
```

It considers:

* Resource availability
* Taints and tolerations
* Node affinity
* Pod affinity/anti-affinity
* Policies and constraints

---

# Why Use Multiple Schedulers?

Sometimes the default scheduler is not enough.

Example scenarios:

* Specialized hardware placement
* GPU-aware scheduling
* Cost-aware scheduling
* AI/ML workload optimization
* Custom business rules
* Latency-sensitive applications

In such cases, you can:

* Build your own scheduler
* Run it alongside the default scheduler
* Assign specific Pods to it

Kubernetes supports running multiple schedulers simultaneously.

---

# Architecture Overview

## Default Flow

```text id="l6f2xj"
Pod Created
     ↓
Default Scheduler
     ↓
Node Assigned
```

---

## Multiple Scheduler Flow

```text id="l2i4ph"
Pod A → Default Scheduler
Pod B → Custom Scheduler
Pod C → Another Scheduler
```

Each scheduler works independently.

---

# Important Concepts

## 1. Each Scheduler Must Have a Unique Name

Default scheduler name:

```text id="8z4dl0"
default-scheduler
```

Custom scheduler example:

```text id="9kqv6u"
my-custom-scheduler
```

The scheduler name is defined in the scheduler configuration file.

---

# Scheduler Configuration File

Example:

```yaml id="f8j5rk"
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration

profiles:
- schedulerName: my-custom-scheduler
```

This defines the scheduler identity.

---

# Ways to Deploy a Custom Scheduler

## Method 1: Run Scheduler Binary Directly

You can run:

```bash id="w1g3dc"
kube-scheduler --config=/path/config.yaml
```

This uses:

* Kubernetes scheduler binary
* Custom config file
* Custom scheduler name

You may:

* Use default kube-scheduler binary
* Or build your own modified scheduler

---

# Method 2: Run Scheduler as a Pod

Common in kubeadm clusters.

Example Pod:

```yaml id="2v4mfr"
apiVersion: v1
kind: Pod
metadata:
  name: my-scheduler
  namespace: kube-system

spec:
  containers:
  - command:
    - kube-scheduler
    - --config=/etc/kubernetes/my-scheduler-config.yaml
    image: k8s.gcr.io/kube-scheduler:v1.28.0
    name: kube-scheduler
```

---

# Important Scheduler Options

## kubeconfig

Used for authentication with API server.

Example:

```bash id="7e1vya"
--kubeconfig=/etc/kubernetes/scheduler.conf
```

Without this:

* Scheduler cannot communicate with Kubernetes API.

---

# Leader Election

## Why Needed?

In High Availability (HA) clusters:

* Multiple master/control-plane nodes exist
* Multiple scheduler replicas may run

Only one scheduler instance should actively schedule Pods.

---

## leaderElect Option

Example:

```yaml id="7k7ztg"
leaderElection:
  leaderElect: true
```

This ensures:

* One active scheduler leader
* Others remain standby

---

# lock Object Name

When using multiple schedulers:

Each scheduler should use different leader election locks.

Example:

```yaml id="6l0h7e"
leaderElection:
  resourceName: my-scheduler
```

This prevents conflicts with the default scheduler.

---

# Deploying Scheduler as a Deployment

This is the modern production approach.

Advantages:

* Self-healing
* Easier upgrades
* Managed replicas
* Better HA

---

# ConfigMap for Scheduler Config

Instead of embedding config files directly, Kubernetes commonly uses ConfigMaps.

Example:

```yaml id="n8s5xv"
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-scheduler-config

data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
    - schedulerName: my-scheduler
```

Mounted into the scheduler Pod as a volume.

---

# Scheduler Deployment Example

```yaml id="9y2fkt"
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-scheduler
  namespace: kube-system

spec:
  replicas: 1

  selector:
    matchLabels:
      component: scheduler

  template:
    metadata:
      labels:
        component: scheduler

    spec:
      containers:
      - name: kube-scheduler
        image: k8s.gcr.io/kube-scheduler:v1.28.0

        command:
        - kube-scheduler
        - --config=/etc/kubernetes/config/my-scheduler-config.yaml

        volumeMounts:
        - name: config-volume
          mountPath: /etc/kubernetes/config

      volumes:
      - name: config-volume
        configMap:
          name: my-scheduler-config
```

---

# RBAC Requirements

Schedulers need permissions to:

* Watch Pods
* Bind Pods to nodes
* Read cluster state

Therefore:

* ServiceAccounts
* ClusterRoles
* ClusterRoleBindings

are required.

---

# Using a Custom Scheduler

To tell Kubernetes which scheduler to use:

Use:

```yaml id="uy7jrs"
schedulerName:
```

---

# Pod Example

```yaml id="xkzvdu"
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  schedulerName: my-custom-scheduler

  containers:
  - name: nginx
    image: nginx
```

Now:

* Default scheduler ignores this Pod
* `my-custom-scheduler` handles it

---

# What Happens if Scheduler is Missing?

If scheduler is:

* Misconfigured
* Crashed
* Not running

Then the Pod remains:

```text id="d4ukpl"
Pending
```

Because no scheduler processes it.

---

# Checking Scheduler Events

Use:

```bash id="2yig7s"
kubectl get events -o wide
```

Look for:

```text id="n3r6be"
Successfully assigned
```

The event source shows which scheduler handled the Pod.

Example:

```text id="q9p8dk"
Source: my-custom-scheduler
```

---

# Viewing Scheduler Logs

If scheduling fails:

```bash id="xq4o4g"
kubectl logs <scheduler-pod> -n kube-system
```

Useful for debugging:

* Authentication issues
* RBAC issues
* Config problems
* Scheduling failures

---

# Verify Scheduler Pods

Check scheduler Pods:

```bash id="t5mp2k"
kubectl get pods -n kube-system
```

You should see:

* Default scheduler
* Custom scheduler Pods

---

# Real-World Use Cases

| Use Case              | Custom Scheduling Logic          |
| --------------------- | -------------------------------- |
| AI/ML workloads       | GPU-aware scheduling             |
| Financial systems     | Low-latency placement            |
| Multi-tenant clusters | Team-specific isolation          |
| Edge computing        | Geographic placement             |
| Cost optimization     | Spot/preemptible node preference |

---

# Key Takeaways

* Kubernetes supports multiple schedulers
* Each scheduler must have a unique name
* Pods select schedulers using `schedulerName`
* Custom schedulers can run as:

  * Binary
  * Pod
  * Deployment
* HA setups use leader election
* ConfigMaps commonly store scheduler configs
* If scheduler is unavailable, Pods stay Pending
* Events and logs help debug scheduling issues

---

# Important Commands

## List scheduler Pods

```bash id="h35d6r"
kubectl get pods -n kube-system
```

## View events

```bash id="klz9pv"
kubectl get events -o wide
```

## View scheduler logs

```bash id="tw5x9k"
kubectl logs <scheduler-pod> -n kube-system
```

---

# Interview Questions

## Can Kubernetes run multiple schedulers?

Yes. Kubernetes supports multiple schedulers simultaneously.

---

## How does a Pod select a scheduler?

Using:

```yaml id="6t6d63"
schedulerName:
```

---

## What happens if the specified scheduler is unavailable?

The Pod remains in Pending state.

---

## Why is leader election required?

To ensure only one scheduler instance actively schedules Pods in HA setups.

---

## Where are scheduler configurations stored?

Usually in:

* Config files
* ConfigMaps mounted into scheduler Pods
