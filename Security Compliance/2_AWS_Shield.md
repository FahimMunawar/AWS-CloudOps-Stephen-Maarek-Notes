# 2. AWS Shield

## Overview

**AWS Shield** protects AWS infrastructure and applications from **DDoS (Distributed Denial of Service)** attacks. It comes in two tiers: a free standard version active for all AWS customers, and an advanced paid version with enhanced protection and incident response.

---

## What Is a DDoS Attack?

```
Attackers (many computers worldwide)
        ↓
Flood your infrastructure with massive request volume
        ↓
Infrastructure overwhelmed → cannot serve real users
= Distributed Denial of Service
```

---

## Shield Standard vs Shield Advanced

| Feature | Shield Standard | Shield Advanced |
|---------|----------------|-----------------|
| **Cost** | Free | ~$3,000/month per organization |
| **Activation** | Automatic — all AWS customers | Opt-in |
| **Attack types** | Layer 3/4: SYN floods, UDP floods, reflection attacks | Sophisticated Layer 3/4/7 attacks |
| **Protected services** | All AWS services | EC2, ELB, CloudFront, Global Accelerator, Route 53 |
| **DDoS Response Team** | No | **24/7 access** to AWS DDoS Response Team (DRT) |
| **Cost protection** | No | Yes — waives higher AWS fees caused by the attack |
| **Automatic WAF rules** | No | Yes — auto-creates WAF rules to mitigate Layer 7 attacks |

---

## Shield Advanced — Key Details

### Protected Services

- Amazon EC2
- Elastic Load Balancing (ELB)
- Amazon CloudFront
- AWS Global Accelerator
- Amazon Route 53

### Automatic Layer 7 Mitigation

Shield Advanced integrates with AWS WAF to automatically:

```
DDoS attack detected at Layer 7
        ↓
Shield Advanced evaluates attack patterns
        ↓
Automatically creates and deploys WAF rules
        ↓
Attack traffic blocked at the edge
```

### Cost Protection

If a DDoS attack causes AWS resource scaling (e.g., EC2 Auto Scaling, data transfer spikes), Shield Advanced absorbs those additional costs — you are not billed for attack-driven usage.

---

## Layer Reference

| Layer | Protocol | Shield Coverage |
|-------|----------|----------------|
| Layer 3 | Network (IP) | Standard + Advanced |
| Layer 4 | Transport (TCP/UDP) | Standard + Advanced |
| Layer 7 | Application (HTTP) | Advanced only (via auto WAF rules) |

---

## Best Practices

✓ **Shield Standard is always on** — no configuration needed; protects against common Layer 3/4 attacks  
✓ **Use Shield Advanced for business-critical applications** — get DRT access, cost protection, and Layer 7 auto-mitigation  
✓ **Combine Shield Advanced with WAF** — Shield Advanced auto-deploys WAF rules during Layer 7 attacks  
✓ **Shield Advanced covers Route 53** — DNS-layer DDoS protection for domain availability  
✓ **Cost protection is a key exam differentiator** — Standard does not protect against attack-driven billing spikes  

---

## SysOps Exam Focus

**Q1: "Your public-facing application is experiencing a SYN flood attack. Which AWS service is already protecting you by default, at no cost?"**
- A) AWS WAF
- B) AWS Shield Standard — automatically active for all AWS customers, protects against Layer 3/4 attacks including SYN floods
- C) AWS Shield Advanced
- D) Amazon GuardDuty
- **Answer: B** — Shield Standard is free and active by default; SYN floods are Layer 4 attacks covered by Standard

**Q2: "During a DDoS attack, your Auto Scaling group scales out significantly, resulting in a large AWS bill. Which service protects you from these extra charges?"**
- A) AWS Shield Standard
- B) AWS Trusted Advisor cost optimization
- C) AWS Shield Advanced — includes cost protection for attack-driven resource scaling
- D) AWS Budgets with an alert
- **Answer: C** — Shield Advanced absorbs additional AWS costs incurred due to a DDoS attack

**Q3: "You need 24/7 access to AWS security experts during a DDoS incident. What must you subscribe to?"**
- A) AWS Shield Standard (free tier)
- B) AWS WAF with managed rules
- C) AWS Shield Advanced — includes access to the AWS DDoS Response Team (DRT)
- D) AWS Security Hub
- **Answer: C** — The DDoS Response Team (DRT) is only available to Shield Advanced subscribers

**Q4: "How does Shield Advanced protect against Layer 7 (HTTP) DDoS attacks?"**
- A) It blocks all HTTP traffic above a fixed request threshold
- B) It automatically creates and deploys WAF rules to detect and block Layer 7 attack patterns
- C) It requires manual rule creation in WAF for each attack
- D) Layer 7 protection is not available in Shield Advanced
- **Answer: B** — Shield Advanced integrates with WAF to automatically evaluate attack patterns and deploy mitigating rules at Layer 7

---

## Quick Reference

```
Shield Standard:
  Free, automatic, all AWS customers
  Protects: Layer 3/4 (SYN flood, UDP flood, reflection attacks)

Shield Advanced:
  ~$3,000/month per organization, opt-in
  Protects: Layer 3/4/7 on EC2, ELB, CloudFront, Global Accelerator, Route 53
  Extras:
    24/7 DDoS Response Team (DRT) access
    Cost protection (no extra billing from attack-driven scaling)
    Auto WAF rule deployment for Layer 7 mitigation

DDoS = flood infrastructure from many sources → deny service to real users
Layer 3/4 → Shield Standard covers
Layer 7   → Shield Advanced + WAF integration
```

---

**File: 2_AWS_Shield.md**
**Status: SysOps-focused, exam-ready, concise format**
