# 13 — Route 53: Health Checks (Hands-On)

## Creating an Endpoint Health Check

**Route 53 → Health checks → Create health check**

| Setting | Detail |
|---|---|
| **Type** | Endpoint |
| **Specify by** | IP address or domain name |
| **Port** | 80 (HTTP) |
| **Path** | `/` (root) or `/health` for real applications |
| **Interval** | Standard (30s) or Fast (10s — higher cost) |
| **Failure threshold** | How many consecutive failures before unhealthy |
| **String matching** | Check first 5,120 bytes of response for specific text |
| **Latency graph** | Optional — track latency over time |
| **Invert status** | Optional — flip healthy/unhealthy |
| **Health checker regions** | Recommended (default) or customize |
| **Alarm on failure** | Optional — create CloudWatch alarm |

---

## Demo: Triggering an Unhealthy Status

1. Remove HTTP (port 80) inbound rule from EC2 security group
2. Health checkers cannot connect → **connection timeout**
3. Health check status changes to **Unhealthy**
4. View last failed check → shows connection timeout error with reason

> The security group acts as a firewall — blocked port = failed health check.

---

## Calculated Health Check

**Type**: Monitors the status of other health checks (children)

- Select which health checks to include (up to 256)
- Define the threshold: healthy when **N of M** children are healthy
- Effectively AND / OR logic across child health checks

```
Example: all 3 children must be healthy → parent = healthy
         any 1 unhealthy → parent = unhealthy
```

---

## CloudWatch Alarm Health Check

- Select region + existing CloudWatch alarm
- Health check mirrors the alarm state
- Used for private resources (EC2 in private subnet, on-premises)
- Requires a CloudWatch alarm to already exist

---

## Quick Reference

```
Endpoint HC:
  IP or hostname, port, path (e.g., /health for real apps)
  Interval: 30s (standard) or 10s (fast/costly)
  Failure: blocked SG port → connection timeout → unhealthy

Calculated HC:
  Combines child health checks
  Set N-of-M threshold (AND/OR logic)

CloudWatch Alarm HC:
  Mirrors alarm state → used for private/on-premises resources

View failed check: Health checks → view last failed check → error details
```
