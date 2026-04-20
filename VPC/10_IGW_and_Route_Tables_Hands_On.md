# 10 — Internet Gateway and Route Tables (Hands-On)

## Goal

Give internet access to public subnets by attaching an IGW and updating route tables.

---

## Step 1: Enable Auto-Assign Public IPv4 on Public Subnets

**Subnets → PublicSubnetA → Actions → Edit subnet settings → Enable auto-assign public IPv4 address**

Repeat for PublicSubnetB.

> Without this, EC2 instances in public subnets won't receive a public IPv4 — even if the IGW and routes are correct.

---

## Step 2: Create and Attach an Internet Gateway

**VPC → Internet Gateways → Create internet gateway** → name: `DemoIGW`

After creation, state = **Detached** — must attach to a VPC:

**Actions → Attach to VPC → DemoVPC**

> IGW alone does NOT give internet access — route table must also be configured.

---

## Step 3: Create Dedicated Route Tables

Create two route tables (instead of using the main/default one):

| Route Table | VPC | Assigned Subnets |
|---|---|---|
| `PublicRouteTable` | DemoVPC | PublicSubnetA, PublicSubnetB |
| `PrivateRouteTable` | DemoVPC | PrivateSubnetA, PrivateSubnetB |

**Route Tables → Create route table → name + VPC → Edit subnet associations → save**

> Best practice: explicitly assign subnets to route tables rather than relying on the implicit main route table association.

---

## Step 4: Add Internet Route to Public Route Table

**PublicRouteTable → Routes → Edit routes → Add route**

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local (auto-existing — routes VPC-internal traffic) |
| `0.0.0.0/0` | `DemoIGW` (internet gateway) |

Logic:
```
Traffic destined for VPC CIDR → local (stays inside VPC)
Everything else (public IPs)  → Internet Gateway → internet
```

---

## Verification

EC2 instance in PublicSubnetA → EC2 Instance Connect → connect → success

```bash
ping google.com   # returns data → internet access confirmed
```

---

## What's Still Missing

Private subnets have no internet access yet — this requires a **NAT Gateway**, covered in the next lecture.

---

## Quick Reference

```
Make a subnet public:
  1. Enable auto-assign public IPv4 on the subnet
  2. Attach IGW to VPC
  3. Create dedicated PublicRouteTable → assign public subnets
  4. Add route: 0.0.0.0/0 → IGW

Route table best practice: explicit subnet associations > relying on main route table

Private subnets: no 0.0.0.0/0 → IGW route → no direct internet access
  → use NAT Gateway for outbound-only internet access
```
