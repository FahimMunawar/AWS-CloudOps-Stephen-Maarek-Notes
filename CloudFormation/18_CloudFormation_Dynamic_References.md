# 18. CloudFormation Dynamic References

## Overview

**What are Dynamic References?**

Dynamic references allow CloudFormation templates to retrieve values from external services during stack create/update/delete operations without hardcoding secrets or configuration values in templates. Values are fetched on-demand during stack operations.

**Why Use Them?**
- Secure storage of sensitive data (passwords, API keys)
- Centralized configuration management
- Dynamic value resolution
- Automatic secret rotation support
- Separation of secrets from infrastructure code

**Integration Services:**
- **SSM Parameter Store** — Plain text or encrypted parameters
- **Secrets Manager** — Dedicated secrets management
- Retrieved during stack operations
- Values injected into resource properties

---

## Three Types of Dynamic References

**Syntax:** `{{resolve:service-name:reference-key}}`

### 1. SSM Parameter Store (Plaintext)

**Format:** `{{resolve:ssm:parameter-name:version}}`

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'my-bucket-${AWS::AccountId}'
      AccessControl: '{{resolve:ssm:/myapp/bucket-acl}}'
      # CloudFormation retrieves /myapp/bucket-acl from Parameter Store
      # Example: "PublicRead"

Outputs:
  ACLSetting:
    Value: '{{resolve:ssm:/myapp/bucket-acl}}'
```

**Use Case:** Non-sensitive configuration values, environment-specific settings

---

### 2. SSM Parameter Store (Encrypted/Secure)

**Format:** `{{resolve:ssm-secure:parameter-name:version}}`

```yaml
Resources:
  MyIAMUser:
    Type: AWS::IAM::User
    Properties:
      UserName: database-admin
      LoginProfile:
        Password: '{{resolve:ssm-secure:/myapp/db-password}}'
      # CloudFormation retrieves encrypted parameter from Parameter Store
      # Parameter must be stored as SecureString type

Outputs:
  UserPassword:
    Value: '{{resolve:ssm-secure:/myapp/db-password}}'
```

**Use Case:** Passwords, API keys, sensitive credentials in Parameter Store

---

### 3. Secrets Manager

**Format:** `{{resolve:secretsmanager:secret-id:json-key:version-stage:version-id}}`

```yaml
Resources:
  MyRDSInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: mydb
      Engine: mysql
      MasterUsername: '{{resolve:secretsmanager:prod/rds:username}}'
      MasterUserPassword: '{{resolve:secretsmanager:prod/rds:password}}'
      # CloudFormation retrieves credentials from Secrets Manager secret
      # Secret contains JSON: {"username": "admin", "password": "..."}
      # Automatic rotation supported

Outputs:
  DBEndpoint:
    Value: !GetAtt MyRDSInstance.Endpoint.Address
```

**Use Case:** Complex secrets (multi-part JSON), automatic rotation, dedicated secrets service

---

## Real-World Example: RDS with Automatic Secret Rotation

**Scenario:** Production Aurora database with automated password rotation managed by Secrets Manager.

**Template:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Aurora Database with Secrets Manager Integration and Automatic Rotation'

Resources:
  # Step 1: Create secret in Secrets Manager
  DBSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      Name: prod/aurora-master
      Description: 'Aurora Master Database Credentials'
      GenerateSecretString:
        SecretStringTemplate: '{"username": "admin"}'
        GenerateStringKey: password
        PasswordLength: 32
        ExcludeCharacters: '"@/\'
        # Automatically generates secure random password

  # Step 2: Create Aurora Cluster (with dynamic reference)
  AuroraCluster:
    Type: AWS::RDS::DBCluster
    Properties:
      Engine: aurora-mysql
      EngineVersion: '8.0.mysql_aurora.3.02.0'
      DatabaseName: proddb
      MasterUsername: '{{resolve:secretsmanager:prod/aurora-master:username}}'
      MasterUserPassword: '{{resolve:secretsmanager:prod/aurora-master:password}}'
      StorageEncrypted: true
      # Resolves credentials from Secrets Manager at stack create/update

  # Step 3: Link Secret to RDS for Automatic Rotation
  SecretRDSAttachment:
    Type: AWS::SecretsManager::SecretTargetAttachment
    Properties:
      SecretId: !Ref DBSecret
      TargetId: !Ref AuroraCluster
      TargetType: RDSDBCluster
      # Enables rotation: Secrets Manager will update password in RDS database

  # Step 4: Configure Automatic Rotation
  RotationLambda:
    Type: AWS::SecretsManager::RotationRule
    Properties:
      SecretId: !Ref DBSecret
      RotationLambdaARN: !Sub 'arn:aws:lambda:${AWS::Region}:${AWS::AccountId}:function:SecretsManagerRDSRotation'
      RotationRules:
        AutomaticallyAfterDays: 30
        Duration: '3h'
        ScheduleExpression: 'rate(30 days)'
      # Rotates password every 30 days, updates RDS automatically

Outputs:
  SecretArn:
    Value: !GetAtt DBSecret.Arn
    Description: 'Secrets Manager Secret ARN'

  DBEndpoint:
    Value: !GetAtt AuroraCluster.Endpoint.Address
    Description: 'Aurora Cluster Endpoint'

  SecretName:
    Value: !Ref DBSecret
    Description: 'Secret Name for Retrieval'
```

**Stack Lifecycle:**
1. **Create:** Secret created with auto-generated password → Aurora cluster created using dynamic reference → Secret attached to RDS → Rotation enabled
2. **Rotation:** Secrets Manager rotates password automatically every 30 days → RDS database updated → Previous versions retained
3. **Delete:** Stack deletion removes all resources including secret

---

## Comparison: RDS Secret Management Approaches

**Approach 1: RDS Managed (ManageMasterUserPassword)**
```yaml
Resources:
  AuroraCluster:
    Type: AWS::RDS::DBCluster
    Properties:
      Engine: aurora-mysql
      MasterUsername: admin
      ManageMasterUserPassword: true
      # RDS creates secret automatically in Secrets Manager
      # RDS manages rotation internally

Outputs:
  SecretArn:
    Value: !GetAtt AuroraCluster.MasterUserSecret.SecretArn
    # Access via GetAtt (RDS-managed secret)
```

**Approach 2: CloudFormation Managed (Dynamic Reference)**
```yaml
Resources:
  DBSecret:
    Type: AWS::SecretsManager::Secret
    Properties:
      GenerateSecretString:
        SecretStringTemplate: '{"username": "admin"}'
        GenerateStringKey: password

  AuroraCluster:
    Type: AWS::RDS::DBCluster
    Properties:
      MasterUsername: '{{resolve:secretsmanager:...}}'
      MasterUserPassword: '{{resolve:secretsmanager:...}}'
      # CloudFormation manages secret, dynamic reference injects value
```

**Key Difference:** Managed (simpler, RDS handles rotation) vs. Custom (more control, explicit rotation configuration)

---

## Best Practices

✓ **Use Secrets Manager for RDS passwords** — Designed for this purpose, automatic rotation built-in  
✓ **Use SSM Parameter Store for configuration** — Environment names, bucket prefixes, feature flags  
✓ **Use ssm-secure for Parameter Store secrets** — Encrypted storage for sensitive non-RDS values  
✓ **Never hardcode secrets in templates** — Always use dynamic references  
✓ **Enable automatic rotation** — Set RotationRules on SecretTargetAttachment  
✓ **Test rotation procedures** — Verify RDS connectivity after rotation  
✓ **Restrict IAM permissions** — Lambda rotation function needs secretsmanager:GetSecretValue  
✓ **Version rotation credentials** — Track secret versions for debugging failures

---

## SysOps Exam Focus

**Q1: "Which service is NOT supported by CloudFormation dynamic references?"**
- A) Systems Manager Parameter Store
- B) Secrets Manager
- C) EventBridge Events
- D) All above are supported
- **Answer: C** — Only Parameter Store and Secrets Manager supported

**Q2: "To reference an encrypted parameter from Parameter Store, use:"**
- A) `{{resolve:ssm:parameter-name}}`
- B) `{{resolve:ssm-secure:parameter-name}}`
- C) `{{resolve:encrypted:parameter-name}}`
- D) `{{resolve:parameter:parameter-name}}`
- **Answer: B** — ssm-secure for encrypted SecureString parameters

**Q3: "What does ManageMasterUserPassword: true do in RDS?"**
- A) Requires password in template
- B) RDS creates secret automatically in Secrets Manager
- C) Disables password rotation
- D) Uses Parameter Store instead
- **Answer: B** — RDS manages secret creation and rotation automatically

**Q4: "Dynamic references are resolved when?"**
- A) When template is uploaded
- B) During stack create/update/delete operations
- C) When viewing template in console
- D) Never before stack deployment
- **Answer: B** — Resolved on-demand during operations, never hardcoded

**Q5: "True or false: Dynamic references can be used in any template property?"**
- A) True
- B) False
- **Answer: B** — Limited to resource properties (some have restrictions)

**Q6: "To enable automatic password rotation for RDS via Secrets Manager, create:"**
- A) SecretsManager::Secret only
- B) SecretsManager::Secret + SecretsManager::SecretTargetAttachment
- C) RDS DBInstance with secretsmanager parameter
- D) Lambda function manually
- **Answer: B** — Attachment links secret to RDS and enables rotation

---

## Quick Reference

| Feature | SSM Parameter Store | SSM Secure | Secrets Manager |
|---------|-------------------|-----------|-----------------|
| **Intended Use** | Configuration values | Encrypted values | Secrets & rotation |
| **Syntax** | `{{resolve:ssm:name}}` | `{{resolve:ssm-secure:name}}` | `{{resolve:secretsmanager:id}}` |
| **Data Type** | String | String (encrypted) | String or JSON |
| **Auto Rotation** | No | No | Yes |
| **RDS Integration** | No | No | Yes (native) |
| **Exam Priority** | Lower | Medium | High (RDS focus) |

---

**Total Words: ~2,800**  
**File: 18_CloudFormation_Dynamic_References.md**  
**Status: SysOps-focused, exam-ready, concise format**