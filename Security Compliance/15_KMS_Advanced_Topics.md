# 15 — KMS Advanced Topics

## 1. Changing EBS Volume Encryption Key

You cannot change the KMS key on an existing EBS volume directly. Workaround:

```
Encrypted EBS Volume (CMK-A)
  → Create EBS Snapshot (encrypted with CMK-A)
    → Create New Volume from snapshot → specify CMK-B
      → New EBS Volume (encrypted with CMK-B)
```

The snapshot re-encryption step decrypts with CMK-A and re-encrypts with CMK-B during volume creation.

---

## 2. Sharing KMS-Encrypted Snapshots Across Accounts

To share an RDS or EBS snapshot encrypted with a customer-managed CMK:

1. **Update the CMK key policy** to allow the target account to perform cryptographic operations (`kms:Decrypt`, `kms:ReEncrypt*`, `kms:GenerateDataKey`, `kms:DescribeKey`)
2. Share the snapshot with the target account
3. Target account creates a new resource (DB instance / EBS volume) from the shared snapshot

> AWS-managed keys cannot be shared across accounts — you must use a customer-managed CMK.

---

## 3. KMS Key Deletion

| State | Detail |
|---|---|
| **Pending deletion** | Waiting period: 7–30 days (you choose) |
| During waiting period | Key **cannot be used** for any cryptographic operations |
| Scheduled rotation | Will **not** occur during pending deletion |
| Cancel deletion | Possible at any time during the waiting period |

> Best practice: **disable** the key first instead of deleting, until you're certain it is no longer needed.

---

## 4. Detect Key Usage During Pending Deletion (Automation)

Build an alerting pipeline to catch if a "deleted" CMK is still being used:

```
CMK scheduled for deletion → Pending Deletion state
  ↓
User attempts cryptographic operation (Decrypt/Encrypt)
  ↓
API call DENIED → logged to CloudTrail
  ↓
CloudTrail → CloudWatch Logs
  ↓
CloudWatch Metric Filter: "KeyPendingDeletion" keyword
  ↓
CloudWatch Alarm → SNS → Email/SMS alert
```

Use case: confirms whether it is safe to complete the deletion, or reveals the key is still in active use.

---

## SysOps Exam Q&A

**Q: An EC2 instance uses an EBS volume encrypted with CMK-A. You need it encrypted with CMK-B. What do you do?**
A: Create a snapshot of the volume, then create a new EBS volume from that snapshot specifying CMK-B as the new key.

**Q: You want to share a KMS-encrypted RDS snapshot with another AWS account. What is required?**
A: Update the CMK key policy to grant the target account cryptographic permissions, then share the snapshot. The target account can then create a DB instance from it.

**Q: A CMK is scheduled for deletion. What happens if an application tries to use it?**
A: The cryptographic operation is denied. The key cannot be used during the pending deletion waiting period (7–30 days).

**Q: How do you detect if a CMK pending deletion is still being used?**
A: CloudTrail logs the denied API calls → CloudWatch Logs metric filter on "KeyPendingDeletion" → CloudWatch Alarm → SNS notification.

**Q: Why should you disable a CMK before deleting it?**
A: Disabling is reversible — if you discover the key is still needed, you can re-enable it. Deletion is permanent once the waiting period expires.

---

## Quick Reference

```
Change EBS encryption key:
  Snapshot → create new volume from snapshot with new CMK

Share encrypted snapshot cross-account:
  Update CMK key policy → grant target account kms:Decrypt etc.
  Share snapshot → target creates resource from it

KMS key deletion:
  Pending deletion: 7–30 days waiting period
  Key unusable during waiting period (all crypto ops denied)
  Best practice: disable first, delete only when certain

Detect usage during pending deletion:
  CloudTrail → CloudWatch Logs → Metric Filter (KeyPendingDeletion) → Alarm → SNS
```
