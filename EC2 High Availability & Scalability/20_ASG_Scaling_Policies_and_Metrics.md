# ASG Scaling Policies and Metrics

Comprehensive guide to Auto Scaling Group scaling policies, including dynamic scaling (target tracking and step scaling), scheduled scaling, predictive scaling, metric selection, cooldown periods, and optimization strategies for responsive infrastructure.

---

## Part 1: Scaling Policies Overview

### Types of Scaling Policies and Strategic Selection

**Understanding Dynamic, Scheduled, and Predictive Scaling**

```
Scaling Policy Landscape:

Overview of Four Policy Types:

1. Dynamic Scaling (Reactive)
   ├─ Trigger: Real-time metrics
   ├─ Response: Up/down based on current demand
   ├─ Types: Target Tracking, Simple/Step Scaling
   ├─ Timing: Immediate (after cooldown)
   └─ Use: Variable workloads, spike response

2. Scheduled Scaling (Predictive)
   ├─ Trigger: Time-based schedule
   ├─ Response: At specific times
   ├─ Planning: You know when traffic increases
   ├─ Timing: Scheduled in advance
   └─ Use: Predictable patterns (business hours)

3. Predictive Scaling (AI-based)
   ├─ Trigger: ML forecast
   ├─ Response: Ahead of demand
   ├─ Learning: Analyzes historical patterns
   ├─ Timing: Proactive before demand appears
   └─ Use: Cyclical workloads, repeating patterns

4. Other (Manual, Webhook)
   ├─ Manual: Human-triggered
   ├─ Webhook: External system triggers
   └─ Use: Special cases

Architecture Comparison:

┌─────────────────────────────────────────────┐
│       Scaling Policy Decision Matrix         │
└──────────────────┬──────────────────────────┘

Your Workload Pattern:

Unpredictable/Variable Load:
├─ Policy: Target Tracking Scaling (Dynamic)
├─ Metric: CPU or RequestCount
├─ Response: Automatic based on metrics
├─ Benefit: Simple, effective, self-adjusting
└─ Example: E-commerce with spiky traffic

Spike Handling Required:
├─ Policy: Step Scaling (Dynamic)
├─ Alarms: Multiple thresholds
├─ Response: Proportional to spike severity
├─ Benefit: Fine-grained control
└─ Example: Video processing on uploads

Predictable Pattern (Business Hours):
├─ Policy: Scheduled Scaling (Known schedule)
├─ Trigger: Time of day, day of week
├─ Response: At specific time
├─ Benefit: Pre-positioned capacity
└─ Example: 9 AM scale-up, 6 PM scale-down

Repeating Cyclical Pattern:
├─ Policy: Predictive Scaling (ML-based)
├─ Learning: Historical analysis
├─ Response: Before demand spike
├─ Benefit: Proactive, learning-based
└─ Example: Weekly pattern repeating

Combination Approach (Recommended):

Most Production:
├─ Scheduled: Handle known patterns
├─ Dynamic: Handle unexpected spikes
├─ Together: Best coverage
└─ Result: Baseline + responsive

Example Workflow:

9 AM Workday Starts:
├─ Scheduled: Scale to 5 (prepare for peak)
├─ Dynamic: If CPU > 70%, scale more
└─ Result: Ready for expected + responsive to actual

3 PM Usual Quiet:
├─ Scheduled: No action
├─ Dynamic: If CPU < 30%, scale down
└─ Result: Cost optimization, responsive

5 PM Spike:
├─ Scheduled: No action
├─ Dynamic: CPU jumps to 85%, scale up
└─ Result: Respond to unexpected surge

Which Policy to Choose:

Decision Tree:

Question 1: Traffic Predictable?

YES → Question 2
    │
    ├─ Known Time Pattern?
    │ │
    │ ├─ YES → Scheduled Scaling
    │ │       (Use calendar-based)
    │ │
    │ └─ NO → Ask: Cyclical Pattern?
    │         │
    │         ├─ YES → Predictive Scaling
    │         │       (Use historical analysis)
    │         │
    │         └─ NO → Scheduled + Dynamic
    │                 (Combination)

NO → Question 2
    │
    └─ Essential Simplicity?
      │
      ├─ YES → Target Tracking
      │       (Keep metric near target)
      │
      └─ NO → Step Scaling
              (Fine-grained control)

Comparison Matrix:

┌─────────────┬──────────────┬───────────┬──────────────┬─────────────┐
│ Policy      │ Setup        │ Simplicity│ Control      │ Responsiveness
├─────────────┼──────────────┼───────────┼──────────────┼─────────────┤
│ Target      │ Easy         │ Very High │ Low          │ Good
│ Tracking    │ (1 value)    │           │ (automatic)  │ (metric-based)
├─────────────┼──────────────┼───────────┼──────────────┼─────────────┤
│ Step        │ Medium       │ Medium    │ High         │ Very Good
│ Scaling     │ (many rules) │           │ (granular)   │ (proportional)
├─────────────┼──────────────┼───────────┼──────────────┼─────────────┤
│ Scheduled   │ Easy         │ High      │ Medium       │ Fair
│ Scaling     │ (times)      │           │ (set times)  │ (time-based)
├─────────────┼──────────────┼───────────┼──────────────┼─────────────┤
│ Predictive  │ Hard         │ Low       │ Low          │ Excellent
│ Scaling     │ (ML setup)   │           │ (AWS manages)│ (proactive)
└─────────────┴──────────────┴───────────┴──────────────┴─────────────┘

Real-World Usage Statistics:

Enterprise Applications:

Target Tracking:
├─ Adoption: ~60% of companies
├─ Reason: Simplicity, effectiveness
└─ Use: Primary policy type

Step Scaling:
├─ Adoption: ~30%
├─ Reason: Fine control needs
└─ Use: Complex workloads

Scheduled:
├─ Adoption: ~50% (also with dynamic)
├─ Reason: Predictable patterns
└─ Use: Combined with others

Predictive:
├─ Adoption: ~10%
├─ Reason: New, requires setup
└─ Use: Advanced, specific cases

Combining Policies (Best Practice):

Recommended Combination:

Use Both:
├─ Scheduled: For known patterns
├─ Dynamic: For surprises
└─ Together: Baseline + responsive

Example Implementation:

Daily Schedule:
├─ 7:00 AM: Scheduled scale to 3
├─ 9:00 AM: Scheduled scale to 8
├─ 5:00 PM: Scheduled scale to 4
├─ 10:00 PM: Scheduled scale to 2

Throughout Day:
├─ Policy: Target tracking CPU 60%
├─ Effect: Can scale up if CPU high
├─ Benefit: Responds to spikes
└─ Result: Baseline + responsive

Result:

Cost Efficiency:
├─ Off-hours: Minimal cost
├─ Business hours: Prepared
└─ Spikes: Handled dynamically

Performance:
├─ Expected: Good capacity
├─ Unexpected: Responsive
└─ Overall: Optimized

Policy Application Order:

All Policies Active:
├─ Scheduled: Executes first (if scheduled time)
├─ Then: Update desired capacity
├─ Then: Dynamic policy evaluates
├─ Result: Works harmoniously

Example Scenario:

9:00 AM Monday:
├─ Scheduled: Scales to 8 (business start)
├─ Dynamic: Still evaluating metrics
├─ Effect: Desired = 8 (from schedule)

9:30 AM (spike):
├─ Scheduled: Still 8 (time not changed)
├─ Dynamic: CPU 85% > target 60%
├─ Action: Scale up to 10
└─ Effect: Desired = 10 (from dynamic policy)

Next Interval (stable):
├─ CPU: Now 65% (distributed)
├─ Dynamic: CPU > 60%, maintain
├─ Effect: Stays at 10

1:00 PM (quiet):
├─ Scheduled: Maybe scale to 6 (lunch)
├─ Dynamic: CPU 35% < 60%
├─ Conflict: Scheduled wins time-based
├─ Effect: Desired = 6

Key Insight:

Policies Don't Conflict:
├─ Work together: Not against
├─ Complement: Each covers different needs
├─ Best: Use multiple policies
└─ Result: Robust, optimized scaling
```

---

## Part 2: Target Tracking Scaling

### Simplest and Most Popular Policy

**Maintaining Metric at Target Value**

```
Target Tracking Scaling Overview:

What It Is:

Definition:
├─ Simple: Single metric, single target value
├─ Automatic: AWS manages all calculations
├─ Adjustment: Scales to maintain target
├─ Example: Keep average CPU at 50%
└─ Result: Effortless metric-based scaling

How It Works:

Basic Principle:

Current Metric Value: 65% CPU
Target Value: 50% CPU
Gap: 65% - 50% = 15% over target
Action: Scale out (add instances)
Result: CPU drops toward 50%

Continuous Process:
├─ Measure: Current metric value
├─ Compare: Against target
├─ Decide: Scale up, down, or hold
├─ Adjust: Desired capacity accordingly
└─ Repeat: Every few minutes

AWS Calculations (Handled Automatically):

What AWS Does:
├─ Monitors: Metric in real-time
├─ Calculates: Current vs target ratio
├─ Determines: Instances needed
├─ Scales: Up or down proportionally
├─ Manages: Gracefully (respects cooldown)
└─ Benefit: You don't do math

You Provide:

Only Two Things:
├─ Metric: Which CloudWatch metric to track
├─ Target: Desired value for that metric
└─ Rest: AWS manages automatically

Example Setup:

Configuration:
├─ Metric: Average CPU Utilization
├─ Target: 50%
└─ ASG: Maintains ~50% CPU

Result:

If Current CPU < 50%:
├─ Action: Scale in (remove instances)
├─ Effect: Less capacity, CPU rises

If Current CPU > 50%:
├─ Action: Scale out (add instances)
├─ Effect: More capacity, CPU falls

Always Tries: To keep metric near target value

Creating Target Tracking Policy:

AWS Console Steps:

Step 1: Navigate to ASG

ASG List:
├─ Select: Your ASG
├─ Tab: "Automatic scaling"
└─ Action: "Create/Update scaling policy"

Step 2: Choose Policy Type

Policy Options:
├─ Select: "Target tracking scaling policy"
├─ Not: Simple or step scaling
└─ Next: Configure policy

Step 3: Select Metric

Built-in Metrics:

Popular Options:
├─ ASGAverageCPUUtilization: Average CPU across all instances
├─ ASGAverageNetworkIn: Average inbound network
├─ ASGAverageNetworkOut: Average outbound network
├─ ALBRequestCountPerTarget: Requests per target (if using ALB)
└─ More: Custom metrics available

Example: ASGAverageCPUUtilization
├─ Meaning: Average CPU across ASG
├─ Range: 0-100%
├─ Good for: CPU-bound applications
└─ Common: Most used metric

Custom Metrics:

If Need Specific:
├─ Option: Create custom metric
├─ Source: Application pushes to CloudWatch
├─ Example: "ActiveConnections" or "QueueDepth"
└─ Advanced: For special needs

Step 4: Set Target Value

Target Value Field:
├─ Example: 50 (for 50% CPU)
├─ Range: Depends on metric
├─ Typical CPU: 40-70%
└─ Choose: Based on your workload

Selecting Target:

Conservative (Higher):
├─ Value: 70-80%
├─ Benefit: Fewer scaling events
├─ Trade: Less spare capacity
└─ Use: Cost-optimized

Balanced (Middle):
├─ Value: 50-60%
├─ Benefit: Good balance
├─ Trade: Moderate scaling
└─ Use: Most common

Aggressive (Lower):
├─ Value: 30-40%
├─ Benefit: More spare capacity
├─ Trade: More instances running
└─ Use: Performance-critical

Example Targets:

Web Server (CPU):
├─ Target: 50-60%
├─ Reason: Balance cost/performance
└─ Typical: 50%

API Server (CPU):
├─ Target: 60-70%
├─ Reason: Respond to spikes quickly
└─ Typical: 60%

Database (CPU):
├─ Target: 70-80%
├─ Reason: Scaleless (hard to scale DB)
└─ Typical: 80%

Step 5: Scale-Out Behavior

Rapid Scale-Out:
├─ Instances added: If metric way over target
├─ Speed: Fast response to spikes
├─ Default: Usually aggressive
└─ Effect: Adds multiple instances quickly

Scale-In Behavior:

Conservative:
├─ Instances removed: If metric way under target
├─ Speed: Slow removal
├─ Default: AWS is conservative here
├─ Effect: Takes longer to scale down
└─ Reason: Avoid thrashing

Disable Scale-In (Option):
├─ Checkbox: "Disable scale-in"
├─ Effect: Only scales up, never down
├─ Use: Cost-neutral testing
└─ Reason: Prevent mistakes

Step 6: Review and Create

Summary:
├─ Policy name: Auto-generated or custom
├─ Metric: Your selected metric
├─ Target: Your target value
├─ Scale-out: How many instances
├─ Scale-in: Whether enabled
└─ Cooldown: Default (usually ok)

Create:
├─ Button: "Create"
└─ Status: Policy active

Target Tracking Examples:

Example 1: Web Application

Setup:
├─ Metric: Average CPU Utilization
├─ Target: 50%
├─ Min instances: 2
├─ Max instances: 10

Behavior:

Normal Traffic:
├─ CPU: ~45-55% (near target)
├─ Instances: 2-4 (balanced)
└─ Cost: Minimal

Evening Peak:
├─ Traffic: Doubles
├─ CPU: Rises to 75%
├─ Action: Scale out → 6 instances
├─ CPU: Drops to 50%
└─ Stable: Serves well

Late Night:
├─ Traffic: Drops 80%
├─ CPU: Falls to 20%
├─ Action: Scale in → 2 instances
├─ Cost: Reduced
└─ Still available: Always 2 instances

Example 2: API Server (RequestCount Metric)

Setup:
├─ Metric: RequestCountPerTarget
├─ Target: 1000 requests/target/minute
├─ Min: 2 instances
├─ Max: 20

Behavior:

Off-peak:
├─ Requests: 500 total
├─ Per target: 250 (low)
├─ Instances: Scale down to 1
└─ Cost: Minimal (but careful, only 1!)

Busy Hour:
├─ Requests: 15,000 total
├─ Per target: 1000 (at target!)
├─ Instances: 15 (15 × 1000 = 15,000)
└─ Perfect: Load distributed optimally

Example 3: Network-Intensive App

Setup:
├─ Metric: NetworkIn
├─ Target: 1 MB/sec/instance
├─ Min: 2 instances
├─ Max: 50

Behavior:

Large Upload:
├─ Network: 30 MB/sec total
├─ Per instance: 30 MB (way over target)
├─ Scale: To 30 instances (30 × 1 = 30)
├─ Per instance: 1 MB (at target)
└─ Complete: Upload faster

Problem with Target Tracking:

Limited Granularity:

Only One Target:
├─ Single metric value
├─ Single target value
├─ Not flexible: What if different needs at different levels?
└─ Solution: Use step scaling for complex needs

Scale-In Disabled:

Permanent Increase:
├─ If you disable scale-in
├─ Never scales down
├─ Cost: Continuous
└─ Not ideal: For cost optimization

Cool Downs:

Cannot Customize:
├─ AWS manages: Cooldown internally
├─ You cannot: Override cooldown
├─ Fixed: Reasonable defaults
└─ Usually fine: But not customizable

Best Practices:

1. Start Conservative:
   ├─ Higher target first (70%)
   ├─ Monitor: Actual behavior
   ├─ Lower: Once comfortable
   └─ Gradual: Fine-tune over days

2. Choose Right Metric:
   ├─ CPU: Most common, works well
   ├─ RequestCount: If using ALB
   ├─ Custom: Only if truly needed
   └─ Consider: What actually limits app

3. Set Realistic Target:
   ├─ Test: Before production
   ├─ Baseline: Understand app behavior
   ├─ Range: 50-70% for CPU typical
   └─ Document: Why this target

4. Monitor Behavior:
   ├─ Dashboard: Watch metric and instance count
   ├─ Alarms: Alert if scaling not working
   ├─ Logs: Review activities
   └─ Adjust: If needed

5. Combine with Scheduled:
   ├─ Scheduled: Known patterns
   ├─ Target tracking: Surprises
   ├─ Together: Best approach
   └─ Result: Robust scaling
```

---

## Part 3: Step Scaling and Simple Scaling

### Fine-Grained Control with CloudWatch Alarms

**Proportional Scaling Responses**

```
Step Scaling Overview:

What It Is:

Definition:
├─ Granular: Different actions at different metric levels
├─ Alarms: Multiple CloudWatch alarms trigger actions
├─ Proportional: Scale amount matches severity
├─ Complex: More configuration than target tracking
└─ Powerful: Ultimate control and flexibility

When to Use:

Best For:
├─ Complex: Workload needs nuanced scaling
├─ Application: Specific scaling behavior needed
├─ Spikes: Different response for spike severity
└─ Performance: Fine-tuned response

Not For:
├─ Simple: Single metric tracking
├─ Start: If new to scaling
└─ Maintenance: More overhead

How It Works:

Define Steps:

Step 1: CPU 60-70%
├─ Action: Add 1 instance
└─ Reason: Mild increase

Step 2: CPU 70-80%
├─ Action: Add 2 instances
└─ Reason: Moderate increase

Step 3: CPU 80-90%
├─ Action: Add 3 instances
└─ Reason: Significant increase

Step 4: CPU > 90%
├─ Action: Add 5 instances
└─ Reason: Critical demand

Proportional Response:

Small Problem:
├─ CPU: 65% (just over ideal)
├─ Action: Add 1 instance
└─ Effect: Incremental response

Bigger Problem:
├─ CPU: 75% (getting serious)
├─ Action: Add 2 instances
└─ Effect: More aggressive response

Crisis:
├─ CPU: 95% (severe)
├─ Action: Add 5 instances
└─ Effect: Maximum response

Creating Step Scaling Policy:

Step 1: Create Alarms

Two Alarms Needed:

Alarm 1: Scale-Out (CPU High)
├─ Metric: Average CPU Utilization
├─ Condition: CPU >= 60%
├─ Action: Trigger step scaling out
└─ Important: Create this FIRST

Alarm 2: Scale-In (CPU Low)
├─ Metric: Average CPU Utilization
├─ Condition: CPU <= 30%
├─ Action: Trigger step scaling in
└─ Important: Create AFTER scale-out

Why Alarm-Based:

Alarms Are:
├─ Decoupled: From ASG (reusable)
├─ Flexible: Can do other actions
├─ Standard: CloudWatch alarms
└─ Powerful: More than just scaling

Step 2: Create Scaling Policy

Navigate:
├─ ASG: Select your ASG
├─ Tab: "Automatic scaling"
├─ Action: "Create scaling policy"
└─ Type: "Step scaling"

Step 3: Define Scale-Out Steps

Scale-Out Behavior:

Trigger:
├─ Alarm: Select scale-out alarm (CPU >= 60%)
└─ Effect: Alarm firing triggers steps

Step Configuration:

Step 1:
├─ Metric: 60 <= CPU < 70%
├─ Action: Add 1 instance
├─ Calculation: Fixed (add 1)

Step 2:
├─ Metric: 70 <= CPU < 80%
├─ Action: Add 2 instances
├─ Calculation: Fixed (add 2)

Step 3:
├─ Metric: 80 <= CPU < 90%
├─ Action: Add 3 instances
├─ Calculation: Fixed (add 3)

Step 4:
├─ Metric: CPU >= 90%
├─ Action: Add 5 instances
├─ Calculation: Fixed (add 5)

Adding Steps:

AWS Console:
├─ Click: "Add step" button
├─ For each: Define metric range and action
├─ Save: All steps together
└─ Result: Policy with multiple steps

Step 4: Define Scale-In Steps

Scale-In Behavior:

Trigger:
├─ Alarm: Select scale-in alarm (CPU <= 30%)
└─ Effect: Alarm firing triggers steps

Step Configuration:

Step 1:
├─ Metric: 20 < CPU <= 30%
├─ Action: Remove 1 instance
├─ Calculation: Fixed (remove 1)

Step 2:
├─ Metric: 10 < CPU <= 20%
├─ Action: Remove 2 instances
├─ Calculation: Fixed (remove 2)

Step 3:
├─ Metric: CPU <= 10%
├─ Action: Remove 3 instances
├─ Calculation: Fixed (remove 3)

Conservative Approach:
├─ Remove: Fewer instances than added
├─ Reason: Don't overshoot downward
└─ Duration: Slower scale-in than scale-out

Step 5: Cooldown Configuration

Cooldown Period:

Default:
├─ Time: 300 seconds (5 minutes)
├─ Effect: No second action during cooldown
└─ Purpose: Let metrics stabilize

Customizing:
├─ Length: Change if needed
├─ Shorter: If app scales fast (can handle)
├─ Longer: If need stability
└─ Typical: Keep 300 seconds

Step 6: Review and Create

Summary Shows:
├─ Scale-out: Trigger and steps
├─ Scale-in: Trigger and steps
├─ Cooldown: Duration
└─ Ready: Create

Create:
├─ Confirm: Settings correct
└─ Status: Policy active

Step Scaling Examples:

Example 1: Web Server with Traffic Spike Levels

Setup:

Scale-Out Steps:
├─ 60-70% CPU: Add 1 instance
├─ 70-80% CPU: Add 2 instances
├─ 80-90% CPU: Add 3 instances
├─ >90% CPU: Add 5 instances

Scale-In Steps:
├─ 30-20% CPU: Remove 1 instance
├─ 20-10% CPU: Remove 2 instances
├─ <10% CPU: Remove 3 instances

Initial: 3 instances

Scenario 1: Mild Increase
├─ Traffic: Modest increase
├─ CPU: Rises to 65%
├─ Trigger: Scale-out alarm
├─ Step: 60-70% range → Add 1
├─ Result: 4 instances
├─ CPU: Stabilizes at 50%
└─ Status: Good response

Scenario 2: Significant Spike
├─ Traffic: Large spike
├─ CPU: Jumps to 82%
├─ Trigger: Scale-out alarm
├─ Step: 80-90% range → Add 3
├─ Result: 6 instances
├─ CPU: Drops to 50%
└─ Status: Good aggressive response

Scenario 3: Crisis (Black Friday)
├─ Traffic: Massive spike
├─ CPU: Shoots to 95%
├─ Trigger: Scale-out alarm
├─ Step: >90% range → Add 5
├─ Result: 8 instances
├─ CPU: Drops to 55%
└─ Status: Maximum response deployed

Example 2: Complex Multi-Step Network-Bound App

Scale-Out Steps (NetworkIn metric):
├─ 50-100 MB/s: Add 1 instance
├─ 100-200 MB/s: Add 3 instances
├─ 200-500 MB/s: Add 5 instances
├─ >500 MB/s: Add 10 instances

Scale-In Steps:
├─ 10-20 MB/s: Remove 1 instance
├─ 5-10 MB/s: Remove 2 instances
├─ <5 MB/s: Remove 3 instances

Usage: Video downloads, large uploads

Scenario:

Off-peak:
├─ Network: 3 MB/s
├─ Instances: 2 (minimum)
└─ Cost: Low

Normal:
├─ Network: 75 MB/s
├─ Instances: 4 (baseline + 2)
└─ Cost: Moderate

Heavy Download Time:
├─ Network: 150 MB/s (people downloading)
├─ Step: 100-200 → Add 3 instances
├─ Instances: 7 (4 + 3)
└─ Load: Distributed

Peak Data Time:
├─ Network: 450 MB/s (many downloads)
├─ Step: 200-500 → Add 5 instances
├─ Instances: 12 (7 + 5)
└─ Fully scaled: Peak capacity

Simple Scaling (Deprecated):

What It Is:

Definition:
├─ Basic: Single alarm → single action
├─ Example: CPU > 70% → Add 1 instance
├─ Limited: No graduation between severity
└─ Status: Deprecated (use step instead)

How Different from Step:

Simple:
├─ Alarm: One alarm
├─ Action: One fixed response
├─ Flexibility: No
└─ Example: Always add 1

Step:
├─ Alarm: One alarm
├─ Actions: Multiple graduated responses
├─ Flexibility: Yes, by metric level
└─ Example: Add 1, 2, 3, or 5 based on severity

Why Step is Better:

Responsiveness:
├─ Proportional: Response matches severity
├─ Effortless: AWS chooses best step
└─ Better: Better scaling behavior

Control:
├─ Fine-grained: Multiple options
├─ Precise: Handle edge cases
└─ Superior: More sophisticated

Step Scaling vs Target Tracking:

Complexity Comparison:

Target Tracking:
├─ Setup time: 5 minutes
├─ Sophistication: Simple
├─ AWS role: High (automatically manages)
├─ Customization: Low (one value)
└─ Use: Default recommendation

Step Scaling:
├─ Setup time: 30 minutes
├─ Sophistication: Complex
├─ AWS role: Lower (you define steps)
├─ Customization: High (many options)
└─ Use: When you need control

When to Choose:

Target Tracking:
✓ Unknown workload characteristics
✓ Starting out with scaling
✓ Simple metric tracking needed
✓ Quick implementation priority

Step Scaling:
✓ Know exact scaling behavior wanted
✓ Different response for different severity
✓ Fine control needed
✓ Time to implement available

Best Practices:

1. Start with Fewer Steps:
   ├─ Complexity: Don't overwhelm
   ├─ Start: 2-3 steps first
   ├─ Monitor: Behavior
   └─ Add: More steps if needed

2. Proportional Actions:
   ├─ Small problem: Small action
   ├─ Big problem: Big action
   ├─ Never: Add more than needed
   └─ Theory: Proportional response

3. Asymmetric Scale-In:
   ├─ Out: Fast (add more)
   ├─ In: Slow (remove fewer)
   ├─ Reason: Prevent thrashing
   └─ Pattern: Safer than symmetric

4. Test Before Production:
   ├─ Simulate: Load to trigger alarms
   ├─ Verify: Scaling behavior matches plan
   ├─ Adjust: Steps if needed
   └─ Confidence: Before prod use

5. Monitor Cooldown:
   ├─ Default: Usually sufficient
   ├─ If too short: Scaling thrashing
   ├─ If too long: Slow response
   └─ Typical: 300 seconds good
```

---

## Part 4: Scaling Metrics and Selection

### Choosing the Right Metric for Your Workload

**CPU, Network, Request Count, and Custom Metrics**

```
Scaling Metric Overview:

Types of Metrics:

Metric Categories:

1. Built-in Metrics (Automatic)
   ├─ CPU Utilization: Processor usage
   ├─ Network In/Out: Bandwidth
   └─ Source: EC2 automatically collects

2. Application Metrics (via ALB)
   ├─ Request Count Per Target: Requests/instance/minute
   ├─ Target Response Time: Latency
   └─ Source: Load balancer collects

3. Custom Metrics (Application)
   ├─ Queue Depth: Items waiting
   ├─ Active Connections: Concurrent users
   ├─ Database Connections: Pool usage
   └─ Source: Application pushes to CloudWatch

Choosing the Right Metric:

Decision Framework:

Question 1: What Limits Performance?

CPU-Limited:
├─ Symptoms: CPU high, memory low, network low
├─ Metric: CPU Utilization
├─ Target: 50-70%
└─ Reason: CPU is bottleneck

Network-Limited:
├─ Symptoms: Network saturated, CPU low
├─ Metric: NetworkIn or NetworkOut
├─ Target: 80% of max bandwidth
└─ Reason: Network is bottleneck

Request-Limited:
├─ Symptoms: Requests queue up
├─ Metric: RequestCountPerTarget (if ALB)
├─ Target: Max requests app handles
└─ Reason: Application request capacity

Connection-Limited:
├─ Symptoms: Can't accept new connections
├─ Metric: Custom metric (active connections)
├─ Target: 80% of max allowed
└─ Reason: Connection limit approaching

Question 2: What Causes Performance Degradation?

Compute Tasks:
├─ Example: Video encoding, data processing
├─ Metric: CPU Utilization
└─ Logic: More instances (more parallel)

I/O Intensive:
├─ Example: Database queries, file transfers
├─ Metric: Network or custom metric
└─ Logic: More instances (more bandwidth)

Request Handling:
├─ Example: Web requests, API calls
├─ Metric: RequestCountPerTarget
└─ Logic: More instances (more connections)

Throughput:
├─ Example: Messages/sec, transactions/sec
├─ Metric: Custom metric
└─ Logic: More instances (more workers)

Built-In Metrics:

Metric 1: CPU Utilization

What It Measures:
├─ Percentage: Average CPU usage
├─ Range: 0-100%
├─ Source: EC2 service (automatic)
└─ Collected: Every 5 minutes (basic) or 1 minute (detailed)

Typical Values:

By Workload:

Web Server:
├─ Idle: 5-10%
├─ Normal: 30-50%
├─ Peak: 60-80%
└─ Overloaded: >90%

API Server:
├─ Idle: 10-15%
├─ Normal: 40-60%
├─ Peak: 70-85%
└─ Overloaded: >95%

Database:
├─ Idle: 20-30%
├─ Normal: 50-70%
├─ Peak: 80-95%
└─ Overloaded: >99%

Batch Processing:
├─ Idle: 1-5%
├─ Processing: 90-99%
└─ Note: Bursty (long high periods)

Good For Scaling:

✓ CPU-bound workloads
✓ Compute-intensive tasks
✓ General purpose servers
✓ Web servers

Not Good For:

✗ I/O bound (network saturates before CPU)
✗ Connection limited (connections, not CPU)
✗ Memory constrained (not measured)

Target Values:

Conservative: 70-80%
├─ Benefit: More spare capacity
├─ Trade: Higher cost
└─ Use: Performance-critical

Balanced: 50-70%
├─ Benefit: Good balance
├─ Trade: Moderate cost
└─ Use: Most common

Aggressive: 40-50%
├─ Benefit: Lots of spare capacity
├─ Trade: Higher cost per request
└─ Use: Reserved for HA requirements

Metric 2: Network In/Out

What It Measures:

NetworkIn:
├─ Bytes: Received per minute
├─ Unit: Bytes/minute or MB/sec
├─ Source: EC2 network interface
└─ Use: Inbound traffic

NetworkOut:
├─ Bytes: Sent per minute
├─ Unit: Bytes/minute or MB/sec
├─ Source: EC2 network interface
└─ Use: Outbound traffic

When to Use:

Network-Bound Workloads:
├─ Large uploads: Video uploads, data imports
├─ Large downloads: Media delivery, file downloads
├─ Streaming: Video streaming, real-time data
├─ IoT: Sensor data aggregation
└─ Pipelines: Data processing pipelines

Example Thresholds:

t2.micro (max 1 Gbps):
├─ Scale trigger: 800 Mbps
├─ Target: ~800 Mbps per instance
└─ 10 Gbps needed: Scale to 13 instances

Application Load Balancer Metrics:

Metric: RequestCountPerTarget

What It Measures:

Definition:
├─ Requests: Per target per minute
├─ Per-target: Divided by instance count
├─ Example: 3000 req/min ÷ 3 instances = 1000 req/target
└─ Purpose: Know capacity per instance

How Calculated:

Total Requests: 3000/min
Instances Active: 3
Per-Target: 3000 ÷ 3 = 1000 req/target/min

If Scale to 5 Instances:
├─ Same traffic: 3000/min
├─ Per-target: 3000 ÷ 5 = 600 req/target/min
└─ Result: Much lower load per instance

Good For:

✓ Request-based scaling
✓ APIs and web services
✓ When using ALB/NLB
✓ User-facing applications

Determining Target Value:

Testing Required:

Step 1: Load Test
├─ Goal: Find max requests instance can handle
├─ Method: Gradual load increase
├─ Monitor: Check when issues start
└─ Result: Max safe capacity

Step 2: Calculate Safety Factor
├─ Max found: 2000 requests/target
├─ Safety: 80% of max
├─ Target: 1600 requests/target
└─ Headroom: 20% buffer

Example:

Measure:
├─ Single instance: Handles 1000 requests/min safely
├─ With buffer: 1000 × 0.8 = 800 requests/target

Setting:
├─ ASG size: 1 instance
├─ Expected: 800 requests/target
├─ Traffic: 800 requests/min served by 1 instance perfectly
├─ High traffic: 2000 requests/min
├─ Calculation: 2000 ÷ 800 = 2.5 → scale to 3 instances
└─ Result: 667 per instance (safe)

Target Response Time (Latency)

What It Measures:

Definition:
├─ Time: Seconds from LB to target
├─ Include: Application processing time
├─ Exclude: Client latency before LB
└─ Purpose: Application responsiveness

Typical Values:

Web Pages:
├─ Target: <0.5 seconds
├─ Meaning: Page generated in <500ms
└─ User: Gets fast response

APIs:
├─ Target: <0.1 seconds
├─ Meaning: Response in <100ms
└─ User: Snappy experience

Database Queries:
├─ Target: <1 second
├─ Meaning: Query completes in <1s
└─ Result: Database might be limiting

Batch Operations:
├─ Target: <10 seconds
├─ Meaning: Operation completes in <10s
└─ Typical: Long operations don't scale this

When to Scale Up by Latency:

If Latency Rising:
├─ Indicates: Overload (slow responses)
├─ Action: Scale out (more capacity)
├─ Effect: Requests handled faster
└─ Result: Latency back to target

Custom Metrics:

Creating Custom Metrics:

Application Level:

Your Code Creates:
├─ Example: "ActiveDatabaseConnections"
├─ Value: Currently active connections
├─ Frequency: Published every 60 seconds
└─ Format: CloudWatch PutMetricData API

Publishing to CloudWatch:

Code Example (Python):
```python
import boto3

cloudwatch = boto3.client('cloudwatch')

active_connections = 150  # Your app's value

cloudwatch.put_metric_data(
    Namespace='MyApp',
    MetricData=[
        {
            'MetricName': 'ActiveConnections',
            'Value': active_connections,
            'Unit': 'Count',
            'Timestamp': datetime.utcnow()
        }
    ]
)
```

Then in ASG:
├─ Create policy: Target tracking
├─ Metric: MyApp/ActiveConnections
├─ Target: 500 (max safe connections)
└─ Result: Scales based on connections

Common Custom Metrics:

1. Queue Depth
   ├─ What: Messages/tasks waiting
   ├─ Publish: By task worker
   ├─ Scale: When queue > X, add workers
   └─ Example: SQS queue worker scaling

2. Active Connections
   ├─ What: Simultaneous users connected
   ├─ Publish: By application
   ├─ Scale: When connections > max, add servers
   └─ Example: Real-time chat, gaming

3. Database Pool Usage
   ├─ What: Database connections in use
   ├─ Publish: By app
   ├─ Scale: When pool > 80%, add servers
   └─ Example: Connection pool exhaustion

4. Memory Usage
   ├─ What: RAM consumption
   ├─ Publish: By CloudWatch agent
   ├─ Scale: When memory > 80%, add servers
   └─ Example: In-memory caching

5. Disk Usage
   ├─ What: Storage consumption
   ├─ Publish: By CloudWatch agent
   ├─ Scale: When disk > 80%, add servers
   └─ Example: Temporary storage workloads

Metric Selection Decision Tree:

Your Workload Type:

Web Server (HTTP):
├─ Primary: CPU Utilization
├─ Secondary: RequestCountPerTarget
├─ Fallback: NetworkOut
└─ Target: CPU 50-60%

API Server (REST/GraphQL):
├─ Primary: RequestCountPerTarget
├─ Secondary: CPU Utilization
├─ Custom: Response latency
└─ Target: 1000 requests/target/min

Database Server:
├─ Primary: CPU Utilization
├─ Secondary: Custom (connection pool)
├─ Note: Scaling DBs difficult
└─ Strategy: Often manual or reserved

Message Queue Worker:
├─ Primary: Custom (queue depth)
├─ Secondary: CPU Utilization
├─ Example: Processes SQS messages
└─ Strategy: Scale based on queue

Real-Time Application:
├─ Primary: Custom (active connections)
├─ Secondary: Memory usage
├─ Example: WebSocket connections
└─ Strategy: Connection-aware scaling

Media Streaming:
├─ Primary: NetworkOut
├─ Secondary: RequestCountPerTarget
├─ Example: Video streaming
└─ Strategy: Bandwidth-aware

Batch Processing:
├─ Primary: CPU Utilization
├─ Secondary: Custom (job queue)
├─ Example: Video encoding
└─ Strategy: CPU-aware

Best Practices:

1. Choose Metric that Directly Limits Performance:
   ├─ Not: Arbitrary metrics
   ├─ Yes: Actual bottleneck
   └─ Test: Find real limiting factor

2. Test Before Production:
   ├─ Load test: Understand behavior
   ├─ Measure: Key metric under load
   ├─ Determine: Scaling point
   └─ Validate: Scaling works as expected

3. Use Right Target Value:
   ├─ Not: Too high (under-provisioned)
   ├─ Not: Too low (wasteful)
   ├─ Yes: Balanced (80% of max)
   └─ Example: CPU 50-70% usually good

4. Monitor Actual Performance:
   ├─ Track: Application metrics
   ├─ Compare: Scaling predictions vs reality
   ├─ Adjust: If different than expected
   └─ Refine: Over time

5. Combine Metrics (for complex apps):
   ├─ Primary: Most limiting metric
   ├─ Secondary: Backup trigger
   ├─ Both: Can create multiple policies
   └─ Never: Over-complicate
```

---

## Part 5: Scaling Cooldown and Optimization

### Preventing Thrashing and Optimizing Response Time

**Managing the Cooldown Period**

```
Scaling Cooldown Overview:

What It Is:

Definition:
├─ Period: Time after scaling action
├─ Duration: Default 300 seconds (5 minutes)
├─ Effect: ASG won't scale during this time
├─ Purpose: Allow metrics to stabilize
└─ Result: Prevents rapid scaling cycles

Why It Exists:

Problem Without Cooldown:

Scenario (No Cooldown):
├─ T=0s: CPU jumps to 75%, trigger scale-out
├─ T=1s: Add 2 instances (desired +2)
├─ T=10s: New instances still booting
├─ T=20s: New instances starting apps
├─ T=30s: Metric might spike higher! (CPU 80%)
├─ T=31s: Trigger ANOTHER scale-out
├─ T=32s: Add 2 MORE instances (not needed)
├─ T=40s: More instances come online
├─ T=50s: New instances starting apps
├─ Cascade: Rapid additions (wasteful)

Result:
├─ Cost: Unnecessary instances
├─ Waste: Added way more than needed
├─ Problem: Thrashing (up-down-up-down)
└─ Duration: Chaos for 5-10 minutes

With Cooldown (Default 5 min):

Same Scenario:
├─ T=0s: CPU jumps to 75%, trigger scale-out
├─ T=1s: Add 2 instances (desired +2)
├─ T=60s: New instances online, apps starting
├─ T=120s: Apps fully running
├─ T=150s: Metric now at 62% (good!)
├─ T=300s: Cooldown ends, can scale again
├─ T=301s: CPU 65%, consider scale action
├─ Result: Stable, measured, appropriate

Benefits of Cooldown:

Prevents Thrashing:
├─ Consequence: No cascade of scaling
├─ Result: Each decision has time to work
└─ Benefit: Stability

Allows Metrics to Settle:
├─ Time: For load to distribute
├─ Effect: More accurate metrics
└─ Decision: Based on stable data

Reduces Costs:
├─ Effect: Add only what needed
├─ Reason: Prevents over-scaling
└─ Savings: Significant with optimized cooldown

Default Cooldown:

Standard Settings:

Global Cooldown:
├─ Default: 300 seconds (5 minutes)
├─ Application: ALL scaling policies
├─ Change: Can be customized per policy
└─ Typical: Rarely changed from default

Specific Policy Cooldown:
├─ Per policy: Can override global
├─ Length: Different for different policies
├─ Use: Advanced scenarios only
└─ Typical: Keep same as global

Evaluating Cooldown:

Is 300 Seconds Right?

Application Startup Time:

Factor 1: Instance Launch
├─ Time: 30-60 seconds (from create to running)
├─ Task: EC2 starting up

Factor 2: User Data Execution
├─ Time: 60-300 seconds (depends on script)
├─ Example (httpd): 30 seconds
├─ Example (Spring Boot): 2-3 minutes
├─ Example (Database migration): 5+ minutes
└─ Your app: Might be different

Factor 3: Application Warmup
├─ JVM warmup: 1-3 minutes (if Java)
├─ Caches: Fill over time
├─ Connections: Establish to databases
└─ Total: 2-5 minutes typical

Total Time from Scaling Decision to Ready:

Calculation:
├─ Launch: 1 minute
├─ User data: 2 minutes (example app with httpd)
├─ Warmup: 1 minute
├─ Total: ~4 minutes
└─ Safety: Add buffer → 5 minutes (300s)

Right Default?
├─ Your app: 4 min startup
├─ Cooldown: 300s = 5 times suitable
└─ Conclusion: Default is good for most

Shortening Cooldown:

Optimization Strategy:

Quick Startup:
├─ AMI: Pre-baked (not user data)
├─ Time: Reduces to 1 minute
├─ Effect: Can scale faster
└─ Benefit: More responsive

Example:

Pre-baked AMI:
├─ Contains: App already compiled/installed
├─ Time to ready: 1-2 minutes
├─ Compared: User data 5+ minutes
└─ Savings: 3-4 minutes

Shortened Cooldown:
├─ Old: 300 seconds (5 minutes)
├─ New: 120 seconds (2 minutes)
├─ Possibility: Can scale more frequently
└─ Benefit: More responsive to spikes

Tradeoff:
├─ Faster: Responds to changes quickly
├─ Risk: Potential thrashing if too short (< app startup)
└─ Safe: Cooldown ≥ max app startup time

Lengthening Cooldown:

When Needed:

Very Long Startup:
├─ Example: Database migrations
├─ Time: 10-15 minutes
├─ Standard: 300 seconds NOT enough
├─ Solution: Increase cooldown

Very Volatile Metrics:
├─ Example: Bursty traffic
├─ Pattern: Spikes and troughs frequently
├─ Issue: Short cooldown = constant scaling
├─ Solution: Longer cooldown (let spikes pass)

Complex Load Distribution:
├─ Multiple: Resource types compete
├─ Time needed: For rebalancing
├─ Effect: Longer settling time
└─ Solution: Extend cooldown

Example:

Your App:
├─ Database migrations on startup
├─ Time needed: 10 minutes fully ready
├─ Cooldown: 300 seconds (too short!)
├─ Adjustment: Increase to 600-900 seconds (10-15 min)
└─ Result: Respects app needs

Optimization: Reducing Cooldown

Strategy 1: Use Pre-Baked AMI

Traditional Approach (User Data):
├─ Instance launch: 1 min
├─ User data runs: 2-3 min
├─ Average startup: 3-4 minutes
├─ Cooldown needed: 300 seconds minimum

Optimized Approach (Pre-baked):
├─ Instance launch: 1 min
├─ App already installed: No script time
├─ Average startup: 1-2 minutes
├─ Cooldown possible: 120 seconds

Building Pre-baked AMI:

Steps:
1. Launch: Single instance manually
2. Install: Everything your app needs
3. Configure: All settings
4. Test: Verify works
5. Create AMI: From this instance
6. Use: In launch template
7. Benefit: Instances ready in 1-2 min

Savings:
├─ Time per instance: 2-3 minutes saved
├─ Scaling: 40% faster
├─ Cost: Faster to optimality
└─ Performance: Responsive to changes

Strategy 2: Enable Detailed Monitoring

Basic Monitoring (5-minute):

Default:
├─ Frequency: Metrics every 5 minutes
├─ Issue: Slow to detect changes
├─ Delay: Up to 5 minutes behind reality
└─ Impact: Scaling decisions delayed

Detailed Monitoring (1-minute):

Enabled:
├─ Frequency: Metrics every 1 minute
├─ Benefit: 5x faster detection
├─ Cost: Small charge (~$3.50/instance/month)
└─ Worth: For important apps

Implication for Cooldown:

With Detailed:
├─ Metric updates: Every 1 minute
├─ Detection: Rapid
├─ Decision: Can scale more frequently
├─ Cooldown: Can be shorter
└─ Example: 60-120 seconds possible

Without Detailed:
├─ Metric updates: Every 5 minutes
├─ Detection: Delayed
├─ Decision: Needs safety margin
├─ Cooldown: Should be 300+ seconds
└─ Example: 300-600 seconds recommended

Strategy 3: Optimize Application Startup

Application Level Changes:

Connection Pools:
├─ Defer: Initialize on first request
├─ Benefit: Doesn't block startup
└─ Effect: Ready faster

Lazy Loading:
├─ Defer: Load modules as needed
├─ Benefit: Faster initial startup
└─ Effect: Ready faster

Background Tasks:
├─ Defer: Run async after startup
├─ Benefit: Responsive immediately
└─ Effect: Ready for traffic faster

JVM Optimization (if Java):
├─ GraalVM: Faster native startup
├─ Container: Smaller faster images
├─ Tuning: -XX flags for faster startup
└─ Benefit: Can shave minutes

Typical Optimizations:

Before Optimization:
├─ Startup sequence: 3-5 minutes
├─ Caching: Build during startup
├─ Dependencies: All at once
└─ Result: Slow to ready

After Optimization:
├─ Startup sequence: 1-2 minutes
├─ Caching: On-demand lazy loading
├─ Dependencies: Load as needed
└─ Result: Fast to ready

Cooldown Tuning Process:

Step 1: Measure Actual Startup Time

Method:
├─ Launch: Single instance from template
├─ Time: From launch to fully ready
├─ Monitor: Application logs
└─ Record: Total time in minutes

Example Results:
├─ Instance running: 1 minute
├─ App starting: 2 minutes
├─ Ready for requests: 3 minutes total
└─ Safety buffer: Add 2 minute safety
└─ Recommended cooldown: 5 minutes (300s)

Step 2: Set Initial Cooldown

Conservative Start:
├─ Formula: (Measured time) × 1.5
├─ Example: 3 min × 1.5 = 4.5 min (270s)
├─ Round up: To 5 minutes (300s) safe
└─ Start: 300 seconds

Step 3: Monitor Scaling Behavior

Watch for Issues:

Thrashing (too short):
├─ Sign: Constant scaling up/down
├─ Cause: Cooldown too short
├─ Fix: Increase cooldown
└─ Action: Try 600 seconds

Sluggish (too long):
├─ Sign: Doesn't respond to spikes
├─ Cause: Cooldown too long
├─ Fix: Decrease cooldown
└─ Action: Try 150 seconds

Perfect:
├─ Sign: Scales when needed, stable otherwise
├─ Stability: Smooth without thrashing
├─ Response: Responds to prolonged spikes
└─ Keep: This setting

Step 4: Fine-Tune

Iterate:

Log tracking:
├─ Record: All scaling events
├─ Measure: Time between events
├─ Check: Appropriate or too frequent?
└─ Adjust: Cooldown if needed

Guidance:

If Scaling Every Few Minutes:
├─ Problem: Metric unstable or cooldown short
├─ Choice: Usually cooldown too short
├─ Action: Increase to next step (e.g., 300→600)
└─ Observe: Monitor for 1 day

If Never Scaling:
├─ Problem: Metric not triggering threshold
├─ Check: Metric calculation correct?
├─ Check: Threshold reachable?
├─ Action: Adjust threshold or cooldown
└─ Note: Cooldown usually not the issue

If Scales Then Nothing:
├─ Timing: Scales well, then stops
├─ Reason: Probably cooldown effect (normal!)
├─ Expect: This is by design
└─ Conclusion: Working as intended

Cooldown Best Practices:

1. Match Cooldown to Startup Time:
   ├─ Calculation: 1.5 × startup time minimum
   ├─ Example: 2 min startup → 300s cooldown
   └─ Safety: Always round up with buffer

2. Use Pre-baked AMIs:
   ├─ Benefit: Reduction from 4 min to 1-2 min
   ├─ Effect: Could reduce cooldown proportionally
   └─ Recommended: Best efficiency

3. Enable Detailed Monitoring:
   ├─ Cost: Few dollars/month
   ├─ Benefit: 5x faster metric updates
   └─ Worth: For dynamic apps

4. Start with Default:
   ├─ Use: 300 seconds initially
   ├─ Monitor: Behavior for several days
   ├─ Adjust: Only after observing patterns
   └─ Caution: Don't over-tune

5. Document Decision:
   ├─ Record: Why you chose this cooldown
   ├─ Note: Startup time measurement
   ├─ Reason: For future troubleshooting
   └─ Share: With team for consistency

Cooldown vs Scaling Policy Types:

Default Cooldown (ASG-level):
├─ Applies to: Simple scaling
├─ Time: 300 seconds default
├─ Override: Each policy can specify
└─ Effect: Enforced per scaling action

Policy-Specific Cooldown:
├─ Step scaling: Can set different
├─ Target tracking: AWS manages internally
├─ Simple scaling: Uses ASG cooldown or custom
└─ Flexibility: Per-policy customization

Interaction:

If ASG Cooldown = 300s, Policy = 120s:
├─ Which wins: Policy-specific if present
├─ Use: Policy 120s for this policy
├─ Result: 120 second wait

If ASG Cooldown = 300s, Policy = none:
├─ Use: ASG default (300s)
├─ Result: 300 second wait

If ASG Cooldown = 600s, Policy = 120s:
├─ Which wins: Policy-specific (lower applies)
├─ Use: Policy 120s
├─ Result: 120 second wait
```

---

## Part 6: Exam Focus and Best Practices

### Critical Concepts and Real-World Application

**Scaling Policies for AWS SysOps Certification**

```
Essential Exam Concepts:

Scaling Policy Types:

Four Main Types:

1. Target Tracking (Most Common)
   ✓ Single metric with target value
   ✓ AWS calculates scaling automatically
   ✓ Simplest to implement (5 minutes)
   ✓ Example: Keep CPU at 50%

2. Step Scaling (Fine Control)
   ✓ Multiple thresholds with graduated actions
   ✓ You define each step and action
   ✓ Moderate complexity (20-30 minutes)
   ✓ Example: Add 1 if 60%, add 3 if 80%

3. Scheduled (Predictable)
   ✓ Time-based capacity changes
   ✓ Runs at specific times
   ✓ Easy to set up (5 minutes)
   ✓ Example: Scale up at 9 AM Friday

4. Predictive (ML-based)
   ✓ Uses historical analysis
   ✓ Forecasts future load
   ✓ Newer, requires data history
   ✓ Example: Cyclical patterns

Cooldown Period:

Definition:
✓ Time after scaling action
✓ ASG won't scale during this time
✓ Allows metrics to stabilize
✓ Default: 300 seconds (5 minutes)

Why It Exists:
✓ Prevent thrashing (cascade scaling)
✓ Allow new instances to stabilize
✓ Let metrics settle to true value
✓ Reduce unnecessary scaling

Tuning Cooldown:
✓ Measure: Actual app startup time
✓ Formula: 1.5 × startup time
✓ Example: 3 min app → 5 min cooldown
✓ Common: Keep default 300s (works for most)

Metrics:

Best Choice by Workload:

CPU Utilization:
✓ Use: CPU-bound applications
✓ Target: 50-70%
✓ Good: Most common, easiest
✓ Watch: For network/connection limits

Network In/Out:
✓ Use: Network-intensive (uploads, streaming)
✓ Target: 80% of instance max
✓ Example: Video download service

RequestCountPerTarget:
✓ Use: Request-based (web, API)
✓ Target: Determined by load testing
✓ Require: ALB or NLB
✓ Example: 1000 requests/target/min

Custom Metrics:
✓ Use: Application-specific needs
✓ Example: Queue depth, active connections
✓ Benefit: Very accurate for your app
✓ Cost: Application code required to push

Metric Selection:

Key Question: What Limits Performance?
✓ CPU saturation: Use CPU metric
✓ Network saturation: Use network metric
✓ Connection limit: Use custom metric
✓ Request queue: Use request count metric

Exam-Style Questions:

1. "Default cooldown period?"
   Answer: 300 seconds (5 minutes)

2. "Which metric for CPU-bound workload?"
   Answer: Average CPU Utilization

3. "How does cooldown prevent thrashing?"
   Answer: Prevents immediate re-scaling, allows metrics to stabilize and new instances to become effective

4. "What's calculated by AWS in target tracking?"
   Answer: How many instances needed to reach target value (you just set target, AWS does math)

5. "When would you use step scaling instead of target tracking?"
   Answer: When you need fine-grained, proportional responses at different metric levels

6. "Default scale-out behavior?"
   Answer: Aggressive (quick addition of instances)

7. "Default scale-in behavior?"
   Answer: Conservative (slower removal, more cautious)

8. "Custom metric example?"
   Answer: Queue depth, active connections, database pool usage

9. "Best practice: Pre-baked vs user data?"
   Answer: Pre-baked AMI (faster startup, can reduce cooldown)

10. "How to reduce cooldown?"
    Answer: Use pre-baked AMI to reduce startup time, so cooldown can be shorter

Real-World Scenarios:

Scenario 1: Sudden Traffic Spike (Black Friday)

Setup:
├─ Metric: CPU Utilization
├─ Target: 60%
├─ Initial: 5 instances
├─ Cooldown: 300 seconds

What Happens:

T=0: Traffic surge hits
├─ CPU: From 40% to 85%
├─ Trigger: Policy activated
├─ Action: Target tracking calculates
├─ Result: Scale to 8 instances (est.)

T=1-60s: New instances launching
├─ Status: Instances booting
├─ Cooldown: Active
├─ Activity: Visible in history
└─ CPU: Still high (old instances)

T=60-120s: User data running
├─ Status: Apps installing
├─ Load: Still on old instances
└─ CPU: Drops slightly as load spreads

T=120-300s: Instances becoming active
├─ Status: Apps online
├─ Load: Spreading to new instances
├─ CPU: Declining toward target
└─ Cooldown: Ending

T=300s+: Cooldown expired
├─ CPU: Measured at ~62% (near 60%)
├─ Decision: May scale another instance
├─ Result: Fine-tuning to exact target

Scenario 2: Web Application with Business Hours

Setup:

Schedule:
├─ 7:00 AM: Scale to 3 instances
├─ 9:00 AM: Scale to 8 instances
├─ 5:00 PM: Scale to 4 instances
├─ 10:00 PM: Scale to 1 instance

Dynamic:
├─ Metric: CPU Utilization
├─ Target: 60%
├─ Cooldown: 300 seconds

Timeline:

7:00 AM Monday:
├─ Scheduled: Scale to 3
├─ Dynamic: Monitoring CPU
├─ Traffic: Starting to arrive

9:00 AM:
├─ Scheduled: Scale to 8
├─ Traffic: Morning peak arrives
├─ Dynamic: CPU tracking 60% as load comes

Noon (lunch spike):
├─ Scheduled: No change (still 8)
├─ Traffic: Increases suddenly
├─ CPU: Jumps to 78%
├─ Dynamic: Scales to 10
├─ Result: Absorbs spike

1:00 PM:
├─ Traffic: Normalizes
├─ CPU: Settles to 62%
├─ Dynamic: No further scaling
└─ Stable: 10 instances handling well

5:00 PM:
├─ Scheduled: Scale to 4
├─ Note: Scheduled overrides dynamic lower
├─ Reasoning: Evening is planned lower traffic
└─ Cost: Reduced for lower-traffic period

10:00 PM:
├─ Scheduled: Scale to 1
├─ Traffic: Minimal
├─ Cost: Minimal
└─ Availability: Always at least 1

Scenario 3: Video Processing (Batch Job)

Setup:

Trigger:
├─ Custom metric: Queue depth
├─ Target: No more than 100 jobs in queue
├─ Processing: Each machine handles 20 jobs/hour
└─ Dynamic: Scale based on queue

Timeline:

8:00 AM:
├─ Queue: 15 jobs
├─ Instances: 1 (minimum)
├─ Processing: 1 job/hour per machine

User Uploads 500 Videos:

8:10 AM:
├─ Queue: 515 jobs
├─ Metric: 515 > 100 (target exceeded 4x)
├─ Step scaling: Add 8 instances
├─ Instances: 9 total
├─ Processing: 9 jobs/hour
└─ Queue size: Should drop

8:30 AM:
├─ Queue: 400 jobs (uploaded more, some processed)
├─ Per instance: 400 ÷ 9 = 44 jobs/instance
├─ Status: Still high
├─ Processing: 9 jobs/hour (still scaling)

9:00 AM:
├─ Queue: 200 jobs (steady processing)
├─ Per instance: 200 ÷ 9 = 22 jobs/instance
├─ Status: Coming down
├─ Processing: Continuing

12:00 PM:
├─ Queue: 50 jobs
├─ Target: 100 (could scale down)
├─ Cooldown: Preventing scale-in (safety)
└─ Processing: Nearly caught up

1:00 PM:
├─ Queue: 10 jobs
├─ Instances: Scale to 2 (minimum)
├─ Processing: Finishing last jobs
└─ Cost: Very low going forward

Exam Comparison Questions:

Compare Target Tracking vs Step Scaling:

Target Tracking:
✓ Setup: 5 minutes
✓ Metric: One only
✓ Complexity: Simple
✓ Control: Low (AWS automatic)
✓ Best for: Most use cases

Step Scaling:
✓ Setup: 30 minutes
✓ Metric: One with multiple thresholds
✓ Complexity: Medium/high
✓ Control: High (you define steps)
✓ Best for: Complex needs

Compare Scheduled vs Dynamic:

Scheduled:
✓ Timing: Predictable times
✓ Precision: Exactly when you plan
✓ Example: 9 AM business start
✓ Best: Known patterns

Dynamic:
✓ Timing: Metric-based (real-time)
✓ Precision: Responds to actual demand
✓ Example: Responding to spike
✓ Best: Unexpected changes

Best Practices:

1. Start Simple:
   ├─ Use: Target tracking first
   ├─ Monitor: Behavior for days
   ├─ Adjust: If needed
   └─ Only then: Consider step scaling

2. Choose Correct Metric:
   ├─ Test: Load test to find bottleneck
   ├─ Measure: What actually limits performance
   ├─ Select: Metric representing that limit
   └─ Avoid: Arbitrary choices

3. Set Realistic Target:
   ├─ Conservative: Start higher (70%)
   ├─ Monitor: Real behavior
   ├─ Lower: Once confident
   └─ Balance: Cost vs performance

4. Use Scheduled + Dynamic:
   ├─ Scheduled: For known patterns
   ├─ Dynamic: For unexpected changes
   ├─ Together: Best approach
   └─ Coverage: Most situations handled

5. Optimize for Faster Scaling:
   ├─ AMI: Pre-baked (reduces startup)
   ├─ Monitoring: Detailed (faster detection)
   ├─ Cooldown: Reduced if possible
   └─ Result: More responsive system

6. Document Scale Decisions:
   ├─ Record: Why each setting chosen
   ├─ Startup: Actual measured time
   ├─ Metric: Why this metric
   ├─ Target: Why this value
   └─ Benefit: Future troubleshooting

Common Mistakes:

Mistake 1: Metric Selection
✗ Not testing before production
✗ Choosing arbitrary metric
✗ Ignoring actual bottleneck
✓ Fix: Load test to identify real limit

Mistake 2: Target Too Low
✗ Result: Constant scaling
✗ Cost: High (over-provisioned)
✗ Thrashing: Continuous changes
✓ Fix: Higher target, then lower gradually

Mistake 3: Cooldown Too Short
✗ Result: Cascade scaling
✗ Cause: Metric unstable due to new instances
✗ Problem: Over-provisioning
✓ Fix: Increase cooldown to app startup time

Mistake 4: Only Dynamic, No Scheduled
✗ Problem: Always reactive
✗ Cost: Higher than needed
✗ Performance: Slower to peak
✓ Fix: Add scheduled scaling for known patterns

Mistake 5: Complex Step Scaling
✗ Hard to manage: Too many steps
✗ Error-prone: Hard to test all scenarios
✗ Maintenance: Difficult to adjust
✓ Fix: Start with target tracking, only escalate if needed

Production Deployment Checklist:

Before Going Live:

Testing:
☐ Load test with realistic traffic
☐ Verify metric tracking correctly
☐ Confirm policy triggers at right time
☐ Verify scaling up works
☐ Verify scaling down works
☐ Check cooldown timing is reasonable
☐ Validate no thrashing occurs
☐ Check cost during scaling

Monitoring:
☐ Dashboard created for metrics
☐ Alarms configured for anomalies
☐ Activity history reviewed
☐ Logs collected and searchable
☐ Team trained on scaling behavior
☐ Runbook for issues created
☐ On-call procedure defined

Documentation:
☐ Metric choice documented (why this metric)
☐ Target value justified (why this target)
☐ Cooldown reasoning explained (why this duration)
☐ Scaling policies listed
☐ Thresholds documented
☐ Runbook for troubleshooting
☐ Contact info for escalation

Configuration:
☐ Min capacity set (ensure HA if needed)
☐ Max capacity set (budget protected)
☐ Desired capacity reasonable
☐ Health checks enabled
☐ Grace period appropriate
☐ Termination policy defined
☐ ASG tags complete

Ready: Deploy to production

Conclusion:

Scaling policies are the brain of elastic infrastructure. By selecting the right policy type, metric, and parameters, you enable your application to automatically respond to demand changes. Master these concepts for certification and production success!
```

---

## Conclusion

You now understand all four scaling policy types (target tracking, step, scheduled, predictive), how to select appropriate metrics for your workload, and how to optimize the scaling cooldown period. These concepts form the foundation of responsive, cost-efficient infrastructure scaling in AWS!
