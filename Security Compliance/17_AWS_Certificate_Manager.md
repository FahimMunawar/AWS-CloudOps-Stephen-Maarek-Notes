# 17 — AWS Certificate Manager (ACM)

## Overview

Provision, manage, and deploy TLS (SSL) certificates for in-flight encryption on AWS services.

| Feature | Detail |
|---|---|
| Public certificates | **Free** + automatic renewal |
| Private certificates | Supported (cost applies) |
| Auto-renewal | 60 days before expiry (ACM-generated only) |
| Cannot use with | **EC2 instances** (public certs cannot be extracted) |

---

## Supported Integrations

- Elastic Load Balancers (CLB, ALB, NLB)
- CloudFront Distributions
- API Gateway

---

## Requesting a Public Certificate

1. List domain names: FQDN (`corp.example.com`) or wildcard (`*.example.com`)
2. Choose validation method:
   - **DNS validation** (preferred) — create a CNAME record in DNS; auto-integrated with Route 53; supports automatic renewal
   - **Email validation** — ACM emails registrar contact addresses
3. Wait a few hours for verification → certificate issued
4. Certificate enrolled for automatic renewal (60 days before expiry)

---

## Imported Certificates

| Property | ACM-Generated | Imported |
|---|---|---|
| Automatic renewal | Yes (60 days before expiry) | **No** |
| Expiry alerts | EventBridge events | EventBridge events (45 days before, configurable) |

### Expiry Monitoring for Imported Certs

**Option 1 — EventBridge:**
```
ACM → daily expiration event (starting 45 days before expiry) → EventBridge
  → Lambda / SNS / SQS
```

**Option 2 — AWS Config:**
```
Config rule: acm-certificate-expiration-check
  → non-compliant certificate → EventBridge → Lambda / SNS / SQS
```

---

## ALB Integration

ALB can redirect HTTP → HTTPS automatically:

```
User (HTTP) → ALB → 301 redirect to HTTPS
User (HTTPS) → ALB (TLS cert from ACM) → Auto Scaling Group (EC2)
```

Set a redirect rule on the ALB listener: HTTP (80) → HTTPS (443).

---

## API Gateway Integration

### Endpoint Types

| Type | Description |
|---|---|
| **Edge-optimized** | Requests routed through CloudFront Edge → API Gateway (one region) |
| **Regional** | Clients in same region as API Gateway |
| **Private** | VPC-only access via interface VPC endpoint + resource policy |

### ACM Certificate Region Rules (Exam Critical)

| Endpoint Type | Certificate Region |
|---|---|
| **Edge-optimized** | Must be in **us-east-1** (CloudFront is global, certs attached to CloudFront) |
| **Regional** | Must be in **same region as API Gateway stage** |

Setup: create a **Custom Domain Name** in API Gateway → attach ACM cert → set CNAME or Alias record in Route 53.

---

## SysOps Exam Q&A

**Q: Can you use ACM to provision TLS certificates directly on EC2 instances?**
A: No — public ACM certificates cannot be extracted and cannot be used on EC2.

**Q: You have an imported certificate in ACM. How do you get notified before it expires?**
A: ACM sends daily EventBridge events starting 45 days before expiry, or use AWS Config rule `acm-certificate-expiration-check` — both can trigger Lambda/SNS/SQS.

**Q: Why is DNS validation preferred over email validation for ACM certificates?**
A: DNS validation supports automatic renewal — ACM can verify domain ownership automatically via the CNAME record without manual intervention.

**Q: You have an Edge-optimized API Gateway. Where must the ACM certificate be created?**
A: us-east-1 — Edge-optimized endpoints use CloudFront, and all CloudFront certificates must be in us-east-1.

**Q: You have a Regional API Gateway in ap-southeast-2. Where must the ACM certificate be?**
A: ap-southeast-2 — same region as the API Gateway stage.

---

## Quick Reference

```
ACM: free public TLS certs, automatic renewal (60 days before expiry)
Cannot use with EC2 (public certs not extractable)

Certificate request:
  DNS validation (preferred) → CNAME in Route 53 → auto-renewal enabled
  Email validation → manual; does not support auto-renewal

Imported certs: no auto-renewal
  Monitor expiry: EventBridge (daily events, 45 days before) or Config rule

ALB: redirect HTTP → HTTPS rule + ACM cert on HTTPS listener

API Gateway ACM cert region:
  Edge-optimized → us-east-1 (CloudFront)
  Regional → same region as API Gateway
```
