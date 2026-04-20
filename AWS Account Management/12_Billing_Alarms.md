# 12 — Billing Alarms

## Key Facts

| Fact | Detail |
|---|---|
| **Billing data region** | `us-east-1` only (CloudWatch → Alarms → Billing) |
| **Scope** | Represents **worldwide** costs — not region-specific |
| **Type** | Actual costs incurred — not projected or per-project |

---

## Prerequisites

1. Go to **Billing Dashboard → Billing Preferences → Enable "Receive Billing Alerts"**
2. Wait **~15 minutes** (or a few hours) for billing metrics to populate in CloudWatch

> Billing metrics will NOT appear in CloudWatch until this is enabled.

---

## Creating a Billing Alarm

**CloudWatch (us-east-1) → Alarms → Billing → Create alarm → Select metric**

### Metric Options

| Level | Metric |
|---|---|
| **Overall account spend** | Billing → Total Estimated Charges |
| **Per service** | Billing → By Service → e.g., Amazon EC2, Amazon S3, Config |

### Example Alarm

```
Metric: Total Estimated Charges
Condition: Greater than $10 USD (static threshold)
Action: Send to SNS topic → email notification
```

---

## Quick Reference

```
Billing alarms → CloudWatch in us-east-1 ONLY
Worldwide costs (not per-region)
Actual spend (not projected)

Setup:
  1. Billing Preferences → Enable Billing Alerts
  2. Wait ~15 min for data to appear
  3. CloudWatch (us-east-1) → Alarms → Billing → Create alarm

Alarm levels:
  Total Estimated Charges → overall account spend
  By Service              → per-service spend (EC2, S3, Config, etc.)
```
