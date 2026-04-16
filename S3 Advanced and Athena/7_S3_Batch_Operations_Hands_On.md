# 7 — S3 Batch Operations — Hands-On

## Overview

Console walkthrough: create and run an S3 Batch Operations job to add tags to multiple objects using a CSV manifest file.

---

## Setup

### Files Needed
- Objects to operate on: `beach.jpg`, `coffee.jpg`, `error.html`, `index.html`
- Manifest file: `s3-batch.csv` — lists the bucket name and object keys to operate on

### CSV Manifest Format
```
bucket-name,beach.jpg
bucket-name,coffee.jpg
bucket-name,error.html
bucket-name,index.html
```
> Update `bucket-name` in the CSV to match your actual bucket name before uploading.

Upload all files (objects + the CSV) to the same bucket.

---

## Creating a Batch Job

### Step 1 — Open Batch Operations
- S3 console left sidebar → **Batch Operations** → **Create job**

### Step 2 — Select Manifest
- Region: match your bucket's region
- Manifest format: **CSV** (or S3 Inventory Report)
- Browse S3 → select your `s3-batch.csv` file

### Step 3 — Choose Operation
Available operation types:

| Operation | Description |
|---|---|
| **Copy** | Copy objects to another bucket |
| **Invoke Lambda** | Run custom Lambda on each object |
| **Replace all object tags** | Set new tags (overwrites existing) |
| **Delete all object tags** | Remove all tags |
| **Restore** | Restore objects from Glacier |
| **Object Lock** | Apply retention settings |
| ACL | Update object ACLs |

For this demo: **Replace all object tags**
- Key: `demo`, Value: `AWS`

### Step 4 — Configure Job
- **Priority**: optional ordering if you have multiple jobs
- **Completion report**: choose All tasks or Failed tasks only
- **Report destination**: browse S3 → choose a path in your bucket (e.g., `batch-results/`)

### Step 5 — IAM Role
S3 Batch needs an IAM role to read/write objects. Create a role:

1. IAM → Roles → Create role
2. Service: **S3** → use case: **S3 Batch Operations**
3. Attach policy: `AmazonS3FullAccess` (for demo; use least privilege in production)
4. Name: `S3BatchDemoRole`

Minimum required permissions (least privilege):
```json
{
  "Actions": [
    "s3:PutObjectTagging",     // apply tags to objects
    "s3:GetObject",            // read the manifest
    "s3:PutObject"             // write results to report destination
  ]
}
```

### Step 6 — Review & Run
- Job is created in **"Awaiting confirmation"** state
- Console shows how many files were detected from the manifest (e.g., 4 files)
- Click **Run job** to confirm execution

---

## Verifying Results

### Check Object Tags
- S3 → Bucket → `coffee.jpg` → scroll to **Tags**
- Tag `demo: AWS` now appears on all objects in the manifest

### Check Completion Report
- S3 → `batch-results/` folder → download the report CSV
- Contains: task IDs, object keys, success/failure status for each object

### Job Summary
- Batch Operations console → click on job → view:
  - Success rate
  - Failed count
  - Manifest used
  - Result report location

---

## Key Points

| Point | Detail |
|---|---|
| **Manifest types** | CSV (manual) or S3 Inventory Report (automated) |
| **Job state** | Starts as "Awaiting confirmation" — must manually run |
| **IAM role required** | S3 Batch needs permission to read objects + write results |
| **Completion report** | Stored in S3; shows per-object success/failure |
| **Operation scope** | Only objects listed in the manifest are affected |

---

## SysOps Exam Q&A

**Q: What are the two supported manifest formats for S3 Batch Operations?**
A: **S3 Inventory Report** and **CSV file**.

**Q: An S3 Batch job is created but not running. What state is it in and what is required?**
A: **"Awaiting confirmation"** — you must manually click **Run job** to start execution.

**Q: What IAM configuration is required for S3 Batch Operations?**
A: An **IAM role** must be created and attached to the job. The role needs S3 permissions to read the manifest, perform the operation, and write the completion report.

---

## Quick Reference

```
Manifest     = CSV (bucket + key per line) or S3 Inventory Report
Operation    = tags, copy, Lambda, restore, ACL, Object Lock
IAM role     = required; needs read object + write results permissions
Job state    = "Awaiting confirmation" → must click Run job
Results      = completion report stored in S3 (per-object success/failure)
Verify       = check object tags/properties after job completes
```
