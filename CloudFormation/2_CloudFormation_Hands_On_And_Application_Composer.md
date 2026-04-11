# 2. CloudFormation Hands-On and Application Composer

## Part 1: Getting Started with CloudFormation

### Important: Region Selection

**Why us-east-1 (Northern Virginia)?**

```
CloudFormation templates contain region-specific resources.

Critical Resources:
├─ AMI IDs (Amazon Machine Image IDs)
│  └─ Different AMI ID in each region
│  └─ Same OS name but different IDs per region
│
├─ Example:
│  ├─ Amazon Linux 2023 in us-east-1: ami-0c55b159cbfafe1f0
│  ├─ Amazon Linux 2023 in us-west-2: ami-0eb9d67f46d91a192
│  ├─ Amazon Linux 2023 in eu-west-1: ami-0d71ea30463e0ff8d
│  └─ All same OS, but different IDs
│
└─ Problem if using different region:
   ├─ Template hardcoded for us-east-1 AMI
   ├─ Create stack in us-west-2
   ├─ AMI ID doesn't exist in that region
   └─ Stack creation fails
```

**Region Selection Steps:**

```
1. Top-right corner: Region selector
2. Click: Region dropdown
3. Search: "Northern Virginia"
4. Select: us-east-1
5. Verify: Top-right shows "N. Virginia"
```

**Why This Matters:**
```
Template Mapping:
└─ Template written for: us-east-1
   ├─ All AMI IDs valid in us-east-1
   ├─ All availability zones: us-east-1a, us-east-1b, etc.
   └─ Works perfectly in us-east-1

Try using different region:
└─ Stack creation fails
   ├─ AMI ID doesn't exist there
   ├─ AZ names different
   └─ Resources can't be found

Solution:
├─ Always use us-east-1 for course templates
├─ Or: Update all resource IDs for your region
└─ Later: Learn to use Mappings for multi-region
```

## Part 2: CloudFormation Console Overview

### Navigating to CloudFormation

**Console Path:**
```
1. AWS Console
2. Search: "CloudFormation"
3. Click: CloudFormation service
4. You're now in CloudFormation home
```

### Initial State

**What You See:**
```
CloudFormation Dashboard:
├─ Stacks section (main area)
├─ Current stacks: "0 stacks" (if new account)
├─ Status filter: All, Active, Deleted
├─ Create Stack button (top-right)
└─ Options: Import template, Design template, etc.

Note:
├─ If you've used AWS Elastic Beanstalk
│  └─ You may see some stacks (Beanstalk creates them)
├─ If you've used Amplify
│  └─ More stacks may appear
└─ This is normal and doesn't affect your learning
```

### Creating a Stack Options

**Available Methods:**

```
1. "Create Stack" → Choose template source:
   ├─ Upload template file (local YAML/JSON)
   ├─ Amazon S3 URL (pre-uploaded template)
   ├─ Design in Application Composer
   └─ Use sample template

2. "Create Stack from template body"
   └─ Paste template code directly

3. "Create Stack from Application Composer"
   └─ Visual drag-and-drop interface
```

## Part 3: AWS Application Composer

### What Is Application Composer?

**Definition:**
```
AWS Application Composer:
├─ Visual designer for CloudFormation
├─ See infrastructure as diagrams
├─ Edit infrastructure visually or via code
├─ Real-time synchronization (visual ↔ code)
├─ Useful for understanding templates
└─ Can edit YAML/JSON side-by-side with canvas
```

### Accessing Application Composer

**Method 1: From Sample Template**
```
CloudFormation Console:
1. Click: Create Stack
2. Choose: Use a sample template
3. Select: Template (e.g., "Multi-AZ WordPress")
4. Click: "View in Application Composer"
5. Opens: Visual editor with canvas
```

**Method 2: Direct Link**
```
AWS Console:
1. Search: "Application Composer"
2. Click: AWS Application Composer service
3. Opens: Visual editor directly
```

### Understanding Application Composer Interface

**Layout:**

```
┌─────────────────────────────────────────────────┐
│  AWS Application Composer                        │
├─────────────────────────────────────────────────┤
│ Left Panel: Components     │  Canvas: Visual     │
│ (drag & drop)             │  Diagram             │
│ ├─ Compute               │  ┌──────────────┐    │
│ ├─ Database              │  │ WebServer    │    │
│ ├─ Network               │  └──────────────┘    │
│ ├─ Storage               │  ┌──────────────┐    │
│ └─ ...                   │  │ Database     │    │
│                          │  └──────────────┘    │
├─────────────────────────────────────────────────┤
│ Right Panel: Code Editor (YAML/JSON)            │
│ AWSTemplateFormatVersion: '2010-09-09'          │
│ Resources:                                       │
│   WebServer:                                   │
│     Type: AWS::EC2::Instance                   │
│     Properties:                                 │
│       ImageId: ami-12345678                    │
└─────────────────────────────────────────────────┘
```

### Viewing Templates in Application Composer

**Code View Options:**

```
1. Click: "Template" tab (or icon)
2. Choose format:
   ├─ YAML (recommended, more readable)
   └─ JSON (alternative format)
3. View: Entire template code
4. Edit: Directly in editor
```

**Visual Representation:**

```
Template Code Block (YAML):
Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Web server SG
      ...
  
  LaunchConfig:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
      ImageId: ami-12345678
      ...
  
  WebServerGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
      LaunchConfigurationName: !Ref LaunchConfig
      ...

  DatabaseInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      ...

Vision Representation (Canvas):
┌─────────────────────────────────────────┐
│         Infrastructure Canvas            │
├─────────────────────────────────────────┤
│                                          │
│  ┌──────────────────┐                   │
│  │ WebServerGroup   │                   │
│  │ (3 EC2 instances)│                   │
│  └────────┬─────────┘                   │
│           │                              │
│  ┌────────▼──────────────┐              │
│  │ WebServerSecurityGroup │             │
│  └──────────────────────┘              │
│                                          │
│  ┌──────────────────────────┐           │
│  │ DatabaseInstance (RDS)   │           │
│  └──────────────────────────┘           │
│                                          │
└─────────────────────────────────────────┘

Click any component:
├─ Properties displayed
├─ Configuration visible
└─ Edit inline if needed
```

**Understanding Component Relationships:**

```
When you click component in canvas:

1. Selection highlights component
2. Right panel shows:
   ├─ Resource name (e.g., WebServerGroup)
   ├─ Resource type (e.g., AWS::AutoScaling::AutoScalingGroup)
   ├─ All properties
   └─ References to other resources

3. Example properties shown:
   ├─ LaunchConfigurationName: !Ref LaunchConfig
   │  └─ Indicates: Uses LaunchConfig resource
   ├─ SecurityGroups: [!Ref WebServerSecurityGroup]
   │  └─ Indicates: Associated with SG
   └─ VPCZoneIdentifier: subnet-12345,subnet-67890
      └─ Indicates: In specific VPCs/Subnets
```

**Benefits of Application Composer:**

```
✓ Visual understanding of infrastructure
✓ See how components relate
✓ Identify dependencies visually
✓ Edit YAML code with live preview
✓ Validate syntax in real-time
✓ Generate diagrams for documentation
✓ Understand complex templates
└─ Great for learning CloudFormation
```

## Part 4: YAML vs JSON Templates

### Template Format Options

**YAML Format:**
```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Simple EC2 instance template
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
```

**JSON Format:**
```json
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "Simple EC2 instance template",
  "Resources": {
    "MyInstance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "AvailabilityZone": "us-east-1a",
        "ImageId": "ami-0c55b159cbfafe1f0",
        "InstanceType": "t2.micro"
      }
    }
  }
}
```

### Comparison

```
Feature              YAML                JSON
────────────────────────────────────────────────────
Readability          Excellent           Good
                     Clean, indented     Cluttered with
                                        brackets

File Size            Smaller             Larger (~40%)
Syntax Strictness    Flexible            Strict
Comments             Yes (# comments)    No comments
Learning Curve       Easier              Harder
Industry Adoption    Preferred           Legacy
AWS Examples         Mostly YAML         Some JSON

Whitespace           Important (indent)  Ignored
Quotes               Often optional      Always needed
Colons               Used for pairs      Used for pairs
Commas               Rarely needed       Required often
```

### Which to Use?

```
Recommendation for SysOps:

✓ Use YAML
  ├─ Industry standard
  ├─ More readable
  ├─ Less error-prone
  ├─ Most examples YAML
  └─ Easier in code review

Learning Path:
1. Learn YAML syntax (this course)
2. Understand JSON format
3. Use whichever fits your team
4. Both are interchangeable
```

**Converting Between Formats:**
```
Tools to Convert:
├─ cloudformation-designer (online)
├─ YAML parsers (convert manually)
├─ Application Composer (shows both)
└─ AWS CLI can convert

AWS CLI Example:
aws cloudformation validate-template --template-body file://template.yaml
aws cloudformation validate-template --template-body file://template.json
```

## Part 5: Hands-On Lab - Creating Your First Stack

### Step 1: Prepare Template

**Template File: 0-just-ec2.yaml**

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
```

**What This Does:**
```
Resources section (mandatory):
├─ MyInstance: Unique resource name (your choice)
│  ├─ Type: AWS::EC2::Instance
│  │  └─ Says: Create EC2 instance
│  └─ Properties: Configuration parameters
│     ├─ AvailabilityZone: us-east-1a
│     │  └─ Which AZ to launch in
│     ├─ ImageId: ami-0c55b159cbfafe1f0
│     │  └─ Which AMI (Amazon Linux 2023)
│     └─ InstanceType: t2.micro
│        └─ Instance size/type

Result:
├─ Single t2.micro EC2 instance
├─ In us-east-1a availability zone
├─ Using Amazon Linux 2023 AMI
└─ No security group, no EIP, extremely basic
```

### Step 2: Create Stack via Console

**Navigation:**
```
1. AWS Console → CloudFormation
2. Click: Create Stack (top-right)
3. Choose: Upload a template file
4. Click: Choose file
5. Select: 0-just-ec2.yaml (from your computer)
6. Click: Next
```

**Stack Details Page:**
```
Fill in:
├─ Stack name: EC2InstanceDemo
│  └─ Must be unique in region
│  └─ Only alphanumeric, hyphen, underscore
│
└─ Parameters: (none for this template)

Additional options (for now, skip):
├─ Tags: Can add later
├─ Permissions (IAM role): Not needed yet
├─ Advanced options: Not needed yet
│  ├─ Rollback options
│  ├─ Timeout
│  └─ Notification options
└─ All can be configured later
```

**Review and Create:**
```
Review Page Shows:
├─ Stack name: EC2InstanceDemo
├─ Template: Displays your template code
├─ No parameters
└─ Ready to create

Click: Create Stack
├─ CloudFormation uploads template to S3
├─ Reference created in CloudFormation service
├─ Stack creation begins immediately
└─ You're redirected to stack details
```

### Step 3: Monitor Stack Creation

**Events Tab (Real-Time Updates):**

```
Stack Creation Timeline:

Event 1: EC2InstanceDemo | CREATE_IN_PROGRESS | Resource creation initiated
   └─ Timestamp: 2024-04-12 14:32:15 UTC

Event 2: MyInstance | CREATE_IN_PROGRESS | Resource creation initiated
   └─ Timestamp: 2024-04-12 14:32:15 UTC
   └─ Type: AWS::EC2::Instance

Event 3: MyInstance | CREATE_COMPLETE | Resource creation successful
   └─ Timestamp: 2024-04-12 14:32:45 UTC
   └─ Physical ID: i-0a1b2c3d4e5f6g7h8 (EC2 instance ID)

Event 4: EC2InstanceDemo | CREATE_COMPLETE | Stack creation successful
   └─ Timestamp: 2024-04-12 14:32:46 UTC
   └─ All resources created
```

**Refresh Button:**
```
Click Refresh to see latest events:
├─ Usually takes 30-60 seconds for EC2
├─ CloudFormation polls status
├─ Events appear in real-time
└─ Don't need to refresh constantly
```

**Status Indicators:**
```
Status Colors:

CREATE_IN_PROGRESS:
├─ Yellow/Blue indicator
├─ Resources being created
└─ Not complete yet

CREATE_COMPLETE:
├─ Green indicator
├─ Resource/Stack created successfully
└─ Ready to use

CREATE_FAILED:
├─ Red indicator
├─ Something went wrong
└─ Check error messages

ROLLBACK_IN_PROGRESS:
├─ Yellow indicator
├─ Stack creation failed, cleaning up
└─ Resources being removed

ROLLBACK_COMPLETE:
├─ Red indicator
├─ Failed and cleaned up
└─ No resources remain
```

### Step 4: View Created Resources

**Resources Tab:**

```
Click: Resources tab (at stack details)

Displays:
├─ Logical ID: MyInstance (your resource name)
├─ Physical ID: i-0a1b2c3d4e5f6g7h8 (actual AWS resource)
├─ Resource Type: AWS::EC2::Instance
└─ Resource Status: CREATE_COMPLETE

Click Physical ID:
├─ Opens: EC2 Console for that instance
├─ Shows: Instance details
├─ Verify: Correct configuration applied
└─ Confirm: Created exactly as specified
```

**Verify Instance Configuration:**

```
In EC2 Console, check:

Instance Type:
├─ Verify: t2.micro (matches template)
├─ ✓ Correct

Availability Zone:
├─ Verify: us-east-1a (matches template)
├─ ✓ Correct

AMI ID (Image ID):
├─ Verify: ami-0c55b159cbfafe1f0
├─ ✓ Correct
├─ AMI name: amzn2-ami-hvm-*-x86_64
└─ OS: Amazon Linux 2023

Confirmation:
├─ Instance created exactly as coded
├─ No manual configuration needed
├─ CloudFormation succeeded
└─ Infrastructure as Code working!
```

### Step 5: Understand Automatic Tagging

**CloudFormation Automatic Tags:**

```
CloudFormation automatically tags ALL resources:

Click: Instance → Tags tab

Tags Created by CloudFormation:
├─ aws:cloudformation:stack-name: EC2InstanceDemo
│  └─ Stack name this resource belongs to
│
├─ aws:cloudformation:logical-id: MyInstance
│  └─ Logical ID from template
│
└─ aws:cloudformation:stack-id: arn:aws:cloudformation:...
   └─ Full ARN of the stack

Benefits:
├─ Track which stack created resource
├─ Cost allocation by stack
├─ Identify orphaned resources
├─ Delete all resources by stack
├─ Audit resource ownership
└─ Multi-stack environments clarity
```

**Why This Matters:**

```
Discovery and Management:

Without Automatic Tags:
├─ Manual instance creation
├─ No way to know ownership
├─ Difficult to group resources
├─ Hard to delete related resources
└─ Cost allocation unclear

With CloudFormation Tags:
├─ Query: Show me all resources for "EC2InstanceDemo"
├─ Cost: Calculate stack cost directly
├─ Delete: Remove entire stack → all tagged resources gone
├─ Audit: Find which stack owns what
└─ Multi-team: Separate stacks per team easily
```

**AWS CLI Query Example:**

```bash
# Find all EC2 instances from stack
aws ec2 describe-instances \
  --filters "Name=tag:aws:cloudformation:stack-name,Values=EC2InstanceDemo" \
  --query 'Reservations[].Instances[].InstanceId'

Output:
i-0a1b2c3d4e5f6g7h8

# Get detailed info
aws ec2 describe-instances \
  --filters "Name=tag:aws:cloudformation:stack-name,Values=EC2InstanceDemo" \
  --query 'Reservations[].Instances[].[InstanceId, InstanceType, State.Name]'

Output:
[
  ["i-0a1b2c3d4e5f6g7h8", "t2.micro", "running"]
]
```

## Part 6: Stack Details and Management

### Stack Info Tab

**What You See:**

```
Stack Information Display:

├─ Stack Name: EC2InstanceDemo
├─ Stack ID: arn:aws:cloudformation:us-east-1:123456789012:stack/EC2InstanceDemo/...
├─ Stack Status: CREATE_COMPLETE
├─ Creation Time: 2024-04-12 14:32:15 UTC
├─ Last Updated: N/A (hasn't been updated)
├─ Template Description: (empty, none provided)
├─ Capabilities: (empty, none required)
├─ Tags: (can add stack-level tags here)
└─ Notifications: (none configured)
```

**Key Information:**

```
Stack ID:
├─ Unique identifier for this stack
├─ Format: arn:aws:cloudformation:region:account:stack/name/id
├─ Used in API calls
├─ Remains the same even if stack renamed (not possible)

Stack Status:
├─ CREATE_COMPLETE: Successful creation
├─ UPDATE_COMPLETE: Successful update
├─ DELETE_COMPLETE: Deleted (shows in deleted stacks)
├─ ROLLBACK_COMPLETE: Creation failed, rolled back
└─ Various others for different states

Creation Time:
├─ When stack was created
├─ Track infrastructure age
└─ Useful for audit logs
```

### Events Tab

**Complete Event History:**

```
Shows every event that occurred during stack lifecycle:

Newest at Top:
1. Stack creation complete
2. Resource creation complete
3. Resource creation in progress
4. Stack creation in progress

Useful for:
├─ Troubleshooting creation issues
├─ Understanding step-by-step process
├─ Identifying where failures occurred
├─ Timing information for each step
└─ Error messages if something failed
```

### Outputs Tab

**What It Shows:**

```
Outputs Section:
├─ Returns values from template
├─ None in this basic template
├─ Later templates will have many outputs

Example of Useful Outputs:
├─ LoadBalancerURL: http://my-alb-1234567890.us-east-1.elb.amazonaws.com
├─ DatabaseEndpoint: mydb.c9akciq32.us-east-1.rds.amazonaws.com
├─ EC2InstanceIP: 10.0.1.42
├─ S3BucketName: my-app-bucket-12345678
└─ Helps you find and use created resources
```

### Parameters Tab

**User Inputs:**

```
Parameters Section:
├─ Shows all parameters provided at creation
├─ This template has no parameters
├─ Later: Learn to use parameters

Example Templates' Parameters:
├─ InstanceType: t2.micro (what user selected)
├─ KeyPair: my-key-pair (for SSH access)
├─ Environment: dev (which environment)
└─ Helpful for customizing templates
```

### Template Tab

**View Template Code:**

```
Displays: Exact template that was uploaded

Shows:
├─ Full YAML/JSON code
├─ Can't edit here (must create new stack)
├─ Can copy code for reference
├─ Useful for documentation

To Modify:
├─ Edit template locally
├─ Upload as new stack
│  └─ Option: Update existing stack with new template
└─ CloudFormation manages differences
```

## Part 7: Deleting the Stack

### Why Delete?

```
Cost Management:
├─ t2.micro is free tier
├─ But good practice to delete after testing
├─ Other resources would incur costs
└─ Clean up resources you're done with

Learning Practice:
├─ Understand resource cleanup
├─ Confirm all resources are deleted
├─ Practice full lifecycle
└─ Prepare for AWS account management
```

### How to Delete

**Deletion Steps:**

```
1. CloudFormation Console → Stacks
2. Select: EC2InstanceDemo stack
3. Click: Delete button (top-right)
4. Confirm: "Delete Stack" dialog
5. Click: Delete Stack (confirm)

What Happens:
├─ Stack status: DELETE_IN_PROGRESS
├─ All resources deleted
│  ├─ EC2 instance terminated
│  ├─ Any other resources removed
│  └─ Tags deleted
├─ Stack status: DELETE_COMPLETE
└─ Stack removed from active list
```

**Verification:**

```
After Deletion:

EC2 Console:
├─ Instance no longer visible in running instances
├─ Filter: "Terminated" to see recently deleted instances
└─ Instance shows as terminated

CloudFormation Console:
├─ Stack no longer in active list
├─ Filter: "Deleted" to see past stacks
├─ History preserved for reference
└─ Can't modify deleted stack

Cost Impact:
├─ No more charges for this instance
├─ Good practice for dev environments
└─ Production: Plan deletions carefully
```

**AWS CLI Deletion:**

```bash
# Delete stack via CLI
aws cloudformation delete-stack \
  --stack-name EC2InstanceDemo

# Monitor deletion
aws cloudformation wait stack-delete-complete \
  --stack-name EC2InstanceDemo

# Verify deletion
aws cloudformation describe-stacks \
  --stack-name EC2InstanceDemo
  # Returns error: Stack doesn't exist
```

## Part 8: Key Takeaways and Best Practices

### What You've Learned

```
✓ Region importance for AMI IDs
✓ CloudFormation console navigation
✓ Application Composer for visualization
✓ YAML template format
✓ Creating a simple EC2 instance stack
✓ Monitoring stack creation in real-time
✓ Viewing created resources
✓ Understanding automatic tagging
✓ Stack information and management tabs
✓ Deleting resources via CloudFormation

Next Steps:
├─ Understanding template parameters
├─ Creating more complex templates
├─ Using outputs for resource discovery
├─ Updating existing stacks
├─ Multi-resource templates
└─ Template best practices
```

### Best Practices from This Lab

```
✓ Always use correct region for templates
  └─ Prevents AMI ID and AZ mismatches

✓ Keep stack names meaningful
  └─ Helps identify what stack does

✓ Monitor creation in Events tab
  └─ Understand what CloudFormation is doing

✓ Review created resources
  └─ Confirm configuration is correct

✓ Clean up test stacks
  └─ Good account hygiene, cost management

✓ Use tags for cost allocation
  └─ CloudFormation provides automatic tags

✓ Document your templates
  └─ Add Description field for clarity

✓ Version control your templates
  └─ Store in Git for change tracking
```

### Exam Preparation

```
SysOps Exam: CloudFormation Basics

Likely Questions:
1. "How to provision infrastructure without manual console work?"
   A) Use CloudFormation templates
   └─ Correct

2. "What happens when you delete a CloudFormation stack?"
   A) All resources deleted automatically
   └─ Correct

3. "Where should CloudFormation templates be stored?"
   A) Amazon S3
   └─ Correct (referenced from S3)

4. "What tags does CloudFormation automatically add?"
   A) aws:cloudformation:stack-name, etc.
   └─ Correct (automatic resource tagging)

5. "Which format is recommended for CloudFormation?"
   A) YAML
   └─ More readable, preferred

Key Points to Remember:
├─ Templates required in S3
├─ Stack = collection of resources
├─ Delete stack = delete all resources
├─ Region-specific resources (AMIs, AZs)
├─ Automatic cost allocation via tags
└─ Infrastructure as Code benefits
```

---

**Total Words: ~8,500**  
**File Created: 2_CloudFormation_Hands_On_And_Application_Composer.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
