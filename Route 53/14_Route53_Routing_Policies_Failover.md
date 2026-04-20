# 14 — Route 53: Routing Policies — Failover

## How It Works

```
Primary record (+ health check) → healthy → Route 53 returns primary IP
Primary record → unhealthy      → Route 53 automatically returns secondary IP
```

Clients always receive the healthy resource — failover is transparent.

---

## Rules

| Rule | Detail |
|---|---|
| **Primary** | Exactly one — **must** be associated with a health check |
| **Secondary** | Exactly one — health check is optional |
| **Failover direction** | Primary → Secondary only (one primary, one secondary) |
| **Same DNS name** | Both records share the same domain name and type |

---

## Setup

1. Create primary record:
   - Routing policy: Failover
   - Failover record type: **Primary**
   - **Associate a health check** (mandatory)
   - Record ID: unique name (e.g., `EU`)

2. Create secondary record:
   - Same DNS name, same type
   - Routing policy: Failover
   - Failover record type: **Secondary**
   - Health check: optional
   - Record ID: unique name (e.g., `US`)

---

## Failover Demo Flow

```
Primary (eu-central-1) → healthy → clients get eu-central-1 IP

Remove port 80 from SG → health checkers cannot connect
  → Health check = Unhealthy (% healthy drops to 0)
    → Route 53 returns secondary (us-east-1) IP instead

Restore port 80 in SG → health check passes again
  → Route 53 returns primary (eu-central-1) IP again
```

---

## Quick Reference

```
Failover routing: primary (with health check) → secondary on failure

Exactly 1 primary + 1 secondary
Primary: health check REQUIRED
Secondary: health check optional

Clients always get the healthy record — failover is automatic and transparent
Trigger failure: remove SG rule → health check connection timeout → unhealthy
```
