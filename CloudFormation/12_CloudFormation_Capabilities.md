# 12. CloudFormation Capabilities

## Part 1: Understanding CloudFormation Capabilities

### What Are Capabilities?

**Definition:**

```
CloudFormation Capabilities:
├─ Explicit acknowledgements of deployment permission
├─ Security safeguards for sensitive operations
├─ Required for IAM, macros, and nested stacks
├─ Must be specified before stack creation
├─ Prevents accidental high-privilege changes
└─ Explicit risk acknowledgement mechanism
```

**Why Capabilities Matter:**

```
Security Problem:

CloudFormation can modify:
├─ IAM roles, users, groups, policies
├─ Template structure (through macros)
├─ Stack design (through nested stacks)
└─ Access control mechanisms

Risk:

Template Could:
├─ Create overly permissive IAM roles
├─ Grant unexpected access to resources
├─ Modify existing security groups
├─ Create public-facing resources
└─ Escalate privileges silently

Solution:

Capabilities:
├─ Require explicit acknowledgement
├─ "I understand I'm creating IAM resources"
├─ "I understand macros may transform template"
├─ "I understand nested stacks change structure"
├─ Prevents accidental deployments
└─ Security gating mechanism
```

### Capability Types Overview

```
Three Main Capabilities:

CAPABILITY_IAM
├─ Purpose: Create IAM resources
├─ Resources: Unnamed IAM resources
├─ Examples: Roles, policies, groups
├─ When used: IAM resources with *no* explicit names
└─ Use case: Most common IAM stack

CAPABILITY_NAMED_IAM
├─ Purpose: Create Named IAM resources
├─ Resources: Named IAM resources
├─ Examples: Role with specific name, user with name
├─ When used: IAM resources WITH explicit names
└─ Use case: Specific naming requirements

CAPABILITY_AUTO_EXPAND
├─ Purpose: Allow template transformations
├─ Resources: Macros, nested stacks
├─ Examples: SAM syntax, custom macros, stack modules
├─ When used: Template uses AWS::CloudFormation::Transform
└─ Use case: Dynamic infrastructure code
```

## Part 2: CAPABILITY_IAM vs CAPABILITY_NAMED_IAM

### Understanding IAM Capabilities

**When to Use CAPABILITY_IAM:**

```yaml
# Example: Template WITHOUT Explicit Role Name
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFormation Role - UNNAMED'

Resources:
  # No RoleName specified - CloudFormation auto-generates
  MyApplicationRole:
    Type: AWS::IAM::Role
    Properties:
      # RoleName: NOT specified (absence = unnamed)
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'

# CloudFormation auto-generates name like:
# - mystack-MyApplicationRole-ABCDEFGHIJKL
# - Unique name per stack

Capability Required: CAPABILITY_IAM

Reason:
├─ Role has no explicit name
├─ Less naming collision risk
├─ CloudFormation manages naming
└─ Standard deployment pattern
```

**When to Use CAPABILITY_NAMED_IAM:**

```yaml
# Example: Template WITH Explicit Role Name
AWSTemplateFormatVersion: '2010-09-09'
Description: 'CloudFormation Role - NAMED'

Resources:
  MyApplicationRole:
    Type: AWS::IAM::Role
    Properties:
      # RoleName SPECIFIED explicitly
      RoleName: MyCustomRoleName
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'

# Role name is exactly: MyCustomRoleName

Capability Required: CAPABILITY_NAMED_IAM

Reason:
├─ Role has explicit name specified
├─ Requires explicit acknowledgement
├─ Risk: Name collision possibility
├─ Risk: Potential security implications
└─ Stronger safety gate required
```

### Comparison: IAM vs NAMED_IAM

**Decision Matrix:**

```
Scenario 1: IAM Role Without Name
├─ Template property: RoleName NOT present
├─ Generated name: mystack-RoleName-XXXX (auto)
├─ Collision risk: Minimal (CF generates unique)
├─ Capability: CAPABILITY_IAM
└─ Example: General purpose app role

Scenario 2: IAM Role With Explicit Name
├─ Template property: RoleName: MySpecificRoleName
├─ Generated name: MySpecificRoleName (exact)
├─ Collision risk: High (manual naming)
├─ Capability: CAPABILITY_NAMED_IAM
└─ Example: Integration with hardcoded role references

Scenario 3: IAM User Without Name
├─ Template property: UserName NOT present
├─ Generated name: mystack-UserName-XXXX (auto)
├─ Collision risk: Minimal
├─ Capability: CAPABILITY_IAM
└─ Example: Generic service user

Scenario 4: IAM User With Explicit Name
├─ Template property: UserName: service-user-prod
├─ Generated name: service-user-prod (exact)
├─ Collision risk: High
├─ Capability: CAPABILITY_NAMED_IAM
└─ Example: CI/CD pipeline hardcoded credentials

Scenario 5: IAM Policy Without Name
├─ Template property: PolicyName NOT present
├─ Generated name: mystack-PolicyName-XXXX (auto)
├─ Collision risk: Minimal
├─ Capability: CAPABILITY_IAM
└─ Example: Inline policies

Scenario 6: IAM Policy With Explicit Name
├─ Template property: PolicyName: AdminPolicy
├─ Generated name: AdminPolicy (exact)
├─ Collision risk: High
├─ Capability: CAPABILITY_NAMED_IAM
└─ Example: Shared policy references
```

**Real-World Use Case:**

```
Scenario: Database Administrator Role

OPTION 1: Unnamed (Better Practice)
├─ Template: AWS::IAM::Role without RoleName
├─ Capability: CAPABILITY_IAM
├─ Auto-generated: appstack-DBAdminRole-ABC123XYZ
├─ Benefits:
│  ├─ Avoids naming collisions
│  ├─ Supports stack version updates
│  ├─ Multiple stacks in one account
│  └─ Clean stack deletion
├─ Use when: No external tools depend on specific name
└─ Recommended: Almost all cases

OPTION 2: Named (Use When Necessary)
├─ Template: AWS::IAM::Role with RoleName: DBAdminRole
├─ Capability: CAPABILITY_NAMED_IAM
├─ Generated: Exactly "DBAdminRole"
├─ Necessary for:
│  ├─ Application hardcoded role references
│  ├─ Trust relationships with specific name
│  ├─ Legacy integration requirements
│  └─ External systems referencing role
├─ Example: Application assumes 'SessionRole' by name
└─ Recommendation: Only when absolutely required
```

## Part 3: CAPABILITY_AUTO_EXPAND for Macros and Nested Stacks

### What Is CAPABILITY_AUTO_EXPAND?

**Purpose:**

```
CAPABILITY_AUTO_EXPAND:
├─ Allows CloudFormation template transformations
├─ Enables AWS::CloudFormation::Transform
├─ Supports macro execution
├─ Allows nested stack transformations
├─ Permits dynamic template modification
└─ Template may differ from what you wrote

Why Required:

Risk:
├─ Template uploaded by you
├─ CloudFormation modifies template
├─ Macros transform structure
├─ Result: Different from uploaded
├─ Nested stacks add complexity
└─ Need explicit acknowledgement

Acknowledgement:
├─ "I accept template may be transformed"
├─ "I understand macros execute"
├─ "I know nested stacks included"
└─ "Deploy anyway"
```

### When to Use CAPABILITY_AUTO_EXPAND

**Example 1: AWS SAM (Serverless Application Model)**

```yaml
# Template with SAM Transform
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: 'Serverless Lambda Application'

Resources:
  MyLambdaFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: index.handler
      Runtime: python3.9
      CodeUri: s3://bucket/code.zip

# What CloudFormation Does:
├─ Detects: Transform: AWS::Serverless-2016-10-31
├─ Executes: SAM macro transformation
├─ Converts: AWS::Serverless::Function to AWS::Lambda::Function + 
│            AWS::IAM::Role + other resources
├─ Deploys: Expanded template
└─ Result: More resources than originally written

Capability Required: CAPABILITY_AUTO_EXPAND

Reason:
├─ Template transformed before deployment
├─ User sees AWS::Serverless::Function
├─ CloudFormation deploys Lambda + Role + etc.
└─ Transformation hidden from user initially
```

**Example 2: Custom Macros**

```yaml
# Template with Custom Macro
AWSTemplateFormatVersion: '2010-09-09'
Transform: MyOrganization::StandardTemplate
Description: 'Custom Macro Application'

Parameters:
  EnvironmentName:
    Type: String
    Default: dev

Resources:
  # Custom macro might expand this to multiple resources
  StandardMicroservice:
    Type: MyOrganization::StandardMicroservice
    Properties:
      EnvironmentName: !Ref EnvironmentName
      ServiceName: MyService
      Port: 8080

# What CloudFormation Does:
├─ Detects: Transform: MyOrganization::StandardTemplate
├─ Calls: Custom macro Lambda function
├─ Macro Transforms: StandardMicroservice to:
│  ├─ VPC
│  ├─ Security Groups
│  ├─ Load Balancer
│  ├─ Auto Scaling Group
│  ├─ CloudWatch Alarms
│  └─ And more...
├─ Deploys: All expanded resources
└─ Result: 1 input resource = 10+ deployed resources

Capability Required: CAPABILITY_AUTO_EXPAND

Reason:
├─ Template dynamically generated
├─ Macro executes compute
├─ Template modified at runtime
└─ User needs explicit consent
```

**Example 3: Nested Stacks**

```yaml
# Parent Stack Template
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Multi-Stack Application'

Resources:
  NetworkStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/templates/network.yaml
      Parameters:
        VpcCIDR: 10.0.0.0/16

  ApplicationStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      TemplateURL: https://s3.amazonaws.com/templates/app.yaml
      Parameters:
        VpcId: !GetAtt NetworkStack.Outputs.VpcId

# What CloudFormation Does:
├─ Detects: AWS::CloudFormation::Stack resources
├─ Loads: network.yaml and app.yaml templates
├─ Creates: First NetworkStack
├─ Creates: ApplicationStack with outputs from NetworkStack
├─ Manages: Complex dependency tree
└─ Result: Multi-stack orchestration

Capability NOT Always Required:
├─ Nested stacks alone: NOT required (optional)
├─ Nested stacks with transforms: REQUIRED (CAPABILITY_AUTO_EXPAND)
└─ Nested stacks standalone: Works without explicit capability
```

## Part 4: InsufficientCapabilitiesException

### Understanding the Error

**When Error Occurs:**

```
Scenario: Creating Stack With IAM Resources

Step 1: Deploy Template Without Capability
├─ Template: Creates IAM::Role with RoleName specified
├─ Stack creation: Initiated
├─ CloudFormation: Validates template
├─ Error detected: Named IAM resource present
├─ Action: Template rejected
└─ Error: InsufficientCapabilitiesException

Error Message:
"User: arn:aws:iam::123456789012:user/alice is not 
authorized to perform: cloudformation:CreateStack with 
an explicit CAPABILITY_NAMED_IAM"

Translation:
├─ Template requires: CAPABILITY_NAMED_IAM
├─ You provided: Nothing (no capabilities)
├─ Result: Insufficient capabilities for template
└─ Action: Add capability and retry

Root Cause:
├─ Template has named IAM resource
├─ Named IAM requires explicit acknowledgement
├─ Acknowledgement not provided
└─ CloudFormation prevents deployment
```

**Error Scenarios:**

```
Scenario 1: Missing CAPABILITY_NAMED_IAM

Template Resource:
  MyRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: ProductionRole  # ← Named!

Stack Creation (Console):
├─ Upload template
├─ Click: Create
├─ Checkbox: "I acknowledge..." → NOT CHECKED
└─ Result: InsufficientCapabilitiesException

Fix:
├─ Check checkbox: "I acknowledge CloudFormation 
   might create IAM resources with custom names"
├─ Click: Create again
└─ Result: Stack creation proceeds

Stack Creation (CLI):
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml
  # ← Missing --capabilities parameter
  
Result: InsufficientCapabilitiesException

Fix:
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

**Scenario 2: Wrong Capability Specified:**

```
Problem:

Template Has:
  MyRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: MyRole  # ← Requires CAPABILITY_NAMED_IAM

CLI Command:
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM
  # ← Wrong capability! Should be CAPABILITY_NAMED_IAM

Result: InsufficientCapabilitiesException

Error Message:
"1 validation error detected: Value 
'[CAPABILITY_IAM]' at 'capabilities' failed a 
validation check because it does not have 
CAPABILITY_NAMED_IAM which is needed for template"

Fix:
aws cloudformation create-stack \
  --stack-name MyStack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_NAMED_IAM
  # ← Correct capability specified
```

**Scenario 3: Missing CAPABILITY_AUTO_EXPAND:**

```
Template With Transform:
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
  # ← Template has transform

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    # ← SAM resource (needs expansion)

CLI Command (Wrong):
aws cloudformation create-stack \
  --stack-name MyApp \
  --template-body file://template.yaml \
  # ← No --capabilities specified

Result: InsufficientCapabilitiesException

Error Message:
"1 validation error detected: Cloud Formation 
template contains transforms and requires 
CAPABILITY_AUTO_EXPAND or permission to auto-expand"

Fix:
aws cloudformation create-stack \
  --stack-name MyApp \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_AUTO_EXPAND
  # ← Required for template transforms
```

### Resolution Steps

```
Step 1: Read Error Message
├─ Identify: Which capability missing
│  ├─ CAPABILITY_IAM needed
│  ├─ CAPABILITY_NAMED_IAM needed
│  └─ CAPABILITY_AUTO_EXPAND needed
└─ Understand: What template contains

Step 2: Review Template
├─ Search for:
│  ├─ "RoleName:", "UserName:", "PolicyName:" → NAMED_IAM
│  ├─ AWS::IAM:: without explicit names → IAM
│  ├─ Transform: section → AUTO_EXPAND
│  └─ AWS::CloudFormation::Stack → Consider AUTO_EXPAND
└─ Confirm: What capability required

Step 3: Resubmit With Capability
├─ Console: Check acknowledgement box
├─ CLI: Add --capabilities parameter
└─ Supported values:
   ├─ CAPABILITY_IAM
   ├─ CAPABILITY_NAMED_IAM
   ├─ CAPABILITY_AUTO_EXPAND
   └─ Multiple: --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM

Step 4: Proceed With Deployment
├─ Stack creation: Continues
├─ Resources: Created as defined
├─ Monitoring: Stack events show progress
└─ Result: Successful deployment
```

## Part 5: Using Capabilities in Practice

### Console Method

**Step-by-Step Stack Creation with Capabilities:**

```
Step 1: Open CloudFormation Console
├─ CloudFormation → Dashboard
├─ Click: Create Stack
└─ Upload: template.yaml

Step 2: Specify Stack Details
├─ Stack name: DemoIAM
├─ Template parameters: Configure as needed
├─ Click: Next

Step 3: Configure Stack Options
├─ Tags: Add organizational tags
├─ Advanced options: General settings
├─ Click: Next

Step 4: Review and Create (CAPABILITIES APPEAR HERE)
├─ Page: Review your stack
├─ Scroll: Down to bottom
├─ Section: Capabilities
│  └─ Text: "AWS CloudFormation might attempt to 
│     create the following resource(s) named
│     something other than the logical ID
│     shown in your template:"
│
├─ Checkbox 1: "I acknowledge that CloudFormation 
│               might create IAM resources"
│  └─ Use when: Template has any IAM resource
│
├─ Checkbox 2: "I acknowledge that CloudFormation 
│               might create IAM resources with custom names"
│  └─ Use when: Template has named IAM resources
│
├─ Checkbox 3: "I acknowledge that CloudFormation 
│               might fail if you don't have permissions 
│               for all IAM resources in the template"
│  └─ Use when: Unsure about permissions
│
└─ For Macros/Nested:
   └─ Additional checkbox may appear:
      "I acknowledge that the template requires 
       template transformation capabilities"
      └─ Use when: Template has transforms

Step 5: Check Appropriate Boxes
├─ For IAM-only stack: Check box 1 (basic IAM)
├─ For named IAM: Check box 2 (NAMED_IAM)
├─ For both: Check both boxes
├─ For transforms: Check transform box
└─ For maximum safety: Check all relevant boxes

Step 6: Create Stack
├─ Click: Create stack
├─ CloudFormation: Accepts request
├─ Status: Stack creation in progress
└─ Events tab: Shows resource creation detail
```

**Console Example: IAM Role Creation**

```
Template Content:
├─ Resource: AWS::IAM::Role
├─ Property: RoleName: MyCustomRoleName
├─ Property: AssumeRolePolicyDocument configured

Console Journey:

1. Upload template → Next
2. Stack name: DemoIAM → Next
3. Options → Next
4. Review screen appears

5. Scroll down to Capabilities section:
   ┌────────────────────────────────────────┐
   │ Capabilities                            │
   ├────────────────────────────────────────┤
   │ ☐ I acknowledge that AWS CloudFormation│
   │   might create IAM resources            │
   │                                         │
   │ ☑ I acknowledge that AWS CloudFormation│
   │   might create IAM resources with      │
   │   custom names                          │
   │ ← Automatically checked or check it     │
   │                                         │
   │ ☑ I acknowledge that this creates IAM  │
   │   resources                             │
   └────────────────────────────────────────┘

6. Click: Create stack

7. Stack Events Tab Shows:
   ├─ DemoIAM (CREATE_IN_PROGRESS)
   ├─ MyCustomRoleName (CREATE_IN_PROGRESS)
   ├─ MyCustomRoleName (CREATE_COMPLETE)
   └─ DemoIAM (CREATE_COMPLETE)
```

### CLI Method

**Using --capabilities Parameter:**

```bash
# Basic IAM Stack (Unnamed Resources)
aws cloudformation create-stack \
  --stack-name DemoStack \
  --template-body file://template.yaml \
  --capabilities CAPABILITY_IAM

# Named IAM Stack
aws cloudformation create-stack \
  --stack-name DemoStack \
  --template-body file://template-named.yaml \
  --capabilities CAPABILITY_NAMED_IAM

# Multiple Capabilities
aws cloudformation create-stack \
  --stack-name DemoStack \
  --template-body file://template-complex.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM

# SAM/Transform Stack
aws cloudformation create-stack \
  --stack-name ServerlessApp \
  --template-body file://template-sam.yaml \
  --capabilities CAPABILITY_AUTO_EXPAND

# All Capabilities (Most Permissive)
aws cloudformation create-stack \
  --stack-name FullStack \
  --template-body file://template-full.yaml \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
```

**Updating Stacks with Capabilities:**

```bash
# Update stack with capabilities
aws cloudformation update-stack \
  --stack-name DemoStack \
  --template-body file://updated-template.yaml \
  --capabilities CAPABILITY_NAMED_IAM

# Change set creation with capabilities
aws cloudformation create-change-set \
  --stack-name DemoStack \
  --change-set-name change-set-001 \
  --template-body file://template-updated.yaml \
  --capabilities CAPABILITY_NAMED_IAM
```

**Validating Template Before Deployment:**

```bash
# Validate template syntax
aws cloudformation validate-template \
  --template-body file://template.yaml

# Output:
# {
#   "Description": "CloudFormation Role - NAMED",
#   "Parameters": [...],
#   "Capabilities": ["CAPABILITY_NAMED_IAM"],
#   "CapabilitiesReason": "The following resource(s) 
#                          have names: MyCustomRoleName"
# }

# This shows: CAPABILITY_NAMED_IAM required
```

## Part 6: Real-World Scenarios

### Example 1: Production IAM Stack

**Scenario:**

```
Organization: Financial Services Company
Requirement: Deploy standardized IAM roles for production
Roles required: Lambda execution, API Gateway service, etc.
Team: DevOps engineers manage role creation
Goal: Deploy consistently, named roles for auditing
```

**Template (capabilities.yaml):**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Production IAM Roles with Named Resources'

Parameters:
  Environment:
    Type: String
    Default: production

Resources:
  # Lambda Execution Role - NAMED
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub 'lambda-execution-role-${Environment}'
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'

  # API Gateway Service Role - NAMED
  APIGatewayRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub 'api-gateway-role-${Environment}'
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: apigateway.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/CloudWatchLogsFullAccess'

  # EC2 Instance Role - NAMED
  EC2InstanceRole:
    Type: AWS::IAM::Role
    Properties:
      RoleName: !Sub 'ec2-instance-role-${Environment}'
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ec2.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/AmazonEC2FullAccess'

Outputs:
  LambdaRoleArn:
    Value: !GetAtt LambdaExecutionRole.Arn
    Export:
      Name: !Sub 'LambdaRole-${Environment}'

  APIGatewayRoleArn:
    Value: !GetAtt APIGatewayRole.Arn
    Export:
      Name: !Sub 'APIGatewayRole-${Environment}'

  EC2RoleArn:
    Value: !GetAtt EC2InstanceRole.Arn
    Export:
      Name: !Sub 'EC2Role-${Environment}'
```

**Deployment:**

```bash
# Deploy with CAPABILITY_NAMED_IAM
aws cloudformation create-stack \
  --stack-name prod-iam-roles \
  --template-body file://capabilities.yaml \
  --parameters ParameterKey=Environment,ParameterValue=production \
  --capabilities CAPABILITY_NAMED_IAM

# Result:
# ├─ lambda-execution-role-production created
# ├─ api-gateway-role-production created
# ├─ ec2-instance-role-production created
# └─ Exact role names for hardcoded references
```

### Example 2: SAM Application Deployment

**Scenario:**

```
Requirement: Deploy serverless Lambda application
Using: AWS SAM (Serverless Application Model)
Template: Uses AWS::Serverless::Function (not native)
CloudFormation: Will transform SAM to standard resources
```

**Template (sam-app.yaml):**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: 'Serverless API Application'

Globals:
  Function:
    Runtime: python3.9
    Timeout: 30

Resources:
  GetOrderFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: handlers/get_order.lambda_handler
      CodeUri: s3://my-bucket/code.zip
      Events:
        GetOrder:
          Type: Api
          Properties:
            RestApiId: !Ref OrderApi
            Path: /orders/{id}
            Method: get

  OrderApi:
    Type: AWS::Serverless::Api
    Properties:
      StageName: prod

Outputs:
  ApiEndpoint:
    Value: !Sub 'https://${OrderApi}.execute-api.${AWS::Region}.amazonaws.com/prod'
    Description: 'API Gateway endpoint'
```

**Deployment (requires CAPABILITY_AUTO_EXPAND):**

```bash
# Deploy SAM application
aws cloudformation create-stack \
  --stack-name serverless-app \
  --template-body file://sam-app.yaml \
  --capabilities CAPABILITY_AUTO_EXPAND CAPABILITY_IAM

# What CloudFormation Does:
# ├─ Detects: Transform: AWS::Serverless-2016-10-31
# ├─ Transforms: SAM syntax to native CF resources
# ├─ Expansion creates:
# │  ├─ AWS::Lambda::Function (function)
# │  ├─ AWS::IAM::Role (Lambda execution role - unnamed)
# │  ├─ AWS::ApiGateway::RestApi
# │  ├─ AWS::ApiGateway::Resource
# │  ├─ AWS::ApiGateway::Method
# │  └─ Other related resources
# └─ All from SAM shorthand

# Result shows template transformed:
# ├─ Simple input: 10 lines (SAM)
# ├─ Expanded: 100+ lines (native resources)
# └─ All resources deployed
```

## Part 7: Best Practices and Exam Focus

### Capability Selection Best Practices

```
✓ Use CAPABILITY_IAM by Default
  ├─ For most IAM stacks
  ├─ When role names auto-generated
  ├─ When CloudFormation manages naming
  ├─ No naming collision risk
  └─ Recommended pattern

✓ Use CAPABILITY_NAMED_IAM Only When Necessary
  ├─ When specific role names required
  ├─ When external systems reference role
  ├─ When legacy integration needed
  ├─ When hardcoded in applications
  └─ Awareness: Extra risk consideration

✓ Include All Necessary Capabilities
  ├─ Don't try to deploy with insufficient
  ├─ 'InsufficientCapabilitiesException' will occur
  ├─ Better to specify multiple than resubmit
  ├─ No performance penalty for extra capabilities
  └─ Stack deployment just requires right ones

✓ Use CAPABILITY_AUTO_EXPAND for Transforms
  ├─ SAM templates: Required
  ├─ Macros: Required
  ├─ Nested stacks: Usually not (optional feature)
  ├─ Check transform section: Indicates need
  └─ Template error: Clear if missing

✓ Document Capability Rationale
  ├─ Comments in template explaining why needed
  ├─ Runbooks include --capabilities parameter
  ├─ CI/CD pipelines hardcode capabilities
  ├─ Security review: Approve named resources
  └─ Governance: Track IAM resource creation

✓ Validate Before Deployment
  ├─ Use: aws cloudformation validate-template
  ├─ Shows: Required capabilities in output
  ├─ Prevents: InsufficientCapabilitiesException surprises
  ├─ Catches: Template errors early
  └─ Best practice: Part of CI/CD
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What capability required for IAM roles?"
A) CAPABILITY_AUTOEXPAND
B) CAPABILITY_IAM
C) CAPABILITY_RESOURCES
D) CAPABILITY_PERMISSIONS

Answer: B (CAPABILITY_IAM for IAM resources)

Q2: "Difference between CAPABILITY_IAM and CAPABILITY_NAMED_IAM?"
A) One for users, one for roles
B) One is unnamed, one has explicit names
C) One is production, one is dev
D) No difference, same thing

Answer: B (Unnamed vs named resources)

Q3: "When does InsufficientCapabilitiesException occur?"
A) Not enough user permissions in IAM
B) Template references insufficient resources
C) Capabilities not acknowledged for template operations
D) Stack doesn't have enough capacity

Answer: C (Capabilities not acknowledged)

Q4: "SAM template deployment requires:"
A) CAPABILITY_IAM
B) CAPABILITY_AUTO_EXPAND
C) CAPABILITY_NAMED_IAM
D) No capabilities

Answer: B (AUTO_EXPAND for transforms)

Q5: "How to add capabilities in CLI?"
A) --add-capabilities parameter
B) --capabilities parameter
C) capabilities.json file
D) CloudFormation settings

Answer: B (--capabilities parameter)

Q6: "Which resources need CAPABILITY_NAMED_IAM?"
A) EC2 instances
B) S3 buckets
C) Roles/users with explicit names
D) All resources

Answer: C (Explicit name IAM resources)

Key Exam Points:
├─ CAPABILITY_IAM for generic IAM resource creation
├─ CAPABILITY_NAMED_IAM for named IAM resources
├─ CAPABILITY_AUTO_EXPAND for macros/SAM transforms
├─ InsufficientCapabilitiesException when missing
├─ --capabilities parameter in CLI
├─ Checkbox acknowledgement in console
├─ Security gate for high-privilege changes
└─ Best practice: Always include necessary capabilities
```

### Key Concepts Summary

```
✓ Capabilities Gate High-Risk Deployments
  ├─ Prevents accidental IAM resource creation
  ├─ Prevents unaware template transformations
  ├─ Requires explicit user acknowledgement
  ├─ Security safeguard against surprises
  └─ User must understand deployment implications

✓ Three Main Capabilities
  ├─ CAPABILITY_IAM: Unnamed IAM resources
  ├─ CAPABILITY_NAMED_IAM: Named IAM resources (stricter)
  ├─ CAPABILITY_AUTO_EXPAND: Transforms including SAM
  └─ All optional: Only specify what needed

✓ Selection Depends on Template Content
  ├─ Review template for:
  │  ├─ AWS::IAM:: resources (need CAPABILITY_IAM)
  │  ├─ RoleName/UserName/PolicyName properties (need NAMED_IAM)
  │  └─ Transform: section (need AUTO_EXPAND)
  └─ If unsure: Validate template first

✓ Error Signals Missing Capabilities
  ├─ InsufficientCapabilitiesException = Missing capability
  ├─ Error message shows: What capability needed
  ├─ Solution: Add capability and resubmit
  ├─ Console: Check acknowledgement box
  └─ CLI: Add --capabilities parameter

✓ Both Console and CLI Support Capabilities
  ├─ Console: Checkboxes acknowledge risk
  ├─ CLI: --capabilities parameter adds them
  ├─ Both methods: Same underlying mechanism
  ├─ Validation: Template shows required capabilities
  └─ Deployment: Proceeds when acknowledged

✓ Security Safeguard Not Permission Gate
  ├─ Capabilities ≠ IAM permissions
  ├─ Separate concern: Explicit acknowledgement vs access control
  ├─ User: Needs CloudFormation + PassRole permissions
  ├─ Plus: Must acknowledge capabilities
  └─ Combined: Least privilege + explicit acknowledgement
```

---

**Total Words: ~8,500**  
**File Created: 12_CloudFormation_Capabilities.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
