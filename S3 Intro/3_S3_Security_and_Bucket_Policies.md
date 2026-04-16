# 3 — S3 Security & Bucket Policies

## Overview

Two main approaches to S3 security: **User-Based** (IAM policies) and **Resource-Based** (Bucket Policies, ACLs). Bucket Policies are the primary and most common method today.

---

## Security Layers

### 1 — User-Based: IAM Policies
- Attached to an **IAM user, group, or role**
- Defines which S3 API calls that principal is allowed to make
- Managed in IAM, not in S3

### 2 — Resource-Based: Bucket Policies *(most common)*
- JSON-based policies attached **directly to the S3 bucket**
- Bucket-wide rules — apply to all objects in the bucket
- Use cases:
  - Grant **public access** to a bucket
  - Allow **cross-account access**
  - Force **encryption at upload**

### 3 — Object ACL (Access Control List)
- Fine-grained, per-object access control
- Can be **disabled** (recommended default)

### 4 — Bucket ACL
- Bucket-level ACL — less common
- Can also be **disabled**

### 5 — Encryption
- Encrypt objects using encryption keys
- Adds another layer of protection for data at rest

> **Best practice:** Use Bucket Policies. ACLs are legacy and can be disabled.

---

## Access Decision Logic

An IAM principal can access an S3 object if:

```
(IAM policy ALLOWS it  OR  Bucket policy ALLOWS it)
              AND
        no explicit DENY exists
```

If either the IAM policy or the Bucket policy grants access — and nothing explicitly denies — access is granted.

---

## Bucket Policy Structure (JSON)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

| Field | Description |
|---|---|
| **Effect** | `Allow` or `Deny` |
| **Principal** | Who the policy applies to (`*` = everyone, or specific account/user ARN) |
| **Action** | S3 API call(s) — e.g., `s3:GetObject`, `s3:PutObject` |
| **Resource** | Bucket and/or object ARN — `/*` means all objects in the bucket |

### Reading the Example Above
- `Principal: *` — anyone (public internet)
- `Action: s3:GetObject` — read/download objects
- `Resource: arn:aws:s3:::example-bucket/*` — all objects in `example-bucket`
- Result: **public read access on all objects** in the bucket

---

## Common Access Patterns

### Pattern 1 — Public Access (Website Visitors)
```
Internet User
      │  HTTP request
      ▼
S3 Bucket Policy (Allow s3:GetObject to Principal: *)
      │
      ▼
S3 Object (publicly readable)
```
- Requires **disabling Block Public Access** on the bucket first

### Pattern 2 — IAM User Access (Same Account)
```
IAM User
      │  API call
      ▼
IAM Policy (Allow s3:GetObject on bucket ARN)
      │
      ▼
S3 Bucket
```
- No bucket policy needed if IAM policy grants access
- Managed purely through IAM

### Pattern 3 — EC2 Instance Access
```
EC2 Instance
      │  uses attached IAM Role
      ▼
EC2 IAM Role (Allow S3 permissions)
      │
      ▼
S3 Bucket
```
- Use an **IAM Role**, not an IAM User, for EC2 → S3 access
- Role is attached to the EC2 instance profile

### Pattern 4 — Cross-Account Access
```
IAM User (Account B)
      │
      ▼
S3 Bucket Policy (Account A) — Allow specific Account B user ARN
      │
      ▼
S3 Bucket (Account A)
```
- **Requires a Bucket Policy** — IAM policies alone cannot grant cross-account S3 access
- Bucket policy must explicitly allow the external account/user ARN

---

## Block Public Access Settings

A safety layer **on top of** Bucket Policies, designed to prevent accidental data leaks.

| Behaviour | Detail |
|---|---|
| If Block Public Access is **ON** | Bucket is never public — even if a Bucket Policy tries to allow it |
| If Block Public Access is **OFF** | Bucket Policy controls public access |
| Account-level setting | Can be applied to **all buckets** in an account |

> **Use case:** If you know a bucket should **never** be public, leave Block Public Access ON. It overrides any misconfigured bucket policy.

```
Block Public Access ON  →  Bucket can NEVER be public (regardless of bucket policy)
Block Public Access OFF →  Bucket Policy determines public access
```

---

## Best Practices

- Use **Bucket Policies** as the primary access control mechanism
- Keep **Block Public Access ON** unless you intentionally need public access
- Set Block Public Access at the **account level** if no buckets should ever be public
- For EC2 → S3: always use **IAM Roles**, never IAM users or access keys on instances
- For cross-account access: use a **Bucket Policy** that explicitly allows the external principal

---

## SysOps Exam Q&A

**Q: What is the most common way to control access to an S3 bucket?**
A: **S3 Bucket Policy** — a JSON resource-based policy attached directly to the bucket.

**Q: An EC2 instance needs to access an S3 bucket. What is the correct approach?**
A: Attach an **IAM Role** with S3 permissions to the EC2 instance. Never use IAM user credentials on instances.

**Q: A bucket policy grants public access but the bucket is still not public. Why?**
A: **Block Public Access** is enabled on the bucket or account. It overrides bucket policies and prevents public access regardless of policy settings.

**Q: How do you grant cross-account S3 access?**
A: Use a **Bucket Policy** that explicitly allows the external account's IAM user or role ARN as the principal.

**Q: An IAM user has no S3 IAM policy but the bucket policy allows their ARN. Can they access the bucket?**
A: **Yes** — access is granted if either the IAM policy OR the Bucket Policy allows it, and there is no explicit deny.

**Q: What happens if Block Public Access is set at the account level?**
A: **All buckets** in the account are blocked from being public — regardless of individual bucket policies.

---

## Quick Reference

```
IAM Policy          = user/role-based; managed in IAM
Bucket Policy       = resource-based; attached to bucket; most common
Object/Bucket ACL   = legacy; can be disabled (recommended)
Cross-account       = requires Bucket Policy (not IAM alone)
EC2 access          = IAM Role (never IAM user credentials)
Block Public Access = overrides bucket policies; prevents public access
Account-level block = applies Block Public Access to ALL buckets in account
```
