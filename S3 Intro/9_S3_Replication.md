# 9 — S3 Replication (CRR & SRR)

## Overview

S3 Replication asynchronously copies objects from a source bucket to a target bucket. Two flavors: **CRR** (Cross-Region Replication) and **SRR** (Same-Region Replication).

---

## CRR vs SRR

| Feature | CRR | SRR |
|---|---|---|
| **Regions** | Source and destination in **different** regions | Source and destination in the **same** region |
| **Use cases** | Compliance, lower latency access, cross-account replication | Log aggregation, live replication between prod/test |
| **Cross-account** | Supported | Supported |

---

## Prerequisites

1. **Versioning must be enabled** on both source and destination buckets
2. Proper **IAM permissions** must be granted to the S3 service (create a new IAM role at setup)
3. Replication is **asynchronous** — happens in the background

---

## Use Cases

| Type | Use Case |
|---|---|
| **CRR** | Regulatory compliance requiring data in multiple regions |
| **CRR** | Lower latency access for users in a different region |
| **CRR** | Replicate data across AWS accounts |
| **SRR** | Aggregate logs from multiple S3 buckets into one |
| **SRR** | Live replication between production and test environments |

---

## Setting Up Replication (Console)

1. Source bucket → **Management** tab → **Replication rules** → **Create replication rule**
2. Name the rule (e.g., `DemoReplicationRule`)
3. **Source**: choose all objects or filter by prefix/tag
4. **Destination**: same account or different account → enter target bucket name
5. **IAM Role**: create a new role (S3 service needs read/write permissions on both buckets)
6. Save — prompted: *"Replicate existing objects?"*

> Existing objects are **not replicated automatically** — see S3 Batch Replication below.

---

## Important Behaviours

### Only New Objects Are Replicated
- Replication applies to objects uploaded **after** the rule is enabled
- Objects that existed before enabling replication are **not replicated**

### Replicate Existing Objects → S3 Batch Replication
- Use **S3 Batch Replication** to replicate:
  - Objects that existed before replication was set up
  - Objects that previously failed replication

### Version IDs Are Preserved
- The version ID of a replicated object in the destination bucket is **identical** to the source
- This ensures consistency across buckets

---

## Delete Marker Replication

| Operation | Replicated by default? | Notes |
|---|---|---|
| **Soft delete** (delete marker added) | **No** — optional setting | Enable "Delete marker replication" in rule settings |
| **Permanent delete** (specific version ID deleted) | **Never replicated** | Protects against malicious deletes propagating |

### Enabling Delete Marker Replication
- Management → Edit replication rule → scroll down → enable **Delete marker replication**
- Once enabled: deleting an object (adding a delete marker) in source → delete marker appears in destination

### Why Permanent Deletes Are Not Replicated
- Deleting a specific version ID is intentionally **blocked from replication**
- Prevents a user from permanently destroying data in both source and destination simultaneously

---

## No Chaining of Replication

```
Bucket 1 ──replicates──► Bucket 2 ──replicates──► Bucket 3

Objects from Bucket 1 are NOT replicated to Bucket 3
```

Replication does not chain — objects must be replicated directly from source to destination.

---

## Best Practices

- Enable versioning on **both buckets** before setting up replication (required)
- Use **S3 Batch Replication** if you need to backfill existing objects
- Enable **delete marker replication** only if you want deletes to propagate (consider carefully)
- Never rely on replication for permanent delete propagation — it is explicitly blocked
- For cross-account replication, the destination bucket policy must allow the source account's replication role

---

## SysOps Exam Q&A

**Q: What must be enabled on both source and destination buckets before setting up S3 replication?**
A: **Versioning** must be enabled on both buckets.

**Q: You enable CRR on a bucket with existing objects. Are those objects replicated?**
A: **No** — only new objects uploaded after the rule is created are replicated. Use **S3 Batch Replication** for existing objects.

**Q: A delete marker is created in the source bucket. Will it appear in the destination bucket?**
A: **Only if delete marker replication is explicitly enabled** in the replication rule settings. It is off by default.

**Q: A user permanently deletes a specific version ID in the source bucket. Is this replicated?**
A: **No** — permanent deletes (version ID deletions) are never replicated. This is by design to prevent malicious deletes.

**Q: Bucket 1 replicates to Bucket 2, and Bucket 2 replicates to Bucket 3. Do objects from Bucket 1 appear in Bucket 3?**
A: **No** — replication does not chain. Bucket 1 objects only replicate to Bucket 2.

**Q: What is the difference between CRR and SRR?**
A: CRR replicates across different AWS regions (compliance, latency, cross-account). SRR replicates within the same region (log aggregation, prod/test mirroring).

**Q: After setting up replication, version IDs in the destination bucket are different from the source. Is this expected?**
A: **No** — version IDs are **preserved** and should be identical in source and destination.

---

## Quick Reference

```
CRR = cross-region replication  (compliance, latency, cross-account)
SRR = same-region replication   (log aggregation, prod/test)

Prerequisites:
  - Versioning ON for both source and destination
  - IAM Role granting S3 read/write on both buckets

New objects only       = only objects uploaded AFTER rule creation
Existing objects       = use S3 Batch Replication
Version IDs            = preserved (same in source and destination)
Delete markers         = NOT replicated by default; optional setting
Permanent deletes      = NEVER replicated (by design)
Replication chaining   = NOT supported (A→B→C does not replicate A to C)
```
