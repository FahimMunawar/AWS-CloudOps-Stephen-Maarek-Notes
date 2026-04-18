# 6 — CloudWatch Logs: Hands-On

## Log Groups Overview

- Each AWS service auto-creates its own log group (Lambda, DataSync, Glue, SSM RunCommand, etc.)
- SSM RunCommand example: one log group → **6 log streams** (one per instance, split into stdout and stderr)
- Within a log stream: browse raw log lines, or use keyword filter to find matching lines (e.g., `installing`, `http`)

---

## Metric Filters

Convert log data into CloudWatch metrics by defining a **filter pattern** against a log group.

### Setup Steps

1. Log Group → **Metric filters** → **Create metric filter**
2. Define **filter pattern** (e.g., `installing`)
3. Test against a log stream to confirm matches
4. Configure the metric:

| Field | Example |
|---|---|
| **Filter name** | `DemoFilter` |
| **Metric namespace** | `DemoMetricFilterNamespace` (custom) |
| **Metric name** | `DemoMetric` |
| **Metric value** | `1` (increment by 1 per match) |
| **Default value** | Optional |
| **Unit** | Optional |

5. Once created, the metric appears in **CloudWatch → Metrics → All metrics** under the custom namespace

> The metric only populates when the filter pattern matches incoming log lines. If no logs are coming in, the metric graph will be empty.

### Alarm from Metric Filter

- After creating the metric filter → click **Create alarm**
- Set a threshold (e.g., count > X) to trigger an alarm
- Useful for: alerting on specific error patterns or keywords in logs

---

## Subscription Filters (from Console)

From a log group → **Subscription filters** → create filter for:

| Destination | Notes |
|---|---|
| Amazon OpenSearch (Elasticsearch) | Direct streaming |
| Kinesis Data Streams | Fan-out to downstream services |
| Kinesis Data Firehose | Near real-time to S3 / OpenSearch |
| Lambda | Custom processing |

> **Limit: up to 2 subscription filters per log group**

---

## Retention Settings

- Edit from log group settings
- Options: **Never expire** → up to **120 months (10 years)**

---

## Exporting to S3 (Batch)

Log Group → **Actions** → **Export data to Amazon S3**

| Field | Detail |
|---|---|
| Date range | Select start and end |
| Stream prefix | Optional — filter specific log streams |
| S3 bucket | Destination bucket |
| Bucket prefix | Optional folder path |

This is a **batch export** — not real-time (up to 12 hours delivery).

---

## Creating a Log Group Manually

1. CloudWatch Logs → **Create log group**
2. Set name, retention policy, optional KMS key for encryption
3. If a KMS key is specified, the Key ID appears in the log group details

---

## CloudWatch Logs Insights (Console)

- Select one or more log groups
- Write a query in the purpose-built query language
- Set a time range (e.g., last hour, last 60 days)
- Run query → view matching records and visualizations
- **Export results** or **save queries** for reuse

### Built-in sample queries include:
- View Lambda latency statistics at 5-minute intervals
- Top 10 source/destination IP pairs from VPC Flow Logs

---

## Quick Reference

```
Metric Filter:
  Filter pattern → matches log lines → increments CloudWatch metric
  Custom namespace → visible in CloudWatch Metrics → create alarm on top

Subscription filters:
  Max 2 per log group
  Destinations: OpenSearch, KDS, Firehose, Lambda

Retention:
  Never expire → up to 120 months (10 years)

Batch export to S3:
  Actions → Export data → choose range, stream prefix, S3 bucket

Logs Insights:
  Query language on historical data
  Multi-log-group support
  Save queries, export results
```
