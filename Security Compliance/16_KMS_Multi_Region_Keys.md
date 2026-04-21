# 16 — KMS Multi-Region Keys

## Overview

A set of KMS keys with the **same key ID and same key material** replicated across multiple AWS regions — allowing encrypt in one region and decrypt in another.

```
Primary Key (us-east-1): mrk-abc123...
  ├── Replica (us-west-2):    mrk-abc123...  (same key ID)
  ├── Replica (eu-west-1):    mrk-abc123...  (same key ID)
  └── Replica (ap-southeast-2): mrk-abc123... (same key ID)
```

---

## Key Properties

| Property | Detail |
|---|---|
| Key ID | **Same across all regions** (prefix: `mrk-`) |
| Key material | Replicated from primary to replicas |
| Key rotation | If primary rotates automatically, rotation is replicated |
| Management | Each replica managed **independently** (own key policy) |
| Global? | **No** — primary + replicas, not a single global key |

---

## Why Use Multi-Region Keys

- Encrypt in one region, **decrypt in another** — no re-encryption needed
- No cross-region API calls required for decryption
- Interchangeable across regions

---

## Use Cases (Specific — Not General Purpose)

| Use Case | Why Multi-Region Key |
|---|---|
| Global client-side encryption | Encrypt client-side in us-east-1, decrypt in eu-west-1 |
| DynamoDB Global Tables | Encrypt DynamoDB data consistently across replicated regions |
| Aurora Global Database | Encrypt Aurora data usable in all global regions |

> AWS recommends **against** using multi-region keys as a general practice — KMS prefers keys scoped to a single region. Use only for the specific use cases above.

---

## SysOps Exam Q&A

**Q: What is the key characteristic that makes KMS multi-region keys work across regions?**
A: The same key ID and key material exist in all regions — encrypting in one region produces ciphertext decryptable in any replica region without re-encryption.

**Q: Are KMS multi-region keys truly global (like a single key)?**
A: No — each replica is managed independently with its own key policy. They share key material but are not a single global resource.

**Q: What are the recommended use cases for KMS multi-region keys?**
A: Global client-side encryption, DynamoDB Global Tables, and Aurora Global Database.

---

## Quick Reference

```
KMS Multi-Region Keys:
  Same key ID (mrk-...) + same key material across regions
  Encrypt in region A → decrypt in region B (no re-encrypt, no cross-region API call)
  Primary + replicas — each managed independently (not global)
  Auto-rotation on primary → replicated to replicas

Use cases (only):
  Global client-side encryption
  DynamoDB Global Tables
  Aurora Global Database

Not recommended for general use — KMS keys should be region-scoped by default
```
