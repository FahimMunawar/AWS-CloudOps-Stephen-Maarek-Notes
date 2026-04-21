# 43 — Egress-Only Internet Gateway (Hands-On)

## Step 1: Create Egress-Only Internet Gateway

**VPC → Egress-Only Internet Gateways → Create egress-only internet gateway**

| Setting | Value |
|---|---|
| Name | `DemoEIGW` |
| VPC | DemoVPC |

Created and attached in one step.

---

## Step 2: Update Private Subnet Route Table

**VPC → Route Tables → PrivateRouteTable → Routes → Edit routes → Add route**

| Destination | Target |
|---|---|
| `::/0` | `DemoEIGW` (egress-only internet gateway) |

Save changes.

---

## Result

EC2 instances in private subnets can now initiate outbound IPv6 connections to the internet — the internet cannot initiate inbound connections to them.

---

## Quick Reference

```
Create: VPC → Egress-Only IGWs → Create → select VPC
Route:  PrivateRouteTable → add ::/0 → Egress-Only IGW

Effect: IPv6 outbound only from private subnet instances
```
