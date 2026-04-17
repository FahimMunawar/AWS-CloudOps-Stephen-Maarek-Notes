# 5. CloudFront Origin Shield

## Overview

**Origin Shield** is an additional, centralized caching layer between CloudFront edge locations and the origin. It acts as a **cache for your cache**, reducing the number of requests that reach the origin and improving origin availability.

---

## Without Origin Shield

```
Edge Location A ──► S3 (fetches file, caches locally)
Edge Location B ──► S3 (fetches same file, caches locally)
Edge Location C ──► S3 (fetches same file, caches locally)

Result: 3 requests hit S3 for the same object
```

Each edge location independently fetches from the origin on a cache miss — origin sees duplicated requests.

---

## With Origin Shield

```
Edge Location A ──┐
Edge Location B ──┼──► Origin Shield ──► S3 (fetches file once)
Edge Location C ──┘         │
                      Caches the file
                      Serves A, B, C from cache
```

Origin Shield consolidates cache misses from all edge locations. The origin only receives **one request** instead of many.

---

## Key Properties

| Feature | Detail |
|---------|--------|
| **Position** | Between edge locations and the origin |
| **Purpose** | Reduce origin load and cost; improve availability |
| **How to enable** | A single setting on the CloudFront distribution origin configuration |
| **Scalability** | Managed by AWS — handles high request volumes |
| **Additional cost** | Yes — Origin Shield incurs an incremental charge |

---

## When to Use Origin Shield

- Origins with **limited capacity** (custom servers, on-premises, throttled APIs)
- Origins where **per-request cost is high** (e.g., paid APIs, expensive compute)
- High-traffic distributions where many edge locations serve the same popular objects
- Any scenario where you want to minimize origin load

---

## Best Practices

✓ **Enable Origin Shield for origins with limited throughput** — prevents overload during traffic spikes  
✓ **Choose the Origin Shield region closest to your origin** — reduces latency between Origin Shield and the origin  
✓ **Use alongside edge caching** — Origin Shield adds a layer; it does not replace edge caching  

---

## SysOps Exam Focus

**Q1: "Multiple CloudFront edge locations are each independently fetching the same object from your origin, causing high load. What feature should you enable?"**
- A) CloudFront cache invalidation
- B) Origin Shield
- C) S3 Transfer Acceleration
- D) Origin Access Control
- **Answer: B** — Origin Shield acts as a centralized cache between all edge locations and the origin, consolidating requests

**Q2: "How would you describe CloudFront Origin Shield?"**
- A) A WAF layer in front of CloudFront
- B) A centralized additional caching layer between edge locations and the origin — a cache for your cache
- C) A feature that replicates origin data to all edge locations
- D) A security group that protects the origin
- **Answer: B** — Origin Shield is a middle-tier cache that reduces duplicate origin fetches from multiple edge locations

**Q3: "Origin Shield is placed between which two components?"**
- A) The user and the edge location
- B) The edge location and the origin
- C) The origin and Amazon S3
- D) Route 53 and CloudFront
- **Answer: B** — Origin Shield sits between CloudFront edge locations and the origin, intercepting cache misses

---

## Quick Reference

```
Without Origin Shield:
  N edge locations → N origin requests for same object

With Origin Shield:
  N edge locations → 1 Origin Shield → 1 origin request

Position: Edge Locations → Origin Shield → Origin
Use when: origin has limited capacity or high per-request cost
Enable:   Distribution → Origin settings → Enable Origin Shield → select region
```

---

**File: 5_CloudFront_Origin_Shield.md**
**Status: SysOps-focused, exam-ready, concise format**
