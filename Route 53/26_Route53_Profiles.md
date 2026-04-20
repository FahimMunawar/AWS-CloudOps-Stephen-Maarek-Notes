# 26 — Route 53 Profiles

## Overview

Centrally manage Route 53 DNS configuration and apply it consistently across many VPCs and AWS accounts.

---

## Supported Resources in a Profile

- Private hosted zones
- Route 53 Resolver rules
- Resolver DNS Firewall rule groups
- VPC Interface Endpoints

---

## How It Works

```
Create Profile
  → Associate resources:
      Private hosted zones
      Resolver rules
      DNS Firewall rule groups
      VPC endpoints
  → Associate with VPCs
    → All associated VPCs receive the same DNS configuration
```

---

## Cross-Account Sharing

Share a profile outside your account using **AWS Resource Access Manager (RAM)**:
- Share the profile → specify target accounts
- Target accounts apply the profile to their VPCs

---

## Additional Configuration (per profile)

- Resolver DNS lookup configuration
- DNSSEC configuration
- Failure mode configuration

---

## Quick Reference

```
Route 53 Profiles: centrally manage DNS config → apply to multiple VPCs/accounts

Resources in a profile:
  Private hosted zones, Resolver rules, DNS Firewall rule groups, VPC endpoints

Apply to VPCs → all receive identical DNS configuration
Share cross-account → AWS RAM

Use case: standardize DNS across a multi-account, multi-VPC environment
```
