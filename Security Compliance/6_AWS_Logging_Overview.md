# 6. AWS Logging Overview

## Overview

AWS provides logging across many services for security, compliance, and audit requirements. All logs can be centralized in S3 and analyzed with Athena — a common exam pattern.

---

## Service Logs Reference

| Service | What It Logs |
|---------|-------------|
| **CloudTrail** | All API calls across AWS services — who did what, when, from where |
| **AWS Config** | Resource configuration changes and compliance state over time |
| **CloudWatch Logs** | Application logs, OS logs, custom metrics — full retention control |
| **VPC Flow Logs** | IP traffic metadata within a VPC (source/dest IP, port, allow/deny) |
| **ELB Access Logs** | Metadata of every request made to a load balancer |
| **CloudFront Logs** | Access logs for CloudFront distributions (requests, status codes, edge location) |
| **WAF Logs** | Full logging of every request analyzed by WAF rules |

---

## Log Analysis Pattern (Exam Critical)

> All logs can be stored in **S3** and analyzed with **Amazon Athena**.

```
Service generates logs
        ↓
Logs stored in S3 bucket
        ↓
Amazon Athena runs SQL queries against S3 logs
        ↓
Insights without needing a running server
```

### Common Exam Scenario

**"EC2 instances were terminated and logs were lost. How do you analyze what happened to your load balancer?"**

→ **ELB Access Logs (in S3) + Amazon Athena**

This pattern applies to all service logs:
- CloudTrail + S3 + Athena
- CloudFront Logs + S3 + Athena
- VPC Flow Logs + S3 + Athena
- WAF Logs + S3 + Athena

---

## Log Storage Best Practices

| Practice | Detail |
|----------|--------|
| **Encrypt logs in S3** | Use SSE-S3 or SSE-KMS on the log bucket |
| **Control access** | IAM policies + S3 bucket policies on the log bucket |
| **MFA Delete** | Enable on the log bucket to prevent accidental deletion |
| **Long-term retention** | Move logs to S3 Glacier for cost savings |
| **Compliance lock** | Use S3 Glacier Vault Lock (WORM) to enforce immutable retention periods (e.g., 7 years) |

---

## Log Retention Decision Tree

```
Need to analyze logs quickly?
  → Keep in S3 Standard + query with Athena

Need long-term retention (compliance)?
  → Move to S3 Glacier (cost saving)

Need tamper-proof immutable logs for compliance?
  → S3 Glacier Vault Lock (no one can delete/modify for defined period)
```

---

## Best Practices

✓ **Always store logs in S3** — provides durability, lifecycle management, and Athena compatibility  
✓ **Use Athena for ad-hoc log analysis** — no servers, pay per query, works with all log types  
✓ **Enable WAF full logging** — captures every request analyzed, useful for security investigations  
✓ **Encrypt the log bucket** — logs contain sensitive metadata and should be protected at rest  
✓ **Apply Glacier Vault Lock for regulated industries** — enforces non-negotiable retention periods  

---

## SysOps Exam Focus

**Q1: "Your EC2 instances were terminated and you lost their application logs. You need to understand traffic patterns to your ALB over the past 30 days. What is the correct approach?"**
- A) Restore the EC2 instances from a snapshot and check their logs
- B) Query ELB Access Logs stored in S3 using Amazon Athena
- C) Check CloudTrail for EC2 API calls
- D) Review CloudWatch metrics for the ALB
- **Answer: B** — ELB Access Logs are written directly to S3 independently of the EC2 instances; Athena can query them with SQL

**Q2: "Which combination of services allows you to run SQL queries against CloudFront access logs?"**
- A) CloudWatch Logs Insights + CloudFront
- B) CloudFront Logs stored in S3 + Amazon Athena
- C) CloudTrail + Amazon QuickSight
- D) VPC Flow Logs + CloudWatch
- **Answer: B** — CloudFront logs to S3; Athena queries S3 data with standard SQL — no server required

**Q3: "You need to retain security audit logs for 7 years and ensure they cannot be deleted or modified by anyone, including administrators. What should you configure?"**
- A) S3 Standard with versioning enabled
- B) S3 Glacier with a Vault Lock policy enforcing a 7-year retention period
- C) CloudWatch Logs with a 7-year retention setting
- D) S3 with MFA Delete enabled
- **Answer: B** — Glacier Vault Lock enforces a WORM (Write Once Read Many) policy; no one can delete or modify objects within the retention window

**Q4: "Which AWS service provides complete logging of every request evaluated by WAF rules?"**
- A) CloudTrail — logs all WAF API calls
- B) WAF full logging — captures every HTTP request analyzed by WAF, stored in S3/Kinesis/CloudWatch
- C) VPC Flow Logs — captures all HTTP traffic
- D) ELB Access Logs — include WAF decisions per request
- **Answer: B** — WAF has its own full request logging feature that records every request evaluated by the Web ACL

---

## Quick Reference

```
AWS Service Logs:
  CloudTrail       → API call audit trail
  Config           → resource config changes + compliance
  CloudWatch Logs  → application/OS logs
  VPC Flow Logs    → IP traffic metadata
  ELB Access Logs  → load balancer request metadata
  CloudFront Logs  → CDN access logs
  WAF Logs         → full request analysis logs

All logs → S3 → Athena (SQL queries, no server needed)

Log security:
  Encrypt: SSE-S3 or SSE-KMS on log bucket
  Access: IAM + bucket policies
  Prevent deletion: MFA Delete
  Long-term: S3 Glacier
  Compliance lock: Glacier Vault Lock (WORM, e.g., 7 years)
```

---

**File: 6_AWS_Logging_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
