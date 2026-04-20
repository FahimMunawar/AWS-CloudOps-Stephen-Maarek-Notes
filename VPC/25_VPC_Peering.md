# 25 — VPC Peering

## Overview

Connects two VPCs over the AWS private network so they behave as if in the same network. Works within an account, across accounts, and across regions.

---

## Requirements

| Requirement | Detail |
|---|---|
| **Non-overlapping CIDRs** | Peering fails if VPC CIDRs overlap |
| **Route tables** | Must be updated in each VPC's subnets to route traffic to the peer VPC CIDR |
| **Non-transitive** | Each pair of VPCs that needs to communicate requires its own peering connection |

---

## Non-Transitive Peering (Critical Exam Concept)

```
VPC A ←──peering──→ VPC B ←──peering──→ VPC C

A cannot reach C through B — a direct A↔C peering is required.
```

Even with A↔B and B↔C connected, traffic does NOT flow A→B→C automatically.

---

## Cross-Account Security Group Reference

In the same region, a security group rule can reference a **security group from a peered VPC** (even across accounts) instead of a CIDR:

```
Inbound rule source: sg-xxxxxxxx (from Account B, peered VPC)
```

This is more precise than a CIDR range and automatically tracks instance IP changes.

---

## Key Properties

| Property | Detail |
|---|---|
| Within same account | Supported |
| Across accounts | Supported |
| Across regions | Supported |
| Transitive routing | **Not supported** |
| Route table update | Required in all subnets that need to communicate |

---

## SysOps Exam Q&A

**Q: VPC A is peered with VPC B, and VPC B is peered with VPC C. Can A reach C?**
A: No — VPC peering is non-transitive. A direct A↔C peering connection must be created.

**Q: What must be done after creating a VPC peering connection for instances to communicate?**
A: Update route tables in each VPC's subnets to route the peer VPC's CIDR to the peering connection.

**Q: Can you peer VPCs across AWS accounts?**
A: Yes — VPC peering supports same account, cross-account, and cross-region.

**Q: Two VPCs have overlapping CIDR blocks. Can they be peered?**
A: No — non-overlapping CIDRs are required for VPC peering.

---

## Quick Reference

```
VPC Peering: connect two VPCs privately (same account, cross-account, cross-region)
  Non-overlapping CIDRs required
  Non-transitive: A↔B + B↔C ≠ A↔C (need explicit A↔C peering)
  Must update route tables in each subnet after peering

Cross-account SG reference (same region):
  SG rule source = peer VPC security group ID (no CIDR needed)
```
