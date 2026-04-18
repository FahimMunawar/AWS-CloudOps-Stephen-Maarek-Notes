# 25 — Service Quotas: Hands-On

## Console Navigation

Service Quotas → **AWS Services** → select service (e.g., Lambda) → view all quotas

---

## Quota Details

- **Grayed out** = not adjustable (hard limit, e.g., async payload 256 KB)
- **Clickable** = adjustable (e.g., Concurrent Executions — default 1000)

Clicking a quota shows:
- Current usage vs applied limit
- Graph of usage over time
- Options to request an increase or create a CloudWatch alarm

---

## Requesting a Quota Increase

1. Click quota → **Request increase at account level**
2. Enter desired value
3. Small increases → **auto-approved**; large increases → **submitted to AWS Support** (takes time)

---

## Creating a CloudWatch Alarm on a Quota

1. Click quota → **CloudWatch alarms** → **Create alarm**
2. Set threshold — e.g., **80% of applied quota value**
3. Name the alarm (e.g., `alarm-quota-lambda-concurrent-execution`)
4. Alarm is created — click the green button to configure actions
5. In CloudWatch Alarms UI → **Edit** → Step 2 → add SNS notification
6. Select SNS topic (e.g., `demo-topic`) → **Save**

### Result
Alarm triggers at 80% of the quota → SNS → email notification → time to request an increase before throttling.

---

## Quick Reference

```
Navigate: Service Quotas → AWS Services → select service
Grayed = hard limit (not adjustable)
Clickable = adjustable → request increase at account level

Increase approval:
  Small = auto-approved
  Large = AWS Support review (takes time)

CloudWatch alarm on quota:
  Threshold: % of applied quota (e.g., 80%)
  Action: SNS → email
  Use: get alerted before reaching the limit
```
