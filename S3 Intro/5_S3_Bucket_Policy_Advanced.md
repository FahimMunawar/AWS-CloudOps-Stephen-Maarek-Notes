# 5 — S3 Bucket Policy — Advanced Examples

## Overview

Advanced bucket policy patterns using **conditions**. You don't need to memorise these for the exam, but you must understand their use cases and how conditions work.

---

## Condition Keys Available in S3 Bucket Policies

| Condition Key | Description |
|---|---|
| `aws:SourceIp` | Restrict by Public IP or Elastic IP (not Private IP) |
| `aws:SourceVpc` / `aws:SourceVpce` | Restrict by VPC or VPC Endpoint (requires VPC endpoints) |
| `aws:PrincipalOrgID` | Restrict to principals within an AWS Organization |
| `s3:x-amz-server-side-encryption` | Enforce encryption on upload |
| `aws:MultiFactorAuthPresent` | Require MFA for access |
| `aws:SourceArn` | Restrict to a specific CloudFront distribution or service |

> **Note:** IP conditions work with Public IPs and Elastic IPs only — **not Private IPs**.

---

## Key Policy Examples

### 1 — Restrict Access to AWS Organization Members

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::example-bucket/*",
  "Condition": {
    "StringNotEquals": {
      "aws:PrincipalOrgID": "o-xxxxxxxxxx"
    }
  }
}
```

- Only allows principals whose account belongs to your **AWS Organization**
- Scalable alternative to listing individual account IDs for cross-account access

---

### 2 — Prevent Upload of Unencrypted Objects

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::example-bucket/*",
  "Condition": {
    "StringNotEquals": {
      "s3:x-amz-server-side-encryption": "AES256"
    }
  }
}
```

- Denies any `PutObject` request that does **not** include the encryption header
- Forces all uploads to be server-side encrypted
- Common exam use case: *"How do you ensure all objects are encrypted at upload?"*

---

### 3 — Restrict Access by IP Address

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::example-bucket/*",
  "Condition": {
    "NotIpAddress": {
      "aws:SourceIp": ["203.0.113.0/24"]
    }
  }
}
```

- Denies all requests **not** coming from the allowed IP range
- Uses `NotIpAddress` condition key with `aws:SourceIp`
- Works with **Public IPs and Elastic IPs only** — not Private IPs

---

### 4 — List Bucket + Download Objects (Resource ARN Difference)

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example-bucket"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

**Critical distinction:**

| Action | Resource ARN | Why |
|---|---|---|
| `s3:ListBucket` | `arn:aws:s3:::bucket` | Applies to the **bucket** itself |
| `s3:GetObject` | `arn:aws:s3:::bucket/*` | Applies to **objects** inside the bucket |

> This is a very common exam trap — `ListBucket` uses the bucket ARN (no `/*`), `GetObject` requires `/*`.

---

### 5 — Require MFA for Access

```json
{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::example-bucket/*",
  "Condition": {
    "BoolIfExists": {
      "aws:MultiFactorAuthPresent": "false"
    }
  }
}
```

- Denies all S3 actions unless the request was made with **MFA authentication**
- Use case: protect sensitive buckets from access without MFA

---

## Summary of Condition Use Cases

| Goal | Condition Key |
|---|---|
| Allow only org members | `aws:PrincipalOrgID` |
| Force encryption on upload | `s3:x-amz-server-side-encryption` |
| Restrict by IP | `aws:SourceIp` + `NotIpAddress` |
| Restrict to VPC/VPC Endpoint | `aws:SourceVpc` / `aws:SourceVpce` |
| Require MFA | `aws:MultiFactorAuthPresent` |
| Restrict to CloudFront only | `aws:SourceArn` (CloudFront OAI/OAC ARN) |

---

## SysOps Exam Q&A

**Q: How do you ensure all objects uploaded to S3 are encrypted?**
A: Add a bucket policy that **denies `s3:PutObject`** when the `s3:x-amz-server-side-encryption` header is not present (or not set to the required value).

**Q: `s3:ListBucket` is failing even though the bucket policy allows it. The ARN used is `bucket/*`. What is wrong?**
A: `s3:ListBucket` applies to the **bucket** itself, not objects. The resource ARN must be `arn:aws:s3:::bucket-name` (without `/*`).

**Q: You want to restrict S3 access to only accounts within your AWS Organization. What condition key do you use?**
A: `aws:PrincipalOrgID` — checks that the requester's account belongs to the specified organization ID.

**Q: Can you use a bucket policy condition to restrict access based on a private IP?**
A: **No** — `aws:SourceIp` works with **public IPs and Elastic IPs only**, not Private IPs. For private IP restriction, use `aws:SourceVpc` or `aws:SourceVpce`.

**Q: How do you restrict S3 access to requests coming only through a specific VPC Endpoint?**
A: Use the `aws:SourceVpce` condition key with the VPC Endpoint ID in the bucket policy.

---

## Quick Reference

```
ListBucket  → resource = bucket ARN (no /*)
GetObject   → resource = bucket ARN + /*
PutObject   → resource = bucket ARN + /*

IP restriction     = aws:SourceIp (public/EIP only, NOT private IP)
VPC restriction    = aws:SourceVpc or aws:SourceVpce
Org restriction    = aws:PrincipalOrgID
Encrypt on upload  = s3:x-amz-server-side-encryption condition (Deny if missing)
MFA required       = aws:MultiFactorAuthPresent: false → Deny
```
