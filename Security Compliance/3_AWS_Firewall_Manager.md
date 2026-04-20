# 3. AWS Firewall Manager

## Overview

**AWS Firewall Manager** is a centralized service to manage firewall rules across **all accounts in an AWS Organization**. Policies are defined once at the region level and automatically applied to existing and newly created resources.

---

## What Firewall Manager Manages

| Policy Type | Applied To |
|-------------|-----------|
| **WAF rules** | ALB, API Gateway, CloudFront |
| **Shield Advanced** | ALB, CLB, NLB, Elastic IP, CloudFront |
| **Security Groups** | EC2, ALB, ENIs in VPC |
| **AWS Network Firewall** | VPC level |
| **Route 53 Resolver DNS Firewall** | DNS-level filtering |

---

## Key Features

| Feature | Detail |
|---------|--------|
| **Scope** | All accounts in an AWS Organization |
| **Policy level** | Regional — policies created per region |
| **Auto-apply** | New resources (e.g., a new ALB) automatically get existing policies applied |
| **Centralized management** | One place to manage WAF, Shield Advanced, Security Groups, Network Firewall |

### Auto-Apply Example

```
Firewall Manager policy: "Apply WAF Web ACL X to all ALBs in the org"
        ↓
New ALB created in any account in the org
        ↓
Firewall Manager automatically applies Web ACL X to the new ALB
```

---

## WAF vs Shield vs Firewall Manager

| Service | Primary Use | When to Use |
|---------|------------|-------------|
| **WAF** | Layer 7 HTTP filtering (SQLi, XSS, IP blocking, rate limiting) | One-time or single-account WAF protection |
| **Shield Advanced** | DDoS protection with SRT access, cost protection, auto WAF rules | High-risk applications frequently targeted by DDoS |
| **Firewall Manager** | Centralized management of WAF, Shield, Security Groups across the org | Multi-account environments; enforce consistent rules everywhere |

### How They Work Together

```
WAF           → define Web ACL rules (the firewall logic)
Shield Adv.   → DDoS protection + auto WAF rule deployment
Firewall Mgr  → deploy WAF rules + Shield Advanced across all org accounts automatically
```

> Use all three together for comprehensive, organization-wide protection.

---

## Best Practices

✓ **Use Firewall Manager when you have multiple AWS accounts** — enforces consistent security policies org-wide  
✓ **Enable auto-remediation** — new resources get policies applied without manual intervention  
✓ **Combine with AWS Organizations** — Firewall Manager requires AWS Organizations to be enabled  
✓ **Use for Security Group standardization** — prevent overly permissive security groups across all accounts  
✓ **Deploy Shield Advanced via Firewall Manager** — one-click protection across all accounts  

---

## SysOps Exam Focus

**Q1: "Your organization has 50 AWS accounts. You want to ensure every ALB across all accounts has the same WAF Web ACL applied, including any ALBs created in the future. What service should you use?"**
- A) Apply the WAF Web ACL manually in each account
- B) AWS Firewall Manager — define a WAF policy once and it applies to all accounts and new resources automatically
- C) AWS Config rule to detect non-compliant ALBs
- D) AWS Organizations SCP to restrict ALB creation without WAF
- **Answer: B** — Firewall Manager is designed exactly for this: org-wide WAF enforcement with automatic application to new resources

**Q2: "What is the key difference between using WAF directly and using WAF through Firewall Manager?"**
- A) WAF supports more rule types when deployed through Firewall Manager
- B) WAF directly is for single-account, one-time protection; Firewall Manager automates WAF deployment across multiple accounts and auto-applies rules to new resources
- C) Firewall Manager replaces WAF entirely and does not use Web ACLs
- D) There is no difference — Firewall Manager is just a UI wrapper for WAF
- **Answer: B** — WAF is the rule engine; Firewall Manager is the multi-account management and automation layer on top of it

**Q3: "Which services can AWS Firewall Manager manage? (Choose the best answer)"**
- A) WAF rules only
- B) WAF, Shield Advanced, Security Groups, AWS Network Firewall, Route 53 Resolver DNS Firewall
- C) WAF and Security Groups only
- D) Shield Advanced and NACLs
- **Answer: B** — Firewall Manager centrally manages WAF, Shield Advanced, Security Groups, Network Firewall, and Route 53 DNS Firewall

---

## Quick Reference

```
Firewall Manager:
  Manages: WAF, Shield Advanced, Security Groups, Network Firewall, DNS Firewall
  Scope: All accounts in AWS Organization, regional policies
  Auto-apply: new resources in any account get policies automatically

WAF vs Shield vs Firewall Manager:
  WAF              → HTTP filtering rules (Layer 7)
  Shield Advanced  → DDoS protection + auto WAF + DRT access
  Firewall Manager → org-wide deployment and enforcement of all the above

Requires: AWS Organizations
Use when: multi-account, need consistent firewall rules, prevent rule gaps on new resources
```

---

**File: 3_AWS_Firewall_Manager.md**
**Status: SysOps-focused, exam-ready, concise format**
