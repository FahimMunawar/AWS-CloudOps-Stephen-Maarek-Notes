# 2 — S3 Buckets & Objects — Hands-On

## Overview

Practical walkthrough of creating an S3 bucket, uploading objects, understanding public vs pre-signed URLs, and working with folders.

---

## Creating a Bucket

### Bucket Type
- **General Purpose** — standard bucket type (default, exam-relevant)
- **Directory** — specific use case, **not covered on the exam**; ignore if you see it

> If your region doesn't show the bucket type option, it defaults to General Purpose automatically.

### Key Settings at Creation

| Setting | Recommended Value | Notes |
|---|---|---|
| **Bucket name** | Unique, personal name | Must be globally unique across all AWS accounts and regions |
| **Object Ownership** | ACL disabled (default) | Recommended security setting |
| **Block Public Access** | Enabled (default) | Blocks all public access — maximum security |
| **Bucket Versioning** | Disabled | Enable later when needed |
| **Default Encryption** | SSE-S3 (Amazon S3 managed key) | All objects encrypted by default |
| **Bucket Key** | Enabled | Reduces encryption costs |

### Bucket Name Tip
- Use a personal, unique name (e.g., `firstname-demo-s3-v1`)
- Generic names like `test` are almost certainly already taken
- Attempting a taken name → error: *"Bucket with the same name already exists"*

---

## S3 Console — Global View

- The S3 console shows **all buckets across all regions** in one view
- Buckets still exist in specific regions (Ireland, London, us-east-1, etc.)
- You can search/filter by bucket name in the console

---

## Uploading Objects

1. Click on bucket → **Upload** → Add files
2. Choose file (e.g., `coffee.jpg`)
3. Destination shown as the bucket name
4. Click Upload → Done

After upload, the object appears in the **Objects** tab with:
- File name
- Size
- Upload date
- Type

---

## Opening Objects: Public URL vs Pre-Signed URL

This is a key concept — two ways to access an S3 object:

### 1 — Open Button (Pre-Signed URL)
- Clicking **Open** in the console generates a **pre-signed URL**
- The URL is long and contains an embedded signature with your credentials
- Works because: S3 verifies *your* credentials are in the URL → grants access
- **Only works for you** — cannot be shared publicly

### 2 — Object URL (Public URL)
- The plain URL shown in the object overview page
- Format: `https://<bucket-name>.s3.<region>.amazonaws.com/<key>`
- Result: **Access Denied** — because the bucket blocks all public access by default

```
Object URL (public):    → ❌ Access Denied (bucket is private)
Open button (pre-signed): → ✅ Works (your credentials embedded in URL)
```

> To make the public URL work, you must explicitly make the object/bucket public — covered in a later lecture.

---

## Pre-Signed URL Explained

A pre-signed URL is a temporary URL that includes:
- The object path
- Your AWS credentials (as a signature)
- An expiry time

S3 verifies the signature → grants access as if you made the request directly.

---

## Working with Folders

- Click **Create folder** inside a bucket → enter folder name (e.g., `images`)
- Upload objects into the folder → destination shows `bucket-name/images/`
- The object key becomes `images/beach.jpg`
- Deleting a folder deletes **all objects within it**
  - Must type `permanently delete` to confirm

> Reminder: Folders are not real in S3 — they are key prefixes. `images/beach.jpg` is just a key that happens to contain `/`.

---

## Best Practices

- Keep **Block Public Access** enabled unless you intentionally need public objects
- Use **default encryption** (SSE-S3) — no additional cost, enabled by default
- Use **personal/unique bucket names** to avoid naming conflicts
- Use the **Open button** (pre-signed URL) during development to test private objects without making them public

---

## SysOps Exam Q&A

**Q: You click the Object URL for a private S3 object and get Access Denied. Why?**
A: The bucket has **Block Public Access** enabled (default). The object URL is the public URL and requires the object/bucket to be publicly accessible.

**Q: What is an S3 pre-signed URL?**
A: A temporary URL with the requester's **credentials embedded as a signature**. S3 validates the signature and grants access on behalf of the signer. Only works for the user who generated it.

**Q: A user needs to access a private S3 object without making it public. What should they use?**
A: A **pre-signed URL** — grants temporary, credential-scoped access without changing bucket permissions.

**Q: What bucket type should you use for standard workloads?**
A: **General Purpose** — Directory buckets are for a specific use case not covered on the exam.

**Q: You delete a folder in S3. What happens to the objects inside?**
A: All objects within the folder (key prefix) are **permanently deleted**.

---

## Quick Reference

```
Bucket type      = General Purpose (default; Directory not exam-relevant)
Block Public Access = ON by default (maximum security)
Default encryption = SSE-S3 (recommended, no extra cost)
Public URL       = plain object URL → Access Denied if bucket is private
Pre-signed URL   = temporary URL with credentials → works for private objects
Folders          = not real; just key prefixes with "/"
Delete folder    = deletes all objects inside it
```
