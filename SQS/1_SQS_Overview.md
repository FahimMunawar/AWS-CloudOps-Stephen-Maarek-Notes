# 1. Amazon SQS Overview

## Overview

**Amazon SQS (Simple Queue Service)** is a fully managed message queuing service used to decouple application components. Producers send messages into a queue; consumers poll and process them. It is the oldest AWS service and the go-to solution for **application decoupling** on the exam.

---

## Core Concepts

```
Producer(s)  ──SendMessage──► SQS Queue ──PollMessages──► Consumer(s)
                               (buffer)                    (process + DeleteMessage)
```

| Role | Description |
|------|-------------|
| **Producer** | Sends messages to the queue using the `SendMessage` API |
| **Queue** | Buffers messages; decouples producers from consumers |
| **Consumer** | Polls the queue, processes messages, deletes them with `DeleteMessage` |

---

## SQS Standard Queue — Key Properties

| Property | Detail |
|----------|--------|
| **Throughput** | Unlimited — no messages-per-second limit |
| **Queue size** | Unlimited number of messages in the queue |
| **Message size** | Max **256 KB** per message |
| **Retention** | Default 4 days; maximum **14 days** |
| **Latency** | < 10 ms on publish and receive |
| **Delivery** | **At least once** — duplicates are possible |
| **Ordering** | **Best effort** — not guaranteed |
| **Max messages per poll** | 10 messages at a time |

---

## Producers

- Use the AWS SDK to call `SendMessage`
- Message content is arbitrary (e.g., order ID, customer ID, instructions)
- Message persists in queue until a consumer deletes it

---

## Consumers

- Run on EC2 instances, on-premises servers, or Lambda functions
- Poll SQS using `ReceiveMessage` — receive up to 10 messages at a time
- Process the message (e.g., insert into RDS)
- Call `DeleteMessage` to remove it from the queue
- Multiple consumers can process messages in parallel — horizontal scaling

---

## SQS + Auto Scaling Group Pattern (Exam Critical)

```
SQS Queue
    │
    │  CloudWatch Metric: ApproximateNumberOfMessages (queue length)
    ↓
CloudWatch Alarm (threshold exceeded)
    ↓
Auto Scaling Group scales OUT → adds more EC2 consumer instances
    ↓
Higher throughput, queue drains faster
```

- Automatically scales consumers based on queue depth
- Common architecture for bursty workloads

---

## Application Decoupling Architecture

**Example: Video Processing**

```
Users ──► Front-End App (ASG)
                │
         SendMessage (video processing request)
                ↓
           SQS Queue (unlimited buffer)
                ↓
Back-End Processing App (ASG) ──► Process video ──► S3 bucket
```

Benefits:
- Front-end and back-end scale independently
- Front-end uses CPU-optimized instances; back-end uses GPU instances
- Queue absorbs spikes — back-end processes at its own pace

---

## SQS Security

| Layer | Detail |
|-------|--------|
| **In-flight encryption** | HTTPS API — enabled by default |
| **At-rest encryption** | KMS keys |
| **Client-side encryption** | Supported — client handles encrypt/decrypt |
| **IAM policies** | Control access to SQS API calls |
| **SQS access policies** | Similar to S3 bucket policies — for cross-account access and service integrations (SNS, S3 events) |

---

## Best Practices

✓ **Use SQS for application decoupling** — the exam key phrase is "decouple applications"  
✓ **Scale consumers with ASG on ApproximateNumberOfMessages** — the standard autoscaling pattern  
✓ **Always delete messages after processing** — failure to delete causes reprocessing by other consumers  
✓ **Design consumers to be idempotent** — at-least-once delivery means the same message may arrive twice  
✓ **Use SQS access policies for cross-account or service-to-service** — when S3 or SNS need to write to the queue  

---

## SysOps Exam Focus

**Q1: "Your application has a front-end tier that receives video upload requests and a back-end tier that processes videos. Processing is slow and is impacting the user experience. What is the recommended architecture?"**
- A) Scale up the front-end EC2 instances with more CPU
- B) Decouple using SQS — front-end sends a message to the queue; back-end Auto Scaling group processes at its own pace
- C) Use ElastiCache to cache video processing results
- D) Move processing to Lambda with a 15-minute timeout
- **Answer: B** — SQS decouples the request (fast front-end) from the processing (slow back-end); both tiers scale independently

**Q2: "Your EC2 consumers are not keeping up with message volume. How should you scale?"**
- A) Increase the SQS message retention period
- B) Use the ApproximateNumberOfMessages CloudWatch metric to trigger an ASG scale-out alarm
- C) Increase the message size limit
- D) Enable SQS FIFO to improve throughput
- **Answer: B** — The queue depth metric is the correct signal to drive consumer ASG scaling

**Q3: "A consumer receives a message from SQS and processes it successfully but forgets to call DeleteMessage. What happens?"**
- A) The message expires after the retention period
- B) SQS automatically deletes messages that are received
- C) The message becomes visible again after the visibility timeout and may be received and processed again by another consumer
- D) The message is moved to the dead-letter queue
- **Answer: C** — Messages are not deleted automatically; failing to call DeleteMessage causes the message to reappear after the visibility timeout

**Q4: "You need to allow an S3 bucket to send event notifications directly to an SQS queue in a different AWS account. What must you configure?"**
- A) IAM role on the S3 bucket with SQS permissions
- B) SQS access policy on the queue granting cross-account access to S3
- C) VPC endpoint between S3 and SQS
- D) KMS key policy allowing S3 to write encrypted messages
- **Answer: B** — SQS access policies (resource-based policies) are used for cross-account and service-to-service access, just like S3 bucket policies

---

## Quick Reference

```
SQS Standard:
  Throughput: unlimited
  Message size: max 256 KB
  Retention: 4 days default, 14 days max
  Delivery: at least once (duplicates possible)
  Ordering: best effort (not guaranteed)
  Poll: up to 10 messages at a time

APIs:
  SendMessage    → producer sends to queue
  ReceiveMessage → consumer polls queue
  DeleteMessage  → consumer confirms processing complete

Scale pattern:
  ApproximateNumberOfMessages → CloudWatch Alarm → ASG scale out

Security:
  In-flight: HTTPS
  At-rest: KMS
  Access: IAM policies + SQS access policies (cross-account, S3/SNS)

Key exam phrase: "decouple applications" → SQS
```

---

**File: 1_SQS_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
