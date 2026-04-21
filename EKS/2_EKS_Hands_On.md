# 2. Amazon EKS — Hands-On

## Overview

Walkthrough of creating an EKS cluster with Auto Mode, configuring IAM roles, adding a Node Group, and exploring key console features.

> **Cost warning:** EKS with Auto Mode provisions large EC2 instances automatically. Follow along visually only unless you intend to pay.

---

## Step 1: Create the Cluster

**Console:** EKS → **Create cluster** → Custom configuration

| Setting | Value |
|---------|-------|
| Mode | **EKS Auto Mode** — automatically creates nodes when Pods can't be scheduled |
| Cluster name | `DemoEKSCluster` |
| Kubernetes version | Default (latest stable) |
| Support type | Standard |
| Node pools (Auto Mode) | General purpose + System |

---

## Step 2: IAM Roles Required

### Cluster Role

1. Click **Create recommended role**
2. Attach required policies manually if not auto-added:
   - `AmazonEKSBlockStoragePolicy`
   - `AmazonEKSClusterPolicy`
   - `AmazonEKSComputePolicy`
   - `AmazonEKSLoadBalancingPolicy`
   - `AmazonEKSNetworkingPolicy`

### Node IAM Role (Auto Mode)

- Role name: `AmazonEKSAutoNodeRole`
- Click **Create recommended role** — policies are pre-selected

---

## Step 3: Networking

| Setting | Value |
|---------|-------|
| VPC | Default VPC |
| Subnets | Select public subnets (3 AZs) |
| IP type | IPv4 |
| Cluster endpoint access | **Public and private** |

---

## Step 4: Observability Options

| Option | Detail |
|--------|--------|
| CloudWatch metrics | CPU, memory, network metrics for nodes/pods |
| Prometheus | Alternative metrics destination |
| Control plane logs | API server, audit, scheduler logs → CloudWatch Logs |

---

## Step 5: Add-Ons

Pre-built extensions for the cluster. Key ones to know:

| Add-on | Purpose |
|--------|---------|
| **Amazon EBS CSI driver** | Allows EBS volumes to be used as PersistentVolumes in the cluster |
| **Amazon EFS CSI driver** | Allows EFS file systems — required for Fargate persistent storage |
| CoreDNS | DNS resolution within the cluster |
| SageMaker | ML workload integration |

> CSI = Container Storage Interface — required to mount EBS or EFS in EKS.

---

## Step 6: After Cluster Creation

**Console:** EKS → DemoEKSCluster

| Tab | What You See |
|-----|-------------|
| **Resources** | Pods, ReplicaSets, Deployments, StatefulSets (Kubernetes objects) |
| **Compute** | Nodes (EC2 instances provisioned by Auto Mode or Node Groups) |
| **Add-ons** | Installed cluster extensions |

---

## Step 7: Create a Node Group (Optional — EC2 Launch Type)

**Console:** Cluster → **Compute** → **Node groups** → **Add node group**

| Setting | Value |
|---------|-------|
| Name | `DemoNodeGroup` |
| Node IAM role | Create recommended role (`NodeGroupRoleEKS`) — 4 policies auto-attached |
| Launch template | Optional — pre-define AMI, instance type, etc. |
| AMI type | Amazon Linux (EKS-optimized) |
| Instance type | e.g., t3.medium |
| ASG min/max | Set desired range |
| Enable node repair | Yes |
| Subnets | Select where to launch nodes |

After creation: 2 EC2 instances appear in the EKS node group and in the EC2 console under Auto Scaling Groups.

---

## Step 8: Fargate Profile (Alternative to Node Groups)

**Console:** Cluster → Compute → **Fargate profiles** → Add

- Define namespace and label selectors
- Matching Pods run on Fargate — no nodes to manage

---

## Cleanup Order

1. Delete all Node Groups
2. Delete Fargate profiles (if any)
3. Delete the cluster

> Cluster cannot be deleted while node groups exist.

---

## Quick Reference

```
EKS cluster creation:
  Mode: Auto Mode (nodes created automatically) or Manual
  Cluster role: create recommended → attach EKS policies
  Node IAM role: AmazonEKSAutoNodeRole
  Networking: VPC + subnets + public+private endpoint access

Compute options:
  Auto Mode node pools → EKS manages EC2 lifecycle
  Node Groups          → you configure ASG + instance type
  Fargate profiles     → serverless, no nodes

Storage (CSI driver add-ons required):
  EBS CSI driver → EBS persistent volumes
  EFS CSI driver → EFS volumes (only option for Fargate)

Cleanup: delete node groups → delete cluster
```

---

**File: 2_EKS_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
