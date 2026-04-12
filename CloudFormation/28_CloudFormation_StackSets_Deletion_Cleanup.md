# 28. CloudFormation StackSets — Deletion & Cleanup

## Overview

Deleting a StackSet requires a specific order of operations. You **cannot delete a StackSet that still has active stack instances** — instances must be removed first.

---

## Two Ways to Remove a Stack Instance

When removing a stack instance from a StackSet, you choose what happens to the underlying stack:

| Option | What Happens | When to Use |
|--------|-------------|-------------|
| **Retain** (detach) | Stack instance removed from StackSet management, but the underlying stack continues to exist in the account | You want to keep the resources but stop managing them via StackSet |
| **Delete** | Stack instance removed from StackSet AND the underlying CloudFormation stack is deleted | Full cleanup — resources are destroyed |

---

## Option 1: Detach (Retain the Stack)

**Console path:** StackSet → Actions → **Delete stacks from StackSet** → enable **Retain stacks**

**Result:**
- Stack instance disappears from the StackSet's **Stack Instances** tab
- The underlying stack still exists in CloudFormation Stacks for that account/region
- Stack is now **unmanaged** — changes to the StackSet no longer affect it

**Use case:** You want to hand off ownership of a specific region's stack to a local team, or you want to preserve resources while decommissioning the StackSet.

```
Before:  StackSet manages 3 instances (Frankfurt, Ireland, London)
Action:  Delete Frankfurt instance with Retain enabled
After:   StackSet manages 2 instances (Ireland, London)
         Frankfurt stack still exists in CloudFormation — but unmanaged
```

---

## Option 2: Delete (Destroy the Stack)

**Console path:** StackSet → Actions → **Delete stacks from StackSet** → leave Retain stacks **disabled**

**Result:**
- Stack instance removed from StackSet
- Underlying CloudFormation stack is deleted
- All resources provisioned by that stack are destroyed

**Tip:** You can target all regions at once and run deletions in **parallel** to speed up cleanup.

---

## Full StackSet Deletion — Correct Order

```
Step 1: Remove all stack instances
          Actions → Delete stacks from StackSet
          Specify all accounts + all regions
          Choose: Delete (not Retain)
          Region concurrency: Parallel (faster)
             ↓
Step 2: Verify Stack Instances tab is empty
             ↓
Step 3: Delete the StackSet itself
          Actions → Delete StackSet → Confirm
             ↓
Step 4: Clean up IAM roles (self-managed permissions only)
          Delete detached stacks still in CloudFormation (if any were Retained)
          Delete AWSCloudFormationStackSetExecutionRole stack
          Delete AWSCloudFormationStackSetAdministrationRole stack
```

> Attempting to delete the StackSet before all instances are removed will fail.

---

## Full Cleanup Checklist (Self-Managed Permissions)

```
☐ Delete all stack instances (choose Delete, not Retain)
☐ Confirm Stack Instances tab is empty
☐ Delete the StackSet (Actions → Delete StackSet)
☐ Delete any Retained stacks still in CloudFormation
☐ Delete the execution role stack (deployed in target accounts)
☐ Delete the administration role stack (deployed in admin account)
```

---

## Best Practices

✓ **Always remove instances before deleting StackSet** — Deletion will fail otherwise  
✓ **Use Retain for stacks you want to keep** — Hands off ownership cleanly without destroying resources  
✓ **Use parallel deletion** — Speeds up removal of many instances across regions  
✓ **Clean up IAM roles last** — Roles are still needed while instances are being deleted  
✓ **Double-check Stack Instances tab** — Ensure it is empty before attempting StackSet deletion  

---

## SysOps Exam Focus

**Q1: "You attempt to delete a StackSet but it fails. What is the most likely cause?"**
- A) The StackSet template has errors
- B) The StackSet still has active stack instances
- C) The IAM execution role has been deleted
- D) The StackSet is in a different region
- **Answer: B** — All stack instances must be removed before the StackSet itself can be deleted

**Q2: "You delete a stack instance from a StackSet with 'Retain stacks' enabled. What happens?"**
- A) The stack and all its resources are deleted
- B) The stack instance is removed from the StackSet but the underlying stack and resources continue to exist
- C) The stack is moved to a different StackSet
- D) The stack is archived in S3
- **Answer: B** — Retain removes StackSet management but preserves the stack and resources

**Q3: "After fully deleting a StackSet with self-managed permissions, what additional cleanup is required?"**
- A) Nothing — CloudFormation cleans up automatically
- B) Delete the S3 bucket used for templates
- C) Delete the administration role and execution role stacks created during setup
- D) Disable AWS Config in each region
- **Answer: C** — IAM role stacks created during setup must be manually deleted

**Q4: "What is the correct order to delete a StackSet?"**
- A) Delete the StackSet → then delete stack instances
- B) Delete IAM roles → delete StackSet → delete instances
- C) Delete all stack instances → delete the StackSet → clean up IAM roles
- D) Delete stack instances and StackSet simultaneously
- **Answer: C** — Instances first, then StackSet, then IAM cleanup

---

## Quick Reference

```
Detach only  →  Delete stacks from StackSet + enable "Retain stacks"
              →  Stack removed from StackSet, resources survive

Full delete  →  Delete stacks from StackSet (Retain disabled)
              →  Stack and resources destroyed

Delete StackSet order:
  1. Remove all instances  →  2. Delete StackSet  →  3. Delete IAM roles
```

---

**File: 28_CloudFormation_StackSets_Deletion_Cleanup.md**
**Status: SysOps-focused, exam-ready, concise format**
