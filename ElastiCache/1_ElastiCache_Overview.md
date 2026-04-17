# 1. ElastiCache Overview

## Overview

**Amazon ElastiCache** is a fully managed in-memory caching service for **Redis** and **Memcached**. It provides high-performance, low-latency data retrieval to reduce load on relational databases for read-intensive workloads. Unlike RDS, using ElastiCache requires **application code changes**.

---

## What Is a Cache?

- An **in-memory database** — stores data in RAM for sub-millisecond access
- Common query results are cached so the database is not hit every time
- Helps make applications **stateless** by storing session data centrally

> AWS manages OS patching, optimization, setup, monitoring, failure recovery, and backups — same model as RDS.

---

## ElastiCache Architectures

### 1. Database Query Caching

```
Application
    │
    ├──► ElastiCache (check cache first)
    │         │
    │    Cache HIT → return data directly (fast, no DB query)
    │         │
    │    Cache MISS → query RDS database
    │                      │
    └──────────────────────► Write result back to cache for next time
```

- Reduces read load on RDS
- Requires a **cache invalidation strategy** to prevent stale data

### 2. Session State (Stateless Applications)

```
User logs in → App writes session data to ElastiCache
        ↓
User hits a different app instance
        ↓
App reads session from ElastiCache → user stays logged in
```

- Decouples session state from the application instance
- Enables horizontal scaling — any instance can serve any user

---

## Redis vs Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Multi-AZ / Auto-failover** | Yes | No |
| **Read replicas** | Yes — scale reads + HA | No replication |
| **Data durability** | AOF persistence | No (data lost on failure) |
| **Backup and restore** | Yes | Serverless version only |
| **Data structures** | Strings, sets, sorted sets, hashes, lists | Simple key-value only |
| **Leaderboards** | Yes — sorted sets | No |
| **Architecture** | Primary + replicas | Multi-node sharding (horizontal partitioning) |
| **Multi-threaded** | No (single-threaded) | Yes |
| **Use case** | HA cache, sessions, leaderboards, pub/sub | Simple high-performance cache, multi-threaded workloads |

### Redis — Think Replication

```
Redis Primary ──replicated──► Redis Replica
(HA, durability, read scaling)
```

### Memcached — Think Sharding

```
Node 1 | Node 2 | Node 3 | Node 4
(data partitioned across nodes, no replication)
```

---

## Key Requirement

> **ElastiCache requires application code changes.** You must update your application to:
> 1. Query the cache before the database
> 2. Write results back to the cache on a cache miss
> 3. Implement a cache invalidation strategy

This is not a drop-in replacement — it is an architectural pattern.

---

## Best Practices

✓ **Use Redis when you need HA, durability, or complex data structures** — sorted sets for leaderboards, AOF for persistence  
✓ **Use Memcached for simple, high-throughput caching with multi-threaded performance** — acceptable to lose cache on failure  
✓ **Implement cache invalidation carefully** — stale data in cache causes incorrect application behavior  
✓ **Use ElastiCache for session storage** — decouples session from app instances, enables stateless scaling  
✓ **Do not query the database if the cache has the answer** — cache hits save DB CPU, RAM, and IOPS  

---

## SysOps Exam Focus

**Q1: "Your application performs the same database queries repeatedly, causing high CPU on RDS. What is the recommended solution to reduce database load?"**
- A) Enable RDS Read Replicas
- B) Add Amazon ElastiCache in front of RDS to cache frequent query results
- C) Increase the RDS instance size
- D) Enable RDS Storage Auto Scaling
- **Answer: B** — ElastiCache caches common query results, reducing repetitive reads against the database

**Q2: "Your application runs on multiple EC2 instances behind a load balancer. Users get logged out when requests are routed to a different instance. What is the solution?"**
- A) Enable sticky sessions on the load balancer
- B) Store session data in Amazon ElastiCache so any instance can retrieve the session
- C) Use EC2 instance store for session data
- D) Use RDS to store session tokens
- **Answer: B** — ElastiCache centralizes session state, making the application stateless and enabling any instance to serve any user

**Q3: "You need a caching layer that supports high availability with automatic failover, read replicas, and sorted sets for a real-time leaderboard. Which ElastiCache engine should you choose?"**
- A) Memcached — it supports sharding for leaderboard data
- B) Redis — it supports Multi-AZ failover, read replicas, and sorted sets natively
- C) Either — both support sorted sets and HA
- D) Neither — use DynamoDB for leaderboards
- **Answer: B** — Redis supports sorted sets (ideal for leaderboards), Multi-AZ with auto-failover, and read replicas; Memcached does not

**Q4: "What is the key operational difference between ElastiCache and RDS from an application perspective?"**
- A) ElastiCache requires no configuration; RDS requires heavy configuration
- B) ElastiCache requires application code changes to implement cache logic; RDS works as a drop-in database
- C) RDS requires code changes; ElastiCache integrates automatically
- D) Both require the same level of application integration
- **Answer: B** — ElastiCache is not transparent to the application; you must code the cache-check, cache-miss, and invalidation logic

---

## Quick Reference

```
ElastiCache:
  Managed in-memory cache: Redis or Memcached
  Reduces DB load for read-intensive workloads
  Requires application code changes

Cache pattern:
  1. App checks cache → cache hit → return data
  2. Cache miss → query DB → write result to cache
  3. Implement cache invalidation strategy

Session store pattern:
  Login → write session to ElastiCache
  Next request (any instance) → read session from ElastiCache

Redis vs Memcached:
  Redis     = HA, replication, persistence, sorted sets, pub/sub
  Memcached = simple sharding, multi-threaded, no replication, no persistence
```

---

**File: 1_ElastiCache_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
