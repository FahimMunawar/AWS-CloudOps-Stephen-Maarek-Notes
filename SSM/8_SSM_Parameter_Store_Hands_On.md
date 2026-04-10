# SSM Parameter Store Hands-On

This note walks through creating parameters with hierarchy, using String and SecureString types, and querying them via the AWS CLI.

## 1. Parameter Types

| Type | Use Case | Encrypted? |
|---|---|---|
| `String` | Plain text config (URLs, hostnames) | No |
| `StringList` | Comma-separated list of values | No |
| `SecureString` | Passwords, secrets | Yes — via KMS |

### Data Types for String
- **Text**: Free-form string value
- **aws:ec2:image**: AMI ID in the correct format — validated by AWS

## 2. Standard vs. Advanced Tier (Quick Reference)

| | Standard | Advanced |
|---|---|---|
| Max parameters | 10,000 | 100,000 |
| Max value size | 4 KB | 8 KB |
| Cross-account sharing | No | Yes |
| Parameter policies | No | Yes |
| Cost | Free | $0.05/param/month |

## 3. Creating Parameters (Console)

### Example Parameters Created

| Name | Type | Value |
|---|---|---|
| `/my-app/dev/db-url` | String | `dev.database.example.com` |
| `/my-app/dev/db-password` | SecureString | `devpassword` (encrypted) |
| `/my-app/prod/db-url` | String | `prod.database.example.com:3306` |
| `/my-app/prod/db-password` | SecureString | `prodpassword` (encrypted) |

### Steps to Create a String Parameter

1. SSM Console → **Parameter Store** → **Create parameter**
2. Name: `/my-app/dev/db-url`
3. Tier: **Standard**
4. Type: **String** → Data type: **Text**
5. Value: `dev.database.example.com`
6. Click **Create parameter**

### Steps to Create a SecureString Parameter

1. Name: `/my-app/dev/db-password`
2. Tier: **Standard**
3. Type: **SecureString**
4. KMS Key: choose `alias/aws/ssm` (AWS-managed default) or a custom KMS key
5. Value: `devpassword`
6. Click **Create parameter**

> The value is stored encrypted. In the console, click **Show decrypted value** to reveal it — this triggers a KMS decrypt call and requires KMS permissions.

### Version History
- Every update to a parameter creates a new version
- All versions are accessible under the **Version history** tab

## 4. Querying Parameters via AWS CLI

### Get Specific Parameters by Name

```bash
aws ssm get-parameters \
  --names /my-app/dev/db-url /my-app/dev/db-password
```

- `String` parameters return their value in plain text
- `SecureString` parameters return the **encrypted ciphertext** by default

### Get SecureString with Decryption

```bash
aws ssm get-parameters \
  --names /my-app/dev/db-url /my-app/dev/db-password \
  --with-decryption
```

- Requires IAM permission to call SSM **and** permission to use the KMS key
- Returns the decrypted plain-text value for SecureString parameters

### Get All Parameters Under a Path

```bash
# Get all parameters under /my-app/dev/
aws ssm get-parameters-by-path \
  --path /my-app/dev
```

### Get All Parameters Recursively Under a Path

```bash
# Get all parameters under /my-app/ and all sub-paths
aws ssm get-parameters-by-path \
  --path /my-app \
  --recursive

# With decryption for SecureString values
aws ssm get-parameters-by-path \
  --path /my-app \
  --recursive \
  --with-decryption
```

- Without `--recursive`: only returns parameters at exactly that path level
- With `--recursive`: returns all parameters at all levels below the path

## 5. Key Takeaways for SysOps Associate

- Use a **hierarchical path** (e.g., `/app/env/param-name`) to organize parameters and simplify IAM policies
- **SecureString** parameters are encrypted at rest via KMS — decryption requires both SSM and KMS permissions
- Use `--with-decryption` in CLI calls to get plain-text values for SecureString parameters
- `get-parameters` fetches by exact name; `get-parameters-by-path` fetches by path prefix
- Add `--recursive` to `get-parameters-by-path` to retrieve all parameters under a namespace tree
- Every parameter update is **versioned** — you can retrieve or roll back to previous versions
- The default KMS key `alias/aws/ssm` is AWS-managed and free to use; custom KMS keys give more control
