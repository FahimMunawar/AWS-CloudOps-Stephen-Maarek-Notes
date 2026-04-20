# 5. Amazon Inspector

## Overview

**Amazon Inspector** runs automated, continuous security assessments on EC2 instances, container images in ECR, and Lambda functions. It identifies software vulnerabilities (CVEs) and network exposure issues, and reports findings to Security Hub and EventBridge.

---

## What Inspector Scans

| Target | How | What It Checks |
|--------|-----|---------------|
| **EC2 instances** | Via SSM Agent | Unintended network accessibility + OS vulnerabilities (CVE) |
| **Container images (ECR)** | On image push | Known vulnerabilities in container packages (CVE) |
| **Lambda functions** | On deployment | Software vulnerabilities in function code and package dependencies |

> Inspector only covers these three targets — not S3, RDS, or other services.

---

## How It Works

```
EC2 / ECR image push / Lambda deployment
        ↓
Amazon Inspector (continuous scanning)
        ↓
Checks against CVE vulnerability database
        ↓
Risk score assigned to each finding (for prioritization)
        ↓
Findings reported to:
  ├── AWS Security Hub (centralized view)
  └── Amazon EventBridge (automation triggers)
```

---

## Scanning Triggers

| Trigger | Action |
|---------|--------|
| EC2 instance running | Continuous scanning via SSM Agent |
| Container image pushed to ECR | Immediate scan on push |
| Lambda function deployed | Scan on deployment |
| CVE database updated | Inspector automatically re-scans all covered resources |

---

## What Inspector Evaluates

| Check Type | Applies To |
|-----------|-----------|
| **Package vulnerabilities (CVE)** | EC2, ECR, Lambda |
| **Network reachability** | EC2 only |

---

## Findings Output

| Destination | Use |
|-------------|-----|
| **AWS Security Hub** | Centralized vulnerability visibility across accounts |
| **Amazon EventBridge** | Trigger automated remediation workflows |

---

## Best Practices

✓ **Ensure SSM Agent is installed on EC2 instances** — Inspector requires it for EC2 assessments  
✓ **Enable Inspector for ECR** — catches vulnerabilities before containers run in production  
✓ **Use EventBridge rules on Inspector findings** — automate remediation (e.g., quarantine instance, notify team)  
✓ **Prioritize by risk score** — Inspector scores each finding; address critical scores first  
✓ **Inspector re-scans on CVE database updates** — no manual trigger needed when new vulnerabilities are published  

---

## SysOps Exam Focus

**Q1: "You want to automatically detect known OS vulnerabilities and network exposure on your EC2 instances continuously. Which service should you use?"**
- A) AWS Config
- B) Amazon Inspector — continuously scans EC2 instances via SSM Agent for CVEs and network reachability
- C) AWS Trusted Advisor
- D) Amazon GuardDuty
- **Answer: B** — Inspector is purpose-built for vulnerability scanning on EC2, ECR, and Lambda

**Q2: "A new CVE is published in the vulnerability database. What does Amazon Inspector do automatically?"**
- A) Nothing — you must manually trigger a new scan
- B) Sends an SNS notification only
- C) Automatically re-scans all covered resources against the updated CVE database
- D) Creates a Security Hub finding without re-scanning
- **Answer: C** — Inspector re-runs assessments automatically when the CVE database is updated

**Q3: "Which AWS services does Amazon Inspector assess? (Choose all that apply)"**
- A) EC2 instances, S3 buckets, RDS databases
- B) EC2 instances, container images in ECR, Lambda functions
- C) Lambda functions, DynamoDB tables, ECS tasks
- D) EC2 instances and ECS containers only
- **Answer: B** — Inspector covers exactly three targets: EC2, ECR container images, and Lambda functions

**Q4: "Where does Amazon Inspector send its findings for centralized visibility and automated response?"**
- A) CloudWatch Logs and S3
- B) AWS Security Hub (centralized view) and Amazon EventBridge (automation)
- C) AWS Config and CloudTrail
- D) Amazon GuardDuty and Macie
- **Answer: B** — Inspector integrates with Security Hub for consolidated findings and EventBridge for automated workflows

---

## Quick Reference

```
Amazon Inspector:
  Purpose: automated security vulnerability scanning
  Targets: EC2 (SSM Agent), ECR container images, Lambda functions
  Scanning: continuous, triggered on resource changes and CVE updates

What it checks:
  Package vulnerabilities (CVE) → EC2, ECR, Lambda
  Network reachability           → EC2 only

Output:
  AWS Security Hub → centralized findings
  Amazon EventBridge → automated remediation triggers
  Risk score → assigned to each finding for prioritization

Re-scans automatically when CVE database is updated
Requires SSM Agent on EC2 for assessments
```

---

**File: 5_Amazon_Inspector.md**
**Status: SysOps-focused, exam-ready, concise format**
