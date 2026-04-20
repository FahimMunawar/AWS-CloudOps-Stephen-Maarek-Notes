# 18 — Route 53: Routing Policies — IP-Based

## How It Works

Route traffic based on the **client's IP address** — define CIDR blocks and map them to specific endpoints.

```
User A: IP in CIDR 203.x.x.x/24 → Location 1 → EC2 at 1.2.3.4
User B: IP in CIDR 200.x.x.x/24 → Location 2 → EC2 at 5.6.7.8
```

---

## Setup

1. Define **locations** — each location is a list of CIDR blocks
2. Map each location to a DNS record value (endpoint IP)
3. When a client's IP matches a CIDR → Route 53 returns the corresponding endpoint

---

## Use Cases

| Use Case | Detail |
|---|---|
| **Performance optimization** | Route clients to the closest endpoint based on known IP ranges |
| **Network cost reduction** | Route ISP traffic to the endpoint that avoids expensive cross-region data transfer |
| **ISP-based routing** | If a specific ISP uses a known CIDR, route them to a specific server |

---

## Quick Reference

```
IP-based routing: route by client IP address (CIDR blocks)
  Define: CIDR block → location → endpoint
  Use cases: performance, network cost, ISP-specific routing

vs Geolocation: IP-based uses raw IP ranges; Geolocation uses geographic location detection
```
