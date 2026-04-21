# 4. ECS Service Auto Scaling

## Overview

ECS tasks can be scaled automatically using **AWS Application Auto Scaling**. Scaling at the task level (ECS service) is separate from scaling the EC2 instances behind them (cluster level). Fargate eliminates the cluster scaling problem entirely.

---

## ECS Service Auto Scaling

Automatically increases or decreases the number of ECS tasks in a service.

### Scaling Metrics

| Metric | Source |
|--------|--------|
| **ECSServiceAverageCPUUtilization** | ECS service CPU |
| **ECSServiceAverageMemoryUtilization** | ECS service RAM |
| **ALBRequestCountPerTarget** | ALB metric |

### Scaling Types

| Type | How |
|------|-----|
| **Target Tracking** | Keep a metric at a specific target value (e.g., CPU at 60%) |
| **Step Scaling** | Scale by defined amounts when CloudWatch Alarm thresholds are crossed |
| **Scheduled Scaling** | Scale ahead of time for predictable traffic patterns |

---

## Task Scaling vs Cluster Scaling (EC2 Launch Type)

> **Critical distinction:** Scaling ECS tasks does NOT automatically scale the underlying EC2 instances.

```
More tasks needed
        ↓
ECS Service Auto Scaling adds tasks
        ↓
If EC2 instances don't have enough CPU/RAM → tasks stay PENDING
        ↓
EC2 cluster must also scale
```

### Two Ways to Scale EC2 Cluster Capacity

| Method | How | Recommendation |
|--------|-----|---------------|
| **ASG Scaling** | Scale ASG based on CPU utilization metric | Basic, less intelligent |
| **ECS Cluster Capacity Provider** | Automatically scales ASG when tasks can't be placed due to insufficient capacity | **Preferred — smarter** |

> Always use **ECS Cluster Capacity Provider** over raw ASG scaling for the EC2 launch type.

---

## Why Fargate Simplifies Auto Scaling

With Fargate:
- No EC2 instances to manage
- Service Auto Scaling = the only scaling you need
- AWS provisions compute resources automatically

```
Fargate:
  Task count ↑ → AWS provisions resources automatically
  No cluster scaling required
```

With EC2 Launch Type:
```
Task count ↑ → may hit capacity limit → must also scale EC2 instances
```

---

## Auto Scaling Flow

```
More users → CPU Usage rises
        ↓
CloudWatch Metric (ECSServiceAverageCPUUtilization)
        ↓
CloudWatch Alarm triggered
        ↓
AWS Application Auto Scaling → increase desired task count
        ↓
New ECS task launched

(EC2 Launch Type only):
        ↓
ECS Capacity Provider detects insufficient cluster capacity
        ↓
Auto Scaling Group scales out → new EC2 instance registered
        ↓
Pending task placed on new instance
```

---

## Best Practices

✓ **Use Fargate when possible** — eliminates the two-layer scaling complexity  
✓ **Use ECS Cluster Capacity Provider over raw ASG scaling** — it responds to actual task placement failures  
✓ **Use Target Tracking for steady workloads** — keeps CPU/memory near a target percentage  
✓ **Use Scheduled Scaling for predictable traffic** — pre-warm before peak hours  
✓ **Monitor both task-level and cluster-level metrics** — both layers need attention on EC2 launch type  

---

## SysOps Exam Focus

**Q1: "Your ECS service CPU utilization is high. Auto Scaling adds more tasks but they remain in PENDING state. What is the likely cause?"**
- A) The task definition has an incorrect image URI
- B) The EC2 cluster does not have enough capacity to place the new tasks — scale the cluster using Capacity Provider or ASG
- C) The ALB target group is at capacity
- D) Fargate is not available in that region
- **Answer: B** — Task scaling and cluster scaling are separate; new tasks can't run if the EC2 instances are full

**Q2: "You are using the EC2 launch type and want the cluster to automatically scale EC2 instances when tasks can't be placed. What is the recommended approach?"**
- A) Set up a CloudWatch Alarm on CPU to trigger ASG scaling
- B) Use ECS Cluster Capacity Provider paired with an ASG — it automatically scales EC2 instances when task placement fails
- C) Manually increase the ASG desired count when tasks are pending
- D) Use Fargate as the cluster type
- **Answer: B** — Capacity Provider is the smarter, preferred method for automatic EC2 cluster scaling

**Q3: "Which metric from an ALB can be used to trigger ECS service auto scaling?"**
- A) ALBActiveConnectionCount
- B) ALBRequestCountPerTarget
- C) ALBTargetResponseTime
- D) ALBHealthyHostCount
- **Answer: B** — ALBRequestCountPerTarget is one of the three supported ECS service auto scaling metrics

---

## Quick Reference

```
ECS Service Auto Scaling (task level):
  Service: AWS Application Auto Scaling
  Metrics: ECSServiceAverageCPUUtilization
           ECSServiceAverageMemoryUtilization
           ALBRequestCountPerTarget
  Types: Target Tracking | Step Scaling | Scheduled

EC2 Launch Type — cluster scaling:
  Option 1: ASG scaling on CPU (basic)
  Option 2: ECS Cluster Capacity Provider (preferred)
    → scales ASG automatically when task placement fails

Fargate:
  Service Auto Scaling only
  No cluster scaling needed — AWS handles compute

Exam tip:
  Task scaling ≠ cluster scaling (EC2 only)
  Fargate = simplest auto scaling story
  Capacity Provider > raw ASG scaling
```

---

**File: 4_ECS_Auto_Scaling.md**
**Status: SysOps-focused, exam-ready, concise format**
