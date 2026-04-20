# 12. AWS KMS — Key Management Service

## Overview

**AWS KMS** is the central encryption key management service in AWS. Anytime you enable encryption on an AWS service (EBS, S3, RDS, SSM, etc.), KMS is almost always behind it. KMS is fully integrated with IAM for authorization and CloudTrail for audit logging of every key usage.

---

## Key Types

### Symmetric vs Asymmetric

| Type | Keys | Usage | Access to Key Material |
|------|------|-------|----------------------|
| **Symmetric** | Single key (encrypt + decrypt) | All AWS service integrations use symmetric keys | Never — only via KMS API calls |
| **Asymmetric** | Public key + Private key | Encrypt/decrypt or sign/verify | Public key downloadable; private key via API only |

> **Asymmetric use case:** External users without AWS access encrypt data using the downloaded public key → you decrypt it using the private key via KMS API.

---

## KMS Key Categories

| Category | Cost | Name Format | Notes |
|----------|------|-------------|-------|
| **AWS Owned Keys** | Free | No visible name | Used by SSE-S3, SSE-DynamoDB (default option) — not visible in KMS console |
| **AWS Managed Keys** | Free | `aws/<service>` (e.g., `aws/rds`, `aws/ebs`) | Free, managed by AWS, only usable within the assigned service |
| **Customer Managed Keys** | $1/month | Custom name | Full control; supports custom key policies, cross-account access |
| **Imported Keys** | $1/month | Custom name | You supply the key material; manual rotation only |

---

## Pricing

| Item | Cost |
|------|------|
| AWS managed keys | Free |
| Customer managed / imported keys | $1/month per key |
| KMS API calls | ~$0.03 per 10,000 calls |

---

## Key Rotation

| Key Type | Rotation |
|----------|---------|
| **AWS managed keys** | Automatic — every 1 year |
| **Customer managed keys** | Optional automatic rotation — configurable period; on-demand rotation also supported |
| **Imported keys** | Manual only — must use an alias to rotate |

---

## KMS Keys are Region-Scoped

The same KMS key **cannot** exist in two regions. Copying encrypted data across regions requires re-encryption.

### Cross-Region EBS Snapshot Copy

```
Region A (us-east-2):
  EBS volume encrypted with KMS Key A
        ↓
  Take snapshot → snapshot encrypted with KMS Key A
        ↓
  Copy snapshot to Region B → AWS re-encrypts with KMS Key B (different key)
        ↓
Region B (ap-southeast-2):
  Encrypted snapshot with KMS Key B
        ↓
  Restore to EBS volume encrypted with KMS Key B
```

---

## KMS Key Policies

Controls who can access a KMS key — similar to S3 bucket policies.

> **Critical:** Without a key policy, **no one** can access the key (unlike IAM where absence of deny allows access).

| Policy Type | Behavior |
|-------------|---------|
| **Default key policy** | Allows the entire AWS account to access the key via IAM policies |
| **Custom key policy** | Define specific users/roles who can use or administer the key |

### Custom Key Policy Use Cases

- Restrict which IAM users/roles can use the key
- Define key administrators separately from key users
- **Cross-account access** — authorize another AWS account to use your key

---

## Cross-Account Encrypted Snapshot Copy

```
Source Account:
  1. Create snapshot encrypted with Customer Managed Key (CMK)
  2. Attach custom key policy → authorize target account
  3. Share encrypted snapshot with target account

Target Account:
  4. Create a COPY of the snapshot → encrypt with target account's own CMK
  5. Restore volume from the copy
```

> Must use a **Customer Managed Key** for cross-account — default and AWS managed keys cannot have custom policies.

---

## Best Practices

✓ **Never store secrets in plaintext** — encrypt with KMS via API/CLI/SDK and store the ciphertext  
✓ **Use CloudTrail to audit KMS key usage** — every API call to KMS is logged  
✓ **Use customer managed keys for cross-account scenarios** — only CMKs support custom key policies  
✓ **Enable automatic key rotation on CMKs** — reduces risk from long-lived key exposure  
✓ **Use key aliases for imported keys** — makes manual rotation transparent to applications  

---

## SysOps Exam Focus

**Q1: "You need to copy an EBS snapshot encrypted with KMS from us-east-1 to eu-west-1. What is required?"**
- A) KMS keys are global — copy the snapshot directly without changes
- B) Take a snapshot, then re-encrypt it with a KMS key in eu-west-1 during the copy operation
- C) Decrypt the snapshot first, copy it, then re-encrypt in eu-west-1
- D) Create a new unencrypted snapshot and copy it to eu-west-1
- **Answer: B** — KMS keys are region-scoped; AWS re-encrypts the snapshot with a different key in the destination region during the copy

**Q2: "Every API call made to use a KMS key is logged where?"**
- A) Amazon CloudWatch Logs
- B) AWS CloudTrail
- C) AWS Config
- D) Amazon S3 access logs
- **Answer: B** — CloudTrail logs every KMS API call (Encrypt, Decrypt, GenerateDataKey, etc.) for audit purposes

**Q3: "You have an encrypted EBS snapshot and want to share it with another AWS account. What must you use?"**
- A) AWS managed key (aws/ebs) — it is automatically shareable
- B) A Customer Managed Key with a custom key policy that authorizes the target account
- C) An AWS owned key with cross-account access enabled
- D) Decrypt the snapshot and share an unencrypted copy
- **Answer: B** — Only Customer Managed Keys support custom key policies; you must grant the target account permission to use the CMK to decrypt the shared snapshot

**Q4: "What is the key rotation behavior for AWS managed KMS keys vs customer managed keys?"**
- A) Both rotate automatically every year; neither can be customized
- B) AWS managed keys rotate automatically every year; customer managed keys require explicit enablement with a configurable period
- C) Customer managed keys rotate every year automatically; AWS managed keys never rotate
- D) Imported keys rotate automatically; customer managed keys require manual rotation only
- **Answer: B** — AWS managed keys auto-rotate yearly; CMK rotation must be enabled and the period is configurable

**Q5: "What happens if you create a KMS key without any key policy?"**
- A) The default IAM deny-all applies — no one can access the key
- B) A default key policy is created that allows the entire AWS account to use the key via IAM policies
- C) AWS automatically applies the aws/kms managed key policy
- D) The key cannot be created without a policy
- **Answer: B** — If no key policy is specified, a default policy is applied that delegates access to IAM; any principal with an appropriate IAM policy can use the key

---

## Quick Reference

```
KMS Key Types:
  AWS Owned       → free, invisible (SSE-S3, SSE-DynamoDB default)
  AWS Managed     → free, aws/<service> format, auto-rotates yearly
  Customer Managed → $1/month, custom name, full control, cross-account
  Imported        → $1/month, manual rotation only (use alias)

Symmetric: one key, encrypt + decrypt — all AWS service integrations
Asymmetric: public + private — external encryption use cases

Region-scoped → cross-region copy requires re-encryption with different key

Key policies:
  No policy = no access (unlike IAM)
  Default = account-wide access via IAM
  Custom = specific users/roles + cross-account access

Cross-account snapshot share:
  CMK required → custom key policy → authorize target account
  Target account copies snapshot with their own CMK

Audit: every KMS API call logged in CloudTrail
```

---

**File: 12_AWS_KMS.md**
**Status: SysOps-focused, exam-ready, concise format**
