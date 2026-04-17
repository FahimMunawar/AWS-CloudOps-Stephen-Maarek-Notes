# 8. RDS Performance Insights

## Overview

**RDS Performance Insights** visualizes database load and helps identify the root cause of performance problems. It shows what the database is waiting on and which SQL queries, users, or hosts are responsible — making it a powerful troubleshooting tool beyond basic CloudWatch metrics.

---

## Key Concept: Database Load

Database load = **number of active sessions** for the database engine at any given time.

```
Graph shows a line: Max vCPUs
        │
        ├── Load below line  → database has headroom
        └── Load above line  → database running at or over capacity
```

---

## Filter Dimensions (Slice By)

| Filter | What It Shows | Use Case |
|--------|--------------|----------|
| **Waits** | Resource the DB is waiting on: CPU, IO, locks | Identify the bottleneck type to decide whether to optimize CPU or IO |
| **SQL** | SQL statements consuming the most load | Find slow or expensive queries to optimize |
| **Host** | Application servers connected to the DB | Identify which server is hammering the database |
| **Users** | Database usernames consuming the most load | Find which application user/account is causing load |

---

## Waits — The Most Important Filter

Waits shows **what your database is blocked on**:

| Wait Type | Meaning |
|-----------|---------|
| `cpu` | Database is CPU-bound — consider scaling up instance |
| `io/...` | Database is I/O-bound — consider faster storage or read replicas |
| `lock/...` | Queries are blocked waiting for locks — optimize transactions |

> Use Waits to decide what kind of upgrade or optimization will have the most impact.

---

## Paid Tier Feature

| Feature | Detail |
|---------|--------|
| **Proactive Recommendations** | AI-generated suggestions to prevent future performance issues |
| **Extended retention** | Longer history of performance data (beyond the free 7-day window) |

---

## Enable Performance Insights

**Console:** RDS → DB → Modify → scroll to Performance Insights → enable → set retention period → Apply

> **Note:** Not supported on `db.t2` instance classes. Requires `db.t3` or larger (e.g., `db.m4.large`).

**View:** RDS → left sidebar → **Performance Insights** → select DB instance

---

## Performance Insights vs Enhanced Monitoring vs CloudWatch

| | CloudWatch | Enhanced Monitoring | Performance Insights |
|-|-----------|---------------------|---------------------|
| **Level** | Hypervisor | OS (agent) | Database engine |
| **Focus** | Infrastructure metrics | OS/process metrics | Query and session load |
| **Granularity** | 1 min (basic) | Up to 1 sec | Near real-time |
| **Key use** | Alarms on storage, CPU | Per-process resource usage | Slow query and bottleneck analysis |
| **Instance requirement** | All | All | Not db.t2 |

---

## Best Practices

✓ **Use Waits first** — it immediately shows whether the bottleneck is CPU, IO, or locking  
✓ **Use SQL filter to find expensive queries** — share findings with the dev team for query optimization  
✓ **Use Host filter to find misbehaving app servers** — useful when an app suddenly opens too many connections  
✓ **Enable on production non-t2 instances** — t2 instances do not support Performance Insights  
✓ **Combine with CloudWatch alarms** — Performance Insights for diagnosis, CloudWatch for alerting  

---

## SysOps Exam Focus

**Q1: "Your RDS database is running slowly. You want to identify which SQL queries are consuming the most database resources. What tool should you use?"**
- A) CloudWatch enhanced monitoring → OS process list
- B) RDS Performance Insights → filter by SQL
- C) RDS event subscriptions → slow query category
- D) AWS Trusted Advisor → RDS recommendations
- **Answer: B** — Performance Insights SQL filter shows which queries are driving the most database load

**Q2: "Performance Insights shows that the database load is above the Max vCPUs line and the Waits breakdown shows mostly io/ waits. What is the recommended action?"**
- A) Increase the number of Read Replicas
- B) The bottleneck is I/O — consider upgrading to a faster storage type (e.g., io1/io2) or adding read replicas to offload reads
- C) Increase the instance's vCPU count — the CPU is the bottleneck
- D) Reduce the backup retention period to free up I/O
- **Answer: B** — io/ waits indicate storage I/O is the bottleneck, not CPU; optimize storage or distribute read load

**Q3: "A team reports that their application is experiencing high database latency. You suspect one application server is sending too many queries. Which Performance Insights filter helps you identify it?"**
- A) Filter by Waits
- B) Filter by SQL
- C) Filter by Host
- D) Filter by Users
- **Answer: C** — The Host filter groups database load by the originating application server

**Q4: "You attempt to enable Performance Insights on a db.t2.micro RDS instance. What happens?"**
- A) It enables successfully with limited metrics
- B) Performance Insights is not supported on db.t2 instance classes
- C) You must first enable enhanced monitoring before enabling Performance Insights
- D) Performance Insights is only available for Aurora, not standard RDS
- **Answer: B** — Performance Insights requires db.t3 or larger; db.t2 is not supported

---

## Quick Reference

```
Performance Insights:
  Visualizes: database load (active sessions) vs. Max vCPUs line
  
  Filter by:
    Waits  → what resource is the bottleneck (CPU, IO, locks)
    SQL    → which queries are most expensive
    Host   → which app server is driving load
    Users  → which DB user is consuming the most

  Paid tier: Proactive Recommendations + extended retention

  Enable: Modify DB → Performance Insights → enable
  Requires: db.t3+ (not supported on db.t2)

  vs CloudWatch    → infrastructure-level metrics + alarms
  vs Enhanced Mon. → OS process-level metrics
  vs Perf Insights → query/session-level database analysis
```

---

**File: 8_RDS_Performance_Insights.md**
**Status: SysOps-focused, exam-ready, concise format**
