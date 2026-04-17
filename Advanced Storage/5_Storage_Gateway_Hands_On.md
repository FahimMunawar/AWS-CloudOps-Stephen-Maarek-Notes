# 5 — Storage Gateway — Hands-On (Console Overview)

## Overview

Quick console walkthrough of Storage Gateway creation options. All exam-relevant details are in [4_AWS_Storage_Gateway.md](4_AWS_Storage_Gateway.md).

---

## Gateway Types Available in Console

| Type | Notes |
|---|---|
| **Amazon S3 File Gateway** | NFS/SMB access to S3 with local cache |
| **Tape Gateway** | iSCSI VTL → S3 Glacier |
| **Volume Gateway** | Block storage backed by S3 (Cached or Stored) |
| FSx File Gateway | Grayed out / being deprecated — ignore |

---

## Hosting Platform Options (All Gateway Types)

| Platform | Location |
|---|---|
| **VMware ESXi** | On-premises data center |
| **Microsoft Hyper-V** | On-premises data center |
| **Linux KVM** | On-premises data center |
| **Amazon EC2** | AWS cloud (gateway runs in your AWS account) |

> **On-premises platforms** (VMware, Hyper-V, KVM): gateway runs close to your servers in the data center — typical use case for hybrid cloud.
> **EC2 option**: gateway runs in AWS — useful for testing or when caching within the cloud is acceptable.

---

## Volume Gateway Modes (Confirmed in Console)

| Mode | Primary Storage | Local Cache | Sync |
|---|---|---|---|
| **Cached Volume** | Amazon S3 | Frequently accessed data only | Continuous |
| **Stored Volume** | On-premises (local) | Entire dataset | Async to S3 |

---

## Quick Reference

```
Console path: Storage Gateway → Create gateway → choose type → choose platform
Platforms: VMware / Hyper-V / KVM (on-prem) or EC2 (AWS)
FSx File Gateway: grayed out / deprecated — not exam-relevant
Volume Gateway modes confirmed: Cached (S3 primary) vs Stored (on-prem primary)
```
