# 9 — S3 Multi-Part Upload (Deep Dive)

## Overview

Multi-part upload splits large files into parts that are uploaded in parallel and then assembled by S3. A lifecycle rule should be used to automatically clean up failed/incomplete uploads.

---

## Rules

| Rule | Detail |
|---|---|
| **Recommended for** | Files > **100 MB** |
| **Required for** | Files > **5 GB** |
| **Maximum parts** | **10,000 parts** per upload |
| **Retry on failure** | Only the failed part needs to be re-uploaded |

---

## How It Works

```
Large file
    │
    ├── Part 1 ──►┐
    ├── Part 2 ──►┤
    ├── Part 3 ──►┤  (uploaded in parallel to S3)
    │   ...       │
    └── Part N ──►┘
                  │
                  ▼
        CompleteMultipartUpload API call
                  │
                  ▼
        S3 concatenates all parts into the final object
```

1. **Initiate** — start the multi-part upload, get an upload ID
2. **Upload parts** — upload each part in parallel (any order)
3. **Complete** — call `CompleteMultipartUpload`; S3 assembles the final file
4. **On failure** — retry only the failed part(s); other parts are not re-uploaded

---

## Benefits

| Benefit | Detail |
|---|---|
| **Speed** | Parallel uploads maximize network bandwidth |
| **Resilience** | Failed parts retry independently — no full restart |
| **Large file support** | Only way to upload files > 5 GB |

---

## Incomplete Multi-Part Uploads

If an upload is interrupted (network outage, app crash, etc.):
- Parts already uploaded remain in S3 **and incur storage costs**
- The upload ID becomes stale — the object is never fully assembled
- These "orphaned" parts are not visible as normal objects but still cost money

### Fix: Lifecycle Rule to Auto-Delete

Create a lifecycle rule to automatically delete incomplete multi-part uploads after N days:

1. S3 → Bucket → **Management** → **Lifecycle rules** → **Create lifecycle rule**
2. Name: e.g., `cleanup-incomplete-multipart`
3. Apply to: all objects (or specific prefix)
4. Action: **Delete expired delete markers or incomplete multipart uploads**
5. Set: **Delete incomplete multipart uploads after X days** (e.g., 3 days)
6. Save

> This is an exam-relevant pattern: *"How do you prevent cost accumulation from stalled multi-part uploads?"* → **Lifecycle rule targeting incomplete multi-part uploads.**

---

## How to Use Multi-Part Upload

Multi-part upload is **not available in the S3 console** for manual use — use the:
- **AWS CLI** — automatically uses multi-part upload for large files
- **AWS SDK** — programmatic control over part size and parallelism

---

## SysOps Exam Q&A

**Q: When is multi-part upload required vs recommended?**
A: **Required** for files > 5 GB. **Recommended** for files > 100 MB.

**Q: What is the maximum number of parts in a multi-part upload?**
A: **10,000 parts**.

**Q: Incomplete multi-part uploads are accumulating storage costs. What is the fix?**
A: Create a **lifecycle rule** with the action "Delete incomplete multipart uploads" after a specified number of days (e.g., 3 days).

**Q: A part fails during a multi-part upload. Does the entire upload need to restart?**
A: **No** — only the failed part needs to be re-uploaded. All successfully uploaded parts are retained.

**Q: How do you trigger multi-part upload from the AWS CLI?**
A: The CLI **automatically** uses multi-part upload for large files — no special flags needed for basic usage.

---

## Quick Reference

```
Recommended > 100 MB, Required > 5 GB
Max parts   = 10,000
Process     = upload parts in parallel → CompleteMultipartUpload → S3 assembles
Retry       = failed parts only (not the whole file)
Orphaned parts = incur cost; invisible in console

Cleanup:
  Lifecycle rule → "Delete incomplete multipart uploads" → after N days
  Prevents cost accumulation from stalled uploads

Tools: CLI or SDK (not S3 console)
```
