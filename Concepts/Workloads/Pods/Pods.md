# Pods

## What is a Pod?
- **Smallest deployable unit** in Kubernetes
- Group of **one or more containers** 
- Containers share:
  - Storage (Volumes)
  - Network resources (same IP address + port space)
  - IPC (Inter-Process Communication)
- Containers are **co-located** and **co-scheduled** (run on the same node)
- Models an application-specific **"logical host"**
- Analogy: like processes running on the same physical/virtual machine

## Shared Context
- Pod shares a single set of **Linux namespaces**, **cgroups**, and other isolation mechanisms
- Individual containers can have further sub-isolation if needed
- Very similar in concept to a group of containers with shared namespaces + filesystem volumes

## Main Use Cases

1. **Single-container Pod** (most common pattern)
   - One application container per Pod
   - Pod acts as a thin wrapper around a single container
   - Kubernetes manages Pods (not containers directly)

2. **Multi-container Pod** (advanced / specific use case)
   - Multiple tightly coupled containers that need to work closely together
   - Examples: 
     - Main app + logging sidecar
     - Web server + helper container for config sync
     - App + proxy / adapter / data pump
   - Containers share storage, network, lifecycle
   - **Only use when containers are strongly interdependent**

## Important Notes
- **Do NOT** use multi-container Pods just for replication / scaling  
  → Use **Deployments**, **ReplicaSets**, **StatefulSets**, etc. for replicas
- Pod can include:
  - **Init Containers** — run sequentially during Pod startup (setup tasks)
  - **Ephemeral Containers** — can be attached temporarily for debugging
- **Requirement**: Every node in the cluster must have a **container runtime** installed (containerd, CRI-O, Docker, etc.)

## When to Use Multi-Container Pods?
- Containers need to **share files** directly (via shared volume)
- Containers need to **communicate via localhost**
- Containers have a **tight lifecycle dependency** (one fails → all should stop)
- One container is clearly a **sidecar** / helper to the main app

## Summary
- **One-container-per-Pod** → standard & recommended for most applications
- **Multi-container Pod** → only for tightly coupled, co-located use cases
- Pods = the atomic scheduling unit in Kubernetes

---

#  Using Pods & Best Practices

## Creating a Pod Directly (Example)

```yaml
# pods/simple-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

Apply it with:
```bash
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

→ This creates a **single-container Pod** running nginx.

## Key Principle: Rarely Create Pods Directly

- Pods are **ephemeral** and **disposable** by design
- They should **not** usually be created individually (even singleton Pods)
- Pods are **not restarted** — containers inside may restart, but the Pod object itself persists until deleted
- Deleting / evicting / node failure → Pod is gone (not restarted automatically)

**Recommended pattern**: Use **workload resources** + controllers instead.

## Workload Resources That Manage Pods

| Resource       | Use Case                                                                 | Replication / Scaling | Stateful? | Controller        |
|----------------|--------------------------------------------------------------------------|-----------------------|-----------|-------------------|
| **Deployment** | Stateless applications (web servers, APIs, microservices)               | Yes                   | No        | ReplicaSet        |
| **ReplicaSet** | Ensures a specified number of Pod replicas are running                   | Yes                   | No        | —                 |
| **StatefulSet**| Stateful applications (databases, key-value stores, ordered scaling)    | Yes                   | Yes       | StatefulSet       |
| **DaemonSet**  | Run one Pod per node (logging agents, monitoring, network proxies)      | One per node          | No        | DaemonSet         |
| **Job**        | Run Pods to completion (batch jobs, one-off tasks)                      | Optional              | No        | Job               |
| **CronJob**    | Scheduled Jobs (periodic batch processing)                               | Optional              | No        | CronJob → Job     |

→ Almost all production workloads use one of these instead of raw Pods.

## Pods & Controllers (Scaling + Self-Healing)

- **Replication** = running multiple identical Pods (horizontal scaling)
- Controllers watch desired state vs. actual state
- If a Pod dies / is evicted / node fails → controller creates replacement Pod
- This is **auto-healing** and the main reason to avoid raw Pods

## Pod Characteristics

- **Ephemeral** — not meant to live forever
- **Disposable** — Kubernetes can terminate and replace them at any time
- **Single instance** of an application per Pod
  - Want more capacity → create **more Pods** (via controller)
- **Shared resources** within a Pod:
  - Networking (same IP, localhost communication)
  - Storage (shared volumes)

## Pod Naming & Hostname

- Pod name must be valid **DNS subdomain** name
- For best compatibility → follow stricter **DNS label** rules
  - lowercase letters, numbers, hyphens
  - no underscores, start/end with alphanumeric

## Pod OS Specification (Stable since v1.25)

```yaml
spec:
  os:
    name: linux    # or "windows"
```

- Tells Kubernetes the expected OS for the Pod's containers
- Helps with:
  - Pod Security Standards (skip irrelevant policies)
  - Future OS-aware scheduling
- **Important in mixed OS clusters**:
  - Label nodes with `kubernetes.io/os=linux` or `=windows`
  - Use `nodeSelector` in Pod spec to target correct OS nodes

**Note (v1.35 behavior)**: `.spec.os.name` does **not** yet influence scheduler node selection.

## Summary – When to Use What

- **Direct Pod creation** → only for:
  - Learning / testing / debugging
  - One-off experiments
- **Production / real applications** → always use:
  - Deployment (most common – stateless)
  - StatefulSet (stateful)
  - Job / CronJob (batch)
  - DaemonSet (node-local)
- Pods = building block
  Controllers + workload resources = how you actually run applications reliably

---


# Workload Reference, Templates, Updates & Generation

## Specifying a Workload Reference (Alpha Feature)
**FEATURE STATE**: Kubernetes v1.35 [alpha] (disabled by default)

- By default, kube-scheduler places **each Pod individually**.
- For **tightly-coupled applications** (e.g., gang scheduling for ML jobs, distributed systems), Pods need **coordinated / simultaneous scheduling**.
- Solution: Link Pod to a **Workload** object via **Workload reference**.
  - Tells scheduler: "This Pod belongs to a group → make placement decisions for the **entire group** at once."
- Enables features like **gang scheduling** (all-or-nothing placement) when GangScheduling plugin + feature gate enabled.
- Field: `spec.workloadRef` (immutable once set)
  - References a Workload object (new API in v1.35) with PodGroup name.
  - If Workload/PodGroup missing → Pod stays unschedulable.
- Use case: Distributed training, MPI jobs, batch jobs requiring all members scheduled together.

→ Enable via **GenericWorkload** feature gate (alpha).

## Pod Templates
- Controllers (Deployment, Job, StatefulSet, DaemonSet, etc.) use **PodTemplate** to create/manage Pods.
- Location: Inside workload resource → `spec.template`
  - Contains full Pod `spec` (containers, volumes, etc.)
  - Excludes fields like `restartPolicy` (handled at workload level, e.g., Job).

**Example** (simple Job with Pod template):

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:               # <-- Pod template starts here
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
  # Pod template ends here
```

- Changing template → **no direct effect** on existing Pods.
- Controller creates **new Pods** with updated template → eventually replaces old ones.
- Different controllers have different **update strategies** (e.g., StatefulSet rolling updates).

→ kubelet ignores template details → abstraction simplifies extensions.

## Pod Update & Replacement Strategy
- Kubernetes **does not update Pods in place** for most fields.
- On template change → controller **replaces** Pods (create new → terminate old).
- Direct updates (via `kubectl patch` / `replace`) possible but **limited**:

  Allowed mutable fields:
  - `spec.containers[*].image`
  - `spec.initContainers[*].image`
  - `spec.activeDeadlineSeconds`
  - `spec.terminationGracePeriodSeconds`
  - `spec.tolerations` (add only)
  - `spec.schedulingGates`

  **ActiveDeadlineSeconds** rules:
  1. Set from unset → positive number
  2. Decrease from positive → smaller non-negative

  **Immutable** fields (cannot change):
  - Most `metadata` (namespace, name, uid, creationTimestamp)
  - If `deletionTimestamp` set → cannot add new finalizers

## Pod Subresources (for Special Updates)
Bypass regular update limits:

- **resize** → Update container resources (`spec.containers[*].resources`)
- **ephemeralContainers** → Add debug ephemeral containers
- **status** → Update Pod status (kubelet + controllers only)
- **binding** → Set `spec.nodeName` (scheduler only)

## Pod Generation & Observed Generation
**FEATURE STATE**: Kubernetes v1.35 [stable] (enabled by default) for generation tracking

- `metadata.generation` (integer)
  - Auto-set by API server
  - New Pod → `1`
  - Increments +1 on **any mutable spec change**

- `status.observedGeneration` (set by kubelet)
  - Tracks which `metadata.generation` the current status reflects
  - **Do NOT** modify from external controllers

**Direct Status Updates** (reflect current generation N):
- Resize status
- Allocated resources
- Ephemeral containers in Waiting state

**Indirect Status Updates** (reflect previous generation N-1 until complete):
- Container `ImageID` (until new image pulled)
- Actual resources in use (during resize)
- Container state / restart policy effects
- Effects of `activeDeadlineSeconds`, `terminationGracePeriodSeconds`, `deletionTimestamp`

→ Helps distinguish: "Is this status from the current spec or still catching up?"

## Summary – Key Takeaways

- **Workload Reference** (v1.35 alpha) → Enables group/coordinated scheduling (gang, etc.)
- **Pod Templates** → How controllers create Pods; changes trigger replacements
- **Updates** → Mostly replacement-based; limited in-place patching
- **Subresources** → For special cases (resize, ephemeral, status, binding)
- **Generation** → Tracks spec changes; observedGeneration shows status sync state

Use workload resources + controllers for production → direct Pod management is rare and limited.

---

# Kubernetes Pods - Resource Sharing, Communication & Advanced Features

## Resource Sharing & Communication

### Storage in Pods
- Pods can define **shared Volumes**
- All containers in the Pod can mount and access the same volumes
- Enables **data sharing** between containers
- Survives container restarts (data persistence within Pod lifecycle)
- See Kubernetes **Storage** docs for volume types (emptyDir, hostPath, persistentVolumeClaim, etc.)

### Pod Networking
- Each Pod gets a **unique cluster IP** (per IP family: IPv4/IPv6)
- All containers in a Pod share the **same network namespace**
  - Same IP address
  - Same network ports
- **Intra-Pod communication**:
  - Use **localhost** (127.0.0.1)
  - Standard OS-level IPC: System V semaphores, POSIX shared memory, etc.
- **Inter-Pod communication**:
  - Must use **IP networking** (Pod IP + port)
  - No direct OS-level IPC between different Pods
- Containers coordinate port usage (avoid conflicts on shared IP)
- Pod hostname = Pod name (DNS resolution inside cluster)

### Pod Security Settings
- Use `securityContext` field (at Pod or container level)
- Controls: user/group IDs, filesystem permissions, capabilities, seccomp, AppArmor, SELinux, etc.

**Basic example**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000           # Sets group ownership on volumes
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: ["sh", "-c", "sleep 1h"]
```

**Recommendations**:
- Follow **Baseline Pod Security Standard**
- Run containers as **non-root** users
- For advanced: capabilities, seccomp profiles, sysctls → see Security Concepts

## Static Pods
- Managed **directly by kubelet** on a specific node
- **Not** observed/controlled by API server (no Deployment, etc.)
- Kubelet restarts them if they fail
- Always bound to **one node**
- Primary use: **self-hosted control plane** components (etcd, kube-apiserver, etc.)
- Kubelet creates **mirror Pod** on API server → visible but **read-only** (cannot control)
- Limitation: Cannot reference other API objects (ServiceAccount, ConfigMap, Secret)

## Pods with Multiple Containers
- Designed for **cooperating processes** forming a **single cohesive unit**
- Containers are **co-located** and **co-scheduled** on same node
- Share: storage, network namespace, lifecycle, dependencies
- **Common patterns**:
  1. **Single-container Pod** — most common (Pod wraps one container)
  2. **Multi-container Pod** — tightly coupled containers
     - Example: Web server + sidecar that updates files in shared volume
     - Main app + logging/monitoring/proxy sidecar
     - Adapter / ambassador pattern

**Sidecar Containers**  
**FEATURE STATE**: Kubernetes v1.33 [stable] (enabled by default)

- Init containers can have `restartPolicy: Always`
- Treated as **sidecars**: run for entire Pod lifetime
- Start **before** main containers
- Keep running until Pod shutdown

## Container Probes
- Kubelet performs periodic diagnostics on containers
- Three probe types:

| Probe Type     | Action Performed By          | Purpose                              |
|----------------|------------------------------|--------------------------------------|
| **livenessProbe**   | Determines if container is healthy (restart if failed) |
| **readinessProbe**  | Determines if container can accept traffic (remove from Service if failed) |
| **startupProbe**    | Delays liveness/readiness until slow-starting container is ready |

- Actions:
  - **ExecAction** — run command inside container
  - **TCPSocketAction** — check TCP port open
  - **HTTPGetAction** — HTTP GET request (status 200–399 = success)

→ Detailed in **Pod Lifecycle** documentation

## Summary – Key Concepts

- **Shared storage** → Volumes for data exchange & persistence
- **Shared network** → localhost + same IP/port space
- **Security** → securityContext for least-privilege
- **Static Pods** → for control plane daemons (node-local)
- **Multi-container** → only for tightly coupled sidecar/main patterns
- **Sidecar feature** (v1.33+) → restartPolicy: Always on init containers
- **Probes** → liveness, readiness, startup for health management

Pods = atomic unit with shared context → enables tight integration where needed.

---