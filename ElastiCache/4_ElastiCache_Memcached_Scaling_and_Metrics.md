# 4. ElastiCache Memcached — Scaling and Metrics

## Overview

Memcached scales horizontally by adding/removing nodes (up to 40) using **Auto Discovery** so clients find new nodes automatically. Vertical scaling requires manually creating a new cluster — Memcached has no backup system, so the new cluster starts empty.

---

## Memcached Cluster Size

- Supports **1 to 40 nodes** per cluster (soft limit)
- Data is **sharded** (partitioned) across nodes — no replication

---

## Horizontal Scaling

Add or remove cache nodes from the cluster.

```
Cluster: 2 nodes → add 2 more → Cluster: 4 nodes
        ↓
Auto Discovery notifies clients of new nodes
        ↓
Client connects to new nodes automatically
```

- No manual endpoint updates required — Auto Discovery handles it
- Newly added nodes start empty; data populates as cache misses occur

---

## Vertical Scaling

Memcached does **not** perform in-place node type changes. The process is manual:

```
Old cluster (small node type) — app connected
        ↓
Create NEW cluster with larger node type
        ↓
New cluster starts EMPTY (no backup/restore in Memcached)
        ↓
Update application to point to new cluster's endpoints
        ↓
App populates new cluster via cache misses
        ↓
Delete old cluster
```

> Unlike Redis, Memcached has no backup mechanism — data must be rebuilt organically by the application.

---

## Memcached Auto Discovery

Allows clients to automatically find all nodes in a cluster without manually tracking individual node endpoints.

### How It Works

```
Client → connects to Configuration Endpoint
              ↓
         Gets IP of Cache Node 1
              ↓
Client → connects to Cache Node 1
              ↓
         Cache Node 1 returns metadata:
         IP addresses of ALL nodes in the cluster
              ↓
Client now knows all node endpoints → connects to any node
```

- All cache nodes know about each other
- When nodes are added/removed, the metadata updates automatically
- Client only needs to know the **configuration endpoint** — not individual node endpoints

---

## Memcached CloudWatch Metrics

| Metric | What It Measures | Action if Problem |
|--------|-----------------|------------------|
| **Evictions** | Non-expired items evicted to make room for new writes | Use better eviction policy, larger nodes, or add more nodes |
| **CPUUtilization** | CPU load on the node | Scale up (larger node) or scale out (add nodes) |
| **SwapUsage** | Swap memory usage | Should stay low — review memory settings |
| **CurrConnections** | Active connections from application | High count may indicate connection leak — review app behavior |
| **FreeableMemory** | Free memory remaining on the host | Low = memory pressure → scale up or out |

---

## Redis vs Memcached — Scaling Comparison

| | Redis (Cluster Mode Disabled) | Memcached |
|-|-------------------------------|-----------|
| **Horizontal scaling** | Add/remove read replicas (max 5) | Add/remove nodes (max 40) |
| **Vertical scaling** | New node group → DNS update (transparent) | New cluster → manual endpoint update |
| **Backup on vertical scale** | Yes — data replicated to new group | No — new cluster starts empty |
| **Client discovery** | Writer/reader endpoint (managed) | Auto Discovery via configuration endpoint |
| **Data on new node** | Replicated from existing data | Empty — filled organically |

---

## Best Practices

✓ **Use Auto Discovery** — point the client to the configuration endpoint, not individual node endpoints  
✓ **Plan for empty cache on vertical scale** — the application must tolerate cold cache and DB fallback  
✓ **Monitor FreeableMemory** — when it drops low, scale before evictions begin  
✓ **Monitor Evictions** — high evictions mean cache memory is insufficient for the working set  
✓ **Add nodes for horizontal scaling** — cheaper than upgrading node type for read throughput  

---

## SysOps Exam Focus

**Q1: "You vertically scale a Memcached cluster to a larger node type. Your application connects to the new cluster but the cache hit rate is 0% for several minutes. Why?"**
- A) Auto Discovery has not propagated the new node endpoints yet
- B) Memcached has no backup or restore — the new cluster starts completely empty; the application must repopulate it
- C) The new node type uses a different cache eviction policy
- D) Vertical scaling in Memcached takes 20–30 minutes before data is available
- **Answer: B** — Memcached has no backup mechanism; after vertical scaling, the new cluster is empty and must be warmed up organically

**Q2: "A Memcached cluster has 4 nodes. You add 2 more nodes. How does the application discover the new nodes without a code change?"**
- A) The application must be restarted to trigger endpoint re-resolution
- B) Auto Discovery — the client connects to the configuration endpoint, which returns metadata about all nodes including the new ones
- C) Route 53 health checks update the DNS record with new node IPs
- D) The application polls the ElastiCache API for node changes
- **Answer: B** — Auto Discovery uses the configuration endpoint to return all current node metadata; clients stay up to date automatically

**Q3: "What is the maximum number of nodes in a Memcached cluster?"**
- A) 5
- B) 15
- C) 40 (soft limit)
- D) 100
- **Answer: C** — Memcached supports 1 to 40 nodes per cluster (soft limit, can be increased)

**Q4: "FreeableMemory on a Memcached node is trending toward zero. What should you do?"**
- A) Increase the SwapUsage threshold
- B) Scale up to a larger node type or add more nodes to distribute the data
- C) Enable AOF persistence to free up memory
- D) Reduce the backup retention period
- **Answer: B** — Low FreeableMemory means the cache is running out of RAM; increase node size or add nodes before evictions begin

---

## Quick Reference

```
Memcached cluster:
  1–40 nodes (soft limit)
  Data sharded across nodes — no replication, no HA

Horizontal scaling:
  Add/remove nodes → Auto Discovery updates clients automatically

Vertical scaling (manual process):
  1. Create new cluster with larger node type
  2. Update app to point to new cluster endpoint
  3. App repopulates cache (starts empty — no backup)
  4. Delete old cluster

Auto Discovery:
  Client → configuration endpoint → gets all node IPs
  All nodes know about each other → client always up to date

Key metrics:
  Evictions        → cache full; scale out or use better eviction policy
  CPUUtilization   → scale up or out
  SwapUsage        → keep low; review memory settings
  CurrConnections  → check app for connection leaks
  FreeableMemory   → scale before it hits zero
```

---

**File: 4_ElastiCache_Memcached_Scaling_and_Metrics.md**
**Status: SysOps-focused, exam-ready, concise format**
