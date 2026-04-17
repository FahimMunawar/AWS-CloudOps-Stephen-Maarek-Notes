# 3 — S3 Access Logs

## Overview

S3 Access Logs record all requests made to an S3 bucket — authorized or denied — and write them as log files to a separate **logging bucket** in the same region.

---

## How It Works

```
Any request (GET, PUT, DELETE, etc.)
       │
       ▼
  S3 Bucket (monitored)
       │  access logs
       ▼
  Logging S3 Bucket (separate bucket, same region)
       │
       ▼
  Amazon Athena (analyze logs with SQL)
```

- Logs include: requester account, bucket name, request time, action, response status, error code
- Can be analyzed with **Amazon Athena**, Amazon Redshift, or other tools

---

## Key Requirements

| Requirement | Detail |
|---|---|
| **Logging bucket location** | Must be in the **same AWS region** as the monitored bucket |
| **Logging bucket** | Must be a **separate bucket** — never the same as the monitored bucket |
| **Requests logged** | All requests — authorized and denied, from any account |

---

## Critical Warning: Logging Loop

> **Never set the logging bucket to be the same as the bucket you are monitoring.**

If the source and logging buckets are the same:
```
PUT object to bucket
    → access log written to same bucket
    → log write triggers another access log
    → that write triggers another access log
    → infinite loop → bucket grows exponentially → massive unexpected cost
```

**Always use a dedicated, separate bucket for access logs.**

---

## Setup (Console)

1. S3 → Source bucket → **Properties** → **Server access logging** → Edit
2. Enable logging
3. **Destination**: browse S3 and select a **different** bucket in the same region
4. Optional: set a log prefix (e.g., `access-logs/`)
5. Save

---

## Use Cases

| Use Case | Detail |
|---|---|
| **Security auditing** | Track who accessed what and when |
| **Unauthorized access investigation** | Find 403 errors and their sources |
| **Usage analysis** | Understand access patterns |
| **Compliance** | Maintain audit trail for regulatory requirements |

---

## SysOps Exam Q&A

**Q: What is S3 Access Logging used for?**
A: To log all requests (authorized or denied) made to an S3 bucket to a separate logging bucket for auditing and analysis.

**Q: A company enables S3 access logging but sets the logging bucket to the same bucket being monitored. What happens?**
A: An **infinite logging loop** — each log write generates a new log entry, causing the bucket to grow exponentially and incurring massive storage costs.

**Q: Where must the S3 access log destination bucket be located?**
A: In the **same AWS region** as the monitored bucket.

**Q: Which AWS service is commonly used to analyze S3 access logs?**
A: **Amazon Athena** — query the logs with SQL to identify patterns, errors, and unauthorized access.

---

## Quick Reference

```
S3 Access Logs = record all requests to a bucket (authorized + denied)
Destination    = separate S3 bucket in the SAME region
Analyze with   = Amazon Athena (SQL queries on log data)

NEVER use the same bucket as source and destination
       → infinite logging loop → exponential cost

Setup: Source bucket → Properties → Server access logging → Enable → set destination
```
