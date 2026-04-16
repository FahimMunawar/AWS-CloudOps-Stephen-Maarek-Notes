# 3. EBS Volume Types

## Overview

EBS volumes come in **six types** grouped into four categories. The key exam skill is knowing which volume type to choose for a given workload — not memorizing exact numbers.

---

## Volume Type Summary

| Type | Category | Boot Volume? | Use Case |
|------|----------|-------------|----------|
| **gp2** | General Purpose SSD | Yes | System boot, dev/test, virtual desktops |
| **gp3** | General Purpose SSD | Yes | Same as gp2 — newer, more flexible |
| **io1** | Provisioned IOPS SSD | Yes | Databases, mission-critical low-latency |
| **io2 Block Express** | Provisioned IOPS SSD | Yes | Highest-performance databases |
| **st1** | Throughput Optimized HDD | No | Big data, data warehousing, log processing |
| **sc1** | Cold HDD | No | Infrequently accessed archive data, lowest cost |

> **Only SSD types (gp2, gp3, io1, io2) can be boot volumes.** HDD types (st1, sc1) cannot.

---

## General Purpose SSD — gp2 vs gp3

| Feature | gp2 | gp3 |
|---------|-----|-----|
| Size range | 1 GiB – 16 TiB | 1 GiB – 16 TiB |
| Baseline IOPS | 3,000 (burst for small volumes) | 3,000 |
| Max IOPS | 16,000 | 16,000 |
| Max throughput | 250 MiB/s | 1,000 MiB/s |
| IOPS and size relationship | **Linked** — 3 IOPS per GiB | **Independent** — set IOPS and throughput separately |
| Max IOPS reached at | 5,334 GiB (5,334 × 3 = 16,000) | Any size — just set the IOPS |

**Key difference:** In gp2, IOPS scales with volume size (linked). In gp3, IOPS and throughput are set **independently**.

---

## Provisioned IOPS SSD — io1 vs io2 Block Express

| Feature | io1 | io2 Block Express |
|---------|-----|-------------------|
| Size range | 4 GiB – 16 TiB | 4 GiB – 64 TiB |
| Max IOPS | 64,000 (Nitro) / 32,000 (other) | 256,000 |
| IOPS:GiB ratio | 50:1 | 1,000:1 |
| Latency | Low | **Sub-millisecond** |
| EBS Multi-Attach | Yes | Yes |

**When to use:** Database workloads sensitive to storage performance and consistency, or when you need **> 16,000 IOPS** (beyond gp2/gp3 max).

> To get **> 32,000 IOPS**, you need an **EC2 Nitro** instance with io1 or io2.

---

## HDD Volumes — st1 vs sc1

| Feature | st1 (Throughput Optimized) | sc1 (Cold HDD) |
|---------|---------------------------|-----------------|
| Size range | 125 GiB – 16 TiB | 125 GiB – 16 TiB |
| Max throughput | 500 MiB/s | 250 MiB/s |
| Max IOPS | 500 | 250 |
| Use case | Big data, data warehousing, logs | Archive, infrequent access |
| Cost | Low | **Lowest** |
| Boot volume? | No | No |

---

## Decision Flow

```
Need a boot volume?
  Yes → gp2/gp3 (general) or io1/io2 (high-perf database)
  No  ↓

Need high IOPS (> 16,000)?
  Yes → io1/io2 (provisioned IOPS SSD)
  No  ↓

Need high throughput, large sequential reads?
  Yes → st1 (throughput optimized HDD)
  No  ↓

Need lowest cost, infrequent access?
  Yes → sc1 (cold HDD)
```

---

## Best Practices

✓ **Default to gp3** for general workloads — newer, more flexible than gp2, independent IOPS/throughput  
✓ **Use io1/io2 for databases** requiring consistent, high IOPS performance  
✓ **Use EC2 Nitro instances** if you need > 32,000 IOPS with io1/io2  
✓ **Never use st1/sc1 as boot volumes** — they do not support booting  
✓ **Use sc1 for archival** when cost is the primary concern and access is infrequent  

---

## SysOps Exam Focus

**Q1: "A database application requires sustained 50,000 IOPS. Which EBS volume type should you use?"**
- A) gp2
- B) gp3
- C) io1 or io2 with an EC2 Nitro instance
- D) st1
- **Answer: C** — gp2/gp3 max out at 16,000 IOPS; io1/io2 on Nitro supports up to 64,000–256,000 IOPS

**Q2: "What is the key difference between gp2 and gp3?"**
- A) gp3 supports larger volumes
- B) gp2 allows independent IOPS and throughput settings
- C) In gp2, IOPS is linked to volume size; in gp3, IOPS and throughput are set independently
- D) gp3 cannot be used as a boot volume
- **Answer: C** — gp2 ties IOPS to GiB (3 IOPS per GiB); gp3 decouples them

**Q3: "Which EBS volume types can be used as boot volumes?"**
- A) All six types
- B) Only gp2 and gp3
- C) gp2, gp3, io1, and io2 (SSD types only)
- D) Only io1 and io2
- **Answer: C** — Only SSD volume types support booting; HDD types (st1, sc1) do not

**Q4: "You need the lowest-cost storage for archive data that is rarely accessed. Which volume type?"**
- A) gp2
- B) st1
- C) sc1
- D) io2
- **Answer: C** — sc1 (Cold HDD) is designed for infrequent access at the lowest cost

**Q5: "At what gp2 volume size do you reach the maximum 16,000 IOPS?"**
- A) 1,000 GiB
- B) 5,334 GiB
- C) 16,000 GiB
- D) 16 TiB
- **Answer: B** — gp2 provides 3 IOPS per GiB; 5,334 × 3 = 16,002 ≈ 16,000 max

**Q6: "Which feature is unique to io1 and io2 volumes?"**
- A) Encryption support
- B) EBS Multi-Attach
- C) Boot volume support
- D) Snapshot capability
- **Answer: B** — Only provisioned IOPS volumes (io1/io2) support EBS Multi-Attach

---

## Quick Reference

```
SSD (boot-capable):
  gp2/gp3  →  General purpose, up to 16,000 IOPS
               gp2: IOPS linked to size (3 IOPS/GiB)
               gp3: IOPS and throughput independent
  io1/io2  →  Provisioned IOPS, up to 64K–256K IOPS
               Requires Nitro for > 32K IOPS
               Supports EBS Multi-Attach

HDD (NOT boot-capable):
  st1      →  Throughput optimized, 500 MiB/s, big data/logs
  sc1      →  Cold HDD, 250 MiB/s, archive, lowest cost
```

---

**File: 3_EBS_Volume_Types.md**
**Status: SysOps-focused, exam-ready, concise format**
