# 10. AWS Security Hub

## Overview

**AWS Security Hub** is a centralized security management service that aggregates findings from multiple AWS security services and partner tools into a single dashboard. It provides automated security checks, compliance status visibility, and EventBridge integration for automated response.

---

## Prerequisite

> **AWS Config must be enabled** before Security Hub can function — it relies on Config for resource configuration checks.

---

## What Security Hub Aggregates

| Service / Tool | Findings Type |
|---------------|--------------|
| **Amazon GuardDuty** | Threat detection findings |
| **Amazon Inspector** | Vulnerability assessment findings |
| **Amazon Macie** | Sensitive data (PII) findings |
| **AWS Firewall Manager** | Firewall policy compliance |
| **IAM Access Analyzer** | Overly permissive resource policies |
| **AWS Systems Manager** | Patch compliance, configuration issues |
| **AWS Config** | Resource configuration compliance |
| **AWS Health** | AWS service health events affecting your account |
| **AWS Partner Solutions** | Third-party security tool findings |

---

## Architecture

```
Multiple AWS accounts
        │
GuardDuty ─┐
Inspector  ─┤
Macie      ─┤
Config     ─┼──► AWS Security Hub (central dashboard)
IAM Analyzer─┤         │
Firewall Mgr─┤         ├──► Security Hub findings (dashboard)
Health     ─┘         │
                       ├──► Amazon EventBridge (automated response)
                       │
                       └──► Amazon Detective (root cause investigation)
```

---

## Key Features

| Feature | Detail |
|---------|--------|
| **Central dashboard** | Security and compliance status across all integrated services |
| **Multi-account** | Aggregates findings across multiple AWS accounts |
| **Automated checks** | Runs automated security checks against enabled standards |
| **Security standards** | Choose compliance frameworks to check against (e.g., CIS, PCI DSS, AWS Foundational) |
| **EventBridge integration** | Security findings trigger events for automated remediation |
| **Amazon Detective** | Investigate the root cause and source of security issues |

---

## Amazon Detective (Related Service)

- Used alongside Security Hub to **investigate security findings**
- Quickly identifies where security issues originated
- Uses ML to analyze CloudTrail, VPC Flow Logs, and GuardDuty findings to build a timeline

---

## Pricing

| Item | Cost |
|------|------|
| Security checks | Per check (first 1,000 checks at one rate, then lower) |
| Findings ingestion | First 10,000 events free; per-finding charge after |
| Trial | 30-day free trial |

---

## Multi-Account Management with AWS Organizations

Security Hub integrates with AWS Organizations for centralized management across all member accounts.

| Option | Detail |
|--------|--------|
| **Admin account** | Use the management account directly |
| **Delegated administrator** | Designate another account as the Security Hub admin |
| **Auto-enable** | Security Hub is automatically enabled on all member accounts |
| **Centralized configuration** | Manage all accounts' Security Hub settings from one place |

### Example Use Case

```
Delegated Admin Account (Security Hub administrator)
        │
        ├── Member Account 1 (auto-enabled)
        ├── Member Account 2 (auto-enabled)
        └── Member Account N (auto-enabled)

Run CIS AWS Benchmark → results aggregated in admin account dashboard
```

---

## Enabling Security Hub

1. Enable **AWS Config** (required prerequisite)
2. Console: AWS Security Hub → Enable Security Hub
3. Select **security standards** to enforce (e.g., CIS AWS Foundations, PCI DSS)
4. Review **integrations** (GuardDuty, Inspector, Macie, etc.)
5. Click **Enable Security Hub**

---

## Best Practices

✓ **Enable AWS Config before Security Hub** — it is a hard dependency  
✓ **Enable Security Hub in all regions and accounts** — threats can originate anywhere  
✓ **Use EventBridge rules on Security Hub findings** — automate remediation for critical findings  
✓ **Use Amazon Detective for investigation** — when a finding appears, Detective helps trace its origin  
✓ **Select appropriate security standards** — align with your compliance requirements (CIS, PCI DSS, etc.)  

---

## SysOps Exam Focus

**Q1: "You want a single dashboard that shows security findings from GuardDuty, Inspector, Macie, and Firewall Manager across multiple AWS accounts. Which service provides this?"**
- A) Amazon EventBridge
- B) AWS Security Hub — aggregates findings from all security services into one central dashboard
- C) AWS Config aggregator
- D) Amazon CloudWatch dashboards
- **Answer: B** — Security Hub is designed to aggregate and centralize findings from multiple security services across accounts

**Q2: "What must be enabled before you can use AWS Security Hub?"**
- A) Amazon GuardDuty
- B) AWS CloudTrail
- C) AWS Config — Security Hub depends on it for resource configuration checks
- D) Amazon Inspector
- **Answer: C** — AWS Config is a prerequisite for Security Hub to function

**Q3: "A Security Hub finding indicates a potential breach. You need to investigate the timeline and source of the activity. Which service should you use?"**
- A) AWS CloudTrail — review API call history manually
- B) Amazon Detective — uses ML to analyze findings and build an investigation timeline
- C) AWS Config — shows resource configuration changes
- D) Amazon Inspector — scans for vulnerabilities
- **Answer: B** — Amazon Detective is purpose-built for security investigation; it analyzes CloudTrail, VPC Flow Logs, and GuardDuty data to identify root causes

**Q5: "You want to manage Security Hub centrally across 40 AWS accounts without enabling it manually in each account. What is the correct approach?"**
- A) Enable Security Hub in each account and link them via Config aggregator
- B) Integrate Security Hub with AWS Organizations and designate a delegated administrator — Security Hub is automatically enabled across all member accounts
- C) Use Firewall Manager to push Security Hub settings to all accounts
- D) Use CloudFormation StackSets to deploy Security Hub in each account
- **Answer: B** — Security Hub + AWS Organizations integration auto-enables Security Hub on member accounts and allows centralized management from the admin account

**Q4: "How does Security Hub support automated responses to security findings?"**
- A) It automatically remediates findings using built-in playbooks
- B) It generates events in Amazon EventBridge, which can trigger Lambda functions or SNS notifications
- C) It sends findings directly to AWS Lambda without EventBridge
- D) Automated responses require a separate Firewall Manager policy
- **Answer: B** — Security Hub findings create EventBridge events; EventBridge rules then trigger Lambda, SNS, or other targets for automated response

---

## Quick Reference

```
Security Hub:
  Purpose: centralized security dashboard across services and accounts
  Prerequisite: AWS Config must be enabled

Aggregates from:
  GuardDuty, Inspector, Macie, Firewall Manager,
  IAM Access Analyzer, Systems Manager, Config, Health, Partners

Features:
  Automated checks → compliance status
  Security standards → CIS, PCI DSS, AWS Foundational
  Multi-account aggregation
  EventBridge → automated remediation
  Amazon Detective → root cause investigation

Pricing:
  Per check + per finding (first 10,000 findings free)
  30-day free trial

Enable order:
  1. Enable AWS Config
  2. Enable Security Hub
  3. Select security standards
  4. Review integrations
```

---

**File: 10_AWS_Security_Hub.md**
**Status: SysOps-focused, exam-ready, concise format**
