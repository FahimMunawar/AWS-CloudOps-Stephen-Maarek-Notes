# 6 — AWS Organizations: SysOps Exam Details

## 1. Reserved Instance & Savings Plans Sharing

- By default, all accounts in an organization share RIs and Savings Plans
- Any account can use another account's unused Reserved Instance → cost savings
- **Sharing can be disabled** per account (including the payer/management account)
- For RI or Savings Plans discounts to apply between two accounts, **both must have sharing turned on**

---

## 2. `aws:PrincipalOrgID` IAM Condition

Allows access to any IAM principal (user or role) from **any account within your organization** — without listing individual account IDs.

### Use Case: S3 Bucket Policy for Org-Wide Access

```json
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": "s3:*",
  "Resource": "arn:aws:s3:::my-bucket/*",
  "Condition": {
    "StringEquals": {
      "aws:PrincipalOrgID": "o-xxxxxxxxxxxx"
    }
  }
}
```

> Benefit: reference the organization ID once instead of specifying every account individually. Any new account added to the org automatically gets access.

---

## 3. Tag Policies

- Enforce **standardized tagging** across all accounts in the organization
- Define allowed tag keys and valid values per resource type
- Goals: resource categorization, cost allocation, attribute-based access control (ABAC)

### Non-Compliance Handling

| Method | Detail |
|---|---|
| **Compliance report** | Generate a list of all non-compliant resources across the org |
| **CloudWatch Events** | Monitor and alert on non-compliant tags in real time |

---

## SysOps Exam Q&A

**Q: Two accounts in an organization want to share Reserved Instance discounts. What must be true?**
A: **Both accounts must have RI sharing enabled.** If either account has sharing disabled, the RI discount does not apply between them.

**Q: How do you grant S3 bucket access to all accounts in your organization without listing each account ID?**
A: Use the `aws:PrincipalOrgID` condition in the bucket policy — it matches any IAM principal from any account in the organization.

**Q: What is the purpose of Tag Policies in AWS Organizations?**
A: To enforce standardized tag keys and values across all accounts, supporting cost allocation, resource categorization, and ABAC. Non-compliant resources can be identified via a report or monitored with CloudWatch Events.

---

## Quick Reference

```
RI / Savings Plans sharing:
  Default: ON for all accounts in org
  Both accounts must have sharing enabled for discounts to apply
  Can disable per account (including management account)

aws:PrincipalOrgID:
  IAM condition key — matches any principal in the org
  Use in resource policies (e.g., S3 bucket policy) instead of listing account IDs
  New accounts added to org automatically qualify

Tag Policies:
  Enforce standard tag keys + allowed values
  Non-compliance: generate report OR monitor with CloudWatch Events
  Useful for: cost allocation, ABAC, resource categorization
```
