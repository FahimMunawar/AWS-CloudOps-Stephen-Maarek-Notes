# 1. RDS Overview

## Overview

**Amazon RDS (Relational Database Service)** is a fully managed cloud database service for SQL-based databases. AWS handles the underlying infrastructure, patching, backups, and scaling — you just use the database.

---

## Supported Database Engines

| Engine | Notes |
|--------|-------|
| **PostgreSQL** | Open source |
| **MySQL** | Open source |
| **MariaDB** | Open source |
| **Oracle** | Commercial |
| **Microsoft SQL Server** | Commercial |
| **IBM DB2** | Commercial |
| **Aurora** | AWS proprietary — covered separately |

---

## RDS vs Self-Managed Database on EC2

| Feature | RDS (Managed) | EC2 Self-Managed |
|---------|--------------|-----------------|
| Provisioning | Automated | Manual |
| OS patching | AWS handles | You handle |
| Continuous backups | Yes (Point in Time Restore) | You set up |
| Monitoring dashboards | Built-in | You configure |
| Read replicas | Built-in feature | You build |
| Multi-AZ / HA | Built-in feature | You architect |
| Maintenance windows | Configurable | You schedule |
| Vertical scaling | Console/API | Instance migration |
| Horizontal scaling | Read replicas | You configure |
| Storage backend | EBS | EBS or instance store |
| SSH access | **Not available** | Available |

> The tradeoff: you give up SSH access in exchange for AWS managing all operational tasks.

---

## RDS Storage Auto Scaling

Automatically increases storage when the database is running low — no manual intervention or downtime required.

### How It Works

```
Application reads/writes heavily to RDS
        ↓
Free storage drops below 10% of allocated storage
        ↓
Low storage condition persists for > 5 minutes
        ↓
At least 6 hours since last modification
        ↓
RDS automatically scales storage up
```

### Configuration

| Setting | Detail |
|---------|--------|
| **Maximum storage threshold** | The upper limit you set — storage will not grow beyond this |
| **Trigger condition** | Free storage < 10% AND persisting > 5 min AND 6 hrs since last change |
| **Supported engines** | All RDS engines |
| **Use case** | Applications with unpredictable or growing storage needs |

> Without Auto Scaling, you would need to manually increase storage, which may require maintenance.

---

## Best Practices

✓ **Enable Storage Auto Scaling** for production databases with unpredictable workloads  
✓ **Set a sensible maximum threshold** — prevents runaway storage growth and unexpected costs  
✓ **Use Multi-AZ for production** — automatic failover if the primary instance fails  
✓ **Use Read Replicas for read-heavy workloads** — offload queries from the primary  
✓ **Use maintenance windows** for scheduled patching to minimize disruption  

---

## SysOps Exam Focus

**Q1: "Which database engines are supported by Amazon RDS?"**
- A) Only MySQL and PostgreSQL
- B) PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, and Aurora
- C) Only open-source databases (PostgreSQL, MySQL, MariaDB)
- D) Any database engine you install
- **Answer: B** — Know all seven supported engines including Oracle, SQL Server, IBM DB2, and AWS Aurora

**Q2: "What is the key operational limitation of Amazon RDS compared to running a database on EC2?"**
- A) RDS does not support automated backups
- B) You cannot SSH into the underlying RDS instance
- C) RDS only supports MySQL
- D) RDS cannot be deployed in a VPC
- **Answer: B** — RDS is managed; AWS controls the EC2 instance underneath, so SSH access is not available

**Q3: "Your RDS instance has 100 GiB of storage allocated. Storage Auto Scaling is enabled with a max of 500 GiB. When will RDS automatically increase storage?"**
- A) When storage usage exceeds 80 GiB
- B) When free storage drops below 10 GiB, persists for over 5 minutes, and 6 hours have passed since the last modification
- C) Every 24 hours automatically
- D) When CPU utilization exceeds 80%
- **Answer: B** — Auto Scaling triggers when free storage < 10% (10 GiB of 100 GiB), condition lasts > 5 min, and 6 hrs have passed since last change

**Q4: "What is the benefit of RDS Point in Time Restore?"**
- A) It allows you to SSH into the RDS instance
- B) It lets you restore the database to any second within the backup retention window
- C) It automatically increases IOPS when load is high
- D) It replicates the database across regions automatically
- **Answer: B** — Continuous backups enable Point in Time Restore to any timestamp within the retention period

---

## Quick Reference

```
RDS = managed relational database (SQL)

Engines: PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, IBM DB2, Aurora

Managed features:
  Automated provisioning + OS patching
  Continuous backups + Point in Time Restore
  Read replicas (read scaling)
  Multi-AZ (HA + DR)
  Storage backed by EBS
  No SSH access

Storage Auto Scaling:
  Trigger: free storage < 10% for > 5 min AND 6 hrs since last change
  Set a max threshold to cap growth
  Supports all RDS engines
```

---

**File: 1_RDS_Overview.md**
**Status: SysOps-focused, exam-ready, concise format**
