# 33 — AWS Direct Connect (DX)

## Overview

A **dedicated private connection** from an on-premises network to AWS — does not go over the public internet.

| Property | Detail |
|---|---|
| Setup time | **Over 1 month** (physical provisioning) |
| Encryption | **None by default** — private but not encrypted |
| VPC side | Requires a Virtual Private Gateway (VGW) |
| IPv4/IPv6 | Both supported |

---

## Use Cases

- High bandwidth / large data set transfers (lower cost, faster than internet)
- Consistent network performance (no public internet variability)
- Real-time data feed applications
- Hybrid cloud (on-premises ↔ AWS private connectivity)

---

## Virtual Interfaces (VIFs)

| VIF Type | Connects To | Use For |
|---|---|---|
| **Private VIF** | VGW → VPC private subnets | EC2 instances, private resources |
| **Public VIF** | AWS public endpoints | S3, Glacier, and other public AWS services |

Both VIF types can run on the **same** Direct Connect connection.

---

## Architecture

```
Corporate Data Center
  → Customer Router/Firewall
    → [dedicated line]
      → Direct Connect Location (AWS + Customer/Partner cage)
        → Private VIF → Virtual Private Gateway → VPC (private subnets)
        → Public VIF  → AWS public services (S3, Glacier)
```

---

## Direct Connect Gateway (Multi-Region / Multi-VPC)

Connect one Direct Connect to multiple VPCs in different regions:

```
On-Premises
  → Direct Connect
    → Private VIF → Direct Connect Gateway
                      ├── VGW → VPC (us-east-1)
                      └── VGW → VPC (eu-west-1)
```

---

## Connection Types

| Type | Speed | Provisioned By |
|---|---|---|
| **Dedicated** | 1 Gbps – 400 Gbps | AWS first, then Direct Connect partner |
| **Hosted** | 50 Mbps – 25 Gbps | AWS Direct Connect partner (capacity on demand) |

> Both take **over 1 month** to establish.

---

## Encryption

Direct Connect is private but **not encrypted**. To add encryption:

```
Direct Connect + VPN (IPsec)
  → encrypted private connection over the Direct Connect physical link
```

More complex to set up but provides both privacy and encryption.

---

## Resiliency (Exam Critical)

### High Resiliency (Critical Workloads)

```
Data Center A → DX Location 1 ──┐
Data Center B → DX Location 2 ──┴──→ AWS
```

One connection per DX location across two locations — if one location fails, the other maintains connectivity.

### Maximum Resiliency (Critical Workloads)

```
Data Center A → DX Location 1 (2 connections) ──┐
Data Center B → DX Location 2 (2 connections) ──┴──→ AWS
```

Two connections per location across two locations (4 total) — separate connections terminating on separate devices. Maximum resilience achieved by redundancy at every layer.

---

## SysOps Exam Q&A

**Q: A company needs to transfer large datasets to AWS within one week using a private connection. Can Direct Connect be used?**
A: No — Direct Connect takes over one month to set up. Use S3 Transfer Acceleration, Snowball, or an existing connection.

**Q: Direct Connect provides a private connection. Is the data encrypted?**
A: No — Direct Connect is private but not encrypted by default. Add a VPN (IPsec) on top for encryption.

**Q: You need to connect one on-premises data center to VPCs in two different AWS regions via Direct Connect. What do you use?**
A: Direct Connect Gateway — establish one DX connection, use a private VIF to the gateway, then connect to VGWs in each region.

**Q: What is the difference between high resiliency and maximum resiliency for Direct Connect?**
A: High resiliency = one connection per DX location across two locations. Maximum resiliency = two connections per DX location across two locations (4 total, on separate devices).

---

## Quick Reference

```
Direct Connect (DX):
  Dedicated private line (not public internet) — setup takes 1+ month
  No encryption by default → add VPN (IPsec) for encryption
  Requires VGW on VPC side

VIFs:
  Private VIF → VGW → VPC private resources
  Public VIF → AWS public services (S3, Glacier)

Direct Connect Gateway: one DX → multiple VPCs across regions

Connection types:
  Dedicated: 1–400 Gbps (AWS + partner)
  Hosted: 50 Mbps – 25 Gbps (partner, on-demand capacity)

Resiliency:
  High: 1 connection × 2 DX locations
  Maximum: 2 connections × 2 DX locations (4 total, separate devices)

Exam trap: DX takes 1+ month → not a solution for urgent data transfer
```
