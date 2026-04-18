# 7 — SCP Examples

## Pattern

SCPs use IAM condition keys to enforce governance rules on resource creation and API calls.

---

## Example 1: Require a Tag to Launch EC2

Deny `RunInstances` if the `department` tag is missing:

```json
{
  "Effect": "Deny",
  "Action": "ec2:RunInstances",
  "Resource": "arn:aws:ec2:*:*:instance/*",
  "Condition": {
    "Null": { "aws:RequestedRegion": "true" }
  }
}
```

> If the `department` tag is absent, the instance cannot be launched.

---

## Example 2: Enforce Tag Value Format

Deny `RunInstances` if the `business-unit` tag does not start with `infra-`:

```json
{
  "Effect": "Deny",
  "Action": "ec2:RunInstances",
  "Resource": "arn:aws:ec2:*:*:instance/*",
  "Condition": {
    "StringNotLike": { "aws:RequestTag/business-unit": "infra-*" }
  }
}
```

> Not just requiring the tag — enforcing a naming convention on its value.

---

## Example 3: Restrict Allowed Tag Values

Deny `RunInstances` if the `deployment-type` tag is not `in-region` or `edge`:

```json
{
  "Effect": "Deny",
  "Action": "ec2:RunInstances",
  "Resource": "arn:aws:ec2:*:*:instance/*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestTag/deployment-type": ["in-region", "edge"]
    }
  }
}
```

> Restricts valid values to a predefined allowlist.

---

## Example 4: Restrict API Calls to Specific Regions

Deny all actions if the request is not made in `eu-central-1` or `us-east-1`:

```json
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["eu-central-1", "us-east-1"]
    }
  }
}
```

> Common exam SCP — locks all API calls to approved regions only.

---

## Condition Keys Used

| Condition Key | Purpose |
|---|---|
| `aws:RequestTag/<key>` | Check tags submitted with the resource creation request |
| `Null` condition | Check whether a tag key is present or absent |
| `StringNotLike` | Deny if value doesn't match a pattern (supports wildcards) |
| `StringNotEquals` | Deny if value doesn't exactly match allowed values |
| `aws:RequestedRegion` | Restrict API calls to specific AWS regions |

---

## Quick Reference

```
SCP tag enforcement:
  Missing tag            → Null condition on aws:RequestTag/<key>
  Tag value format       → StringNotLike with wildcard (e.g., "infra-*")
  Tag value allowlist    → StringNotEquals with array of allowed values

Region restriction:
  aws:RequestedRegion + StringNotEquals → deny all actions outside approved regions
  Action: *, Resource: * → blocks everything not in the allowlist
```
