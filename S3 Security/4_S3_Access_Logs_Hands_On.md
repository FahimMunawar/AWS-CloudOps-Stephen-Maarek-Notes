# 4 — S3 Access Logs — Hands-On

## Overview

Console walkthrough: enable S3 server access logging on a source bucket, verify the auto-updated bucket policy on the logging bucket, and confirm log delivery.

---

## Setup Steps

### Step 1 — Create a Logging Bucket
- Create a dedicated S3 bucket (e.g., `stephane-access-log-v3`)
- Must be in the **same region** as the bucket you want to monitor
- Keep Block Public Access enabled

### Step 2 — Enable Server Access Logging on Source Bucket
1. S3 → Source bucket → **Properties** → **Server access logging** → Edit
2. **Enable** logging
3. **Destination**: browse and select your logging bucket (`stephane-access-log-v3`)
4. **Prefix** (optional): e.g., `logs/` — leave blank to write logs to the root of the logging bucket
5. **Log object key format**: two options
   - Default (date-based key) — sufficient for most use cases
   - Custom with S3 event time or log file delivery time
6. Click **Save changes**

---

## What Happens Automatically

When logging is enabled, AWS **automatically updates the bucket policy** on the logging (destination) bucket to allow the S3 logging service to write objects:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "logging.s3.amazonaws.com"
  },
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::stephane-access-log-v3/*"
}
```

- You do **not** need to manually add this policy — S3 handles it
- Verify: logging bucket → **Permissions** → **Bucket policy**

---

## Log Delivery Timing

- Logs are **not delivered instantly** — takes a few hours for the first logs to appear
- After the initial delay, logs accumulate in the logging bucket as activity occurs

---

## Log File Contents

Each log file contains one line per request with fields such as:
- Bucket owner and bucket name
- Request timestamp
- Requester IP and identity
- API operation (e.g., `GET`, `PUT`, `DELETE`)
- HTTP status code and error code
- Bytes transferred
- Object key accessed

The format is space-delimited and can be difficult to read raw. Use **Amazon Athena** to query the logs with SQL for meaningful analysis.

---

## Key Points

| Point | Detail |
|---|---|
| **Bucket policy auto-updated** | S3 automatically grants logging service write permission to destination bucket |
| **Same region** | Source and logging bucket must be in the same AWS region |
| **Log delivery delay** | First logs may take hours to appear |
| **Separate bucket** | Never use the same bucket as source and destination |
| **Use Athena to analyze** | Raw logs are hard to read; SQL queries make analysis practical |

---

## SysOps Exam Q&A

**Q: When you enable S3 server access logging, what happens to the destination bucket's policy?**
A: AWS **automatically adds a bucket policy** granting `s3:PutObject` to the `logging.s3.amazonaws.com` service principal — no manual policy change needed.

**Q: How long does it take for S3 access logs to appear in the logging bucket?**
A: Logs are typically delivered within a few hours — not instantly.

**Q: What is the best tool to analyze S3 access logs?**
A: **Amazon Athena** — create a table pointing to the logging bucket and run SQL queries (as covered in the Athena hands-on).

---

## Quick Reference

```
Source bucket  = bucket you want to monitor
Logging bucket = separate bucket, same region
Enable:        Source bucket → Properties → Server access logging → Enable
Auto policy:   S3 updates destination bucket policy automatically
               (logging.s3.amazonaws.com gets s3:PutObject)
Delivery delay = hours (not instant)
Analyze with   = Amazon Athena (SQL on raw logs)
```
