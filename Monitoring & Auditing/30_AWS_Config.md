# 30 — AWS Config

## Overview

AWS Config audits and records the compliance and configuration history of your AWS resources. It answers questions like: are my security groups misconfigured? Have my resources changed? Are my resources compliant with company policies?

> **Config does NOT prevent actions** — it only evaluates and reports compliance. It does not replace IAM.

---

## Key Questions Config Answers

- Is there unrestricted SSH access to any security group?
- Do S3 buckets have public access enabled?
- Has an ALB configuration changed over time?

---

## Scope

- **Per-region service** — must configure separately per region
- Can **aggregate data across regions and accounts** into one central view
- Store configuration history in **S3** → query with **Athena**

---

## Config Rules

| Type | Detail |
|---|---|
| **AWS Managed Rules** | 75+ pre-built rules (e.g., check EBS type, check S3 public access) |
| **Custom Rules** | Defined using a **Lambda function** |

### Evaluation Triggers

| Trigger | Example |
|---|---|
| **On configuration change** | Evaluate EBS disk type whenever a new EBS volume is created |
| **Periodic** | Every 2 hours, check that all EBS disks are gp2 |

---

## Pricing

| Item | Cost |
|---|---|
| Configuration item recorded | $0.003 per item per region |
| Config rule evaluation | $0.001 per evaluation per region |

> No free tier — costs can grow quickly in large accounts.

---

## Resource Timeline View

For any resource, Config shows:
- **Compliance state over time** (compliant / non-compliant)
- **Configuration changes over time** (who changed what, when)
- **Linked CloudTrail** — view the API calls that caused each change

---

## Remediations

Config can automatically fix non-compliant resources using **SSM Automation Documents**.

```
Config Rule: IAM access key older than 90 days → non-compliant
  └── Auto-remediation: SSM Document RevokeUnusedIAMUserCredentials
        └── Deactivates expired access keys
```

- Use **AWS-managed SSM documents** or create your own
- Custom docs can invoke a **Lambda function** for arbitrary logic
- **Retry**: if still non-compliant after remediation, retry up to **5 times**

---

## Notifications

| Method | Use Case |
|---|---|
| **EventBridge** | Trigger on non-compliance → Lambda, SNS, SQS, etc. |
| **SNS (direct from Config)** | Send all config change + compliance notifications to SNS → filter with SNS filtering → admin email / Slack |

---

## SysOps Exam Q&A

**Q: Does AWS Config prevent non-compliant resources from being created?**
A: **No** — Config only evaluates and reports compliance. It does not block actions. Use IAM or SCPs for preventive controls.

**Q: How do you automatically fix non-compliant resources in AWS Config?**
A: Use **auto-remediation** with an **SSM Automation Document** — either AWS-managed (e.g., `RevokeUnusedIAMUserCredentials`) or a custom document. Retries up to **5 times**.

**Q: AWS Config is a global service. True or false?**
A: **False** — Config is **per-region**. You must enable it in each region. Data can be aggregated across regions and accounts.

**Q: How do you view who changed a resource's configuration in AWS Config?**
A: View the resource's **configuration timeline** in Config, then click through to the linked **CloudTrail** event for the API call details.

**Q: How do you query historical Config data stored in S3?**
A: Use **Amazon Athena** — serverless SQL queries on the S3 bucket where Config stores configuration history.

---

## Quick Reference

```
Config = audit + compliance recording for AWS resources
NOT preventive — does not block actions

Rules:
  AWS Managed: 75+ built-in
  Custom: Lambda function
  Triggers: on config change OR periodic interval

Remediations:
  SSM Automation Document (managed or custom)
  Can invoke Lambda for custom logic
  Retries: up to 5 times

Resource timeline: compliance history + config changes + CloudTrail API calls

Notifications:
  EventBridge → any downstream target
  SNS (direct) → filter → admin email / Slack

Storage: S3 → query with Athena
Scope: per-region; aggregate across regions/accounts for central view

Pricing: $0.003/config item + $0.001/rule evaluation (no free tier)
```
