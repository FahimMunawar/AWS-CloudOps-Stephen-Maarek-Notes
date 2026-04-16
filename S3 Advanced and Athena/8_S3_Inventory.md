# 8 — S3 Inventory

## Overview

S3 Inventory generates a report listing all objects in a bucket with their metadata. It is the preferred alternative to running `s3:ListObjects` API calls repeatedly, especially at scale.

---

## What S3 Inventory Provides

| Information | Detail |
|---|---|
| Object keys | All object names in the bucket |
| Version IDs | All current or all versions (configurable) |
| Size | Object size in bytes |
| Last modified | Timestamp |
| Storage class | e.g., STANDARD, GLACIER |
| Encryption status | Whether each object is encrypted and which type |
| Replication status | e.g., PENDING, COMPLETED, FAILED |
| ETag | Object entity tag |
| Bucket Key status | Whether S3 Bucket Key is enabled |
| Object Lock | Retention settings |

---

## Use Cases

| Use Case | How |
|---|---|
| **Identify unencrypted objects** | Filter CSV/report by encryption column |
| **Audit replication status** | Check replication status column |
| **Count objects in bucket** | Row count of inventory report |
| **Total storage of old versions** | Include all versions + sum size column |
| **Compliance/regulatory audits** | Export and archive daily/weekly reports |

---

## Output Formats

| Format | Best For |
|---|---|
| **CSV** | S3 Batch Operations, Excel analysis |
| **ORC** | Athena, Spark, Hive query engines |
| **Parquet** | Athena, Spark, Hive query engines |

> Use **CSV** if you plan to feed the output into S3 Batch Operations or open it in Excel. Use **ORC/Parquet** for analytics tools like Athena.

---

## Frequency

- **Daily** — first report delivered within **48 hours** of setup
- **Weekly**

---

## Compatible Query Tools

- **Amazon Athena** (AWS)
- **Amazon Redshift** (AWS)
- Apache Presto, Hive, Spark (external/on-premises)

---

## Setup (Console)

1. S3 → Source bucket → **Management** → scroll to **Inventory configurations** → **Create inventory configuration**
2. **Name**: e.g., `DemoInventory`
3. **Prefix**: optional — leave blank for entire bucket
4. **Object versions**: Current version only, or All versions
5. **Destination bucket**: must be in the **same region** as the source bucket
   - AWS auto-adds a bucket policy to the destination to allow S3 to write reports
6. **Frequency**: Daily or Weekly
7. **Output format**: CSV, ORC, or Parquet
8. **Additional fields**: select Size, Storage class, Encryption, Replication status, etc.
9. Save

> **Region requirement:** Source and destination buckets must be in the same AWS region.

---

## Inventory Report Structure

Reports are stored in the destination bucket under a path like:

```
destination-bucket/
└── DemoInventory/
    └── 2024-07-04T00-00Z/
        ├── manifest.json      ← metadata about the report
        ├── manifest.checksum  ← integrity check
        └── data/
            └── <uuid>.csv.gz  ← actual inventory data (gzipped)
```

### manifest.json contains:
- `sourceBucket` — the bucket that was inventoried
- `destinationBucket` — where the report was written
- `fileFormat` — CSV / ORC / Parquet
- `fileSchema` — column names/order
- `files` — list of data file paths

### CSV columns (example):
```
bucket-name, object-key, version-id, is-latest, is-delete-marker, size, last-modified, etag, storage-class, encryption-status, ...
```

---

## Cleanup

When finished, **disable or delete the inventory configuration** to stop daily report generation:
- S3 → Bucket → Management → Inventory configurations → Delete/Disable

---

## Best Practices

- Combine with **S3 Batch Operations**: use inventory to find objects needing action (e.g., unencrypted), then pass the list to Batch Operations
- Use **daily frequency** for active buckets; weekly for archival buckets
- Store inventory reports in a **separate dedicated bucket** to keep reports organized
- Use **Athena** with ORC/Parquet format for fast SQL queries over large inventory reports

---

## SysOps Exam Q&A

**Q: What is S3 Inventory used for?**
A: Generating periodic (daily/weekly) reports listing all objects in a bucket with their metadata — used for auditing, compliance, encryption status checks, and feeding S3 Batch Operations.

**Q: Which output format should you use for S3 Inventory if you plan to use S3 Batch Operations?**
A: **CSV** — it is the required format for S3 Batch Operations manifests.

**Q: You want to find all unencrypted objects in an S3 bucket. What is the recommended approach?**
A: Use **S3 Inventory** to generate a report with the encryption status column, then filter for unencrypted objects, then pass the list to **S3 Batch Operations** to encrypt them.

**Q: How long does it take to receive the first S3 Inventory report?**
A: Up to **48 hours** for the first report after configuration.

**Q: Can the S3 Inventory destination bucket be in a different region than the source bucket?**
A: **No** — source and destination buckets must be in the **same AWS region**.

---

## Quick Reference

```
S3 Inventory = periodic object listing with metadata
Frequency    = daily (first report within 48 hours) or weekly
Formats      = CSV (Batch Ops / Excel), ORC / Parquet (Athena / Spark)
Same region  = source and destination buckets must match region
Output path  = destination-bucket/InventoryName/date/manifest.json + data/
Query tools  = Athena, Redshift, Presto, Hive, Spark

Cleanup      = disable inventory config when done (stops daily charges)

Common pattern:
  Inventory (find unencrypted) → Athena (filter) → Batch Operations (encrypt)
```
