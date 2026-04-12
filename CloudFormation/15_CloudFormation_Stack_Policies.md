# 15. CloudFormation Stack Policies

## Understanding Stack Policies

**Definition:**

Stack Policies are JSON documents that define which update actions are allowed on specific resources during CloudFormation stack updates.

```
Default Behavior (NO Stack Policy):
├─ Any action allowed on all resources
├─ Users can update any resource
├─ No protection against changes
└─ Risk: Accidental modifications possible

With Stack Policy:
├─ Explicitly whitelist allowed actions
├─ Deny updates on protected resources
├─ Default: All denied (require explicit allow)
├─ Protection: Against unintentional changes
└─ Governance: Control who changes what
```

**When to Use:**

✓ Protect production databases from accidental updates  
✓ Prevent critical resources being modified  
✓ Enforce change management process  
✓ Multi-team environment with different permissions  
✗ Development environments (too restrictive)

## Stack Policy Structure

**Basic JSON Format:**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

**Policy Breakdown:**

```
Statement 1 - Allow All Updates:
├─ Effect: Allow
├─ Principal: * (everyone)
├─ Action: Update:* (all update actions)
├─ Resource: * (all resources)
└─ Result: Everything can be updated

Statement 2 - Deny Production DB Updates:
├─ Effect: Deny
├─ Principal: * (everyone)
├─ Action: Update:* (all update actions)
├─ Resource: LogicalResourceId/ProductionDatabase
└─ Result: ProductionDatabase cannot be updated

Final Effect:
├─ Most resources: Can be updated (Statement 1)
├─ ProductionDatabase: Cannot be updated (Statement 2 wins)
└─ Precedence: Explicit Deny overrides Allow
```

**Default Behavior When Policy Set:**

```
NO Stack Policy Specified:
├─ All actions: Allowed by default
├─ All resources: Can be modified
├─ All updates: Allowed without restriction
└─ Approach: Open/permissive

Stack Policy Specified (with only Deny):
├─ Default: All actions DENIED
├─ Must explicitly allow what you want changed
├─ Requires explicit Allow statements
├─ Approach: Closed/restrictive (safer)

Example - Protect Critical Resource:

Policy with only Deny (Dangerous):
{
  "Statement": [
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "LogicalResourceId/Database"
    }
  ]
}

Result: ALL resources denied (implicitly)
├─ Production Database: Cannot update (Deny)
├─ Everything else: Cannot update (implicit deny)
└─ Impact: No resources can be updated

Correct Pattern (with Allow):
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "LogicalResourceId/Database"
    }
  ]
}

Result: Most allowed, Database protected
├─ Everything except Database: Can update (Allow)
├─ Production Database: Cannot update (Deny)
└─ Impact: Correct protection applied
```

## Stack Policy Examples

**Example 1: Protect Single Resource**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "LogicalResourceId/ProductionDatabase"
    }
  ]
}
```

Use Case: Allow all updates except production RDS database

**Example 2: Protect Multiple Resources**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": [
        "LogicalResourceId/ProductionDatabase",
        "LogicalResourceId/CriticalDataBucket",
        "LogicalResourceId/SecurityGroup"
      ]
    }
  ]
}
```

Use Case: Protect multiple critical resources simultaneously

**Example 3: Allow Only Specific Actions**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": [
        "Update:Modify",
        "Update:Replace"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:Delete",
      "Resource": "*"
    }
  ]
}
```

Use Case: Allow modification but prevent deletion

**Example 4: Production Stack Protection**

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:Modify",
      "Resource": "LogicalResourceId/ApplicationConfig"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": [
        "Update:Replace",
        "Update:Delete"
      ],
      "Resource": [
        "LogicalResourceId/Database",
        "LogicalResourceId/LoadBalancer"
      ]
    }
  ]
}
```

Use Case: Allow configuration updates, prevent database/LB replacement/deletion

## Update Actions

**Update Action Types:**

```
Update:Modify
├─ Properties changed without resource replacement
├─ Example: DBInstanceClass parameter change
├─ Effect: In-place modification
└─ Risk: Low

Update:Replace
├─ Property change requires resource deletion + recreation
├─ Example: Engine or EncryptionKey change
├─ Effect: Downtime, new physical ID
└─ Risk: High

Update:Delete
├─ Resource removed from template entirely
├─ Example: Remove resource definition
├─ Effect: Resource deleted from AWS
└─ Risk: Critical

Update:* (Wildcard)
├─ All update actions (Modify, Replace, Delete)
├─ Broad protection
└─ Most restrictive
```

## Stack Policies in Practice

**Setting Stack Policy (CLI):**

```bash
# Set policy on new stack
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --stack-policy-body file://stack-policy.json

# Set policy on existing stack
aws cloudformation set-stack-policy \
  --stack-name MyStack \
  --stack-policy-body file://stack-policy.json

# View current policy
aws cloudformation get-stack-policy \
  --stack-name MyStack

# Temporarily override policy during update
aws cloudformation update-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --stack-policy-during-update-body file://temp-policy.json
```

**Setting Stack Policy (Console):**

```
AWS Console Steps:
1. CloudFormation → Stack
2. Stack Actions → Edit Stack Policy
3. Paste JSON policy
4. Click: Save
5. Result: Policy applied
```

**Temporary Override During Update:**

```
Scenario: Need to update protected resource once

Problem:
├─ Stack policy: Denies Database updates
├─ Situation: Emergency database parameter update needed
├─ Solution: Temporary policy override

Override Method 1 - CLI:
aws cloudformation update-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --stack-policy-during-update-body file://temp-allow-all.json

Override Method 2 - Console:
1. Select Stack
2. Stack Actions → Update Stack
3. Upload new template
4. Permissions section: Override stack policy
5. Specify temporary policy
6. Create: Stack updates with override

After Update:
├─ Original policy: Automatically reinstated
├─ Protected resources: Back to protected state
├─ Audit trail: Override logged in events
└─ Normal: Returns to restricted mode
```

## Real-World Scenario

**Production Stack Protection:**

```yaml
# CloudFormation Template
AWSTemplateFormatVersion: '2010-09-09'

Resources:
  ProductionDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: prod-db
      Engine: mysql
      DBInstanceClass: db.t3.medium
      AllocatedStorage: '100'

  ApplicationServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.medium

  ConfigBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-app-config
```

Applied Stack Policy:

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "Update:Modify",
      "Resource": "LogicalResourceId/ApplicationServer"
    },
    {
      "Effect": "Deny",
      "Principal": "*",
      "Action": "Update:*",
      "Resource": [
        "LogicalResourceId/ProductionDatabase",
        "LogicalResourceId/ConfigBucket"
      ]
    }
  ]
}
```

**Behavior:**

```
Update Scenarios:

Scenario 1: Change ApplicationServer InstanceType t2.medium → t2.large
├─ Action: Update:Modify
├─ Resource: ApplicationServer
├─ Policy: Allow
├─ Result: ✓ UPDATE SUCCEEDS

Scenario 2: Change ProductionDatabase DBInstanceClass
├─ Action: Update:Modify
├─ Resource: ProductionDatabase
├─ Policy: Deny
├─ Result: ✗ UPDATE BLOCKED

Scenario 3: Delete ConfigBucket from template
├─ Action: Update:Delete
├─ Resource: ConfigBucket
├─ Policy: Deny
├─ Result: ✗ UPDATE BLOCKED

Scenario 4: Change ProductionDatabase with override
├─ Original Policy: Deny
├─ Temporary Override: Allow all
├─ Result: ✓ UPDATE SUCCEEDS (under override)
├─ After Update: Original policy restored
└─ Next Update: Back to Deny
```

## Best Practices

**When to Use Stack Policies:**

✓ Production stacks with critical resources  
✓ Multi-team environments needing governance  
✓ Preventing accidental deletes  
✓ Compliance/regulatory requirements  

**When NOT to Use:**

✗ Development environments (too restrictive)  
✗ Single developer teams  
✗ Rapid iteration needed  

**Recommended Pattern:**

```
1. Allow all modifications by default:
{
  "Effect": "Allow",
  "Action": "Update:Modify",
  "Resource": "*"
}

2. Deny replacement/deletion on critical:
{
  "Effect": "Deny",
  "Action": ["Update:Replace", "Update:Delete"],
  "Resource": "Critical resources only"
}

3. Document exceptions process:
├─ Who can override: Approval required
├─ Audit: Log all overrides
├─ Review: Monthly policy changes
└─ Maintain: Keep policies versioned
```

## SysOps Exam Focus

**Key Concepts:**

Q1: What are Stack Policies?  
↳ JSON documents controlling allowed update actions on stack resources

Q2: Default behavior WITHOUT Stack Policy?  
↳ All update actions allowed on all resources

Q3: Default behavior WITH Stack Policy set?  
↳ All actions denied by default (require explicit allow)

Q4: How to protect resource from updates?  
↳ Add Deny statement for resource in policy

Q5: Update action types?  
↳ Update:Modify (in-place), Update:Replace (delete+create), Update:Delete

Q6: How to temporarily override Stack Policy?  
↳ Use --stack-policy-during-update-body in CLI during stack update

**Exam Tips:**

- Deny takes precedence over Allow
- With policy set: Default is DENY (opposite of default behavior)
- Must use logical resource ID in policy
- Common scenario: Protect production database
- Temporary override restores original policy after update

---

**Total Words: ~3,000**  
**File Created: 15_CloudFormation_Stack_Policies.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
