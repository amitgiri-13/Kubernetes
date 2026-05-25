# Kubernetes Multiple Schedulers Lab

## Check Default Scheduler Pod

List pods in `kube-system` namespace:

```bash
kubectl get pods -n kube-system
```

Default scheduler pod:

```bash
kube-scheduler-controlplane
```

---

## Check Scheduler Image

Describe the scheduler pod:

```bash
kubectl describe pod kube-scheduler-controlplane -n kube-system
```

Find the image section:

```bash
registry.k8s.io/kube-scheduler:v1.23.0
```

---

## Verify Service Account

Check service accounts:

```bash
kubectl get sa -n kube-system
```

Custom scheduler service account:

```bash
my-scheduler
```

---

## Create ConfigMap

Configuration file already exists.

Create ConfigMap from file:

```bash
kubectl create configmap my-scheduler-config \
  --from-file=/root/my-scheduler-config.yaml \
  -n kube-system
```

Verify:

```bash
kubectl get configmap -n kube-system
```

---

## Deploy Custom Scheduler

Open scheduler manifest:

```bash
vi /root/my-scheduler.yaml
```

Update image field with the same scheduler image:

```yaml
image: registry.k8s.io/kube-scheduler:v1.23.0
```

Manifest already contains:

* Liveness probe
* Readiness probe
* ConfigMap volume mount
* Service account

Create scheduler deployment:

```bash
kubectl create -f /root/my-scheduler.yaml
```

Verify:

```bash
kubectl get pods -n kube-system
```

---

## Create Pod Using Custom Scheduler

Open pod definition:

```bash
vi /root/nginx-pod.yaml
```

Add scheduler name:

```yaml
spec:
  schedulerName: my-scheduler
```

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  schedulerName: my-scheduler
  containers:
    - name: nginx
      image: nginx
```

Create pod:

```bash
kubectl create -f /root/nginx-pod.yaml
```

Verify pod:

```bash
kubectl get pods
```

---

# Key Concepts

* Kubernetes supports multiple schedulers.
* Default scheduler runs as a pod in `kube-system`.
* Custom schedulers require:

  * Service Account
  * RBAC permissions
  * Scheduler configuration
  * Scheduler deployment
* Pods use custom schedulers with:

```yaml
schedulerName: my-scheduler
```