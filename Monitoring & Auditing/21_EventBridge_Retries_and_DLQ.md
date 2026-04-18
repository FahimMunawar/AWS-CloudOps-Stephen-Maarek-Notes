# 21 — EventBridge: Retries and Dead Letter Queue (DLQ)

## Overview

When EventBridge cannot deliver an event to a target (target unavailable, network issue, etc.), it retries according to a configurable retry policy. Undeliverable events are sent to a Dead Letter Queue (DLQ) for later processing.

---

## Retry Policy

| Setting | Default | Configurable |
|---|---|---|
| **Maximum event age** | 24 hours | Yes |
| **Retry attempts** | 185 | Yes |

---

## Dead Letter Queue (DLQ)

- Implemented using **Amazon SQS**
- Receives events that could not be delivered after all retry attempts or after the maximum event age is exceeded
- Allows you to inspect and reprocess failed events

---

## Flow

```
EventBridge Rule fires → deliver to target
                              │
                         Delivery fails
                              │
                         Retry (up to 185 attempts / 24 hours)
                              │
                    All retries exhausted
                              │
                         → SQS Dead Letter Queue
```

---

## Quick Reference

```
Retry policy:
  Max event age:    24 hours (default)
  Retry attempts:   185 (default)

DLQ: SQS queue — stores undeliverable events for inspection and reprocessing
Failure causes: target unavailable, network issues
```
