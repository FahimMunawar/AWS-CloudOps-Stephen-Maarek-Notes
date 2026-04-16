# 4. EBS Snapshots

## Overview

EBS snapshots are point-in-time backups of EBS volumes. They are the mechanism for migrating volumes across AZs/regions, automating backups, and enabling disaster recovery.

---

## Snapshot Basics

- Take a snapshot of an EBS volume at **any point in time**
- **Detaching the volume first is recommended** (not required) — avoids consistency issues while the volume is in use
- Snapshots can be **copied across AZs and regions** — this is how you migrate EBS data

```
EBS Volume (us-east-1a)
        ↓
    Snapshot
        ↓
Restore in us-east-1b (or any other AZ/region)
        ↓
New EBS Volume (us-east-1b)
```

---

## Amazon Data Lifecycle Manager (DLM)

Automates the creation, retention, and deletion of EBS snapshots and EBS-backed AMIs.

| Feature | Detail |
|---------|--------|
| **Schedule backups** | Cron-based automated snapshot creation |
| **Cross-account copies** | Automatically copy snapshots to other AWS accounts |
| **Delete outdated backups** | Policy-based retention and cleanup |
| **Targeting** | Uses **resource tags** to identify volumes/instances to back up |

**Example:** Tag an EBS volume or EC2 instance with `Environment = Prod` → DLM automatically creates scheduled snapshots.

### DLM Limitations

- Cannot manage snapshots or AMIs created **outside of DLM**
- Cannot manage **instance-store backed AMIs** — only EBS-backed

---

## Fast Snapshot Restore (FSR)

### The Problem

Snapshots are stored internally in **Amazon S3**. When restoring a snapshot to an EBS volume, the first read of each block has **high latency** because the data is pulled from S3 on demand.

### Two Solutions

| Method | How It Works | Cost |
|--------|-------------|------|
| **Manual initialization** | Attach volume to EC2, read entire volume using `dd` or `fio` command | Free (just EC2 time) |
| **Fast Snapshot Restore (FSR)** | AWS pre-initializes the entire volume for you — no I/O latency on first read | **Very expensive** (~$500/month per snapshot per AZ) |

### FSR Usage Pattern

```
Take snapshot → Enable FSR → Wait for full initialization
        ↓
Restore to EBS volume (zero first-read latency)
        ↓
Disable FSR immediately to stop billing
```

- Billed **per minute** while enabled
- Can be enabled via DLM on snapshots automatically (but very costly if left on)
- Best for: production workloads where initial I/O latency is unacceptable

---

## EBS Snapshot Archive

Move snapshots to a cheaper storage tier.

| Feature | Detail |
|---------|--------|
| **Cost savings** | Archive tier is **75% cheaper** than standard |
| **Restore time** | **24 to 72 hours** to restore from archive |
| **Use case** | Snapshots you need to keep but rarely restore |

```
Standard snapshot → Move to Archive tier (75% cheaper)
                         ↓
              Need it back? 24–72 hours to restore
```

---

## Recycle Bin for EBS Snapshots

Protection against accidental snapshot deletion.

| Feature | Detail |
|---------|--------|
| **Default behavior** | Deleted snapshots are **gone permanently** |
| **With Recycle Bin** | Deleted snapshots go to a bin instead |
| **Retention period** | Configurable: **1 day to 1 year** |
| **Recovery** | Restore from bin at any time within the retention window |

```
Delete snapshot → Recycle Bin (retained for configured period)
                       ↓
              Recover if needed, or auto-deleted after retention expires
```

---

## Best Practices

✓ **Detach volumes before snapshotting** when possible — ensures data consistency  
✓ **Use DLM to automate backups** — tag resources and let policies handle creation/retention/deletion  
✓ **Use Snapshot Archive for long-term retention** — 75% cost savings on rarely-needed snapshots  
✓ **Enable Recycle Bin** — protects against accidental deletion with configurable retention  
✓ **Disable FSR immediately after use** — leaving it on is extremely expensive  
✓ **Use snapshots to migrate across AZs** — the only way to move EBS data between AZs  

---

## SysOps Exam Focus

**Q1: "You need to move an EBS volume from us-east-1a to us-east-1b. What should you do?"**
- A) Detach the volume and re-attach it in us-east-1b
- B) Create a snapshot, then restore the snapshot as a new volume in us-east-1b
- C) Use EBS replication to copy the volume
- D) Create an AMI from the volume
- **Answer: B** — Snapshots are the mechanism for moving EBS data across AZs

**Q2: "What AWS service automates the creation, retention, and deletion of EBS snapshots?"**
- A) AWS Backup
- B) Amazon Data Lifecycle Manager (DLM)
- C) AWS Systems Manager
- D) Amazon S3 Lifecycle Policies
- **Answer: B** — DLM uses resource tags and policies to manage EBS snapshots and EBS-backed AMIs

**Q3: "After restoring a snapshot to a new EBS volume, the first reads are slow. Why?"**
- A) The volume is not encrypted
- B) The volume type is sc1
- C) Snapshot data is stored in S3 and blocks are pulled on first access
- D) The volume is in a different AZ than the snapshot
- **Answer: C** — Snapshot data resides in S3; first-time block reads have latency until pulled

**Q4: "How can you eliminate first-read latency on a restored EBS snapshot without reading the entire volume manually?"**
- A) Use gp3 instead of gp2
- B) Enable Fast Snapshot Restore (FSR) on the snapshot
- C) Use EBS Multi-Attach
- D) Encrypt the snapshot before restoring
- **Answer: B** — FSR pre-initializes all blocks so there is no first-read latency

**Q5: "You want to keep old snapshots at the lowest cost but can tolerate 24–72 hours to restore. What should you use?"**
- A) Move snapshots to S3 Glacier
- B) Use EBS Snapshot Archive tier
- C) Enable FSR on old snapshots
- D) Use DLM to compress snapshots
- **Answer: B** — Archive tier is 75% cheaper with a 24–72 hour restore time

**Q6: "How can you protect against accidental deletion of EBS snapshots?"**
- A) Enable MFA Delete on the snapshot
- B) Set up a Recycle Bin with a retention period
- C) Use DeletionPolicy: Retain in CloudFormation
- D) Enable versioning on the snapshot
- **Answer: B** — Recycle Bin holds deleted snapshots for a configurable period (1 day to 1 year)

---

## Quick Reference

```
Snapshots:
  Take at any time     →  detach volume first for consistency (recommended)
  Cross-AZ/region      →  copy snapshot, restore in target AZ/region

DLM:
  Automates snapshots  →  uses resource tags to identify targets
  Limitation           →  cannot manage snapshots created outside DLM

FSR:
  Eliminates first-read latency  →  ~$500/month per snapshot per AZ
  Best practice                  →  enable, restore, disable immediately

Archive:
  75% cheaper          →  24–72 hours to restore

Recycle Bin:
  Catches deleted snapshots  →  retention: 1 day to 1 year
```

---

**File: 4_EBS_Snapshots.md**
**Status: SysOps-focused, exam-ready, concise format**
