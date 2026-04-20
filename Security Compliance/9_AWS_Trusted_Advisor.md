# 9. AWS Trusted Advisor

## Overview

**AWS Trusted Advisor** provides a high-level assessment of your AWS account across six categories, offering automated recommendations without requiring any installation. Free tier available; full checks require Business or Enterprise support.

---

## Six Check Categories

| Category | Examples |
|----------|---------|
| **Cost Optimization** | Underutilized EC2 instances, idle RDS instances, unattached EBS volumes |
| **Performance** | High utilization EC2, CloudFront optimizations |
| **Security** | EBS public snapshots, RDS public snapshots, IAM usage, MFA on root |
| **Fault Tolerance** | Multi-AZ RDS, EC2 across AZs, EBS snapshot age |
| **Service Limits** | Approaching or exceeding AWS service quotas |
| **Operational Excellence** | Best practice checks for operational health |

---

## Support Plan Tiers

| Feature | Free (Basic/Developer) | Business / Enterprise |
|---------|----------------------|----------------------|
| **Checks available** | Core checks only (security + service limits) | **Full set of checks** |
| **Programmatic access** | No | Yes — AWS Support API |
| **CloudWatch integration** | No | Yes — service limit metrics |
| **Organizations aggregation** | No | Yes |

---

## Automation and Integration

| Integration | Use |
|-------------|-----|
| **CloudWatch** | Monitor Trusted Advisor metrics (e.g., service limit usage) |
| **SNS** | Alert administrators when thresholds are exceeded |
| **AWS Security Hub** | Consolidate Trusted Advisor security findings |
| **AWS Config** | Complement config compliance checks |
| **Compute Optimizer** | Right-sizing recommendations |

### Notification Flow

```
Trusted Advisor check → metric threshold hit
        ↓
CloudWatch Alarm
        ↓
SNS topic → email/SMS to admin
```

---

## Multi-Account (AWS Organizations)

Requirements to aggregate Trusted Advisor across all org accounts:

1. AWS Organizations must have **all features enabled**
2. Member accounts must have **Business, Enterprise On-Ramp, or Enterprise** support
3. Enable **Trusted Access with AWS Organizations** in the management account (or delegated admin)

### Architecture

```
Member Account 1 ──► Trusted Advisor findings
Member Account 2 ──► Trusted Advisor findings  ──► Management Account (aggregated view)
Member Account N ──► Trusted Advisor findings
```

---

## Best Practices

✓ **Check Trusted Advisor regularly for service limits** — avoid unexpected throttling as usage grows  
✓ **Set up CloudWatch alarms on service limit metrics** — proactive alerts before limits are hit  
✓ **Use Organizations aggregation for multi-account oversight** — one view for all accounts  
✓ **Upgrade to Business support for full checks** — core checks miss many important recommendations  
✓ **Integrate with Security Hub** — Trusted Advisor security findings appear alongside GuardDuty, Inspector, Macie  

---

## SysOps Exam Focus

**Q1: "You want to receive an automatic alert when your account is approaching an AWS service limit. What is the correct setup?"**
- A) Enable AWS Config with a service limit rule
- B) Use Trusted Advisor service limit checks + CloudWatch alarm + SNS notification
- C) Set up GuardDuty to monitor service usage
- D) Use AWS Budgets with a service limit threshold
- **Answer: B** — Trusted Advisor exposes service limit metrics to CloudWatch; set an alarm and notify via SNS

**Q2: "Which support plan is required to get programmatic access to Trusted Advisor checks via the AWS Support API?"**
- A) Developer support
- B) Basic support (free)
- C) Business or Enterprise support
- D) Any paid support plan
- **Answer: C** — Programmatic access to Trusted Advisor via the Support API requires Business or Enterprise support

**Q3: "You want to aggregate Trusted Advisor findings across all 30 accounts in your AWS Organization. What are the prerequisites?"**
- A) Enable Trusted Advisor in each account individually
- B) AWS Organizations with all features enabled + Business/Enterprise support on member accounts + enable Trusted Access in the management account
- C) AWS Organizations with consolidated billing only
- D) Enable AWS Config aggregator across all accounts
- **Answer: B** — All three conditions are required: full org features, Business/Enterprise support, and Trusted Access enabled in the management account

**Q4: "Which of the following is NOT a Trusted Advisor check category?"**
- A) Cost Optimization
- B) Security
- C) Data Encryption
- D) Service Limits
- **Answer: C** — The six categories are: Cost Optimization, Performance, Security, Fault Tolerance, Service Limits, and Operational Excellence. "Data Encryption" is not a standalone category.

---

## Quick Reference

```
Trusted Advisor:
  No installation — high-level AWS account assessment
  6 categories: Cost Optimization, Performance, Security,
                Fault Tolerance, Service Limits, Operational Excellence

Support tiers:
  Free: core security + service limit checks only
  Business/Enterprise: full checks + Support API + CloudWatch integration

Automation:
  CloudWatch alarm on service limit metric → SNS → admin alert

Integrations: Security Hub, Config, Compute Optimizer

Multi-account aggregation (Organizations):
  Requirements:
    1. All features enabled in AWS Organizations
    2. Business/Enterprise support on member accounts
    3. Trusted Access enabled in management account
```

---

**File: 9_AWS_Trusted_Advisor.md**
**Status: SysOps-focused, exam-ready, concise format**
