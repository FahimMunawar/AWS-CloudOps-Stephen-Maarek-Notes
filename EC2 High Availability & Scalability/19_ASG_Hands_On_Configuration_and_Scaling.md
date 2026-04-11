# ASG Hands-On: Configuration and Scaling Operations

Step-by-step practical guide to creating, configuring, and operating Auto Scaling Groups in AWS, including launch template creation, ALB integration, health checks, and manual scaling operations.

---

## Part 1: Pre-Deployment Preparation

### Prerequisites and Environment Setup

**Preparing for ASG Implementation**

```
Pre-Flight Checklist:

Resources to Complete First:

1. Load Balancer (if using):
   ├─ Create: ALB or NLB (before ASG)
   ├─ Listeners: Port 80/443
   ├─ Target Groups: At least one
   └─ Status: Running and configured

2. Security Group:
   ├─ Required: For instance access
   ├─ Rules: Port 80 (HTTP), 443 (HTTPS), 22 (SSH)
   └─ Status: Created and ready

3. Key Pair:
   ├─ Required: For EC2 access
   ├─ Stored: In local ~/.ssh/ directory
   └─ Status: Available for selection

4. IAM Role (Optional):
   ├─ If needed: For instance permissions
   ├─ Example: S3 access, CloudWatch write, etc.
   └─ Status: Created with appropriate policies

5. Networking:
   ├─ VPC: Selected
   ├─ Subnets: Multiple AZs selected
   └─ Status: Ready for multi-AZ deployment

Cleanup Current Environment:

Before ASG Creation:
├─ AWS Console: EC2 → Instances
├─ Existing instances: Terminate all
├─ Reason: Clean slate for ASG
├─ Action: Select all → Terminate instances
└─ Wait: All terminated (status: terminated)

Verification:

Instances Dashboard:
├─ Shows: 0 instances running
├─ Status: Clean to start
└─ Ready: For ASG creation

Load Balancer Status:
├─ Check: ALB still running
├─ Target group: Showing 0 healthy targets
├─ Expected: No instances yet
└─ Ready: To register ASG instances

Architecture Overview:

Planning Phase:

┌─────────────────────────────────────────────┐
│         What We're Building                  │
└──────────────────┬──────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
   Create               Create ASG
 Launch Template    (references template)
        │                     │
        ▼                     ▼
┌─────────────┐         ┌──────────────┐
│ Defines:    │         │ Creates:     │
├─ AMI        │         ├─ Min: 1      │
├─ Type       │         ├─ Max: varies │
├─ SG         │         ├─ Desired: 1  │
├─ Key pair   │         └─ Instances   │
└─ User data  │         from template  │
              │              │
              └──────┬───────┘
                     ▼
            ┌────────────────┐
            │ ALB Target     │
            │ Group          │
            │ (if used)      │
            └────────┬───────┘
                     ▼
            ┌────────────────┐
            │ Users receive  │
            │ balanced       │
            │ requests       │
            └────────────────┘

Hands-On Workflow:

Step 1: Create launch template
    ↓
Step 2: Create ASG (references template)
    ↓
Step 3: Verify first instance creation
    ↓
Step 4: Check target group registration
    ↓
Step 5: Test load balancer
    ↓
Step 6: Scale up (add instances)
    ↓
Step 7: Verify instance registration
    ↓
Step 8: Test load balancing
    ↓
Step 9: Scale down (remove instances)
    ↓
Step 10: Activity history review

Initial Configuration:

Development/Learning Setup:
├─ Min capacity: 1
├─ Max capacity: 2-3
├─ Desired: 1 (scale manually to practice)
├─ No automatic scaling: Focus on manual first
└─ Simple: Easy to manage and test

Production Setup (Later):
├─ Min capacity: 2+ (HA)
├─ Max capacity: Based on budget
├─ Desired: Expected baseline
├─ Auto scaling: Configured and tested
└─ Monitoring: Dashboards and alarms
```

---

## Part 2: Launch Template Creation

### Building the Instance Blueprint

**Creating and Configuring Launch Templates**

```
Launch Template Creation Process:

AWS Console Navigation:

From EC2 Dashboard:
├─ Left sidebar: "Launch Templates"
├─ Button: "Create launch template"
├─ Page: Template creation wizard
└─ Ready: To begin configuration

Step 1: Template Basic Information

Launch Template Name:
├─ Field: "Launch template name"
├─ Example: "my-demo-template"
├─ Purpose: Identify template in ASG
├─ Naming: Descriptive, lowercase, hyphens
└─ Note: Cannot be changed after creation

Description (Optional):
├─ Field: "Template version description"
├─ Example: "Demo template for ASG practice"
├─ Purpose: Document template purpose
├─ Benefit: Help with versioning
└─ Optional: Helpful but not required

Template Details:
├─ Provide guidance: Can check for first time
├─ Help: Show additional details
└─ Skip: For experienced users

Step 2: Application and OS Image (AMI)

Select AMI:

Quick Start Options:
├─ Amazon Linux: Fast, minimal, AWS-optimized
├─ Amazon Linux 2: Newer, more stable
├─ Ubuntu: Popular, large package repository
├─ Windows: If Windows servers needed
└─ Custom: Your own built AMI

Example: Amazon Linux 2

Action: Click "Browse more AMIs"
├─ Category: "Quick Start"
├─ Filter: "Amazon Linux"
├─ Select: "Amazon Linux 2 AMI (HVM), SSD Volume Type"
├─ Architecture: "x86_64" (standard)
└─ Free Tier: Check for free tier eligible

Selecting Version:

Free Tier Eligible:
├─ Marked: "Free tier eligible" badge
├─ Cost: No charge for this AMI
├─ Good: For learning/practice
└─ Production: May want different AMI

Latest Version:
├─ Strategy: Use latest for security patches
├─ But: Requires occasional updates
└─ Tradeoff: Security vs. stability

Specific Version:
├─ Option: Choose exact AMI ID
├─ Use: When consistency critical
└─ Example: ami-0c02fb55f10a31b67

Step 3: Instance Type

Select Instance Type:

From Template:
├─ Dropdown: "Instance type"
├─ Purpose: CPU, memory, network
├─ Example: "t2.micro"
├─ Free Tier: "t2.micro", "t2.small"
└─ Selection: Available in your region

t2.micro (Free Tier):
├─ vCPU: 1 virtual CPU
├─ Memory: 1 GB RAM
├─ Good for: Learning, low-traffic testing
├─ Performance: Burstable (can go faster temporarily)
└─ Cost: Free tier eligible

t2.small (Slightly larger):
├─ vCPU: 1 vCPU
├─ Memory: 2 GB RAM
├─ Good for: Light production
└─ Cost: Low cost

t3.medium (Production option):
├─ vCPU: 2 vCPU
├─ Memory: 4 GB RAM
├─ Good for: Production web servers
└─ Cost: $0.05/hour approximately

Instance Type Override:
├─ Advanced: Multiple types allowed
├─ Use: Flexibility in instance selection
├─ Advanced: Skip for now
└─ Simple: Single type sufficient

Step 4: Key Pair Selection

SSH Key Pair:

What It Is:
├─ Authentication: Public-private key pair
├─ Purpose: Secure shell (SSH) access
├─ Usage: Connect to instance on command line
└─ Security: Only you have private key

Selecting Key Pair:

Find Key Pair:
├─ Dropdown: "Key pair (login)"
├─ List: All available key pairs
├─ Look: "EC2 Tutorial" or similar
├─ Select: Your key pair
└─ Note: Create new if none exist

Example Selection:
├─ Dropdown: EC2 Tutorial
├─ Status: Already created
├─ Stored: In ~/.ssh/EC2\ Tutorial.pem
└─ Ready: Use this key

Creating New Key Pair (if needed):

Option: "Create new key pair"
├─ Name: "my-ec2-key"
├─ Type: RSA or ED25519
├─ Format: .pem (Linux/Mac) or .ppk (Windows PuTTY)
├─ Download: Save immediately
└─ Permissions: chmod 400 on Linux/Mac

Step 5: Network Settings

Security Group Selection:

What It Is:
├─ Firewall: Virtual firewall rules
├─ Purpose: Control inbound/outbound traffic
├─ Scope: Instance level
└─ Required: Must select one

Finding Security Group:

Option 1: Use Existing
├─ Dropdown: "Security group"
├─ List: Existing security groups
├─ Look: "launch-wizard-1" or custom name
├─ Verify: Rules include HTTP (80), SSH (22)
└─ Select: Appropriate security group

Option 2: Create New
├─ Action: "Create security group"
├─ Name: "web-server-sg"
├─ VPC: Your VPC
├─ Rules to add:
│  ├─ HTTP: Port 80, from 0.0.0.0/0
│  ├─ HTTPS: Port 443, from 0.0.0.0/0
│  └─ SSH: Port 22, from your IP
└─ Create: Save and use

Example Security Group Rules:

Rule 1: HTTP (for web server)
├─ Type: HTTP
├─ Protocol: TCP
├─ Port: 80
├─ Source: 0.0.0.0/0 (anywhere)
└─ Purpose: Web traffic

Rule 2: HTTPS (for secure)
├─ Type: HTTPS
├─ Protocol: TCP
├─ Port: 443
├─ Source: 0.0.0.0/0
└─ Purpose: Encrypted web traffic

Rule 3: SSH (for administration)
├─ Type: SSH
├─ Protocol: TCP
├─ Port: 22
├─ Source: Your IP or office IP
└─ Purpose: Remote shell access

Step 6: Storage Configuration

EBS Root Volume:

Default Configuration:
├─ Device name: /dev/xvda
├─ Size: 8 GB (default)
├─ Type: gp2 (general purpose)
├─ Delete on termination: Yes (checked)
└─ Iops: Default for gp2

Adjusting Storage:

Size:
├─ Current: 8 GB
├─ For learning: 8 GB sufficient
├─ For production: May need 20-50 GB
└─ Adjust: If needed for application

Type:
├─ gp2: General purpose (default, good balance)
├─ gp3: Newer general purpose (better value)
├─ io1/io2: High I/O demanding (databases)
├─ st1: Throughput optimized (big data)
└─ Recommendation: gp2 or gp3 for web apps

Delete on Termination:
├─ Checked: Yes (usually correct)
├─ Effect: Volume deleted when instance terminates
├─ Reason: Cleanup, cost savings
└─ Keep: Checked for most cases

Step 7: Advanced Details

Advanced Details Section:

Collapsible: "Advanced details" section
├─ Many options: Most can be left default
├─ Important: User data (see below)
├─ IAM instance profile: If application needs AWS access
└─ Monitoring: Can be left default

IAM Instance Profile:

What It Is:
├─ Permissions: What instance can do in AWS
├─ Example: Read S3, write CloudWatch
├─ Required: If instance accesses AWS services
├─ Optional: If no AWS access needed

Selecting Profile:
├─ Dropdown: "IAM instance profile"
├─ List: Available profiles
├─ None: Can leave empty for learning
└─ Select: If application needs one

Example Use:
├─ Profile: EC2-S3-ReadOnly
├─ Effect: Instance can read S3 buckets
└─ Use: For pulling configuration or app code

User Data Script:

What It Is:
├─ Script: Runs at instance startup
├─ Language: Bash (#!/bin/bash) for Linux
├─ Timing: Once per instance launch
├─ Purpose: Initialize, install software, configure
└─ Key: Must work without interaction

Example User Data (Web Server):

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```

Breaking Down Script:

Line 1: #!/bin/bash
├─ Purpose: Specifies script language
└─ Required: First line

Line 2: yum update -y
├─ Action: Update system packages
├─ Flag: -y means answer "yes" to prompts
└─ Timing: Happens at startup

Line 3: yum install -y httpd
├─ Action: Install Apache web server
├─ Flag: -y automatic approval
└─ Result: Web server installed

Line 4: systemctl start httpd
├─ Action: Start web server service
├─ Timing: Now running
└─ Effect: Listening on port 80

Line 5: echo "Hello..." > /var/www/html/index.html
├─ Action: Create HTML file
├─ Content: "Hello from [hostname]"
├─ Location: Web root directory
└─ Result: Accessible at http://instance-ip/

Entering User Data:

Location: "User data" field in Advanced Details
├─ Type: Textarea
├─ Paste: Entire bash script
├─ Verify: Script correct
└─ Warning: Will run on every launch

Testing User Data:
├─ Best: Test before using in ASG
├─ Method: Launch single instance, verify works
├─ Purpose: Catch errors early
└─ Benefit: Before scaling to many instances

Step 8: Monitoring

Detailed Monitoring:

Option: "Detailed CloudWatch monitoring"
├─ Unchecked: Basic monitoring (5-minute intervals)
├─ Checked: Detailed monitoring (1-minute intervals)
├─ Cost: Small additional charge for detailed
└─ Recommendation: Leave unchecked for learning

Monitoring in ASG:
├─ Less important: For learning labs
├─ More important: For production
└─ Can enable: Later if needed

Step 9: Review and Create

Review All Settings:

Verify Section:
├─ Template name: Correct?
├─ AMI: Amazon Linux 2?
├─ Instance type: t2.micro?
├─ Key pair: Correct?
├─ Security group: Allows HTTP/SSH?
├─ Storage: 8 GB gp2?
├─ User data: Present and correct?
└─ Ready: All settings correct?

Creating Template:

Button: "Create launch template"
├─ Action: Submitted
├─ Status: Template being created
├─ Duration: Seconds
└─ Redirect: Template details page

Confirmation:

Success Page:
├─ Shows: Template created
├─ ID: Template ID displayed
├─ Version: Version 1 created
├─ Next: Return to EC2 dashboard
└─ Ready: Use in ASG creation

Template Details:

Already Created:
├─ Name: "my-demo-template"
├─ Version: "$Default" (Version 1)
├─ Summary: All settings visible
├─ Ready: For ASG to use
└─ Location: Left sidebar → "Launch Templates"

Viewing Template:

To See Template Again:
├─ EC2 Dashboard: Left sidebar
├─ Click: "Launch Templates"
├─ Find: "my-demo-template"
├─ Click: Open template
└─ View: All settings and versions

Common Issues:

AMI Not Available in Region:

Problem:
├─ Message: "AMI not available"
├─ Reason: Selected AMI not in region
└─ Solution: Select different AMI or change region

Free Tier Not Selected:

Problem:
├─ During: Launching instances
├─ Issue: Higher costs than expected
└─ Prevention: Always select free tier for learning

User Data Not Executing:

Problem:
├─ Symptom: Web server not running
├─ Likely: User data has error
└─ Solution: Check user data script for errors
```

---

## Part 3: Creating the Auto Scaling Group

### ASG Configuration and Launch

**Step-by-Step ASG Creation**

```
Creating Auto Scaling Group:

AWS Console Navigation:

From EC2 Dashboard:
├─ Left sidebar: "Auto Scaling Groups"
├─ Button: "Create Auto Scaling group"
├─ Page: ASG creation wizard
└─ Ready: Step 1 of several

Step 1: Choose Launch Template

Select a Launch Template:

Dropdown: "Launch template"
├─ List: Available templates
├─ Find: "my-demo-template"
├─ Click: Select it
└─ Status: Template selected

Template Version:

Dropdown: "Version"
├─ Option: $Default (recommended)
├─ Or: $Latest
├─ Or: Specific version (1, 2, 3, etc.)
├─ Selection: $Default uses default version
└─ Good: For most cases

Template Selection:

Verify: Correct template selected
└─ Summary: Shows template name and version

Action: Click "Next"
└─ Proceed: To step 2

Step 2: Choose Instance Launch Options

Instance Type Requirements:

Advanced Feature:
├─ Complexity: High
├─ For learning: Not needed
├─ Default: Accept launch template
└─ Action: Click "Reset to launch template defaults"

Network:

VPC Selection:
├─ Dropdown: "VPC"
├─ Choose: Your VPC
├─ Or: Default VPC (if using)
└─ Selected: Only one VPC per ASG

Availability Zones:

Option 1: Subnets Selection (Recommended)
├─ Shows: Subnets in your region
├─ Select: Multiple subnets (different AZs)
├─ Example: 3 subnets in us-east-1a, 1b, 1c
├─ Benefit: Multi-AZ redundancy
└─ Action: Check multiple subnets

Option 2: Availability Zones
├─ Select: By AZ instead
├─ Less granular: Than subnet selection
└─ OK: But subnet method preferred

Example Multi-AZ Setup:
├─ Subnet 1: us-east-1a-private-web
├─ Subnet 2: us-east-1b-private-web
├─ Subnet 3: us-east-1c-private-web
└─ Result: Instances spread across 3 AZs

AZ Distribution:

Option: "AZ rebalancing"
├─ Default: Balanced best effort
├─ Effect: Distributes instances fairly
├─ Alternative: Capacit optimized (advanced)
└─ Recommendation: Keep default

Step 3: Load Balancer Configuration

Load Balancer Option:

Three Choices:

1. No Load Balancer:
   ├─ Selection: "Do not attach to a load balancer"
   ├─ Effect: Instances created but not in LB
   ├─ Use: Testing only
   └─ Limitation: Manual traffic direction

2. Attach to Existing ALB/NLB:
   ├─ Selection: "Attach to a new load balancer"
   ├─ Effect: Instances register to LB
   ├─ Use: Recommended for production
   └─ Requirement: Create LB first

3. Attach to Existing Load Balancer:
   ├─ Selection: Yes (recommended)
   ├─ Process: Select existing LB/target group
   └─ Result: Instances auto-register

Attaching to Existing Load Balancer:

Choose Load Balancer Type:

If Already Created:
├─ Dropdown: "Load balancer type"
├─ Options: Application Load Balancer or Network Load Balancer
├─ Select: Based on what you created
└─ Example: Application Load Balancer

Select Target Group:

Target Group Dropdown:
├─ List: Available target groups
├─ Find: "demo-tg-alb" or similar
├─ Select: Your target group
└─ Effect: Instances will be registered here

Example Target Group:
├─ Name: demo-tg-alb
├─ Protocol: HTTP
├─ Port: 80
├─ Health check path: /
├─ Status: Ready to receive instances

Load Balancer vs VPC Lattice:

VPC Lattice:
├─ Newer: Advanced networking feature
├─ For learning: Skip this
├─ For labs: Not needed
└─ Leave: Unchecked

Zonal Shift Features:

Zonal Shift:
├─ Advanced: Traffic management
├─ For learning: Not needed
└─ Leave: Unchecked

Step 4: Health Checks

Health Check Configuration:

Health Check Type:

Option 1: EC2 Status Checks
├─ What: EC2 instance running check
├─ Level: Infrastructure level
├─ Limitation: Doesn't check app
└─ Not ideal: Alone for production

Option 2: ELB Health Checks
├─ What: Application level check
├─ Level: Load balancer verifies response
├─ Better: For real app health
└─ Recommended: When LB attached

Selecting Both (Recommended):

If LB Attached:
├─ Check: "Enable load balancer health checks"
├─ Also: Keep EC2 health checks
├─ Effect: Both types active
└─ Result: Best health detection

Health Check Grace Period:

Purpose:
├─ Delay: Before first health check
├─ Reason: Instance needs time to boot
├─ Duration: Seconds
└─ Default: 300 seconds (5 minutes)

Adjusting Grace Period:

Based on App:
├─ Quick app: 300-600 seconds ok
├─ Slow startup: 600-1800 seconds
├─ Example: Java app needs 2-5 minutes
└─ Recommendation: 300 seconds minimum

Example Configuration:
├─ EC2 checks: Enabled
├─ ELB checks: Enabled
├─ Grace period: 300 seconds
└─ Effect: Comprehensive health monitoring

Step 5: Capacity Configuration

Minimum Capacity (Min Size):

What It Means:
├─ Minimum: Instances always running
├─ Guarantee: Won't go below this
├─ Example: Min=1 means always at least 1
└─ Trade: Cost vs redundancy

Setting Minimum:

For Learning:
├─ Set: 1 (cost optimization)
├─ Effect: Single instance minimum
└─ Trade: No HA backup

For Production:
├─ Set: 2+ (high availability)
├─ Effect: Always redundant
└─ Cost: Slightly higher

Example: Set to 1
└─ Field: "Minimum capacity: 1"

Maximum Capacity (Max Size):

What It Means:
├─ Maximum: Most instances can reach
├─ Limit: ASG won't exceed this
├─ Example: Max=10 means never more than 10
└─ Control: Cost/quota protection

Setting Maximum:

For Learning:
├─ Set: 2-3 (manageable)
├─ Effect: Limited scaling for testing
└─ Benefit: Low cost even if scale-out

For Production:
├─ Set: Based on budget
├─ Example: $10/day budget = 20 instances
└─ Formula: (Daily budget / Hourly cost) ÷ 24

Example: Set to 3
└─ Field: "Maximum capacity: 3"

Desired Capacity (Initial Desired):

What It Means:
├─ Initial: Instances to launch now
├─ Target: Current goal
├─ Range: Must be between Min and Max
├─ Example: Desired=1 launches 1 instance

Setting Desired:

For Learning:
├─ Set: 1 (minimal cost)
├─ Effect: 1 instance launches now
└─ Later: Scale manually to test

For Production:
├─ Set: Expected baseline
├─ Example: Desired=3 for web app
└─ Scaling: Adjusts up/down from here

Example: Set to 1
└─ Field: "Desired capacity: 1"

Capacity Summary:

Configuration:
├─ Min: 1
├─ Desired: 1
├─ Max: 3
└─ Meaning: Currently 1 instance, can grow to 3, won't drop below 1

Step 6: Automatic Scaling

Scaling Policies:

For Learning:
├─ Skip: Leave as "None"
├─ Reason: Will test manual scaling first
├─ Later: Add policies after basics work
└─ Action: Click "Next" to skip

Advanced Features (Skip for Now):

Optional sections:
├─ VPC endpoints: Advanced
├─ Additional settings: Leave default
└─ Action: Skip by clicking "Next"

Step 7: Notifications (Optional)

SNS Notifications:

Purpose:
├─ Alert: Via email when scaling happens
├─ Useful: For monitoring
├─ Required: No
└─ For learning: Skip

Action:
├─ Leave: Unchecked
└─ Click: "Next"

Step 8: Tags (Optional but Recommended)

Tagging Strategy:

Benefits:
├─ Organization: Easy to find resources
├─ Billing: Track costs by tag
├─ Automation: Tag-based actions
└─ Good practice: Always tag

Common Tags:

Name:
├─ Key: Name
├─ Value: Demo-ASG-Web
└─ Effect: Shows in console

Environment:
├─ Key: Environment
├─ Value: dev, staging, or prod
└─ Effect: Identify tier

Application:
├─ Key: Application
├─ Value: website, api, database, etc.
└─ Effect: Group related resources

Example Tags:
├─ Name: Demo-ASG
├─ Environment: dev
├─ Application: web
└─ Team: platform

Adding Tags:

Click: "Add tag"
├─ Key: Name
├─ Value: Demo-ASG
├─ Apply to: Checked
└─ Effect: Tag applied to ASG and instances

Step 9: Final Review

Review Page:

Verify All:
├─ Name: Demo ASG (or your name)
├─ Launch template: my-demo-template
├─ Min capacity: 1
├─ Max capacity: 3
├─ Desired: 1
├─ VPC: Correct
├─ Subnets: Multiple AZs
├─ Load balancer: Connected
├─ Health checks: Both enabled
├─ Grace period: 300 seconds
└─ Tags: Present

Creating ASG:

Button: "Create Auto Scaling group"
├─ Action: ASG created
├─ Redirect: ASG details page
└─ Status: Launching first instance

Confirmation:

Success Page:
├─ Shows: ASG created
├─ Name: Your ASG name
├─ Description: Summary
├─ Action: Click to view ASG
└─ Next: Monitor instance creation

ASG Details Page:

Shows:
├─ ASG name
├─ Capacity: Min, Max, Desired
├─ Launch template: Referenced
├─ VPC and subnets
├─ Load balancer info
└─ Tabs: Activity, Instance management, etc.

Common Issues During Creation:

VPC/Subnet Error:

Problem:
├─ Message: "No subnets selected"
├─ Reason: Subnet selection required
└─ Solution: Select at least one subnet

Load Balancer Error:

Problem:
├─ Message: "Target group not found"
├─ Reason: Target group doesn't exist
└─ Solution: Create target group in ALB first

Capacity Error:

Problem:
├─ Message: "Min > Max"
├─ Reason: Min capacity exceeds max
└─ Solution: Adjust so Min ≤ Max

Launch Template Error:

Problem:
├─ Message: "Template not found"
├─ Reason: Template not selected
└─ Solution: Select launch template in step 1
```

---

## Part 4: Monitoring Initial Instance Launch

### Tracking First Instance Creation

**Watching ASG Create Its First Instance**

```
Instance Creation Process:

Timeline:

T=0 seconds: ASG Created
├─ Status: Created successfully
├─ Desired capacity: 1
├─ Current instances: 0
└─ Action: Begins launching instance

T=0-30 seconds: Instance Launching
├─ Status: In ASG activity history
├─ Message: "Launching 1 instance"
├─ Reason: "Desired capacity increased"
└─ Visible: Can see in activity

T=30-90 seconds: EC2 Instance Starting
├─ Status: Instance booting
├─ EC2 state: Running (but still initializing)
├─ Console: Shows in EC2 instances list
└─ IP addresses: Being assigned

T=90-180 seconds: Application Initializing
├─ Status: User data script running
├─ Apache: Being installed and started
├─ Web server: Becoming ready
└─ Health check: About to start

T=180-300 seconds: Grace Period
├─ Status: Health checks waiting for grace period
├─ Instance: Running but health checks ignored
├─ Reason: Give time for application startup
└─ Next: After grace period, health checks active

T=300+ seconds: Health Checks Active
├─ Status: First health check executed
├─ LB: Checking instance response
├─ Port 80: Attempting HTTP connection
└─ Response: Checking for 200 OK status

T=300-330 seconds: Instance Transitions to Healthy
├─ Status: "InService" (target group shows healthy)
├─ Traffic: Now receiving requests
├─ Console: Instance shows healthy
└─ Complete: Instance fully operational

Monitoring Instance Creation:

Activity History:

Access:
├─ ASG page: Select your ASG
├─ Tab: "Activity"
└─ View: All scaling activities

Activity Entry Example:

Timestamp: 2024-03-15 10:30:00
├─ Status: Successful
├─ Details: "Launching 1 instance"
├─ Instance: i-0c4d5e6f7a8b9c0d1 (ID displayed)
├─ Cause: "User initiated"
└─ Message: "Desired capacity increased from 0 to 1"

Refreshing Activity:

Browser Refresh:
├─ Button: Refresh (or browser F5)
├─ Timing: Every 10-20 seconds
├─ Watch: Activity updating
└─ Observe: Instance progress

Instance Management Tab:

Access:
├─ ASG page: Select your ASG
├─ Tab: "Instance management"
└─ View: All instances

Instance Status:

Columns:
├─ Instance ID: i-1234567890abcdef0
├─ Instance type: t2.micro
├─ AZ: us-east-1a
├─ Lifecycle: "Normal" or "Terminating"
├─ Health status: "Healthy" or "Unhealthy"
├─ LB Target Status: "InService" or "OutOfService"
└─ Protected: From termination (yes/no)

Example Instance:

Instance ID: i-abc123def456
├─ Type: t2.micro
├─ AZ: us-east-1b
├─ Lifecycle: Normal (running)
├─ Health: Healthy (after grace period)
├─ LB Status: InService (ready for traffic)
└─ Protected: No

EC2 Instances Tab:

Direct EC2 View:

Access:
├─ Left sidebar: "Instances"
├─ View: All instances in account
├─ Filter: Can search by name or tag
└─ See: Your launched instance

Instance Details:

Columns:
├─ State: running, terminated, stopping, etc.
├─ Instance ID: i-xxx
├─ Instance type: t2.micro
├─ Private IP: 10.0.x.x
├─ Public IP: (if applicable)
├─ VPC: VPC-xxx
├─ Subnet: subnet-xxx
├─ Status checks: Initializing → Passed
└─ Tags: Name, Environment, etc.

Example Instance:

State: running (green checkmark)
├─ Instance ID: i-abc123def456
├─ Type: t2.micro
├─ Private IP: 10.0.1.45
├─ Status checks: 2/2 passed
└─ Source: ASG (visible in console)

Target Group Status:

Finding Target Group:

Access:
├─ Left sidebar: "Target Groups"
├─ Find: "demo-tg-alb" (your target group)
├─ Click: Open it
└─ Tab: "Targets"

Target Group Targets View:

Columns:
├─ Instance ID: i-abc123def456
├─ Port: 80
├─ Health status: "Initial" → "Healthy"
├─ Description: Status reason
└─ Registered: When registered

Timeline View:

T=Instance Launch: "Initial" Status
├─ Reason: "Elb.RegistrationInProgress"
└─ Message: Being registered to target group

T=Grace Period: Still "Initial"
├─ Reason: "Elb.RegistrationInProgress"
└─ Message: Waiting for grace period

T=Grace Period End: Health Check Active
├─ Reason: "Health checks started"
└─ Message: LB begins checking

T=Health Check Passes: "Healthy"
├─ Reason: "N/A"
└─ Message: Instance responding properly

Expanding Target Details:

Click: Arrow to expand
├─ Shows: Instance IP and port
├─ Shows: Health check results
├─ Shows: Response time
└─ Useful: For debugging

Verifying Load Balancer Connection:

Testing After Healthy Status:

Browser Test:

URL: ALB DNS name
├─ Example: demo-alb-123456789.us-east-1.elb.amazonaws.com
└─ In address bar: Paste and press Enter

Expected Response:

Page Shows:
├─ Title: "Hello from [hostname]"
├─ Hostname: Instance name (from user data)
└─ Success: Page loads (confirms working)

Finding ALB DNS Name:

Access:
├─ Left sidebar: "Load Balancers"
├─ Find: "demo-alb" or your ALB
├─ Tab: "Details"
├─ Field: "DNS name"
└─ Copy: Full DNS name

Example DNS: demo-alb-123456789.us-east-1.elb.amazonaws.com

Testing Flow:

1. Browser: Navigate to ALB DNS
2. Request: Sent to load balancer
3. LB: Routes to target group
4. Target: Instance receives request
5. Response: Web server returns hello page
6. Browser: Shows page with instance hostname
7. Success: End-to-end working

Troubleshooting Unhealthy Instance:

If Instance Shows Unhealthy:

Possible Causes:

1. Still initializing:
   ├─ Status: "Initial"
   ├─ Action: Wait 5-10 minutes
   └─ Resolution: Usually auto-resolves

2. Security group blocking 80:
   ├─ Status: Timeout on health check
   ├─ Fix: Add HTTP (port 80) to SG
   └─ Action: Modify security group

3. User data script error:
   ├─ Status: Port 80 not listening
   ├─ Fix: Check user data for errors
   └─ Action: Connect via SSH, debug

4. Application not starting:
   ├─ Status: Connection refused
   ├─ Fix: Verify user data installs app
   └─ Action: Connect to instance, check logs

Investigation Steps:

SSH to Instance:

Get IP:
├─ EC2 Instances page
├─ Copy: Public IP or Private IP
└─ Note: Public IP better for external access

SSH Command:
```bash
ssh -i ~/.ssh/EC2\ Tutorial.pem ec2-user@10.0.1.45
```

Logged In:

Check web server:
```bash
systemctl status httpd
```

Check if running:
├─ Active: running ✓ (good)
├─ Inactive: dead ✗ (problem)
└─ Action: systemctl start httpd if needed

Check web root:
```bash
curl http://localhost/
```

View file:
├─ Should show: "Hello from hostname"
├─ Or error: File not found
└─ Fix: Check /var/www/html/index.html

Check logs:
```bash
sudo tail -f /var/log/httpd/access_log
```

See requests:
├─ Shows: Recent HTTP requests
├─ Missing: If not receiving traffic yet
└─ Verify: Port 80 open

Automatic Recovery:

If Unhealthy Persists:

ASG Action:
├─ Detection: Marked unhealthy for too long
├─ Action: Terminates instance
├─ Replacement: Launches new instance
└─ Result: Self-healing occurs

Termination Process:

1. Fail detection
2. Deregister from LB (drain)
3. Terminate instance
4. Launch replacement
5. Register to LB
6. Health check new instance
7. Back to healthy

Timeline: 10-15 minutes for full recovery

Monitoring Summary:

Key Checks:

After Creation, Verify:

1. Activity history shows "Launching 1 instance" ✓
2. Instance management shows 1 instance ✓
3. EC2 instances tab shows running instance ✓
4. Status checks show "2/2 passed" ✓
5. Target group shows instance "Healthy" ✓
6. Browser to ALB shows "Hello" page ✓
7. All items check: ASG working correctly ✓

If All Check: Ready for next steps
If Any Fail: Troubleshoot that specific item
```

---

## Part 5: Manual Scaling Operations

### Adding and Removing Instances

**Scaling Up and Down**

```
Scaling Up (Adding Instances):

Goal: Increase capacity from 1 to 2 instances

Before Scaling:

Current State:
├─ Min: 1
├─ Max: 3
├─ Desired: 1  
├─ Running: 1 instance
└─ Target group: 1 healthy instance

Scaling Up Process:

Step 1: Edit ASG

Access:
├─ ASG dashboard: Select "Demo ASG"
├─ Button: "Edit" (or "Update")
└─ Or: Right-click → "Edit"

Step 2: Update Desired Capacity

Field: "Desired capacity"
├─ Current value: 1
├─ Change to: 2
└─ Also: Increase Max to 2 or higher

Example Update:
├─ Min: 1 (unchanged)
├─ Max: 3 (unchanged, already high enough)
├─ Desired: 1 → 2 (CHANGE THIS)
└─ Action: Set desired = 2

Step 3: Save Changes

Button: "Update"
├─ Action: Changes submitted
├─ Effect: Scaling begins immediately
└─ Confirmation: "ASG updated successfully"

Instance Launch Begins:

Immediate After Update:

Activity History Updated:
├─ New entry: "Launching 1 instance"
├─ Reason: "Desired capacity changed from 1 to 2"
├─ Status: Active
└─ Progress: Instance launching

Timeline:

T=0-60 seconds: Instance Launch Initiated
├─ Console: "Launching" appears in activity
├─ EC2: New instance appearing in instances list
├─ State: Running or pending

T=60-120 seconds: Instance Booting
├─ EC2: State transitions to "running"
├─ User data: Script starting to execute
├─ Web server: Being installed

T=120-300 seconds: Application Initializing
├─ Httpd: Installing and starting
├─ Grace period: Beginning
├─ Health check: Queued but not started

T=300 seconds: Grace Period Ends
├─ Health check: First check executed
├─ LB: Connects to port 80
├─ Response: On way from web server

T=300-330 seconds: Health Check Passes
├─ Status: "Healthy" in target group
├─ Traffic: Now receiving requests
└─ Complete: Scaling done

Monitoring Scale-Up:

Activity History:

Refresh: Every 10-20 seconds
├─ T=now: "Launching 1 instance"
├─ T=2min: Entry updates with status
├─ T=5min: Usually "Successful"
└─ Watch: Progress of instance creation

Instance Management:

Shows:
├─ Count: Now 2 instances listed
├─ New instance: Recently launched
├─ Health: "Initializing" then "Healthy"
├─ LB status: "OutOfService" then "InService"
└─ After 5+ min: Should all show healthy/InService

EC2 Instances:

Shows:
├─ Count: 2 instances total
├─ New instance: Recent start time
├─ Status checks: "Initializing" then "2/2 Passed"
├─ Private IP: New IP assigned
└─ ASG: Both tagged as ASG members

Target Group:

Shows:
├─ Count: 2 targets listed
├─ New target: Shows "Initial"... "Healthy"
├─ Both: Eventually show healthy/InService
└─ Targets: Both receiving health checks

Testing After Scale-Up:

Browser Refresh:

URL: ALB DNS name
```bash
demo-alb-123456789.us-east-1.elb.amazonaws.com
```

Observations:

First Request:
├─ Hostname: Instance 1 (original)
└─ Hello from: hostname1

Second Request:
├─ Hostname: Instance 2 (new)
└─ Hello from: hostname2

Pattern:
├─ Requests: Alternate between instances
├─ Round-robin: ALB distributes equally
├─ Both: Responding to requests
└─ Success: Load balancing working

Multiple Refreshes:

Observation:
├─ First request: Instance 1
├─ Second request: Instance 2
├─ Third request: Instance 1
├─ Pattern: Round-robin distribution
└─ Conclusion: LB working correctly

Load Distribution Working:
├─ Requests: Spread fairly
├─ Traffic: Both instances getting load
├─ Verification: Manual testing confirms
└─ Result: ASG + LB integrated properly

Scaling Down (Removing Instances):

Goal: Decrease capacity from 2 to 1 instance

Before Scaling Down:

Current State:
├─ Min: 1
├─ Max: 3
├─ Desired: 2
├─ Running: 2 instances
└─ Target group: 2 healthy instances

Scaling Down Process:

Step 1: Edit ASG

Access:
├─ ASG dashboard: Select "Demo ASG"
├─ Button: "Edit"
└─ Or: Right-click → "Edit"

Step 2: Update Desired Capacity

Field: "Desired capacity"
├─ Current value: 2
├─ Change to: 1
└─ Action: Set desired = 1

Example Update:
├─ Min: 1 (unchanged)
├─ Max: 3 (unchanged)
├─ Desired: 2 → 1 (CHANGE THIS)
└─ Action: Single instance target

Step 3: Save Changes

Button: "Update"
├─ Action: Changes submitted
├─ Effect: Scaling down begins
└─ Confirmation: "ASG updated successfully"

Instance Termination Begins:

Immediate After Update:

Activity History Updated:
├─ New entry: "Terminating 1 instance"
├─ Reason: "Desired capacity changed from 2 to 1"
├─ Status: Active
└─ Progress: Instance terminating

Which Instance Terminates?

ASG Selection Logic:
├─ Strategy: Balanced termination
├─ Selection: Oldest instance usually
├─ But: May choose based on AZ distribution
├─ Fair: Predictable termination order
└─ First: Review which might terminate

Termination Timeline:

T=0-30 seconds: Termination Initiated
├─ Activity: "Terminating 1 instance" shown
├─ Instance: Selected for termination
└─ Action: Beginning termination process

T=30-60 seconds: Deregistration from LB
├─ LB: Instance marked "draining"
├─ Requests: In-flight requests continue
├─ Grace: Connection draining honored
└─ New requests: Not sent to this instance

T=60-300 seconds: Draining Period
├─ Timeout: 300 seconds (5 minutes) by default
├─ Purpose: Finish existing requests
├─ Monitoring: Watch active connections drop
└─ Duration: Can be shorter if all requests done

T=300-330 seconds: Instance Terminates
├─ Action: SIGTERM sent to instance
├─ Process: Graceful shutdown begins
├─ State: "Terminating" then "Terminated"
└─ Complete: Instance shut down

T=330+ seconds: Removal
├─ Console: No longer shows in instances
├─ EC2: May stay in history briefly
├─ ASG: Desired capacity maintained
└─ Result: Capacity now 1 instance

Monitoring Scale-Down:

Activity History:

Refresh: Every 10-20 seconds
├─ Entry: "Terminating 1 instance"
├─ Status: Active then Successful
├─ Reason: "Desired capacity decreased"
└─ Timeline: Fast (usually 5-10 minutes)

Target Group:

Watch Deregistration:
├─ Start: 2 healthy targets
├─ Then: One shows "draining"
├─ Then: Draining target disappears
├─ Result: 1 healthy target left
└─ Time: 5-10 minutes for full drain

EC2 Instances:

Watch Termination:
├─ Start: 2 instances "running"
├─ Then: One shows "terminating"
├─ Then: Count goes to 1
├─ Status: Remaining instance still running
└─ Result: 1 instance remains

Instance Management:

Shows:
├─ Count: Now 1 instance listed
├─ Remaining: Original instance still there
├─ Terminated: Instance no longer shown
└─ Health: Remaining one shows healthy

Testing After Scale-Down:

Browser Refresh:

URL: ALB DNS name
```bash
demo-alb-123456789.us-east-1.elb.amazonaws.com
```

Observation:

All Requests:
├─ Go to: Single remaining instance
├─ Hostname: Always same
└─ Success: Single instance handling all

Verification:
├─ Only 1 IP: Requests go to one instance
├─ No switching: No load balancing visible (only 1 target)
└─ Confirmed: Correct instance terminated

Wait During Drain:

Connection Draining:

What Happens:
├─ During: Old requests complete
├─ New: Not accepted on draining instance
├─ Time: Potentially several minutes
└─ Benefit: No abrupt request drops

If You Refresh Immediately:
├─ May see: Temporary 502 (bad gateway)
├─ Reason: If request happened to route to draining instance
├─ Brief: Corrects in seconds
└─ After: All requests go to healthy instance

Patience During Drain:

Recommendation:
├─ Wait: 5-10 minutes after scale order
├─ Reason: Respects in-flight requests
├─ Benefit: Smooth, graceful scaling
└─ Result: No user-visible interruption

Manual Scaling Workflow Summary:

Scale Up Workflow:

1. Edit ASG desired: 1 → 2
   ↓
2. Instance launches
   ↓
3. User data runs (2-3 min)
   ↓
4. Grace period (5 min)
   ↓
5. Health check passes
   ↓
6. Target group shows 2 healthy
   ↓
7. LB distributes to both
   ↓
8. Browser shows round-robin
   ↓
9. Complete: Both instances active

Scale Down Workflow:

1. Edit ASG desired: 2 → 1
   ↓
2. Termination selected
   ↓
3. Deregister from LB (drain)
   ↓
4. Drain grace period (5 min)
   ↓
5. Instance terminate
   ↓
6. Target group shows 1 healthy
   ↓
7. LB only routes to remaining
   ↓
8. Browser shows single instance
   ↓
9. Complete: 1 instance running

Observations:

ASG Power:

Self-Managed:
├─ Register: New instances auto-register to LB
├─ Deregister: Removed instances gracefully drained
├─ Health: Automatic health check integration
├─ Scale: Can add/remove instances easily
└─ Users: Zero downtime during scaling

Manual Testing Benefits:

Understand:
├─ How: Instances are created
├─ Why: Health checks take time
├─ When: Instance becomes healthy
├─ What: Load balancer does
└─ Result: Full understanding of ASG + LB integration

Ready for Automation:

After Testing:
├─ Confident: Manual scaling works
├─ Next: Can add automatic scaling policies
├─ Foundation: Understand what policies control
└─ Goal: Full automation with protection

Common Observations:

During Scale-Up:

Activity Takes Time:
├─ Not instant: Takes 5-15 minutes
├─ Reason: Instance boot + user data + health checks
├─ Normal: This is expected
└─ No action: Don't repeatedly edit

During Scale-Down:

Draining is Polite:
├─ Waits: For in-flight requests
├─ Doesn't: Abruptly kill connections
├─ Time: Can take 5-10 minutes
└─ Benefit: No user-visible errors

Health Check Status:

Watch Transition:
├─ Initial: OutOfService (during grace)
├─ Transition: Health check passing
├─ Final: InService (ready for traffic)
└─ Normal: Grace period waiting is expected
```

---

## Part 6: Exam Focus and Best Practices

### Critical Takeaways and Key Concepts

**ASG Hands-On Certification Focus**

```
Exam-Critical Concepts:

Auto Scaling Group Creation:

Step 1: Launch Template First
✓ Must exist before ASG
✓ Contains: AMI, instance type, SG, key pair, user data
✓ Versioning: Supported (unlike deprecated launch config)

Step 2: Create ASG
✓ Name: Descriptive identifier
✓ Reference: Launch template
✓ Capacity: Min, Max, Desired
✓ Network: VPC and multi-AZ selected

Step 3: Configure LB Integration
✓ Optional: But recommended
✓ Effect: Auto-register new instances
✓ Health: Can use LB health checks
✓ Deregistration: Automatic with graceful drain

Step 4: Set Health Checks
✓ EC2 checks: Instance level
✓ ELB checks: Application level (if LB)
✓ Grace period: Default 300 seconds
✓ Both: Recommended for reliability

Step 5: Activity Monitoring
✓ Activity tab: Shows all scaling actions
✓ Instance mgmt: Current instance status
✓ Target group: Health and registration status
✓ EC2 console: Verify instances exist

Hands-On Workflow:

Creation Flow:

Launch Template → ASG → Instance Launch → Registration → Health Check → InService

Timing:

Total: 5-15 minutes
├─ Creation: Instant
├─ Launch: 1-2 minutes
├─ User data: 1-3 minutes (app startup)
├─ Grace period: 5 minutes (by default)
├─ Health check: Passes after grace
└─ InService: 5-15 minutes post-creation

Key Metrics:

Desired vs Actual:
✓ Desired: Target capacity
✓ Current: Running instances
✓ Goal: Desired = Current
✓ Time: Takes minutes to match

Scaling Activity:

Manual Edit:
✓ Change: Desired capacity
✓ Save: Update ASG
✓ Action: Launches or terminates instances
✓ Timeline: 5-15 minutes to complete

Health Transitions:

Initial:
├─ State: "Initial" or "OutOfService"
├─ Duration: Grace period
└─ Why: Giving app time to start

Healthy:
├─ State: "InService"
├─ Duration: Until termination or failure
└─ Traffic: Receiving requests

Unhealthy:
├─ State: "Unhealthy"
├─ Action: Marked and scheduled for termination
├─ Replacement: New instance launched
└─ Timeline: 5-15 minutes for replacement

Load Balancer Integration:

Auto-Registration:

New Instance:
✓ Launched: By ASG
✓ Auto-registered: To target group
✓ Visible: In target group targets list
✓ Health check: Begins immediately
✓ Grace period: Honored before real checks

Deregistration:

Termination:
✓ Detected: By ASG
✓ Deregister: From load balancer
✓ Drain: Respect in-flight requests
✓ Grace: Deregistration delay honored
✓ Terminate: After grace or drain complete

Graceful Scaling:

No Connection Loss:
✓ Existing: Connections complete naturally
✓ New: Not sent to draining instance
✓ Clients: Don't see interruptions
✓ User: Experiences seamless scaling

Activity History:

Events Recorded:

Scaling Out:
✓ "Launching N instances"
✓ "Desired capacity changed from X to Y"
✓ "Successful" (after complete)

Scaling In:
✓ "Terminating N instances"
✓ "Desired capacity changed from X to Y"
✓ "Successful" (after complete)

Health Replacement:
✓ "Terminating unhealthy instance"
✓ "Launching replacement"
✓ "Health status failed"

Manual vs Automatic:

Manual Scaling:
✓ Edit: Change desired capacity
✓ When: You decide
✓ How much: You specify
✓ Good for: Learning, testing, manual control

Automatic Scaling:
✓ Edit: Create scaling policies
✓ When: Metrics trigger
✓ How much: Policy determines
✓ Good for: Production, responsive, autonomous

Troubleshooting Scenarios:

Scenario: Instance Stuck in "Initializing"

Possible Causes:
├─ User data: Script has error
├─ Web server: Not starting properly
├─ Port: Security group doesn't allow 80
├─ App: Takes longer than grace period to start
└─ Fix: Check user data, extend grace, fix SG

Investigation:
├─ SSH: Connect to instance
├─ Check: systemctl status httpd
├─ Verify: curl http://localhost/
├─ Logs: tail /var/log/httpd/error_log
└─ Fix: Resolve issue, ASG will replace

Scenario: Instance Shows Unhealthy

Possible Causes:
├─ App crashed: Not responding
├─ Port wrong: Health check uses wrong port
├─ Status code: Not returning 200
├─ Timeout: App too slow to respond
└─ Fix: Investigate application issue

ASG Action:
├─ Detection: Failed 2-3 health checks
├─ Wait: Can take several minutes
├─ Action: Terminates unhealthy instance
├─ Launch: Replacement instance
└─ Result: Self-healing

Scenario: Scaling Not Happening

If Manual Edit:
├─ Check: Desire changed?
├─ Check: Max high enough?
├─ Wait: Takes 5-15 minutes
└─ Verify: Activity history shows activity

If Desired Not Changing:
├─ Reason: May already be at target
├─ Capacity: May be constrained
└─ Action: Manually edit again if needed

Exam Questions:

1. "Name two components of a launch template"
   Answer: Any two of: AMI, instance type, security group, key pair, IAM role, user data, storage, tags

2. "What is the default health check grace period?"
   Answer: 300 seconds (5 minutes)

3. "How long does scaling typically take?"
   Answer: 5-15 minutes (instance boot + user data + health checks)

4. "Can you change desired capacity without changing min/max?"
   Answer: Yes, as long as it stays within min ≤ desired ≤ max

5. "What happens to in-flight requests during scale-in?"
   Answer: Connection draining honors them (graceful termination)

6. "Where do you monitor scaling activities?"
   Answer: Activity tab in ASG console

7. "Can instances auto-register to load balancer?"
   Answer: Yes, if target group configured during ASG creation

8. "What runs at every instance launch?"
   Answer: User data script (once per instance)

9. "How is instance selected for termination?"
   Answer: Balanced strategy (oldest instance usually)

10. "Do you pay for ASG component?"
    Answer: No, ASG itself free. You pay only for EC2 instances

Best Practices:

Pre-Launch:

✓ Test: Launch template with single instance first
✓ Verify: User data script works
✓ Check: Security group allows needed ports
✓ Create: Load balancer and target group
✓ Plan: Capacity settings (min/max/desired)

During Launch:

✓ Monitor: Activity history for progress
✓ Wait: 5-15 minutes for healthy status
✓ Verify: Target group shows healthy
✓ Test: Browser request returns response
✓ Observe: Instance added to AWS console

Scaling Operations:

✓ Change: Desired capacity only (not min/max usually)
✓ Expect: 5-15 minute lag
✓ Monitor: Activity and target group status
✓ Test: Load balancing still working
✓ Patience: Drain period can take several minutes

Troubleshooting:

✓ SSH: Connect to investigate issues
✓ Logs: Check application logs
✓ Service: Verify web server running
✓ Port: Test curl http://localhost/
✓ Allow: Scaling replacement time

Transition from Manual to Automatic:

Phase 1: Manual (Today)
├─ Master: Creating ASG
├─ Understand: Health checks
├─ Learn: Scaling timeline
├─ Test: LB integration
└─ Gain confidence

Phase 2: Automatic (Next)
├─ Build on: Manual foundation
├─ Add: Scaling policies
├─ Monitor: Metrics and alarms
├─ Refine: Based on observations
└─ Automate: Scaling decisions

Hands-On Mastery:

You Should Now:

✓ Create launch template from scratch
✓ Specify user data script
✓ Create ASG referencing template
✓ Configure multi-AZ deployment
✓ Integrate with load balancer
✓ Monitor health check grace period
✓ Verify instance registration
✓ Test load balancing manually
✓ Scale up by increasing desired
✓ Wait for new instance health checks
✓ Verify load distribution
✓ Scale down gracefully
✓ Understand activity history
✓ Troubleshoot if unhealthy
✓ Know EC2 and target group console locations

Confidence Level:

Ready For:

✓ Automatic scaling policies (next step)
✓ Production deployment (with monitoring)
✓ Troubleshooting scaling issues
✓ Monitoring ASG health
✓ Updating configurations
✓ Understanding automation foundation

Key Insights:

Learned:

✓ ASG creates identical instances from template
✓ Load balancer crucial for distributed traffic
✓ Health checks enable self-healing
✓ Grace period critical for app startup
✓ Manual scaling shows ASG power
✓ Activity history is key debugging tool
✓ Target group shows real-time health
✓ Connection draining protects requests
✓ Timeline is 5-15 minutes (be patient)
✓ Integration makes everything automatic

Next Steps:

Ready For:

✓ Adding scaling policies (automatic)
✓ Dashboard creation (monitoring)
✓ Alarm configuration (alerts)
✓ Production deployment (full setup)
✓ Multi-tier application (multiple ASGs)
✓ Blue-green deployments (gradual rollout)

Conclusion:

You've now experienced the complete ASG workflow manually. This foundation makes automatic scaling and advanced configurations intuitive. Master these basics, automate next!
```

---

## Conclusion

This hands-on guide walked you through creating, launching, and scaling an Auto Scaling Group in AWS from start to finish. You've learned the practical skills needed to understand ASG behavior, monitor its operations, and troubleshoot issues. The foundation is now set for automatic scaling policies and production deployments!
