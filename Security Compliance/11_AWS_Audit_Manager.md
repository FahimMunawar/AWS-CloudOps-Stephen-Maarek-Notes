# 11. AWS Audit Manager

## Overview

**AWS Audit Manager** continuously audits your AWS usage to help you assess risk, maintain compliance, and prepare evidence for formal audits. It automates evidence collection across your resources against selected compliance frameworks.

---

## Supported Compliance Frameworks

| Framework | Description |
|-----------|-------------|
| **GDPR** | EU data protection regulation |
| **HIPAA** | US healthcare data privacy |
| **PCI DSS** | Payment card industry security standard |
| **SOC 2** | Security, availability, and confidentiality controls |
| And others | Custom frameworks also supported |

---

## How Audit Manager Works

```
1. Select compliance framework (GDPR, HIPAA, PCI, SOC 2, etc.)
        ↓
2. Define scope (accounts, regions, services)
        ↓
3. Automated Evidence Collection (continuous)
        ↓
4. Control Review / Delegate to auditors
        ↓
5. Identify non-compliant resources → root cause analysis
        ↓
6. Generate audit-ready reports + evidence folders
```

---

## Key Features

| Feature | Detail |
|---------|--------|
| **Continuous auditing** | Automatically collects evidence as your environment changes |
| **Framework templates** | Pre-built for GDPR, HIPAA, PCI DSS, SOC 2, and more |
| **Evidence folders** | Organized collection of compliance evidence per control |
| **Delegation** | Assign controls to resource owners for validation |
| **Audit-ready reports** | Generated reports with evidence, ready for submission |
| **Multi-account scope** | Can cover multiple AWS accounts and regions |

---

## Best Practices

✓ **Enable Audit Manager before an audit cycle begins** — continuous evidence collection requires time to build  
✓ **Define scope carefully** — include all accounts and regions where regulated workloads run  
✓ **Delegate controls to resource owners** — distribute the review burden across teams  
✓ **Address non-compliant findings immediately** — Audit Manager surfaces gaps continuously, not just at audit time  
✓ **Use alongside Security Hub and Config** — Audit Manager complements them for a complete compliance picture  

---

## SysOps Exam Focus

**Q1: "Your organization is preparing for a PCI DSS audit and needs to continuously collect evidence of compliance across all AWS accounts. Which service automates this?"**
- A) AWS Config with compliance rules
- B) AWS Audit Manager — selects the PCI DSS framework and continuously collects evidence across accounts
- C) AWS Security Hub with the PCI DSS standard enabled
- D) AWS CloudTrail with S3 log archiving
- **Answer: B** — Audit Manager is purpose-built for audit preparation; it automates evidence collection against specific compliance frameworks

**Q2: "What is the primary output of AWS Audit Manager?"**
- A) Real-time threat alerts sent to EventBridge
- B) Audit-ready reports and evidence folders organized by compliance control
- C) Automated remediation of non-compliant resources
- D) Security findings aggregated into a central dashboard
- **Answer: B** — Audit Manager produces structured evidence folders and reports ready to submit to auditors

---

## Service Comparison — Compliance Tools

| Service | Primary Use |
|---------|------------|
| **AWS Config** | Track resource configuration changes and compliance rules |
| **Security Hub** | Centralize security findings across services |
| **Audit Manager** | Prepare audit evidence for compliance frameworks (GDPR, HIPAA, PCI, SOC 2) |
| **Trusted Advisor** | Best practice recommendations across cost, security, performance |

---

## Quick Reference

```
AWS Audit Manager:
  Purpose: continuous audit evidence collection for compliance frameworks
  Frameworks: GDPR, HIPAA, PCI DSS, SOC 2, custom

Workflow:
  1. Select framework
  2. Define scope (accounts + regions)
  3. Automated evidence collection (continuous)
  4. Control review / delegate to owners
  5. Root cause analysis for gaps
  6. Generate audit-ready reports

Output: evidence folders + compliance reports → ready for auditors
```

---

**File: 11_AWS_Audit_Manager.md**
**Status: SysOps-focused, exam-ready, concise format**
