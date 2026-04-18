# 34 — CloudWatch vs CloudTrail vs Config

## One-Line Distinction

| Service | Purpose |
|---|---|
| **CloudWatch** | Performance metrics, dashboards, alerts, log aggregation |
| **CloudTrail** | Record API calls made within your account (who did what) |
| **Config** | Record configuration changes + evaluate compliance against rules |

---

## ELB Example: All Three Services Applied

| Service | What It Tells You for an ELB |
|---|---|
| **CloudWatch** | Incoming connection count, error code % over time, performance dashboard (global if needed) |
| **Config** | Security group rule changes, SSL certificate modifications, compliance rules (e.g., SSL cert must be assigned, no unencrypted traffic allowed) |
| **CloudTrail** | Who made API calls — who changed the SG, who removed or swapped the SSL cert |

---

## Summary

```
CloudWatch  → WHAT is happening (metrics, logs, performance)
CloudTrail  → WHO did it (API call audit — identity, time, source IP)
Config      → HOW it changed (configuration diff + compliance evaluation over time)

All three are complementary — not overlapping.
```

---

## SysOps Exam Q&A

**Q: You need to know who changed the SSL certificate on a load balancer. Which service?**
A: **CloudTrail** — records the API call with the IAM identity that made the change.

**Q: You want to enforce a rule that all load balancers must have an SSL certificate. Which service?**
A: **AWS Config** — define a compliance rule; non-compliant resources are flagged (and can be auto-remediated).

**Q: You want to monitor the number of 5XX errors on your load balancer over time. Which service?**
A: **CloudWatch** — metrics and dashboards for performance monitoring.

**Q: You want to see a timeline of configuration changes to a load balancer. Which service?**
A: **AWS Config** — resource timeline shows configuration diffs + linked CloudTrail events.
