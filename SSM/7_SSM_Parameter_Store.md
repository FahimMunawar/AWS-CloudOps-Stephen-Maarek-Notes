# SSM Parameter Store

This note covers AWS SSM Parameter Store — a secure, serverless service for storing configuration data and secrets with versioning, encryption, and hierarchical organization.

## 1. What is SSM Parameter Store?

- **Purpose**: Centralized, secure storage for configuration values and secrets
- **Serverless**: No infrastructure to manage — scalable and durable by default
- **Encryption**: Optional — plain text or encrypted via **KMS**
- **Versioning**: Every parameter update is tracked with a version number
- **Security**: Access controlled through **IAM policies**
- **Notifications**: **EventBridge** integration for parameter change events
- **CloudFormation**: Stacks can reference Parameter Store values as input parameters

## 2. Plain Text vs. Encrypted Parameters

```
Application
    │
    ├── Plain text parameter
    │       └── IAM check → SSM returns value directly
    │
    └── Encrypted parameter
            └── IAM check → SSM calls KMS to decrypt → returns decrypted value
                           (application must also have KMS key access)
```

- For encrypted parameters, the application needs **both** SSM access (IAM) and KMS key access

## 3. Parameter Hierarchy

Parameters are stored in a **path-based hierarchy**, similar to a file system:

```
/my-department/
  └── my-app/
        ├── dev/
        │     ├── DB-URL
        │     └── DB-password
        └── prod/
              ├── DB-URL
              └── DB-password
```

### Benefits of Hierarchy

- IAM policies can grant access at any level of the path:
  - Entire department: `/my-department/*`
  - One app: `/my-department/my-app/*`
  - One environment: `/my-department/my-app/dev/*`
- Dev Lambda → access `/my-app/dev/*`
- Prod Lambda → access `/my-app/prod/*`
- Clean separation of environments with minimal IAM complexity

## 4. Special Parameter Types

### Secrets Manager Reference
- Parameter Store can reference **AWS Secrets Manager** secrets directly using the path:
  `/aws/reference/secretsmanager/<secret-name>`
- Allows accessing Secrets Manager secrets through the Parameter Store API

### AWS Public Parameters
- AWS publishes read-only parameters you can query
- Example: Get the latest Amazon Linux 2 AMI ID for your current region:
  `/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2`
- Useful in CloudFormation and automation scripts to avoid hardcoding AMI IDs

## 5. Parameter Tiers

| Feature | Standard | Advanced |
|---|---|---|
| Max parameter size | 4 KB | 8 KB |
| Parameter policies (TTL, expiry) | No | Yes |
| Cost | Free | $0.05 per parameter per month |

## 6. Advanced Parameter Policies

Only available on **Advanced tier** parameters. Assign multiple policies to one parameter.

### Expiration Policy
- Sets a TTL (expiration date) on a parameter
- Forces users to update or delete sensitive data (e.g., passwords) by a deadline
- EventBridge sends a notification **N days before** the parameter expires

### No-Change Notification Policy
- Triggers an EventBridge notification if a parameter has **not been updated** within N days
- Useful for enforcing regular rotation of credentials

### Example: Expiration with 15-Day Warning

```
Parameter TTL set → expires on [timestamp]
        │
        ▼ (15 days before expiry)
EventBridge notification → alert team to rotate the value
        │
        ▼ (on expiry, if not updated)
Parameter is automatically deleted
```

## 7. Key Takeaways for SysOps Associate

- Parameter Store stores **plain text config** and **encrypted secrets** (via KMS)
- For encrypted parameters, the caller needs access to **both SSM and the KMS key**
- Use a **hierarchical path structure** to organize parameters and simplify IAM policies
- **Standard tier** is free (4 KB max); **Advanced tier** adds policies and 8 KB size ($0.05/month)
- **Parameter policies** (Advanced only) enable TTL/expiry and no-change notifications via EventBridge
- CloudFormation can use Parameter Store values as **stack input parameters**
- AWS publishes **public parameters** — useful for dynamically resolving the latest AMI IDs
- Access Secrets Manager secrets through the Parameter Store API using the `/aws/reference/secretsmanager/` path prefix
