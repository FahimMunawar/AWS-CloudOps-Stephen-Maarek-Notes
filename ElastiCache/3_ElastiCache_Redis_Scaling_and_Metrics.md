# 3. ElastiCache Redis — Scaling and Metrics

## Overview

Redis scaling strategies differ between **cluster mode disabled** (single shard) and **cluster mode enabled** (multiple shards). Key CloudWatch metrics help identify when and how to scale.

---

## Cluster Mode Disabled — Scaling

Single shard: one primary + up to 5 read replicas.

| Scaling Type | How | Notes |
|-------------|-----|-------|
| **Horizontal** | Add or remove read replicas | Max 5 replicas; scales read capacity |
| **Vertical** | Change to larger/smaller node type | One-click operation |

### Vertical Scaling — What Happens Internally

```
Request to change node type
        ↓
ElastiCache creates a NEW node group with the new instance type
        ↓
Data is replicated from old node group to new node group
        ↓
DNS update by ElastiCache → application transparently connects to new group
        ↓
Old node group removed
```

> Transparent to the application — no connection string change needed.

---

## Cluster Mode Enabled — Scaling

Multiple shards, each with its own primary and replicas.

### Two Scaling Modes

| Mode | Downtime | Use When |
|------|----------|----------|
| **Online scaling** | No downtime | Horizontal scaling, vertical scaling — performance may degrade briefly |
| **Offline scaling** | Cluster goes down | Node type changes, engine version upgrades — more configuration options |

### Horizontal Scaling (Online or Offline)

| Operation | Description |
|-----------|-------------|
| **Resharding** | Add or remove shards (scale out/in) |
| **Shard rebalancing** | Redistribute key space evenly across existing shards |

### Vertical Scaling (Online)

- Change node type across all shards (one-click)
- Happens behind the scenes — same internal DNS update process as cluster mode disabled

---

## Redis CloudWatch Metrics

| Metric | What It Measures | Action if High/Low |
|--------|-----------------|-------------------|
| **Evictions** | Non-expired items evicted to make room for new writes | Memory full — choose better eviction policy (e.g., LRU), use larger node, or add shards |
| **CPUUtilization** | CPU load on the Redis node | Scale up (larger node) or scale out (add nodes/shards) |
| **SwapUsage** | Swap memory used | Should not exceed **50 MB** — review memory settings if it does |
| **CurrConnections** | Current active connections to the cluster | High count may indicate app is not reusing connections — review connection behavior |
| **DatabaseMemoryUsagePercentage** | % of available memory in use | High % → increase node size or scale out |
| **NetworkBytesIn / NetworkBytesOut** | Network throughput | Baseline monitoring |
| **ReplicationBytes** | Data being replicated to replicas | Higher = more replication activity |
| **ReplicationLag** | Lag between primary and replica | Should be low — high lag = replicas are behind |

### Evictions — Key Detail

Evictions occur when the cache is full and must remove existing items to store new ones.

```
Cache memory full
        ↓
New write arrives
        ↓
ElastiCache evicts a non-expired item (based on eviction policy)
        ↓
High evictions = cache too small or wrong eviction policy
```

**Solutions:**
1. Choose an appropriate eviction policy (e.g., `allkeys-lru` — evict least recently used)
2. Use a larger node type (more memory)
3. Add more nodes/shards (cluster mode enabled)

---

## Scaling Decision Summary

| Scenario | Solution |
|----------|---------|
| High read load, cluster mode disabled | Add read replicas (up to 5) |
| Need more memory, single node | Vertical scale — larger node type |
| Large dataset exceeds single shard | Enable cluster mode + add shards |
| Need to change engine version | Offline scaling (cluster mode enabled) |
| High evictions | Larger node or add shards + better eviction policy |
| High CPU | Scale up (vertical) or scale out (horizontal) |
| Swap > 50 MB | Review memory configuration |

---

## Best Practices

✓ **Monitor Evictions** — the primary indicator that your cache is undersized  
✓ **Keep SwapUsage under 50 MB** — excessive swap indicates memory pressure  
✓ **Use online scaling when possible** — avoids downtime for horizontal and vertical changes  
✓ **Use offline scaling only when necessary** — required for engine version upgrades or certain node type changes  
✓ **Monitor ReplicationLag** — high lag means replicas are serving stale data  

---

## SysOps Exam Focus

**Q1: "Your ElastiCache Redis cluster (cluster mode disabled) is experiencing high read load. What is the correct horizontal scaling approach?"**
- A) Add more shards to the cluster
- B) Add read replicas — up to 5 are supported in cluster mode disabled
- C) Enable cluster mode and rebalance key space
- D) Increase the node type to handle more connections
- **Answer: B** — Cluster mode disabled scales horizontally by adding read replicas (max 5); sharding requires cluster mode enabled

**Q2: "You scale up your ElastiCache Redis node to a larger instance type. Your application connection string does not change, yet it connects to the new node automatically. Why?"**
- A) ElastiCache uses a static IP for all node types
- B) ElastiCache creates a new node group, replicates data, then performs a DNS update — the application reconnects transparently
- C) The application restarts and discovers the new endpoint via service discovery
- D) The old node is resized in-place without any DNS change
- **Answer: B** — Vertical scaling internally creates a new node group, replicates data, then updates DNS — transparent to the application

**Q3: "Your ElastiCache Redis cluster shows high Evictions. What does this indicate and what are the solutions?"**
- A) Too many client connections — reduce connection pool size
- B) Cache memory is full — non-expired items are being removed; solutions: use LRU eviction policy, increase node size, or add shards
- C) The replication lag is too high — add more replicas
- D) CPU is saturated — scale up the node type
- **Answer: B** — High evictions mean the cache is running out of memory; the correct response is to optimize the eviction policy or increase capacity

**Q4: "An ElastiCache Redis cluster in cluster mode enabled needs a major engine version upgrade. What type of scaling is required?"**
- A) Online scaling — no downtime required for version upgrades
- B) Offline scaling — the cluster must be taken down for engine version changes
- C) Vertical scaling via the console — applies without downtime
- D) Create a new cluster and use DNS failover
- **Answer: B** — Engine version upgrades require offline scaling, which takes the cluster down but allows more configuration changes

**Q5: "What does a high ReplicationLag metric indicate in an ElastiCache Redis cluster?"**
- A) The primary node is running out of memory
- B) Replicas are behind the primary — reads from replicas may return stale data
- C) Too many client connections are open
- D) The cluster is about to trigger an automatic failover
- **Answer: B** — High replication lag means replicas have not caught up with the primary; users reading from lagging replicas see outdated data

---

## Quick Reference

```
Cluster mode disabled scaling:
  Horizontal → add/remove read replicas (max 5)
  Vertical   → change node type (creates new group → replicate → DNS update)

Cluster mode enabled scaling:
  Online  → horizontal (resharding, rebalancing) + vertical (node type change)
  Offline → engine version upgrades, more config changes

Key metrics:
  Evictions              → cache full; use LRU policy or increase capacity
  CPUUtilization         → scale up or out if high
  SwapUsage              → must stay < 50 MB
  CurrConnections        → check app connection behavior if high
  DatabaseMemoryUsage%   → scale if approaching 100%
  ReplicationLag         → low = good; high = stale reads on replicas
```

---

**File: 3_ElastiCache_Redis_Scaling_and_Metrics.md**
**Status: SysOps-focused, exam-ready, concise format**
