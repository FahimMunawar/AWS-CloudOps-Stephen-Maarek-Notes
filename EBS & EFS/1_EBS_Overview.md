# 1. EBS Overview

## Overview

**EBS (Elastic Block Store)** is a network-attached storage service for EC2 instances. EBS volumes persist data independently of the instance lifecycle — even after an instance is terminated, the data on the EBS volume remains. Think of them as **network USB sticks** you can plug into different machines.

---

## What is an EBS Volume?

| Property | Detail |
|----------|--------|
| **Type** | Network drive (not physical) — communicates with instances over the network |
| **Persistence** | Data survives instance termination |
| **Billing** | Pay for **provisioned capacity** (GB + IOPS), not actual usage |
| **Capacity** | Must provision size and IOPS upfront — can increase over time |
| **Latency** | Slight network latency since it's not a local disk |

---

## AZ-Locked

EBS volumes are **bound to the Availability Zone** where they are created.

```
us-east-1a volume  →  us-east-1a instance  ✓
us-east-1a volume  →  us-east-1b instance  ✗  (not allowed)
```

To move a volume across AZs: create a **snapshot**, then restore it in the target AZ.

---

## Attachment Rules

| Rule | Detail |
|------|--------|
| One volume → one instance | A volume can only be attached to **one instance at a time** (at CCP level) |
| Multiple volumes → one instance | An instance **can have multiple** EBS volumes attached |
| Unattached volumes | Volumes can exist **without being attached** — attach on demand |
| Cross-AZ | **Not possible** — volume and instance must share the same AZ |

```
us-east-1a                         us-east-1b
──────────────────                 ──────────────────
EC2 Instance A                     EC2 Instance C
  ├── EBS Volume 1                   └── EBS Volume 4
  └── EBS Volume 2
                                   EBS Volume 5 (unattached)
EC2 Instance B
  └── EBS Volume 3
```

- Instance A has two volumes — allowed
- Volume 5 is unattached — allowed
- Volumes in `us-east-1a` cannot attach to instances in `us-east-1b`

---

## Delete on Termination

Controls whether an EBS volume is **automatically deleted** when its EC2 instance is terminated.

| Volume Type | Default |
|-------------|---------|
| **Root volume** | Enabled — deleted on termination |
| **Additional volumes** | Disabled — preserved on termination |

Configurable at launch time via the console (checkbox in the storage configuration section).

---

## Best Practices

✓ **Use snapshots to move data across AZs** — Direct cross-AZ attachment is not possible  
✓ **Disable delete on termination for root volumes** if you need to preserve data after instance termination  
✓ **Use unattached volumes for failover** — Pre-provision volumes and attach on demand when needed  
✓ **Increase capacity over time** — No need to over-provision upfront; size and IOPS can be adjusted later  

---

## SysOps Exam Focus

**Q1: "An EBS volume in us-east-1a needs to be attached to an instance in us-east-1b. What should you do?"**
- A) Detach the volume and attach it to the instance in us-east-1b
- B) Create a snapshot of the volume, then create a new volume from the snapshot in us-east-1b
- C) Copy the volume directly to us-east-1b
- D) Move the instance to us-east-1a instead
- **Answer: B** — EBS volumes are AZ-locked; snapshots are the way to move data across AZs

**Q2: "You terminate an EC2 instance and notice the root EBS volume was also deleted. Why?"**
- A) EBS volumes are always deleted on termination
- B) The root volume has Delete on Termination enabled by default
- C) The instance type does not support persistent storage
- D) The EBS volume was in a different AZ
- **Answer: B** — Root volumes have Delete on Termination enabled by default; additional volumes do not

**Q3: "Can an EC2 instance have more than one EBS volume attached?"**
- A) No — one instance can only have one EBS volume
- B) Yes — an instance can have multiple EBS volumes attached simultaneously
- C) Only if the instance is in a placement group
- D) Only with io2 volume types
- **Answer: B** — Multiple EBS volumes can be attached to a single instance

**Q4: "An EBS volume exists but is not attached to any instance. Is this allowed?"**
- A) No — unattached EBS volumes are automatically deleted after 24 hours
- B) No — EBS volumes must always be attached to an instance
- C) Yes — EBS volumes can exist unattached and be attached on demand
- D) Yes — but only if they are encrypted
- **Answer: C** — Unattached volumes are allowed and billed for their provisioned capacity

**Q5: "What are you billed for with an EBS volume?"**
- A) The amount of data actually stored on the volume
- B) The provisioned capacity (size in GB and IOPS), regardless of usage
- C) Only IOPS consumed
- D) Only when the volume is attached to a running instance
- **Answer: B** — Billing is based on provisioned capacity, not actual usage

---

## Quick Reference

```
EBS = network-attached persistent storage for EC2

Key rules:
  AZ-locked         →  volume and instance must be in the same AZ
  One-to-one        →  one volume attaches to one instance at a time
  Many-to-one       →  one instance can have multiple volumes
  Unattached OK     →  volumes can exist without attachment
  Snapshots         →  the only way to move data across AZs

Delete on Termination:
  Root volume       →  enabled by default (deleted)
  Additional volume →  disabled by default (preserved)
```

---

**File: 1_EBS_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
