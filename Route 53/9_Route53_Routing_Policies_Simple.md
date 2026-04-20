# 9 — Route 53: Routing Policies — Simple

## Routing Policies Overview

Route 53 routing policies control how Route 53 responds to DNS queries — **not how traffic flows**. The DNS only returns an answer; clients use that answer to make the actual connection.

Available routing policies:
- Simple, Weighted, Failover, Latency-based, Geolocation, Multi-value Answer, Geoproximity

---

## Simple Routing Policy

- Routes traffic to a **single resource** (typically)
- **Multiple values** can be returned in one record → client picks one **randomly** (client-side)
- **No health checks** — cannot be associated with health checks
- If using an **Alias** with Simple policy → only **one AWS resource** can be the target

---

## Behavior

```
Single value:
  Client → DNS query → Route 53 returns one IP → client connects

Multiple values (same record):
  Client → DNS query → Route 53 returns multiple IPs → client picks randomly
```

---

## Hands-On

1. Create record: `simple.yourdomain.com`, type A, routing policy = Simple
2. Add one or more IP values (one per line)
3. `dig simple.yourdomain.com` → shows all returned IPs
4. Browser randomly uses one of the returned IPs

---

## Quick Reference

```
Simple routing:
  Single or multiple IP values in one record
  Multiple values → client picks randomly (client-side load balancing)
  No health checks supported
  Alias + Simple → one AWS resource only

DNS routing ≠ traffic routing:
  DNS just answers "which IP?" — actual traffic goes client → server directly
```
