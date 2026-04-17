# 10. CloudFront with ALB Sticky Sessions

## Overview

When an Application Load Balancer uses **sticky sessions**, the ALB uses a cookie to route the same user to the same backend EC2 instance. To make sticky sessions work through a CloudFront distribution, you must **whitelist the sticky session cookie** in CloudFront's cache behavior.

---

## The Problem

```
User → CloudFront → ALB (sticky sessions enabled)
```

If CloudFront strips cookies before forwarding to the ALB, the ALB never sees the sticky session cookie and **cannot route the user to the correct EC2 instance**.

---

## The Solution

Whitelist the ALB sticky session cookie in CloudFront's cache behavior so it is passed through to the ALB.

```
User sends request with cookie:
  AWSALB=<session-value>
        ↓
CloudFront (cookie whitelist: AWSALB)
  → Forwards AWSALB cookie to ALB
        ↓
ALB reads AWSALB cookie → routes to same EC2 instance
```

| Cookie Name | Purpose |
|-------------|---------|
| `AWSALB` | Default ALB sticky session cookie |
| `AWSALBCORS` | ALB sticky session cookie for CORS requests |

---

## Configuration

**CloudFront behavior settings:**
- Cookie forwarding: **Whitelist**
- Add `AWSALB` (and `AWSALBCORS` if needed) to the whitelist

This ensures the sticky session cookie reaches the ALB without forwarding all cookies (which would hurt caching).

---

## Security Note

Set the CloudFront TTL to a value **less than the authentication cookie's expiry**. This prevents CloudFront from serving a cached response after a session has expired.

> The exam is unlikely to test this detail — focus on the cookie whitelisting requirement.

---

## SysOps Exam Focus

**Q1: "You have CloudFront in front of an ALB with sticky sessions enabled. Users are being routed to different EC2 instances on each request instead of staying on the same one. What is the cause?"**
- A) The ALB health checks are failing
- B) CloudFront is not forwarding the sticky session cookie to the ALB
- C) The EC2 instances are in different AZs
- D) CloudFront does not support ALB origins
- **Answer: B** — CloudFront strips cookies by default; if `AWSALB` is not whitelisted, the ALB cannot maintain session affinity

**Q2: "How do you fix sticky session behavior when using CloudFront in front of an ALB?"**
- A) Disable caching in CloudFront
- B) Whitelist the AWSALB cookie in the CloudFront cache behavior so it is forwarded to the ALB
- C) Enable Origin Shield
- D) Use a Network Load Balancer instead
- **Answer: B** — Whitelisting the sticky session cookie allows it to pass through CloudFront to the ALB

---

## Quick Reference

```
ALB sticky sessions + CloudFront:
  Problem:   CloudFront strips cookies → ALB loses session context
  Fix:       Whitelist AWSALB cookie in CloudFront cache behavior

Cookie names:
  AWSALB      →  standard sticky session
  AWSALBCORS  →  sticky session for CORS

Config:  Behavior → Cookie forwarding: Whitelist → add AWSALB
```

---

**File: 10_CloudFront_ALB_Sticky_Sessions.md**
**Status: SysOps-focused, exam-ready, concise format**
