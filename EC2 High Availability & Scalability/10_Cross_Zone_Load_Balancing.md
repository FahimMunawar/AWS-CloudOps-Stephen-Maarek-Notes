# Cross-Zone Load Balancing: Traffic Distribution Across Availability Zones

Cross-zone load balancing distributes incoming traffic evenly across all registered instances in all availability zones, rather than limiting distribution to instances within the same AZ, with different default behaviors and cost implications across load balancer types.

---

## Part 1: Understanding Cross-Zone Load Balancing

### What is Cross-Zone Load Balancing?

**Definition**
```
Cross-Zone Load Balancing:
├─ Feature that distributes traffic across AZs
├─ Without it: Traffic limited to instances in same AZ
├─ With it: Traffic distributed to all instances in all AZs
├─ Implementation: Automatic at load balancer level
└─ Cost implications: Varies by load balancer type
```

**Key Concept**

```
Load Balancer Architecture:

Typical Setup:
├─ Availability Zone 1 (us-east-1a)
│  └─ Load Balancer Node + EC2 instances
│
├─ Availability Zone 2 (us-east-1b)
│  └─ Load Balancer Node + EC2 instances
│
└─ Availability Zone 3 (us-east-1c)
   └─ Load Balancer Node + EC2 instances

Each AZ has:
├─ Its own load balancer node (instance)
├─ Its own EC2 backend instances
└─ Network isolated from other AZs

Question:
├─ When load balancer node in AZ-1 receives traffic
├─ Should it only send to AZ-1 instances?
├─ Or to instances in AZ-1, AZ-2, AZ-3?
└─ Cross-zone balancing answers this
```

---

## Part 2: Cross-Zone Load Balancing: Enabled vs Disabled

### Scenario: Imbalanced Instance Distribution

**Setup**
```
Availability Zone 1 (us-east-1a):
├─ Load Balancer Node: LB-1
├─ EC2 Instances:
│  ├─ Instance-1
│  └─ Instance-2
└─ Total: 2 instances

Availability Zone 2 (us-east-1b):
├─ Load Balancer Node: LB-2
├─ EC2 Instances:
│  ├─ Instance-3
│  ├─ Instance-4
│  ├─ Instance-5
│  ├─ Instance-6
│  ├─ Instance-7
│  ├─ Instance-8
│  ├─ Instance-9
│  ├─ Instance-10
│  └─ Instance-11 (added for 9 total)
└─ Total: 8+ instances

Total across all AZs: 10 instances
```

### Scenario: WITH Cross-Zone Load Balancing ENABLED

**Traffic Distribution Pattern**

```
Client makes requests:
└─ Sends 50% traffic to LB-1 (AZ-1 node)
└─ Sends 50% traffic to LB-2 (AZ-2 node)

LB-1 (AZ-1 Node) Processing:
├─ Receives: 50% of total traffic
├─ Must distribute to: ALL 10 instances (all AZs)
├─ Distribution: 50% / 10 = 5% to each instance
└─ Result:
   ├─ AZ-1 Instance-1: 5%
   ├─ AZ-1 Instance-2: 5%
   ├─ AZ-2 Instance-3: 5%
   ├─ AZ-2 Instance-4: 5%
   ├─ ... (continues for all 10)
   └─ AZ-2 Instance-11: 5%

LB-2 (AZ-2 Node) Processing:
├─ Receives: 50% of total traffic
├─ Must distribute to: ALL 10 instances (all AZs)
├─ Distribution: 50% / 10 = 5% to each instance
└─ Result: Same 5% to each instance

FINAL Traffic Distribution (With Cross-Zone):
├─ Instance-1 (AZ-1): 5% + 5% = 10% total ✓
├─ Instance-2 (AZ-1): 5% + 5% = 10% total ✓
├─ Instance-3 (AZ-2): 5% + 5% = 10% total ✓
├─ Instance-4 (AZ-2): 5% + 5% = 10% total ✓
├─ ... (continues)
└─ ALL instances: 10% each (perfectly balanced)

Key Point:
├─ ALL instances treated equally
├─ Regardless of which AZ they're in
├─ Perfect load distribution
└─ May send traffic across AZs
```

**Diagram: Cross-Zone ENABLED**

```
┌──────────────────────────────────────────────────┐
│ Internet Clients                                 │
└──────────────────────────┬───────────────────────┘
                           │
                     50%   │   50%
                     ┌─────┴─────┐
                     ▼           ▼
        ┌─────────────────┐   ┌────────────────┐
        │  LB-1 (AZ-1)    │   │ LB-2 (AZ-2)    │
        └────────┬────────┘   └────────┬───────┘
                 │                     │
        ┌────────┴─────────────────────┴────────┐
        │ Distributes to ALL 10 instances       │
        │ (across both AZs)                     │
        ▼
    [Each receives 5% from each LB node]
    [Total: 10% from both LB nodes combined]
        │
    ┌───┴────────────────────────────────────────┐
    │                                            │
    ▼ AZ-1 (2 instances)     ▼ AZ-2 (8 instances)
┌─────────┐ ┌──────────┐  ┌──┴──┐ ... ┌─────────┐
│ Inst-1  │ │ Inst-2   │  │Inst3│     │ Inst-11 │
│  10%    │ │   10%    │  │10%  │     │  10%    │
└─────────┘ └──────────┘  └─────┘ ... └─────────┘

Result: Perfect distribution, each instance 10%
Cost: Inter-AZ data transfer charges (some LBs)
```

### Scenario: WITHOUT Cross-Zone Load Balancing DISABLED

**Traffic Distribution Pattern**

```
Client makes requests:
└─ Sends 50% traffic to LB-1 (AZ-1 node)
└─ Sends 50% traffic to LB-2 (AZ-2 node)

LB-1 (AZ-1 Node) Processing:
├─ Receives: 50% of total traffic
├─ Must distribute to: ONLY AZ-1 instances (2)
├─ Distribution: 50% / 2 = 25% to each instance
└─ Result:
   ├─ AZ-1 Instance-1: 25%
   └─ AZ-1 Instance-2: 25%
   └─ (AZ-2 instances: 0% from LB-1)

LB-2 (AZ-2 Node) Processing:
├─ Receives: 50% of total traffic
├─ Must distribute to: ONLY AZ-2 instances (8)
├─ Distribution: 50% / 8 = 6.25% to each instance
└─ Result:
   ├─ AZ-2 Instance-3: 6.25%
   ├─ AZ-2 Instance-4: 6.25%
   ├─ ... (continues)
   └─ AZ-2 Instance-11: 6.25%
   └─ (AZ-1 instances: 0% from LB-2)

FINAL Traffic Distribution (Without Cross-Zone):
├─ Instance-1 (AZ-1): 25% total ✓ (HIGH!)
├─ Instance-2 (AZ-1): 25% total ✓ (HIGH!)
├─ Instance-3 (AZ-2): 6.25% total ✗ (LOW!)
├─ Instance-4 (AZ-2): 6.25% total ✗ (LOW!)
├─ ... (continues)
└─ Instance-11 (AZ-2): 6.25% total ✗ (LOW!)

Key Point:
├─ AZ-1 instances: 4x MORE traffic (25% vs 6.25%)
├─ Load IMBALANCED by AZ
├─ Doesn't use cross-AZ traffic
├─ Respect AZ boundaries
```

**Diagram: Cross-Zone DISABLED**

```
┌──────────────────────────────────────────────────┐
│ Internet Clients                                 │
└──────────────────────────┬───────────────────────┘
                           │
                     50%   │   50%
                     ┌─────┴─────┐
                     ▼           ▼
        ┌─────────────────┐   ┌────────────────┐
        │  LB-1 (AZ-1)    │   │ LB-2 (AZ-2)    │
        └────────┬────────┘   └────────┬───────┘
                 │                     │
        ┌────────┴──────────┐  ┌──────┴────────┐
        │ Distributes       │  │ Distributes   │
        │ ONLY to AZ-1      │  │ ONLY to AZ-2  │
        │ (2 instances)     │  │ (8 instances) │
        ▼                   ▼  ▼               ▼
    ┌─────────┐ ┌──────────┐  ┌──┴──┐ ... ┌─────────┐
    │ Inst-1  │ │ Inst-2   │  │Inst3│     │ Inst-11 │
    │  25%    │ │  25%     │  │6.25%│     │ 6.25%   │
    └─────────┘ └──────────┘  └─────┘ ... └─────────┘

Result: IMBALANCED, AZ-1 gets 4x more traffic than AZ-2
Cost: NO inter-AZ charges (stays in AZ boundary)
```

### Comparison: With vs Without

**Side-by-Side Comparison**

```
Aspect                  WITH Cross-Zone    WITHOUT Cross-Zone
────────────────────────────────────────────────────────────
Traffic per instance    10% (balanced)      25% (AZ-1)
                                            6.25% (AZ-2)

Load distribution       Perfect             IMBALANCED

Instance utilization    Even across AZs     Uneven across AZs

Cross-AZ traffic        YES (flows across)  NO (stays in AZ)

Data transfer charges   YES (inter-AZ)      NO (within AZ)

CPU utilization         Even                Uneven

Memory usage            Even                Uneven

Scaling efficiency      Better              Worse

High availability       Better              Worse

Cost (inter-AZ)         Higher              Lower

Network performance     Slightly higher     Slightly lower
                        latency             latency (fewer hops)

Use case                Most production     Special cases
                        applications

Best for:               General purpose     Cost-sensitive
                                            or low-traffic
```

---

## Part 3: Defaults by Load Balancer Type

### Application Load Balancer (ALB)

**Default Behavior**

```
ALB Cross-Zone Load Balancing:

Default Setting: ENABLED by default ✓

Characteristics:
├─ Enabled automatically
├─ Can be disabled if desired
├─ Applies to all target groups
├─ OR can be configured per target group
└─ No additional charges for inter-AZ data

Implication:
├─ Traffic distributed evenly across all AZs
├─ Perfect load distribution by default
├─ Additional inter-AZ traffic has no cost
└─ User doesn't need to do anything
```

**Configuration for ALB**

```
Two Configuration Levels:

1. Load Balancer Level (Affects all target groups):
   ├─ ALB Attributes
   ├─ "Cross-zone load balancing"
   ├─ Default: Enabled
   ├─ Can toggle: On/Off
   └─ Applies globally to all traffic

2. Target Group Level (Per target group override):
   ├─ Target Group Attributes
   ├─ "Cross-zone load balancing"
   ├─ Options:
   │  ├─ Inherit from load balancer
   │  ├─ Force ON (override LB setting)
   │  └─ Force OFF (override LB setting)
   └─ Can have different settings per target group
```

**Why ALB Enables by Default**

```
Reasons for Default Enable:

1. ALB is designed for web/HTTP traffic
   ├─ Web applications benefit from even load
   ├─ Global users distributed across regions
   └─ Cross-AZ fair to all instances

2. No cost penalty for ALB
   ├─ AWS doesn't charge ALB for inter-AZ data
   ├─ Fair to enable by default
   ├─ User gets perfect load distribution free
   └─ No cost incentive to disable

3. Simplicity
   ├─ Most users want fair distribution
   ├─ Default enables optimal behavior
   ├─ No configuration needed for most
   └─ Good user experience
```

### Network Load Balancer (NLB)

**Default Behavior**

```
NLB Cross-Zone Load Balancing:

Default Setting: DISABLED by default ✗

Characteristics:
├─ Disabled automatically
├─ Can be enabled if desired
├─ Must explicitly enable if needed
├─ Additional charges when enabled
└─ Data transfer between AZs costs money

Implication:
├─ Traffic stays within each AZ
├─ May result in load imbalance
├─ Additional inter-AZ traffic incurs charges
└─ User must enable if cross-AZ needed
```

**Why NLB Disabled by Default**

```
Reasons for Default Disable:

1. NLB is performance-focused
   ├─ Extreme throughput/latency sensitive
   ├─ Cross-AZ traffic adds latency
   ├─ Users optimize for performance
   └─ Disable by default respects performance goals

2. Cost considerations
   ├─ NLB charges for inter-AZ data transfer
   ├─ AWS passes cost to user
   ├─ User should be aware and intentional
   ├─ Disabled by default = no surprise charges
   └─ User can enable if they accept cost

3. Use case mismatch
   ├─ NLB often used for extreme scenarios
   ├─ Gaming, finance, big data
   ├─ These care about AZ locality
   ├─ Cross-AZ may not be desired
   └─ Default disable respects this
```

**Configuration for NLB**

```
NLB Attributes Configuration:

Location:
├─ Load Balancer Attributes
├─ NOT at target group level
└─ Single setting for entire NLB

Setting:
├─ Cross-zone load balancing
├─ Default: Disabled
├─ Toggling: Can enable/disable
└─ Cost: Incurs charges when enabled

Enable Process:
1. NLB Details page
2. Attributes section
3. Edit: Cross-zone load balancing
4. Toggle: ON
5. Message: "This may include regional charges"
6. Save
```

### Gateway Load Balancer (GWLB)

**Default Behavior**

```
GWLB Cross-Zone Load Balancing:

Default Setting: DISABLED by default ✗

Characteristics:
├─ Disabled automatically
├─ Can be enabled if desired
├─ Must explicitly enable if needed
├─ Additional charges when enabled
└─ Similar to NLB behavior

Implication:
├─ Traffic stays within each AZ
├─ Security appliances organized by AZ
├─ Enables AZ-local filtering
├─ Additional inter-AZ traffic incurs charges
```

**Why GWLB Disabled by Default**

```
Reasons for Default Disable:

1. Security appliance locality
   ├─ Firewalls often regional/local
   ├─ Compliance: Keep data within AZ
   ├─ Local security policies
   └─ Cross-AZ might violate policies

2. Cost considerations (like NLB)
   ├─ Charges for inter-AZ traffic
   ├─ Disabled by default = no surprises
   ├─ User decides if needed

3. Network appliance design
   ├─ Many appliances AZ-aware
   ├─ Built for locality
   ├─ Cross-AZ against design
   └─ Default disable respects design
```

**Configuration for GWLB**

```
GWLB Attributes Configuration:

Location:
├─ Load Balancer Attributes
├─ NOT at target group level
└─ Single setting for entire GWLB

Setting:
├─ Cross-zone load balancing
├─ Default: Disabled
├─ Toggling: Can enable/disable
└─ Cost: Incurs charges when enabled

Enable Process:
1. GWLB Details page
2. Attributes section
3. Edit: Cross-zone load balancing
4. Toggle: ON
5. Message: "Data charges may apply"
6. Save
```

### Classic Load Balancer (CLB)

**Default Behavior**

```
CLB Cross-Zone Load Balancing:

Default Setting: DISABLED by default ✗

Characteristics:
├─ Disabled automatically
├─ Can be enabled if desired
├─ Legacy load balancer (deprecated)
├─ If enabled: NO charges for inter-AZ data
└─ DEPRECATED - Not recommended for new projects

Implication:
├─ Traffic stays within each AZ
├─ May result in load imbalance
├─ No inter-AZ charges if enabled
├─ But: Full CLB deprecated anyway

Why Different from NLB/GWLB?
├─ CLB is legacy (pre-ALB/NLB)
├─ No inter-AZ charges (older pricing model)
├─ Rarely enabled or used
└─ Not relevant for modern AWS

Exam Note:
├─ Unlikely to appear on exam
├─ Focus on ALB/NLB instead
├─ CLB deprecated by AWS
└─ New projects: Don't use CLB
```

---

## Part 4: Cost Implications

### Data Transfer Charges

**Inter-Availability Zone Data Transfer**

```
Cost Model:

Charge Applied To:
├─ Data flowing between different AZs
├─ Usually: $0.01 per GB (varies by region)
├─ Applies in both directions
└─ Downstream bandwidth cost

When Charged:

ALB with Cross-Zone ENABLED:
├─ Charge: NONE ✓
├─ AWS absorbs cost
├─ No inter-AZ charges for ALB
└─ Free to enable

NLB with Cross-Zone ENABLED:
├─ Charge: YES ✗
├─ AWS charges: $0.01/GB (typical)
├─ Add up quickly in high-traffic
└─ Must enable intentionally

GWLB with Cross-Zone ENABLED:
├─ Charge: YES ✗
├─ Same pricing as NLB
├─ Data transfer between AZs costs

CLB with Cross-Zone ENABLED:
├─ Charge: NONE ✓
├─ Even older pricing: free inter-AZ
└─ But CLB deprecated anyway
```

**Cost Calculation Example**

```
Scenario: NLB with Cross-Zone ENABLED

High-Traffic Application:
├─ Traffic: 100 TB/month cross-AZ
├─ Pricing: $0.01 per GB
├─ Calculation: 100 TB = 100,000 GB
│              100,000 GB × $0.01 = $1,000/month
├─ Result: $1,000/month additional cost!
└─ Over 1 year: $12,000!

If Cross-Zone DISABLED:
├─ Cost: $0 (no cross-AZ traffic)
├─ Tradeoff: Load imbalance
└─ Savings: $12,000/year

Decision Point:
├─ Is $12,000/year worth even load distribution?
├─ Or accept imbalance to save money?
└─ Depends on application
```

**Cost Comparison Table**

```
Load Balancer    Default    Can Change?   Cost if Enabled
─────────────────────────────────────────────────────────
ALB              Enabled    Yes           FREE ✓
NLB              Disabled   Yes           $$ (charges)
GWLB             Disabled   Yes           $$ (charges)
CLB              Disabled   Yes           FREE ✓

Recommendation:
├─ Use ALB by default (enabled free)
├─ Use NLB if cost-sensitive (keep disabled)
├─ Enable NLB cross-zone only if needed
└─ Evaluate cost vs. load distribution tradeoff
```

---

## Part 5: When to Use Each Option

### When to Enable Cross-Zone Load Balancing

**Enable Cross-Zone When**

```
✓ Using ALB (already enabled, stay enabled)
  └─ No cost penalty, perfect load distribution

✓ Unbalanced AZ instance distribution
  ├─ Need even load across all instances
  ├─ Can't perfectly balance instances per AZ
  └─ Accept inter-AZ cost for fairness

✓ High availability critical
  ├─ One AZ fails → traffic redirected
  ├─ Other AZ instances take all load
  ├─ Need them to handle it gracefully
  └─ Cross-zone helps here

✓ Geographic redundancy
  ├─ Application must survive full AZ loss
  ├─ Instances ready in other AZs
  ├─ Cross-zone ensures they're used
  └─ Important for SLA

✓ Can absorb cost
  └─ Calculate inter-AZ traffic cost
  └─ Include in budget
  └─ Acceptable for application

✓ Global audience
  ├─ Users throughout region
  ├─ No AZ affinity needed
  └─ Even distribution best
```

### When to Disable Cross-Zone Load Balancing

**Disable Cross-Zone When**

```
✓ Cost-sensitive application
  ├─ High traffic volume
  ├─ Inter-AZ charges add up
  ├─ Budget constraints
  └─ Accept load imbalance for cost savings

✓ Low traffic
  ├─ Running 3-5 instances
  ├─ Even distribution not critical
  ├─ Cost savings minimal anyway
  └─ AZ locality preferred

✓ Single instance per AZ
  ├─ Already balanced if 1 per AZ
  ├─ No need for cross-zone
  ├─ Keep traffic local
  └─ Better latency

✓ Compliance/locality requirement
  ├─ Data residency rules
  ├─ Must stay in specific AZ
  ├─ PII data localization
  └─ Legal requirement

✓ Extreme performance focus
  ├─ Latency-sensitive application
  ├─ Cross-AZ adds hops/latency
  ├─ NLB for gaming/finance
  └─ Disable for millisecond optimization

✓ Application designed for AZ locality
  ├─ Architecture assumes local instances
  ├─ Session storage in AZ-specific storage
  ├─ Failover within AZ only
  └─ Not designed for cross-AZ
```

### Decision Tree

```
Do you have ALB?
├─ YES → Keep cross-zone ENABLED (it's free!)
└─ NO → Continue below

Do you have unbalanced instance distribution?
├─ YES & can afford cost → Enable cross-zone
├─ YES & cost-conscious → Keep disabled, accept imbalance
└─ NO → Continue below

Do you have compliance/locality requirements?
├─ YES → Disable (data must stay in AZ)
└─ NO → Continue below

Is cost a major concern?
├─ YES → Disable (save inter-AZ charges)
└─ NO → Enable for better distribution

Do you need cross-AZ high availability?
├─ YES → Enable (ensure AZ failure survivable)
└─ NO → Disable (unnecessary)

Default Decision:
├─ ALB: Keep enabled (already enabled)
├─ NLB/GWLB: Disable (cost-sensitive)
│  └─ Enable only if needed
└─ CLB: Don't use (deprecated)
```

---

## Part 6: Configuration Examples

### Configuring ALB Cross-Zone

**Enable/Disable at Load Balancer Level**

```
Location:
1. AWS Console → EC2 → Load Balancers
2. Select: Your ALB
3. Tab: Attributes

Configuration:
├─ Find: Cross-zone load balancing
├─ Toggle: ON (enabled) or OFF (disabled)
└─ Click: Save

Effect:
├─ Applies to ALL traffic
├─ All target groups affected
├─ Unless overridden per target group
```

**Override at Target Group Level**

```
Location:
1. AWS Console → EC2 → Target Groups
2. Select: Your target group
3. Tab: Attributes

Configuration:
├─ Find: Cross-zone load balancing
├─ Options:
│  ├─ Inherit from load balancer (default)
│  ├─ Force ON (always enabled)
│  └─ Force OFF (always disabled)
└─ Click: Save

Use Case:
├─ One target group: Same-AZ instances
├─ Another target group: Cross-AZ instances
├─ Different settings per target group
```

### Configuring NLB Cross-Zone

**Enable/Disable**

```
Location:
1. AWS Console → EC2 → Load Balancers
2. Select: Your NLB
3. Tab: Attributes

Configuration:
├─ Find: Cross-zone load balancing
├─ Default: OFF
├─ Toggle: ON to enable
├─ Warning: "This may include regional charges"
└─ Click: Save

Cost Warning:
├─ AWS displays cost warning
├─ Indicates inter-AZ charges
├─ Must acknowledge before enabling
└─ Helps prevent accidental costs
```

### Configuring GWLB Cross-Zone

**Enable/Disable**

```
Location:
1. AWS Console → EC2 → Load Balancers
2. Select: Your GWLB
3. Tab: Attributes

Configuration:
├─ Find: Cross-zone load balancing
├─ Default: OFF
├─ Toggle: ON to enable
├─ Warning: "Data charges may apply"
└─ Click: Save

Similar to NLB:
├─ Disabled by default
├─ Cost warning displayed
├─ Inter-AZ charges incurred
└─ Must intentionally enable
```

---

## Part 7: Key Takeaways for SysOps Associate

### Cross-Zone Load Balancing Essentials

**What to Remember**

```
1. Definition:
   └─ Distributes traffic across instances in all AZs
   └─ Versus limited to instances in same AZ

2. Defaults by Load Balancer:
   ├─ ALB: Enabled by default (FREE)
   ├─ NLB: Disabled by default (cost concern)
   ├─ GWLB: Disabled by default (cost concern)
   └─ CLB: Disabled (deprecated anyway)

3. Cost Model:
   ├─ ALB: NO charges for cross-zone
   ├─ NLB: Charges for cross-zone ($0.01/GB typical)
   ├─ GWLB: Charges for cross-zone
   └─ CLB: NO charges (but deprecated)

4. Load Distribution Impact:
   ├─ With cross-zone: Even distribution across all instances
   ├─ Without: Traffic stays in AZ, may be unbalanced
   ├─ Example: 2 instances AZ-1, 8 instances AZ-2
   │  ├─ With: Each gets 10%
   │  └─ Without: AZ-1 each 25%, AZ-2 each 6.25%

5. Configuration:
   ├─ ALB: Can override per target group
   ├─ NLB: Single setting for entire NLB
   ├─ GWLB: Single setting for entire GWLB
   └─ CLB: Legacy (avoid)

6. High Availability:
   ├─ Cross-zone enables cross-AZ failover
   ├─ If one AZ down, others can handle load
   ├─ Important for SLA/availability

7. Performance:
   ├─ Cross-zone: Add latency (cross-AZ hops)
   ├─ Same-AZ: Lower latency
   ├─ Trade-off: Fairness vs latency

8. Compliance:
   ├─ Some regulations require data locality
   ├─ Must disable cross-zone if needed
   └─ Depends on application/industry
```

### Exam Likely Questions

**Question Types**

```
1. "Which load balancer has cross-zone enabled by default?"
   A) NLB
   B) GWLB
   C) ALB
   D) CLB
   
   Answer: C) ALB
   └─ ALB: Enabled by default
   └─ NLB/GWLB: Disabled by default
   └─ CLB: Deprecated

2. "Enabling cross-zone on NLB incurs what cost?"
   A) No cost
   B) Inter-AZ data transfer charges
   C) Direct NLB charges
   D) Bandwidth charges
   
   Answer: B) Inter-AZ data transfer charges
   └─ NLB doesn't absorb cost
   └─ AWS charges for inter-AZ data
   └─ Disabled by default to prevent surprises

3. "Why is cross-zone disabled by default for NLB?"
   A) Poor performance
   B) Cost (inter-AZ charges)
   C) Not supported
   D) Configuration complex
   
   Answer: B) Cost (inter-AZ charges)
   └─ NLB charges for cross-AZ traffic
   └─ User decides if benefit worth cost

4. "Application has 2 instances in AZ-1, 8 in AZ-2.
    What happens WITHOUT cross-zone load balancing?"
   A) Perfect 10% distribution
   B) AZ-1 gets more traffic (imbalanced)
   C) AZ-2 gets more traffic (imbalanced)
   D) Traffic rejected
   
   Answer: B) AZ-1 gets more traffic (imbalanced)
   └─ AZ-1 has fewer instances
   └─ Each AZ-1 instance: 25%
   └─ Each AZ-2 instance: 6.25%

5. "ALB with cross-zone = additional cost?"
   A) Yes, $0.01/GB
   B) Yes, $0.02/GB
   C) No, free
   D) Yes, hourly charge
   
   Answer: C) No, free
   └─ ALB doesn't charge for cross-AZ
   └─ Already included in ALB pricing
   └─ No additional inter-AZ charges

6. "How to configure different cross-zone settings
    for different target groups on same ALB?"
   A) Not possible
   B) Configure at target group level
   C) Configure at ALB level
   D) Use separate ALBs
   
   Answer: B) Configure at target group level
   └─ ALB allows per target group override
   └─ One group: cross-zone ON
   └─ Another group: cross-zone OFF

7. "GWLB cross-zone disabled. When to enable?"
   A) Always enable for high availability
   B) Enable if need cross-AZ distribution + accept costs
   C) Never enable for security
   D) Enable only for testing
   
   Answer: B) Enable if need cross-AZ distribution + accept costs
   └─ Cost-benefit decision
   └─ Default disabled (cost concern)
   └─ Enable if fairness > cost
```

### Exam Focus Points

```
Key Concepts to Memorize:

1. ALB Default: ENABLED, FREE
   └─ Most liberal, user-friendly default

2. NLB/GWLB Default: DISABLED, CHARGED if enabled
   └─ Cost-conscious defaults

3. CLB Default: DISABLED, FREE if enabled
   └─ But deprecated (not tested much)

4. Load Distribution:
   ├─ Enabled: Even across all instances
   ├─ Disabled: Even within each AZ only

5. Cost Model:
   ├─ ALB: No inter-AZ charges
   ├─ NLB: Inter-AZ charges if enabled
   ├─ GWLB: Inter-AZ charges if enabled
   └─ CLB: No inter-AZ charges

6. Configuration:
   ├─ ALB: Can be per target group
   ├─ NLB/GWLB: Single NLB/GWLB setting
```

---

## Part 8: Hands-On Summary

### What Was Demonstrated

```
1. ALB Cross-Zone Configuration:
   ├─ Location: ALB Attributes
   ├─ Default: ENABLED
   ├─ Can: Disable if desired
   └─ Message: No inter-AZ charges

2. Target Group Override:
   ├─ Location: Target Group Attributes
   ├─ Options: Inherit / Force ON / Force OFF
   └─ Effect: Different settings per target group

3. NLB Cross-Zone Configuration:
   ├─ Location: NLB Attributes
   ├─ Default: DISABLED
   ├─ Can: Enable if desired
   └─ Warning: "May include regional charges"

4. GWLB Cross-Zone Configuration:
   ├─ Location: GWLB Attributes
   ├─ Default: DISABLED
   ├─ Can: Enable if desired
   └─ Warning: "Data charges may apply"

5. Did NOT Demonstrate:
   ├─ CLB (deprecated, not practical)
   └─ Actual traffic testing (would need instances)
```

### Summary Guidance

```
Take-Away Actions:

1. For ALB in production:
   ├─ Keep cross-zone ENABLED (default)
   ├─ No cost, perfect distribution
   └─ Only disable if specific reason

2. For NLB in production:
   ├─ Keep cross-zone DISABLED (default)
   ├─ Saves inter-AZ charges
   ├─ Enable ONLY if:
   │  ├─ Unbalanced instances need fairness
   │  └─ Plus budget to absorb cost

3. For GWLB in production:
   ├─ Keep cross-zone DISABLED (default)
   ├─ Saves inter-AZ charges
   ├─ Enable ONLY if:
   │  ├─ Security appliances need cross-AZ access
   │  └─ Plus budget to absorb cost

4. When designing architecture:
   ├─ Consider cross-zone implications
   ├─ Balance instances evenly across AZs
   ├─ Enables better scaling and HA

5. For exam preparation:
   ├─ Remember ALB default = ENABLED
   ├─ Remember NLB/GWLB default = DISABLED
   ├─ Remember: ALB charges = none, NLB = inter-AZ cost
```