# 30. CloudFormation Troubleshooting

## Overview

Common CloudFormation failure states and their root causes, along with how to resolve them. This is a high-value exam topic covering stack failures, StackSet failures, and region-portability issues.

---

## DELETE_FAILED

Stack deletion fails — CloudFormation cannot remove one or more resources.

### Common Causes and Fixes

| Cause | Fix |
|-------|-----|
| **S3 bucket is not empty** | Empty the bucket manually, or use a Custom Resource with Lambda to automate emptying on delete |
| **Security group is in use** | Detach the security group from all EC2 instances before deleting; cannot delete an SG that is referenced by running instances |
| **Resource has dependencies outside the stack** | Remove the external dependency manually before retrying deletion |
| **Persistent resource you can't delete** | Add `DeletionPolicy: Retain` to skip deletion of that specific resource |

> **Exam tip:** S3 bucket deletion failure due to non-empty bucket is one of the most commonly tested CloudFormation scenarios. The fix is either manual emptying or a Lambda-backed Custom Resource.

**Using DeletionPolicy as an escape hatch:**
```yaml
Resources:
  ProblematicResource:
    Type: AWS::SomeService::SomeResource
    DeletionPolicy: Retain    # CloudFormation skips this resource during deletion
    Properties:
      ...
```

---

## UPDATE_ROLLBACK_FAILED

Stack update failed AND the automatic rollback also failed — stack is now stuck.

### Common Causes

- Resources were **changed outside CloudFormation** (drift) — rollback tries to restore to pre-update state but the resource is now in an unexpected state
- **Insufficient IAM permissions** — CloudFormation doesn't have rights to revert certain resources
- **Auto Scaling Group** not receiving enough signals during rollback
- A resource that no longer exists was expected during rollback

### Resolution

```
Step 1: Check the Events tab — the event log shows which resource caused the failure

Step 2: Manually fix the underlying issue
         (e.g., restore drifted resource, grant IAM permissions, fix ASG)

Step 3: Issue ContinueUpdateRollback API call
         → CloudFormation retries the rollback from where it left off
         → If the error is fixed, rollback completes successfully
```

**CLI command:**
```bash
aws cloudformation continue-update-rollback \
  --stack-name MyStack
```

> You can also skip specific resources during rollback if they are causing the issue:
```bash
aws cloudformation continue-update-rollback \
  --stack-name MyStack \
  --resources-to-skip LogicalResourceId1
```

---

## StackSet OUTDATED Stack Instance

A stack instance shows status `OUTDATED` after a StackSet operation fails.

### Common Causes

| Cause | Detail |
|-------|--------|
| **Insufficient permissions** | Execution role in target account lacks permissions to create the specified resources |
| **Non-unique global resource names** | S3 bucket names are globally unique — the same name used in multiple accounts/regions will fail in all but the first |
| **Missing or broken trust relationship** | Admin account cannot assume the execution role in the target account |
| **Service quota exceeded** | Target account has reached a limit (e.g., EC2 instance count) and cannot provision more |
| **Service not available in region** | Template uses a service that doesn't exist in the target region |

### Resolution Steps

```
1. Check StackSet Operations tab → identify which stack instance(s) failed
2. Check the individual stack's Events tab in the target account/region
3. Identify root cause (permissions, quota, naming conflict, trust)
4. Fix the issue (update IAM role, request quota increase, use dynamic names)
5. Retry the StackSet operation
```

---

## Template Property Errors

**Rule:** You can only set properties in CloudFormation that you can set through the AWS Console.

**Example:** You cannot set `PrivateDnsName` on an EC2 instance in CloudFormation — because you cannot set it in the EC2 console either. AWS assigns it automatically.

**Test yourself:** Before adding a property to a CloudFormation resource, ask: *"Can I set this in the AWS Console manually?"*
- Yes → You can set it in CloudFormation
- No → You cannot set it in CloudFormation

---

## Template Works in One Region but Fails in Another

### Common Causes and Fixes

| Cause | Fix |
|-------|-----|
| **Hardcoded AMI IDs** | AMI IDs are region-specific. Use `Mappings` to map region to correct AMI ID, or use SSM Parameter Store to look up the latest AMI |
| **Service not available in target region** | Check regional service availability before deploying |
| **Hardcoded ARNs** | ARNs contain region. Use `!Sub` with `${AWS::Region}` instead of hardcoding |
| **Hardcoded S3 bucket names** | Bucket names are globally unique. A name used in one region blocks the same name in another. Use `!Sub 'bucket-${AWS::AccountId}-${AWS::Region}'` for unique names |
| **Hardcoded region-specific values** | Replace any literal region strings with `${AWS::Region}` pseudo parameter |

**Correct approach for AMI IDs:**
```yaml
Mappings:
  RegionAMIMap:
    us-east-1:
      AMI: ami-0abcdef1234567890
    eu-west-1:
      AMI: ami-0fedcba9876543210

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionAMIMap, !Ref AWS::Region, AMI]
```

**Correct approach for unique S3 bucket names:**
```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'my-bucket-${AWS::AccountId}-${AWS::Region}'
      # Unique per account AND region
```

---

## Troubleshooting Reference Table

| Problem | Status | Key Causes | Fix |
|---------|--------|-----------|-----|
| Stack delete fails | DELETE_FAILED | Non-empty S3 bucket, SG in use | Empty resource manually or use Custom Resource; add DeletionPolicy: Retain |
| Stack update + rollback both fail | UPDATE_ROLLBACK_FAILED | Drift, IAM permissions, ASG signals | Fix manually → ContinueUpdateRollback API |
| StackSet instance stuck | OUTDATED | Permissions, naming conflicts, quota, trust | Fix root cause → retry StackSet operation |
| Template property invalid | CREATE_FAILED | Property not settable by AWS | Remove property — if you can't set it in console, you can't in CloudFormation |
| Template fails in new region | CREATE_FAILED | Hardcoded AMI/ARN/bucket name | Use Mappings, pseudo parameters, dynamic names |

---

## Best Practices

✓ **Always check the Events tab** — First stop for any CloudFormation failure  
✓ **Use Custom Resource to empty S3 on delete** — Prevents DELETE_FAILED for buckets with data  
✓ **Use DeletionPolicy: Retain as a last resort** — Skips the problematic resource without blocking the rest  
✓ **Use ContinueUpdateRollback — not re-deploy** — Retrying a fresh update on a stuck stack makes things worse  
✓ **Never hardcode AMI IDs or ARNs** — Use Mappings or pseudo parameters for region portability  
✓ **Use dynamic S3 bucket names** — `${AWS::AccountId}-${AWS::Region}` suffix guarantees global uniqueness  
✓ **Check StackSet execution role permissions** — Most StackSet failures are IAM-related  

---

## SysOps Exam Focus

**Q1: "Your CloudFormation stack is in DELETE_FAILED. The S3 bucket could not be deleted. What is the most likely reason?"**
- A) The bucket policy is too restrictive
- B) The S3 bucket is not empty
- C) CloudFormation does not support S3 bucket deletion
- D) The IAM role lacks s3:DeleteBucket permission
- **Answer: B** — CloudFormation cannot delete non-empty S3 buckets

**Q2: "How can you automate S3 bucket emptying during stack deletion?"**
- A) Set DeletionPolicy: Delete on the bucket
- B) Use a Lambda-backed Custom Resource that empties the bucket on the Delete event
- C) Use cfn-init to empty the bucket
- D) Enable S3 Lifecycle policies
- **Answer: B** — Lambda Custom Resource is the correct automation approach

**Q3: "Your stack is in UPDATE_ROLLBACK_FAILED. What is the correct next step?"**
- A) Delete and redeploy the stack
- B) Manually fix the underlying issue, then call ContinueUpdateRollback
- C) Run Detect Drift and revert changes
- D) Delete the failed resources manually and retry the update
- **Answer: B** — Fix root cause, then ContinueUpdateRollback resumes the rollback

**Q4: "A StackSet stack instance is OUTDATED after a failed operation. What is a likely cause?"**
- A) The StackSet template has a syntax error
- B) The target account's execution role lacks permissions to create the specified resources
- C) The StackSet was deployed to too many regions
- D) Drift was detected on the stack instance
- **Answer: B** — Insufficient permissions in the execution role is the most common StackSet failure

**Q5: "Your CloudFormation template works in us-east-1 but fails in eu-west-1. The error says the AMI ID is invalid. What is the cause?"**
- A) AMI IDs are account-specific and must be re-registered
- B) AMI IDs are region-specific; the hardcoded ID doesn't exist in eu-west-1
- C) CloudFormation doesn't support EC2 in eu-west-1
- D) The instance type is not available in eu-west-1
- **Answer: B** — AMI IDs are regional; hardcoded IDs won't work across regions

**Q6: "You want to deploy the same CloudFormation template across multiple regions without S3 bucket name conflicts. What should you use for the bucket name?"**
- A) A hardcoded globally unique name
- B) `!Sub 'bucket-${AWS::AccountId}-${AWS::Region}'`
- C) `!Ref AWS::BucketName`
- D) A random string generated by Lambda
- **Answer: B** — AccountId + Region suffix guarantees uniqueness per account per region

**Q7: "Can you set the PrivateDnsName property on an EC2 instance in CloudFormation?"**
- A) Yes, using !Sub with the VPC DNS name
- B) No — you cannot set properties in CloudFormation that you cannot set manually in the AWS Console
- C) Yes, but only in VPCs with DNS hostnames enabled
- D) Yes, using a Custom Resource
- **Answer: B** — AWS assigns PrivateDnsName automatically; it cannot be set manually or via CloudFormation

---

**File: 30_CloudFormation_Troubleshooting.md**
**Status: SysOps-focused, exam-ready, concise format**
