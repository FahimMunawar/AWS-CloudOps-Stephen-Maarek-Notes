# 31 — Site-to-Site VPN (Hands-On)

## Three Steps to Establish a Site-to-Site VPN (Exam Summary)

1. Create a **Customer Gateway (CGW)** — on-premises side
2. Create a **Virtual Private Gateway (VGW)** — AWS side
3. Create a **Site-to-Site VPN Connection** linking the two

---

## Step 1: Customer Gateway

**VPC → VPN → Customer Gateways → Create Customer Gateway**

| Field | Value |
|---|---|
| Name | e.g., `OnPremCGW` |
| BGP ASN | Optional (advanced) |
| IP address | **Public IP of the on-premises gateway device** (how AWS reaches it) |
| Certificate ARN | Optional — for certificate-based authentication |

> Not demoed — requires actual on-premises infrastructure.

---

## Step 2: Virtual Private Gateway

**VPC → VPN → Virtual Private Gateways → Create Virtual Private Gateway**

| Field | Value |
|---|---|
| Name | e.g., `DemoVGW` |
| ASN | Amazon default ASN or custom |

After creation: **Actions → Attach to VPC** → select your VPC.

---

## Step 3: Site-to-Site VPN Connection

**VPC → VPN → Site-to-Site VPN Connections → Create VPN Connection**

| Field | Value |
|---|---|
| Target gateway type | Virtual private gateway |
| Virtual private gateway | Select the VGW created above |
| Customer gateway | Select the CGW created above |
| Routing options | Static or dynamic (BGP) |
| Tunnel options | (Advanced — not exam-required) |

---

## Post-Setup Requirement (Exam Critical)

Enable **route propagation** on VPC subnet route tables so VPN routes are advertised:

**Route Tables → select table → Route Propagation → Enable**

---

## Quick Reference

```
Site-to-Site VPN setup (3 steps):
  1. Create Customer Gateway (CGW) → on-premises public IP + optional cert
  2. Create Virtual Private Gateway (VGW) → attach to VPC
  3. Create Site-to-Site VPN Connection → link VGW + CGW

Post-setup: enable route propagation on route tables
Tunnel/routing details: not exam-required at this level
```
