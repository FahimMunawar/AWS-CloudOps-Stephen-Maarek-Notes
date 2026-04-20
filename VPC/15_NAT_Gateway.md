# 15 — NAT Gateway

## Overview

AWS-managed NAT service for outbound internet access from private subnets. Replaces NAT instances.

| Feature | Detail |
|---|---|
| **Managed by** | AWS (no administration required) |
| **Placement** | Created in a **public subnet**, specific AZ |
| **IP** | Elastic IP (automatically assigned) |
| **Bandwidth** | 5 Gbps → auto-scales to 100 Gbps |
| **Security Groups** | Not required — not applicable |
| **Availability** | Resilient **within one AZ** — multi-AZ requires multiple NAT Gateways |
| **Cost** | Per hour + data transfer |

---

## Traffic Flow

```
Private EC2 (private subnet)
  → NAT Gateway (public subnet)
    → Internet Gateway
      → Internet
```

> NAT Gateway **cannot work without an Internet Gateway**. The public subnet where the NAT Gateway lives must have a route to the IGW.

---

## Important Constraint

A NAT Gateway **cannot be used by EC2 instances in the same subnet** — it only serves instances in *other* (private) subnets.

---

## High Availability Architecture

NAT Gateway is resilient within its AZ — **not** cross-AZ. For full fault tolerance:

```
AZ A                          AZ B
├── Public Subnet A           ├── Public Subnet B
│     └── NAT GW A            │     └── NAT GW B
└── Private Subnet A          └── Private Subnet B
      └── route → NAT GW A          └── route → NAT GW B
```

- Each AZ's private subnet routes to its **own** NAT Gateway
- If AZ A goes down, AZ B still has internet access via NAT GW B
- No cross-AZ routing needed — if an AZ fails, its instances are unreachable anyway

---

## NAT Gateway vs NAT Instance (Exam Comparison)

| Factor | NAT Gateway | NAT Instance |
|---|---|---|
| Availability | HA within AZ (managed) | Manual failover needed |
| Bandwidth | Up to 100 Gbps | Depends on instance type |
| Maintenance | AWS-managed | User-managed (patching, SGs) |
| Security Groups | Not required | Must configure |
| Elastic IP | Auto-assigned | Must attach manually |
| Bastion Host use | **Cannot** be used as bastion | Can double as bastion |
| Cost | Per hour + data transfer | Per hour (EC2) + data transfer |
| Status | Recommended | End of support (Dec 2020) |

---

## SysOps Exam Q&A

**Q: What does a NAT Gateway require to provide internet access?**
A: It must be in a public subnet with an Internet Gateway — NAT Gateway alone is not sufficient.

**Q: A NAT Gateway in us-east-1a fails. What happens to instances in us-east-1b?**
A: If each AZ has its own NAT Gateway, us-east-1b instances are unaffected. If they share one NAT Gateway, us-east-1b loses outbound internet.

**Q: Why don't NAT Gateways need security groups?**
A: AWS manages all NAT Gateway networking internally — there are no inbound rules to configure.

**Q: Can a NAT Gateway serve instances in its own subnet?**
A: No — it can only be used from other (private) subnets.

---

## Quick Reference

```
NAT Gateway:
  Public subnet + Elastic IP (auto) + IGW required
  No security groups needed
  Bandwidth: 5 Gbps → 100 Gbps auto-scale
  HA within AZ only → one NAT GW per AZ for fault tolerance

vs NAT Instance:
  Manual failover, manual SG config, bandwidth = instance size
  Can be bastion host; NAT Gateway cannot
  NAT Gateway preferred in all production scenarios
```
