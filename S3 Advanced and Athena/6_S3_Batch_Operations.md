# 6 — S3 Batch Operations

## Overview

S3 Batch Operations lets you run bulk actions on large numbers of existing S3 objects with a single request, with built-in retry management, progress tracking, and reporting.

---

## What You Can Do with S3 Batch Operations

| Operation | Example |
|---|---|
| **Modify object metadata/properties** | Update metadata on millions of objects at once |
| **Copy objects between buckets** | Bulk copy with transforms |
| **Encrypt unencrypted objects** | Retroactively encrypt all objects in a bucket ← **exam favourite** |
| **Modify ACLs or tags** | Bulk tag update |
| **Restore from Glacier** | Restore many objects at once |
| **Invoke Lambda function** | Custom action on every object in the list |

---

## Why Use Batch Operations Instead of Custom Scripts?

| Feature | S3 Batch Operations | Custom Script |
|---|---|---|
| **Retry management** | Built-in | Manual |
| **Progress tracking** | Built-in | Manual |
| **Completion notifications** | Yes | Manual |
| **Reports** | Yes | Manual |
| **Scale** | Billions of objects | Limited by script |

---

## How a Batch Job Works

```
S3 Inventory (list of objects in bucket)
       │
       ▼
  Amazon Athena (filter — e.g., only unencrypted objects)
       │
       ▼
  Object list (CSV)
       │
       ▼
  S3 Batch Operation (object list + action + optional params)
       │
       ▼
  S3 processes every object → progress tracking + completion report
```

### Job Components
1. **List of objects** — generated via S3 Inventory + Athena (or provided directly)
2. **Action to perform** — e.g., encrypt, copy, invoke Lambda
3. **Optional parameters** — e.g., destination bucket, encryption key

---

## Key Exam Pattern: Encrypt Unencrypted Objects

1. Use **S3 Inventory** to get a full object list including encryption status
2. Use **Athena** to query and filter — return only unencrypted objects
3. Pass filtered list to **S3 Batch Operations** with encrypt action
4. Batch retroactively encrypts all unencrypted objects

---

## SysOps Exam Q&A

**Q: You need to encrypt all existing unencrypted objects in an S3 bucket. What is the recommended approach?**
A: Use **S3 Inventory** to list objects with their encryption status, **Athena** to filter unencrypted objects, then **S3 Batch Operations** to encrypt them all at once.

**Q: What are the advantages of S3 Batch Operations over a custom script?**
A: Built-in **retry management**, **progress tracking**, **completion notifications**, and **reports** — no custom error handling needed.

**Q: What two services are used to generate a filtered object list for S3 Batch Operations?**
A: **S3 Inventory** (get the full object list) + **Amazon Athena** (query and filter the list).

**Q: What can you do with S3 Batch Operations and Lambda?**
A: Invoke a **Lambda function** on every object in the list to perform any custom processing action.

---

## Quick Reference

```
S3 Batch Operations = bulk actions on existing S3 objects

Use cases:
  - Encrypt unencrypted objects (key exam scenario)
  - Copy objects between buckets
  - Modify metadata, ACLs, tags
  - Restore from Glacier in bulk
  - Invoke Lambda per object

Generate object list:
  S3 Inventory → Athena (filter) → CSV list → S3 Batch

Benefits vs script:
  Built-in retries, progress, notifications, reports
```
