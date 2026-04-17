# 2. ElastiCache — Hands-On

## Overview

A walkthrough of creating a Redis OSS node-based ElastiCache cluster, reviewing key configuration options, and cleaning up.

---

## Step 1: Choose Engine

**Console:** ElastiCache → Create cluster

| Engine | Notes |
|--------|-------|
| **Valkey** | AWS-recommended Redis replacement |
| **Redis OSS** | Open-source Redis — same options as Valkey |
| **Memcached** | Simple sharding, no replication |

For this hands-on: **Redis OSS**

---

## Step 2: Deployment Option

| Option | Description |
|--------|-------------|
| **Serverless** | Fully managed, no node selection |
| **Node-based cluster** | Full control over configuration — use this to see all options |

Select: **Node-based cluster** → **Configure all settings** (not Easy Create)

---

## Step 3: Cluster Configuration

| Setting | Value |
|---------|-------|
| Cluster mode | **Disabled** — single shard, 1 primary + up to 5 read replicas |
| Cluster mode enabled | Multiple shards across multiple servers (horizontal sharding) |
| Cluster name | `DemoCluster` |
| Location | AWS Cloud (or AWS Outposts for on-premises) |
| Multi-AZ | Disabled (enable for production HA) |
| Auto-failover | Enabled |
| Engine version | Default |
| Port | 6379 (Redis default) |
| Node type | `cache.t2.micro` or `cache.t3.micro` (free tier eligible) |
| Number of replicas | 0 (use 1+ in production with Multi-AZ) |

---

## Step 4: Subnet Group

Create a new subnet group:

| Setting | Value |
|---------|-------|
| Name | `my-first-subnet-group` |
| VPC | Default VPC |
| Subnets | Auto-selected (or specify manually) |

---

## Step 5: Security and Encryption

| Setting | Options |
|---------|---------|
| Encryption at rest | Enable + specify KMS key |
| Encryption in transit | Enable to unlock access control options |
| Access control (if in-transit enabled) | **Redis AUTH** (password/token) or **User Group ACL** |
| Security groups | Control network access from application tier |

> If encryption in transit is disabled, Redis AUTH and user group ACL are not available.

---

## Step 6: Additional Options

| Option | Notes |
|--------|-------|
| Backup | Enable automatic backups with retention period |
| Maintenance window | Schedule minor version upgrades |
| Logs | Slow logs / engine logs → export to CloudWatch Logs |
| Tags | Standard AWS resource tagging |

---

## Step 7: Connect (After Creation)

After the cluster is available, connection details are found under the cluster:

| Endpoint Type | Use |
|--------------|-----|
| **Primary endpoint** | Write operations |
| **Reader endpoint** | Read operations (if replicas exist) |

> Connecting to ElastiCache requires application code — there is no console-based query interface like SQLectron for RDS.

---

## Step 8: Cleanup

**Console:** ElastiCache → Select cluster → **Actions** → **Delete**

- Skip final backup for demo clusters
- Type the cluster name → confirm deletion

---

## Best Practices

✓ **Enable Multi-AZ with at least 1 replica for production** — provides automatic failover  
✓ **Enable encryption in transit + Redis AUTH** — prevents unauthorized cache access  
✓ **Use a subnet group** — controls which VPC subnets ElastiCache can run in  
✓ **Export slow logs to CloudWatch Logs** — identify slow cache operations  
✓ **Use cluster mode enabled for large datasets** — horizontal sharding across multiple shards  

---

## SysOps Exam Focus

**Q1: "You want to run ElastiCache on your on-premises infrastructure alongside AWS. What option allows this?"**
- A) ElastiCache Serverless with VPN connectivity
- B) AWS Outposts — ElastiCache can be deployed on Outposts hardware in your data center
- C) ElastiCache does not support on-premises deployment
- D) Deploy ElastiCache in a VPC with a Direct Connect attachment
- **Answer: B** — AWS Outposts extends AWS services including ElastiCache to on-premises environments

**Q2: "What is the difference between ElastiCache cluster mode disabled and cluster mode enabled?"**
- A) Cluster mode disabled uses Memcached; enabled uses Redis
- B) Cluster mode disabled = single shard with one primary and up to 5 replicas; enabled = multiple shards for horizontal data partitioning
- C) Cluster mode enabled disables replication
- D) There is no functional difference — only a naming convention
- **Answer: B** — Cluster mode enabled allows data to be sharded across multiple primaries; disabled keeps all data on one primary with optional read replicas

**Q3: "How do you enable access control (password authentication) for a Redis ElastiCache cluster?"**
- A) Set a password in the subnet group configuration
- B) Enable encryption in transit — this unlocks Redis AUTH and User Group ACL options
- C) Add an IAM policy to the ElastiCache cluster
- D) Access control is always enabled by default
- **Answer: B** — Encryption in transit must be enabled before Redis AUTH or User Group ACL become available

---

## Quick Reference

```
ElastiCache Redis creation:
  Engine: Redis OSS (or Valkey)
  Deployment: Node-based cluster → Configure all settings
  Cluster mode: Disabled (1 shard) or Enabled (multiple shards)
  Node type: cache.t2.micro / cache.t3.micro (free tier)
  Replicas: 0 for dev, 1+ for production
  Multi-AZ: disable for dev, enable for production

Security:
  Encryption at rest: KMS key
  Encryption in transit: required for Redis AUTH / User Group ACL
  Security group: restrict to app tier

Endpoints:
  Primary endpoint → writes
  Reader endpoint  → reads (if replicas exist)

Cleanup: Actions → Delete → skip backup → confirm name
```

---

**File: 2_ElastiCache_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
