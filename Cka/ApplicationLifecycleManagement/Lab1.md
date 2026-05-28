# Kubernetes Rolling Updates and Rollbacks 

## Lab Objective

This lab demonstrates:

* Rolling Updates
* Recreate Strategy
* Deployment Upgrades
* Zero Downtime Deployment
* Deployment Downtime
* Updating container images
* Inspecting deployment strategies

---

# 1. Initial Environment Inspection

## Check Pods

```bash id="q9drxp"
kubectl get pods
```

Observation:

* Frontend application Pods are running
* Deployment exists with 4 replicas

---

## Check Deployments

```bash id="ndspk9"
kubectl get deployments
```

Output shows:

* Deployment name: `frontend`
* Replicas: `4`

---

# 2. Access Application

Application initially displays:

```text id="m9vx4q"
Blue Color
```

This confirms:

* Current application version is active
* Service routing works properly

---

# 3. Run Load Testing Script

## Execute Curl Test

```bash id="7nyq4d"
./curl-test.sh
```

Purpose:

* Sends multiple requests to application
* Simulates user traffic
* Helps observe deployment behavior during updates

Initial result:

```text id="yk1yl5"
All responses return BLUE
```

---

# 4. Inspect Deployment Details

## Describe Deployment

```bash id="r3y1gu"
kubectl describe deployment frontend
```

---

## Important Findings

### Number of Pods

```text id="nmvt7d"
4 Pods
```

---

## Current Container Image

```text id="1jl6qv"
kodekloud/webapp-color:v1
```

---

## Current Deployment Strategy

```text id="wpv3t7"
RollingUpdate
```

---

# 5. Rolling Update Strategy

## Behavior

Rolling Update:

* Updates Pods gradually
* Avoids downtime
* Old Pods replaced one at a time

### Key Point

```text id="ozl4hm"
RollingUpdate is the default deployment strategy
```

---

# 6. Upgrade Application to Version 2

## Method 1 (Alternative)

Edit deployment manually:

```bash id="u4s83q"
kubectl edit deployment frontend
```

Change image:

```yaml id="e1r5mf"
image: kodekloud/webapp-color:v2
```

---

## Method 2 (Used in Lab)

### Set Image Command

```bash id="5lcj61"
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
```

Where:

* `frontend` → deployment name
* `simple-webapp` → container name

---

## Verify Update

```bash id="x8gq8f"
kubectl describe deployment frontend
```

Result:

```text id="f8dyzz"
Image updated to v2
```

---

# 7. Observe Rolling Update

## Run Curl Test Again

```bash id="mjlwm2"
./curl-test.sh
```

---

## Observed Behavior

### Initially

```text id="00m2h4"
Mostly BLUE responses
```

### During Update

```text id="2lh8zy"
Mix of BLUE and GREEN responses
```

### After Completion

```text id="ct5hyj"
All responses become GREEN
```

---

# Why This Happens

Rolling update replaces Pods gradually:

```text id="ewujvl"
Old Pod → New Pod
Old Pod → New Pod
```

Traffic gets routed to:

* Old Pods (Blue)
* New Pods (Green)

during transition.

---

# 8. Rolling Update Configuration

## Inspect Strategy Details

```bash id="4w09du"
kubectl describe deployment frontend
```

Important section:

```text id="1h0gwo"
25% max unavailable
```

---

# Maximum Pods Down During Upgrade

Deployment has:

```text id="06n0tw"
4 Replicas
```

25% of 4:

```text id="q5u2y2"
1 Pod
```

Meaning:

* Only 1 Pod can be unavailable at a time

This ensures:

```text id="z2g0fd"
Application remains available during updates
```

---

# 9. Change Strategy to Recreate

## Edit Deployment

```bash id="mijv04"
kubectl edit deployment frontend
```

---

## Modify Strategy

Change:

```yaml id="6kx5sx"
strategy:
  type: RollingUpdate
```

To:

```yaml id="6zz7s7"
strategy:
  type: Recreate
```

---

## Remove RollingUpdate Parameters

Delete:

```yaml id="3jlwm4"
rollingUpdate:
  maxUnavailable: 25%
```

Because:

```text id="ah1zkv"
Recreate strategy does not use rollingUpdate settings
```

---

## Verify Strategy

```bash id="n5ah9k"
kubectl describe deployment frontend
```

Result:

```text id="mvq3p9"
StrategyType: Recreate
```

---

# 10. Upgrade Application to Version 3

## Update Image

```bash id="qup5db"
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v3
```

---

# 11. Observe Recreate Strategy Behavior

## Run Curl Test

```bash id="a6zuvm"
./curl-test.sh
```

---

## Observations

### During Upgrade

Requests fail:

```text id="sowjlwm"
Connection failures / Bad Gateway
```

Reason:

```text id="a8npf0"
All old Pods are terminated before new Pods start
```

This causes:

```text id="4f0fvn"
Temporary application downtime
```

---

## Browser Result

Application temporarily shows:

```text id="r5f1k9"
Bad Gateway
```

---

## After Upgrade Completion

Application becomes:

```text id="pyz0kc"
RED Color
```

Version:

```text id="g53oj9"
Application Version 3
```

---

# RollingUpdate vs Recreate

| Feature          | RollingUpdate | Recreate          |
| ---------------- | ------------- | ----------------- |
| Downtime         | No            | Yes               |
| Pod Replacement  | Gradual       | All at once       |
| Default Strategy | Yes           | No                |
| User Experience  | Seamless      | Interrupted       |
| Availability     | High          | Low during update |

---

# Important Commands Summary

| Task                | Command                                                        |
| ------------------- | -------------------------------------------------------------- |
| Get Pods            | `kubectl get pods`                                             |
| Get Deployments     | `kubectl get deployments`                                      |
| Describe Deployment | `kubectl describe deployment frontend`                         |
| Edit Deployment     | `kubectl edit deployment frontend`                             |
| Update Image        | `kubectl set image deployment/frontend simple-webapp=image:v2` |
| Run Load Test       | `./curl-test.sh`                                               |

---

# Key Concepts Learned

## Rolling Update

* Default Kubernetes deployment strategy
* Zero downtime deployment
* Incremental Pod replacement

---

## Recreate Strategy

* Deletes all old Pods first
* Causes temporary downtime
* Simpler deployment model

---

## MaxUnavailable

Controls:

```text id="a3u5lm"
How many Pods can be unavailable during update
```

Example:

```text id="tmryzi"
25% of 4 replicas = 1 Pod
```

---

# Real Production Recommendation

## Use RollingUpdate for:

* Web applications
* APIs
* Production workloads
* High availability systems

## Use Recreate for:

* Legacy applications
* Applications requiring complete shutdown
* Non-critical systems
* Stateful compatibility scenarios
