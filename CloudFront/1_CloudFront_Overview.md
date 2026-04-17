# 1. CloudFront Overview

## Overview

**Amazon CloudFront** is a Content Delivery Network (CDN). It improves read performance by caching content at **edge locations** around the world, reducing latency for users regardless of where they are. Whenever you see "CDN" in an exam question — think CloudFront.

---

## How CloudFront Works

```
User (Los Angeles)
        ↓
Edge Location (Los Angeles)  ←── Cache hit? Serve directly
        │                          Cache miss?  ↓
        └──────────────────► Origin (e.g., S3 in Sydney)
                                    ↓
                            Content fetched and cached
                            at edge for future requests
```

- First request: edge location **fetches from origin**, caches the result
- Subsequent requests: served **directly from cache** (no round-trip to origin)
- Cache TTL is typically ~1 day

---

## Key Benefits

| Benefit | Detail |
|---------|--------|
| **Lower latency** | Content cached near users via 216+ global points of presence |
| **Improved user experience** | Users served from the nearest edge location |
| **DDoS protection** | Distributed globally — attacks are absorbed across edge locations |
| **Shield + WAF integration** | Works with AWS Shield and Web Application Firewall for deeper security |

---

## CloudFront Origins

An **origin** is the backend that CloudFront fetches content from.

| Origin Type | Detail |
|-------------|--------|
| **S3 Bucket** | Distribute and cache static files; or use CloudFront to upload files to S3; secured with **Origin Access Control (OAC)** |
| **VPC Origin** | Private ALB, private NLB, or private EC2 instances inside a VPC |
| **Custom HTTP Origin** | Any public HTTP backend — public ALB, EC2, static S3 website (must enable static website hosting on bucket) |

---

## CloudFront + S3: Security with OAC

```
User → CloudFront Edge → (private network) → S3 Bucket
                                              (secured via OAC + bucket policy)
```

- The S3 bucket **does not need to be public**
- **Origin Access Control (OAC)** grants CloudFront permission to read from S3
- S3 bucket policy is updated to allow only CloudFront to access the bucket

---

## CloudFront vs S3 Cross-Region Replication

| Feature | CloudFront | S3 Cross-Region Replication |
|---------|------------|----------------------------|
| Network | Global Edge (216+ PoPs) | Specific regions you configure |
| Caching | Yes (~1 day TTL) | No caching — near real-time sync |
| Coverage | Every edge location | Only regions you set up |
| Access type | Read (cached) | Read-only |
| Best for | **Static content** available worldwide | **Dynamic content** at low latency in a few regions |

> **Key distinction:** CloudFront = CDN (cache globally). S3 Replication = copy bucket to specific regions in near real-time.

---

## Best Practices

✓ **Use CloudFront for static assets** — images, JS, CSS, video — benefit most from caching  
✓ **Secure S3 origins with OAC** — never make the S3 bucket public when using CloudFront  
✓ **Use CloudFront for DDoS mitigation** — distributed edge network absorbs attack traffic  
✓ **Use S3 replication (not CloudFront) when real-time consistency matters** — CloudFront cache may be stale  

---

## SysOps Exam Focus

**Q1: "A user in Brazil is accessing a static website hosted in an S3 bucket in Sydney. How can you reduce their latency?"**
- A) Enable S3 Transfer Acceleration
- B) Set up CloudFront with the S3 bucket as origin
- C) Replicate the bucket to a South American region
- D) Use Route 53 latency-based routing
- **Answer: B** — CloudFront caches the content at an edge location near Brazil, eliminating the round-trip to Sydney

**Q2: "What does Origin Access Control (OAC) do in a CloudFront + S3 setup?"**
- A) Encrypts objects in S3 before serving them through CloudFront
- B) Allows CloudFront to access a private S3 bucket without making the bucket public
- C) Replicates the S3 bucket to multiple regions
- D) Enables CloudFront to cache S3 objects indefinitely
- **Answer: B** — OAC grants CloudFront permission to read from S3, so the bucket can remain private

**Q3: "When should you use S3 Cross-Region Replication instead of CloudFront?"**
- A) When you need to cache static assets globally
- B) When content must be available at low latency in a few specific regions and updated in near real-time
- C) When you need DDoS protection
- D) When users are spread across 216+ edge locations
- **Answer: B** — CRR is for near real-time replication to specific regions; CloudFront is for global caching of static content

**Q4: "Which CloudFront origin type is used for applications running inside a private VPC subnet?"**
- A) S3 Bucket origin
- B) Custom HTTP origin
- C) VPC origin
- D) Regional Edge Cache
- **Answer: C** — VPC origin connects CloudFront to private ALB, NLB, or EC2 instances inside a VPC

**Q5: "A customer says their content is outdated even though the S3 bucket was updated. Why?"**
- A) OAC is blocking the update
- B) CloudFront is serving cached content with a TTL that hasn't expired yet
- C) S3 replication lag is causing the delay
- D) The edge location is in a different region than the bucket
- **Answer: B** — CloudFront caches content at edge locations (typically ~1 day); updates aren't served until the TTL expires or the cache is invalidated

---

## Quick Reference

```
CloudFront = CDN — caches content at 216+ edge locations globally

Origins:
  S3 bucket          →  static files, secured with OAC
  VPC origin         →  private ALB/NLB/EC2
  Custom HTTP        →  public ALB, EC2, static S3 website

vs S3 CRR:
  CloudFront  →  global cache, ~1 day TTL, static content
  S3 CRR      →  near real-time, specific regions, dynamic content

Security:
  OAC + bucket policy  →  keep S3 bucket private
  Shield + WAF         →  DDoS and application-layer protection
```

---

**File: 1_CloudFront_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
