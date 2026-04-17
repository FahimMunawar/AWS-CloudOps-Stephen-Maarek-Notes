# 4 — AWS Storage Gateway

## Overview

AWS Storage Gateway is a **bridge between on-premises infrastructure and AWS cloud storage**. It enables hybrid cloud architectures by exposing AWS storage services (S3, EBS, Glacier) as standard protocols to on-premises applications.

---

## Why Storage Gateway?

| Use Case | Description |
|---|---|
| **Disaster recovery** | Back up on-premises data to the cloud |
| **Backup and restore** | Cloud-native backups with on-premises access |
| **Cloud migration** | Gradual migration of data to AWS |
| **Storage tiering** | Hot data on-premises, cold data in cloud |
| **Low-latency cache** | Store data in cloud; cache recently accessed data on-premises |

---

## AWS Storage Types (Context)

| Type | AWS Services |
|---|---|
| **Block** | EBS, EC2 Instance Store |
| **File** | EFS, FSx |
| **Object** | S3, S3 Glacier |

S3 is proprietary (not NFS-compatible) — Storage Gateway bridges this gap for on-premises access.

---

## Three Types of Storage Gateway

### 1 — S3 File Gateway

**Goal:** Expose S3 buckets to on-premises applications as a standard NFS or SMB file share.

```
Application Server (on-premises)
       │  NFS or SMB protocol
       ▼
  S3 File Gateway (VM in on-premises data center)
       │  HTTPS
       ▼
  Amazon S3 Bucket (Standard, Standard-IA, One Zone-IA, Intelligent-Tiering)
       │  lifecycle policy (optional)
       ▼
  S3 Glacier (archive)
```

| Feature | Detail |
|---|---|
| **Protocols** | NFS, SMB |
| **Backend** | Amazon S3 |
| **Supported classes** | Standard, Standard-IA, One Zone-IA, Intelligent-Tiering — **not Glacier directly** |
| **Glacier transition** | Via S3 lifecycle policy |
| **Local cache** | Most recently used data cached on the gateway for low-latency access |
| **IAM** | IAM role required per File Gateway |
| **SMB + AD** | Active Directory integration for user authentication (Windows environments) |

> Most recently used files are cached on the gateway — not the entire S3 bucket.

---

### 2 — Volume Gateway

**Goal:** Back up on-premises block storage volumes to the cloud via iSCSI.

```
Application Server (on-premises)
       │  iSCSI protocol
       ▼
  Volume Gateway (VM in on-premises data center)
       │
       ▼
  Amazon S3 (volume data) → EBS Snapshots (for restore/migration)
```

| Mode | How it works | Use case |
|---|---|---|
| **Cached Volumes** | Data stored in S3; frequently accessed data cached on-premises | Low-latency access to recent data, S3 as primary storage |
| **Stored Volumes** | Entire dataset kept on-premises; scheduled backup to S3 as EBS snapshots | Full on-premises access + cloud backup |

| Feature | Detail |
|---|---|
| **Protocol** | iSCSI (block storage) |
| **Backend** | Amazon S3 → EBS Snapshots |
| **Purpose** | Backup on-premises volumes; restore as EBS volumes on AWS |

---

### 3 — Tape Gateway

**Goal:** Replace physical tape backup with cloud-based virtual tapes — for companies using tape backup systems.

```
Backup Server (on-premises, tape-based)
       │  iSCSI VTL (Virtual Tape Library)
       ▼
  Tape Gateway (VM in on-premises data center)
       │
       ▼
  Amazon S3 (active virtual tapes)
       │  archive (lifecycle)
       ▼
  S3 Glacier / Glacier Deep Archive (archived tapes)
```

| Feature | Detail |
|---|---|
| **Protocol** | iSCSI VTL (Virtual Tape Library) |
| **Backend** | Amazon S3 (active) → Glacier / Deep Archive (archived) |
| **Compatible with** | Leading backup software vendors |
| **Use case** | Modernize tape backup without changing backup processes |

---

## Summary Diagram

```
On-Premises                    Storage Gateway              AWS Cloud
─────────────                  ───────────────              ─────────
File share users  ─NFS/SMB──►  S3 File Gateway   ─HTTPS──► S3
App servers       ─iSCSI───►   Volume Gateway    ────────► S3 → EBS Snapshots
Backup server     ─iSCSI───►   Tape Gateway      ────────► S3 → Glacier
```

---

## Gateway Deployment

- Storage Gateway runs as a **VM** deployed in your **corporate data center**
- If no virtual servers are available on-premises: AWS provides a **hardware appliance** option

---

## Storage Gateway Type Decision Guide

| Scenario | Gateway Type |
|---|---|
| On-premises apps need NFS/SMB access to S3 | **S3 File Gateway** |
| Windows environment needing SMB + AD auth | **S3 File Gateway** (with AD integration) |
| Back up on-premises volumes to cloud | **Volume Gateway** |
| On-premises volumes with low-latency access + cloud backup | **Volume Gateway (Cached)** |
| Keep all data on-premises + scheduled S3 backup | **Volume Gateway (Stored)** |
| Replace physical tape backup with cloud tapes | **Tape Gateway** |

---

## SysOps Exam Q&A

**Q: What is AWS Storage Gateway used for?**
A: To bridge **on-premises infrastructure** with **AWS cloud storage**, enabling hybrid cloud architectures using standard protocols (NFS, SMB, iSCSI).

**Q: An on-premises application needs to access S3 using NFS. What should you use?**
A: **S3 File Gateway** — translates NFS/SMB requests into S3 HTTPS API calls.

**Q: What is the difference between cached volumes and stored volumes in Volume Gateway?**
A: **Cached volumes**: data stored in S3, frequently accessed data cached on-premises. **Stored volumes**: all data on-premises, with scheduled S3 backup as EBS snapshots.

**Q: A company wants to keep its tape backup process but store tapes in the cloud. What service should they use?**
A: **Tape Gateway** — creates a virtual tape library (VTL) backed by S3 and Glacier, compatible with existing backup software.

**Q: Can S3 File Gateway store data directly in S3 Glacier?**
A: **No** — S3 File Gateway supports Standard, Standard-IA, One Zone-IA, and Intelligent-Tiering. To archive to Glacier, use an S3 **lifecycle policy**.

**Q: What protocol does the Volume Gateway use?**
A: **iSCSI** — block storage protocol.

---

## Quick Reference

```
Storage Gateway = bridge between on-premises and AWS storage

S3 File Gateway:
  Protocol: NFS or SMB
  Backend:  S3 (not Glacier directly — use lifecycle policy)
  Cache:    most recently used files cached on gateway
  Auth:     AD integration for SMB

Volume Gateway:
  Protocol: iSCSI
  Cached:   data in S3, recent data cached on-prem
  Stored:   all data on-prem, scheduled S3 backup → EBS snapshots

Tape Gateway:
  Protocol: iSCSI VTL
  Backend:  S3 → Glacier / Deep Archive
  Use for:  replace physical tape backups

All gateways run as VMs in on-premises data center
```
