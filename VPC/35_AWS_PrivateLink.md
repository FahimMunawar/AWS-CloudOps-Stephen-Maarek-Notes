# 35 — AWS PrivateLink (VPC Endpoint Services)

## Problem: Exposing a Service to Many VPCs

| Option | Issue |
|---|---|
| Make it public (IGW) | Goes over public internet — not reliable or secure |
| VPC Peering | Requires N peering connections; opens entire network, not just one service |
| **PrivateLink** | Scalable, secure, no peering/IGW/NAT/route tables needed |

> **Exam trigger:** "Expose a service to thousands of VPCs (own or other accounts)" → **PrivateLink**

---

## How PrivateLink Works

```
Service VPC                           Customer VPC
─────────────────                     ─────────────────
Application Service                   Consumer Application
       │                                     │
Network Load Balancer (NLB)  ←──────→  ENI (PrivateLink)
       │                    PrivateLink      │
       └── private AWS network ──────────────┘
```

- Service side: requires a **Network Load Balancer (NLB)** — most common; Gateway Load Balancer also supported
- Consumer side: an **ENI** is created in the customer VPC
- No VPC peering, no IGW, no NAT, no route table changes needed
- Traffic stays entirely on the **private AWS network**

---

## Multi-AZ Fault Tolerance

If the NLB is deployed across multiple AZs → create ENIs in corresponding AZs on the consumer side → fault-tolerant PrivateLink setup.

---

## PrivateLink with ECS (Advanced Example)

```
ECS Tasks → ALB → NLB (ALB as NLB target)
                     │
                  PrivateLink
                     │
         ┌───────────┴───────────┐
         │                       │
    ENI (other VPC)    Direct Connect / VPN (on-premises)
```

ALB can be registered as a target of the NLB, allowing PrivateLink to front an ECS-backed service.

---

## SysOps Exam Q&A

**Q: You need to expose an internal service in your VPC to thousands of customer VPCs securely without opening the whole network. What do you use?**
A: AWS PrivateLink — place a Network Load Balancer in front of your service; customers create ENIs in their VPCs; connection is established privately without VPC peering.

**Q: What is required on the service provider side to set up PrivateLink?**
A: A Network Load Balancer (or Gateway Load Balancer) in the service VPC.

**Q: Does PrivateLink require VPC peering?**
A: No — PrivateLink does not require VPC peering, IGW, NAT gateway, or route table changes.

---

## Quick Reference

```
PrivateLink (VPC Endpoint Services):
  Expose one service to thousands of VPCs (own or other accounts)
  No VPC peering / IGW / NAT / route tables needed
  Traffic stays on private AWS network

Service side:  Network Load Balancer (NLB)
Consumer side: ENI in customer VPC

Multi-AZ: NLB in multiple AZs → ENIs in matching AZs → fault tolerant

vs VPC Peering: peering opens entire network; PrivateLink exposes only one service
```
