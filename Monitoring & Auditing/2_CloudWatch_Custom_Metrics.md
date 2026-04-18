# 2 — CloudWatch Custom Metrics

## Overview

Custom metrics let you push any application or system data into CloudWatch using the `PutMetricData` API. Common use cases: EC2 RAM usage, disk space, active user counts, application-level KPIs.

---

## PutMetricData API

The API call used to push custom metrics into CloudWatch.

### Key Parameters

| Parameter | Description |
|---|---|
| **Namespace** | Custom namespace name (e.g., `MyNameSpace`) — appears separately from AWS-managed namespaces |
| **MetricName** | Name of your metric (e.g., `MemoryUsage`, `ActiveUsers`) |
| **Value** | The metric value |
| **Unit** | Unit of measurement (e.g., Bytes, Count, Percent) |
| **Dimensions** | Key-value attributes to scope the metric (e.g., `InstanceId`, `Environment`) — up to 30 |
| **Timestamp** | When the data point occurred |
| **StorageResolution** | Metric resolution — Standard (60s) or High Resolution (1/5/10/30s) |

---

## Metric Resolution Options

| Type | StorageResolution Value | Frequency |
|---|---|---|
| **Standard** | 60 | Every **1 minute** |
| **High Resolution** | 1, 5, 10, or 30 | Every **1, 5, 10, or 30 seconds** |

---

## Timestamp Flexibility — Key Exam Point

> CloudWatch accepts custom metric data points with timestamps **up to 2 weeks in the past** or **up to 2 hours in the future** without error.

**Why this matters:**
- If your EC2 instance clock is misconfigured, metrics will be accepted but will land at the wrong time on the graph
- **Always ensure EC2 instance time is correctly configured** (synchronized with NTP) when pushing custom metrics

---

## CLI Example

```bash
aws cloudwatch put-metric-data \
  --namespace "MyNameSpace" \
  --metric-name "Buffers" \
  --value 12345 \
  --unit Bytes \
  --dimensions InstanceId=i-1234567890abcdef0,InstanceType=t2.micro
```

After pushing, the custom namespace `MyNameSpace` appears in CloudWatch → Metrics alongside all AWS-managed namespaces.

---

## How CloudWatch Agent Uses PutMetricData

The **CloudWatch Unified Agent** (installed on EC2 instances) uses `PutMetricData` internally to push system-level metrics that are not collected by default, such as:
- **RAM/memory usage**
- **Disk space usage**
- **Process counts**
- **Custom application metrics**

---

## Custom Namespace in the Console

- After pushing, the namespace appears under **CloudWatch → Metrics → All metrics**
- Listed alongside AWS-managed namespaces (AWS/EC2, AWS/ELB, etc.) but in a separate "Custom namespaces" section
- You can filter by the dimensions you defined (e.g., InstanceId, InstanceType)

---

## SysOps Exam Q&A

**Q: How do you push custom metrics to CloudWatch?**
A: Use the `PutMetricData` API (via CLI or SDK) — specify namespace, metric name, value, unit, dimensions, and optionally a timestamp and storage resolution.

**Q: What is the maximum time in the past you can use for a custom metric timestamp?**
A: **2 weeks in the past**. CloudWatch also accepts timestamps up to **2 hours in the future**.

**Q: Custom metrics are appearing at incorrect times in CloudWatch. What is the likely cause?**
A: The **EC2 instance clock is not synchronized** — it is not using NTP. Metric data is accepted regardless but lands at the wrong timestamp.

**Q: What are the two storage resolution options for custom metrics?**
A: **Standard** (1-minute / 60s) and **High Resolution** (1, 5, 10, or 30 seconds).

**Q: Where do custom metric namespaces appear in CloudWatch?**
A: In **CloudWatch → Metrics → All metrics**, listed separately from AWS-managed namespaces (which all begin with `AWS/`).

---

## Quick Reference

```
API: PutMetricData
Namespace: custom name (e.g., MyNameSpace) — appears in CloudWatch separate from AWS/* namespaces
Dimensions: key-value pairs to scope metric (e.g., InstanceId, Environment) — up to 30

Timestamp flexibility:
  Accepts up to 2 weeks in the PAST
  Accepts up to 2 hours in the FUTURE
  → Ensure EC2 clock is NTP-synced or timestamps will be wrong

Storage resolution:
  Standard      = 60s (1 minute)
  High-res      = 1s, 5s, 10s, or 30s

CloudWatch Agent uses PutMetricData to push RAM, disk, etc.
```
