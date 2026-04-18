# 3 — CloudWatch Anomaly Detection

## Overview

CloudWatch Anomaly Detection uses machine learning to model the expected behaviour of a metric over time, then triggers alarms when the actual value deviates from that expected range — instead of using a fixed static threshold.

---

## How It Works

```
CloudWatch learns metric's normal pattern over time
       │
       ▼
Creates a "expected value band" (upper + lower bounds)
       │
       ▼
Metric goes outside the band  →  Anomaly detected  →  CloudWatch Alarm triggered
```

---

## Key Features

| Feature | Detail |
|---|---|
| **No static threshold needed** | Alarm fires based on deviation from expected behaviour |
| **ML-based model** | Trained on historical metric data automatically |
| **Exclude time periods** | Exclude specific events or time windows from training (e.g., known maintenance windows, bad data periods) |
| **Dynamic band** | Expected range adjusts to time-of-day, day-of-week patterns |

---

## When to Use

- Metrics with **cyclical or seasonal patterns** (e.g., higher traffic during business hours)
- When a **static threshold** would be too noisy or too rigid
- When you want alarms to adapt automatically as baseline behaviour changes

---

## SysOps Exam Q&A

**Q: What is CloudWatch Anomaly Detection?**
A: A feature that uses ML to model the expected value range of a metric and triggers a CloudWatch alarm when the actual value falls outside that range — replacing the need for a static threshold.

**Q: How is a CloudWatch Anomaly Detection alarm different from a standard CloudWatch alarm?**
A: A standard alarm uses a **fixed static threshold** (e.g., CPU > 80%). An anomaly detection alarm uses a **dynamic ML-based expected range** — it fires when the metric deviates from its normal pattern.

**Q: Can you exclude specific time periods from Anomaly Detection training?**
A: **Yes** — you can exclude known bad data periods or maintenance windows so they don't skew the model's baseline.

---

## Quick Reference

```
Anomaly Detection = ML-based expected value band for a metric
vs static threshold = fires when metric deviates from normal pattern, not a fixed number
Exclude periods    = remove bad data / maintenance windows from training
Use case           = metrics with seasonal/cyclical patterns where static thresholds are rigid
```
