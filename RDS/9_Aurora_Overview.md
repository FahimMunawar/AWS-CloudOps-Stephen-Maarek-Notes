# 9. Amazon Aurora Overview

## Overview

**Amazon Aurora** is AWS's proprietary, cloud-optimized relational database engine compatible with MySQL and PostgreSQL. It delivers significantly higher performance than standard RDS, with built-in high availability, auto-scaling storage, and sub-30-second failover — at approximately 20% higher cost than RDS.

---

## Aurora vs RDS — Key Comparisons

| Feature | RDS MySQL/PostgreSQL | Amazon Aurora |
|---------|---------------------|---------------|
| **Performance** | Baseline | 5x MySQL, 3x PostgreSQL |
| **Storage growth** | Manual (Auto Scaling with threshold) | Automatic: 10 GiB → up to 256 TiB |
| **Read replicas** | Up to 15 | Up to 15, sub-10ms lag |
| **Failover time** | ~60–120 seconds | **< 30 seconds** |
| **HA by default** | No (must enable Multi AZ) | Yes — cloud-native |
| **Storage copies** | EBS in one AZ | **6 copies across 3 AZs** |
| **Cost** | Baseline | ~20% more than RDS |
| **Driver compatibility** | Native MySQL/PostgreSQL | Same drivers work |

---

## Aurora Storage Architecture

Aurora uses a **shared, distributed storage volume** — not a traditional EBS volume.

| Property | Detail |
|----------|--------|
| **Copies** | 6 copies of every write, spread across 3 AZs |
| **Write quorum** | 4 out of 6 copies needed — survives 1 AZ failure |
| **Read quorum** | 3 out of 6 copies needed — highly available for reads |
| **Self-healing** | Corrupted data repaired automatically via peer-to-peer replication |
| **Volumes** | Hundreds of volumes striped — reduces risk significantly |
| **Auto-expanding** | Starts at 10 GiB, grows automatically up to 256 TiB — no manual intervention |

```
Write data (blue):  6 copies → AZ-A ✓  AZ-B ✓  AZ-C ✓
Write data (orange): 6 copies → AZ-A ✓  AZ-B ✓  AZ-C ✓
Storage is logical, distributed, self-healing, auto-expanding
```

---

## Aurora Cluster Architecture

```
                    ┌─────────────────────────────────┐
                    │       Shared Storage Volume      │
                    │  (auto-expanding, self-healing)  │
                    └──────────────┬──────────────────┘
                                   │ writes only
                            ┌──────▼──────┐
Writer Endpoint  ──────────►│   Master    │
(DNS, always points         └─────────────┘
 to current master)
                            ┌─────────────┐
Reader Endpoint  ──────────►│  Replica 1  │◄── reads
(load balances              ├─────────────┤
 across all replicas)       │  Replica 2  │◄── reads
                            ├─────────────┤
                            │  Replica N  │◄── reads (up to 15)
                            └─────────────┘
                                   ▲
                             Auto Scaling
                         (adjusts replica count)
```

---

## Writer Endpoint and Reader Endpoint

| Endpoint | Points To | Purpose |
|----------|-----------|---------|
| **Writer endpoint** | Current master (DNS-based) | Always routes writes to the correct master, even after failover |
| **Reader endpoint** | All read replicas (load balanced) | Distributes read connections across replicas — connection-level load balancing |

> Load balancing in Aurora happens at the **connection level**, not the statement level.

---

## Failover

- Automatic failover to a read replica in **< 30 seconds**
- Any read replica can be promoted to master
- Writer endpoint automatically redirects after failover — no application change needed

---

## Aurora Features Summary

| Feature | Detail |
|---------|--------|
| **Automatic failover** | < 30 seconds |
| **Backup and recovery** | Continuous, Point-in-Time Restore |
| **Backtrack** | Restore to any point in time **without using backups** (rewinds in-place) |
| **Auto-scaling replicas** | Scale read replicas up/down automatically |
| **Automated patching** | Zero-downtime patching |
| **Advanced monitoring** | Built-in Performance Insights support |
| **Cross-region replication** | Supported via Aurora Global Database |
| **Security** | Encryption at rest and in transit, IAM auth, VPC isolation |

---

## Best Practices

✓ **Use the writer endpoint for all writes** — it survives failover without application changes  
✓ **Use the reader endpoint for all reads** — connection load balancing is handled automatically  
✓ **Enable auto-scaling on read replicas** — let Aurora adjust replica count based on load  
✓ **Use Aurora for high-traffic production workloads** — the performance/HA gains justify the 20% cost premium  
✓ **Use Backtrack for quick point-in-time recovery** — faster than restore from backup for recent mistakes  

---

## SysOps Exam Focus

**Q1: "Your application needs a MySQL-compatible database with automatic storage scaling, sub-30-second failover, and up to 15 read replicas. Which service should you use?"**
- A) RDS MySQL with Multi AZ
- B) Amazon Aurora MySQL-compatible
- C) RDS MySQL with Read Replicas
- D) DynamoDB
- **Answer: B** — Aurora provides all these features natively: auto-expanding storage, fast failover, and up to 15 replicas

**Q2: "How many copies of data does Aurora store, and across how many Availability Zones?"**
- A) 2 copies across 2 AZs
- B) 3 copies across 3 AZs
- C) 6 copies across 3 AZs
- D) 6 copies across 6 AZs
- **Answer: C** — Aurora stores 6 copies of every write across 3 AZs; writes require 4/6 and reads require 3/6

**Q3: "Your Aurora cluster has 5 read replicas. The number of replicas fluctuates based on load. How should your application connect to the read replicas?"**
- A) Hardcode each replica's endpoint in the application
- B) Use the writer endpoint and enable read routing
- C) Use the reader endpoint — it load balances connections across all replicas automatically
- D) Use Route 53 latency routing to the replicas
- **Answer: C** — The reader endpoint abstracts the replica list and load balances at the connection level

**Q4: "What is Aurora Backtrack and how does it differ from Point-in-Time Restore?"**
- A) Backtrack creates a new database instance; Point-in-Time Restore restores in place
- B) Backtrack rewinds the existing database to a previous point in time without creating a new instance; Point-in-Time Restore creates a new DB instance
- C) They are identical features with different names
- D) Backtrack only works with Aurora PostgreSQL
- **Answer: B** — Backtrack rewinds the database in place (fast, no new instance); Point-in-Time Restore always creates a new instance

**Q5: "An Aurora master instance fails. How long does it take for failover to complete, and what happens to the writer endpoint?"**
- A) ~60–120 seconds; application must update connection string to new master endpoint
- B) < 30 seconds; the writer endpoint automatically redirects to the promoted read replica
- C) < 30 seconds; application must manually promote a read replica
- D) ~5 minutes; Aurora takes a snapshot and restores to a new instance
- **Answer: B** — Aurora failover completes in under 30 seconds and the writer endpoint DNS automatically points to the new master

---

## Quick Reference

```
Aurora:
  Compatible with: MySQL, PostgreSQL (same drivers)
  Performance: 5x MySQL RDS, 3x PostgreSQL RDS
  Storage: auto-expands 10 GiB → 256 TiB, no manual action needed
  Copies: 6 across 3 AZs | Write quorum: 4/6 | Read quorum: 3/6
  Failover: < 30 seconds (vs ~60-120s for RDS Multi AZ)
  Read replicas: up to 15, sub-10ms lag, auto-scaling supported
  Cost: ~20% more than RDS

Endpoints:
  Writer endpoint → always points to current master (survives failover)
  Reader endpoint → load balances reads across all replicas (connection-level)

Key features:
  Backtrack      → rewind in place, no new instance
  Auto-scaling   → replicas scale up/down automatically
  Zero-downtime  → automated patching
  Self-healing   → peer-to-peer storage repair
```

---

**File: 9_Aurora_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
