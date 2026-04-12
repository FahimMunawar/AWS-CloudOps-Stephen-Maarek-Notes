# 10. CloudFormation Rollbacks and Failure Handling

## Part 1: Understanding CloudFormation Rollbacks

### What Are Rollbacks?

**Definition:**

```
Rollbacks:
├─ Automatic mechanism to handle stack failures
├─ Reverts stack to last known stable state
├─ Triggered when resource creation/update fails
├─ Removes failed resources
├─ Maintains stack consistency
└─ Critical for infrastructure reliability
```

**Why Rollbacks Matter:**

```
Without Rollbacks:
├─ Failed stack left in inconsistent state
├─ Some resources created, others failed
├─ Manual cleanup required
├─ Infrastructure in limbo
├─ Unpredictable state
└─ Difficult to retry

With Rollbacks:
├─ Failed stack cleaned up automatically
├─ Infrastructure returns to previous working state
├─ Clear, consistent endpoint
├─ Can investigate failure
├─ Can retry with fixes
└─ Professional infrastructure management
```

### Rollback Scenarios

**Scenario 1: Stack Creation Failure**

```
Timeline:
├─ User creates stack with template
├─ CloudFormation begins resource creation
├─ Resource 1 created: Success ✓
├─ Resource 2 created: Success ✓
├─ Resource 3 creation: FAILS ✗
│  ├─ Resources 1, 2, 3 deleted (rollback)
│  └─ Stack status: CREATE_FAILED
├─ User can view events/logs
├─ Fix template
└─ Retry stack creation

Result:
├─ No resources remain
├─ No manual cleanup needed
├─ Clean slate for retry
└─ Safe default behavior
```

**Scenario 2: Stack Update Failure**

```
Timeline:
├─ Stack exists with Resource A (working)
├─ User updates stack via new template
├─ CloudFormation begins update
├─ Update Resource A: Success ✓
├─ Create Resource B: FAILS ✗
│  ├─ Changes reversed (automatic)
│  ├─ Resource A returned to previous state
│  ├─ Resource B creation reverted
│  └─ Stack status: UPDATE_FAILED
├─ User can view events/logs
├─ Fix template
└─ Retry stack update

Result:
├─ Stack returns to last working state
├─ Partial changes not left behind
├─ Infrastructure consistent
└─ Safe, automatic reversal
```

## Part 2: Stack Creation Failures

### Creation Failure: Default Behavior

**Rollback on Failure (Default):**

```
Stack Creation Process:

1. Start: CloudFormation receives template
   └─ Status: CREATE_IN_PROGRESS

2. Creating Resources:
   ├─ SecurityGroup-1: Created ✓
   ├─ SecurityGroup-2: Created ✓
   ├─ EC2 Instance: CREATION FAILS ✗
   │  └─ Invalid AMI ID (ami-nonexistent)
   └─ Status: CREATE_FAILED

3. Default Rollback Triggered:
   ├─ SecurityGroup-2: Deleted
   ├─ SecurityGroup-1: Deleted
   ├─ All created resources removed
   └─ Status: ROLLBACK_COMPLETE

4. Result:
   ├─ Stack: No resources exist
   ├─ No manual cleanup
   ├─ Events logged for debugging
   └─ User can investigate and retry
```

**Behavior:**

```
✓ Automatic rollback (GOOD)
  ├─ No orphaned resources
  ├─ Clean, consistent state
  ├─ No additional charges
  └─ Default for safety

Limitation:
├─ Cannot inspect created resources
├─ Cannot see intermediate state
├─ Must infer from logs what failed
└─ Troubleshooting harder
```

### Creation Failure: Preserve Resources

**Disable Rollback (Preserve Successfully Created):**

```
Stack Creation Process (With Preserve Option):

Console Selection:
├─ Stack failure options:
│  ├─ [ ] Roll back all stack resources (DEFAULT)
│  └─ [✓] Preserve successfully provisioned resources (SELECTED)

Creation Continues:
├─ SecurityGroup-1: Created ✓
├─ SecurityGroup-2: Created ✓
├─ EC2 Instance: CREATION FAILS ✗
│  └─ Invalid AMI ID
└─ Status: CREATE_FAILED

What Happens:
├─ SecurityGroup-1: KEPT (successful)
├─ SecurityGroup-2: KEPT (successful)
├─ EC2 Instance: NOT attempted (failed)
└─ Status: CREATE_FAILED with preserved resources

User Can:
├─ Inspect working resources
├─ Check configuration
├─ Troubleshoot what's wrong
├─ View resource properties
└─ Better understanding of failure

MUST DO:
├─ Delete entire stack when done
├─ Cannot update to fix template
├─ Cannot leave orphaned resources
└─ Clean up after investigation
```

**When to Use Preserve Option:**

```
✓ Use When:
├─ Debugging complex template
├─ Need to inspect intermediate state
├─ Want to understand what worked
├─ Development/testing environment
└─ One-off investigation

✗ Don't Use When:
├─ Production environment
├─ Want automatic cleanup
├─ Trust your template
├─ Want quick retry
└─ Cost conscious (resources running)

Exam Context:
├─ Know the option exists
├─ Default is rollback all
├─ Preserve is for troubleshooting
├─ Development use case primarily
└─ Must delete stack after
```

## Part 3: Stack Update Failures

### Update Failure: Automatic Rollback

**Default Update Rollback:**

```
Stack Update Process (Update Fails):

Initial State:
├─ Instance: i-123456 (working)
├─ SecurityGroup: sg-111111 (working)
└─ Status: CREATE_COMPLETE

User Updates Stack (new template):
├─ Update Instance config: ✓ (in-progress)
├─ Create new SecurityGroup: ✓ (created successfully)
├─ Modify existing SecurityGroup: FAILS ✗
│  └─ Invalid parameter value
└─ Status: UPDATE_FAILED

Automatic Rollback Triggered:
├─ Undo: SecurityGroup modification → reverted
├─ Undo: New SecurityGroup → deleted
├─ Undo: Instance update → reverted to previous config
├─ All changes reversed automatically
└─ Status: UPDATE_ROLLBACK_COMPLETE

Result:
├─ Instance: i-123456 (back to original config)
├─ SecurityGroup: sg-111111 (unchanged)
├─ No manual intervention needed
├─ Infrastructure state consistent
└─ User can investigate and retry
```

**Comparison: Create vs Update Failure**

```
Create Failure (Default):
├─ All resources deleted
├─ Stack: ROLLBACK_COMPLETE
├─ Completely empty
└─ Clean slate for retry

Update Failure (Default):
├─ All changes reversed
├─ Stack: UPDATE_ROLLBACK_COMPLETE
├─ Returns to previous working state
├─ Infrastructure continuous
└─ No downtime for successful resources
```

### Update Failure: Preserve Resources on Rollback Failure

**Preserve During Update Failure:**

```
When Creating Stack Update, Two Options Offered:

Option 1: Roll back all stack resources (DEFAULT)
└─ All changes reverted if update fails

Option 2: Preserve successfully provisioned resources
└─ Keep successful changes if update fails

Example Using Option 2:

Update Attempted:
├─ Create SecurityGroup-A: ✓ Successful
├─ Create SecurityGroup-B: ✓ Successful
├─ Create EC2 Instance: FAILS ✗ (invalid AMI)
└─ Status: UPDATE_FAILED

With "Preserve" Option:
├─ SecurityGroup-A: KEPT (created successfully)
├─ SecurityGroup-B: KEPT (created successfully)
├─ EC2 changes: ROLLED BACK
└─ Allows inspection of what worked

Result:
├─ Partial update remains
├─ Some resources from new template
├─ Can troubleshoot
├─ Must eventually delete stack
└─ Useful for complex templates
```

## Part 4: Rollback Failures (Critical)

### When Rollback Itself Fails

**Rollback Failure Scenario:**

```
Stack Update with Rollback Failure:

Update Process:
├─ Create Resource A: ✓
├─ Update Resource B: FAILS ✗
└─ Rollback initiated

During Rollback:
├─ Delete Resource A: FAILS ✗
│  └─ Manual changes to Resource A
│  └─ Cannot delete programmatically
│  └─ Resource is now in orphaned state
├─ Status: UPDATE_ROLLBACK_FAILED
└─ Requires manual intervention

Root Cause:
├─ Resource was modified outside CloudFormation
├─ CloudFormation cannot revert manual changes
├─ Cannot safely rollback
└─ Stack left in inconsistent state
```

**Typical Causes:**

```
Why Rollback Fails:

1. Manual Resource Changes
   ├─ User manually modified resource in console
   ├─ CloudFormation cannot revert manual edits
   ├─ Example: User deleted subnet that template created
   └─ Resource state inconsistent

2. External State Changes
   ├─ Third-party system modified resource
   ├─ Resource permissions changed
   ├─ Resource dependency deleted
   └─ Cannot rollback to invalid state

3. Insufficient Permissions
   ├─ IAM role no longer has permissions
   ├─ User deleted IAM policy
   ├─ Account permissions changed
   └─ Cannot delete resources

4. Resource Already Deleted
   ├─ Resource manually deleted before rollback
   ├─ Cannot delete twice
   ├─ CloudFormation doesn't know resource gone
   └─ Rollback attempts to delete non-existent resource
```

### Stack Recovery: ContinueUpdateRollback

**Recovery Process:**

```
When Stack Has UPDATE_ROLLBACK_FAILED Status:

Option 1: Manual Fix (Recommended)
├─ Identify what was manually changed
├─ Revert manual changes
├─ Clean up orphaned resources
├─ Delete remains manually
└─ Start over with clean template

Option 2: ContinueUpdateRollback
├─ Tell CloudFormation to try rollback again
├─ Use after fixing rootcause
├─ API/CLI/Console option
├─ Prerequisite: Fix the blocked resources first!
└─ Try to complete rollback

Using ContinueUpdateRollback:

AWS Console:
├─ Select stack with UPDATE_ROLLBACK_FAILED
├─ Click: Continue Update Rollback
└─ CloudFormation retries rollback

AWS CLI:
aws cloudformation continue-update-rollback \
  --stack-name MyStackName

AWS API:
ContinueUpdateRollback(StackName='MyStackName')

Outcome:
├─ If successful: Status becomes UPDATE_ROLLBACK_COMPLETE
├─ If fails again: Requires more manual intervention
└─ Stack eventually reaches consistent state
```

**Typical Recovery Workflow:**

```
Step 1: Stack has UPDATE_ROLLBACK_FAILED
├─ View stack events
├─ Identify which resource blocked rollback
└─ Note resource ID that failed to delete

Step 2: Manual Investigation
├─ Go to console (EC2, RDS, S3, etc.)
├─ Find the blocked resource
├─ Check what's wrong:
│  ├─ Is it manually modified?
│  ├─ Does it have dependencies?
│  ├─ Are there permissions issues?
│  └─ Is it still needed?
└─ Determine fix strategy

Step 3: Manual Remediation
├─ Option A: Revert manual changes
│  └─ Change resource back to CloudFormation state
├─ Option B: Delete resource manually
│  └─ Remove it so CloudFormation can proceed
├─ Option C: Fix permissions
│  └─ Restore IAM permissions
└─ Fix all blocked resources

Step 4: ContinueUpdateRollback
├─ Execute continue-update-rollback command
├─ CloudFormation retries deletion
├─ Previously blocked resources now delete
└─ Stack reaches UPDATE_ROLLBACK_COMPLETE

Step 5: Delete Stack
├─ aws cloudformation delete-stack
├─ Start fresh with fixed template
└─ Test in dev first next time
```

## Part 5: Hands-On Practice: Failure Scenarios

### Lab 1: Creation Failure with Rollback

**Template: trigger-failure.yaml (Intentional Failure)**

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: "Template with intentional failure - invalid AMI ID"

Resources:
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "SSH access"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Server access"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-nonexistent  # INVALID - will cause failure
      InstanceType: t2.micro
      SecurityGroupIds:
        - !Ref SSHSecurityGroup
        - !Ref ServerSecurityGroup

Problem in Template:
├─ ImageId: ami-nonexistent (fake AMI)
├─ Does not exist in region
├─ EC2 cannot launch instance
└─ Stack creation will fail
```

**Lab Procedure: Observe Default Rollback**

```
Step 1: Create Stack Using CloudFormation Console
├─ Open: AWS CloudFormation Console → Create Stack
├─ Upload: trigger-failure.yaml template
├─ Enter Stack Name: TriggerCreationFailure
├─ Accept all defaults
└─ Note: Stack failure options shows two choices

Step 2: Choose Default Option
├─ Select: "Roll back all stack resources" (DEFAULT)
├─ Click: Next, Submit
└─ Stack creation begins

Step 3: Observe Creation Progress
├─ Status shows: CREATE_IN_PROGRESS
├─ First: SecurityGroup (SSH) created ✓
├─ Second: SecurityGroup (Server) created ✓
├─ Third: EC2 Instance creation starts
├─ Event: "ImageId is not found in this region"
├─ Status changes to: CREATE_FAILED
└─ Automatic rollback triggered

Step 4: Observe Automatic Rollback
├─ Events show:
│  ├─ SSHSecurityGroup deletion initiated
│  ├─ ServerSecurityGroup deletion initiated
│  ├─ MyInstance deletion (never created, skipped)
│  └─ All deletions complete
├─ Status: ROLLBACK_COMPLETE
├─ Resources tab: EMPTY (no resources remain)
└─ Result: Clean, consistent state

Conclusion:
├─ Default rollback worked perfectly
├─ All successfully created resources deleted
├─ Stack left in clean state
├─ No manual cleanup needed
└─ This is the DEFAULT safe behavior
```

### Lab 2: Creation Failure with Preserved Resources

**Same Template, Different Rollback Option**

```
Step 1: Create Stack Again
├─ File: trigger-failure.yaml (same template)
├─ Stack Name: TriggerCreationFailurePreserve
└─ Template uploaded

Step 2: Select Preservation Option
├─ Stack failure options:
│  ├─ [ ] Roll back all stack resources
│  └─ [✓] Preserve successfully provisioned resources
├─ This is NOT default option
└─ Click: Next, Submit

Step 3: Observe Creation with Preservation
├─ Status: CREATE_IN_PROGRESS
├─ SSHSecurityGroup: Created ✓
├─ ServerSecurityGroup: Created ✓
├─ EC2 Instance: Creation FAILS ✗
└─ Status: CREATE_FAILED

Step 4: Important - Preservation Happened
├─ Resources tab shows:
│  ├─ SSHSecurityGroup: EXISTS (preserved)
│  └─ ServerSecurityGroup: EXISTS (preserved)
├─ Both working security groups remain
├─ No deletions occurred
└─ Stack left with orphaned resources

Step 5: Investigation
├─ Click on each security group
├─ View properties, rules
├─ Understand what was created
├─ Troubleshoot from logs
└─ Now understand the failure better

Step 6: Cleanup Required
├─ NO UPDATE possible with orphaned resources
├─ Cannot fix template and update
├─ MUST DELETE entire stack
├─ Select: Delete Stack
├─ Confirmation: All resources will be deleted
├─ SecurityGroups deleted
├─ Stack deleted
└─ Now clean to retry

CRITICAL POINT:
├─ Preserved resources NOT recoverable
├─ Cannot be fixed and reused
├─ Cannot be updated with new template
├─ MUST be deleted completely
├─ Only for troubleshooting/investigation
└─ Not for production use
```

### Lab 3: Update Failure with Automatic Rollback

**Working Template First, Then Broken Update**

```
Part A: Create Working Stack

Step 1: Create with Template: 0-just-ec2.yaml
├─ Stack Name: FailureOnUpdate
├─ Template creates simple EC2 instance
├─ Stack status: CREATE_COMPLETE
├─ Instance running normally
└─ Status shows: ✓ Active, working

Part B: Attempt Failed Update

Step 2: Update Stack
├─ Select: FailureOnUpdate stack
├─ Click: Update Stack
├─ Choose: "Replace current template"
├─ Upload: trigger-failure.yaml (broken)
├─ Click: Next

Step 3: Stack Update Configuration
├─ Parameter: GroupDescription = "hello"
├─ Stack failure options:
│  ├─ [✓] Roll back all stack resources (DEFAULT)
│  └─ [ ] Preserve successfully provisioned resources
├─ Keep default option
└─ Click: Next, Submit

Step 4: Update Process Beginning
├─ Status: UPDATE_IN_PROGRESS
├─ Events log shows:
│  ├─ SSHSecurityGroup creation initiated
│  ├─ ServerSecurityGroup creation initiated
│  ├─ Both successfully created
│  └─ Now attempting instance update...

Step 5: Failure During Update
├─ Event: EC2 Instance update FAILS
│  └─ Reason: ImageId ami-nonexistent not found
├─ Status changes: UPDATE_FAILED
├─ Automatic rollback triggered
└─ CloudFormation begins reverting changes

Step 6: Automatic Rollback In Progress
├─ Events logged for rollback:
│  ├─ Instance reverted to previous state
│  ├─ SSHSecurityGroup: DELETED
│  ├─ ServerSecurityGroup: DELETED
│  └─ All new changes reversed

Step 7: Rollback Complete
├─ Status: UPDATE_ROLLBACK_COMPLETE
├─ Resources tab:
│  ├─ Shows ONLY original EC2 instance (unchanged)
│  ├─ No security groups (rollback deleted them)
│  └─ Stack exactly as it was before update
├─ Instance: Still running
├─ No orphaned resources
└─ Infrastructure consistent

Outcome:
├─ Update failed gracefully
├─ All changes automatically reverted
├─ Original state restored
├─ No manual cleanup needed
├─ User can retry with fixed template
└─ Professional, automatic failure handling
```

## Part 6: Best Practices and Exam Focus

### Rollback Best Practices

```
✓ Test Templates in Development First
  ├─ Use development account
  ├─ Test creation failures
  ├─ Test update scenarios
  ├─ Verify rollback behavior
  └─ Then deploy to production

✓ Use Default Rollback Option
  ├─ Safest option
  ├─ Automatic cleanup
  ├─ Consistent state
  ├─ No manual intervention
  └─ Production recommended

✓ Only Use Preserve for Debugging
  ├─ Development environment only
  ├─ Troubleshooting specific failures
  ├─ Investigation of template issues
  ├─ Always clean up after
  └─ Do NOT use in production

✓ Monitor Stack Events
  ├─ Watch events during creation/update
  ├─ Understand failure reasons
  ├─ Identify template issues
  ├─ Learn from failures
  └─ Improve templates over time

✓ Never Manually Modify CloudFormation Resources
  ├─ CloudFormation assumes it owns resources
  ├─ Manual changes break assumptions
  ├─ Can cause rollback failures
  ├─ Makes stack unreliable
  └─ Always modify via template

✓ Use Version Control for Templates
  ├─ Git track all templates
  ├─ Easy to revert to working version
  ├─ Document template changes
  ├─ Team collaboration
  └─ Audit trail of infrastructure

✓ Handle Rollback Failures Immediately
  ├─ Cannot use stack while in rollback fail state
  ├─ Must fix root cause
  ├─ ContinueUpdateRollback if needed
  ├─ Eventually delete and recreate
  └─ Don't leave stacks in bad state
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What's default rollback behavior on creation failure?"
A) Resources kept for troubleshooting
B) All resources deleted automatically
C) User must choose
D) Cannot rollback on creation failure

Answer: B (all resources deleted by default)

Q2: "When to use 'Preserve Provisioned Resources'?"
A) Always in production
B) Default for all stacks
C) Troubleshooting development templates
D) Never use this option

Answer: C (only for dev troubleshooting)

Q3: "What happens on stack update failure?"
A) Resources left as-is
B) Only failed resource reverted
C) All changes automatically rolled back
D) Stack deleted completely

Answer: C (all changes reverted automatically)

Q4: "What causes UPDATE_ROLLBACK_FAILED?"
A) Rollback process itself encounters error
B) Cannot delete resources during rollback
C) Manual changes prevent automatic rollback
D) All of above

Answer: D (all are causes)

Q5: "How to recover from UPDATE_ROLLBACK_FAILED?"
A) Delete and recreate stack
B) Fix root cause, then ContinueUpdateRollback
C) Update stack with correct template
D) Reboot the resources

Answer: B (fix issue, then continue rollback)

Q6: "Can preserve resources be updated later?"
A) Yes, update template and reapply
B) No, must delete entire stack
C) Yes, but at reduced functionality
D) No, they become unmanaged

Answer: B (must delete entire stack)

Key Exam Points:
├─ Default rollback deletes all resources
├─ Update failures auto-rollback (good)
├─ Rollback failures require manual fix
├─ Preserve option is for debugging only
├─ ContinueUpdateRollback for recovery
└─ Never manually modify CF resources
```

### Key Concepts Summary

```
✓ Rollbacks ensure infrastructure consistency
  ├─ Automatic vs manual rollback
  ├─ Protects against partial failures
  ├─ Maintains clean state
  └─ Safe default behavior

✓ Creation failures use two options
  ├─ Default: Rollback all (delete all resources)
  ├─ Alternative: Preserve (keep what worked)
  ├─ Preserve for troubleshooting only
  └─ Default for production

✓ Update failures automatically rollback
  ├─ All changes reverted
  ├─ Returns to previous state
  ├─ No downtime for successful resources
  ├─ No manual intervention needed
  └─ Safe update mechanism

✓ Rollback failures are critical
  ├─ Occur when rollback itself fails
  ├─ Usually due to manual resource changes
  ├─ Stack left in inconsistent state
  ├─ Requires manual remediation
  └─ Use ContinueUpdateRollback

✓ Manual resource changes are dangerous
  ├─ Break CloudFormation assumptions
  ├─ Can cause rollback failures
  ├─ Make stacks unreliable
  ├─ Always modify via templates
  └─ Avoid console manual edits

✓ Recovery workflow
  ├─ Identify blocked resources
  ├─ Fix root cause
  ├─ Use ContinueUpdateRollback
  ├─ Retry with correct template
  └─ Maintain version control

✓ Testing is essential
  ├─ Test in development first
  ├─ Verify rollback behavior
  ├─ Understand failure scenarios
  ├─ Confident production deployment
  └─ Learn from failures
```

---

**Total Words: ~8,500**  
**File Created: 10_CloudFormation_Rollbacks.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
