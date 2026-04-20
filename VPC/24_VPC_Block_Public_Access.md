# 24 — VPC Block Public Access (BPA)

## Overview

A regional, account-level control that centrally blocks internet access to/from VPCs and subnets — overrides all other network configurations (IGW, SG, NACL).

---

## BPA Modes

| Mode | Effect |
|---|---|
| **Bidirectional** | Blocks ALL internet traffic in and out — no internet access whatsoever |
| **Ingress-only** | Blocks inbound internet traffic only; outbound via NAT Gateway or Egress-Only IGW still allowed |

---

## Exclusions

Individual VPCs or subnets can be excluded from BPA with their own mode:

| Exclusion Mode | Effect |
|---|---|
| **Bidirectional** | Excluded VPC/subnet allows all internet traffic (in + out) |
| **Egress-only** | Excluded VPC/subnet allows outbound internet only |

---

## Key Properties

- Enabled **per AWS Region**
- Sits **above** all other controls — IGW, security groups, NACLs, route tables cannot override it
- Verify blocked traffic using **VPC Reachability Analyzer**

---

## Use Cases

- Enforce compliance: guarantee no workload in an account ever touches the internet
- Ingress-only mode: allow instances to pull updates/data outbound without exposing them to inbound internet access

---

## SysOps Exam Q&A

**Q: You need to guarantee that no resource in an AWS account can ever be accessed from the internet, regardless of security group or route table configuration. What do you use?**
A: VPC Block Public Access in bidirectional mode — it overrides all other network controls at the account/region level.

**Q: What is the difference between BPA ingress-only mode and bidirectional mode?**
A: Ingress-only blocks all inbound internet traffic but allows outbound via NAT Gateway or Egress-Only IGW. Bidirectional blocks all internet traffic in both directions.

---

## Quick Reference

```
VPC Block Public Access (BPA):
  Regional, account-level — overrides IGW, SG, NACL, route tables

  Bidirectional: no internet in or out
  Ingress-only: outbound allowed (NAT GW / Egress-Only IGW); inbound blocked

  Exclusions per VPC/subnet:
    Bidirectional exclusion: full internet access for that VPC/subnet
    Egress-only exclusion: outbound only for that VPC/subnet

  Verify with: VPC Reachability Analyzer
```
