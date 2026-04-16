# 1. S3 Lifecycle Rules and Analytics

## Overview

S3 Lifecycle Rules automate the transition of objects between storage classes and the deletion of expired objects. S3 Analytics helps identify the optimal transition points by analyzing access patterns.

---

## Storage Class Transition Flow

Objects can only be transitioned **downward** (to cheaper/colder tiers):

```
Standard
    ↓
Standard-IA  →  Intelligent-Tiering
    ↓
One-Zone-IA
    ↓
Glacier Flexible Retrieval
    ↓
Glacier Deep Archive
```

---

## Lifecycle Rule Types

### Transition Actions
Move objects to a different storage class after a set number of days.

| Example | Rule |
|---------|------|
| Move to Standard-IA | After 60 days from creation |
| Move to Glacier | After 180 days (6 months) |

### Expiration Actions
Delete objects (or specific versions) after a set period.

| Use Case | Rule |
|----------|------|
| Access log files | Delete after 365 days |
| Old non-current versions | Delete after N days (requires versioning) |
| Incomplete multi-part uploads | Delete if not completed within 14 days |

---

## Lifecycle Rule Scope

Rules can be scoped to:
- **Entire bucket** — applies to all objects
- **Prefix** — applies to objects under a specific path (e.g., `thumbnails/`)
- **Object tags** — applies to objects with specific tags (e.g., `department = finance`)

---

## Exam Scenario 1: Source Images and Thumbnails

**Requirements:**
- Source images: immediately retrievable for 60 days, then up to 6 hours acceptable
- Thumbnails: easily re-creatable, only needed for 60 days

**Solution:**

| Object | Storage Class | Lifecycle Rule |
|--------|--------------|----------------|
| Source images (`/source/`) | S3 Standard | Transition to Glacier Flexible Retrieval after 60 days |
| Thumbnails (`/thumbnails/`) | S3 One-Zone-IA | Expire (delete) after 60 days |

- **One-Zone-IA** for thumbnails: infrequently accessed + can be recreated (no need for multi-AZ durability)
- Use **prefix-based rules** to apply different policies to `source/` vs `thumbnails/`

---

## Exam Scenario 2: Versioned Bucket with Recovery Requirements

**Requirements:**
- Immediately recover deleted objects for 30 days
- After 30 days, recoverable within 48 hours for up to 365 days

**Solution:**

1. Enable **S3 Versioning** — deleted objects become hidden behind a delete marker (not truly deleted)
2. Lifecycle rules on **non-current versions**:

```
Non-current version created
        ↓ (after 30 days)
Transition to Standard-IA
        ↓ (after 365 days)
Transition to Glacier Deep Archive
```

- Current version (with delete marker) allows immediate recovery for 30 days
- Non-current versions in Standard-IA allow recovery within 48 hours (within 365 days)

---

## S3 Analytics

Analyzes access patterns and provides **recommendations for optimal lifecycle transition timing**.

| Feature | Detail |
|---------|--------|
| Works with | Standard → Standard-IA transitions |
| Does NOT work with | One-Zone-IA, Glacier |
| Output | CSV report with statistics and recommendations |
| Update frequency | Daily |
| Time to first data | **24 to 48 hours** |

Use S3 Analytics as a **first step** before creating lifecycle rules — let it gather data, then implement rules based on its recommendations.

---

## Best Practices

✓ **Use prefix-based rules** to apply different lifecycle policies to different object types in the same bucket  
✓ **Always expire incomplete multi-part uploads** — they accumulate silently and incur storage costs  
✓ **Enable versioning before setting non-current version lifecycle rules** — versioning must exist first  
✓ **Run S3 Analytics for 24–48 hours** before defining transition days — base rules on actual access data  
✓ **Use One-Zone-IA only for re-creatable or non-critical data** — it lacks multi-AZ durability  

---

## SysOps Exam Focus

**Q1: "Objects in your S3 bucket are frequently accessed for 30 days then rarely accessed. What lifecycle rule should you apply?"**
- A) Move to Glacier immediately
- B) Move to Standard-IA after 30 days
- C) Delete objects after 30 days
- D) Enable Intelligent-Tiering
- **Answer: B** — Standard-IA is designed for infrequently accessed objects that still need occasional retrieval

**Q2: "Your S3 bucket has thousands of incomplete multi-part uploads consuming storage. How do you clean them up automatically?"**
- A) Enable bucket versioning
- B) Add a lifecycle expiration rule to delete incomplete multi-part uploads after N days
- C) Use S3 Analytics to detect them
- D) Enable S3 Replication
- **Answer: B** — Lifecycle rules have a specific action type for expiring incomplete multi-part uploads

**Q3: "You want to know how long objects are typically accessed before they become infrequently accessed. What tool should you use?"**
- A) AWS Cost Explorer
- B) CloudWatch
- C) S3 Analytics
- D) AWS Trusted Advisor
- **Answer: C** — S3 Analytics analyzes access patterns and produces daily CSV reports with transition recommendations

**Q4: "A company needs deleted S3 objects to be immediately recoverable for 30 days, then recoverable within 48 hours for one year. What is the correct approach?"**
- A) Create daily snapshots of the bucket
- B) Enable versioning and create lifecycle rules to transition non-current versions to Standard-IA then Glacier Deep Archive
- C) Enable Cross-Region Replication
- D) Use S3 Object Lock
- **Answer: B** — Versioning keeps deleted objects as non-current versions; lifecycle rules move them to cheaper tiers over time

**Q5: "For which storage class transition does S3 Analytics NOT provide recommendations?"**
- A) Standard to Standard-IA
- B) Standard-IA to Glacier
- C) Standard to Glacier Flexible Retrieval
- D) All of the above except A
- **Answer: D** — S3 Analytics only provides recommendations for Standard → Standard-IA transitions

---

## Quick Reference

```
Lifecycle transition order (downward only):
  Standard → Standard-IA → One-Zone-IA → Glacier Flexible → Deep Archive

Rule types:
  Transition  →  move to cheaper class after N days
  Expiration  →  delete after N days (objects, versions, incomplete uploads)

Rule scope:
  Entire bucket | Prefix | Object tags

S3 Analytics:
  Recommends: Standard → Standard-IA only
  Output: daily CSV report
  Delay: 24–48 hours to first results
```

---

**File: 1_S3_Lifecycle_Rules_and_Analytics.md**
**Status: SysOps-focused, exam-ready, concise format**
