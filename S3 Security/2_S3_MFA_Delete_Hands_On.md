# 2 — S3 MFA Delete — Hands-On

## Overview

MFA Delete can only be enabled/disabled via the **AWS CLI** using **root account credentials**. The S3 console does not provide this option.

---

## Prerequisites

1. **S3 bucket with versioning enabled**
2. **Virtual MFA device registered to the root account**
   - IAM → Security Credentials → Multi-Factor Authentication → note the MFA device ARN
3. **Root account access keys** (created temporarily — delete after use)

> **Warning:** Root access keys are extremely sensitive. Create them only for this task and **delete immediately after**.

---

## Step-by-Step

### Step 1 — Create Root Access Keys
- Log in as **root** → click account name → **Security Credentials**
- Multi-Factor Authentication (MFA) → note the **MFA device ARN**
- Access Keys → **Create access key** → download the key file

### Step 2 — Configure AWS CLI Profile for Root
```bash
aws configure --profile root-mfa-delete-demo
# Enter: Access Key ID, Secret Access Key, Region (e.g., eu-west-1)
```

Verify it works:
```bash
aws s3 ls --profile root-mfa-delete-demo
```

### Step 3 — Enable MFA Delete
```bash
aws s3api put-bucket-versioning \
  --bucket demo-stephane-mfa-delete-2020 \
  --versioning-configuration Status=Enabled,MFADelete=Enabled \
  --mfa "arn:aws:iam::ACCOUNT-ID:mfa/root-account-mfa-device CURRENT-MFA-CODE" \
  --profile root-mfa-delete-demo
```

- Replace the ARN with your MFA device ARN from Step 1
- Replace `CURRENT-MFA-CODE` with the live code from your MFA app (e.g., `710343`)
- MFA codes expire every 30 seconds — if it fails, wait for the next code

### Step 4 — Verify in Console
S3 → Bucket → Properties → Bucket Versioning:
- Shows: `Bucket versioning: Enabled`
- Shows: `Multi-Factor Authentication (MFA) delete: Enabled`

### Step 5 — Test MFA Delete is Enforced
1. Upload an object to the bucket
2. Delete the object (adds a delete marker — this still works, no MFA needed)
3. Try to **permanently delete a specific version ID** via the console
4. Result: **Error** — *"You cannot delete this object because MFA delete is enabled"*

Only a CLI command with a valid MFA code can permanently delete a version.

### Step 6 — Disable MFA Delete
```bash
aws s3api put-bucket-versioning \
  --bucket demo-stephane-mfa-delete-2020 \
  --versioning-configuration Status=Enabled,MFADelete=Disabled \
  --mfa "arn:aws:iam::ACCOUNT-ID:mfa/root-account-mfa-device CURRENT-MFA-CODE" \
  --profile root-mfa-delete-demo
```

### Step 7 — Delete Root Access Keys (Critical)
- IAM → Security Credentials → Access Keys → **Deactivate** → **Delete**
- Never leave root access keys active

---

## Key Behaviours Observed

| Action | MFA Required? | Result |
|---|---|---|
| Delete object (adds delete marker) | No | Succeeds — soft delete still works |
| Permanently delete a version via console | N/A | **Blocked** — console cannot do this when MFA Delete is on |
| Permanently delete a version via CLI (with MFA code) | Yes | Succeeds |
| Enable/disable MFA Delete | Yes (root only) | CLI only |

---

## SysOps Exam Q&A

**Q: How do you enable MFA Delete on an S3 bucket?**
A: Via **AWS CLI** using the `aws s3api put-bucket-versioning` command with `MFADelete=Enabled`, the MFA device ARN, and a current MFA code — using the **root account**. Cannot be done from the console.

**Q: A user tries to permanently delete an S3 object version and gets an error saying MFA delete is enabled. What must they do?**
A: Use the **AWS CLI** with a valid **MFA code** to perform the deletion. Console-based permanent deletion is blocked when MFA Delete is enabled.

**Q: After enabling MFA Delete, can you still perform soft deletes (adding delete markers)?**
A: **Yes** — soft deletes (no version ID specified) do not require MFA. Only permanent version deletions and suspending versioning require MFA.

---

## Quick Reference

```
MFA Delete: CLI only (not console)
Account required: root account only

Enable:
  aws s3api put-bucket-versioning \
    --bucket BUCKET_NAME \
    --versioning-configuration Status=Enabled,MFADelete=Enabled \
    --mfa "MFA_DEVICE_ARN CURRENT_CODE" \
    --profile root-profile

Disable: same command with MFADelete=Disabled

After use: ALWAYS delete root access keys immediately
Soft delete (delete marker): allowed without MFA
Permanent version delete:     blocked in console; requires CLI + MFA code
```
