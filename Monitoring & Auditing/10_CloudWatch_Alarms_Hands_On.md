# 10 — CloudWatch Alarms: Hands-On

## Demo: Terminate EC2 on High CPU

### Goal
Create an alarm that **terminates an EC2 instance** if CPU utilization stays above 95% for 15 minutes.

---

## Steps

### 1. Create the Alarm

1. CloudWatch → **Alarms** → **Create alarm**
2. **Select metric** → EC2 → Per-Instance Metrics → paste instance ID → find `CPUUtilization`
3. Configure metric:
   - **Statistic:** Average
   - **Period:** 5 minutes (matches default EC2 metric frequency)
4. Set condition:
   - **Threshold type:** Static
   - **Condition:** Greater than `95`
   - **Datapoints to alarm:** `3 out of 3` → requires 15 consecutive minutes above threshold

### 2. Set Action

- **Action type:** EC2 action
- **When in ALARM:** Terminate the instance

### 3. Name and create

- Name: `Terminate EC2 on high CPU`
- Review and confirm

---

## New Alarm State

After creation, the alarm starts in **INSUFFICIENT_DATA** — it needs at least 15 minutes of metric data before it can transition to OK or ALARM.

---

## Testing with set-alarm-state

Instead of waiting for a real CPU spike, force the alarm state via CLI:

```bash
aws cloudwatch set-alarm-state \
  --alarm-name "Terminate EC2 on high CPU" \
  --state-value ALARM \
  --state-reason "testing"
```

### Result

1. Alarm transitions: **INSUFFICIENT_DATA → ALARM**
2. EC2 action fires: **Terminate instance**
3. Alarm history shows: `Updated from OK to In alarm → Action successfully executed`
4. EC2 console: instance enters **shutting-down → terminated** state

---

## Key Takeaways

- **3 out of 3 datapoints** at 5-minute period = alarm requires **15 consecutive minutes** in breach
- EC2 actions (stop/terminate/reboot/recover) are set directly on the alarm — no SNS needed
- `set-alarm-state` is the fastest way to test alarm actions without generating real load
- Alarm history records every state transition and action outcome

---

## Quick Reference

```
Alarm on CPUUtilization > 95% for 3/3 × 5min = 15 minutes sustained breach
Action: EC2 → Terminate instance
Test: aws cloudwatch set-alarm-state --state-value ALARM --state-reason "testing"
New alarm state: INSUFFICIENT_DATA (until enough metric data collected)
History: shows state transitions + action execution results
```
