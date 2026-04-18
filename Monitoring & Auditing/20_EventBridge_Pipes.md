# 20 — EventBridge Pipes

## Overview

EventBridge Pipes connects data sources to targets with optional filtering and enrichment — **no code required**.

---

## Architecture

```
Source (pulled by Pipe)
  └── Filter (optional — keep only matching events)
        └── Enrichment (optional — transform/augment data)
              └── Target (destination)
```

---

## Sources (Pipe pulls from these)

DynamoDB Streams, Kinesis Data Streams, Amazon MQ, Amazon MSK, Amazon SQS, Apache Kafka

---

## Enrichment Services (optional)

Lambda, Step Functions, API Gateway, EventBridge API Destination

---

## Targets

Kinesis Data Firehose, Kinesis Data Stream, EventBridge Bus, Redshift, SQS, SNS, API Gateway, ECS Task, and all other EventBridge targets

---

## vs EventBridge Rules

| Feature | EventBridge Rules | EventBridge Pipes |
|---|---|---|
| **Source** | AWS service events (push) | Streaming/queue sources (pull) |
| **Code required** | No | No |
| **Filtering** | Event pattern | Built-in filter step |
| **Enrichment** | Not built-in | Built-in enrichment step |

---

## Quick Reference

```
Pipes = no-code source → [filter] → [enrich] → target

Sources: DynamoDB Streams, Kinesis, MQ, MSK, SQS, Kafka
Enrich:  Lambda, Step Functions, API Gateway, API Destination
Targets: Any EventBridge target (Firehose, KDS, SQS, SNS, ECS, Redshift, etc.)

Use case: move/transform data between AWS services without writing consumer code
```
