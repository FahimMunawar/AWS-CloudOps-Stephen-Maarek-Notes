# 7. EBS vs EFS vs Instance Store

## Overview

A comparison of the three EC2 storage options — EBS (block storage), EFS (shared network file system), and Instance Store (ephemeral local storage). Knowing when to use each is a core exam skill.

---

## Side-by-Side Comparison

| Feature | EBS | EFS | Instance Store |
|---------|-----|-----|----------------|
| **Type** | Network block storage | Network file system (NFS) | Physical local disk |
| **Attach to multiple instances** | No (except io1/io2 Multi-Attach) | **Yes — hundreds of instances** | No — tied to one instance |
| **AZ scope** | Locked to one AZ | **Multi-AZ** | Locked to the physical host |
| **Data persistence** | Persists after instance stop/terminate (configurable) | Persists independently | **Lost on stop/terminate** |
| **OS support** | Linux and Windows | **Linux only** (POSIX) | Linux and Windows |
| **Capacity** | Provisioned (pay for provisioned size) | **Auto-scales, pay-per-use** | Fixed (based on instance type) |
| **Migration across AZs** | Snapshot → restore in new AZ | Built-in multi-AZ | Not possible |
| **Cost** | ~$0.10/GB (gp2) | ~$0.30/GB (~3x gp2) | Included in instance price |
| **Cost savings options** | Choose cheaper volume types | Storage tiers (IA, Archive — up to 90% savings) | None |

---

## EBS Key Points

- Attached to **one instance at a time** (Multi-Attach for io1/io2 is an edge case)
- **AZ-locked** — must snapshot to migrate across AZs
- gp2: IOPS scales with disk size | gp3/io1: IOPS set independently
- **Snapshots use IO** — avoid running backups during high-traffic periods
- Root EBS volume **deleted by default** on instance termination (can be disabled)

```
EC2 (AZ-1) ←──► EBS Volume (AZ-1)

Migration:
  EBS Volume → Snapshot → Restore in AZ-2 → New EBS Volume (AZ-2)
```

---

## EFS Key Points

- **Shared across hundreds of instances** in multiple AZs via mount targets
- **Linux only** — uses POSIX file system standard
- Higher price than EBS, but **storage tiers** (Standard, IA, Archive) reduce costs significantly
- Great for: WordPress, shared content, data sharing between instances

```
EC2 (AZ-1) ──┐
EC2 (AZ-1) ──┤
EC2 (AZ-2) ──┼──► EFS File System
EC2 (AZ-2) ──┤
EC2 (AZ-3) ──┘
```

---

## Instance Store Key Points

- **Physically attached** to the EC2 host — highest possible I/O performance
- **Ephemeral** — all data lost if the instance is stopped, terminated, or the underlying hardware fails
- Use for: temporary caches, buffers, scratch data — never for persistent storage

---

## Decision Flow

```
Need shared access across multiple instances/AZs?
  Yes → EFS
  No  ↓

Need persistent block storage for one instance?
  Yes → EBS
  No  ↓

Need maximum I/O and can tolerate data loss?
  Yes → Instance Store
```

---

## SysOps Exam Focus

**Q1: "You need a file system shared by EC2 instances across three Availability Zones. Which storage option should you use?"**
- A) EBS with Multi-Attach
- B) EFS
- C) Instance Store
- D) S3
- **Answer: B** — EFS supports multi-AZ mounts across hundreds of instances; EBS Multi-Attach is limited to io1/io2 within a single AZ

**Q2: "Your EBS-backed application is experiencing degraded performance during nightly backups. What is the cause?"**
- A) The EBS volume is too small
- B) EBS snapshots consume IO, impacting application performance
- C) The security group is blocking backup traffic
- D) The instance type does not support snapshots
- **Answer: B** — Snapshot operations use IO on the volume, so backups during high traffic can degrade performance

**Q3: "An EC2 instance is terminated. The root EBS volume is gone but an Instance Store volume is also gone. Why?"**
- A) Both are deleted by default — this is expected behavior
- B) The root EBS had Delete on Termination enabled; Instance Store data is always lost on termination
- C) The instance was in a placement group
- D) The volumes were not encrypted
- **Answer: B** — Root EBS deletes by default (configurable); Instance Store is always ephemeral

**Q4: "Which storage option requires no capacity provisioning and charges only for data stored?"**
- A) EBS gp3
- B) Instance Store
- C) EFS
- D) EBS io2
- **Answer: C** — EFS auto-scales and bills per GiB used; EBS requires provisioned capacity

**Q5: "A WordPress site needs shared storage accessible from Linux instances in multiple AZs. EFS is chosen. Is Windows supported?"**
- A) Yes — EFS supports all operating systems
- B) No — EFS only supports Linux-based AMIs (POSIX)
- C) Yes — but only with SMB protocol enabled
- D) No — EFS only supports Amazon Linux
- **Answer: B** — EFS uses the POSIX file system and is Linux only

---

## Quick Reference

```
EBS:
  One volume → one instance (AZ-locked)
  Persistent, provisioned capacity
  Migrate via snapshots
  Backups use IO — avoid during peak traffic

EFS:
  One FS → many instances (multi-AZ)
  Linux only, pay-per-use, auto-scales
  Storage tiers for up to 90% savings

Instance Store:
  Physical local disk — highest I/O
  Ephemeral — data lost on stop/terminate
  Use for caches/buffers only
```

---

**File: 7_EBS_vs_EFS_vs_Instance_Store.md**
**Status: SysOps-focused, exam-ready, concise format**
