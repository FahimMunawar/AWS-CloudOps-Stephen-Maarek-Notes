# 10 — Route 53: Routing Policies — Weighted

## How It Works

Assign a weight to each record — Route 53 returns each record proportionally based on relative weight.

```
Traffic % to a record = record weight / sum of all weights

Example:
  ap-southeast-1: weight 10  → 10/100 = 10% of traffic
  us-east-1:      weight 70  → 70/100 = 70% of traffic
  eu-central-1:   weight 20  → 20/100 = 20% of traffic
```

> Weights do **not** need to sum to 100 — they are relative, not absolute.

---

## Key Rules

| Rule | Detail |
|---|---|
| **Same name and type** | All records in a weighted set must share the same DNS name and record type |
| **Health checks** | Can be associated (unlike Simple) |
| **Weight = 0** | Stops sending traffic to that resource |
| **All weights = 0** | All records returned with equal weight |
| **Record ID** | Each record in the set needs a unique identifier |

---

## Use Cases

- Cross-region load balancing
- Canary / blue-green deployments (send small % to new version)
- Gradual traffic shifting between resources

---

## Structure (vs Simple)

| Simple | Weighted |
|---|---|
| One record, multiple values | Multiple records, one value each |
| Client picks randomly | Route 53 distributes by weight |
| No health checks | Health checks supported |

---

## Quick Reference

```
Weighted routing: proportional traffic distribution
  Traffic % = record weight / sum of all weights
  Weights don't need to sum to 100

Rules:
  Same DNS name + type across all records in the set
  Each record has one value + one weight + unique Record ID
  Weight = 0 → stop sending traffic to that record
  All = 0   → equal distribution

Supports health checks
Use cases: multi-region LB, canary deploys, traffic shifting
```
