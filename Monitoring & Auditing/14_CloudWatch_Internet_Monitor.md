# 14 — CloudWatch Internet Monitor

## Overview

CloudWatch Internet Monitor uses AWS's global network footprint to monitor internet health and detect how internet issues impact your AWS-hosted applications and end users.

---

## What It Monitors

- City-level outages
- ASN (Autonomous System Number) issues
- Cloud location impacts
- Global traffic patterns and health events

---

## Data Sources & Outputs

| Output | Destination |
|---|---|
| Metrics and logs | CloudWatch Logs and Metrics |
| Global health events | Amazon EventBridge (for alerting) |
| Recommendations | Latency improvement suggestions in the dashboard |

---

## Quick Reference

```
Internet Monitor = dashboard + alerting for internet health affecting AWS apps
Data source: AWS global network footprint
Monitors: city outages, ASNs, cloud locations, traffic patterns
Outputs: CloudWatch Logs/Metrics + EventBridge health events
Use: detect internet issues before they impact users; get latency recommendations
```
