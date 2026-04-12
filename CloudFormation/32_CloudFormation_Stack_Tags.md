# 32. CloudFormation Stack-Level Tags

## Overview

**What are Stack-Level Tags?**

Tags applied directly to a CloudFormation stack (not inside the template) are **automatically propagated** to all supported resources created by that stack.

---

## How It Works

```
Stack tags defined (key: value)
        ↓
CloudFormation deploys stack
        ↓
Tags automatically applied to all supported resources
(EC2 instances, S3 buckets, RDS, security groups, etc.)
```

**Set via console:** During stack creation/update → Configure stack options → Tags section

**Set via CLI:**
```bash
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --tags Key=Environment,Value=Production \
         Key=Team,Value=Platform \
         Key=CostCenter,Value=Engineering
```

---

## Key Points

- Tags applied at the **stack level** propagate to all **supported resources** in the stack
- Tags defined **inside the template** (in resource `Properties`) apply only to that specific resource
- Stack-level tags are **additive** — they are merged with any resource-level tags already defined
- Useful for **cost allocation**, **resource discovery**, and **governance**

---

## Use Case: Finding All Resources for a Stack

```
Apply tag:  Project = MyApp  to the CloudFormation stack
        ↓
All EC2 instances, RDS databases, S3 buckets, etc. get  Project = MyApp
        ↓
Use Tag Editor or Cost Explorer to find/filter all MyApp resources
```

---

## SysOps Exam Focus

**Q1: "You apply a tag to a CloudFormation stack in the console. Where does this tag get applied?"**
- A) Only to the stack itself in CloudFormation
- B) Only to EC2 instances within the stack
- C) Automatically to all supported resources created by the stack
- D) Only to resources that explicitly define tags in the template
- **Answer: C** — Stack-level tags propagate to all supported resources automatically

**Q2: "What is the benefit of using stack-level tags over resource-level tags in a template?"**
- A) Stack-level tags override resource-level tags
- B) A single tag on the stack automatically applies to all resources without modifying the template
- C) Stack-level tags are applied before resource creation
- D) Stack-level tags support more tag keys than resource-level tags
- **Answer: B** — One tag on the stack covers all resources without editing the template

---

## Quick Reference

```
Stack tag  →  auto-propagated to all supported resources in the stack
Template resource tag  →  applies to that resource only

Use stack tags for:  cost allocation, resource grouping, environment labeling
```

---

**File: 32_CloudFormation_Stack_Tags.md**
**Status: SysOps-focused, exam-ready, concise format**
