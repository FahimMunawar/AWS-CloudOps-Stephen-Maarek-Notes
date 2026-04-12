# 27. CloudFormation StackSets — Managing Stack Instances

## Overview

After a StackSet is created, you can add more stack instances to cover additional accounts and regions at any time — without recreating the StackSet.

---

## Adding Stacks to an Existing StackSet

**Console path:** StackSet → Actions → **Add stacks to StackSet**

**What you can specify:**
- Additional **account IDs** (or OU IDs if using Organizations)
- Additional **regions** to deploy into
- Optional **parameter overrides** for the new instances only

**Example:**

```
Existing StackSet instances:
  Account: 123456789012  |  Region: eu-central-1  (Frankfurt)
  Account: 123456789012  |  Region: eu-west-1      (Ireland)

After "Add stacks to StackSet":
  Account: 123456789012  |  Region: eu-central-1   (Frankfurt)  ← existing
  Account: 123456789012  |  Region: eu-west-1       (Ireland)   ← existing
  Account: 123456789012  |  Region: eu-west-2       (London)    ← new
```

---

## Parameter Overrides

When adding new stack instances, you can **override specific parameters** for those instances only — the rest inherit the StackSet-level defaults.

**Use case:** Deploy the same template to a new region but with a different instance type or bucket name.

```
StackSet default:  InstanceType = t2.micro
Override for London region:  InstanceType = t3.micro
```

This avoids creating a separate StackSet just for minor regional differences.

---

## StackSet Operations

Every action on a StackSet (create, update, add, delete instances) is tracked as an **operation**:

| Operation Type | Triggered By |
|---------------|-------------|
| CREATE | Initial StackSet creation |
| UPDATE | Updating StackSet template or parameters |
| ADD | Adding new stack instances |
| DELETE | Removing stack instances or deleting the StackSet |

**Operations tab** in the console shows status, start time, and outcome of every operation.

---

## Stack Instance Statuses

| Status | Meaning |
|--------|---------|
| **CURRENT** | Stack instance matches the StackSet definition |
| **OUTDATED** | StackSet was updated but this instance hasn't been updated yet |
| **INOPERABLE** | Stack instance is in a failed state and needs manual intervention |

---

## Managing Instances — Available Actions

From the **Actions** menu on a StackSet:

| Action | Purpose |
|--------|---------|
| **Add stacks to StackSet** | Deploy to new accounts or regions |
| **Override StackSet parameters** | Apply different parameter values to specific instances |
| **Edit StackSet** | Update the template or global parameters (affects all instances) |
| **Delete stacks from StackSet** | Remove specific instances without deleting the StackSet |
| **Delete StackSet** | Remove all instances and the StackSet itself |

---

## Best Practices

✓ **Add regions incrementally** — Start with one region, validate, then expand  
✓ **Use parameter overrides sparingly** — Too many overrides make the StackSet hard to maintain  
✓ **Monitor the Operations tab** — Each add/update creates a trackable operation with success/failure status  
✓ **Check stack instance status after operations** — All instances should be `CURRENT` when done  
✓ **Delete instances before deleting StackSet** — Attempting to delete a StackSet with active instances will fail  

---

## SysOps Exam Focus

**Q1: "How do you deploy an existing StackSet to a new region?"**
- A) Create a new StackSet for the new region
- B) Use Actions → Add stacks to StackSet and specify the new region
- C) Update the StackSet template with the new region
- D) Manually create a stack in the new region
- **Answer: B** — Add stacks to StackSet allows expanding to new accounts/regions

**Q2: "What does a stack instance status of OUTDATED mean?"**
- A) The stack instance failed to deploy
- B) The StackSet was updated but this instance has not been updated yet
- C) The stack instance was manually modified causing drift
- D) The region is no longer supported
- **Answer: B** — OUTDATED means the StackSet definition changed but the instance hasn't caught up

**Q3: "You need the same StackSet template deployed in London with a different parameter value. What should you use?"**
- A) Create a separate StackSet for London
- B) Parameter overrides when adding the London stack instance
- C) Hardcode the value in the template for London
- D) Use a condition in the template based on region
- **Answer: B** — Parameter overrides allow per-instance customization without a new StackSet

---

## Quick Reference

```
Add coverage  →  Actions → Add stacks to StackSet → specify account + region
Customize     →  Override parameters for specific instances
Track         →  Operations tab shows every action and its status
Verify        →  Stack Instances tab — all should show CURRENT when done
```

---

**File: 27_CloudFormation_StackSets_Managing_Instances.md**
**Status: SysOps-focused, exam-ready, concise format**
