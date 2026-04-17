# 11. Aurora Backups, Backtracking, and Cloning

## Overview

Aurora provides three data recovery and cloning mechanisms: **automated backups** (continuous, point-in-time), **backtracking** (in-place rewind, no new cluster), and **database cloning** (fast copy for test environments). Know the key differences for the exam.

---

## Comparison Table

| Feature | Automated Backups | Backtracking | Database Cloning |
|---------|------------------|--------------|-----------------|
| **Purpose** | Point-in-time recovery | Rewind database in place | Create a copy for testing |
| **Retention** | 1–35 days | Up to 72 hours back | Permanent new cluster |
| **Creates new cluster?** | **Yes** | **No** — in-place | **Yes** — but uses shared volume initially |
| **Can be disabled?** | **No** — always on | Optional feature | On-demand action |
| **Engine support** | Aurora MySQL + PostgreSQL | **Aurora MySQL only** | Aurora MySQL + PostgreSQL |
| **Speed** | Standard restore time | Fast rewind | Very fast (copy-on-write) |
| **Use case** | DR, recover from data loss | Undo a bad change quickly | Staging/test from production data |

---

## 1. Automated Backups

- Retention: **1 to 35 days** — cannot be set to 0 (cannot be disabled)
- Enables **Point-in-Time Recovery (PITR)** — restore to within **5 minutes** of current time
- Restore creates a **new Aurora database cluster** — update application connection string afterward

---

## 2. Aurora Backtracking

- Rewinds the database **back and forth in time** — up to **72 hours**
- **In-place restore** — no new cluster is created
- Supported on **Aurora MySQL only** (not PostgreSQL)
- Useful for quickly undoing an accidental DELETE or schema change

```
Current state (bad)
        ↓
Backtrack → rewind to yesterday 4:00 PM
        ↓
Realize you need 5:00 PM instead
        ↓
Backtrack forward → rewind to yesterday 5:00 PM
        ↓
Same cluster, no new endpoint
```

---

## 3. Aurora Database Cloning

- Creates a **new Aurora cluster** from an existing one
- Uses **copy-on-write protocol**:
  - Initially, new cluster shares the same storage volume as the original
  - As new writes come in, data is copied to a separate volume only when changed
  - Result: near-instant clone with minimal storage overhead at creation time

```
Original cluster (production)
        ↓
Clone action (one click)
        ↓
New cluster shares original volume (copy-on-write)
        ↓
Writes diverge → new volume grows independently
```

**Primary use case:** Create a production-like test or staging environment quickly and cheaply.

---

## Best Practices

✓ **Use backtracking for quick recovery from human error** — faster than restoring from backup, no new cluster needed  
✓ **Use cloning for test environments** — one click, production data, no impact on the original cluster  
✓ **Remember: automated backups cannot be disabled in Aurora** — unlike RDS where you can set retention to 0  
✓ **PITR restores to a new cluster** — plan for connection string update in the recovery runbook  
✓ **Backtracking is Aurora MySQL only** — do not assume it works for Aurora PostgreSQL  

---

## SysOps Exam Focus

**Q1: "A developer accidentally ran a DELETE statement on an Aurora MySQL database and removed critical rows. You need to recover the data with minimal disruption. What is the fastest approach?"**
- A) Restore from the most recent automated backup to a new cluster
- B) Use Aurora Backtracking to rewind the database in place to just before the DELETE
- C) Promote a read replica and restore from its snapshot
- D) Use database cloning to create a copy and extract the deleted rows
- **Answer: B** — Backtracking rewinds in place without creating a new cluster, making it the fastest recovery option for Aurora MySQL

**Q2: "What is the key difference between Aurora automated backup restore and Aurora Backtracking?"**
- A) Automated backups retain data for 72 hours; backtracking retains for 35 days
- B) Automated backup restore creates a new cluster; backtracking performs an in-place rewind of the existing cluster
- C) Backtracking creates a new cluster; automated backup restores in place
- D) They are functionally identical
- **Answer: B** — Backup restore = new cluster; backtracking = in-place, same cluster, no endpoint change

**Q3: "You want to create a staging environment using your Aurora production database. The process should be fast and not impact the production cluster. What should you use?"**
- A) Take a manual snapshot and restore it to a new cluster
- B) Create a Read Replica and promote it
- C) Use Aurora database cloning — it creates a new cluster using copy-on-write and completes almost instantly
- D) Export data to S3 and import into a new cluster
- **Answer: C** — Database cloning uses copy-on-write to create a new cluster nearly instantly with no impact on the source

**Q4: "Can Aurora automated backups be disabled?"**
- A) Yes — set the retention period to 0
- B) Yes — disable in the backup settings during cluster creation
- C) No — Aurora automated backups cannot be disabled; retention must be set between 1 and 35 days
- D) Yes — but only during the maintenance window
- **Answer: C** — Unlike RDS, Aurora does not allow setting retention to 0; automated backups are always enabled

**Q5: "Aurora Backtracking is available for which engine type?"**
- A) Aurora PostgreSQL only
- B) Both Aurora MySQL and Aurora PostgreSQL
- C) Aurora MySQL only
- D) Any Aurora engine that uses IO Optimized storage
- **Answer: C** — Backtracking currently supports Aurora MySQL only, not Aurora PostgreSQL

---

## Quick Reference

```
Aurora data recovery options:

Automated Backups:
  Retention: 1–35 days (cannot disable)
  Recovery: PITR to within 5 min of now
  Restore: creates NEW cluster

Backtracking:
  Rewind: up to 72 hours back/forward
  Restore: IN-PLACE (no new cluster)
  Engine: Aurora MySQL ONLY

Database Cloning:
  Creates: new cluster (copy-on-write, near-instant)
  Use case: test/staging from production data
  Impact on source: none

Remember:
  Backup restore  → new cluster
  Backtracking    → same cluster, in-place
  Cloning         → new cluster, fast
```

---

**File: 11_Aurora_Backups_Backtracking_Cloning.md**
**Status: SysOps-focused, exam-ready, concise format**
