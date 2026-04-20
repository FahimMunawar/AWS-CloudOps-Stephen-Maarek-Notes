# 17 — AWS Compute Optimizer

## Overview

Reduces costs and improves performance by recommending optimal AWS resource configurations using ML analysis of CloudWatch metrics.

---

## Supported Resources

- EC2 instances
- EC2 Auto Scaling Groups
- EBS volumes
- Lambda functions
- ECS on Fargate
- Aurora and RDS databases
- Commercial software licenses

---

## Key Features

| Feature | Detail |
|---|---|
| **Recommendation types** | Over-provisioned, under-provisioned, or optimized |
| **Analysis method** | ML on resource configuration + CloudWatch utilization metrics |
| **Potential savings** | Up to **25%** cost reduction |
| **Export** | Recommendations exportable to **Amazon S3** |
| **IAM policy required** | `ComputeOptimizerReadOnlyAccess` |

---

## SysOps Exam Q&A

**Q: An EC2 instance is not appearing in Compute Optimizer recommendations. Why?**
A: The instance is too new — insufficient metrics data. Keep it running for at least **30 hours** so Compute Optimizer can collect enough CloudWatch data to generate recommendations.

**Q: What IAM policy is required to view Compute Optimizer recommendations?**
A: `ComputeOptimizerReadOnlyAccess` (AWS-managed policy).

---

## Quick Reference

```
Compute Optimizer: ML-based right-sizing recommendations
  Uses: CloudWatch metrics + resource configuration
  Savings: up to 25%
  Export: Amazon S3

Supported: EC2, ASG, EBS, Lambda, ECS Fargate, Aurora, RDS, commercial licenses

IAM: ComputeOptimizerReadOnlyAccess required to view recommendations

Instance not appearing → insufficient data → run instance for ≥30 hours
```
