# 11. AWS Global Accelerator

## Overview

**AWS Global Accelerator** solves the latency problem of global users accessing an application deployed in a single region. Instead of routing over the unpredictable public internet, it uses the **AWS private global network** from the nearest edge location to your application — reducing hops, latency, and packet loss.

---

## The Problem: Public Internet Routing

```
User (America) ──5 router hops──► ALB (India)   ← slow, unreliable
User (Europe)  ──4 router hops──► ALB (India)   ← slow, unreliable
User (Australia)──6 router hops──► ALB (India)  ← slow, unreliable
```

Each hop adds latency and risk of dropped connections.

---

## The Solution: Global Accelerator

```
User (America)   ──► Edge Location (US)   ──private AWS network──► ALB (India)
User (Europe)    ──► Edge Location (EU)   ──private AWS network──► ALB (India)
User (Australia) ──► Edge Location (AUS)  ──private AWS network──► ALB (India)
```

- Users connect to the **nearest edge location** via Anycast IP
- Traffic travels the rest of the way over the **private, optimized AWS network**
- Fewer hops, lower latency, more stable

---

## Anycast IP — How It Works

| IP Type | Behavior |
|---------|----------|
| **Unicast** | One server = one IP; client must know which IP to use |
| **Anycast** | Multiple servers share the same IP; client is automatically routed to the nearest one |

Global Accelerator provides **two static Anycast IP addresses** that are global — clients always use the same IPs regardless of where they are.

---

## Key Features

| Feature | Detail |
|---------|--------|
| **Two static global IPs** | Anycast IPs that route to the nearest edge location |
| **Consistent performance** | Intelligent routing to lowest-latency edge; private AWS network |
| **Health checks** | Monitors endpoints; automatic failover in **< 1 minute** if one region fails |
| **No client cache issues** | IPs never change — no DNS TTL / cached IP problems |
| **DDoS protection** | AWS Shield integrated automatically |
| **Security** | Only 2 external IPs need to be whitelisted by clients |

---

## Supported Endpoints

- Elastic IPs
- EC2 instances
- Application Load Balancers (ALB)
- Network Load Balancers (NLB)
- Public **or** private (works with private VPC resources)

---

## Global Accelerator vs CloudFront

| Feature | CloudFront | Global Accelerator |
|---------|------------|-------------------|
| **Uses AWS edge locations** | Yes | Yes |
| **DDoS protection (Shield)** | Yes | Yes |
| **Caching** | **Yes** — content served from edge | **No** — all requests proxied to origin |
| **Protocol** | HTTP/HTTPS only | TCP and UDP |
| **Best for** | Static/dynamic web content, API acceleration | Gaming, IoT, VoIP, non-HTTP, any TCP/UDP app |
| **IP addresses** | CloudFront domain (DNS-based) | **2 static Anycast IPs** (IP-based) |
| **Failover** | DNS-based (subject to TTL) | **< 1 minute** (Anycast, no DNS caching) |
| **HTTP static IP requirement** | Not supported | Supported |

---

## When to Use Global Accelerator (Exam Patterns)

- Application requires **static IP addresses** accessible globally
- **Non-HTTP workloads**: gaming, IoT, VoIP (TCP/UDP)
- Need **fast, deterministic regional failover** (< 1 minute, no DNS caching issues)
- Multi-region ALB/NLB with global users needing low latency

---

## Best Practices

✓ **Use Global Accelerator when static IPs are required** — CloudFront uses domain names, not static IPs  
✓ **Use for disaster recovery** — health checks + sub-1-minute failover across regions  
✓ **Use CloudFront for cacheable content** — Global Accelerator does not cache  
✓ **Whitelist only the two Anycast IPs** in client-side firewalls — they never change  

---

## SysOps Exam Focus

**Q1: "Global users are experiencing high latency accessing your ALB deployed in eu-west-1. What service can reduce latency for users worldwide without moving the application?"**
- A) CloudFront
- B) AWS Global Accelerator
- C) S3 Transfer Acceleration
- D) Route 53 latency routing
- **Answer: B** — Global Accelerator routes users to the nearest edge location then uses the private AWS network to reach the ALB, bypassing the slow public internet

**Q2: "What is the key difference between Unicast and Anycast IP?"**
- A) Anycast assigns a unique IP to each server; Unicast shares one IP across servers
- B) Unicast assigns one IP to one server; Anycast allows multiple servers to share the same IP, routing clients to the nearest one
- C) Anycast is used for HTTP only; Unicast works for all protocols
- D) There is no functional difference between the two
- **Answer: B** — Anycast routes clients to the nearest server sharing that IP; Unicast routes to a specific server

**Q3: "Which scenario is best served by AWS Global Accelerator rather than CloudFront?"**
- A) Caching images and static files at edge locations
- B) A multiplayer gaming application requiring low-latency UDP connections worldwide
- C) Serving a static website from an S3 bucket globally
- D) Accelerating API responses by caching at edge locations
- **Answer: B** — Global Accelerator supports UDP and non-HTTP workloads; CloudFront is HTTP/HTTPS and caching only

**Q4: "What advantage does Global Accelerator have over Route 53 latency-based routing for failover?"**
- A) Global Accelerator supports more regions
- B) Global Accelerator uses Anycast IPs — failover is nearly instant (< 1 minute) with no DNS caching issues; Route 53 failover depends on DNS TTL
- C) Route 53 does not support health checks
- D) Global Accelerator is cheaper for inter-region traffic
- **Answer: B** — Anycast-based routing is immediate; DNS-based routing (Route 53) is delayed by client DNS cache TTL

**Q5: "A client firewall needs to whitelist IPs to access your application globally. You use Global Accelerator. How many IPs need to be whitelisted?"**
- A) One per AWS region the application runs in
- B) One per edge location globally
- C) Two — Global Accelerator provides two static Anycast IPs
- D) None — Global Accelerator uses domain-based routing
- **Answer: C** — Global Accelerator provides exactly two static global Anycast IPs that never change

---

## Quick Reference

```
Global Accelerator:
  2 static Anycast IPs → nearest edge location → private AWS network → your app
  No caching — all requests reach the origin
  Supports TCP and UDP
  Failover < 1 minute via health checks

vs CloudFront:
  CloudFront     = CDN, caches at edge, HTTP only, DNS-based
  Global Accel.  = proxy, no cache, TCP/UDP, static IPs, fast failover

Use Global Accelerator when:
  - Static global IPs required
  - Non-HTTP (gaming, IoT, VoIP)
  - Fast regional failover needed
  - Private ALB/NLB endpoints
```

---

**File: 11_AWS_Global_Accelerator.md**
**Status: SysOps-focused, exam-ready, concise format**
