# 4. CloudFront Cache Invalidation

## Overview

When you update content at the origin (e.g., S3), CloudFront edge locations continue to serve the old cached version until the TTL expires. **Cache Invalidation** forces edge locations to evict specific cached objects immediately, without waiting for TTL expiry.

---

## The Problem

```
S3 Bucket updated (new index.html + new images)
        ↓
CloudFront edge locations still have old versions
        ↓
Users see outdated content until TTL expires (e.g., 1 day)
```

---

## The Solution: CloudFront Invalidation

```
Admin creates invalidation:
  Path 1: /index.html       (specific file)
  Path 2: /images/*         (all images)
        ↓
CloudFront notifies all edge locations
        ↓
Edge locations evict matching cached objects
        ↓
Next user request: cache miss → fetch fresh content from origin
        ↓
Updated content served to users
```

---

## Invalidation Path Syntax

| Path | What Gets Invalidated |
|------|-----------------------|
| `/*` | Every cached object across all edge locations |
| `/index.html` | One specific file |
| `/images/*` | All objects under the `/images/` path |
| `/css/style.css` | One specific nested file |

---

## How to Create an Invalidation

**Console:** CloudFront → select distribution → **Invalidations** tab → **Create invalidation**

**CLI:**
```bash
aws cloudfront create-invalidation \
  --distribution-id EDFDVBD6EXAMPLE \
  --paths "/index.html" "/images/*"
```

---

## Key Points

- Invalidation applies to **all edge locations** globally — not just one
- Each `/*` invalidation counts as one invalidation request (first 1,000/month free; charged after)
- Invalidation removes files from cache — the **next request** re-fetches from origin
- Useful after deployments, content updates, or emergency removals

---

## Best Practices

✓ **Invalidate specific paths** rather than `/*` when possible — more efficient and avoids unnecessary cache purges  
✓ **Use versioned filenames** (e.g., `style.v2.css`) as an alternative to invalidation — new filename = new cache key = no stale content  
✓ **Run invalidations after every deployment** that changes static assets  
✓ **Use `/*` sparingly** — it purges everything, causing a surge of origin requests as the cache rebuilds  

---

## SysOps Exam Focus

**Q1: "You updated files in your S3 bucket but CloudFront users still see the old content. The TTL is set to 1 day. What is the fastest fix?"**
- A) Delete and recreate the CloudFront distribution
- B) Create a CloudFront cache invalidation for the affected paths
- C) Change the TTL to 0 seconds
- D) Re-upload the files with different names
- **Answer: B** — Cache invalidation forces edge locations to evict the stale cache immediately

**Q2: "What does the invalidation path `/images/*` do?"**
- A) Deletes all images from the S3 bucket
- B) Removes all cached objects under the /images/ path from every edge location
- C) Blocks users from accessing the /images/ path
- D) Reduces the TTL for /images/ objects to 1 minute
- **Answer: B** — Invalidation evicts matching cached objects from all edge locations globally

**Q3: "What is a best practice alternative to running cache invalidations on every deployment?"**
- A) Set TTL to 0 for all objects
- B) Use versioned file names (e.g., app.v2.js) so each version has a unique cache key
- C) Disable caching entirely
- D) Use S3 versioning instead of CloudFront
- **Answer: B** — Versioned file names create new cache keys automatically, eliminating the need for invalidations

---

## Quick Reference

```
Invalidation = force edge locations to evict cached objects

Path syntax:
  /*           →  invalidate everything (use sparingly)
  /index.html  →  specific file
  /images/*    →  all objects under a path

Effect: next request after invalidation = cache miss → fetched from origin
Cost: first 1,000 invalidation paths/month free; charged after

Alternative: versioned filenames (app.v2.js) = no invalidation needed
```

---

**File: 4_CloudFront_Cache_Invalidation.md**
**Status: SysOps-focused, exam-ready, concise format**
