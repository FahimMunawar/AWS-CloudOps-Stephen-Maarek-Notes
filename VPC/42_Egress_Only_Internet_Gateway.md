# 42 — Egress-Only Internet Gateway

## Overview

IPv6-only equivalent of a NAT Gateway — allows **outbound-only** IPv6 internet access from instances, while preventing the internet from initiating inbound IPv6 connections.

| Gateway | Protocol | Direction |
|---|---|---|
| Internet Gateway | IPv4 + IPv6 | Bidirectional |
| NAT Gateway | IPv4 only | Outbound only |
| **Egress-Only IGW** | **IPv6 only** | **Outbound only** |

> Must update route tables to use the egress-only internet gateway.

---

## Route Table Comparison

### Public Subnet (web server — bidirectional IPv4 + IPv6)

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `fd00::/56` (VPC IPv6) | local |
| `0.0.0.0/0` | Internet Gateway |
| `::/0` | Internet Gateway |

### Private Subnet (server — outbound only)

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `fd00::/56` (VPC IPv6) | local |
| `0.0.0.0/0` | **NAT Gateway** (IPv4 outbound) |
| `::/0` | **Egress-Only Internet Gateway** (IPv6 outbound) |

---

## Architecture

```
Private Subnet EC2 Instance
  │
  ├── IPv4 outbound → NAT Gateway → Internet Gateway → Internet
  │
  └── IPv6 outbound → Egress-Only Internet Gateway → Internet
                       (internet CANNOT initiate inbound IPv6 connection)
```

---

## SysOps Exam Q&A

**Q: An EC2 instance in a private subnet needs outbound-only IPv6 internet access. What do you use?**
A: Egress-Only Internet Gateway — add a route `::/0 → Egress-Only IGW` to the private subnet route table.

**Q: What is the IPv6 equivalent of a NAT Gateway?**
A: Egress-Only Internet Gateway.

**Q: What route table entry enables IPv6 outbound-only access from a private subnet?**
A: `::/0 → Egress-Only Internet Gateway`

---

## Quick Reference

```
Egress-Only Internet Gateway:
  IPv6 only (NAT Gateway = IPv4 only)
  Outbound only — internet cannot initiate inbound IPv6 connections
  Must add route: ::/0 → Egress-Only IGW in private subnet route table

Private subnet full outbound setup:
  IPv4: 0.0.0.0/0 → NAT Gateway
  IPv6: ::/0      → Egress-Only Internet Gateway
```
