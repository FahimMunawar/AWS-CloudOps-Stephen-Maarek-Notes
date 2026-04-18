# 1 — AWS Health Dashboard

## Overview

Two distinct parts under the AWS Health Dashboard:

| Dashboard | Old Name | Purpose |
|---|---|---|
| **Service History** | AWS Service Health Dashboard | General health of all AWS services across all regions |
| **Your Account Health** | AWS Personal Health Dashboard (PHD) | Alerts and guidance for events that impact **your** account specifically |

---

## Service History

- Shows health status for every AWS service in every region
- View past incidents by day
- **RSS feed** available to subscribe to updates
- General information — not personalized to your account

---

## Account Health Dashboard

- Provides alerts and **remediation guidance** for events impacting your resources
- Shows performance and availability for services **you are actually using**
- Delivers **proactive notifications** for scheduled maintenance activities
- Can **aggregate data across your entire AWS Organization**
- Access: top-right corner of the AWS console (bell icon area)
- **Global service**

### What It Shows

| Item | Detail |
|---|---|
| **Active events** | Outages or issues directly impacting your account |
| **Event log** | History of past events affecting your account |
| **Scheduled changes** | Upcoming AWS maintenance that may affect your resources |
| **Remediation guidance** | Steps to address the issue |

---

## Key Distinction

```
Service History       → general AWS-wide status (all services, all regions)
Account Health        → personalized view (only services/resources YOU use)

Example:
  EC2 issue in us-east-2
    Service History: might show a general EC2 degradation
    Account Health:  alerts only if YOUR EC2 instances are impacted
```

---

## SysOps Exam Q&A

**Q: Which dashboard shows the health of all AWS services globally, not specific to your account?**
A: **Service History** (formerly AWS Service Health Dashboard) — general status across all regions and services.

**Q: Which dashboard alerts you about events that directly impact your AWS resources?**
A: **Account Health Dashboard** (formerly AWS Personal Health Dashboard) — personalized alerts, remediation guidance, and scheduled maintenance notifications.

**Q: How do you get a single health view across all accounts in your AWS Organization?**
A: Enable Organization aggregation in the **Account Health Dashboard** — it aggregates health data for all member accounts.

---

## Quick Reference

```
Service History:
  General AWS service health (all regions, all services)
  RSS feed available
  No personalization

Account Health Dashboard (PHD):
  Personalized to your account and resources
  Alerts + remediation guidance + scheduled maintenance
  Global service
  Aggregate across AWS Organization
  Access: top-right console icon (bell area)
```
