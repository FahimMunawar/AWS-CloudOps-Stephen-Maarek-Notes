# 3 — VPC Overview and Creation

## Key Facts

| Fact | Detail |
|---|---|
| **VPCs per region** | Max **5** (soft limit — can be increased) |
| **CIDRs per VPC** | Max **5** |
| **Min CIDR size** | `/28` = 16 IP addresses |
| **Max CIDR size** | `/16` = 65,536 IP addresses |
| **IP type** | Private IPv4 only |

---

## Allowed Private IPv4 Ranges

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

---

## CIDR Selection Rule

**Do not overlap** your VPC CIDR with other VPCs, corporate networks, or any networks you may need to connect to later (VPC peering, VPN, Direct Connect).

```
VPC A: 10.0.0.0/16
VPC B: 10.1.0.0/16   ← non-overlapping ✓
Corporate: 192.168.0.0/24

If VPCs share overlapping CIDRs → cannot connect them together
```

---

## Quick Reference

```
VPC: Virtual Private Cloud — isolated network within an AWS region
  Max 5 per region (soft limit)
  Max 5 CIDRs per VPC
  CIDR size: /28 (min, 16 IPs) to /16 (max, 65,536 IPs)
  Private IPv4 only: 10/8, 172.16/12, 192.168/16

CIDR planning: avoid overlap with other VPCs/on-premises networks
```
