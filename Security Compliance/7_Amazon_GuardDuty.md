# 7. Amazon GuardDuty

## Overview

**Amazon GuardDuty** is an intelligent threat detection service that uses machine learning, anomaly detection, and third-party threat intelligence to identify malicious activity in your AWS account. One-click enable, no software to install, 30-day free trial.

---

## How GuardDuty Works

```
Input data sources (automatic + optional)
        ↓
GuardDuty (ML + anomaly detection + threat intel)
        ↓
Findings generated
        ↓
Amazon EventBridge event created
        ↓
Automated response: Lambda (remediation) or SNS (notification)
```

---

## Input Data Sources

### Automatic (Always Analyzed)

| Source | What GuardDuty Looks For |
|--------|--------------------------|
| **CloudTrail event logs** | Unusual API calls, unauthorized deployments |
| **CloudTrail management events** | e.g., CreateVpcSubnet — unusual account activity |
| **CloudTrail S3 data events** | GetObject, ListObjects, DeleteObject — data exfiltration patterns |
| **VPC Flow Logs** | Unusual internet traffic, suspicious IP addresses |
| **DNS Logs** | EC2 instances sending encoded data in DNS queries (sign of compromise) |

### Optional Features (Enable Separately)

| Source | Detects |
|--------|---------|
| **EKS audit logs** | Kubernetes cluster threats |
| **RDS and Aurora login events** | Unusual database login patterns |
| **EBS volumes** | Malware on attached volumes |
| **Lambda network activity** | Suspicious Lambda network behavior |
| **S3 data events** | Additional S3 threat patterns |
| **EKS runtime monitoring** | Container runtime threats |

---

## Findings and Response

- GuardDuty generates **findings** when threats are detected
- Each finding creates an event in **Amazon EventBridge**
- EventBridge rules trigger automated responses:

| Target | Use |
|--------|-----|
| **AWS Lambda** | Automated remediation (e.g., isolate EC2, revoke credentials) |
| **SNS** | Alert security team |

---

## Notable Use Case — Cryptocurrency Attacks

GuardDuty has a **dedicated finding type for cryptocurrency mining** — it analyzes network patterns and API usage to detect EC2 instances being used for crypto mining without authorization.

---

## Best Practices

✓ **Enable GuardDuty in all regions and accounts** — threats can originate from any region  
✓ **Use EventBridge + Lambda for automated remediation** — respond to findings without manual intervention  
✓ **Enable optional features for broader coverage** — especially RDS login events and EKS if you use those services  
✓ **Do not disable GuardDuty** — disabling it generates a finding itself, and you lose continuous monitoring  
✓ **Use GuardDuty findings in Security Hub** — centralize threat visibility across services  

---

## SysOps Exam Focus

**Q1: "You want to detect if an EC2 instance in your account has been compromised and is communicating with a known command-and-control server. Which service provides this detection?"**
- A) AWS Config
- B) Amazon Inspector
- C) Amazon GuardDuty — analyzes VPC Flow Logs and DNS logs for suspicious traffic patterns
- D) AWS CloudTrail
- **Answer: C** — GuardDuty uses VPC Flow Logs and DNS logs to detect communication with malicious IPs and encoded DNS queries from compromised instances

**Q2: "How does GuardDuty notify you when a threat is detected?"**
- A) It sends an email directly to the root account email address
- B) It creates a finding and publishes an event to Amazon EventBridge, which can trigger Lambda or SNS
- C) It automatically opens an AWS Support ticket
- D) It writes a finding to CloudTrail
- **Answer: B** — GuardDuty findings trigger EventBridge events; from there, Lambda or SNS handle response and notification

**Q3: "Your security team suspects EC2 instances are being used for unauthorized cryptocurrency mining. Which service can detect this automatically?"**
- A) AWS Trusted Advisor
- B) Amazon Macie
- C) Amazon GuardDuty — has a dedicated finding type for cryptocurrency attack detection
- D) AWS Config
- **Answer: C** — GuardDuty includes dedicated threat intelligence and findings specifically for cryptocurrency mining activity

**Q4: "Which data sources does GuardDuty analyze by default (without any additional configuration)?"**
- A) VPC Flow Logs, CloudTrail logs, DNS logs
- B) VPC Flow Logs, S3 access logs, EKS audit logs
- C) CloudTrail, RDS login events, Lambda activity
- D) DNS logs and EBS volume scans only
- **Answer: A** — GuardDuty automatically analyzes VPC Flow Logs, CloudTrail event/management logs, and DNS logs; all other sources are optional

---

## Quick Reference

```
GuardDuty:
  Purpose: intelligent threat detection (ML + anomaly detection + threat intel)
  Setup: one click, no software, 30-day free trial

Automatic data sources:
  CloudTrail (management + S3 data events)
  VPC Flow Logs
  DNS Logs

Optional sources:
  EKS audit logs, RDS/Aurora login, EBS, Lambda network, S3 data events

Response flow:
  Finding detected → EventBridge event → Lambda (remediate) / SNS (alert)

Notable: dedicated cryptocurrency mining detection finding

vs Inspector:
  Inspector  = vulnerability scanning (CVE, network exposure) on EC2/ECR/Lambda
  GuardDuty  = threat detection (malicious activity, anomalies) across the account
```

---

**File: 7_Amazon_GuardDuty.md**
**Status: SysOps-focused, exam-ready, concise format**
