# 19. AWS Resource Access Manager (RAM)

## Overview

**AWS Resource Access Manager (RAM)** allows you to share AWS resources you own with other AWS accounts — within your organization or with any external account. The goal is to avoid resource duplication and simplify cross-account networking.

---

## What You Can Share

| Resource | Notes |
|----------|-------|
| **VPC Subnets** | Most exam-relevant — share networking layer across accounts |
| AWS Transit Gateway | Cross-account network transit |
| Route 53 Resolver Rules | DNS forwarding rules |
| License Manager Configurations | Software license sharing |
| Aurora DB Clusters | Database sharing |
| Capacity Reservations | EC2 reserved capacity |
| CodeBuild Projects | CI/CD sharing |
| Dedicated Hosts | Hardware sharing |

> **Cannot share:** security groups, the default VPC.

---

## VPC Subnet Sharing — Key Rules

| Rule | Detail |
|------|--------|
| **What is shared** | The subnet (networking layer only) |
| **What is NOT shared** | Resources inside the VPC (EC2, RDS, ALB, etc.) |
| **Who can deploy** | Each participant account deploys its own resources into the shared subnet |
| **Visibility** | Participants **cannot** view, modify, or delete resources that belong to other accounts |
| **Organization requirement** | VPC subnet sharing requires accounts to be in the **same AWS Organization** |

---

## Use Case — Shared VPC Architecture

```
VPC Owner Account
  ├── Private Subnet (shared via RAM)
  │     ├── Account 1: EC2 + ALB  (Account 2 and Owner cannot see these)
  │     ├── Account 2: EC2 instances  (Account 1 and Owner cannot see these)
  │     └── VPC Owner: RDS database  (Account 1 and 2 cannot see this)
  │
  └── All resources communicate via PRIVATE IP
      (no public internet required)
```

**Security group cross-account reference:**
- The VPC Owner can modify the RDS security group to allow the security group of Account 1's EC2 instances
- This enables Account 1's EC2 → VPC Owner's RDS over private IP
- Account 1's ALB security group can be referenced by Account 2's EC2 instances — private communication across accounts

---

## Benefits

| Benefit | Detail |
|---------|--------|
| **No public IPs needed** | All cross-account communication uses private IPs within the shared VPC |
| **Security** | Resources are isolated per account — no cross-account visibility |
| **Simplified networking** | One VPC, many accounts — no VPC peering or Transit Gateway required for basic use |
| **Cost** | Avoids duplicating VPCs, NAT Gateways, and other infrastructure per account |

---

## Console Workflow

1. **RAM** → **Create resource share**
2. Name the share (e.g., `MyFirstResourceShare`)
3. Select resource type → **Subnets** → select subnet
4. Add principals — account ID, OU, or entire organization
5. Create → recipient account must **accept** the share

---

## Best Practices

✓ **Use RAM for shared VPC subnets** — simplest way to give multiple accounts access to the same network layer  
✓ **Use within AWS Organizations** — subnet sharing requires same-org membership  
✓ **Reference security groups cross-account** — enables least-privilege access between accounts without exposing public IPs  
✓ **Remember: shared subnet ≠ shared resources** — each account owns and manages its own resources  

---

## SysOps Exam Focus

**Q1: "You want multiple AWS accounts in your organization to deploy EC2 instances into the same VPC subnet so they can communicate over private IPs. What service should you use?"**
- A) VPC Peering — connect VPCs in each account
- B) AWS Resource Access Manager — share the VPC subnet with other accounts in the organization
- C) AWS Transit Gateway — route traffic between account VPCs
- D) AWS PrivateLink — expose services to other accounts
- **Answer: B** — RAM is designed exactly for this: share a subnet so multiple accounts deploy into the same network layer

**Q2: "Account A shares a VPC subnet with Account B using RAM. Account A deploys an RDS database into the subnet. Can Account B see or modify that RDS database?"**
- A) Yes — the shared subnet means all resources are visible to all participants
- B) No — participants can only manage their own resources; they cannot view, modify, or delete resources belonging to other accounts
- C) Yes — but only if Account B has the correct IAM policy
- D) Yes — Account B can see but not modify it
- **Answer: B** — RAM shares the networking layer only; each account's resources remain invisible to other participants

**Q3: "Which of the following CANNOT be shared using AWS Resource Access Manager?"**
- A) VPC Subnets
- B) AWS Transit Gateway
- C) Security Groups
- D) Route 53 Resolver Rules
- **Answer: C** — Security groups and the default VPC cannot be shared via RAM

**Q4: "What is the primary exam signal for AWS Resource Access Manager?"**
- A) Centrally managing IAM policies across accounts
- B) Sharing VPC subnets across accounts to avoid resource duplication and enable private IP communication
- C) Monitoring cross-account API calls in CloudTrail
- D) Replicating S3 buckets across accounts
- **Answer: B** — RAM = shared VPC subnets = cross-account private networking without duplication

---

## Quick Reference

```
AWS Resource Access Manager (RAM):
  Purpose: share AWS resources with other accounts
  Scope: any account OR within AWS Organization

  Most exam-relevant resource: VPC Subnets
  Requires same AWS Organization for subnet sharing

  Cannot share: security groups, default VPC

  Shared subnet rules:
    ✓ Each account deploys its own resources into the subnet
    ✓ Resources communicate via private IP (no public IP needed)
    ✗ Participants cannot see/modify/delete other accounts' resources
    ✗ The shared thing is the network layer only

  Console: RAM → Create resource share → select subnet
           → add principals (account/OU/org) → recipient accepts

Exam signal: "share VPC subnet across accounts" → RAM
```

---

**File: 19_AWS_Resource_Access_Manager.md**
**Status: SysOps-focused, exam-ready, concise format**
