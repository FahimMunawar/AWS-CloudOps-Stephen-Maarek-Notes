# 5. ECS Service Rolling Updates

## Overview

When updating an ECS service to a new task definition revision, AWS performs a **rolling update** controlled by two settings: **Minimum Healthy Percent** and **Maximum Percent**. These determine how many tasks can be terminated and created simultaneously during the update.

---

## Key Settings

| Setting | Default | Meaning |
|---------|---------|---------|
| **Minimum Healthy Percent** | 100 | The minimum % of running tasks that must remain healthy during the update |
| **Maximum Percent** | 200 | The maximum % of tasks (old + new) that can run simultaneously |

> 100% = current desired task count. If desired = 4 tasks, then 100% = 4 tasks.

---

## Scenario 1: Min 50%, Max 100% (starting with 4 tasks)

```
Start: 4 old tasks (100%)
        ↓
Terminate 2 old tasks → 2 running (50% = minimum)
        ↓
Create 2 new tasks → 4 running (100% = maximum)
        ↓
Terminate 2 old tasks → 2 new running (50%)
        ↓
Create 2 new tasks → 4 new tasks (100%)
✓ Update complete
```

- Tasks are **terminated first** (capacity drops below 100% temporarily)
- Suitable when brief capacity reduction is acceptable

---

## Scenario 2: Min 100%, Max 150% (starting with 4 tasks)

```
Start: 4 old tasks (100%)
        ↓
Cannot terminate — minimum is 100%
Create 2 new tasks → 6 running (150% = maximum)
        ↓
Terminate 2 old tasks → 4 running (100%)
        ↓
Create 2 new tasks → 6 running (150%)
        ↓
Terminate 2 old tasks → 4 new tasks (100%)
✓ Update complete
```

- **New tasks are created first**, then old are terminated
- Maintains full capacity throughout — no downtime
- Costs more temporarily (running more tasks)

---

## Comparison

| | Min 50%, Max 100% | Min 100%, Max 150% |
|-|-------------------|-------------------|
| **Capacity during update** | Drops to 50% | Stays at 100% |
| **Extra cost** | No | Yes (extra tasks run temporarily) |
| **Risk** | Brief capacity reduction | No capacity reduction |
| **Use case** | Cost-sensitive updates | Zero-downtime deployments |

---

## Best Practices

✓ **Use Min 100%, Max 200% for production** — no downtime, new tasks start before old ones stop  
✓ **Use Min 50%, Max 100% for cost-sensitive non-critical services** — brief reduction is acceptable  
✓ **Monitor Events tab during update** — shows task start/stop sequence and any failures  
✓ **Set Max > 100%** — otherwise no new tasks can launch until old ones are terminated first  

---

## SysOps Exam Focus

**Q1: "You update an ECS service with Minimum Healthy Percent = 50% and Maximum Percent = 100%. You have 4 running tasks. What happens first during the update?"**
- A) 2 new tasks are created first, then 2 old tasks are terminated
- B) 2 old tasks are terminated first (to stay at or above 50%), then 2 new tasks are created
- C) All 4 tasks are terminated and replaced simultaneously
- D) 1 task is terminated and 1 new task is created at a time
- **Answer: B** — With Max = 100%, no extra tasks can be created; old tasks must be terminated first to make room

**Q2: "You need to update an ECS service with zero downtime — full capacity must be maintained at all times. Which settings should you use?"**
- A) Minimum Healthy Percent = 50%, Maximum Percent = 100%
- B) Minimum Healthy Percent = 100%, Maximum Percent = 150%
- C) Minimum Healthy Percent = 0%, Maximum Percent = 200%
- D) Minimum Healthy Percent = 100%, Maximum Percent = 100%
- **Answer: B** — Min 100% prevents task termination until new tasks are running; Max 150% allows new tasks to launch before old ones stop

---

## Quick Reference

```
ECS Rolling Update settings:
  Minimum Healthy % → floor: minimum % of tasks that must stay running
  Maximum %         → ceiling: max % of (old + new) tasks running at once
  100% = current desired task count

Scenario: Min 50%, Max 100%
  → Terminate old first → create new (capacity dips)
  → Cost-effective, brief capacity reduction

Scenario: Min 100%, Max 150%
  → Create new first → terminate old (no capacity dip)
  → Zero downtime, temporary extra cost

Exam tip: Max > 100% needed to create new tasks before terminating old
```

---

**File: 5_ECS_Rolling_Updates.md**
**Status: SysOps-focused, exam-ready, concise format**
