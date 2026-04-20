# 8 — VPC IP Address Manager (IPAM)

## Overview

Centrally plan, track, and monitor IP address spaces across hundreds of accounts and VPCs — single source of truth for all IP address information.

---

## Key Components

| Component | Description |
|---|---|
| **Scope** | Top-level container for all IP address space (default public + private scopes) |
| **IP Pool** | Collection of CIDRs; can be divided into sub-pools per region |
| **Child Pool** | Regional sub-pool allocated from the parent pool |

---

## Architecture Example

```
IPAM (management account)
  └── Scope
        └── Parent IP Pool: 10.0.0.0/8
              ├── Region A child pool: 10.0.0.0/16
              │     └── VPCs in Region A → must be within /16 ✓
              └── Region B child pool: 10.1.0.0/16
                    └── VPCs in Region B → must be within /16 ✓
                          (non-overlapping with Region A)
```

VPCs created outside the assigned child pool → refused.

---

## Use Cases

- Multi-account IP address management (prevent overlaps)
- Audit public IP usage
- Identify and reclaim unused IP addresses
- Monitor CIDR allocation
- Alert on potential overlapping IP ranges

---

## Quick Reference

```
IPAM: centrally manage IP address space across accounts/regions
  Scope → IP Pool → Child Pools (per region/account)
  Enforces non-overlapping CIDRs across all VPCs

Use cases: multi-account IP governance, unused IP reclamation, overlap detection
```
