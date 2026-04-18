# 5 — CloudWatch Logs

## Overview

CloudWatch Logs is the central place to store, query, and stream application logs in AWS. Logs are encrypted by default and can be retained from 1 day to 10 years (or indefinitely).

---

## Structure

```
Log Group  (e.g., /aws/lambda/my-function — represents one application)
  └── Log Stream  (one log instance / log file / container)
        └── Log Events  (timestamped log lines)
```

---

## Log Sources

| Source | Notes |
|---|---|
| **SDK / CloudWatch Logs Agent** | Push logs from any application; Unified Agent is the current standard (old agent deprecated) |
| **CloudWatch Unified Agent** | Also collects system metrics (RAM, disk) and logs |
| **Elastic Beanstalk** | Application logs sent automatically |
| **ECS** | Container logs sent directly |
| **Lambda** | Function logs sent automatically |
| **VPC Flow Logs** | Network metadata for the VPC |
| **API Gateway** | All requests made to the API |
| **CloudTrail** | Audit events, filter-based |
| **Route 53** | All DNS queries |

---

## Expiration Policy

- **Never expire** (default — indefinite retention)
- **1 day to 10 years** — configurable per log group

---

## CloudWatch Logs Insights

A purpose-built **query engine** for historical log data (not real-time).

### Capabilities
- Write queries with auto-detected fields
- Filter by conditions, calculate aggregates, sort, limit
- Visualize results (charts)
- Save queries and add to CloudWatch Dashboards
- **Query multiple log groups at once, even across accounts**

### Common built-in query examples
- 25 most recent log events
- Events containing exceptions or errors
- Filter by specific IP address

> **Exam tip:** CloudWatch Logs Insights is a **query engine for historical data only** — it is NOT a real-time streaming solution.

---

## Exporting Logs

### Batch Export → Amazon S3

| Detail | Value |
|---|---|
| API call | `CreateExportTask` |
| Delivery time | Up to **12 hours** |
| Type | Batch — **not real-time** |

### Real-Time / Near Real-Time → Subscription Filters

Use **Subscription Filters** to stream log events to:

| Destination | Use Case |
|---|---|
| **Kinesis Data Streams** | Fan-out to KDA, Firehose, EC2, Lambda |
| **Kinesis Data Firehose** | Near real-time to S3, OpenSearch |
| **Lambda** | Custom processing or managed → OpenSearch |

---

## Cross-Account Log Aggregation

Aggregate logs from multiple accounts and regions into a single destination using **Subscription Destinations**.

```
Account A                          Account B (recipient)
─────────────────────              ──────────────────────────────────
CloudWatch Logs                    Kinesis Data Stream
  └── Subscription Filter    →     Subscription Destination (virtual)
                                     └── Destination Access Policy
                                           (allows Account A to send)
                                     └── IAM Role
                                           (can send to KDS, assumable by Account A)
```

**Full flow:**
```
Account A logs
  → Subscription Filter
  → Destination (Account B)
  → Kinesis Data Stream
  → Kinesis Data Firehose
  → Amazon S3 (near real-time)
```

Requirements:
1. **Subscription Destination** in recipient account pointing to the KDS
2. **Destination Access Policy** — grants sender account permission to write
3. **IAM Role** in recipient account — grants KDS put access, assumable by sender account

---

## SysOps Exam Q&A

**Q: What is the structure of CloudWatch Logs?**
A: **Log Groups** (per application) → **Log Streams** (per instance/container/log file) → **Log Events** (timestamped lines).

**Q: How long can logs be retained in CloudWatch Logs?**
A: From **1 day to 10 years**, or **never expire** (indefinite retention).

**Q: You need to query CloudWatch Logs for errors over the past week. What do you use?**
A: **CloudWatch Logs Insights** — a query engine for historical log data that supports filtering, aggregation, sorting, and visualizations.

**Q: Is CloudWatch Logs Insights real-time?**
A: **No** — it queries historical data only. For real-time streaming, use Subscription Filters.

**Q: You need to export CloudWatch Logs to S3. What is the API call and how long does it take?**
A: **CreateExportTask** — delivery can take up to **12 hours** (batch, not real-time).

**Q: You need near real-time delivery of CloudWatch Logs to S3. What should you use?**
A: **Subscription Filter → Kinesis Data Firehose → S3** (near real-time, not a batch export).

**Q: How do you aggregate CloudWatch Logs from multiple AWS accounts into one central location?**
A: Use **Subscription Filters + Subscription Destinations** — each sender account sends to a destination in the recipient account that points to a Kinesis Data Stream. Requires a Destination Access Policy and an IAM role in the recipient account.

---

## Quick Reference

```
Structure:
  Log Group → Log Stream → Log Events
  Retention: 1 day – 10 years, or never expire
  Encryption: default (managed keys) or custom KMS

Log sources:
  SDK, Unified Agent, Beanstalk, ECS, Lambda,
  VPC Flow Logs, API Gateway, CloudTrail, Route53

Logs Insights:
  Query historical data (NOT real-time)
  Multi-account, multi-log-group queries
  Save queries → add to dashboards

Export to S3:
  CreateExportTask → batch → up to 12 hours → NOT real-time

Real-time streaming (Subscription Filters):
  → Kinesis Data Streams
  → Kinesis Data Firehose  → S3 / OpenSearch (near real-time)
  → Lambda

Cross-account aggregation:
  Subscription Filter → Destination → KDS (recipient account)
  Requires: Destination Access Policy + IAM Role in recipient account
```
