# 38 — AWS Transit Gateway

## Overview

A hub-and-spoke network transit hub that provides **transitive peering** between thousands of VPCs, on-premises networks, Direct Connect, and VPN connections — all through a single gateway.

```
             VPC A
               │
VPC B ──── Transit Gateway ──── Direct Connect Gateway → On-Premises
               │
VPC C      VPN Connection
```

All attached VPCs can communicate with each other and with on-premises — no need to create individual VPC peering connections.

---

## Key Properties

| Property | Detail |
|---|---|
| Scope | Regional (can work cross-region via peering) |
| Cross-account sharing | Via AWS Resource Access Manager (RAM) |
| Route control | Route tables on the TGW — control which VPCs/connections can communicate |
| IP Multicast | **Only AWS service supporting IP multicast** |
| Connects to | VPCs, Direct Connect Gateway, Site-to-Site VPN, other Transit Gateways |

---

## ECMP — Increase VPN Bandwidth (Exam)

**ECMP = Equal-Cost Multi-Path routing** — forwards packets over multiple best paths simultaneously.

| Setup | Tunnels | Max Throughput |
|---|---|---|
| VPN → VGW (direct to VPC) | 2 tunnels (1 connection used) | 1.25 Gbps |
| VPN → Transit Gateway (1 connection) | 2 tunnels (both used via ECMP) | **2.5 Gbps** |
| VPN → Transit Gateway (2 connections) | 4 tunnels | **5.0 Gbps** |
| VPN → Transit Gateway (3 connections) | 6 tunnels | **7.5 Gbps** |

> Each additional Site-to-Site VPN attachment doubles the throughput. Cost: charged per GB through the TGW.

---

## Share Direct Connect Across Multiple Accounts

```
Corporate Data Center
  → Direct Connect Location
    → Direct Connect Gateway
      → Transit Gateway
          ├── VPC (Account A)
          └── VPC (Account B)
```

One Direct Connect connection shared across multiple accounts and VPCs via Transit Gateway — no need for separate Direct Connect per account.

---

## SysOps Exam Q&A

**Q: You have 10 VPCs and need them all to communicate with each other and with on-premises. What is the most scalable solution?**
A: Transit Gateway — transitive routing means all VPCs connect through the hub without individual peering connections.

**Q: Which AWS service supports IP multicast?**
A: Transit Gateway — it is the only AWS service that supports IP multicast.

**Q: How do you increase Site-to-Site VPN throughput beyond 1.25 Gbps?**
A: Connect the VPN to a Transit Gateway and use ECMP — each VPN connection uses both tunnels, giving 2.5 Gbps per connection. Add more VPN connections to multiply throughput further.

**Q: How do you share a single Direct Connect connection across multiple AWS accounts?**
A: Use Transit Gateway with Direct Connect Gateway — attach multiple account VPCs to the TGW; all share the single DX connection.

**Q: How do you control which VPCs can communicate with each other through a Transit Gateway?**
A: Create route tables on the Transit Gateway and configure which attachments (VPCs/VPNs) are associated and propagated to each table.

---

## Quick Reference

```
Transit Gateway: hub-and-spoke for VPCs + VPN + Direct Connect + cross-account
  Transitive routing — no VPC peering mesh needed
  Share across accounts: AWS RAM
  Route control: TGW route tables

IP Multicast: Transit Gateway only

ECMP (increase VPN bandwidth):
  VPN → VGW: 1.25 Gbps (2 tunnels, 1 used)
  VPN → TGW (1x): 2.5 Gbps (2 tunnels, ECMP both)
  VPN → TGW (2x): 5.0 Gbps | (3x): 7.5 Gbps
  Cost: per GB through TGW

Share Direct Connect: DX → DX Gateway → TGW → multiple account VPCs
```
