# 24 — Service Quotas

## Overview

Service Quotas shows all AWS service limits in your account and how close you are to reaching them. You can create CloudWatch Alarms directly on top of quota metrics.

---

## Use Case

```
Service Quotas monitors: Lambda Concurrent Executions (default limit: 1000)
  → CloudWatch Alarm threshold: 900
    → Alarm triggers → notification
      → Admin requests quota increase before throttling occurs
```

---

## Service Quotas vs Trusted Advisor

| Feature | Service Quotas | Trusted Advisor |
|---|---|---|
| **Coverage** | All quotas across all services | ~50 service limit checks |
| **CloudWatch integration** | Yes — create alarms directly | Yes — results published to CloudWatch |
| **Recommended for quota monitoring** | **Yes** | Less complete — limited checks |

> **Exam tip:** Prefer **Service Quotas + CloudWatch Alarms** over Trusted Advisor for quota monitoring — more complete coverage.

---

## Quick Reference

```
Service Quotas = monitor all AWS service limits in your account
Create CloudWatch Alarm on a quota → alert before throttling occurs
Example: Lambda Concurrent Executions > 900 (limit 1000) → alarm → request increase

vs Trusted Advisor: only ~50 service limit checks — less complete
Recommendation: use Service Quotas for quota alerting
```
