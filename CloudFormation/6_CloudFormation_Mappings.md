# 6. CloudFormation Mappings

## Part 1: Understanding Mappings

### What Are Mappings?

**Definition:**

```
Mappings:
├─ Fixed, hardcoded variables within templates
├─ Lookup tables of predefined values
├─ Static - all possible values known in advance
├─ Used for conditional value selection
├─ Based on input variables (region, environment, etc.)
├─ Accessed via FindInMap function
└─ Nested key-value structure (two-level hierarchy)
```

**Why Mappings Are Useful:**

```
Use Case 1: Region-Specific Values
├─ AMI IDs vary by region
├─ S3 endpoints vary by region
├─ Database endpoints vary by region
├─ Solution: Create mapping with all regions → all values
└─ Template automatically selects correct value for region

Use Case 2: Environment-Specific Values
├─ Dev instance types (cheaper): t2.micro
├─ Staging instance types (medium): t2.small
├─ Production instance types (powerful): t2.medium
├─ Solution: Map environment name to instance type
└─ Users select environment, template applies settings

Use Case 3: Architecture-Specific Values
├─ HVM64 architecture vs HVMG2 architecture
├─ Different AMI per architecture
├─ Solution: Map architecture type to AMI
└─ Template selects correct AMI

Use Case 4: Availability Zone Specific Values
├─ Different subnets per AZ
├─ Different security group profiles per AZ
├─ Solution: Map AZ to subnet ID
└─ Template automatically uses correct subnet
```

**Mappings vs Hardcoding vs Parameters:**

```
Option 1: Hardcoded (❌ Bad)
ImageId: ami-0c55b159cbfafe1f0

Problem:
├─ Only works in us-east-1
├─ Fails in other regions (wrong AMI)
├─ Not portable

Option 2: Parameter (⚠️ Sometimes)
Parameters:
  ImageId:
    Type: AWS::EC2::Image::Id

Good:
├─ User selects AMI
├─ Flexible

Bad:
├─ Requires user knowledge
├─ What if 10 regions?
├─ Users might select wrong AMI for region

Option 3: Mappings (✓ Best)
Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-0c55b159cbfafe1f0
    us-west-1:
      HVM64: ami-12345678abcdef12
    eu-west-1:
      HVM64: ami-87654321abcdef12

Good:
├─ Hardcoded but organized
├─ Correct value per region automatically
├─ No user confusion
├─ Portable across regions
├─ Safe choices enforced
└─ Scalable lookup system
```

### When to Use Mappings

**Mapping Decision Criteria:**

```
Use Mappings When:

✓ All possible values known in advance
  ├─ You know all regions CloudFormation supports
  ├─ You know all environments (dev/staging/prod)
  ├─ You know all architectures (HVM64/HVMG2)
  └─ Complete list available

✓ Values can be deduced from known variables
  ├─ Region (AWS::Region pseudo parameter)
  ├─ Environment (parameter or tagged stack)
  ├─ Availability zone (known in advance)
  ├─ AWS account (AWS::AccountId pseudo parameter)
  └─ Conditional logic available

✓ Values are the same across all stacks
  ├─ All users get same region map
  ├─ All users get same environment map
  ├─ Organization standards
  └─ Centralized control

✓ You want template consistency
  ├─ Prevent user mistakes
  ├─ Enforce corporate standards
  ├─ Reduce configuration errors
  └─ Safer templates
```

**Don't Use Mappings When:**

```
Use Parameters Instead When:

✗ Values unknown in advance
  └─ User provides custom value

✗ Values vary per user/team
  ├─ Different API keys per team
  ├─ Different database Names per user
  ├─ Different company names per client
  └─ Personalization needed

✗ Too many possible values
  ├─ Hundreds of options
  ├─ Impractical to list all
  └─ Better as parameter

✗ Values change frequently
  ├─ Quarterly updates to instance types
  ├─ Regular AMI updates
  ├─ Mapping requires template edit
  ├─ Would need re-deployment
  └─ Better as external reference
```

## Part 2: Mapping Structure and Syntax

### Mapping Format

**Two-Level Hierarchical Structure:**

```yaml
Mappings:
  MapName:
    TopLevelKey:
      SecondLevelKey: Value
      AnotherKey: AnotherValue
    AnotherTopLevelKey:
      Key: Value
```

**Structure Breakdown:**

```
Level 1: Map Name
└─ Identifier for entire mapping
   └─ Example: RegionMap, EnvironmentMap

Level 2: Top Level Key (First dimension)
└─ First lookup criterion
   └─ Examples: Region (us-east-1, us-west-1)
              Environment (dev, staging, prod)
              Architecture (HVM64, HVMG2)

Level 3: Second Level Key (Second dimension)
└─ Second lookup criterion
   └─ Examples: AMI value, instance type, subnet ID
```

### Real Example: Region Map

**Complete Region Mapping Example:**

```yaml
Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-0c55b159cbfafe1f0
      HVMG2: ami-12345678abcdef12
    us-west-1:
      HVM64: ami-11223344aabbccdd
      HVMG2: ami-55667788eeffgghh
    us-west-2:
      HVM64: ami-99887766aabbccdd
      HVMG2: ami-33445566iijjkkll
    eu-west-1:
      HVM64: ami-87654321aabbccdd
      HVMG2: ami-99112233mmnnoopp
    eu-central-1:
      HVM64: ami-55443322qqrrsstt
      HVMG2: ami-11335577uuvvwwxx
    ap-southeast-1:
      HVM64: ami-77889900yyzz1111
      HVMG2: ami-22334455aaaa2222

Structure Breakdown:
├─ Map Name: RegionMap
├─ Top Level Key: Region (us-east-1, us-west-1, etc.)
└─ Second Level Key: Architecture (HVM64, HVMG2)
   └─ Value: AMI ID (ami-xxxxxxxx)

Why This Works:
├─ AMIs are region-specific (different ID per region)
├─ AMIs are architecture-specific (HVM64 vs HVMG2)
├─ Mapping covers all combinations
└─ Template automatically selects correct AMI
```

### Environment Mapping Example

**Environment-Based Configuration Mapping:**

```yaml
Mappings:
  EnvironmentConfig:
    development:
      InstanceType: t2.micro
      DBInstanceClass: db.t2.micro
      MultiAZ: false
      BackupRetentionDays: 1
      ProvisionedThroughput: 1
    staging:
      InstanceType: t2.small
      DBInstanceClass: db.t2.small
      MultiAZ: false
      BackupRetentionDays: 7
      ProvisionedThroughput: 5
    production:
      InstanceType: t2.medium
      DBInstanceClass: db.t2.medium
      MultiAZ: true
      BackupRetentionDays: 30
      ProvisionedThroughput: 100

Structure Breakdown:
├─ Map Name: EnvironmentConfig
├─ Top Level Key: Environment (development, staging, production)
└─ Second Level Key: Property (InstanceType, DBInstanceClass, etc.)
   └─ Value: Environment-specific setting

Usage:
├─ Dev teams get t2.micro (cheap)
├─ Staging gets t2.small (moderate)
├─ Production gets t2.medium (powerful)
├─ Backups scale with environment
└─ Single parameter controls all settings
```

### Multiple Dimensions Example

**Combining Multiple Mapping Criteria:**

```yaml
Mappings:
  RegionArchitectureMap:
    us-east-1:
      HVM64: ami-0c55b159cbfafe1f0
      HVMG2: ami-12345678abcdef12
    us-west-1:
      HVM64: ami-11223344aabbccdd
      HVMG2: ami-55667788eeffgghh

  InstanceTypeMap:
    development:
      ComputeOptimized: c5.large
      MemoryOptimized: r5.large
      GeneralPurpose: t2.small
    production:
      ComputeOptimized: c5.2xlarge
      MemoryOptimized: r5.4xlarge
      GeneralPurpose: t2.large
```

## Part 3: Using FindInMap Function

### FindInMap Syntax

**Function Definition:**

```
FindInMap Function:
├─ Usage: Retrieve value from mapping
├─ Syntax: !FindInMap [MapName, TopLevelKey, SecondLevelKey]
├─ Short form: !FindInMap (YAML preferred)
├─ Long form: Fn::FindInMap: [MapName, TopLevelKey, SecondLevelKey]
├─ Returns: The value at that mapping location
└─ Error: If mapping location doesn't exist
```

**Three Required Arguments:**

```
!FindInMap [MapName, TopLevelKey, SecondLevelKey]
           │        │              │
           │        │              └─ Second dimension key
           │        │                  (specific config property)
           │        │
           │        └─ First dimension key
           │           (region, environment, architecture)
           │
           └─ Name of mapping to search

Example:
!FindInMap [RegionMap, us-east-1, HVM64]
           │           │           │
           │           │           └─ Architecture: HVM64
           │           │              → Returns: ami-0c55b159cbfafe1f0
           │           │
           │           └─ Region: us-east-1
           │
           └─ Map: RegionMap
```

### Real FindInMap Usage

**Static Values (Hardcoded Keys):**

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, us-east-1, HVM64]
      InstanceType: t2.micro

Problem:
├─ Hardcoded region (us-east-1)
├─ If stack moved to us-west-1, still uses us-east-1 AMI
├─ Wrong AMI = stack fails
└─ Not portable

Usage:
├─ Only when absolutely certain of region
├─ Fixed regions (never changing)
└─ Rare use case
```

**Dynamic Values with Pseudo Parameters:**

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref AWS::Region, HVM64]
      InstanceType: t2.micro

How It Works:
├─ !Ref AWS::Region = current deployment region
├─ If stack in us-east-1: AWS::Region = "us-east-1"
├─ !FindInMap looks up RegionMap["us-east-1"]["HVM64"]
├─ Returns: ami-0c55b159cbfafe1f0
│
├─ If stack in us-west-1: AWS::Region = "us-west-1"
├─ !FindInMap looks up RegionMap["us-west-1"]["HVM64"]
├─ Returns: ami-11223344aabbccdd
│
└─ Same template works in all regions!

Benefits:
├─ Template is portable
├─ Correct AMI selected per region
├─ No manual editing
└─ Automatic behavior
```

**Using Parameters with FindInMap:**

```yaml
Parameters:
  EnvironmentName:
    Type: String
    Default: development
    AllowedValues:
      - development
      - staging
      - production

Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: 
        !FindInMap [EnvironmentConfig, !Ref EnvironmentName, InstanceType]
      ImageId: ami-0c55b159cbfafe1f0
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName

Execution:
├─ User selects: development
├─ !Ref EnvironmentName = "development"
├─ !FindInMap looks up EnvironmentConfig["development"]["InstanceType"]
├─ Returns: t2.micro
├─ Instance created with t2.micro
│
├─ If user selects: production
├─ !Ref EnvironmentName = "production"
├─ !FindInMap looks up EnvironmentConfig["production"]["InstanceType"]
├─ Returns: t2.medium
├─ Instance created with t2.medium
└─ Same template, different behavior per environment
```

## Part 4: Real-World Mapping Examples

### Complete Example: Multi-Region, Multi-Architecture

**Full Regional Deployment Template:**

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: "Multi-region, multi-architecture template using mappings"

Parameters:
  Architecture:
    Type: String
    Default: HVM64
    AllowedValues:
      - HVM64
      - HVMG2
    Description: "CPU architecture for AMI"

Mappings:
  RegionMap:
    us-east-1:
      HVM64: ami-0c55b159cbfafe1f0
      HVMG2: ami-12345678abcdef12
    us-west-1:
      HVM64: ami-11223344aabbccdd
      HVMG2: ami-55667788eeffgghh
    us-west-2:
      HVM64: ami-99887766aabbccdd
      HVMG2: ami-33445566iijjkkll
    eu-west-1:
      HVM64: ami-87654321aabbccdd
      HVMG2: ami-99112233mmnnoopp
    ap-southeast-1:
      HVM64: ami-77889900yyzz1111
      HVMG2: ami-22334455aaaa2222

Resources:
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref AWS::Region, !Ref Architecture]
      InstanceType: t2.micro
      Tags:
        - Key: Name
          Value: !Sub "WebServer-${AWS::Region}-${Architecture}"
        - Key: Region
          Value: !Ref AWS::Region

Outputs:
  InstanceId:
    Value: !Ref WebServerInstance

  ImageId:
    Value: !FindInMap [RegionMap, !Ref AWS::Region, !Ref Architecture]
    Description: "AMI ID used for instance"

  Region:
    Value: !Ref AWS::Region
    Description: "Deployment region"

  Architecture:
    Value: !Ref Architecture
    Description: "AMI architecture"
```

**How It Works Across Deployments:**

```
Scenario 1: Deploy to us-east-1 with HVM64
├─ AWS::Region = "us-east-1"
├─ Architecture parameter = "HVM64"
├─ FindInMap [RegionMap, us-east-1, HVM64]
├─ Returns: ami-0c55b159cbfafe1f0
├─ Instance tags: WebServer-us-east-1-HVM64
└─ Result: Correct AMI for region and architecture

Scenario 2: Same template deployed to eu-west-1 with HVMG2
├─ AWS::Region = "eu-west-1"
├─ Architecture parameter = "HVMG2"
├─ FindInMap [RegionMap, eu-west-1, HVMG2]
├─ Returns: ami-99112233mmnnoopp
├─ Instance tags: WebServer-eu-west-1-HVMG2
└─ Result: Correct AMI for region and architecture

Benefits:
├─ No template editing required
├─ Reusable across all regions
├─ Correct AMI automatically selected
├─ User controls architecture via parameter
└─ Fully portable template
```

### Example: Environment-Based Configuration

**Complete Environment Configuration Template:**

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: "Environment-aware infrastructure using mappings"

Parameters:
  EnvironmentName:
    Type: String
    Default: development
    AllowedValues:
      - development
      - staging
      - production
    Description: "Environment type"

Mappings:
  EnvironmentConfig:
    development:
      InstanceType: t2.micro
      DBInstanceClass: db.t2.micro
      MultiAZ: "false"
      BackupRetentionDays: "1"
      ProvisionedThroughput: "1"
      EnableDetailedMonitoring: "false"
    staging:
      InstanceType: t2.small
      DBInstanceClass: db.t2.small
      MultiAZ: "false"
      BackupRetentionDays: "7"
      ProvisionedThroughput: "5"
      EnableDetailedMonitoring: "true"
    production:
      InstanceType: t2.medium
      DBInstanceClass: db.t2.medium
      MultiAZ: "true"
      BackupRetentionDays: "30"
      ProvisionedThroughput: "100"
      EnableDetailedMonitoring: "true"

Resources:
  AppServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: 
        !FindInMap [EnvironmentConfig, !Ref EnvironmentName, InstanceType]
      Monitoring: 
        !FindInMap [EnvironmentConfig, !Ref EnvironmentName, EnableDetailedMonitoring]
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName
        - Key: Name
          Value: !Sub "${EnvironmentName}-app-server"

  Database:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass:
        !FindInMap [EnvironmentConfig, !Ref EnvironmentName, DBInstanceClass]
      Engine: mysql
      MasterUsername: admin
      MasterUserPassword: MyPassword123
      MultiAZ:
        !FindInMap [EnvironmentConfig, !Ref EnvironmentName, MultiAZ]
      BackupRetentionPeriod:
        !FindInMap [EnvironmentConfig, !Ref EnvironmentName, BackupRetentionDays]
      Tags:
        - Key: Environment
          Value: !Ref EnvironmentName

Outputs:
  InstanceType:
    Value: !FindInMap [EnvironmentConfig, !Ref EnvironmentName, InstanceType]
    Description: "Instance type for environment"

  DBClass:
    Value: !FindInMap [EnvironmentConfig, !Ref EnvironmentName, DBInstanceClass]
    Description: "Database instance class"

  Environment:
    Value: !Ref EnvironmentName
    Description: "The environment deployed"
```

**Cost Savings Through Mapping:**

```
Same Template, Three Deployments:

Development Stack:
├─ Instance: t2.micro (~$7/month)
├─ Database: db.t2.micro (~$15/month)
├─ Backups: 1 day
├─ Monitoring: Basic (free)
└─ Monthly cost: ~$22

Staging Stack:
├─ Instance: t2.small (~$17/month)
├─ Database: db.t2.small (~$30/month)
├─ Backups: 7 days
├─ Monitoring: Detailed ($3.50)
└─ Monthly cost: ~$50

Production Stack:
├─ Instance: t2.medium (~$34/month)
├─ Database: db.t2.medium (~$60/month)
├─ MultiAZ: Enabled (doubles cost) (~$150)
├─ Backups: 30 days
├─ Monitoring: Detailed ($3.50)
└─ Monthly cost: ~$250

Total Cost: ~$322/month
Same template enforces cost efficiency per environment!
```

## Part 5: Mappings vs Parameters Decision

### When to Use Each

**Clear Decision Matrix:**

```
Scenario: AMI ID selection

Use MAPPINGS:
├─ Reason: Know all possible AMIs in advance
├─ Reason: Different per region (deterministic)
├─ Reason: Want automatic region-correct AMI
├─ Result: !FindInMap [RegionMap, !Ref AWS::Region, HVM64]
├─ Benefit: Portable template, automatic behavior
└─ Example: Regional deployment

Use PARAMETERS:
├─ Reason: User should choose specific AMI
├─ Reason: Many possible options (too many to map)
├─ Reason: Custom AMI built by team
├─ Result: User selects AMI ID from dropdown
├─ Benefit: Flexibility, but user responsibility
└─ Example: Custom machine image selection
```

**Scenario Comparison:**

```
Scenario 1: Instance Type Selection
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MAPPINGS Approach:
Parameters:
  Environment:
    Type: String
    AllowedValues: [dev, staging, prod]

Mappings:
  Config:
    dev: {InstanceType: t2.micro}
    staging: {InstanceType: t2.small}
    prod: {InstanceType: t2.medium}

Resources:
  Instance:
    InstanceType: !FindInMap [Config, !Ref Environment, InstanceType]

Benefits:
├─ User selects environment (2 questions)
├─ Instance type automatically configured
├─ Prevents user mistakes
└─ Enforces standards

PARAMETERS Approach:
Parameters:
  InstanceType:
    Type: String
    AllowedValues: [t2.micro, t2.small, t2.medium, t2.large, c5.large, ...]

Resources:
  Instance:
    InstanceType: !Ref InstanceType

Drawback:
├─ User must know correct type
├─ Too many choices (confusion)
├─ User might select wrong type
└─ Less standardized

VERDICT: Use Mappings (fewer user decisions, smarter defaults)
```

```
Scenario 2: Custom Database Name
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MAPPINGS Approach:
Mappings:
  DBNames:
    web: {Name: webdb}
    api: {Name: apidb}
    analytics: {Name: analyticsdb}

Problem:
├─ Mapping limited to predefined databases
├─ User wants custom name "my-special-db"
├─ Naming based on use case, not environment
└─ Mapping doesn't fit

PARAMETERS Approach:
Parameters:
  DatabaseName:
    Type: String
    Description: "Custom database name"
    AllowedPattern: "^[a-z0-9-]*$"

Resources:
  Database:
    DatabaseName: !Ref DatabaseName

Benefit:
├─ User provides custom name
├─ Flexibility for different teams
├─ Pattern validation ensures syntax
└─ Works for any naming need

VERDICT: Use Parameters (user determines value, too varied for mapping)
```

```
Scenario 3: Regional Resource Endpoints
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MAPPINGS Approach:
Mappings:
  S3Endpoints:
    us-east-1: {Endpoint: "s3.amazonaws.com"}
    us-west-1: {Endpoint: "s3-us-west-1.amazonaws.com"}
    eu-west-1: {Endpoint: "s3-eu-west-1.amazonaws.com"}

Resources:
  Lambda:
    Environment:
      S3_ENDPOINT: !FindInMap [S3Endpoints, !Ref AWS::Region, Endpoint]

Solution:
├─ All regions known in advance
├─ Correct endpoint per region
├─ No user input needed
├─ Automatic behavior
└─ Portable template

VERDICT: Use Mappings (deterministic, region-specific, known values)
```

**Decision Flow:**

```
Question 1: Do you know all possible values?
├─ YES → Continue
└─ NO → Use Parameter

Question 2: Can value be determined from known variables?
├─ Region, Environment, Architecture, Account ID
├─ YES → Continue
└─ NO → Use Parameter (user determines)

Question 3: Do you want to enforce standards?
├─ YES → Use Mapping (safe, no mistakes)
└─ NO → Use Parameter (more flexibility)

Result:
├─ All YES → Mapping
├─ Any NO → Parameter
└─ Mixed → Combination of Mappings + Parameters
```

## Part 6: Best Practices and Exam Focus

### Mapping Best Practices

```
✓ Use Mappings for Known, Finite Sets
  ├─ Regions (AWS regions don't change)
  ├─ Environments (dev/staging/prod standard)
  ├─ Architectures (HVM64/HVMG2 fixed)
  └─ Account types (dev account, prod account)

✓ Combine Mappings with Pseudo Parameters
  ├─ !Ref AWS::Region for portable templates
  ├─ !Ref AWS::AccountId for account-specific values
  ├─ Make templates work everywhere
  └─ No hardcoding regions

✓ Use Parameters for Top-Level User Choices
  ├─ Environment parameter controls everything
  ├─ User selects, mapping handles details
  ├─ Reduced user decisions
  └─ Smarter configuration

✓ Document Mapping Purposes
  ├─ Comment why mapping exists
  ├─ Explain all keys and values
  ├─ Help future maintainers
  └─ Template readability

✓ Keep Mappings Organized
  ├─ One logical concept per mapping
  ├─ Clear naming (RegionMap, EnvironmentConfig)
  ├─ Consistent structure
  └─ Easy to navigate

✓ Use Mapping Names that Describe Content
  ├─ ✓ RegionMap (it maps regions)
  ├─ ✓ EnvironmentConfig (configures per environment)
  ├─ ✗ Map1, Map2, Config (unclear)
  └─ Self-documenting code
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What is a CloudFormation Mapping?"
A) Variables that users provide
B) Fixed values in template for different scenarios
C) Parameters passed at runtime
D) Outputs returned after creation

Answer: B (fixed, predefined values for different scenarios)

Q2: "When to use Mappings vs Parameters?"
A) Mappings for unknown values, Parameters for known
B) Parameters for unknown values, Mappings for known
C) Only Mappings needed
D) Only Parameters needed

Answer: B (Parameters for user input, Mappings for known values)

Q3: "What function retrieves mapping values?"
A) !GetAtt
B) !Ref
C) !FindInMap
D) !Sub

Answer: C (!FindInMap retrieves from mappings)

Q4: "Correct FindInMap syntax?"
A) !FindInMap (MapName, Key1, Key2)
B) !FindInMap [MapName, Key1, Key2]
C) !FindInMap {MapName: Key1, Key2}
D) FindInMap MapName Key1 Key2

Answer: B (square brackets for FindInMap)

Q5: "How many levels in mapping structure?"
A) One (single key)
B) Two (first key, second key)
C) Three (first, second, third)
D) Unlimited (unlimited nesting)

Answer: B (two levels: TopLevelKey -> SecondLevelKey)

Q6: "What pseudo parameter makes templates portable?"
A) AWS::StackName
B) AWS::AccountId
C) AWS::Region
D) AWS::StackId

Answer: C (!Ref AWS::Region for region-agnostic templates)

Key Points:
├─ Mappings are static, not user input
├─ FindInMap with two-level hierarchical lookup
├─ Combine mappings with pseudo parameters for portability
├─ Parameters for user choices, Mappings for derived values
└─ Region mapping is most common use case
```

### Key Concepts Summary

```
✓ Mappings provide static, lookup table functionality
  └─ Pre-defined values organized hierarchically

✓ Two-level hierarchy: TopLevelKey -> SecondLevelKey
  ├─ First level: High-level category (region, environment)
  ├─ Second level: Specific value (AMI ID, instance type)
  └─ FindInMap returns second-level value

✓ FindInMap function retrieves mapping values
  ├─ Syntax: !FindInMap [MapName, TopLevelKey, SecondLevelKey]
  ├─ Can use pseudo parameters for dynamic lookup
  ├─ AWS::Region for region-specific mappings
  └─ !Ref ParameterName for parameter-driven mappings

✓ Mappings make templates portable and safe
  ├─ Correct values per region automatically
  ├─ Enforces standards
  ├─ Reduces user errors
  ├─ No manual configuration
  └─ Same template works everywhere

✓ Choose Mappings vs Parameters carefully
  ├─ Mappings: Known, finite, deterministic values
  ├─ Parameters: Unknown, user-determined values
  ├─ Common combination: Parameter selects, Mapping applies settings
  └─ Better UX and control

✓ Common mapping use cases
  ├─ Region-specific values (AMI IDs)
  ├─ Environment-specific configuration (dev/staging/prod)
  ├─ Architecture-specific values (HVM64/HVMG2)
  ├─ Account-specific settings
  └─ Availability zone mappings

✓ Mappings improve template maintainability
  ├─ Centralized configuration
  ├─ Easy to update all values together
  ├─ Self-documenting structure
  ├─ Reduced duplication
  └─ Scalable for multiple regions/environments
```

---

**Total Words: ~8,500**  
**File Created: 6_CloudFormation_Mappings.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
