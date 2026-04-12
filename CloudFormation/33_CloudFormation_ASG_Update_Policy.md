# 33. CloudFormation — Auto Scaling Group Update Policy

## Overview

CloudFormation can manage how EC2 instances within an Auto Scaling Group (ASG) are updated when a stack update occurs (e.g., changing the AMI). This is controlled via the `UpdatePolicy` attribute on the ASG resource — not in `Properties`.

---

## Two Update Strategies

| Strategy | Attribute | Approach | Risk to existing ASG |
|----------|-----------|----------|---------------------|
| **Rolling Update** | `AutoScalingRollingUpdate` | Updates instances in batches, in place | Low — existing ASG is modified gradually |
| **Replacing Update** | `AutoScalingReplacingUpdate` | Creates a brand new ASG, swaps when ready | None — old ASG untouched until new one succeeds |

---

## Option 1: Rolling Update

Updates instances **in batches** within the existing ASG. CloudFormation terminates a batch of old instances and replaces them with new ones (using the updated launch configuration/template).

```yaml
Resources:
  MyASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    UpdatePolicy:
      AutoScalingRollingUpdate:
        MinInstancesInService: 1      # Keep at least 1 instance running during update
        MaxBatchSize: 2               # Update max 2 instances at a time
        PauseTime: PT5M               # Wait 5 minutes for new instances to pass health checks
        WaitOnResourceSignals: true   # Wait for cfn-signal from new instances before continuing
    Properties:
      MinSize: 2
      MaxSize: 6
      DesiredCapacity: 4
      LaunchTemplate:
        LaunchTemplateId: !Ref MyLaunchTemplate
        Version: !GetAtt MyLaunchTemplate.LatestVersionNumber
```

**Key parameters:**

| Parameter | Purpose |
|-----------|---------|
| `MinInstancesInService` | Minimum healthy instances to keep running during the update |
| `MaxBatchSize` | How many instances to replace per batch |
| `PauseTime` | How long to wait between batches (ISO 8601 format) |
| `WaitOnResourceSignals` | If true, waits for cfn-signal from new instances before moving to next batch |

**Flow:**
```
Update triggered (e.g., new AMI in launch template)
        ↓
Batch 1: Replace 2 old instances with 2 new ones
        ↓
Wait PauseTime (or cfn-signal if WaitOnResourceSignals: true)
        ↓
Batch 2: Replace next 2 old instances
        ↓
... repeat until all instances updated
        ↓
Update complete — same ASG, all new instances
```

---

## Option 2: Replacing Update

Creates an **entirely new ASG** alongside the old one. The old ASG is only deleted after the new one is confirmed healthy.

```yaml
Resources:
  MyASG:
    Type: AWS::AutoScaling::AutoScalingGroup
    UpdatePolicy:
      AutoScalingReplacingUpdate:
        WillReplace: true             # Must be set to true to enable this strategy
    Properties:
      MinSize: 2
      MaxSize: 6
      DesiredCapacity: 4
      LaunchTemplate:
        LaunchTemplateId: !Ref MyLaunchTemplate
        Version: !GetAtt MyLaunchTemplate.LatestVersionNumber
```

**Flow:**
```
Update triggered
        ↓
New ASG created with updated configuration
(both old and new ASG exist simultaneously)
        ↓
New ASG reaches desired capacity + passes health checks
        ↓
Success? → Old ASG deleted, new ASG takes over
Failure? → New ASG deleted, old ASG kept intact (no disruption)
```

**Important:** During the transition, you temporarily need **double the EC2 capacity** (both ASGs running at the same time). Ensure your account has sufficient EC2 quota.

---

## Rolling vs Replacing — Comparison

| | Rolling Update | Replacing Update |
|-|---------------|-----------------|
| **Existing ASG** | Modified in place | Untouched until swap |
| **Two ASGs at once** | No | Yes (temporarily) |
| **EC2 capacity needed** | Normal | Double during transition |
| **Rollback on failure** | Partial — some instances may be new | Clean — old ASG preserved entirely |
| **Speed** | Slower (batches) | Faster (parallel creation) |
| **Risk** | Medium | Low — old ASG never touched |

---

## Best Practices

✓ **Use Rolling Update for gradual, low-risk updates** — Good for large ASGs where you want minimal disruption  
✓ **Use Replacing Update for clean, safe swaps** — Old ASG is never modified, easy rollback  
✓ **Set WaitOnResourceSignals: true with Rolling** — Ensures new instances are healthy before next batch  
✓ **Ensure EC2 quota for Replacing Update** — Account must support double capacity during transition  
✓ **Use PauseTime generously** — Give new instances enough time to pass health checks before proceeding  

---

## SysOps Exam Focus

**Q1: "What CloudFormation attribute controls how an ASG is updated during a stack update?"**
- A) Properties
- B) UpdatePolicy
- C) CreationPolicy
- D) DependsOn
- **Answer: B** — UpdatePolicy on the ASG resource controls update behavior

**Q2: "With AutoScalingRollingUpdate, what does MaxBatchSize control?"**
- A) The maximum number of ASGs that can exist simultaneously
- B) The maximum number of instances replaced per batch during a rolling update
- C) The maximum number of instances in the ASG
- D) The maximum wait time between batches
- **Answer: B** — MaxBatchSize defines how many instances are replaced per batch

**Q3: "You use AutoScalingReplacingUpdate with WillReplace: true. The update fails. What happens?"**
- A) Both old and new ASGs are deleted
- B) The new ASG is kept and the old one is deleted
- C) The new ASG is deleted and the old ASG is kept intact
- D) CloudFormation rolls back the entire stack
- **Answer: C** — On failure, new ASG is deleted and old ASG is preserved unchanged

**Q4: "What is a key infrastructure consideration when using AutoScalingReplacingUpdate?"**
- A) You need a load balancer attached to the ASG
- B) You need double EC2 capacity available since both old and new ASGs run simultaneously during transition
- C) You need to enable Auto Scaling lifecycle hooks
- D) You need to use a launch configuration instead of a launch template
- **Answer: B** — Both ASGs run in parallel temporarily, requiring double the normal capacity

**Q5: "Which update strategy leaves the existing ASG completely untouched until the new one is confirmed healthy?"**
- A) AutoScalingRollingUpdate
- B) AutoScalingScheduledAction
- C) AutoScalingReplacingUpdate
- D) AutoScalingRollingReplace
- **Answer: C** — Replacing update creates a new ASG; old one is never modified

---

## Quick Reference

```
UpdatePolicy on ASG:

Rolling    →  AutoScalingRollingUpdate
              In-place batch replacement
              Parameters: MinInstancesInService, MaxBatchSize, PauseTime

Replacing  →  AutoScalingReplacingUpdate + WillReplace: true
              New ASG created alongside old one
              Success → old deleted | Failure → new deleted, old preserved
```

---

**File: 33_CloudFormation_ASG_Update_Policy.md**
**Status: SysOps-focused, exam-ready, concise format**
