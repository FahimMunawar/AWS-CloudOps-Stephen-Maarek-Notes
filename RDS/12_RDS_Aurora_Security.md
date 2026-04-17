# 12. RDS and Aurora Security

## Overview

RDS and Aurora provide multiple layers of security: encryption at rest and in transit, IAM-based authentication, network access controls, and audit logging. These are exam-tested topics — know each layer and its constraints.

---

## Encryption at Rest

| Property | Detail |
|----------|--------|
| **Mechanism** | KMS encryption on the storage volume |
| **Scope** | Master DB and all read replicas encrypted together |
| **When set** | Defined at **launch time** — cannot be changed after creation |
| **Replicas** | If master is unencrypted, read replicas **cannot** be encrypted |

### Encrypting an Existing Unencrypted Database

You cannot enable encryption in place. The process is:

```
Unencrypted DB
        ↓
Take a snapshot (unencrypted)
        ↓
Restore snapshot as encrypted (choose KMS key at restore time)
        ↓
New encrypted DB cluster/instance
```

---

## Encryption in Transit (In-Flight)

| Property | Detail |
|----------|--------|
| **Default** | TLS/SSL enabled by default on all RDS and Aurora instances |
| **Client requirement** | Must use AWS TLS root certificates (available on the AWS website) |
| **Configuration** | No extra setup needed on the database side — enforce on the client |

---

## Authentication

| Method | Description |
|--------|-------------|
| **Username + Password** | Classic credential-based authentication |
| **IAM Authentication** | EC2 instances (or other AWS services) authenticate using their IAM role — no username/password needed |

> IAM authentication eliminates the need to manage database credentials for AWS workloads.

---

## Network Access Control

- Controlled via **VPC Security Groups**
- Allow or deny specific IPs, port ranges, or security groups
- Best practice: allow only the application tier security group, not `0.0.0.0/0`

---

## SSH Access

| Service | SSH Access |
|---------|-----------|
| Standard RDS / Aurora | **Not available** — managed service |
| RDS Custom | **Available** — AWS exposes the underlying OS |

---

## Audit Logs

| Property | Detail |
|----------|--------|
| **Purpose** | Record all queries, connections, and activity on the database |
| **Retention** | Logs expire after a short period within RDS/Aurora |
| **Long-term retention** | Export to **CloudWatch Logs** to retain indefinitely |

```
Audit logs enabled on RDS/Aurora
        ↓
Logs available in the RDS console (short-lived)
        ↓
Export to CloudWatch Logs → retain as long as needed
```

---

## Security Summary

| Layer | Mechanism |
|-------|-----------|
| At-rest encryption | KMS — set at launch, covers master + replicas |
| In-transit encryption | TLS by default — client uses AWS root certificates |
| Authentication | Username/password or IAM roles |
| Network | VPC Security Groups |
| OS access | None (managed) — RDS Custom is the exception |
| Audit | Audit Logs → CloudWatch Logs for long-term retention |

---

## Best Practices

✓ **Enable encryption at launch** — you cannot enable it later without snapshot/restore  
✓ **Use IAM authentication for EC2 workloads** — eliminates credential management  
✓ **Export audit logs to CloudWatch Logs** — RDS native retention is too short for compliance  
✓ **Restrict security groups to the app tier only** — never open database ports to the internet  
✓ **Enforce TLS on the client side** — in-flight encryption is available by default but must be used  

---

## SysOps Exam Focus

**Q1: "You have an existing unencrypted RDS database and need to encrypt it. What is the correct process?"**
- A) Enable encryption in the Modify settings
- B) Take a snapshot of the unencrypted database, then restore it as an encrypted database using KMS
- C) Enable KMS on the existing instance and restart it
- D) Create a Read Replica with encryption enabled and promote it
- **Answer: B** — Encryption cannot be enabled on an existing instance; snapshot → restore as encrypted is the only path

**Q2: "An EC2 application needs to connect to RDS without storing database credentials. What should you use?"**
- A) Store credentials in environment variables on the EC2 instance
- B) Use AWS Secrets Manager with automatic rotation
- C) Enable IAM database authentication and use the EC2 instance's IAM role to authenticate
- D) Use a hardcoded username and password in the application config
- **Answer: C** — IAM authentication allows AWS services with IAM roles to authenticate to RDS without passwords

**Q3: "Your security team requires that all database queries be logged and retained for 90 days. How do you achieve this with RDS?"**
- A) Enable automated backups with a 90-day retention period
- B) Enable audit logs on RDS and export them to CloudWatch Logs; set the log group retention to 90 days
- C) Enable enhanced monitoring — it captures all query activity
- D) Use RDS Performance Insights with the paid tier for 90-day retention
- **Answer: B** — Audit logs capture query activity; they must be exported to CloudWatch Logs for long-term retention since RDS native retention is short

**Q4: "You create an unencrypted RDS primary instance and then try to create an encrypted read replica. What happens?"**
- A) The encrypted replica is created successfully
- B) AWS automatically encrypts the replica using the default KMS key
- C) The operation fails — if the primary is unencrypted, read replicas cannot be encrypted
- D) The replica is created but encryption is applied only to new writes
- **Answer: C** — Read replica encryption requires the primary to also be encrypted; you cannot mix encrypted and unencrypted in the same replication chain

---

## Quick Reference

```
At-rest encryption:
  KMS, set at launch only
  Master + replicas encrypted together
  Unencrypted master → replicas cannot be encrypted
  Encrypt existing DB: snapshot → restore as encrypted

In-transit encryption:
  TLS enabled by default
  Client must use AWS TLS root certificates

Authentication:
  Username/password (classic)
  IAM roles (no credentials needed — for EC2/AWS services)

Network: VPC Security Groups — restrict to app tier only

SSH: Not available (RDS Custom is the exception)

Audit logs:
  Enable on RDS/Aurora → short-lived in console
  Export to CloudWatch Logs for long-term retention
```

---

**File: 12_RDS_Aurora_Security.md**
**Status: SysOps-focused, exam-ready, concise format**
