# 4. RDS Multi AZ — Failover Conditions

## Overview

Multi AZ automatically fails over from the primary to the standby DB under specific conditions. These triggers are directly tested on the SysOps exam — know all of them.

---

## Failover Triggers

| Condition | Description |
|-----------|-------------|
| **Primary DB failure** | The primary instance crashes or becomes unavailable |
| **OS software patching** | AWS patches the OS of the primary instance |
| **Loss of network connectivity** | Primary becomes unreachable due to network issues |
| **Instance type modification** | You modify the primary (e.g., change instance class) |
| **Primary busy / unresponsive** | Primary is overloaded and stops responding |
| **Underlying storage failure** | EBS or storage layer fails on the primary |
| **AZ outage** | The entire availability zone hosting the primary goes down |
| **Manual failover** | You initiate a **Reboot with failover** action |

---

## Manual Failover

To intentionally trigger a failover (e.g., for testing):

> **RDS Console → Select DB → Reboot → Check "Reboot with failover" → Confirm**

This promotes the standby to primary and demotes the old primary to standby.

---

## Best Practices

✓ **Test failover periodically** — use Reboot with failover to verify your app reconnects cleanly  
✓ **Use a short connection retry in your application** — failover completes in ~60–120 seconds  
✓ **Do not SSH into the primary** — patching and instance changes trigger failover automatically  

---

## SysOps Exam Focus

**Q1: "You modify the instance type of your primary RDS Multi AZ instance. What happens?"**
- A) The modification is rejected — you must disable Multi AZ first
- B) The modification is applied and an automatic failover to the standby occurs
- C) The modification is queued for the next maintenance window only
- D) Both primary and standby are modified simultaneously
- **Answer: B** — Modifying the primary instance triggers a failover to the standby during the change

**Q2: "How do you manually trigger a Multi AZ failover for testing purposes?"**
- A) Stop and start the primary instance
- B) Use the Reboot with failover option on the DB instance
- C) Disable Multi AZ and re-enable it
- D) Take a snapshot and restore it to the standby
- **Answer: B** — Reboot with failover initiates a controlled failover from primary to standby

**Q3: "Which of the following will NOT trigger an RDS Multi AZ failover?"**
- A) The primary DB becomes unresponsive
- B) An AZ outage occurs
- C) A Read Replica falls behind in replication
- D) OS-level software patching of the primary
- **Answer: C** — Read Replica replication lag is independent of Multi AZ; it does not trigger failover

---

## Quick Reference

```
Multi AZ failover triggers:
  - Primary DB failure
  - OS software patching on primary
  - Loss of network connectivity to primary
  - Instance type modification
  - Primary busy / unresponsive
  - Storage failure on primary
  - AZ outage
  - Manual: Reboot with failover

Manual failover: Console → DB → Reboot → Reboot with failover
```

---

**File: 4_RDS_Multi_AZ_Failover_Conditions.md**
**Status: SysOps-focused, exam-ready, concise format**
