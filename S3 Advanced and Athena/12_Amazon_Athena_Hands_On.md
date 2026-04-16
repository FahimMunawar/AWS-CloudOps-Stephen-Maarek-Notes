# 12 — Amazon Athena — Hands-On

## Overview

Console walkthrough: configure Athena to query S3 access logs using SQL — create a results bucket, define a database and table, then run analytics queries.

---

## Setup: Query Result Location

Before running any query, Athena requires an S3 bucket to store query results.

1. Athena → **Settings** → Edit
2. Enter an S3 bucket URI: `s3://aws-athena-results-bucket/`
3. Save

> Every query result is saved here as a CSV file — required before Athena will execute any query.

---

## Step-by-Step: Query S3 Access Logs

### Step 1 — Create a Database
```sql
CREATE DATABASE s3_access_logs_db;
```
After running, the new database appears in the left-hand database selector.

### Step 2 — Create a Table (from S3 Access Log format)
```sql
CREATE EXTERNAL TABLE s3_access_logs_db.mybucket_logs (
  BucketOwner STRING,
  Bucket STRING,
  RequestDateTime STRING,
  RemoteIP STRING,
  Requester STRING,
  RequestID STRING,
  Operation STRING,
  Key STRING,
  RequestURI_operation STRING,
  RequestURI_key STRING,
  RequestURI_httpProtoVersion STRING,
  HTTPStatus STRING,
  ErrorCode STRING,
  BytesSent BIGINT,
  ObjectSize BIGINT,
  TotalTime STRING,
  TurnAroundTime STRING,
  Referrer STRING,
  UserAgent STRING,
  VersionId STRING
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  'serialization.format' = '1',
  'input.regex' = '([^ ]*) ([^ ]*) ...'
)
LOCATION 's3://your-log-bucket-name/';
```

> - Change `LOCATION` to your actual log bucket ARN/path
> - If logs are in a subfolder: `s3://bucket-name/prefix/` (trailing slash required)
> - The query format comes directly from the AWS S3 + Athena documentation

### Step 3 — Preview the Table
Right-click the table name in the left panel → **Preview table**

Athena runs:
```sql
SELECT * FROM s3_access_logs_db.mybucket_logs LIMIT 10;
```

Returns: bucket owner, bucket name, request datetime, IP address, requester, request ID, operation, status code, etc.

---

## Example Analytics Queries

### Count Requests by HTTP Status and Operation
```sql
SELECT
  HTTPStatus,
  RequestURI_operation,
  COUNT(*) AS request_count
FROM s3_access_logs_db.mybucket_logs
GROUP BY HTTPStatus, RequestURI_operation
ORDER BY request_count DESC;
```

Sample results:
- `404` + `GET` → 142 requests (Not Found — worth investigating)
- `200` + `HEAD` → 101 requests
- `403` + `GET` → N requests (Unauthorized — potential security issue)

### Find Unauthorized Access Attempts (403)
```sql
SELECT *
FROM s3_access_logs_db.mybucket_logs
WHERE HTTPStatus = '403';
```

- 403 = unauthorized/forbidden access
- Use this to identify if someone is repeatedly attempting unauthorized access to your bucket

---

## Key Points

| Concept | Detail |
|---|---|
| **Results bucket** | Must be configured before any query runs |
| **Database** | Created with `CREATE DATABASE` — logical grouping of tables |
| **Table** | Maps to data in S3; schema defined in Athena, data stays in S3 |
| **LOCATION** | Must end with a trailing slash (`/`) |
| **Data not moved** | Athena reads directly from S3 — no ETL or loading required |
| **Cost** | Based on data scanned per query (~30 MB in demo) |
| **Serverless** | No servers to provision or manage |

---

## Common Workflow for Log Analysis

```
S3 (access logs / VPC flow logs / CloudTrail)
       │
       ▼
Athena: CREATE DATABASE → CREATE TABLE (LOCATION = S3 path)
       │
       ▼
Run SQL queries → results saved to results S3 bucket
       │
       ▼
Investigate 403s, 404s, anomalies
```

---

## SysOps Exam Q&A

**Q: Before running an Athena query, what must be configured?**
A: An **S3 bucket for query results** — Athena requires a results location before it will execute any query.

**Q: Does Athena move or copy data out of S3?**
A: **No** — Athena queries data directly in S3. No data is moved or loaded.

**Q: What is required at the end of the S3 LOCATION path in an Athena table definition?**
A: A **trailing slash** — e.g., `s3://bucket-name/prefix/`.

**Q: You want to analyze S3 access logs for unauthorized access attempts. What HTTP status should you filter on?**
A: **403** — Forbidden/Unauthorized. Filter `WHERE HTTPStatus = '403'` to find unauthorized access attempts.

---

## Quick Reference

```
Setup:       Settings → set query results S3 bucket (required first)
Database:    CREATE DATABASE db_name;
Table:       CREATE EXTERNAL TABLE ... LOCATION 's3://bucket/prefix/';
             └── trailing slash required
             └── LOCATION must point to your actual log bucket
Preview:     right-click table → Preview table (SELECT * LIMIT 10)
404 queries: unauthorized or missing objects worth investigating
403 queries: potential unauthorized access attempts
Data stays in S3 — Athena never moves it
```
