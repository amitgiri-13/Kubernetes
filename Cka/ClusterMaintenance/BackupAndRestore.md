# Kubernetes Backup and Restore

In Kubernetes, there are three major components you should consider backing up:

1. **Resource Configurations (YAML Manifests)**
2. **etcd Database**
3. **Persistent Volumes (Application Data)**

---

# 1. Backup Resource Configurations

The preferred approach is to store all Kubernetes manifests in Git.

Example:

```yaml
deployment.yaml
service.yaml
configmap.yaml
secret.yaml
```

Store them in a Git repository.

Advantages:

* Version control
* Easy recovery
* Team collaboration
* GitOps-friendly

If the cluster is lost:

```bash
kubectl apply -f .
```

recreates the resources.

---

## Export Existing Resources

If resources were created imperatively:

```bash
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml
```

Specific resources:

```bash
kubectl get deployments --all-namespaces -o yaml
kubectl get services --all-namespaces -o yaml
kubectl get configmaps --all-namespaces -o yaml
kubectl get secrets --all-namespaces -o yaml
```

---

## Tools for Kubernetes Resource Backup

Popular tool:

* Velero

Velero backs up:

* Kubernetes resources
* Persistent volumes
* Entire namespaces
* Cluster state

---

# 2. Backup etcd

etcd stores:

* Nodes
* Pods
* Deployments
* Services
* Secrets
* ConfigMaps
* Namespaces
* Cluster state

Everything managed by Kubernetes is stored in etcd.

---

## Find etcd Data Directory

For kubeadm clusters:

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

Look for:

```yaml
--data-dir=/var/lib/etcd
```

---

## Create etcd Snapshot

Use:

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
```

Output:

```text
Snapshot saved at snapshot.db
```

---

## Verify Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

Example:

```text
HASH      REVISION    TOTAL KEYS
xxxx      12345       1500
```

---

# Restore etcd Snapshot

Restore snapshot:

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
  --data-dir /var/lib/etcd-from-backup
```

Output:

```text
Restored snapshot
```

A new data directory is created:

```text
/var/lib/etcd-from-backup
```

---

## Update etcd Configuration

Edit:

```bash
vi /etc/kubernetes/manifests/etcd.yaml
```

Change:

```yaml
--data-dir=/var/lib/etcd
```

to:

```yaml
--data-dir=/var/lib/etcd-from-backup
```

Also update volume mounts if necessary.

---

## Restart etcd

Static pod manifests are monitored automatically by kubelet.

Or restart kubelet:

```bash
systemctl daemon-reload
systemctl restart kubelet
```

Verify:

```bash
kubectl get nodes
kubectl get pods -A
```

---

# 3. Backup Persistent Volumes

Kubernetes resources can be restored from etcd or YAML files.

Application data requires separate backup.

Examples:

* Database files
* User uploads
* Application storage

Methods:

* Storage snapshots
* Cloud provider snapshots
* Volume backup tools
* Velero volume backup

Examples:

* Amazon Web Services EBS Snapshots
* Google Cloud Persistent Disk Snapshots
* Microsoft Azure Disk Snapshots

---

# CKA Exam Commands

### Take etcd Backup

```bash
ETCDCTL_API=3 etcdctl snapshot save snapshot.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=<ca.crt> \
--cert=<server.crt> \
--key=<server.key>
```

### Check Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot status snapshot.db
```

### Restore Snapshot

```bash
ETCDCTL_API=3 etcdctl snapshot restore snapshot.db \
--data-dir /var/lib/etcd-restore
```

### Find etcd Certificates

```bash
kubectl describe pod etcd-controlplane -n kube-system
```

Look for:

```text
--cert-file
--key-file
--trusted-ca-file
```

---

# Exam Tip

For CKA backup and restore questions:

1. Locate etcd certificates from the etcd static pod manifest.
2. Take a snapshot using `etcdctl snapshot save`.
3. Verify using `snapshot status`.
4. Restore using `snapshot restore`.
5. Update `etcd.yaml` to point to the restored data directory.
6. Restart kubelet and verify cluster recovery.

In real-world environments, a combination of:

* Git repositories for manifests
* etcd snapshots for cluster state
* Storage snapshots for persistent data

provides a complete Kubernetes backup strategy.
