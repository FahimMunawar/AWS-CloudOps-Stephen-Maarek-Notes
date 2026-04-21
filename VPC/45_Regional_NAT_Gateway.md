# 45 — Regional NAT Gateway (RNAT)

## Overview

A newer type of NAT Gateway created at the **VPC level** (not per-AZ subnet) — highly available and shared automatically across all AZs.

---

## Regional vs Traditional NAT Gateway

| Property | Traditional NAT Gateway | Regional NAT Gateway |
|---|---|---|
| Placement | Per public subnet, per AZ | VPC-level (one per VPC) |
| HA across AZs | Requires one NAT GW per AZ | Automatic — shared across all AZs |
| Public subnet required | Yes — must deploy in public subnet | **No** — no public subnets needed |
| New AZ support | Must manually create new NAT GW | Auto-scales to new AZs |

---

## Architecture

```
VPC
  │
Regional NAT Gateway (VPC-level)
  │
  └── route table: 0.0.0.0/0 → Internet Gateway
  
Private Subnet A (AZ-1) ──┐
Private Subnet B (AZ-2) ──┤── route table: 0.0.0.0/0 → Regional NAT GW
Private Subnet C (AZ-3) ──┘
```

No public subnets needed to host the gateway — eliminating the risk of accidentally deploying resources in public subnets.

---

## Benefits

- **Simplified architecture** — one NAT Gateway per VPC instead of one per AZ
- **No public subnets required** — reduces attack surface
- **Auto-scaling** — automatically serves new AZs as they are added

---

## SysOps Exam Q&A

**Q: What is the key architectural benefit of a Regional NAT Gateway over a traditional NAT Gateway?**
A: A single Regional NAT Gateway serves all AZs in a VPC automatically — no need to create one per AZ or maintain public subnets to host it.

---

## Quick Reference

```
Regional NAT Gateway:
  Created at VPC level — not per-AZ subnet
  Shared across all AZs automatically (scales with new AZs)
  No public subnets required to host it
  Route: private subnet → 0.0.0.0/0 → Regional NAT GW → IGW

vs Traditional: one per AZ in public subnet → more operational overhead
```
