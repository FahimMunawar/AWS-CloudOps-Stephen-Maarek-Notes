# 1. Amazon ECS Overview

## Overview

**Amazon ECS (Elastic Container Service)** runs Docker containers on AWS as **ECS Tasks** inside an **ECS Cluster**. Two launch types are available: EC2 (you manage infrastructure) and Fargate (serverless). The exam strongly favors Fargate for its simplicity.

---

## Launch Types

### EC2 Launch Type

| Property | Detail |
|----------|--------|
| **Infrastructure** | You provision and maintain EC2 instances |
| **Agent** | Each EC2 instance must run the **ECS Agent** to register with the cluster |
| **Container placement** | AWS starts/stops containers on the registered EC2 instances |
| **Management burden** | High — you manage EC2 instances, scaling, patching |

```
ECS Cluster (EC2 Launch Type)
  ├── EC2 Instance (ECS Agent) ── Container A, Container B
  ├── EC2 Instance (ECS Agent) ── Container C
  └── EC2 Instance (ECS Agent) ── Container D
```

### Fargate Launch Type

| Property | Detail |
|----------|--------|
| **Infrastructure** | None — fully serverless |
| **Agent** | Not required — AWS manages everything |
| **Container placement** | Define CPU/RAM in task definition → AWS runs it |
| **Scaling** | Increase number of tasks — no EC2 management |
| **Management burden** | Minimal |

> **Exam tip:** When a question mentions "no servers to manage" or "serverless containers," the answer is Fargate.

---

## IAM Roles for ECS

### EC2 Instance Profile Role (EC2 Launch Type only)

Used by the **ECS Agent** running on the EC2 instance:

| API Call | Purpose |
|----------|---------|
| ECS API | Register instance with the cluster |
| CloudWatch Logs | Send container logs |
| ECR API | Pull Docker images |
| Secrets Manager / SSM Parameter Store | Access sensitive data |

### ECS Task Role (Both EC2 and Fargate)

- One role **per task definition** — different tasks can have different roles
- Defined in the **task definition**

```
Task A  ── Task A Role ──► S3 API calls
Task B  ── Task B Role ──► DynamoDB API calls
```

> **Key distinction:** EC2 Instance Profile = ECS Agent permissions. Task Role = application container permissions.

---

## Load Balancer Integration

| Load Balancer | When to Use |
|--------------|-------------|
| **ALB (Application Load Balancer)** | Recommended for most use cases; works with Fargate |
| **NLB (Network Load Balancer)** | High throughput / high performance; AWS PrivateLink |
| **Classic Load Balancer** | Not recommended — no advanced features; does not support Fargate |

> ALB is the standard choice for ECS — supports HTTP/HTTPS, path-based routing, and Fargate.

---

## Data Persistence — EFS with ECS

For persistent, shared storage across ECS tasks:

| Property | Detail |
|----------|--------|
| **File system** | Amazon EFS (network file system) |
| **Compatibility** | Works with both EC2 and Fargate launch types |
| **Multi-AZ** | Tasks in any AZ can access the same EFS data |
| **Serverless combo** | Fargate + EFS = fully serverless (no servers, no storage to manage) |

```
ECS Cluster
  ├── Task (AZ-A) ──┐
  ├── Task (AZ-B) ──┼──► Amazon EFS (shared persistent storage)
  └── Task (AZ-C) ──┘
```

**Use case:** Persistent multi-AZ shared storage for containers — e.g., shared content, configuration, or state.

> **Note:** S3 cannot be mounted as a file system. EFS is the correct choice for shared persistent storage with ECS.

---

## Best Practices

✓ **Use Fargate for new workloads** — no infrastructure to manage, easier scaling  
✓ **Define a separate Task Role per task** — least privilege; tasks only access what they need  
✓ **Use ALB in front of ECS** — supports path routing, health checks, and Fargate  
✓ **Use EFS for shared persistent storage** — the only network file system compatible with both launch types  
✓ **Store secrets in Secrets Manager or SSM Parameter Store** — reference them via the EC2 Instance Profile or Task Role  

---

## SysOps Exam Focus

**Q1: "You want to run Docker containers on AWS without managing any EC2 instances. Which ECS launch type should you use?"**
- A) EC2 Launch Type with Auto Scaling
- B) Fargate Launch Type — serverless, no EC2 instances to manage
- C) ECS with an EKS cluster
- D) EC2 Launch Type with Spot instances
- **Answer: B** — Fargate abstracts all server management; you only define CPU/RAM in the task definition

**Q2: "An ECS task running on Fargate needs to read objects from S3. How do you grant it the required permissions?"**
- A) Attach an IAM policy to the ECS cluster
- B) Create an ECS Task Role with S3 read permissions and assign it in the task definition
- C) Add the S3 permission to the EC2 instance profile
- D) Use a VPC endpoint — no IAM permissions needed
- **Answer: B** — Task Roles are the mechanism for granting container-level permissions; they apply to both EC2 and Fargate

**Q3: "Multiple ECS tasks across three AZs need to share the same file data. What storage solution should you use?"**
- A) EBS volume — mount to each task
- B) S3 — mount as a file system
- C) Amazon EFS — a network file system accessible from tasks in any AZ
- D) Instance store — fastest option for shared data
- **Answer: C** — EFS is the only network file system that supports multi-AZ shared storage for ECS (both EC2 and Fargate)

**Q4: "What is the purpose of the EC2 Instance Profile role in an ECS EC2 Launch Type cluster?"**
- A) It grants permissions to the Docker containers running on the instance
- B) It grants permissions to the ECS Agent on the EC2 instance to make API calls to ECS, ECR, CloudWatch, and Secrets Manager
- C) It is the same as the ECS Task Role
- D) It is required for Fargate launch type only
- **Answer: B** — The EC2 Instance Profile is used by the ECS Agent (not the containers); containers use Task Roles

---

## Fargate Ephemeral Storage

| Property | Detail |
|----------|--------|
| **Range** | 20 GiB (default) to **200 GiB** |
| **Scope** | Shared among containers within the same Fargate task |
| **Encryption** | Encrypted at rest |
| **Lifetime** | **Ephemeral** — deleted when the task stops |
| **Use case** | Temporary scratch space, inter-container file sharing within a task |

> EFS is required for persistent storage that survives task termination.

---

## ECS Log Routing — FireLens (Sidecar Pattern)

Send ECS task logs to CloudWatch or third-party destinations using a **FireLens** sidecar container.

| Component | Detail |
|-----------|--------|
| **FireLens** | Log router built on **Fluent Bit** or **Fluentd** |
| **Pattern** | Sidecar — runs as an additional container in the same task definition |
| **Destinations** | CloudWatch Logs, S3, Kinesis, Datadog, Splunk, and more |
| **Config** | Defined in the task definition alongside your application container |

```
ECS Task Definition:
  ├── App container (your application)
  └── FireLens container (sidecar — log router)
              ↓
       CloudWatch Logs / Partner destination
```

---

## Quick Reference

```
ECS Launch Types:
  EC2      → you manage EC2 instances + ECS Agent on each instance
  Fargate  → serverless, define CPU/RAM in task def, AWS runs it

IAM Roles:
  EC2 Instance Profile → ECS Agent permissions (EC2 only)
    → ECS API, ECR, CloudWatch Logs, Secrets Manager
  ECS Task Role → container permissions (EC2 + Fargate)
    → defined in task definition, one per task type

Load Balancers:
  ALB → recommended (HTTP/HTTPS, Fargate support)
  NLB → high throughput / PrivateLink
  CLB → not recommended, no Fargate support

Data Persistence:
  EFS → shared, multi-AZ, compatible with EC2 + Fargate
  Best combo: Fargate + EFS = fully serverless

Exam signal: "serverless containers" → Fargate
```

---

**File: 1_ECS_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
