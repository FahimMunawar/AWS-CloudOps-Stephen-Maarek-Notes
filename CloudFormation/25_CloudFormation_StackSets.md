# 25. CloudFormation StackSets

## Overview

**What are StackSets?**

StackSets allow you to deploy CloudFormation stacks across **multiple AWS accounts and regions** in a single operation. A StackSet is managed from one administrator account; the deployed copies in target accounts are called **stack instances**.

**Key Concepts:**

| Term | Definition |
|------|-----------|
| **StackSet** | The template + configuration managed in the administrator account |
| **Stack Instance** | A deployed copy of the StackSet in a specific account + region |
| **Administrator Account** | The account that creates and manages the StackSet |
| **Target Account** | The account(s) where stack instances are deployed |

**When you update a StackSet** in the administrator account, all stack instances across all target accounts and regions are updated automatically.

---

## Permission Models

Two models exist depending on whether you use AWS Organizations:

### 1. Self-Managed Permissions (without Organizations)

You manually create IAM roles in both administrator and target accounts with trust relationships between them.

```
Administrator Account
  └─ AWSCloudFormationStackSetAdministrationRole
       └─ Trusts → AWSCloudFormationStackSetExecutionRole (in each target account)

Target Account (each one)
  └─ AWSCloudFormationStackSetExecutionRole
       └─ Trusted by → Administrator account's administration role
```

- **You** must create these roles manually in every account
- More operational overhead
- Required when accounts are not part of an AWS Organization

---

### 2. Service-Managed Permissions (with AWS Organizations)

AWS Organizations automatically creates the required IAM roles on your behalf.

```
Management Account  ──OR──  Delegated Administrator Account
        │
        ▼
   StackSet deployed to target member accounts
        │
        ├─ Prod OU → Account A, Account B
        └─ Dev OU  → Account C, Account D
```

**Requirements:**
- Must enable **Trusted Access** in AWS Organizations
- Must have **All Features** enabled in the Organization
- No manual IAM role creation needed

**Advantages over self-managed:**
- Automatically deploy to **new accounts** added to the Organization
- **Delegate administration** to specific member accounts (delegated administrator)
- Scales to any number of accounts without manual setup

---

## Automatic Deployment to New Accounts

With service-managed permissions, you can configure StackSets to automatically deploy stack instances to new accounts when they join an OU:

```
New account joins Prod OU
        ↓
StackSet detects new account
        ↓
Stack instance automatically deployed to new account
        ↓
Account is instantly compliant with organizational standards
```

**Use case:** Enforce baseline security, logging, or compliance stacks across every account in your organization automatically.

---

## Delegated Administrator

Instead of always using the management account, you can delegate StackSet administration to a **member account**:

- The delegated administrator can manage StackSets on behalf of the organization
- Useful for separating concerns (e.g., security team manages security stacks)
- Requires trusted access to be enabled in Organizations

```
Management Account
  └─ Delegates StackSet admin to → Security Member Account
                                          ↓
                                   Manages StackSets
                                   across all OUs
```

---

## StackSets with AWS Organizations — Full Picture

```
AWS Organization
  │
  ├─ Management Account (or Delegated Admin)
  │     └─ Creates / Updates / Deletes StackSet
  │
  ├─ Prod OU
  │     ├─ Account A  → Stack Instance (auto-deployed)
  │     └─ Account B  → Stack Instance (auto-deployed)
  │
  └─ Dev OU
        ├─ Account C  → Stack Instance (auto-deployed)
        └─ Account D  → Stack Instance (auto-deployed)
              ↑
        New account joins Dev OU → stack instance auto-created
```

---

## Self-Managed vs Service-Managed — Comparison

| Feature | Self-Managed | Service-Managed |
|---------|-------------|----------------|
| **IAM roles** | Created manually | Created automatically by Organizations |
| **Requires AWS Organizations** | No | Yes |
| **Auto-deploy to new accounts** | No | Yes |
| **Delegated administrator** | No | Yes |
| **Operational overhead** | High | Low |
| **Best for** | Standalone accounts / limited org use | Large organizations, governance at scale |

---

## Best Practices

✓ **Use service-managed permissions with Organizations** — Eliminates manual IAM role management  
✓ **Enable auto-deployment** — Ensures new accounts immediately receive required stacks  
✓ **Use delegated administrator** — Keep management account clean; delegate to security/ops accounts  
✓ **Enable Trusted Access in Organizations** — Required for service-managed permissions to work  
✓ **Test StackSet in one region/account first** — Validate before rolling out to all targets  
✓ **Use OU targeting** — Deploy to entire OUs rather than individual accounts for easier governance  

---

## SysOps Exam Focus

**Q1: "What is a StackSet stack instance?"**
- A) The StackSet template stored in the administrator account
- B) A deployed copy of the StackSet in a specific target account and region
- C) An IAM role in the target account
- D) A nested stack within a StackSet
- **Answer: B** — Stack instance = one deployment of the StackSet in a specific account + region

**Q2: "With self-managed permissions, what must you create manually?"**
- A) S3 bucket for templates
- B) IAM roles with trust relationships in both administrator and target accounts
- C) CloudTrail trail in each account
- D) VPC in each target account
- **Answer: B** — Administration role in admin account + execution role in each target account

**Q3: "What is required to use service-managed permissions with StackSets?"**
- A) A separate AWS account for StackSet management
- B) AWS Organizations with All Features enabled and Trusted Access enabled
- C) Manual IAM role creation in every account
- D) AWS Config enabled in all accounts
- **Answer: B** — Organizations + All Features + Trusted Access

**Q4: "You want new accounts joining your organization to automatically receive a baseline security stack. What feature enables this?"**
- A) CloudFormation Drift Detection
- B) StackSet self-managed permissions
- C) Automatic deployment to new accounts in StackSets with service-managed permissions
- D) CloudFormation hooks
- **Answer: C** — Auto-deployment option in service-managed StackSets

**Q5: "What is the purpose of a delegated administrator in StackSets?"**
- A) To replace the management account entirely
- B) To allow a member account to manage StackSets on behalf of the organization
- C) To create IAM roles in target accounts
- D) To approve stack updates before deployment
- **Answer: B** — Delegated admin = member account authorized to manage StackSets

**Q6: "When a StackSet is updated in the administrator account, what happens to stack instances?"**
- A) Nothing — instances must be manually updated
- B) Only instances in the same region are updated
- C) All stack instances across all target accounts and regions are updated
- D) Instances are deleted and recreated
- **Answer: C** — StackSet updates propagate to all instances automatically

**Q7: "Which permission model requires NO manual IAM role creation?"**
- A) Self-managed permissions
- B) Service-managed permissions with AWS Organizations
- C) Both require manual IAM role creation
- D) Neither — CloudFormation creates roles automatically in all cases
- **Answer: B** — Service-managed with Organizations auto-creates IAM roles

---

## Quick Reference

```
StackSets = Deploy one CloudFormation template → Many accounts + regions

Permission Models:
  Self-Managed    → Manual IAM roles, no Organizations required
  Service-Managed → Auto IAM roles, requires AWS Organizations

Key Features (Service-Managed only):
  ✓ Auto-deploy to new accounts joining an OU
  ✓ Delegated administrator support
  ✓ No IAM role management overhead
```

---

**File: 25_CloudFormation_StackSets.md**
**Status: SysOps-focused, exam-ready, concise format**
