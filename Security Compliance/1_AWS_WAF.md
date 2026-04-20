# 1. AWS WAF — Web Application Firewall

## Overview

**AWS WAF** is a Layer 7 (HTTP) web application firewall that protects against common web exploits such as SQL injection and cross-site scripting. It is deployed on specific AWS services and controlled via **Web ACLs** with customizable rules.

---

## Supported Deployment Targets

WAF can only be deployed on:

| Target | Notes |
|--------|-------|
| **Application Load Balancer (ALB)** | Regional |
| **API Gateway** | Regional |
| **CloudFront** | Global |
| **AppSync GraphQL API** | Regional |
| **Cognito User Pools** | Regional |

> **Cannot** be deployed on Network Load Balancer (NLB) — NLB operates at Layer 4 (TCP/UDP); WAF is Layer 7 only.

---

## Web ACL Rules

Web ACLs (Access Control Lists) define what to allow, block, or count.

| Rule Type | Description |
|-----------|-------------|
| **IP set** | Filter by IP address — up to 10,000 IPs per set; use multiple rules for more |
| **HTTP headers / body** | Inspect request content |
| **URI strings** | Match specific URL paths |
| **SQL injection protection** | Detect and block SQLi patterns |
| **Cross-site scripting (XSS)** | Detect and block XSS patterns |
| **Size constraints** | Block requests exceeding a size limit (e.g., > 2 MB) |
| **Geo match** | Allow or block traffic from specific countries |
| **Rate-based rules** | Limit requests per IP per time window — DDoS protection |

---

## Web ACL Scope

| Deployment | Scope |
|-----------|-------|
| ALB, API Gateway, AppSync, Cognito | **Regional** |
| CloudFront | **Global** |

---

## Rule Groups

A **rule group** is a reusable collection of rules that can be added to multiple Web ACLs. Used to organize and share common rule sets across applications.

---

## Fixed IP + WAF Architecture

**Problem:** WAF requires ALB (Layer 7), but ALB has no fixed IP. Some clients require whitelisting fixed IPs.

**Solution:** Use Global Accelerator in front of the ALB.

```
Client (needs fixed IP to whitelist)
        ↓
Global Accelerator (provides 2 static Anycast IPs)
        ↓
ALB (in a single region)  ←── Web ACL (WAF) attached here
        ↓
EC2 instances
```

- Global Accelerator provides **fixed static IPs**
- WAF Web ACL is attached to the **ALB** in the same region
- Both requirements (fixed IP + Layer 7 protection) are satisfied

---

## Layer Comparison (Exam Context)

| Layer | Protocol | Services |
|-------|----------|---------|
| **Layer 4** | TCP / UDP | NLB — WAF **cannot** be used here |
| **Layer 7** | HTTP / HTTPS | ALB, API Gateway, CloudFront, AppSync, Cognito — WAF applies here |

---

## Best Practices

✓ **Know the WAF deployment targets** — the exam commonly tests NLB as a distractor (WAF does not support NLB)  
✓ **Use rate-based rules for DDoS mitigation** — limit requests per IP per second  
✓ **Use geo match rules** — block traffic from countries where your app has no users  
✓ **Use rule groups for shared rule sets** — apply the same rules to multiple Web ACLs  
✓ **Use Global Accelerator + ALB + WAF** when fixed IPs and Layer 7 filtering are both required  

---

## SysOps Exam Focus

**Q1: "You need to protect your application from SQL injection and block traffic from specific countries. Which service should you use and where can it be deployed?"**
- A) AWS Shield Advanced on the NLB
- B) AWS WAF Web ACL deployed on the ALB, API Gateway, CloudFront, AppSync, or Cognito
- C) VPC Security Groups with SQL injection rules
- D) AWS WAF on the NLB for Layer 4 protection
- **Answer: B** — WAF operates at Layer 7 and supports SQL injection protection and geo match rules on supported targets

**Q2: "A client requires your application to have a fixed IP address for whitelisting. Your application uses an ALB with WAF. How do you provide a fixed IP?"**
- A) Assign an Elastic IP directly to the ALB
- B) Use Global Accelerator in front of the ALB — it provides 2 static Anycast IPs while WAF remains on the ALB
- C) Move WAF to an NLB which supports Elastic IPs
- D) Use Route 53 with a static A record
- **Answer: B** — ALBs do not support fixed IPs; Global Accelerator provides 2 static IPs while the WAF Web ACL stays on the ALB

**Q3: "You want to prevent a single IP address from sending more than 100 requests per minute to your API Gateway. Which WAF rule type should you use?"**
- A) IP set rule
- B) Size constraint rule
- C) Rate-based rule — limits requests per IP within a time window
- D) Geo match rule
- **Answer: C** — Rate-based rules count requests per IP and block or throttle IPs that exceed the defined threshold

**Q4: "Where are WAF Web ACLs scoped regionally vs globally?"**
- A) All Web ACLs are global
- B) Web ACLs for ALB, API Gateway, AppSync, and Cognito are regional; Web ACLs for CloudFront are global
- C) Web ACLs for CloudFront are regional; all others are global
- D) WAF scope depends on the VPC configuration
- **Answer: B** — CloudFront is a global service so its WAF Web ACLs are defined globally; all other supported targets use regional Web ACLs

---

## Quick Reference

```
AWS WAF:
  Layer: 7 (HTTP) — NOT Layer 4 (TCP/UDP)
  Cannot deploy on: NLB

Supported targets:
  ALB, API Gateway, AppSync, Cognito → Regional Web ACL
  CloudFront                         → Global Web ACL

Web ACL rule types:
  IP set          → filter up to 10,000 IPs per set
  HTTP headers/body/URI → inspect request content
  SQL injection   → detect SQLi
  XSS             → detect cross-site scripting
  Size constraint → block oversized requests
  Geo match       → allow/block by country
  Rate-based      → requests per IP per time window (DDoS protection)

Rule groups → reusable rule collections applied to multiple Web ACLs

Fixed IP + WAF pattern:
  Global Accelerator (fixed IPs) → ALB (WAF Web ACL) → EC2
```

---

**File: 1_AWS_WAF.md**
**Status: SysOps-focused, exam-ready, concise format**
