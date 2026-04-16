# 6. EFS — Elastic File System Overview

## Overview

**Amazon EFS (Elastic File System)** is a managed, scalable network file system (NFS) that can be mounted simultaneously on **multiple EC2 instances across different Availability Zones**. Unlike EBS (one volume → one instance, AZ-locked), EFS is a shared, multi-AZ file system.

---

## EFS vs EBS — Key Differences

| Feature | EBS | EFS |
|---------|-----|-----|
| Protocol | Block storage | NFS (network file system) |
| Attach to multiple instances | No (one instance at a time) | **Yes — many instances, across AZs** |
| AZ scope | Locked to one AZ | **Multi-AZ** (or single-AZ option) |
| Capacity planning | Must provision size in advance | **No provisioning — auto-scales, pay per use** |
| OS support | Linux and Windows | **Linux only** (POSIX) |
| Cost | ~$0.10/GB (gp2) | ~$0.30/GB (**~3x gp2**), but pay-per-use |

---

## Architecture

```
                    ┌─── EC2 (us-east-1a) ──┐
                    │                        │
Security Group ──►  EFS File System  ◄──── EC2 (us-east-1b)
                    │                        │
                    └─── EC2 (us-east-1c) ──┘
```

- All instances connect to the **same file system** simultaneously
- Access controlled by a **security group** attached to EFS mount targets
- Encryption at rest via **KMS**

---

## Use Cases

- Content management and web serving
- Data sharing across instances
- WordPress and CMS platforms
- Any workload requiring shared, POSIX-compatible storage

---

## Performance Modes (Set at Creation)

| Mode | Latency | Throughput | Use Case |
|------|---------|------------|----------|
| **General Purpose** (default) | Low | Standard | Web servers, CMS, general workloads |
| **Max I/O** | Higher | Higher, highly parallel | Big data, media processing |

---

## Throughput Modes

| Mode | Behavior | Use Case |
|------|----------|----------|
| **Bursting** | Throughput scales with storage size (e.g., 1 TiB → 50 MiB/s + burst to 100 MiB/s) | Default, works for most workloads |
| **Provisioned** | Set throughput independently of storage size (e.g., 1 GiB/s for 1 TiB) | Predictable high-throughput needs |
| **Elastic** | Automatically scales throughput up/down based on workload (up to 3 GiB/s reads, 1 GiB/s writes) | Unpredictable workloads |

---

## Storage Classes and Lifecycle

### Storage Tiers

| Tier | Access Pattern | Cost |
|------|---------------|------|
| **Standard** | Frequently accessed | Highest storage cost |
| **Infrequent Access (EFS-IA)** | Less frequently accessed | Lower storage, retrieval fee |
| **Archive** | Rarely accessed (few times/year) | Lowest storage cost |

### Lifecycle Policies

Automatically move files between tiers based on access patterns:

```
File in EFS Standard
        ↓ (not accessed for 60 days — lifecycle policy)
Moved to EFS-IA
        ↓ (not accessed for longer)
Moved to EFS Archive
```

Using the right storage classes can achieve **up to 90% cost savings**.

---

## Availability Options

| Option | AZs | Use Case | IA Compatible |
|--------|-----|----------|---------------|
| **Standard (Multi-AZ)** | Multiple | Production workloads — resilient to AZ failure | Yes |
| **One Zone** | Single | Development, non-critical — cheaper | Yes (One Zone-IA) |

- One Zone still includes **backups** by default
- One Zone-IA combines single-AZ with infrequent access tier for maximum savings

---

## Best Practices

✓ **Use EFS when multiple instances need shared access** — EBS cannot be shared across instances (except Multi-Attach io1/io2)  
✓ **Enable lifecycle policies** to move infrequently accessed files to IA/Archive tiers — up to 90% savings  
✓ **Choose Elastic throughput** for unpredictable workloads — avoids over-provisioning  
✓ **Use One Zone for dev/test** — significantly cheaper than Multi-AZ  
✓ **Use Multi-AZ for production** — protects against AZ failures  
✓ **Enable encryption at rest** with KMS for compliance  

---

## SysOps Exam Focus

**Q1: "You need shared storage accessible from EC2 instances in multiple Availability Zones. Which service should you use?"**
- A) EBS with Multi-Attach
- B) Amazon EFS
- C) Instance Store
- D) S3
- **Answer: B** — EFS is a multi-AZ NFS that can be mounted on many instances simultaneously

**Q2: "EFS is compatible with which operating systems?"**
- A) Linux and Windows
- B) Linux only
- C) Windows only
- D) Linux, Windows, and macOS
- **Answer: B** — EFS uses the POSIX file system standard and only supports Linux-based AMIs

**Q3: "How does EFS billing differ from EBS?"**
- A) EFS charges for provisioned capacity like EBS
- B) EFS is pay-per-use — you pay only for the storage you consume, with no capacity provisioning
- C) EFS is free up to 5 GiB
- D) EFS charges per IOPS
- **Answer: B** — EFS auto-scales and bills per GiB used; EBS bills for provisioned capacity

**Q4: "A file in EFS Standard has not been accessed in 60 days. How can you automatically reduce its storage cost?"**
- A) Manually move it to S3 Glacier
- B) Configure a lifecycle policy to move it to EFS Infrequent Access (IA) tier
- C) Create a snapshot of the file system
- D) Compress the file using gzip
- **Answer: B** — Lifecycle policies automatically transition files to IA or Archive tiers based on access patterns

**Q5: "Which EFS throughput mode should you choose for workloads with unpredictable traffic patterns?"**
- A) Bursting
- B) Provisioned
- C) Elastic
- D) Max I/O
- **Answer: C** — Elastic mode automatically scales throughput up and down based on actual workload

**Q6: "You want the cheapest EFS option for a development environment. What configuration should you use?"**
- A) Multi-AZ with Standard storage
- B) One Zone with Infrequent Access (One Zone-IA)
- C) Multi-AZ with Provisioned throughput
- D) One Zone with Max I/O
- **Answer: B** — One Zone-IA combines single-AZ (cheaper) with infrequent access tier (lowest storage cost)

---

## Quick Reference

```
EFS = shared NFS, multi-AZ, Linux only, pay-per-use, auto-scales

vs EBS:
  EBS: one volume → one instance, AZ-locked, provision capacity
  EFS: one FS → many instances, multi-AZ, no provisioning

Performance modes (set at creation):
  General Purpose (default)  →  low latency, web/CMS
  Max I/O                    →  higher throughput, big data

Throughput modes:
  Bursting     →  scales with storage size
  Provisioned  →  set throughput independently
  Elastic      →  auto-scales (unpredictable workloads)

Storage tiers:
  Standard → IA → Archive  (lifecycle policies, up to 90% savings)

Availability:
  Multi-AZ   →  production
  One Zone   →  dev/test (cheaper, still has backups)
```

---

**File: 6_EFS_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
