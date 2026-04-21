# 2. Amazon SQS — Hands-On

## Overview

Walkthrough of creating an SQS Standard queue, sending and receiving messages, observing visibility timeout behavior, deleting messages, and exploring queue management options.

---

## Step 1: Create a Queue

**Console:** SQS → Create queue

| Setting | Value |
|---------|-------|
| Type | **Standard** (or FIFO — covered later) |
| Name | `DemoQueue` |
| Visibility timeout | Default (30 seconds) |
| Delivery delay | Default |
| Message retention | 4 days (max: 14 days) |
| Max message size | 256 KB |
| Encryption | **SSE-SQS** (Amazon SQS managed key — similar to SSE-S3) |

### Encryption Options

| Option | Detail |
|--------|--------|
| **Disabled** | No encryption |
| **SSE-SQS** | AWS-managed key for SQS (default, free) |
| **SSE-KMS** | Customer KMS key — choose `alias/aws/sqs` or your own CMK; set data key reuse period (e.g., 5 min) to reduce KMS API calls |

### Access Policy

- Resource-based policy similar to S3 bucket policy
- Define who can send messages and who can receive
- Options: queue owner only, or specific accounts/users/roles
- Generates a JSON policy document

---

## Step 2: Send and Receive Messages

**Console:** Queue → **Send and receive messages**

### Send a Message

1. Enter message body (e.g., `hello world!`)
2. Optionally add message attributes (key-value metadata)
3. Click **Send message**

→ Messages available count increases to 1

### Receive a Message

1. Click **Poll for messages**
2. Message appears in the list
3. Click message → **Details** to view:
   - Message ID
   - MD5 hash of body
   - Sent timestamp
   - **Receive count** (how many times this message has been received)
   - Size in bytes
   - Message body

### Visibility Timeout Behavior

> If you receive a message but do NOT delete it within the visibility timeout (default 30 seconds), the message becomes visible again and can be received by other consumers.

```
Message received (receive count: 1)
        ↓
Visibility timeout expires (30 seconds, not deleted)
        ↓
Message visible again in queue
        ↓
Polled again (receive count: 2, 3...)
```

### Delete a Message

- Select message → **Delete**
- Signals to SQS that the message has been successfully processed
- Message is permanently removed — no further deliveries

---

## Step 3: Queue Management Options

| Action | Detail |
|--------|--------|
| **Edit** | Modify visibility timeout, retention, encryption, etc. |
| **Purge** | Delete ALL messages in the queue — type `purge` to confirm; useful in development, avoid in production |
| **Delete queue** | Permanently remove the queue and all messages |

---

## Step 4: Monitoring

**Console:** Queue → **Monitoring** tab

| Metric | Use |
|--------|-----|
| **Messages available** | Current backlog size |
| **ApproximateAgeOfOldestMessage** | How long the oldest message has been waiting — useful for ASG scaling trigger |
| **Messages in flight** | Received but not yet deleted |
| **Messages deleted** | Throughput indicator |

---

## Key Observations from the Demo

| Observation | Explanation |
|-------------|-------------|
| Message received multiple times | Visibility timeout expired before deletion — at-least-once delivery |
| Receive count increments | Each time a message is polled without deletion |
| After DeleteMessage | Message never appears again — processing confirmed |
| Poll returns up to 10 messages | Standard poll behavior |

---

## Quick Reference

```
Create queue:
  Type: Standard | Name: DemoQueue
  Retention: 4 days | Max size: 256 KB
  Encryption: SSE-SQS (default) or SSE-KMS

Send: SQS → Queue → Send and receive → enter body → Send message
Receive: Poll for messages → view details → Delete when processed

Visibility timeout:
  If not deleted within timeout → message reappears → at-least-once delivery
  Default: 30 seconds

Purge queue: deletes ALL messages — development use only

Monitoring:
  ApproximateNumberOfMessages → drive ASG scale-out
  ApproximateAgeOfOldestMessage → alternative scaling signal
```

---

**File: 2_SQS_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
