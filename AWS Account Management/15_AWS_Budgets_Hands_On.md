# 15 — AWS Budgets: Hands-On

## Access

Billing Console → left sidebar → **Budgets** → **Create a budget**

---

## Two Creation Modes

| Mode | Detail |
|---|---|
| **Templates (simplified)** | Zero Spend, Monthly Cost, Daily Savings Plan coverage — quick setup, limited options |
| **Customize (advanced)** | Full control over all budget types, filters, alerts, and actions |

---

## Cost Budget Setup (Customize)

| Setting | Options |
|---|---|
| **Period** | Daily / Monthly / Quarterly / Annually |
| **Budget type** | Recurring (auto-renews) or Expiring (ends at period end) |
| **Method** | Fixed amount or auto-adjusting |
| **Budget amount** | e.g., $30/month |

### Filters (Budget Scope)

- All AWS services (default)
- Filter by: **Service** (EC2, KMS, etc.), **Region**, **Instance type**, **Usage type**, **Tags**, **Availability Zone**
- Multiple dimensions can be combined

### Cost Types

- Unblended / Amortized / Blended costs
- Include/exclude: refunds, credits

---

## Budget Alerts

Up to **5 alerts per budget**:

| Alert Type | Trigger |
|---|---|
| Actual | When actual spend reaches % of budgeted amount (e.g., 80%) |
| Forecast | When forecasted spend is on track to reach % of budget |

### Notification Channels

- Email recipients
- **Amazon SNS** topic
- **Amazon Chatbot** → Slack or Amazon Chime

---

## Budget Actions

When an alert threshold is met, automatically trigger an action:

| Action | Effect |
|---|---|
| **Attach IAM policy** | Restrict users/groups/roles (e.g., block EC2 launches) |
| **Attach SCP** | Apply guardrail to an org root or OU |
| **Stop EC2 instances** | Stop running instances in a specified region |
| **Stop RDS instances** | Stop running RDS instances in a specified region |

> Requires an IAM role with budget actions permissions (create role for AWS service: Budgets).

---

## All Budget Types (Customize)

| Type | Track |
|---|---|
| **Cost** | Dollar spend vs. budgeted amount |
| **Usage** | Specific usage metrics (e.g., Data Transfer Inter-AZ) |
| **Savings Plan** | Utilization or coverage of your Savings Plan |
| **Reservation** | Utilization or coverage of Reserved Instances (EC2, RDS, etc.) |

---

## Quick Reference

```
Budgets access: Billing Console → Budgets

Cost budget key settings:
  Period: daily/monthly/quarterly/annually
  Recurring vs. expiring
  Filters: service, region, instance type, tags, AZ

Alerts:
  Actual: triggered when actual spend hits %
  Forecast: triggered when on track to hit %
  Channels: email / SNS / Chatbot (Slack, Chime)
  Max: 5 alerts per budget

Actions on threshold:
  Attach IAM policy (restrict users)
  Attach SCP (restrict OU/org)
  Stop EC2 / Stop RDS instances
  Requires IAM role with Budget Actions permissions

4 budget types: Cost, Usage, Savings Plan, Reservation
```
