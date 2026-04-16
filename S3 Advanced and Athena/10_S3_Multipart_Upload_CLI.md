# 10 — S3 Multi-Part Upload — CLI Commands

## Overview

Step-by-step CLI commands for performing a manual multi-part upload using `aws s3api`. All multi-part upload operations use the `s3api` subcommand (not `s3`).

---

## Command Reference

### 1 — Create a Bucket
```bash
aws s3 mb s3://multipartuploadstephandemo
```

### 2 — Create a Large Test File (100 MB)
```bash
dd if=/dev/zero of=100MB.txt bs=1M count=100
```

### 3 — Split File into Parts (~35 MB each)
```bash
split -b 35m 100MB.txt 100MP_part_
# Creates: 100MP_part_aa (35MB), 100MP_part_ab (35MB), 100MP_part_ac (~30MB)
```

### 4 — Initiate the Multi-Part Upload
```bash
aws s3api create-multipart-upload \
  --bucket multipartuploadstephandemo \
  --key 100MB.txt
```

**Response:**
```json
{
  "Bucket": "multipartuploadstephandemo",
  "Key": "100MB.txt",
  "UploadId": "abc123xyz..."
}
```

> **Save the UploadId** — required for all subsequent commands.

```bash
UploadId="abc123xyz..."    # store as variable
```

### 5 — List In-Progress Multi-Part Uploads
```bash
aws s3api list-multipart-uploads --bucket multipartuploadstephandemo
```
Shows all initiated but not yet completed uploads.

### 6 — Upload Each Part
```bash
# Part 1
aws s3api upload-part \
  --bucket multipartuploadstephandemo \
  --key 100MB.txt \
  --part-number 1 \
  --body 100MP_part_aa \
  --upload-id $UploadId

# Part 2
aws s3api upload-part \
  --bucket multipartuploadstephandemo \
  --key 100MB.txt \
  --part-number 2 \
  --body 100MP_part_ab \
  --upload-id $UploadId

# Part 3
aws s3api upload-part \
  --bucket multipartuploadstephandemo \
  --key 100MB.txt \
  --part-number 3 \
  --body 100MP_part_ac \
  --upload-id $UploadId
```

Each command returns an **ETag** — save each one for the complete step.

> **Note:** Parts do **not appear** in the S3 console or object list until the upload is completed.

### 7 — List Parts (Verify Uploads)
```bash
aws s3api list-parts \
  --bucket multipartuploadstephandemo \
  --key 100MB.txt \
  --upload-id $UploadId
```
Shows part number, ETag, and size for each uploaded part.

### 8 — Complete the Multi-Part Upload
```bash
aws s3api complete-multipart-upload \
  --bucket multipartuploadstephandemo \
  --key 100MB.txt \
  --upload-id $UploadId \
  --multipart-upload '{
    "Parts": [
      {"PartNumber": 1, "ETag": "<etag-from-part-1>"},
      {"PartNumber": 2, "ETag": "<etag-from-part-2>"},
      {"PartNumber": 3, "ETag": "<etag-from-part-3>"}
    ]
  }'
```

S3 assembles all parts into the final object. The object now appears in the bucket.

---

## Key Behaviours

| Behaviour | Detail |
|---|---|
| Parts not visible in console | Parts only appear after `complete-multipart-upload` is called |
| Parts incur storage cost | Even incomplete parts are stored and billed until deleted |
| ETags required to complete | Each part returns an ETag; all must be provided to complete |
| Parts can be uploaded in any order | Part numbers determine final assembly order, not upload order |
| `s3` vs `s3api` | `s3 mb` to create bucket; everything else uses `s3api` |

---

## Abort an Incomplete Upload
```bash
aws s3api abort-multipart-upload \
  --bucket multipartuploadstephandemo \
  --key 100MB.txt \
  --upload-id $UploadId
```
Deletes all uploaded parts and removes the upload ID.

---

## SysOps Exam Q&A

**Q: Which AWS CLI subcommand is used for multi-part upload operations?**
A: **`aws s3api`** — not `aws s3`.

**Q: Why don't uploaded parts appear in the S3 console?**
A: Parts are not assembled into an object until `complete-multipart-upload` is called. They exist as uncommitted parts and are invisible in the object list.

**Q: What information is required to complete a multi-part upload?**
A: The **UploadId** and the **ETag + PartNumber** for every uploaded part.

**Q: How do you clean up a stalled multi-part upload via CLI?**
A: Run `aws s3api abort-multipart-upload` with the bucket, key, and upload ID. Or use a lifecycle rule to auto-delete incomplete uploads after N days.

---

## Quick Reference

```
s3 mb          → create bucket
s3api create-multipart-upload  → get UploadId
s3api upload-part              → upload each part (save ETag per part)
s3api list-multipart-uploads   → see pending uploads
s3api list-parts               → verify parts uploaded
s3api complete-multipart-upload → assemble final object (needs all ETags)
s3api abort-multipart-upload   → cancel and clean up parts

Parts invisible in console until complete is called
ETags: returned per part; all required to complete
```
