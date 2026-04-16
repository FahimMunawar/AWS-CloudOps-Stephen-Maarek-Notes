# 2. EBS Volumes — Hands-On

## Overview

A practical walkthrough of creating, attaching, and deleting EBS volumes, and verifying the **AZ-lock** and **Delete on Termination** behaviors in the AWS console.

---

## Viewing EBS Volumes on an Instance

**Console path:** EC2 → Instances → select instance → **Storage** tab → Block devices

- Shows all attached volumes with size, type, and device name
- Click the volume ID to navigate to the **Volumes** interface
- You can also access volumes directly: EC2 → **Elastic Block Store → Volumes**

---

## Creating and Attaching a Volume

### Step 1: Identify the Instance's AZ

EC2 → Instances → select instance → **Networking** tab → **Availability Zone**

> The new volume **must** be created in the same AZ as the target instance.

### Step 2: Create the Volume

1. EC2 → Volumes → **Create volume**
2. Volume type: `GP2`
3. Size: `2 GiB`
4. Availability Zone: same as instance (e.g., `eu-west-1b`)
5. Click **Create volume**

### Step 3: Attach the Volume

1. Select the new volume → **Actions → Attach volume**
2. Choose the target instance
3. Click **Attach volume**

### Verification

EC2 → Instances → Storage tab → Block devices now shows **two volumes** (8 GiB root + 2 GiB new).

> To actually use the new volume (mount, format), see the AWS documentation: *"Make an Amazon EBS volume available for use on Linux"*.

---

## Demonstrating the AZ Lock

1. Create another volume in a **different AZ** (e.g., `eu-west-1a` when instance is in `eu-west-1b`)
2. Select volume → Actions → Attach volume
3. **Result:** No instances available to attach — the volume and instance are in different AZs

```
Volume in eu-west-1a  →  Instance in eu-west-1b  →  Attach fails (no instances shown)
```

This volume can be deleted since it serves no purpose: Actions → **Delete volume**.

---

## Demonstrating Delete on Termination

### Check the Attribute

EC2 → Instances → Storage tab → Block devices → scroll right to **Delete on Termination** column:

| Volume | Size | Delete on Termination |
|--------|------|-----------------------|
| Root volume | 8 GiB | **Yes** |
| Additional volume | 2 GiB | **No** |

### Where This Is Configured

During instance launch → Storage section → **Advanced** → the root volume row shows `Delete on termination: Yes` by default. You can change it to `No` to preserve the root volume.

### What Happens on Instance Termination

1. Terminate the instance
2. Go to EC2 → Volumes → refresh
3. **Result:**
   - The 8 GiB root volume **disappears** (deleted automatically)
   - The 2 GiB additional volume changes status to **Available** (preserved)

```
Instance terminated
        ↓
Root volume (Delete on Termination: Yes)        → deleted
Additional volume (Delete on Termination: No)   → detached, status: Available
```

---

## Best Practices

✓ **Always check your instance's AZ** before creating a new volume — mismatched AZs cannot attach  
✓ **Set Delete on Termination to No** on root volumes if you need the data to survive instance termination  
✓ **Delete unused volumes** — unattached volumes still incur storage charges  
✓ **Use the Storage tab** on the instance for a quick overview of all attached block devices  

---

## SysOps Exam Focus

**Q1: "You created a new EBS volume but cannot attach it to your EC2 instance. What is the most likely cause?"**
- A) The volume is encrypted
- B) The volume is in a different AZ than the instance
- C) The instance already has a root volume
- D) The volume type is incompatible
- **Answer: B** — EBS volumes can only be attached to instances in the same AZ

**Q2: "After terminating an EC2 instance, the root volume is gone but an additional 2 GiB volume remains. Why?"**
- A) Additional volumes cannot be deleted
- B) The root volume had Delete on Termination enabled; the additional volume did not
- C) The additional volume was in a different AZ
- D) EBS volumes are never deleted automatically
- **Answer: B** — Root volumes default to Delete on Termination: Yes; additional volumes default to No

**Q3: "Where can you change the Delete on Termination setting for a root volume?"**
- A) In the EBS Volumes console after the instance is running
- B) During instance launch, in the Storage section under Advanced settings
- C) In the IAM role attached to the instance
- D) In the security group settings
- **Answer: B** — The setting is configured at launch time in the storage configuration

**Q4: "You have an unattached EBS volume. Are you still being charged for it?"**
- A) No — you only pay when a volume is attached to a running instance
- B) Yes — you pay for provisioned capacity regardless of attachment status
- C) Only if the volume contains data
- D) Only for the first 30 days
- **Answer: B** — Unattached volumes incur charges based on their provisioned size

---

## Quick Reference

```
View volumes:     EC2 → Instance → Storage tab → Block devices
Create volume:    EC2 → Volumes → Create volume (must match instance AZ)
Attach:           Select volume → Actions → Attach volume → choose instance
Delete:           Select volume → Actions → Delete volume

AZ rule:          Volume and instance must be in the same AZ
Delete on Term:   Root = Yes (default)  |  Additional = No (default)
After terminate:  Root deleted, additional volumes become Available
```

---

**File: 2_EBS_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
