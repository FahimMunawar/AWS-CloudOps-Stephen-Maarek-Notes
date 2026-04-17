# 2. RDS Read Replicas and Multi AZ

## Overview

**Read Replicas** scale read throughput by distributing SELECT queries across up to 15 replica instances. **Multi AZ** provides high availability and disaster recovery via a synchronous standby. These are the two most exam-critical RDS concepts — know the difference cold.

---

## Read Replicas

### Key Facts

| Property | Detail |
|----------|--------|
| **Purpose** | Scale reads (SELECT queries only) |
| **Max replicas** | 15 |
| **Replication type** | **Asynchronous** — eventually consistent |
| **Placement options** | Same AZ, cross-AZ, cross-region |
| **Allowed SQL** | SELECT only — no INSERT, UPDATE, DELETE |
| **Promotion** | Can be promoted to a standalone database |
| **Connection string** | App must be updated to include replica endpoints |

### Classic Use Case

```
Production app  ──read/write──► Main RDS instance
                                     │
                         (async replication)
                                     ↓
Reporting app   ──read────────► Read Replica
```

Offload analytics/reporting to a replica — production performance is unaffected.

---

## Read Replica Network Costs

| Scenario | Replication Cost |
|----------|-----------------|
| Same region, different AZ | **Free** — managed service exception |
| Cross-region | **Charged** — data crosses regional boundary |

> Same-region replication (even cross-AZ) is free for RDS managed service.

---

## Multi AZ

### Key Facts

| Property | Detail |
|----------|--------|
| **Purpose** | High availability / Disaster Recovery |
| **Replication type** | **Synchronous** — write accepted only after standby confirms |
| **Standby usage** | **Cannot read or write** — standby only |
| **Failover** | Automatic via single DNS name |
| **Failover triggers** | AZ failure, network loss, instance failure, storage failure |
| **Manual intervention** | None required — app reconnects to same DNS name |

### Architecture

```
Application ──► One DNS name
                    │
           ┌────────┴────────┐
           ▼                 ▼
       Master DB         Standby DB
       (AZ A)            (AZ B)
   reads + writes     no traffic (standby only)
           │                 ▲
           └──synchronous────┘
              replication
```

---

## Read Replica as Multi AZ

A Read Replica can itself be configured as Multi AZ — this is a common exam question.

- The replica handles reads
- The replica's standby provides HA for the replica itself
- Useful for cross-region DR where you also need replica HA

---

## Single AZ → Multi AZ Migration

**Zero downtime operation** — no need to stop the database.

```
Click Modify → Enable Multi AZ
        ↓
RDS automatically takes a snapshot of the main DB
        ↓
Snapshot is restored into a new standby instance
        ↓
Synchronization is established between main and standby
        ↓
Multi AZ is active — standby is caught up
```

> The entire process is handled by AWS. Your application continues running throughout.

---

## Read Replicas vs Multi AZ — Side by Side

| Feature | Read Replicas | Multi AZ |
|---------|--------------|----------|
| **Primary purpose** | Read scaling | High availability / DR |
| **Replication** | Asynchronous | Synchronous |
| **Consistency** | Eventually consistent | Strongly consistent |
| **Can serve reads?** | Yes | No (standby only) |
| **Failover** | Manual promotion | Automatic |
| **DNS** | Separate endpoint per replica | Single DNS name |
| **Cross-region?** | Yes | No (same region, different AZ) |
| **Max instances** | 15 | 1 standby |

---

## Best Practices

✓ **Use Read Replicas for reporting/analytics** — prevents read-heavy workloads from degrading production  
✓ **Use Multi AZ for production databases** — automatic failover with no code changes required  
✓ **Remember: standby in Multi AZ is not readable** — if you need a readable standby, use Aurora  
✓ **Cross-region Read Replicas incur network fees** — factor into cost estimates  
✓ **Migrate Single AZ to Multi AZ without downtime** — just click Modify and enable it  

---

## SysOps Exam Focus

**Q1: "Your analytics team wants to run heavy reports against your production RDS database without impacting performance. What is the best solution?"**
- A) Enable Multi AZ on the production database
- B) Create a Read Replica and point the reporting application to it
- C) Take a daily snapshot and restore it for reporting
- D) Increase the instance size of the production database
- **Answer: B** — Read Replicas offload read-heavy workloads from the primary without affecting production

**Q2: "What type of replication does RDS Multi AZ use, and what does that mean for consistency?"**
- A) Asynchronous — eventually consistent
- B) Synchronous — every write is confirmed on the standby before being acknowledged
- C) Asynchronous — strongly consistent
- D) Batch replication — consistent after each batch
- **Answer: B** — Multi AZ uses synchronous replication; data is written to both master and standby simultaneously

**Q3: "Can a Read Replica itself be configured as Multi AZ?"**
- A) No — Read Replicas and Multi AZ are mutually exclusive
- B) Yes — a Read Replica can have its own standby for high availability
- C) Only if the primary is also Multi AZ
- D) Only for cross-region replicas
- **Answer: B** — Read Replicas can be set up as Multi AZ; this is a common exam question

**Q4: "You need to convert an RDS Single AZ instance to Multi AZ. What is the impact on availability during the conversion?"**
- A) The database must be stopped for approximately 20 minutes
- B) A maintenance window is required; the database restarts
- C) Zero downtime — modify the setting and AWS handles the rest automatically
- D) You must create a new Multi AZ instance and migrate data manually
- **Answer: C** — Single AZ to Multi AZ is a zero-downtime operation; RDS snapshots and restores internally

**Q5: "Your RDS Read Replica is in us-east-1b and the primary is in us-east-1a. What is the cost of replication traffic?"**
- A) Standard cross-AZ data transfer rates
- B) Free — same-region replication between AZs is free for RDS managed service
- C) Free only if both instances use the same instance type
- D) Charged at the inter-region rate
- **Answer: B** — RDS managed service replication within the same region is free regardless of AZ

**Q6: "What happens to the standby instance in an RDS Multi AZ setup during normal operations?"**
- A) It serves read traffic to reduce load on the primary
- B) It is used for automated backups
- C) It receives no traffic — it is purely a failover target
- D) It handles write traffic when the primary is under load
- **Answer: C** — The standby in Multi AZ is not accessible; it only becomes the primary after failover

---

## Quick Reference

```
Read Replicas:
  Purpose:     Scale reads
  Replication: Asynchronous (eventually consistent)
  Traffic:     SELECT only
  Max:         15 replicas
  Placement:   Same AZ / cross-AZ / cross-region
  Cost:        Same-region replication is FREE; cross-region costs money
  Use case:    Reporting, analytics, read-heavy workloads

Multi AZ:
  Purpose:     High availability / Disaster Recovery
  Replication: Synchronous (strongly consistent)
  Standby:     No reads, no writes — failover only
  Failover:    Automatic via single DNS name
  Use case:    Production databases needing HA

Single AZ → Multi AZ:
  Zero downtime — Modify → Enable Multi AZ
  AWS: snapshot → restore to standby → sync established

Read Replica can be Multi AZ: Yes
```

---

**File: 2_RDS_Read_Replicas_and_Multi_AZ.md**
**Status: SysOps-focused, exam-ready, concise format**
