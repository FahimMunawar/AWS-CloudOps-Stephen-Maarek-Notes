# 2 — Amazon Route 53 Overview

## Overview

- **Highly available, scalable, fully managed, authoritative DNS**
- **Authoritative** = you control and can update DNS records
- Also a **domain registrar** (register domain names directly in Route 53)
- Supports **health checks** on resources
- Only AWS service with a **100% availability SLA**
- Named after DNS port **53**

---

## DNS Record Structure

Each record contains:

| Field | Description |
|---|---|
| **Domain / Subdomain** | e.g., `example.com` |
| **Record Type** | A, AAAA, CNAME, NS, etc. |
| **Value** | e.g., `12.34.56.78` |
| **Routing Policy** | How Route 53 responds to queries |
| **TTL** | Time-to-live — how long DNS resolvers cache the record |

---

## Record Types (Must Know)

| Type | Maps | Notes |
|---|---|---|
| **A** | Hostname → IPv4 | e.g., `example.com → 1.2.3.4` |
| **AAAA** | Hostname → IPv6 | Same concept as A, but IPv6 |
| **CNAME** | Hostname → Hostname | Target must be an A or AAAA record |
| **NS** | Name servers for the hosted zone | Controls how traffic is routed to a domain |

### CNAME Restriction

- Cannot create a CNAME for the **Zone Apex** (root domain, e.g., `example.com`)
- Can create a CNAME for subdomains (e.g., `www.example.com`)

---

## Hosted Zones

A **container for DNS records** — defines how to route traffic to a domain and its subdomains.

**Cost: $0.50/month per hosted zone**

### Two Types

| Type | Accessible From | Use Case |
|---|---|---|
| **Public Hosted Zone** | Anyone on the internet | Public-facing domains (e.g., `app.example.com → public IP`) |
| **Private Hosted Zone** | Within your VPC only | Internal domains (e.g., `api.example.internal → private IP`) |

### Private Hosted Zone Example

```
VPC resources query private hosted zone:

webapp.example.internal    → 10.0.0.5  (EC2 instance 1)
api.example.internal       → 10.0.0.10 (EC2 instance 2)
database.example.internal  → 10.0.0.20 (RDS)

EC2 instance 1 → asks "what is api.example.internal?"
  → Private hosted zone answers with 10.0.0.10
    → EC2 connects to EC2 instance 2 via private IP
```

---

## Pricing

| Item | Cost |
|---|---|
| Hosted zone | $0.50/month |
| Domain registration | $12+/year (minimum) |

> Route 53 is **not free** — creating records requires a hosted zone.

---

## SysOps Exam Q&A

**Q: What does "authoritative DNS" mean?**
A: You (the customer) can update the DNS records — you have full control over the DNS service.

**Q: Which AWS service has a 100% availability SLA?**
A: **Amazon Route 53**.

**Q: Can you create a CNAME record for example.com (the zone apex)?**
A: **No** — CNAME cannot be created for the zone apex / top node of a DNS namespace. Use an Alias record instead (covered in a later lecture).

**Q: What is the difference between a public and private hosted zone?**
A: **Public** = answers queries from anyone on the internet. **Private** = answers queries only from within your VPC, for private/internal domain names.

---

## Quick Reference

```
Route 53: authoritative DNS + domain registrar + health checks
  100% availability SLA (only AWS service with this guarantee)
  Port 53 = origin of the name

Record types (must know):
  A     → hostname → IPv4
  AAAA  → hostname → IPv6
  CNAME → hostname → hostname (NOT for zone apex)
  NS    → name servers for hosted zone

Hosted zones ($0.50/month):
  Public  → internet-accessible, public IPs
  Private → VPC-only, private IPs, internal domain names

Domain registration: $12+/year
CNAME zone apex restriction: use Alias record instead
```
