# 1 — S3 MFA Delete

## Overview

MFA Delete adds a mandatory multi-factor authentication check before allowing **destructive S3 versioning operations**. It is an extra protection layer against accidental or malicious permanent deletions.

---

## What Requires MFA

| Operation | MFA Required? |
|---|---|
| **Permanently delete an object version** | **Yes** |
| **Suspend Versioning on a bucket** | **Yes** |
| Enable Versioning | No |
| List deleted versions | No |

> Only destructive, hard-to-reverse operations require MFA.

---

## How It Works

When MFA Delete is enabled, any API call for a protected operation must include:
- A valid **MFA device code** (from a mobile app like Google Authenticator or a hardware MFA device)
- Without the code → the operation is rejected by S3

---

## Prerequisites & Restrictions

| Requirement | Detail |
|---|---|
| **Versioning** | Must be enabled on the bucket (MFA Delete is a versioning feature) |
| **Who can enable/disable MFA Delete** | **Only the bucket owner using the root account** |
| **IAM users** | Cannot enable or disable MFA Delete — even admins |

> Using the root account is generally discouraged, but MFA Delete is one of the few legitimate use cases for it in S3.

---

## Key Facts for the Exam

- MFA Delete ≠ MFA login — it's a per-operation check on specific S3 API calls
- Protects against both **accidental** and **malicious** permanent deletions
- Can only be configured via the **AWS CLI or SDK** (not the S3 console)
- Requires the root account to enable — not even IAM admins can do it

---

## SysOps Exam Q&A

**Q: What is the purpose of MFA Delete in S3?**
A: To require a valid MFA code before allowing **permanent deletion of object versions** or **suspension of versioning** — protecting against accidental or malicious data loss.

**Q: Which account can enable or disable MFA Delete on an S3 bucket?**
A: Only the **root account** (bucket owner). IAM users cannot enable or disable MFA Delete.

**Q: Which S3 operations require MFA when MFA Delete is enabled?**
A: **Permanently deleting a specific object version** and **suspending versioning** on the bucket.

**Q: Is MFA required to enable versioning or list deleted versions when MFA Delete is on?**
A: **No** — these are non-destructive operations and do not require MFA.

**Q: What must be enabled before you can use MFA Delete?**
A: **S3 Versioning** must be enabled on the bucket.

---

## Quick Reference

```
MFA Delete = extra protection for destructive versioning operations

Requires MFA:
  - Permanently delete a specific object version
  - Suspend versioning

Does NOT require MFA:
  - Enable versioning
  - List deleted versions

Prerequisites:
  - Versioning must be ON
  - Only root account can enable/disable MFA Delete
  - Configured via CLI/SDK only (not S3 console)
```
