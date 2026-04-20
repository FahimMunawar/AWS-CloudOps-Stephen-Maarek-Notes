# 7 — VPC and Subnet CIDR Modification Limits

## VPC CIDR

| Action | Allowed |
|---|---|
| Modify existing IPv4 CIDR | **No** |
| Remove + add new IPv4 CIDR | Yes |
| Add a secondary IPv4 CIDR | Yes (up to 5 total) |
| Create new VPC with different CIDR | Yes |

**When to use**: VPC CIDR conflicts with another VPC or on-premises CIDR block.

---

## Subnet CIDR

| Action | Allowed |
|---|---|
| Modify existing IPv4 CIDR | **No** |
| Create new subnet with larger CIDR | Yes |

**When to use**: subnet runs out of available IP addresses.

---

## Quick Reference

```
VPC CIDR: cannot modify → remove + add new, or add secondary (max 5), or create new VPC
Subnet CIDR: cannot modify → create new subnet with larger CIDR block

Common exam scenario:
  CIDR conflict with on-premises/other VPC → create new VPC with non-overlapping CIDR
  Subnet out of IPs → create new larger subnet
```
