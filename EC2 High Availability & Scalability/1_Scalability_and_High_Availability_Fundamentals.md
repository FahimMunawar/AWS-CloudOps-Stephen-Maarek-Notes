# Scalability and High Availability Fundamentals

This note covers foundational concepts of scalability and high availability, critical for understanding AWS architecture and exam questions. These concepts are frequently confused but essential to differentiate on the SysOps exam.

## Overview

**Key Definitions**
- **Scalability**: System's ability to handle increasing load by adapting
- **High Availability**: System's ability to remain operational despite failures
- **Related but Different**: Both important for production systems but serve different purposes

**Exam Key Point**: Scalability and high availability are often confused. Understanding the distinction is crucial for exam success.

## Part 1: Scalability

### Definition of Scalability

**Scalability** = The ability of your application or system to handle a **greater load** by **adapting**.

**Two Types of Scalability**
1. **Vertical Scalability** (Scale Up/Down)
2. **Horizontal Scalability** (Scale Out/In / Elasticity)

### Vertical Scalability (Scaling Up)

#### Concept
- **Action**: Increase the size/power of existing instances
- **Direction**: Single instance grows larger
- **Hardware**: More CPU, RAM, or better processor

#### Call Center Analogy
```
Scenario: Phone operator overloaded with calls

Before (Junior Operator):
- Can handle: 5 calls per minute
- Capacity: Limited speed and skill

After Vertical Scaling (Senior Operator):
- Can handle: 10 calls per minute
- Same position, better capability
- Faster, more experienced, can handle more

Result: Capacity doubled by upgrading the individual
```

#### EC2 Example: Vertical Scaling

**Before Scaling**
```
Instance Type: t2.micro
├─ vCPU: 1
├─ Memory: 1 GB
└─ Application: Running web application
```

**Application Overloaded**
- High CPU usage (90-100%)
- High memory pressure
- Application responses slow
- Need more power

**After Vertical Scaling**
```
Instance Type: t2.large (upgraded)
├─ vCPU: 2
├─ Memory: 8 GB
└─ Application: Running same app, more capacity
```

**Result**: Same application, much more processing power

#### When to Use Vertical Scaling

**Ideal Scenarios**
- Non-distributed systems
- Databases (RDS, ElastiCache)
- Single-instance applications
- Monolithic architectures

**Not Ideal For**
- Web applications (multiple instances better)
- Stateless services (horizontal better)
- High availability requirements

#### Vertical Scaling Limits

**Hardware Limits**
- **Smallest Instance**: t2.nano
  - vCPU: 1
  - Memory: 0.5 GB
  - Basic applications only

- **Largest Instance**: Au-t12tb1.metal
  - vCPU: 450
  - Memory: 12.3 TB
  - Massive data processing

- **Gap**: Enormous range from smallest to largest

**Real Limits**
- Physical hardware constraints
- Cost escalation (larger instances exponentially more expensive)
- Instance availability (not all sizes available in all regions)
- Downtime required (restart instance to change type)

#### Vertical Scaling Disadvantages
- **Downtime**: Instance must restart during scaling
- **Hardware Limits**: Eventually hits maximum instance size
- **Cost**: Very expensive at top end
- **Single Point of Failure**: If instance fails, entire application down

### Horizontal Scalability (Scaling Out / Elasticity)

#### Concept
- **Action**: Increase the number of instances/systems
- **Direction**: Multiple identical instances
- **Distribution**: Load distributed across instances

#### Call Center Analogy
```
Scenario: Phone operators overloaded with calls

Before (One Operator):
- Operator: 1 person
- Capacity: 10 calls per minute
- Overloaded: Still can't keep up

After Horizontal Scaling (Multiple Operators):
- Operator 1: Handling calls
- Operator 2: Hired, handling calls
- Operator 3: Hired, handling calls
- ...
- Operator 6: Hired, handling calls

Capacity Growth:
- 1 operator: 10 calls/min
- 6 operators: 60 calls/min
- Added distribution, not upgrading

Result: Capacity multiplied by adding more workers
```

#### EC2 Example: Horizontal Scaling (Scale Out)

**Before Scaling: Single Instance**
```
Auto Scaling Group: 1 instance
├─ Instance 1: t2.micro
│  ├─ Running web application
│  ├─ Handling 100 requests/second
│  └─ CPU: 90% (struggling)
└─ Problem: Cannot handle spike to 200 requests/second
```

**Scale Out (Add More Instances)**
```
Auto Scaling Group: 3 instances
├─ Instance 1: t2.micro (requests/sec: ~67)
├─ Instance 2: t2.micro (requests/sec: ~67)
├─ Instance 3: t2.micro (requests/sec: ~66)
└─ Load Balancer: Distributes traffic

Result: 200 requests/second handled easily
```

**Scale In (Remove Instances)**
```
Auto Scaling Group: 2 instances
├─ Instance 1: t2.micro (requests/sec: ~100)
├─ Instance 2: t2.micro (requests/sec: ~100)
└─ Load Balancer: Distributes traffic

Result: Reduced to number needed, costs decrease
```

#### Horizontal Scaling Architecture

**Requires Load Balancing**
```
Internet Traffic
       ↓
   ┌───────────────┐
   │ Load Balancer │
   └───────┬───────┘
           │
      ┌────┼────┐
      ↓    ↓    ↓
   ┌─────────────────────┐
   │  Instance 1 (t2.m)  │
   │  Running App        │
   └─────────────────────┘
   ┌─────────────────────┐
   │  Instance 2 (t2.m)  │
   │  Running App        │
   └─────────────────────┘
   ┌─────────────────────┐
   │  Instance 3 (t2.m)  │
   │  Running App        │
   └─────────────────────┘
```

#### Horizontal Scaling Requirements

**Distributed System**
- Application designed for multiple instances
- Stateless (or session stored externally)
- No local data storage
- Database or external cache shared

**Load Distribution**
- Load Balancer required
- ELB (Classic), ALB, NLB options
- Distributes incoming requests
- Health checks on instances

**AutoScaling**
- Automatically adds/removes instances
- Based on metrics (CPU, memory, custom)
- Scales out when demand increases
- Scales in when demand decreases

#### Horizontal Scaling Advantages
- **No Downtime**: Add/remove instances without stopping others
- **Unlimited Scaling**: Can add many instances
- **High Availability**: If one fails, others continue
- **Cost Efficient**: Scale based on demand
- **Modern Cloud**: Native to AWS and cloud platforms

#### Horizontal Scaling Disadvantages
- **Complexity**: Requires load balancer, shared storage
- **Application Design**: Must be horizontally scalable
- **Stateless Requirement**: More complex than stateful systems
- **Data Consistency**: More challenging across instances

---

## Part 2: High Availability (HA)

### Definition of High Availability

**High Availability** = Running your application/system in **at least two availability zones (AZs)** or **two data centers** to survive **data center loss**.

**Key Concept**: If one data center fails, system continues operating from another.

### High Availability: Passive Model

#### Concept
- **Standby Instance**: Secondary instance in different AZ
- **State**: Not actively serving traffic
- **Failover**: Automatic or manual switch if primary fails
- **Example**: RDS Multi-AZ

#### Call Center Analogy
```
Scenario: Building backup for call center

Setup:
- Primary Building: New York
  ├─ 3 operators on phones
  ├─ All calls routed here
  └─ Active, handling all traffic

- Secondary Building: San Francisco
  ├─ 3 operators on standby
  ├─ Not handling calls normally
  └─ Ready if primary fails

Disaster Scenario (New York Internet Down):
- Primary Building (New York): DOWN
- Calls automatically reroute to San Francisco
- Standby operators activate
- Service continues

Result: High availability through redundancy
```

#### RDS Multi-AZ Example

**Normal Operation**
```
Primary Database (AZ-1a)
├─ Region: us-east-1a
├─ Status: Active
├─ Serving: All read/write traffic
└─ Performance: Normal

Standby Database (AZ-1b)
├─ Region: us-east-1b
├─ Status: Passive/Standby
├─ Serving: No traffic
├─ Role: Failover standby
```

**AZ-1a Failure**
```
Primary Database (AZ-1a)
├─ Status: DOWN ✗
└─ Cannot serve traffic

Automatic Failover (few seconds)
└─ Standby promoted to primary

Standby Database (AZ-1b) → Now Active
├─ Status: Active
├─ Serving: All read/write traffic
└─ DNS updated automatically
```

#### Passive HA Characteristics
- **Failover Required**: Must detect and switch to standby
- **Brief Downtime**: Few seconds during failover
- **Simple Setup**: No load balancer needed
- **Cost**: Two instances running (both charged)
- **Automated**: Automatic failover available

### High Availability: Active Model

#### Concept
- **Multiple Active Instances**: All serving traffic simultaneously
- **Load Distribution**: Load balanced across instances
- **Geography**: Multiple AZs, all active
- **Redundancy**: Built-in, no waiting for failover

#### Call Center Analogy
```
Scenario: Active phone center across both buildings

Setup:
- Primary Building (New York):
  ├─ 3 operators actively taking calls
  ├─ Shift 1: All answering phones
  └─ Handling 50% of calls

- Secondary Building (San Francisco):
  ├─ 3 operators actively taking calls
  ├─ Shift 1: All answering phones
  └─ Handling 50% of calls

Call Distribution:
- Load Balancer: Intelligent call routing
- New York: Getting 50% of calls
- San Francisco: Getting 50% of calls

Disaster (New York Building Goes Down):
- New York Building: OFFLINE ✗
- San Francisco: Already handling calls ✓
- No downtime: They just handle 100% of calls now

Result: Immediate failover, no downtime
```

#### EC2 Auto Scaling Group with Multi-AZ

**Active HA Setup**
```
Load Balancer (Multi-AZ)
       ↓
    ┌──┴──┐
    ↓     ↓

AZ-1a (us-east-1a)    AZ-1b (us-east-1b)
├─ Instance 1         ├─ Instance 3
│  Status: Active     │  Status: Active
│  Serving: Traffic   │  Serving: Traffic
└─ 50% load           └─ 50% load

├─ Instance 2         ├─ Instance 4
│  Status: Active     │  Status: Active
│  Serving: Traffic   │  Serving: Traffic
└─ Backup             └─ Backup
```

**AZ-1a Failure**
```
AZ-1a (FAILED ✗)      AZ-1b (us-east-1b ✓)
├─ Instance 1: DOWN   ├─ Instance 3: Active
├─ Instance 2: DOWN   ├─ Instance 4: Active
└─ Cannot serve       └─ Serving: 100% load

Result: Zero downtime, AZ-1b handles full load
```

#### Active HA Characteristics
- **No Failover Wait**: Already handling traffic
- **Zero Downtime**: Instance failure is seamless
- **Load Distributed**: Inherent load balancing
- **Cost**: All instances running (higher cost)
- **Scalability**: Combines with horizontal scaling

### Passive vs. Active: Comparison

| Aspect | Passive HA | Active HA |
|--------|-----------|----------|
| **Standby Status** | Not serving traffic | Serving traffic |
| **Failover** | Manual or automatic switch | Automatic (load balancer) |
| **Downtime** | Few seconds | None (zero downtime) |
| **Cost** | Two instances running | All instances running |
| **Complexity** | Simple | Moderate (requires LB) |
| **Use Case** | Databases, non-distributed | Web apps, distributed systems |
| **Performance** | Cached data warm on failover | No warm-up needed |
| **Example** | RDS Multi-AZ | ASG Multi-AZ with LB |

---

## Part 3: EC2 Scalability and HA Implementations

### EC2 Vertical Scaling

**Scaling Up**
1. Stop instance: `aws ec2 stop-instances`
2. Change instance type: `aws ec2 modify-instance-attribute`
3. Start instance: `aws ec2 start-instances`
4. **Downtime**: Yes (few minutes)

**Scaling Down**
- Same process, smaller instance type
- **Cost**: Reduced charges

**Use Cases**
- Applications must stay on single instance
- Databases that can't be distributed
- Existing infrastructure

### EC2 Horizontal Scaling (Auto Scaling Group)

**Scaling Out** (Add Instances)
```bash
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 5
```

**Scaling In** (Remove Instances)
```bash
aws autoscaling set-desired-capacity \
  --auto-scaling-group-name my-asg \
  --desired-capacity 2
```

**Automatic Scaling**
- CloudWatch metrics trigger scaling
- CPU > 70%: Scale out
- CPU < 30%: Scale in
- Custom metrics possible

### EC2 High Availability

**Type 1: Single AZ (No HA)**
```
Auto Scaling Group: 1 AZ only
├─ Instance 1: AZ-1a
├─ Instance 2: AZ-1a
└─ Problem: If AZ fails, all instances down
```

**Type 2: Multi-AZ HA**
```
Auto Scaling Group: Multi-AZ enabled
├─ Instance 1: AZ-1a
├─ Instance 2: AZ-1b
└─ Benefit: Survives AZ failure
```

**Load Balancer Multi-AZ**
```
Classic LB / ALB / NLB: Multi-AZ
├─ LB Nodes: AZ-1a, AZ-1b
├─ Instances: Multiple AZs
└─ High Availability: Built-in
```

---

## Part 4: Combined Scalability and HA

### Best Practice: Both Together

**Production Architecture**
```
High Availability (Multi-AZ):
├─ AZ-1a: Instance 1, Instance 2
└─ AZ-1b: Instance 3, Instance 4

Scalability (Auto Scaling Group):
├─ Scale out: More instances added
├─ Scale in: Instances removed
└─ Applies to both AZs

Load Balancer:
├─ Distributes traffic: Across instances
├─ Functions: Multi-AZ enabled
└─ Health checks: Removes failed instances
```

**Benefits**
- **High Availability**: Survives AZ failure
- **Scalability**: Handles traffic spikes
- **Cost Efficiency**: Scale down when not needed
- **Resilience**: Multiple layers of redundancy

### AWS Services Supporting Scalability and HA

**Scalable + HA Services**
- EC2 + Auto Scaling Group + Load Balancer
- ECS (Elastic Container Service)
- Lambda (serverless, automatic scaling)
- DynamoDB (horizontal scaling)
- S3 (unlimited horizontal scaling)

**Scalable Only (Vertical)**
- RDS (single instance, but Multi-AZ for HA)
- ElastiCache (single node, but Multi-AZ clusters)
- ECR (storage scales)

---

## Part 5: Instance Size Range

### From Smallest to Largest

**Smallest**
```
t2.nano
├─ vCPU: 1
├─ Memory: 0.5 GB
├─ Use: Development, testing
└─ Cost: ~$4.75/month
```

**Other Common Sizes**
```
t2.micro: 1 vCPU, 1 GB
t2.small: 1 vCPU, 2 GB
t2.medium: 2 vCPU, 4 GB
t2.large: 2 vCPU, 8 GB
t2.xlarge: 4 vCPU, 16 GB
```

**Largest**
```
Au-t12tb1.metal
├─ vCPU: 450
├─ Memory: 12.3 TB
├─ Use: Massive parallel processing
└─ Cost: $650k+/month
```

**Scale Range**: 0.5 GB → 12.3 TB RAM, 1 → 450 vCPUs
- Over 24,000x difference in capacity
- Demonstrates vertical scaling potential
- Hardware limits real and substantial

---

## Part 6: Distinguishing in Exam Questions

### Key Differentiators

**Question: "How to handle increased traffic?"**

**Answer Depends On:**
- **Vertical Scalability**: Your application is monolithic, single instance only
- **Horizontal Scalability**: Your application is stateless and can run on multiple instances

**Question: "How to survive data center failure?"**

**Answer:** High Availability
- Not scalability question
- About redundancy, not throughput

**Question: "Increasing instance from t2.micro to t2.large"**

**Answer:** Vertical Scalability
- Not high availability
- Single instance increasing in power

**Question: "Adding more instances behind load balancer"**

**Answer:** Horizontal Scalability (Elasticity)
- Multiple instances
- Distribution of load

**Question: "Running same system in two AZs"**

**Answer:** High Availability (Active Model)
- Both serving traffic
- Could have scalability too

### Common Exam Tricks

**Trap 1: HA ≠ Scalability**
- Multi-AZ doesn't increase throughput
- Just ensures availability if AZ fails
- Many students confuse these

**Trap 2: Vertical ≠ Elasticity**
- Vertical scaling requires downtime (usually)
- Elasticity implies no downtime, dynamic changes
- RDS is vertically scalable but not elastic

**Trap 3: Single AZ Multiple Instances**
- Horizontal scalability ✓
- High availability ✗
- Fails if entire AZ goes down

**Remember the Call Center**
- Junior → Senior Operator = Vertical
- 1 Operator → 6 Operators = Horizontal
- One Building → Two Buildings = High Availability

---

## Key Takeaways for SysOps Associate

### Definitions (Critical)
- **Scalability**: Ability to handle greater load by adapting
- **Vertical Scalability**: Increase instance size (scale up/down)
- **Horizontal Scalability**: Increase number of instances (scale out/in)
- **High Availability**: Run in 2+ AZs/data centers to survive failure

### EC2 Implementation
- **Vertical**: t2.nano to Au-t12tb1.metal range
- **Horizontal**: Auto Scaling Groups with multiple instances
- **HA**: Multi-AZ enabled for ASG and Load Balancers

### When to Use
- **Vertical**: Databases, monolithic apps, non-distributed systems
- **Horizontal**: Web apps, stateless services, modern applications
- **HA**: Production systems requiring high uptime
- **Both**: Real production environments combine all three

### Exam Focus
- Distinguish scalability from high availability
- Know vertical vs. horizontal differences
- Understand architectures that implement each
- Recognize Common exam tricks differentiating these concepts
- Remember: These are linked but different concepts