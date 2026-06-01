# Kubernetes Node Maintenance: Drain, Cordon, and Uncordon

This lecture explains what happens when a node becomes unavailable and how to safely take nodes out of service for maintenance.

---

## What Happens When a Node Goes Down?

Assume a cluster has:

* Multiple replicas of a Blue application.
* A single pod running a Green application.

When a node fails:

### Blue Application (Multiple Replicas)

* Other replicas continue serving traffic.
* Users are generally unaffected.

### Green Application (Single Pod)

* The only pod is lost.
* Users experience downtime.

---

## Kubernetes Node Failure Handling

When a node becomes unreachable:

1. Kubernetes waits for the node to recover.
2. If the node returns quickly, pods continue running.
3. If the node remains unavailable for too long, Kubernetes considers it dead.

### Pod Eviction Timeout

By default, Kubernetes waits **5 minutes** before evicting pods from an unreachable node.

```
Default Pod Eviction Timeout = 5 minutes
```

After this timeout:

* Pods on the failed node are marked dead.
* Pods managed by a ReplicaSet/Deployment are recreated elsewhere.
* Standalone pods are lost permanently.

### Example

**Before node failure**

```
Node1
 ├─ Blue Pod
 └─ Green Pod

Node2
 └─ Blue Pod
```

**After Node1 fails for > 5 minutes**

```
Node2
 ├─ Blue Pod
 └─ New Blue Pod (recreated)

Green Pod -> Lost
```

Because the Green Pod wasn't managed by a ReplicaSet, Kubernetes does not recreate it.

---

# Safe Node Maintenance

Instead of shutting down a node and hoping it returns within 5 minutes, Kubernetes provides maintenance commands.

---

## 1. Drain a Node

```bash
kubectl drain node01 --ignore-daemonsets
```

### What Drain Does

* Gracefully evicts workloads.
* ReplicaSets/Deployments create replacement pods on other nodes.
* Marks the node as **unschedulable**.

Think of drain as:

> "Move all workloads away and prevent new workloads from coming here."

### Result

Before:

```
Node01
 ├─ App Pod A
 └─ App Pod B
```

After drain:

```
Node01 (Unschedulable)
  No Application Pods

Node02
 ├─ App Pod A
 └─ App Pod B
```

Now maintenance can be performed safely.

---

## 2. Uncordon a Node

After maintenance:

```bash
kubectl uncordon node01
```

### What Uncordon Does

* Makes the node schedulable again.
* New pods can now be placed on the node.

Important:

Pods that were moved during drain **do not automatically move back**.

Kubernetes only schedules future pods onto the node when needed.

---

## 3. Cordon a Node

```bash
kubectl cordon node01
```

### What Cordon Does

* Marks node as unschedulable.
* Existing pods continue running.
* No new pods are scheduled.

### Difference from Drain

| Command    | Existing Pods              | New Pods    |
| ---------- | -------------------------- | ----------- |
| `drain`    | Evicted/Moved              | Not allowed |
| `cordon`   | Stay running               | Not allowed |
| `uncordon` | No effect on existing pods | Allowed     |

---

# Typical Maintenance Workflow

### Step 1: Drain

```bash
kubectl drain node01 --ignore-daemonsets
```

### Step 2: Perform Maintenance

```bash
sudo reboot
```

or

```bash
sudo apt update
sudo apt upgrade
```

### Step 3: Bring Node Back

Verify node status:

```bash
kubectl get nodes
```

### Step 4: Uncordon

```bash
kubectl uncordon node01
```

---

# CKA Exam Tips

Remember these commands:

```bash
kubectl drain <node> --ignore-daemonsets
kubectl cordon <node>
kubectl uncordon <node>
```

### Quick Memory Trick

* **Drain** = Evict pods + block scheduling.
* **Cordon** = Block scheduling only.
* **Uncordon** = Allow scheduling again.

These commands are commonly tested in the CKA exam and are essential for performing Kubernetes node maintenance with minimal application disruption.
