# 6 — AWS Security Token Service (STS)

## Overview

STS is the backbone of AWS security — it issues **temporary, limited-privilege credentials** for accessing AWS resources.

| Property | Detail |
|---|---|
| Token validity | Up to 1 hour (must be refreshed on expiry) |
| Access type | Temporary — automatically expire |
| Usage | Issue an API call to STS to receive credentials |

---

## STS API Calls

| API | Use Case |
|---|---|
| **AssumeRole** | Assume a role within the same account or cross-account |
| **AssumeRoleWithSAML** | Users authenticated via SAML exchange assertion for STS credentials |
| **AssumeRoleWithWebIdentity** | Users authenticated via OIDC (Google, Facebook, etc.) — **deprecated; use Cognito instead** |
| **GetSessionToken** | MFA-based authentication — get credentials after MFA login |

---

## AssumeRole Flow

```
IAM User / Principal
  → calls STS AssumeRole (must have iam:AssumeRole permission)
    → STS verifies permissions
      → returns temporary credentials (AccessKey + SecretKey + SessionToken + Expiry)
        → principal uses credentials to act as the assumed role
```

**Setup required:**
1. Define an IAM role with the desired permissions
2. Define the trust policy — which principals can assume this role
3. Call `AssumeRole` → receive temporary credentials

---

## Cross-Account Access with STS

Example: Dev account users need to access an S3 bucket in Prod account.

```
Dev Account                         Prod Account
─────────────────                   ─────────────────────────────
IAM Group (developers)              IAM Role: UpdateApp
  │                                   ├── Permissions: S3 access
  │  1. Grant permission               └── Trust policy: allow Dev account
  │     to AssumeRole UpdateApp
  │
  │  2. Call STS AssumeRole ──────────────────────────────→ STS
  │                                                          │
  │  3. Receive temporary credentials ←────────────────────┘
  │
  └──→ Access S3 bucket in Prod using temp credentials
```

**Key:** The IAM role in Prod must explicitly trust the Dev account in its trust policy — unauthorized groups (e.g., testers) cannot assume it.

---

## SysOps Exam Q&A

**Q: A developer in Account A needs temporary access to an S3 bucket in Account B. What is the mechanism?**
A: Create an IAM role in Account B with S3 permissions and a trust policy allowing Account A. The developer calls `AssumeRole` via STS, gets temporary credentials, and uses them to access the bucket.

**Q: What STS API is used when a user is authenticated via SAML?**
A: `AssumeRoleWithSAML` — the SAML assertion is exchanged for temporary STS credentials.

**Q: What STS API is now deprecated in favor of Cognito?**
A: `AssumeRoleWithWebIdentity` — use Amazon Cognito Federated Identity Pools instead.

**Q: What must be configured before an IAM user can call AssumeRole?**
A: The target IAM role must have a trust policy allowing that principal to assume it, and the user must have `iam:AssumeRole` permission in their own policy.

---

## Quick Reference

```
STS: issues temporary credentials (up to 1 hour, auto-expire)

Key APIs:
  AssumeRole          → same/cross-account role assumption
  AssumeRoleWithSAML  → SAML federation → temp credentials
  AssumeRoleWithWebIdentity → OIDC (deprecated → use Cognito)
  GetSessionToken     → MFA-authenticated credentials

Cross-account access:
  1. Create IAM role in target account (permissions + trust policy allowing source account)
  2. Source account user calls AssumeRole
  3. STS returns temp credentials → access target account resources
```
