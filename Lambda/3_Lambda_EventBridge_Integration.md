# 36 — Lambda + EventBridge Integration

## Overview

Two primary patterns for triggering Lambda from EventBridge:

1. **Scheduled / CRON** — trigger Lambda on a time-based schedule (e.g., every 1 minute, every 1 hour)
2. **Event Pattern** — trigger Lambda when something happens in AWS (e.g., CodePipeline state change, EC2 termination)

---

## Two Trigger Patterns

### Pattern 1 — Scheduled (CRON / Rate)

```
EventBridge Rule (rate: 1 minute)
        │
        ▼
  Lambda Function (performs task)
```

- Use case: serverless CRON jobs, periodic data processing, health checks
- Rate expression example: `rate(1 minute)` or `rate(1 hour)`
- Cron expression example: `cron(0 10 * * ? *)` — every day at 10:00 AM UTC

### Pattern 2 — Event Pattern

```
AWS Service Event (e.g., CodePipeline state change)
        │
        ▼
  EventBridge Rule (matches event pattern)
        │
        ▼
  Lambda Function (reacts to event)
```

- Use case: automation triggered by infrastructure changes
- Examples: EC2 instance terminated, S3 bucket policy changed, CodePipeline failed

---

## EventBridge Rule vs EventBridge Scheduler

| | EventBridge Rule (Schedule) | EventBridge Scheduler |
|---|---|---|
| Location | EventBridge → Rules | EventBridge → Scheduler |
| Use case | Simple rate/cron rules | More advanced scheduling features |
| Resource policy | Auto-configured on Lambda | Auto-configured on Lambda |
| Exam focus | **Yes** | Less common |

> When creating a rule with "Schedule" type, AWS prompts you to use EventBridge Scheduler. Clicking **"Continue to create rule"** creates a classic EventBridge Rule instead — same capability, but relevant for understanding **resource-based policies**.

---

## How EventBridge Invokes Lambda: Resource-Based Policy

When you create an EventBridge rule targeting a Lambda function, AWS **automatically adds a resource-based policy** to the Lambda function.

### What the Policy Looks Like
```json
{
  "Effect": "Allow",
  "Principal": {
    "Service": "events.amazonaws.com"
  },
  "Action": "lambda:InvokeFunction",
  "Resource": "arn:aws:lambda:<region>:<account-id>:function:lambda-demo-eventbridge",
  "Condition": {
    "ArnLike": {
      "AWS:SourceArn": "arn:aws:events:<region>:<account-id>:rule/InvokeLambdaEveryMinute"
    }
  }
}
```

### Key Points
- **Principal**: `events.amazonaws.com` — EventBridge service is granted invoke permission
- **Condition**: `ArnLike` on `SourceArn` — only **this specific rule** can invoke the function (not any EventBridge rule)
- This is a **resource-based policy** on Lambda (not an IAM role policy)
- AWS configures this **automatically** when you set the target in the EventBridge console

---

## Event Payload Sent to Lambda

When EventBridge triggers Lambda, it passes a JSON event object. Example for a scheduled rule:

```json
{
  "version": "0",
  "id": "<event-id>",
  "detail-type": "Scheduled Event",
  "source": "aws.events",
  "account": "<account-id>",
  "time": "2024-01-15T10:00:00Z",
  "region": "eu-west-1",
  "resources": [
    "arn:aws:events:<region>:<account-id>:rule/InvokeLambdaEveryMinute"
  ],
  "detail": {}
}
```

- `source`: always `aws.events` for EventBridge scheduled rules
- `detail-type`: `"Scheduled Event"` for scheduled rules
- `detail`: empty `{}` for scheduled rules (no specific payload)
- `resources`: ARN of the EventBridge rule that triggered the function

---

## Hands-On Steps

1. Create Lambda function (e.g., `lambda-demo-eventbridge`, Python 3.9)
2. Go to **EventBridge → Rules → Create Rule**
3. Name: `InvokeLambdaEveryMinute`
4. Event bus: `default`
5. Rule type: **Schedule** → click "Continue to create rule"
6. Schedule pattern: `rate(1 minute)`
7. Target: **Lambda function** → select your function
8. AWS auto-configures the resource-based policy
9. Verify: Lambda console → **Triggers** tab shows EventBridge source
10. Monitor: Lambda → **Monitor** → **View CloudWatch Logs**

### Cleanup
- **Disable the EventBridge rule** when done to stop invocations (avoid unexpected costs)

---

## Monitoring Invocations

- Lambda → Monitor tab → Invocations metric (may take a few minutes to populate)
- Lambda → Monitor → **View CloudWatch Logs** → select latest log stream
- Print the event in code to inspect the full payload:

```python
def lambda_handler(event, context):
    print(event)  # Logs full EventBridge event JSON to CloudWatch
    return "OK"
```

---

## Best Practices

- Use **rate expressions** for simple intervals; use **cron expressions** for complex schedules
- Always check **resource-based policy** on Lambda to confirm EventBridge has invoke permission
- **Disable rules** (not delete) during testing pauses to preserve configuration
- Use **CloudWatch Logs** to debug what event payload Lambda actually received
- For event pattern rules, use the **Event Pattern preview** in console to validate before saving

---

## SysOps Exam Q&A

**Q: How does EventBridge get permission to invoke a Lambda function?**
A: AWS automatically adds a **resource-based policy** to the Lambda function granting `lambda:InvokeFunction` to the `events.amazonaws.com` principal, scoped to the specific rule's ARN via a condition.

**Q: What is the difference between EventBridge Rule (Schedule) and EventBridge Scheduler?**
A: Both can trigger targets on a schedule. EventBridge Scheduler has more advanced features, but EventBridge Rules are the classic approach and more commonly tested. They behave the same for basic rate/cron scheduling.

**Q: A Lambda function should run every day at 9:00 AM. What is the recommended setup?**
A: Create an **EventBridge Rule** with a **cron schedule expression** targeting the Lambda function.

**Q: What does the `detail` field contain in the EventBridge event payload for a scheduled rule?**
A: It is an **empty object `{}`** — scheduled rules do not pass additional detail data.

**Q: After a hands-on demo, Lambda keeps getting invoked and costs are accumulating. What should you do?**
A: **Disable the EventBridge rule** — this stops invocations without deleting the rule configuration.

---

## Quick Reference

```
Scheduled trigger  = EventBridge Rule with rate() or cron() → Lambda
Event trigger      = EventBridge Rule with event pattern → Lambda
Permission model   = resource-based policy on Lambda (auto-added by AWS)
Principal          = events.amazonaws.com
Condition          = ArnLike on specific rule ARN (scoped, not wildcard)
Event payload key  = source: "aws.events", detail-type: "Scheduled Event"
Cleanup            = disable rule (not delete) to stop invocations
```
