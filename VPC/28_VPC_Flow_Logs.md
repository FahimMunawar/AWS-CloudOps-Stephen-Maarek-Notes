# 28 — VPC Flow Logs

## Overview

Capture metadata about IP traffic flowing through VPC interfaces — for monitoring and troubleshooting connectivity.

| Capture Level | Scope |
|---|---|
| VPC | All traffic in the entire VPC |
| Subnet | All traffic in a specific subnet |
| ENI | Traffic on a specific elastic network interface |

Also captures managed interfaces: ELB, RDS, ElastiCache, Redshift, Workspaces, NAT Gateway, Transit Gateway.

---

## Log Destinations

- Amazon S3 (best for analysis with Athena)
- CloudWatch Logs (best for streaming analysis with Logs Insights)
- Kinesis Data Firehose

---

## Flow Log Record Fields

```
version | account-id | interface-id | srcaddr | dstaddr | srcport | dstport | protocol | packets | bytes | start | end | action | log-status
```

| Field | Use |
|---|---|
| `srcaddr` / `dstaddr` | Identify problematic IPs (repeated denies, attack sources) |
| `srcport` / `dstport` | Identify problematic ports |
| `action` | `ACCEPT` or `REJECT` — whether SG or NACL allowed/denied |

---

## Troubleshooting SG vs NACL with Flow Logs

Use the `action` field on inbound and outbound records:

| Inbound | Outbound | Root Cause |
|---|---|---|
| REJECT | — | NACL **or** SG blocking inbound |
| ACCEPT | REJECT | **NACL only** — SG is stateful, so if inbound accepted, outbound auto-allowed |
| — | REJECT | NACL **or** SG blocking outbound |
| ACCEPT (outbound) | REJECT (inbound return) | **NACL only** — stateless, must explicitly allow return traffic |

> Key insight: if inbound is ACCEPT but outbound is REJECT → must be a NACL issue (SG stateful return traffic would auto-allow).

---

## Architectures

### 1. Top IP Contributors

```
VPC Flow Logs → CloudWatch Logs → CloudWatch Contributor Insights
  → Top 10 IP addresses contributing most traffic
```

### 2. SSH/RDP Attack Detection

```
VPC Flow Logs → CloudWatch Logs
  → Metric Filter (SSH port 22 / RDP port 3389)
    → CloudWatch Alarm (unusual spike)
      → SNS notification
```

### 3. SQL Analysis

```
VPC Flow Logs → S3
  → Athena (SQL queries on flow log data)
    → QuickSight (visualization)
```

---

## IAM Permissions Required

The VPC Flow Logs service role must have:

```
logs:CreateLogGroup
logs:CreateLogStream
logs:PutLogEvents
```

Without these, flow logs will fail to publish to CloudWatch Logs.

---

## SysOps Exam Q&A

**Q: A VPC flow log shows inbound ACCEPT but outbound REJECT. Is this a security group or NACL issue?**
A: NACL — security groups are stateful and automatically allow return traffic for accepted inbound connections. An outbound REJECT after inbound ACCEPT must be the NACL blocking the response.

**Q: How do you run SQL queries against VPC flow logs?**
A: Send flow logs to S3, then use Amazon Athena to query them. Visualize with QuickSight.

**Q: You want to alert when SSH traffic to your VPC spikes unexpectedly. How?**
A: VPC Flow Logs → CloudWatch Logs → Metric Filter (port 22) → CloudWatch Alarm → SNS.

**Q: What permissions does the VPC Flow Logs IAM role need to publish to CloudWatch Logs?**
A: `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`.

---

## Quick Reference

```
VPC Flow Logs: capture IP traffic metadata at VPC / subnet / ENI level

Destinations: S3 (Athena analysis), CloudWatch Logs (Insights/alarms), Kinesis Firehose

Troubleshoot NACL vs SG:
  Inbound ACCEPT + outbound REJECT → NACL issue (SG is stateful)
  Inbound REJECT → NACL or SG issue

Architectures:
  Top IPs: Flow Logs → CW Logs → Contributor Insights
  SSH/RDP alert: Flow Logs → CW Logs → Metric Filter → Alarm → SNS
  SQL analysis: Flow Logs → S3 → Athena → QuickSight

IAM role needs: logs:CreateLogGroup, logs:CreateLogStream, logs:PutLogEvents
```
