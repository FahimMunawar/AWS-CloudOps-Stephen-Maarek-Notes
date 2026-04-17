# 6. RDS Events and Logging

## Overview

RDS generates **events** for database lifecycle changes and **logs** for database activity. Both can be routed to monitoring and alerting systems — events via SNS or EventBridge, logs via CloudWatch Logs.

---

## RDS Events

Events are generated for changes to:

| Source Type | Example Events |
|-------------|---------------|
| **DB instances** | State changed from pending → running, backup initiated, failover |
| **Snapshots** | Snapshot created, deleted, copied |
| **Parameter groups** | Parameter group modified |
| **Security groups** | Security group changed |
| **Clusters** | Cluster created, modified |

> Events are visible in the console for approximately **24 hours**.

---

## RDS Event Subscriptions

Subscribe to events and route them to **SNS** for notifications.

| Setting | Options |
|---------|---------|
| **Destination** | SNS topic (existing ARN or create new) |
| **Source type** | Instances, snapshots, security groups, parameter groups, clusters |
| **Instances** | All instances or specific instances |
| **Event categories** | All categories or specific ones (e.g., backup, failover, maintenance) |

**Console:** RDS → Events → Event subscriptions → Create event subscription

---

## EventBridge Integration

RDS events are also published to **Amazon EventBridge** automatically.

```
RDS Event (e.g., DB backup completed)
        ↓
EventBridge rule matches event pattern
        ↓
Target: Lambda / SNS / SQS / Step Functions
```

Use EventBridge when you need automated responses (not just notifications).

---

## RDS Database Logs

| Log Type | Content |
|----------|---------|
| **Error log** | Database errors and warnings |
| **General log** | All SQL statements executed |
| **Slow query log** | Queries exceeding a threshold execution time |
| **Audit log** | User activity, connections, queries (compliance) |

Logs can be viewed, watched, or downloaded directly from the console:

**Console:** RDS → DB instance → Logs & events tab

---

## Sending Logs to CloudWatch Logs

Enable log export during creation or by modifying the database:

**Console:** Modify DB → Additional configuration → Log exports → select logs to export

Once exported to CloudWatch Logs, you can:

```
CloudWatch Logs (RDS logs exported here)
        ↓
Metric Filter (e.g., count occurrences of "Error" keyword)
        ↓
CloudWatch Alarm (threshold exceeded)
        ↓
SNS Topic → notification to DBA
```

---

## Events vs Logs — Key Difference

| | RDS Events | Database Logs |
|-|-----------|--------------|
| **What it covers** | Database lifecycle (state changes, backups, failovers) | Database activity (queries, errors, connections) |
| **Destination** | SNS, EventBridge | CloudWatch Logs |
| **Alerting** | Event subscriptions | Metric filters + CloudWatch Alarms |
| **Use case** | Ops notifications | Performance monitoring, security auditing |

---

## Best Practices

✓ **Create event subscriptions for failover and backup events** — get notified when critical state changes occur  
✓ **Export error logs to CloudWatch Logs** — enables metric filters for automated alerting on errors  
✓ **Use EventBridge for automated remediation** — trigger Lambda functions in response to RDS events  
✓ **Enable slow query log** — identify performance issues before they impact production  
✓ **Use audit log for compliance** — track all user activity on the database  

---

## SysOps Exam Focus

**Q1: "You want to receive an SNS notification whenever an RDS automated backup completes. What should you configure?"**
- A) CloudWatch Logs metric filter on the RDS error log
- B) An RDS event subscription targeting the backup event category, sending to an SNS topic
- C) A CloudWatch alarm on the RDS BackupRetentionPeriod metric
- D) An EventBridge rule on the EC2 backup API call
- **Answer: B** — RDS event subscriptions allow per-category filtering and route notifications to SNS

**Q2: "A DBA wants to be alerted when the word 'ERROR' appears frequently in the RDS MySQL error log. What is the correct approach?"**
- A) Set up an RDS event subscription for error events
- B) Enable enhanced monitoring and set a threshold
- C) Export the error log to CloudWatch Logs, create a metric filter on 'ERROR', and set a CloudWatch alarm
- D) Use AWS Config to monitor log content
- **Answer: C** — Log-based alerting requires exporting to CloudWatch Logs, then applying a metric filter and alarm

**Q3: "You need an automated action to run whenever an RDS failover occurs. What service should you use?"**
- A) RDS event subscriptions → SNS → manual response
- B) Amazon EventBridge rule matching the RDS failover event → Lambda function
- C) CloudWatch Logs metric filter on the RDS error log
- D) AWS Config rule triggered on RDS state change
- **Answer: B** — EventBridge enables automated responses to RDS events; Lambda can execute remediation logic

**Q4: "Where do you configure RDS to send its MySQL error and slow query logs for analysis?"**
- A) RDS → Snapshots → Export
- B) RDS → Modify DB → Additional configuration → Log exports → select logs → CloudWatch Logs
- C) CloudWatch → Log groups → Create → attach to RDS
- D) RDS → Parameter group → enable logging parameters
- **Answer: B** — Log export to CloudWatch Logs is enabled in the DB modification settings under Additional configuration

---

## Quick Reference

```
RDS Events:
  Sources: instances, snapshots, parameter groups, security groups, clusters
  Destinations: SNS (event subscriptions) or EventBridge (automated rules)
  Console: RDS → Events → Event subscriptions

RDS Logs:
  Types: error, general, slow query, audit
  View: RDS → DB → Logs & events
  Export: Modify DB → Additional configuration → Log exports → CloudWatch Logs

Log alerting pipeline:
  RDS logs → CloudWatch Logs → Metric filter → CloudWatch Alarm → SNS → DBA

Events vs Logs:
  Events = lifecycle (state changes, backups, failover) → SNS / EventBridge
  Logs   = activity (queries, errors) → CloudWatch Logs + metric filters
```

---

**File: 6_RDS_Events_and_Logging.md**
**Status: SysOps-focused, exam-ready, concise format**
