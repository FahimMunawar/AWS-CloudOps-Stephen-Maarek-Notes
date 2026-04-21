# 3. Amazon ECS — Create a Service (Hands-On)

## Overview

Walkthrough of creating a task definition, launching it as an ECS service on Fargate with an ALB, scaling tasks up and down, and cleaning up.

---

## Step 1: Create a Task Definition

**Console:** ECS → Task definitions → **Create new task definition**

| Setting | Value |
|---------|-------|
| Task definition name | `nginxdemos-hello` |
| Launch type | Fargate (serverless) |
| OS / Architecture | Linux |
| Task CPU | 0.5 vCPU |
| Task memory | 1 GB |
| Task role | None (only needed if container calls AWS APIs) |
| Task execution role | Leave as default — auto-created by ECS |

### Container Configuration

| Setting | Value |
|---------|-------|
| Container name | `nginxdemos-hello` |
| Image URL | `nginxdemos/hello` (pulls from Docker Hub) |
| Port mapping | 80 → 80 |
| Storage | Default 21 GiB ephemeral (Fargate) |
| Environment variables, logging | Default |

Click **Create** → task definition created (revision 1)

---

## Step 2: Create a Service

**Console:** ECS → Clusters → DemoCluster → Services → **Create**

| Setting | Value |
|---------|-------|
| Task definition family | `nginxdemos-hello` |
| Revision | Latest |
| Service name | (auto-generated or custom) |
| Compute | Capacity provider strategy → Fargate |
| Platform version | Latest |
| Deployment type | Replica |
| Desired tasks | 1 |

### Networking

| Setting | Value |
|---------|-------|
| Subnets | Default VPC subnets |
| Security group | Create new — allow HTTP (port 80) from anywhere |
| Public IP | Enabled |

### Load Balancer

| Setting | Value |
|---------|-------|
| Type | Application Load Balancer |
| Name | `DemoALBForECS` |
| Listener port | 80 |
| Target group | Create new → `nginxdemosTG` → port 80 |

Click **Create service**

---

## Step 3: Verify the Deployment

1. Service status: **Active** | Desired: 1 | Running: 1
2. Target group → **DemoALBForECS** → 1 IP registered (container private IP)
3. Copy ALB DNS name → paste in browser → **nginx welcome page loads**

**Console navigation:**
- Service → **Tasks** tab → click task → view config, private IP, logs
- Service → **Events** tab → task started, registered in target group, steady state

---

## Step 4: Scale Up

**Console:** Service → **Update service** → change Desired tasks to `3`

```
ECS requests 2 more Fargate tasks
        ↓
AWS provisions resources (pending → activating → running)
        ↓
ALB registers 3 targets
        ↓
Browser refresh → load balancing across 3 containers (IP changes each refresh)
```

---

## Step 5: Scale Down and Cleanup

To stop all containers (preserve service definition):

1. **Update service** → Desired tasks = `0`
2. **Auto Scaling Group** → Desired capacity = `0`
3. Verify: Tasks tab shows 0 running

> To fully delete: delete service → delete task definition → delete cluster

---

## Key Observations

| Observation | Explanation |
|-------------|-------------|
| Task role left empty | This container makes no AWS API calls |
| Task execution role auto-created | ECS creates `ecsTaskExecutionRole` automatically if missing |
| ALB created during service creation | Can be created on-the-fly or pre-existing |
| 3 tasks, each gets a different IP | ALB round-robins across all registered targets |
| Scale up takes ~30–60 seconds | Fargate provisions infrastructure on demand |

---

## Best Practices

✓ **Use Fargate for new services** — no EC2 instances to manage  
✓ **Set Task Role explicitly** — if your container calls AWS APIs, define least-privilege task role  
✓ **Use ALB for HTTP services** — supports health checks, path routing, and target group management  
✓ **Scale desired tasks to 0 to pause** — preserves service config without incurring container charges  
✓ **Check Events tab for deployment issues** — shows registration failures and health check problems  

---

## Quick Reference

```
Task Definition:
  Name, launch type (Fargate), CPU/memory, container image + ports
  Task role → AWS API permissions for the container
  Task execution role → ECS Agent permissions (auto-created)

Service:
  Links task definition to a cluster
  Deployment: replica → N desired tasks across the cluster
  Networking: subnets + security group + public IP
  Load balancer: ALB → listener → target group

Scale:
  Update service → change desired tasks
  Fargate provisions automatically (no EC2 to manage)

Cleanup (preserve service):
  Desired tasks → 0 | ASG desired → 0
```

---

**File: 3_ECS_Service_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
