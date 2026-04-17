# 14. RDS Proxy

## Overview

**Amazon RDS Proxy** is a fully managed, serverless database proxy that sits between your application and RDS/Aurora. It pools connections, reduces failover time, and enforces IAM authentication — solving three distinct problems in one service.

---

## Why Use RDS Proxy?

| Problem | Without Proxy | With RDS Proxy |
|---------|--------------|----------------|
| Many app connections | Each app opens its own DB connection → CPU/RAM stress | Connections pooled → fewer connections to DB |
| Failover (Multi AZ / Aurora) | App must handle reconnect → slower recovery | Proxy handles failover transparently — up to **66% faster** |
| IAM enforcement | No built-in way to enforce IAM-only auth | Proxy enforces IAM authentication + stores credentials in Secrets Manager |

---

## Key Properties

| Property | Detail |
|----------|--------|
| **Type** | Fully managed, serverless, auto-scaling |
| **Availability** | Multi-AZ — highly available |
| **Network access** | **VPC only** — never publicly accessible |
| **Code changes** | None — just point app to proxy endpoint instead of DB endpoint |
| **Supported engines** | MySQL, PostgreSQL, MariaDB, SQL Server, Aurora MySQL, Aurora PostgreSQL |

---

## Architecture

```
Lambda functions (×1000)          EC2 app servers
         │                               │
         └──────────────┬────────────────┘
                        ▼
               RDS Proxy (VPC only)
               (pools + manages connections)
                        │
              ┌─────────┴──────────┐
              ▼                    ▼
        RDS Primary           RDS Standby
     (or Aurora master)     (failover target)
```

---

## Three Core Use Cases

### 1. Connection Pooling
- Reduces open connections and timeouts to the database
- Reduces CPU and RAM stress on the DB instance
- Critical when many short-lived clients connect simultaneously

### 2. Faster Failover
- RDS Proxy absorbs the failover event
- Application keeps its connection to the proxy — no reconnect needed
- Failover time reduced by **up to 66%** for both RDS and Aurora

### 3. IAM Authentication + Secrets Manager
- Enforces IAM-only authentication to the database
- Database credentials stored and rotated securely in **AWS Secrets Manager**
- No plaintext credentials in application code

---

## Lambda + RDS Proxy (Important Exam Pattern)

Lambda functions are ephemeral — they scale to hundreds or thousands of instances rapidly. Each function opening its own DB connection causes:
- Connection exhaustion on the DB
- Open connections left behind after function termination
- Timeouts and errors

**Solution:** Lambda → RDS Proxy → RDS/Aurora

The proxy pools and manages all Lambda connections, protecting the database.

---

## Best Practices

✓ **Always use RDS Proxy with Lambda** — Lambda's burst scaling will exhaust direct DB connections  
✓ **Use for any workload with many short-lived connections** — microservices, containers, serverless  
✓ **Enable IAM authentication via RDS Proxy** — removes credential management from application code  
✓ **Store DB credentials in Secrets Manager** — automatic rotation, no hardcoded passwords  
✓ **Remember: RDS Proxy is VPC-only** — never expose it publicly  

---

## SysOps Exam Focus

**Q1: "Your Lambda functions are connecting directly to an RDS database. During peak load, the database becomes unresponsive due to too many open connections. What is the recommended fix?"**
- A) Increase the RDS instance size
- B) Place an RDS Proxy between Lambda and RDS to pool connections
- C) Enable RDS Multi AZ to distribute connections
- D) Reduce Lambda concurrency limits
- **Answer: B** — RDS Proxy pools Lambda connections, preventing connection exhaustion on the database

**Q2: "What is the failover improvement provided by RDS Proxy for Multi AZ RDS instances?"**
- A) Eliminates failover entirely
- B) Reduces failover time by up to 66%
- C) Doubles the number of standby instances
- D) Moves the failover process to the application layer
- **Answer: B** — RDS Proxy handles the failover transparently, reducing application-visible failover time by up to 66%

**Q3: "You want to ensure that only IAM-authenticated principals can connect to your RDS database. What service enforces this?"**
- A) VPC Security Groups with IAM conditions
- B) RDS Proxy — it can enforce IAM authentication and store credentials in Secrets Manager
- C) AWS Config rule on RDS authentication settings
- D) Enable IAM database authentication directly on the RDS instance without a proxy
- **Answer: B** — RDS Proxy is the managed way to enforce IAM authentication and integrate with Secrets Manager for credential storage

**Q4: "Can an RDS Proxy be accessed from the public internet?"**
- A) Yes — if you enable public access during creation
- B) Yes — but only with SSL/TLS enabled
- C) No — RDS Proxy is accessible only from within the VPC
- D) Yes — via an internet gateway attached to the VPC
- **Answer: C** — RDS Proxy is never publicly accessible; it only accepts connections from within the VPC

**Q5: "What code changes are required to use RDS Proxy with an existing application?"**
- A) Add the AWS SDK RDS Proxy library and update the connection method
- B) No code changes — update the connection string to point to the proxy endpoint instead of the DB endpoint
- C) Enable proxy mode in the database driver configuration
- D) Add a middleware layer between the application and the database
- **Answer: B** — RDS Proxy is transparent to the application; only the endpoint in the connection string changes

---

## Quick Reference

```
RDS Proxy:
  Fully managed, serverless, auto-scaling, multi-AZ
  VPC only — never publicly accessible
  No code changes — update connection string to proxy endpoint

Supported: MySQL, PostgreSQL, MariaDB, SQL Server, Aurora MySQL/PostgreSQL

Three reasons to use:
  1. Connection pooling   → reduces DB CPU/RAM stress, fewer open connections
  2. Faster failover      → up to 66% faster for RDS and Aurora
  3. IAM enforcement      → IAM-only auth + credentials in Secrets Manager

Key use case:
  Lambda → RDS Proxy → RDS/Aurora
  (prevents connection exhaustion from Lambda burst scaling)
```

---

**File: 14_RDS_Proxy.md**
**Status: SysOps-focused, exam-ready, concise format**
