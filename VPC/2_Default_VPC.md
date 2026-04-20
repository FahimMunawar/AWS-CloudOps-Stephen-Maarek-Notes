# 2 — Default VPC

## Overview

Every new AWS account includes a **default VPC** so you can launch resources immediately without networking setup.

| Feature | Default VPC |
|---|---|
| **CIDR** | `172.31.0.0/16` (65,536 IPs) |
| **Internet connectivity** | Yes — has an Internet Gateway attached |
| **EC2 public IPv4** | Auto-assigned to all instances |
| **Public + private DNS** | Provided for all instances |

> Best practice: create your own VPC for production. Use the default VPC only for initial learning/testing.

---

## Default VPC Components

### Subnets
- 3 subnets — one per AZ (for high availability)
- Each subnet has its own CIDR (e.g., `172.31.0.0/20` = 4,096 IPs)
- **Available IPs = 4,091** (not 4,096) — 5 IPs are reserved by AWS (explained later)
- **Auto-assign public IPv4** = enabled — all instances get a public IP automatically

### Route Table (Main / Default)
- Two rules:
  1. Local route: traffic within the VPC CIDR routes locally
  2. Default route: all other traffic (`0.0.0.0/0`) → Internet Gateway
- **Implicitly associated** with all subnets (no explicit association needed)

### Internet Gateway
- Attached to the default VPC
- Enables internet access for EC2 instances
- Without it, instances would have no internet connectivity

### Network ACL
- Default: allow all inbound and outbound traffic on all protocols

---

## 5 Reserved IPs per Subnet

Each subnet has 5 IPs reserved by AWS — covered in detail in a later lecture.
```
/20 subnet = 4,096 total − 5 reserved = 4,091 available
```

---

## Quick Reference

```
Default VPC:
  CIDR: 172.31.0.0/16
  3 subnets (one per AZ), auto-public IPv4
  Internet Gateway attached → instances get internet access
  Main route table: local + 0.0.0.0/0 → IGW
  Network ACL: allow all inbound + outbound

5 IPs reserved per subnet → available = total − 5
Best practice: create custom VPC for production workloads
```
