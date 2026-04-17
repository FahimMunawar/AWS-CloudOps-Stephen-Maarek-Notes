# 2 — Amazon FSx — Hands-On (Console Overview)

## Overview

Quick console walkthrough of the FSx creation options. No new concepts — all exam-relevant details are in [1_Amazon_FSx.md](1_Amazon_FSx.md).

---

## Console Path

FSx → **Create file system** → choose one of four types

---

## Notable Console Options Per Type

### FSx for Lustre
- Deployment type: **Scratch** or **Persistent**
- Storage type: SSD or HDD
- Throughput capacity
- VPC and subnet placement
- Encryption settings

### FSx for Windows File Server
- Deployment: **Multi-AZ** (production) or **Single-AZ** (dev/test)
- Storage type: SSD or HDD
- Active Directory integration:
  - AWS Managed Microsoft AD
  - Self-managed Microsoft AD (on-premises)
- Additional: auditing, backup, maintenance windows

### FSx for NetApp ONTAP
- Deployment: Multi-AZ or Single-AZ
- Storage efficiency features (enabled at creation):
  - **Deduplication**
  - **Compression**
  - **Compaction**
- Quick Create or Standard Create options

### FSx for OpenZFS
- Deployment options similar to others
- Compatible with Linux, Windows, macOS

---

## Quick Reference

```
Exam focus = knowing WHICH file system to choose, not console steps
See file 1 for: feature comparisons, use cases, decision guide
Console key point: storage efficiency (dedup/compression) enabled at NetApp ONTAP creation
```
