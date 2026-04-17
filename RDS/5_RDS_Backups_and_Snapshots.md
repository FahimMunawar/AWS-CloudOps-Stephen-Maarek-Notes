# 5. RDS Backups and Snapshots

## Overview

RDS provides two mechanisms for data protection: **automated backups** (continuous, point-in-time recovery) and **manual snapshots** (on-demand, never expire). These differ in retention, sharing, and restore behavior — all exam-tested.

---

## Automated Backups vs Manual Snapshots

| Property | Automated Backups | Manual Snapshots |
|----------|------------------|-----------------|
| **Type** | Continuous | On-demand |
| **Point-in-time recovery** | Yes — any second within retention window | No — restores to snapshot moment only |
| **Retention period** | 0–35 days (set by you) | **Never expire** |
| **Disable** | Set retention to **0** | Delete manually |
| **Taken on Multi AZ** | From **standby** (not master) | From standby if Multi AZ enabled |
| **IO impact** | Occurs during maintenance window | Stops IO for seconds to minutes |
| **Incremental** | Yes (after first full backup) | Yes (after first full snapshot) |
| **Retained after DB deletion** | Optional — choose to retain or delete | Always retained until manually deleted |
| **Shareable** | **No** — must copy first | **Yes** — share directly with other accounts |

---

## Restore Behavior

> **Restoring always creates a NEW database instance — no in-place restore.**

This applies to both automated backups and manual snapshots.

```
Restore from backup/snapshot
        ↓
New RDS DB instance is created
        ↓
Update your application connection string to point to new instance
```

---

## Snapshot Sharing

Works identically to EBS snapshot sharing.

| Scenario | Requirement |
|----------|-------------|
| Share **unencrypted** snapshot | Share directly — no extra steps |
| Share **encrypted** snapshot | Must also share the **CMK** used to encrypt it |
| Share **automated** snapshot | Must **copy it first** → then share the copy |
| Share **manual** snapshot | Share directly with target AWS account |

### Encrypted Snapshot Sharing Flow

```
Account A has encrypted snapshot
        ↓
Account A shares: snapshot + KMS CMK permission → Account B
        ↓
Account B uses KMS key to decrypt snapshot
        ↓
Account B launches new RDS instance from snapshot
```

---

## Backup Window

- Configured per database (e.g., 30-minute window at night)
- Automated backups run during this window
- On Multi AZ: backup is taken from the **standby** to avoid impacting the primary

---

## Console Locations

| Item | Console Location |
|------|-----------------|
| Automated backups | RDS → Automated backups |
| Manual snapshots | RDS → Snapshots → Manual |
| System snapshots | RDS → Snapshots → System |
| Copy/share snapshot | Select snapshot → Actions → Copy / Share |
| Backup settings | DB → Maintenance & backups tab |

---

## Best Practices

✓ **Set retention to 35 days for production** — maximizes point-in-time recovery window  
✓ **Take a final manual snapshot before deleting a database** — automated backups can be lost  
✓ **For Multi AZ, backups run from the standby** — no IO impact on the primary  
✓ **Copy before sharing automated snapshots** — automated backups cannot be shared directly  
✓ **Share the CMK when sharing encrypted snapshots** — recipient needs key access to decrypt  

---

## SysOps Exam Focus

**Q1: "You need to restore an RDS database to a specific point in time from 10 days ago. What must be configured?"**
- A) Manual snapshots taken every 10 days
- B) Automated backups with a retention period of at least 10 days
- C) A Read Replica in another AZ
- D) Cross-region backup replication
- **Answer: B** — Point-in-time recovery requires automated backups with a retention period that covers the target date

**Q2: "You want to share an RDS snapshot with another AWS account. The snapshot is encrypted with a customer-managed KMS key. What must you do?"**
- A) Share the snapshot only — encryption is handled automatically
- B) Decrypt the snapshot first, then share the unencrypted copy
- C) Share both the snapshot and the KMS CMK permissions with the target account
- D) Automated snapshots can be shared directly without any extra steps
- **Answer: C** — The recipient account must have access to the CMK to decrypt the snapshot when launching an instance

**Q3: "A developer wants to share an automated RDS snapshot with a partner account. What must happen first?"**
- A) Enable cross-account access in IAM
- B) Copy the automated snapshot to create a manual snapshot, then share the copy
- C) Automated snapshots can be shared directly from the console
- D) Convert the automated snapshot to an AMI
- **Answer: B** — Automated snapshots cannot be shared directly; they must be copied first, and the copy can then be shared

**Q4: "What happens when you restore an RDS snapshot?"**
- A) The existing database is overwritten in place
- B) The snapshot is mounted as a read-only volume on the existing instance
- C) A new RDS database instance is created from the snapshot
- D) The database is rolled back to the snapshot state with no new instance
- **Answer: C** — Restoring always creates a new instance; update the application's connection string afterward

**Q5: "You take a manual RDS snapshot and then delete the database. What happens to the snapshot?"**
- A) It is automatically deleted along with the database
- B) It is retained for 35 days then deleted
- C) It is retained indefinitely — manual snapshots do not expire
- D) It is moved to S3 Glacier automatically
- **Answer: C** — Manual snapshots never expire; they persist until you explicitly delete them

**Q6: "Your RDS database has Multi AZ enabled. When an automated backup runs, which instance is used?"**
- A) The primary (master) instance — backups always use the primary
- B) The standby instance — to avoid IO impact on the primary
- C) Both instances simultaneously for redundancy
- D) A temporary read replica created for backup purposes
- **Answer: B** — On Multi AZ, automated backups and snapshots are taken from the standby to avoid impacting production

---

## Quick Reference

```
Automated Backups:
  Continuous + point-in-time recovery
  Retention: 0–35 days (0 = disabled)
  Runs during maintenance window
  Multi AZ: taken from standby
  Cannot share directly → copy first

Manual Snapshots:
  On-demand, never expire
  IO impact: seconds to minutes
  Shareable with other accounts
  Can copy cross-region

Snapshot sharing:
  Unencrypted  → share directly
  Encrypted    → share snapshot + CMK
  Automated    → copy first, then share copy

Restore: always creates a NEW DB instance
```

---

**File: 5_RDS_Backups_and_Snapshots.md**
**Status: SysOps-focused, exam-ready, concise format**
