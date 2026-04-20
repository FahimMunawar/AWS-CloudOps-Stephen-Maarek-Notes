# 9 — Internet Gateway (IGW)

## Overview

Allows resources inside a VPC to connect to the internet.

| Feature | Detail |
|---|---|
| **Scaling** | Horizontally scaled, highly available, redundant (managed by AWS) |
| **Created** | Separately from the VPC — must be attached |
| **VPC limit** | One IGW per VPC (and one VPC per IGW) |
| **Alone is not enough** | IGW must be paired with a Route Table entry to enable internet access |

---

## What Makes a Subnet Public

Two things required:
1. **Internet Gateway** attached to the VPC
2. **Route Table** with a rule sending `0.0.0.0/0` → IGW

```
EC2 instance
  → Router (VPC)
    → Internet Gateway
      → Internet
```

Without a route table entry pointing to the IGW, the IGW does nothing.

---

## Quick Reference

```
Internet Gateway:
  Enables internet access for VPC resources
  1 IGW per VPC, 1 VPC per IGW
  Must be attached to the VPC

IGW alone ≠ internet access
IGW + route table (0.0.0.0/0 → IGW) = public subnet
```
