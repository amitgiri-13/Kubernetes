### Cluster Maintenance in Kubernetes 

**cluster maintenance activities** that Kubernetes administrators perform to keep a cluster healthy, secure, and available.

#### 1. Operating System Upgrades

* Learn what happens when a node becomes unavailable.
* Understand the impact of:

  * Unexpected node failures.
  * Intentionally taking a node offline for maintenance.
* Common maintenance tasks include:

  * Applying OS patches.
  * Security updates.
  * Kernel upgrades.

#### 2. Cluster Upgrade Process

Before upgrading a Kubernetes cluster, it's important to understand:

* Kubernetes release cycles.
* Version numbering and compatibility.
* Upgrade best practices:

  * When to upgrade.
  * Which version to upgrade to.
  * Supported version skew between components.


#### 3. End-to-End Cluster Upgrade Lab

* Upgrade a Kubernetes cluster yourself.
* Work with a live cluster running applications.
* Verify that workloads continue functioning during and after the upgrade.

#### 4. Backup and Restore

* Creating cluster backups.
* Protecting cluster configuration and state.
* Understanding recovery procedures.

#### 5. Disaster Recovery Exercise

1. Take a backup of the cluster.
2. Simulate a disaster or cluster failure.
3. Restore the cluster from backup.
4. Return the cluster to its previous working state.

---

### Key Learning Objectives

By the end of this section, you should be able to:

* Safely maintain Kubernetes worker and control-plane nodes.
* Drain and restore nodes during maintenance.
* Understand Kubernetes versioning and upgrade strategies.
* Perform cluster upgrades with minimal downtime.
* Create and restore cluster backups.
* Recover from cluster failures and disaster scenarios.

This section is particularly important for the **Certified Kubernetes Administrator (CKA)** exam because cluster upgrades, node maintenance, and backup/restore operations are common exam objectives and real-world administrative tasks.
