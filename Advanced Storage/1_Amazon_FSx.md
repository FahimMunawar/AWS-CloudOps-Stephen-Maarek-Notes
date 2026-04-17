# 1 — Amazon FSx

## Overview

Amazon FSx is a fully managed service for launching **third-party, high-performance file systems** on AWS. Think of FSx as "RDS but for file systems" — AWS manages the infrastructure while you use the file system of your choice.

---

## Four FSx File System Types

| FSx Type | Key Protocol | Primary Use Case |
|---|---|---|
| **FSx for Windows File Server** | SMB, NTFS | Windows workloads, Active Directory integration |
| **FSx for Lustre** | POSIX | HPC, machine learning, large-scale computing |
| **FSx for NetApp ONTAP** | NFS, SMB, iSCSI | Lift-and-shift from ONTAP/NAS, broad OS compatibility |
| **FSx for OpenZFS** | NFS | Lift-and-shift from ZFS, high IOPS workloads |

---

## FSx for Windows File Server

| Feature | Detail |
|---|---|
| **Protocol** | SMB (Server Message Block), Windows NTFS |
| **Active Directory** | Fully integrated for user security |
| **ACLs & User Quotas** | Supported |
| **Linux mounting** | **Yes** — can be mounted on Linux EC2 instances (exam tip) |
| **On-premises integration** | Microsoft DFS (Distributed File System) to group with on-prem Windows File Servers |
| **Performance** | Tens of GB/s, millions of IOPS, hundreds of PB |
| **Storage options** | SSD (low latency — databases, analytics) or HDD (cost-effective — home directories, CMS) |
| **High availability** | Multi-AZ configuration available |
| **On-premises access** | Via private connection (VPN or Direct Connect) |
| **Backup** | Daily backups to Amazon S3 |

> **Exam tip:** FSx for Windows File Server looks Windows-only but **can be mounted on Linux EC2 instances**.

---

## FSx for Lustre

**Lustre = Linux + Cluster** — a distributed file system for massive-scale, high-performance computing.

| Feature | Detail |
|---|---|
| **Use cases** | HPC, machine learning, video processing, financial modeling, EDA |
| **Performance** | Hundreds of GB/s, millions of IOPS, sub-millisecond latency |
| **Storage options** | SSD (low latency, small random I/O) or HDD (throughput-intensive, large sequential I/O) |
| **S3 integration** | Read S3 as a file system; write computation output back to S3 |
| **On-premises access** | Via VPN or Direct Connect |
| **AZ scope** | Lives within a single AZ |

### Deployment Options: Scratch vs Persistent

| Feature | Scratch | Persistent |
|---|---|---|
| **Storage type** | Temporary | Long-term |
| **Data replication** | None — data lost if server fails | Replicated within the same AZ |
| **Performance** | 6× higher burst than persistent; 200 MB/s/TB | Standard |
| **On server failure** | Data lost | Files transparently replaced within minutes |
| **Use case** | Short-term processing, cost-optimized | Long-term, sensitive data |
| **Data copies** | 1 | 2 (within same AZ) |

> **Exam keyword:** "HPC" or "high-performance computing" → FSx for Lustre

---

## FSx for NetApp ONTAP

| Feature | Detail |
|---|---|
| **Protocols** | NFS, SMB, iSCSI |
| **Primary use case** | Migrate workloads from ONTAP or NAS on-premises to AWS |
| **OS compatibility** | Linux, Windows, macOS, VMware Cloud on AWS, WorkSpaces, EC2, ECS, EKS |
| **Auto-scaling** | Storage automatically shrinks or grows |
| **Features** | Snapshots, replication, data compression, data de-duplication |
| **Cloning** | **Point-in-time instantaneous cloning** — great for staging/testing |
| **Cost** | Low cost |

> **Exam tip:** "ONTAP", "NAS", "data de-duplication", or "instant cloning for testing" → FSx for NetApp ONTAP

---

## FSx for OpenZFS

| Feature | Detail |
|---|---|
| **Protocols** | NFS (multiple versions) |
| **Primary use case** | Migrate workloads from ZFS on-premises to AWS |
| **OS compatibility** | Linux, macOS, Windows |
| **Performance** | Up to **1 million IOPS**, < **0.5 ms latency** |
| **Features** | Snapshots, compression, low cost |
| **De-duplication** | **Not supported** (unlike NetApp ONTAP) |
| **Cloning** | **Point-in-time instantaneous cloning** |

> **Exam tip:** "ZFS", "1 million IOPS with sub-millisecond latency", or "NFS only" → FSx for OpenZFS

---

## Decision Guide: Which FSx to Choose

| Scenario | FSx Type |
|---|---|
| Windows shares, Active Directory, SMB protocol | **Windows File Server** |
| HPC, ML, large-scale computing, S3 integration | **Lustre** |
| Short-term compute with max performance | **Lustre (Scratch)** |
| Long-term compute storage with replication | **Lustre (Persistent)** |
| Migrate from ONTAP or NAS, need NFS + SMB + iSCSI | **NetApp ONTAP** |
| Need data de-duplication or instant cloning | **NetApp ONTAP** |
| Migrate from ZFS, need NFS, need 1M IOPS | **OpenZFS** |

---

## SysOps Exam Q&A

**Q: A company needs a managed file system that integrates with Microsoft Active Directory. Which FSx type should they use?**
A: **FSx for Windows File Server** — supports SMB, NTFS, and Active Directory integration.

**Q: Can FSx for Windows File Server be mounted on Linux EC2 instances?**
A: **Yes** — despite being a Windows file system, it can be mounted on Linux EC2 instances.

**Q: A scientific research team needs a high-performance file system for HPC workloads with S3 integration. Which FSx type?**
A: **FSx for Lustre** — designed for HPC, ML, and large-scale computing with seamless S3 read/write integration.

**Q: What is the difference between Lustre Scratch and Persistent deployment?**
A: **Scratch** = temporary, no replication, 6× higher burst performance, data lost on server failure. **Persistent** = long-term, replicated within same AZ, files transparently replaced on failure.

**Q: Which FSx type supports NFS, SMB, and iSCSI simultaneously?**
A: **FSx for NetApp ONTAP**.

**Q: A company wants to instantly clone their production file system to create a test environment. Which FSx type supports this?**
A: **FSx for NetApp ONTAP** or **FSx for OpenZFS** — both support point-in-time instantaneous cloning.

**Q: Which FSx type does NOT support data de-duplication?**
A: **FSx for OpenZFS** — de-duplication is a NetApp ONTAP feature, not available in OpenZFS.

---

## Quick Reference

```
FSx for Windows File Server:
  Protocol: SMB, NTFS  |  AD integrated  |  Linux mountable  |  Multi-AZ  |  Daily S3 backup

FSx for Lustre:
  Keyword: HPC, ML, large-scale computing
  S3 integration (read input / write output)
  Scratch: temp, no replication, 6× burst performance
  Persistent: long-term, replicated within AZ

FSx for NetApp ONTAP:
  Protocol: NFS + SMB + iSCSI
  Auto-scaling  |  de-duplication  |  instant cloning
  Migrate from: ONTAP or NAS on-premises

FSx for OpenZFS:
  Protocol: NFS only
  1 million IOPS, <0.5ms latency
  Instant cloning  |  No de-duplication
  Migrate from: ZFS on-premises
```
