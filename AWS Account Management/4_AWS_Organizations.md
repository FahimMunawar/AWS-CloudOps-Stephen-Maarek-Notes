# 4 — AWS Organizations

## Overview

- **Global service** — manage multiple AWS accounts from a single place
- **Management account**: the main account that created the organization
- **Member accounts**: all other accounts (can only belong to one organization)

---

## Key Benefits

| Benefit | Detail |
|---|---|
| **Consolidated billing** | Single payment method on management account; pays for all accounts |
| **Aggregated usage discounts** | EC2, S3, etc. usage summed across all accounts → volume pricing tiers |
| **Reserved Instance sharing** | Unused RIs in one account benefit other accounts |
| **Savings Plans sharing** | Discounts shared across the organization |
| **Automated account creation** | API to create accounts programmatically |
| **Centralized CloudTrail** | Enable once, send logs from all accounts to a central S3 account |
| **Centralized CloudWatch logs** | Send logs from all accounts to a central logging account |
| **Cross-account admin roles** | Administer all member accounts from the management account |

---

## Organizational Units (OUs)

```
Root OU
  └── Management Account
  └── Dev OU
        └── Member Account
        └── Member Account
  └── Prod OU
        └── HR OU
              └── Member Account
        └── Finance OU
              └── Member Account
```

### OU Organization Strategies

| Strategy | Example OUs |
|---|---|
| **Business unit** | Sales, Retail, Finance |
| **Environment** | Prod, Test, Dev |
| **Project-based** | Project A, Project B, Project C |

Mix and match freely — you define the structure.

---

## Security Advantage: Service Control Policies (SCPs)

- IAM policies applied to **OUs or individual accounts**
- Restrict what **all users and roles** within that scope can do
- **Management account is ALWAYS exempt** — SCPs never apply to it (safety mechanism)
- Explicit **allows** must be present all the way down the OU hierarchy for an action to be permitted

### SCP Inheritance Rule

```
For an action to be allowed on an account:
  Root OU must allow it
  + every parent OU must allow it
  + the account itself must not deny it
```

### SCP Example: OU Hierarchy

```
Root OU: FullAWSAccess (allow all)
  │
  ├── Management Account
  │     └── SCPs NEVER apply (always full admin)
  │
  ├── Sandbox OU: FullAWSAccess + Deny S3
  │     ├── Account A: FullAWSAccess + Deny EC2
  │     │     → Can do anything EXCEPT S3 (Sandbox deny) and EC2 (Account deny)
  │     ├── Account B: (no extra SCP)
  │     │     → Can do anything EXCEPT S3 (Sandbox deny)
  │     └── Account C: (no extra SCP)
  │           → Can do anything EXCEPT S3 (Sandbox deny)
  │
  └── Workloads OU: FullAWSAccess
        ├── Test OU: Allow EC2 only
        │     └── Account D → EC2 only
        └── Prod OU: FullAWSAccess
              ├── Account E → anything
              └── Account F → anything
```

### SCP Policy Examples

**Blocklist approach** — allow all, then deny specific services:
```json
{
  "Effect": "Allow", "Action": "*", "Resource": "*"
},
{
  "Effect": "Deny", "Action": "dynamodb:*", "Resource": "*"
}
```

**Allowlist approach** — permit only specific services:
```json
{
  "Effect": "Allow",
  "Action": ["ec2:*", "cloudwatch:*"],
  "Resource": "*"
}
```

---

## SysOps Exam Q&A

**Q: Can an SCP restrict the management account?**
A: **No** — the management account always has full admin power; SCPs never apply to it.

**Q: An account in an OU has FullAWSAccess but the parent OU has Deny S3. Can the account use S3?**
A: **No** — explicit denies at any level in the hierarchy override allows. The account cannot use S3.

**Q: What is the difference between multiple accounts and multiple VPCs for security?**
A: **Accounts** provide stronger isolation than VPCs — accounts are fully separate security boundaries.

**Q: How do you share Reserved Instance discounts across accounts in an organization?**
A: Enable **RI sharing** in the organization's billing settings — unused RIs in one account automatically benefit other accounts.

**Q: How do you centralize CloudTrail logs for all accounts?**
A: Create an **Organization Trail** — enables CloudTrail in all accounts and sends logs to a central S3 bucket.

---

## Quick Reference

```
Organizations: global service, multi-account management

Accounts:
  Management account: 1 per org, full admin, SCPs never apply
  Member accounts: created or invited, one org only

OUs: nest hierarchically (Root → OU → sub-OU → account)
  Strategies: business unit / environment / project

Billing: consolidated, single payment method
  Discounts: aggregated usage + RI sharing + Savings Plans sharing

SCPs:
  Applied to OU or account (not management account)
  Explicit ALLOW required at every level in the hierarchy
  Deny at any level = denied (overrides allows below)
  Blocklist: allow all + deny specific
  Allowlist: allow only specific services

Centralization:
  CloudTrail  → Organization Trail → central S3
  CloudWatch  → central logging account
  Admin roles → cross-account roles from management account
```
