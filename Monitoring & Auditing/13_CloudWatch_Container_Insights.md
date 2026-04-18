# 13 — CloudWatch Container Insights

## Overview

CloudWatch Container Insights collects, aggregates, and summarizes metrics and logs from containerized applications — no sidecar pattern required.

---

## Supported Services

- Amazon ECS
- AWS Fargate
- Amazon EKS
- Red Hat OpenShift on AWS

---

## Metrics Collected

CPU, memory, network utilization, disk usage, tasks, services

---

## Visibility Levels

| Level | What You Get |
|---|---|
| **Default** | Cluster-level and service-level metrics |
| **Enhanced Visibility** | Adds task-level and container-level metrics |

Enable Enhanced Visibility for: high resource usage, throttling issues, performance debugging.

---

## Enablement

- At the **account level**, or
- When **creating a cluster** (cluster-level container insights)

---

## Quick Reference

```
Container Insights = metrics + logs for ECS, Fargate, EKS, OpenShift
No sidecar needed
Default:  cluster + service level metrics
Enhanced: adds task + container level metrics
Use cases: high CPU/memory, throttling, performance issues
```
