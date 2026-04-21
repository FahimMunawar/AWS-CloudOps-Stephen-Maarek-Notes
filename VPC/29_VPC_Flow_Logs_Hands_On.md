# 29 — VPC Flow Logs (Hands-On)

## Goal

Create two flow logs (S3 + CloudWatch Logs), observe live traffic, and query logs with Athena.

---

## Flow Log 1: Send to Amazon S3

**VPC → DemoVPC → Flow Logs → Create flow log**

| Setting | Value |
|---|---|
| Name | `DemoS3FlowLog` |
| Filter | All |
| Aggregation interval | 1 min (demo) — use 10 min in production |
| Destination | Amazon S3 |
| S3 bucket ARN | Create bucket `demo-vpc-flow-logs` → copy ARN |

> AWS automatically creates and attaches the required bucket policy — no manual bucket policy needed.

---

## Flow Log 2: Send to CloudWatch Logs

**VPC → DemoVPC → Flow Logs → Create flow log**

| Setting | Value |
|---|---|
| Name | `DemoFlowLogCloudWatch` |
| Filter | All |
| Destination | CloudWatch Logs |
| Log group | Create `VPCFlowLogs` in CloudWatch (retention: 1 day for demo) |
| IAM role | Create `flowlogsrole` |

### IAM Role for Flow Logs

**IAM → Create role → Custom trust policy:**

```json
{
  "Principal": {
    "Service": "vpc-flow-logs.amazonaws.com"
  }
}
```

**Permission policy:** `CloudWatchLogsFullAccess`

---

## Observing Flow Logs in CloudWatch

Each ENI gets its own log stream. Find your instance's ENI ID (EC2 → Networking → ENI ID) and match it to the log stream.

**Typical traffic observed:**
- Repeated `REJECT` entries from external IPs = internet scanners/attackers probing the instance
- `ACCEPT` entries = legitimate traffic (e.g., your SSH sessions, curl requests)

> If a specific IP is repeatedly attacking, you can add a NACL deny rule to block it.

---

## Querying Flow Logs with Athena

### Step 1: Set Athena Query Result Location

**Athena → Settings → Manage** → specify S3 bucket: `s3://demo-athena-bucket/Athena/`

### Step 2: Create Table (from AWS documentation template)

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS vpc_flow_logs (
  version int, account string, interfaceid string,
  sourceaddress string, destinationaddress string,
  sourceport int, destinationport int, protocol int,
  numpackets int, numbytes bigint, starttime int,
  endtime int, action string, logstatus string
)
PARTITIONED BY (date date, region string, account_id string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
LOCATION 's3://<your-vpc-flow-logs-bucket>/AWSLogs/<account-id>/vpcflowlogs/<region>/';
```

### Step 3: Add Partition

```sql
ALTER TABLE vpc_flow_logs ADD PARTITION (date='2024-01-01', region='eu-central-1', account_id='<account-id>')
LOCATION 's3://<bucket>/AWSLogs/<account-id>/vpcflowlogs/<region>/2024/01/01/';
```

> AWS Glue can automate partition management for ongoing analysis.

### Step 4: Query — Find All Rejected Traffic

```sql
SELECT * FROM vpc_flow_logs WHERE action = 'REJECT' LIMIT 100;
```

Useful follow-up queries: group by `sourceaddress` to rank top attacking IPs.

---

## Use Cases Summary

| Destination | Best For |
|---|---|
| CloudWatch Logs | Real-time metric filters, alarms for ongoing attacks |
| Amazon S3 + Athena | Batch analysis, complex SQL queries, QuickSight visualization |

---

## Cleanup

Delete both flow logs and the S3 buckets to avoid ongoing costs.

---

## Quick Reference

```
Flow Log → S3: AWS auto-creates bucket policy; use Athena for SQL analysis
Flow Log → CW Logs: create log group + IAM role (vpc-flow-logs.amazonaws.com trust)

IAM role trust: Service = vpc-flow-logs.amazonaws.com
  Permissions: CloudWatchLogsFullAccess

Athena setup:
  1. Set query result S3 location
  2. CREATE EXTERNAL TABLE with flow log schema
  3. ALTER TABLE ADD PARTITION for date/region
  4. SELECT * WHERE action = 'REJECT'

Aggregation: 1 min = more records (demo); 10 min = cost-efficient (production)
```
