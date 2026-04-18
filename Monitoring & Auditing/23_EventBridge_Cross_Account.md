# 23 — EventBridge Cross-Account Event Delivery

## Overview

EventBridge can send events to targets in a different AWS account. Both sides require explicit permissions — the source must be allowed to send, and the destination must be allowed to receive.

---

## Pattern 1 — EventBridge Bus → EventBridge Bus (cross-account)

| Side | What's Required |
|---|---|
| **Source account bus** | Resource policy allowing `events:PutEvents` to the destination bus |
| **Destination account bus** | Resource policy allowing the source account's root to `PutEvents` on this bus |

---

## Pattern 2 — EventBridge Bus → SQS Queue (cross-account)

| Side | What's Required |
|---|---|
| **Source account** | IAM Role with permission to `sqs:SendMessage` on the destination queue |
| **Destination account queue** | Resource-based policy allowing the source account to `SendMessage` |

---

## Applies To

The same two-sided permission pattern applies to all cross-account targets that support resource-based policies:

**SQS, SNS, Lambda, API Gateway, Kinesis Data Streams**

---

## Quick Reference

```
Cross-account delivery = both sides need explicit permissions:
  Source side: IAM Role (or bus policy) allowing send/put to destination
  Destination side: resource-based policy allowing source account to send/put

Bus → Bus:   source bus policy + destination bus policy
Bus → SQS:   source IAM role (sqs:SendMessage) + destination SQS resource policy
Same pattern: SNS, Lambda, API Gateway, Kinesis Data Streams
```
