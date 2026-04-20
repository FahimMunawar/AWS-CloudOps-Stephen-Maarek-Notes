# 21 — Managed Prefix Lists

## Overview

A prefix list is a **named set of one or more CIDR blocks** used to simplify security group rules and route tables.

---

## Types

| Type | Managed By | Can Create/Modify/Delete | Shareable |
|---|---|---|---|
| **Customer-Managed** | You | Yes | Yes — across accounts or AWS Organization |
| **AWS-Managed** | AWS | No | No — read-only |

---

## Customer-Managed Prefix Lists

### Problem They Solve

Without prefix lists, adding three CIDRs to 10 security groups across 5 accounts = 150 individual rule entries. Any CIDR change means updating all of them.

### With Prefix Lists

```
Account A: define prefix list = [10.0.0.0/16, 172.16.0.0/12, 192.168.1.0/24]
  ↓ share via RAM or within Organization
Account B/C/D: security group rule → source = prefix list
  ↓
Update CIDRs once in the prefix list → all linked security groups update automatically
```

**Benefits:**
- Single source of truth for CIDR sets
- One update propagates to all linked security groups / route tables
- Separates network management from application team responsibilities

---

## AWS-Managed Prefix Lists

Pre-built CIDR lists for specific AWS services — automatically updated by AWS as IP ranges change.

| Service | Use Case |
|---|---|
| Amazon S3 | Allow S3 IP ranges in security groups |
| CloudFront | Allow CloudFront edge IPs |
| DynamoDB | Allow DynamoDB IP ranges |
| Ground Station | Allow Ground Station IPs |

> You cannot create, modify, share, or delete AWS-managed prefix lists.

---

## SysOps Exam Q&A

**Q: You need to allow traffic from Amazon S3 IP ranges in a security group. What is the cleanest approach?**
A: Reference the AWS-managed prefix list for S3 as the security group rule source — AWS keeps the IP list up to date automatically.

**Q: You manage 20 security groups across 8 accounts that all reference the same set of corporate CIDR blocks. How do you simplify updates?**
A: Create a customer-managed prefix list with those CIDRs, share it across accounts, and reference the prefix list in all security groups — one update propagates everywhere.

---

## Quick Reference

```
Prefix List = named set of CIDRs used in SG rules or route tables

Customer-managed:
  You define CIDRs → share across accounts/org
  Update once → all linked SGs/route tables update
  Use case: multi-account CIDR management

AWS-managed:
  AWS defines + updates automatically (S3, CloudFront, DynamoDB, Ground Station)
  Read-only — cannot modify or delete

Benefit: eliminates repetitive rule management, single source of truth
```
