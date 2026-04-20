# 13 — AWS Cost Explorer

## Overview

Visualize, understand, and manage AWS costs and usage over time with custom reports and dashboards.

---

## Key Features

| Feature | Detail |
|---|---|
| **Custom reports** | Analyze cost and usage data with filters and groupings |
| **Granularity** | Monthly, hourly, or resource-level breakdown |
| **Scope** | High-level (all accounts) or drilled down per service/resource |
| **Savings Plan recommendations** | Suggests optimal Savings Plans based on your actual usage |
| **Usage forecast** | Predicts spend up to **18 months** based on historical usage |

---

## Use Cases

### Monthly Cost by Service
View cost per AWS service over time → identify expensive instance types → optimize sizing and usage.

### Hourly / Resource-Level Analysis
Drill down to individual EC2 instances hour-by-hour → understand cost spikes → right-size resources.

### Savings Plan Recommendations
Cost Explorer analyzes usage patterns → recommends Savings Plans → shows estimated monthly savings vs. current spend.

### Usage Forecasting
Based on historical cost data → forecasts expected spend with confidence intervals → up to 18 months ahead → useful for budget planning.

---

## Quick Reference

```
Cost Explorer: visualize + manage AWS costs
  Granularity: monthly → hourly → resource level
  Scope: all accounts or per service/resource

Key use cases (exam):
  Cost analysis     → monthly cost by service, identify expensive resources
  Savings Plans     → recommendations based on actual usage patterns
  Forecasting       → up to 18 months, based on historical usage

Note: most likely the only billing service tested on the SysOps exam
```
