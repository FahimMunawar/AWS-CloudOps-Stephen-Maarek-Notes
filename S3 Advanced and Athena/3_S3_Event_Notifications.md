# 3 — S3 Event Notifications

## Overview

S3 Event Notifications let you react automatically to events happening in an S3 bucket by routing them to SNS, SQS, Lambda, or EventBridge.

---

## Event Types

| Event | Description |
|---|---|
| **ObjectCreated** | Object uploaded, copied, or multipart upload completed |
| **ObjectRemoved** | Object deleted (including delete markers) |
| **ObjectRestored** | Object restored from Glacier |
| **Replication** | Replication events (missed threshold, failed, etc.) |

- Can **filter by object name** (prefix or suffix) — e.g., only objects ending in `.jpg`
- Typically delivered **within seconds**, but can take up to a minute or longer

---

## Destinations (4 options)

### Option 1 — Direct Targets (Classic)

```
S3 Event
   ├──► SNS Topic   (resource access policy on SNS)
   ├──► SQS Queue   (resource access policy on SQS)
   └──► Lambda      (resource policy on Lambda function)
```

### Option 2 — Amazon EventBridge (Enhanced)

```
S3 Event ──► EventBridge (ALL events, automatically) ──► 18+ AWS services
```

---

## IAM Permissions Required

S3 uses **resource-based policies** on the destination — not IAM roles.

| Destination | Permission Type | What It Does |
|---|---|---|
| **SNS Topic** | SNS Resource Access Policy | Allows S3 to publish messages to the topic |
| **SQS Queue** | SQS Resource Access Policy | Allows S3 to send messages to the queue |
| **Lambda** | Lambda Resource Policy | Allows S3 to invoke the function |

> S3 does not assume an IAM role to send events. Instead, the **target resource must grant S3 permission** via its own resource policy — same model as bucket policies.

---

## EventBridge Integration

All S3 events automatically flow into **Amazon EventBridge**, regardless of whether you configure a direct destination.

### Advantages over direct targets

| Feature | Direct (SNS/SQS/Lambda) | EventBridge |
|---|---|---|
| Number of destinations | 1 per rule | 18+ services |
| Filtering | Prefix/suffix only | Metadata, object size, object name |
| Multiple targets | No | Yes (fan-out) |
| Reliability | Standard | Archive + replay events |
| Extra destinations | — | Step Functions, Kinesis Streams, Firehose, and more |

---

## Architecture Examples

### Basic: Generate thumbnails on image upload
```
S3 (*.jpg uploaded)
       │
       ▼
  Lambda Function → generates thumbnail → stores in S3
```
- Direct S3 → Lambda notification with `.jpg` suffix filter
- Lambda resource policy grants S3 invoke permission

### Advanced: Fan-out with EventBridge
```
S3 Event → EventBridge
               ├──► Lambda (generate thumbnail)
               ├──► Kinesis (stream event data)
               └──► Step Functions (workflow trigger)
```

---

## Best Practices

- Use **direct targets** (SNS/SQS/Lambda) for simple, single-destination event routing
- Use **EventBridge** when you need multiple destinations, advanced filtering, or event replay
- Always configure the **resource policy on the destination**, not an IAM role on S3
- Add suffix filters (e.g., `.jpg`, `.csv`) to avoid triggering on every object event

---

## SysOps Exam Q&A

**Q: What are the four possible destinations for S3 Event Notifications?**
A: **SNS**, **SQS**, **Lambda function**, and **Amazon EventBridge**.

**Q: How does S3 get permission to send events to an SQS queue?**
A: By attaching an **SQS resource access policy** to the queue that allows the S3 service principal to send messages. S3 does not use an IAM role.

**Q: You need S3 events to trigger multiple AWS services simultaneously. What should you use?**
A: **Amazon EventBridge** — it receives all S3 events automatically and can route them to 18+ destinations simultaneously with advanced filtering.

**Q: What is the difference between filtering in direct S3 event notifications vs EventBridge?**
A: Direct notifications support **prefix and suffix** filtering only. EventBridge supports filtering by **metadata, object size, and object name** — much more granular.

**Q: An S3 notification is configured to invoke Lambda but it fails with an access error. What is the likely fix?**
A: Add a **Lambda resource policy** granting `lambda:InvokeFunction` permission to the `s3.amazonaws.com` service principal for the specific bucket.

---

## Quick Reference

```
Direct targets:   SNS, SQS, Lambda
EventBridge:      ALL events flow here automatically; routes to 18+ services

Permissions (resource-based policies, NOT IAM roles):
  SNS  → SNS resource access policy
  SQS  → SQS resource access policy
  Lambda → Lambda resource policy

EventBridge advantages:
  - Multiple simultaneous destinations
  - Advanced filtering (metadata, size, name)
  - Event archive and replay
  - Access to Step Functions, Kinesis, Firehose, etc.

Delivery: seconds typically; up to 1 minute possible
```
