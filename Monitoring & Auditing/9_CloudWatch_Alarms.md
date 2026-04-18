# 9 — CloudWatch Alarms

## Overview

CloudWatch Alarms trigger notifications or actions when a metric crosses a defined threshold. You can apply sampling, percentage, max, min, and other statistical functions to the metric evaluation.

---

## Alarm States

| State | Meaning |
|---|---|
| **OK** | Metric is within threshold — alarm not triggered |
| **ALARM** | Threshold breached — action triggered |
| **INSUFFICIENT_DATA** | Not enough data to determine alarm state |

---

## Period

How long the alarm evaluates the metric before deciding state.

- Supports **high-resolution custom metrics**: 10s, 30s, or multiples of 60s
- Longer periods smooth out spikes; shorter periods react faster

---

## Alarm Targets (Actions)

| Target | Actions Available |
|---|---|
| **EC2 Instance** | Stop, Terminate, Reboot, Recover |
| **Auto Scaling** | Scale out, Scale in |
| **SNS** | Send notification → hook to Lambda for custom actions |

---

## Composite Alarms

Standard alarms monitor a **single metric**. Composite Alarms combine multiple alarms using **AND / OR** logic.

```
Alarm A (CPU utilization > 80%)    ─┐
                                    ├─ AND → Composite Alarm → SNS
Alarm B (IOPS > threshold)         ─┘
```

### Benefits
- Reduces alarm noise — only alert when a meaningful combination of conditions is true
- Example: alert only when CPU is HIGH **and** network is LOW (not when both are high, which might be expected load)

---

## EC2 Instance Recovery

### Status Checks Available

| Check | What It Monitors |
|---|---|
| **Instance status check** | EC2 virtual machine health |
| **System status check** | Underlying hardware host health |
| **EBS status check** | Attached EBS volume health |

### Recovery Behavior

When a CloudWatch Alarm on a status check triggers EC2 recovery:
- Instance is migrated to a new host
- **Preserved:** private IP, public IP, Elastic IP, metadata, placement group
- **SNS alert** can be sent to notify of the recovery event

---

## Alarm on CloudWatch Logs Metric Filter

```
CloudWatch Logs
  └── Metric Filter (e.g., count of "ERROR" in logs)
        └── CloudWatch Alarm (e.g., count > 5 in 5 minutes)
              └── SNS → notification / Lambda
```

This allows alerting directly on log content patterns without a separate monitoring tool.

---

## Testing Alarms — set-alarm-state

To test that an alarm triggers the correct action without waiting for the threshold to be breached:

```bash
aws cloudwatch set-alarm-state \
  --alarm-name "MyAlarm" \
  --state-value ALARM \
  --state-reason "Testing alarm action"
```

> **Exam tip:** Use `set-alarm-state` via CLI to manually trigger an alarm for testing purposes.

---

## SysOps Exam Q&A

**Q: What are the three states of a CloudWatch Alarm?**
A: **OK** (within threshold), **ALARM** (threshold breached), **INSUFFICIENT_DATA** (not enough data to evaluate).

**Q: What are the three main targets a CloudWatch Alarm can act on?**
A: 1) **EC2 actions** (stop/terminate/reboot/recover), 2) **Auto Scaling** (scale out/in), 3) **SNS** (notifications, which can chain to Lambda).

**Q: What is a Composite Alarm and why would you use one?**
A: A Composite Alarm monitors multiple other alarms using AND/OR logic. Use it to reduce alarm noise — e.g., only alert when CPU is high AND network is low simultaneously.

**Q: When EC2 instance recovery is triggered by a CloudWatch Alarm, what is preserved?**
A: **Private IP, public IP, Elastic IP, instance metadata, and placement group** — the instance moves to new hardware but keeps its identity.

**Q: How do you test a CloudWatch Alarm without breaching its threshold?**
A: Use the CLI command `aws cloudwatch set-alarm-state` to manually force the alarm into the ALARM state.

---

## Quick Reference

```
Alarm states: OK | ALARM | INSUFFICIENT_DATA

Targets:
  EC2:          stop / terminate / reboot / recover
  Auto Scaling: scale out / scale in
  SNS:          notify → Lambda → custom actions

Composite Alarms:
  Combine multiple alarms with AND / OR
  Reduces noise — only alert on meaningful combinations

EC2 Recovery:
  Triggered by: instance / system / EBS status check alarm
  Preserved: private IP, public IP, EIP, metadata, placement group

Logs → Metric Filter → Alarm → SNS (alert on log content patterns)

Test alarms: aws cloudwatch set-alarm-state --state-value ALARM
```
