# 7 — IAM Policy Simulator

## Overview

A tool to test and troubleshoot IAM policies **without applying them** to a live environment.

Access: search "IAM policy simulator" → AWS documentation link → console URL

---

## What It Can Test

| Policy Type | Supported |
|---|---|
| Identity-based policies (user, group, role) | Yes |
| Resource-based policies | Yes |
| Permission boundaries | Yes |
| SCPs (AWS Organizations) | Yes |

---

## How to Use

1. Select an entity: IAM user, group, or role
2. Filter policies to test (all attached or select specific ones)
3. Select an AWS service (e.g., Amazon SQS)
4. Select specific actions (e.g., `DeleteMessage`, `DeleteQueue`)
5. Click **Run Simulation**

**Result:** Allowed or Denied + the specific policy statement that matched (or "implicit deny" if no statement matched)

---

## Use Cases

- Verify a user has the correct permissions before they need them
- Debug an "Access Denied" error — identify which policy is blocking
- Test the effect of removing a specific policy from a user
- Validate SCP impact on identity-based policies
- Test permission boundaries are working as intended

---

## SysOps Exam Q&A

**Q: A developer reports getting "Access Denied" on SQS. How do you quickly identify which policy is blocking them?**
A: Use the IAM Policy Simulator — select the user, choose SQS actions, run the simulation. It shows whether the action is denied and which statement (or lack thereof) caused it.

**Q: What types of policies can be tested in the IAM Policy Simulator?**
A: Identity-based policies, resource-based policies, permission boundaries, and SCPs.

---

## Quick Reference

```
IAM Policy Simulator:
  Test IAM permissions without applying to live environment
  Supports: identity-based, resource-based, permission boundaries, SCPs

  Select entity → filter policies → select service + actions → Run Simulation
  Result: Allowed / Denied + matching statement or "implicit deny"

  Use for: pre-deployment validation, Access Denied debugging, SCP impact testing
```
