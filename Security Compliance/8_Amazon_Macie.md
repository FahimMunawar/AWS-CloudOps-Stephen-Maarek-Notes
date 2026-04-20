# 8. Amazon Macie

## Overview

**Amazon Macie** is a fully managed data security service that uses machine learning and pattern matching to discover and protect **sensitive data in S3 buckets** — specifically PII (Personally Identifiable Information). One-click enable, specify buckets, done.

---

## How Macie Works

```
S3 buckets (containing data)
        ↓
Macie (ML + pattern matching)
        ↓
Discovers and classifies sensitive data (PII, credentials, etc.)
        ↓
Findings published to Amazon EventBridge
        ↓
Integrations: SNS (alerts), Lambda (automated response)
```

---

## Key Facts

| Property | Detail |
|----------|--------|
| **Scope** | S3 buckets only |
| **What it finds** | PII — names, addresses, credit card numbers, SSNs, etc. |
| **Method** | Machine learning + pattern matching |
| **Output** | Findings → EventBridge → SNS / Lambda |
| **Setup** | One click — specify target S3 buckets |

---

## Best Practices

✓ **Enable Macie on S3 buckets storing customer data** — ensure PII is discovered before it becomes a compliance issue  
✓ **Route findings to SNS** — alert the security team when sensitive data is detected in unexpected buckets  
✓ **Use Lambda for automated response** — e.g., restrict access or tag the bucket when PII is found  

---

## SysOps Exam Focus

**Q1: "You need to automatically detect if any S3 bucket in your account contains personally identifiable information (PII). Which service should you use?"**
- A) Amazon Inspector
- B) Amazon GuardDuty
- C) Amazon Macie — uses ML and pattern matching to discover PII in S3 buckets
- D) AWS Config with an S3 managed rule
- **Answer: C** — Macie is purpose-built for sensitive data discovery (PII) in S3

**Q2: "How does Macie notify you when it finds sensitive data?"**
- A) It sends an email to the root account
- B) It publishes findings to Amazon EventBridge, which can trigger SNS or Lambda
- C) It writes directly to CloudTrail
- D) It creates a Config rule violation
- **Answer: B** — Macie findings flow through EventBridge, enabling automated notifications and remediation

---

## Service Comparison — Security Services

| Service | Purpose | Data Source |
|---------|---------|-------------|
| **GuardDuty** | Threat detection (malicious activity) | CloudTrail, VPC Flow Logs, DNS |
| **Inspector** | Vulnerability scanning (CVE, network) | EC2, ECR, Lambda |
| **Macie** | Sensitive data discovery (PII) | S3 buckets only |

---

## Quick Reference

```
Amazon Macie:
  Purpose: discover PII and sensitive data in S3
  Method: ML + pattern matching
  Scope: S3 buckets only
  Setup: one click, specify buckets
  Output: findings → EventBridge → SNS / Lambda

vs GuardDuty: threat detection across the account
vs Inspector: vulnerability scanning on EC2/ECR/Lambda
vs Macie:     PII/sensitive data in S3
```

---

**File: 8_Amazon_Macie.md**
**Status: SysOps-focused, exam-ready, concise format**
