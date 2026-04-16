# 11 — S3 Replication Time Control (RTC)

## Overview

S3 Replication Time Control (RTC) is an optional feature that adds a **guaranteed SLA** to S3 replication speed, with CloudWatch monitoring included.

---

## What RTC Provides

| Feature | Detail |
|---|---|
| **Replication SLA** | **99.99%** of new objects replicated within **15 minutes** |
| **Predictability** | Guaranteed, auditable replication time |
| **Monitoring** | Publishes **CloudWatch metrics** for replication lag |
| **Alerts** | Notifies you if replication falls behind |
| **Scope** | Works for same-region (SRR) and cross-region (CRR) replication |

---

## When to Use RTC

- Company requires **compliance** with strict data replication timelines
- Business requirements mandate **predictable replication latency**
- You need **auditable proof** that replication completed within a defined window

---

## Cost

- RTC has an **additional cost** on top of standard replication pricing
- Charged **per GB** replicated with RTC enabled

---

## SysOps Exam Q&A

**Q: A company requires that S3 objects are replicated to another region within 15 minutes. What feature should you enable?**
A: **S3 Replication Time Control (RTC)** — guarantees 99.99% of objects are replicated within 15 minutes.

**Q: How do you monitor replication lag when using S3 RTC?**
A: RTC automatically publishes **CloudWatch metrics** that track replication latency and can trigger alerts if replication falls behind.

**Q: Does S3 RTC work with same-region replication (SRR)?**
A: **Yes** — RTC works with both CRR (cross-region) and SRR (same-region) replication.

---

## Quick Reference

```
RTC = S3 Replication Time Control
SLA = 99.99% of objects replicated within 15 minutes
Monitoring = CloudWatch metrics (automatic when RTC is enabled)
Works with = CRR and SRR
Cost = additional per-GB charge
Use when = compliance or business requires predictable replication time
```
