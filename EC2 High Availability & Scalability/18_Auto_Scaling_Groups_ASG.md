# Auto Scaling Groups (ASG)

Comprehensive guide to AWS Auto Scaling Groups, including dynamic capacity management, scaling policies, launch templates, and integration with load balancers for automatic infrastructure scaling based on demand.

---

## Part 1: Auto Scaling Groups Overview

### Automated Capacity Management

**ASG Fundamentals**

```
What is an Auto Scaling Group (ASG):

Definition:
├─ Service: AWS EC2 Auto Scaling
├─ Purpose: Automatically adjust number of EC2 instances
├─ Based on: Metrics, schedules, or manual triggers
├─ Cost: ASG itself is free (pay for instances)
└─ Power: Automated infrastructure scaling

Core Capabilities:

Scale Out:
├─ Action: Add EC2 instances
├─ When: Demand increases
├─ Effect: Increased capacity and redundancy
└─ Speed: Minutes (instance creation time)

Scale In:
├─ Action: Remove EC2 instances
├─ When: Demand decreases
├─ Effect: Reduced costs
└─ Speed: EC2 termination time

Dynamic Sizing:
├─ Minimum: Floor on instance count
├─ Maximum: Ceiling on instance count
├─ Desired: Current target capacity
└─ Result: Size varies based on demand

Problem Solved:

Before ASG:
├─ Manual scaling: Call AWS API to add/remove instances
├─ Reactive: Scale after problems occur
├─ Overnight support: Requires someone awake
├─ Inefficient: Either overly generous (wasteful) or tight (risky)
└─ Cost: Overspending or underperformance

With ASG:
├─ Automatic: Scale based on metrics
├─ Proactive: Respond to changes in real-time
├─ 24/7: No manual intervention needed
├─ Efficient: Right-sized for current demand
└─ Cost: Pay only for what you need

Architecture Overview:

┌─────────────────────────────────────────────┐
│          Application Demand                  │
└──────────────┬──────────────────────────────┘
               │
               ▼
        ┌─────────────┐
        │  CloudWatch │
        │   Alarms    │
        └──────┬──────┘
               │
               ▼
        ┌─────────────┐
        │     ASG     │──── Launch Template (how to create)
        │  (Decision) │──── Min/Max/Desired (capacity limits)
        └──────┬──────┘──── Scaling Policies (rules)
               │
       ┌───────┴────────┐
       │                │
    Scale In         Scale Out
       │                │
       ▼                ▼
   Terminate        Launch
 EC2 Instances   EC2 Instances
       │                │
       └────────┬───────┘
              ▼
        ┌──────────────┐
        │ Registered   │
        │ with Load    │
        │ Balancer     │
        │ (if used)    │
        └──────────────┘
               │
               ▼
        Distribute Traffic
     (Automatic balancing)

Key Benefits:

High Availability:
├─ Redundancy: Multiple instances across AZs
├─ Failure: Instance dies → ASG replaces it
├─ Uptime: Applications stay available
├─ Detection: Health checks + replacement
└─ Result: Self-healing infrastructure

Cost Optimization:
├─ Efficiency: Scale down when not needed
├─ Hours: Reduce instances overnight
├─ Weekdays: Scale up during business hours
├─ Events: Handle traffic spikes efficiently
└─ Result: Lower AWS bills

Performance:
├─ Elasticity: Responsive to demand changes
├─ Latency: No degradation during scale-out
├─ Throughput: Increased during peaks
└─ SLA: Better uptime guarantees

Operational Excellence:
├─ Automation: No manual intervention
├─ Consistency: Identical instances launched
├─ Self-healing: Failed instances replaced
└─ Scalability: Linear with demand

When to Use ASG:

Perfect For:
├─ Web applications: Variable traffic
├─ APIs: Spiky load patterns
├─ Batch processing: Known capacity bumps
├─ Microservices: Per-service scaling
├─ Multi-tier: Scale different layers independently

Not Ideal For:
├─ Stateful applications: Session loss on scaledown
├─ Databases: Scaling stateless only
├─ Long-lived connections: Session affinity needed
└─ Legacy apps: Not cloud-native designed

Real-World Example:

Scenario: E-commerce site
├─ Off-peak: 2 AM - 9 AM (2 instances)
├─ Morning ramp: 9 AM - 11 AM (gradually scale to 5)
├─ Business hours: 11 AM - 5 PM (5-8 instances)
├─ Evening peak: 5 PM - 8 PM (8-10 instances)
├─ Night ramp: 8 PM - 2 AM (gradually scale down to 2)
├─ Holiday peak: Special event (20 instances)
├─ Cost: Right-sized for each period
└─ Performance: Never under-resourced

ASG Components:

1. Launch Template
   ├─ What: Blueprint for creating instances
   ├─ Contains: AMI, instance type, security groups, etc.
   └─ Use: Every instance launched from same template

2. Scaling Policies
   ├─ What: Rules for when to scale
   ├─ Types: Target tracking, step scaling, scheduled
   └─ Triggers: CloudWatch metrics, schedules

3. Capacity Settings
   ├─ Min: Minimum instances always running
   ├─ Max: Maximum instances that can run
   ├─ Desired: Current target count
   └─ Ranges: Define boundaries for scaling

4. Load Balancer Integration
   ├─ Registration: New instances auto-registered
   ├─ Health checks: Use LB health checks
   ├─ Deregistration: Remove unhealthy instances
   └─ Distribution: LB spreads traffic

5. Monitoring
   ├─ CloudWatch: Track metrics
   ├─ Alarms: Trigger scaling actions
   ├─ Logs: Record scaling events
   └─ Dashboards: Visualize trends

ASG States:

Healthy:
├─ Status: Instance passing health checks
├─ Traffic: Receiving load balancer traffic
├─ Action: None
└─ Monitoring: Continue normal operation

Unhealthy:
├─ Detection: Failed health check
├─ Status: Marked unhealthy by LB or ASG
├─ Action: ASG terminates and replaces
└─ Result: New healthy instance launched

Terminating:
├─ Status: Instance being shut down
├─ Reason: Scaling in, unhealthy, or manual
├─ Duration: Deregistration delay honored
└─ Result: Account for instance removed

Launching:
├─ Status: New instance starting up
├─ Duration: Boot time + health check passing
├─ Warm-up: Grace period before load balancing
└─ Result: Instance becomes healthy and receives traffic

ASG vs EC2 Instance:

EC2 Instance:
├─ Lifecycle: Manual - you control creation/termination
├─ Scaling: Manual - you add/remove instances
├─ Health check: Not automatic
├─ Replacement: Manual if it fails
└─ Use: One-off servers, long-term infrastructure

ASG:
├─ Lifecycle: Automatic - based on policies
├─ Scaling: Automatic - based on metrics
├─ Health check: Integrated with LB
├─ Replacement: Automatic if unhealthy
└─ Use: Dynamic workloads, elasticity

ASG vs ALB:

ALB (Load Balancer):
├─ Purpose: Distribute traffic
├─ Function: Routes requests
├─ Scaling: Different dimension (itself doesn't hold instances)
├─ When: Needed when multiple instances exist
└─ Requirement: Not required, but highly beneficial

ASG (Auto Scaling Group):
├─ Purpose: Manage instance count
├─ Function: Creates/destroys instances
├─ Scaling: Different dimension (capacity, not traffic)
├─ When: Needed for elasticity
└─ Requirement: Not required for ALB to work

Together:
├─ ALB: Distributes traffic among instances
├─ ASG: Adjusts number of instances
├─ Combination: Perfect partnership
└─ Result: Elastic, scalable, reliable system

Pricing:

ASG Cost:
├─ ASG itself: FREE
├─ EC2 instances: Pay as usual
├─ Unused: No charge for 0 instances
└─ Model: Pay for what you run

Example Cost Scenario:

Instance type: t3.medium ($0.042/hour)
Peak hours (10 hours/day): 5 instances
Offpeak (14 hours/day): 2 instances

Daily cost:
├─ Peak: 5 × $0.042 × 10 = $2.10
├─ Offpeak: 2 × $0.042 × 14 = $1.18
├─ Total: $3.28/day
├─ Monthly: ~$98.40 (at constant 5 avg)
└─ Savings: vs. constant 5 = $98.4, vs. constant 0 (overkill) = $126

ASG Workflow:

1. Create Launch Template
   └─ Define: How to launch instances

2. Create ASG
   ├─ Reference: Launch template
   ├─ Set: Min, max, desired capacity
   └─ Attach: Load balancer (if using)

3. Set Scaling Policies
   ├─ Define: When to scale in
   ├─ Define: When to scale out
   └─ Associate: CloudWatch alarms

4. Monitor and Adjust
   ├─ Watch: Metrics and alarms
   ├─ Observe: Scaling behavior
   ├─ Adjust: Policies if needed
   └─ Iterate: Optimize over time

5. Maintain and Update
   ├─ Version: Launch template versions
   ├─ Patch: Update AMI when needed
   ├─ Replace: Old instances with new version
   └─ Test: Before production deployment

Best Practices (Preview):

1. Use Launch Templates
   ├─ Not: Launch configurations (deprecated)
   └─ Benefit: Versioning, flexibility

2. Set Realistic Limits:
   ├─ Min: At least 1 (unless singleton)
   ├─ Max: Cost limits (prevent runaway)
   └─ Desired: Initial target

3. Use Health Checks:
   ├─ Type: ELB preferred with load balancer
   ├─ Grace: Allow time for warm-up
   └─ Frequency: Default usually good

4. Monitor Scaling Events:
   ├─ CloudWatch: Watch activities
   ├─ Alarms: Alert on issues
   └─ Logs: Debug scaling problems

5. Test Before Production:
   ├─ Scaling: Test scale-out and scale-in
   ├─ Replacement: Verify unhealthy instance replacement
   ├─ Performance: Monitor during scaling
   └─ Rollback: Have manual override
```

---

## Part 2: Capacity Settings

### Min, Max, and Desired Capacity

**Defining ASG Size Boundaries**

```
Capacity Parameters Overview:

Three Key Settings:

1. Minimum Capacity (Min Size)
   ├─ Definition: Minimum instances always running
   ├─ Lowest: 1 (always at least one)
   ├─ Default: Often 1 or higher
   ├─ Guarantee: ASG maintains at least this many
   ├─ Example: Min=2 means 2 instances always exist
   └─ Purpose: Ensure baseline redundancy

2. Maximum Capacity (Max Size)
   ├─ Definition: Maximum instances ASG can launch
   ├─ Limit: Cost and capacity ceiling
   ├─ Upper bound: ASG won't exceed this
   ├─ Example: Max=10 means never more than 10
   └─ Purpose: Prevent runaway scaling (cost control)

3. Desired Capacity (Initial Desired)
   ├─ Definition: Current/target instance count
   ├─ Starting point: How many at launch
   ├─ Dynamic: Changes based on scaling policies
   ├─ Range: Must be between Min and Max
   ├─ Example: Desired=4 means start with 4 instances
   └─ Purpose: Define initial state and target

Relationship:

Mathematical:
├─ Constraint: Min ≤ Desired ≤ Max
├─ Always: Min instances running minimum
├─ Never: Exceeds Max instances
├─ Can be: 0 if Min=0 (though unusual)
└─ Enforce: AWS prevents invalid configurations

Visual:

Min=2, Desired=4, Max=10:

┌────────────────────────────────────────────┐
│  0  1  2  3  4  5  6  7  8  9  10          │
└────────────────────────────────────────────┘
       └─ Min      └─ Desired └─ Max

Example States:
├─ Currently running: 4 instances
├─ Can scale down to: 2 (hit min)
├─ Can scale up to: 10 (hit max)
├─ Current target: 4 (desired)
└─ Scaling range: 2-10 instances

Example Configurations:

Configuration 1: Static (No Scaling)
├─ Min: 3
├─ Desired: 3
├─ Max: 3
├─ Effect: Always exactly 3 instances
├─ Scaling: Disabled effectively
└─ Use: Testing, specific workloads

Configuration 2: Flexible
├─ Min: 1
├─ Desired: 2
├─ Max: 5
├─ Effect: 1-5 instances available
├─ Scaling: Full range possible
└─ Use: Cost-optimized with elasticity

Configuration 3: Aggressive
├─ Min: 2
├─ Desired: 4
├─ Max: 20
├─ Effect: Starts with 4, scales up to 20
├─ Scaling: Lots of headroom
└─ Use: High-traffic applications

Configuration 4: Minimal
├─ Min: 0
├─ Desired: 0
├─ Max: 10
├─ Effect: Starts with 0 instances
├─ Scaling: Only up if needed
└─ Use: On-demand, event-driven workloads

Changing Desired Capacity:

Manual Change:
├─ How: AWS console, CLI, API
├─ Manual: Not automatic, you trigger
├─ Effect: Immediate (or as fast as launch)
└─ Use: On-demand scaling, testing

Automatic Change:
├─ How: Scaling policies with CloudWatch
├─ Based on: Metrics (CPU, memory, custom)
├─ Effect: Automatic based on conditions
└─ Use: Normal production operation

Scaling Out Example:

Initial State:
├─ Min: 2
├─ Current desired: 2
├─ Max: 10
├─ Instance count: 2

Trigger: High CPU alarm
├─ Alarm: Average CPU > 70%
├─ Action: Scale out (add 2 instances)

After Scale-Out:
├─ Min: 2 (unchanged)
├─ Current desired: 4 (increased)
├─ Max: 10 (unchanged)
├─ Instance count: 4 (now)
├─ Capacity: More instances serving traffic
└─ Effect: Reduced load per instance

Scaling In Example:

Start State:
├─ Min: 2
├─ Current desired: 10
├─ Max: 10
├─ Instance count: 10

Trigger: Low CPU alarm
├─ Alarm: Average CPU < 30% (continuing)
├─ Action: Scale in (remove 2 instances)

After Scale-In:
├─ Min: 2 (unchanged)
├─ Current desired: 8 (decreased)
├─ Max: 10 (unchanged)
├─ Instance count: 8 (now)
├─ Capacity: Fewer instances, cost reduced
└─ Effect: Reduced infrastructure cost

Reaching Limits:

Hitting Min:

Scenario:
├─ Current desired: 3
├─ Min set to: 2
├─ Scaling policy: Scale in by 2
├─ Request: Scale in (remove 2 instances)

Result:
├─ Calculation: 3 - 2 = 1 instance
├─ Min constraint: Would go below minimum of 2
├─ Action: Scale down only to 2 (min boundary)
└─ Actual result: Remove only 1 instance (to reach min)

Hitting Max:

Scenario:
├─ Current desired: 9
├─ Max set to: 10
├─ Scaling policy: Scale out by 5
├─ Request: Scale out (add 5 instances)

Result:
├─ Calculation: 9 + 5 = 14 instances
├─ Max constraint: Would exceed maximum of 10
├─ Action: Scale up only to 10 (max boundary)
└─ Actual result: Add only 1 instance (to reach max)

Warm-Up Period:

When Instance Launches:

Timeline:
├─ T=0s: ASG launches instance
├─ T=1m: Instance boots, application starts
├─ T=1m+: Application handling requests
├─ T=2m: Actually ready (variable by app)

Warm-Up Grace Period:
├─ Duration: Configured (default 300 seconds)
├─ Purpose: Don't count initial metrics
├─ Benefit: Avoid immediate scale-out due to startup load
└─ Example: CPU might spike during startup

Example:

Without Warm-Up:
├─ New instance starts
├─ CPU: High (JVM starting, caches building)
├─ Metric: 85% CPU (just starting!)
├─ Action: Scaling alarm triggers
├─ Result: Adds more instances (wrong!)

With 300s Warm-Up:
├─ New instance starts
├─ Metric: Ignored for 5 minutes
├─ Then: Normal metrics used
├─ Result: Correct scaling decisions

Desired Capacity Strategies:

Strategy 1: Start High, Scale Based on Demand
├─ Min: 2
├─ Initial Desired: 5
├─ Max: 20
├─ Scaling: Scale down when quieter
├─ Benefit: Fast response, always ready
├─ Cost: Higher baseline

Strategy 2: Start Low, Scale Based on Demand
├─ Min: 1
├─ Initial Desired: 1
├─ Max: 20
├─ Scaling: Scale up when needed
├─ Benefit: Cost-optimized
└─ Risk: Slow response to traffic spikes

Strategy 3: Balanced
├─ Min: 2
├─ Initial Desired: 3
├─ Max: 15
├─ Scaling: Scale both directions as needed
├─ Benefit: Good balance
└─ Target: Most common approach

Scaling When Changing Configuration:

Change Min from 2 to 4:

What Happens:
├─ Current running: 3 instances
├─ New minimum: 4 instances
├─ Action: Deploy 1 new instance immediately
├─ Effect: Maintain minimum capacity guarantee
└─ Result: 4 instances now running

Change Max from 10 to 8:

What Happens:
├─ Current running: 9 instances
├─ New maximum: 8 instances
├─ Action: Terminate 1 instance immediately
├─ Effect: Enforce new maximum
└─ Result: 8 instances now running

Change Desired from 2 to 6:

What Happens:
├─ Current running: 2 instances
├─ New target: 6 instances
├─ Action: Launch 4 new instances
├─ Effect: Reach target
└─ Result: 6 instances now running

Capacity Planning:

Determine Minimum:

Questions:
├─ Can users be served by 1 instance? N
├─ Need redundancy? Y (minimum 2 usually)
├─ Expected baseline traffic? 3 instances
├─ Answer: Set Min to at least 2 (redundancy)

Determine Maximum:

Questions:
├─ Peak traffic support: 20 instances needed
├─ Budget limit: $1000/month
├─ Instance cost: $50/month each
├─ Max affordable: 20 instances (20×$50)
├─ Answer: Set Max to 20

Determine Initial Desired:

Questions:
├─ Expected launch traffic: 100 requests/sec
├─ Per-instance capacity: 25 requests/sec
├─ Instances needed: 4
├─ Answer: Set Desired to 4

Real-World Examples:

Example 1: E-commerce Black Friday

Normal:
├─ Min: 2
├─ Desired: 3
├─ Max: 10

Black Friday:
├─ Min: 2 (never go below)
├─ Desired: 8 (prepare for surge)
├─ Max: 30 (allow big scaling)

Setup:
├─ Timeline: Scheduled scale up Friday 12am
├─ During: Monitor and adjust
├─ Post-event: Scale back down Monday
└─ Cost: Peak cost for peak demand

Example 2: API with Daily Pattern

Configuration:
├─ Min: 1
├─ Max: 10
├─ Desired: varies (schedule-based)

Daily Schedule:
├─ 12am-8am: Desired=1 (night, low traffic)
├─ 8am-9am: Desired=3 (morning ramp)
├─ 9am-5pm: Desired=6 (peak)
├─ 5pm-12am: Desired=3 (evening taper)

Savings:
├─ Offpeak: 1 instance (cheap)
├─ Peak: 6 instances (justified)
└─ Average: ~3.75 instances (cost optimized)

Example 3: Burst Workload

Configuration:
├─ Min: 0
├─ Max: 100
├─ Desired: 0 (starts empty)

When Event Happens:
├─ Trigger: Event starts
├─ Desired changes to: 50
├─ Instances: Launch rapidly
├─ Traffic: Served by fleet
├─ Duration: While event active

After Event:
├─ Trigger: Event ends
├─ Desired changes to: 0
├─ Instances: Terminate
├─ Cost: Only paid for active period
└─ Efficiency: Perfect match for demand

ASG Capacity Best Practices:

1. Set Realistic Minimums:
   ├─ Not: 0 for HA applications (no redundancy)
   ├─ Consider: Minimum for business continuity
   ├─ Set: Min to at least 1 (usually 2+)
   └─ Rationale: Failures need backup

2. Set Upper Bounds:
   ├─ Not: Unlimited (prevents cost runaway)
   ├─ Calculate: Maximum affordable cost
   ├─ Set: Max to cost-safe number
   └─ Buffer: Add 20% extra capacity room

3. Monitor Capacity Usage:
   ├─ Track: Running instances over time
   ├─ Analyze: Min/max/average
   ├─ Adjust: Based on actual patterns
   └─ Optimize: Over time

4. Plan for Failures:
   ├─ Min: More than 1 (unless stateless)
   ├─ Across: Multiple AZs
   ├─ Redundancy: Higher min helps
   └─ Availability: Better SLA with min>1

5. Test Limits:
   ├─ Scale: Test to max manually
   ├─ Verify: Performance doesn't degrade
   ├─ Check: No quota exceeded errors
   └─ Confidence: Know limits work in practice
```

---

## Part 3: Launch Templates

### Instance Configuration Blueprint

**Defining How to Launch Instances**

```
Launch Template Overview:

What It Is:

Definition:
├─ Template: Reusable configuration for instances
├─ Purpose: Define how each instance should be created
├─ Components: AMI, instance type, storage, networking, etc.
├─ Usage: Referenced by ASG to launch identical instances
└─ Benefit: Consistency across instance fleet

Why Launch Templates (Not Launch Configurations):

Launch Configuration (Deprecated):
├─ Status: Deprecated (still works but not updated)
├─ Versioning: Not supported
├─ Editing: Cannot edit (delete and recreate)
├─ Recommendation: Don't use for new projects

Launch Template (Recommended):
├─ Status: Current standard
├─ Versioning: Supports multiple versions
├─ Editing: Edit without recreating
├─ Recommendation: Use for all new ASGs
└─ Flexibility: More features and capabilities

Benefits of Launch Templates:

Versioning:
├─ Multiple: Create versions without creating new template
├─ Track: Each version separately
├─ Rollback: Easy revert to previous version
├─ Override: Can specify version when launching

Flexibility:
├─ Editing: Update template without recreating ASG
├─ Mix: Use different versions in same ASG if needed
├─ Default: Specify a default version
└─ Immutable: Create new for major changes

Reusability:
├─ Multiple ASGs: Same template for different ASGs
├─ On-Demand: Use template for manual launches too
├─ Sharing: Share template across teams/accounts
└─ Consistency: Guarantee identical instances

Launch Template Components:

Basic Configuration:

Template Name:
├─ Identifier: Unique name for template
├─ Format: Lowercase, hyphens allowed
├─ Example: web-app-template
└─ Purpose: Identify when creating ASG

Image (AMI):
├─ What: Amazon Machine Image
├─ Contains: OS, software, configuration
├─ Selection: Choose appropriate AMI
├─ Custom: Can use custom built AMIs
└─ Versioning: Select specific version

Instance Type:
├─ What: EC2 instance size/family
├─ Options: t3.micro, t3.small, m5.large, c5.xlarge, etc.
├─ Selection: Based on workload
├─ Flexibility: Can specify multiple types (ASG picks)
└─ Recommendation: Use flexible types for savings

Storage Configuration:

Root Volume (EBS):
├─ Size: Disk space for OS and app
├─ Type: gp2 (general), io1 (high I/O), st1 (throughput)
├─ Delete on Termination: Usually yes (clean up)
├─ Iops: Provisioned (io1/io2)
└─ Examples: 20GB for OS, 50GB for app

Additional Volumes:
├─ Not needed: For ASG typically
├─ If needed: Define in template
└─ Option: Add later if needed

Networking:

Security Group:
├─ What: Firewall rules for instance
├─ Selection: Choose security group
├─ Multiple: Can select multiple
├─ Ports: Defines ingress/egress rules
└─ Important: Correct SG for your workload

VPC & Subnet:
├─ Not in template: Usually ASG specifies
├─ Alternative: Can specify in template if needed
├─ Flexibility: ASG can override

Network Interfaces:
├─ Primary: Always created
├─ Secondary: Optional additional interfaces
├─ Typical: Just primary sufficient

Key Pair:

SSH Key:
├─ What: Authentication for EC2 access
├─ Specify: Which key pair to use
├─ Access: Connecting to instances
├─ Option: Can be added/removed later
└─ Note: Not recommended for production (use Systems Manager)

IAM Role:

What It Is:
├─ Permissions: What instance can do in AWS
├─ Examples: Read S3, write to DynamoDB, etc.
├─ Instance Profile: Created to associate with instance
└─ Recommended: Always use something

Why Needed:
├─ Credentials: Instance needs to authenticate to AWS
├─ Permissions: Define what actions allowed
├─ Security: Temporary credentials (automatic refresh)
└─ Practice: Best practice to use minimal permissions

Example IAM Roles:
├─ EC2-Role-S3-ReadOnly: Read S3 buckets
├─ EC2-Role-AppDeployment: Deploy from CodeDeploy
├─ EC2-Role-Monitoring: Write to CloudWatch
└─ EC2-Role-AppAccess: Application-specific permissions

Advanced Configuration:

User Data:

What It Is:
├─ Script: Runs when instance starts
├─ Language: Bash (Linux) or PowerShell (Windows)
├─ Purpose: Initial setup and configuration
├─ Runs once: Per instance launch
└─ Visible: In instance metadata

Example User Data:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```

Benefits:
├─ Automation: No manual configuration
├─ Consistency: Same setup for all instances
├─ Speed: Reduces time to ready
└─ Scalability: Enables ASG automation

Limitations:
├─ Executes: Only once at launch
├─ Permissions: Limited to instance user
├─ Logging: Needs manual setup for output
└─ Complexity: Avoid overly complex scripts

Monitoring (CloudWatch):

Enable Detailed Monitoring:
├─ Default: Basic monitoring (5-minute granularity)
├─ Detailed: Enhanced monitoring (1-minute granularity)
├─ Cost: Additional cost for detailed
├─ Recommendation: Enable for important metrics

Metrics Collected:
├─ CPU utilization
├─ Network in/out
├─ Disk read/write
├─ Status checks
└─ Custom: Via CloudWatch agent

Tagging:

Tags Overview:
├─ What: Key-value metadata for resources
├─ Purpose: Organization, billing, automation
├─ Examples: Environment=prod, Team=backend, CostCenter=123
└─ Propagate: Can propagate to volumes

Common Tags:
├─ Name: Human-readable identifier
├─ Environment: prod, staging, dev
├─ Application: app-name
├─ Team: team-name
├─ CostCenter: billing code
└─ Custom: As needed

Capacity Reservation:

Capacity Reservation:
├─ Scope: On-demand capacity reservation
├─ Purpose: Guarantee capacity availability
├─ Cost: Charged even if unused (or covered by savings plan)
├─ Advanced: Usually not needed for ASG
└─ Use case: Critical workloads with capacity concerns

Creating Launch Template:

Step 1: Open AWS Console
├─ Service: EC2
├─ Region: Select your region
└─ Left menu: Launch Templates

Step 2: Create New Template
├─ Button: "Create launch template"
├─ Page: Template creation form
└─ Ready: Start filling information

Step 3: Basic Information
├─ Template name: web-app-v1
├─ Description: (optional) Web app production template
└─ Checkbox: "Provide guidance" (helpful for first time)

Step 4: Application Image (AMI)
├─ Click: "Browse images" or search
├─ Filter: Find your AMI
├─ Category: Amazon Linux, Ubuntu, Custom, Marketplace
├─ Select: Choose specific AMI
└─ Version: Latest or specific version

Step 5: Instance Type
├─ Select: t3.medium (example)
├─ Flexibility: Can select multiple types
└─ Option: Let ASG choose based on availability

Step 6: Key Pair
├─ Select: Available key pair
├─ Or: Create new (if needed)
├─ Option: May be optional for some use cases
└─ Note: For EC2 Instance Connect preferred over SSH key

Step 7: Security Group
├─ Select: Existing security group
├─ Or: Create new (if needed)
├─ Rules: Review rules (what ports open?)
└─ Example: Port 80 (HTTP), 443 (HTTPS), 22 (SSH)

Step 8: Storage (EBS)
├─ Root: Usually /dev/xvda
├─ Size: 20-50GB typical for web apps
├─ Type: gp2 (general purpose)
├─ Delete: Check "Delete on termination" (usually yes)
└─ Iops: Default for gp2 sufficient

Step 9: IAM Role
├─ Select: Choose role for instances
├─ Or create: If needed for application
├─ Permission: Minimum permissions principle
└─ Example: Role for CodeDeploy agent, CloudWatch access

Step 10: User Data
├─ Paste: Your initialization script (if needed)
├─ Format: Bash script for Linux
├─ Testing: Test script before using
└─ Logging: Consider redirecting output

Step 11: Monitoring
├─ Checkbox: "Detailed monitoring"
├─ Cost: Slightly higher
├─ Benefit: 1-minute granularity
└─ Recommendation: Enable for often-scaling apps

Step 12: Tagging
├─ Add tags: Environment, Application, Team, etc.
├─ Key-Value: Enter tag information
├─ Propagate: To volumes (checkbox)
└─ Purpose: Organization and billing

Step 13: Review and Create
├─ Review: All settings
├─ Verify: Settings are correct
├─ Create: Launch template
└─ Ready: Now reference in ASG

Launch Template Versions:

Why Multiple Versions:

Scenario:
├─ V1: Ubuntu 20.04 with app version 1.0
├─ Release: App version 2.0 available
├─ Decision: Update to new version
├─ Choice: Create new version or new template?
└─ Answer: Create new version (easy rollback)

Creating New Version:

From Existing:
1. Find: Template in AWS Console
2. Actions: "Create version from this template"
3. Changes: Make updates (new AMI, instance type, etc.)
4. Create: Save as new version
5. Version number: Auto-assigned (V1, V2, V3, etc.)

Using Different Versions:

In ASG:
├─ ASG points to template
├─ Can use: $Latest (default) or specific version
├─ Upgrade: ASG uses new instances from latest
├─ But: Existing instances unchanged until replaced
└─ Strategy: Controlled rollout via replacement

Rollback:

If Issue with V2:
├─ Switch: ASG to use V1 again
├─ Effect: New launches use V1
├─ Existing: V2 instances continue running
├─ Replace: Manually or via scaling actions
└─ Option: Terminate V2 instances to force replace

Template Example:

Web Application Template:

Name: ecommerce-web-v2

AMI:
├─ Ubuntu 22.04 base
├─ Pre-installed: Nginx

Instance Type:
├─ Defaults to: t3.medium
├─ Option: Multiple types allowed

Security Group:
├─ Port 80: HTTP from anywhere
├─ Port 443: HTTPS from anywhere
├─ Port 22: SSH from office IPs only
└─ Database: Restricted to internal SG

Storage:
├─ Root: 30GB gp2
└─ Delete on termination: Yes

IAM Role:
├─ Permissions: Read S3 for assets
├─ CloudWatch: Write application logs
└─ Policy: Minimal required

User Data:
```bash
#!/bin/bash
apt-get update
apt-get install -y nginx
systemctl start nginx
# Deploy application from S3
aws s3 cp s3://app-bucket/app-v2.tar .
tar -xf app-v2.tar
```

Tags:
├─ Name: ecommerce-web-prod
├─ Environment: production
├─ Application: ecommerce
├─ Version: v2
└─ CostCenter: sales-engineering

Monitoring: Enabled

Template Best Practices:

1. Version Management:
   ├─ Never: Edit existing versions
   ├─ Always: Create new version for changes
   ├─ Naming: Include version in template name or description
   └─ Tracking: Document what changed in each version

2. Minimize User Data:
   ├─ Prefer: Bake into AMI
   ├─ Simple: Keep user data script simple
   ├─ Fast: Reduces instance startup time
   └─ Reliable: Pre-tested in AMI vs script

3. Security:
   ├─ Secrets: Never hardcode in user data
   ├─ IAM: Use roles, not keys
   ├─ SG: Minimal required access
   └─ Key pair: Consider EC2 Instance Connect instead

4. Tagging:
   ├─ Consistent: Use same tags across templates
   ├─ Complete: Tag every template
   ├─ Automation: Use for resource organization
   └─ Billing: Track costs by tag

5. Testing:
   ├─ Launch: Test template before ASG use
   ├─ Verify: User data executes correctly
   ├─ Performance: Check startup time
   └─ IAM: Verify permissions sufficient

```

---

## Part 4: Launch and Scaling

### Getting Instances into ASG

**Instance Lifecycle and Scaling Actions**

```
Creating ASG:

Prerequisites:

Before ASG:
├─ Launch template: Must exist
├─ VPC: Where ASG deploys
├─ Subnets: In which AZs/subnets
├─ Load balancer (optional): If using one
└─ Security group: Defined in template

Step 1: Access ASG Console

In AWS Console:
├─ Service: EC2
├─ Left menu: Auto Scaling Groups
├─ Button: "Create Auto Scaling group"
└─ Begin: ASG creation wizard

Step 2: Choose Launch Template

Select Template:
├─ Dropdown: Choose your template
├─ Version: Default ($Latest) or specific
├─ Verify: Correct template selected
└─ Next: Proceed

Step 3: Configure Group Details

Group Name:
├─ Name: ecommerce-web-asg
├─ Description: (optional)
└─ Important: Identifiable, describes purpose

Capacity:

Minimum:
├─ Minimum capacity: 2
├─ Purpose: Always at least 2 instances
└─ Reason: Redundancy/HA

Desired:
├─ Initial desired: 3
├─ Purpose: Start with 3 instances
└─ Will scale: Based on policy later

Maximum:
├─ Maximum capacity: 10
├─ Purpose: Never more than 10
└─ Reason: Cost control

VPC/Subnets:
├─ VPC: Select your VPC
├─ Subnets: Choose AZs and subnets
├─ Why: Multi-AZ for HA
└─ Example: us-east-1a, us-east-1b, us-east-1c

Step 4: Configure Advanced Options

Load Balancer:

Option 1: No Load Balancer
├─ Direct access: Via instance IPs
├─ Not recommended: For web apps

Option 2: Attach Load Balancer
├─ Select: ALB or NLB
├─ Target group: Choose target group
├─ Instances auto-register: When launched
├─ Health checks: Use LB health checks
└─ Recommended: For production

Health Checks:

Type:

Option 1: EC2 Status Checks
├─ Source: EC2 service
├─ Checks: Instance status and system status
├─ Only: EC2 level
└─ Limitation: Doesn't check application

Option 2: ELB Health Checks
├─ Source: Load balancer
├─ Checks: Application responding correctly
├─ Requires: LB attached
├─ Recommended: Preferred option
└─ Benefit: Application level

Grace Period:
├─ Duration: Seconds to wait for health
├─ Default: 300 seconds (5 minutes)
├─ Purpose: Warm-up time for instances
├─ Set to: Your application startup + buffer

Termination Policies:
├─ Default: Balance between AZs, terminate older
├─ Custom: Can specify preferred termination order
└─ Advanced: Usually leave default

Step 5: Add Tags

Instance Tags:
├─ Propagate: Spread to instances
├─ Tags: Same as template or additional
├─ Examples: Name, Environment, etc.
└─ Benefit: Auto-tagging of instances

Step 6: Review and Create

Verify:
├─ Group name: Correct
├─ Capacity: Min, max, desired set
├─ Template: Correct version
├─ VPC/Subnets: Right AZs
├─ Load balancer: If needed
└─ Health check: Grace period appropriate

Create:
├─ Button: "Create Auto Scaling group"
└─ Status: ASG created and scaling activity begins

ASG State After Creation:

Initial State:
├─ Capacity: Launching desired instances
├─ Instances: Being created
├─ Timeline: 5-10 minutes typically
└─ Status: "Initial" in ASG details

Launching Instances:

Activity:
├─ Count: Launching X instances
├─ Time: Per instance 1-2 minutes
├─ Parallel: Multiple launch simultaneously
└─ Total: Desired capacity reached

Health Checks:

Verification:
├─ Each instance: Health check initiated
├─ Grace period: Ignored during warm-up
├─ After grace: Actual health check
└─ Status: "Healthy" when passing

Load Balancer Registration:

If LB Attached:
├─ Instances: Auto-registered to target group
├─ Traffic: Will receive requests once healthy
├─ Distribution: LB spreads traffic
└─ Scaling: New instances auto-register

Ready State:
├─ Status: ASG in service
├─ Capacity: At desired level
├─ Instances: Healthy and receiving traffic
└─ Ready: For automatic scaling

Instance Lifecycle in ASG:

States:

1. Pending
   ├─ Status: Being launched
   ├─ Duration: 1-2 minutes
   ├─ Traffic: Not receiving yet
   └─ Next: Moving to InService

2. InService
   ├─ Status: Running and healthy
   ├─ Duration: Until termination
   ├─ Traffic: Receiving requests
   └─ Health: Monitored continuously

3. Terminating
   ├─ Status: Being shut down
   ├─ Reason: Scaling in, unhealthy, or manual
   ├─ Duration: Respects deregistration delay
   └─ Next: Terminated (removed)

4. Terminated
   ├─ Status: No longer running
   ├─ Reason: Previous termination action
   ├─ Visible: In termination history (temporarily)
   └─ Duration: Removed from ASG

Health Check Replacement:

When Unhealthy:

Detection:
├─ Failure: Health check fails
├─ Action: Mark as unhealthy
├─ Duration: Grace period respected first time
└─ Status: "Unhealthy" in ASG details

Termination:
├─ Wait: Deregistration delay (for LB drain)
├─ Action: Terminate instance
├─ Reason: Automatic replacement
└─ Duration: Quick

Replacement:
├─ Trigger: New instance launched
├─ Count: Maintain desired capacity
├─ Timeline: Replacement starts immediately
└─ Effect: Transparent to users (load on others)

Example Timeline:

T=10:00:00: Instance running, healthy

T=10:05:00: Health check fails
├─ Status: Mark unhealthy
└─ Action: Initiate replacement

T=10:05:30: Deregistration delay starts
├─ Reason: Allow in-flight requests to finish
└─ Duration: 300 seconds (default)

T=10:10:30: Termination completes
├─ Action: Instance shutdown
├─ Replacement: New instance launching
└─ Status: ASG desired capacity maintained

T=10:12:00: New instance healthy
├─ Status: InService
├─ Traffic: Receiving requests
└─ Result: Automatic healing complete

Scaling In (Remove Instances):

Trigger:

Metric Condition:
├─ Example: CPU < 25% for 5 minutes
├─ Alarm: Triggered
├─ Action: Initiate scale in
└─ Cooldown: Wait before next scaling action

Scaling In Action:
├─ Count: How many to remove?
├─ Decision: Based on policy
├─ Example: Remove 1 instance
└─ Capacity: Desired - 1

Termination Selection:

How ASG Chooses Which to Terminate:

Default Strategy (Balanced):
├─ 1. Most instances in oldest launch config
├─ 2. Most instances in oldest launch template version
├─ 3. Instance closest to next billing hour
└─ Result: Predictable, fair termination

Example:
├─ Instance A: V1 template, launched 10 min ago
├─ Instance B: V2 template, launched 5 min ago
├─ Instance C: V2 template, launched 8 min ago
├─ Termination: Instance A (oldest V1)
└─ Reason: Oldest launch template first

Termination Process:

Graceful:
├─ Step 1: Deregister from LB (drain)
├─ Step 2: Respect deregistration delay
├─ Step 3: Terminate instance
├─ Step 4: Desired capacity maintained
└─ Duration: 5-10 minutes total

Effect on Users:
├─ Minimal: In-flight requests complete
├─ Transparent: New traffic routed to other instances
├─ Health: Service continues
└─ Experience: Seamless

Scaling Out (Add Instances):

Trigger:

Metric Condition:
├─ Example: CPU > 70% for 2 minutes
├─ Alarm: Triggered
├─ Action: Initiate scale out
└─ Cooldown: Wait before next action

Scaling Out Action:
├─ Count: How many to add?
├─ Decision: Based on policy
├─ Example: Add 2 instances
└─ Capacity: Desired + 2

Launching Process:

Steps:
├─ 1. Launch template: retrieved
├─ 2. AZ: Selected (balanced across AZs)
├─ 3. Instance: Created
├─ 4. Warm-up: Grace period ignored
├─ 5. Health check: Verify passing
├─ 6. LB registration: Auto-register if LB
├─ 7. Traffic: Start receiving requests
└─ Duration: 5-10 minutes total

Effect on Users:
├─ Immediate: More capacity available
├─ Gradual: Load transfers to new instances
├─ Performance: Should improve
└─ Load: Reduced per instance

Cooldown Period:

Purpose:
├─ Prevent: Rapid scale in/out cycles
├─ Stability: Allow metrics to stabilize
├─ Cost: Reduce unnecessary scaling
└─ Efficiency: Smart pacing of scaling actions

Duration:
├─ Default: 300 seconds (5 minutes)
├─ Configurable: Per scaling policy
├─ When: Starts after scaling action
└─ Effect: No new scaling during this time

Example:

T=10:00:00: CPU high, scale out (add 2)
├─ Loading: 2 new instances

T=10:05:00: Cooldown ends
├─ CPU: Maybe still high
├─ Allow: More scaling possible

T=10:07:00: CPU still high, scale out (add 2 more)
├─ OK: After cooldown

vs

Without Cooldown:

T=10:00:00: CPU high, scale out (add 2)
T=10:00:30: More metrics, scale out (add 2 more)
T=10:01:00: Repeat quickly
└─ Result: Cascade of scaling (wasteful)

Successful Scaling Example:

Time: 9:00 AM (Business hours starting)

Setup:
├─ Min: 2
├─ Desired: 2
├─ Max: 10
└─ Scaling policy: Scale out if CPU > 70%

9:00 - 9:15 AM:

Trigger:
├─ Traffic: Increasing
├─ CPU on instances: Rising
├─ Average: 75% CPU

Action:
├─ Alarm: Triggered (> 70%)
├─ Scale out: Add 2 instances
├─ Desired: 2 → 4
├─ Status: Launching 2 instances

9:10 - 9:20 AM:

Status:
├─ Total: 4 instances running
├─ CPU: Distributed, now average 60%
├─ Performance: Improved
├─ Load: Now manageable
└─ Cooldown: Preventing more scaling

9:20 - 9:30 AM:

Traffic: Continues high
├─ CPU: Still average 72%
├─ Alarm: Triggered again (after cooldown)
├─ Scale out: Add 2 more instances
├─ Desired: 4 → 6
└─ Status: Launching 2 more

By 9:35 AM:

Final State:
├─ Total: 6 instances running
├─ CPU: Distributed, average 55%
├─ Performance: Excellent
├─ Traffic: Being served efficiently
└─ Users: Fast response times

Manual Scaling:

Override Automatic:

When Needed:
├─ Test: Testing scaling manually
├─ Emergency: Need immediate scaling
├─ Anomaly: Metrics not triggering as expected
└─ Adjustment: Change desired directly

How:

AWS Console:
1. Find: ASG in Auto Scaling Groups
2. Edit: Group details
3. Desired: Change capacity number
4. Save: Apply change
5. Effect: Immediate scaling action

CLI:
```bash
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 5
```

Immediate Effect:
├─ If higher: Launches instances
├─ If lower: Terminates instances
└─ Duration: Respects graceful termination
```

---

## Part 5: Scaling Policies and CloudWatch

### Automatic Scaling Decisions

**Metrics, Alarms, and Scaling Triggers**

```
Scaling Policies Overview:

Purpose:
├─ Goals: Define when and how to scale
├─ Based on: CloudWatch metrics or schedules
├─ Actions: Scale in or out automatically
├─ Benefit: No manual intervention
└─ Smart: Responds to actual demand

Three Policy Types:

1. Target Tracking
   ├─ Simplest: Specify target metric value
   ├─ AWS manages: Scaling adjustments
   ├─ Example: Keep CPU at 50%
   ├─ Automatic: Scales up/down as needed
   └─ Recommended: Most common

2. Step Scaling
   ├─ Granular: Different actions by metric level
   ├─ Example: Scale by 1 if 60-70% CPU, by 2 if 70-80%
   ├─ Manual: You define each step
   ├─ Flexible: Fine-grained control
   └─ Advanced: More configuration

3. Simple Scaling
   ├─ Basic: Single condition triggers single action
   ├─ Example: If CPU > 70%, add 1 instance
   ├─ Simple: Easiest to understand
   ├─ Limited: Less flexibility
   └─ Deprecated: Step scaling preferred

CloudWatch Metrics:

Common Metrics for Scaling:

CPU Utilization:
├─ Metric: Average CPU percentage
├─ Source: EC2 monitoring
├─ Range: 0-100%
├─ Example threshold: > 70% triggers scale out
├─ Good for: CPU-bound workloads

Network In/Out:
├─ Metric: Network bandwidth
├─ Source: EC2 monitoring
├─ Use: High I/O applications
├─ Example: Scale out if bandwidth > 10 MB/s

Custom Metrics:
├─ Application: Any metric from your app
├─ Examples: Active connections, queue depth, requests/sec
├─ Source: Application sends to CloudWatch
├─ Advantage: Very specific to your workload
└─ Benefit: More accurate scaling decisions

ALB/ELB Metrics:

Request Count:
├─ Metric: Requests per minute per target
├─ Source: Load balancer
├─ Use: API traffic-based scaling
├─ Example: Scale out if > 1000 req/min per target

Target Response Time:
├─ Metric: Average time to respond
├─ Source: Load balancer
├─ Use: Performance-based scaling
├─ Example: Scale out if latency > 1 second

Unhealthy Hosts:
├─ Metric: Number of unhealthy targets
├─ Source: Load balancer
├─ Use: Health-based scaling
├─ Example: Scale out if unhealthy > 1

CloudWatch Alarms:

What They Are:

Definition:
├─ Monitor: CloudWatch metric
├─ Threshold: Trigger value
├─ State: Normal, Alarm, or Insufficient Data
├─ Action: What to do when state changes
└─ Notification: Alert via SNS or execute action

States:

OK:
├─ Meaning: Metric within normal range
├─ Action: None (or none specified)
└─ Duration: Continuous until threshold crossed

Alarm:
├─ Meaning: Metric exceeded threshold
├─ Action: Trigger scaling action
├─ Duration: Until metric returns to normal

Insufficient Data:
├─ Meaning: Not enough data points
├─ Reason: Metric just started, no history
├─ Duration: Until sufficient data collected
└─ Effect: Alarm won't trigger yet

Creating Alarms:

For Scaling Out (High Load):

Metric:
├─ Name: CPU Utilization
├─ Statistic: Average
├─ Period: 300 seconds (5 minutes)
└─ Threshold: >= 70%

For Scaling In (Low Load):

Metric:
├─ Name: CPU Utilization
├─ Statistic: Average
├─ Period: 300 seconds
└─ Threshold: <= 25%

Evaluation Periods:
├─ How many: Periods must exceed threshold
├─ Example: 2 periods of 300s = 10 minutes
├─ Purpose: Prevent spurious scaling
└─ Balance: Not too sensitive, not too slow

Target Tracking Scaling:

Simplest Approach:

Concept:
├─ Metric: Choose one metric
├─ Target: Specify desired value
├─ AWS: Scales to maintain it
└─ Example: Keep CPU at 50%

How It Works:

AWS Calculation:
├─ Current: Metric value now
├─ Target: Desired value
├─ Difference: Current - Target
├─ Adjustment: Scale up/down to close gap
└─ Continuous: Maintains close to target

Example:

Configuration:
├─ Metric: Average CPU
├─ Target: 50%

Scenario 1:
├─ Current: 65% CPU
├─ Gap: 65 - 50 = 15% over
├─ Action: Scale out (add instances)
└─ Effect: CPU drops toward 50%

Scenario 2:
├─ Current: 30% CPU
├─ Gap: 30 - 50 = 20% under
├─ Action: Scale in (remove instances)
└─ Effect: CPU rises back toward 50%

Advantages:

Simplicity:
├─ One metric, one target
├─ AWS handles scaling math
├─ Less configuration needed
└─ Easy to understand and maintain

Responsiveness:
├─ Continuous: Always working toward target
├─ Automatic: Scale up/down as needed
├─ Balanced: Not too aggressive
└─ Stable: Reduces oscillation

Common Values:

Recommended Targets:
├─ CPU: 50-70% (balance)
├─ Request count: Based on per-target capacity
├─ Latency: Time-based threshold

Step Scaling:

More Granular Control:

Concept:
├─ Steps: Different actions at different levels
├─ Example: +1 if 60%, +2 if 70%, +3 if 80%
├─ Conditional: Different responses
└─ Complex: More configuration

How It Works:

Define Steps:

Scale Out Steps:
├─ Step 1: If 60% <= CPU < 70%, add 1 instance
├─ Step 2: If 70% <= CPU < 80%, add 2 instances
├─ Step 3: If 80% <= CPU < 90%, add 3 instances
├─ Step 4: If CPU >= 90%, add 4 instances
└─ Proportional: Bigger problems = bigger response

Scale In Steps:
├─ Step 1: If 40% >= CPU > 30%, remove 1 instance
├─ Step 2: If 30% >= CPU > 20%, remove 2 instances
│  └─ More aggressive: Going down faster
└─ Limit: Don't go below min

When to Use:

Step Scaling:
├─ Workload: Predictable patterns of load
├─ Response: Different actions at different levels
├─ Fine-tuning: Want specific control
└─ Example: Financial trading app (very sensitive)

Target Tracking:
├─ Workload: Variable and unpredictable
├─ Response: Consistent metric target
├─ Simple: Less configuration
└─ Example: Web application (standard)

Scheduled Actions:

Predictable Patterns:

When:
├─ Know: Exactly when traffic increases
├─ Example: 9 AM workday starts, 5 PM workday ends
├─ Repeat: Same pattern daily or weekly
└─ Predictive: Plan scaling in advance

Action:

Scale Out Scheduled:
├─ Time: 9:00 AM every weekday
├─ Action: Set desired capacity to 5
├─ Effect: Ready before traffic arrives
└─ Benefit: No waiting for metrics to trigger

Scale In Scheduled:
├─ Time: 6:00 PM every day
├─ Action: Set desired capacity to 2
├─ Effect: Reduce cost after business hours
└─ Benefit: No waiting, cost savings immediate

Example Schedule:

Configuration:

Morning Ramp:
├─ Action 1: 7:00 AM set desired to 3
├─ Action 2: 8:00 AM set desired to 5
└─ Effect: Gradual ramp before 9 AM start

Business Hours:
├─ Action: 9:00 AM set desired to 8
└─ Effect: Full capacity for peak hours

Evening Scale:
├─ Action: 6:00 PM set desired to 3
└─ Effect: Reduce after hours

Night Mode:
├─ Action: 10:00 PM set desired to 1
└─ Effect: Minimum overnight

Combining Policies:

Best Practice:

Use Both:
├─ Scheduled: Handle predictable patterns
├─ Target Tracking: Handle unexpected spikes
└─ Together: Best of both worlds

Example:

9 AM - 5 PM (Scheduled):
├─ Schedule: Set desired to 5
├─ Policy: Still tracks CPU, can scale above if needed
└─ Effect: Ready for expected peak, scales more if needed

5 PM - 9 AM (Scheduled):
├─ Schedule: Set desired to 2
├─ Policy: Still tracks CPU, won't allow scaling below min
└─ Effect: Cost-optimized, but responsive if needed

Monitoring Scaling Activities:

Activity History:

Location:
├─ AWS Console → ASG → Activity
├─ Shows: All scaling actions
└─ Details: What scaled, when, why

Example Entry:

Timestamp: 2023-12-15 10:30:00
├─ Event: "Launching 2 instances"
├─ Reason: "Target Tracking Scaling Policy ALB Target Average CPU"
├─ Status: "Successful"
└─ Result: Instances in InService

CloudWatch Logs:

Metrics to Monitor:

ASG Size:
├─ Metric: Current capacity
├─ Trend: How it changes over time
├─ Alerts: Excessive scaling
└─ Insight: Is policy working?

Scaling Actions:
├─ Count: How many scale-out/scale-in activities?
├─ Frequency: Are they happening often?
├─ Alerts: Too frequent = unstable
└─ Adjustment: May need policy tuning

Alarm States:
├─ Metric: When alarms trigger
├─ Threshold: How often exceeded
├─ Trend: Increasing/stable/decreasing
└─ Analysis: Required threshold adjustment

Metrics Dashboard:

Key Visualizations:

Graph 1: ASG Size Over Time
├─ Y-axis: Number of instances
├─ X-axis: Time
├─ Shows: Scaling pattern
└─ Pattern: Should match expected behavior

Graph 2: CPU Utilization
├─ Y-axis: Average CPU %
├─ X-axis: Time
├─ Shows: Load pattern
└─ Stability: Should track near target

Graph 3: Unhealthy Instances
├─ Y-axis: Count
├─ X-axis: Time
├─ Shows: Instance health
└─ Alert: Any spikes indicate issues

Example Dashboard:

Date: Monday, December 15

Time: 6 AM
├─ Instances: 1 (night minimum)
├─ CPU: 15% (very low, overnight)

Time: 9 AM
├─ Scheduled: Scale to 4
├─ Instances: Rising (launching)
├─ CPU: Rising (traffic arriving)

Time: 10 AM
├─ Instances: 4 (reached scheduled)
├─ CPU: 62% (under target 70%)
├─ Scaling: Stable

Time: 12 PM (noon)
├─ Instances: 5 (auto-scaled for lunch spike)
├─ CPU: 68% (near target)
├─ Pattern: Expected lunch traffic

Time: 6 PM
├─ Scheduled: Scale to 2
├─ Instances: Declining (terminating)
├─ CPU: Stabilizing

Time: 10 PM
├─ Instances: 1 (night minimum)
├─ CPU: 12% (low)
└─ Trend: Perfect daily pattern

Troubleshooting Scaling Issues:

Problem: Not Scaling Out

Investigation:

Check:
├─ Policy: Is it enabled?
├─ Alarm: Is it in Alarm state?
├─ Metric: Is it actually high?
├─ Capacity: Already at maximum?
└─ Cooldown: In cooldown period?

Solution:
├─ Verify: Policy settings
├─ Check: Alarm configuration
├─ Lower: Max capacity if trying there
├─ Wait: Cooldown period to end
└─ Test: Manual scale to verify

Problem: Scaling Too Aggressively

Investigation:

Check:
├─ Threshold: Set too low?
├─ Cooldown: Too short?
├─ Scale increment: Scaling too many at once?
├─ Oscillation: Rapidly scaling up/down?

Solution:
├─ Raise: Threshold (higher CPU needed)
├─ Increase: Cooldown period
├─ Reduce: Instances added per scale-out
├─ Stabilize: Policy settling

Problem: Unhealthy Instances

Investigation:

Check:
├─ Application: Is it healthy?
├─ Monitoring: What's the error?
├─ Health check timeout: Grace period correct?
└─ Traffic: Too much for new instance?

Solution:
├─ Fix: Application issue
├─ Extend: Grace period
├─ Reduce: Instances added per action
└─ Monitor: Application startup logs

Scaling Policy Best Practices:

1. Start Conservative:
   ├─ Target: Higher-than-expected threshold
   ├─ Reason: Avoid over-scaling
   ├─ Monitor: Real metrics for days
   └─ Adjust: Once comfortable

2. Use Target Tracking:
   ├─ Simplicity: Easier to manage
   ├─ AWS: Does calculations
   ├─ Effective: Works for most cases
   └─ Recommended: Default choice

3. Monitor Continuously:
   ├─ Dashboard: Live view
   ├─ Alerts: For abnormal behavior
   ├─ Logs: Review events
   └─ Adjust: Based on real data

4. Combine Policies:
   ├─ Scheduled: For known patterns
   ├─ Target Tracking: For spike response
   ├─ Together: Best coverage
   └─ Example: Daytime scheduled + always tracking

5. Test Before Production:
   ├─ Staging: Test scaling first
   ├─ Verify: Behavior as expected
   ├─ Performance: Check during scale-out/in
   └─ Confidence: Know it works
```

---

## Part 6: Exam Focus and Integration

### Critical Concepts and Best Practices

**ASG for AWS SysOps Certification**

```
Essential ASG Facts for Exam:

Definitions:

Auto Scaling Group:
✓ Automatically add/remove EC2 instances
✓ Based on metrics, schedules, or manual action
✓ Cost: ASG is free (pay for instances)
✓ Integrated: With load balancers
✓ Self-healing: Replaces unhealthy instances

Scaling:

Scale Out:
✓ Add instances
✓ During high demand
✓ Increase capacity

Scale In:
✓ Remove instances
✓ During low demand
✓ Reduce costs

Capacity Parameters:

Min Size:
✓ Minimum instances always running
✓ Lowest floor
✓ Guarantee baseline capacity

Max Size:
✓ Maximum allowed instances
✓ Upper ceiling
✓ Cost control limit

Desired Capacity:
✓ Current target number
✓ Between min and max
✓ Changes automatically based on policy

Launch Template:

What:
✓ Blueprint for instance creation
✓ Contains: AMI, instance type, SG, user data, IAM role
✓ Reusable: Multiple instances from same template
✓ Versions: Multiple versions supported

Components:
✓ AMI: Which OS/software
✓ Instance type: Size (t3.small, m5.large, etc.)
✓ Security group: Firewall rules
✓ Storage: EBS volumes
✓ IAM role: Permissions
✓ User data: Initialization script
✓ Tags: Metadata

Scaling Policies:

Target Tracking:
✓ Simplest to use
✓ Specify: Target metric value
✓ AWS manages: Scaling adjustments
✓ Example: Keep CPU at 50%

Step Scaling:
✓ Granular control
✓ Different actions at different levels
✓ More configuration
✓ Example: Add 1 if 60%, add 2 if 70%

Scheduled:
✓ Executed at specific times
✓ Predictable patterns
✓ Example: Scale up 9 AM, down 5 PM

CloudWatch Alarms:

Role:
✓ Monitor metrics
✓ Trigger actions
✓ Send notifications

Metrics Used:
✓ CPU utilization
✓ Network in/out
✓ Request count
✓ Target response time
✓ Custom metrics

Health Checks:

Types:

EC2 Status:
✓ Instance running and healthy (at EC2 level)
✓ Doesn't check application

ELB Status:
✓ Application responding correctly
✓ Load balancer level
✓ Recommended when LB attached

Grace Period:
✓ Seconds to wait before health check
✓ Default: 300 seconds (5 minutes)
✓ Purpose: Allow warm-up

Termination:

Unhealthy Instance:
✓ ASG detects failing health check
✓ Terminates instance
✓ Launches replacement
✓ Maintains desired capacity

Selection:
✓ ASG chooses which to terminate
✓ Balanced: Fair distribution
✓ Old: Older instances first

Integration with Load Balancer:

Benefits:
✓ Auto-register: New instances to LB
✓ Health checks: Integrated
✓ Drain: Graceful deregistration
✓ Traffic: Distributed automatically

Without LB:
✓ Still works: ASG manages instances
✓ Limited: No load distribution
✓ Direct IP: Users access instances directly

With LB:
✓ Powerful: Combined capabilities
✓ Auto-register: New instances
✓ Auto-deregister: Unhealthy or terminated
✓ Recommended: For production

Exam Scenario Questions:

1. "What happens when an ASG instance fails?"
   A) Service degrades, manual RDP needed
   B) ASG terminates failover instance, creates new one
   C) Manual intervention required to replace
   D) Service stops until manual restart
   
   Answer: B) ASG terminates unhealthy instance, creates new one
   └─ Know: Self-healing capability

2. "Min=2, Max=10, Desired=5. Can ASG scale to 15 instances?"
   A) Yes, if demand is high
   B) No, max is limit
   C) Yes if policy allows
   D) Only if manually triggered
   
   Answer: B) No, max is limit
   └─ Know: Max is ceiling, enforced

3. "What would you use for consistent, constant load?"
   A) Min=Max=Desired (same value)
   B) Min=0, Max=100
   C) Desired=1
   D) No capacity parameters
   
   Answer: A) Min=Max=Desired (same value)
   └─ Know: Static setup

4. "Which does launch template NOT contain?"
   A) AMI ID
   B) Instance type
   C) Load balancer IP
   D) Security group
   
   Answer: C) Load balancer IP
   └─ Know: ASG references LB separately

5. "What is the free component of ASG?"
   A) Everything (EC2, storage, LB)
   B) Just ASG service
   C) ASG + load balancer
   D) Nothing (all costs)
   
   Answer: B) Just ASG service
   └─ Know: Pay for instances launched

6. "Scale-out triggered when CPU > 70%. What does 'cooldown' do?"
   A) Prevents scaling for set period
   B) Delays all actions
   C) Cools down instances
   D) Resets metrics
   
   Answer: A) Prevents scaling for set period
   └─ Know: 300 seconds default

7. "How does ASG choose which instance to terminate?"
   A) Random
   B) Balanced, then oldest launch template
   C) Newest instances first
   D) User chooses
   
   Answer: B) Balanced, then oldest launch template
   └─ Know: Fair, predictable termination

8. "ALB target instances not receiving requests. Why?"
   A) ALB failed
   B) Instances unhealthy or starting up
   C) ASG error
   D) Policy misconfiguration
   
   Answer: B) Instances unhealthy or starting up
   └─ Know: Grace period and health checks

9. "Which scaling policy simplest to implement?"
   A) Step scaling
   B) Simple scaling
   C) Target tracking
   D) Custom metrics
   
   Answer: C) Target tracking
   └─ Know: AWS manages the math

10. "ASG can replace unhealthy instances automatically?"
    A) Only if load balancer attached
    B) Only if manually triggered
    C) Always, with health checks
    D) Not possible
    
    Answer: C) Always, with health checks
    └─ Know: Self-healing capability

Common Scenarios:

Scenario 1: Website with Heavy Traffic 9-5

Solution:
├─ Min: 2 (HA redundancy)
├─ Max: 20 (cost ceiling)
├─ Desired: Initially 3
├─ Scheduled scale-up: 9 AM to 8
├─ Scheduled scale-down: 6 PM to 3
├─ Policy: Target track CPU 60%
└─ Result: Responsive to spikes, cost-optimized

Scenario 2: Background Job Processing

Solution:
├─ Min: 0 (only when needed)
├─ Max: 50 (large capacity)
├─ Desired: Start 0
├─ Scale policy: Queue depth > 100 items
├─ Scale down: Queue < 10 items
└─ Result: Cost-effective, event-driven

Scenario 3: Microservice API

Solution:
├─ Min: 2 (redundancy)
├─ Max: 15
├─ Desired: 3
├─ Policy: Target track request count per target 100
├─ Health: ELB-based
└─ Result: Always responsive

Scenario 4: Database-Heavy App

Solution:
├─ Min: 4 (connect pooling)
├─ Max: 10
├─ Desired: 5
├─ Policy: Conservative (CPU target 40%)
├─ Reason: DB connections limited
└─ Result: Protect database from overload

Architecture Combining Components:

Internet Traffic
    ↓
  ALB (distribute)
    ↓
  ASG (manage instances)
    ├─ Min 2, Max 10, Desired 3
    ├─ LaunchTemplate (how to create)
    ├─ Scaling Policies (when to scale)
    └─ Health Checks (replace unhealthy)

CloudWatch (monitor)
    ├─ Metrics (CPU, network, custom)
    ├─ Alarms (thresholds)
    └─ Dashboard (visualization)

Result: Elastic, self-healing, scalable system

ASG Best Practices:

1. Use Load Balancer:
   ├─ Always: For web/API applications
   ├─ Benefit: Auto-register, health checks
   └─ Result: Full ASG power

2. Define Health Checks:
   ├─ Use: Load balancer health checks
   ├─ Grace: Appropriate warm-up time
   └─ Result: Auto-replacement of failed instances

3. Set Realistic Capacity:
   ├─ Min: Never 0 (unless on-demand only)
   ├─ Max: Based on budget
   ├─ Desired: Expected load
   └─ Result: Right-sized infrastructure

4. Use Scaling Policies:
   ├─ Start: Target tracking
   ├─ Add: Scheduled for known patterns
   ├─ Monitor: Adjust based on results
   └─ Result: Automatic, optimized scaling

5. Monitor and Alert:
   ├─ Dashboard: Watch metrics
   ├─ Alarms: Unusual behavior
   ├─ Logs: Scaling events
   └─ Result: Early detection of issues

6. Test Thoroughly:
   ├─ Staging: Test scaling before production
   ├─ Verify: Manual scaling works
   ├─ Check: Health replacement works
   ├─ Monitor: During production rollout
   └─ Result: Confidence in process

7. Document Process:
   ├─ Config: Record capacity settings
   ├─ Policies: Document scaling rules
   ├─ Runbook: How to troubleshoot
   └─ Result: Easy maintenance and support

Comparing ASG to Alternatives:

Manual Scaling:
├─ Simple: Easy to understand
├─ Control: You decide everything
├─ Manual: Requires someone watching
├─ Costly: Slow to respond to spikes

ASG Scheduled:
├─ Predictable: Works for known patterns
├─ Granular: Control exact timing
├─ Limited: Only at specified times
├─ Good: For business hours scaling

ASG Automatic (Policy-based):
├─ Responsive: Reacts to actual metrics
├─ Autonomous: No manual intervention
├─ Intelligent: Based on real data
├─ Recommended: Default approach

ASG + LB Together:
├─ Powerful: Full elasticity
├─ Complete: All pieces integrated
├─ Automated: Everything automatic
├─ Best: For modern applications
```

---

## Conclusion

Auto Scaling Groups provide the foundation for elastic, self-healing infrastructure in AWS. Combined with load balancers and monitoring, ASGs enable applications to respond automatically to changing demand while maintaining reliability and optimizing costs. Master these concepts for AWS SysOps certification and production deployment success!
