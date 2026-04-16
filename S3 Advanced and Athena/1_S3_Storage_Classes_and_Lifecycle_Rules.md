# 1 — S3 Storage Classes, Lifecycle Rules & Analytics

## Overview

S3 objects can be transitioned between storage classes automatically using **Lifecycle Rules**. S3 Analytics helps determine the optimal transition timing.

---

## Storage Class Transition Flow

Objects can only move **down** the hierarchy (to less expensive/slower tiers):

```
Standard
   │
   ▼
Standard-IA ──────────────────────────┐
   │                                  │
   ▼                                  │
Intelligent-Tiering                   │
   │                                  │
   ▼                                  │
One-Zone-IA ──────────────────────────┤
   │                                  │
   ▼                                  ▼
Glacier Flexible Retrieval        Glacier Deep Archive
   │
   ▼
Glacier Deep Archive
```

| Storage Class | Use Case |
|---|---|
| **Standard** | Frequently accessed data |
| **Standard-IA** | Infrequently accessed, rapid retrieval |
| **Intelligent-Tiering** | Unknown or changing access patterns |
| **One-Zone-IA** | Infrequently accessed, easily recreatable, single AZ |
| **Glacier Flexible Retrieval** | Archival, minutes-to-hours retrieval |
| **Glacier Deep Archive** | Long-term archival, up to 48-hour retrieval |

---

## Lifecycle Rules

Rules that **automate** object transitions and deletions. Can be applied to:
- An **entire bucket**
- A specific **prefix** (e.g., `thumbnails/`)
- Specific **object tags** (e.g., `department: finance`)

### Two Types of Actions

#### 1 — Transition Actions
Move objects to a cheaper storage class after a set number of days.

| Example | Rule |
|---|---|
| Move to Standard-IA | After 60 days from creation |
| Move to Glacier | After 6 months |

#### 2 — Expiration Actions
Permanently delete objects after a set number of days.

| Example | Rule |
|---|---|
| Delete access logs | After 365 days |
| Delete old object versions | After N days (requires versioning) |
| Delete incomplete multi-part uploads | After 14 days (e.g., stalled uploads) |

---

## Exam Scenario 1 — Source Images + Thumbnails

**Requirements:**
- Thumbnails: easily recreatable, only needed for 60 days
- Source images: immediate retrieval for 60 days, then up to 6 hours acceptable

**Solution:**

| Object Type | Storage Class | Lifecycle Rule |
|---|---|---|
| Source images (`source/`) | **Standard** | Transition to **Glacier** after 60 days |
| Thumbnails (`thumbnails/`) | **One-Zone-IA** | **Expire (delete)** after 60 days |

- One-Zone-IA for thumbnails: infrequent access + easily recreatable = OK to lose one AZ copy
- Use **prefix** to differentiate rules between source and thumbnails

---

## Exam Scenario 2 — Deleted Object Recovery with Versioning

**Requirements:**
- Deleted objects: immediately recoverable for 30 days
- After 30 days: recoverable within 48 hours for up to 365 days total

**Solution:**

1. Enable **S3 Versioning** — deleted objects become hidden by a delete marker (not permanently removed)
2. Lifecycle rule: transition **non-current versions** to **Standard-IA** (for 30-day window)
3. Lifecycle rule: transition **non-current versions** to **Glacier Deep Archive** after 30 days (48-hour retrieval SLA)

```
Day 0–30:   Non-current versions in Standard-IA  → immediate retrieval
Day 30–365: Non-current versions in Glacier Deep Archive → 48-hour retrieval
```

> **"Non-current versions"** = all versions except the latest (the current) version of an object.

---

## S3 Analytics

Helps determine the **optimal number of days** to transition objects between storage classes.

| Feature | Detail |
|---|---|
| **Works with** | Standard and Standard-IA only |
| **Does NOT work with** | One-Zone-IA or Glacier |
| **Output** | CSV report with recommendations and statistics |
| **Update frequency** | Daily |
| **Time to first data** | 24–48 hours after enabling |

- S3 Analytics runs on top of your S3 bucket
- Use the CSV report as input to design or improve lifecycle rules

---

## Best Practices

- Use **prefixes** and **tags** to apply different lifecycle rules to different object types in the same bucket
- Always use **expiration rules** for incomplete multi-part uploads — stalled uploads consume storage and generate costs
- Use **S3 Analytics** (wait 24–48 hours) before setting transition days — let data guide the decision
- Combine versioning + lifecycle rules on non-current versions for cost-effective recovery windows

---

## SysOps Exam Q&A

**Q: What is the purpose of S3 Lifecycle Rules?**
A: To automatically **transition objects** to cheaper storage classes or **expire (delete)** objects after a specified number of days, reducing storage costs.

**Q: A company needs thumbnails deleted after 60 days and they are infrequently accessed. Which storage class and action?**
A: Store in **One-Zone-IA** (infrequent access, easily recreatable) with a lifecycle **expiration rule** after 60 days.

**Q: How do you automatically clean up stalled multi-part uploads in S3?**
A: Create a lifecycle rule with an **expiration action** targeting incomplete multi-part uploads (e.g., after 14 days).

**Q: A company wants deleted objects immediately recoverable for 30 days and within 48 hours for up to a year. What is the design?**
A: Enable versioning + lifecycle rules to transition **non-current versions** to Standard-IA (0–30 days) then **Glacier Deep Archive** (30–365 days).

**Q: Which S3 storage classes does S3 Analytics support?**
A: **Standard and Standard-IA only** — it does not support One-Zone-IA or Glacier.

**Q: How long does it take for S3 Analytics to produce data after being enabled?**
A: **24 to 48 hours** for initial data; updated daily after that.

---

## Quick Reference

```
Lifecycle rule targets:
  - Entire bucket, or prefix (path), or object tags

Transition action  = move to cheaper class after N days
Expiration action  = delete object/version after N days

Key expiration uses:
  - Access logs after 365 days
  - Old non-current versions (versioning)
  - Incomplete multi-part uploads (after ~14 days)

Non-current versions = all versions except the latest

S3 Analytics:
  - Works: Standard, Standard-IA only
  - Output: CSV report, updated daily
  - First data: 24–48 hours
  - Use to inform lifecycle rule transition timing
```
