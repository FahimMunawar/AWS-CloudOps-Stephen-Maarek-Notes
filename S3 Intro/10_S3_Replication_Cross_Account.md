# 10 — S3 Replication: Cross-Account Settings

## Overview

When replicating S3 objects across AWS accounts, extra IAM and bucket policy configuration is required. A key exam concept is **object ownership** — by default, the source account retains ownership of replicated objects even in the destination account's bucket.

---

## Cross-Account Replication Setup

```
Account A (Source)                    Account B (Destination)
──────────────────                    ──────────────────────
Source Bucket                         Destination Bucket
IAM Role (replication role)  ──────►  Bucket Policy (must allow Account A's IAM role)
```

### Required Configuration

#### 1 — IAM Role in Account A (Source)
The replication IAM role needs permissions to:
- Read objects from the source bucket
- Replicate objects to the destination bucket (cross-account)

#### 2 — Destination Bucket Policy in Account B
Must explicitly allow the Account A IAM role to:
- Replicate objects (`s3:ReplicateObject`)
- Replicate deletes (`s3:ReplicateDelete`)
- Get/put bucket versioning (`s3:GetBucketVersioning`, `s3:PutBucketVersioning`)

```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::ACCOUNT-A-ID:role/ReplicationRole"
  },
  "Action": [
    "s3:ReplicateObject",
    "s3:ReplicateDelete",
    "s3:GetBucketVersioning",
    "s3:PutBucketVersioning"
  ],
  "Resource": [
    "arn:aws:s3:::destination-bucket",
    "arn:aws:s3:::destination-bucket/*"
  ]
}
```

---

## Object Ownership: Default vs Owner Override

This is the key exam concept in this lecture.

### Default Behaviour (No Owner Override)
```
Account A uploads object → replicates to Account B's bucket
Object owner in Account B's bucket = Account A  ← source account retains ownership
```

- Account B **cannot** read the object by default (they don't own it)
- This can cause access issues in the destination account

### With Owner Override Enabled
```
Account A uploads object → replicates to Account B's bucket
Object owner in Account B's bucket = Account B  ← destination account owns the object
```

- Account B has full ownership and can read/manage the object
- Required when Account B needs to access or share the replicated objects

---

## Enabling Owner Override

### Step 1 — Add to Replication Rule
Enable the **"Owner override"** (Replica Ownership Override) option in the replication rule configuration.

### Step 2 — Add IAM Permission to Source Role (Account A)
The replication IAM role in Account A must have:
```json
{
  "Effect": "Allow",
  "Action": "s3:ObjectOwnerOverrideToBucketOwner",
  "Resource": "arn:aws:s3:::destination-bucket/*"
}
```

### Step 3 — Add to Destination Bucket Policy (Account B)
The destination bucket policy must also allow the source IAM role to apply the ownership override:
```json
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::ACCOUNT-A-ID:role/ReplicationRole"
  },
  "Action": "s3:ObjectOwnerOverrideToBucketOwner",
  "Resource": "arn:aws:s3:::destination-bucket/*"
}
```

---

## Decision Guide

| Scenario | Use Owner Override? |
|---|---|
| Cross-account backup — Account A keeps control | No |
| Account B needs to read/share replicated objects | **Yes** |
| Destination account should fully own the data | **Yes** |

---

## SysOps Exam Q&A

**Q: Objects are replicated from Account A to Account B's S3 bucket, but Account B cannot read them. What is the likely cause?**
A: The **owner override option is not enabled**. By default, Account A (source) retains object ownership even in Account B's bucket. Enable Replica Ownership Override and configure the required IAM permissions.

**Q: What is the default object ownership behaviour in cross-account S3 replication?**
A: The **source account (Account A) remains the owner** of replicated objects in the destination bucket, even though the bucket belongs to Account B.

**Q: What three things must be configured to enable owner override in cross-account S3 replication?**
A: 1) Enable the **Owner Override** option in the replication rule. 2) Add `s3:ObjectOwnerOverrideToBucketOwner` to the **source IAM role**. 3) Allow `s3:ObjectOwnerOverrideToBucketOwner` in the **destination bucket policy**.

**Q: What permissions must the destination bucket policy grant for cross-account replication to work?**
A: `s3:ReplicateObject`, `s3:ReplicateDelete`, `s3:GetBucketVersioning`, `s3:PutBucketVersioning` — granted to the source account's replication IAM role.

---

## Quick Reference

```
Cross-account replication requires:
  1. IAM Role in Account A (source) with replication permissions
  2. Destination Bucket Policy in Account B allowing Account A's IAM role

Default ownership = source account (Account A) owns objects in destination
Problem           = Account B cannot read replicated objects

Fix (Owner Override):
  1. Enable "Owner override" in replication rule
  2. Add s3:ObjectOwnerOverrideToBucketOwner to source IAM role
  3. Add s3:ObjectOwnerOverrideToBucketOwner to destination bucket policy
```
