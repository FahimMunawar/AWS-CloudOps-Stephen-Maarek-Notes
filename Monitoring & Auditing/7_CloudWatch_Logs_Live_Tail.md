# 7 — CloudWatch Logs: Live Tail

## Overview

Live Tail is a real-time log streaming feature in CloudWatch Logs — useful for active debugging without waiting for log events to be indexed and queried.

---

## How It Works

1. Navigate to a log group → click **Start tailing** (or use the Live Tail UI)
2. Optionally filter by:
   - **Log group** (required)
   - **Log stream** (optional)
   - **Filter pattern** (optional keyword/expression)
3. Live Tail waits and displays log events as they arrive in real time

---

## Demo Flow

```
Create log group: demo-log-group
  └── Create log stream: DemoLogStream
        └── Open Live Tail → filter on log group + stream
              └── Post a log event ("hello world") via Actions → Create log event
                    └── Event appears immediately in the Live Tail UI
```

From each event in Live Tail you can see:
- Timestamp of when it occurred
- Log group name
- Clickable link to the exact log stream the event came from

---

## Pricing

| Tier | Allowance |
|---|---|
| **Free** | ~1 hour per day |
| **Paid** | Charged beyond free tier |

> **Important:** Always cancel/close your Live Tail session when done — leaving it open consumes your free quota and may incur charges.

---

## SysOps Exam Q&A

**Q: What is CloudWatch Logs Live Tail used for?**
A: Real-time streaming of log events as they arrive — primarily for active debugging. Events appear immediately without needing a Logs Insights query.

**Q: How is Live Tail different from CloudWatch Logs Insights?**
A: Logs Insights queries **historical** data. Live Tail shows **real-time** incoming log events as they are posted.

---

## Quick Reference

```
Live Tail = real-time log event streaming for active debugging
Filter by: log group, log stream (optional), filter pattern (optional)
Free tier: ~1 hour/day — close session when done to avoid charges
vs Logs Insights: Insights = historical query; Live Tail = real-time stream
```
