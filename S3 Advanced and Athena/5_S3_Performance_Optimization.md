# 5 — S3 Performance Optimization

## Overview

S3 has strong baseline performance out of the box, and three key techniques to further optimize upload and download speeds: Multi-Part Upload, Transfer Acceleration, and Byte-Range Fetches.

---

## Baseline Performance

| Metric | Value |
|---|---|
| **First byte latency** | 100–200 milliseconds |
| **PUT/COPY/POST/DELETE** | **3,500 requests/second/prefix** |
| **GET/HEAD** | **5,500 requests/second/prefix** |
| **Number of prefixes** | Unlimited |

### What is a "Prefix"?

A prefix is everything in the object key **between the bucket name and the file name**.

```
Bucket: my-bucket
Object key: /folder1/sub1/file  →  prefix = /folder1/sub1
Object key: /folder1/sub2/file  →  prefix = /folder1/sub2
Object key: /folder2/sub1/file  →  prefix = /folder2/sub1
Object key: /folder2/sub2/file  →  prefix = /folder2/sub2
```

Each prefix gets its own independent throughput quota.

### Scaling with Multiple Prefixes

| Prefixes | Max GET/HEAD per second |
|---|---|
| 1 | 5,500 |
| 2 | 11,000 |
| 4 | **22,000** |

> Spread reads evenly across prefixes to multiply throughput — no extra configuration needed.

---

## Optimization 1 — Multi-Part Upload (Uploads)

| Rule | Detail |
|---|---|
| **Recommended for** | Files > **100 MB** |
| **Required for** | Files > **5 GB** |
| **How it works** | File split into parts → parts uploaded in **parallel** → S3 reassembles |

```
Large file
    │
    ├── Part 1 ──►┐
    ├── Part 2 ──►┤ S3 (parallel upload) → S3 reassembles into complete file
    ├── Part 3 ──►┤
    └── Part N ──►┘
```

- Maximizes available bandwidth by parallelizing transfers
- If one part fails, only that part needs to be re-uploaded

---

## Optimization 2 — S3 Transfer Acceleration (Upload & Download)

Speeds up transfers by routing through an **AWS Edge Location** first, then using the fast private AWS backbone network.

```
User (USA) ──public internet──► Edge Location (USA) ──private AWS network──► S3 (Australia)
           (short hop)                                (long, fast hop)
```

| Feature | Detail |
|---|---|
| **What it does** | Minimizes public internet travel; maximizes private AWS network |
| **Edge locations** | 200+ worldwide (more than AWS regions) |
| **Compatible with** | Multi-part upload |
| **Use case** | Uploading/downloading across geographic regions |

- Best benefit when source and destination are **far apart geographically**
- Small additional cost per GB transferred

---

## Optimization 3 — S3 Byte-Range Fetches (Downloads)

Parallelize **GET** requests by requesting specific byte ranges of a file independently.

```
Large file in S3
    │
    ├── GET bytes 0–1MB    ──►┐
    ├── GET bytes 1–2MB    ──►┤ (parallel requests) → merge on client
    ├── GET bytes 2–3MB    ──►┤
    └── GET bytes 3–4MB    ──►┘
```

### Two Use Cases

| Use Case | Description |
|---|---|
| **Speed up downloads** | Parallelize GETs across byte ranges → faster overall download |
| **Retrieve partial file** | Request only the bytes you need (e.g., first 50 bytes = file header) |

- Better resilience: if one range request fails, retry only that small range
- Example: request only the first 50 bytes to read a file header without downloading the entire file

---

## Summary: Which Technique for What

| Goal | Technique |
|---|---|
| Speed up **large file uploads** | Multi-Part Upload |
| Speed up **long-distance uploads/downloads** | Transfer Acceleration |
| Speed up **large file downloads** | Byte-Range Fetches |
| Read **only part of a file** | Byte-Range Fetches |
| **Increase throughput** without any configuration | Spread objects across multiple prefixes |

---

## SysOps Exam Q&A

**Q: What is the S3 throughput limit per prefix?**
A: **3,500 PUT/COPY/POST/DELETE** and **5,500 GET/HEAD** requests per second per prefix.

**Q: You have a bucket with 4 prefixes and spread reads evenly. What is the max GET throughput?**
A: 4 × 5,500 = **22,000 GET/HEAD requests per second**.

**Q: When is multi-part upload required vs recommended?**
A: **Required** for files > 5 GB. **Recommended** for files > 100 MB.

**Q: A user in Europe is uploading large files to an S3 bucket in Asia-Pacific and experiencing slow speeds. What should you enable?**
A: **S3 Transfer Acceleration** — routes the upload through a nearby edge location then over the fast private AWS network to the target region.

**Q: Is S3 Transfer Acceleration compatible with multi-part upload?**
A: **Yes** — they can be used together.

**Q: How do you speed up downloading a very large S3 file?**
A: Use **S3 Byte-Range Fetches** — parallelize GET requests by requesting different byte ranges simultaneously.

**Q: You only need to read the first 100 bytes of an S3 object (file header). How do you avoid downloading the entire file?**
A: Issue a **byte-range GET request** for bytes 0–99 — S3 returns only that portion of the object.

---

## Quick Reference

```
Baseline:
  PUT/COPY/POST/DELETE = 3,500 req/sec/prefix
  GET/HEAD             = 5,500 req/sec/prefix
  Prefix               = path between bucket and filename
  Scale tip            = spread objects across prefixes to multiply throughput

Multi-Part Upload:
  Recommended > 100 MB, Required > 5 GB
  Parallel part upload → S3 reassembles

Transfer Acceleration:
  User → Edge Location (public internet) → S3 (private AWS network)
  Best for cross-region/cross-continent transfers
  Compatible with multi-part upload

Byte-Range Fetches:
  Parallel GET requests for different byte ranges → faster downloads
  Also: retrieve partial file (e.g., header bytes only)
```
