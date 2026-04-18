# 32 — AWS Config: Aggregators

## Overview

An AWS Config **Aggregator** collects Config data (rules, resources, compliance) from multiple source accounts and regions into a single central view.

---

## Architecture

```
Account A (source)   Account B (source)
  AWS Config           AWS Config
      │                    │
      └────────┬───────────┘
               ▼
     Aggregator Account (central)
       └── Aggregator
             └── Aggregated view: all compliant/non-compliant resources
                                  across all accounts and all regions
```

---

## Key Rules

| Item | Detail |
|---|---|
| **Where aggregator is created** | Central account only — NOT in each source account |
| **What it aggregates** | Rules, resources, compliance status across accounts + regions |
| **What it does NOT do** | Deploy or manage rules centrally |
| **Rules management** | Done at the individual source account level |

---

## Authorization

| Scenario | Authorization Required |
|---|---|
| **Using AWS Organizations** | No manual authorization needed — automatic |
| **Not using AWS Organizations** | Must create an authorization in **each** source account allowing the aggregator account to collect data |

> With Organizations: create aggregator in the management account — authorizations are automatic.
> Without Organizations: create authorization in Account A → then Account B → then aggregator can pull data.

---

## Deploying Rules Across Accounts

The aggregator only collects and displays data — it does not deploy rules.

To deploy Config rules to multiple accounts and regions, use **CloudFormation StackSets**.

```
CloudFormation StackSets
  └── Deploy Config rules as a CloudFormation Stack
        └── Across multiple accounts + multiple regions simultaneously
```

---

## SysOps Exam Q&A

**Q: Where do you create a Config Aggregator?**
A: In one **central aggregator account only** — not in each source account.

**Q: Do you need to authorize each source account when using AWS Organizations?**
A: **No** — Organizations handles authorization automatically. Manual per-account authorization is only needed when NOT using Organizations.

**Q: Can you use a Config Aggregator to centrally manage and deploy Config rules?**
A: **No** — the aggregator only aggregates data (rules, resources, compliance). To deploy rules across accounts/regions, use **CloudFormation StackSets**.

**Q: How do you get a single compliance view across all accounts and regions?**
A: Create a Config **Aggregator** in a central account — it aggregates compliant/non-compliant resource data from all source accounts and regions.

---

## Quick Reference

```
Aggregator: created in ONE central account
  Aggregates: rules + resources + compliance from all source accounts + regions
  Does NOT: deploy or manage rules centrally

Authorization:
  With AWS Organizations     → automatic (create aggregator in management account)
  Without AWS Organizations  → manual authorization required in EACH source account

Rules deployment across accounts/regions:
  Use CloudFormation StackSets (not the aggregator)
```
