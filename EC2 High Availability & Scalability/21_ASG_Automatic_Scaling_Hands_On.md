# ASG Automatic Scaling Hands-On Demonstration

Step-by-step practical guide to implementing and observing Auto Scaling Group scaling policies in action, including scheduled actions, predictive scaling, dynamic scaling with CPU stress testing, and CloudWatch alarm integration.

---

## Part 1: Scaling Policy Types and Setup

### Overview of Three Scaling Categories

**Scheduled, Predictive, and Dynamic Scaling**

```
Three Scaling Policy Categories:

Overview:

1. Scheduled Actions
   ├─ What: Pre-planned scaling at specific times
   ├─ When: You know exactly when to scale
   ├─ Setup: Easy (5 minutes)
   ├─ Use: Business hours scaling, known events
   └─ Example: Scale up 9 AM weekdays, down 6 PM

2. Predictive Scaling Policies
   ├─ What: ML-driven forecasting
   ├─ When: Historical patterns available
   ├─ Setup: Complex (requires data history)
   ├─ Use: Cyclical workloads, repeating patterns
   └─ Example: Weekly pattern repeats (set it once)

3. Dynamic Scaling Policies
   ├─ What: Real-time metric-based
   ├─ When: Immediate response needed
   ├─ Setup: Medium (policy creation)
   ├─ Use: Variable workloads, spikes
   └─ Example: Scale when CPU > 70%

When to Use Each:

Decision by Workload:

Predictable Pattern (Business Hours):
└─ Use: Scheduled actions

Cyclical Pattern (Weekly/Monthly Repeats):
└─ Use: Predictive scaling (if history exists)

Variable Unpredictable:
└─ Use: Dynamic scaling

Known Spike Events:
├─ Predictable timing: Scheduled actions
└─ Unknown timing: Dynamic scaling

Best Practice:
└─ Combine: All three (scheduled + dynamic common)

Architecture Overview:

┌─────────────────────────────────────────────┐
│       ASG Automatic Scaling Decision        │
└──────────────────┬──────────────────────────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
   Scheduled   Predictive  Dynamic
      │           │          │
      └───────────┴──────────┘
              │
              ▼
       Evaluate All
       Choose Action
              │
              ▼
       Desired Capacity
       Change (if needed)
              │
              ▼
       Scale-Out or
       Scale-In Activity
              │
              ▼
       New Instances or
       Terminate Instances
```

---

## Part 2: Scheduled Actions

### Pre-Planned Scaling at Specific Times

**Setting Up Time-Based Scaling**

```
Scheduled Actions Overview:

What They Are:

Definition:
├─ Trigger: Specific date and time
├─ Action: Change desired/min/max capacity
├─ Recurrence: Once or repeating schedule
├─ Purpose: Pre-positioned capacity
└─ Example: Scale up every Monday 9 AM

When to Use:

Perfect For:
├─ Business hours: 9 AM scale up, 6 PM scale down
├─ Weekly patterns: Specific days busier
├─ Known events: Black Friday, sales
├─ Predictable workloads: Same pattern repeats
└─ Cost optimization: Scale down after hours

Types of Schedules:

One-Time:
├─ Date: Specific date (e.g., March 15)
├─ Time: Specific time (e.g., 10:00 AM)
├─ Duration: Just one instance in time
└─ Use: Single events (conference, launch)

Recurring:
├─ Pattern: Repeats regularly
├─ Options: Daily, weekly, monthly, custom cron
├─ Duration: Ongoing until end date
└─ Use: Regular patterns (business hours)

Creating Scheduled Actions:

AWS Console Navigation:

Step 1: Access ASG

ASG Selection:
├─ Service: EC2 → Auto Scaling Groups
├─ Find: Your ASG
├─ Click: To open details
└─ Ready: Configure actions

Step 2: Navigate to Scheduled Actions

Tab: "Scheduled actions"
├─ Button: Create scheduled action
└─ Form: Scheduled action creation

Step 3: Configure Action

Basic Information:

Name:
├─ Field: "Scheduled action name"
├─ Example: "scale-up-monday-morning"
├─ Purpose: Identify the action
└─ Note: Descriptive for clarity

Time Zone:
├─ Selector: Your time zone
├─ Important: Matches your business hours
└─ Example: America/New_York

Start Time:
├─ Date: When action starts
├─ Time: Hour and minute
├─ Recurrence: None for one-time
└─ Example: 2024-03-18 09:00 AM

End Time (Optional):
├─ Date: When action stops repeating
├─ Example: 2024-12-31 (end of year)
├─ Leave blank: If ongoing
└─ Use: To disable seasonal action

Recurrence (Optional):

Options:

None:
├─ Meaning: One-time only
├─ Use: Single events
└─ Example: Conference next Saturday

Daily:
├─ Meaning: Every day same time
├─ Use: Daily business hours
└─ Example: 9 AM every day

Weekly:
├─ Meaning: Specific days of week
├─ Days: Choose which days
├─ Use: Weekday vs weekend patterns
└─ Example: Monday-Friday 9 AM

Monthly:
├─ Meaning: Same day each month
├─ Use: Monthly patterns
└─ Example: 1st of month 9 AM

Cron Expression:
├─ Format: cron(09 * * MON-FRI *)
├─ Meaning: 9 AM Monday-Friday
├─ Power: Full customization
└─ Use: Complex patterns

Step 4: Specify Capacity Values

Min Capacity:
├─ Field: "Minimum capacity"
├─ Example: 1 or 2
├─ Leave blank: Keep current
└─ Note: Won't go below this

Max Capacity:
├─ Field: "Maximum capacity"
├─ Example: 10
├─ Leave blank: Keep current
└─ Note: Won't go above this

Desired Capacity:
├─ Field: "Desired capacity"
├─ Example: 5 (for business hours)
├─ Leave blank: Keep current
└─ Note: Primary value changed

Choice:
├─ Often: Only change desired
├─ Allow: Min/max to stay fixed
└─ Result: Simpler management

Example Configurations:

Configuration 1: Business Hours Scaling

Morning Scale-Up:
├─ Name: "business-hours-scale-up"
├─ Recurrence: Monday-Friday
├─ Start time: 09:00 AM
├─ Desired: 8 instances
└─ Purpose: Ready for peak business

Evening Scale-Down:
├─ Name: "business-hours-scale-down"
├─ Recurrence: Monday-Friday
├─ Start time: 06:00 PM
├─ Desired: 2 instances
└─ Purpose: Cost optimize after hours

Weekend Minimal:
├─ Name: "weekend-minimal"
├─ Recurrence: Saturday, Sunday
├─ Desired: 1 instance
└─ Purpose: Minimal weekend traffic

Example 2: Event-Based Scaling

Black Friday Preparation:
├─ Name: "black-friday-scale-up"
├─ Schedule: Friday 12:00 AM (midnight)
├─ Desired: 30 instances
├─ Purpose: Ready for huge traffic spike

Black Friday Scale-Down:
├─ Name: "black-friday-scale-down"
├─ Schedule: Monday 8:00 AM (after event)
├─ Desired: 5 instances (back to normal)
└─ Purpose: Return to normal capacity

Example 3: Complex Weekly Pattern

Pattern: Technology company

Monday-Wednesday:
├─ Name: "mid-week"
├─ Desired: 6 instances
└─ Reason: Moderate traffic

Thursday-Friday:
├─ Name: "end-week-peak"
├─ Desired: 10 instances
└─ Reason: Planning meetings, reports due

Saturday-Sunday:
├─ Name: "weekend"
├─ Desired: 2 instances
└─ Reason: Minimal/no work

Creating Multiple Scheduled Actions:

Workflow:

Step 1: Create "scale-up" action
├─ Time: 9 AM weekdays
├─ Desired: 8
└─ Save

Step 2: Create "scale-down" action
├─ Time: 6 PM weekdays
├─ Desired: 2
└─ Save

Step 3: Create "weekend" action
├─ Time: Friday 6 PM (no work Sat-Sun)
├─ Desired: 1
└─ Save

Result:
├─ All created and active
├─ Each runs on its own schedule
├─ Together they create pattern
└─ Cost and performance optimized

Viewing Scheduled Actions:

In AWS Console:

Tab: "Scheduled actions"
├─ List: All actions for this ASG
├─ Name: Identification
├─ State: Active or inactive
├─ Min/Max/Desired: Current values
├─ Recurrence: When it runs
├─ Next run: When scheduled action executes next
└─ Actions: Edit or delete

Editing Scheduled Actions:

Changes:

Click: Edit action
├─ Modify: Any field (time, capacity, recurrence)
├─ Save: Changes applied
└─ Effect: Next execution uses new values

Deleting Scheduled Actions:

Remove:

Click: Delete action
├─ Confirm: Are you sure?
├─ Result: Action removed
└─ Effect: No longer runs

Monitoring Scheduled Actions:

Activity History:

When Action Executes:
├─ Entry appears: "Scheduled action executed"
├─ Shows: Capacity change (from X to Y)
├─ Time: Exact timestamp
└─ Status: Successful or failed

CloudWatch Logs:

Event Recorded:
├─ Service: AWS::AutoScaling
├─ Event: ScheduledActionExecuted
├─ Details: ASG, new capacity, action name
└─ Duration: When it happened

Activity History Example:

Timestamp: 2024-03-18 09:00:00 UTC
├─ Event: "Scheduled action executed"
├─ Reason: "business-hours-scale-up"
├─ Capacity: Changed from 2 to 8
├─ Status: Successful
└─ Next Action: Monday 6 PM scale-down

Scheduled Actions Best Practices:

1. Plan Schedule First:
   ├─ Map: When traffic typically high/low
   ├─ Document: Expected pattern
   ├─ Align: With business hours
   └─ Result: Optimized schedule

2. Use Descriptive Names:
   ├─ Clear: What does it do?
   ├─ Time: When does it run?
   ├─ Example: "weekday-morning-scale-up"
   └─ Benefit: Easy to manage

3. Start Conservative:
   ├─ Estimate: Required capacity
   ├─ Add buffer: 20-30% extra
   ├─ Monitor: Actual usage
   └─ Adjust: Once data available

4. Combine with Dynamic:
   ├─ Scheduled: Baseline capacity
   ├─ Dynamic: Handle surprises
   ├─ Together: Best approach
   └─ Coverage: All scenarios

5. Document and Share:
   ├─ Record: Why each time chosen
   ├─ Capacity: Why that specific number
   ├─ Team: Ensure everyone knows schedule
   └─ Runbook: For troubleshooting

Limitations:

Scheduled actions cannot:
├─ Respond: To unexpected spikes
├─ Adapt: If pattern changes
├─ Handle: Unpredictable events
└─ Solution: Combine with dynamic

For Unexpected Events:
├─ Use: Dynamic scaling
├─ Combined: Scheduled + Dynamic
└─ Result: Full coverage
```

---

## Part 3: Predictive Scaling Policies

### Machine Learning-Based Forecasting

**Automatic Forecast and Scaling**

```
Predictive Scaling Overview:

What It Is:

Definition:
├─ Technology: Machine learning
├─ Data: Analyzes historical metrics
├─ Forecast: Predicts future load
├─ Action: Scales proactively
└─ Result: Ahead of demand

When to Use:

Best For:
├─ Cyclical workloads: Patterns repeat
├─ Weekly patterns: Similar each week
├─ Monthly patterns: Same each month
├─ Seasonal: Year to year cycles
└─ Repeating: Predictable structure

Not For:
├─ Unpredictable: Random load spikes
├─ New apps: Insufficient history
├─ One-time: Single events
└─ Rare: Infrequent patterns

How It Works:

Process:

1. Data Collection:
   ├─ Period: Last 2+ weeks of data
   ├─ Metrics: CPU, network, request count, etc.
   ├─ Granularity: Hourly buckets
   └─ Analysis: Patterns extracted

2. Pattern Recognition:
   ├─ Identifies: Weekly patterns
   ├─ Recognizes: Similar time slots
   ├─ Groups: Similar demand profiles
   └─ Creates: Forecasting model

3. Forecast Generation:
   ├─ Next steps: Predicts next 48 hours
   ├─ Frequency: Updated daily
   ├─ Accuracy: Improves with data
   └─ Range: Shows confidence interval

4. Scaling Decisions:
   ├─ Pre-scales: Before demand arrives
   ├─ Timing: Early enough to launch instances
   ├─ Amount: Based on forecast
   └─ Result: Capacity ready when needed

Example Scenario:

Retail Store Website:

Week 1-4 Data:
├─ Mondays 9 AM: Traffic spikes to 1000 req/min
├─ Every Friday: Traffic spikes to 1500 req/min
├─ Saturdays: Moderate traffic
├─ Sundays: Low traffic
├─ Pattern: Clearly cyclical

ML Analysis:
├─ Learns: Friday = peak day
├─ Learns: 9 AM = traffic starts
├─ Learns: Pattern repeats weekly
└─ Model: Created with 95% confidence

Forecasting:
├─ Next Monday 8:45 AM: Scale up to 8 instances
├─ Next Friday 8:00 AM: Scale up to 12 instances
├─ Saturday 8:00 AM: Scale to 6 instances
└─ Result: Ready 15 minutes before spike

Setting Up Predictive Scaling:

Requirements:

Data History:
├─ Minimum: 2 weeks of data
├─ Better: 4+ weeks
├─ Best: 8+ weeks
└─ Before: Can't use before history exists

Metric Selection:
├─ One metric: Per predictive policy
├─ Options: CPU, network, request count, custom
├─ Choice: Most limiting metric
└─ Target: Set target utilization

AWS Console Steps:

Step 1: Navigate to Policies

ASG Page:
├─ Tab: "Automatic scaling"
├─ Section: "Scaling policies"
├─ Button: "Create scaling policy"
└─ Select: "Predictive scaling policy"

Step 2: Configure Policy

Policy Name:
├─ Field: "Predictive scaling policy name"
├─ Example: "retail-weekly-pattern"
└─ Purpose: Identify policy

Metric:
├─ Options: CPU, Network, ALB RequestCount, Custom
├─ Select: Your limiting metric
└─ Example: CPUUtilization

Target Value:
├─ Field: "Target utilization (%)"
├─ Range: 0-100 (percentage)
├─ Example: 50%
└─ Meaning: Maintain at this level

Step 3: Additional Settings

Scaling Mode:

Option 1: Forecast and Scale
├─ Meaning: Full predictive (pre-scales)
├─ Action: Scales before demand
├─ Benefit: Proactive
└─ Best: Normal use

Option 2: Forecast Only
├─ Meaning: Analyze only
├─ Action: No actual scaling (test mode)
├─ Use: Learning, testing
└─ Benefit: Observe without action

Maximum Instance Age:
├─ Option: Set instance refresh
├─ Meaning: Replace instances periodically
├─ Use: Update AMI versions
└─ Advanced: Usually leave default

Pre-Scaling Buffer:
├─ Option: Add time before scaling
├─ Example: Scale 30 minutes early
├─ Use: Ensure capacity ready
└─ Advanced: Usually default ok

Step 4: Review and Create

Preview:
├─ Shows: Forecast for next 7 days
├─ Visualization: When scales will happen
├─ Timing: How many instances predicted
└─ Check: Does it match expectations?

Create:
├─ Button: "Create"
└─ Status: Policy active

Predictive Scaling Real-World Example:

Scenario: SaaS Productivity App

History (2 weeks observed):

Monday-Friday:
├─ 8 AM: Traffic starts (scale to 5)
├─ 12 PM: Lunch peak (scale to 8)
├─ 5 PM: End of work (scale to 3)
├─ 10 PM: Minimal (scale to 1)

Weekend:
├─ Saturday/Sunday: Low (scale to 1)

Pattern Recognized:
├─ Weekly cycle: Monday-Friday identical
├─ Daytime: Peaks at lunch
├─ Evening: Drops after work
├─ Weekend: Minimal usage
└─ Confidence: 92%

Predictive Forecast (Week 3):

Monday (Predicted):
├─ 7:30 AM: Forecast traffic increase
├─ 7:45 AM: Pre-scale to 5 instances
├─ 8:00 AM: Users arrive (ready!)
├─ 12:00 PM: Scale to 8 (lunch peak)
├─ 5:00 PM: Scale to 3 (work ends)
└─ 10:00 PM: Scale to 1

Result:
├─ When users arrive: Capacity ready
├─ No wait time: Instant response
├─ Cost: Optimized per demand
└─ Performance: Excellent

Monitoring Predictive Scaling:

Activity History:

Entry Example:

Timestamp: 2024-03-18 07:45:00
├─ Event: "Predictive scaling executed"
├─ Policy: "retail-weekly-pattern"
├─ Reason: "Forecast predicts traffic spike"
├─ Action: "Desired capacity changed from 1 to 5"
├─ Status: "Successful"
└─ Forecast: "Based on 4 weeks of history"

Forecast Dashboard:

Visualization:
├─ Chart: 7-day forecast
├─ Shows: When scaling happens
├─ Indicates: Confidence level
├─ Helpful: Verify forecasting accuracy
└─ Use: Understand predictions

Adjusting Predictions:

If Forecasting Wrong:

Issue: Scaling too much or too little
├─ Cause: Pattern changed
├─ Solution: Observe new pattern (2+ weeks)
├─ Note: ML needs data to learn
└─ Timeline: 2-4 weeks to adjust

Issue: Scaling wrong time
├─ Cause: Pattern different than expected
├─ Solution: Check historical data
├─ Note: Historical outlier might confuse ML
└─ Action: Remove anomalies if possible

Predictive Scaling Best Practices:

1. Wait for Data:
   ├─ Collect: At least 2 weeks
   ├─ Better: 4-8 weeks
   ├─ Reason: More accurate model
   └─ Result: Better predictions

2. Use Stable Metric:
   ├─ Choose: Most limiting metric
   ├─ Avoid: Noisy/chaotic metrics
   ├─ Test: Metric consistency
   └─ Result: Reliable forecasting

3. Monitor Accuracy:
   ├─ Track: Forecast vs actual
   ├─ Measure: How often correct?
   ├─ If poor: Investigate pattern change
   └─ Adjust: If needed

4. Combine with Dynamic:
   ├─ Predictive: Baseline forecast
   ├─ Dynamic: Handle surprises
   ├─ Example: Predictive + Target Tracking
   └─ Coverage: Both patterns + spikes

5. Document Patterns:
   ├─ Record: What pattern exists
   ├─ Why: Business reason for pattern
   ├─ Expected: How often cycles
   └─ Benefit: Team understanding

Limitations:

Predictive Scaling cannot:
├─ Predict: New/unprecedented patterns
├─ Handle: One-time events
├─ Learn: From insufficient data
├─ Scale: Brand new applications
└─ Solution: Combine with scheduled + dynamic

When Patterns Change:
├─ Takes time: ML re-learns
├─ Duration: 2+ weeks observed
├─ Until then: May be inaccurate
└─ Mitigation: Use target tracking meanwhile

Cost Consideration:
├─ Usage: Slight extra cost for ML
├─ Benefit: Cost savings from optimization
├─ ROI: Positive for most applications
└─ Worth: For predictable workloads
```

---

## Part 4: Dynamic Scaling - Target Tracking Hands-On

### Practical Demonstration with CPU Stress Testing

**Setting Up and Observing Target Tracking Scaling**

```
Dynamic Scaling Setup:

Prerequisites:

Before Starting:

1. ASG created:
   ├─ Name: Your ASG
   ├─ Min: 1 instance
   ├─ Max: 3 (minimum, for testing)
   ├─ Desired: 1
   └─ Status: One instance running and healthy

2. Instance running:
   ├─ State: Running
   ├─ Health: Healthy (in target group if ALB)
   ├─ SSH: EC2 Instance Connect available
   └─ Ready: For stress testing

3. CloudWatch:
   ├─ Metrics: EC2 publishing to CloudWatch
   ├─ CPU: Being tracked
   └─ Available: For policy decisions

Creating Target Tracking Policy:

Step 1: Navigate to ASG

Console:
├─ Service: EC2 → Auto Scaling Groups
├─ Find: Your ASG
├─ Click: To open details
└─ Ready: Create policy

Step 2: Access Automatic Scaling Tab

Tab: "Automatic scaling"
├─ Section: "Scaling policies"
├─ List: Shows existing policies
└─ Button: "Create scaling policy"

Step 3: Select Policy Type

Policy Type:
├─ Radio: "Target tracking scaling"
├─ (Not: Simple scaling or step scaling)
└─ Click: Select and proceed

Step 4: Configure Target Tracking Policy

Policy Name:
├─ Field: "Desired_tracking_scaling_policy"
├─ Or custom: "target-cpu-50" (descriptive)
├─ Purpose: Identify policy in list
└─ Example: "cpu-target-tracking"

Metric:
├─ Dropdown: "Choose a metric"
├─ Options: ASGAverageCPUUtilization, Network, etc.
├─ Select: "ASGAverageCPUUtilization"
└─ Meaning: Average CPU across all instances

Target Value:
├─ Field: "Target value"
├─ Value: 40 (%)
├─ Meaning: Keep CPU at 40%
├─ Note: Can adjust based on workload
└─ Why 40: Conservative for demonstration (easy to achieve)

Adjust Policy:

Scale-Out Behavior:
├─ Default: AWS-managed (usually good)
├─ Options: Rapid or measured
├─ Typical: Leave default
└─ Effect: Aggressively adds when over target

Scale-In Behavior:
├─ Default: AWS-managed (usually good)
├─ Disable checkbox: Don't scale in (if wanted)
├─ Typical: Leave enabled
└─ Effect: Conservative removal

Cooldown:
├─ AWS manages: Internally
├─ Cannot customize: In target tracking
├─ Default: Reasonable based on app
└─ Why: AWS optimizes automatically

Step 5: Review and Create

Summary:
├─ Shows: Policy name
├─ Shows: Metric (CPU)
├─ Shows: Target (40%)
├─ Shows: ASG name
└─ Verify: All correct

Create:
├─ Button: "Create scaling policy"
├─ Status: Policy being created
├─ Confirmation: "Policy created successfully"
└─ Ready: Policy active and monitoring

Initial State After Creation:

Observations:

Policy Active:
├─ Listed: In "Scaling policies"
├─ Status: Active
├─ CloudWatch: Alarms being created
└─ Monitoring: Metrics tracked

Current CPU:
├─ Value: Likely very low (<5%)
├─ Reason: Instance idle
├─ Below target: 40% (no scaling needed)
└─ Expected: One instance remains

AWS CloudWatch Alarms Created:

Background Activity:

Automatic Alarms:
├─ AlarmHigh: CPU > 40% (scale out)
├─ AlarmLow: CPU < 28% (scale in)
├─ Purpose: Trigger scaling actions
└─ Created: Automatically by target tracking

Why Two Alarms?

AlarmHigh (Scale Out):
├─ Trigger: CPU exceeds target + buffer
├─ Threshold: ~40% (at or above)
├─ Effect: Adds instances
└─ Purpose: Respond to high load

AlarmLow (Scale In):
├─ Trigger: CPU drops below threshold
├─ Threshold: ~28% (lower than high)
├─ Purpose: Hysteresis (prevents oscillation)
├─ Effect: Removes instances conservatively

Hysteresis Explained:

Why Different Thresholds?

Problem Without Hysteresis:
├─ Threshold: 40% for both up and down
├─ Scenario: CPU at 39.5% (just below)
├─ Action: Scale down (remove instance)
├─ Result: CPU jumps to 42% (not enough now)
├─ Reaction: Scale up immediately
├─ Problem: Rapid oscillation (thrashing!)

With Hysteresis (Different Thresholds):
├─ Scale-out: At 40% (add instance)
├─ Scale-in: At 28% (remove instance)
├─ Gap: 12% cushion between
├─ Effect: Stable hysteresis
├─ Benefit: No oscillation
└─ Result: Smooth scaling

Intentional Design:
├─ Purpose: Stability
├─ Effect: Conservative scale-in
├─ Reason: Better than thrashing
└─ Result: Better performance

CPU Stress Testing:

Purpose:

Why Stress Test:

Need:
├─ Observe: Scaling in real-time
├─ Trigger: Scale-out threshold
├─ Demonstrate: Policy working
└─ Learn: Timing and behavior

Method:
├─ Artificially increase: CPU utilization
├─ 100% CPU: Easy to exceed 40% target
├─ Duration: Temporary (for testing)
└─ Monitor: Scaling response

Installing Stress Test Tool:

Step 1: Connect to Instance

Method:

EC2 Instance Connect:
├─ Service: EC2 → Instances
├─ Find: Your instance
├─ Connect: EC2 Instance Connect (button)
├─ Shell: Opens in browser
└─ Ready: Terminal connected

Alternative (SSH):
├─ IP: Get public or private IP
├─ Command: ssh -i key.pem ec2-user@IP
├─ Connection: Your machine to instance
└─ Alternative: If Instance Connect unavailable

Step 2: Install Stress Utility

Command 1 (Update packages):
```bash
sudo yum update -y
```

Command 2 (Install stress):
```bash
sudo yum install -y stress
```

Verification:
```bash
stress --help
```

Should show: Usage information
├─ Meaning: Installed successfully
└─ Ready: Use stress tool

Step 3: Run Stress to Increase CPU

Command:
```bash
stress -c 4 --timeout 600s
```

Breaking Down Command:

-c 4:
├─ Meaning: Create 4 CPU workers
├─ Effect: Each uses 1 CPU fully
├─ Result: 400% CPU usage (on 1 vCPU instance → 100%)
└─ Power: Maxes out available CPU

--timeout 600s:
├─ Meaning: Run for 600 seconds (10 minutes)
├─ Effect: Limited duration (won't run forever)
├─ Reason: Control test length
└─ Can adjust: Change 600 to different seconds

Execution:
```
stress: info: [1234] dispatching cachehunks from pool...
stress: info: [1234] stressing 4 CPU cores
stress: info: [1234] successful run completed in 600s
```

Output:
├─ Shows: Running and stressing
├─ Status: Active stress
└─ Duration: Until timeout or stop

Monitoring Stress Test:

While Stress Running:

In Another Terminal (or Another Session):

Check CPU:
```bash
top
```

Shows:
├─ Processes: stress tasks using CPU
├─ CPU: %CPU columns showing load
├─ Tasks: Multiple stress processes
└─ Load: High CPU % (near 100%)

CloudWatch Metrics:

AWS Console:
├─ Service: CloudWatch
├─ Metrics: EC2 Metrics
├─ Instance: Select your instance
├─ CPU Utilization: Shows spike to ~100%
└─ Timeline: Rising in real-time

ASG Monitoring:

View While Running:

CloudWatch Tab:
├─ Open: ASG → Monitoring tab
├─ Chart: CPU Utilization graph
├─ Observe: Spike to 100%
├─ Real-time: Updates every minute
└─ Watch: Scale-out activity begin

Activity History:

Access: ASG → Activity → Activity history
├─ Watch: Full screen or background
├─ Already: Policies evaluating
├─ When: CPU exceeds 40%, entry appears
└─ Trigger: Scale-out action initiated

Scale-Out Observation:

Timeline During Stress:

T=0-300 seconds: Stress running
├─ CPU: Rising toward 100%
├─ Metric: Reported to CloudWatch
└─ Policy: Evaluating metrics

T=300-360 seconds (depends on metric period):
├─ CloudWatch: Evaluates if exceeds threshold
├─ Alarm: Goes to "ALARM" state
├─ Trigger: ASG scaling action begins
└─ Activity: "Launching 1 instance" appears

T=360-600 seconds: Scale-out in progress
├─ New instance: Starts booting
├─ User data: Begins running
├─ Original: Still at stress
└─ Combined: CPU starting to reduce

T=600+ seconds: Multiple effects
├─ New instance: Online and taking load
├─ Load distribution: Spreading across 2 instances
├─ CPU per instance: Starting to drop
└─ Target: Moving toward 40%

Expected Scaling Behavior:

During Stress Test:

Desired Instances:
├─ T=0: 1 instance
├─ T=300-360s: Scale triggers
├─ T=360s+: Desired = 2
├─ T=400s+: Perhaps desired = 3
└─ Final: 3 instances (max specified)

Each Scale Event:

- Launch time: 30-90 seconds
- User data: 30-180 seconds
- Ready: 2-5 minutes
- Distribution: Load spreads

CPU Trend:

When Observe:
├─ Before: 100% on 1 instance
├─ During scaling: Slightly down as 2nd starts
├─ After 2nd ready: Down to ~50% (2 instances handle same load)
├─ Further: May scale to 3 if still >40%
└─ Stabilize: Usually 2-3 instances for full stress

Stopping Stress Test:

When Timeout Ends:

After 10 Minutes:
```
stress: info: successful run completed in 600s
```

Shell prompt returns:
├─ Command: Complete
├─ Stress: No longer running
├─ Impact: CPU back to normal
└─ Result: Can observe scale-in

Alternative: Stop Early

Early Termination:
```bash
Ctrl+C  # Press Ctrl and C together
```

Effect:
├─ Immediately: Stress stops
├─ CPU: Drops back to low
├─ Scale-in: Will trigger after cooldown
└─ Typical: Let timer finish (cleaner)

Observing Scale-In:

After Stress Ends:

Timeline:

T=600s (30 min mark): Stress ends
├─ CPU: Drops immediately
├─ Metric: Falls rapidly
└─ CloudWatch: Reports low value

T=600-660s: Metrics update
├─ Period: Waits for metric update
├─ Measurement: Check if < 28%
└─ Cooldown: Checking prevents thrashing

T=660-720s: Scale-in triggers
├─ Alarm: AlarmLow activates
├─ Action: Begin scale-in
├─ Heartbeat: Activity shows "Terminating 1 instance"
└─ Selection: ASG chooses instance to terminate

T=720-1200s: Termination completes
├─ Instance: Graceful shutdown
├─ Deregistration: From ALB (if used)
├─ Final: Instance removed
└─ Result: Back to 1 instance

Activity History During Scale-In:

Entry 1 (Scale-Out - Initial):
```
Timestamp: Time (during stress)
Status: Successful
Message: "Launching 1 instance"
Reason: "Target Tracking Scaling policy high-cpu-target"
Result: Desired capacity changed from 1 to 2
```

Entry 2 (Additional scale-out if CPU still high):
```
Timestamp: Later (stress continued)
Status: Successful
Message: "Launching 1 instance"
Reason: "Target Tracking Scaling policy high-cpu-target"
Result: Desired capacity changed from 2 to 3
```

Entry 3 (Scale-In - After stress ends):
```
Timestamp: After stress stops
Status: Successful
Message: "Terminating 1 instance"
Reason: "Target Tracking Scaling policy high-cpu-target"
Result: Desired capacity changed from 3 to 2
```

Entry 4 (Additional scale-in):
```
Timestamp: Later (cooldown expired, CPU still low)
Status: Successful
Message: "Terminating 1 instance"
Reason: "Target Tracking Scaling policy high-cpu-target"
Result: Desired capacity changed from 2 to 1
```

Viewing CloudWatch Alarms:

Access Alarms:

Service: CloudWatch
├─ Right panel: Alarms
├─ Filter: Your ASG alarms
└─ Find: AlarmHigh and AlarmLow

Alarm Details:

Click: AlarmHigh
├─ Metric: ASGAverageCPUUtilization
├─ Threshold: 40%
├─ Evaluation periods: Multiple data points
├─ Data points required: Typically 3
└─ Duration: ~3 minutes before alarm state

Graph showing:
├─ Normal: CPU low (below 40%)
└─ Alarm trigger: CPU spikes (>40% for duration)

Alarm Actions:
├─ During stress: State = ALARM
├─ Alarm ends: Sends scaling action
├─ After stress: State = OK
└─ Cool down: Prevents immediate re-trigger

Understanding Alarm Thresholds:

Threshold Values:

AlarmHigh:
├─ What: CPU >= 40% AND
├─ Duration: 3 consecutive data points
├─ Time: 3 minutes (if 1-minute metrics)
└─ Meaning: Sustained high CPU (not just spike)

AlarmLow:
├─ What: CPU < 28% AND
├─ Duration: 15 consecutive data points
├─ Time: 15 minutes (if 1-minute metrics)
└─ Meaning: Sustained low CPU (conservative)

Why Conservative Scale-In?

Stability:
├─ Reason: Prevent unnecessary cycling
├─ Benefit: Don't remove if temporary high
├─ Effect: Scales in slower than out
└─ Result: Lower risk of under-capacity

Real-World Impact:

During Stress:
├─ Immediately: Scales out (fast response)
└─ Time: Seconds when CPU exceeds 40%

After Stress:
├─ Immediately: CPU drops rapidly
├─ But: Won't scale in for 15 minutes
├─ Reason: Wait to ensure sustained low
└─ Result: Extra instances kept briefly

Comprehensive Timeline Example:

Complete Demonstration:

T=0:00: Start stress on instance 1
├─ Command: stress -c 4 --timeout 600s
└─ Status: Running

T=0:00-3:00: CPU rises
├─ Metric: 0% → 50% → 100%
└─ CloudWatch: Tracking increase

T=3:00-4:00: First scale trigger
├─ Alarm: AlarmHigh activated
├─ Action: Launching 1 instance (instance 2)
└─ Message: "Launching 1 instance" in Activity

T=4:00-6:00: Instance 2 boots
├─ State: Running
├─ Health: Initializing
└─ CPU: Still 100% on instance 1

T=6:00-6:30: Instance 2 full ready
├─ Status: Healthy
├─ Load: Starting to take requests
├─ CPU: Now ~50% per instance (2 total capable)
└─ Target: 40% but still 50%, so still high

T=6:30-7:00: Evaluation continues
├─ CPU: Still ~50% average
├─ Status: Above 40% target
├─ Decision: May scale again
└─ Action: Depends on exact value

T=7:00-7:30: Possible second scale
├─ If CPU still high: Launch instance 3
├─ Desired: Changed from 2 to 3
├─ New launch: Instance 3 starting
└─ Distribution: 3 instances now

T=7:30-9:30: Instance 3 boots
├─ State: Running
├─ Health: Initializing
├─ CPU: Distributed across 3 instances (improves)
└─ Total: Load now 1/3 per instance

T=10:00: Stress ends
├─ Command: Timeout (600 seconds)
└─ CPU: Drops to ~5%

T=10:00-11:00: CPU low but instances remain
├─ CPU: Very low (<5%)
├─ Instances: Still 3 running
├─ Reason: Waiting for scale-in cooldown
└─ Alarm: AlarmLow not yet triggered

T=11:00-26:00: Scale-in waiting
├─ Duration: ~15 minutes for AlarmLow
├─ During: Continuous low CPU reading
├─ Purpose: Ensure sustained low, not just dip
└─ Cost: Paying for extra instances during this

T=26:00: Scale-in trigger
├─ Alarm: AlarmLow activated (finally!)
├─ Action: Terminating 1 instance (instance 3)
├─ Message: "Terminating 1 instance" in Activity
└─ Desired: Changed from 3 to 2

T=26:30-31:00: Instance 3 terminates
├─ State: Terminating to Terminated
├─ Deregistration: From ALB grace period
└─ Result: Instance removed

T=31:00-46:00: Wait for next scale-in
├─ Duration: Another ~15 minutes
├─ Instances: 2 running
├─ CPU: Very low
└─ Waiting: Next AlarmLow trigger

T=46:00: Second scale-in
├─ Alarm: AlarmLow triggered again
├─ Action: Terminating final extra instance
├─ Desired: Changed from 2 to 1
└─ Message: "Terminating 1 instance"

T=46:30-51:00: Last extra instance terminates
├─ State: Terminating
├─ Final: Back to 1 instance
└─ Result: Desired capacity = Min capacity = 1

End Result:

Final State:
├─ Instances: 1 (returned to minimum)
├─ CPU: Low and normal
├─ Capacity: At baseline
└─ Cycle: Complete

Observations:

Fast scale-out:
├─ Response: ~3-4 minutes
├─ Reason: Immediate when threshold exceeded
└─ Benefit: Quick response to traffic spikes

Slow scale-in:
├─ Response: ~25-30 minutes total
├─ Reason: Wait 15 min + terminate 5 min + wait 15 min again
└─ Benefit: Conservative, prevents waste

Total cycle:
├─ Full test: ~1 hour total
├─ Useful: Demonstrates policy in action
└─ Scale-up fast: Scale-down slow (normal)

Conclusion from Testing:

What We Learned:

✓ Static policy works automatically
✓ CPU threshold triggers correctly
✓ Scaling happens in real-time
✓ Multiple instances launched as needed
✓ Load distribution works
✓ Scale-in conservative (safe)
✓ Activity history tracks everything
✓ CloudWatch shows metrics visually

What We Observed:

✓ Target tracking automates scaling
✓ Alarms created in background
✓ Desired capacity changes based on metrics
✓ Instances launched/terminated as needed
✓ Cooldown prevents thrashing
✓ Performance improved with more instances
✓ Cost savings from scale-in
✓ Complete end-to-end automation

Next Steps:

After Testing:

Clean Up:
├─ Delete: Scaling policies (if done testing)
├─ Terminate: Extra instances (if scaling left them)
├─ Verify: Back to minimal state
└─ Cost: Stop unnecessary charges

Or Keep For Monitoring:
├─ Leave: Policies active
├─ Monitor: Real production load
├─ Observe: How policy behaves with real traffic
└─ Adjust: Target or thresholds if needed

Production Deployment:

Ready for:
├─ Real application: Deploy with confidence
├─ Auto scaling: Works as expected
├─ Monitoring: Understood behavior
├─ Support: Ready to troubleshoot
└─ Team: Can explain to others
```

---

## Part 5: Cleanup and Best Practices

### Removing Test Policies and Production Guidelines

**Post-Demonstration Procedures**

```
Cleanup Steps:

After Demonstration:

Remove Scaling Policies:

For Testing Purposes:
├─ If temporary: Delete scaling policy
├─ Access: ASG → Automatic scaling tab
├─ Find: Your target tracking policy
├─ Action: Delete or disable
└─ Result: No more automatic scaling

Cleanup Process:

Step 1: View Policies
├─ Open: ASG details
├─ Tab: "Automatic scaling"
├─ List: "Scaling policies"
└─ Find: Policy to remove

Step 2: Delete Policy
├─ Select: Policy name
├─ Click: Delete (or Remove)
├─ Confirm: Are you sure?
└─ Result: Policy removed

Step 3: Verify Removal
├─ Check: List now empty (or fewer policies)
├─ Effect: No more automatic scaling
├─ Manual: Can still scale manually
└─ Verify: Policies gone

Terminate Extra Instances:

If Scaling Left Instances:

Situation:
├─ Current: 3 instances running
├─ Desired: Back to 1
├─ Need: Reduce capacity
└─ Method: Edit ASG

Steps:

1. Edit ASG
   ├─ Open: ASG details
   ├─ Button: Edit
   └─ Form: Open for editing

2. Reduce Capacity
   ├─ Min: 1
   ├─ Max: 1
   ├─ Desired: 1
   └─ Save: Apply changes

3. Termination
   ├─ Effect: Excess instances terminate
   ├─ Duration: 5-10 minutes
   ├─ Result: Back to 1 instance
   └─ Verify: Check EC2 Instances tab

CloudWatch Alarms:

Remove Alarms:

Location:
├─ Service: CloudWatch
├─ Section: Alarms
├─ Find: AlarmHigh and AlarmLow (created by policy)
└─ Note: Auto-deleted when policy deleted (usually)

Manual Deletion (if needed):
├─ Select: Each alarm
├─ Action: Delete
├─ Confirm: Remove
└─ Result: Alarm removed

Production Best Practices:

Planning:

1. Understand Your Workload:
   ├─ Map: Traffic patterns
   ├─ Identify: Bottleneck (CPU, network, connections)
   ├─ Measure: Performance metrics
   └─ Result: Informed metric selection

2. Choose Right Metric:
   ├─ Test: Load test to identify limit
   ├─ Measure: Which metric hits limit first
   ├─ Select: That metric for scaling
   └─ Verify: Metric correlates to performance

3. Set Realistic Target:
   ├─ Conservative: Start higher (70%)
   ├─ Monitor: Behavior for days/weeks
   ├─ Lower: Gradually as confident
   └─ Balance: Cost vs performance

Implementation:

4. Start with Scheduled + Target Tracking:
   ├─ Scheduled: For known patterns
   ├─ Target tracking: For surprises
   ├─ Together: Most common approach
   └─ Coverage: Both predictable and spikes

5. Test Before Production:
   ├─ Staging: Test policies first
   ├─ Verify: Behavior as expected
   ├─ Adjust: If needed
   └─ Confidence: Before prod

6. Monitor Continuously:
   ├─ Dashboard: Watch key metrics
   ├─ Alarms: Alert on anomalies
   ├─ Logs: Review activities regularly
   └─ Trend: Monitor over time

Maintenance:

7. Document Decisions:
   ├─ Record: Why each policy chosen
   ├─ Metric: Why this metric
   ├─ Target: Why this value
   ├─ Cooldown: Why adjusted if changed
   └─ Benefit: Future troubleshooting

8. Review Regularly:
   ├─ Monthly: Check if policies still valid
   ├─ Quarterly: Review costs and performance
   ├─ Yearly: Full assessment
   └─ Adjust: If workload pattern changed

9. Prepare for Failures:
   ├─ Max capacity: Set cost limit
   ├─ Min capacity: Ensure HA
   ├─ Grace period: Appropriate for app
   └─ Monitoring: Alert on both scale-out and scale-in

Common Production Scenarios:

Scenario 1: Critical Hours Protection

Setup:
├─ Scheduled: Morning scale-up early
├─ Dynamic: Target tracking 50%
├─ Combined: Never capacity starved
└─ Result: Always ready for traffic

Scenario 2: Cost Optimization

Setup:
├─ Scheduled: Scale down after hours
├─ Dynamic: Maintain efficiency
├─ Combined: Right-sized capacity
└─ Result: Lower costs

Scenario 3: Event Response

Setup:
├─ Scheduled: Pre-scale for known event
├─ Dynamic: Handle unexpected beyond forecast
├─ Max: Set to budget limit
└─ Result: Handle surge, controlled cost

Scenario 4: Development/Testing

Setup:
├─ Minimal: Single instance (cost)
├─ Manual: Scale as needed for testing
├─ No policies: Keep simple
└─ Result: Full control, low cost

Key Takeaways:

What We Accomplished:

✓ Understood: Four scaling policy types
✓ Created: Target tracking policy
✓ Observed: Real-time scaling
✓ Monitored: Activity history
✓ Learned: CloudWatch alarms integration
✓ Tested: Scaling up during stress
✓ Tested: Scaling down after stress
✓ Cleaned up: Removed test infrastructure

Ready For:

✓ Production deployment
✓ Real workload scaling
✓ Monitoring and troubleshooting
✓ Cost optimization
✓ Performance assurance
✓ Team leadership and explanation

Next Topics:

Future Learning:
├─ ASG with multiple regions
├─ Cross-region failover
├─ Advanced monitoring
├─ Cost optimization
├─ Multi-tier applications
└─ Real-world deployment patterns
```

---

## Conclusion

Through this hands-on demonstration, you've seen Auto Scaling Groups in action:
- Created and observed a target tracking scaling policy
- Stressed CPU to trigger scale-out
- Allowed stress to end and observed scale-in
- Understood CloudWatch alarm integration
- Verified activity history tracking

This practical experience bridges the gap between understanding policies theoretically and seeing them work in real time. You're now ready to implement automatic scaling in production environments!
