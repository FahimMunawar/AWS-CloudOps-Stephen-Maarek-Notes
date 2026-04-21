# 30 — Site-to-Site VPN

## Overview

Connects an AWS VPC to an on-premises corporate data center over an **encrypted VPN tunnel through the public internet**.

```
Corporate Data Center                    AWS VPC
─────────────────────                    ────────────────────
Customer Gateway (CGW)  ←── VPN ──→  Virtual Private Gateway (VGW)
(on-premises device)      (encrypted,    (attached to VPC)
                          public internet)
```

---

## Two Components

| Component | Side | Description |
|---|---|---|
| **Virtual Private Gateway (VGW)** | AWS | VPN concentrator; created and attached to VPC; customizable ASN |
| **Customer Gateway (CGW)** | On-premises | Software or physical device; tested devices listed by AWS |

---

## Customer Gateway IP Address (Exam)

| CGW Type | IP to Use |
|---|---|
| Public (direct internet IP) | Public IP of the CGW device |
| Private (behind NAT device with NAT-T enabled) | **Public IP of the NAT device** |

---

## Setup Requirements (Exam Critical)

1. Create VGW → attach to VPC
2. Create CGW → specify IP address
3. Create site-to-site VPN connection (VGW ↔ CGW)
4. **Enable route propagation** in VPC route tables — without this, the VPN will not route traffic even if the tunnel is up
5. **Security group ICMP rule** — if pinging EC2 from on-premises, enable inbound ICMP on the instance's security group

---

## AWS VPN CloudHub

Connect **multiple on-premises sites** through a single VGW using a hub-and-spoke model.

```
Site A (CGW-A) ──┐
Site B (CGW-B) ──┤── VGW (single) ──→ VPC
Site C (CGW-C) ──┘
        ↕              ↕
  Sites can also communicate with each other through the VGW
```

| Property | Detail |
|---|---|
| Use case | Primary or secondary connectivity between multiple sites |
| Cost | Low — hub-and-spoke over VPN |
| Traffic | Over public internet (VPN encrypted) |
| Setup | Multiple site-to-site VPN connections on same VGW + enable dynamic routing + configure route tables |

---

## SysOps Exam Q&A

**Q: You set up a site-to-site VPN but traffic is not routing between VPC and on-premises. What is likely missing?**
A: Route propagation is not enabled in the VPC subnet route tables — this must be explicitly enabled for the VPN routes to be advertised.

**Q: An on-premises customer gateway has a private IP and is behind a NAT device. What IP do you configure in AWS for the CGW?**
A: The public IP of the NAT device (NAT traversal / NAT-T must be enabled on the NAT device).

**Q: EC2 ping from on-premises is failing despite a working site-to-site VPN. What should you check?**
A: The EC2 security group inbound rule — ICMP protocol must be explicitly allowed.

**Q: You need to connect 3 branch offices to AWS and allow the branches to communicate with each other. What do you use?**
A: AWS VPN CloudHub — establish site-to-site VPN from each branch CGW to the same VGW, enable dynamic routing. Branches can communicate through the VGW hub.

---

## Quick Reference

```
Site-to-Site VPN:
  VGW (AWS side) + CGW (on-premises) → encrypted tunnel over public internet

CGW IP:
  Public CGW → use its public IP
  Private CGW behind NAT (NAT-T) → use NAT device's public IP

Required after tunnel setup:
  Enable route propagation in VPC route tables
  Enable inbound ICMP on SG if pinging EC2 from on-premises

VPN CloudHub:
  Multiple CGWs → one VGW → sites communicate through hub
  Low-cost, public internet (encrypted), dynamic routing required
```
