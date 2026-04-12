# 13. CloudFormation DeletionPolicy

## Part 1: Understanding DeletionPolicy

### What Is DeletionPolicy?

**Definition:**

```
CloudFormation DeletionPolicy:
├─ Attribute applied to individual resources
├─ Controls behavior when resource deleted
├─ Triggered: When removed from template OR stack deleted
├─ Three options: Delete (default), Retain, Snapshot
├─ Per-resource control (not stack-wide)
├─ Safety mechanism for data preservation
└─ Prevents accidental data loss
```

**Why DeletionPolicy Matters:**

```
Problem: Default Stack Deletion

Stack Deletion Command:
├─ CloudFormation deletes stack
├─ All resources automatically deleted
├─ Data lost permanently
│  ├─ Database records gone
│  ├─ S3 bucket contents deleted
│  ├─ EBS volumes destroyed
│  └─ No backup or recovery option
├─ Accidental deletions catastrophic
└─ No way to preserve critical data

Solution: DeletionPolicy

Control Per Resource:
├─ S3 bucket: DeletionPolicy Retain → Data preserved
├─ RDS database: DeletionPolicy Snapshot → Backup taken
├─ Security group: DeletionPolicy Retain → Rules kept
├─ EC2 instance: DeletionPolicy Delete → Normal deletion
├─ EBS volume: DeletionPolicy Snapshot → Final backup
└─ Selective data protection
```

### DeletionPolicy Scope

**Resource-Level Control:**

```
CloudFormation Stack:
├─ Resource 1 (S3 Bucket): DeletionPolicy Retain
├─ Resource 2 (RDS DB): DeletionPolicy Snapshot
├─ Resource 3 (EC2): DeletionPolicy Delete
├─ Resource 4 (Security Group): DeletionPolicy Retain
└─ Each: Potentially different behavior

Stack Deletion Flow:

Delete Stack Command:
├─ Resource 1 (S3): NOT deleted (Retain)
│  └─ Bucket + contents preserved
├─ Resource 2 (RDS): Snapshot created, then deleted
│  └─ Database backed up, then removed
├─ Resource 3 (EC2): Normal deletion
│  └─ Instance terminated
├─ Resource 4 (SG): NOT deleted (Retain)
│  └─ Security group rules preserved
└─ Result: Mixed outcomes per policy

Orphaned Resources:

After Stack Deletion:
├─ S3 bucket still exists (Retain)
├─ RDS snapshot exists (from Snapshot policy)
├─ EC2 instance gone (Delete)
├─ Security group still exists (Retain)
├─ These are now "orphaned" resources
├─ Manual cleanup required
└─ Billing continues for orphaned resources
```

### Template Syntax

**Basic DeletionPolicy Syntax:**

```yaml
Resources:
  MyResource:
    Type: AWS::Service::ResourceType
    Properties:
      Property1: Value1
      Property2: Value2
    DeletionPolicy: Delete  # or Retain or Snapshot
```

**DeletionPolicy Placement:**

```yaml
Resources:
  # CORRECT: DeletionPolicy at resource level
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-data-bucket
    DeletionPolicy: Retain  # ✓ Correct

  # CORRECT: Can omit if using default
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.micro
    # DeletionPolicy: Delete  # Optional (default)

  # CORRECT: Nested (inside Resource, outside Properties)
  MyVolume:
    Type: AWS::EC2::Volume
    Properties:
      AvailabilityZone: us-east-1a
      Size: 100
    DeletionPolicy: Snapshot  # ✓ Correct location

  # INCORRECT: Inside Properties
  WrongLocation:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
      DeletionPolicy: Retain  # ✗ Wrong - should be outside
    # Correct would be:
    # DeletionPolicy: Retain  # Outside Properties

Syntax Rules:
├─ Placement: At resource level (not in Properties)
├─ Format: "DeletionPolicy: PolicyValue"
├─ Values: Delete, Retain, Snapshot
├─ Optional: Only needed if not using default
├─ Multiple: Different for each resource
└─ Inheritance: NOT inherited from parent stack
```

## Part 2: DeletionPolicy Delete

### Understanding Delete Policy

**Default Behavior:**

```
DeletionPolicy: Delete
├─ Resource removed from template
├─ CloudFormation deletes resource
├─ Default for all resources (if not specified)
├─ No preservation of data
├─ Clean removal
└─ No backup created

When Applied:

Stack Template:
  MyInstance:
    Type: AWS::EC2::Instance
    # DeletionPolicy: Delete (implicit/default)

Scenario 1: Remove from Template
├─ Edit template
├─ Delete MyInstance resource definition
├─ Update stack
├─ CloudFormation: Deletes instance
└─ Result: Instance terminated

Scenario 2: Delete Stack
├─ Stack delete command issued
├─ CloudFormation: Removes all resources
├─ Result: All instances terminated
└─ Data: Lost permanently
```

**Delete Policy on EC2 Instance:**

```yaml
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.micro
    DeletionPolicy: Delete  # Explicit (also default)

# When stack deleted:
# ├─ Instance terminated
# ├─ EBS root volume deleted
# ├─ Elastic IP disassociated
# └─ All instance data lost
```

### S3 Bucket Exception

**Special Case: S3 Bucket with DeletionPolicy Delete**

```
S3 Delete Policy Exception:

Normal Resources with Delete:
├─ Resource deleted immediately
├─ No conditions or checks
├─ Clean removal

S3 Bucket with Delete:
├─ IF bucket is EMPTY: Deleted successfully
├─ IF bucket is NOT EMPTY: Deletion FAILS
│  └─ Error: S3BucketNotEmpty
└─ CloudFormation: Stack deletion blocked

Why This Exception?

Safety Mechanism:
├─ Prevents accidental data loss
├─ S3 buckets often contain critical data
├─ Deleting non-empty bucket = data loss
├─ AWS prevents this automatically
└─ Forces explicit handling

Real Example:

Stack Contains S3 Bucket:
  DataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: my-app-data
    DeletionPolicy: Delete

Stack Deletion Initiated:
├─ CloudFormation attempts: Delete bucket
├─ Checks: Is bucket empty?
├─ Result: Bucket has 1000 objects
├─ Action: Deletion fails
├─ Error: "S3 bucket is not empty"
├─ Status: Stack deletion BLOCKED
└─ Reason: Prevents data loss
```

**Handling Non-Empty S3 Bucket Deletion:**

```
Problem: S3 Bucket with Objects + DeletionPolicy Delete

Option 1: Manual Emptying (Simple)
├─ Step 1: Delete all objects from bucket manually
├─ Step 2: Retry stack deletion
├─ Result: CloudFormation deletes empty bucket
├─ Time: Manual, error-prone
└─ Use case: One-time cleanup

Option 2: Custom Resource (Automated)
├─ Purpose: Lambda function to empty bucket
├─ CloudFormation: Invokes Lambda during deletion
├─ Lambda: Removes all objects from bucket
├─ Then: S3 bucket deleted normally
├─ Time: Automated, reliable
└─ Use case: Production stacks

Option 3: DeletionPolicy Retain (Safest)
├─ Bucket NOT deleted with stack
├─ Preserved for recovery
├─ No data loss
├─ Manual cleanup later
└─ Use case: Critical data preservation
```

**Manual S3 Deletion Process:**

```
Stack with Non-Empty S3 Bucket:

Step 1: Identify Bucket
├─ CloudFormation console → Stack resources
├─ Note: Bucket name (e.g., my-bucket-abc123)

Step 2: Empty Bucket (AWS Console)
├─ S3 console → Select bucket
├─ Select: All objects
├─ Click: Delete
├─ Confirm: Yes, delete all

Step 3: Empty Versions (if versioning enabled)
├─ S3 console → Bucket → Versions
├─ Select: All versions (delete markers + versions)
├─ Click: Delete
├─ Confirm: Yes

Step 4: Retry Stack Deletion
├─ CloudFormation console → Select stack
├─ Click: Delete stack
├─ CloudFormation: Attempts deletion again
├─ Bucket now empty: Deletion succeeds
└─ Result: Stack and empty bucket deleted

Alternative (AWS CLI):

# Empty bucket quickly
aws s3 rm s3://my-bucket --recursive

# Then delete stack normally
aws cloudformation delete-stack --stack-name MyStack
```

**Custom Resource Approach (Advanced):**

```yaml
# Advanced: Auto-Empty S3 Bucket
AWSTemplateFormatVersion: '2010-09-09'
Description: 'S3 Bucket with Automatic Cleanup'

Resources:
  # Lambda function to empty bucket
  EmptyS3BucketFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.9
      Handler: index.handler
      Role: !GetAtt LambdaExecutionRole.Arn
      Code:
        ZipFile: |
          import boto3
          import json
          
          s3 = boto3.client('s3')
          
          def handler(event, context):
              bucket_name = event['ResourceProperties']['BucketName']
              
              try:
                  # List and delete all objects
                  response = s3.list_objects_v2(Bucket=bucket_name)
                  
                  if 'Contents' in response:
                      for obj in response['Contents']:
                          s3.delete_object(Bucket=bucket_name, Key=obj['Key'])
                  
                  print(f"Emptied bucket: {bucket_name}")
                  return {
                      'Status': 'SUCCESS',
                      'PhysicalResourceId': bucket_name
                  }
              except Exception as e:
                  print(f"Error: {e}")
                  return {
                      'Status': 'FAILED',
                      'PhysicalResourceId': bucket_name
                  }

  # Execution role for Lambda
  LambdaExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: lambda.amazonaws.com
            Action: 'sts:AssumeRole'
      ManagedPolicyArns:
        - 'arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole'
      Policies:
        - PolicyName: S3Access
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 's3:ListBucket'
                  - 's3:DeleteObject'
                Resource: '*'

  # Custom resource to trigger Lambda
  EmptyBucketCustomResource:
    Type: AWS::CloudFormation::CustomResource
    Properties:
      ServiceToken: !GetAtt EmptyS3BucketFunction.Arn
      BucketName: !Ref DataBucket

  # Data bucket with Delete policy
  DataBucket:
    Type: AWS::S3::Bucket
    DependsOn: EmptyBucketCustomResource
    Properties:
      BucketName: my-data-bucket
    DeletionPolicy: Delete

# When stack deleted:
# ├─ Lambda function triggered
# ├─ All objects in bucket deleted
# ├─ Custom resource deleted
# ├─ S3 bucket now empty
# ├─ Delete policy applied: Bucket deleted
# └─ Clean stack removal
```

## Part 3: DeletionPolicy Retain

### Understanding Retain Policy

**Retain Behavior:**

```
DeletionPolicy: Retain
├─ Resource PRESERVED when deleted from template
├─ Resource PRESERVED when stack deleted
├─ Becomes "orphaned" resource in AWS
├─ No automatic cleanup
├─ Resource still costs money (billing continues)
├─ Requires manual deletion later
└─ Data safety priority
```

**When to Use Retain:**

```
Use Cases for Retain:

1. Critical Data Resources
   ├─ Databases with irreplaceable data
   ├─ S3 buckets with archives
   ├─ DynamoDB tables with history
   └─ Example: Multi-year analytics data

2. Compliance/Regulatory
   ├─ Data retention requirements
   ├─ Audit logs must be kept
   ├─ Legal holds on data
   └─ Example: Financial transaction records

3. Shared Resources
   ├─ Security groups used by other stacks
   ├─ IAM roles referenced elsewhere
   ├─ Subnets in shared VPC
   └─ Example: Organization-wide security group

4. Backup/Archive
   ├─ Data archive requirements
   ├─ Disaster recovery resources
   ├─ Historical backups
   └─ Example: Annual backup storage

5. Debugging/Troubleshooting
   ├─ Keep logs for analysis
   ├─ Preserve failed resources for investigation
   ├─ Keep infrastructure for post-mortem
   └─ Example: CloudWatch logs, application logs
```

**Retain Policy Examples:**

```yaml
# Example 1: DynamoDB Table (Critical Data)
Resources:
  AnalyticsDatabase:
    Type: AWS::DynamoDB::Table
    Properties:
      TableName: analytics-data
      BillingMode: PAY_PER_REQUEST
      AttributeDefinitions:
        - AttributeName: EventId
          AttributeType: S
      KeySchema:
        - AttributeName: EventId
          KeyType: HASH
    DeletionPolicy: Retain
    # Data preserved: Years of analytics retained

# Example 2: S3 Bucket (Compliance Archive)
Resources:
  ComplianceArchive:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: compliance-archive-2024
      VersioningConfiguration:
        Status: Enabled
    DeletionPolicy: Retain
    # Bucket kept: Regulatory requirements

# Example 3: Security Group (Shared)
Resources:
  SharedSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Organization-wide security group
      VpcId: vpc-12345678
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 10.0.0.0/8
    DeletionPolicy: Retain
    # SG kept: Used by other application stacks

# Example 4: CloudWatch Logs Group
Resources:
  ApplicationLogs:
    Type: AWS::Logs::LogGroup
    Properties:
      LogGroupName: /aws/lambda/my-app
      RetentionInDays: 365
    DeletionPolicy: Retain
    # Logs kept: For long-term analysis
```

### Retain Policy Implications

**Orphaned Resources:**

```
Stack Deletion with Retain:

Before Deletion:
├─ CloudFormation Stack: myapp-prod
├─ Resources:
│  ├─ EC2 Instance (Delete)
│  ├─ S3 Bucket (Retain)
│  ├─ RDS Database (Retain)
│  └─ Security Group (Retain)
└─ Status: All resources active and managed

Stack Delete Initiated:
├─ EC2 Instance: Terminated (Delete policy)
├─ S3 Bucket: Remains (Retain policy)
├─ RDS Database: Remains (Retain policy)
├─ Security Group: Remains (Retain policy)
└─ CloudFormation Stack: Deleted

After Deletion:
├─ CloudFormation Stack: DELETED
├─ Orphaned Resources (Still exist):
│  ├─ S3 Bucket: myapp-prod-bucket
│  ├─ RDS Database: myapp-prod-db
│  └─ Security Group: myapp-prod-sg
├─ Status: "Detached" from CloudFormation
├─ Management: Manual only (AWS console/CLI)
└─ Billing: CONTINUES for retained resources
```

**Orphaned Resource Cleanup:**

```
Manual Cleanup Required:

Step 1: Identify Orphaned Resources
├─ CloudFormation console: Stack deleted
├─ AWS console (S3/RDS/EC2): Resources still visible
├─ Note: ARNs and identifiers

Step 2: Manual Deletion (One By One)
├─ S3 console: Delete bucket manually
├─ RDS console: Delete database manually
├─ EC2 console: Delete security group manually
├─ Lambda/CloudLogs console: Delete logs manually
└─ No CloudFormation automation

Effort Required:
├─ Lost infrastructure-as-code automation
├─ Manual for each resource type
├─ Error-prone process
├─ No rollback option if mistake
└─ Time-consuming

Cost Impact:
├─ Resources still incur charges
├─ No immediate cleanup = $$ wasted
├─ Example: RDS db.t3.medium = ~$400/month
├─ Example: S3 storage = $0.023/GB/month
└─ Budget overruns possible if forgotten
```

**Best Practice: Document Retained Resources:**

```
CloudFormation Template Documentation:

# Add comments explaining retention
AWSTemplateFormatVersion: '2010-09-09'
Description: 'Production Application Stack'

Parameters:
  EnvironmentName:
    Type: String
    Default: prod

Resources:
  # IMPORTANT: Data Preservation
  # This database contains 3+ years of transaction data
  # DO NOT delete without backup verification
  # Retained to preserve historical records for compliance
  TransactionDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: !Sub '${EnvironmentName}-transactions'
      Engine: postgres
      DBInstanceClass: db.t3.medium
      AllocatedStorage: '100'
    DeletionPolicy: Retain
    # ↑ RETAINED: Manually delete after backup

  # Shared infrastructure
  # Referenced by networking-stack and app-stack-2
  # Keep even if this stack deleted
  OrganizationSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupName: org-general-sg
      GroupDescription: Organization-wide security group
    DeletionPolicy: Retain
    # ↑ RETAINED: Used by other stacks

  # Temporary application files
  # Delete with stack (standard pattern)
  TemporaryDataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'temp-${EnvironmentName}-${AWS::AccountId}'
    DeletionPolicy: Delete  # Auto-deleted (needs to be empty)

Outputs:
  # Document retained resources for cleanup team
  DatabaseIdentifier:
    Value: !Ref TransactionDatabase
    Description: 'RETAINED - Manual deletion required'
  
  SecurityGroupId:
    Value: !GetAtt OrganizationSecurityGroup.GroupId
    Description: 'RETAINED - Shared resource, do not delete'

# Cleanup Runbook:
# 1. Verify database backup exists in S3
# 2. Delete any dependent application stacks first
# 3. Manually delete retained RDS instance from RDS console
# 4. Manually delete retained security group from EC2 console
# 5. Remove from organization tracking system
```

## Part 4: DeletionPolicy Snapshot

### Understanding Snapshot Policy

**Snapshot Behavior:**

```
DeletionPolicy: Snapshot
├─ Final snapshot created before deletion
├─ Resource then deleted normally
├─ Snapshot preserved indefinitely
├─ Data backup created automatically
├─ Backup available for recovery
├─ Snapshot costs money (storage)
└─ Balance: Data safety + recovery
```

### Supported Services for Snapshots

**Services Supporting Snapshot Policy:**

```
Storage Services:
├─ AWS::EC2::Volume (EBS volumes)
├─ AWS::EC2::Snapshot (from EBS)
└─ Snapshot: Full EBS volume state captured

Database Services:
├─ AWS::RDS::DBInstance (relational databases)
├─ AWS::RDS::DBCluster (RDS clusters)
├─ Snapshot: Database state + data
└─ Snapshot: Can be used to restore

Data Analytics Services:
├─ AWS::Redshift::Cluster (Redshift data warehouse)
├─ Snapshot: Data warehouse cluster state
└─ Recovery: Restore from snapshot

Cache Services:
├─ AWS::ElastiCache::CacheCluster
├─ AWS::ElastiCache::ReplicationGroup
├─ Snapshot: Cache data cluster state
└─ Supports: Redis snapshots

Document Database Services:
├─ AWS::DocDB::DBCluster (DocumentDB)
├─ AWS::Neptune::DBCluster (Neptune graph DB)
├─ Snapshot: Document database state
└─ Supports: Cluster-level snapshots

Supported AWS Services Summary:
├─ ✓ EBS Volumes
├─ ✓ RDS (MySQL, PostgreSQL, MariaDB, Oracle, SQL Server)
├─ ✓ RDS Aurora Clusters
├─ ✓ Redshift Clusters
├─ ✓ ElastiCache (Redis, Memcached)
├─ ✓ Neptune Clusters
├─ ✓ DocumentDB Clusters
├─ ✓ More services (check docs for latest)
└─ ✗ NOT S3 (use Retention instead)
```

**When to Use Snapshot Policy:**

```
Use Cases for Snapshot:

1. Database Deletion with Final Backup
   ├─ Production RDS cleanup
   ├─ Final backup guaranteed
   ├─ Data recovery possible
   └─ Example: Prod database decommission

2. Volume Deletion with Archive
   ├─ EBS volume lifecycle end
   ├─ Archive final state
   ├─ Cost analysis possible
   └─ Example: Project storage wrap-up

3. Cache Cluster Backup
   ├─ ElastiCache cluster shutdown
   ├─ Session data preserved
   ├─ Recovery if needed
   └─ Example: Hot standby to cold

4. Data Warehouse Decommission
   ├─ Redshift cluster cleanup
   ├─ Final analytics snapshot
   ├─ Historical data archived
   └─ Example: End-of-quarter analytics

5. Disaster Recovery
   ├─ Development environment cleanup
   ├─ Final state captured
   ├─ Recovery for troubleshooting
   └─ Example: Failed deployment cleanup
```

### Snapshot Policy Examples

**EBS Volume with Snapshot:**

```yaml
Resources:
  ApplicationDataVolume:
    Type: AWS::EC2::Volume
    Properties:
      AvailabilityZone: us-east-1a
      Size: 100  # 100 GB
      VolumeType: gp3
      Encrypted: true
    DeletionPolicy: Snapshot
    # Result when deleted:
    # ├─ Snapshot created: Volume state captured
    # ├─ EBS volume deleted
    # ├─ Snapshot remains in S3
    # └─ Manual cleanup needed for snapshot

# CloudFormation Events During Deletion:
# ├─ Volume-Snapshot (CREATE_IN_PROGRESS)
# ├─ Volume-Snapshot (CREATE_COMPLETE) - Snapshot ID shown
# ├─ ApplicationDataVolume (DELETE_IN_PROGRESS)
# └─ ApplicationDataVolume (DELETE_COMPLETE)
```

**RDS Database with Snapshot:**

```yaml
Resources:
  ProductionDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceIdentifier: prod-mysql-db
      Engine: mysql
      DBInstanceClass: db.t3.medium
      AllocatedStorage: '50'
      MasterUsername: admin
      MasterUserPassword: !Sub '{{resolve:secretsmanager:db-secret:SecretString:password}}'
      MultiAZ: true
      BackupRetentionPeriod: 30
    DeletionPolicy: Snapshot
    # Result when deleted:
    # ├─ Final snapshot created automatically
    # ├─ Database instance deleted
    # ├─ Backup retention: 30 days
    # ├─ Snapshot: Accessible for 30+ days
    # └─ Can restore: Full database recovery

# CloudFormation Events During Deletion:
# ├─ prod-mysql-db-final-snapshot (CREATE_IN_PROGRESS)
# ├─ prod-mysql-db-final-snapshot (CREATE_COMPLETE) - Snapshot ARN
# ├─ ProductionDatabase (DELETE_IN_PROGRESS)
# └─ ProductionDatabase (DELETE_COMPLETE)
```

**Redshift Cluster with Snapshot:**

```yaml
Resources:
  DataWarehouse:
    Type: AWS::Redshift::Cluster
    Properties:
      ClusterIdentifier: analytics-warehouse
      NodeType: dc2.large
      NumberOfNodes: 3
      MasterUsername: awsuser
      MasterUserPassword: !Sub '{{resolve:secretsmanager:redshift-secret:SecretString:password}}'
    DeletionPolicy: Snapshot
    # Result when deleted:
    # ├─ Automated final snapshot taken
    # ├─ Data warehouse cluster terminated
    # ├─ Snapshot holds full cluster data
    # └─ Restore: Recreate cluster from snapshot
```

### Snapshot Storage and Costs

**Snapshot Cost Implications:**

```
Snapshot Storage Costs:

EBS Snapshot:
├─ Storage cost: $0.05 per GB-month
├─ Example: 100 GB volume
│  ├─ Monthly cost: 100 GB × $0.05 = $5/month
│  └─ Annual cost: $5 × 12 = $60/year
├─ Multiple snapshots: Costs accumulate
└─ Retention: No automatic cleanup

RDS Snapshot:
├─ Backup storage: $0.095 per GB-month
├─ Example: 500 GB database
│  ├─ Monthly cost: 500 GB × $0.095 = $47.50/month
│  └─ Annual cost: $47.50 × 12 = $570/year
├─ Snapshots keep data indefinitely
└─ Manual deletion required

ElastiCache Snapshot:
├─ Backup storage (varies by region)
├─ Usually: $0.02-0.03 per GB-month
├─ Accumulate if not cleaned
└─ Manual cleanup: Required

Snapshot Cleanup:

Forgotten Snapshots:
├─ Created: Year ago (DeletionPolicy: Snapshot)
├─ Still existing: Yes
├─ Billing: Ongoing (~$5-50/month each)
├─ Examples: 10 forgotten snapshots = $50-500/month
└─ Impact: Runaway cloud costs

Best Practice:

Snapshot Lifecycle:
├─ Take: Final snapshot before deletion
├─ Store: 30-90 days for recovery
├─ Archive: Move old snapshots to Glacier
├─ Delete: After retention period ends
├─ Review: Monthly snapshot inventory
└─ Tag: Snapshots with creation date, purpose, email
```

## Part 5: Real-World Scenario - Hands-On Lab

### DeletionPolicy Demo Lab

**Template: deletionpolicy.yaml**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'DeletionPolicy Demonstration'

Parameters:
  AvailabilityZone:
    Type: AWS::EC2::AvailabilityZone::Name
    Description: 'Choose an availability zone'

Resources:
  # Security Group - RETAINED (Retained policy)
  ApplicationSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Application security group - will be retained
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
          Description: HTTPS access
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: HTTP access
      Tags:
        - Key: Name
          Value: DemoAppSecurityGroup
    DeletionPolicy: Retain
    # This security group will be RETAINED after stack deletion

  # EBS Volume - SNAPSHOT (Snapshot policy)
  DataVolume:
    Type: AWS::EC2::Volume
    Properties:
      AvailabilityZone: !Ref AvailabilityZone
      Size: 1  # 1 GB for demo
      VolumeType: gp3
      Encrypted: true
      Tags:
        - Key: Name
          Value: DemoDataVolume
    DeletionPolicy: Snapshot
    # Final snapshot will be created, then volume deleted

Outputs:
  SecurityGroupId:
    Value: !GetAtt ApplicationSecurityGroup.GroupId
    Description: 'Security Group ID (will be retained)'
    Export:
      Name: DemoSecurityGroupId

  VolumeId:
    Value: !GetAtt DataVolume.VolumeId
    Description: 'EBS Volume ID (will be snapshotted then deleted)'
    Export:
      Name: DemoVolumeId
```

**Lab Steps: Creation**

```
Step 1: Create Stack

AWS Console:
├─ CloudFormation → Create Stack
├─ Upload template: deletionpolicy.yaml
├─ Stack name: DeletionPolicyDemo
├─ Parameters: Choose AZ (e.g., us-east-1a)
├─ Options: Default
├─ Review: Check IAM/Capabilities
├─ Create: Launch stack

Expected Result:
├─ Stack: CREATE_IN_PROGRESS
├─ Events:
│  ├─ ApplicationSecurityGroup (CREATE_IN_PROGRESS)
│  ├─ ApplicationSecurityGroup (CREATE_COMPLETE)
│  ├─ DataVolume (CREATE_IN_PROGRESS)
│  ├─ DataVolume (CREATE_COMPLETE)
│  └─ DeletionPolicyDemo (CREATE_COMPLETE)
├─ Status: All resources created
└─ Time: ~2-3 minutes

Verify Creation:

EC2 Console:
├─ Security Groups: ApplicationSecurityGroup visible
├─ Volumes: DataVolume visible (1 GB, encrypted)
└─ Tags: Name=DemoAppSecurityGroup, Name=DemoDataVolume
```

**Lab Steps: Deletion with Observation**

```
Step 2: Delete Stack and Observe DeletionPolicy Behavior

CloudFormation Console:
├─ Select: DeletionPolicyDemo stack
├─ Click: Delete
├─ Confirm: Yes, delete

Watch Events Tab:
├─ DeletionPolicyDemo (DELETE_IN_PROGRESS)

Application Security Group (Retain):
├─ ApplicationSecurityGroup (DELETE_SKIPPED)
├─ Message: "Skipped (DeletionPolicy: Retain)"
├─ Status: RETAINED

Data Volume (Snapshot):
├─ DataVolume-1 (CREATE_IN_PROGRESS)
│  └─ Final snapshot created
├─ DataVolume-1 (CREATE_COMPLETE)
│  └─ Snapshot ID: snap-0abc123def456 (example)
├─ DataVolume (DELETE_IN_PROGRESS)
├─ DataVolume (DELETE_COMPLETE)
├─ Message: "Successfully deleted volume after creating snapshot"
└─ Status: DELETED with SNAPSHOT

Final Result:
├─ DeletionPolicyDemo (DELETE_COMPLETE)
├─ Stack: Deleted successfully
└─ Status: DELETE_COMPLETE
```

**Lab Steps: Post-Deletion Verification**

```
Step 3: Verify Retained and Snapshotted Resources

EC2 Security Groups:
├─ Go to: EC2 → Security Groups
├─ Search: DemoAppSecurityGroup
├─ Result: FOUND (not deleted)
├─ Status: Active
├─ Inbound Rules:
│  ├─ 443/tcp from 0.0.0.0/0 (HTTPS)
│  └─ 80/tcp from 0.0.0.0/0 (HTTP)
├─ Notes: Still exists, not managed by CF
└─ Action: Manual deletion required

EC2 Volumes:
├─ Go to: EC2 → Volumes
├─ Search: DemoDataVolume
├─ Result: NOT FOUND (successfully deleted)
├─ Status: Volume is gone
└─ Snapshot: Taken before deletion

EC2 Snapshots:
├─ Go to: EC2 → Snapshots
├─ Filter: All snapshots
├─ Look for: recent snap-* (from demo)
├─ Result: FOUND (1 GB snapshot)
├─ Details:
│  ├─ Snapshot ID: snap-0abc123def456 (example)
│  ├─ Volume ID: vol-12345678 (original volume)
│  ├─ Created: Just now (minutes ago)
│  ├─ Size: 1 GB
│  ├─ Status: completed
│  └─ Description: (may be empty or auto-generated)

Tags on Resources:
├─ Security Group:
│  └─ Name: DemoAppSecurityGroup (visible)
├─ Snapshot:
│  └─ No tags (but original volume name preserved)
```

**Lab Steps: Cleanup**

```
Step 4: Manual Cleanup of Retained/Snapshotted Resources

Cleanup Order:
├─ Security Group (still in use? Check references first)
├─ Snapshot (archive first if needed)
└─ Document what was kept

Cleanup Process:

1. Security Group Cleanup (if not needed):
   ├─ EC2 → Security Groups
   ├─ Select: DemoAppSecurityGroup
   ├─ Actions: Delete security group
   ├─ Confirm: Yes, delete
   └─ Result: Removed (manual)

2. Snapshot Archival (if long-term storage):
   ├─ EC2 → Snapshots
   ├─ Select: snap-0abc123def456
   ├─ Actions: Copy to and tag with date
   ├─ Archive location: S3 or separate snapshot copy
   └─ Benefit: Cost-effective long-term storage

3. Snapshot Deletion (if not needed):
   ├─ EC2 → Snapshots
   ├─ Select: snap-0abc123def456
   ├─ Actions: Delete snapshot
   ├─ Confirm: Yes, delete
   └─ Result: Removed (manual)

Cost Verification:

Before Cleanup:
├─ Security Group: $0 (no charges)
├─ Snapshot (1 GB): ~$0.05/month
├─ Total: $0.05/month

After Cleanup:
├─ All resources removed
├─ No charges
└─ Total: $0

Learning:
├─ DeletionPolicy: Retain = Manual deletion later
├─ DeletionPolicy: Snapshot = Final backup, manual deletion later
├─ Forgot to cleanup = Ongoing charges
└─ Always: Document retention rationale
```

## Part 6: Best Practices and Exam Focus

### Decision Matrix: Which DeletionPolicy?

```
┌──────────────────────────────────────────────┬────────────┬──────────┬──────────┐
│ Resource Type / Scenario                     │ Delete     │ Retain   │ Snapshot │
├──────────────────────────────────────────────┼────────────┼──────────┼──────────┤
│ EC2 Instance (temporary)                     │ ✓ Default  │          │          │
│ EC2 Instance (production)                    │            │ ✓ Keep   │ ✓ Also   │
│ EBS Volume (application)                     │            │          │ ✓ Best   │
│ S3 Bucket (temporary)                        │ ✓ If empty │ Risky    │ N/A      │
│ S3 Bucket (data archive)                     │            │ ✓ Best   │ N/A      │
│ RDS Database (production)                    │            │ ✓ or     │ ✓ Best   │
│ RDS Database (dev/test)                      │ ✓ Quick    │ ✓ Maybe  │          │
│ DynamoDB Table (critical)                    │            │ ✓ Keep   │ N/A      │
│ ElastiCache (session store)                  │ ✓ Default  │          │ ✓ Archive│
│ Redshift Cluster (analytics)                 │            │ ✓ or     │ ✓ Final  │
│ Security Group (org-wide)                    │            │ ✓ Shared │          │
│ IAM Role (production)                        │            │ ✓ Shared │ N/A      │
│ Lambda Function (code)                       │ ✓ Default  │          │ N/A      │
│ CloudWatch Loggroup (compliance)             │            │ ✓ Keep   │ N/A      │
└──────────────────────────────────────────────┴────────────┴──────────┴──────────┘

Legend:
├─ ✓ Best = Recommended for this resource type
├─ ✓ Also = Alternative that works
├─ Risky = Can work but dangerous
├─ N/A = Not supported by service
└─ ? = Consider context
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What is the default DeletionPolicy?"
A) Retain
B) Snapshot
C) Delete
D) Preserve

Answer: C (Delete - automatic removal)

Q2: "DeletionPolicy Retain preserves resource when:"
A) Template updated
B) Resource property changed
C) Stack deleted
D) Never

Answer: C (Stack deletion only)

Q3: "S3 bucket with DeletionPolicy Delete fails when:"
A) Resource is very large
B) Bucket has read-only policies
C) Bucket is not empty
D) CloudFormation has insufficient permissions

Answer: C (Non-empty bucket exception)

Q4: "Which service does NOT support DeletionPolicy Snapshot?"
A) RDS DBInstance
B) EBS Volume
C) S3 Bucket
D) Redshift Cluster

Answer: C (S3 doesn't support snapshot)

Q5: "Orphaned resources after stack deletion occur with:"
A) DeletionPolicy Delete
B) DeletionPolicy Retain
C) DeletionPolicy Snapshot
D) All deletion policies

Answer: B (Retain creates orphaned resources)

Q6: "What happens to snapshot after cluster deleted:"
A) Deleted automatically after 7 days
B) Preserved indefinitely (manual deletion)
C) Moved to archive automatically
D) Deleted immediately

Answer: B (Snapshots persist, manual cleanup needed)

Key Exam Concepts:
├─ Default: Delete (automatic removal)
├─ Retain: Preserves resources (manual cleanup)
├─ Snapshot: Final backup before deletion
├─ S3 Exception: Delete fails if bucket not empty
├─ Orphaned: Retained/snapshotted resources not managed by CF
├─ Costs: Retained resources and snapshots incur charges
└─ Manual: Cleanup required for retained/snapshotted resources
```

### Best Practices Checklist

```
✓ Use DeletionPolicy Delete by Default
  ├─ Standard for temporary/dev resources
  ├─ Clean removal on stack deletion
  ├─ No orphaned resources
  └─ Cost-effective

✓ Use DeletionPolicy Retain for Critical Data
  ├─ Databases with irreplaceable data
  ├─ S3 buckets with archives
  ├─ Regulatory compliance data
  ├─ Shared infrastructure resources
  └─ Always document rationale

✓ Use DeletionPolicy Snapshot for Databases
  ├─ RDS: Always snapshot on deletion
  ├─ Redshift: Always snapshot on deletion
  ├─ ElastiCache: Snapshot for important data
  ├─ EBS: Snapshot volumes with data
  └─ Final backup guaranteed

✓ Document All Retained Resources
  ├─ Add comments in template
  ├─ Explain why resource retained
  ├─ Note manual cleanup requirement
  ├─ Update runbooks for operators
  └─ Tag resources with retention reason

✓ Handle S3 Bucket Deletion Carefully
  ├─ Must empty bucket manually OR
  ├─ Use custom resource to auto-empty OR
  ├─ Use DeletionPolicy Retain
  ├─ Plan S3 cleanup in advance
  └─ Don't forget S3 versioning cleanup

✓ Set Snapshots or Snapshots to Archive
  ├─ Review snapshots monthly
  ├─ Delete old snapshots not needed
  ├─ Archive important snapshots to Glacier
  ├─ Remove snapshot tags after retention
  └─ Prevent runaway snapshot costs

✓ Plan for Orphaned Resource Cleanup
  ├─ Runbooks must include manual cleanup
  ├─ Assign: Who deletes orphaned resources
  ├─ Timeline: When to delete (30/60/90 days)
  ├─ Verification: Confirm all deleted
  └─ Cost analysis: Impact of forgotten cleanup

✓ Test DeletionPolicy Behavior
  ├─ Test stack: Before production
  ├─ Verify: What gets deleted/retained
  ├─ Snapshot: Confirm created properly
  ├─ Cleanup: Test orphaned resource removal
  ├─ Timing: Understand stack deletion duration
  └─ Rollback: Have recovery plan
```

---

**Total Words: ~8,500**  
**File Created: 13_CloudFormation_DeletionPolicy.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
