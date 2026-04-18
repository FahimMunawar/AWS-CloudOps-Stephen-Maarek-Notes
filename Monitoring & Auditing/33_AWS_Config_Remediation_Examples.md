# 33 — AWS Config: Remediation Examples

## Pattern

```
Config rule → resource non-compliant → trigger SSM Automation Document → resource fixed
```

---

## Example 1: Disable Incoming SSH (Port 22)

| Step | Detail |
|---|---|
| **Resource** | EC2 Security Group with port 22 open |
| **Config Rule** | Flags SG as non-compliant |
| **SSM Document** | `AWS-DisableIncomingSSHOnPort22` |
| **Action** | Removes the port 22 inbound rule from the security group |

```
EC2 SG: port 22 open
  → Config rule: NON-COMPLIANT
    → SSM Automation: AWS-DisableIncomingSSHOnPort22
      → Port 22 inbound rule removed → COMPLIANT
```

---

## Example 2: Enable S3 Bucket Logging

| Step | Detail |
|---|---|
| **Resource** | S3 bucket with logging disabled |
| **Config Rule** | Flags bucket as non-compliant |
| **SSM Document** | `AWS-ConfigureS3BucketLogging` |
| **Action** | Enables access logging on the bucket |

```
S3 bucket: logging disabled
  → Config rule: NON-COMPLIANT
    → SSM Automation: AWS-ConfigureS3BucketLogging
      → Logging enabled → COMPLIANT
```

---

## Quick Reference

```
AWS-DisableIncomingSSHOnPort22   → removes port 22 inbound rule from SG
AWS-ConfigureS3BucketLogging     → enables access logging on S3 bucket

Both are AWS-managed SSM Automation Documents — no custom code needed
Triggered automatically by Config auto-remediation
```
