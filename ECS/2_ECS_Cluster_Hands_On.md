# 2. Amazon ECS — Create a Cluster (Hands-On)

## Overview

Walkthrough of creating an ECS cluster with multiple capacity providers (Fargate, Fargate Spot, and EC2 via ASG), and registering an EC2 container instance.

---

## Step 1: Create the Cluster

**Console:** ECS → Clusters → **Create cluster**

| Setting | Value |
|---------|-------|
| Cluster name | `DemoCluster` |

---

## Step 2: Choose Infrastructure (Capacity Providers)

| Option | Description |
|--------|-------------|
| **Fargate only** | Serverless — AWS manages everything |
| **Fargate + Managed Instances** | Fargate + AWS manages EC2 instances for you |
| **Fargate + Self-managed instances** | Fargate + you manage your own EC2 Auto Scaling Group |

For this demo: **Self-managed instances** (to see all options)

### Self-Managed Instance Configuration

| Setting | Value |
|---------|-------|
| Auto Scaling Group type | On-demand |
| Instance type | `t3.micro` |
| EC2 instance role | `ecsInstanceRole` (create if not exists) |
| Infrastructure role | Create new (for ECS Managed Instances) |
| Max capacity | 2 |
| SSH | Not needed |
| Root EBS volume | 30 GiB minimum |

> AWS automatically creates an Auto Scaling Group named `Infra-ECS-Cluster-*` for the cluster.

---

## Step 3: Capacity Providers in the Cluster

After creation, the cluster has **three capacity providers**:

| Capacity Provider | Type |
|------------------|------|
| **FARGATE** | Serverless Fargate tasks |
| **FARGATE_SPOT** | Fargate using Spot pricing (cheaper, interruptible) |
| **ASG provider** | EC2 instances managed by the Auto Scaling Group |

---

## Step 4: Launch a Container Instance (EC2)

To add an EC2 instance to the cluster:

1. ECS → Cluster → **Infrastructure** tab
2. Edit the ASG provider → set **Desired capacity** to `1`
3. Wait for the EC2 instance to launch and register

After registration, under **Container instances**:

| Property | Value |
|----------|-------|
| Running tasks | 0 |
| CPU available | 1024 units |
| Memory available | ~982 MiB |

Tasks will be scheduled on this instance until capacity is exhausted.

---

## IAM Roles Created

| Role | Purpose |
|------|---------|
| `ecsInstanceRole` | EC2 Instance Profile — used by the ECS Agent on each EC2 instance |
| Infrastructure role | Used by ECS to manage the ASG and instances |

---

## What Was Built

```
DemoCluster
    ├── Capacity: FARGATE          (serverless tasks)
    ├── Capacity: FARGATE_SPOT     (spot serverless tasks)
    └── Capacity: ASG (EC2)        (t3.micro container instance)
              └── Container Instance: 1024 CPU / 982 MiB memory
```

---

## Best Practices

✓ **Use Fargate for simplicity** — no ASG or EC2 instances to manage  
✓ **Use FARGATE_SPOT for cost savings** — suitable for fault-tolerant batch workloads  
✓ **Create `ecsInstanceRole` before creating the cluster** — required for EC2 launch type  
✓ **Set max capacity on the ASG** — prevents uncontrolled instance scaling  

---

## Quick Reference

```
Create ECS cluster:
  Name: DemoCluster
  Infrastructure: Fargate / Managed EC2 / Self-managed EC2

Capacity providers (all three available after creation):
  FARGATE        → serverless
  FARGATE_SPOT   → serverless + spot pricing
  ASG provider   → EC2 instances via Auto Scaling Group

Register EC2 container instance:
  Infrastructure tab → edit desired capacity → wait for instance
  Instance registers with CPU/memory capacity displayed

IAM:
  ecsInstanceRole → ECS Agent on EC2 instances
  Infrastructure role → ECS manages ASG
```

---

**File: 2_ECS_Cluster_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
