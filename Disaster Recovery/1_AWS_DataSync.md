# 1 — AWS DataSync

## Overview

Move and synchronize large amounts of data between on-premises/other clouds and AWS, or between AWS storage services.

---

## Supported Protocols (On-Premises / Other Cloud)

NFS, SMB, HDFS, and other protocols — requires a **DataSync agent** installed on-premises or in the source cloud.

---

## Destinations

| Destination | Notes |
|---|---|
| **Amazon S3** | All storage classes, including Glacier |
| **Amazon EFS** | Elastic File System |
| **Amazon FSx** | All variants supported |

---

## Key Characteristics

| Feature | Detail |
|---|---|
| **Replication** | Scheduled — NOT continuous (hourly, daily, or weekly) |
| **Direction** | Bidirectional — on-premises ↔ AWS, or AWS ↔ AWS |
| **Metadata preservation** | NFS POSIX + SMB permissions preserved |
| **Agent required** | Yes, for NFS/SMB servers; NO agent for AWS-to-AWS |
| **Throughput** | Up to **10 Gbps** per task |
| **Bandwidth limit** | Configurable to avoid saturating the network |

> **Exam key**: DataSync is the only option that **preserves file metadata and permissions** when migrating files.

---

## Architecture Diagrams

### On-Premises → AWS

```
On-premises NFS/SMB server
  → DataSync Agent (installed on-premises)
    → Encrypted connection → AWS DataSync service
      → S3 / EFS / FSx
```

Synchronization is bidirectional — AWS can also sync back to on-premises.

### No Network Capacity → Use Snowcone

```
Snowcone device (DataSync agent pre-installed)
  → Runs on-premises, pulls data
    → Ship device to AWS
      → Sync to S3 / EFS / FSx
```

> AWS Snowcone comes with the DataSync agent pre-installed — use it when network bandwidth is insufficient.

### AWS-to-AWS (No Agent)

```
Amazon S3 / EFS / FSx
  → AWS DataSync (no agent needed)
    → Amazon S3 / EFS / FSx
      (metadata preserved)
```

---

## SysOps Exam Q&A

**Q: Which service preserves file permissions and metadata when migrating files to AWS?**
A: **AWS DataSync** — preserves NFS POSIX and SMB metadata. This is its unique differentiator.

**Q: DataSync is configured but there is not enough network bandwidth. What is the solution?**
A: Use **AWS Snowcone** — it has the DataSync agent pre-installed; ship the device to AWS to complete the transfer.

**Q: Is DataSync replication continuous?**
A: **No** — DataSync runs on a schedule: hourly, daily, or weekly.

**Q: Do you need an agent for AWS-to-AWS DataSync transfers?**
A: **No** — agent is only required when connecting to on-premises NFS/SMB servers or other clouds.

---

## Quick Reference

```
DataSync: scheduled data sync (NOT continuous)
  Schedule: hourly / daily / weekly
  Max throughput: 10 Gbps per task (configurable limit)

Sources: on-premises NFS/SMB/HDFS, other cloud → requires agent
AWS-to-AWS: S3 ↔ EFS ↔ FSx → no agent required

Destinations: S3 (all classes incl. Glacier), EFS, FSx

Key differentiator: preserves metadata + file permissions (NFS POSIX + SMB)

No network? → AWS Snowcone (DataSync agent pre-installed) → ship to AWS
```
