# 19 — AWS Secrets Manager

## Overview

A managed service for storing, rotating, and managing secrets — distinct from SSM Parameter Store by its built-in rotation capability.

| Feature | Detail |
|---|---|
| **Rotation** | Force rotation every X days (automated) |
| **Secret generation** | Lambda function generates new secret on rotation |
| **Encryption** | KMS-encrypted |
| **RDS/Aurora integration** | Native — stores and rotates DB credentials automatically |

---

## Key Differentiator vs SSM Parameter Store

Secrets Manager is purpose-built for secrets with **mandatory rotation support** and native database integration. When the exam mentions secrets + RDS/Aurora + rotation → think **Secrets Manager**.

---

## Rotation

- Define a rotation schedule (every X days)
- A Lambda function is invoked to generate the new secret value
- The new secret is stored and the old one is retired
- Native integrations (RDS MySQL, PostgreSQL, Aurora, etc.) handle credential updates automatically

---

## Multi-Region Secrets

Replicate a secret across multiple AWS regions — Secrets Manager keeps replicas in sync with the primary.

```
Primary Secret (us-east-1)
  └── Replica Secret (eu-west-1)  ← kept in sync automatically
```

**Use cases:**
- **Disaster recovery** — promote replica to standalone if primary region fails
- **Multi-region apps** — same secret used in multiple regions
- **Cross-region RDS replication** — same credentials access the replicated DB in another region

---

## SysOps Exam Q&A

**Q: An application needs to automatically rotate its RDS database password every 30 days. What service do you use?**
A: AWS Secrets Manager — it natively integrates with RDS and can automatically rotate credentials on a schedule using a Lambda function.

**Q: What is required to automate secret generation on rotation in Secrets Manager?**
A: A Lambda function that generates the new secret value — Secrets Manager invokes it on the rotation schedule.

**Q: You have an RDS database replicated to a secondary region. How do you give the secondary region app access to the DB credentials?**
A: Use Secrets Manager multi-region secrets — replicate the primary secret to the secondary region so the same credentials are available locally.

**Q: How are secrets encrypted in Secrets Manager?**
A: Using AWS KMS.

---

## Quick Reference

```
Secrets Manager:
  Store + rotate secrets (every X days) via Lambda function
  KMS-encrypted
  Native RDS/Aurora integration — rotates DB credentials automatically

Exam trigger: secrets + RDS/Aurora + rotation → Secrets Manager

Multi-region secrets:
  Primary → replica(s) kept in sync
  Use cases: DR (promote replica), multi-region apps, cross-region RDS replication
```
