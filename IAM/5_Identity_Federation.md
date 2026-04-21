# 5 — Identity Federation

## Overview

Federation allows users **outside of AWS** to assume temporary roles and access AWS resources — no IAM user creation required.

```
User (no AWS account)
  → authenticate with trusted third party
    → receive temporary AWS credentials
      → access AWS Console or API
```

**Key principle:** Identity is managed externally — user management happens outside AWS.

---

## Third-Party Authentication Sources

- LDAP
- Microsoft Active Directory (SAML 2.0 implementation)
- Single Sign-On (SSO)
- OpenID Connect
- Amazon Cognito

---

## Three Federation Types

### 1. SAML 2.0 Federation (Enterprise)

For large enterprises with Microsoft Active Directory or any SAML 2.0-compliant identity provider.

**CLI/API flow:**
```
User
  → authenticates with IDP (e.g., Active Directory)
    → IDP returns SAML assertion (token)
      → call AssumeRoleWithSAML to STS
        → STS trades SAML assertion → temporary security credentials
          → access AWS (S3, EC2, etc.)
```

**Console flow:**
```
Browser
  → accesses IDP portal (web-based)
    → authenticates → IDP returns SAML assertion
      → POST to AWS SSO endpoint
        → SSO talks to STS → validates → redirects to AWS Management Console
```

**Result:** No IAM users needed for employees — they use their existing corporate credentials.

---

### 2. Custom Identity Broker (Enterprise, non-SAML)

Used when the enterprise does **not** have a SAML 2.0-compliant IDP.

- You must **build** the identity broker application
- The broker determines the appropriate IAM policy per user
- The broker has elevated permissions to call STS directly for any policy

```
User/Browser
  → Identity Broker (custom-built)
    → validates against corporate identity store
      → broker calls STS → requests credentials with specific IAM policy
        → STS returns credentials
          → user accesses AWS Console or API
```

**More work than SAML** — policy determination logic must be programmed manually.

---

### 3. Amazon Cognito (Web/Mobile Apps — Public Users)

For apps where end users need to access AWS resources (e.g., upload to S3). Does not require creating IAM users per user.

```
App
  → login to identity provider (Cognito User Pool, Google, Facebook, SAML, OpenID)
    → receive token from IDP
      → present token to Cognito Federated Identity Pool
        → Cognito verifies token with IDP → calls STS
          → returns temporary AWS credentials with pre-defined IAM policy
            → app accesses AWS (S3, DynamoDB, etc.) directly
```

> Web Identity Federation is the older alternative — AWS now recommends Cognito instead. Web Identity Federation is no longer tested on the exam.

---

## Comparison

| Type | Use Case | Custom Code? | IDP Type |
|---|---|---|---|
| SAML 2.0 | Enterprise (corporate users) | No | SAML 2.0 IDP (e.g., Active Directory) |
| Custom Broker | Enterprise (no SAML 2.0) | Yes — build broker | Any corporate identity store |
| Cognito | Web/mobile app (public users) | No | Google, Facebook, Cognito User Pool, OpenID |

---

## SysOps Exam Q&A

**Q: A company uses Microsoft Active Directory for employee authentication. They want employees to access the AWS Console without creating IAM users. What do you use?**
A: SAML 2.0 federation — AD is SAML 2.0 compliant; employees authenticate via AD, receive a SAML assertion, and exchange it for temporary AWS credentials via STS.

**Q: A mobile app needs to let users upload files to S3 using their Google account. What service do you use?**
A: Amazon Cognito with Federated Identity Pools — users authenticate via Google, Cognito trades the token for temporary STS credentials with an IAM policy allowing S3 access.

**Q: An enterprise does not have a SAML 2.0 identity provider. How do they implement federation?**
A: Custom Identity Broker — build an application that validates user identity against the corporate store, then calls STS to get temporary credentials with an appropriate IAM policy.

**Q: What is the key benefit of identity federation?**
A: Users outside AWS get temporary access without needing IAM user accounts — user management stays in the external identity system.

---

## Quick Reference

```
Federation: external users get temporary AWS credentials — no IAM users needed

SAML 2.0 (enterprise):
  User → IDP (AD) → SAML assertion → AssumeRoleWithSAML (STS) → temp credentials
  Console: IDP portal → SAML assertion → AWS SSO endpoint → Management Console

Custom Broker (enterprise, non-SAML):
  Build broker → validate identity → broker calls STS → temp credentials
  More work — broker must determine IAM policy per user

Cognito (web/mobile apps):
  App → IDP (Google/Facebook/Cognito) → token → Cognito Federated Identity Pool
  → STS → temp credentials → access AWS resources
  Replaces Web Identity Federation (deprecated for exam)
```
