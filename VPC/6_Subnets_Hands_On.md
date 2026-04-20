# 6 — Subnets (Hands-On)

## Four Subnets Created for DemoVPC

| Name | AZ | CIDR | Size | Usable IPs |
|---|---|---|---|---|
| `PublicSubnetA` | eu-central-1a | `10.0.0.0/24` | 256 | 251 |
| `PublicSubnetB` | eu-central-1b | `10.0.1.0/24` | 256 | 251 |
| `PrivateSubnetA` | eu-central-1a | `10.0.16.0/20` | 4,096 | 4,091 |
| `PrivateSubnetB` | eu-central-1b | `10.0.32.0/20` | 4,096 | 4,091 |

> **Usable = Total − 5** (5 IPs reserved per subnet by AWS)

---

## Design Decisions

| Decision | Reason |
|---|---|
| Public subnets → smaller (`/24`) | Front-facing resources (load balancers) don't need many IPs |
| Private subnets → larger (`/20`) | EC2 instances, databases — need more IPs |
| Two AZs (1a and 1b) | High availability — resources can span AZs |
| Non-overlapping CIDRs | Required — overlapping CIDRs would fail subnet creation |

---

## Note

At this stage, all four subnets look identical — nothing makes them public or private yet. Public vs private is determined by the **Internet Gateway** and **route table** configuration, covered in the next lecture.

---

## Quick Reference

```
Create subnet: VPC → Subnets → Create subnet
  Select VPC → add subnet name + AZ + CIDR (repeat for each)
  Multiple subnets can be created in one flow

Usable IPs = total − 5 (always 5 reserved)
Public/private distinction: NOT yet set — requires IGW + route table config

CIDRs used:
  Public A:  10.0.0.0/24
  Public B:  10.0.1.0/24
  Private A: 10.0.16.0/20
  Private B: 10.0.32.0/20
```
