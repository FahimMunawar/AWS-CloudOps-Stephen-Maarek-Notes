# 22 — EventBridge + SSM Automation

## Overview

SSM Automations can be triggered as EventBridge targets — enabling automated infrastructure responses to events or schedules without manual intervention.

---

## Pattern

```
EC2 instance starts (user action)
  → EventBridge: EC2 Instance State-change Notification (state: running)
    → SSM Automation (e.g., bootstrap instance — install software)
```

---

## Trigger Types

| Trigger | Example |
|---|---|
| **Event-driven** | EC2 starts → run bootstrap automation |
| **Scheduled** | Every night → run patching automation |

---

## Quick Reference

```
SSM Automation = valid EventBridge target
Trigger: event pattern (e.g., EC2 state change) OR schedule (cron/rate)
Use case: automate infrastructure tasks (bootstrap, patch, remediate) in response to events
```
