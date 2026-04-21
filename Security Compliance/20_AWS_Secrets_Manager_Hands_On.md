# 20 — AWS Secrets Manager (Hands-On)

## Pricing

| Item | Cost |
|---|---|
| Secret storage | $0.40 per secret per month |
| API calls | $0.05 per 10,000 API calls |
| Free trial | 30 days per secret |

---

## Secret Types

| Type | Input Format |
|---|---|
| RDS / Aurora / Redshift / DocumentDB credentials | Username + password (linked to DB) |
| Other database | Username + password |
| **Other type of secret** | Key-value pairs (multiple allowed) |

For **Other type**, you can store multiple key-value pairs in one secret:

```
Key: api_key          Value: abc123...
Key: api_secret_key   Value: xyz789...
```

Or paste raw JSON in plaintext mode for bulk entry.

---

## Creating a Secret (Key-Value)

1. **Secrets Manager → Store a new secret → Other type of secret**
2. Add key-value pairs
3. Choose encryption key: AWS default or custom KMS key
4. **Secret name**: e.g., `prod/my-secret-api`
5. Add description + tags (optional)
6. **Configure rotation** (optional):
   - Enable automatic rotation → set interval (1–365 days, e.g., 60 days)
   - Select or create a **Lambda function** to perform the rotation
   - Lambda must have IAM permissions to update the secret
7. Review → Store

---

## Automatic Rotation

When enabled, Secrets Manager invokes the Lambda function on schedule to:
- Generate new credentials
- Update the third-party service / database
- Store the new value in Secrets Manager

> Lambda function logic is user-defined — e.g., generate a new API key, refresh OAuth token, or update a database password.

---

## Retrieving a Secret (Python Example)

```python
import boto3
from botocore.exceptions import ClientError

def get_secret(secret_name, region_name):
    client = boto3.client("secretsmanager", region_name=region_name)
    response = client.get_secret_value(SecretId=secret_name)
    return response["SecretString"]  # JSON string of key-value pairs
```

Same pattern available for Go, JavaScript, Java, and other languages via the console sample code.

---

## RDS-Linked Secret

When creating a secret for RDS/Redshift/DocumentDB:
- Secrets Manager stores the username and password
- Links the secret directly to the database
- Can enable rotation — Lambda updates credentials in both Secrets Manager **and** the linked database automatically

---

## Deleting a Secret

Deletion has a **waiting period** (configurable) — prevents accidental immediate deletion and allows recovery if needed.

---

## Quick Reference

```
Secrets Manager Hands-On:
  Other type → key-value pairs (multiple KV per secret)
  RDS type → username + password + DB link + rotation support

Rotation: enable → set interval (days) → Lambda function invoked on schedule
  Lambda generates new secret + updates target service

Pricing: $0.40/secret/month + $0.05/10k API calls (30-day free trial)

Retrieve: client.get_secret_value(SecretId=name) → SecretString (JSON)
Delete: has waiting period for safety
```
