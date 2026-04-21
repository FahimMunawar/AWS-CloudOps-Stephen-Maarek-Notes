# 21 — Secrets Manager Monitoring & Troubleshooting

## CloudTrail Integration

CloudTrail captures two categories of Secrets Manager activity:

| Category | Examples |
|---|---|
| **API calls** | `GetSecretValue`, `PutSecretValue`, `DeleteSecret` |
| **Non-API service events** | Rotation lifecycle, version deletion events |

Non-API service events are unique to Secrets Manager — they are not triggered by direct API calls but are automatically recorded by CloudTrail.

---

## Non-API Service Events

| Event | Meaning |
|---|---|
| `RotationStarted` | Rotation process has begun |
| `RotationSucceeded` | Rotation completed successfully |
| `RotationFailed` | Rotation failed — action required |
| `RotationAbandoned` | Manual change was made to a secret instead of automated rotation |
| `StartSecretVersionDelete` | Version deletion initiated |
| `CancelSecretVersionDelete` | Version deletion cancelled |
| `EndSecretVersionDelete` | Version deletion completed |

---

## Alerting on Rotation Failures

Build an alerting pipeline using CloudTrail → CloudWatch:

```
Secrets Manager rotation fails
  → RotationFailed event → CloudTrail
    → CloudTrail logs → CloudWatch Logs
      → CloudWatch Metric Filter (filter for "RotationFailed")
        → CloudWatch Alarm (threshold: ≥ 1 occurrence)
          → SNS → Email / SMS alert
```

---

## Troubleshooting Rotation Failures

When rotation fails, two places to investigate:

| Source | What it shows |
|---|---|
| **CloudTrail** | Which event failed and when |
| **Lambda function logs (CloudWatch Logs)** | Actual error message from code execution — **best for root cause** |

Rotation flow:
```
Secrets Manager → invokes Lambda function → Lambda updates target (e.g., RDS password)
  → if Lambda fails → RotationFailed event logged in CloudTrail
```

**Additional alert option:** Monitor Lambda function error rates directly in CloudWatch — a spike indicates rotation issues without waiting for CloudTrail event processing.

---

## SysOps Exam Q&A

**Q: How do you get alerted when a Secrets Manager rotation fails?**
A: CloudTrail captures the `RotationFailed` non-API event → forward to CloudWatch Logs → Metric Filter → CloudWatch Alarm → SNS notification. Or monitor Lambda function error rates.

**Q: Where is the best place to debug why a Secrets Manager rotation failed?**
A: Lambda function logs in CloudWatch Logs — they contain the actual code execution error, unlike CloudTrail which only records that the event occurred.

**Q: What does a `RotationAbandoned` event in CloudTrail indicate?**
A: A manual change was made directly to the secret instead of going through automated rotation.

---

## Quick Reference

```
Secrets Manager → CloudTrail logs:
  API calls (GetSecretValue, DeleteSecret, etc.)
  Non-API events (RotationStarted/Succeeded/Failed/Abandoned, version deletion)

Alert on rotation failure:
  CloudTrail → CloudWatch Logs → Metric Filter (RotationFailed) → Alarm → SNS

Troubleshoot rotation failure:
  CloudTrail → what failed and when
  Lambda logs (CloudWatch Logs) → why it failed (best source)
  Lambda error rate metric → proactive alerting
```
