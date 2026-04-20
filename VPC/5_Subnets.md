# 5 — Subnets

## Overview

A subnet is a sub-range of IPv4 addresses within a VPC. Each subnet lives in one AZ.

---

## 5 Reserved IP Addresses per Subnet

AWS reserves the first 4 and last 1 IP in every subnet — they cannot be assigned to EC2 instances.

Example: `10.0.0.0/24`

| IP | Reserved For |
|---|---|
| `10.0.0.0` | Network address |
| `10.0.0.1` | AWS VPC router |
| `10.0.0.2` | AWS-provided DNS |
| `10.0.0.3` | Reserved for future use |
| `10.0.0.255` | Network broadcast (not supported in VPC — reserved) |

---

## Exam Tip: Subnet Sizing

**Available IPs = Total IPs − 5**

```
Need 29 usable IPs:
  /27 = 32 IPs − 5 = 27 available → NOT enough
  /26 = 64 IPs − 5 = 59 available → sufficient ✓

Rule: always size up one notch to account for the 5 reserved IPs
```

---

## Public vs Private Subnets

What makes a subnet public or private is covered in the next lecture (Internet Gateway + route table settings).

---

## Quick Reference

```
Subnet = sub-range of VPC CIDR, within one AZ
5 IPs reserved per subnet: .0 (network), .1 (router), .2 (DNS), .3 (future), .255 (broadcast)

Sizing formula: usable IPs = total − 5
  Need N IPs → pick subnet where (2^(32-mask) − 5) ≥ N
  Example: need 29 → use /26 (64 − 5 = 59)
```
