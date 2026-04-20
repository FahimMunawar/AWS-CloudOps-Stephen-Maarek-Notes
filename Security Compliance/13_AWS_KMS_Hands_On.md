# 13. AWS KMS — Hands-On

## Overview

Walkthrough of creating a customer managed KMS key, reviewing key policies and rotation, and using the CLI to encrypt and decrypt a file.

---

## Part 1: Explore AWS Managed Keys

**Console:** KMS → AWS managed keys

- Keys created automatically by AWS services (e.g., `aws/ebs`, `aws/sqs`)
- Key policy uses a `ViaService` condition — e.g., EBS key only accessible via the `ec2.amazonaws.com` service
- Cryptographic config: symmetric, KMS origin, encrypt/decrypt

---

## Part 2: Create a Customer Managed Key

**Console:** KMS → Customer managed keys → **Create key**

> **Cost:** $1/month per key — skip creation if you don't want to incur charges.

| Setting | Value |
|---------|-------|
| Key type | Symmetric |
| Key usage | Encrypt and decrypt |
| Key origin | KMS (AWS generates the key) |
| Regionality | Single region (most common) |
| Alias | `tutorial` |
| Key administrators | Leave blank → default key policy |
| Key users | Leave blank → all IAM principals with correct IAM policies |
| Other AWS accounts | Add account IDs here for cross-account access |

### Default Key Policy

```json
{
  "Effect": "Allow",
  "Principal": { "AWS": "arn:aws:iam::<account-id>:root" },
  "Action": "kms:*",
  "Resource": "*"
}
```

Allows any IAM principal in the account to use the key **if they have IAM permissions** to do so.

---

## Part 3: Key Rotation

**Console:** KMS → Key → **Key rotation** tab → Edit

| Option | Detail |
|--------|--------|
| Automatic rotation | Enable and set period (90 days to 2,560 days) |
| Default period | 365 days (1 year) |
| On-demand rotation | Click "Rotate key now" button |
| Rotation history | All rotations logged in the console |

---

## Part 4: Key Actions

| Action | Use |
|--------|-----|
| Disable key | Temporarily prevent use of the key |
| Schedule key deletion | Set a waiting period (7–30 days) before permanent deletion |
| Aliases | Use `alias/tutorial` instead of the full key ARN |

---

## Part 5: Encrypt and Decrypt with CLI

### Setup — Create a plaintext file

```bash
echo "SuperSecretPassword" > ExampleSecretFile.txt
```

### Step 1 — Encrypt

```bash
aws kms encrypt \
  --key-id alias/tutorial \
  --plaintext fileb://ExampleSecretFile.txt \
  --query CiphertextBlob \
  --output text \
  --region eu-west-2 > ExampleSecretFileEncrypted.base64
```

### Step 2 — Base64 decode to binary

```bash
# Linux/Mac
base64 --decode ExampleSecretFileEncrypted.base64 > ExampleSecretFileEncrypted

# Windows
certutil -decode ExampleSecretFileEncrypted.base64 ExampleSecretFileEncrypted
```

> The resulting binary file is unreadable — this is what you would safely store or share.

### Step 3 — Decrypt

```bash
aws kms decrypt \
  --ciphertext-blob fileb://ExampleSecretFileEncrypted \
  --query Plaintext \
  --output text \
  --region eu-west-2 > ExampleFileDecrypted.base64
```

> KMS automatically knows which key to use — the key ID is embedded in the encrypted blob.

### Step 4 — Base64 decode the decrypted output

```bash
# Linux/Mac
base64 --decode ExampleFileDecrypted.base64 > ExampleFileDecrypted.txt

# Windows
certutil -decode ExampleFileDecrypted.base64 ExampleFileDecrypted.txt
```

### Result

`ExampleFileDecrypted.txt` contains the original plaintext: `SuperSecretPassword`

---

## Encryption/Decryption Flow Summary

```
Plaintext file
    ↓ kms encrypt (key-id: alias/tutorial)
Ciphertext blob (base64)
    ↓ base64 decode
Binary encrypted file  ← safe to store or share
    ↓ kms decrypt (key embedded in blob)
Plaintext base64
    ↓ base64 decode
Original plaintext file
```

---

## Best Practices

✓ **Use alias instead of key ID in CLI commands** — aliases persist through key rotation  
✓ **Never store plaintext secrets** — always encrypt with KMS before storing in env vars or code  
✓ **Enable automatic key rotation** — reduces exposure from long-lived key material  
✓ **Schedule deletion with a waiting period** — gives time to detect and cancel accidental deletion  
✓ **Use cross-account key access** — add target account ID in the key policy for snapshot sharing scenarios  

---

## Quick Reference

```
Create CMK:
  KMS → Customer managed keys → Create key
  Type: Symmetric | Origin: KMS | Single region
  Cost: $1/month

Key policy options:
  Default → account-wide IAM-delegated access
  Custom  → specific users/roles + cross-account account IDs

Key rotation:
  Automatic: 90–2560 days (default 365)
  On-demand: manual trigger anytime

CLI encrypt/decrypt:
  encrypt  --key-id alias/<name> --plaintext fileb://<file>
  decrypt  --ciphertext-blob fileb://<encrypted-file>
  KMS auto-detects the key from the ciphertext blob on decrypt

Aliases: use alias/name instead of full ARN — survives key rotation
```

---

**File: 13_AWS_KMS_Hands_On.md**
**Status: SysOps-focused, exam-ready, concise format**
