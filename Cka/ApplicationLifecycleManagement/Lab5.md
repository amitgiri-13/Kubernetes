# Kubernetes Init Containers Lab Notes

This lab focuses on identifying, troubleshooting, and configuring Init Containers.

---

# Question 1: Which Pod Has an Init Container?

List all pods:

```bash
kubectl get pods
```

Output:

```text
NAME
red
green
blue
```

To inspect all pods:

```bash
kubectl describe pods
```

### Observation

**Red Pod**

```text
Containers:
  red-container
```

**Green Pod**

```text
Containers:
  green-container-1
  green-container-2
```

**Blue Pod**

```text
Init Containers:
  init-service

Containers:
  blue-container
```

### Answer

```text
blue
```

---

# Question 2: Which Image Is Used by the Init Container?

Inspect blue pod:

```bash
kubectl describe pod blue
```

Output:

```text
Init Containers:
  init-service
    Image: busybox
```

### Answer

```text
busybox
```

---

# Question 3: What Is the State of the Init Container?

From:

```bash
kubectl describe pod blue
```

Output:

```text
State: Terminated
```

### Answer

```text
Terminated
```

---

# Question 4: Why Is It Terminated?

Output:

```text
Reason: Completed
Exit Code: 0
```

The init container executed successfully.

Example command:

```yaml
command:
- sh
- -c
- sleep 5
```

Execution flow:

```text
Pod Created
     ↓
Init Container Runs
     ↓
Sleep 5 Seconds
     ↓
Exit Successfully
     ↓
Main Container Starts
```

### Answer

```text
Completed
```

---

# Question 5: How Many Init Containers Does Purple Pod Have?

Inspect:

```bash
kubectl describe pod purple
```

Output:

```text
Init Containers:
  warmup-1
  warmup-2
```

### Answer

```text
2
```

---

# Question 6: What Is the State of Purple Pod?

Check:

```bash
kubectl get pods
```

Output:

```text
purple   0/1   Init:0/2
```

Or:

```bash
kubectl describe pod purple
```

Output:

```text
Status: Pending
```

### Answer

```text
Pending
```

---

# Question 7: How Long Until the Application Starts?

Pod configuration:

```yaml
initContainers:
- name: warmup-1
  command:
  - sleep
  - "600"

- name: warmup-2
  command:
  - sleep
  - "1200"
```

Calculation:

```text
600 seconds = 10 minutes
1200 seconds = 20 minutes

Total = 30 minutes
```

Execution sequence:

```text
warmup-1 (10 min)
        ↓
warmup-2 (20 min)
        ↓
Application Starts
```

### Answer

```text
30 minutes
```

---

# Question 8: Add an Init Container to Red Pod

Requirement:

* Image: busybox
* Sleep: 20 seconds

Example YAML:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: red

spec:
  initContainers:
  - name: busybox-init
    image: busybox
    command:
    - sleep
    - "20"

  containers:
  - name: red-container
    image: nginx
```

Because many pod fields are immutable, editing often requires:

```bash
kubectl replace --force -f pod.yaml
```

---

# Question 9: Orange Pod Is Failing

Check status:

```bash
kubectl get pods
```

Output:

```text
orange   0/1   Init:CrashLoopBackOff
```

This indicates:

```text
An Init Container is failing
```

---

# Troubleshooting Init Container Failures

### Step 1: Describe the Pod

```bash
kubectl describe pod orange
```

Look for:

```text
Init Containers:
```

and

```text
State: Terminated
Reason: Error
Exit Code: 127
```

---

### Step 2: Check Init Container Logs

Regular logs command:

```bash
kubectl logs orange
```

won't help because the main container never started.

Instead:

```bash
kubectl logs orange -c init-myservice
```

Output:

```text
sleeeep: not found
```

---

### Root Cause

Typo in command:

```yaml
command:
- sleeeep
- "2"
```

Correct:

```yaml
command:
- sleep
- "2"
```

---

### Fix

Edit pod:

```bash
kubectl edit pod orange
```

Correct typo:

```yaml
command:
- sleep
- "2"
```

Recreate if necessary:

```bash
kubectl replace --force -f orange.yaml
```

---

### Verify

Check pod:

```bash
kubectl get pods
```

Output:

```text
orange   1/1   Running
```

Check init container:

```bash
kubectl describe pod orange
```

Output:

```text
State: Terminated
Reason: Completed
Exit Code: 0
```

---

# Init Container Troubleshooting Checklist

When a pod is stuck in:

```text
Init:0/1
Init:0/2
Init:CrashLoopBackOff
```

Use:

```bash
kubectl describe pod <pod-name>
```

Then inspect init container logs:

```bash
kubectl logs <pod-name> -c <init-container-name>
```

Common causes:

| Issue                | Symptom                    |
| -------------------- | -------------------------- |
| Command typo         | Exit Code 127              |
| Missing file         | Init failure               |
| Database unreachable | Init stuck                 |
| API unavailable      | Init stuck                 |
| Script error         | CrashLoopBackOff           |
| Wrong image          | Container creation failure |

---

# CKA Exam Tips

### List Init Containers

```bash
kubectl describe pod <pod>
```

or

```bash
kubectl get pod <pod> -o yaml
```

---

### View Init Container Logs

```bash
kubectl logs <pod> -c <init-container>
```

---

### Check Init Container Status

```bash
kubectl describe pod <pod>
```

Look for:

```text
Init Containers:
State:
Reason:
Exit Code:
```

---

### Remember

```text
Init Container
    ↓
Must Succeed
    ↓
Main Container Starts
```

If any Init Container fails, the application container never starts. This is one of the most frequently tested concepts in Kubernetes troubleshooting and CKA exams.
