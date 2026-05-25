# Kubernetes Scheduler Profiles

## Kubernetes Scheduler Workflow

When a Pod is created, it goes through multiple scheduling phases before being assigned to a node.

---

# Scheduler Phases

## 1. Scheduling Queue

Pods first enter the scheduling queue.

Pods are sorted based on priority.

Higher priority pods are scheduled first.

### PriorityClass Example

```yaml id="l6z3bh"
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000000
globalDefault: false
description: "High priority workloads"
```

Pod using priority class:

```yaml id="pc0mho"
spec:
  priorityClassName: high-priority
```

### Plugin Used

```text id="whi7sq"
PrioritySort Plugin
```

Purpose:

* Sorts pods in queue based on priority

---

## 2. Filter Phase

Scheduler removes nodes that cannot run the pod.

### Example

Pod requires:

```text id="ajgtdw"
10 CPU
```

Nodes without enough CPU are filtered out.

### Plugins Used

#### NodeResourcesFit

Checks resource availability.

#### NodeName

Matches specific node name if defined.

```yaml id="wsvnh9"
spec:
  nodeName: worker-1
```

#### NodeUnschedulable

Filters nodes marked unschedulable.

Example command:

```bash id="q18tvu"
kubectl cordon <node-name>
```

---

## 3. Score Phase

Remaining nodes are scored.

Node with highest score gets selected.

### Example

| Node   | CPU Left After Scheduling | Score |
| ------ | ------------------------- | ----- |
| Node A | 2                         | Low   |
| Node B | 6                         | High  |

Node B gets selected.

### Plugins Used

#### NodeResourcesFit

Scores nodes based on free resources.

#### ImageLocality

Prefers nodes already having required container image.

Benefit:

* Faster pod startup
* Reduced image pull time

---

## 4. Bind Phase

Pod is finally assigned to selected node.

### Plugin Used

```text id="7gmb2d"
DefaultBinder
```

Purpose:

* Binds pod to node

---

# Scheduler Plugins

Kubernetes scheduler behavior is controlled by plugins.

A plugin can participate in multiple phases.

Example:

```text id="u2zjlwm"
NodeResourcesFit
```

Used in:

* Filter phase
* Score phase

---

# Extension Points

Scheduler provides extension points where plugins run.

## Main Extension Points

| Extension Point | Purpose                     |
| --------------- | --------------------------- |
| QueueSort       | Sort pods                   |
| PreFilter       | Pre-checks before filtering |
| Filter          | Remove invalid nodes        |
| PostFilter      | Actions after filtering     |
| PreScore        | Before scoring              |
| Score           | Rank nodes                  |
| Reserve         | Reserve resources           |
| Permit          | Approval stage              |
| PreBind         | Before binding              |
| Bind            | Assign pod                  |
| PostBind        | After binding               |

---

# Scheduler Profiles

Before Kubernetes v1.18:

* Multiple schedulers required multiple binaries/processes

Problems:

* More maintenance
* Higher resource usage
* Possible race conditions

---

# Multi Scheduler Profiles (v1.18+)

Kubernetes introduced multiple scheduler profiles inside a single scheduler binary.

Benefits:

* Single scheduler process
* Easier management
* Reduced race conditions

---

# Scheduler Profile Configuration

Example:

```yaml id="q56jhh"
profiles:
  - schedulerName: default-scheduler

  - schedulerName: my-scheduler

  - schedulerName: my-scheduler-2
```

Each profile acts like a separate scheduler.

Pods select scheduler using:

```yaml id="8uof2v"
spec:
  schedulerName: my-scheduler
```

---

# Customizing Plugins Per Profile

Each profile can enable or disable plugins.

Example:

```yaml id="dlb1ij"
profiles:
- schedulerName: my-scheduler-2
  plugins:
    score:
      disabled:
        - name: TaintToleration
      enabled:
        - name: MyCustomPlugin
```

---

# Disable Entire Extension Point

Example:

```yaml id="lttk2r"
profiles:
- schedulerName: my-scheduler-3
  plugins:
    prescore:
      disabled:
        - name: "*"

    score:
      disabled:
        - name: "*"
```

This disables all plugins in:

* PreScore
* Score

---

# Important Concepts

## PriorityClass

Controls pod scheduling priority.

---

## Filtering

Removes unsuitable nodes.

---

## Scoring

Ranks suitable nodes.

---

## Binding

Assigns pod to node.

---

## Scheduler Plugins

Control scheduler behavior.

---

## Scheduler Profiles

Allow multiple scheduler behaviors in a single scheduler process.

---

# Key Advantages of Scheduler Profiles

* Single scheduler binary
* Easier management
* Custom scheduling logic
* Reduced race conditions
* Better extensibility
* Plugin customization per scheduler

---

# Common Built-in Plugins

| Plugin            | Purpose                     |
| ----------------- | --------------------------- |
| PrioritySort      | Sort pods by priority       |
| NodeResourcesFit  | Resource checks and scoring |
| NodeName          | Match node name             |
| NodeUnschedulable | Ignore cordoned nodes       |
| ImageLocality     | Prefer cached images        |
| DefaultBinder     | Bind pod to node            |

---

# Summary

Kubernetes scheduler works in phases:

1. Queue
2. Filter
3. Score
4. Bind

Scheduler plugins control behavior at each phase.

Scheduler profiles allow running multiple customized scheduling behaviors inside a single scheduler process.


