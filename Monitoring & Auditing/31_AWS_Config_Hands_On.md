# 31 — AWS Config: Hands-On

## Initial Setup

1. Config → **Get started**
2. Recording scope: **All supported resources in this region** (or specific types)
3. Toggle **Include global resources** (IAM users, groups, roles, managed policies) — adds cost
4. Create a **Config service-linked role** (auto-created)
5. **S3 bucket**: auto-named, stores all configuration history
6. **SNS notifications**: optional — streams all changes to one topic
7. Click through to confirm — role and bucket are created, recording begins

> Resources are discovered progressively — may take time for all to appear.

---

## Viewing Resources

Config → **Resources** → filter by resource type (e.g., EC2 Security Group)

- **No compliance status** = no rules defined yet
- Click a resource → view:
  - **Rules applied** and their compliance status
  - **Resource configuration** (current settings)
  - **Resource timeline** — configuration changes + linked CloudTrail events

---

## Adding a Rule

Config → **Rules** → **Add rule** → AWS Managed Rule or Custom (Lambda)

### Example: `restricted-ssh`
- Checks that no security group allows unrestricted inbound SSH (port 22 from `0.0.0.0/0`)
- **Trigger**: on resource configuration change (EC2 Security Groups)
- No additional parameters required

After adding:
- Evaluation runs automatically
- Resources tagged **Compliant** or **Non-compliant**

---

## Compliance Workflow Demo

```
Non-compliant SG: port 22 open from anywhere (0.0.0.0/0)
  → restricted-ssh rule = NON-COMPLIANT

Fix: delete the port 22 inbound rule from the security group
  → Config detects configuration change
    → Rule re-evaluates
      → Resource now COMPLIANT
```

Resource timeline shows the full sequence:
1. Initial configuration recorded
2. Rule evaluated → non-compliant
3. CloudTrail event: `RevokeSecurityGroupIngress`
4. Configuration change recorded (port 22 rule deleted)
5. Rule re-evaluated → compliant

---

## Setting Up Remediation

Resource → Rule → **Actions** → **Manage remediation**

| Option | Detail |
|---|---|
| **Manual** | Triggered manually when needed |
| **Automatic** | Triggered when resource is non-compliant; configure retries + retry interval |

Remediation action = an **SSM Automation Document** (AWS-managed or custom).

> Choose a remediation document that makes sense for the specific rule — the document must match the non-compliance being fixed.

---

## Cross-Account Aggregation

Config → **Aggregators** — aggregate Config data across multiple accounts and regions into one central view.

---

## Notifications (from Settings)

| Method | Setup |
|---|---|
| **SNS (all events)** | Configure in Settings → SNS topic |
| **EventBridge (filtered)** | Create EventBridge rules in CloudWatch Events console to intercept specific non-compliance events |

---

## Quick Reference

```
Setup: record all resources → S3 bucket → service-linked role
Global resources (IAM): optional, adds cost

Rules: AWS Managed (75+) or Custom (Lambda)
  restricted-ssh: flags SGs with port 22 open to 0.0.0.0/0
  Trigger: on config change OR periodic

Resource timeline: config changes → CloudTrail events → compliance evaluations

Remediation: SSM Automation Document
  Manual or Automatic (with retries)
  Must match the compliance issue being fixed

Aggregators: cross-account/region Config data in one view
Notifications: SNS (all) or EventBridge (filtered by rule/event type)
```
