# 11 — Amazon Athena

## Overview

Amazon Athena is a **serverless SQL query service** that analyzes data stored directly in Amazon S3 — no database to provision or manage. Built on the **Presto** engine.

---

## Core Concept

```
Data in S3 (CSV, JSON, ORC, Parquet, Avro)
       │
       ▼
  Amazon Athena (serverless SQL queries)
       │
       ▼
  Results (query output, stored back in S3)
       │
       ▼
  Amazon QuickSight (dashboards & reports)
```

- Data stays **in S3** — Athena queries it in place, no movement required
- You write standard **SQL** — Athena handles the execution

---

## Supported Formats

| Format | Notes |
|---|---|
| CSV | Common, human-readable |
| JSON | Common, human-readable |
| **ORC** | Columnar — recommended for performance |
| **Parquet** | Columnar — recommended for performance |
| Avro | Row-based |

---

## Pricing

- Pay per **terabyte of data scanned**
- No infrastructure to provision
- Use columnar formats + compression + partitioning to **scan less data = lower cost**

---

## Use Cases

| Use Case | Detail |
|---|---|
| Ad hoc queries | One-off analytics on S3 data |
| Business intelligence | Connect to QuickSight for dashboards |
| Log analysis | Query VPC Flow Logs, ALB access logs, CloudTrail trails |
| S3 Inventory analysis | Query inventory reports to find unencrypted objects, etc. |

> **Exam tip:** Any question asking about *"serverless SQL query on S3 data"* → answer is **Amazon Athena**.

---

## Performance Optimizations (Exam-Tested)

### 1 — Use Columnar Formats (Most Important)
- Use **Apache Parquet** or **ORC** instead of CSV/JSON
- Columnar = only the queried columns are scanned → huge cost and speed improvement
- To convert CSV → Parquet: use **AWS Glue** as an ETL job

### 2 — Compress Data
- Compress files to reduce data scanned
- Supported: bzip2, gzip, lz4, snappy, zstd, zlib, deflate
- Smaller files = less data scanned = lower cost

### 3 — Partition Datasets in S3
- Organize data in S3 using a folder path that Athena can filter on at query time
- Format: `column=value/column=value/...`

Example — flight data partitioned by year/month/day:
```
s3://my-bucket/flights/
    year=1991/month=01/day=01/data.parquet
    year=1991/month=01/day=02/data.parquet
    year=1991/month=02/day=01/data.parquet
```

When you query `WHERE year=1991 AND month=01`, Athena only scans `year=1991/month=01/` — skips all other data.

### 4 — Use Large Files (> 128 MB)
- Many small files = high overhead per file
- Fewer large files (128 MB+) = better scan performance
- Consolidate small files before ingesting into S3 for Athena queries

---

## Performance Summary

| Optimization | Benefit |
|---|---|
| Parquet / ORC format | Only scan needed columns → lower cost + faster |
| Compression | Less data to scan → lower cost |
| Partitioning | Skip irrelevant S3 prefixes → faster + lower cost |
| Large files (128 MB+) | Reduce per-file overhead → faster queries |
| Glue ETL | Convert CSV → Parquet automatically |

---

## Federated Query

Athena can query data **outside S3** using **Data Source Connectors** (Lambda functions).

```
Amazon Athena
       │
       ▼
  Data Source Connector (Lambda function — one per source)
       │
       ├──► DynamoDB
       ├──► RDS / Aurora
       ├──► Redshift
       ├──► ElastiCache
       ├──► DocumentDB
       ├──► CloudWatch Logs
       ├──► SQL Server / MySQL
       ├──► HBase on EMR
       └──► On-premises databases
```

- Each connector is a **Lambda function** that runs the query against the external source
- Results are stored back in **Amazon S3** for further analysis
- Allows a single Athena query to join data across multiple sources

---

## Athena + QuickSight Integration

```
S3 (raw data) → Athena (SQL queries) → QuickSight (dashboards & visualizations)
```

- QuickSight connects directly to Athena as a data source
- Common architecture for business intelligence on S3 data

---

## SysOps Exam Q&A

**Q: What is Amazon Athena?**
A: A **serverless SQL query service** that analyzes data directly in S3 without moving it. Built on Presto.

**Q: How is Athena priced?**
A: **Per terabyte of data scanned**. Use columnar formats, compression, and partitioning to reduce scanned data and cost.

**Q: Which two file formats give the best Athena performance and lowest cost?**
A: **Apache Parquet** and **ORC** — columnar formats that only scan the queried columns.

**Q: How do you convert CSV data to Parquet for Athena?**
A: Use **AWS Glue** as an ETL job to convert CSV → Parquet.

**Q: A company wants to query VPC Flow Logs using SQL without provisioning a database. What should they use?**
A: **Amazon Athena** — directly queries log data stored in S3 using standard SQL.

**Q: What is Athena Federated Query?**
A: A feature that allows Athena to query data in external sources (DynamoDB, RDS, CloudWatch Logs, on-premises databases) using **Data Source Connectors** (Lambda functions).

**Q: How do partitions in S3 improve Athena query performance?**
A: By organizing data into prefix paths (`year=1991/month=01/`), Athena skips entire folders that don't match query filters — scanning far less data.

---

## Quick Reference

```
Athena = serverless SQL on S3 data (Presto engine)
Pricing = per TB scanned
Formats = CSV, JSON, ORC, Parquet, Avro

Best formats      = Parquet / ORC (columnar, scan only needed columns)
Convert CSV→Parquet = AWS Glue ETL
Partitioning      = S3 path as column=value → skip irrelevant folders
File size         = prefer large files (128 MB+) over many small ones
Compression       = reduces data scanned

Federated Query   = query non-S3 sources via Lambda Data Source Connectors
Result storage    = query results saved back to S3
Visualization     = Athena → QuickSight for dashboards

Exam trigger: "serverless SQL on S3" → Athena
```
