# 3. CloudFront Caching

## Overview

CloudFront caches objects at each **edge location**. Every cached object is identified by a **cache key**. The goal is to maximize the **cache hit ratio** — serving as many requests as possible from the edge without reaching the origin.

---

## Cache Flow

```
Client request
        ↓
Edge Location
        ├── Cache HIT  → Return cached object to client immediately
        │
        └── Cache MISS → Forward to origin
                              ↓
                        Get response from origin
                              ↓
                        Cache response at edge (with TTL)
                              ↓
                        Return response to client
```

Next request for the same cache key → cache hit, origin not contacted.

---

## Cache Key

- A **unique identifier** for each cached object at an edge location
- By default: the **hostname + URL path** (e.g., `example.com/images/beach.jpg`)
- Can be extended to include: **HTTP headers**, **cookies**, **query strings**

---

## Cache Policy

Controls what gets cached and for how long.

| Control | Options |
|---------|---------|
| **Cache based on** | HTTP headers, cookies, query strings |
| **TTL (Time to Live)** | 0 seconds to 1 year |
| **TTL headers** | `Cache-Control` or `Expires` headers from origin |
| **Policy type** | Custom (create your own) or AWS managed (predefined) |

> More cache key components = more unique cache entries = **lower cache hit ratio**.
> Keep the cache key as minimal as needed for correctness.

---

## Cache Invalidation

Force CloudFront to evict cached objects before their TTL expires.

- API: **`CreateInvalidation`**
- Scope: invalidate specific paths (e.g., `/images/*`) or all objects (`/*`)
- Use when you update content at the origin and need it served immediately

```
Update file in S3
        ↓
CloudFront still serves old cached version until TTL expires
        ↓
Run CreateInvalidation on /* or /specific-path
        ↓
Next request fetches fresh content from origin
```

---

## Best Practices

✓ **Keep cache keys minimal** — only include headers/cookies/query strings that genuinely affect the response  
✓ **Set appropriate TTLs** — long TTL = fewer origin requests; short TTL = fresher content  
✓ **Use Cache-Control headers at the origin** to control TTL per object type  
✓ **Use CreateInvalidation after deployments** to force fresh content without waiting for TTL expiry  
✓ **Use managed cache policies** (predefined by AWS) to get started quickly  

---

## SysOps Exam Focus

**Q1: "You updated an image in your S3 bucket but CloudFront is still serving the old version. What should you do?"**
- A) Delete the CloudFront distribution and recreate it
- B) Make the S3 bucket public
- C) Run a CreateInvalidation on the affected path
- D) Wait for the TTL to expire — nothing else can be done
- **Answer: C** — CreateInvalidation forces CloudFront to evict the cached object and fetch fresh content from the origin

**Q2: "What is a CloudFront cache key?"**
- A) An IAM key used to authenticate requests to CloudFront
- B) A unique identifier for each cached object, by default based on hostname and URL path
- C) The KMS key used to encrypt cached content
- D) The API key required to call CreateInvalidation
- **Answer: B** — Cache key = unique identifier per cached object, defaults to hostname + path

**Q3: "Adding query strings and cookies to the cache key will have what effect?"**
- A) Increase the cache hit ratio
- B) Decrease the cache hit ratio because more unique keys are generated
- C) No effect on caching
- D) Improve security
- **Answer: B** — More cache key components = more unique entries = lower hit ratio (more cache misses)

**Q4: "How can you control how long CloudFront caches an object?"**
- A) Only by setting the TTL in the CloudFront distribution settings
- B) Via the TTL in the cache policy, or by setting Cache-Control / Expires headers at the origin
- C) By configuring the S3 bucket lifecycle policy
- D) Via an IAM policy on the edge location
- **Answer: B** — TTL can be controlled via the cache policy (0s to 1 year) or via Cache-Control / Expires headers from the origin

---

## Quick Reference

```
Cache key:      hostname + URL path (default)
                + headers / cookies / query strings (optional)

TTL:            0 seconds to 1 year
                Set via cache policy OR Cache-Control / Expires header

Invalidation:   CreateInvalidation API
                Paths: /specific/path or /* (all objects)
                Use after origin updates for immediate refresh

Goal:           Maximize cache hit ratio
                = minimize requests reaching the origin
```

---

**File: 3_CloudFront_Caching.md**
**Status: SysOps-focused, exam-ready, concise format**
