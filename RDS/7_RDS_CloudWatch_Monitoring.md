# 7. RDS CloudWatch Monitoring

## Overview

RDS integrates with CloudWatch at two levels: **basic metrics** from the hypervisor (default) and **enhanced monitoring** from an agent running on the DB instance (opt-in). The exam tests your ability to interpret these metrics for troubleshooting.

---

## Basic CloudWatch Metrics (Hypervisor-Level)

Available by default — no configuration required.

| Metric | What It Tells You |
|--------|------------------|
| **DatabaseConnections** | Number of active client connections |
| **SwapUsage** | Memory pressure — high swap = instance needs more RAM |
| **ReadIOPS / WriteIOPS** | I/O operations per second — high values may indicate EBS IOPS limit reached |
| **ReadLatency / WriteLatency** | Storage latency — high values indicate a storage bottleneck |
| **ReadThroughput / WriteThroughput** | Data transfer rate to/from storage |
| **DiskQueueDepth** | Operations waiting to be executed — high = storage saturation |
| **FreeStorageSpace** | Remaining storage — triggers Auto Scaling if enabled |
| **CPUUtilization** | CPU load on the DB instance |

---

## Enhanced Monitoring (Agent-Level)

An agent runs **inside the DB instance** and reports metrics at up to **1-second granularity**.

| Property | Detail |
|----------|--------|
| **Source** | Agent on the DB instance (not the hypervisor) |
| **Granularity** | 1s, 5s, 10s, 15s, 30s, or 60s |
| **New metrics** | 50+ CPU, memory, file system, and disk I/O metrics |
| **Extra view** | OS process list — shows per-process CPU and memory usage |
| **IAM role** | Requires a monitoring role (RDS can auto-create a default role) |

### Enable Enhanced Monitoring

**Console:** RDS → DB → Modify → scroll to Monitoring → Enable enhanced monitoring → select granularity → Continue → Apply immediately

### View Enhanced Monitoring

**Console:** RDS → DB → Monitoring tab → dropdown → **Enhanced monitoring**

For OS process list: Monitoring tab → dropdown → **OS process list**

---

## Basic vs Enhanced Monitoring

| | Basic (CloudWatch) | Enhanced Monitoring |
|-|--------------------|---------------------|
| **Source** | Hypervisor | Agent on DB instance |
| **Default** | Yes | No — must enable |
| **Granularity** | 1 minute | Up to 1 second |
| **Metric count** | ~10 core metrics | 50+ metrics |
| **Process-level detail** | No | Yes (OS process list) |

---

## Troubleshooting with Metrics

| Symptom | Likely Metric | Action |
|---------|--------------|--------|
| Slow queries | High ReadLatency / WriteLatency | Check EBS type/IOPS, consider io1/io2 |
| Storage full | Low FreeStorageSpace | Enable or increase Auto Scaling threshold |
| IOPS limit hit | ReadIOPS / WriteIOPS at ceiling | Upgrade EBS volume type or increase provisioned IOPS |
| I/O backlog | High DiskQueueDepth | Storage is saturated — reduce concurrency or upgrade storage |
| Too many connections | High DatabaseConnections | Use RDS Proxy or increase max_connections |
| High CPU | CPUUtilization | Scale up instance or offload reads to replica |

---

## Best Practices

✓ **Enable enhanced monitoring for production** — per-second granularity catches transient spikes  
✓ **Watch DiskQueueDepth** — a sustained queue depth > 1 indicates storage is a bottleneck  
✓ **Set CloudWatch alarms on FreeStorageSpace** — get notified before storage runs out  
✓ **Use OS process list to identify runaway processes** — enhanced monitoring shows per-process resource use  
✓ **Monitor DatabaseConnections alongside CPU** — high connections + high CPU = need for RDS Proxy  

---

## SysOps Exam Focus

**Q1: "Your RDS database is experiencing high read latency. Which CloudWatch metric should you investigate first?"**
- A) DatabaseConnections
- B) SwapUsage
- C) ReadLatency and DiskQueueDepth
- D) FreeStorageSpace
- **Answer: C** — High ReadLatency combined with high DiskQueueDepth indicates a storage I/O bottleneck

**Q2: "You need per-second CPU and memory metrics for your RDS instance, including a breakdown by OS process. What must you enable?"**
- A) CloudWatch detailed monitoring
- B) RDS Performance Insights
- C) Enhanced monitoring with an appropriate granularity setting
- D) CloudTrail logging for RDS API calls
- **Answer: C** — Enhanced monitoring deploys an agent on the instance and provides per-second metrics and OS process details

**Q3: "An RDS instance has high DiskQueueDepth. What does this indicate?"**
- A) The database has too many client connections
- B) The instance is running low on memory
- C) Storage I/O is saturated — operations are queuing because the storage layer cannot keep up
- D) The automated backup is running and consuming disk resources
- **Answer: C** — DiskQueueDepth represents pending I/O operations; a sustained high value means the storage is a bottleneck

**Q4: "What is the key difference between basic CloudWatch metrics and enhanced monitoring for RDS?"**
- A) Basic metrics are free; enhanced monitoring is not available for RDS
- B) Basic metrics come from the hypervisor; enhanced monitoring comes from an agent inside the DB instance and provides 50+ metrics at up to 1-second granularity
- C) Enhanced monitoring only works with Aurora, not standard RDS
- D) Basic metrics include OS process details; enhanced monitoring is limited to CPU and memory
- **Answer: B** — Enhanced monitoring uses an in-instance agent, giving deeper visibility than hypervisor-level metrics

---

## Quick Reference

```
Basic CloudWatch (hypervisor, default, 1-min):
  DatabaseConnections, SwapUsage
  ReadIOPS/WriteIOPS, ReadLatency/WriteLatency
  ReadThroughput/WriteThroughput
  DiskQueueDepth, FreeStorageSpace, CPUUtilization

Enhanced Monitoring (agent, opt-in, up to 1-sec):
  50+ metrics: CPU, memory, file system, disk I/O
  OS process list (per-process CPU/memory)
  Enable: Modify DB → Monitoring → Enhanced monitoring
  View: Monitoring tab → dropdown → Enhanced monitoring

Troubleshooting shortcuts:
  High latency        → check DiskQueueDepth + IOPS
  Low FreeStorage     → increase threshold or manual scale
  High DiskQueueDepth → storage bottleneck → upgrade EBS
  High connections    → consider RDS Proxy
```

---

**File: 7_RDS_CloudWatch_Monitoring.md**
**Status: SysOps-focused, exam-ready, concise format**
