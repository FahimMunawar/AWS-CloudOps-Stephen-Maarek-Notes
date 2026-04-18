# 28 — CloudTrail + EventBridge Integration

## Overview

Every AWS API call is logged by CloudTrail **and** delivered as an event to EventBridge. This lets you create rules that react to any API call in your account.

---

## How It Works

```
User makes API call (e.g., DeleteTable)
  → Logged in CloudTrail
  → Event appears in EventBridge (default event bus)
    → EventBridge rule matches the event
      → Target: SNS, Lambda, etc.
```

---

## Example Patterns

| API Call | Service | Use Case |
|---|---|---|
| `DeleteTable` | DynamoDB | Alert when a DynamoDB table is deleted |
| `AssumeRole` | IAM (STS) | Alert whenever a role is assumed in your account |
| `AuthorizeSecurityGroupIngress` | EC2 | Alert when inbound security group rules are modified |

---

## Quick Reference

```
Any API call → CloudTrail → EventBridge → rule → action (SNS, Lambda, etc.)

Key examples:
  DeleteTable (DynamoDB)              → alert on table deletion
  AssumeRole (IAM/STS)               → alert on role assumption
  AuthorizeSecurityGroupIngress (EC2) → alert on SG inbound rule changes

Pattern: CloudTrail captures it → EventBridge routes it → SNS/Lambda reacts
```
