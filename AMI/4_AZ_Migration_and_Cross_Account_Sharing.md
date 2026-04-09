# EC2 Migration Between AZs and Cross-Account AMI Sharing

This note covers two important AMI use cases: migrating EC2 instances between availability zones and sharing AMIs across AWS accounts with various permission models.

## Part 1: Migrating EC2 Between Availability Zones

### Overview

**Migration Method**
- **Purpose**: Move EC2 instance from one AZ to another
- **Process**: Use AMI as the migration mechanism
- **Data Preservation**: All data, applications, and configurations transferred
- **Downtime**: Instance is unavailable during migration

### Step-by-Step Migration Process

#### 1. Create AMI from Source Instance
- **Source Instance**: In original AZ (e.g., us-east-1a)
- **Create Image**: Actions → Image and templates → Create image
- **Name**: Descriptive name (e.g., "prod-instance-migration")
- **Wait**: For AMI creation to complete

#### 2. Launch New Instance from AMI
- **Select AMI**: Search for created AMI
- **Instance Type**: Same or different type as original
- **Network Configuration**:
  - **VPC**: Same VPC as original
  - **Subnet**: Select subnet in **different AZ** (e.g., us-east-1b)
  - **Public IP**: Assign as needed
- **Security Groups**: Select appropriate security groups
- **Key Pair**: Use existing key or create new

#### 3. Verify Migration
- **Data Integrity**: Confirm all data present
- **Applications Running**: Test all services
- **Network Connectivity**: Verify security groups and routes
- **DNS Updates**: Update Route 53 or application config if needed

#### 4. Terminate Original Instance
- **Backup**: Confirm migration successful first
- **Delete AMI**: Clean up AMI if no longer needed
- **Delete Snapshots**: Remove EBS snapshots to reduce costs

### Migration Example
```
Source Instance:
- Location: eu-central-1c
- Instance ID: i-0123456789abcdef0
- Applications: Web server, database client
- EBS Volume: 100 GB

Steps:
1. Create AMI from source instance
2. Launch new instance in eu-central-1b from AMI
3. Verify all applications working
4. Update Route 53 or application config to point to new instance
5. Terminate old instance
6. Deregister AMI and delete snapshots

Result:
- New instance in eu-central-1b with identical configuration
- All data and applications migrated
- Old instance terminated to reduce costs
```

### Limitations and Considerations
- **Data Synchronization**: Only works for point-in-time migration
- **Active Databases**: May need replication for live data
- **Load Balancers**: Update target groups to new instance
- **DNS**: May need TTL changes before migration
- **Monitoring**: Update CloudWatch dashboards and alarms

---

## Part 2: Cross-Account AMI Sharing

### Overview

**Sharing vs. Copying**
- **Sharing**: AMI remains owned by source account, other accounts get access
- **Copying**: Other accounts create their own copy, becoming owners
- **Key Differences**: Permissions, encryption, and long-term management differ

### AMI Sharing Scenarios

#### 1. Unencrypted AMI Sharing (Simplest)

**Requirements**
- AMI has unencrypted EBS volumes
- No KMS keys needed
- Can be shared with specific accounts or public

**Sharing Process**
- **Source Account A**: Creates unencrypted AMI
- **Share Settings**: Edit AMI permissions
- **Target Account B**: Can launch directly from shared AMI
- **Ownership**: Remains with Account A
- **Cost**: Account B pays for EC2 instance; Account A pays for snapshot storage

**Limitations**
- No encryption protection
- Source account maintains storage costs for snapshots
- Source account controls AMI lifecycle

#### 2. Encrypted AMI Sharing (Cross-Account)

**Requirements**
- AMI encrypted with Customer Managed Key (CMK)
- CMK must be shared with target account
- KMS permissions required for target account

**Sharing Process**
- **Account A**: Creates AMI encrypted with CMK-A
- **Share AMI**: Edit AMI permissions, add Account B
- **Share KMS Key**: Grant Account B permissions to:
  - `kms:Decrypt` - Decrypt volumes during instance launch
  - `kms:DescribeKey` - Describe key details
  - `kms:GenerateDataKey` - For snapshot copying
  - `kms:ReEncrypt` - For re-encryption during copy

**KMS Policy Example**
```json
{
    "Sid": "Allow cross-account AMI operations",
    "Effect": "Allow",
    "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-B-ID:root"
    },
    "Action": [
        "kms:Decrypt",
        "kms:DescribeKey",
        "kms:GenerateDataKey",
        "kms:ReEncrypt"
    ],
    "Resource": "*",
    "Condition": {
        "StringEquals": {
            "kms:ViaService": [
                "ec2.us-east-1.amazonaws.com",
                "ec2.us-east-1.amazonaws.com"
            ]
        }
    }
}
```

**Permissions Model**
- **Account A (Source)**: Owns AMI and CMK-A
- **Account B (Target)**: Has permission to decrypt using CMK-A
- **Account B Launch**: Can launch instances from Account A's encrypted AMI
- **Account B Cannot**: Cannot re-encrypt or modify the AMI

---

### AMI Copying (Cross-Account)

#### Concept
- **Copying**: Target account creates independent copy of AMI
- **Ownership Transfer**: Target becomes owner of copied AMI
- **Key Requirement**: Source must grant read access to EBS snapshots
- **Enhanced Control**: Target can encrypt with own keys

#### Unencrypted AMI Copy Process

**Requirements**
- Shared AMI with read snapshot permissions
- Target account has copy permissions
- Source must grant `CreateVolumePermission` on snapshots

**Copy Steps**
1. **Source Account A**: Shares AMI with Account B
2. **Grant Read Access**: Allow Account B to read EBS snapshots
3. **Account B Initiates Copy**: Copy image API call
4. **Account B Creates AMI**: New AMI owned by Account B
5. **Independent Management**: Account B manages copy

**Benefits of Copying**
- **Ownership**: Target account owns the AMI
- **Independence**: Can modify, deregister, or share independently
- **Cost Isolation**: Each account pays own snapshot storage
- **Disaster Recovery**: Decoupled from source account

#### Encrypted AMI Copy with Re-encryption

**Scenario**
- Source AMI encrypted with CMK-A (Account A)
- Target wants own copy encrypted with CMK-B (Account B)

**Requirements**
- Source must share AMI
- Source must grant KMS permissions for CMK-A to Account B
- Target must have CMK-B in account B

**Copy Process**
1. **Account A**: Shares encrypted AMI with Account B
2. **Account A**: Grants Account B KMS decrypt permissions on CMK-A
3. **Account B**: Reads source snapshot and decrypts using CMK-A
4. **Account B**: Re-encrypts snapshot using own CMK-B
5. **Account B**: Creates new AMI with CMK-B encryption
6. **Result**: Account B now owns AMI encrypted with its own key

**Workflow Example**
```
Account A (Source):
├── AMI: "my-app-ami" 
├── EBS Snapshot: encrypted with CMK-A
└── Share settings: Grant Account B read access

Account B (Target):
├── Receive shared AMI
├── Copy AMI API call
├── Decrypt snapshot with CMK-A (via KMS permissions)
├── Re-encrypt with CMK-B
└── Result: New AMI "my-app-ami-b" (owned by Account B)

Outcome:
- Two separate AMIs, both functional
- Account A maintains original
- Account B has independent copy
- Each encrypted with own keys
- No ongoing dependency
```

---

### Practical: Sharing AMI Permissions

#### Access Console Location
- **Path**: EC2 Console → AMIs → Select AMI
- **Option**: Actions → Edit AMI permissions

#### Permission Options

##### 1. Private AMI (Default)
- **State**: Not shared with anyone
- **Availability**: Only visible in source account
- **Encryption**: Can be encrypted or unencrypted
- **Use Case**: Internal development or testing

##### 2. Public AMI
- **State**: Available to anyone on AWS
- **Visibility**: Searchable in AMI catalog
- **Encryption**: Only unencrypted AMIs can be public
- **Warning**: Ensure no sensitive data included
- **Use Case**: Open-source projects, shared resources

##### 3. Specific Account Sharing
- **Selection**: "Add account ID"
- **Input**: AWS Account IDs (12-digit numbers)
- **Multiple Accounts**: Can list multiple IDs
- **Scope**: Only specified accounts can access
- **Encrypted**: Works with encrypted AMIs if KMS permissions granted

##### 4. Organization/OU Sharing
- **Selection**: "Share with Organization/OU"
- **Input**: Organization ARN or OU ARN
- **Scope**: All accounts in organization or specific OU
- **Centralized Management**: Useful for multi-account strategies
- **Compliance**: For regulated enterprises

#### Create Volume Permission Option
- **Checkbox**: "Add create volume permission to the associated snapshots"
- **Purpose**: Allows target accounts to create volumes from snapshots independently
- **Impact**: Enables greater flexibility for target accounts
- **When to Use**: When sharing AMI for long-term use across accounts
- **Permission Grant**: Applies to all snapshots backing the AMI

#### Permission Management Example
```
Scenario: Share production web server AMI with development team

Steps:
1. EC2 Console → AMIs
2. Select production AMI
3. Actions → Edit AMI permissions
4. Choose sharing type: Specific Account IDs
5. Enter development account IDs
6. Check "Add create volume permission"
7. Save permissions

Result:
- Dev accounts can launch instances from AMI
- Dev accounts can copy AMI to own account
- Dev accounts can create volumes from snapshots
- Production account retains ownership
```

---

## 4. Comparison: Sharing vs. Copying

| Aspect | Sharing | Copying |
|--------|---------|---------|
| **Ownership** | Source account | Target account |
| **Ongoing Relationship** | Dependent on source | Independent |
| **Storage Costs** | Source pays | Target pays (separate AMI) |
| **Modifications** | Cannot modify | Can modify copy |
| **Encryption** | With CMK sharing permission | Can re-encrypt with own key |
| **Deregistration** | Source controls | Target controls |
| **Use Case** | Temporary access, shared templates | Permanent deployment, independence |
| **Management** | Simpler, centralized | More overhead, decentralized |

---

## 5. Cross-Account Strategy Best Practices

### For Source Account (Sharing)
- **Document Intent**: Clearly mark shared vs. private AMIs
- **Version Control**: Use naming conventions with version numbers
- **Security Review**: Ensure no sensitive data before sharing
- **Permissions Audit**: Regularly review sharing settings
- **Encryption**: Use CMK for sensitive workloads
- **Cost Tracking**: Monitor snapshot storage costs

### For Target Account (Receiving)
- **Copy for Stability**: Copy instead of share for critical workloads
- **Re-encrypt**: Use own KMS keys for security
- **Test Thoroughly**: Launch and validate before production
- **Document Source**: Record which AMI came from which account
- **Version Tracking**: Track updates from source account

### Multi-Account Architecture
- **Hub-and-Spoke**: Central account maintains master AMIs
- **Distribution**: Shares to regional or team accounts
- **Standardization**: Consistent images across organization
- **Compliance**: Centralized security controls
- **Automation**: Use AWS Systems Manager Image Builder for consistency

---

## 6. Troubleshooting Common Issues

### AMI Sharing Issues
- **Permission Denied**: Verify KMS permissions for encrypted AMIs
- **Cannot Access**: Confirm account ID in sharing settings
- **Snapshots Not Visible**: Check snapshot permissions independently
- **Encryption Failures**: Ensure KMS key policy includes target account

### AMI Copying Issues
- **Copy Fails**: Verify source snapshot permissions
- **Re-encryption Fails**: Confirm target CMK exists and has permissions
- **Slow Copying**: Large snapshots take time; monitor progress
- **Storage Not Freed**: Source snapshots retained even after copy

### Cross-Account Access Issues
- **Authentication Fails**: Verify cross-account role trust relationships
- **API Errors**: Check IAM permissions in both accounts
- **KMS Errors**: Review CMK policy and key usage permissions

---

## 7. Key Takeaways for SysOps Associate

### AZ Migration
- **AMI Method**: Use AMI to migrate instances between AZs
- **Process**: Create AMI → Launch in new AZ → Terminate old
- **Same Configuration**: All data and applications transferred
- **No Special Steps**: Standard AMI creation and launch process

### Sharing Basics
- **Sharing Preserves Ownership**: Source account retains ownership
- **Two Sharing Types**: Unencrypted (simple) or encrypted (requires KMS)
- **Encrypted AMI**: Must share both AMI and KMS permissions
- **Public Not Recommended**: Only for open-source or non-sensitive content

### Copying Advantages
- **Ownership Transfer**: Target account becomes owner
- **Independence**: Decoupled from source account
- **Re-encryption**: Can encrypt with own keys during copy
- **Snapshots**: Target has independent snapshots

### KMS with Cross-Account
- **Permissions Required**: Decrypt, DescribeKey, GenerateDataKey, ReEncrypt
- **Policy Based**: Configured in CMK policy
- **Region Specific**: Set for specific EC2 regions
- **Two Key Model**: Source has CMK-A, target has CMK-B

### Practical Implementation
- **Console Location**: Edit AMI Permissions → Actions menu
- **Account IDs Required**: 12-digit AWS account numbers
- **Org/OU Sharing**: Use ARNs for organization-wide sharing
- **Volume Permissions**: Enable for independent snapshot operations

### Exam Focus Points
- **Sharing Definition**: Ownership remains with source
- **Copying Definition**: Ownership transfers to target
- **Encrypted Sharing**: Requires KMS key sharing
- **Migration Pattern**: AMI is primary tool for AZ migration
- **Cost Considerations**: Owner typically pays snapshot storage costs