# 3. SQS Key Concepts

## Overview

Core SQS concepts tested on the CloudOps exam: visibility timeout, dead-letter queues, message retention, access policies, and the priority queue pattern.

---

## Message Visibility Timeout

| Property | Detail |
|----------|--------|
| **What it does** | After a message is polled, it becomes invisible to all other consumers for the timeout duration |
| **Default** | 30 seconds |
| **Purpose** | Gives the consumer time to process and delete the message before others can see it |
| **If not deleted in time** | Message becomes visible again → can be received by another consumer (at-least-once delivery) |

```
Consumer polls message → message hidden from others (30 sec)
        ├── Processed + DeleteMessage called → permanently removed
        └── Timeout expires without delete → message reappears in queue
```

---

## Dead Letter Queue (DLQ)

| Property | Detail |
|----------|--------|
| **Trigger** | Message exceeds the **Maximum Receive Threshold** (e.g., received 5 times without being deleted) |
| **Action** | Message is moved to a separate SQS queue designated as the DLQ |
| **Purpose** | Isolate problem messages for debugging — prevents poison messages from blocking the queue |
| **DLQ type** | Standard SQS queue (or FIFO if source is FIFO) |

```
Message polled → processing fails → message reappears
        ↓
Polled again → fails again (repeat N times)
        ↓
Receive count exceeds maxReceiveCount threshold
        ↓
Message moved to Dead Letter Queue
```

---

## Message Retention Period

| Property | Detail |
|----------|--------|
| **Range** | 1 minute to **14 days** |
| **Default** | **4 days** |
| **Behavior** | SQS automatically deletes messages that exceed the retention period |
| **Use** | Set longer retention for messages that may wait for slow consumers |

---

## SQS Access Policy

- Resource-based policy (same concept as S3 bucket policy)
- Controls who can send messages to and receive messages from the queue
- Required for:
  - **Cross-account access** — allow another AWS account to use the queue
  - **Service-to-queue** — allow S3 events or SNS to write to the queue

---

## Priority Queue Pattern

SQS does not support message priority natively within a single queue. Use **two separate queues**:

```
High priority messages  ──► High Priority SQS Queue
Low priority messages   ──► Low Priority SQS Queue

Consumer logic:
  1. Poll High Priority Queue first
  2. Process all high priority messages
  3. If High Priority Queue is empty → poll Low Priority Queue
  4. Process low priority messages
```

| Component | Detail |
|-----------|--------|
| **High priority queue** | Standard SQS queue — polled first |
| **Low priority queue** | Standard SQS queue — polled only when high priority is empty |
| **Consumer logic** | Must be implemented in application code |
| **Why two queues** | SQS has no built-in priority field — priority must be enforced at the consumer level |

---

## Best Practices

✓ **Set visibility timeout longer than your processing time** — prevents unnecessary redeliveries  
✓ **Configure a DLQ for all production queues** — catch and inspect poison messages without losing them  
✓ **Monitor DLQ message count** — a non-zero DLQ is a signal that something is failing in processing  
✓ **Set retention period based on consumer SLA** — if consumers can be down for hours, extend retention  
✓ **Use two queues for priority use cases** — implement poll order in consumer code  

---

## SysOps Exam Focus

**Q1: "A message in an SQS queue has been received 6 times but never deleted. What should be configured to prevent it from blocking the queue indefinitely?"**
- A) Increase the visibility timeout
- B) Set a Dead Letter Queue with a maxReceiveCount of 5 — after 5 failed attempts the message moves to the DLQ
- C) Enable FIFO queue ordering
- D) Purge the queue
- **Answer: B** — The DLQ maxReceiveCount threshold moves persistently failing messages out of the main queue for inspection

**Q2: "Your SQS consumer takes 45 seconds to process a message. The visibility timeout is 30 seconds. What is the result?"**
- A) The message is deleted after 30 seconds automatically
- B) The message becomes visible again after 30 seconds and may be received and processed a second time
- C) The consumer receives an error and the message is moved to the DLQ
- D) The queue pauses until the consumer finishes processing
- **Answer: B** — The visibility timeout expires before processing completes; the message reappears and risks duplicate processing — increase the timeout beyond 45 seconds

**Q3: "You need to process urgent messages before standard messages using SQS. How do you implement this?"**
- A) Set a priority attribute on each message in a single SQS queue
- B) Use FIFO queue with priority ordering enabled
- C) Create two separate SQS queues — consumers poll the high priority queue first, then the low priority queue when it is empty
- D) Use message delay to push low priority messages to the back
- **Answer: C** — SQS has no native priority feature; the two-queue pattern with consumer-side poll ordering is the correct solution

**Q4: "What is the maximum message retention period for SQS?"**
- A) 4 days
- B) 7 days
- C) 14 days
- D) 30 days
- **Answer: C** — SQS retains messages for a minimum of 1 minute and a maximum of 14 days; the default is 4 days

---

## Quick Reference

```
Visibility Timeout:
  Default: 30 seconds
  Message hidden from others while being processed
  Not deleted in time → reappears → at-least-once delivery
  Fix: set timeout > processing time

Dead Letter Queue (DLQ):
  Trigger: message receive count > maxReceiveCount
  Moves problem messages to a separate queue for debugging
  Monitor DLQ count in CloudWatch

Message Retention:
  Default: 4 days | Min: 1 min | Max: 14 days
  Auto-deleted after retention period

Access Policy:
  Resource-based (like S3 bucket policy)
  Use for: cross-account, S3 events → SQS, SNS → SQS

Priority Pattern:
  Two queues: high + low priority
  Consumer polls high first → low only when high is empty
  No native priority in SQS
```

---

**File: 3_SQS_Key_Concepts.md**
**Status: SysOps-focused, exam-ready, concise format**
