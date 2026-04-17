# 10. Amazon Aurora — Hands-On

## Overview

A walkthrough of creating an Aurora MySQL cluster with a writer and reader instance, exploring endpoints, replica auto-scaling, and global database — then cleaning up.

---

## Step 1: Create the Aurora Cluster

**Console:** RDS → Create database → Standard create → **Aurora (MySQL-compatible)**

| Setting | Value |
|---------|-------|
| Engine | Aurora (MySQL-Compatible) |
| Version | Default (e.g., 3.04.1) — check filter for feature support |
| Template | **Production** (exposes all options) |
| DB cluster identifier | `database-2` |
| Master username | `admin` |
| Cluster storage | Aurora Standard (or IO Optimized for high I/O workloads) |
| Instance class | `db.t3.medium` (burstable) or Serverless v2 (ACU min/max) |
| Availability | **Create Aurora Replica in different AZ** — enables HA + fast failover |
| Public access | Yes (for demo purposes) |
| VPC security group | Create new → `demo-database-aurora` |
| Port | 3306 (MySQL) |
| Initial database name | `mydb` |
| Backup retention | 1 day |
| Deletion protection | Optional |

> **Serverless v2:** Instead of choosing an instance type, set minimum and maximum **Aurora Capacity Units (ACUs)** — the cluster scales between them automatically.

---

## Step 2: Cluster Topology After Creation

After creation, the cluster contains:

| Instance | Role | AZ |
|----------|------|----|
| Writer instance | Accepts writes | AZ-A |
| Reader instance | Serves reads | AZ-B (different AZ) |

---

## Step 3: Endpoints

Select the cluster (`database-2`) → **Connectivity & security**

| Endpoint | Type | Use |
|----------|------|-----|
| **Writer endpoint** | Cluster-level DNS | Always routes to the current master — use for writes |
| **Reader endpoint** | Cluster-level DNS | Load balances across all reader instances — use for reads |
| Instance endpoint | Per-instance DNS | Direct connection to a specific instance (not recommended for production) |

> Always use the writer and reader endpoints in your application — they survive failover and replica changes.

---

## Step 4: Add Replica Auto-Scaling

**Console:** Select cluster → **Actions** → **Add replica auto scaling**

| Setting | Value |
|---------|-------|
| Policy name | e.g., `read-replica-scaling-policy` |
| Target metric | Average Aurora Replica CPU utilization or average connections |
| Target value | e.g., 60% |
| Min replicas | 1 |
| Max replicas | Up to 15 |

Auto-scaling adds or removes read replicas automatically based on the metric. The reader endpoint always reflects the current replica set.

---

## Step 5: Aurora Global Database (Optional)

**Console:** Select cluster → **Actions** → **Add AWS Region**

- Adds a replica of the entire cluster in another AWS region
- Requires a compatible Aurora version and instance size (large or above)
- Use for cross-region DR and global read performance

---

## Step 6: Storage Configuration Options

| Option | Use Case |
|--------|----------|
| **Aurora Standard** | Cost-effective; moderate I/O workloads |
| **Aurora I/O Optimized** | High read/write workloads; predictable I/O cost |

---

## Step 7: Cleanup

> Delete in the correct order — you cannot delete the cluster until all instances are removed.

1. Select reader instance → **Actions** → **Delete** → type `delete me` → confirm
2. Select writer instance → **Actions** → **Delete** → type `delete me` → confirm
3. Once both instances are deleted → select the cluster → **Actions** → **Delete** → confirm

---

## Best Practices

✓ **Always use writer/reader endpoints** — direct instance endpoints break on failover  
✓ **Create reader instance in a different AZ** — enables fast failover and cross-AZ read distribution  
✓ **Use replica auto-scaling for variable read workloads** — scale from 1 to 15 replicas based on demand  
✓ **Choose IO Optimized only if I/O is consistently high** — Aurora Standard is cheaper for most workloads  
✓ **Delete instances before the cluster** — Aurora requires all instances removed before cluster deletion  

---

## SysOps Exam Focus

**Q1: "An Aurora cluster has a writer and three reader instances. A new replica is added by auto-scaling. How does the application connect to read from all replicas without any code change?"**
- A) Update the application with each new reader's endpoint
- B) Use the reader endpoint — it automatically load balances across all current reader instances
- C) Use Route 53 to resolve each reader's IP
- D) Use the writer endpoint with read routing enabled
- **Answer: B** — The reader endpoint is cluster-managed and always reflects the current set of readers, including newly added ones

**Q2: "You want your Aurora database to scale compute capacity automatically without selecting a fixed instance type. What option should you use?"**
- A) Aurora Standard with Read Replica auto-scaling
- B) Aurora Serverless v2 — set minimum and maximum ACUs
- C) Enable RDS Storage Auto Scaling
- D) Select the largest available instance class
- **Answer: B** — Aurora Serverless v2 scales between min and max Aurora Capacity Units automatically based on demand

**Q3: "You try to delete an Aurora cluster but the delete option is greyed out. What must you do first?"**
- A) Disable deletion protection only
- B) Delete all reader and writer instances first, then delete the cluster
- C) Take a final snapshot before deletion
- D) Remove the VPC security group association
- **Answer: B** — Aurora clusters cannot be deleted while instances exist; delete reader then writer instances first

---

## Quick Reference

```
Aurora cluster creation:
  Engine: Aurora MySQL or PostgreSQL compatible
  Template: Production (for all options)
  Cluster storage: Standard or IO Optimized
  Instance: db.t3.medium / Serverless v2 (min/max ACU)
  Replica: Create reader in different AZ for HA

Endpoints:
  Writer endpoint → always points to master
  Reader endpoint → load balances across all replicas

Replica auto-scaling:
  Actions → Add replica auto scaling
  Metric: CPU % or connections
  Min: 1 | Max: 15

Global database:
  Actions → Add AWS Region (requires large instance + compatible version)

Cleanup order:
  1. Delete reader instance
  2. Delete writer instance
  3. Delete cluster
```

---

**File: 10_Aurora_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
