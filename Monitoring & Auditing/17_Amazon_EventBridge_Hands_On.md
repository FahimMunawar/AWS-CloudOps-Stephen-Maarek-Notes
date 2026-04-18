# 17 — Amazon EventBridge: Hands-On

## Four Main Options in the Console

| Option | Purpose |
|---|---|
| **Rule with event pattern** | React to events from AWS services |
| **Schedule rule** | Old-style cron/rate-based trigger |
| **EventBridge Scheduler** | New dedicated scheduling service |
| **Pipe** | Source → optional filter/enrichment → target |
| **Schema Registry** | Browse event schemas for all AWS events |

---

## Demo 1 — Event Pattern Rule: Notify on EC2 Shutdown/Termination

### Steps

1. **Create rule** → Event pattern
2. Source: **Service Events** → EC2 → `EC2 Instance State-change Notification`
3. View the event schema and sample events to identify filter values:
   - Sample event 5: `"state": "shutting-down"`
   - Sample event 6: `"state": "terminated"`
4. Add filter on `state` field → **Equal** → values: `shutting-down`, `terminated`
5. Target: **SNS Topic** → select `demo-topic`
6. IAM role: **auto-created** by EventBridge (permission to publish to SNS)
7. Configure retry policy and optional dead-letter queue
8. Name: `NotifyEC2InstanceShutdownOrTerminated` → **Create**

### Result
Any EC2 state change to `shutting-down` or `terminated` triggers an SNS notification.

---

## Demo 2 — Schedule Rule: Invoke Lambda Every Hour

1. Left sidebar → **Schedules** → **Create schedule**
2. Name: `InvokeLambdaEveryHour`
3. Schedule type: **Recurring** → **Rate-based** → `1 hour`
4. Flexible time window: **Off**
5. Target: **Invoke Lambda function** → select your function
6. Create

---

## Event Buses

| Bus | Description |
|---|---|
| **Default** | AWS-generated service events |
| **Custom** | Create your own — applications send events via `PutEvents` |

- **Archive**: capture events from a bus, store with indefinite or set retention
- **Replay**: re-send archived events (useful for debugging)

---

## Other Console Features

| Feature | Notes |
|---|---|
| **Partner event sources** | Ingest events from SaaS partners (Auth0, Datadog, etc.) directly into EventBridge |
| **API destinations** | Connect EventBridge to an external HTTP endpoint (outside AWS) |
| **Schema Registry** | Browse schemas for all AWS service events or register your own custom schemas |

---

## Quick Reference

```
Event pattern rule: Service Events → filter on event fields → target (SNS, Lambda, etc.)
  IAM role auto-created when targeting AWS services

Schedule:
  Rate-based (e.g., every 1 hour) or Cron-based
  Target: Lambda, ECS, Kinesis Firehose, and more

Event buses:
  Default = AWS services
  Custom  = your own apps via PutEvents

Archive + Replay: store events → replay for debugging

Partner sources: Auth0, Datadog, etc. → direct EventBridge integration
API destinations: EventBridge → external HTTP endpoint
Schema Registry: understand event shape before building rules
```
