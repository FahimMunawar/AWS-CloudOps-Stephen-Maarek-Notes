# 21 — Route 53: S3 Website Integration

## Overview

Host a static website on S3 and access it via a custom domain using Route 53 Alias records.

> **HTTP only** — S3 websites do not support HTTPS. Use CloudFront for HTTPS with a custom domain.

---

## Critical Requirement

**S3 bucket name must exactly match the Route 53 record name.**

```
Route 53 record: blog.example.com
S3 bucket name:  blog.example.com  ← must be identical
```

> This is an exam question — if the bucket name doesn't match the record, it won't work.

---

## Setup Steps

1. **Create S3 bucket** — name must match the target domain (e.g., `blog.example.com`)
2. **Allow public access** — uncheck "Block all public access"
3. **Upload files** — `index.html`, assets, etc. — grant public read on upload
4. **Enable static website hosting** — Properties → Static website hosting → Enable
   - Index document: `index.html`
5. **Create Route 53 Alias record**:
   - Record name: `blog` (becomes `blog.example.com`)
   - Type: **A**
   - Toggle: **Alias**
   - Route traffic to: **S3 website endpoint**
   - Region: select the region where the bucket was created
   - Endpoint: select your bucket (matches by name)

---

## How It Works

```
User → blog.example.com
  → Route 53 Alias A record
    → S3 website endpoint (s3-website.<region>.amazonaws.com)
      → S3 bucket named blog.example.com
        → serves index.html
```

---

## Limitations

| Feature | S3 Website |
|---|---|
| HTTP | Supported |
| HTTPS | **Not supported** — use CloudFront |
| Custom domain | Supported via Route 53 Alias |

---

## Quick Reference

```
S3 website + Route 53:
  Bucket name = Route 53 record name (EXACT match required)
  Bucket: public access + static website hosting enabled
  Record: A (Alias) → S3 website endpoint → select region → select bucket

HTTP only — HTTPS requires CloudFront in front of S3
```
