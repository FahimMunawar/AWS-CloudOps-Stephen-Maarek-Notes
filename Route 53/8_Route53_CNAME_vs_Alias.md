# 8 — Route 53: CNAME vs Alias Records

## The Problem

AWS resources (ALB, CloudFront, etc.) expose AWS hostnames. You want to map your own domain to them.

---

## CNAME vs Alias

| Feature | CNAME | Alias |
|---|---|---|
| **Maps** | Hostname → any hostname | Hostname → AWS resource |
| **Zone Apex** | NOT allowed (`example.com`) | **Allowed** (`example.com`) |
| **Record type** | CNAME | A or AAAA only |
| **TTL** | You set it | Set automatically by Route 53 (cannot change) |
| **Cost** | Standard Route 53 query cost | **Free** |
| **Health check** | No native support | Yes — native health check capability |
| **AWS-specific** | No | Yes — Route 53 extension to DNS |

---

## Alias Record Targets

Alias records can point to:
- Elastic Load Balancers (ALB, NLB, CLB)
- CloudFront Distributions
- API Gateway
- Elastic Beanstalk environments
- S3 **Websites** (not S3 buckets — must be website-enabled)
- VPC Interface Endpoints
- Global Accelerator
- Route 53 records in the same hosted zone

> **Cannot** create an Alias record targeting an **EC2 DNS name**.

---

## Zone Apex Rule (Exam Critical)

```
mydomain.com (zone apex):
  CNAME → ERROR: "CNAME not permitted at apex of zone"
  Alias → OK (use A or AAAA type + select AWS resource)

www.mydomain.com (non-root):
  CNAME → OK
  Alias → OK
```

---

## Hands-On Summary

| Record | Type | Value | Result |
|---|---|---|---|
| `myapp.stephanetheteacher.com` | CNAME | ALB DNS name | Works — non-root domain |
| `myalias.stephanetheteacher.com` | A (Alias) | ALB (eu-central-1) | Works — free + health check |
| `stephanetheteacher.com` | CNAME | ALB DNS name | **FAILS** — zone apex error |
| `stephanetheteacher.com` | A (Alias) | ALB (eu-central-1) | **Works** — alias allowed at apex |

---

## SysOps Exam Q&A

**Q: You want example.com (the root domain) to point to an ALB. What do you use?**
A: An **Alias record** of type A — CNAME is not allowed at the zone apex.

**Q: What record type cannot be set as an Alias target?**
A: **EC2 DNS names** — you cannot create an Alias pointing to an EC2 instance DNS name.

**Q: What is the TTL of an Alias record?**
A: You **cannot set it** — Route 53 sets it automatically.

**Q: What is the cost of querying an Alias record vs a CNAME?**
A: Alias records are **free**. CNAME queries are charged at standard Route 53 rates.

---

## Quick Reference

```
CNAME:
  Hostname → any hostname
  Non-root only (NOT zone apex)
  Standard query cost, no native health check

Alias (Route 53-specific):
  Hostname → AWS resource
  Works at zone apex (e.g., example.com)
  Always type A or AAAA
  TTL: automatic (cannot set)
  Free + native health check

Alias targets: ELB, CloudFront, API GW, Beanstalk, S3 websites, VPC endpoints, Global Accelerator, same-zone Route 53 records
NOT a valid alias target: EC2 DNS name
```
