# 11. CloudFormation Security with IAM Service Roles

## Part 1: Understanding CloudFormation Service Roles

### What Are Service Roles?

**Definition:**

```
CloudFormation Service Roles:
├─ IAM roles dedicated to CloudFormation service
├─ Grant CloudFormation permissions to create resources
├─ Allow CloudFormation to operate on user's behalf
├─ Enable separation of permissions
├─ Implement least privilege principle
└─ User doesn't need direct resource permissions
```

**Traditional vs Service Role Approach:**

```
Traditional Approach (WITHOUT Service Role):

User:
├─ CloudFormation permissions (create/update/delete stack)
├─ EC2 permissions (create/terminate instances)
├─ S3 permissions (create/delete buckets)
├─ RDS permissions (create/delete databases)
├─ Lambda permissions (create/update functions)
└─ ... many more resource permissions

Problem:
├─ User has ALL resource permissions
├─ Violates least privilege principle
├─ User could create resources outside CloudFormation
├─ No separation of concerns
├─ Over-privileged account
└─ Security risk

Service Role Approach (WITH Service Role):

User:
├─ CloudFormation permissions (create/update/delete stack)
├─ IAM PassRole permission
└─ ONLY CloudFormation permissions (no resource perms)

CloudFormation Service Role:
├─ EC2 permissions (create/terminate instances)
├─ S3 permissions (create/delete buckets)
├─ RDS permissions (create/delete databases)
├─ Lambda permissions (create/update functions)
└─ ... only needed resource permissions

Flow:
├─ User creates stack template
├─ User passes service role to CloudFormation
├─ CloudFormation assumes the role
├─ CloudFormation creates resources using role permissions
├─ User never directly touched resources
└─ Least privilege maintained
```

### How Service Roles Work

**Permission Flow:**

```
Step 1: User Creates Stack
├─ User has: CloudFormation permissions + IAM PassRole
├─ User submits: Template + ServiceRole
├─ User: No direct resource permissions needed

Step 2: User Passes Service Role to CloudFormation
├─ User executes PassRole on ServiceRole
├─ CloudFormation receives: ServiceRole ARN
├─ Permission: iam:PassRole

Step 3: CloudFormation Assumes Role
├─ CloudFormation service assumes ServiceRole
├─ Temporary credentials granted
├─ Credentials valid for stack operations

Step 4: CloudFormation Creates Resources
├─ Using ServiceRole permissions
├─ EC2 instance created: ServiceRole has EC2:RunInstances
├─ S3 bucket created: ServiceRole has S3:CreateBucket
├─ Lambda function created: ServiceRole has Lambda:CreateFunction
├─ All using ServiceRole permissions

Step 5: User Monitors Stack
├─ User watches CloudFormation events
├─ Stack status updates
├─ Resources created successfully
└─ All via ServiceRole permissions

Result:
├─ User never directly contacted EC2/S3/Lambda
├─ User only invited CloudFormation (via PassRole)
├─ CloudFormation did the work (via ServiceRole)
├─ Separation of concerns achieved
└─ Least privilege principle maintained
```

## Part 2: IAM PassRole Permission

### What Is PassRole?

**Definition:**

```
IAM PassRole Action:

Permission Name: iam:PassRole
├─ Purpose: Allow entity to pass role to service
├─ Essential for: Service role usage
├─ Why needed: Security control
└─ Restricts: Who can use which roles

How It Works:

Without PassRole:
├─ User can create roles
├─ User CANNOT pass roles to services
├─ User cannot use role for delegated operations
├─ User cannot invoke service role at all
└─ Service role remains locked down

With PassRole:
├─ User can pass role to CloudFormation
├─ CloudFormation can use role
├─ User delegates work to CloudFormation
├─ Service role can be invoked
└─ Controlled delegation achieved
```

### PassRole Policy Example

**Required IAM Policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::ACCOUNT_ID:role/CloudFormationRole"
    }
  ]
}
```

**Policy Breakdown:**

```
Effect: Allow
└─ Explicitly allows the action

Action: iam:PassRole
├─ Service: IAM
├─ Action: PassRole
└─ Ability to pass role

Resource: arn:aws:iam::ACCOUNT_ID:role/CloudFormationRole
├─ For: Specific IAM role
├─ Only this role can be passed
├─ Even more restricted could be:
│  └─ arn:aws:iam::ACCOUNT_ID:role/CloudFormation*
│     (If using role naming convention)

Result:
├─ User can ONLY pass CloudFormationRole
├─ User cannot pass other roles
├─ User cannot create arbitrary roles
└─ Fine-grained security control
```

**Minimal User Policy for CloudFormation:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:CreateStack",
        "cloudformation:UpdateStack",
        "cloudformation:DeleteStack",
        "cloudformation:DescribeStacks",
        "cloudformation:ListStacks"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::ACCOUNT_ID:role/CloudFormationRole"
    }
  ]
}
```

**What This Policy Allows:**

```
User CAN:
├─ Create CloudFormation stacks
├─ Update CloudFormation stacks
├─ Delete CloudFormation stacks
├─ View stack information
├─ List all stacks
├─ Pass CloudFormationRole to CloudFormation
└─ Delegate work to CloudFormation

User CANNOT:
├─ Create EC2 instances directly
├─ Create S3 buckets directly
├─ Create RDS databases directly
├─ Modify roles or permissions
├─ Pass different roles
└─ Use other CloudFormation roles

Result:
├─ User can orchestrate infrastructure via CloudFormation
├─ User cannot create infrastructure directly
├─ User cannot misuse permissions
├─ Least privilege achieved
└─ All infrastructure via CloudFormation (auditable)
```

## Part 3: CloudFormation Service Role

### Creating CloudFormation Service Role

**Role Creation Steps:**

```
Step 1: Create IAM Role
├─ Service: CloudFormation
└─ Trust relationship: CloudFormation service

Trust Policy (Automatic):
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudformation.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

This allows: CloudFormation service to assume role

Step 2: Attach Permissions
├─ Individual resource permissions
├─ OR managed policies
└─ Only what CloudFormation needs

Step 3: Trust the Role
├─ Add trust relationship
├─ Allow CloudFormation to assume it
└─ Already done in step 1

Result:
├─ Role ready for CloudFormation
├─ Has all needed permissions
├─ Trust relationship configured
└─ Can be passed to CloudFormation
```

### CloudFormation Service Role Policy

**Example: S3-Only Role**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

**Use Case:**

```
Role Name: CloudFormationRole-S3
├─ Purpose: CloudFormation can create S3 stacks
├─ Permissions: Full S3 access
├─ When template needs: S3 buckets, lifecycle rules, etc.
└─ CloudFormation uses this role for S3 operations

Template Using S3 Resources:
├─ User creates template with: AWS::S3::Bucket
├─ User invokes: CloudFormation with ServiceRole
├─ CloudFormation: Uses role to create bucket
├─ User: Never touched S3 console
└─ CloudFormation: Fully manages lifecycle
```

**Multi-Service Role:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "s3:*",
        "rds:*",
        "lambda:*"
      ],
      "Resource": "*"
    }
  ]
}
```

**Use Case:**

```
Role Name: CloudFormationRole-MultiService
├─ Purpose: CloudFormation can create multi-tier stacks
├─ Permissions: EC2, S3, RDS, Lambda
├─ When template needs: Multiple service resources
└─ CloudFormation uses this role for all operations

Complex Template:
├─ Resources: VPC, EC2, S3, RDS, Lambda
├─ User submits: Template + MultiServiceRole
├─ CloudFormation: Creates all resources
├─ Using role: For each service operation
└─ User: Only authorized CloudFormation operations
```

## Part 4: Using Service Roles in CloudFormation

### Console Method

**Creating Stack with Service Role:**

```
Step 1: Open CloudFormation Console
├─ CloudFormation → Create Stack
├─ Upload template file
├─ Click: Next

Step 2: Specify Stack Details
├─ Stack name: MyApplicationStack
├─ Template parameters: Fill as needed
├─ Click: Next

Step 3: Configure Stack Options
├─ This is where service role appears
├─ IAM Role dropdown
├─ Options:
│  ├─ [Leave empty] - Use user permissions
│  └─ [Select DemoRole for CFN with S3 capabilities]
├─ Click: Select the role
└─ Continue: Click Next

Step 4: Review and Confirm
├─ Review stack details
├─ Confirm role selection: "DemoRole for CFN with S3..."
├─ Acknowledge: Yes, create stack
└─ Click: Create Stack

Result:
├─ Stack creation begins
├─ CloudFormation assumes service role
├─ Resources created using role permissions
├─ User: Only monitored stack, didn't create resources
└─ All operations: Under CloudFormation + role
```

**Console Location:**

```
AWS CloudFormation Console → Create Stack/Update Stack

Permissions Section:
├─ Field: "Permissions"
├─ Label: "IAM Role (Optional)"
├─ Dropdown: Shows available roles
├─ Default: "[No Role] - Use user permissions"
├─ Selection: Choose CloudFormationRole
└─ Help text: "Specify the IAM role for CloudFormation"
```

### CLI Method

**Creating Stack with Service Role:**

```bash
# CLI Command with Service Role
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --role-arn arn:aws:iam::123456789012:role/CloudFormationRole

# Breakdown:
# --stack-name: Stack identifier
# --template-body: Template file
# --role-arn: Service role ARN (CRITICAL)

# Role ARN format:
# arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME

# Example with actual values:
aws cloudformation create-stack \
  --stack-name production-app \
  --template-body file://app-template.yaml \
  --role-arn arn:aws:iam::123456789012:role/CloudFormationRole-Production

# Without role (uses user permissions):
aws cloudformation create-stack \
  --stack-name test-app \
  --template-body file://app-template.yaml
  # No --role-arn = uses user's IAM permissions
```

**Update Stack with Service Role:**

```bash
aws cloudformation update-stack \
  --stack-name MyStack \
  --template-body file://updated-template.yaml \
  --role-arn arn:aws:iam::123456789012:role/CloudFormationRole

# Same role can be used for multiple operations
# Role remains associated with stack
# All future updates use same role
```

**Delete Stack (Role Not Needed):**

```bash
# Delete doesn't need role parameter
# But uses role established at creation time
aws cloudformation delete-stack \
  --stack-name MyStack

# Role permissions still apply for deletion
# E.g., S3 bucket deletion, EC2 termination
```

## Part 5: Real-World Security Examples

### Example 1: Developer Without Direct Access

**Scenario:**

```
Organization: Tech Company
├─ Developer Team: Alice, Bob, Charlie
├─ Requirement: Deploy apps via CloudFormation
├─ Requirement: No direct resource access to production
└─ Goal: Least privilege security

Current Problem:
├─ Give developers EC2, S3, RDS permissions
├─ Developers could create resources outside CF
├─ Could accidentally/maliciously modify resources
├─ Could bypass infrastructure review
└─ Security risk

Solution: CloudFormation Service Roles
```

**IAM Setup:**

```
Step 1: Create Developer Group
Name: DevelopersCloudFormation

Policy:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "cloudformation:*"
      ],
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": "arn:aws:iam::ACCOUNT_ID:role/CloudFormationRole-Prod"
    }
  ]
}

Permissions:
├─ Full CloudFormation access (create/update/delete)
├─ PassRole on production CF role ONLY
├─ NO direct EC2, S3, RDS permissions
└─ NO other role PassRole permissions

Result:
├─ Alice, Bob, Charlie: All in this group
├─ Can deploy apps via CloudFormation
├─ Cannot create resources directly
├─ Cannot bypass infrastructure review
└─ Forced through CloudFormation+role
```

**CloudFormation Role Setup:**

```
Role Name: CloudFormationRole-Prod

Permissions:
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "elasticloadbalancing:*",
        "autoscaling:*",
        "s3:*",
        "rds:*",
        "cloudwatch:*"
      ],
      "Resource": "*"
    }
  ]
}

Trust Relationship:
{
  "Effect": "Allow",
  "Principal": {
    "Service": "cloudformation.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
}

Result:
├─ CloudFormation can do these things
├─ Users CANNOT do these directly
├─ Only CloudFormation can invoke role
└─ Full resource creation via CF only
```

**Developer Workflow:**

```
Alice wants to deploy application:

1. Alice creates CloudFormation template
   ├─ Specifies EC2, load balancer, RDS, S3
   └─ All resources in template

2. Alice creates stack via console
   ├─ Selects CloudFormationRole-Prod
   ├─ Submits stack
   └─ Alice has PassRole permission (checked)

3. CloudFormation process starts
   ├─ Assumes CloudFormationRole-Prod
   ├─ Gets temporary credentials
   ├─ Creates all resources
   └─ Alice: Monitoring progress only

4. Stack creation complete
   ├─ Alice views outputs
   ├─ Application deployed
   ├─ Alice: Tested via CF outputs
   └─ No direct resource creation

Audit Trail:
├─ CloudFormation events logged (CF did work)
├─ Resource creation logged (CF role did work)
├─ Alice: No direct resource API calls
└─ Governance: All infrastructure via CF
```

### Example 2: Restricting By Environment

**Multi-Environment Setup:**

```
Environments:
├─ Development
├─ Staging
├─ Production

Roles Created:
├─ CloudFormationRole-Dev (full permissions)
├─ CloudFormationRole-Staging (restricted permissions)
└─ CloudFormationRole-Prod (strict permissions)

User Permissions:

Developer User:
├─ Can use CloudFormationRole-Dev (iam:PassRole allowed)
├─ Can use CloudFormationRole-Staging (iam:PassRole allowed)
├─ CANNOT use CloudFormationRole-Prod (iam:PassRole denied)
└─ Result: Cannot deploy to production directly

Security Engineer:
├─ Can use CloudFormationRole-Dev (iam:PassRole allowed)
├─ Can use CloudFormationRole-Staging (iam:PassRole allowed)
├─ Can use CloudFormationRole-Prod (iam:PassRole allowed)
└─ Result: Can deploy to all environments

DevOps User:
├─ Can use CloudFormationRole-Dev (iam:PassRole allowed)
├─ Can use CloudFormationRole-Staging (iam:PassRole allowed)
├─ Can use CloudFormationRole-Prod (iam:PassRole allowed)
└─ Result: Full access for infrastructure management
```

**Role Restrictions:**

```
CloudFormationRole-Dev:
├─ Permissions: Full resource creation
├─ Cost: Not limited
├─ Instance types: All (t2.micro to r5.24xlarge)
├─ Resource deletion: Allowed
└─ Use: Fast iteration, testing

CloudFormationRole-Staging:
├─ Permissions: Resource creation with limits
├─ Instance types: Only t2/t3 (no large instances)
├─ Enhanced monitoring: Mandatory
├─ Resource deletion: Requires review (restricted)
└─ Use: Pre-production validation

CloudFormationRole-Prod:
├─ Permissions: Restricted resource creation
├─ Instance types: Approved production types only
├─ Enhanced monitoring: Mandatory
├─ Multi-AZ: Mandatory
├─ Backups: Mandatory
├─ Resource deletion: Not allowed (policy blocks)
└─ Use: Strict production safety
```

## Part 6: Best Practices and Exam Focus

### Security Best Practices

```
✓ Always Use Service Roles in Production
  ├─ Never give users direct resource permissions
  ├─ Minimize user privilege
  ├─ Enforce CloudFormation-only deployment
  ├─ Audit trail of who deployed what
  └─ Governance enforced

✓ Create Role Per Environment
  ├─ Development: Full permissions
  ├─ Staging: Media permissions
  ├─ Production: Minimal permissions
  ├─ Role restrictions prevent misconfig
  └─ Environment protection

✓ Implement PassRole Restrictions
  ├─ Users can only pass specific roles
  ├─ Restrict by role name pattern
  ├─ Or specific role ARN
  ├─ No arbitrary role passing
  └─ Role delegation controlled

✓ Principle of Least Privilege
  ├─ Role permissions: Only what CloudFormation needs
  ├─ Remove unused permissions
  ├─ Delete unused roles
  ├─ Regularly audit role permissions
  └─ Security hardening ongoing

✓ Use Resource-Level Permissions
  ├─ Restrict role to specific resources
  ├─ Example: S3 buckets with prefix
  ├─ Example: EC2 instances with tags
  ├─ More secure than wildcard
  └─ Production recommendation

✓ Enable CloudFormation Events
  ├─ CloudTrail: Track all CF operations
  ├─ CloudWatch Events: Monitor CF events
  ├─ SNS Notifications: Alert on stack changes
  ├─ Alarms: Monitor stack status
  └─ Observability for governance

✓ Regular Role Reviews
  ├─ Quarterly audit of roles
  ├─ Remove unused permissions
  ├─ Update for new services
  ├─ Version control role definitions
  └─ Security maintenance
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What is a CloudFormation service role?"
A) Role used by users to access CloudFormation
B) IAM role that CloudFormation assumes for resources
C) Role for managing CloudFormation templates
D) Role for CloudFormation monitoring

Answer: B (role CF assumes to create resources)

Q2: "What permission required to pass role to CF?"
A) iam:AssumeRole
B) iam:CreateRole
C) iam:PassRole
D) iam:AttachRole

Answer: C (iam:PassRole for role delegation)

Q3: "Benefit of using service role?"
A) Easier for users to create resources
B) Centralized permission management
C) Least privilege principle enforcement
D) Faster stack creation

Answer: C (least privilege separation)

Q4: "When should you use service role?"
A) Only for multi-user teams
B) Only for production
C) For any CloudFormation deployment
D) Never, use user permissions

Answer: C (best practice for all CF)

Q5: "CloudFormation stack without service role uses:"
A) Default AWS permissions
B) User's IAM permissions
C) Service role automatically
D) Root account permissions

Answer: B (uses user's permissions if no role specified)

Q6: "How to specify service role in CLI?"
A) --user-role parameter
B) --iam-role parameter
C) --role-arn parameter
D) --service-role parameter

Answer: C (--role-arn for role specification)

Key Exam Points:
├─ Service roles separate user and resource permissions
├─ iam:PassRole required from users
├─ CloudFormation assumes role for operations
├─ Least privilege best practice
├─ Optional but recommended for production
└─ CLI and console both support service roles
```

### Key Concepts Summary

```
✓ Service Roles Enable Delegation
  ├─ User doesn't need resource permissions
  ├─ User passes role to CloudFormation
  ├─ CloudFormation uses role for operations
  ├─ Separation of concerns
  └─ Least privilege achieved

✓ IAM PassRole Permission Critical
  ├─ iam:PassRole required from user
  ├─ Permission to delegate to services
  ├─ Controllable: Which roles can be passed
  └─ Essential for service role usage

✓ CloudFormation Service Role Created by Admin
  ├─ Admin creates role with CF permissions
  ├─ Admin establishes trust relationship
  ├─ Admin grants only needed permissions
  ├─ Reusable for multiple stacks
  └─ Central governance point

✓ User-Focused Permissions Minimal
  ├─ User: CloudFormation permissions only
  ├─ User: iam:PassRole to specific roles
  ├─ User: No resource permissions needed
  ├─ User: Cannot directly create resources
  └─ Permission: CloudFormation-mediated only

✓ Role-Focused Permissions Comprehensive
  ├─ Role: All resource creation permissions
  ├─ Role: Specific to template needs
  ├─ Role: Assumable only by CloudFormation
  ├─ Role: Can be restricted by environment
  └─ Permission: Everything CF needs, nothing more

✓ Security Benefits Significant
  ├─ Least privilege enforcement
  ├─ Centralized governance
  ├─ Audit trail maintained
  ├─ Environment separation possible
  └─ Production safety enhanced

✓ Optional but Recommended
  ├─ Technically optional
  ├─ Can deploy without role (risky)
  ├─ Best practice: Always use role
  ├─ Production: Role mandatory
  └─ Governance: User-facing requirement
```

---

**Total Words: ~8,500**  
**File Created: 11_CloudFormation_Security_With_IAM_Roles.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
