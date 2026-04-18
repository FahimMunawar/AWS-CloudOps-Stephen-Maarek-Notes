# 9 — AWS Control Tower: Hands-On

> **Warning**: Setting up a Landing Zone creates real accounts and resources with ongoing costs (~60 minutes to complete). Do not run this casually.

---

## Landing Zone Setup Steps

1. **Home Region** — where Control Tower is based
2. **Region deny setting** — optionally block specific regions
3. **Additional governance regions** — regions to monitor for compliance
4. **OUs created automatically**:
   - **Security OU** — contains log archive + audit accounts
   - **Sandbox OU** — for other/custom accounts
5. **Shared accounts** — provide unique emails for:
   - Log archive account
   - Audit account
6. **Account access** — use **IAM Identity Center** (SSO) — recommended default
7. **CloudTrail** — enable across the entire Landing Zone
8. **S3 log storage** — optional
9. **KMS encryption** — optional

---

## What Control Tower Creates Automatically

| Resource | Detail |
|---|---|
| **2 OUs** | Security (Core) + Sandbox (Custom) |
| **3 shared accounts** | Management + Log Archive + Audit |
| **IAM Identity Center** | Native cloud directory with SSO access |
| **20 preventive guardrails** | SCPs to enforce policies |
| **2 detective guardrails** | Detect configuration violations |

---

## Guardrail Examples (Auto-Applied)

- Disallow deletion of log archive
- Disallow public read access to log archive
- Disallow configuration changes to CloudTrail

---

## Management Rules

- **Always manage accounts through Control Tower** — not directly through Organizations
- Add new OUs and accounts via **Account Factory** in the Control Tower console
- Users managed via **IAM Identity Center** — single SSO portal URL for access to all accounts

---

## IAM Identity Center (SSO)

After setup, a **user portal URL** is available. Users can:
- Log in once and access any enrolled AWS account
- Switch between accounts (Audit, Log Archive, Management, etc.)
- Get management console access or programmatic CLI credentials per account

---

## Quick Reference

```
Landing Zone setup: ~60 minutes, creates real accounts (costly)

Auto-created:
  OUs: Security (Core) + Sandbox (Custom)
  Accounts: Management + Log Archive + Audit (3 total)
  Guardrails: 20 preventive (SCPs) + 2 detective
  SSO: IAM Identity Center with user portal

Key settings:
  Home Region: Control Tower base
  Region deny: block unapproved regions
  IAM Identity Center: recommended account access method
  CloudTrail: enable org-wide

Management rule: use Control Tower (not Organizations directly) to manage accounts/OUs
```
