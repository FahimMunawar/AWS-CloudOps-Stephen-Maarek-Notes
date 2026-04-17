# 2. CloudFront with S3 — Hands-On

## Overview

A practical walkthrough of creating a CloudFront distribution backed by a private S3 bucket, verifying that content is accessible through CloudFront without making the S3 bucket public.

---

## Step 1: Create and Populate the S3 Bucket

1. S3 Console → **Create bucket**
   - Name: `demo-cloudfront-<yourname>`
   - Leave all defaults (block public access **ON** — bucket stays private)
2. Upload files: `index.html`, `beach.jpeg`, `coffee.jpg`

**Verify bucket is private:**
- Click an object → copy **Object URL** → paste in browser → **Access Denied** (expected)
- The pre-signed URL (via "Open" button) works, but requires signed authentication

---

## Step 2: Create a CloudFront Distribution

**Console:** CloudFront → **Create distribution**

### Plan Selection

| Plan | Best For |
|------|----------|
| **Free** | Learning, basic CDN, DNS protection, geo-blocking, global CDN |
| Business / Enterprise | Edge KV store, advanced DDoS, WAF, uptime SLA, WordPress protection |
| Pay as you go | Variable traffic, pay per request |

> For this lab: select **Free plan**.

### Distribution Configuration

| Setting | Value |
|---------|-------|
| Distribution name | `demo-new-cloudfront` |
| Origin type | **Amazon S3** |
| S3 bucket | `demo-cloudfront-<yourname>` |
| Private S3 access | **Yes — allow CloudFront to access private bucket** |
| Cache settings | Use recommended S3 cache settings |
| WAF | Not required for this lab |
| Custom domain | Optional (skip for lab) |

Click **Create distribution**.

---

## Step 3: Verify the Auto-Generated Bucket Policy

After distribution creation, CloudFront **automatically updates the S3 bucket policy**:

```
S3 → Bucket → Permissions → Bucket Policy
```

The policy allows CloudFront to call `s3:GetObject` on the bucket:

```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "cloudfront.amazonaws.com"
  },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::demo-cloudfront-<yourname>/*",
  "Condition": {
    "StringEquals": {
      "AWS:SourceArn": "arn:aws:cloudfront::<account-id>:distribution/<distribution-id>"
    }
  }
}
```

- The bucket remains **private** — only CloudFront can access it
- This is **OAC (Origin Access Control)** in action

---

## Step 4: Access Content via CloudFront

Copy the CloudFront **domain name** (e.g., `d1234abcd.cloudfront.net`) and test:

| URL | Expected Result |
|-----|----------------|
| `https://<domain>/` | Access Denied (no default root object set) |
| `https://<domain>/coffee.jpg` | ✓ Image loads |
| `https://<domain>/beach.jpeg` | ✓ Image loads |
| `https://<domain>/index.html` | ✓ Full page with images loads |

**Caching in action:** Access `beach.jpeg` twice — the second load is nearly instantaneous because CloudFront served it from the edge cache.

---

## What Happened Under the Hood

```
Browser → CloudFront Edge Location
              │
              ├── Cache hit?  → Serve immediately (fast)
              │
              └── Cache miss? → Fetch from S3 (private, via OAC)
                                       → Cache at edge
                                       → Serve to user
```

The S3 bucket **never needs to be public**. CloudFront fetches privately using OAC and caches at the edge.

---

## Best Practices

✓ **Never make S3 public when using CloudFront** — use OAC + bucket policy instead  
✓ **Let CloudFront auto-generate the bucket policy** — it creates the correct OAC-based policy automatically  
✓ **Set a default root object** (`index.html`) to avoid Access Denied on the bare domain URL  
✓ **Use the free plan for learning** — sufficient for basic CDN, geo-blocking, and global distribution  

---

## SysOps Exam Focus

**Q1: "You set up CloudFront with an S3 origin. Users can access files via CloudFront but get Access Denied when using the S3 object URL directly. Is this correct behavior?"**
- A) No — S3 must be public for CloudFront to work
- B) Yes — OAC restricts S3 access to CloudFront only; direct S3 access is intentionally blocked
- C) No — the bucket policy is misconfigured
- D) Yes — but only for encrypted objects
- **Answer: B** — OAC keeps S3 private; only CloudFront can fetch content from the bucket

**Q2: "After creating a CloudFront distribution with an S3 origin and private access enabled, what is automatically created in S3?"**
- A) A new IAM role for CloudFront
- B) A bucket policy allowing the CloudFront distribution to call s3:GetObject
- C) An S3 replication rule to sync content to edge locations
- D) A CORS configuration for CloudFront
- **Answer: B** — CloudFront automatically adds a bucket policy granting OAC-based access

**Q3: "A user accesses an image via CloudFront for the first time and it loads slowly. The second access is instant. Why?"**
- A) The S3 bucket needed time to warm up
- B) The first request fetched from the S3 origin and cached it at the edge; subsequent requests are served from cache
- C) CloudFront compressed the image on second access
- D) The user's browser cached it locally
- **Answer: B** — First request = cache miss (fetch from S3); second request = cache hit (served from edge)

---

## Quick Reference

```
Setup order:
  1. Create private S3 bucket → upload files
  2. Create CloudFront distribution → origin: S3, enable private access
  3. CloudFront auto-generates bucket policy (OAC)
  4. Access files via CloudFront domain name: <domain>/filename

Bucket policy: auto-created by CloudFront, allows s3:GetObject from the distribution only
Access pattern: direct S3 URL → Access Denied | CloudFront URL → works
Caching: first request fetches from S3, subsequent requests served from edge cache
```

---

**File: 2_CloudFront_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
