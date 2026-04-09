# AMI Management in Production

This note covers governance and compliance strategies for using AMIs in production environments, including tag-based approval workflows, IAM restrictions, and AWS Config compliance monitoring.

## Overview

**Production AMI Governance**
- **Goal**: Ensure only approved AMIs are launched in production
- **Enforcement**: Combination of tags, IAM policies, and AWS Config rules
- **Compliance**: Audit trail and remediation of non-compliant instances
- **Risk Mitigation**: Prevents use of untested or unauthorized images

**Three-Pillar Approach**
1. **Tagging Strategy**: Mark approved AMIs with specific tags
2. **IAM Policy**: Restrict EC2 launches to tagged AMIs
3. **AWS Config**: Monitor and report non-compliant instances

## Part 1: AMI Approval through Tagging

### Tag-Based Approval Strategy

#### Tag Structure
```
Tag Key: Environment
Tag Value: production

Tag Key: Approved
Tag Value: true

Tag Key: Owner
Tag Value: platform-team

Tag Key: ApprovedDate
Tag Value: 2024-04-10

Tag Key: Version
Tag Value: 1.0.0
```

#### What is an Approved AMI?
- **Definition**: AMI with specific tags indicating approval status
- **Tagged AMIs**: Pass governance checks
- **Untagged AMIs**: Not approved, should not be used
- **Explicitly Rejected**: Could have tag `Approved: false`

#### Approval Workflow

**Step 1: AMI Creation**
- Developer/Testing team creates AMI
- No approval tags initially
- Listed as "not approved"

**Step 2: Review Process**
- Security team reviews AMI
- Testing team validates functionality
- Compliance team checks standards

**Step 3: Approval and Tagging**
- Approved: Add tag `Approved: true`
- Add tag `Environment: production`
- Add tag `ApprovedDate: {date}`
- Add tag `Owner: {team}`

**Step 4: Deployment Ready**
- AMI now available for production launches
- Users can only launch if tags present
- Audit trail of approval maintained

#### Tagging Best Practices
- **Standardized Keys**: Consistent naming across organization
- **Immutable Tags**: Don't change tags of approved AMIs
- **Versioning**: Track version numbers in tags
- **Lifecycle**: Track approval dates and expiration
- **Ownership**: Tag with responsible team/person
- **Cost Allocation**: Tags for chargeback and billing

### Implementation: Adding Approval Tags

#### AWS Console Method
1. EC2 Console → AMIs
2. Select AMI
3. Tags tab
4. "Edit tags"
5. Add tags:
   - `Approved: true`
   - `Environment: production`
   - `ApprovedDate: 2024-04-10`
6. Save

#### AWS CLI Method
```bash
# Tag AMI as approved
aws ec2 create-tags \
  --resources ami-0123456789abcdef0 \
  --tags Key=Approved,Value=true \
         Key=Environment,Value=production \
         Key=ApprovedDate,Value=2024-04-10 \
  --region us-east-1

# Verify tags
aws ec2 describe-images \
  --image-ids ami-0123456789abcdef0 \
  --region us-east-1
```

#### Tag Governance
- **Who Can Tag**: Restrict to approval team only
- **Prevent Circumvention**: IAM policy restricts tag modification
- **Audit Trail**: CloudTrail logs tagging operations
- **Approval Process**: Documented before tagging applied

## Part 2: IAM Policy Restriction

### Restricting EC2 Launch to Approved AMIs

#### IAM Policy with Tag Condition

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowLaunchWithApprovedAMI",
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": "arn:aws:ec2:*:*:instance/*",
            "Condition": {
                "StringEquals": {
                    "ec2:ImageType": "machine"
                }
            }
        },
        {
            "Sid": "RestrictRunInstancesWithApprovedAMI",
            "Effect": "Allow",
            "Action": "ec2:RunInstances",
            "Resource": "arn:aws:ec2:*:*:image/*",
            "Condition": {
                "StringEquals": {
                    "ec2:ResourceTag/Approved": "true",
                    "ec2:ResourceTag/Environment": "production"
                }
            }
        },
        {
            "Sid": "AllowOtherEC2Permissions",
            "Effect": "Allow",
            "Action": [
                "ec2:CreateVolume",
                "ec2:CreateNetworkInterface",
                "ec2:CreateTags"
            ],
            "Resource": "*"
        }
    ]
}
```

### How the Policy Works

#### Deny by Default
- Users cannot launch instances without explicit permission
- AMI resource must be in policy

#### Condition-Based Allow
- **Tag Match Required**: 
  - `Approved: true`
  - `Environment: production`
- Both conditions must match
- If either tag missing or different: Launch denied

#### Result
```
Scenario 1: Launch from approved AMI
├─ User attempts: ec2:RunInstances
├─ Check tags: Approved=true, Environment=production
├─ Condition matches: YES
└─ Result: ALLOWED ✓

Scenario 2: Launch from unapproved AMI (no tags)
├─ User attempts: ec2:RunInstances
├─ Check tags: No tags present
├─ Condition matches: NO
└─ Result: DENIED ✗

Scenario 3: Launch from non-production tagged AMI
├─ User attempts: ec2:RunInstances
├─ Check tags: Approved=true, Environment=staging
├─ Condition matches: NO (Environment ≠ production)
└─ Result: DENIED ✗
```

### Implementation Steps

#### Step 1: Create IAM Policy
1. AWS Console → IAM → Policies
2. "Create policy"
3. Paste JSON policy above
4. Review and name: `AllowApprovedAMILaunch`
5. Create policy

#### Step 2: Attach to User/Group/Role
1. IAM → Users (or Groups/Roles)
2. Select target user
3. "Add permissions"
4. Attach policy: `AllowApprovedAMILaunch`
5. Confirm

#### Step 3: Test Policy
```bash
# Test 1: Try launching from approved AMI
aws ec2 run-instances \
  --image-id ami-approved-123 \
  --instance-type t2.micro \
  --region us-east-1
# Result: SUCCESS ✓

# Test 2: Try launching from unapproved AMI
aws ec2 run-instances \
  --image-id ami-unapproved-456 \
  --instance-type t2.micro \
  --region us-east-1
# Result: UnauthorizedOperation ✗
```

### Preventing Policy Circumvention

#### Restrict Tag Modification
```json
{
    "Sid": "DenyUnauthorizedTagModification",
    "Effect": "Deny",
    "Action": [
        "ec2:CreateTags",
        "ec2:DeleteTags"
    ],
    "Resource": "arn:aws:ec2:*:*:image/*",
    "Condition": {
        "StringEquals": {
            "aws:username": "regularUser"
        }
    }
}
```

#### Restrict to Approval Team Only
- Only platform/security team can apply tags
- Regular users cannot modify AMI tags
- IAM policy enforces this restriction

## Part 3: AWS Config Compliance Monitoring

### AWS Config Rule for AMI Compliance

#### Concept
- Config rules evaluate EC2 instances continuously
- Check if instances launched from approved AMIs
- Flag non-compliant instances
- Enable automated remediation

#### Two-Type AMI Scenario

**Compliant Instances**
- Launched from approved AMI
- Has tags: `Approved: true`, `Environment: production`
- Config marks: COMPLIANT ✓
- Allowed to run

**Non-Compliant Instances**
- Launched from unapproved AMI
- Missing approval tags OR wrong tags
- Config marks: NON-COMPLIANT ✗
- Trigger alerts and remediation

### Creating AWS Config Rule

#### Custom Lambda-Based Rule

**Step 1: Create Lambda Function**

```python
import boto3
import json

ec2 = boto3.client('ec2')
config = boto3.client('config')

def lambda_handler(event, context):
    instance_id = event['configuration_item']['resource_id']
    
    # Get instance details
    instances = ec2.describe_instances(
        InstanceIds=[instance_id]
    )
    
    instance = instances['Reservations'][0]['Instances'][0]
    image_id = instance['ImageId']
    
    # Get image tags
    images = ec2.describe_images(ImageIds=[image_id])
    image = images['Images'][0]
    
    # Check for approval tag
    tags = {tag['Key']: tag['Value'] for tag in image.get('Tags', [])}
    
    is_compliant = (
        tags.get('Approved') == 'true' and
        tags.get('Environment') == 'production'
    )
    
    # Report to Config
    config.put_evaluations(
        Evaluations=[{
            'ComplianceResourceType': 'AWS::EC2::Instance',
            'ComplianceResourceId': instance_id,
            'Compliance': {
                'ComplianceType': 'COMPLIANT' if is_compliant else 'NON_COMPLIANT'
            },
            'OrderingTimestamp': event['notificationCreationTime']
        }],
        ResultToken=event['resultToken']
    )
```

**Step 2: Deploy Lambda Function**
1. AWS Lambda Console
2. Create function: `check-approved-ami`
3. Paste code above
4. Execution role: Must have EC2 and Config permissions
5. Deploy

**Step 3: Create Config Rule**
1. AWS Config Console
2. Rules → Create rule
3. Select: Custom Lambda rule
4. Lambda function: `check-approved-ami`
5. Trigger: `CHANGE` (when EC2 instances launch)
6. Create rule

#### Pre-Built Rule: Alternative
- AWS Config has pre-built rules
- Search: `ami-approved-tag-check`
- Simpler than custom Lambda
- May have limited customization

### Config Rule Workflow

```
EC2 Instance Launched
    ↓
Config Rule Triggers
    ↓
Lambda Function Executes
    ├─ Get instance image ID
    ├─ Retrieve image tags
    ├─ Check for approval tags
    └─ Evaluate compliance
    ↓
Report to Config
    ├─ COMPLIANT: Approved AMI ✓
    └─ NON_COMPLIANT: Unapproved AMI ✗
    ↓
Compliance Dashboard
    ├─ Track compliance %
    ├─ List non-compliant instances
    └─ Alert on violations
```

### Monitoring Compliance

#### AWS Config Dashboard
1. AWS Config Console
2. Rules → View rule: `check-approved-ami`
3. **Compliance Summary**:
   - COMPLIANT: X instances
   - NON_COMPLIANT: Y instances
   - Last updated: timestamp

#### Non-Compliant Instance Details
- **Instance ID**: EC2 instance violating rule
- **Image ID**: Unapproved AMI used
- **Launch Time**: When instance was created
- **Responsible User**: From CloudTrail logs
- **Remediation**: Terminate or re-launch

#### Viewing Instance Details
```bash
# List non-compliant instances
aws configservice get-compliance-details-by-config-rule \
  --config-rule-name check-approved-ami \
  --compliance-types NON_COMPLIANT

# Get details of instance
aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef0 \
  --query 'Reservations[0].Instances[0]'
```

## Part 4: Production Governance Flow

### Complete End-to-End Process

```
DEVELOPMENT PHASE
├─ Developer creates AMI
├─ AMI untagged (not approved)
└─ Cannot be launched in production (IAM policy)

REVIEW & APPROVAL PHASE
├─ Security team reviews AMI
├─ Testing team validates
├─ Compliance team approves
└─ Tags applied: Approved=true, Environment=production

PRODUCTION DEPLOYMENT
├─ Users attempt launch
├─ IAM policy checks tags
├─ Tags match conditions
├─ Launch ALLOWED ✓
└─ Instance running on approved AMI

ONGOING COMPLIANCE
├─ Config rule evaluates instance
├─ Instance matches approved AMI
├─ Config marks COMPLIANT ✓
├─ No action needed
└─ Compliance score increases

VIOLATION SCENARIO
├─ User attempts to bypass (launch unapproved AMI)
├─ IAM policy denies launch ✗
├─ If somehow instance launched
├─ Config rule marks NON_COMPLIANT ✗
├─ Alert triggered
├─ Remediation initiated
└─ Instance terminated
```

## Part 5: Practical Implementation Example

### Scenario: Company Production Environment

**Requirements**
- All production EC2 must use approved AMIs
- Approval team: platform-engineering
- Automatic detection of violations
- Compliance audit trail

#### Step 1: AMI Approval Process

**Create Approved AMI**
```bash
# Create AMI
aws ec2 create-image \
  --instance-id i-0123456789abcdef0 \
  --name MyProdAMI-v1.0.0 \
  --description "Production web server AMI"

# Tag as approved
aws ec2 create-tags \
  --resources ami-1234567890abcdef0 \
  --tags \
    Key=Approved,Value=true \
    Key=Environment,Value=production \
    Key=Owner,Value=platform-engineering \
    Key=ApprovedDate,Value=2024-04-10 \
    Key=Version,Value=1.0.0
```

#### Step 2: IAM Policy for Users

**Attach to Developer Group**
```bash
# Attach policy allowing only approved AMI launches
aws iam attach-group-policy \
  --group-name developers \
  --policy-arn arn:aws:iam::123456789012:policy/AllowApprovedAMILaunch
```

#### Step 3: Config Rule Setup

**Create Compliance Rule**
```bash
# Create Config rule
aws configservice put-config-rule \
  --config-rule '{
    "ConfigRuleName": "check-approved-ami",
    "Source": {
      "Owner": "LAMBDA",
      "SourceIdentifier": "arn:aws:lambda:us-east-1:123456789012:function:check-approved-ami"
    },
    "Scope": {
      "ComplianceResourceTypes": ["AWS::EC2::Instance"]
    }
  }'
```

#### Step 4: Verification

**Compliant Launch**
```bash
# Developer launches from approved AMI
aws ec2 run-instances \
  --image-id ami-1234567890abcdef0 \
  --instance-type t2.micro \
  --region us-east-1

# Result: SUCCESS ✓
# Instance launches successfully
# Config marks COMPLIANT
```

**Non-Compliant Attempt**
```bash
# Developer tries unapproved AMI
aws ec2 run-instances \
  --image-id ami-aaaaaaaaaaaaaaaa \
  --instance-type t2.micro \
  --region us-east-1

# Result: Access Denied ✗
# IAM policy blocks launch
# Prevents non-compliant instance
```

## Part 6: Best Practices for Production

### AMI Management
- **Versioning**: Use semantic versioning (1.0.0, 1.0.1, etc.)
- **Documentation**: Maintain changelog of AMI updates
- **Lifecycle**: Archive old AMIs, delete after retention period
- **Security Scanning**: Scan AMIs before approval
- **Regular Updates**: Update base images regularly

### Governance Structure
- **Approval Team**: Dedicated team reviewing AMIs
- **Documented Process**: Clear approval workflow
- **Audit Trail**: Maintain records of approvals
- **Escalation Path**: Process for urgent or exception approvals
- **Regular Review**: Quarterly review of governance policies

### Tagging Strategy
- **Consistency**: Standardized tag keys across organization
- **Completeness**: All approved AMIs tagged identically
- **Automation**: Auto-tag at approval time
- **Enforcement**: IAM policies preventing tag circumvention
- **Rotation**: Archive or tag old versions differently

### Config Monitoring
- **Continuous Evaluation**: Always active for compliance
- **Alerting**: SNS notifications on non-compliance
- **Remediation**: Automatic termination or manual review
- **Reporting**: Regular compliance reports for management
- **Retention**: Keep compliance history for audit

### Cost Optimization
- **Deregister Old AMIs**: Remove unused versions
- **Delete Snapshots**: Free up storage on old AMIs
- **Regional Distribution**: Only distribute AMIs where needed
- **Cleanup Schedule**: Regular removal of non-compliant stopped instances

## Part 7: Key Takeaways for SysOps Associate

### Core Concepts
- **Three-Pillar Approach**: Tags + IAM Policy + Config Rules
- **Tag Strategy**: Mark approved AMIs with specific tags
- **IAM Enforcement**: Only allow launches with correct tags
- **Compliance Monitoring**: Config detects non-compliant instances
- **Governance**: Production-ready AMI management

### Practical Implementation
- **Approval Tags**: `Approved: true`, `Environment: production`
- **IAM Conditions**: Restrict EC2 runs to tagged AMIs
- **Config Rules**: Monitor instance AMI compliance
- **Workflow**: Create → Review → Tag → Deploy → Monitor
- **Remediation**: Detect and address violations

### Exam Focus Points
- **Purpose**: Ensure only approved AMIs used in production
- **Tag-Based**: Tagging system controls approval status
- **IAM Restriction**: Policy-enforced launch restrictions
- **Config Detection**: AWS Config finds compliance violations
- **Combination Required**: All three components work together
- **Security Control**: Prevents unauthorized image deployment
- **Audit Trail**: CloudTrail and Config logs prove compliance
- **Scalability**: Approach works for single account or organization
- **Compliance**: Regulatory requirement enforcement mechanism
- **No Manual Audit**: Automated compliance checking