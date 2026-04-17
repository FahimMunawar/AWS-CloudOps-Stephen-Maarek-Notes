# 9. CloudFront Caching — Deep Dive

## Overview

CloudFront can cache based on **HTTP headers, cookies, and query string parameters**. The goal is always to **maximize the cache hit ratio** by forwarding only the minimum necessary values to the origin.

---

## The Core Rule

> **Fewer forwarded values = more cache hits = fewer origin requests**

Every header, cookie, or query string parameter you forward creates more unique cache keys, which means more cache misses.

---

## 1. Headers

HTTP requests include headers (e.g., `Host`, `Authorization`, `User-Agent`).

| Configuration | Behavior | Caching Performance |
|--------------|----------|---------------------|
| **Forward all headers** | All headers pass to origin | No caching (TTL must = 0) |
| **Forward whitelist** | Only specified headers pass to origin | Caching based on whitelisted header values |
| **Forward none (default)** | Only default headers pass; rest stripped | Best caching performance |

### Two Types of Header Settings (Important Distinction)

| Setting Type | Where Set | Purpose | Affects Cache? |
|-------------|-----------|---------|---------------|
| **Origin Custom Headers** | Origin settings | Add constant headers to every request going to origin (e.g., `X-From-CloudFront: true`) | **No** — constant, not used as cache key |
| **Cache Behavior Headers** | Behavior settings | Whitelist of headers to cache on and forward to origin | **Yes** — used to determine cache key |

---

## 2. Cookies

Cookies arrive as a header named `Cookie` containing key-value pairs (e.g., `username=john; lang=en; userID=123`).

| Configuration | Behavior | Caching Performance |
|--------------|----------|---------------------|
| **None (default)** | Cookies not forwarded; caching ignores cookies | Best |
| **Whitelist** | Only specified cookies forwarded; caching based on those values | Good |
| **All** | All cookies forwarded to origin | Worst |

---

## 3. Query String Parameters

Query strings appear in the URL (e.g., `?color=red&size=large`).

| Configuration | Behavior | Caching Performance |
|--------------|----------|---------------------|
| **None (default)** | Query strings stripped; not forwarded; not cached on | Best |
| **Whitelist** | Only specified params forwarded; caching based on those values | Good |
| **All** | All query strings forwarded to origin | Worst |

---

## 4. TTL (Time to Live)

Controls how long CloudFront keeps an object in cache before re-fetching from origin.

### TTL Control Methods

| Method | How |
|--------|-----|
| **Origin header** | Origin returns `Cache-Control: max-age=<seconds>` or `Expires` header |
| **Behavior setting** | Set Min TTL, Max TTL, Default TTL in the cache behavior |

### TTL Boundary Rules

```
Origin returns Cache-Control: max-age=X
        │
        ├── X < Min TTL  →  Min TTL is used
        ├── X > Max TTL  →  Max TTL is used
        └── X is missing →  Default TTL is used
        └── X within range → X is used
```

| Setting | Role |
|---------|------|
| **Min TTL** | Floor — cache at least this long even if origin says less |
| **Max TTL** | Ceiling — never cache longer than this regardless of origin |
| **Default TTL** | Used when origin sends no cache control header |

> Best practice: let the origin control TTL via `Cache-Control: max-age` and set Min/Max as guardrails.

---

## 5. Maximizing Cache Hit Ratio

### Configuration Best Practices

| Action | Impact |
|--------|--------|
| Forward no headers (or minimum required) | Reduces unique cache keys |
| Forward no cookies (or whitelist only required) | Reduces unique cache keys |
| Forward no query strings (or whitelist only required) | Reduces unique cache keys |
| Use `Cache-Control: max-age` at origin | Optimizes TTL per object |
| Monitor cache hit rate in CloudWatch | Catch degradation early |

### Separate Static and Dynamic Content

The most effective pattern is to use **two origins in the same distribution**:

```
CloudFront Distribution
        │
        ├── /static/* ──► S3 Bucket (no headers, no cookies, no QS)
        │                  → Maximum cache hits
        │
        └── /api/* ──────► ALB + EC2 / API Gateway + Lambda
                            (whitelist only needed headers/cookies)
                           → Optimized for application behavior
```

- Static content: no personalization needed → cache aggressively
- Dynamic content: personalized per user → cache only what's consistent

---

## Best Practices

✓ **Forward as few headers/cookies/query strings as possible** — each additional value creates more cache misses  
✓ **Use `Cache-Control: max-age` at the origin** — gives per-object TTL control  
✓ **Separate static and dynamic origins** — different caching strategies for each  
✓ **Whitelist only what your application actually needs** — review which params genuinely affect the response  
✓ **Monitor `CacheHitRate` in CloudWatch** — target 70–80%+ for a healthy distribution  

---

## SysOps Exam Focus

**Q1: "You configure CloudFront to forward all headers to the origin. What is the effect on caching?"**
- A) Caching improves because more context is passed
- B) There is no caching — all requests go to the origin; TTL must be set to 0
- C) Only dynamic content bypasses the cache
- D) Caching only applies to POST requests
- **Answer: B** — Forwarding all headers disables caching; every request is a cache miss

**Q2: "What is the difference between Origin Custom Headers and Cache Behavior headers in CloudFront?"**
- A) They are the same feature with different names
- B) Origin Custom Headers add constant values to all origin requests (not cached on); Cache Behavior headers are whitelisted and used as part of the cache key
- C) Origin Custom Headers are used for caching; Cache Behavior headers are passed to the origin only
- D) Origin Custom Headers apply to S3 origins only; Cache Behavior headers apply to HTTP origins
- **Answer: B** — Origin Custom Headers are constant (not cache keys); Cache Behavior headers are used to determine unique cache entries

**Q3: "Your CloudFront origin returns no Cache-Control header. Which TTL value will CloudFront use?"**
- A) Min TTL
- B) Max TTL
- C) Default TTL
- D) 0 seconds (no caching)
- **Answer: C** — When the origin sends no cache control header, the Default TTL from the behavior settings is applied

**Q4: "You want to maximize cache hit ratio for a website that has both static assets (images, CSS) and a dynamic API. What is the recommended approach?"**
- A) Set TTL to 0 for all content
- B) Forward all headers and cookies to ensure accuracy
- C) Separate static and dynamic origins in the same distribution — no headers/cookies for static, whitelist only for dynamic
- D) Use two separate CloudFront distributions
- **Answer: C** — Splitting into two origins with different cache behaviors optimizes each independently

**Q5: "An origin returns Cache-Control: max-age=7200 (2 hours). The cache behavior has Min TTL=86400 (1 day). What TTL is applied?"**
- A) 7200 seconds (origin header wins)
- B) 86400 seconds (Min TTL is applied because origin value is below the minimum)
- C) 0 seconds (conflict causes no caching)
- D) The average of 7200 and 86400
- **Answer: B** — When origin's max-age is less than Min TTL, the Min TTL takes precedence

---

## Quick Reference

```
Three cache key components:
  Headers         →  none / whitelist / all (none = best caching)
  Cookies         →  none / whitelist / all (none = best caching)
  Query strings   →  none / whitelist / all (none = best caching)

TTL:
  Cache-Control: max-age=X   →  preferred method at origin
  Min TTL / Max TTL          →  guardrails in behavior settings
  Default TTL                →  used when origin sends no header

Origin Custom Headers   →  constant headers added to all origin requests (NOT cache keys)
Cache Behavior headers  →  whitelisted headers used AS cache keys

Best practice: separate static (/static/* → S3) and dynamic (/api/* → ALB) origins
```

---

**File: 9_CloudFront_Caching_Deep_Dive.md**
**Status: SysOps-focused, exam-ready, concise format**
