# 6. Amazon EKS Overview

## Overview

**Amazon EKS (Elastic Kubernetes Service)** is a managed Kubernetes service on AWS. Kubernetes is an open-source container orchestration system — an alternative to ECS with a different API. EKS is the right choice when your company already uses Kubernetes or needs cloud-agnostic container management.

---

## ECS vs EKS

| | ECS | EKS |
|-|-----|-----|
| **API** | AWS proprietary | Kubernetes (open-source) |
| **Portability** | AWS only | Works on any cloud (Azure, GCP, on-prem) |
| **Tasks** | ECS Tasks | Kubernetes Pods |
| **Hosts** | Container instances / Fargate | Worker Nodes / Fargate |
| **Use case** | AWS-native container workloads | Multi-cloud, existing K8s, K8s API required |

---

## When to Use EKS

- Company already uses Kubernetes on-premises or in another cloud
- Need to migrate containers between clouds (Kubernetes is cloud-agnostic)
- Team is already familiar with the Kubernetes API
- Want AWS to manage the Kubernetes control plane

---

## Launch Modes

| Mode | Description |
|------|-------------|
| **EC2** | You deploy EC2 Worker Nodes; EKS schedules Pods on them |
| **Fargate** | Serverless — no nodes to manage; AWS runs Pods directly |

---

## Node Types (EC2 Mode)

| Type | Description | Instance Types |
|------|-------------|---------------|
| **Managed Node Groups** | AWS creates and manages EC2 nodes in an ASG | On-Demand + Spot |
| **Self-managed Nodes** | You create nodes, register to EKS cluster, manage ASG yourself | On-Demand + Spot |
| **Fargate** | No nodes — fully serverless | N/A |

> Self-managed nodes can use the pre-built **Amazon EKS Optimized AMI** or a custom AMI.

---

## Architecture

```
VPC (3 AZs: public + private subnets)
    ├── Public subnet  → Load Balancer (public-facing)
    └── Private subnet → EKS Worker Nodes (ASG)
                              ├── Node 1 → Pod A, Pod B
                              ├── Node 2 → Pod C
                              └── Node 3 → Pod D
```

- Pods = containers (Kubernetes equivalent of ECS tasks)
- Nodes = EC2 instances running the Pods
- Services are exposed via private or public load balancers

---

## Data Volumes (Storage)

Requires a **StorageClass manifest** and a **CSI (Container Storage Interface) compliant driver**.

| Storage | Notes |
|---------|-------|
| **Amazon EBS** | Block storage per node |
| **Amazon EFS** | **Only storage class compatible with Fargate** |
| **Amazon FSx for Lustre** | High-performance workloads |
| **Amazon FSx for NetApp ONTAP** | Enterprise file storage |

> For Fargate-based EKS Pods, **EFS is the only supported persistent storage option**.

---

## Best Practices

✓ **Use EKS when Kubernetes portability is required** — ECS is simpler but AWS-only  
✓ **Use Managed Node Groups for most EC2 workloads** — AWS handles lifecycle and patching  
✓ **Use Fargate for serverless Kubernetes** — no node management, simpler operations  
✓ **Use EFS for persistent storage on Fargate Pods** — EBS is not compatible with Fargate in EKS  
✓ **Use Spot Instances with Managed Node Groups** — cost savings for fault-tolerant workloads  

---

## SysOps Exam Focus

**Q1: "Your company uses Kubernetes on-premises and wants to migrate to AWS while keeping the same Kubernetes API and tooling. Which service should you use?"**
- A) Amazon ECS with Fargate
- B) Amazon EKS — managed Kubernetes on AWS, compatible with existing K8s tooling
- C) AWS Batch
- D) Amazon ECS with EC2 launch type
- **Answer: B** — EKS is the AWS-managed Kubernetes service and is cloud-agnostic; ECS uses a proprietary API

**Q2: "You are running EKS workloads on Fargate and need persistent shared storage for Pods across multiple AZs. Which storage option should you use?"**
- A) Amazon EBS — attach one volume per Pod
- B) Amazon EFS — the only CSI storage class compatible with EKS Fargate
- C) Amazon FSx for Lustre
- D) Instance store — fastest option
- **Answer: B** — EFS is the only persistent storage option compatible with Fargate in EKS; EBS requires a node attachment

**Q3: "What is the difference between Managed Node Groups and Self-managed Nodes in EKS?"**
- A) Managed Node Groups support Spot instances; self-managed nodes do not
- B) Managed Node Groups: AWS creates and manages EC2 nodes in an ASG; Self-managed: you create, register, and manage nodes in your own ASG
- C) Self-managed nodes use Fargate; Managed Node Groups use EC2
- D) There is no functional difference — only pricing differs
- **Answer: B** — Managed Node Groups offload node lifecycle management to AWS; self-managed gives more customization control

---

## Quick Reference

```
Amazon EKS:
  Kubernetes = open-source, cloud-agnostic
  Alternative to ECS — different API, same goal (run containers)
  
  Terminology:
    ECS Tasks = Kubernetes Pods
    Container instances = Worker Nodes

Launch modes:
  EC2 → you manage nodes (Managed Node Groups or Self-managed)
  Fargate → serverless, no nodes

Node types:
  Managed Node Groups → AWS manages ASG, EC2 lifecycle
  Self-managed        → you manage ASG, can use EKS Optimized AMI
  Fargate             → no nodes at all

Data volumes (CSI driver required):
  EBS     → per-node block storage
  EFS     → shared, multi-AZ, Fargate-compatible (only option for Fargate)
  FSx     → Lustre or NetApp ONTAP

Use EKS when: existing K8s, multi-cloud, K8s API required
Use ECS when: AWS-native, simpler setup preferred
```

---

**File: 6_Amazon_EKS_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
