# 2. Amazon SNS — Hands-On

## Overview

Walkthrough of creating an SNS Standard topic, subscribing an email endpoint, confirming the subscription, publishing a test message, and cleaning up.

---

## Step 1: Create a Topic

**Console:** SNS → Topics → **Create topic**

| Setting | Standard | FIFO |
|---------|---------|------|
| **Ordering** | Best-effort | Strictly preserved |
| **Delivery** | At least once | Exactly once |
| **Throughput** | Unlimited | Up to 300 publishes/sec |
| **Subscribers** | SQS, Lambda, HTTPS, SMS, Email, Mobile | **SQS only** |
| **Name requirement** | Any name | Must end with `.fifo` |

For this demo: **Standard**, name `MyFirstTopic`

Other options:
- **Encryption** — enable KMS for at-rest encryption
- **Access policy** — defines who can publish to the topic (similar to S3 bucket policy / SQS access policy)

---

## Step 2: Create a Subscription

**Console:** Topic → **Create subscription**

| Setting | Value |
|---------|-------|
| Protocol | Email (options: Kinesis Firehose, SQS, Lambda, Email, Email-JSON, HTTP, HTTPS, SMS) |
| Endpoint | Your email address |
| Subscription filter policy | Optional — filter which messages reach this subscriber |

> **Subscription filter policy:** Allows a subscriber to receive only a subset of messages. Useful when multiple subscribers need different subsets of the same topic's messages.

Click **Create subscription** → status shows **Pending confirmation**

---

## Step 3: Confirm the Subscription

1. Check your email inbox for an AWS notification
2. Click **Confirm subscription** in the email
3. Refresh the console — subscription status changes to **Confirmed**

> Subscriptions remain pending until confirmed by the endpoint owner. Unconfirmed subscriptions do not receive messages.

---

## Step 4: Publish a Message

**Console:** Topic → **Publish message**

| Setting | Value |
|---------|-------|
| Subject | (optional) |
| Message body | `hello world` |

Click **Publish message** → email arrives at the subscribed address with the message body.

---

## Step 5: SQS Fan-Out (Pattern Reference)

To implement fan-out:
1. Create multiple SQS queues
2. For each queue, create an SNS subscription with protocol = **SQS**
3. One SNS publish → all SQS queues receive the message

```
SNS Topic
    ├──► SQS Queue A (subscriber 1)
    ├──► SQS Queue B (subscriber 2)
    └──► SQS Queue C (subscriber 3)
```

---

## Step 6: Cleanup

1. Subscription → **Delete**
2. Topic → **Delete** → type `delete me` → confirm

---

## Protocol Reference (Exam)

| Protocol | Delivery Target |
|----------|----------------|
| Email | Email address (human-readable) |
| Email-JSON | Email address (raw JSON) |
| HTTP / HTTPS | Webhook endpoint |
| SMS | Mobile phone number |
| SQS | SQS queue (fan-out) |
| Lambda | Lambda function |
| Kinesis Data Firehose | Stream to S3, Redshift |
| Mobile push | GCM (Android), APNS (Apple), ADM |

---

## Quick Reference

```
Create topic:
  Standard → unlimited throughput, all subscriber types
  FIFO → ordered, exactly-once, SQS only, name ends in .fifo

Create subscription:
  Protocol: Email / SQS / Lambda / HTTP(S) / SMS / Firehose
  Confirm: click link in confirmation email (pending → confirmed)

Filter policy: optional — subscriber receives only matching messages

Publish: Topic → Publish message → body → Publish

Fan-out pattern:
  One SNS topic → multiple SQS queues as subscribers

Cleanup: delete subscription → delete topic (type "delete me")
```

---

**File: 2_SNS_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
