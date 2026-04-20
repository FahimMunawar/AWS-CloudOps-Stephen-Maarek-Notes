# 16 — Cost Allocation Tags & Cost and Usage Reports

## Cost Allocation Tags

Used to separate and track AWS costs at a detailed level in cost reports.

### Two Types

| Type | Prefix | Example | Detail |
|---|---|---|---|
| **AWS-generated** | `aws:` | `aws:createdBy` | Auto-applied by AWS; shows who created a resource |
| **User-defined** | `user:` | `user:Environment`, `user:Owner` | Defined and applied by your team |

### Activation

**Cost Management → Cost Allocation Tags**
- Enable individual tags (AWS-generated or user-defined) → they appear in cost reports
- Tags must be **activated** before they appear in reports
- Tags must also be **applied to resources** (e.g., EC2 instance tag `Environment=dev`)

### Use Case

```
Tag EC2 instances with: Environment=dev / test / prod
Activate "Environment" as cost allocation tag
  → Cost report separates spend by dev / test / prod
```

---

## Cost and Usage Reports (CUR)

Most comprehensive AWS cost and usage data available.

| Feature | Detail |
|---|---|
| **Content** | All services used, pricing, reservations (RI), metadata |
| **Granularity** | Hourly or daily line items |
| **Tags included** | All activated cost allocation tags |
| **Delivery** | Daily export to **S3** |
| **Analysis tools** | **Athena**, **Redshift**, **QuickSight** |

### Setup

**Cost and Usage Reports → Create report**
1. Report name, optional resource IDs, auto-refresh
2. Specify S3 bucket (new or existing) + region
3. Time granularity: Hourly or Daily
4. Report versioning: Create new version or Overwrite
5. Optional integrations: Athena / Redshift / QuickSight + compression format

> Reports can take up to **24 hours** to start delivering after creation.

---

## AWS Usage Report (per service)

**Cost Management → AWS Usage Report**
- Select a service (e.g., Amazon EC2)
- Choose usage types, billing period, granularity (by day/hour)
- Download as CSV: columns include Operation, UsageType, Service, StartTime, EndTime, UsageValue

---

## Quick Reference

```
Cost Allocation Tags:
  AWS-generated: prefix "aws:" (e.g., aws:createdBy) — auto-applied
  User-defined:  prefix "user:" — you define and apply manually
  Must be activated in Cost Management console
  Must be applied to resources before appearing in reports

Cost and Usage Reports (CUR):
  Most comprehensive billing data
  Granularity: hourly or daily
  Delivery: S3 (daily export)
  Analysis: Athena / Redshift / QuickSight
  Setup time: up to 24 hours to start

Usage Report: per-service CSV download (Operation, UsageType, UsageValue, time)
```
