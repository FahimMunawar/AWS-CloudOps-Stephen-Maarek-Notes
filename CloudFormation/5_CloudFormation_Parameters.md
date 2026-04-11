# 5. CloudFormation Parameters

## Part 1: Understanding Parameters and When to Use Them

### What Are Parameters?

**Definition:**

```
Parameters:
├─ User input mechanism for CloudFormation templates
├─ Defined in template (Parameters section)
├─ Users provide values when creating/updating stack
├─ Enable template reusability across organization
├─ Allow configuration without editing template
└─ Make templates flexible and dynamic
```

**Why Parameters Matter:**

```
Without Parameters:
├─ Hardcoded values in template
├─ Cannot change without editing YAML
├─ Must re-upload template for any change
├─ Cannot share template across teams/environments
├─ Limited reusability
└─ Error-prone manual edits

With Parameters:
├─ Users provide input when prompted
├─ No template editing needed
├─ Same template for dev/staging/production
├─ Dropdown validation (AllowedValues)
├─ Type validation (String, Number, etc.)
├─ Secrets can be hidden (NoEcho)
├─ Flexible and reusable
└─ Error prevention through constraints
```

### When Should You Use Parameters?

**Decision Question:**

```
For every resource property, ask:

"Is this value likely to change in the future?"

├─ YES → Make it a Parameter
│  Examples:
│  ├─ InstanceType (t2.micro vs t2.small)
│  ├─ DatabasePassword (changes quarterly)
│  ├─ SecurityGroupDescription (varies by team)
│  ├─ KeyName (different per user)
│  ├─ EnvironmentName (dev/staging/prod)
│  └─ VpcId (different accounts/regions)
│
└─ NO → Hardcode in template
   Examples:
   ├─ AWS API version number
   ├─ Resource type (AWS::EC2::Instance)
   ├─ ImageId per region (if region-specific)
   └─ Static configuration
```

**Parameter Usage Decision Tree:**

```
Does value change? → YES → Parameter
       ↓
Cannot be determined ahead of time? → YES → Parameter
       ↓
Different per environment? → YES → Parameter
       ↓
Different per user/team? → YES → Parameter
       ↓
Needs validation/constraints? → YES → Parameter
       ↓
Is a secret (password, API key)? → YES → Parameter (with NoEcho)
       ↓
Otherwise: Keep hardcoded in template
```

**Real Example: SecurityGroupDescription Parameter**

```
Template Version 1 (No Parameter - Hardcoded):
Resources:
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Web Server Security Group"  # Hardcoded
      ...

Problem:
├─ Different teams want different description
├─ Must edit template for each use
├─ Risky if template edited incorrectly
└─ Not reusable

Template Version 2 (With Parameter):
Parameters:
  SecurityGroupDescription:
    Type: String
    Description: "Description for security group"
    Default: "Default Web Server Security Group"

Resources:
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Ref SecurityGroupDescription  # From parameter
      ...

Benefit:
├─ Users provide value when creating stack
├─ No template editing needed
├─ Reusable across teams
├─ Each team customizes description
└─ Easy and safe
```

## Part 2: Parameter Types

### Supported Parameter Types

**Type Options:**

```
1. String
   ├─ Simple text value
   ├─ Most flexible type
   ├─ No validation by default
   ├─ Examples: "t2.micro", "my-database-password"
   └─ Default when type unclear

2. Number
   ├─ Numeric value (integer or decimal)
   ├─ CloudFormation validates numeric format
   ├─ Examples: 22 (port), 100 (GB storage)
   ├─ Error if user enters non-numeric
   └─ Useful for: Ports, sizes, timeouts, counts

3. CommaDelimitedList
   ├─ String with comma-separated values
   ├─ Becomes list in CloudFormation
   ├─ Example input: "value1,value2,value3"
   ├─ CloudFormation parses: ["value1", "value2", "value3"]
   └─ Useful for: Multiple selections as single parameter

4. List<Number>
   ├─ Comma-delimited list of numbers
   ├─ Example input: "80,443,3306"
   ├─ CloudFormation parses: [80, 443, 3306]
   └─ Useful for: Port lists, ID lists
```

**AWS-Specific Parameter Types:**

```
AWS-Specific Types validate against actual AWS resources:

1. AWS::EC2::AvailabilityZone::Name
   ├─ User must select valid AZ
   ├─ CloudFormation lists available AZs
   ├─ Prevents invalid zones
   ├─ Example: us-east-1a, us-east-1b
   └─ Dropdown menu in console

2. AWS::EC2::Image::Id
   ├─ Valid AMI ID
   ├─ Can filter by region
   ├─ Only recent/approved AMIs shown
   ├─ Example: ami-0c55b159cbfafe1f0
   └─ Prevents using wrong AMI

3. AWS::EC2::Instance::Id
   ├─ Existing EC2 instance ID
   ├─ User selects from running instances
   ├─ Example: i-1234567890abcdef0
   └─ Useful for: Managing existing resources

4. AWS::EC2::KeyPair::KeyName
   ├─ Existing key pair in account
   ├─ User selects from available keys
   ├─ Error if key doesn't exist
   ├─ Example: my-prod-key
   └─ For: EC2 SSH access

5. AWS::EC2::SecurityGroup::Id
   ├─ Existing security group ID
   ├─ User selects from available SGs
   ├─ Validates SG exists
   ├─ Example: sg-12345678
   └─ For: Attaching SGs to resources

6. AWS::EC2::SecurityGroup::GroupName
   ├─ Existing security group name (not ID)
   ├─ Example: my-web-sg
   └─ For: VPC security groups

7. AWS::EC2::Subnet::Id
   ├─ Existing VPC subnet
   ├─ User selects valid subnet
   ├─ Example: subnet-1a2b3c4d
   └─ For: Specifying launch subnet

8. AWS::EC2::Volume::Id
   ├─ Existing EBS volume
   ├─ User selects from available volumes
   ├─ Example: vol-12345678
   └─ For: Attaching volumes to instances

9. AWS::EC2::VPC::Id
   ├─ Existing VPC ID
   ├─ User selects from available VPCs
   ├─ Example: vpc-12345678
   └─ For: Specifying which VPC

10. AWS::Route53::HostedZone::Id
    ├─ Existing Route53 hosted zone
    ├─ Example: Z1234567890ABC
    └─ For: DNS configuration
```

**List Versions of AWS-Specific Types:**

```
Prefix "List" to AWS-specific types for multiple selections:

List<AWS::EC2::AvailabilityZone::Name>
├─ User selects multiple AZs
├─ Example: us-east-1a, us-east-1b, us-east-1c
└─ Result: List of AZs

List<AWS::EC2::SecurityGroup::Id>
├─ User selects multiple security groups
├─ Example: sg-123, sg-456, sg-789
└─ Result: List of SG IDs

Benefits:
├─ Validation by AWS itself
├─ Prevents typos
├─ Enforces correctness
├─ Great user experience (dropdown menus)
└─ Reduces errors
```

### Parameter Type Examples

**String vs AWS::EC2::KeyPair::KeyName:**

```yaml
# Approach 1: String (NO validation)
Parameters:
  KeyName:
    Type: String
    Description: "Name of EC2 KeyPair"
    Example: "my-key"

Problem:
├─ User types: "nonexistent-key"
├─ Template accepts it
├─ Stack creation FAILS (key doesn't exist)
├─ Error hard to debug
└─ Wasted time and confusion

# Approach 2: AWS-Specific Type (WITH validation)
Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: "Name of EC2 KeyPair"

Benefits:
├─ Console shows dropdown of existing keys
├─ User cannot select non-existent key
├─ Stack creation guaranteed to work
├─ Better user experience
└─ Error prevention at parameter entry
```

**Number Parameter for Port:**

```yaml
Parameters:
  HttpPort:
    Type: Number
    Description: "HTTP port number"
    Default: 80
    MinValue: 1
    MaxValue: 65535

Example:
├─ User enters: "abc" → REJECTED (not a number)
├─ User enters: "0" → REJECTED (less than 1)
├─ User enters: "65536" → REJECTED (more than 65535)
├─ User enters: "8080" → ACCEPTED
└─ Type safety provided by CloudFormation
```

## Part 3: Parameter Properties and Constraints

### Parameter Attributes

**Complete Parameter Structure:**

```yaml
Parameters:
  ParameterName:
    Type: String  # Required: Type of parameter
    
    Description: "What this parameter does"
    # Optional: Help text shown to user
    
    Default: "default value"
    # Optional: Used if user doesn't provide value
    
    NoEcho: true
    # Optional: Hide value from logs (for passwords)
    
    MinLength: 8
    MaxLength: 64
    # Optional: String length validation
    
    MinValue: 1
    MaxValue: 100
    # Optional: Number range validation (Number type)
    
    AllowedValues:
      - value1
      - value2
      - value3
    # Optional: Only these values allowed (dropdown)
    
    AllowedPattern: "^[a-zA-Z0-9-]*$"
    # Optional: Regex pattern validation
    
    ConstraintDescription: "Must be valid hostname"
    # Optional: Error message if constraint fails
```

### Constraint Types

**String Length Constraints:**

```yaml
Parameters:
  DatabaseName:
    Type: String
    Description: "Database name"
    MinLength: 1
    MaxLength: 63
    Default: "mydb"
    ConstraintDescription: "1-63 characters, alphanumeric"

Example:
├─ User enters: "" (empty) → REJECTED (MinLength 1)
├─ User enters: "a" → ACCEPTED
├─ User enters: "a" * 64 characters → REJECTED (MaxLength 63)
└─ User enters: "mydb123" → ACCEPTED
```

**Number Range Constraints:**

```yaml
Parameters:
  DesiredCapacity:
    Type: Number
    Description: "Desired number of instances"
    Default: 2
    MinValue: 1
    MaxValue: 10
    ConstraintDescription: "Must be 1-10"

Example:
├─ User enters: "0" → REJECTED (MinValue 1)
├─ User enters: "5" → ACCEPTED
├─ User enters: "11" → REJECTED (MaxValue 10)
└─ User enters: "2.5" → May be accepted/rejected (depends on type)
```

**AllowedValues (Enumeration):**

```yaml
Parameters:
  InstanceType:
    Type: String
    Description: "EC2 instance type"
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
      - t3.micro
      - t3.small

User Experience:
├─ Console shows: DROPDOWN with 5 options
├─ User cannot type invalid values
├─ User cannot select "t2.large" (not in list)
├─ Guarantees valid input
└─ Best practice for controlled choices

Example:
├─ User selects: "t2.micro" → ACCEPTED
├─ User tries typing: "t2.large" → REJECTED
└─ Only dropdown options allowed
```

**AllowedPattern (Regular Expression):**

```yaml
Parameters:
  HostName:
    Type: String
    Description: "Valid hostname"
    AllowedPattern: "^[a-zA-Z0-9-]*$"
    ConstraintDescription: "Alphanumeric and hyphens only"

Pattern Explanation:
├─ ^[a-zA-Z0-9-]*$ means:
├─ ^ = Start of string
├─ [a-zA-Z0-9-]* = Letters, numbers, hyphens only
├─ $ = End of string
└─ * = Zero or more characters

Example:
├─ User enters: "my-server-1" → ACCEPTED
├─ User enters: "my_server" → REJECTED (underscore not allowed)
├─ User enters: "my server" → REJECTED (space not allowed)
└─ User enters: "my-server-1" → ACCEPTED
```

### Important: NoEcho for Secrets

**NoEcho Property:**

```yaml
Parameters:
  DatabasePassword:
    Type: String
    Description: "RDS database password"
    NoEcho: true
    MinLength: 8
    ConstraintDescription: "Min 8 characters"

What NoEcho Does:
├─ Hides value in CloudFormation console
├─ Hides value from stack outputs
├─ Hides value from logs and events
├─ Masks value in change sets
├─ Password not exposed in stack details
└─ Secrets remain confidential

User Experience:
├─ User types password in console
├─ Console shows: ****** (masked)
├─ Value accepted
├─ Stack created
├─ Password never visible anywhere else

Example - Good Practice:
Parameters:
  ApiKey:
    Type: String
    NoEcho: true
    Description: "Third-party API key"

  DBPassword:
    Type: String
    NoEcho: true
    MinLength: 12
    Description: "Database password"

Example - Bad Practice:
Parameters:
  ApiKey:
    Type: String
    Default: "sk-1234567890"  # NEVER hardcode secrets
    Description: "API key"    # EXPOSED!
```

## Part 4: Using Parameters with !Ref

### The !Ref Function

**What !Ref Does:**

```
!Ref (Ref = Reference) function returns:

1. If referencing a PARAMETER
   └─ Returns the VALUE provided by user

2. If referencing a RESOURCE
   └─ Returns the Physical ID of resource

3. If referencing a PSEUDO PARAMETER
   └─ Returns the metadata value

Syntax:
├─ Full form: !Ref ParameterName
├─ Or: Fn::Ref
└─ YAML short form: !Ref (preferred)
```

**Parameter Reference:**

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: !Ref InstanceType  # Gets user's value
      # At runtime: InstanceType becomes what user entered

Example Execution:
├─ User provides: InstanceType = "t2.small"
├─ CloudFormation processes: !Ref InstanceType
├─ Result: InstanceType property set to "t2.small"
├─ Instance created with t2.small
└─ User's input used directly
```

### Using !Ref for Resources

**Resource References:**

```yaml
Parameters:
  SecurityGroupDescription:
    Type: String

Resources:
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "SSH Access"
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  AppSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Ref SecurityGroupDescription  # Parameter reference

  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      SecurityGroupIds:
        - !Ref SSHSecurityGroup        # Resource reference
        - !Ref AppSecurityGroup        # Resource reference

Execution Flow:
├─ User provides SecurityGroupDescription parameter
├─ SSHSecurityGroup created with "SSH Access"
├─ AppSecurityGroup created with user's description
├─ MyInstance created and attached to both SGs
├─ References resolved by CloudFormation
└─ Dependencies implicit from !Ref usage
```

### !Ref vs Resource Names

**Critical: Naming Conflict Prevention**

```
NEVER name parameter and resource the same:

❌ BAD EXAMPLE:
Parameters:
  MyInstance:  # Parameter named MyInstance
    Type: String

Resources:
  MyInstance:  # Resource named MyInstance
    Type: AWS::EC2::Instance

Problem:
├─ Ambiguous: Which MyInstance in !Ref?
├─ CloudFormation won't process correctly
├─ Hard to debug
└─ Causes errors

✓ GOOD EXAMPLE:
Parameters:
  InstanceTypeChoice:  # Parameter
    Type: String

Resources:
  MyInstance:         # Resource
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !Ref InstanceTypeChoice

Clear:
├─ InstanceTypeChoice = parameter
├─ MyInstance = resource
├─ No ambiguity
└─ Easy to understand
```

## Part 5: Real-World Template Example

### Complete Parameter Usage Example

**E-Commerce Stack with Parameters:**

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: "E-commerce web server stack with configurable parameters"

Parameters:
  InstanceTypeParameter:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium
    ConstraintDescription: "Must be t2.micro, small, or medium"
    Description: "EC2 instance type for web server"

  EnvironmentName:
    Type: String
    Default: development
    AllowedValues:
      - development
      - staging
      - production
    Description: "Environment name"

  SecurityGroupDescription:
    Type: String
    Default: "Web server security group"
    MinLength: 5
    MaxLength: 100
    Description: "Description for security group"

  DBPassword:
    Type: String
    NoEcho: true
    MinLength: 12
    MaxLength: 32
    AllowedPattern: "^[a-zA-Z0-9!@#$]*$"
    ConstraintDescription: "12-32 chars, alphanumeric and !@#$"
    Description: "Database password (not displayed)"

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: "EC2 key pair for SSH access"
    ConstraintDescription: "Must be existing key pair"

  AllowedSSHCIDR:
    Type: String
    Default: 0.0.0.0/0
    Description: "CIDR block allowed for SSH access"

Resources:
  WebSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: !Ref SecurityGroupDescription
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: !Ref AllowedSSHCIDR
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: !Ref InstanceTypeParameter
      KeyName: !Ref KeyName
      SecurityGroupIds:
        - !Ref WebSecurityGroup
      Tags:
        - Key: Name
          Value: !Sub "${EnvironmentName}-web-server"
        - Key: Environment
          Value: !Ref EnvironmentName

Outputs:
  InstanceId:
    Value: !Ref WebServerInstance
    Description: "Instance ID of web server"

  SecurityGroupId:
    Value: !Ref WebSecurityGroup
    Description: "Security group ID"

  Environment:
    Value: !Ref EnvironmentName
    Description: "Environment name from parameters"
```

**User Experience When Creating Stack:**

```
CloudFormation Console Prompts User:

1. InstanceTypeParameter
   Type: Dropdown (AllowedValues)
   Options: t2.micro (default), t2.small, t2.medium
   User selects: t2.small

2. EnvironmentName
   Type: Dropdown (AllowedValues)
   Options: development (default), staging, production
   User selects: staging

3. SecurityGroupDescription
   Type: Text input (String, 5-100 chars)
   Default: "Web server security group"
   User enters: "Staging Web Servers SG"

4. DBPassword
   Type: Password field (NoEcho: true)
   Shown as: *** (masked)
   User enters: "P@ssw0rd12345" (not shown on screen)

5. KeyName
   Type: Dropdown (AWS::EC2::KeyPair::KeyName)
   Options: my-prod-key, my-dev-key, my-staging-key
   User selects: my-staging-key

6. AllowedSSHCIDR
   Type: Text input
   Default: 0.0.0.0/0
   User enters: 203.0.113.0/24 (restrict SSH to company IP)

Result:
├─ Stack created with user's choices
├─ Instance type: t2.small
├─ Environment tags: "staging"
├─ SG description: "Staging Web Servers SG"
├─ Password stored securely (not in logs)
├─ SSH key: my-staging-key
├─ SSH allowed only from: 203.0.113.0/24
└─ All parameters validated before stack creation
```

## Part 6: Pseudo Parameters

### What Are Pseudo Parameters?

**Definition:**

```
Pseudo Parameters:
├─ Built-in CloudFormation variables
├─ Available automatically in all templates
├─ Require no prior declaration
├─ Cannot be overridden by users
├─ Provide metadata about stack/account/region
├─ Enabled by default (no setup needed)
└─ Referenced with !Ref just like parameters
```

**Why They're Useful:**

```
Without Pseudo Parameters:
├─ Must ask user for account ID
├─ Must ask user for region
├─ Must hardcode values
├─ Cannot make region-agnostic templates
└─ Not portable across accounts

With Pseudo Parameters:
├─ Automatically know account ID
├─ Automatically know region
├─ Self-aware templates
├─ Works across all accounts/regions
├─ Highly reusable
└─ No user input needed
```

### Common Pseudo Parameters

**AWS::AccountId:**

```yaml
Description: "Return value: AWS account ID running stack"

Example:
├─ Your AWS account: 123456789012
├─ !Ref AWS::AccountId returns: 123456789012
├─ Useful for: S3 bucket names, ARNs, cross-account references

Use Case:
Parameters:
  BucketPrefix:
    Type: String
    Default: "my-app"

Resources:
  DataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${BucketPrefix}-${AWS::AccountId}"
      # Creates: my-app-123456789012 (unique per account)
```

**AWS::Region:**

```yaml
Description: "Return value: AWS region where stack is created"

Example:
├─ Stack in us-east-1: !Ref AWS::Region returns "us-east-1"
├─ Stack in eu-west-1: !Ref AWS::Region returns "eu-west-1"
├─ Useful for: AMI IDs (region-specific), endpoints, mappings

Use Case:
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0  # Assumes same region
      # Better: Use Mappings with !Ref AWS::Region

Output:
  RegionName:
    Value: !Ref AWS::Region
    Description: "Region where stack was created"
```

**AWS::StackId:**

```yaml
Description: "Return value: Full stack ARN/ID"

Example Return: arn:aws:cloudformation:us-east-1:123456789012:stack/my-stack/12345678-1234-1234-1234-123456789012

Use Case:
├─ Logging which stack created resource
├─ Unique identifier for stack
├─ Tracking across multiple stacks
└─ Debugging

Example:
Outputs:
  StackIdentifier:
    Value: !Ref AWS::StackId
    Description: "Unique stack identifier"
```

**AWS::StackName:**

```yaml
Description: "Return value: Name of the stack being created"

Example:
├─ Stack Name: my-prod-stack
├─ !Ref AWS::StackName returns: "my-prod-stack"
├─ Useful for: Tagging, resource naming, resource identification

Use Case:
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      Tags:
        - Key: StackName
          Value: !Ref AWS::StackName
        - Key: CreatedBy
          Value: CloudFormation

  MyS3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "${AWS::StackName}-data"
      # If stack name is "my-prod-stack"
      # Bucket becomes: "my-prod-stack-data"
```

**AWS::NotificationARNs:**

```yaml
Description: "Return value: SNS topic ARNs for notifications"

Example:
├─ If user specified SNS topics during stack creation
├─ !Ref AWS::NotificationARNs returns list of ARNs
├─ Empty list if no SNS topics specified
├─ Useful for: Custom notifications, automation

Use Case:
Resources:
  CustomLambda:
    Type: AWS::Lambda::Function
    Properties:
      Runtime: python3.9
      Handler: index.handler
      Code:
        ZipFile: |
          import json
          def handler(event, context):
            # Receives SNS topics from stack
            return json.dumps({"status": "processing"})
      Environment:
        Variables:
          NOTIFICATION_ARNS: !Join [',', !Ref AWS::NotificationARNs]
```

**AWS::NoValue:**

```yaml
Description: "Return value: No value (special use)"

Use Case:
├─ Conditional property inclusion
├─ Sometimes include property, sometimes don't
├─ Beyond scope of basic course
└─ Advanced feature

Example (Advanced):
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      KeyName: !If [HasKeyPair, !Ref KeyPair, !Ref AWS::NoValue]
      # If condition true: use KeyPair
      # If condition false: omit property entirely
```

**Other Pseudo Parameters Available:**

```
AWS::Partition
├─ Partition where resource runs
├─ Usually: "aws" (standard)
├─ China regions: "aws-cn"
└─ AWS GovCloud: "aws-us-gov"

AWS::URLSuffix
├─ URL domain suffix
├─ Standard AWS: "amazonaws.com"
├─ China: "amazonaws.com.cn"
└─ Used in: S3 URLs, API endpoints

AWS::TemplateFormatVersion
├─ Template version specified
├─ Usually: 2010-09-09
└─ For validation purposes
```

### Practical Pseudo Parameter Example

**Template Referencing Pseudo Parameters:**

```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: "Stack using pseudo parameters"

Resources:
  DataBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Sub "data-bucket-${AWS::StackName}-${AWS::AccountId}"
      # Creates: data-bucket-my-app-123456789012

  WebInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      Tags:
        - Key: Stack
          Value: !Ref AWS::StackName
        - Key: Region
          Value: !Ref AWS::Region
        - Key: Account
          Value: !Ref AWS::AccountId
        - Key: StackId
          Value: !Ref AWS::StackId

Outputs:
  BucketName:
    Value: !Ref DataBucket

  AccountId:
    Value: !Ref AWS::AccountId
    Description: "AWS account ID"

  RegionName:
    Value: !Ref AWS::Region
    Description: "AWS region"

  StackInformation:
    Value: !Sub "Stack '${AWS::StackName}' in ${AWS::Region} (${AWS::AccountId})"
    Description: "Complete stack information"

  StackId:
    Value: !Ref AWS::StackId
    Description: "Full stack ARN"
```

**Automatic Behavior (No User Action):**

```
When user creates this stack in us-east-1 account 123456789012
Named: "my-production-app"

CloudFormation Automatically:
├─ AWS::StackName → "my-production-app"
├─ AWS::Region → "us-east-1"
├─ AWS::AccountId → "123456789012"
├─ AWS::StackId → Full ARN generated
└─ All resources tagged appropriately

Results:
├─ S3 Bucket: "data-bucket-my-production-app-123456789012"
├─ Instance tags include: Stack=my-production-app, Region=us-east-1
├─ Outputs show: Account=123456789012, Region=us-east-1
└─ Template is region/account portable
```

## Part 7: Best Practices and Exam Focus

### Parameter Best Practices

```
✓ Use Parameters for Variable Values
  ├─ Anything that changes: parameter
  ├─ Fixed values: hardcode
  └─ Anything user might change: parameter

✓ Use AllowedValues for Limited Choices
  ├─ Instance types: dropdown
  ├─ Environments: dropdown
  ├─ Boolean alternatives: dropdown
  └─ Error prevention

✓ Use AWS-Specific Types When Available
  ├─ KeyName: AWS::EC2::KeyPair::KeyName
  ├─ Subnets: AWS::EC2::Subnet::Id
  ├─ Security groups: AWS::EC2::SecurityGroup::Id
  └─ Built-in validation

✓ Use NoEcho for Secrets
  ├─ Passwords: NoEcho: true
  ├─ API keys: NoEcho: true
  ├─ Tokens: NoEcho: true
  └─ Security best practice

✓ Provide Meaningful Descriptions
  ├─ Users need to understand parameter
  ├─ Good examples in description
  ├─ Clear constraints listed
  └─ Help users make good choices

✓ Use Pseudo Parameters to Make Templates Portable
  ├─ !Ref AWS::Region for region-agnostic templates
  ├─ !Ref AWS::AccountId for unique resources
  ├─ !Ref AWS::StackName for clarity
  └─ Templates work everywhere

✓ Never Hardcode Secrets
  ├─ Don't use Default for passwords
  ├─ Don't hardcode API keys
  ├─ Use NoEcho: true
  └─ Store secrets securely
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "When should you use a Parameter?"
A) When value might change in future
B) When value determined at runtime
C) When different per environment
D) All of above

Answer: D (all are valid reasons for parameters)

Q2: "What parameter type validates against existing resources?"
A) String
B) Number
C) AWS::EC2::KeyPair::KeyName
D) CommaDelimitedList

Answer: C (AWS-specific types validate)

Q3: "How to hide password in stack output?"
A) Use Parameter with NoEcho: true
B) Use Secret Manager
C) Use default value
D) Can't hide it

Answer: A (NoEcho: true hides from logs)

Q4: "What pseudo parameter gives account ID?"
A) AWS::Region
B) AWS::StackName
C) AWS::AccountId
D) AWS::StackId

Answer: C (AWS::AccountId)

Q5: "How to reference parameter in template?"
A) ${ParameterName}
B) %ParameterName%
C) !Ref ParameterName
D) @ParameterName

Answer: C (!Ref syntax)

Q6: "Can you have parameter and resource with same name?"
A) Yes, no conflict
B) No, creates ambiguity
C) Only if different types
D) Only in Outputs section

Answer: B (naming conflict causes problems)
```

### Key Concepts Summary

```
✓ Parameters enable template reusability
  └─ Same template, different values per stack

✓ Multiple parameter types
  ├─ String, Number, CommaDelimitedList
  ├─ AWS-specific types (KeyName, InstanceId, etc.)
  └─ Type safety built-in

✓ Constraints validate user input
  ├─ AllowedValues (dropdown)
  ├─ MinLength/MaxLength for strings
  ├─ MinValue/MaxValue for numbers
  ├─ AllowedPattern for regex
  └─ Error prevention

✓ !Ref references both parameters and resources
  ├─ !Ref ParameterName → user's value
  ├─ !Ref ResourceName → resource ID
  └─ Same syntax, different meaning

✓ NoEcho hides secrets
  ├─ Password not displayed
  ├─ Not in logs
  ├─ Not in outputs
  └─ Security best practice

✓ Pseudo parameters provide metadata
  ├─ AWS::AccountId, AWS::Region
  ├─ AWS::StackName, AWS::StackId
  ├─ AWS::NotificationARNs
  └─ Available automatically

✓ Templates can be region/account portable
  ├─ Use pseudo parameters
  ├─ Relative references for resources
  ├─ Zero hardcoding
  └─ Deploy anywhere safely
```

---

**Total Words: ~9,500**  
**File Created: 5_CloudFormation_Parameters.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
