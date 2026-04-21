# 4. Amazon EKS — Cluster Insights and Autoscaling

## Overview

EKS provides **Cluster Insights** for detecting misconfigurations and upgrade issues, and offers three generations of **cluster autoscaling** — from the legacy Cluster Autoscaler to the modern EKS Auto Mode.

---

## EKS Cluster Insights

A built-in feature that detects issues and provides recommendations.

| Insight Type | Purpose |
|-------------|---------|
| **Configuration Insights** | Identify misconfigurations in your EKS cluster |
| **Upgrade Insights** | Identify issues that could block a Kubernetes version upgrade |

### Cluster Upgrade Process

1. Review **Upgrade Insights** in the EKS console — identify any blocking issues
2. Update the **cluster control plane** (via console or CLI)
3. Update **cluster components** (add-ons, node groups)

> Both CLI and console are equally supported for upgrades.

---

## EKS Autoscaling — Three Generations

### 1. Cluster Autoscaler (Legacy)

| Property | Detail |
|----------|--------|
| **Mechanism** | Adjusts number of nodes in the cluster |
| **Node type** | Homogeneous — all nodes in an ASG are the **same instance type** |
| **Scaling trigger** | Pods can't be scheduled due to insufficient capacity |
| **Management** | Self-managed — you deploy and configure it |

```
Pod can't be scheduled (no capacity)
        ↓
Cluster Autoscaler detects
        ↓
ASG scales out → new node (same instance type)
        ↓
Pod scheduled
```

---

### 2. Karpenter (Open Source)

| Property | Detail |
|----------|--------|
| **Mechanism** | Launches right-sized EC2 instances in response to load |
| **Node types** | Heterogeneous — mix of instance types, Spot, Fargate |
| **Speed** | Under **1 minute** to provision new nodes |
| **Flexibility** | Selects the best instance type for the pending workload |
| **Management** | Self-managed open-source project |

---

### 3. EKS Auto Mode (Modern — Recommended)

| Property | Detail |
|----------|--------|
| **Mechanism** | AWS-managed version of Karpenter |
| **Node types** | Heterogeneous — mix of resource types automatically selected |
| **Management** | Fully managed — no need to deploy or configure Karpenter |
| **Speed** | Under 1 minute (same as Karpenter) |
| **Use case** | Recommended for new EKS deployments |

> EKS Auto Mode = AWS-managed Karpenter. You get Karpenter's flexibility without the operational overhead.

---

## Comparison Table

| | Cluster Autoscaler | Karpenter | EKS Auto Mode |
|-|--------------------|-----------|---------------|
| **Generation** | Legacy | Modern (OSS) | Modern (managed) |
| **Node types** | Same instance type per ASG | Any type, Spot, Fargate | Any type, Spot, Fargate |
| **Speed** | Slower | < 1 minute | < 1 minute |
| **Management** | Self-managed | Self-managed | AWS-managed |
| **Recommended** | No | Yes (if self-managing) | **Yes (preferred)** |

---

## Best Practices

✓ **Use EKS Auto Mode for new clusters** — AWS manages Karpenter, no extra setup required  
✓ **Review Cluster Insights before upgrading** — prevents failed upgrades from undetected issues  
✓ **Follow the upgrade sequence** — Insights → control plane → components; skipping steps causes failures  
✓ **Prefer heterogeneous node scaling** — Karpenter/Auto Mode reduces cost by selecting optimal instance types  

---

## SysOps Exam Focus

**Q1: "Before upgrading your EKS cluster to a new Kubernetes version, what should you do first?"**
- A) Update the node groups immediately, then the control plane
- B) Review EKS Cluster Insights — specifically Upgrade Insights — to identify any issues that could block the upgrade
- C) Delete and recreate the cluster on the new version
- D) Enable Karpenter before the upgrade
- **Answer: B** — Upgrade Insights surface blocking issues before you begin; always review before touching the control plane

**Q2: "Your EKS cluster uses Cluster Autoscaler. Pods are pending. What is a known limitation of Cluster Autoscaler compared to Karpenter?"**
- A) Cluster Autoscaler cannot scale below the minimum ASG size
- B) Cluster Autoscaler scales nodes of the same instance type per ASG — it cannot select right-sized or mixed instance types
- C) Cluster Autoscaler does not work with EC2 instances
- D) Cluster Autoscaler requires EKS Auto Mode to function
- **Answer: B** — Cluster Autoscaler is homogeneous per ASG; Karpenter/Auto Mode can pick optimal instance types dynamically

**Q3: "What is EKS Auto Mode?"**
- A) A feature that automatically upgrades the Kubernetes version
- B) An AWS-managed version of Karpenter that automatically scales the cluster with right-sized instances in under a minute
- C) A deployment mode that removes the need for a cluster control plane
- D) A Fargate-only mode for EKS clusters
- **Answer: B** — EKS Auto Mode is AWS-managed Karpenter; you get flexible, fast autoscaling without deploying Karpenter yourself

**Q4: "Which EKS Cluster Insight type would you use to find misconfigurations in your running cluster?"**
- A) Upgrade Insights
- B) Configuration Insights
- C) Performance Insights
- D) Security Insights
- **Answer: B** — Configuration Insights identifies misconfigurations; Upgrade Insights identifies issues blocking a version upgrade

---

## Quick Reference

```
EKS Cluster Insights:
  Configuration Insights → detect misconfigurations
  Upgrade Insights       → detect issues blocking Kubernetes upgrades

Upgrade sequence:
  1. Review Upgrade Insights
  2. Update control plane
  3. Update cluster components

EKS Autoscaling evolution:
  Cluster Autoscaler (legacy)
    → scales ASG, same instance type, slower
  Karpenter (OSS)
    → right-sized instances, mixed types, Spot, Fargate, <1 min
  EKS Auto Mode (recommended)
    → AWS-managed Karpenter, no deployment needed, <1 min

Exam tip:
  Auto Mode = managed Karpenter = preferred for new deployments
  Cluster Autoscaler = legacy, homogeneous nodes only
```

---

**File: 4_EKS_Cluster_Insights_and_Autoscaling.md**
**Status: SysOps-focused, exam-ready, concise format**
