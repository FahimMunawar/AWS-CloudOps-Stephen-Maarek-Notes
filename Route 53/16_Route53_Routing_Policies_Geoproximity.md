# 16 — Route 53: Routing Policies — Geoproximity

## How It Works

Routes traffic based on the geographic location of **users AND resources**, with the ability to shift traffic using a **bias** value.

---

## Bias

| Bias Value | Effect |
|---|---|
| **Positive (e.g., +50)** | Expand the resource's coverage area → attract more traffic |
| **Zero** | Default coverage — traffic split by proximity |
| **Negative** | Shrink coverage area → attract less traffic |

---

## Visual Example

```
Bias = 0 in both regions:
  us-west-1  |  us-east-1
  ← users → dividing line ← users →
  (equal split based on proximity)

Bias = 0 (west), +50 (east):
  us-west-1 | us-east-1
  ← smaller ← dividing line shifts left → larger coverage →
  (east attracts more users due to higher bias)
```

---

## Resource Types

| Resource Type | How to Specify Location |
|---|---|
| **AWS resources** | Specify the AWS region (automatically located) |
| **Non-AWS resources** | Specify latitude and longitude |

---

## Requirement

Must use **Route 53 Traffic Flow** (advanced feature) to leverage the bias.

---

## Geoproximity vs Geolocation

| Feature | Geoproximity | Geolocation |
|---|---|---|
| **Based on** | User + resource proximity + bias | User's actual location only |
| **Traffic shifting** | Yes — via bias | No |
| **Use case** | Shift traffic between regions | Serve different content by location |

---

## Quick Reference

```
Geoproximity: route by proximity + shift traffic with bias
  Positive bias → expand coverage → more traffic
  Negative bias → shrink coverage → less traffic
  Bias = 0      → pure proximity (like latency routing)

Resources:
  AWS: specify region
  Non-AWS: specify lat/lon

Requirement: Route 53 Traffic Flow must be enabled

Exam key: use Geoproximity to shift traffic from one region to another by increasing bias
```
