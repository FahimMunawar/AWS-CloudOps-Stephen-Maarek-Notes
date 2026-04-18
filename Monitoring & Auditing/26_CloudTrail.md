# 26 — AWS CloudTrail

## Overview

CloudTrail provides governance, compliance, and auditing for your AWS account by recording a history of all API calls and events — **enabled by default**.

Sources recorded: AWS Console, SDK, CLI, IAM Users, IAM Roles, AWS services.

Default retention: **90 days** in CloudTrail. For longer retention → send to **S3** and query with **Athena**.

---

## Event Types

### 1. Management Events (logged by default)

Operations performed on AWS resources.

| Sub-type | Examples |
|---|---|
| **Read Events** | `ListUsers`, `DescribeInstances` — don't modify resources |
| **Write Events** | `DeleteTable`, `AttachRolePolicy` — modify resources |

> Write Events are higher risk — they can cause damage to your infrastructure.

### 2. Data Events (NOT logged by default — high volume)

| Service | Operations |
|---|---|
| **S3 object-level** | `GetObject` (Read), `PutObject` / `DeleteObject` (Write) |
| **Lambda invocations** | `Invoke` API — how many times functions are called |

Can be enabled; Read and Write can be separated.

### 3. CloudTrail Insights Events (must enable + pay)

Unusual activity detected by ML analysis — see below.

---

## CloudTrail Insights

Analyzes Management Events to detect unusual patterns.

### What it detects
- Inaccurate resource provisioning
- Hitting service limits
- Burst of IAM actions
- Gaps in periodic maintenance activity

### How it works

```
Management Events → CloudTrail Insights (analyzes baseline)
                          │
                    Anomaly detected
                          │
              ┌───────────┼────────────────┐
              ▼           ▼                ▼
     CloudTrail Console  S3 bucket   EventBridge event
                                          │
                                     → automate (email, Lambda, etc.)
```

---

## Event Retention & Long-Term Storage

| Tier | Retention | Query tool |
|---|---|---|
| **CloudTrail default** | 90 days | CloudTrail console |
| **Long-term (S3)** | Indefinite | Amazon Athena |

---

## Common Use Case

> "An EC2 instance was terminated. Who did it?"

→ Look in **CloudTrail** — the `TerminateInstances` API call will be recorded with the caller's identity, time, and source IP.

---

## SysOps Exam Q&A

**Q: Is CloudTrail enabled by default?**
A: **Yes** — all Management Events are logged by default.

**Q: What are the three types of CloudTrail events?**
A: **Management Events** (default, on/off for Read/Write), **Data Events** (not default — S3 object-level and Lambda invocations), **Insights Events** (paid — ML-based anomaly detection).

**Q: Why are Data Events not logged by default?**
A: They are **high-volume operations** (e.g., every S3 GetObject) — logging them by default would generate enormous costs and data volumes.

**Q: How do you retain CloudTrail events beyond 90 days?**
A: Send events to an **S3 bucket**, then use **Amazon Athena** to query them.

**Q: What does CloudTrail Insights detect?**
A: Unusual management activity — inaccurate provisioning, service limit bursts, IAM action bursts, maintenance gaps — by comparing against a learned baseline.

**Q: Where do CloudTrail Insights findings appear?**
A: In the **CloudTrail console**, optionally in **S3**, and as **EventBridge events** for automation.

---

## Quick Reference

```
CloudTrail = audit log of all API calls in your AWS account (enabled by default)
Sources: Console, SDK, CLI, IAM Users/Roles, AWS services

Event types:
  Management (default): resource operations — Read / Write separable
  Data (not default):   S3 object-level (Get/Put/Delete) + Lambda Invoke
  Insights (paid):      ML anomaly detection on Management Events

Insights findings → CloudTrail console + S3 + EventBridge

Retention:
  Default: 90 days in CloudTrail
  Long-term: S3 → query with Athena

Who deleted X? → check CloudTrail
```
