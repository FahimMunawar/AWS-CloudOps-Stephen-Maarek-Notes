# 4 — VPC Creation (Hands-On)

## Steps

**VPC → Your VPCs → Create VPC** (do NOT use the wizard — build step by step)

| Field | Value | Notes |
|---|---|---|
| **Name tag** | `DemoVPC` | — |
| **IPv4 CIDR** | `10.0.0.0/16` | Max allowed size is /16 |
| **IPv6 CIDR** | None | Can add later |
| **Tenancy** | Default | Dedicated = dedicated hardware, very expensive |

---

## What Gets Auto-Created

When a VPC is created, AWS automatically creates:
- A **main route table**
- A **main network ACL**

---

## Adding More CIDRs After Creation

**Actions → Edit CIDRs → Add new IPv4 CIDR**

Example: add `10.1.0.0/16` as a second CIDR block.

> Up to **5 IPv4 CIDRs** per VPC. Each must not overlap with others.

---

## Tenancy Options

| Option | Detail |
|---|---|
| **Default** | EC2 instances share underlying hardware (standard cost) |
| **Dedicated** | EC2 instances run on dedicated physical hardware (very expensive) |

---

## Quick Reference

```
Create VPC: name + IPv4 CIDR (/16 max) + tenancy (Default)
  Auto-created: main route table + main network ACL

Post-creation: add up to 5 IPv4 CIDRs via Actions → Edit CIDRs
Tenancy: Default (shared hardware) vs Dedicated (expensive, isolated hardware)
```
