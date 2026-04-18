# 1 — CloudWatch Metrics

## Overview

CloudWatch provides metrics for every AWS service. Metrics tell you how a service is behaving over time, enabling monitoring, troubleshooting, and automated scaling decisions.

---

## Core Concepts

| Concept | Description |
|---|---|
| **Metric** | A time-series data point for a specific measurement (e.g., CPUUtilization) |
| **Namespace** | Logical grouping of metrics by service (e.g., `AWS/EC2`, `AWS/ELB`) |
| **Dimension** | An attribute that identifies the resource a metric belongs to (e.g., InstanceId, Environment) |
| **Max dimensions per metric** | **30** |
| **Timestamp** | Every metric data point has a timestamp |
| **Dashboard** | Visual board combining multiple metrics across services |

---

## EC2 Metrics

### Default Monitoring (Free)
- Metrics pushed every **5 minutes**
- Available by default for all EC2 instances

### Detailed Monitoring (Paid)
- Metrics pushed every **1 minute**
- Additional cost
- Benefits:
  - Faster reaction to changing metrics
  - Enables faster **ASG scale-out/in** decisions

### Important: RAM (Memory) is NOT a Default Metric

| Metric | Pushed by Default? |
|---|---|
| CPU Utilization | Yes |
| NetworkIn / NetworkOut | Yes |
| DiskReadOps / DiskWriteOps | Yes |
| **Memory (RAM) Usage** | **No — must be pushed as a custom metric** |

> **Exam tip:** EC2 memory usage is not available in CloudWatch by default. You must install the **CloudWatch Agent** on the instance and configure it to push memory as a custom metric.

---

## Free Tier Detailed Monitoring

- Enables up to **10 detailed monitoring metrics** in the free tier

---

## CloudWatch Metrics Console

### Navigation
- CloudWatch → **Metrics** (left sidebar) → All metrics

### Metric Namespaces Visible
- `AWS/EC2` (per-instance and per-ASG metrics)
- `AWS/ELB`
- `AWS/AutoScaling`
- `AWS/EBS`
- `AWS/EFS`
- And many more — one namespace per AWS service

### Filtering
- Filter by **service namespace**
- Filter by **dimension** (e.g., specific InstanceId)
- Filter by **time range** (preset or custom range)

### Display Options
- Line graph, stacked area, bar, number, pie chart
- Add to a **dashboard**
- Download as **CSV**
- Share metric view

---

## Default Metric Interval Summary (EC2)

| Monitoring Type | Metric Frequency | Cost |
|---|---|---|
| Default monitoring | Every **5 minutes** | Free |
| Detailed monitoring | Every **1 minute** | Paid |

---

## SysOps Exam Q&A

**Q: How often does EC2 send metrics to CloudWatch by default?**
A: Every **5 minutes**. Enable detailed monitoring for every **1 minute** (additional cost).

**Q: An ASG needs to scale in/out faster based on EC2 metrics. What should you enable?**
A: **Detailed monitoring** on the EC2 instances — provides 1-minute resolution metrics instead of 5-minute.

**Q: Memory usage is not visible in CloudWatch for an EC2 instance. Why?**
A: **RAM/memory is not a default CloudWatch metric for EC2.** It must be pushed as a custom metric using the **CloudWatch Agent** installed on the instance.

**Q: What is a CloudWatch dimension?**
A: An attribute that identifies which specific resource a metric belongs to — for example, `InstanceId=i-1234567890abcdef0` scopes a CPU metric to one specific EC2 instance. Up to **30 dimensions** per metric.

**Q: What is a CloudWatch namespace?**
A: A logical container for metrics belonging to a specific AWS service — e.g., `AWS/EC2`, `AWS/ELB`, `AWS/EBS`.

---

## Quick Reference

```
Metrics = time-series data points for AWS services
Namespace = logical group per service (AWS/EC2, AWS/ELB, etc.)
Dimension = attribute scoping a metric to a resource (up to 30 per metric)

EC2 default  = 5-minute intervals (free)
EC2 detailed = 1-minute intervals (paid) → faster ASG scaling

RAM not default → must use CloudWatch Agent + custom metric

Console: CloudWatch → Metrics → filter by namespace/dimension/time
Outputs: graphs, dashboards, CSV download, share
```
