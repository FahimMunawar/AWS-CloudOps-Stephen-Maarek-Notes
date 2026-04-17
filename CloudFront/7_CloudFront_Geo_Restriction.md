# 7. CloudFront Geo Restriction

## Overview

**Geo Restriction** lets you control which countries can or cannot access your CloudFront distribution. Commonly used to comply with copyright laws or regional content licensing requirements.

---

## How It Works

| Mode | Behavior |
|------|----------|
| **Allow list** | Only users from the specified countries can access the distribution |
| **Block list** | Users from specified countries are denied access |

- Country is determined by matching the user's IP against a **third-party Geo IP database**
- Applies at the **distribution level** — affects all origins and behaviors in that distribution

---

## Configuration

**Console:** CloudFront → Distribution → **Security** → **Geographic restrictions** → Edit

- Select **Allow list** or **Block list**
- Add countries to the list
- Save changes

> **Plan requirement:** Geographic restrictions require the **Pay as You Go** plan or a paid plan. The free CloudFront plan does not include this feature.

---

## Use Cases

- **Copyright compliance** — block access from countries where you don't have distribution rights
- **Regulatory compliance** — restrict content to specific regions (e.g., GDPR, data sovereignty)
- **Licensing agreements** — limit content to countries covered by a content license

---

## SysOps Exam Focus

**Q1: "You need to prevent users in specific countries from accessing your CloudFront distribution. What feature should you use?"**
- A) CloudFront signed URLs
- B) CloudFront Geo Restriction with a block list
- C) AWS WAF IP-based rules
- D) S3 bucket policy with IP conditions
- **Answer: B** — Geo Restriction block list denies access to users from specified countries

**Q2: "How does CloudFront determine which country a user is from for geo restriction purposes?"**
- A) The user's account billing address
- B) The user's browser language settings
- C) The user's IP address matched against a third-party Geo IP database
- D) The DNS server the user queries
- **Answer: C** — CloudFront uses a Geo IP database to map IP addresses to countries

**Q3: "You want only users from France and Germany to access your CloudFront distribution. What is the correct configuration?"**
- A) Create a block list with all countries except France and Germany
- B) Create an allow list with France and Germany
- C) Create signed URLs restricted to French and German IPs
- D) Use an AWS WAF rule with geographic match conditions
- **Answer: B** — An allow list permits only the specified countries; all others are denied

---

## Quick Reference

```
Geo Restriction:
  Allow list  →  only listed countries can access
  Block list  →  listed countries are denied access

IP → country mapping: third-party Geo IP database
Use cases: copyright, regulatory compliance, content licensing
Plan requirement: Pay as You Go or paid plan (not free plan)
```

---

**File: 7_CloudFront_Geo_Restriction.md**
**Status: SysOps-focused, exam-ready, concise format**
