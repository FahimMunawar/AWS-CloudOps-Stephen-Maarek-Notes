# 29. CloudFormation Drift Detection

## Overview

**What is Drift?**

Drift occurs when the actual configuration of a resource differs from its expected configuration as defined in the CloudFormation template. This happens when resources are **manually changed outside of CloudFormation** (e.g., directly in the EC2 or VPC console).

**CloudFormation does not protect against manual changes** — drift detection is how you find out they happened.

---

## What Counts as Drift

| Action | Drift? |
|--------|--------|
| Manual change via AWS Console / CLI / SDK | **Yes** — drifted |
| Change made via CloudFormation stack update | No |
| Change made via StackSet update | No |
| Change made directly to a stack instance (outside StackSet) | Technically not drift, but not best practice |

---

## Drift Detection Scope

You can detect drift at multiple levels:

| Level | What It Checks |
|-------|---------------|
| **Individual resource** | One specific resource within a stack |
| **Stack** | All resources within a CloudFormation stack |
| **StackSet** | Runs drift detection on every stack instance across all accounts/regions |

When drift is detected at any level, the status bubbles up:
```
Drifted resource → Drifted stack → Drifted stack instance → Drifted StackSet
```

---

## Drift Statuses

| Status | Meaning |
|--------|---------|
| **IN_SYNC** | Resource matches the template definition |
| **DRIFTED** | One or more resources differ from the template |
| **NOT_CHECKED** | Drift detection has not been run on this resource |
| **MODIFIED** | Resource exists but properties have changed |
| **DELETED** | Resource was removed outside of CloudFormation |
| **ADDED** | Resource was added directly (CloudFormation doesn't track additions) |

---

## How to Detect Drift

**Console path:**

```
CloudFormation → Stack → Stack Actions → Detect Drift
                                ↓
                    Drift detection runs (async)
                                ↓
Stack Actions → View Drift Results
```

**What the results show:**

For each drifted resource:
- **Expected** — what the CloudFormation template defines
- **Actual** — what currently exists in AWS
- **Differences** — specific property-level changes highlighted

---

## Real-World Example: Security Group Drift

**Template defines:**
```yaml
Resources:
  HTTPSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'HTTP access'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'SSH access'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 10.0.0.0/8
```

**Manual changes made via EC2 console:**
- HTTPSecurityGroup: Changed CIDR, added description, added HTTPS rule
- SSHSecurityGroup: Deleted the SSH inbound rule entirely

**Drift result:**

```
Stack Status: DRIFTED

HTTPSecurityGroup → MODIFIED
  Difference 1: CidrIp — Expected: 0.0.0.0/0  |  Actual: 10.0.0.0/8
  Difference 2: Rule added — HTTPS port 443 from 0.0.0.0/0 (not in template)

SSHSecurityGroup → MODIFIED
  Difference 1: Rule removed — SSH port 22 rule no longer exists
```

---

## Responding to Drift

Once drift is detected, you have two options:

**Option A: Update the template to match reality**
- If the manual change was intentional and correct
- Update the CloudFormation template to reflect the new configuration
- Run a stack update to bring CloudFormation in sync

**Option B: Revert the resource to match the template**
- If the manual change was unauthorized or accidental
- Run a stack update — CloudFormation will restore the resource to the template definition
- Use CloudTrail to investigate who made the change and when

```
Drift detected
      ↓
Was the change intentional?
  Yes → Update template to match actual → Stack update
  No  → Investigate via CloudTrail → Revert via stack update
```

---

## Using CloudTrail with Drift

CloudTrail records who made changes and when. After detecting drift, use CloudTrail to:
- Identify the IAM user or role that made the unauthorized change
- Determine the exact time of the change
- Gather evidence for security or compliance review

---

## Drift Detection on StackSets

- Running drift detection on a StackSet applies it to **every stack instance** across all accounts and regions
- Drift is reported at the **resource → stack → stack instance → StackSet** chain
- You can stop a drift detection operation on a StackSet if needed

---

## Best Practices

✓ **Run drift detection regularly** — Catch unauthorized changes before they cause incidents  
✓ **Automate drift detection** — Use EventBridge + Lambda to schedule periodic drift checks  
✓ **Use CloudTrail alongside drift** — Drift tells you *what* changed; CloudTrail tells you *who* and *when*  
✓ **Treat drifted stacks as incidents** — Investigate the root cause, don't just revert blindly  
✓ **Fix drift by updating via CloudFormation** — Never re-apply the same manual change to "match" actual  
✓ **Apply drift detection to StackSets** — Governance across all accounts requires StackSet-level drift checks  

---

## SysOps Exam Focus

**Q1: "What is CloudFormation drift?"**
- A) A CloudFormation stack that has failed to deploy
- B) When a resource's actual configuration differs from what the CloudFormation template defines due to manual changes
- C) A CloudFormation template that has syntax errors
- D) When a stack update fails midway
- **Answer: B** — Drift = manual changes made outside CloudFormation

**Q2: "You manually delete an inbound rule from a security group created by CloudFormation. What drift status will that resource show?"**
- A) DELETED
- B) NOT_CHECKED
- C) MODIFIED
- D) ADDED
- **Answer: C** — The resource still exists but its properties have changed → MODIFIED

**Q3: "How do you initiate drift detection on a CloudFormation stack?"**
- A) CloudFormation → Stack → Edit → Run drift
- B) CloudFormation → Stack → Stack Actions → Detect Drift
- C) EC2 Console → Security Groups → Check drift
- D) AWS Config → Compliance → Detect drift
- **Answer: B** — Stack Actions → Detect Drift is the correct path

**Q4: "Drift detection on a StackSet applies to:"**
- A) Only the administrator account
- B) Only the most recently updated stack instance
- C) Every stack instance across all target accounts and regions
- D) Only stack instances in the same region as the StackSet
- **Answer: C** — StackSet drift detection covers all instances

**Q5: "After detecting drift, you want to revert the resource to its template definition. What is the correct approach?"**
- A) Manually reapply the original configuration via the console
- B) Delete and recreate the stack
- C) Run a CloudFormation stack update — CloudFormation will restore the resource to match the template
- D) Use CloudTrail to undo the change
- **Answer: C** — A stack update reconciles the resource back to the template definition

**Q6: "Which service helps you find out WHO made a manual change that caused drift?"**
- A) AWS Config
- B) CloudFormation drift results
- C) AWS CloudTrail
- D) Amazon GuardDuty
- **Answer: C** — CloudTrail logs API calls including who changed the resource and when

**Q7: "True or false: A change made via a CloudFormation stack update is considered drift."**
- A) True
- B) False
- **Answer: B** — Drift only occurs from changes made **outside** CloudFormation

---

## Quick Reference

```
Drift = manual change made outside CloudFormation

Detect:   Stack → Stack Actions → Detect Drift → View Drift Results
Results:  Expected vs Actual, property-level differences shown
Scope:    Individual resource | Stack | StackSet (all instances)

Response:
  Intentional change  → Update template → stack update
  Unauthorized change → Investigate CloudTrail → stack update to revert
```

---

**File: 29_CloudFormation_Drift_Detection.md**
**Status: SysOps-focused, exam-ready, concise format**
