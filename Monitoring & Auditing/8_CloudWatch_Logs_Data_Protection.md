# 8 — CloudWatch Logs: Data Protection

## Overview

CloudWatch Logs Data Protection automatically detects and masks sensitive data in logs — both at-rest and in transit — using machine learning and configurable data identifiers.

---

## How It Works

```
Lambda function sends log:
  "User signed in with email foobar@example.com"
         │
         ▼
CloudWatch Log Group (with Data Protection Policy: email identifier)
         │
         ▼
Sensitive data masked:
  "User signed in with email ***@***.***"
```

---

## Data Identifiers

| Type | Detail |
|---|---|
| **Managed (AWS-provided)** | 100+ patterns: email, passwords, credit cards, SSNs, etc. |
| **Custom** | Define your own patterns for business-specific sensitive data |

---

## Audit & Alerting

### Audit reports — send findings to:
- Another CloudWatch log group
- Amazon S3
- Kinesis Data Firehose

### Alerting on findings:

```
Log Group (Data Protection Policy)
  └── Metric Filter on: LogEventsWithFindings
        └── CloudWatch Alarm
              └── SNS → Alert
```

The metric **`LogEventsWithFindings`** increments whenever sensitive data is detected.

---

## Where Masking Applies

Sensitive data is masked in:
- **CloudWatch Logs Insights** queries
- **Metric filters**
- **Subscription filters**

> Only users with the **unmask permission** (`logs:Unmask`) can view the original unmasked data.

---

## SysOps Exam Q&A

**Q: What is CloudWatch Logs Data Protection?**
A: A feature that uses ML to detect and mask sensitive data (emails, SSNs, credit cards, etc.) in CloudWatch Logs at-rest and in transit, based on a configurable data protection policy.

**Q: How do you get alerted when sensitive data is found in CloudWatch Logs?**
A: Create a metric filter on **`LogEventsWithFindings`** → CloudWatch Alarm → SNS notification.

**Q: Who can view unmasked sensitive data in CloudWatch Logs?**
A: Only users with the **`logs:Unmask`** IAM permission.

**Q: Where can audit findings from a data protection policy be sent?**
A: Another **CloudWatch log group**, **Amazon S3**, or **Kinesis Data Firehose**.

---

## Quick Reference

```
Data Protection Policy = ML-based masking of sensitive data in CloudWatch Logs
Data identifiers: 100+ managed (email, SSN, credit card) + custom
Masking applies to: Logs Insights, metric filters, subscription filters
Unmask permission: logs:Unmask (restricted users only)

Alert on findings:
  Metric: LogEventsWithFindings → CloudWatch Alarm → SNS

Audit reports → another log group / S3 / Kinesis Firehose
```
