# 3. Amazon EKS — Logging and Metrics

## Overview

EKS logging and metrics have two separate layers: the **control plane** (EKS itself) and the **data plane** (nodes and containers). Each layer uses a different mechanism to send data to CloudWatch.

---

## Control Plane Logging

Logs from EKS itself — native integration with CloudWatch Logs.

| Log Type | What It Captures |
|----------|-----------------|
| **API server** | All Kubernetes API requests |
| **Audit** | Who did what, when — security and compliance |
| **Authenticator** | IAM-to-Kubernetes identity mapping |
| **Controller manager** | Cluster state reconciliation |
| **Scheduler** | Pod placement decisions |

- Enabled per log type — you choose which to send
- Destination: **CloudWatch Logs**

---

## Node and Container Logging

Logs from workloads running on EKS nodes require a log driver.

| Component | Purpose | Destination |
|-----------|---------|-------------|
| **Fluent Bit** (preferred) | Collect and route container logs | CloudWatch Logs |
| **Fluentd** | Alternative log driver | CloudWatch Logs |

> Fluent Bit or Fluentd must be deployed as a DaemonSet or sidecar to collect logs from nodes and containers.

---

## Node and Container Metrics

| Component | Purpose | Destination |
|-----------|---------|-------------|
| **CloudWatch Agent** | Collect node and container metrics (CPU, memory, disk, network) | CloudWatch Metrics |

> The CloudWatch Agent provides **metrics**. Fluent Bit/Fluentd provides **logs**. Both are required for full observability.

---

## Architecture Summary

```
EKS Control Plane
  └── API server, audit, authenticator,
      controller manager, scheduler logs
              ↓
        CloudWatch Logs (native integration)

EKS Data Plane (nodes + containers)
  ├── CloudWatch Agent → CloudWatch Metrics (CPU, memory, etc.)
  └── Fluent Bit / Fluentd → CloudWatch Logs (container logs)

CloudWatch Logs + CloudWatch Metrics
              ↓
    CloudWatch Container Insights
    (dashboard: nodes, pods, tasks, services)
```

---

## CloudWatch Container Insights

A dashboarding facility built on top of CloudWatch that aggregates EKS observability data.

| Scope | What You See |
|-------|-------------|
| **Nodes** | CPU, memory, network per EC2 node |
| **Pods** | Resource utilization per pod |
| **Tasks** | Task-level metrics |
| **Services** | Service-level health and performance |

> Container Insights requires both the **CloudWatch Agent** (metrics) and **Fluent Bit/Fluentd** (logs) to be deployed.

---

## Best Practices

✓ **Enable all control plane log types** — audit and API server logs are critical for security investigations  
✓ **Deploy Fluent Bit over Fluentd** — lower resource footprint, AWS-native integration  
✓ **Deploy CloudWatch Agent as a DaemonSet** — ensures every node ships metrics  
✓ **Use Container Insights** — single pane of glass for nodes, pods, and services  

---

## SysOps Exam Focus

**Q1: "You want to capture logs from the Kubernetes API server and audit trail in your EKS cluster. Where do these logs go?"**
- A) They are stored in the EKS node's local disk
- B) Enable control plane logging — API server and audit logs stream to CloudWatch Logs natively
- C) Deploy Fluent Bit on each node to capture control plane logs
- D) Install the CloudWatch Agent on the control plane
- **Answer: B** — EKS control plane logs have native CloudWatch Logs integration; Fluent Bit is for workload/container logs

**Q2: "You need to collect logs from containers running on EKS nodes and send them to CloudWatch Logs. What must you deploy?"**
- A) The CloudWatch Agent — it collects both metrics and logs
- B) Fluent Bit or Fluentd as a log driver on each node
- C) Enable control plane logging in the EKS console
- D) Install the ECS Agent on each node
- **Answer: B** — Container/node logs require Fluent Bit or Fluentd; the CloudWatch Agent only provides metrics

**Q3: "What is CloudWatch Container Insights used for in an EKS environment?"**
- A) It replaces the need for Fluent Bit and the CloudWatch Agent
- B) It provides dashboards for monitoring nodes, pods, tasks, and services using data from CloudWatch Metrics and Logs
- C) It is a Kubernetes-native tool that runs inside the cluster
- D) It only monitors ECS clusters, not EKS
- **Answer: B** — Container Insights is a CloudWatch dashboarding layer that visualizes data already flowing into CloudWatch from the Agent and Fluent Bit

**Q4: "What is the difference between deploying the CloudWatch Agent and Fluent Bit on EKS nodes?"**
- A) They are interchangeable — either one collects both metrics and logs
- B) CloudWatch Agent → metrics; Fluent Bit → logs. Both are needed for full observability
- C) Fluent Bit collects metrics; CloudWatch Agent collects logs
- D) Only one can be deployed at a time on a node
- **Answer: B** — Metrics and logs use separate tools; the CloudWatch Agent handles metrics and Fluent Bit handles log routing

---

## Quick Reference

```
EKS Observability — two layers:

Control plane (EKS infrastructure):
  Logs: API server, audit, authenticator,
        controller manager, scheduler
  Destination: CloudWatch Logs (native, no agent needed)

Data plane (nodes + containers):
  Metrics: CloudWatch Agent → CloudWatch Metrics
  Logs:    Fluent Bit / Fluentd → CloudWatch Logs

CloudWatch Container Insights:
  Unified dashboard — nodes, pods, tasks, services
  Requires: CloudWatch Agent + Fluent Bit both deployed

Exam tip:
  Control plane logs → native integration (no agent)
  Container logs     → Fluent Bit or Fluentd required
  Container metrics  → CloudWatch Agent required
```

---

**File: 3_EKS_Logging_and_Metrics.md**
**Status: SysOps-focused, exam-ready, concise format**
