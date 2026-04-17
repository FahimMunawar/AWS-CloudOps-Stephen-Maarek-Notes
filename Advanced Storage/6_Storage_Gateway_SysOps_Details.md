# 6 — Storage Gateway: SysOps Exam Details

## Overview

Operational details about Storage Gateway that are specifically tested on the SysOps exam: POSIX compliance, reboot procedures, activation, and cache monitoring.

---

## File Gateway: POSIX Compliance

- File Gateway is **POSIX compliant** (Linux file system standard)
- Object metadata in S3 stores Linux file attributes:
  - **Ownership** (user/group)
  - **Permissions** (read/write/execute)
  - **Timestamps**
- This allows Linux applications to use File Gateway as a native file system with correct metadata behaviour

---

## Rebooting the Storage Gateway VM

Different procedure depending on gateway type:

### File Gateway — Simple
```
Restart the Storage Gateway VM directly
```

### Volume Gateway or Tape Gateway — Must Follow Order
```
1. Stop the Storage Gateway Service
   (via Console, VM local console, or Storage Gateway API)
        │
        ▼
2. Reboot the Storage Gateway VM
        │
        ▼
3. Start the Storage Gateway Service
   (via Console or API)
```

> **Exam tip:** For Volume/Tape Gateway, stopping the service before rebooting prevents data corruption.

---

## Activating a Storage Gateway

Two methods to get an **Activation Key**:

### Method 1 — Gateway VM CLI (Recommended)
- Start the Gateway VM
- Select: `0: Get Activation Key`
- Enter the key in the Storage Gateway Console

### Method 2 — Web Request on Port 80
- Make an HTTP request to the Gateway VM on **Port 80**
- Requires Port 80 to be open on the Gateway VM
- Console uses this request to activate the gateway

---

## Activation Failure — Common Causes

| Cause | Fix |
|---|---|
| **Port 80 not open** | Open Port 80 on the Gateway VM security group (Method 2 only) |
| **Time sync issue** | Ensure Gateway VM is synced to an **NTP (Network Time Protocol) server** — time must not be too far off |

> If the Gateway VM clock is significantly different from the NTP server time, activation will fail.

---

## Volume Gateway Cache Mode: Monitoring & Tuning

In **Cached Volume** mode, only recently accessed data is stored locally. Monitor cache efficiency via CloudWatch.

### Key CloudWatch Metrics

| Metric | What It Measures | Target |
|---|---|---|
| **CacheHitPercent** | % of read requests served from local cache | **High** (e.g., 80%+) = efficient cache |
| **CachePercentUsed** | % of cache storage currently in use | Not too high — avoid filling cache completely |

### Cache Hit vs Cache Miss

```
Cache Hit  → data served from local gateway cache → low latency, stays in data center
Cache Miss → data fetched from S3 → higher latency, goes to cloud
```

A low `CacheHitPercent` (e.g., 30%) means most reads go to S3 — high latency, poor performance.

### Fix: Increase Cache Disk Size

If `CacheHitPercent` is consistently low or `CachePercentUsed` is consistently high:

```
1. Clone the current cache volume to a larger disk
2. Select the new (larger) disk as the cache volume on the Volume Gateway
```

There is no in-place resize — you must clone and reassign.

---

## SysOps Exam Q&A

**Q: You need to reboot a Volume Gateway VM for maintenance. What is the correct procedure?**
A: 1) **Stop the Storage Gateway Service** (console/API/VM local console), 2) **Reboot the VM**, 3) **Start the Storage Gateway Service** again. Do not simply restart the VM directly.

**Q: A Storage Gateway activation is failing. What are the two most likely causes?**
A: 1) **Port 80 is not open** on the Gateway VM (if using the web request activation method). 2) The **Gateway VM clock is not synchronized** with an NTP server.

**Q: What CloudWatch metric tells you whether your Volume Gateway cache is efficient?**
A: **CacheHitPercent** — a high percentage means the cache is serving most reads locally (good). A low percentage means frequent S3 fetches (poor performance).

**Q: CacheHitPercent is consistently at 30% on a Volume Gateway. What should you do?**
A: **Increase the cache disk size** — clone the cache volume to a larger disk, then reassign it as the cache volume on the gateway.

**Q: Is the S3 File Gateway POSIX compliant?**
A: **Yes** — it stores Linux file metadata (ownership, permissions, timestamps) in S3 object metadata.

---

## Quick Reference

```
File Gateway: POSIX compliant (ownership, permissions, timestamps in S3 metadata)

Reboot procedure:
  File Gateway:          restart VM directly
  Volume/Tape Gateway:   stop service → reboot VM → start service

Activation methods:
  1. Gateway VM CLI → Get Activation Key (recommended)
  2. HTTP request to Port 80 on Gateway VM

Activation failure causes:
  - Port 80 not open (method 2)
  - VM clock not NTP-synced

Volume Gateway Cache metrics:
  CacheHitPercent  = high is good; low = frequent S3 fetches
  CachePercentUsed = should not be too high

Fix low cache efficiency: clone cache volume → assign larger disk as cache
```
