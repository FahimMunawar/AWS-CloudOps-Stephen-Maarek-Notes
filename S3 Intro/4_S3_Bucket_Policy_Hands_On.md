# 4 — S3 Bucket Policy — Hands-On

## Overview

Step-by-step walkthrough of creating a bucket policy to allow public read access to all objects in an S3 bucket.

---

## Steps to Make a Bucket Public

### Step 1 — Disable Block Public Access

1. S3 → Bucket → **Permissions** tab
2. **Block public access** → Edit
3. Uncheck **"Block all public access"** → Save
4. Confirm with `confirm`

> **Warning:** Only do this when you intentionally want a public bucket. Enabling public access on a bucket containing sensitive company data causes data leaks.

After this step, the Permissions overview shows: *"Objects can be public"*

### Step 2 — Create a Bucket Policy

1. Permissions tab → scroll to **Bucket policy** → Edit
2. Use the **Policy Generator** or **Policy Examples** in the documentation

---

## Using the AWS Policy Generator

| Field | Value |
|---|---|
| **Type of Policy** | S3 Bucket Policy |
| **Effect** | Allow |
| **Principal** | `*` (everyone) |
| **AWS Service** | Amazon S3 |
| **Actions** | `s3:GetObject` |
| **Amazon Resource Name (ARN)** | `arn:aws:s3:::your-bucket-name/*` |

### ARN Format — Critical Detail
```
Bucket ARN:          arn:aws:s3:::example-bucket
Policy Resource ARN: arn:aws:s3:::example-bucket/*
                                                 ^^
                                      slash + star = all objects
```

`s3:GetObject` applies to **objects**, not the bucket itself. The `/*` at the end targets all objects inside the bucket.

### Generated Policy
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

### Step 3 — Save the Policy
- Paste the generated JSON into the Bucket policy editor
- Remove any extra whitespace that may cause validation errors
- Click **Save changes**

---

## Result

- Bucket policy is now applied
- The plain **Object URL** now works publicly:
  ```
  https://example-bucket.s3.eu-west-1.amazonaws.com/coffee.jpg  → ✅ Accessible
  ```
- All objects in the bucket are now publicly readable

---

## Common Mistake: Wrong Resource ARN

| ARN | Effect |
|---|---|
| `arn:aws:s3:::example-bucket` | Targets the **bucket** — GetObject won't work here |
| `arn:aws:s3:::example-bucket/*` | Targets **objects** inside bucket — correct for GetObject |

Always append `/*` when writing policies for object-level actions (`GetObject`, `PutObject`, etc.).

---

## Best Practices

- Only disable Block Public Access when you **intentionally** need a public bucket
- Use the **Policy Generator** to avoid JSON syntax errors
- After saving, verify in Permissions tab that the policy shows as applied
- For most production buckets, keep Block Public Access **ON** and use pre-signed URLs instead

---

## SysOps Exam Q&A

**Q: What two steps are required to make S3 objects publicly accessible?**
A: 1) **Disable Block Public Access** on the bucket, and 2) **Attach a Bucket Policy** that allows `s3:GetObject` for `Principal: *`.

**Q: A bucket policy with `s3:GetObject` is applied to `arn:aws:s3:::my-bucket` but public access still fails. Why?**
A: The resource ARN is missing `/*`. `s3:GetObject` is an object-level action and requires `arn:aws:s3:::my-bucket/*` to target objects.

**Q: Block Public Access is still enabled but a bucket policy allows public reads. Is the bucket public?**
A: **No** — Block Public Access overrides the bucket policy. You must disable Block Public Access first.

**Q: What is the correct Principal value in a bucket policy to allow anyone to read objects?**
A: `"Principal": "*"` — the wildcard grants access to all users including unauthenticated public users.

---

## Quick Reference

```
Step 1: Disable Block Public Access (Permissions tab → Edit)
Step 2: Add Bucket Policy with:
        Principal: *
        Action:    s3:GetObject
        Resource:  arn:aws:s3:::bucket-name/*   ← must include /*

Bucket ARN alone      → targets bucket (not objects)
Bucket ARN + /*       → targets all objects inside bucket
Public URL works only after BOTH steps are complete
```
