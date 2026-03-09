# ETCD

A distributed, reliable key-value store used by Kubernetes to persist cluster configuration, state, and metadata. It ensures that all control plane components have a consistent view of the cluster.

## Setup etcd

```bash
# install (fedora)
sudo dnf install etcd

# enable and start
sudo systemctl enable etcd
sudo systemctl start etcd

# checking status
sudo systemctl status etcd

# set environment variables for client
export ETCDCTL_API=3
export ETCDCTL_ENDPOINTS=http://127.0.0.1:2379

```

Basic Key-Value Operations

```bash
# put a key
etcdctl put mykey "myvalue"

# get a key
etcdctl get mykey

# get all keys
etcdctl get "" --prefix

# delete a key
etcdctl del mykey

# delete key with prefix
etcdctl del "" --prefix
```

Advanced Operation

```bash
# check cluster health
etcdctl endpoint health

# list members 
etcdctl member list

# add member
etcdctl member add <name> --peer-urls=<url>

# remove member
etcdctl member remove <ID>

# snapshot
etcdctl snapshot save snapshot.db

# restore snapshot
etcdctl snapshot restore snapshot.db
```


