
# Container Lifecycle Hooks in Kubernetes

Kubernetes provides **lifecycle hooks** for containers, allowing code to run at specific points during a container's lifecycle.

---

## 1. Overview

* Similar to lifecycle hooks in programming frameworks (e.g., Angular)
* Enables containers to react to **management events**
* Hooks execute a **handler** when triggered

---

## 2. Container Hooks

| Hook           | Trigger                                                                                                      | Notes                                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------- |
| **PostStart**  | Immediately after container creation                                                                         | May run **before or simultaneously with ENTRYPOINT**. No parameters are passed.                                                 |
| **PreStop**    | Immediately before container termination (API request, probe failure, preemption, resource contention, etc.) | Must complete before TERM signal is sent. Termination grace period countdown starts **before execution**. No parameters passed. |
| **StopSignal** | Defines a custom signal to stop the container, overriding image STOPSIGNAL                                   | Used for custom termination behavior                                                                                            |

> For more details: [Termination of Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#termination-of-pods), [Stop Signals](https://kubernetes.io/docs/concepts/containers/container-lifecycle-hooks/#stopsignal)

---

## 3. Hook Handler Implementations

Three types of handlers can be used:

| Type      | Description                                                                        |
| --------- | ---------------------------------------------------------------------------------- |
| **Exec**  | Runs a command inside the container’s cgroups and namespaces (e.g., `pre-stop.sh`) |
| **HTTP**  | Sends an HTTP request to a container endpoint                                      |
| **Sleep** | Pauses execution for a specified duration                                          |

---

## 4. Hook Execution Behavior

* **PostStart**

  * Triggered **when container is created**
  * ENTRYPOINT and PostStart run **simultaneously**
  * HTTP hooks may be unreliable (container process may not have started)
  * Long hooks may **block container from running**

* **PreStop**

  * Executes **before TERM signal**
  * Must finish **before container stops**
  * Hook + container stop time counted against `terminationGracePeriodSeconds`
  * Example: If grace period is 60s, hook takes 55s, container stops in 10s → container killed after 60s

* **Failure Behavior**

  * If PostStart or PreStop hook fails → **container is killed**
  * Keep hooks lightweight, except for necessary long-running tasks (e.g., state saving)

---

## 5. Hook Delivery Guarantees

* Hooks are **at least once**

  * May be called multiple times for a single event
* Typically **single delivery**
* Rare double deliveries possible (e.g., kubelet restarts during hook)

---

## 6. Debugging Hook Handlers

* Hook logs **not exposed in Pod events**
* Failures generate events:

  * PostStart → `FailedPostStartHook`
  * PreStop → `FailedPreStopHook`
* Example: To test failure:

  ```yaml
  postStart:
    exec:
      command: ["badcommand"]
  ```

  * Run: `kubectl describe pod lifecycle-demo`
  * Event shows hook failure

---

## 7. Summary

* Lifecycle hooks provide **container awareness of lifecycle events**
* Two main hooks: **PostStart** and **PreStop**
* Handlers types: **Exec, HTTP, Sleep**
* Execution affects **container startup and termination**
* Hooks should be **lightweight** but can include necessary long-running commands
* Hook delivery is **at least once**, so implementations must handle possible retries

---

