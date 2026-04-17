# 13. Aurora Serverless, Global Database, and CloudWatch Metrics

## Overview

Aurora offers two advanced deployment options — **Serverless** (auto-scaling compute with no capacity planning) and **Global Database** (multi-region replication with sub-second lag). Key CloudWatch metrics help monitor replica lag and connection health.

---

## Aurora Serverless

| Property | Detail |
|----------|--------|
| **Scaling** | Automatic — based on actual usage (ACUs scale up/down) |
| **Best for** | Infrequent, intermittent, or unpredictable workloads |
| **Capacity planning** | None required |
| **Pricing** | Pay per second of actual usage |
| **Infrastructure** | Proxy fleet managed by Aurora — transparent to the client |

### Architecture

```
Client application
        ↓
Aurora Proxy Fleet (managed by Aurora)
        ↓
Aurora DB instances (scale in/out automatically)
        ↓
Shared storage volume
```

- The proxy fleet handles connection routing
- Client always connects to the same endpoint — scaling is invisible

---

## Aurora Global Database

| Property | Detail |
|----------|--------|
| **Primary region** | 1 — handles all reads and writes |
| **Secondary regions** | Up to **10** — read-only |
| **Replication lag** | **< 1 second** from primary to secondary |
| **Read replicas per secondary** | Up to **16** |
| **Disaster recovery RTO** | **< 1 minute** — promote a secondary to primary |
| **Use case** | Global low-latency reads + cross-region DR |

### Architecture

```
Primary Region (reads + writes)
        │
        │  replication < 1 second
        ↓
Secondary Region 1 (read-only) ──► local app reads
Secondary Region 2 (read-only) ──► local app reads
...up to 10 secondary regions

Disaster recovery:
  Promote secondary → primary in < 1 minute
```

> Writes always go to the primary region — secondary regions are read-only until promoted.

---

## Aurora CloudWatch Metrics

| Metric | What It Measures | Why It Matters |
|--------|-----------------|----------------|
| **AuroraReplicaLag** | Lag (ms) of a specific replica behind the primary | High lag = eventual consistency issues for users on that replica |
| **AuroraReplicaLagMaximum** | Maximum lag across all replicas in the cluster | Shows worst-case consistency gap |
| **AuroraReplicaLagMinimum** | Minimum lag across all replicas in the cluster | Shows best-case consistency |
| **DatabaseConnections** | Current number of client connections | High connections may require RDS Proxy |
| **InsertLatency** | Average time to perform an INSERT operation | Indicates write performance issues |

### Replica Lag Impact

```
High AuroraReplicaLag
        ↓
Different users reading from different replicas
get different versions of the same data
        ↓
Eventual consistency problem — users see stale reads
```

---

## Best Practices

✓ **Use Aurora Serverless for dev/test or spiky workloads** — eliminates over-provisioning costs  
✓ **Use Aurora Global Database for cross-region DR** — sub-1-minute RTO with < 1 second replication lag  
✓ **Alert on AuroraReplicaLagMaximum** — a rising maximum lag signals that some users will see stale data  
✓ **Monitor DatabaseConnections** — pair with RDS Proxy if connection count grows too high  
✓ **Secondary regions in Global Database are read-only** — writes must go to the primary region  

---

## SysOps Exam Focus

**Q1: "Your application has unpredictable traffic — heavy during business hours, near-zero overnight. You want to minimize database costs without managing capacity. What Aurora option is best?"**
- A) Aurora with a large instance class and Read Replicas
- B) Aurora Serverless — automatically scales compute based on usage, billed per second
- C) Aurora Global Database with a secondary region for overnight traffic
- D) Aurora with scheduled instance resizing using Lambda
- **Answer: B** — Aurora Serverless scales to zero during idle periods and charges per second, making it ideal for unpredictable workloads

**Q2: "Your Aurora Global Database primary is in us-east-1 with a secondary in eu-west-1. A user in Europe writes data. Where does the write go?"**
- A) eu-west-1 — the nearest region handles writes
- B) us-east-1 — all writes must go to the primary region
- C) Both regions simultaneously for consistency
- D) Whichever region responds first
- **Answer: B** — Secondary regions in Aurora Global Database are read-only; writes always go to the primary region

**Q3: "An Aurora cluster has multiple read replicas. You notice that AuroraReplicaLagMaximum is very high. What is the user impact?"**
- A) Writes are being rejected by the primary
- B) Users reading from the lagging replica may see stale data — eventual consistency issue
- C) The primary instance is approaching its IOPS limit
- D) The writer endpoint is not routing correctly
- **Answer: B** — High replica lag means the replica has not caught up with the primary; users on that replica see outdated data

**Q4: "What is the RTO for failing over to a secondary region in Aurora Global Database?"**
- A) < 30 seconds
- B) < 1 minute
- C) < 5 minutes
- D) Depends on database size
- **Answer: B** — Aurora Global Database supports cross-region promotion with an RTO of less than 1 minute

**Q5: "How does Aurora Serverless handle scaling transparently to the application?"**
- A) The application must reconnect to a new endpoint as instances scale
- B) A proxy fleet managed by Aurora routes connections; scaling is invisible to the client
- C) The application polls the Aurora API for the current instance count
- D) Read replicas absorb extra load while the primary remains fixed
- **Answer: B** — The Aurora-managed proxy fleet abstracts the underlying instance changes; the client always connects to the same endpoint

---

## Quick Reference

```
Aurora Serverless:
  Auto-scales ACUs based on actual usage
  Pay per second | No capacity planning
  Best for: infrequent, spiky, unpredictable workloads
  Proxy fleet: managed by Aurora, transparent to client

Aurora Global Database:
  Primary regions: 1 (reads + writes)
  Secondary regions: up to 10 (read-only)
  Replication lag: < 1 second
  Read replicas per secondary: up to 16
  DR failover RTO: < 1 minute
  Writes → primary region only

CloudWatch metrics:
  AuroraReplicaLag        → lag of one replica
  AuroraReplicaLagMaximum → worst-case lag across all replicas
  AuroraReplicaLagMinimum → best-case lag across all replicas
  DatabaseConnections     → current client connections
  InsertLatency           → average INSERT time
```

---

**File: 13_Aurora_Serverless_Global_Monitoring.md**
**Status: SysOps-focused, exam-ready, concise format**
