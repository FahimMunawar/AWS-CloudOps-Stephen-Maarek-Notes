# 31. CloudFormation Stack Failure Options

## Overview

When a CloudFormation stack creation fails, you can control what happens next using the **on-failure** option. This is set at stack creation time — either in the console under "Stack failure options" or via the CLI.

---

## Three On-Failure Options

| Option | CLI Value | Stack Status | Resources | Use When |
|--------|-----------|-------------|-----------|----------|
| **Rollback** | `ROLLBACK` | ROLLBACK_COMPLETE | All deleted | Default — production deployments |
| **Do Nothing** | `DO_NOTHING` | CREATE_FAILED | Preserved | Debugging — inspect what was created |
| **Delete** | `DELETE` | (stack deleted) | All deleted + stack gone | Clean slate — no trace left behind |

---

## Option Details

### Rollback (Default)
```bash
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --on-failure ROLLBACK
```
- Stack creation fails → all created resources are rolled back and deleted
- Stack remains in `ROLLBACK_COMPLETE` state (visible in console)
- Must delete the stack manually before you can retry with the same name

---

### Do Nothing
```bash
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --on-failure DO_NOTHING
```
- Stack creation fails → all successfully created resources are **kept**
- Stack enters `CREATE_FAILED` state
- You can SSH into EC2 instances, check logs, troubleshoot the failure
- Once fixed, manually roll back or delete the stack and redeploy

> **Exam focus:** `DO_NOTHING` is the key option for debugging — it is equivalent to the console setting "Preserve successfully provisioned resources" covered in lesson 22.

---

### Delete
```bash
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --on-failure DELETE
```
- Stack creation fails → all resources deleted **and the stack itself is removed**
- Nothing remains — not even the stack entry in CloudFormation
- Use when you want zero trace of a failed deployment

---

## Console Equivalent

In the CloudFormation console during stack creation:

**Stack failure options:**
- `Roll back all stack resources` → equivalent to `ROLLBACK`
- `Preserve successfully provisioned resources` → equivalent to `DO_NOTHING`

(Delete-on-failure is CLI/API only — not a distinct console option)

---

## Comparison: Do Nothing vs Rollback

| | ROLLBACK | DO_NOTHING |
|-|----------|------------|
| **Resources after failure** | Deleted | Preserved |
| **Stack status** | ROLLBACK_COMPLETE | CREATE_FAILED |
| **Can SSH to EC2?** | No — instance deleted | Yes — instance still running |
| **Best for** | Production | Development / debugging |

---

## SysOps Exam Focus

**Q1: "A CloudFormation stack creation fails. You want to SSH into the EC2 instance to check the bootstrap logs. Which on-failure option should you use?"**
- A) ROLLBACK
- B) DELETE
- C) DO_NOTHING
- D) PRESERVE
- **Answer: C** — DO_NOTHING keeps resources alive for inspection

**Q2: "What is the default behavior when a CloudFormation stack creation fails?"**
- A) Stack enters CREATE_FAILED and resources are preserved
- B) Stack and all created resources are rolled back and deleted
- C) Stack is deleted but resources are retained
- D) CloudFormation retries the creation automatically
- **Answer: B** — ROLLBACK is the default

**Q3: "With on-failure=DELETE, what happens after a stack creation failure?"**
- A) Resources are deleted but the stack entry remains
- B) Resources are preserved and the stack enters CREATE_FAILED
- C) Both resources and the stack entry are deleted entirely
- D) The stack is rolled back to the previous successful state
- **Answer: C** — DELETE removes everything including the stack itself

**Q4: "Which on-failure option leaves the stack in CREATE_FAILED status?"**
- A) ROLLBACK
- B) DELETE
- C) DO_NOTHING
- D) RETAIN
- **Answer: C** — Only DO_NOTHING results in CREATE_FAILED with resources preserved

---

## Quick Reference

```
On-failure options (set at stack creation — cannot change after):

ROLLBACK    → fail → delete resources → ROLLBACK_COMPLETE  (default)
DO_NOTHING  → fail → keep resources  → CREATE_FAILED       (debugging)
DELETE      → fail → delete resources + stack → gone        (clean slate)
```

---

**File: 31_CloudFormation_Stack_Failure_Options.md**
**Status: SysOps-focused, exam-ready, concise format**
