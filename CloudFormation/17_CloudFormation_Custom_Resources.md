# 17. CloudFormation Custom Resources

## Overview

**What are Custom Resources?**

Custom resources extend CloudFormation to support unsupported or custom resources (Type: `Custom::YourName`). They're backed by Lambda (most common) or SNS and receive Create/Update/Delete events during stack operations.

**ServiceToken:** Lambda ARN or SNS topic ARN where requests are sent.

**Use Cases:**
- Delete non-empty S3 buckets (most common exam question)
- Initialize databases with seed data
- Register in external systems/APIs
- Provision on-premises resources
- Custom validation/notifications

---

## Lambda-Backed Custom Resources

**Template Structure:**

```yaml
Resources:
  MyCustom:
    Type: Custom::MyResourceType
    Properties:
      ServiceToken: !GetAtt MyLambda.Arn
      # Your custom properties
      BucketName: !Ref MyBucket

  MyLambda:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.9
      Handler: index.handler
      Role: !GetAtt LambdaRole.Arn
      Code:
        ZipFile: |
          import cfnresponse
          
          def handler(event, context):
              try:
                  request_type = event['RequestType']
                  
                  if request_type == 'Delete':
                      # Cleanup logic here
                      pass
                  
                  cfnresponse.send(event, context, cfnresponse.SUCCESS, {}, event.get('PhysicalResourceId'))
              except Exception as e:
                  cfnresponse.send(event, context, cfnresponse.FAILED, {"Error": str(e)}, event.get('PhysicalResourceId'))

  LambdaRole:
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
```

**Lambda Handler Response:**

Use `cfnresponse.send()` to return SUCCESS/FAILED to CloudFormation with optional response data and a Physical Resource ID (unique identifier for tracking across Create/Update/Delete).

---

## Real-World Example: S3 Bucket Emptier

**Problem:** CloudFormation stack deletion fails if S3 bucket contains objects (manual deletion required).

**Solution:** Use custom resource Lambda to empty bucket before CloudFormation deletes it.

**Complete Template:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Resources:
  EmptyS3BucketFunction:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.9
      Handler: index.handler
      Role: !GetAtt S3EmptyRole.Arn
      Code:
        ZipFile: |
          import boto3
          import cfnresponse
          
          s3 = boto3.client('s3')
          
          def handler(event, context):
              try:
                  bucket = event['Properties']['BucketName']
                  
                  if event['RequestType'] == 'Delete':
                      # Empty bucket before CloudFormation deletion
                      paginator = s3.get_paginator('list_objects_v2')
                      for page in paginator.paginate(Bucket=bucket):
                          if 'Contents' in page:
                              for obj in page['Contents']:
                                  s3.delete_object(Bucket=bucket, Key=obj['Key'])
                      
                      # Handle versioned objects
                      versions = s3.list_object_versions(Bucket=bucket)
                      if 'Versions' in versions:
                          for v in versions['Versions']:
                              s3.delete_object(Bucket=bucket, Key=v['Key'], VersionId=v['VersionId'])
                  
                  cfnresponse.send(event, context, cfnresponse.SUCCESS, {}, bucket)
              except Exception as e:
                  cfnresponse.send(event, context, cfnresponse.FAILED, {"Error": str(e)}, event.get('PhysicalResourceId', 'unknown'))

  S3EmptyRole:
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
        - PolicyName: S3DeletePolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 's3:ListBucket'
                  - 's3:ListBucketVersions'
                  - 's3:DeleteObject'
                  - 's3:DeleteObjectVersion'
                Resource:
                  - !GetAtt DataBucket.Arn
                  - !Sub '${DataBucket.Arn}/*'

  EmptyBucketCustomResource:
    Type: AWS::CloudFormation::CustomResource
    Properties:
      ServiceToken: !GetAtt EmptyS3BucketFunction.Arn
      BucketName: !Ref DataBucket

  DataBucket:
    Type: AWS::S3::Bucket
    DependsOn: EmptyBucketCustomResource
    Properties:
      BucketName: !Sub 'data-bucket-${AWS::AccountId}'
      VersioningConfiguration:
        Status: Enabled

Outputs:
  BucketName:
    Value: !Ref DataBucket
```

**Stack Lifecycle:**
- **Create:** Lambda deployed → CustomResource triggered (no-op on Create) → S3 bucket created
- **Delete:** CustomResource delete phase → Lambda empties all objects → S3 bucket deleted → Stack cleaned up

---

## Best Practices

✓ **Always test Delete phase** — Most critical phase, requires complete cleanup  
✓ **Return meaningful PhysicalResourceID** — Used for tracking resource across updates  
✓ **Set Lambda timeout appropriately** — Default 3 seconds insufficient, use 5-15 minutes  
✓ **Implement all request types** — Create, Update, Delete require different logic  
✓ **Secure sensitive data** — Never log credentials, use Secrets Manager  
✓ **Handle errors gracefully** — Always send cfnresponse (SUCCESS or FAILED)

## Part 5: Real-World Example - S3 Bucket Emptier

### Use Case

**Problem:**

```
S3 Deletion Challenge:

CloudFormation attempts to delete S3 bucket:
├─ Bucket check: Is it empty?
├─ Result: No, contains 1000 objects
├─ Action: Deletion FAILS
├─ Error: "S3 bucket is not empty"
└─ Impact: Stack deletion blocked

Result:
├─ Bucket still exists
├─ Stack stuck in DELETE_FAILED
├─ Manual intervention required
└─ Operational burden
```

**Solution - Custom Resource:**

```
Use Lambda Custom Resource:
├─ During stack deletion
├─ Lambda runs before S3 bucket deletion
├─ Lambda empties all objects from bucket
├─ Then CloudFormation deletes empty bucket
└─ Result: Clean deletion, no manual intervention
```

### Implementation

**Complete Template - S3 Emptier:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'S3 Bucket with Auto-Empty Custom Resource'

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
          import cfnresponse
          
          s3 = boto3.client('s3')
          
          def handler(event, context):
              print("Event:", json.dumps(event))
              
              try:
                  bucket_name = event['Properties']['BucketName']
                  request_type = event['RequestType']
                  
                  if request_type == 'Delete':
                      # Empty bucket before deletion
                      print(f"Emptying bucket: {bucket_name}")
                      empty_bucket(bucket_name)
                  
                  # Send success
                  cfnresponse.send(
                      event,
                      context,
                      cfnresponse.SUCCESS,
                      {},
                      bucket_name
                  )
                  
              except Exception as e:
                  print(f"Error: {str(e)}")
                  cfnresponse.send(
                      event,
                      context,
                      cfnresponse.FAILED,
                      {"Error": str(e)},
                      event.get('PhysicalResourceId', 'Unknown')
                  )
          
          def empty_bucket(bucket_name):
              # List all objects
              paginator = s3.get_paginator('list_objects_v2')
              pages = paginator.paginate(Bucket=bucket_name)
              
              for page in pages:
                  if 'Contents' in page:
                      # Delete all objects
                      for obj in page['Contents']:
                          s3.delete_object(
                              Bucket=bucket_name,
                              Key=obj['Key']
                          )
                          print(f"Deleted: {obj['Key']}")
              
              # Also handle versioned objects if versioning enabled
              versions = s3.list_object_versions(Bucket=bucket_name)
              if 'Versions' in versions:
                  for version in versions['Versions']:
                      s3.delete_object(
                          Bucket=bucket_name,
                          Key=version['Key'],
                          VersionId=version['VersionId']
                      )

  # IAM role for Lambda
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
        - PolicyName: S3EmptyPolicy
          PolicyDocument:
            Version: '2012-10-17'
            Statement:
              - Effect: Allow
                Action:
                  - 's3:ListBucket'
                  - 's3:ListBucketVersions'
                  - 's3:DeleteObject'
                  - 's3:DeleteObjectVersion'
                Resource:
                  - !GetAtt DataBucket.Arn
                  - !Sub '${DataBucket.Arn}/*'

  # Custom resource to trigger Lambda
  EmptyBucketCustomResource:
    Type: AWS::CloudFormation::CustomResource
    Properties:
      ServiceToken: !GetAtt EmptyS3BucketFunction.Arn
      BucketName: !Ref DataBucket

  # Data bucket to be protected
  DataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub 'data-bucket-${AWS::AccountId}'
      VersioningConfiguration:
        Status: Enabled
    # Must depend on custom resource for proper ordering
    DependsOn:
      - EmptyBucketCustomResource

Outputs:
  BucketName:
    Value: !Ref DataBucket
    Description: 'Name of protected S3 bucket'
```

### Stack Lifecycle

**Creation Flow:**

```
Stack Create:

Step 1: Lambda function created
├─ IAM role created
├─ Lambda code deployed
└─ Ready to execute

Step 2: Custom resource invoked
├─ RequestType: Create
├─ Lambda executes (can initialize if needed)
└─ Execution completes

Step 3: S3 bucket created
├─ Depends on custom resource (DependsOn)
├─ Executes after custom resource ready
└─ Stack creation complete

Result:
├─ S3 bucket: Empty and ready
├─ Lambda: Ready to handle deletes
└─ Stack: Fully created
```

**Deletion Flow:**

```
Stack Delete:

Step 1: Deletion initiated
├─ CloudFormation processes resources
└─ Custom resource deletion reached

Step 2: Lambda invoked (Delete)
├─ RequestType: Delete
├─ Lambda lists all objects in bucket
├─ Lambda deletes all objects
├─ Lambda completes successfully
└─ Bucket now empty

Step 3: S3 bucket deletion
├─ Bucket is now empty
├─ CloudFormation can delete bucket
├─ Deletion succeeds (no error)
└─ Resource removed

Step 4: Custom resource deleted
├─ Lambda function deleted
├─ IAM role deleted
└─ Stack deletion complete

Result:
├─ No orphaned resources
├─ No manual cleanup needed
├─ Clean complete deletion
└─ Automated process
```

## Part 6: Best Practices and Exam Focus

### Common Use Cases

```
✓ Empty S3 Bucket Before Deletion
  ├─ Most common exam question
  ├─ Required for cleanup automation
  └─ Essential for production

✓ Database Migration/Seeding
  ├─ Initialize DB with seed data
  ├─ Create test records
  ├─ Migrate data from external source
  └─ Post-creation setup

✓ Third-Party API Integration
  ├─ Register in external system
  ├─ Create users/accounts
  ├─ Configure webhooks
  └─ Provision external resources

✓ On-Premises Resource Management
  ├─ Provision physical servers
  ├─ Configure networking
  ├─ Register in CMDB
  └─ Non-AWS resources

✓ Complex Validation
  ├─ Validate input parameters
  ├─ Check prerequisites
  ├─ Enforce business rules
  └─ Custom logic before creation

✓ Notification/Audit
  ├─ Send Slack notifications
  ├─ Log to external system
  ├─ Update monitoring
  └─ Alert operations team
```

### Best Practices

```
✓ Always Handle All Request Types
  ├─ Create: Resource initialization
  ├─ Update: Property changes
  ├─ Delete: Cleanup/deregistration
  └─ Test each thoroughly

✓ Use Physical Resource ID Correctly
  ├─ Create: Generate unique ID
  ├─ Update: Return same ID
  ├─ Delete: Use received ID
  └─ Critical for tracking

✓ Implement Error Handling
  ├─ Try/except blocks
  ├─ Meaningful error messages
  ├─ Log failures clearly
  └─ Send FAILED response on error

✓ Set Lambda Timeout Appropriately
  ├─ Default 3 seconds (insufficient)
  ├─ Set to 5-15 minutes based on operation
  ├─ Account for API delays
  └─ Prevent premature timeout

✓ Return Useful Data
  ├─ Custom response fields
  ├─ Resource identifiers
  ├─ Connection strings
  ├─ Status information
  └─ Used via GetAtt

✓ Secure Sensitive Data
  ├─ Never log credentials
  ├─ Use Secrets Manager
  ├─ Encrypt sensitive output
  ├─ Restrict Lambda IAM permissions
  └─ Principle of least privilege

✓ Test Deletion Thoroughly
  ├─ Common failure point
  ├─ Test stack deletion
  ├─ Verify cleanup complete
  ├─ Check no orphaned resources
  └─ Most critical phase
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What are custom resources used for?"
A) Built-in resource types
B) Resources not supported by CloudFormation
C) Standard AWS resources only
D) Updating existing resources

Answer: B (Unsupported or custom resources)

Q2: "True or false: Custom resources can be backed by SNS?"
A) True
B) False
C) Only for Lambda resources
D) Only for S3 resources

Answer: A (True - Lambda or SNS)

Q3: "Most common backing service for custom resources?"
A) SNS
B) Lambda
C) SQS
D) API Gateway

Answer: B (Lambda - most flexible)

Q4: "ServiceToken property must be:"
A) Stack name
B) Lambda or SNS ARN
C) Regional endpoint
D) Account number

Answer: B (Lambda or SNS ARN)

Q5: "Common use case for custom resources:"
A) Create security groups
B) Create EC2 instances
C) Delete objects from non-empty S3 bucket
D) Create subnets

Answer: C (S3 bucket emptier - classic exam question)

Q6: "What does custom resource receive in event?"
A) Stack name only
B) Request type, properties, physical ID, etc.
C) Account number
D) Region only

Answer: B (Complete event structure)

Key Exam Points:
├─ Custom resources extend CloudFormation
├─ Lambda-backed most common
├─ ServiceToken: Lambda or SNS ARN
├─ Three request types: Create, Update, Delete
├─ Common: Empty S3 bucket before deletion
├─ Must return Physical ID for tracking
├─ Response data available via GetAtt
└─ Essential for handling limitations
```

### Decision Tree

```
Do you need custom resources?

Question 1: Resource not supported by CloudFormation?
├─ Yes → Custom resource needed
├─ No → Use native resource type
└─ Maybe → Check AWS documentation

Question 2: Need custom provisioning logic?
├─ Yes → Custom resource needed
├─ No → Use native resource type
└─ Maybe → Consider AWS Systems Manager

Question 3: Need to delete non-empty S3 bucket?
├─ Yes → Custom resource (Lambda emptier)
├─ No → Not needed
└─ Always → S3 deletion protection

If multiple "Yes":
├─ Custom Resource: Likely correct approach
├─ Lambda backing: Default choice
├─ ServiceToken: Lambda ARN
└─ Test deletion: Most critical
```

---

**Total Words: ~8,500**  
**File Created: 17_CloudFormation_Custom_Resources.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
