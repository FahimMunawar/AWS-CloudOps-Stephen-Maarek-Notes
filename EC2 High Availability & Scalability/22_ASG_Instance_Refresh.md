# ASG Instance Refresh: Rolling Updates and AMI Management

Comprehensive guide to Auto Scaling Group Instance Refresh feature, enabling zero-downtime rolling updates of all instances in an ASG with new launch templates, including minimum healthy percentage management, warm-up configuration, and production deployment patterns.

---

## Part 1: Instance Refresh Overview and Concepts

### Rolling Updates Without Downtime

**Understanding the Instance Refresh Feature**

```
Instance Refresh Overview:

What It Is:

Definition:
├─ Feature: Native Auto Scaling Group capability
├─ Purpose: Update all instances with new launch template
├─ Pattern: Gradual rolling replacement
├─ Downtime: Zero (continuous service)
└─ Automation: Fully managed by AWS

Problem It Solves:

Challenge:
├─ Scenario: 10 EC2 instances running
├─ Update needed: New AMI (security patch, app update)
├─ Old method: Manually terminate each instance
├─ Risk: Downtime if not careful
├─ Duration: Hours of manual work

Solution:
├─ Use: Instance Refresh
├─ Automation: AWS does the rolling update
├─ Safety: Maintains minimum capacity
├─ Speed: Parallelized terminations
└─ Result: All instances updated automatically

Why It's Useful:

Maintenance Needs:
├─ Security patches: Update OS
├─ Application updates: Deploy new version
├─ Configuration changes: Update user data
├─ AMI refresh: New base image
└─ All: Require new launch template

Benefits:
├─ Automation: No manual instance management
├─ Safety: Controlled termination pace
├─ Availability: Maintains capacity during update
├─ Speed: Faster than manual
├─ Reliability: Less error-prone
└─ Transparency: Activity tracked and monitorable

How It Works:

High-Level Flow:

┌─────────────────────────────────────┐
│     Auto Scaling Group               │
│   (10 instances with V1 template)    │
└──────────────┬──────────────────────┘
               │
               ▼
    Admin: Create new launch template V2
    (Updated AMI, same config)
               │
               ▼
    Admin: Start Instance Refresh API
    (Specify min healthy %, warm-up time)
               │
               ▼
       Instance Refresh Begins
    ┌─────────────────────────────┐
    │ Phase 1: Terminate batch    │
    │ (per min healthy %)         │
    │ Old instances go down       │
    │ Load drops temporarily      │
    └─────────────────────────────┘
               │
               ▼
    ┌─────────────────────────────┐
    │ Phase 2: Launch batch       │
    │ (matching instances) gone   │
    │ New instances boot          │
    │ User data runs              │
    │ Joins load balancer         │
    └─────────────────────────────┘
               │
               ▼
    ┌─────────────────────────────┐
    │ Phase 3: Warmup wait        │
    │ (specified warm-up seconds) │
    │ New instances stabilize     │
    │ Ready for real traffic      │
    └─────────────────────────────┘
               │
               ▼
    ┌─────────────────────────────┐
    │ Phase 4: Next batch         │
    │ Repeat for remaining        │
    │ Continue rolling            │
    │ Until all instances updated │
    └─────────────────────────────┘
               │
               ▼
    ┌─────────────────────────────┐
    │ Complete: All instances     │
    │ running new launch template │
    │ Same capacity maintained    │
    │ Zero downtime achieved      │
    └─────────────────────────────┘
```

Common Use Cases:

Scenario 1: Security Patch

Situation:
├─ Current: 10 instances with Ubuntu 20.04 (vulnerable)
├─ Update: Ubuntu 20.04 with latest patches
├─ Done via: New AMI (patched)
├─ Launch template: Updated to use new AMI
└─ Implementation: Instance Refresh

Process:
├─ Step 1: Create new AMI (patched OS)
├─ Step 2: Create new launch template (referencing new AMI)
├─ Step 3: Start Instance Refresh
├─ Step 4: AWS terminates instances in batches
├─ Step 5: New instances boot with patched OS
├─ Step 6: Refresh completes

Result:
├─ All 10 instances: Now patched
├─ Uptime: 100% (rolling refresh)
├─ Duration: 30-60 minutes (depending on instance startup)
└─ Status: No service interruption

Scenario 2: Application Deployment

Situation:
├─ Current: 8 instances with application v1.0
├─ Release: Application v1.1
├─ Deployment method: New AMI with v1.1 pre-installed
├─ Template update: New launch template with new AMI
└─ Deployment: Instance Refresh

Process:
├─ Step 1: Build new Docker image with v1.1
├─ Step 2: Create new AMI (packer + Docker image)
├─ Step 3: Update launch template
├─ Step 4: Start Instance Refresh
├─ Step 5: Old instances (v1.0) terminated
├─ Step 6: New instances (v1.1) launched
├─ Step 7: Complete - all running v1.1

Result:
├─ Deployment: Automated and safe
├─ Version: Consistent (all same)
├─ Time: Minutes (not manual hours)
└─ Confidence: Low risk deploy

Scenario 3: Configuration Update

Situation:
├─ Current: 5 instances with old user data
├─ Change: Update environment variables in user data
├─ Implementation: New launch template user data
├─ Instances: Need to restart with new config
└─ Solution: Instance Refresh

Process:
├─ Step 1: Create new launch template (updated user data)
├─ Step 2: Start Instance Refresh
├─ Step 3: Instances terminate and restart
├─ Step 4: New config active on all instances
└─ Refresh: Complete

Result:
├─ Configuration: Consistent across all
├─ Downtime: None (rolling)
└─ Effort: Automatic

How Different from Manual Replacement:

Manual Method (Old Way):

Steps:
├─ 1: Terminate instance 1 manually
├─ 2: Wait for new instance to boot
├─ 3: Verify health
├─ 4: Terminate instance 2
├─ 5: Wait and verify
├─ ... repeat for all 10 instances

Duration:
├─ Each iteration: 5 minutes (boot + verify)
├─ 10 instances: 50 minutes
└─ Risk: Manual errors, monitoring burden

Issues:
├─ Tedious: Repetitive manual steps
├─ Slow: Sequential (one at a time)
├─ Error-prone: Manual participation
├─ Monitoring: Constant watching needed
└─ Stressful: Real-time attention required

Automated Refresh Method (New Way):

Steps:
├─ 1: Create new launch template
├─ 2: Click "Start Instance Refresh"
├─ 3: Specify min healthy % and warm-up
├─ 4: AWS handles everything
└─ Done: Sit back and monitor

Duration:
├─ Parallel: Multiple instances at once
├─ 10 instances: 20-30 minutes (2-3 parallel)
└─ Fast: By design parallelized

Benefits:
├─ Simple: Just click a button
├─ Predictable: Consistent timing
├─ Safe: AWS-managed pace
├─ Observable: Clear activity history
└─ Hands-off: No manual intervention needed

Terminology:

Important Terms:

Launch Template Version:
├─ Definition: Snapshot of template at specific time
├─ V1: Original template
├─ V2: After modification
├─ Reason: Enables rollback if needed
└─ Use: Instance Refresh references specific version

Minimum Healthy Percentage:
├─ Definition: % of instances that must healthy
├─ Example: 60% means can remove up to 40%
├─ Purpose: Ensures capacity maintained
└─ Impact: Affects refresh speed

Warm-up Time (or Warm-up Period):
├─ Definition: Seconds to wait before considering instance ready
├─ Duration: Application-specific (30-300 seconds typical)
├─ Purpose: Allow instance to stabilize
└─ Effect: Before marking as "in service"

Instance Refresh Status:

Possible States:

Pending:
├─ Meaning: Refresh about to start
├─ Duration: Seconds
└─ Next: Active

Active (or In Progress):
├─ Meaning: Refresh currently running
├─ Duration: Minutes to hours
├─ Activity: Instances terminating/launching
└─ Observable: Activity history shows batches

Successful:
├─ Meaning: Refresh completed successfully
├─ Result: All instances updated
├─ Verification: All running new template
└─ Status: Check ASG shows new template

Failed or Cancelled:
├─ Meaning: Refresh stopped (error or manual cancel)
├─ Status: Mixed (some old, some new)
├─ Recovery: Manual intervention or new refresh
└─ Error: Check activity history for details

Architecture Diagram:

Instance Refresh Process:

┌────────────────────────────────────────────────────────┐
│                  Auto Scaling Group                      │
│                                                          │
│  Instance Refresh Configuration:                        │
│  ├─ Min Healthy %: 60%                                 │
│  ├─ Warm-up Time: 120 seconds                          │
│  └─ New Template: V2 (updated AMI)                     │
│                                                          │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Before Refresh:                                        │
│  ────────────────────────────────────────────────────  │
│  Old Template (V1):                                     │
│  Instance-1 ──────────┐                                │
│  Instance-2 ──────────┤                                │
│  Instance-3 ──────────┤                                │
│  Instance-4 ──────────┤────► ALB/NLB                   │
│  Instance-5 ──────────┤      Load Balancer             │
│  Instance-6 ──────────┤                                │
│  Instance-7 ──────────┘                                │
│  Instance-8 (spare)                                     │
│                                                          │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Refresh Batch Strategy (60% healthy):                 │
│  Can terminate: 40% (max 3 of 8 instances)             │
│  Must keep: 60% (min 5 running)                        │
│                                                          │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Batch 1 Termination:                                  │
│  Instance-1 ──────────┐                                │
│  Instance-2 ──────────┤                                │
│  Instance-3 ──────────┤                                │
│  Instance-4 (GONE)    │  ┌─► TERMINATED (old template)│
│  Instance-5 ──────────┤                                │
│  Instance-6 ──────────┘                                │
│  Instance-7 (GONE)    │  ┌─► TERMINATED (old template)│
│  Instance-8 (spare)                                     │
│                                                          │
│  Healthy Count: 5/8 = 62.5% ✓ (above 60% minimum)     │
│  Load Distribution: 5 instances carrying traffic       │
│                                                          │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Batch 1 Launch:                                       │
│  New-Instance-1 ─────┐                                 │
│  New-Instance-2 ─────┤  (launching with V2 template)  │
│  New-Instance-3 ─────┤  (taking 30-90s to boot)       │
│  Instance-1 ─────────┤                                 │
│  Instance-2 ─────────┤────► ALB/NLB                    │
│  Instance-3 ─────────┤      Load Balancer              │
│  Instance-5 ─────────┤                                 │
│  Instance-6 ─────────┘                                 │
│                                                          │
│  Wait: Warm-up Time (120 seconds)                      │
│  Purpose: Allow new instances to stabilize            │
│                                                          │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Batch 1 Complete:                                     │
│  New-Instance-1 ─────┐                                 │
│  New-Instance-2 ─────┤  (ready and healthy)           │
│  New-Instance-3 ─────┤  (registered with LB)          │
│  Instance-1 ─────────┤                                 │
│  Instance-2 ─────────┤────► ALB/NLB                    │
│  Instance-3 ─────────┤      Load Balancer              │
│  Instance-5 ─────────┤                                 │
│  Instance-6 ─────────┘                                 │
│                                                          │
│  Healthy Count: 8/8 = 100% ✓                          │
│  Ready: Next batch can begin                           │
│                                                          │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Batch 2 Repeats (Instance-8, others):                 │
│  Same termination/launch/warm-up cycle                │
│                                                          │
├────────────────────────────────────────────────────────┤
│                                                          │
│  Final State:                                          │
│  New-Instance-1 ─────┐                                 │
│  New-Instance-2 ─────┤                                 │
│  New-Instance-3 ─────┤                                 │
│  New-Instance-4 ─────┤  (all V2 template)             │
│  New-Instance-5 ─────┤  (all updated)                 │
│  New-Instance-6 ─────┤────► ALB/NLB                    │
│  New-Instance-7 ─────┤      Load Balancer              │
│  New-Instance-8 ─────┘                                 │
│                                                          │
│  ✓ Refresh Complete                                    │
│  ✓ All instances: V2 template                          │
│  ✓ Zero downtime observed                              │
│  ✓ Capacity maintained throughout                      │
│                                                          │
└────────────────────────────────────────────────────────┘
```

Instance Refresh vs Other Deployment Methods:

Comparison with Alternatives:

Method 1: Blue-Green

How:
├─ Create: New ASG (green)
├─ Migrate: Traffic from old (blue) to new
├─ Delete: Old ASG after successful test
└─ Cost: Running both ASGs temporarily

Pros:
├─ Instant rollback: Switch back to old
├─ Testing: Full test before cutover
├─ Risk: Very low
└─ Duration: One DNS/LB change

Cons:
├─ Cost: 2x infrastructure temporarily
├─ Complexity: Manage two ASGs
├─ Overkill: For simple updates
└─ Time: Longer due to setup

Use: Major versions, risky changes

Method 2: Canary Deployment

How:
├─ Update: Small % of instances
├─ Monitor: Performance and errors
├─ Gradual: Increase % if good
├─ Full: Switch 100% after validation
└─ Manual: Requires monitoring and decisions

Pros:
├─ Testing: Real traffic validation
├─ Risk mitigation: Slow rollout
├─ Rollback: Switch back if issues
└─ Confidence: Validated before full

Cons:
├─ Complex: Multiple states
├─ Monitoring: Continuous attention
├─ Duration: Hours or days
├─ Manual: Not fully automated
└─ Mixed: Different versions during deployment

Use: High-risk changes, performance-critical

Method 3: Instance Refresh (Rolling Update)

How:
├─ Terminate: Batch of instances
├─ Launch: New ones with new template
├─ Wait: Warm-up time
├─ Repeat: Next batch
└─ Automatic: Fully AWS-managed

Pros:
├─ Simplicity: One command (or click)
├─ Speed: Parallelized (multiple batches)
├─ Safety: Minimum healthy % maintained
├─ Automatic: No manual monitoring
└─ Standard: Most common for updates

Cons:
├─ No instant rollback: Requires new refresh
├─ Mixed versions: During refresh period
├─ Downtime: Possible if % too low
└─ Commitment: Starts process

Use: Normal updates, security patches, routine deployments

Method 4: Manual Termination

How:
├─ Terminate: One instance
├─ Wait: New one boots
├─ Repeat: For each instance
└─ Manual: All done by hand

Pros:
├─ Control: Full visibility at each step
├─ Flexibility: Can pause/stop anytime
├─ Learning: Understand each step
└─ No automation: Simple to understand

Cons:
├─ Tedious: Repetitive steps
├─ Time-consuming: Hours for many instances
├─ Error-prone: Manual work risk
├─ Monitoring: Constant attention
└─ Not scalable: Doesn't work well for 50+ instances

Use: Learning, emergency situations, small deployments

Choosing the Right Method:

Decision Table:

Update Type | Risk Level | Scale | Timeline | Method
─────────────────────────────────────────────────────
AMI patch   | Low        | 5-20  | 1 hour   | Refresh
OS security | Low        | 5-20  | 1 hour   | Refresh
Config      | Low        | 5-20  | 1 hour   | Refresh
App v minor | Medium     | 5-20  | 30 min   | Refresh
App v major | High       | 5-20  | Hours    | Blue-Green
New feature | High       | Any   | Hours    | Canary
Emergency   | Unknown    | Any   | Minutes  | Manual

Best Practice:
├─ Default: Use Instance Refresh
├─ If risky: Use Blue-Green or Canary
├─ If learning: Manual
└─ Most common: Refresh (90% of updates)
```

---

## Part 2: Minimum Healthy Percentage Configuration

### Controlling Update Pace and Capacity

**Understanding and Setting Min Healthy Percentage**

```
Minimum Healthy Percentage Explained:

What It Means:

Definition:
├─ Calculation: (Healthy instances / Total instances) × 100
├─ Controls: How many instances can terminate at once
├─ Purpose: Ensures capacity never drops too low
├─ Timing: Applied to each refresh batch
└─ Impact: Determines refresh speed

Simple Example:

Scenario 1: 10 instances, 70% minimum healthy

Calculation:
├─ Total: 10 instances
├─ Minimum healthy: 70% of 10 = 7 instances
├─ Can terminate at once: 10 - 7 = 3 instances
└─ Constraint: At least 7 must be running

Scenario 2: 4 instances, 60% minimum healthy

Calculation:
├─ Total: 4 instances
├─ Minimum healthy: 60% of 4 = 2.4 → 3 instances (rounded up)
├─ Can terminate at once: 4 - 3 = 1 instance
└─ Constraint: At least 3 must be running

Scenario 3: 20 instances, 50% minimum healthy

Calculation:
├─ Total: 20 instances
├─ Minimum healthy: 50% of 20 = 10 instances
├─ Can terminate at once: 20 - 10 = 10 instances
└─ Constraint: Can terminate half at once

Impact on Refresh Duration:

Lower Percentage (Faster Refresh):

Example: 50% minimum, 10 instances

Timeline:
├─ Batch 1: Terminate 5, launch 5 (t=0 to 5 min)
├─ Wait: Warm-up time (t=5 to 7 min)
├─ Batch 2: Terminate 5, launch 5 (t=7 to 12 min)
├─ Wait: Warm-up time (t=12 to 14 min)
├─ Complete: (t=14 min total)
└─ Speed: Very fast, high parallelization

Trade-off:
├─ Capacity: Drops to 50% during refresh
├─ Slower: More connections queuing
├─ Risk: Higher load per instance
└─ Risk: If issues, more instances affected

Use when:
├─ Workload: Can handle reduced capacity
├─ Load: Low during update window
├─ Predictability: Update at off-peak
└─ Instance: Beefy (can handle extra)

Higher Percentage (Slower Refresh):

Example: 90% minimum, 10 instances

Timeline:
├─ Batch 1: Terminate 1, launch 1 (t=0 to 5 min)
├─ Wait: Warm-up time (t=5 to 7 min)
├─ Batch 2: Terminate 1, launch 1 (t=7 to 12 min)
├─ Wait: Warm-up time (t=12 to 14 min)
├─ Batch 3: Terminate 1, launch 1 (t=14 to 19 min)
├─ ... continue for 10 batches
├─ Complete: (t=~100 min total)
└─ Speed: Very slow, sequential

Trade-off:
├─ Capacity: Always at 90%+ (safe)
├─ Performance: Consistent throughout
├─ Risk: Very low (safe margin)
└─ Duration: Takes much longer (hours)

Use when:
├─ Workload: Cannot handle reduced capacity
├─ SLA: Strict availability requirement
├─ Load: High or business-critical
├─ Stability: Peak hours (no reduction acceptable)

Common Percentages and Scenarios:

50% Minimum Healthy:

Characteristics:
├─ Speed: Very fast (2 batches for 10 instances)
├─ Capacity: Half during update (5/10)
├─ Load: 2x per instance temporarily
├─ Risk: Medium (single failure during batch)
└─ Duration: 15-30 minutes

When to use:
├─ Off-peak updates: Midnight or 3 AM
├─ Stateless apps: No session affinity
├─ Beefy instances: t3.large or bigger
├─ Predictable load: Know it's low
└─ Example: Batch processing, background jobs

NOT for:
├─ Business hours
├─ Stateful applications
├─ Sensitive services
├─ Unknown load patterns

75% Minimum Healthy:

Characteristics:
├─ Speed: Fast (3-4 batches for 10)
├─ Capacity: 75%+ maintained
├─ Load: 25% temporarily distributed
├─ Risk: Low-medium
└─ Duration: 45-60 minutes

When to use:
├─ Normal business hours
├─ Typical workload: Spans moderate hours
├─ Standard instances: t3.medium
├─ Balanced: Speed vs stability
└─ Example: Web applications, most services

Popular choice:
├─ Default recommended
├─ Works well for most
├─ Safe and reasonable
└─ Standard practice

100% Minimum Healthy (Slowest but Safest):

Characteristics:
├─ Speed: Very slow (sequential)
├─ Capacity: Always 100% (no change)
├─ Load: No increase (each instance: stable)
├─ Risk: Minimal (always at full)
└─ Duration: Hours (1+ hours for 10 instances)

When to use:
├─ Peak hours: Can't reduce
├─ Critical: Business-essential
├─ Database: Stateful applications
├─ Strict SLA: 99.99% uptime
├─ Example: Payment processing, financial services

Cost:
├─ Duration: Much longer (more hours of instances)
├─ Consider: Set time limit
└─ Monitoring: More time to watch

Calculation and Rounding:

How AWS Rounds:

Rule: Round Up (Conservative)

Example 1:
├─ Instances: 10
├─ Percentage: 65%
├─ Calculation: 10 × 0.65 = 6.5
├─ Rounded: 7 instances (up)
└─ Effect: Terminate max 3 at once

Example 2:
├─ Instances: 7
├─ Percentage: 50%
├─ Calculation: 7 × 0.5 = 3.5
├─ Rounded: 4 instances (up)
└─ Effect: Terminate max 3 at once

Rounding Direction:
├─ Why: Errs on conservative side
├─ Effect: Safer (more instances running)
├─ Result: Slightly slower refresh
└─ Benefit: Lower risk

Real-world Examples:

Example 1: Web Application (10 instances)

Setup:
├─ Total instances: 10
├─ Minimum healthy: 75%
├─ Warm-up: 60 seconds
├─ Refresh timing: Off-peak (2 AM)

Calculation:
├─ Healthy threshold: 10 × 0.75 = 7.5 → 8 instances
├─ Can terminate: 10 - 8 = 2 per batch
└─ Batches needed: 5 (to refresh all 10)

Timeline:
├─ 0:00: Batch 1 - terminate 2 instances
├─ 0:05: Launch 2 new instances
├─ 1:05: Warm-up done, Batch 2 starts
├─ 1:10: Terminate 2 instances
├─ 1:15: Launch 2 new instances
├─ 2:15: Warm-up done, Batch 3 starts
├─ 2:20: Terminate 2 instances
├─ 2:25: Launch 2 new instances  
├─ 3:25: Warm-up done, Batch 4 starts
├─ 3:30: Terminate 2 instances
├─ 3:35: Launch 2 new instances
├─ 4:35: Warm-up done, Batch 5 starts
├─ 4:40: Terminate final 2 instances
├─ 4:45: Launch final 2 new instances
├─ 5:45: Warm-up done, refresh complete
└─ Total duration: ~5 hours 45 minutes

Capacity Throughout:
├─ Starting: 10/10
├─ After terminate: 8/10 (80% of desired)
├─ After launch: 8 old + 2 new = 10/10
├─ Consistently: 8-10 instances running
└─ Maintained: Always ≥75%

Example 2: Batch Processing (20 instances)

Setup:
├─ Total instances: 20
├─ Minimum healthy: 50%
├─ Warm-up: 120 seconds
├─ Refresh timing: Weekend (low batch load)

Calculation:
├─ Healthy threshold: 20 × 0.50 = 10 instances
├─ Can terminate: 20 - 10 = 10 per batch
└─ Batches needed: 2 (very fast)

Timeline:
├─ 0:00: Batch 1 - terminate 10 instances
├─ 0:05: Launch 10 new instances
├─ 1:50: Warm-up done (120 sec)
├─ 1:55: Batch 2 - terminate 10 old instances
├─ 2:00: Launch 10 new instances
├─ 3:45: Warm-up done
├─ 3:45: Complete
└─ Total duration: ~3 hours 45 minutes

Capacity Throughout:
├─ Starting: 20/20
├─ After terminate: 10/20 (50%)
├─ After launch: 10 old + 10 new = 20/20
├─ Second batch: Similar pattern
└─ Maintained: Exactly 50% minimum

Speed:
├─ Only: 2 batches (very quick)
├─ Parallelization: 10 at a time
└─ Safe: 50% capacity maintained during batch job processing

Example 3: Critical Financial System (4 instances)

Setup:
├─ Total instances: 4
├─ Minimum healthy: 100%
├─ Warm-up: 300 seconds
├─ Refresh timing: Maintenance window (planned downtime)

Calculation:
├─ Healthy threshold: 4 × 1.00 = 4 instances
├─ Can terminate: 4 - 4 = 0 at once (only 1 at a time)
└─ Batches needed: 4 (sequential)

Timeline:
├─ 0:00: Batch 1 - terminate 1 instance (1 old remains + 3 old)
├─ 0:05: Launch 1 new instance (3 old + 1 new)
├─ 5:05: Warm-up done (300 sec)
├─ 5:10: Batch 2 - terminate 1 old instance
├─ 5:15: Launch 1 new instance
├─ 10:15: Warm-up done
├─ 10:20: Batch 3 - terminate 1 old instance
├─ 10:25: Launch 1 new instance
├─ 15:25: Warm-up done
├─ 15:30: Batch 4 - terminate last old instance
├─ 15:35: Launch last new instance
├─ 20:35: Warm-up done
└─ Total duration: ~20 hours (!) - very slow but safe

Capacity Throughout:
├─ Always: 4/4 running (100%)
├─ Never: Any reduction
├─ Safety: Maximum
└─ Trade-off: Duration is very long

Finding Your Sweet Spot:

Decision Process:

Step 1: Assess Your Workload

Questions:
├─ Can you reduce capacity by 50%? → Yes: 50% ok
├─ Can you reduce capacity by 25%? → Yes: 75% ok
├─ Cannot reduce capacity at all? → 90-100% needed
└─ Peak hours + capacity-sensitive? → 80-90%

Step 2: Choose Initial Percentage

Conservative Approach:
├─ Start high: 80-90%
├─ Monitor: Actual duration
├─ If slow: Dial back to 75%
└─ Iterative: Adjust based on experience

Aggressive Approach:
├─ Start reasonable: 75%
├─ Monitor: Any issues (capacity, errors)?
├─ If no: Keep it
└─ If performance issues: Increase to 80-85%

Step 3: Test in Dev/Staging

Before Production:
├─ Stage identical: ASG setup
├─ Run test refresh: With chosen %
├─ Monitor: Duration and behavior
├─ Load test: Verify capacity enough
└─ Validate: Before production use

Step 4: Adjust Based on Observation

After Testing:
├─ If too slow (>1 hour): Lower %
├─ If concerns about capacity: Raise %
├─ If errors during batch: Raise %
├─ If smooth: Stick with it
└─ Document: Your chosen %

Best Practice Recommendations:

By Workload Type:

Static Content / CDN Edge:
├─ Minimum: 50%
├─ Reason: Inherently redundant
└─ Comfortable: Can handle half

Web API Servers:
├─ Minimum: 70-75%
├─ Reason: Moderate user load
└─ Balanced: Speed vs capacity

Database Servers:
├─ Minimum: 90-100%
├─ Reason: Stateful, cannot reduce
└─ Critical: Always full

Stateless Services:
├─ Minimum: 60-70%
├─ Reason: Flexible, no affinity
└─ Quick: Parallel refresh ok

Stateful Applications:
├─ Minimum: 80-90%
├─ Reason: Session maintenance
└─ Conservative: Safe reduction

Batch/Background Jobs:
├─ Minimum: 50-60%
├─ Reason: Progressive processing
└─ Speed: Off-peak timing available

General Recommendation:

Start with: 75%
├─ Reason: Good balance
├─ Safe: Maintains decent capacity
├─ Fast: Still reasonable speed
├─ Default: Most use this
└─ Adjust: Based on your needs

Then adjust:
├─ Too slow? Lower to 70% or 65%
├─ Capacity concerns? Raise to 80% or 85%
├─ Business hours? Go to 85-90%
├─ Off-peak? Can do 50-60%
└─ Document: What you chose and why
```

---

## Part 3: Warm-up Time Configuration

### Preparing New Instances for Service

**Understanding Instance Warm-up Periods**

```
Warm-up Time Explained:

What It Is:

Definition:
├─ Wait period: After new instance launches
├─ Purpose: Allow instance to stabilize
├─ Application startup: Initialization time
├─ Load balancer registration: Health check waiting
└─ Before: Considering instance "in service"

Why It's Needed:

Instance Startup Timeline:

t=0s: Launch command issued
├─ State: Pending
└─ Status: Requesting hardware

t=5-30s: Instance boots
├─ Process: OS kernel loading
├─ State: Running
└─ Status: Instance available

t=30-60s: System startup
├─ Process: Services starting
├─ User data: Beginning execution
└─ Status: Initializing

t=60-180s: Application startup
├─ Process: App loading, config reading
├─ Database: Connections established
├─ Cache: Warm-up data
└─ Status: Application initializing

t=180-300s: Application ready
├─ Process: All systems ready
├─ Connections: Active and working
├─ Performance: Normal/full capacity
└─ Status: Ready to handle traffic

Problems Without Warm-up:

If Marked "Ready" Too Early:

Scenario:
├─ t=90s: Instance appears ready
├─ Action: Immediately send traffic
├─ Problem: App still initializing
└─ Result: Errors and slow responses

Traffic During Boot:

Request arrives:
├─ At: t=100s
├─ Find: Database connections not ready
├─ Get: Connection timeout
├─ Return: 500 Error
└─ User: Sees failure

Cache Not Warmed:

Query arrives:
├─ Expected: 10ms (from cache)
├─ Reality: Cache empty, hit database
├─ Time: 200ms (database query)
├─ Result: User sees latency spike
└─ Experience: Slow application

With Proper Warm-up:

Scenario:
├─ t=90s: Instance running, not ready
├─ t=100s: Send health check, fails (no response yet)
├─ t=200s: Send health check, succeeds (now ready)
├─ t=201s: Instance marked "in service"
├─ t=201s: Now receive production traffic
└─ Result: Only fully-ready instances get traffic

Calculating Your Warm-up Time:

What to Measure:

Start: Instance launch begins
├─ Record: Time t=0
└─ Method: CloudWatch timestamp

Measure: Time until ready
├─ Method 1: SSH to instance, verify app running
├─ Method 2: Health check endpoint responds
├─ Method 3: Metrics normal (CPU low, app operational)
└─ Record: Seconds until fully ready

Example Application: Web Server with caching

Startup Process:

t=0s: Launch issued
├─ State: Pending
└─ Action: None

t=15s: OS boots
├─ State: Running
├─ SSH: Can connect
└─ Action: User data script starts

t=20s: User data runs commands
├─ Action 1: Update packages (yum update)
├─ Action 2: Install web server (yum install httpd)
├─ Duration: 30-60 seconds
└─ Current: t=20-70s

t=70s: Packages installed, web server starts
├─ Action: systemctl start httpd
├─ State: Listening on port 80
├─ Response: Can connect but serves placeholder
└─ Health check: May pass at connection level

t=120s: Application starts running
├─ Action: Application code executing
├─ Process: Loading database connection
├─ Process: Reading cache (Redis)
├─ Process: Initializing in-memory structures
└─ Status: Mostly ready

t=180s: All initialization complete
├─ State: Fully ready
├─ Health check: All endpoints working
├─ Performance: Normal latency
├─ Application: Serving requests properly
└─ Conclusion: Ready for production traffic

Measurement Result:
├─ Launch to ready: 180 seconds (3 minutes)
├─ Recommended warm-up: 180 seconds
└─ Safe warm-up: 200-240 seconds (add buffer)

Common Warm-up Times:

Simple Application:

Examples:
├─ Static content server: 30-60 seconds
├─ Simple health check: 10-20 seconds
└─ Just web server: 45-90 seconds

Typical Use:
├─ Warm-up time: 60 seconds
└─ Conservative: 90 seconds

Moderate Complexity:

Examples:
├─ Web app + database: 2-3 minutes
├─ API server + cache: 1.5-2 minutes
└─ Content service + loading: 2-3 minutes

Typical Use:
├─ Warm-up time: 120 seconds
└─ Conservative: 180 seconds

Complex Application:

Examples:
├─ Full data load: 5+ minutes
├─ Machine learning model: 2-5 minutes
├─ Large cache population: 3-5 minutes
└─ Database replication: 5-10 minutes

Typical Use:
├─ Warm-up time: 300 seconds (5 minutes)
└─ Conservative: 300-600 seconds

Configuring Warm-up During Refresh:

AWS Console Steps:

Step 1: Navigate to Instance Refresh Setup

During Policy Creation:
└─ Look for: Warm-up time field

Or During Refresh Start:
└─ Option available: When starting refresh

Step 2: Set Warm-up Value

Field Name: "Warm-up period" or "Warm-up time"
├─ Input: Numeric value
├─ Unit: Seconds
├─ Example: 120 (for 2 minutes)
├─ Min: 0 seconds (not recommended!)
└─ Max: 3600 seconds (1 hour, rare)

Step 3: Apply and Start

Action: Start/Create refresh
├─ Effect: Warm-up applied to all batches
└─ Duration: Each batch waits this long

Real-World Scenarios:

Scenario 1: Simple Web App (1-2 minutes startup)

Application:
├─ Type: Nginx web server
├─ Stack: Static content only
├─ Dependencies: Minimal (few packages)
└─ Startup: ~60 seconds

Warm-up Configuration:
├─ Measurement: 60 seconds to full capacity
├─ Safety buffer: Add 30-50%
├─ Set: 90-100 seconds
└─ Conservative: 120 seconds (2 minutes)

Reasoning:
├─ Simple: Less can go wrong
├─ Fast boot: Minimal packages
├─ Buffer: OS load variations
└─ Safe: Better 20s extra than 20s short

Scenario 2: Web API with Cache (2-3 minutes)

Application:
├─ Type: Node.js or Python API
├─ Stack: App + Redis cache + database
├─ Dependencies: Database connections, cache warming
└─ Startup: ~180 seconds

Warm-up Configuration:
├─ Measurement: 180 seconds for full startup
├─ Add safety: 20 seconds
├─ Set: 200 seconds
└─ Conservative: 240 seconds (4 minutes)

Reasoning:
├─ Complex: Multiple dependencies
├─ Startup: Varies by load
├─ Buffer: Network variations, DB delays
└─ Safe: Take longer, ensure ready

Scenario 3: Machine Learning Service (5+ minutes)

Application:
├─ Type: ML model serving
├─ Stack: Model loading, GPU initialization
├─ Dependencies: Large files, GPU setup
└─ Startup: 300-600 seconds

Warm-up Configuration:
├─ Measurement: 300-600 seconds
├─ Add safety: Not much needed (already long)
├─ Set: 600 seconds
└─ Conservative: 600 seconds (10 minutes)

Reasoning:
├─ Heavy: Long initialization
├─ GPU: Warm-up itself time-consuming
├─ Buffer: Already substantial warm-up
└─ Safe: ML models need full load

Impact on Refresh Duration:

With Different Warm-up Times:

Example: 10 instances, 75% min healthy, 3 instances per batch

Scenario 1: 60-second warm-up

Per batch:
├─ Terminate: 2-3 instances (30s)
├─ Launch: 2-3 new instances (30-90s to running)
├─ Warm-up: 60 seconds
├─ Build: Subtotal ~150-180 seconds
└─ Per batch: ~2.5-3 minutes

For 4 batches:
├─ Duration: 4 × 3 min = 12 minutes
└─ Total: ~12-15 minutes

Scenario 2: 180-second warm-up

Per batch:
├─ Terminate: 2-3 instances (30s)
├─ Launch: 2-3 new instances (30-90s to running)
├─ Warm-up: 180 seconds
├─ Build: Subtotal ~240-300 seconds
└─ Per batch: ~4-5 minutes

For 4 batches:
├─ Duration: 4 × 5 min = 20 minutes
└─ Total: ~20-25 minutes

Scenario 3: 300-second warm-up

Per batch:
├─ Terminate: 2-3 instances (30s)
├─ Launch: 2-3 new instances (30-90s to running)
├─ Warm-up: 300 seconds
├─ Build: Subtotal ~360-420 seconds
└─ Per batch: ~6-7 minutes

For 4 batches:
├─ Duration: 4 × 7 min = 28 minutes
└─ Total: ~28-35 minutes

Key Observation:
├─ Warm-up dominated: Most of refresh time
├─ Doubling warm-up: Roughly doubles duration
└─ Trade-off: Better safe than sorry

Best Practice:

1. Measure Real Startup:
   ├─ Launch instance manually
   ├─ SSH and check when ready
   ├─ OR: Monitor health check transitions
   ├─ OR: Monitor metrics (CPU, network settle to normal)
   └─ Record: The time

2. Add Safety Buffer:
   ├─ Small apps (1 min): Add 30-50% → 90-120s
   ├─ Medium apps (3 min): Add 20-30% → 180-240s
   ├─ Large apps (5+ min): Add 10-20% → 350-600s
   └─ Reason: Variations in load, network, etc.

3. Test in Staging:
   ├─ Run actual refresh with chosen warm-up
   ├─ Monitor: All instances become healthy
   ├─ Check: No errors during batch startup
   ├─ Verify: Performance normal after refresh
   └─ Confirm: Warm-up sufficient

4. Document and Iterate:
   ├─ Record: Warm-up time used
   ├─ Note: Why you chose this value
   ├─ Track: Any issues (too short, too long)
   ├─ Adjust: If needed for next refresh
   └─ Share: With team for consistency

Common Mistakes:

Mistake 1: Too Short Warm-up

Problem:
├─ Instance marked ready: Before app initialized
├─ Traffic sent early: App crashes or errors
├─ User experience: Failures and slow responses
└─ Result: Degraded service during refresh

Solution:
├─ Always measure actual startup time
├─ Add 50% buffer
└─ Better: Slow refresh than errors

Mistake 2: No Warm-up (Zero)

Problem:
├─ Assumes: Instance immediately ready
├─ Reality: Takes time to initialize
├─ Effect: Many failed requests
└─ Result: Outage perception during refresh

Solution:
├─ Always: Set real warm-up time
├─ Never: Leave as 0 (unless stateless + fast)
└─ Typical: 60-300 seconds minimum

Mistake 3: Too Long Warm-up

Problem:
├─ Refresh: Takes hours
├─ Duration: Unacceptable length
├─ Impact: Maintenance window too long
└─ Operational: Refreshes rarely attempted

Solution:
├─ Measure actual time
├─ Don't over-buffer
├─ Typical max: 5-10 minutes
└─ If longer: Consider blue-green instead

Mistake 4: Not Accounting for Variability

Problem:
├─ App normally: 90 seconds to ready
├─ Sometimes: Takes 120+ seconds (slow cloud)
├─ Set warm-up: 90 seconds
├─ Result: Occasionally instances marked ready too early
└─ Impact: Intermittent failures

Solution:
├─ Measure: Multiple times under load
├─ Account for: P95 or P99 startup time
├─ Set: Conservative to typical +50%
└─ Better: 120s (safe) than 90s (risky)
```

---

## Part 4: Instance Refresh Hands-On

### Practical Implementation and Monitoring

**Step-by-Step Demonstration**

```
Starting Instance Refresh:

Prerequisites:

Before Starting Refresh:

1. Current ASG exists:
   ├─ Running instances: Minimum 2-3
   ├─ Launch template: V1 (original)
   ├─ Status: All healthy
   └─ Load balancer: Connected (if applicable)

2. New launch template ready:
   ├─ Template: Created and configured
   ├─ Version: V2 or higher
   ├─ Changes: AMI updated, user data changed, or config altered
   ├─ Tested: In dev/staging (recommended)
   └─ Ready: To replace current template

3. ASG updated to new template:
   ├─ *Important*: ASG must reference new template
   ├─ Edit: ASG properties
   ├─ Launch Template: Select new V2
   ├─ Version: Specify V2 (or $Latest)
   └─ Save: ASG now points to V2

   Note: Creating refresh doesn't auto-update template
         You must manually point ASG to V2 first!

4. Plan your refresh:
   ├─ Warm-up time: Determined (from prior section)
   ├─ Min healthy: Chosen (from prior section)
   ├─ Downtime window: If needed, scheduled
   ├─ Monitoring: Prepared for observation
   └─ Rollback plan: If issues occur

AWS Console Steps:

Step 1: Navigate to Auto Scaling Groups

Service Navigation:
├─ Service: EC2 → Auto Scaling Groups
├─ Find: Your ASG from list
├─ Click: ASG name to open details
└─ Page: ASG details page opens

Step 2: Verify Current Template

Check Current State:

In "Details" tab:
├─ Field: "Launch template"
├─ Value: Should show V2 (new template)
├─ If V1: Edit ASG, update to V2 (critical!)
├─ Verify: Yes, we want V2 for refresh
└─ Status: Template ready for refresh

Check Instance State:

In "Instance management" tab:
├─ List: Current instances
├─ Status: All should be healthy
├─ Version: All still using V1 (before refresh)
├─ Count: At least 2-3 for meaningful test
└─ Ready: For refresh to update them

Step 3: Access Instance Refresh

Tab: "Instance management" or direct from ASG page
├─ Look for: "Instance refresh" section/button
├─ Or: Under "Actions" dropdown
├─ Option: "Start instance refresh"
└─ Click: Opens refresh configuration dialog

Step 4: Configure Instance Refresh

Refresh Configuration Dialog:

Minimum Healthy Percentage:
├─ Field: "Minimum healthy percentage"
├─ Example: 75
├─ Meaning: Keep 75% of instances running
├─ Effect: Can terminate 25% at once
└─ Set: Based on your chosen value

Warm-up Time:
├─ Field: "Warm-up period (seconds)"
├─ Example: 120
├─ Unit: Seconds
├─ Meaning: Wait 120 seconds per instance
└─ Set: Based on your measurement

Optional Settings:

Strategy (newer AWS versions):
├─ Option 1: Rolling - default
├─ Option 2: Chill (wait longer between batches)
└─ Typical: Rolling is fine

Preferences:
├─ Option: Scale new instances in on start refresh
├─ Meaning: Increase desired capacity if needed to maintain healthy %
├─ Typical: Leave default
└─ Advanced: Usually doesn't matter

Instance Warmup Override:
├─ Option: Override with specific value
├─ Default: Use ASG global warm-up if set
├─ When: Only if different for refresh
└─ Typical: Leave blank (use default)

Step 5: Review and Start

Review Before Starting:

Verify:
├─ Minimum healthy %: Correct value
├─ Warm-up time: Matches your measurement
├─ ASG name: Correct ASG selected
├─ Instances: Will be updated appropriately
└─ Understand: What will happen during refresh

Start:

Button: "Start instance refresh"
├─ Click: Initiates the refresh
├─ Confirmation: Dialog may show summary
├─ Status: Changes to "In Progress"
└─ Action: Refresh begins immediately

Monitoring Instance Refresh:

Activity History:

Access:

In ASG details:
├─ Tab: "Activity"
└─ View: "Activity history"

Real-time Observation:

What You'll See:

Refresh starts:
├─ Time: t=0:00
├─ Event: "Instance Refresh Started"
├─ Status: "In Progress"
├─ Instances: Being managed by refresh process
└─ Action: Monitoring begins

First batch termination:
├─ Time: t=0:30-1:00
├─ Event: "Launching 2 instances" (or however many)
├─ Status: "Successful"
├─ Reason: "Instance Refresh"
├─ Count: Desired capacity might have decreased
└─ Observation: Old instances gone

New instances boot:
├─ Time: t=1:00-3:00
├─ Event: "Initializing new instances"
├─ Progress: Can be seen in Instance management tab
├─ Status: "Pending" → "Running" → "InService"
└─ Details: New template being used

Warm-up period:
├─ Time: t=3:00-5:00 (if 120s warm-up)
├─ Event: Waiting for health check pass
├─ Status: Instance "initializing"
├─ Health: EC2 + ALB checks in progress
└─ Waiting: Until warm-up expires or health check passes

Next batch:
├─ Time: t=5:00+
├─ Event: Repeat pattern with next old instances
├─ Status: Refresh rolling forward
└─ Progress: Visible in activity history

Completion:
├─ Time: Varies with settings (15-120+ minutes)
├─ Event: "Instance Refresh Completed Successfully"
├─ Status: "Successful"
├─ Verification: All instances running new template
└─ Result: Refresh done

Instance Management Tab:

Observation Points:

Instances List:

View: "Instance management" tab
├─ Before: All instances from old launch template
├─ During: Mix of old and new instances
├─ After: All from new launch template

Example During Refresh:

Instance-1 (new, healthy)
Instance-2 (new, initializing - in warmup)
Instance-3 (old, in service)
Instance-4 (old, in service)
Instance-5 (terminating)
Instance-6 (pending - old)

Refresh ID:

AWS assigns refresh ID:
├─ Format: Something like "i-refresh-xxx"
├─ Purpose: Track this specific refresh
├─ Visible: In refresh details
├─ Use: Reference if checking logs
└─ Duration: Can query refresh status via CLI

CloudWatch Metrics:

Monitoring Refresh Progress:

Available Metrics:

ASG Metrics:
├─ GroupDesiredCapacity: Stays same (usually)
├─ GroupInServiceInstances: Varies as old replaced
├─ GroupInServiceCapacity: Shows instance count
├─ GroupTerminatingInstances: Count terminating
└─ GroupPendingInstances: Count starting

CPU Utilization:

Chart: CPU by instance
├─ Old instances: Normal load
├─ New instances: Increasing as warm-up ends
├─ Overall: May be lower during hot refresh
└─ Pattern: Distributed across all

Instance Launch Tracking:

In CloudWatch:

Logs availability:
├─ ASG logs: Activity auto-logged
├─ EC2 logs: Can query user data execution
├─ Could: View individual instance system logs
└─ Useful: Troubleshooting if issues

Handling Issues During Refresh:

Common Scenarios:

Scenario 1: Refresh Too Slow

Problem:
├─ Observation: Refresh taking longer than expected
├─ Cause: Warm-up time is very long
├─ Duration: Hours (if high min healthy %)
└─ Acceptable: Check against plan

Solution:
├─ If planned: Continue monitoring (normal)
├─ If unexpected: Check warm-up time used
├─ Confidence: Usually not a problem
└─ Action: Let it run (will eventually complete)

Scenario 2: Instance Health Checks Failing

Problem:
├─ Observation: New instances not becoming "InService"
├─ Symptom: Red health status in instance list
├─ Cause: Application not starting correctly
├─ Effect: Refresh hangs (can't declare ready)

Solution:

Investigate:

Step 1: SSH to a failing new instance
├─ Command: ssh -i key.pem ec2-user@IP
└─ Connection: Should work (instance running)

Step 2: Check application status
├─ Command: systemctl status myapp
├─ Or: ps aux | grep myapp
├─ Check: Is it actually running?
└─ Expected: Application process visible

Step 3: Check logs
├─ Location: /var/log/myapp/
├─ Or: systemctl logs myapp
├─ Look for: Errors, initialization issues
└─ Find: Why app not starting

Step 4: Check user data
├─ View: EC2 console → Instance → Details → User data
├─ Or: On instance: cat /var/lib/cloud/instance/user-data
├─ Compare: Old vs new template user data
└─ Spot: Any obvious issues?

Common issues:
├─ Packages: Not installed (yum failed)
├─ Permissions: File ownership issues
├─ Syntax: Errors in user data script
├─ Network: Cannot reach database/external services
└─ Disk: Insufficient space for application

Solution:

Fix issue:
├─ Find: Root cause from logs
├─ Create: New AMI with fix applied
├─ Create: New launch template V3
├─ Stop: Current refresh (cancel button)
└─ Update: ASG to V3, start new refresh

Or:

Rollback:
├─ Stop: Current refresh (cancel button)
├─ Point: ASG back to old template V1
├─ Update: Instances back to old template
└─ Duration: Required for fix development

Scenario 3: Refresh Failures / Errors

Problem:
├─ Observation: Refresh marked "Failed"
├─ Status: "Failed" in instance refresh section
├─ Cause: Various (permissions, template issue, etc.)
└─ Effect: Some instances updated, some old

Solution:

Check Status:

Tab: "Instance management" → Check refresh details
├─ Status: Shows "Failed"
├─ Error message: Usually provided
├─ Affected: Shows which instances failed
└─ Insight: Message describes issue

Common Errors:

1. "Launch Template not found"
   ├─ Cause: Template deleted or inaccessible
   ├─ Fix: Verify template exists, try again
   └─ Prevention: Don't delete template during refresh

2. "Permission denied"
   ├─ Cause: IAM role lacks permissions
   ├─ Fix: Check IAM attached to ASG
   └─ Prevention: Verify IAM before refresh

3. "Maximum instance limit reached"
   ├─ Cause: Account instance limit (e.g., 50 instances)
   ├─ Fix: Request quota increase or reduce capacity
   └─ Prevention: Check limits before refresh

Recovery Options:

Option 1: Retry Refresh
├─ Action: Fix issue, start new refresh
├─ Old state: Some instances still at old template
├─ New refresh: Will update remaining
└─ Result: Eventually all updated

Option 2: Rollback
├─ Point ASG: Back to old template V1
├─ New refresh: Update all back to V1
├─ Then: Fix issue and re-do
└─ Result: Consistent state restored

Option 3: Manual Termination
├─ Terminate: Any remaining old instances
├─ Manual: If refresh stuck and needs clearing
└─ Warning: Use only if refresh truly stuck

Successful Completion:

Verification:

Check Instance Template:

In Instance Management:
├─ Refresh: Shows status "Successful" or "Completed"
├─ All instances: Should show new template launch template
├─ Click: Instance → Details → Launch template
├─ Verify: All show V2 or new template
└─ Confirmation: All instances updated

Check Application Status:

All Instances Healthy:
├─ Status: All showing "InService"
├─ Health Checks: All passing
├─ Color: Green (healthy)
└─ Ready: All serving traffic

Application Functionality:

Test basic operation:
├─ Browser: Hit application URL
├─ Performance: Should be normal
├─ Errors: Should be none
├─ Latency: Should match pre-refresh
└─ Confidence: Application working

Activity History:

Complete Refresh Record:

View: Activity → Activity history
├─ Find: All refresh events
├─ Start event: "Instance Refresh started"
├─ Batches: Multiple "Launching X instances" and "Terminating Y instances"
├─ Duration: From start to end timestamp
├─ End event: "Instance Refresh completed successfully"
└─ Record: You can see complete history

Refresh details visible:
├─ Number of batches: How many rounds
├─ Total instances updated: Count
├─ Duration: Minutes/hours taken
└─ Status: Success

Timeline Example (Total Refresh):

Assuming: 8 instances, 75% min healthy, 120s warm-up

10:00:00 - Start
├─ Event: "Instance Refresh started"
├─ Configuration: "Minimum healthy: 75%, Warm-up: 120s"
└─ Status: "In Progress"

10:05:00 - Batch 1 termination
├─ Event: "Terminating 2 instances"
├─ Instances: Old-Instance-1, Old-Instance-2
├─ New desired: 6 (removing 2 from 8)
└─ Living: 6 instances still running

10:10:00 - Batch 1 launch
├─ Event: "Launching 2 instances"
├─ Desired: 8 (bringing back to original)
├─ New: New-Instance-1, New-Instance-2 (starting)
└─ Total: 6 old + 2 new launching

10:12:00 - Batch 1 launch complete
├─ Event: "2 new instances running"
├─ New-Instance-1: Running, initializing
├─ New-Instance-2: Running, initializing
└─ Warmup: Waiting 120 seconds

11:14:00 - Batch 1 warmup done
├─ Event: "Instance warm-up complete"
├─ New-Instance-1: Now InService
├─ New-Instance-2: Now InService
└─ Ready: Next batch can begin

10:14:00 - Batch 2 termination
├─ Event: "Terminating 2 instances"
├─ Old-Instance-3, Old-Instance-4: Gone
├─ Total: 4 old + 2 new running
└─ Living: 6/8 (1 extra cycle for example)

10:19:00 - Batch 2 launch
├─ Event: "Launching 2 instances"
├─ New-Instance-3, New-Instance-4: Launching
└─ Warmup: Waiting 120 seconds

10:21:00 - Batch 2 warmup done
├─ Event: "Ready for next batch"
└─ Status: Ready (4 old + 4 new healthy)

10:21:00 - Batch 3 termination
├─ Event: "Terminating 2 instances"
├─ Old-Instance-5, Old-Instance-6: Gone
└─ Living: 4 old + 4 new

10:26:00 - Batch 3 launch
├─ Event: "Launching 2 instances"
├─ New-Instance-5, New-Instance-6: Launching
└─ Warmup: Waiting 120 seconds

10:28:00 - Batch 3 warmup done
├─ Event: "Ready for next batch"
└─ Status: Ready (2 old + 6 new healthy)

10:28:00 - Batch 4 termination
├─ Event: "Terminating 2 instances"
├─ Old-Instance-7, Old-Instance-8: Gone (last old!)
└─ Living: 6 new instances

10:33:00 - Batch 4 launch
├─ Event: "Launching 2 instances"
├─ New-Instance-7, New-Instance-8: Launching
└─ Warmup: Waiting 120 seconds

10:35:00 - Batch 4 warmup done
├─ Event: "All instances new template"
└─ Status: 8/8 new instances healthy

10:35:00 - Complete
├─ Event: "Instance refresh completed successfully"
├─ Duration: 35 minutes total
├─ Batches: 4 (terminated 8 old, launched 8 new)
├─ Status: Success
└─ Result: All running V2 template

Post-Refresh Validation:

After Successful Refresh:

Verify Template:
├─ Check: All instances use new template
├─ Command: AWS CLI: aws autoscaling describe-auto-scaling-instances
├─ Or: Console: Instance management tab
└─ Confirm: "LaunchTemplate" shows V2

Verify Application:
├─ Functionality: Test main features
├─ Performance: Metrics show expected levels
├─ Errors: Logs show no errors
└─ Users: Report normal experience

Monitor for Side Effects:
├─ Duration: 30 minutes after refresh
├─ Metrics: CPU, memory, network normal
├─ Health: All instances reporting healthy
├─ Errors: Check application logs
└─ Success: No issues after refresh

Document:
├─ Record: Refresh date and time
├─ Note: Template updated from V1 to V2
├─ What: Changed (AMI, config, etc.)
├─ Status: Successful
└─ Knowledge: For team reference

Cleanup if Needed:

Old Templates:
├─ Keep or delete: Old V1 template
├─ Safe: Keep for rollback (in case)
├─ Storage: No cost (just metadata)
└─ Recommendation: Keep for week, then delete

Old AMI:
├─ Safe: Can delete if not used elsewhere
├─ Verify: No other templates reference it
├─ Storage: Costs for snapshots
└─ Recommendation: Delete after confirming refresh success
```

---

## Part 5: Instance Refresh Strategies and Best Practices

### Production Deployment Patterns

**Real-World Implementation Approaches**

```
Rolling Refresh vs Other Strategies:

When to Use Instance Refresh:

Best For:

Applications:
├─ Stateless: No session affinity
├─ Manageable startup: Under 5 minutes
├─ High availability: Already redundant (min 2+)
└─ Tolerates brief load increase: Per instance

Updates:
├─ Routine: Security patches, OS updates
├─ Minor versions: Bug fixes, small features
├─ Configuration: User data, environment vars
├─ AMI: Base image refresh
└─ Frequency: Regular (for maintenance)

NOT Best For:

Applications:
├─ Stateful: Sessions tied to instance
├─ Very large startup: > 10 minutes
├─ Single instance: No redundancy
└─ Slow to stabilize: Needs long warm-up

Updates:
├─ Major versions: Could have compatibility
├─ Database migrations: Schema changes required
├─ Risk: High compatibility risk
└─ Testing: Insufficient pre-test

Scenario 1: Development Environment

Use Instance Refresh:
├─ Frequency: Multiple times per day
├─ Updates: Developers pushing new versions
├─ Capacity: Can take longer (not prod)
├─ Risk: Low (dev-only impact)
└─ Benefit: Automated, consistent

Configuration:
├─ Min healthy: 50% (fast)
├─ Warm-up: 30-60 seconds (short)
├─ Time: Off-peak doesn't matter
└─ Scale: 2-3 instances typical

Result:
├─ Duration: 5-10 minutes
├─ Effort: Click button
└─ Status: Always consistent

Scenario 2: Production Environment

Use Instance Refresh:
├─ Frequency: Once per week or monthly
├─ Updates: Security patches, stable releases
├─ Capacity: Must maintain > 80% always
├─ Risk: Medium (production, but test-validated)
└─ Benefit: Automated, trackable, safe

Configuration:
├─ Min healthy: 85-90% (very safe)
├─ Warm-up: 120-180 seconds (measured)
├─ Time: Maintenance window (planned)
├─ Scale: 20+ instances typical

Result:
├─ Duration: 30-90 minutes
├─ Monitoring: Continuous observation
└─ Status: Validated throughout

Scenario 3: Compliance/Regulatory

Use Instance Refresh:
├─ Requirement: Document all changes
├─ Tracking: Audit trail required
├─ Updates: Patching on schedule
├─ Frequency: Monthly or quarterly
└─ Process: Formal change management

Configuration:
├─ Min healthy: 95-100% (maximum safety)
├─ Warm-up: 300 seconds (conservative)
├─ Documentation: Every update tracked
├─ Approval: Change board required
└─ Testing: Pre-deployment validation

Result:
├─ Duration: 1-2 hours
├─ Auditability: Full history
└─ Status: Compliant and documented

Combining with Other Deployment Methods:

Situation 1: Using with Scheduled Scaling

Pattern:
├─ Scheduled Scale-out: 7 AM → 20 instances
├─ During day: Handle peak traffic with 20
├─ Instance Refresh: 2 PM (mid-day, mid-scale)
├─ Update: While having 20 instances (parallel batches)
├─ Scheduled Scale-in: 6 PM → 5 instances
└─ Result: All instances have new template

Example Timeline:

7:00 AM: Scheduled scale to 20 instances (all V1)
2:00 PM: Start instance refresh to V2
  - Min healthy: 70% (14 instances)
  - Batch size: 6 instances at once
  - Batches: ~3-4 needed
  - Duration: 30-45 minutes with batches
2:45 PM: Refresh complete, all 20 on V2
6:00 PM: Scheduled scale to 5 instances (all on V2)

Benefit:
├─ Timing: High capacity speeds refresh (parallel batches)
├─ Update: During day, not off-peak
├─ Efficiency: Combined two operations
└─ Result: Scaled fleet always current

Situation 2: Using with Blue-Green for High Risk

Pattern:
├─ New ASG created: "Green" (V2 template)
├─ Parallel: Green ASG exists with Green instances
├─ Test: Thoroughly validate Green
├─ If ok: Switch DNS/LB to Green
├─ Then: Terminate Old(Blue) ASG
└─ Alternative: Run Blue + Green - use instance refresh on both

Why Blue-Green instead sometimes:
├─ Risk: Very high compatibility concern
├─ Rollback: Need instant switch back
├─ Testing: Need full soak test before cutover
└─ Large changes: Major version upgrade

Timeline:

Day 1: Pre-staging
├─ Create: New Blue ASG (current V1)
└─ Prepare: Green ASG template (new V2)

Day 2: Quick switch
├─ Launch: Green ASG (V2)
├─ Validate: Green instances healthy
├─ Switch: DNS/LB to Green
├─ Verify: Users on Green, working
├─ Delete: Blue ASG (old)

Alternative (Instance Refresh on both):
├─ Refresh Blue: Update current to V1.5 (intermediate)
├─ Test: Monitor performance
├─ Refresh Blue again: Update to V2 (final)
├─ Result: Phased approach if less risky

Situation 3: Canary with Instance Refresh

Pattern:
├─ Partial Refresh: Update 1 instance manually (canary)
├─ Monitor: Watch metrics (errors, latency, CPU)
├─ Duration: Soak test 30 minutes+
├─ If good: Full instance refresh on remaining
├─ If bad: Terminate canary, troubleshoot
└─ Retry: After fixing

Steps:

Step 1: Manual canary
├─ New template: V2 created
├─ Edit instance: Terminate 1 old, let ASG launch new
├─ Or: Manually terminate one instance
├─ Result: 1 new, rest old
├─ Watch: Metrics for 30 minutes

Step 2: Validate
├─ Errors: Any 5xx errors? ✓ No errors
├─ Latency: P95 latency normal? ✓ 45ms normal
├─ CPU: CPU levels expected? ✓ 50% normal
├─ Logs: Any warnings? ✓ No warnings
└─ Decision: Proceed with full refresh

Step 3: Full refresh
├─ Start: Instance refresh on entire ASG
├─ Config: Min healthy 75%, warm-up 120s
├─ Wait: Complete
└─ Result: All updated, canary validated approach

Post-Refresh Monitoring:

Immediate (First 5 minutes):

Metrics to watch:
├─ Error rate: Should be 0%
├─ Latency: Should be normal (no spike)
├─ CPU: Should stabilize
├─ Memory: Should stabilize
└─ Availability: Should be 100%

If problems:
├─ Errors: Check application logs
├─ Latency: Check if CPU-bound or I/O
├─ Instability: Check if warm-up too short
└─ Action: Investigate immediately

Short-term (5 minutes to 1 hour):

Observation:
├─ Stability: Metrics settling to normal
├─ Cache warming: If relevant, performance settling
├─ Connection pool: Database connections stable
├─ User impact: Complaints should stop
└─ Confidence: Growing

If still issues:
├─ Continue monitoring
├─ Check: Is new version working?
├─ Investigate: Any unexpected behavior?
└─ Action: Review logs for clues

Medium-term (1-24 hours):

Validation:
├─ Everything: Operating normally
├─ Performance: Matching baseline
├─ Errors: Zero or expected background
├─ Capacity: Stable
└─ Conclusion: Refresh successful

If problems appeared:
├─ Root cause: Investigation ongoing
├─ Rollback?: Decide based on severity
└─ Fix: Prepare for next refresh attempt

Long-term (1+ days):

Documentation:
├─ Record: Date, time, duration
├─ Changes: What was updated
├─ Result: Success, any issues
├─ Learning: For next refresh
└─ Archive: For compliance/audit

Troubleshooting Common Issues:

Problem: Refresh Very Slow

Diagnosis:

Check:
├─ Duration: How long so far?
├─ Min healthy: What % set?
├─ Warm-up: How many seconds?
└─ Expected: Calculate theoretical time

Calculation:
├─ Per batch: Launch (1-3 min) + Warm-up + Termination
├─ Example: 3 min launch + 2 min warm-up + 1 min = 6 min per batch
├─ Batches: 10 instances with 75% = 2-3 batches
├─ Total: 12-18 minutes expected
└─ If > 2 hours: Something off

Solution:

If slow but expected:
├─ Continue: Let it run
├─ Monitor: Watch for errors
└─ Action: None needed (normal)

If slower than expected:
├─ Check: Instances stuck initializing?
├─ Look: Application logs (if SSH accessible)
├─ Verify: Warm-up not too long
└─ Option: Can cancel and retry with lower %

Problem: Mix of Old/New Instances (Refresh Failed)

Diagnosis:

Situation:
├─ Some instances: V2 template (new)
├─ Some instances: V1 template (old)
├─ Refresh: Shows "Failed"
└─ Cause: Error during refresh

Finding the Issue:

Activity history:
├─ Look for: Last successful action
├─ After that: Error actions
├─ Message: Should describe issue
└─ Example: "Failed: Permission denied" or "Launch Template not found"

Solution:

Depending on error:
├─ Permission: Check IAM role
├─ Template: Verify template exists
├─ Other: Read full error message
└─ Fix: Address root cause

Recovery:

Option A: Fix and retry refresh
├─ Step 1: Fix issue (add IAM, recreate template, etc.)
├─ Step 2: Start new instance refresh
├─ Step 3: AWS updates remaining old instances
└─ Result: Eventually all on V2

Option B: Rollback and retry
├─ Step 1: Point ASG back to V1
├─ Step 2: Terminate all V2 instances
├─ Manual: Easier via ASG min/max adjustment
├─ Step 3: Start fresh refresh after fix
└─ Result: Consistent state before retry

Problem: Instances Not Becoming Healthy

Diagnosis:

Symptom:
├─ New instances: Started but not "InService"
├─ Status: Stuck in "Initializing"
├─ Warm-up: Taking very long
├─ Refresh: Seems stalled
└─ Problem: Instances won't declare "healthy"

Finding the Issue:

SSH to Instance:
├─ Command: ssh -i key.pem ec2-user@IP
├─ Connection: Should work
└─ Next: Check what's wrong

Check Application:
```bash
# Is app running?
ps aux | grep myapp
systemctl status myapp

# Any errors?
systemctl logs myapp -n 50

# Can connect to dependencies?
telnet database.example.com 5432
nc -zv cache.example.com 6379
```

Check User Data Success:
```bash
cat /var/log/cloud-init-output.log
# Look for: Success or errors
```

Common Causes:

1. Application crashed:
   ├─ Symptom: Process not found
   ├─ Cause: App init failed
   ├─ Fix: Check logs, resolve issue
   └─ Example: Missing dependency, permission denied

2. Database unreachable:
   ├─ Symptom: Connect timeout
   ├─ Cause: Security group, network
   ├─ Fix: Check security groups, network ACLs
   └─ Example: Database port blocked

3. Out of disk space:
   ├─ Symptom: Cannot write files
   ├─ Cause: Volume full
   ├─ Fix: Increase volume size in template
   └─ Example: Large application or logs

4. User data syntax error:
   ├─ Symptom: Script stops partway
   ├─ Cause: Bash error
   ├─ Fix: Check script output, fix syntax
   └─ Example: Typo, missing quotes

5. Warm-up too short:
   ├─ Symptom: Health check fails because not ready
   ├─ Cause: Insufficient time to initialize
   ├─ Fix: Increase warm-up
   └─ Example: Set 60s, actually need 180s

Solution:

Fix issue:
├─ Identify: Root cause from logs
├─ Create: New AMI with fix applied
├─ New template: V3 with corrected AMI
├─ Stop: Current refresh (cancel button)
├─ Update ASG: Point to V3
├─ Retry: Start new refresh
└─ Monitor: Check instances become healthy

Best Practices:

1. Always Test First:
   ├─ Dev/Staging: Run refresh first
   ├─ Observe: Behavior on non-prod
   ├─ Validate: New template works
   ├─ Duration: Measure actual timing
   └─ Before: Production attempt

2. Choose Right Min Healthy:
   ├─ Conservative: Start high (85-90%)
   ├─ Monitor: Observe duration
   ├─ Adjust: Lower if too slow
   ├─ Safe: Never go below 60%
   └─ Document: Your choice and why

3. Set Realistic Warm-up:
   ├─ Measure: Count actual startup time
   ├─ Add buffer: 50% more
   ├─ Test: Verify in staging
   ├─ Not too long: Still reasonable (<10 min)
   └─ Document: Your measurement

4. Schedule Appropriately:
   ├─ Off-peak: If high min healthy
   ├─ Business hours: If low min healthy (50%)
   ├─ Maintenance window: Plan if multiple hours
   ├─ Communication: Notify team
   └─ Monitoring: Available to watch

5. Monitor Throughout:
   ├─ Active: Watch activity history
   ├─ Metrics: CPU, latency, errors normal?
   ├─ Alert: Should issues arise
   ├─ Duration: How long actually taking?
   └─ Ready: To intervene if needed

6. Document Everything:
   ├─ Before: What updating and why
   ├─ During: Duration and observations
   ├─ After: Any issues, lessons learned
   ├─ Team: Share knowledge
   └─ Playbook: Build runbook for next time

7. Have a Rollback Plan:
   ├─ If issues: What will you do?
   ├─ Rollback: Back to old template
   ├─ Duration: How long to rollback?
   ├─ Comms: Who to notify?
   └─ Testing: Before re-attempting fix

Maintenance Windows:

Planning Your Refresh:

Identifying Good Times:

Low traffic periods:
├─ Time: Identify when load minimal
├─ Duration: How long does it stay low?
├─ Frequency: Same time daily? (peak hours pattern)
└─ Example: 2-4 AM for many apps

Off-peak days:
├─ Weekends: Lower traffic typical
├─ Holidays: Check calendar
├─ Seasonal: Plan around busy periods
└─ Example: Update Sunday 2 AM

Team availability:
├─ Monitoring: Who available?
├─ Leadership: VP/team lead present?
├─ Support: Can team respond if issues?
└─ Schedule: Coordinate with team

Communicating the Refresh:

Before:
├─ Notification: Email/Slack to team
├─ Details: When, what, expected duration
├─ Impact: Who affected? (none if load balanced)
├─ Approval: Get sign-off if required
└─ Escalation: Who to contact if issues

During:
├─ Updates: Live progress (every 15 min)
├─ Status: "50% complete", etc.
├─ Issues: Immediately notify if arise
└─ ETA: When expected to finish

After:
├─ Completion: Announce done
├─ Success: All instances updated
├─ Validation: Tested and working
├─ Documentation: Record for review
└─ Debrief: Team lesson learned meeting if issues
```

---

## Part 6: Exam Focus and Key Takeaways

### Critical Concepts and Real-World Scenarios

**Interview and AWS Certification Preparation**

```
Key Concepts to Remember:

Instance Refresh Overview:

Definition:
├─ Feature: AWS ASG native rolling update
├─ Purpose: Update all instances with new launch template
├─ Mechanism: Gradual replacement (rolling)
├─ Downtime: Zero (continuous replacement)
└─ Trigger: API call or AWS console

Why Use:
├─ Automation: No manual instance termination
├─ Safety: Controlled capacity maintenance
├─ Speed: Parallelized for many instances
├─ Tracking: Full activity history
└─ Consistency: All instances guaranteed updated

When to Use:
├─ AMI updates: New OS/patches
├─ App updates: New version deployed
├─ Config changes: User data, environment vars
├─ Regular cadence: Weekly security patches
└─ Automated deployments: CI/CD integration

Minimum Healthy Percentage:

Definition:
├─ Calculation: % of instances that must running
├─ Impact: How many can terminate at once
├─ Speed: Lower % = faster refresh
└─ Safety: Higher % = more protection

Common Values:
├─ 50%: Very fast (half at once) - off-peak only
├─ 75%: Balanced (default recommendation)
├─ 90%: Slow but very safe
├─ 100%: Sequential, maximum safety
└─ Choose: Based on your capacity tolerance

Effect on Duration:
├─ 50%: ~20 min (10 instances)
├─ 75%: ~45 min (10 instances)
├─ 90%: ~90 min (10 instances)
└─ Rule: Higher % = longer duration

Warm-up Time:

Definition:
├─ Wait period: After instance launches
├─ Purpose: Allow app initialization
├─ Duration: Application-specific
└─ Effect: Can take more time than startup

Purpose:
├─ Boot OS: 30-60 seconds
├─ User data: 30-180 seconds
├─ App startup: 30-300 seconds
├─ Stabilize: 30-120 seconds
└─ Total: 2-10 minutes typical

Must Measure:
├─ Launch manually: Record startup time
├─ Add buffer: 50% safety margin
├─ Test in staging: Verify before prod
└─ Adjust: If experiences vary

Impact on Duration:
├─ 60s: Shorter total duration
├─ 180s: Moderate total duration
├─ 300s: Longer total duration
└─ Per batch: Warm-up is largest component

Pre-Refresh Requirements:

Critical: Before Starting

ASG must reference new launch template:
├─ Edit ASG: Update template
├─ Template version: V2 or new version
├─ Verify: ASG shows new template
└─ Common mistake: Forget this step!

New launch template must be ready:
├─ Created: Template exists
├─ Configured: All settings correct
├─ Tested: In dev/staging recommended
├─ AMI/user data: Updated appropriately
└─ Permissions: If using new IAM role

Current instances must be healthy:
├─ Status: All InService
├─ Health checks: Passing
├─ Not already: Terminating
└─ Baseline: All ready to update

Backup/Rollback plan in place:
├─ Know: Old template V1 still exists
├─ Can: Point ASG back if needed
├─ Understand: How long rollback takes
└─ Ready: Minutes to rollback if issues

Refresh Status States:

Pending: Initializing
├─ Meaning: About to start
├─ Duration: Seconds
├─ Action: None (starting)

Active/In Progress: Running
├─ Meaning: Refresh happening
├─ Duration: Minutes to hours
├─ Observation: Activity history shows updates

Successful: Complete
├─ Meaning: All instances updated
├─ Status: All new template
├─ Verification: Done

Failed: Stopped with errors
├─ Meaning: Error occurred
├─ Impact: Mixed old/new instances
├─ Action: Fix and retry

Cancelled: Manually stopped
├─ Meaning: Admin stopped it
├─ Impact: Mixed old/new
├─ Action: Decide: retry or rollback

Typical Interview Questions:

Question 1: "What's Instance Refresh?"

Good answer:
├─ Definition: Native ASG feature for rolling updates
├─ Purpose: Update all instances with new launch template
├─ Benefit: Zero downtime, automated, safe
├─ Process: Gradually terminates old, launches new
└─ Use: Deploying patches, new AMIs, config changes

Question 2: "How does minimum healthy percentage work?"

Good answer:
├─ Definition: Minimum % of instances that must running
├─ Impact: Controls how many can terminate concurrently
├─ Example: 75% means can terminate 25% at once
├─ Trade-off: Lower % faster but less safe
├─ Speed: 50% much faster than 90%
└─ Selection: Based on workload capacity tolerance

Question 3: "Why need a warm-up time?"

Good answer:
├─ Reason: Instances take time to initialize
├─ Process: OS boot + user data + app startup
├─ Without: Instances marked ready too early
├─ Result: Errors when traffic sent to unprepared instance
├─ Solution: Specify wait time (warm-up)
├─ Measurement: Should measure actual startup time
└─ Typical: 60-300 seconds depending on app

Question 4: "How do you choose min healthy percentage?"

Good answer:
├─ Method: Depends on workload
├─ High availability workload: 80-90%
├─ Must handle reduced capacity: 75%
├─ Can scale small temporarily: 60-70%
├─ Test in staging: Observe actual duration
├─ Adjust: Based on tolerance
└─ Recommendation: Start 75%, adjust from there

Question 5: "What if refresh fails halfway?"

Good answer:
├─ State: ASG has mix of old and new instances
├─ Options: (a) Fix and retry, (b) Rollback
├─ Retry: After fixing issue (permissions, template, etc.)
├─ Rollback: Point ASG back to old template, refresh to V1
├─ Outcome: Cannot leave mixed forever (bad)
├─ Timeline: Recovery takes 15-60 minutes typically
└─ Prevention: Test thoroughly before production

Question 6: "When NOT to use Instance Refresh?"

Good answer:
├─ Bad for: Major version upgrades (compatibility risk)
├─ Bad for: Database migrations (schema changes)
├─ Bad for: Stateful applications (session loss)
├─ Bad for: Very long startup (> 10 minutes)
├─ Better alternatives: Blue-green, canary, manual
└─ Typical use: Routine patches, minor versions

Question 7: "How does it differ from terminating manually?"

Good answer:
├─ Manual: Click terminate, one at a time
├─ Manual issues: Hours of work, error-prone
├─ Refresh benefits: Fully automated
├─ Refresh benefits: Maintains minimum capacity
├─ Refresh benefits: Tracks progress in activity history
├─ Refresh benefits: Parallelized (faster)
└─ Reality: Instance Refresh is almost always better

Real-World Scenarios:

Scenario: Weekly Security Patches

Situation:
├─ Frequency: Every Tuesday 2 AM
├─ Target: 10 web servers
├─ Change: New AMI with OS patches
├─ Risk: Low (patches tested)

Setup:
├─ New template: V2 with patched AMI
├─ Min healthy: 75%
├─ Warm-up: 120 seconds
├─ Timing: Tuesday 2 AM

Execution:
├─ 2:00 AM: Refresh starts
├─ 2:25 AM: Refresh complete (~25 min)
├─ 2:30 AM: Validation all instances patched
└─ Result: All servers updated, zero downtime

Scenario: Application Deployment

Situation:
├─ Release: New app version (v2.1)
├─ Build: Docker image created, pushed to ECR
├─ Deployment: Replace all instances
├─ Target: 4 instances

Setup:
├─ New AMI: Packer build (Docker image v2.1)
├─ New template: V3 referencing new AMI
├─ Min healthy: 75%
├─ Warm-up: 180 seconds (app takes time to init)

Execution:
├─ 10:00 AM: Refresh starts
├─ 10:02 AM: Terminate 1 instance
├─ 10:07 AM: New instance booting
├─ 10:10 AM: New instance running, warming up
├─ 10:13 AM: New instance ready
├─ 10:13 AM: Next batch (repeat)
├─ 10:26 AM: Refresh complete (~25 min)

Staging first:
├─ Staging ASG: Also refresh from V2 to V3
├─ Observation: "Takes 25 min for 4 instances"
├─ Validation: All instances healthy, app working
├─ Then: Production refresh with confidence

Scenario: Planned Infrastructure Change

Situation:
├─ Change: Update instance type (t3.micro → t3.small)
├─ Reason: App needs more resources
├─ Scale: 20 instances
├─ Plan: Compliance, formal change control

Setup:
├─ New template: V4 with t3.small type
├─ Min healthy: 90% (very conservative, compliance)
├─ Warm-up: 120 seconds
├─ Time: Scheduled maintenance window

Execution Process:
├─ Week before: Change request filed
├─ Day before: Approval received
├─ Testing: Dev/Staging environment tested
├─ Day of: Maintenance window scheduled (no user traffic)
├─ Window time: 2 hours allocated

Actual execution:
├─ 12:00 AM: Refresh starts (20 instances, 90% min, 3-4 per batch)
├─ 12:00-12:30: Batch 1 (2 instances)
├─ 12:30-1:00: Batch 2 (2 instances)
├─ 1:00-1:30: Batch 3 (2 instances)
├─ 1:30-2:00: Batch 4 (2 instances)
├─ ... continues for 10 batches total
├─ Estimated: 1.5 hours
└─ Within: 2-hour window

Monitoring:
├─ Continuous: Watch activity, metrics
├─ 30 min: 30% complete
├─ 1 hour: 60% complete
├─ 1:30 AM: Complete
├─ Validation: All 20 instances on t3.small
├─ Performance: Metrics verify fine
└─ Documentation: All recorded in audit trail

Common Exam Questions:

Question 1 (Scenario):
"You have 100 web servers running. You need to update the AMI. What's best approach?"

Answer options:
(a) Manually terminate each server, wait for new one → WRONG (tedious, error-prone)
(b) Use Instance Refresh with 75% min healthy → CORRECT (automated, safe)
(c) Use blue-green deployment → Wrong for this (overkill, cost)
(d) Update instances one per day manually → Wrong (very slow)

Questions and Answers:

Q: Minimum healthy percentage 70% with 10 instances. How many can terminate simultaneously?
A: 3 instances (10 × 0.30 = 3). Calculation: Total - (Total × min healthy %).

Q: Warm-up time is set to 5 minutes. What does this mean?
A: Wait 5 minutes after instance launches before marking it "InService" and ready for traffic.

Q: Instance Refresh for major application version upgrade—good idea?
A: No. Better: Blue-green or canary deployment. Refresh good for patches, not major version changes.

Q: What happens if needs to roll back during refresh?
A: Stop/cancel refresh, point ASG back to old template, refresh again to revert instances.

Q: Refresh takes longer than expected. What's likely cause?
A: (a) Very high min healthy % (lots of batches), (b) Long warm-up time, or (c) Slow instance startup.

Q: Can update ASG launch template AFTER starting refresh?
A: No. Template must be ready and ASG configured before starting refresh.

Q: Two templates (V1, V2) but ASG still using V1. Can start refresh to V2?
A: Only if ASG updated to V2 first (critical step). Refresh uses template ASG points to.

Key Reminders:

DO:
├─ ✓ Test new template in staging first
├─ ✓ Update ASG template reference before refresh
├─ ✓ Measure actual app startup time
├─ ✓ Add safety buffer to warm-up time (50%)
├─ ✓ Choose min healthy based on workload
├─ ✓ Monitor activity history and metrics
├─ ✓ Document refresh and results
└─ ✓ Have rollback plan ready

DON'T:
├─ ✗ Forget to update ASG template (common mistake!)
├─ ✗ Set warm-up too short (causes errors)
├─ ✗ Use for major version upgrades (risk)
├─ ✗ Ignore monitoring during refresh
├─ ✗ Leave ASG with mixed old/new versions
├─ ✗ Refresh during peak hours (unless low %)
│   └─ Exception: If 50% min healthy chosen off-peak
├─ ✗ Assume it's always better than manual
│   └─ Exception: For routine patches it usually is
└─ ✗ Skip testing in non-prod first

Production Checklist:

Before Instance Refresh:

Pre-Refresh:
├─ [ ] New launch template created (V2, V3, etc.) 
├─ [ ] Template contains all required changes (AMI, user data, type, etc.)
├─ [ ] Template tested in staging environment
├─ [ ] ASG has been updated to reference new template
├─ [ ] Verify ASG currently points to new template (double-check!)
├─ [ ] New template works properly (health checks pass)
├─ [ ] Warm-up time determined by measurement (not guessed)
├─ [ ] Min healthy percentage chosen based on workload
├─ [ ] Refresh duration estimated (tells team how long)
├─ [ ] Monitoring prepared (dashboard open, alerts ready)
├─ [ ] Team notified (timing, expected duration, what to watch)
├─ [ ] Rollback plan documented (how to go back if needed)

During Refresh:
├─ [ ] Monitoring: Watch activity history regularly  
├─ [ ] Metrics: CPU, latency, error rates normal?
├─ [ ] Duration: Is it proceeding on expected timeline?
├─ [ ] Health checks: Are new instances passing?
├─ [ ] Capacity: Is load distributed correctly?
├─ [ ] Issues: Any appearing? (note them)
└─ [ ] Communication: Update team every 15+ minutes

Post-Refresh:
├─ [ ] Refresh status: Marked "Successful"?
├─ [ ] All instances: Using new template confirmed
├─ [ ] Functionality: Application working correctly?  
├─ [ ] Performance: Metrics back to normal?
├─ [ ] Issues: Any lingering problems?
├─ [ ] Validation: Full testing of key features
├─ [ ] Documentation: Record date, duration, changes, result
├─ [ ] Team: Communicate completion
└─ [ ] Archive: Save refresh history for compliance
```

---

## Conclusion

Instance Refresh is AWS's native solution for keeping your Auto Scaling Groups current with minimal operational overhead. By understanding minimum healthy percentage, warm-up time configuration, and following production best practices, you can automate infrastructure updates with confidence.

Key success factors:
- Always update ASG template reference before refresh
- Measure application startup time for accurate warm-up
- Choose min healthy percentage based on workload tolerance
- Test thoroughly in staging before production
- Monitor continuously during refresh
- Have rollback procedures ready

Instance Refresh transforms infrastructure maintenance from a tedious, error-prone manual process into a reliable, automated, fully-tracked operation.
