# 6 — S3 Private Access via VPC Endpoints

## Overview

Two ways to access Amazon S3 privately from within a VPC — without traffic traversing the public internet: **Gateway Endpoint** (preferred) and **Interface Endpoint** (for on-premises access).

---

## Option 1 — VPC Gateway Endpoint (Preferred)

```
EC2 Instance (private subnet)
       │
       ▼
VPC Gateway Endpoint for S3 (updates route table)
       │  private AWS network
       ▼
Amazon S3
```

| Feature | Detail |
|---|---|
| **Cost** | **Free** |
| **Scope** | Only resources within the same VPC |
| **How it works** | Updates the VPC route table to route S3 traffic through the endpoint |
| **DNS** | Continue using the public S3 DNS name — routing happens transparently |
| **Security Group** | EC2 security group must allow outbound traffic to the endpoint |
| **VPC setting required** | DNS support must be enabled on the VPC |
| **Use case** | EC2 instances in private subnets accessing S3 — **99% of use cases** |

### Setup Requirements
1. Enable **DNS support** on the VPC
2. Create a Gateway Endpoint for S3 in your VPC (updates route tables automatically)
3. Ensure EC2 security group allows outbound traffic to the endpoint

---

## Option 2 — VPC Interface Endpoint

```
EC2 Instance (private subnet)    On-Premises (via VPN or Direct Connect)
       │                                    │
       ▼                                    ▼
  ENI (Elastic Network Interface) ◄─────────┘
  (has its own security group)
       │  private AWS network
       ▼
Amazon S3
```

| Feature | Detail |
|---|---|
| **Cost** | ~**$0.01/hour per AZ** (deploy one per AZ for HA) |
| **Scope** | VPC resources **and** on-premises (via VPN or Direct Connect) |
| **How it works** | Deploys ENIs in your subnets — traffic routed through ENIs |
| **DNS** | DNS support + DNS hostnames must both be enabled on the VPC |
| **Security Group** | ENI has its own security group controlling inbound/outbound |
| **Use case** | On-premises systems need private S3 access |

---

## Comparison

| Feature | Gateway Endpoint | Interface Endpoint |
|---|---|---|
| **Cost** | **Free** | ~$0.01/hr/AZ |
| **On-premises access** | No | **Yes** (via VPN/Direct Connect) |
| **Scope** | VPC only | VPC + on-premises |
| **Implementation** | Route table update | ENI deployment |
| **VPC requirements** | DNS support | DNS support + DNS hostnames |
| **Exam preference** | **Primary answer** | Use when on-premises needed |

---

## When to Use Each

| Scenario | Endpoint Type |
|---|---|
| EC2 in private subnet needs S3 access | **Gateway Endpoint** |
| On-premises servers need private S3 access via VPN/Direct Connect | **Interface Endpoint** |
| Cost-sensitive, VPC-only access | **Gateway Endpoint** |

---

## SysOps Exam Q&A

**Q: An EC2 instance in a private subnet needs to access S3 without going over the public internet. What is the recommended solution?**
A: **VPC Gateway Endpoint for S3** — it's free, routes S3 traffic privately within the VPC, and covers 99% of use cases.

**Q: What is the cost of a VPC Gateway Endpoint for S3?**
A: **Free** — no charge for Gateway Endpoints.

**Q: On-premises servers need private access to S3 via Direct Connect. What endpoint type is required?**
A: **VPC Interface Endpoint** — Gateway Endpoints are VPC-only; Interface Endpoints support on-premises access via VPN or Direct Connect.

**Q: What VPC settings must be enabled for a Gateway Endpoint to work?**
A: **DNS support** must be enabled on the VPC.

**Q: What VPC settings must be enabled for an Interface Endpoint to work?**
A: Both **DNS support** and **DNS hostnames** must be enabled on the VPC.

---

## Quick Reference

```
Gateway Endpoint (S3):
  Cost:    Free
  Access:  VPC resources only
  Requires: DNS support ON, route table updated automatically
  Exam:    Default answer for "private S3 access from EC2"

Interface Endpoint (S3):
  Cost:    ~$0.01/hr/AZ
  Access:  VPC + on-premises (VPN or Direct Connect)
  Requires: DNS support + DNS hostnames both ON
  ENI deployed per AZ with its own security group
  Exam:    Use when on-premises access to S3 is required
```
