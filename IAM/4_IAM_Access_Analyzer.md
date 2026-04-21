# 4 — IAM Access Analyzer

## Overview

Identifies resources in your account that are shared with **external principals** (outside your zone of trust) and reports them as findings.

**Cost: Free**

---

## Supported Resource Types

- S3 Buckets
- IAM Roles
- KMS Keys
- Lambda Functions and Layers
- SQS Queues
- Secrets Manager Secrets

---

## Zone of Trust

Define what counts as "internal" — anything outside this zone with access to your resources generates a finding:

| Zone of Trust Option | Scope |
|---|---|
| AWS Account | Single account — cross-account access = finding |
| AWS Organization | Entire org — access from outside org = finding |

---

## How It Works

```
S3 Bucket policy grants access to:
  ├── Role (same account)        ← inside zone of trust → OK
  ├── User (same account)        ← inside zone of trust → OK
  ├── VPC endpoint               ← inside zone of trust → OK
  ├── External AWS account       ← outside zone of trust → FINDING
  └── External client (IP)       ← outside zone of trust → FINDING
```

---

## Finding States

| State | Meaning |
|---|---|
| **Active** | External access exists — review required |
| **Resolved** | Access has been removed (e.g., policy updated) |
| **Archived** | Acknowledged as intentional — suppressed from active view |

---

## Hands-On Workflow

1. **IAM → Access Analyzer → Create analyzer** → name it, set zone of trust (current account) → Create
2. Review findings — each shows: resource type, external principal, access level (read/write)
3. For unintended access: go to the resource, fix the policy, **Rescan** → status changes to Resolved
4. For intended access: **Archive** the finding → moves to Archived tab
5. Create **Archive Rules** to automatically archive future findings matching specific criteria

---

## SysOps Exam Q&A

**Q: How do you identify which S3 buckets in your account are accessible by external AWS accounts?**
A: Enable IAM Access Analyzer with the zone of trust set to your account — it will report any bucket with a policy granting access to external principals as a finding.

**Q: An IAM Access Analyzer finding shows an SQS queue is accessible by anyone. What steps do you take?**
A: Review the SQS access policy, remove the overly permissive statement, then rescan — the finding status will change to Resolved.

**Q: You intentionally make an S3 bucket public. How do you prevent it from cluttering your findings?**
A: Archive the finding, or create an archive rule to automatically suppress findings that match that criteria.

---

## Quick Reference

```
IAM Access Analyzer (free):
  Finds resources shared with external principals (outside zone of trust)
  Resources: S3, IAM roles, KMS keys, Lambda, SQS, Secrets Manager

Zone of trust: AWS Account or AWS Organization

Finding states: Active → fix policy + Rescan → Resolved
               Active → Archive (intentional) → Archived

Archive rules: auto-archive future findings matching defined criteria
```
