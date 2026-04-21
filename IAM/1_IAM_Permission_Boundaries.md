# 1 — IAM Permission Boundaries

## Overview

A permission boundary is an advanced IAM feature that sets the **maximum permissions** an IAM entity (user or role) can be granted — it does not grant permissions itself.

> Supported for: **users and roles only** — not groups.

---

## How It Works

The effective permissions = **intersection** of:
1. Permission boundary (what is allowed to be allowed)
2. Identity-based policy (what is actually allowed)

```
Permission Boundary:   Allow s3:*, cloudwatch:*, ec2:*
Identity-based Policy: Allow iam:CreateUser

Effective Permission:  NONE
```

`iam:CreateUser` is outside the boundary → the action is denied regardless of the identity-based policy.

---

## Relationship with SCPs (Venn Diagram)

```
        ┌─────────────────────────────────────┐
        │              SCP                     │
        │    ┌──────────────────────────┐      │
        │    │    Permission Boundary   │      │
        │    │   ┌──────────────────┐   │      │
        │    │   │  Identity-based  │   │      │
        │    │   │     Policy       │   │      │
        │    │   │  ← Effective →   │   │      │
        │    │   └──────────────────┘   │      │
        │    └──────────────────────────┘      │
        └─────────────────────────────────────┘
```

All three must allow an action for it to be permitted.

---

## Use Cases

| Use Case | How Permission Boundary Helps |
|---|---|
| Delegate IAM user creation to non-admins | Dev can create users but only within the boundary you set |
| Allow developers to self-assign policies | Prevent privilege escalation — they can't grant themselves more than the boundary allows |
| Restrict one specific user (not the whole account) | More granular than Organizations SCP |
| Test an SCP before applying it org-wide | Use boundary on a single user/role to validate behavior |

---

## SysOps Exam Q&A

**Q: A developer has an IAM policy that allows `iam:CreateUser`. Their permission boundary only allows `s3:*`. Can they create IAM users?**
A: No — the effective permission is the intersection of the boundary and the policy. `iam:CreateUser` is outside the boundary, so it is denied.

**Q: You want to allow a team lead to create IAM users but prevent them from ever granting admin access. What do you use?**
A: Attach a permission boundary to their IAM user/role that excludes admin-level actions — any policy they create or attach cannot exceed the boundary.

**Q: Can permission boundaries be applied to IAM groups?**
A: No — permission boundaries apply only to IAM users and roles, not groups.

**Q: What is the difference between a permission boundary and an SCP?**
A: An SCP restricts an entire AWS account or OU in Organizations. A permission boundary restricts a specific IAM user or role within an account.

---

## Quick Reference

```
Permission Boundary:
  Defines MAXIMUM permissions for an IAM user or role
  Does NOT grant permissions — only limits them
  Effective permission = intersection of boundary + identity-based policy
  Applies to: users and roles (NOT groups)

vs SCP:
  SCP = account/OU level (Organizations)
  Permission Boundary = individual user/role level

Use cases:
  Delegate IAM management without privilege escalation risk
  Restrict specific user without affecting whole account
  Test SCP behavior on a single entity
```
