# 27 — CloudTrail: Hands-On

## Console Navigation

CloudTrail → **Event history** (left sidebar)

- Shows last **90 days** of Management Events
- All API calls across the account are listed here

---

## Demo: Trace a Terminated EC2 Instance

1. Terminate an EC2 instance (EC2 console → right-click → Terminate)
2. Wait ~5 minutes for the event to appear in CloudTrail
3. CloudTrail → Event history → find `TerminateInstances`

### Event details visible:
- **Event name**: `TerminateInstances`
- **Event source**: `ec2.amazonaws.com`
- **Access key used**
- **Region**
- **Full raw event JSON** (user identity, instance ID, time, source IP, etc.)

---

## Quick Reference

```
Event history = last 90 days of Management Events, viewable in console
~5 min delay before events appear
Each event shows: API call name, source, IAM identity, access key, region, full JSON

Use: "Who terminated this instance?" → CloudTrail event history → TerminateInstances
```
