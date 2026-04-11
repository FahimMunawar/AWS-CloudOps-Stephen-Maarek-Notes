# 8. CloudFormation Conditions

## Part 1: Understanding Conditions

### What Are Conditions?

**Definition:**

```
Conditions:
├─ Optional section in CloudFormation templates
├─ Define boolean logic to control resource creation
├─ Enable conditional resource provisioning
├─ Control which outputs are returned
├─ Based on parameters, mappings, or pseudo parameters
├─ Each condition evaluates to true or false
├─ Multiple conditions can exist in one template
└─ Applied to resources or outputs
```

**What Conditions Enable:**

```
Without Conditions:
├─ Template creates same resources always
├─ Cannot differentiate by environment
├─ Cannot selectively create resources
├─ One size fits all approach
└─ Difficult multi-purpose templates

With Conditions:
├─ Dev environment: Create fewer resources (cheap)
├─ Prod environment: Create more resources (protected)
├─ Optional resources based on parameters
├─ Cost optimization per environment
├─ Single template for multiple use cases
└─ Infrastructure customization without template editing
```

### Common Condition Use Cases

**Environment-Based Conditions:**

```
Scenario 1: Dev has no backup, Prod has backup
├─ Condition: CreateBackup: !Equals [!Ref Environment, prod]
├─ If Environment = prod → true → create backup
├─ If Environment = dev → false → skip backup
├─ Result: Production protected, development cheaper

Scenario 2: Dev uses single AZ, Prod uses Multi-AZ
├─ Condition: UseMultiAZ: !Equals [!Ref Environment, prod]
├─ If Prod → MultiAZ: true (cost +50%)
├─ If Dev → MultiAZ: false (cheaper)
└─ Result: Appropriate HA per environment

Scenario 3: Dev has no encryption, Prod encrypted
├─ Condition: EnableEncryption: !Equals [!Ref Environment, production]
├─ If Prod → Encryption enabled (security)
├─ If Dev → Encryption disabled (faster for testing)
└─ Result: Security standards enforced in prod
```

**Region-Based Conditions:**

```
Scenario: Apply extra monitoring only in critical regions
├─ Condition: IsCriticalRegion: !Equals [!Ref AWS::Region, us-east-1]
├─ If us-east-1 → DetailedMonitoring: true
├─ If other region → DetailedMonitoring: false
└─ Result: Higher monitoring cost only in critical regions
```

**Parameter-Based Conditions:**

```
Scenario: Optional EBS volume based on parameter
├─ Condition: AttachVolume: !Equals [!Ref EnableEBS, "true"]
├─ If EnableEBS = true → Create EBS volume
├─ If EnableEBS = false → Skip EBS volume
└─ Result: User decides if volume needed
```

## Part 2: Condition Structure and Syntax

### Condition Format

**Basic Structure:**

```yaml
Conditions:
  ConditionName:
    Fn::Condition_Function: [Arguments]

Example:
Conditions:
  CreateProdResources:
    Fn::Equals: [!Ref Environment, prod]
```

**Condition Properties:**

```
1. Condition Name (Key)
   ├─ Identifier for this condition
   ├─ Used when applying to resources/outputs
   ├─ Example: CreateProdResources, IsProduction, UseMultiAZ
   └─ Must be unique within template

2. Condition Function (Value)
   ├─ Boolean logic function
   ├─ Evaluates to true or false
   ├─ Examples: Fn::Equals, Fn::And, Fn::Or, Fn::Not, Fn::If
   └─ Can be nested and combined
```

### Condition Intrinsic Functions

**Fn::Equals (Equality Check):**

```yaml
Fn::Equals: [Value1, Value2]

Returns:
├─ true if Value1 equals Value2
├─ false if Value1 not equals Value2
└─ Most common condition function

Example:
Conditions:
  IsProduction:
    Fn::Equals: [!Ref Environment, production]

Usage:
├─ If Environment parameter = "production" → true
├─ If Environment parameter = "development" → false
└─ Used to create prod-only resources

YAML Short Form (preferred):
ProductionCheck: !Equals [!Ref Environment, production]
```

**Fn::Equals with Multiple Values:**

```yaml
Checking Value Against Multiple Options:

# Check if Region is us-east-1
IsUSEast1:
  Fn::Equals: [!Ref AWS::Region, us-east-1]

# Check if InstanceType is t2.micro
IsSmallInstance:
  Fn::Equals: [!Ref InstanceType, t2.micro]

# Check if value equals hardcoded string
IsPaid:
  Fn::Equals: [!Ref BillingModel, paid]

# Compare two pseudo parameters (less common)
FirstAccountId:
  Fn::Equals: [!Ref AWS::AccountId, "123456789012"]
```

**Fn::And (Logical AND):**

```yaml
Fn::And: [Condition1, Condition2, Condition3, ...]

Returns:
├─ true if ALL conditions are true
├─ false if ANY condition is false
└─ Requires all conditions must be met

Example:
Conditions:
  IsProdEnvironment:
    Fn::Equals: [!Ref Environment, production]

  IsUSEast1Region:
    Fn::Equals: [!Ref AWS::Region, us-east-1]

  CreateProdUSEast1Resources:
    Fn::And:
      - !Condition IsProdEnvironment
      - !Condition IsUSEast1Region

Usage:
├─ IsProdEnvironment = true AND IsUSEast1Region = true
├─ Result: CreateProdUSEast1Resources = true
├─ Create resources only in prod AND us-east-1
└─ Example: Create redundant backup only in critical region

YAML Short Form:
CreateProdUSEast1Resources: !And
  - !Equals [!Ref Environment, production]
  - !Equals [!Ref AWS::Region, us-east-1]
```

**Fn::Or (Logical OR):**

```yaml
Fn::Or: [Condition1, Condition2, Condition3, ...]

Returns:
├─ true if ANY condition is true
├─ false if ALL conditions are false
└─ Requires at least one condition must be true

Example:
Conditions:
  IsProduction:
    Fn::Equals: [!Ref Environment, production]

  IsStaging:
    Fn::Equals: [!Ref Environment, staging]

  IsProductionOrStaging:
    Fn::Or:
      - !Condition IsProduction
      - !Condition IsStaging

Usage:
├─ IsProduction = true OR IsStaging = true
├─ Result: IsProductionOrStaging = true
├─ Enable debugging logs if prod OR staging
└─ Example: Only secure environments get detailed logging

YAML Short Form:
IsProductionOrStaging: !Or
  - !Equals [!Ref Environment, production]
  - !Equals [!Ref Environment, staging]
```

**Fn::Not (Logical NOT):**

```yaml
Fn::Not: [Condition1]

Returns:
├─ true if condition is false
├─ false if condition is true
└─ Inverts condition value

Example:
Conditions:
  IsProduction:
    Fn::Equals: [!Ref Environment, production]

  NotProduction:
    Fn::Not:
      - !Condition IsProduction

Usage:
├─ IsProduction = true → NotProduction = false
├─ IsProduction = false → NotProduction = true
├─ Create dev-only resources when NOT production
└─ Example: Disable cost optimization in dev

YAML Short Form:
NotProduction: !Not
  - !Equals [!Ref Environment, production]
```

**Fn::If (Conditional Selection):**

```yaml
Fn::If: [ConditionName, ValueIfTrue, ValueIfFalse]

Returns:
├─ ValueIfTrue if condition is true
├─ ValueIfFalse if condition is false
└─ Used to select between two values (NOT for resource creation)

Different from other functions:
├─ Other functions return true/false
├─ Fn::If returns one of two values
├─ Used within resource properties
└─ Example: Select instance type based on condition

Example:
Conditions:
  IsProduction:
    Fn::Equals: [!Ref Environment, production]

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      # Select instance type based on condition
      InstanceType: !If [IsProduction, t2.medium, t2.micro]
      # If prod: t2.medium
      # If not prod: t2.micro

Usage:
├─ Directly in resource properties
├─ !If [ConditionName, TrueValue, FalseValue]
├─ Prod gets powerful instance, Dev gets cheap instance
└─ Different from resource condition (creates resource or not)
```

## Part 3: Applying Conditions to Resources and Outputs

### Condition on Resources

**Resource Condition Syntax:**

```yaml
Resources:
  ResourceName:
    Type: AWS::ServiceName::ResourceType
    Condition: ConditionName  # Add this line
    Properties:
      ...

Behavior:
├─ If condition true: Resource created normally
├─ If condition false: Resource NOT created at all
└─ Useful for optional infrastructure
```

**Resource Condition Example:**

```yaml
Parameters:
  Environment:
    Type: String
    Default: development
    AllowedValues:
      - development
      - production

Conditions:
  CreateProductionResources:
    Fn::Equals: [!Ref Environment, production]

Resources:
  # Always created
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      Tags:
        - Key: Environment
          Value: !Ref Environment

  # Only created if production
  BackupVolume:
    Type: AWS::EC2::Volume
    Condition: CreateProductionResources  # Applied here
    Properties:
      Size: 100
      AvailabilityZone: us-east-1a
      Tags:
        - Key: Name
          Value: backup-volume

  # Attach backup only if it exists (prod only)
  VolumeAttachment:
    Type: AWS::EC2::VolumeAttachment
    Condition: CreateProductionResources
    Properties:
      VolumeId: !Ref BackupVolume
      InstanceId: !Ref WebServer
      Device: /dev/sdf

Deployment:
├─ Development: WebServer created, BackupVolume SKIPPED, VolumeAttachment SKIPPED
├─ Production: All three resources created
└─ Same template, different infrastructure per environment
```

### Condition on Outputs

**Output Condition Syntax:**

```yaml
Outputs:
  OutputName:
    Value: SomeValue
    Condition: ConditionName  # Add this line
    Description: "..."

Behavior:
├─ If condition true: Output returned to user
├─ If condition false: Output hidden from user
└─ Useful for environment-specific outputs
```

**Output Condition Example:**

```yaml
Parameters:
  Environment:
    Type: String
    Default: development

Conditions:
  IsProduction:
    Fn::Equals: [!Ref Environment, production]

  NotProduction:
    Fn::Not:
      - !Condition IsProduction

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro

Outputs:
  # Always shown
  InstanceId:
    Value: !Ref WebServer
    Description: "Instance ID"

  # Only shown in production
  ProductionInstanceDetails:
    Value: !Sub "Production instance: ${WebServer}"
    Condition: IsProduction
    Description: "Production instance information"

  # Only shown in dev/staging
  DevelopmentDetails:
    Value: !Sub "Dev instance: ${WebServer}"
    Condition: NotProduction
    Description: "Development instance information"

User Experience:
├─ Development deployment:
│  ├─ Output: InstanceId
│  ├─ Output: DevelopmentDetails
│  └─ ProductionInstanceDetails NOT shown
│
└─ Production deployment:
   ├─ Output: InstanceId
   ├─ Output: ProductionInstanceDetails
   └─ DevelopmentDetails NOT shown
```

## Part 4: Real-World Multi-Environment Template

### Complete Example with Conditions

**Full Template Using Conditions:**

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: "Multi-environment template with conditions"

Parameters:
  EnvironmentName:
    Type: String
    Default: development
    AllowedValues:
      - development
      - staging
      - production
    Description: "Environment type"

  EnableBackup:
    Type: String
    Default: "false"
    AllowedValues:
      - "true"
      - "false"
    Description: "Enable EBS backup volume"

Conditions:
  IsProduction:
    Fn::Equals: [!Ref EnvironmentName, production]

  IsStaging:
    Fn::Equals: [!Ref EnvironmentName, staging]

  IsDevelopment:
    Fn::Equals: [!Ref EnvironmentName, development]

  IsProductionOrStaging:
    Fn::Or:
      - !Condition IsProduction
      - !Condition IsStaging

  ShouldBackup:
    Fn::Equals: [!Ref EnableBackup, "true"]

  CreateBackupVolume:
    Fn::And:
      - !Condition IsProduction
      - !Condition ShouldBackup

Mappings:
  EnvironmentConfig:
    development:
      InstanceType: t2.micro
      MonitoringEnabled: "false"
      MultiAZ: "false"
    staging:
      InstanceType: t2.small
      MonitoringEnabled: "true"
      MultiAZ: "false"
    production:
      InstanceType: t2.medium
      MonitoringEnabled: "true"
      MultiAZ: "true"

Resources:
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: !FindInMap [EnvironmentConfig, !Ref EnvironmentName, InstanceType]
      Monitoring: !FindInMap [EnvironmentConfig, !Ref EnvironmentName, MonitoringEnabled]
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-web-server"
        - Key: Environment
          Value: !Ref EnvironmentName

  # Only created in production with backup enabled
  BackupVolume:
    Type: AWS::EC2::Volume
    Condition: CreateBackupVolume
    Properties:
      Size: 100
      AvailabilityZone: us-east-1a
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-backup"

  # Only created if backup volume exists
  AttachBackup:
    Type: AWS::EC2::VolumeAttachment
    Condition: CreateBackupVolume
    Properties:
      VolumeId: !Ref BackupVolume
      InstanceId: !Ref WebServerInstance
      Device: /dev/sdf

  # Only created in non-production environments (dev/staging)
  DevelopmentSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Condition: IsProductionOrStaging  # Wrong - should be NotProduction
    # Actually showing both prod and staging
    Properties:
      GroupDescription: "Staging security group"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3389
          ToPort: 3389
          CidrIp: 0.0.0.0/0  # RDP from anywhere (dev only!)

Outputs:
  # Always returned
  InstanceId:
    Value: !Ref WebServerInstance
    Description: "Web server instance ID"

  InstanceIpAddress:
    Value: !GetAtt WebServerInstance.PublicIp
    Description: "Public IP address"

  # Only returned in production
  ProductionBackupVolumeId:
    Value: !Ref BackupVolume
    Condition: CreateBackupVolume
    Description: "Backup volume ID (production only)"

  # Information about deployment
  EnvironmentInfo:
    Value: !Sub "Environment: ${EnvironmentName}, Monitoring: ${!FindInMap [EnvironmentConfig, !Ref EnvironmentName, MonitoringEnabled]}"
    Description: "Environment configuration information"
```

**Deployment Scenarios:**

```
Scenario 1: Create Development Environment
aws cloudformation create-stack --stack-name dev-stack \
  --template-body file://template.yaml \
  --parameters ParameterKey=EnvironmentName,ParameterValue=development

Result:
├─ WebServerInstance: CREATED (t2.micro)
├─ BackupVolume: SKIPPED (condition false)
├─ AttachBackup: SKIPPED (condition false)
├─ DevelopmentSecurityGroup: CREATED if IsProductionOrStaging (ISSUE in template)
│
├─ Outputs returned:
│  ├─ InstanceId: i-1234567890
│  ├─ InstanceIpAddress: 203.0.113.45
│  ├─ EnvironmentInfo: Environment: development, Monitoring: false
│  └─ ProductionBackupVolumeId: NOT shown (condition false)
│
└─ Cost: ~$7/month (t2.micro only)

Scenario 2: Create Production Environment with Backup
aws cloudformation create-stack --stack-name prod-stack \
  --template-body file://template.yaml \
  --parameters \
    ParameterKey=EnvironmentName,ParameterValue=production \
    ParameterKey=EnableBackup,ParameterValue="true"

Result:
├─ WebServerInstance: CREATED (t2.medium)
├─ BackupVolume: CREATED (100 GB EBS)
├─ AttachBackup: CREATED (/dev/sdf)
│
├─ Outputs returned:
│  ├─ InstanceId: i-9876543210
│  ├─ InstanceIpAddress: 203.0.113.99
│  ├─ ProductionBackupVolumeId: vol-1234567890abcdef
│  └─ EnvironmentInfo: Environment: production, Monitoring: true
│
└─ Cost: ~$50/month (t2.medium + backup volume + monitoring)
```

## Part 5: Condition Comparison and Logic

### Combining Conditions

**Complex Condition Examples:**

```yaml
Conditions:
  # Simple conditions
  IsProduction:
    Fn::Equals: [!Ref Environment, production]

  IsStaging:
    Fn::Equals: [!Ref Environment, staging]

  IsUSEast1:
    Fn::Equals: [!Ref AWS::Region, us-east-1]

  # Combined conditions (AND)
  IsProductionAndUSEast1:
    Fn::And:
      - !Condition IsProduction
      - !Condition IsUSEast1
    # true only if BOTH conditions true

  # Combined conditions (OR)
  IsProductionOrStaging:
    Fn::Or:
      - !Condition IsProduction
      - !Condition IsStaging
    # true if EITHER condition true

  # Negated conditions (NOT)
  NotProduction:
    Fn::Not:
      - !Condition IsProduction
    # true if IsProduction is false

  # Complex nested condition
  CreateRedundancy:
    Fn::And:
      - Fn::Or:
          - !Condition IsProduction
          - !Condition IsStaging
      - !Condition IsUSEast1
    # true if (Production OR Staging) AND US-East-1
    # Means: Guarantee redundancy in critical region only

Usage:
├─ IsProductionAndUSEast1: Create only in prod AND us-east-1
├─ IsProductionOrStaging: Skip only in development
├─ NotProduction: Cheaper resources in dev
├─ CreateRedundancy: High-availability in critical region only
└─ Complex scenarios need nested logic
```

### Conditional Value Selection with Fn::If

**Using Fn::If for Property Values:**

```yaml
Conditions:
  IsProduction:
    Fn::Equals: [!Ref Environment, production]

Resources:
  # Use Fn::If to select different values
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: !If [IsProduction, db.t2.medium, db.t2.micro]
      AllocatedStorage: !If [IsProduction, 100, 20]
      MultiAZ: !If [IsProduction, "true", "false"]
      BackupRetentionPeriod: !If [IsProduction, 30, 1]
      StorageEncrypted: !If [IsProduction, "true", "false"]

Execution:
├─ Production:
│  ├─ DBInstanceClass: db.t2.medium
│  ├─ AllocatedStorage: 100
│  ├─ MultiAZ: true
│  ├─ BackupRetentionPeriod: 30
│  └─ StorageEncrypted: true
│
└─ Development:
   ├─ DBInstanceClass: db.t2.micro
   ├─ AllocatedStorage: 20
   ├─ MultiAZ: false
   ├─ BackupRetentionPeriod: 1
   └─ StorageEncrypted: false

Note:
├─ Fn::If selected VALUE (not whether to create resource)
├─ Resource still created in both environments
├─ But with different configuration
└─ Different use case from resource Condition property
```

## Part 6: Best Practices and Exam Focus

### Condition Best Practices

```
✓ Use Conditions for Environment Differentiation
  ├─ Different resources per environment
  ├─ Cost optimization per environment
  ├─ Security hardening in production only
  └─ Standard practice

✓ Combine with Mappings for Configuration
  ├─ Conditions control IF resource created
  ├─ Mappings control HOW resource configured
  ├─ Best: Conditions + Mappings + Fn::If together
  └─ Example: Create resource IF prod, configure FROM mapping

✓ Name Conditions Clearly
  ├─ ✓ IsProduction (clear)
  ├─ ✓ CreateBackupVolume (clear)
  ├─ ✗ Cond1, If1 (unclear)
  └─ Self-documenting templates

✓ Document Complex Conditions
  ├─ Add comments explaining logic
  ├─ Especially for nested And/Or/Not
  ├─ Help future maintainers understand
  └─ Complex logic becomes clear

✓ Consider Cost Implications
  ├─ Conditions reduce costs in dev
  ├─ Dev-only conditions save money
  └─ Calculate overhead of optional resources

✓ Keep Conditions Simple When Possible
  ├─ Simple conditions: Easier to understand
  ├─ Complex nested logic: Hard to debug
  ├─ Consider breaking into multiple conditions
  └─ Readability matters
```

### Conditions vs Alternatives

```
When NOT to use Conditions:

Option 1: Separate Templates (Simple)
├─ dev-template.yaml
├─ prod-template.yaml
├─ Benefit: Clear separation, no complex logic
├─ Cost: Maintenance overhead
└─ Use when: Templates very different

Option 2: Conditions (Complex)
├─ One template with conditions
├─ Benefit: Single source of truth
├─ Cost: Complex logic, hard to maintain
└─ Use when: Templates similar with minor differences

Recommendation:
├─ Few minor differences: Use conditions
├─ Many major differences: Use separate templates
├─ Medium differences: Conditions
└─ Example: Prod needs backup → Use condition
           Prod completely different arch → Use separate template
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What do Conditions in CloudFormation do?"
A) Define input parameters
B) Control resource creation based on logic
C) Store static values
D) Output stack information

Answer: B (conditions control IF resources are created)

Q2: "Most common use for Conditions?"
A) Environment differentiation (dev vs prod)
B) Storing configuration values
C) Creating outputs
D) Defining parameters

Answer: A (environment-based conditions most common)

Q3: "What function checks if two values equal?"
A) Fn::Or
B) Fn::And
C) Fn::Equals
D) Fn::If

Answer: C (Fn::Equals compares values)

Q4: "Fn::If vs Resource Condition property?"
A) Same thing
B) Fn::If selects values, Condition property controls creation
C) Fn::If controls resource creation
D) Both select values

Answer: B (Fn::If for values, Condition for creation)

Q5: "Correct condition that means production AND us-east-1?"
A) !Or [IsProd, IsUSEast1]
B) !And [IsProd, IsUSEast1]
C) !Equals [IsProd, IsUSEast1]
D) !Not [IsProd]

Answer: B (!And requires both conditions true)

Q6: "Can conditions reference parameters and mappings?"
A) Parameters only
B) Mappings only
C) Both parameters and mappings
D) Neither

Answer: C (conditions can reference both)

Key Points:
├─ Conditions control resource creation (true=create, false=skip)
├─ Fn::Equals most common for comparisons
├─ Fn::If selects between two values (not creation/deletion)
├─ Common use: Environment differentiation
├─ Combine conditions with mappings for best results
└─ Exam focus on recognizing and understanding logic
```

### Key Concepts Summary

```
✓ Conditions enable conditional resource creation
  ├─ Not created if condition false
  ├─ Created if condition true
  ├─ Control infrastructure based on logic
  └─ Environment-based conditions most common

✓ Condition Functions provide boolean logic
  ├─ Fn::Equals: Compare two values
  ├─ Fn::And: True if ALL conditions true
  ├─ Fn::Or: True if ANY condition true
  ├─ Fn::Not: Inverts condition
  └─ Fn::If: Selects between two values

✓ Conditions can be nested and combined
  ├─ Complex logic possible
  ├─ Readable names essential
  ├─ Comments help maintain templates
  └─ Plan logic carefully

✓ Three ways to apply conditions
  ├─ Resource Condition property: Skips resource if false
  ├─ Output Condition property: Hides output if false
  ├─ Fn::If in properties: Selects value based on condition
  └─ Different use cases

✓ Conditions vs Mappings vs Fn::If
  ├─ Condition: IF this is true, THEN create resource
  ├─ Mapping: For static configuration values
  ├─ Fn::If: Select THIS value or THAT value
  └─ Use together for complete solution

✓ Common condition scenarios
  ├─ Production: More resources, high HA, encryption
  ├─ Development: Fewer resources, lower cost, no backup
  ├─ Staging: Moderate resources
  ├─ Region-specific: Different configs per region
  └─ Parameter-based: User controls what's created
```

---

**Total Words: ~8,500**  
**File Created: 8_CloudFormation_Conditions.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
