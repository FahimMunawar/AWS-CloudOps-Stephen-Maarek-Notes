# 41 — IPv6 in VPC

## Overview

IPv6 is the successor to IPv4, designed to provide 3.4 × 10³⁸ unique addresses. In AWS, all IPv6 addresses are **public and internet-routable**.

---

## IPv4 vs IPv6 in AWS VPC

| Property | IPv4 | IPv6 |
|---|---|---|
| Can be disabled | **No** — always enabled | Optional |
| Address type | Private (internal) + Public (optional) | Always public |
| Format | `x.x.x.x` (decimal) | `x:x:x:x:x:x:x:x` (hexadecimal, 8 groups) |
| Example | `10.0.1.5` | `2001:0db8:85a3:0000:0000:8a2e:0370:7334` |

---

## Dual-Stack Mode

When IPv6 is enabled on a VPC, instances operate in **dual-stack mode**:

- Each EC2 instance gets: **private IPv4** + **public IPv6**
- Can communicate to internet via either IPv4 or IPv6
- Internet Gateway supports both IPv4 and IPv6

```
EC2 Instance
  Private IPv4: 10.0.1.5 (internal only)
  Public IPv6:  2001:db8::1 (public, internet-routable)
      │
  Internet Gateway (handles both IPv4 and IPv6)
      │
  Internet
```

---

## Exam Scenario: IPv6 Troubleshooting

**Symptom:** Cannot launch a new EC2 instance in an IPv6-enabled VPC/subnet.

**Wrong assumption:** IPv6 addresses are exhausted.

**Actual cause:** The **IPv4 address space in the subnet is exhausted** — IPv4 cannot be disabled and is still required for every instance.

**Solution:** Add a new IPv4 CIDR block to the subnet (or VPC).

> IPv6 space is virtually inexhaustible — if instance launch fails, always check IPv4 availability first.

---

## SysOps Exam Q&A

**Q: An IPv6-enabled subnet returns an error when launching a new EC2 instance. What is the most likely cause?**
A: The IPv4 CIDR space is exhausted — IPv4 cannot be disabled and every instance requires an IPv4 address. Solution: add a new IPv4 CIDR to the subnet.

**Q: Can you disable IPv4 for a VPC that has IPv6 enabled?**
A: No — IPv4 cannot be disabled for VPCs or subnets regardless of IPv6 configuration.

**Q: What type of IP addresses are IPv6 addresses in AWS?**
A: Always public and internet-routable — there are no private IPv6 addresses in AWS VPCs.

---

## Quick Reference

```
IPv6 in VPC:
  Always public + internet-routable
  IPv4 cannot be disabled (always required)
  Dual-stack: EC2 gets private IPv4 + public IPv6
  IGW supports both IPv4 and IPv6

Exam trap: can't launch EC2 in IPv6 subnet?
  → IPv4 space exhausted (not IPv6)
  → Fix: add new IPv4 CIDR to subnet/VPC
```
