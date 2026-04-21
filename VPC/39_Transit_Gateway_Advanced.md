# 39 — Transit Gateway Advanced Features

## 1. Inter-Region Communication

Connect VPCs across AWS regions by peering Transit Gateways:

```
Region A                    Region B
────────────                ────────────
VPC-1 ─┐                   VPC-3 ─┐
VPC-2 ─┴─ TGW-A ←── peering ──→ TGW-B ─┴─ VPC-4
```

- One Transit Gateway per region
- Peer TGWs together → transitive inter-region VPC communication
- Follows AWS best practices for multi-region networking

---

## 2. Cross-VPC Security Group Referencing

Normally, security group rules can only reference SGs within the same VPC. With Transit Gateway, you can reference a security group from a **peered VPC** in your SG rules.

**Use case:**
```
VPC-A: DB instance SG (database-sg)
VPC-B: Web server SG (webserver-sg)

SG rule on database-sg:
  Inbound: allow TCP 3306 from webserver-sg (VPC-B)  ← cross-VPC SG reference
```

**Requirement:** Enable **Security Group Referencing Support** on the Transit Gateway attachment when creating it.

---

## SysOps Exam Q&A

**Q: How do you enable VPCs in different AWS regions to communicate with each other?**
A: Deploy one Transit Gateway per region and peer the Transit Gateways together — VPCs in each region connect through their regional TGW, and TGW peering provides inter-region transit.

**Q: You want a security group in VPC-A to reference a security group in VPC-B. What is required?**
A: A Transit Gateway connecting both VPCs, with **Security Group Referencing Support** enabled on the TGW attachment.

---

## Quick Reference

```
TGW inter-region:
  One TGW per region → peer TGWs → inter-region VPC connectivity

Cross-VPC SG referencing via TGW:
  Requires: Security Group Referencing Support enabled on TGW attachment
  Allows: SG rules to reference SGs from peered VPCs
```
