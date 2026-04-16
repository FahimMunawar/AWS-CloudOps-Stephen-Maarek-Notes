# 5. EBS Snapshots — Hands-On

## Overview

A practical walkthrough of creating snapshots, restoring to different AZs, copying across regions, archiving, using DLM for automated backups, and setting up the Recycle Bin for accidental delete protection.

---

## Creating a Snapshot

1. EC2 → Volumes → select a volume → **Actions → Create snapshot**
2. Snapshot inherits the encryption state of the source volume (unencrypted volume → unencrypted snapshot)
3. Navigate to EC2 → **Snapshots** to see the snapshot being created

---

## Restoring a Snapshot to a Different AZ

1. Select snapshot → **Actions → Create volume from snapshot**
2. Change the **Availability Zone** to a different AZ (e.g., `eu-west-1a` → `eu-west-1b`)
3. Optionally enable **encryption** with a KMS key (even if the source was unencrypted)
4. Click **Create volume**

```
Snapshot (from eu-west-1a volume)
        ↓
Create volume → select eu-west-1b
        ↓
New volume in eu-west-1b (migrated)
```

> This is the standard method for moving EBS data across AZs.

---

## Copying a Snapshot Across Regions

1. Select snapshot → **Actions → Copy snapshot**
2. Choose a **destination region** (e.g., `us-east-1` for disaster recovery)
3. Optionally enable **encryption** with a KMS key in the target region
4. Any volumes created from the copied snapshot will also be encrypted

---

## Fast Snapshot Restore (FSR)

**Console path:** Select snapshot → **Actions → Manage fast snapshot restore**

- Enable per snapshot, per AZ
- **Cost warning:** $0.75/hour per AZ — extremely expensive
- Only use for large production volumes where first-read latency is unacceptable
- **Workflow:** Enable FSR → wait for initialization → restore volume → disable FSR immediately

> Do not enable FSR on small volumes or for learning purposes — costs add up quickly.

---

## Snapshot Archive

**Console path:** Select snapshot → **Storage tier** tab → **Archive snapshot**

| Tier | Cost | Restore Time |
|------|------|-------------|
| Standard | Normal pricing | Immediate |
| Archive | **75% cheaper** | 24–72 hours |

1. Select snapshot → **Actions → Archive snapshot**
2. Snapshot moves to archive tier (takes a moment to transition)
3. To restore: must wait 24–72 hours before the snapshot is usable again

---

## Data Lifecycle Manager (DLM)

**Console path:** EC2 → **Lifecycle Manager** → **Create lifecycle policy**

### Policy Configuration

| Setting | Example Value |
|---------|--------------|
| Target resource type | Volumes or Instances |
| Target tags | `Environment = Prod` |
| IAM role | Default (auto-created if needed) |
| Policy state | Enabled or Disabled |

### Schedule Configuration

| Setting | Example Value |
|---------|--------------|
| Frequency | Daily, every 12 hours |
| Start time | 09:00 UTC |
| Retention count | Keep last 10 snapshots |

### Advanced Options

| Feature | Detail |
|---------|--------|
| Fast Snapshot Restore | Can enable on DLM-created snapshots (very expensive — not recommended) |
| Cross-region copy | Copy snapshots to up to **3 additional regions** for disaster recovery |
| Cross-account sharing | Share snapshots with other AWS accounts |
| Tagging | Auto-tag snapshots created by the policy |

> **Important:** Tag your volumes/instances with the matching tags (e.g., `Environment = Prod`) or DLM will not target them.

---

## Recycle Bin

**Console path:** EC2 → **Recycle Bin** → **Create retention rule**

### Creating a Retention Rule

| Setting | Options |
|---------|---------|
| Resource type | EBS Snapshots or AMIs |
| Scope | All resources or filtered by resource tags |
| Retention period | **1 day to 1 year** |
| Rule lock | Unlocked (modifiable) or Locked (immutable for a set period) |

### Rule Lock Settings

| Mode | Behavior |
|------|----------|
| **Unlocked** | Rule can be modified or deleted by anyone at any time |
| **Locked** | Rule cannot be changed for a specified lock period — protects against intentional or accidental rule removal |

### How It Works

```
Delete a snapshot
        ↓
Snapshot moves to Recycle Bin (not permanently deleted)
        ↓
Within retention period → Actions → Recover snapshot
        ↓
Snapshot reappears in the Snapshots console
```

After the retention period expires, the snapshot is permanently deleted.

---

## Best Practices

✓ **Use DLM with resource tags** to automate snapshot schedules — don't rely on manual snapshots  
✓ **Enable cross-region copy in DLM** for disaster recovery — up to 3 regions  
✓ **Set up Recycle Bin retention rules** before you need them — protection against accidental deletes  
✓ **Lock retention rules** in production to prevent tampering  
✓ **Archive old snapshots** you rarely restore — 75% cost savings  
✓ **Encrypt snapshots when copying** — even if the source was unencrypted, the copy can be encrypted  

---

## SysOps Exam Focus

**Q1: "You have an unencrypted EBS snapshot. Can you create an encrypted volume from it?"**
- A) No — you must encrypt the source volume first
- B) Yes — enable encryption when creating the volume from the snapshot
- C) Yes — but only with the default AWS-managed KMS key
- D) No — snapshots and volumes must share the same encryption state
- **Answer: B** — You can enable encryption at restore time, even from an unencrypted snapshot

**Q2: "How do you automate daily snapshots of all production EBS volumes?"**
- A) Use CloudWatch Events to trigger a Lambda function
- B) Use Data Lifecycle Manager with a tag-based policy targeting `Environment = Prod`
- C) Use AWS Backup only
- D) Create a cron job on each EC2 instance
- **Answer: B** — DLM uses resource tags and schedules to automate snapshot creation and retention

**Q3: "You accidentally deleted an EBS snapshot. How can you recover it?"**
- A) Contact AWS Support
- B) Restore from the Recycle Bin if a retention rule was configured before deletion
- C) Use CloudTrail to undo the delete
- D) Snapshots cannot be recovered once deleted
- **Answer: B** — Recycle Bin holds deleted snapshots for the configured retention period

**Q4: "What does locking a Recycle Bin retention rule do?"**
- A) Encrypts the snapshots in the bin
- B) Prevents the rule from being modified or deleted for a specified lock period
- C) Extends the retention period automatically
- D) Prevents any snapshots from being deleted
- **Answer: B** — Locking makes the rule immutable for a set duration, protecting against tampering

**Q5: "You need to copy an EBS snapshot to another region for disaster recovery. What feature of DLM handles this automatically?"**
- A) Fast Snapshot Restore
- B) Cross-region copy (up to 3 additional regions)
- C) Snapshot Archive
- D) Cross-account sharing
- **Answer: B** — DLM supports automated cross-region copy to up to 3 regions

---

## Quick Reference

```
Create snapshot:     Volumes → Actions → Create snapshot
Restore to new AZ:   Snapshots → Actions → Create volume → change AZ
Copy cross-region:   Snapshots → Actions → Copy snapshot → select region
Archive:             Snapshots → Archive snapshot (75% cheaper, 24–72h restore)
FSR:                 Snapshots → Manage fast snapshot restore ($0.75/hr/AZ)

DLM:
  Target by tags    →  schedule snapshots automatically
  Cross-region      →  up to 3 regions
  Retention         →  keep last N snapshots

Recycle Bin:
  Create rule       →  1 day to 1 year retention
  Lock rule         →  prevent modification for safety
  Recover           →  Recycle Bin → Resources → Recover
```

---

**File: 5_EBS_Snapshots_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
