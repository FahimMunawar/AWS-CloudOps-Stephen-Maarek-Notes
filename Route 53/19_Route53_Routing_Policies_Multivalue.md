# 19 — Route 53: Routing Policies — Multi-Value Answer

## How It Works

Returns **up to 8 healthy records** per query — client picks one (client-side load balancing).

```
Client DNS query → Route 53 returns up to 8 healthy IPs → client chooses one
```

---

## Key Rules

| Rule | Detail |
|---|---|
| **Max records returned** | Up to **8 healthy records** per query |
| **Health checks** | Supported and recommended — only healthy records are returned |
| **Same DNS name** | All records in the set share the same name |
| **Client-side LB** | Client randomly selects from returned IPs |
| **Not a substitute for ELB** | Multi-Value is DNS-level; ELB is a real load balancer |

---

## Multi-Value vs Simple (multiple values)

| Feature | Simple (multiple values) | Multi-Value |
|---|---|---|
| **Health checks** | NOT supported | Supported |
| **Unhealthy resource** | Can still be returned | Excluded from response |
| **Max values** | No set limit | Up to 8 healthy |

> Multi-Value is safer because it filters out unhealthy records before returning results.

---

## Demo Behavior

```
3 records (us-east-1, ap-southeast-1, eu-central-1) — all healthy
  → dig returns all 3 IPs

Invert eu-central-1 health check → becomes Unhealthy
  → dig returns only 2 IPs (eu-central-1 excluded)
```

---

## Quick Reference

```
Multi-Value: return up to 8 healthy records per query
  Health checks: supported (unhealthy records excluded)
  Client: picks one from returned list (client-side LB)
  NOT a replacement for ELB

vs Simple (multiple values):
  Simple: no health checks → unhealthy IPs can be returned
  Multi-Value: health-checked → only healthy IPs returned

Each record: one IP + health check + unique Record ID + same DNS name
```
