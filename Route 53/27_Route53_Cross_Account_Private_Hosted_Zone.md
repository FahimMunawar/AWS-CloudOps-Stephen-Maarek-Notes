# 27 — Route 53: Cross-Account Private Hosted Zone

## Overview

Associate a private hosted zone in one account with a VPC in a different account.

---

## Steps

```
Account A (owns the private hosted zone):
  → Create VPC association authorization
    aws route53 create-vpc-association-authorization \
      --hosted-zone-id <zone-id> \
      --vpc VPCRegion=<region>,VPCId=<vpc-id-from-account-B>

Account B (owns the VPC):
  → Create the VPC association (as normal)
    → Authorization from Account A allows it
```

---

## Quick Reference

```
Cross-account private hosted zone association:
  1. Account A: create VPC association authorization for Account B's VPC
  2. Account B: create VPC association → accepted automatically

Use case: share a private hosted zone (e.g., company-internal DNS) with VPCs in other accounts
```
