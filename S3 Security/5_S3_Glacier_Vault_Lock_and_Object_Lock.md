# 5 — S3 Glacier Vault Lock & S3 Object Lock

## Overview

Two mechanisms to enforce **WORM (Write Once Read Many)** protection — preventing objects from being modified or deleted. Used for compliance and data retention.

---

## WORM Model

> **Write Once Read Many** — once written, data cannot be modified or deleted for a defined period (or ever). Used for legal, regulatory, and compliance requirements.

---

## Part 1 — S3 Glacier Vault Lock

| Feature | Detail |
|---|---|
| **Applies to** | Entire S3 Glacier Vault |
| **How** | Create a Vault Lock Policy → lock the policy itself |
| **Once locked** | Policy cannot be changed or deleted — by anyone (including AWS) |
| **Objects** | Cannot be deleted once vault is locked |
| **Use case** | Compliance, legal data retention |

```
Create Vault Lock Policy
         │
         ▼
Lock the policy (irreversible)
         │
         ▼
Objects in vault → permanently protected, cannot be deleted by anyone
```

Simple and absolute — no exceptions, no overrides.

---

## Part 2 — S3 Object Lock

More granular than Glacier Vault Lock — applied **per object version**, not per bucket.

**Prerequisite:** Versioning must be enabled on the bucket.

### Two Retention Modes

#### Compliance Mode (Strictest)
| Rule | Detail |
|---|---|
| **Who can override** | **No one** — not even the root user |
| **Retention mode** | Cannot be changed |
| **Retention period** | Cannot be shortened (can only be extended) |
| **Object versions** | Cannot be overwritten or deleted |
| **Use case** | Maximum compliance — absolute protection |

#### Governance Mode (Flexible)
| Rule | Detail |
|---|---|
| **Most users** | Cannot override or delete |
| **Admin users** | Can change retention settings or delete (via IAM permission) |
| **Use case** | Protection with admin override capability |
| **IAM permission** | `s3:BypassGovernanceRetention` grants override access |

### Comparison Table

| Feature | Compliance Mode | Governance Mode |
|---|---|---|
| Root user can delete | **No** | No (unless has bypass permission) |
| Admins can override | **No** | **Yes** (with IAM permission) |
| Retention period changeable | Extend only | Yes (with admin permission) |
| Strictness | Maximum | Moderate |

### Retention Period
- Set when applying the lock to an object
- Applies to both compliance and governance modes
- Can be **extended** but not shortened in compliance mode

---

## Legal Hold

A special, independent protection separate from retention modes.

| Feature | Detail |
|---|---|
| **Duration** | **Indefinite** — no expiry date |
| **Independence** | Overrides retention period — even if retention period has expired, legal hold keeps the object protected |
| **Use case** | Legal investigations — object needed as evidence |
| **IAM permission to apply/remove** | `s3:PutObjectLegalHold` |

```
Object with Legal Hold
       │
       ▼
Protected indefinitely — regardless of retention mode or period
       │
  Legal investigation ends
       │
       ▼
Admin with s3:PutObjectLegalHold removes the hold → object no longer protected
```

---

## Summary Comparison

| Feature | Glacier Vault Lock | Object Lock (Compliance) | Object Lock (Governance) | Legal Hold |
|---|---|---|---|---|
| Scope | Entire vault | Per object version | Per object version | Per object version |
| Override possible | No | No | Yes (admin) | No (until removed) |
| Retention period | Permanent | Fixed, extendable | Fixed, changeable | Indefinite |
| IAM bypass | No | No | `s3:BypassGovernanceRetention` | `s3:PutObjectLegalHold` |

---

## SysOps Exam Q&A

**Q: What is the WORM model in S3?**
A: **Write Once Read Many** — objects can be written once and then cannot be modified or deleted for a specified period.

**Q: What is the difference between S3 Object Lock Compliance mode and Governance mode?**
A: In **Compliance mode**, no one (not even root) can override or delete — the most strict option. In **Governance mode**, most users are restricted but **admins with IAM permissions** can override.

**Q: An object in Compliance mode has a retention period of 1 year. Can an admin shorten it to 6 months?**
A: **No** — in Compliance mode, the retention period **cannot be shortened**. It can only be extended.

**Q: What is the purpose of a Legal Hold in S3 Object Lock?**
A: Protects an object **indefinitely** regardless of its retention mode or period — used during legal investigations. Removed by a user with `s3:PutObjectLegalHold` permission.

**Q: What is the difference between Glacier Vault Lock and S3 Object Lock?**
A: **Glacier Vault Lock** applies to the entire vault via a locked policy. **S3 Object Lock** applies per object version in a versioned S3 bucket, with two retention modes (Compliance/Governance) and a Legal Hold option.

**Q: What must be enabled before you can use S3 Object Lock?**
A: **Versioning** must be enabled on the S3 bucket.

---

## Quick Reference

```
WORM = Write Once Read Many (no modifications or deletions)

Glacier Vault Lock:
  - Lock entire vault via a policy
  - Once locked: no one can change or delete anything — ever

S3 Object Lock (per object version, versioning required):
  Compliance mode  = strictest; no one can override, even root
  Governance mode  = admins with s3:BypassGovernanceRetention can override
  Retention period = set at lock time; extendable (compliance: cannot shorten)
  Legal Hold       = indefinite protection; independent of retention
                     applied/removed via s3:PutObjectLegalHold
```
