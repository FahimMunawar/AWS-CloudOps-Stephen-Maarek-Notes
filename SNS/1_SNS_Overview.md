# 1. Amazon SNS Overview

## Overview

**Amazon SNS (Simple Notification Service)** implements the **Pub/Sub (Publish-Subscribe)** pattern. A single producer publishes one message to an SNS topic, and all subscribers receive it automatically — eliminating point-to-point integrations.

---

## The Problem SNS Solves

**Without SNS (direct integration):**
```
Buying Service ──► Email notification (write integration)
Buying Service ──► Fraud service (write integration)
Buying Service ──► Shipping service (write integration)
Buying Service ──► SQS Queue (write integration)
= N integrations to maintain
```

**With SNS (Pub/Sub):**
```
Buying Service ──► SNS Topic ──► Email
                          ├──► Fraud service
                          ├──► Shipping service
                          └──► SQS Queue
= 1 integration, unlimited subscribers
```

---

## How SNS Works

| Component | Role |
|-----------|------|
| **Producer** | Publishes messages to one SNS topic |
| **SNS Topic** | The channel — receives and fans out messages |
| **Subscribers** | Receive all messages published to the topic (unless filtered) |

---

## SNS Limits

| Limit | Value |
|-------|-------|
| Subscriptions per topic | Up to 12,500,000+ |
| Topics per account | Up to 100,000 (soft limit) |

---

## Subscriber Types

| Subscriber | Description |
|------------|-------------|
| **Email** | Direct email delivery |
| **SMS / Mobile push** | Text messages and mobile app notifications (GCM, APNS, ADM) |
| **HTTP / HTTPS endpoint** | Webhook-style delivery |
| **Amazon SQS** | Fan-out to queues |
| **AWS Lambda** | Trigger serverless functions |
| **Kinesis Data Firehose** | Stream to S3, Redshift, etc. |

---

## AWS Services That Publish to SNS

SNS is the standard notification channel for many AWS services:

- CloudWatch Alarms
- Auto Scaling Group notifications
- CloudFormation state changes
- AWS Budgets
- S3 bucket events
- RDS events
- DynamoDB, DMS, Lambda, and more

> Any time an AWS service needs to notify you of an event, it sends to SNS.

---

## Publishing to SNS

### Standard (Topic Publish)

```
1. Create SNS topic
2. Create one or more subscriptions
3. Publish message to topic → all subscribers receive it
```

### Direct Publish (Mobile Apps)

```
1. Create platform application (GCM / APNS / ADM)
2. Create platform endpoint
3. Publish to endpoint → delivered to specific mobile device
```

---

## Security

| Layer | Detail |
|-------|--------|
| **In-flight encryption** | HTTPS API — enabled by default |
| **At-rest encryption** | KMS keys |
| **Client-side encryption** | Client responsibility — encrypt before publish, decrypt on receive |
| **IAM policies** | Control access to all SNS API calls |
| **SNS access policies** | Resource-based (like S3 bucket policy) — for cross-account access and service-to-SNS writes (e.g., S3 events) |

---

## Message Filtering

By default, every subscriber receives **every message** published to the topic.

A **subscription filter policy** (JSON document) can restrict which messages a subscriber receives.

```json
{
  "order_type": ["electronics"]
}
```

| Property | Detail |
|----------|--------|
| **Scope** | One filter policy per subscriber |
| **Format** | JSON — define which attribute values to accept |
| **Default** | No filter = receives all messages |
| **Per subscriber** | Each subscriber can have a different filter policy |

```
SNS Topic (all order events)
    ├──► SQS Queue A  (filter: order_type = electronics)  → electronics only
    ├──► SQS Queue B  (filter: order_type = clothing)     → clothing only
    └──► Lambda        (no filter)                         → all messages
```

---

## Cross-Region Delivery

SNS can deliver messages to subscribers in **different AWS regions**.

| Subscriber | Cross-Region Support |
|------------|---------------------|
| SQS Queue | Yes — queue in any region |
| Lambda | Yes — function in any region |

**Requirement:** The subscriber's resource-based policy must allow the SNS topic from the source region to publish to it.

```
us-east-1 SNS Topic ──► eu-west-2 SQS Queue
                         (SQS access policy must allow sns.amazonaws.com from us-east-1)
```

---

## Best Practices

✓ **Use SNS for fan-out** — one message to many receivers without managing N integrations  
✓ **Combine SNS + SQS (fan-out pattern)** — SNS fans out to multiple SQS queues for parallel processing  
✓ **Use message filtering** — subscribers can receive only the messages relevant to them  
✓ **Use SNS access policies for cross-account and S3 event delivery** — resource-based policy just like SQS  
✓ **Use Lambda subscriber for serverless processing** — no infrastructure needed for message handling  

---

## SysOps Exam Focus

**Q1: "You want to send an order event to an email team, a fraud detection service, and an SQS queue simultaneously with a single publish call. What should you use?"**
- A) Three separate SQS queues with independent producers
- B) Amazon SNS topic with three subscribers: email, HTTP endpoint (fraud), SQS queue
- C) EventBridge with three rules
- D) Direct Lambda invocation with fan-out logic
- **Answer: B** — SNS fan-out delivers one message to all subscribers simultaneously with a single publish

**Q2: "An S3 bucket needs to write event notifications to an SNS topic in a different AWS account. What must be configured?"**
- A) IAM role on S3 with SNS publish permissions
- B) SNS access policy on the topic granting the S3 service in the source account permission to publish
- C) VPC endpoint between S3 and SNS
- D) KMS key policy allowing S3 to encrypt SNS messages
- **Answer: B** — SNS access policies (resource-based) are used for service-to-SNS writes and cross-account access

**Q3: "What is the key difference between SNS and SQS?"**
- A) SNS stores messages for up to 14 days; SQS does not store messages
- B) SQS is pull-based (consumers poll); SNS is push-based (topic pushes to all subscribers immediately)
- C) SQS supports fan-out; SNS only supports one subscriber
- D) SNS requires consumers to delete messages; SQS does not
- **Answer: B** — SQS: consumers poll and delete; SNS: publisher sends once and all subscribers receive immediately

---

## Quick Reference

```
SNS = Pub/Sub model
  Producer publishes to SNS topic → all subscribers receive

Subscriber types:
  Email, SMS, HTTP/HTTPS, SQS, Lambda, Kinesis Firehose

AWS services → SNS:
  CloudWatch Alarms, ASG, CloudFormation, Budgets, S3, RDS...

Publish methods:
  Topic Publish (standard) → SDK publish to topic
  Direct Publish (mobile)  → platform endpoint → GCM/APNS/ADM

Security:
  In-flight: HTTPS (default)
  At-rest: KMS
  Access: IAM policies + SNS access policies (cross-account, S3→SNS)

SNS vs SQS:
  SQS = pull (consumer polls) | SNS = push (topic delivers to all)
  SQS = one consumer per message | SNS = all subscribers get the message
```

---

**File: 1_SNS_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
