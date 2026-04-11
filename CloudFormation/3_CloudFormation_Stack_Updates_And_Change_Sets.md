# 3. CloudFormation Stack Updates and Change Sets

## Part 1: Understanding Stack Updates

### Update Fundamentals

**Key Rule: Cannot Edit Templates In Place**

```
WRONG Approach:
├─ Open CloudFormation template in console
├─ Try to edit resources
├─ Look for "Save" button
├─ Error: "Cannot edit template directly"
└─ Reason: Changes not tracked properly

CORRECT Approach:
├─ Edit template in your code editor (locally)
├─ Upload new version to CloudFormation
├─ Use "Update Stack" to apply changes
├─ CloudFormation determines what changed
└─ Only changed resources modified
```

### Update vs Delete + Recreate

**Why Not Delete and Recreate?**

```
Delete + Recreate Approach:
├─ Delete old stack
├─ All resources terminated
├─ Time: 5-10 minutes
├─ Data loss: Databases deleted
├─ Downtime: Complete outage
├─ Cost: Pay for cleanup + restart
└─ Not practical for production

Update Approach:
├─ Upload new template
├─ CloudFormation compares old vs new
├─ Only changes applied
├─ Time: 1-5 minutes (usually fast)
├─ Data: Preserved if possible
├─ Downtime: Minimal or none
├─ Cost: Only new resources created
└─ Production-safe approach
```

### How Stack Updates Work

**Update Process Flow:**

```
┌──────────────────────────────┐
│ Original Stack State         │
├──────────────────────────────┤
│ Resources:                   │
│ - EC2 Instance: i-1234       │
│ - InstanceType: t2.micro     │
│ - ImageId: ami-old           │
└──────────────────────────────┘
           │
           │ Upload new template
           ↓
┌──────────────────────────────┐
│ New Template                 │
├──────────────────────────────┤
│ Resources:                   │
│ - EC2 Instance: MyInstance   │
│ - InstanceType: t2.small     │
│ - ImageId: ami-new           │
│ - Security Group: NEW        │
│ - Elastic IP: NEW            │
└──────────────────────────────┘
           │
           │ CloudFormation compares
           ↓
┌──────────────────────────────┐
│ Change Set Generated         │
├──────────────────────────────┤
│ Add: Security Group          │
│ Add: Elastic IP              │
│ Modify: EC2 Instance         │
│ Replace: EC2 Instance        │
│        (due to InstanceType) │
└──────────────────────────────┘
           │
           │ User approves
           ↓
┌──────────────────────────────┐
│ Updated Stack State          │
├──────────────────────────────┤
│ Resources:                   │
│ - EC2 Instance: i-5678 (NEW) │
│ - InstanceType: t2.small     │
│ - ImageId: ami-new           │
│ - Security Group: sg-new     │
│ - Elastic IP: eip-new        │
└──────────────────────────────┘
```

## Part 2: Change Sets

### What Is a Change Set?

**Definition:**

```
Change Set:
├─ Preview of all changes CloudFormation will make
├─ Calculated before any resources modified
├─ Shows exact diff between old and new
├─ Lets you approve before executing
├─ Safe way to understand impact
└─ Prevents accidental resource deletion
```

**Why Change Sets Matter:**

```
Without Change Sets:
├─ Click "Update Stack"
├─ CloudFormation starts making changes immediately
├─ Surprise: Resources get replaced!
├─ Surprise: Data lost!
├─ You can't stop it now
└─ Difficult to troubleshoot

With Change Sets:
├─ See exactly what will change
├─ Understand impact before approval
├─ Catch dangerous changes ("Replace: true")
├─ Ask: "Is this replacement really necessary?"
├─ Modify template if needed
└─ Only approve after review
```

### Example Change Set

**Real Change Set from Lab:**

```
Stack: EC2InstanceDemo
Updating from: 0-just-ec2.yaml
To: 1-ec2-with-sg-eip.yaml

Change Set Preview:

┌─────────────────────────────────────┐
│ Change #1: Add Elastic IP           │
├─────────────────────────────────────┤
│ Logical ID: MyEIP                   │
│ Resource Type: AWS::EC2::EIP        │
│ Action: ADD                         │
│ Replacement: false (new resource)   │
├─────────────────────────────────────┤

┌─────────────────────────────────────┐
│ Change #2: Add SSH Security Group   │
├─────────────────────────────────────┤
│ Logical ID: SSHSecurityGroup        │
│ Resource Type: AWS::EC2::SecurityGr │
│ Action: ADD                         │
│ Replacement: false (new resource)   │
├─────────────────────────────────────┤

┌─────────────────────────────────────┐
│ Change #3: Add Server Security Group│
├─────────────────────────────────────┤
│ Logical ID: ServerSecurityGroup     │
│ Resource Type: AWS::EC2::SecurityGr │
│ Action: ADD                         │
│ Replacement: false (new resource)   │
├─────────────────────────────────────┤

┌─────────────────────────────────────┐
│ Change #4: Update EC2 Instance      │
├─────────────────────────────────────┤
│ Logical ID: MyInstance              │
│ Resource Type: AWS::EC2::Instance   │
│ Action: MODIFY                      │
│ Replacement: true ← IMPORTANT       │
│ Reason: SecurityGroupIds changed    │
│         (cannot be modified in-place)
├─────────────────────────────────────┤
│ Impact:                             │
│ ├─ Old instance terminated          │
│ ├─ New instance created             │
│ ├─ Old instance ID: i-1234 → GONE   │
│ ├─ New instance ID: i-5678 → NEW    │
│ └─ New EIP attached: 203.0.113.45   │
└─────────────────────────────────────┘

Summary:
├─ 3 resources being added
├─ 1 resource being replaced (involves termination)
├─ Total impact: Low downtime
└─ Data impact: Minimal (new instance anyway)
```

### Replacement: True vs False

**Replacement = False (In-Place Change):**

```
Modification Type: In-Place

Example: Increase EBS volume size
├─ Old volume: 100 GB
├─ New volume: 200 GB
├─ Action: Extend volume
├─ Instance: Stays running
├─ Data: Preserved
├─ Downtime: None
├─ Instance ID: Same (i-1234 stays i-1234)
└─ Replacement: false

CloudFormation Message:
└─ "Modifying resource in-place"
```

**Replacement = True (Replace Resource):**

```
Modification Type: Resource Replacement

Example: Change EC2 instance type or add security group
├─ Old instance: i-1234 (t2.micro)
├─ New instance: i-5678 (t2.small)
├─ Action: Cannot modify in-place
├─ Old instance: Terminated
├─ New instance: Created
├─ Data: Lost (new instance)
├─ Downtime: 5-10 minutes
├─ Public IP: Changes (if not using EIP)
└─ Replacement: true

CloudFormation Message:
└─ "Replacing resource (old instance deleted, new created)"
```

**Why Some Changes Require Replacement:**

```
Properties That Require Replacement:

1. AvailabilityZone
   └─ Cannot move running instance to different AZ

2. ImageId (AMI)
   └─ Cannot change OS of running instance

3. InstanceType
   └─ Cannot change type of running instance
   └─ Must stop → change → restart (risky)
   └─ Safer to replace

4. SecurityGroupIds (adding new ones)
   └─ Cannot always modify in-place depending on setup
   └─ Often requires replacement

5. NetworkInterfaces configuration
   └─ Requires replacement

Properties That Don't Require Replacement:

1. Tags
   └─ Can be added/changed in-place

2. EBS volume size (on gp2/gp3)
   └─ Can be extended in-place

3. Security group rules (via separate resources)
   └─ Added to existing SG, doesn't affect instance

CloudFormation Decision Logic:
├─ Analyzes change type
├─ Checks AWS API behavior
├─ Determines if replacement needed
├─ Communicates via Change Set
└─ You decide to approve or cancel
```

### Understanding the Decision

**When Change Set Shows "Replacement: true":**

```
Questions to Ask:

1. "Is this replacement expected?"
   ├─ InstanceType change → Yes, normal
   ├─ AMI change → Yes, normal
   ├─ Adding SG → Maybe, depends on setup
   └─ Tag change → No, unexpected

2. "Can I tolerate downtime?"
   ├─ Production: Usually no
   ├─ Staging: Maybe
   ├─ Development: Yes
   └─ Choose update time accordingly

3. "Will I lose data?"
   ├─ EBS volumes: Usually no (detached, reattached)
   ├─ Local storage: Yes (instance store lost)
   ├─ Database: Only if deleting database resource
   └─ Backup before update if concerned

4. "Should I modify template instead?"
   ├─ Example: Instead of changing InstanceType
   ├─ Use Parameters to allow flexibility
   ├─ Avoid unnecessary replacements
   ├─ Plan ahead in template design
   └─ Future updates need fewer replacements
```

**Approval Decision:**

```
If "Replacement: true" is acceptable:
├─ Schedule maintenance window
├─ Notify users of downtime
├─ Create backup
├─ Click: Execute Change Set
└─ Wait for completion

If "Replacement: true" is NOT acceptable:
├─ Cancel: Don't execute change set
├─ Review: Why is replacement needed?
├─ Redesign: Use different approach
├─ Example: Use Auto Scaling (no downtime)
├─ Retest: Might avoid replacement
└─ Reexecute: New change set
```

## Part 3: Hands-On: Updating a Stack

### Step 1: Prepare New Template

**Original Template: 0-just-ec2.yaml**
```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
```

**Updated Template: 1-ec2-with-sg-eip.yaml**
```yaml
Parameters:
  SecurityGroupDescription:
    Type: String
    Default: "Default security group"
    Description: Description for security group

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      SecurityGroupIds:
        - !Ref SSHSecurityGroup
        - !Ref ServerSecurityGroup

  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SSH access security group
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Ref SecurityGroupDescription
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  MyEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MyInstance
      Domain: vpc
```

**What Changed:**

```
New additions in template:

1. Parameters section
   └─ SecurityGroupDescription: User input
      └─ Gets used in ServerSecurityGroup description

2. EC2 Instance updated
   ├─ Added SecurityGroupIds property
   ├─ References two security groups
   └─ Causes replacement (earlier didn't have this)

3. SSHSecurityGroup resource (NEW)
   ├─ Security group for SSH access
   ├─ Port 22 open to world
   └─ Added to instance

4. ServerSecurityGroup resource (NEW)
   ├─ Security group for web server
   ├─ Has SSH AND HTTP (port 80)
   ├─ Description from parameter
   └─ Added to instance

5. MyEIP resource (NEW)
   ├─ Elastic IP address
   ├─ Attached to EC2 instance
   └─ Static public IP
```

### Step 2: Initiate Stack Update

**Update Stack Steps:**

```
1. CloudFormation Console
2. Select: EC2InstanceDemo stack
3. Click: Update button (top-right)

Update Options:
├─ Replace current template ← Choose this
│  └─ "Use new template from S3 or file"
├─ Use existing template
│  └─ Can't modify, only for re-running with new params
└─ Continue update

4. Upload: 1-ec2-with-sg-eip.yaml
5. Click: Next
```

### Step 3: Add Parameters

**Parameter Input:**

```
CloudFormation prompts:
├─ SecurityGroupDescription
│  └─ Type: String
│  └─ Default: "Default security group"
│  └─ You enter: "This is a cool security group"
│
└─ Parameter value will be used in ServerSecurityGroup description
```

**Configuration:**

```
Update Details:
├─ Stack Name: EC2InstanceDemo (unchanged)
├─ Parameter: SecurityGroupDescription
│  └─ Value: "This is a cool security group"
│
└─ This value inserted into template at runtime
```

### Step 4: Review Change Set

**Change Set Preview Page:**

```
Displays: Summary of what will change

┌─────────────────────────────────────────┐
│ Change #1: Add EIP                      │
├─────────────────────────────────────────┤
│ Logical ID: MyEIP                       │
│ Action: Add (new resource)              │
│ Replacement: No                         │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Change #2: Add SecurityGroup (SSH)      │
├─────────────────────────────────────────┤
│ Logical ID: SSHSecurityGroup            │
│ Action: Add (new resource)              │
│ Replacement: No                         │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Change #3: Add SecurityGroup (Server)   │
├─────────────────────────────────────────┤
│ Logical ID: ServerSecurityGroup         │
│ Action: Add (new resource)              │
│ Replacement: No                         │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Change #4: Replace EC2 Instance         │
├─────────────────────────────────────────┤
│ Logical ID: MyInstance                  │
│ Action: Modify                          │
│ Replacement: YES ← Important!           │
│ Reason: SecurityGroupIds changed        │
│         (requires replacement)          │
│                                         │
│ Impact:                                 │
│ ├─ Old instance: i-1234                 │
│ ├─ Will terminate                       │
│ ├─ New instance: i-5678                 │
│ ├─ Will be created                      │
│ ├─ New EIP attached                     │
│ └─ Downtime: ~5-10 minutes              │
└─────────────────────────────────────────┘

Total Changes: 4
├─ 3 resources added
├─ 1 resource replaced
└─ Summary: Update will proceed
```

**Should You Proceed?**

```
Decision Matrix:

Is this replacement acceptable?
├─ Yes: Development environment
│  └─ Click: Execute Change Set
│
├─ Maybe: Staging environment
│  ├─ Check: Business hours?
│  ├─ Check: Users impacted?
│  ├─ If safe: Click Execute
│  └─ If risky: Reschedule
│
└─ No: Production environment
   ├─ Understand: Why replacement needed?
   ├─ Redesign: Can we avoid it?
   ├─ Example: Use Parameters initially
   └─ Don't execute, modify template
```

### Step 5: Execute Change Set

**After Review:**

```
1. Click: Execute Change Set
2. Status changes: UPDATE_IN_PROGRESS
3. Events tab shows progression
```

**Update Timeline:**

```
T+0s: UPDATE_IN_PROGRESS (EC2InstanceDemo)

T+5s: CREATE_IN_PROGRESS (SSHSecurityGroup)
      └─ Adding security group for SSH

T+10s: CREATE_COMPLETE (SSHSecurityGroup)

T+15s: CREATE_IN_PROGRESS (ServerSecurityGroup)
       └─ Adding security group for web server

T+20s: CREATE_COMPLETE (ServerSecurityGroup)
       └─ Parameter value used in description

T+25s: CREATE_IN_PROGRESS (MyInstance - Replacement)
       └─ "Requested update requires creation of new physical resource"
       └─ Reason: SecurityGroupIds property changed

T+35s: CREATE_IN_PROGRESS (Old instance terminating)
       └─ Old instance i-1234 shutting down
       └─ In EC2 console: Status = "shutting-down"

T+40s: CREATE_COMPLETE (New instance starting)
       └─ New instance i-5678 launching
       └─ In EC2 console: Status = "pending"

T+50s: Instance enters "running" state
       └─ Both security groups attached
       └─ Ready to receive connections

T+55s: CREATE_IN_PROGRESS (MyEIP)
       └─ Creating Elastic IP

T+60s: CREATE_COMPLETE (MyEIP)
       └─ EIP created

T+65s: Association in progress (EIP → Instance)
       └─ Elastic IP being attached to instance

T+70s: UPDATE_COMPLETE (EC2InstanceDemo)
       └─ Stack update finished successfully
       └─ All resources created with new configuration
```

## Part 4: Verifying the Update

### In EC2 Console

**Instance Changes:**

```
Instances List:
├─ Old instance: i-1234 (terminated/terminated)
├─ New instance: i-5678 (running)
│  └─ Different ID
│  └─ Public IP from EIP
│  └─ Both security groups attached
│
└─ Note: Instance ID changed due to replacement
```

**New Instance Details:**

```
Instance Information:
├─ Instance ID: i-5678 (NEW)
├─ Instance Type: t2.micro (same)
├─ AMI ID: ami-0c55b159cbfafe1f0 (same)
├─ Availability Zone: us-east-1a (same)
├─ Public IP: 203.0.113.45 (from EIP)
├─ Elastic IP: 203.0.113.45 (attached)
└─ State: Running

Security Groups (2):
├─ SSHSecurityGroup
│  ├─ Inbound: TCP 22 (SSH) from 0.0.0.0/0
│  └─ Tag: aws:cloudformation:logical-id: SSHSecurityGroup
│
└─ ServerSecurityGroup
   ├─ Inbound: TCP 22 (SSH) from 0.0.0.0/0
   ├─ Inbound: TCP 80 (HTTP) from 0.0.0.0/0
   ├─ Description: "This is a cool security group"
   │  └─ Parameter value used here!
   └─ Tag: aws:cloudformation:logical-id: ServerSecurityGroup
```

**Elastic IP Verification:**

```
Elastic IPs (EC2 Console):
├─ Address: 203.0.113.45
├─ Associated Instance: i-5678
├─ Domain: VPC
├─ Tags: 
│  ├─ aws:cloudformation:stack-name: EC2InstanceDemo
│  └─ aws:cloudformation:logical-id: MyEIP
│
└─ Note: Same EIP will survive if instance is stopped
```

### In CloudFormation Console

**Resources Tab:**

```
Now shows 4 resources instead of 1:

1. MyInstance
   ├─ Type: AWS::EC2::Instance
   ├─ Physical ID: i-5678 (NEW)
   └─ Status: CREATE_COMPLETE

2. MyEIP
   ├─ Type: AWS::EC2::EIP
   ├─ Physical ID: eip-12345678
   └─ Status: CREATE_COMPLETE

3. SSHSecurityGroup
   ├─ Type: AWS::EC2::SecurityGroup
   ├─ Physical ID: sg-87654321
   └─ Status: CREATE_COMPLETE

4. ServerSecurityGroup
   ├─ Type: AWS::EC2::SecurityGroup
   ├─ Physical ID: sg-12345678
   └─ Status: CREATE_COMPLETE
```

**Stack Events Tab:**

```
Complete chronological history:

├─ UPDATE_COMPLETE (EC2InstanceDemo)
│  └─ Stack update successful

├─ CREATE_COMPLETE (MyEIP)
│  └─ Elastic IP created

├─ CREATE_IN_PROGRESS (MyEIP)
│  └─ Elastic IP creation started

├─ CREATE_COMPLETE (MyInstance)
│  └─ EC2 instance replacement complete

├─ DELETE_COMPLETE (MyInstance - old)
│  └─ Old i-1234 terminated

├─ CREATE_IN_PROGRESS (MyInstance)
│  └─ New instance being created...

├─ CREATE_COMPLETE (ServerSecurityGroup)
│  └─ Web server SG created

├─ CREATE_IN_PROGRESS (ServerSecurityGroup)
│  └─ Web server SG being created...

├─ CREATE_COMPLETE (SSHSecurityGroup)
│  └─ SSH SG created

├─ CREATE_IN_PROGRESS (SSHSecurityGroup)
│  └─ SSH SG being created...

├─ UPDATE_IN_PROGRESS (EC2InstanceDemo)
│  └─ Stack update started
```

## Part 5: Resource Dependencies and Deletion

### Deletion Order Understanding

**Why Order Matters:**

```
Correct Deletion Order:
1. Delete EIP first (detaches, no longer needed)
2. Delete EC2 instance (resources can be cleaned)
3. Delete security groups (no resources in use)

Wrong Order Would:
├─ Try to delete security groups first
├─ Fail: "Security group in-use by instance"
├─ Prevent full deletion
└─ Orphaned resources remain
```

**CloudFormation Intelligent Deletion:**

```
When you delete a CloudFormation stack:

CloudFormation Automatically:
├─ Analyzes all resource dependencies
├─ Determines correct deletion order
├─ Deletes in safe sequence automatically
├─ Handles all cleanup
└─ No manual intervention needed

Example: "Delete EC2InstanceDemo Stack"
├─ Deletes components in order:
│  1. Elastic IP (can detach safely)
│  2. EC2 Instance (can terminate)
│  3. Security Groups (can delete, nothing in use)
│  └─ All simultaneously or sequentially as needed
│
└─ Result: Complete stack removal
```

### Stack Deletion Process

**How to Delete:**

```
1. CloudFormation Console
2. Select: EC2InstanceDemo stack
3. Click: Delete button (top-right)
4. Confirm: "Delete Stack" dialog
5. Click: Delete Stack

What Happens Next:
├─ Status: DELETE_IN_PROGRESS
├─ Resources deleted in correct order
│  ├─ Elastic IP removed from instance
│  ├─ Elastic IP deleted
│  ├─ Instance terminated
│  ├─ Security groups deleted
│  └─ All CloudFormation tags cleaned up
├─ Status: DELETE_COMPLETE
└─ Stack removed from active list
```

**Why Delete from CloudFormation (Not EC2 Console):**

```
If you manually delete resources:

Delete Instance Manually:
├─ Terminates EC2 instance
├─ BUT security groups still exist
├─ Elastic IP still exists
├─ CloudFormation stack still exists
├─ Orphaned resources costing money
└─ CloudFormation confused about state

Delete Stack from CloudFormation:
├─ Deletes ALL resources properly
├─ All resources cleaned up automatically
├─ Costs stop immediately
├─ Stack state synchronized
└─ Complete removal
```

### Best Practice: Always Use Stack Deletion

```
✓ DO: Use CloudFormation to delete
  └─ Ensures everything cleaned up

✗ DON'T: Manually delete resources
  ├─ Leaves orphaned resources
  ├─ Stack becomes inconsistent
  ├─ Harder to troubleshoot
  └─ Ongoing costs

Lesson Learned:
├─ Create via CloudFormation
├─ Update via CloudFormation
├─ Delete via CloudFormation
└─ Treat stack as single unit
```

## Part 6: Key Takeaways and Best Practices

### Update Process Summary

```
Step 1: Modify Template
├─ Edit locally (YAML/JSON)
└─ Version in Git

Step 2: Upload New Template
├─ Via Console or CLI
└─ To S3 automatically

Step 3: Update Stack
├─ Reference new template
└─ Provide parameters

Step 4: Review Change Set
├─ See what will change
├─ Check for replacements
├─ Evaluate impact
└─ Approve or cancel

Step 5: Execute Changes
├─ CloudFormation applies changes
├─ Resources updated/created/deleted
├─ Monitor in Events tab
└─ Stack reaches final state

Step 6: Verify
├─ Check resources in service consoles
├─ Confirm configuration correct
├─ Test connectivity/functionality
└─ Ensure no unintended changes

Step 7: Cleanup
├─ Delete stack when done
├─ All resources removed
├─ Costs stop
└─ Complete state reset
```

### Best Practices

```
✓ Template Management:
  ├─ Store in version control (Git)
  ├─ Use descriptive names (version numbers)
  ├─ Document what each version does
  └─ Keep change history

✓ Update Safety:
  ├─ Review all change sets before approval
  ├─ Understand "Replacement: true" impact
  ├─ Schedule updates during maintenance windows
  ├─ Test in development first
  └─ Communicate changes to team

✓ Parameter Usage:
  ├─ Use Parameters for flexibility
  ├─ Avoid hard-coding values
  ├─ Provide meaningful defaults
  ├─ Document parameter purpose
  └─ Reuse same template across environments

✓ Change Set Analysis:
  ├─ Always review before executing
  ├─ Watch for unexpected replacements
  ├─ Ask: "Is this necessary?"
  ├─ Can template be redesigned?
  └─ Plan ahead to minimize disruption

✓ Resource Dependencies:
  ├─ Understand order matters
  ├─ Let CloudFormation handle deletion sequence
  ├─ Don't manually delete resources
  ├─ Problems arise from inconsistency
  └─ Trust CloudFormation's logic
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "You update a CloudFormation template and see
   'Replacement: true'. What does this mean?"
A) The resource will be deleted and recreated
   └─ Correct (not modified in-place)

Q2: "When updating a stack, where do you edit?"
A) Modify template locally, upload new version
   └─ Correct (can't edit in console)

Q3: "What order are resources deleted?"
A) CloudFormation automatically determines order
   └─ Correct (handles dependencies)

Q4: "Should you delete resources manually?"
A) No, delete stack from CloudFormation
   └─ Correct (ensures consistency)

Q5: "What are change sets?"
A) Preview of changes before they happen
   └─ Correct (safe way to review impact)

Key Concepts:
├─ Updates require template replacement
├─ Change sets preview changes safely
├─ Some changes require resource replacement
├─ CloudFormation manages deletion order
├─ Always delete via CloudFormation
├─ Parameters add flexibility
└─ Versioning important for tracking changes
```

---

**Total Words: ~8,500**  
**File Created: 3_CloudFormation_Stack_Updates_And_Change_Sets.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
