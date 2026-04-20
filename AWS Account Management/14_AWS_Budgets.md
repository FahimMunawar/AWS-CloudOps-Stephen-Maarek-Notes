# 14 — AWS Budgets

## Overview

Create budgets based on actual or forecasted charges and send alarms when costs exceed thresholds.

---

## Budget Types

| Type | Purpose |
|---|---|
| **Cost** | Alert when dollar spend exceeds a threshold |
| **Usage** | Alert when usage (hours, GB, requests) exceeds a threshold |
| **Reservation** | Track RI utilization (under-utilized = wasted spend) |
| **Savings Plan** | Track Savings Plan utilization |

### Reservation Budget — Supported Services
EC2, ElastiCache, RDS, Redshift

---

## Key Features

| Feature | Detail |
|---|---|
| **Notifications** | Up to **5 notifications per budget** |
| **Alert triggers** | Actual charges OR forecast charges |
| **Filters** | Service, linked account, tag, purchase option, and more |
| **Granularity** | Same filtering options as Cost Explorer |

---

## Pricing

| Budget # | Cost |
|---|---|
| First 2 budgets | **Free** |
| Each additional | **$0.02/day per budget** |

---

## Budgets vs. Billing Alarms

| Feature | Billing Alarms | AWS Budgets |
|---|---|---|
| Alert on forecast | No (actual only) | Yes |
| Filter by service/tag | No | Yes |
| RI utilization tracking | No | Yes |
| Granularity | Account-wide | Highly granular |
| Multiple notifications | 1 per alarm | Up to 5 per budget |

> For the exam: Budgets is more flexible and granular than Billing Alarms.

---

## Quick Reference

```
Budgets: set cost/usage/RI/Savings Plan thresholds → alert when exceeded

4 types: Cost, Usage, Reservation (RI), Savings Plan
  RI budget: EC2, ElastiCache, RDS, Redshift

Up to 5 notifications per budget
Alert on: actual spend OR forecasted spend
Filters: service, account, tag, purchase option (same as Cost Explorer)

Pricing: first 2 free → $0.02/day each after

vs. Billing Alarms: Budgets is more granular, supports forecasts and RI tracking
```
