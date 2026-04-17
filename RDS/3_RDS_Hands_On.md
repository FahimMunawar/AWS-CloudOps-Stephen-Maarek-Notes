# 3. RDS — Hands-On

## Overview

A walkthrough of creating a MySQL RDS instance, connecting to it with a SQL client (SQLectron), exploring key management options (Read Replica, snapshots, monitoring), and cleaning up.

---

## Step 1: Create the Database

**Console:** RDS → Databases → **Create database** → Standard create

| Setting | Value |
|---------|-------|
| Engine | MySQL |
| Template | **Free tier** (Single AZ, one instance) |
| DB identifier | (default) |
| Master username | `admin` |
| Credential management | Self-managed (or AWS Secrets Manager — costs extra) |
| Master password | (set your own) |
| Authentication | Password auth (IAM database auth also available) |
| Instance class | `db.t4g.micro` (free tier default) |
| Storage | 20 GiB |
| Storage autoscaling | Enabled — set max threshold (e.g., 1000 GiB) |
| Connectivity | Don't connect to EC2 |
| VPC | Default VPC |
| Public access | **Yes** (needed to connect from local client) |
| VPC Security Group | Create new → name: `demo-rds` |
| Port | 3306 (MySQL default) |

> Production templates unlock Multi AZ options: 2-instance DB instance or 3-instance DB cluster.

---

## Step 2: Verify Security Group

After creation, check the security group inbound rules:

- Protocol: TCP
- Port: 3306
- Source: Your IP (adjust to `0.0.0.0/0` temporarily if connection fails)

---

## Step 3: Connect with SQLectron

Download: SQLectron GUI → latest release for your OS

| Field | Value |
|-------|-------|
| Name | `RDS demo` |
| Database type | MySQL |
| Server address | RDS endpoint from console |
| Port | 3306 |
| User | `admin` |
| Password | (your master password) |
| Initial database | `mydb` |

Click **Test** → should show connection successful → **Save** → **Connect**

---

## Step 4: Basic SQL Operations (Out of Scope for Exam)

```sql
-- Create a table
CREATE TABLE mytable (name VARCHAR(20), first_name VARCHAR(20));

-- Insert a row
INSERT INTO mytable VALUES ('Maarek', 'Stephane');

-- Read rows
SELECT * FROM mytable;
```

---

## Step 5: Key Management Actions

| Action | Where | Notes |
|--------|-------|-------|
| **Create Read Replica** | Actions → Create read replica | Choose Multi AZ for the replica if needed |
| **Monitoring** | Monitoring tab | CPU, DB connections, IOPS, etc. |
| **Take snapshot** | Actions → Take snapshot | Manual point-in-time backup |
| **Restore to point in time** | Actions → Restore | Restores within backup retention window |
| **Migrate snapshot** | Actions → Copy snapshot | Supports cross-region migration |
| **Modify instance** | Modify | Change instance type, storage, Multi AZ, etc. |

---

## Step 6: Cleanup

> Always delete to avoid charges — RDS charges per instance-hour.

1. **Modify** → scroll to bottom → uncheck **Deletion protection** → **Continue** → Apply immediately
2. **Actions** → **Delete** → uncheck "Create final snapshot" → type `delete me` → confirm

---

## Best Practices

✓ **Enable Storage Auto Scaling** — prevents downtime from storage exhaustion  
✓ **Use Secrets Manager for credentials in production** — more secure than self-managed passwords  
✓ **Restrict security group inbound to known IPs** — avoid `0.0.0.0/0` in production  
✓ **Enable deletion protection on production databases** — prevents accidental deletion  
✓ **Monitor DB connections in CloudWatch** — high connection count signals scaling need  

---

## SysOps Exam Focus

**Q1: "You are creating an RDS instance and want the database password to be automatically rotated and stored securely. What option should you choose during creation?"**
- A) Self-managed password with IAM authentication
- B) AWS Secrets Manager for credential management
- C) Store the password in Parameter Store manually
- D) Use an EC2 instance profile to manage credentials
- **Answer: B** — AWS Secrets Manager integrates directly with RDS and supports automatic rotation; there is an additional cost

**Q2: "An RDS instance was accidentally deleted and no final snapshot was taken. What should have been configured to prevent this?"**
- A) Storage Auto Scaling
- B) Multi AZ
- C) Deletion protection
- D) Enhanced monitoring
- **Answer: C** — Deletion protection prevents the database from being deleted until the setting is explicitly disabled

**Q3: "You need to move an RDS snapshot from us-east-1 to eu-west-1 for disaster recovery. What action do you take?"**
- A) Restore the snapshot, then use DMS to migrate the data
- B) Use Actions → Copy snapshot and select the destination region
- C) Enable Multi AZ and promote the standby in eu-west-1
- D) Export the snapshot to S3 and import it in eu-west-1
- **Answer: B** — RDS supports cross-region snapshot copy directly from the console

---

## Quick Reference

```
Create RDS (Free Tier):
  Engine: MySQL | Template: Free tier | Single AZ
  Storage: 20 GiB | Auto Scaling: enabled
  Public access: Yes | SG: allow TCP 3306 from your IP

Connect: SQLectron → endpoint:3306 → admin / password

Key actions:
  Create Read Replica  →  Actions → Create read replica
  Take snapshot        →  Actions → Take snapshot
  Point-in-time restore→  Actions → Restore to point in time
  Cross-region copy    →  Actions → Copy snapshot

Cleanup:
  1. Modify → disable deletion protection
  2. Actions → Delete → skip final snapshot → confirm
```

---

**File: 3_RDS_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
