# 26. CloudFormation StackSets — Hands-On (Enable AWS Config)

## Overview

**Use Case:** Deploy AWS Config across multiple regions in the same account (or multiple accounts) using StackSets with self-managed permissions. This ensures all resource configurations are tracked across all target regions from a single operation.

---

## Step 1: Create the StackSet Administration Role

This IAM role lives in the **administrator account** and is assumed by CloudFormation to orchestrate deployments.

**What it does:**
- Assumed by the CloudFormation service
- Has permission to assume `AWSCloudFormationStackSetExecutionRole` in any target account

**Deploy via CloudFormation template:**
```yaml
# AWSCloudFormationStackSetAdministrationRole
Resources:
  AdministrationRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: AWSCloudFormationStackSetAdministrationRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: cloudformation.amazonaws.com
            Action: 'sts:AssumeRole'
      Policies:
        - PolicyName: AssumeExecutionRole
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action: 'sts:AssumeRole'
                Resource: 'arn:aws:iam::*:role/AWSCloudFormationStackSetExecutionRole'
```

---

## Step 2: Create the StackSet Execution Role

This IAM role lives in **each target account** (including the same account if deploying locally). It is assumed by the administration role to actually create resources.

**Parameter required:** Administrator Account ID (to establish the trust relationship)

**What it does:**
- Trusts the administration role in the admin account
- Has `AdministratorAccess` to create any resources in the target account

**Deploy via CloudFormation template (in each target account):**
```yaml
# AWSCloudFormationStackSetExecutionRole
Parameters:
  AdministratorAccountId:
    Type: String
    Description: AWS Account ID of the administrator account

Resources:
  ExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: AWSCloudFormationStackSetExecutionRole
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              AWS: !Sub 'arn:aws:iam::${AdministratorAccountId}:root'
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/AdministratorAccess'
```

> **Note:** If deploying to the same account (single-account setup), deploy this template in the same account using your own Account ID as the parameter.

---

## Step 3: Create the StackSet

Once IAM roles are in place, create the StackSet with your target template.

**StackSet creation settings:**

| Setting | Value | Notes |
|---------|-------|-------|
| **Template** | `enable-aws-config.yaml` | S3 upload or local upload |
| **StackSet Name** | `demo-StackSet` | Descriptive name |
| **Admin Role ARN** | `AWSCloudFormationStackSetAdministrationRole` | Created in Step 1 |
| **Execution Role Name** | `AWSCloudFormationStackSetExecutionRole` | Created in Step 2 |
| **Target Accounts** | Account ID(s) | Or OU IDs if using Organizations |
| **Regions** | `eu-central-1`, `eu-west-1` | Frankfurt + Ireland |
| **Region Concurrency** | Parallel | Deploy to all regions simultaneously |
| **Max Concurrent Accounts** | 1+ | Higher = faster, lower = safer |
| **Failure Tolerance** | 0 | How many stacks can fail before aborting |

---

## Deployment Options Explained

**Max Concurrent Accounts:**
- How many accounts are updated simultaneously
- Set high for speed, low for safety on critical deployments

**Failure Tolerance:**
- How many stack instances can fail before the entire StackSet operation is aborted
- Useful for large rollouts where some failure is acceptable

**Region Concurrency:**
- `Sequential` — deploy one region at a time
- `Parallel` — deploy all regions simultaneously (faster)

---

## Notable Template Features in enable-aws-config.yaml

The AWS Config template used in this demo is a great reference for advanced CloudFormation patterns:

**Parameter grouping via Metadata:**
```yaml
Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: 'Recorder Configuration'
        Parameters:
          - AllSupported
          - IncludeGlobalResourceTypes
```
Groups parameters in the console UI for better UX.

**`AWS::NoValue` pseudo variable:**
```yaml
Properties:
  RecordingGroup:
    ResourceTypes: !If
      - IsAllSupported
      - !Ref AWS::NoValue     # Omits the property entirely when not needed
      - !Ref ResourceTypes
```
`AWS::NoValue` effectively removes the property from the resource — a clean way to make properties optional based on conditions.

**Nested conditions:**
```yaml
Conditions:
  CreateSubscription: !And
    - !Condition CreateTopic
    - !Not
      - !Equals [!Ref NotificationEmail, '']
```
Combines multiple conditions with `And` and `Not` — shows how conditions can be composed.

**`!FindInMap` for user-friendly frequency values:**
```yaml
Mappings:
  FrequencyMap:
    One_Hour:
      Value: One_Hour
    Three_Hours:
      Value: TwentyFour_Hours

# Maps human-friendly parameter choices to actual CloudFormation-accepted values
```

**`DependsOn` for ordering:**
```yaml
  ConfigRecorder:
    Type: AWS::Config::ConfigurationRecorder
    DependsOn: ConfigBucketPolicy     # Must exist before recorder starts
```

**`DeletionPolicy: Retain` on S3:**
```yaml
  ConfigBucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain            # Bucket persists even if stack is deleted
```

---

## StackSet Lifecycle in Console

**Stack Instances tab:** Shows one entry per account + region combination:

```
Account: 123456789012  |  Region: eu-central-1  |  Status: CURRENT
Account: 123456789012  |  Region: eu-west-1      |  Status: CURRENT
```

**Operations tab:** Shows history of create/update/delete operations across all instances.

**Result:** AWS Config is now enabled in Frankfurt AND Ireland via a single StackSet deployment.

---

## Self-Managed Permissions — Setup Summary

```
Step 1: Deploy admin role template in administrator account
         → Creates AWSCloudFormationStackSetAdministrationRole

Step 2: Deploy execution role template in EACH target account
         → Creates AWSCloudFormationStackSetExecutionRole
         → Provide admin account ID as parameter

Step 3: Create StackSet referencing both role names
         → Deploy template to target accounts + regions
```

> This manual setup is only required for **self-managed permissions**. With **AWS Organizations + service-managed permissions**, Steps 1 and 2 are handled automatically.

---

## Best Practices

✓ **Use Organizations + service-managed when possible** — Eliminates manual IAM role setup  
✓ **Use parallel region concurrency** — Speeds up deployments significantly  
✓ **Set failure tolerance appropriately** — Allow some failures for large org rollouts, zero tolerance for critical infra  
✓ **Use `DeletionPolicy: Retain` for data resources** — Config buckets should outlive the stack  
✓ **Reference AWS sample templates** — `enable-aws-config.yaml` is a great CloudFormation pattern reference  
✓ **Validate IAM trust relationships carefully** — Wrong account ID in execution role breaks the entire setup  

---

## SysOps Exam Focus

**Q1: "In self-managed StackSets, the execution role must be deployed in:"**
- A) The administrator account only
- B) Every target account
- C) The AWS Organizations management account
- D) The region where StackSets are deployed
- **Answer: B** — Execution role must exist in each target account

**Q2: "What does AWS::NoValue do in a CloudFormation template?"**
- A) Sets a property to null
- B) Removes the property entirely from the resource definition
- C) Sets the property to an empty string
- D) Skips the entire resource
- **Answer: B** — Effectively omits the property, useful in conditional configurations

**Q3: "You want to deploy a StackSet to 3 regions simultaneously. Which setting controls this?"**
- A) Max Concurrent Accounts
- B) Failure Tolerance
- C) Region Concurrency set to Parallel
- D) Stack Instance Count
- **Answer: C** — Parallel region concurrency deploys to all regions at the same time

**Q4: "Which DeletionPolicy ensures an S3 bucket is kept after the stack is deleted?"**
- A) Delete
- B) Snapshot
- C) Retain
- D) Preserve
- **Answer: C** — `DeletionPolicy: Retain` keeps the resource after stack deletion

**Q5: "What is the trust relationship on the StackSet administration role?"**
- A) Trusted by target account IAM users
- B) Trusted by the CloudFormation service
- C) Trusted by AWS Organizations
- D) Trusted by the execution role
- **Answer: B** — The administration role's AssumeRolePolicyDocument trusts `cloudformation.amazonaws.com`

**Q6: "Failure Tolerance in StackSets refers to:"**
- A) The number of regions that can fail before a rollback
- B) The number of stack instances that can fail before the operation is aborted
- C) The timeout before CloudFormation retries a failed deployment
- D) The maximum number of concurrent accounts
- **Answer: B** — How many stack instance failures are acceptable before aborting the operation

---

## Quick Reference: IAM Role Summary

| Role | Account | Assumed By | Purpose |
|------|---------|-----------|---------|
| `AWSCloudFormationStackSetAdministrationRole` | Administrator | CloudFormation service | Orchestrates StackSet operations |
| `AWSCloudFormationStackSetExecutionRole` | Each target account | Administration role | Actually creates resources in target account |

---

**File: 26_CloudFormation_StackSets_HandsOn.md**
**Status: SysOps-focused, exam-ready, concise format**
