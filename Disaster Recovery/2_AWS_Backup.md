# 2 — AWS Backup

## Overview

Fully managed service to centrally manage and automate backups across AWS services — no custom scripts or manual processes.

---

## Supported Services

EC2, EBS, S3, RDS (all engines), Aurora, DynamoDB, DocumentDB, Neptune, EFS, FSx (Lustre + Windows File Server), AWS Storage Gateway (Volume Gateway)

---

## Key Features

| Feature | Detail |
|---|---|
| **Cross-region backups** | Push backups to another region for DR |
| **Cross-account backups** | Backup across multiple AWS accounts |
| **Point-in-time recovery** | Supported for eligible services (e.g., Aurora) |
| **On-demand and scheduled** | Both supported |
| **Tag-based policies** | Backup only resources with specific tags (e.g., `Environment=production`) |
| **Backup storage** | Internal S3 bucket managed by AWS Backup |

---

## Backup Plans

Define a reusable backup policy:

| Setting | Options |
|---|---|
| **Frequency** | Every 12 hours, daily, weekly, monthly, or custom cron expression |
| **Backup Window** | Time window for when backup runs |
| **Cold Storage Transition** | Never, or after N days/weeks/months/years |
| **Retention Period** | Always, or N days/weeks/months/years |

---

## Vault Lock (WORM)

Enforces **Write Once Read Many** on backup vault — backups cannot be deleted or altered.

- Protects against inadvertent or malicious deletion
- Protects against retention period shortening
- **Even the root user cannot delete backups** when Vault Lock is enabled

```
Vault Lock enabled → backups are immutable
  → cannot delete
  → cannot shorten retention period
  → root user: still blocked
```

---

## Quick Reference

```
AWS Backup: central, automated backup management across all major AWS services

Backup Plan settings:
  Frequency: cron/hourly/daily/weekly/monthly
  Cold storage transition: after N days/weeks/months/years
  Retention: configurable

Features: cross-region, cross-account, PITR, tag-based policies
Storage: internal S3 bucket (managed by AWS Backup)

Vault Lock (WORM):
  Backups immutable — cannot delete or alter retention
  Even root user is blocked
  Use for: compliance, protection against malicious/accidental deletion
```
