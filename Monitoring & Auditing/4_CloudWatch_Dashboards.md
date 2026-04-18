# 4 — CloudWatch Dashboards

## Overview

CloudWatch Dashboards provide a customizable, global view of your metrics and alarms — across multiple AWS accounts and regions in a single pane of glass.

---

## Key Features

| Feature | Detail |
|---|---|
| **Global scope** | Dashboards can include metrics from **multiple regions and multiple AWS accounts** |
| **Time zone & range** | Configurable time zone and time range — all widgets on a dashboard share the same time window |
| **Auto-refresh** | Set automatic refresh interval |
| **Sharing** | Share dashboards with users who do **not have an AWS account** |
| **Custom metrics** | Include metrics from custom namespaces (PutMetricData) alongside AWS metrics |

> **Exam tip:** CloudWatch Dashboards are **global** — you can add a metric from `us-east-1` and `ap-southeast-2` on the same dashboard even if you're working in `eu-central-1`.

---

## Pricing

| Tier | Price |
|---|---|
| **Free** | 3 dashboards, up to 50 metrics each |
| **Paid** | $3 per dashboard per month |

---

## Dashboard Types

### Automatic Dashboards
- Pre-configured by AWS per service (e.g., Auto Scaling dashboard)
- Filter by **Resource Group** (e.g., DevGroup, ProdGroup)
- Show metrics AWS considers most relevant for that service

### Custom Dashboards
- Fully configurable — add any metric, alarm, or text widget
- Build from scratch or combine multiple automatic dashboards

---

## Widget Types Available

| Widget | Use Case |
|---|---|
| **Line** | Metric trends over time |
| **Stacked area** | Aggregate metrics |
| **Number** | Current value of a single metric |
| **Text** | Labels, descriptions, documentation |
| **Logs table** | CloudWatch Logs insights results |
| **Alarm status** | Visual alarm state (OK/ALARM/INSUFFICIENT_DATA) |

---

## Creating a Custom Dashboard (Console)

1. CloudWatch → **Dashboards** → **Create dashboard**
2. Name the dashboard
3. **Add widget** → choose widget type
4. Select metric:
   - Choose namespace (AWS or custom)
   - Choose region (can be any region — not just your current one)
   - Select specific metric and dimension
5. Repeat to add more widgets
6. Set time range (e.g., last 180 minutes, last 1 month)
7. **Save dashboard** — changes are not persisted until saved

---

## Cross-Region Metrics

When adding a metric widget:
- Default shows metrics from your **current region**
- Change the region selector to pick **any AWS region**
- Multiple widgets from different regions display side by side on the same dashboard

---

## SysOps Exam Q&A

**Q: Can a CloudWatch Dashboard display metrics from multiple AWS regions?**
A: **Yes** — dashboards are global and can include metrics from any region and any AWS account on the same dashboard.

**Q: How many free dashboards does CloudWatch provide?**
A: **3 dashboards** with up to **50 metrics each**, for free. Additional dashboards cost $3/month each.

**Q: Can you share a CloudWatch Dashboard with someone who doesn't have an AWS account?**
A: **Yes** — dashboards can be shared with external users who have no AWS account.

**Q: All widgets on a CloudWatch Dashboard share the same time window. True or false?**
A: **True** — when you change the time range on a dashboard, all widgets update to the same time window simultaneously.

---

## Quick Reference

```
Dashboards = global (multi-region, multi-account in one view)
Free tier  = 3 dashboards × 50 metrics
Paid       = $3/dashboard/month

Types:
  Automatic = pre-built by AWS per service, filter by resource group
  Custom    = build from scratch with any widgets

Widgets: Line, Stacked Area, Number, Text, Logs Table, Alarm Status
Shared time = all widgets on same dashboard use same time range
Share with non-AWS users = yes
Save dashboard = required to persist changes
```
