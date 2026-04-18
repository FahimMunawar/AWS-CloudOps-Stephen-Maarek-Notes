# 3 — AWS Health + EventBridge Integration

## Overview

AWS Health Dashboard events can trigger EventBridge rules, enabling automated notifications and remediation.

```
AWS Health event → EventBridge → Lambda / SNS / SQS / Kinesis / etc.
```

---

## Event Types

| Type | Description |
|---|---|
| **Account events** | Events affecting resources within your specific account |
| **Public events** | Regional availability events for AWS services (not account-specific) |

---

## EventBridge Targets

Any standard EventBridge target is available:
- Lambda
- SNS
- SQS
- Kinesis Data Streams
- EC2 actions
- And more

---

## Example Patterns

### 1. EC2 Scheduled Maintenance → Email Notification

```
Health: EC2 instances scheduled for update
  → EventBridge rule
    → SNS topic → email notification
```

### 2. Exposed IAM Access Keys → Auto-Delete

```
Health: exposed IAM access key detected
  → EventBridge rule
    → Lambda function
      → DeleteAccessKey API call → key removed automatically
```

### 3. EC2 Instance Scheduled for Retirement → Auto-Restart

```
Health: EC2 instance scheduled for retirement
  → EventBridge rule
    → EC2 action: restart instance
      → Instance restarted → issue resolved
```

---

## Use Cases Summary

| Goal | Pattern |
|---|---|
| Notify on scheduled maintenance | Health → EventBridge → SNS → email |
| Capture event data | Health → EventBridge → Kinesis / S3 |
| Auto-remediate exposed IAM keys | Health → EventBridge → Lambda → DeleteAccessKey |
| Auto-restart retiring EC2 instances | Health → EventBridge → EC2 restart action |

---

## Quick Reference

```
Health + EventBridge: react to account and public AWS health events

Account events → affect YOUR resources
Public events  → regional service availability

Automation patterns:
  Exposed IAM keys      → Lambda → delete keys
  EC2 retirement        → EC2 action → restart instance
  Scheduled maintenance → SNS → email alert

All standard EventBridge targets are available (Lambda, SNS, SQS, Kinesis, EC2 actions)
```
