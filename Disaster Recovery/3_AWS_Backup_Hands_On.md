# 3 — AWS Backup: Hands-On

## Creating a Backup Plan

**AWS Backup → Create Backup plan**

Three options:
- **Start with a template** — quickest; pre-built rules (e.g., `Daily-35day-Retention`, `Daily-Monthly-1yr-Retention`)
- **Build a new plan** — custom rules from scratch
- **Define plan using JSON** — programmatic

---

## Backup Rules (within a plan)

A plan can have **multiple rules** (e.g., daily + monthly):

| Setting | Detail |
|---|---|
| **Backup vault** | Default (AWS-managed) or create your own |
| **Frequency** | Hourly / Daily / Weekly / Monthly / Custom cron |
| **Backup window** | Start time (e.g., 5:00 AM UTC) + window duration |
| **Cold storage transition** | Never, or after N days/weeks/months/years |
| **Retention period** | e.g., 5 weeks (daily), 1 year (monthly) |
| **Copy to another region** | Optional — for cross-region DR |

---

## Assigning Resources

After creating the plan → **Assign resources**

| Option | Detail |
|---|---|
| **Specific resource types** | Select service (e.g., DynamoDB) + specific resource or all |
| **All resource types + tags** | Back up any resource matching a tag condition |

### Tag-Based Assignment Example

```
Key: environment
Value: production
  → Any resource (EC2, EBS, RDS, etc.) tagged environment=production
      is automatically included in this backup plan
```

**IAM role**: use the default role (auto-created with correct permissions) or specify your own.

---

## End-to-End Flow

```
Create Backup Plan (rules: frequency, retention, cold storage, cross-region copy)
  → Assign resources (by resource type or tag)
    → Backups run automatically on schedule
      → Stored in Backup Vault
```

---

## Quick Reference

```
Create plan: template / custom / JSON
  Rules per plan: multiple (e.g., daily + monthly)
  Each rule: vault, frequency, window, cold transition, retention, cross-region copy

Assign resources:
  Specific type + resource  → explicit selection
  All types + tag condition → dynamic (e.g., environment=production)

IAM role: default (auto-created) or custom
Backup output: stored in Backup Vault (viewable in AWS Backup console)
```
