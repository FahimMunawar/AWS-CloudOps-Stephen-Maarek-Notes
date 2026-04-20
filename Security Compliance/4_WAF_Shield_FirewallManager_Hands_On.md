# 4. WAF, Shield, and Firewall Manager — Hands-On

## Overview

Console walkthrough of creating a WAF Web ACL with rules, reviewing Shield Standard vs Advanced subscription, and exploring Firewall Manager's centralized policy management.

---

## Part 1: AWS WAF — Create a Protection Pack

**Console:** AWS WAF → Create protection pack

### Step 1: Describe Your App

Select what best describes your application (API, web, or both).

### Step 2: Select Resources to Protect

| Resource Type | Notes |
|--------------|-------|
| Amazon CloudFront distributions | Global rule |
| Amazon API Gateway | Regional |
| Application Load Balancer | Regional |
| AppSync APIs | Regional |
| Cognito User Pools | Regional |
| Amplify applications | Global |
| AppRunner services | Regional |

> If no resources exist in the account, skip this step.

### Step 3: Select WebACL Resource Type

| Option | Scope |
|--------|-------|
| CloudFront / Amplify | Global WebACL |
| All other services (ALB, API GW, etc.) | Regional WebACL |

### Step 4: Choose Initial Protections

#### AWS Managed Rules (Recommended — Free)

| Rule | Purpose |
|------|---------|
| Rate limit for GET requests | Throttle high-volume GET traffic |
| Rate limit for POST/PUT/DELETE | Throttle high-volume write traffic |
| Amazon IP reputation list | Block known malicious IPs |
| Anonymous IP protection | Block anonymizing proxies/VPNs |
| Managed IP data protection | AWS-curated threat IP list |
| Bot control | Detect and block bot traffic |

#### Additional Protections (Essentials Pack)

- IP allow list / IP block list
- Geographic restriction

#### Custom Rules (Build Your Own)

| Rule Type | Example |
|-----------|---------|
| IP-based | Block or allow specific IP ranges |
| Geo-based | Block traffic from specific countries |
| Rate-based | Limit requests per IP per time window |
| Custom | Match on HTTP headers, URI, body content |

**Example geo-based rule:**
- Select countries → Action: Block

#### AWS Paid Managed Rules (Examples)

| Rule | Cost |
|------|------|
| Bot Control | ~$10/month + per-request fees |
| Account Takeover Prevention | Per-request pricing |
| Layer 7 attack protection | ~$62–63 per 10M requests |

### Step 5: Name and Create

- WebACL name: e.g., `myWebACL`
- Optional: configure logging
- Click **Create protection pack**

> **Cost:** ~$5/month per WebACL + rule charges. Do not create if not needed.

---

## Part 2: AWS Shield

**Console:** AWS Shield → Overview

| Tier | Cost | Notes |
|------|------|-------|
| **Standard** | Free | Already active — no action needed |
| **Advanced** | ~$3,000/month | Subscribe via "Subscribe to Shield Advanced" button |

Shield Advanced provides:
- Global infrastructure DDoS protection
- 24/7 DDoS Response Team access
- Cost protection for attack-driven scaling
- Automatic WAF rule deployment for Layer 7

> Do not subscribe unless required — $3,000/month commitment.

---

## Part 3: AWS Firewall Manager

**Console:** AWS Firewall Manager → Getting started

- Requires an **admin account** configured within AWS Organizations
- Create a **Firewall Manager policy** — applies firewall rules to all accounts in the org
- **Cost:** ~$100/month per policy

Use cases:
- Enforce WAF rules across all accounts
- Deploy Shield Advanced org-wide
- Standardize Security Groups across all VPCs

---

## Service Summary

| Service | Purpose | Scope |
|---------|---------|-------|
| **WAF** | Layer 7 HTTP filtering per application | Per resource (ALB, CloudFront, etc.) |
| **Shield** | DDoS protection (Layer 3/4/7) | Per account (Standard) or org-wide (Advanced via Firewall Mgr) |
| **Firewall Manager** | Centralized policy management | All accounts in AWS Organization |

---

## Cost Reference

| Service | Cost |
|---------|------|
| WAF WebACL | ~$5/month per WebACL |
| WAF managed rules | Varies — free to ~$63 per 10M requests |
| Shield Standard | Free |
| Shield Advanced | ~$3,000/month per organization |
| Firewall Manager policy | ~$100/month per policy |

---

**File: 4_WAF_Shield_FirewallManager_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
