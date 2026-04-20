# 16 — NAT Gateway (Hands-On)

## Goal

Replace the stopped NAT instance with a managed NAT Gateway to restore outbound internet access for private subnets.

---

## The Black Hole Problem

After stopping the NAT instance, the private route table shows a **black hole** — the route `0.0.0.0/0 → ENI` is inactive because the target no longer exists.

> Black hole = route pointing to a stopped/terminated resource. Another reason managed services (NAT Gateway) are preferred over NAT instances.

---

## Step 1: Create NAT Gateway

**VPC → NAT Gateways → Create NAT gateway**

| Setting | Value |
|---|---|
| Name | `DemoNATGW` |
| Subnet | `PublicSubnetA` (public subnet — required) |
| Connectivity type | Public |
| Elastic IP | Click **Allocate Elastic IP** |

Click **Create NAT gateway** — state will show **Pending** for a few minutes.

---

## Step 2: Update Private Route Table

**VPC → Route Tables → PrivateRouteTable → Routes → Edit routes**

Remove the black hole entry, then add:

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | `DemoNATGW` (NAT Gateway) |

Save — both routes will show as **Active** once the NAT Gateway is ready.

---

## Verification

```bash
# On private EC2 (via bastion SSH)
curl google.com       # returns HTML → internet access confirmed
ping google.com       # works

# Practical use case
sudo yum update       # update OS packages without making instance public
```

No security group rules were configured — NAT Gateway requires none.

---

## HA Reminder

This demo uses one NAT Gateway in `PublicSubnetA` (eu-central-1a only). For production:

```
PublicSubnetA → DemoNATGW-A  ←  PrivateSubnetA route table
PublicSubnetB → DemoNATGW-B  ←  PrivateSubnetB route table
```

Each AZ gets its own NAT Gateway — if one AZ fails, the other continues unaffected.

---

## Quick Reference

```
NAT Gateway Hands-On:
  1. VPC → NAT Gateways → Create → PublicSubnetA + allocate Elastic IP
  2. Wait for state: Pending → Active
  3. PrivateRouteTable: replace black hole with 0.0.0.0/0 → DemoNATGW
  4. Verify: curl/ping from private EC2 via bastion

No security group config needed
For HA: one NAT GW per AZ, each AZ's private route table points to its own NAT GW
```
