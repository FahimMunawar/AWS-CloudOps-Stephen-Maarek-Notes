# 3 — FSx for Windows: Single-AZ vs Multi-AZ

## Overview

FSx for Windows File Server has two deployment options. For high availability and automatic failover, always choose **Multi-AZ**.

---

## Single-AZ Options

| Generation | Storage Support | Replication |
|---|---|---|
| **Single-AZ 1** | SSD only | Within the AZ only |
| **Single-AZ 2** | SSD and HDD | Within the AZ only |

- Data is replicated **within** the single AZ automatically
- No cross-AZ replication
- If the AZ goes down → file system is unavailable

---

## Multi-AZ

| Feature | Detail |
|---|---|
| **AZs** | Primary + Standby in two separate AZs |
| **Replication** | **Synchronous** — data written to both AZs simultaneously |
| **Failover** | **Automatic** — if primary fails, standby takes over |
| **Use case** | Production workloads requiring high availability |

```
Primary AZ                    Standby AZ
FSx File System  ←──sync──►  FSx File System (standby)
       │
  Failure detected
       │
       ▼
Automatic failover to standby
```

---

## Exam Rule

> **Always choose Multi-AZ over two Single-AZ deployments when you need failover.**

The exam may present "use 2 Single-AZ file systems and replicate between them" as an option — this is the wrong answer. Use **Multi-AZ** instead.

---

## SysOps Exam Q&A

**Q: You need FSx for Windows with automatic failover across availability zones. What should you choose?**
A: **FSx for Windows Multi-AZ** — provides synchronous replication and automatic failover between primary and standby file systems.

**Q: What is the difference between Single-AZ 1 and Single-AZ 2 for FSx for Windows?**
A: **Single-AZ 1** supports SSD only. **Single-AZ 2** supports both SSD and HDD.

**Q: Is Multi-AZ or two Single-AZ deployments recommended for high availability in FSx for Windows?**
A: **Multi-AZ** — it handles synchronous replication and automatic failover natively. Managing replication manually between two Single-AZ systems is the wrong approach on the exam.

---

## Quick Reference

```
Single-AZ 1  = SSD only, replication within AZ
Single-AZ 2  = SSD + HDD, replication within AZ
Multi-AZ     = synchronous replication across 2 AZs, automatic failover

Exam rule: need failover → always choose Multi-AZ (not 2x Single-AZ)
```
