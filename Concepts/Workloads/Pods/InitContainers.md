# Init Containers

## What are Init Containers?
- **Specialized containers** that run **before** application containers in a Pod
- Used for **initialization tasks**, setup scripts, or utilities not present in the app image
- Defined in the Pod spec under `initContainers` (array, parallel to `containers`)
- Run **sequentially** — each must complete successfully before the next starts
- **Always run to completion** (exit with code 0 = success)

## Key Characteristics

- Identical to regular containers in most ways:
  - Support: resource requests/limits, volumes, securityContext, env, envFrom, etc.
- Important **differences** from regular app containers:
  - **No** support for:
    - `lifecycle` (postStart, preStop)
    - `livenessProbe`
    - `readinessProbe`
    - `startupProbe`
  - Must complete successfully → Pod readiness depends on init completion
  - If fails → kubelet **restarts** that init container (unless `restartPolicy: Never` → Pod fails)

## Init Containers vs. Sidecar Containers

| Feature                        | Init Containers                          | Sidecar Containers (v1.33+ stable)          |
|--------------------------------|------------------------------------------|---------------------------------------------|
| **Lifetime**                   | Run once and complete                    | Run continuously (entire Pod lifetime)      |
| **Start Order**                | Before app containers                    | Can start before or alongside app containers|
| **restartPolicy**              | Always `OnFailure` / `Never` (implicit)  | Can be `Always` (sidecar mode)              |
| **Probes**                     | No liveness/readiness/startup            | Supports all probes                         |
| **Purpose**                    | Setup, preconditions, one-time tasks     | Ongoing auxiliary services (logging, proxy)|
| **Resource Sharing**           | Shares Pod resources (but short-lived)   | Shares Pod resources continuously           |
| **Defined In**                 | `spec.initContainers`                    | `spec.initContainers` with `restartPolicy: Always` |

→ Init = setup phase only  
→ Sidecar = long-running helpers

## How Init Containers Work

1. kubelet starts first init container
2. Waits for it to exit with code 0
3. If fails → restart (backoff delay)
4. Once successful → next init container
5. After **all** init containers succeed → app containers start (in parallel)
6. Pod status reflects init container statuses in `.status.initContainerStatuses`

## Advantages & Use Cases

- Separate **utilities/tools** (sed, awk, curl, dig, python, mysql client, etc.) without bloating app image
- Independent build/deploy roles (app team ≠ infra/setup team)
- Different **filesystem view** / access:
  - Can mount Secrets/ConfigMaps that app containers cannot see
- **Block/delay** app startup until preconditions met:
  - Wait for database ready
  - Wait for ConfigMap/Secret to appear
  - Wait for network dependency/service
  - Clone Git repo → shared volume
  - Generate config files
- **Security**: Keep dangerous tools (e.g., netcat, debuggers) out of production app image → smaller attack surface

## Example YAML Snippet

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app.kubernetes.io/name: MyApp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup myservice.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', "until nslookup mydb.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for mydb; sleep 2; done"]
```

## Important Notes

- Resource requests/limits for init containers are **considered separately** during scheduling (see Resource sharing docs)
- Init containers **share volumes** with app containers → great for passing data/config
- If Pod has `restartPolicy: Never` and init fails → entire Pod fails (no retries)
- Status: Check `kubectl describe pod` or `.status.initContainerStatuses`

## Summary – When to Use Init Containers

Use init containers when you need to:
- Perform setup or wait tasks **before** your app starts
- Keep app image lean and secure
- Separate concerns (setup vs. runtime)
- Enforce preconditions (dependencies ready)

Do **not** use for long-running services → use sidecars (with `restartPolicy: Always` on init containers since v1.33)

---

#  Detailed Behavior & Resource Management

## Pod Startup Sequence (with Init Containers)

1. kubelet waits until **networking** and **storage** are ready for the Pod
2. Init containers run **sequentially** in the order listed in `spec.initContainers`
3. Each init container must **exit successfully** (code 0) before the next starts
4. After **all** init containers succeed → app containers start (in parallel)
5. Pod becomes **Ready** only after all init containers + app containers are ready

## Restart & Retry Behavior

- If an init container fails (runtime error or non-zero exit):
  - kubelet retries according to Pod's `restartPolicy`
  - But init containers always use **effective restartPolicy: OnFailure** (even if Pod is `Always`)
- If Pod `restartPolicy: Never` and init fails → entire Pod fails
- Pod **cannot be Ready** until all init containers succeed
  - Status: `Pending` phase with `Initialized: False` condition
- **Ports** defined on init containers are **not** published/aggregated to Services
- If Pod restarts (any reason) → **all init containers re-execute** from the beginning

## Updating Init Containers

- **Direct Pod updates** (patch/replace):
  - Only `image` field of init container can be changed
  - Changing image **does not** restart/recreate the Pod
  - If Pod hasn't started yet → new image may be used on next start
- **In Pod template** (Deployment, StatefulSet, Job, etc.):
  - Any field can usually be changed
  - Behavior depends on controller's update strategy (e.g., rolling update → new Pods created)
- **No automatic restart** just from changing init container image (v1.20+)

## Idempotency Requirement

- Init containers can be **re-run**, **retried**, or **re-executed** multiple times
- Code must be **idempotent** (safe to run multiple times)
  - Especially important for writes to `emptyDir` volumes
  - Check if file/output already exists before creating/overwriting

## Probes & Validation Restrictions

- Init containers support **almost all** fields of app containers
- **Prohibited** (validation error):
  - `readinessProbe` (cannot be ready before completion)
  - `livenessProbe`, `startupProbe` (not meaningful for completion-based execution)
  - `lifecycle` hooks (postStart/preStop)
- Sidecar containers (with `restartPolicy: Always`) support probes

## Preventing Infinite Failures

- Use `activeDeadlineSeconds` on the **Pod** to set overall timeout
  - Includes time spent in init containers
  - If exceeded → Pod terminated
- **Caution**: Only recommended for **Jobs** (batch workloads)
  - On long-running Pods (Deployment/StatefulSet), it will kill healthy running Pods after deadline
- Alternative: Use readiness/liveness on app containers + proper retry logic in init

## Container Name Uniqueness

- All container names (init + app) in a Pod must be **unique**
- Duplicate names → **validation error**

## Resource Sharing & Effective Requests/Limits

Init containers affect Pod scheduling and resource allocation:

1. **Effective init request/limit** for each resource:
   - Highest request or limit among **all** init containers
   - If no limit → considered unlimited (highest possible)

2. **Pod's effective request/limit** for a resource:
   - **max**( sum of all app containers' requests/limits , effective init request/limit )

3. **Scheduling** uses **effective Pod requests/limits**
   - Init containers can "reserve" resources temporarily (released after init phase)

4. **QoS class**:
   - Determined by effective Pod requests/limits
   - Applies equally to init and app containers

5. **Quota & Limits** enforced based on effective Pod values

→ Init containers can temporarily require **more resources** than the app needs long-term

## Linux cgroups Behavior

- Pod-level cgroups use **effective Pod request/limit**
- Same rules as scheduler → init containers influence cgroup setup during their execution

## Reasons a Pod Restarts (Causing Init Re-execution)

1. **Pod infrastructure container** restarted (rare, requires node root access)
2. All containers terminated + `restartPolicy: Always` → Pod restart
   - Init completion record **lost** due to garbage collection (edge case)

**Important (v1.20+)**:
- Changing init container image **does NOT** cause restart
- Lost completion record due to GC **does NOT** cause restart

(For Kubernetes < v1.20 → behavior may differ; check version docs)

## Summary – Critical Takeaways

- Init containers run **sequentially**, **to completion**, **before** app containers
- **Idempotent** code is mandatory
- **No readinessProbe** allowed
- **Resource requests** can be higher during init (affects scheduling)
- Pod **restarts** → init containers **re-run**
- Use `activeDeadlineSeconds` **only** for Jobs
- Effective resources = max(init peak, app sum)

---