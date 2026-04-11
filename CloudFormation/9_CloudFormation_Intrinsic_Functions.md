# 9. CloudFormation Intrinsic Functions

## Part 1: Overview of Intrinsic Functions

### What Are Intrinsic Functions?

**Definition:**

```
Intrinsic Functions:
├─ Built-in CloudFormation functions
├─ Used in templates to perform operations
├─ Manipulate values, references, conditions
├─ Available automatically (no setup needed)
├─ Called throughout template sections
└─ Essential for dynamic template behavior
```

**Complete List of Intrinsic Functions:**

```
CRITICAL (Blue - Must Know for SysOps Exam):
├─ !Ref (Reference parameter or resource)
├─ !GetAtt (Get resource attribute)
├─ !FindInMap (Lookup mapping value)
├─ !ImportValue (Import exported stack value)
├─ !Join (Concatenate strings with delimiter)
├─ !Sub (String substitution/interpolation)
└─ Condition functions (!If, !Equals, !And, !Or, !Not)

IMPORTANT (Secondary):
├─ !Base64 (Encode string to Base64)
├─ !Cidr (Generate CIDR blocks)
├─ !GetAZs (Get availability zones)
├─ !Select (Select item from list)
├─ !Split (Split string into list)
└─ !Transform (Use macros)

ADVANCED (Rarely Used):
├─ !Length (Get list length)
├─ !ToJsonString (Convert to JSON)
└─ !ForEach (Iterate over values)
```

**Function Syntax:**

```
YAML Short Form (Preferred):
├─ !Ref ParameterName
├─ !GetAtt ResourceName.AttributeName
├─ !FindInMap [MapName, Key1, Key2]
├─ !Sub "String with ${Variable}"
└─ Most readable, most commonly used

YAML Long Form:
├─ Ref: ParameterName
├─ Fn::GetAtt: [ResourceName, AttributeName]
├─ Fn::FindInMap: [MapName, Key1, Key2]
├─ Fn::Sub: "String with ${Variable}"
└─ Less common, harder to read

JSON Form:
├─ { "Ref": "ParameterName" }
├─ { "Fn::GetAtt": ["ResourceName", "AttributeName"] }
└─ Used in JSON templates
```

## Part 2: !Ref Function (Most Important)

### Ref Function Definition

**What !Ref Does:**

```
!Ref can reference:

1. Parameters
   ├─ Returns the parameter value provided by user
   ├─ Example: !Ref Environment returns "production"
   └─ Used for: User input integration

2. Resources
   ├─ Returns the physical ID of resource
   ├─ EC2: Returns instance ID (i-1234567890)
   ├─ S3: Returns bucket name
   ├─ RDS: Returns database ID
   └─ Used for: Cross-resource references

3. Pseudo Parameters
   ├─ Special built-in parameters
   ├─ Example: !Ref AWS::Region, !Ref AWS::AccountId
   └─ Used for: Automatic metadata
```

### Ref with Parameters

**Parameter Reference Example:**

```yaml
Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: "EC2 key pair for SSH"

  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small
      - t2.medium

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      KeyName: !Ref KeyName              # Returns user's selected key
      InstanceType: !Ref InstanceType    # Returns user's selected type

Execution:
├─ User selects: KeyName = "my-prod-key"
├─ !Ref KeyName → "my-prod-key"
├─ User selects: InstanceType = "t2.small"
├─ !Ref InstanceType → "t2.small"
└─ Instance created with user selections
```

### Ref with Resources

**Resource Reference Example:**

```yaml
Resources:
  # First, create VPC
  CompanyVpc:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16

  # Then create subnet inside VPC
  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref CompanyVpc        # Reference to VPC resource
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: us-east-1a

  # Then create instance in subnet
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      SubnetId: !Ref PublicSubnet    # Reference to subnet resource
      Tags:
        - Key: VPC-ID
          Value: !Ref CompanyVpc     # Output VPC ID for reference

Execution:
├─ CompanyVpc created: Physical ID = vpc-12345678
├─ !Ref CompanyVpc → vpc-12345678
├─ PublicSubnet created with VpcId = vpc-12345678
├─ SubnetId for subnet created = subnet-87654321
├─ !Ref PublicSubnet → subnet-87654321
├─ WebServer created with SubnetId = subnet-87654321
└─ All resources properly linked through Ref
```

### Ref Return Values by Resource Type

**Common Ref Return Values:**

```
Resource Type: EC2 Instance
└─ !Ref MyInstance → Instance ID (i-1234567890abcdef0)

Resource Type: S3 Bucket
└─ !Ref MyBucket → Bucket name (my-bucket-name)

Resource Type: RDS Database
└─ !Ref MyDatabase → Database identifier (mydb)

Resource Type: Security Group
└─ !Ref MySecurityGroup → Security group ID (sg-12345678)

Resource Type: Subnet
└─ !Ref MySubnet → Subnet ID (subnet-87654321)

Resource Type: VPC
└─ !Ref MyVpc → VPC ID (vpc-12345678)

Resource Type: IAM Role
└─ !Ref MyRole → Role ARN

Resource Type: Lambda Function
└─ !Ref MyFunction → Function ARN

Note:
├─ Different resource types return different identifiers
├─ Always check documentation for exact return value
├─ Most resources return ID or ARN
└─ Used in Outputs to show created resources
```

## Part 3: !GetAtt Function (Get Attributes)

### GetAtt Function Definition

**What !GetAtt Does:**

```
!GetAtt retrieves resource attributes:

├─ Syntax: !GetAtt ResourceName.AttributeName
├─ Returns specific attribute of created resource
├─ Different from !Ref (which returns ID)
├─ Resource must support the attribute
└─ Documented in CloudFormation resource reference
```

**!Ref vs !GetAtt Comparison:**

```
Resource: EC2 Instance

!Ref MyInstance:
├─ Returns: Instance ID (i-1234567890abcdef0)
├─ Physical identifier
└─ Used for: Resource linkage

!GetAtt MyInstance.PublicIp:
├─ Returns: Public IP address (203.0.113.45)
├─ Specific attribute
└─ Used for: User information

!GetAtt MyInstance.PrivateIp:
├─ Returns: Private IP (10.0.1.100)
└─ Used for: Internal references

!GetAtt MyInstance.AvailabilityZone:
├─ Returns: AZ where instance runs (us-east-1a)
└─ Used for: Zone-specific resources

!GetAtt MyInstance.PublicDnsName:
├─ Returns: Public DNS name
└─ Used for: Accessing instance by name
```

### Common GetAtt Attributes

**EC2 Instance Attributes:**

```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro

Outputs:
  InstanceId:
    Value: !Ref MyInstance
    Description: "Instance ID from Ref"

  AvailabilityZone:
    Value: !GetAtt MyInstance.AvailabilityZone
    Description: "AZ where instance runs"

  PublicIP:
    Value: !GetAtt MyInstance.PublicIp
    Description: "Public IP address"

  PrivateIP:
    Value: !GetAtt MyInstance.PrivateIp
    Description: "Private IP address"

  PublicDNS:
    Value: !GetAtt MyInstance.PublicDnsName
    Description: "Public DNS name"

  PrivateDNS:
    Value: !GetAtt MyInstance.PrivateDnsName
    Description: "Private DNS name"
```

**RDS Database Attributes:**

```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t2.micro
      Engine: mysql
      MasterUsername: admin

Outputs:
  # !Ref returns: Database identifier
  DatabaseId:
    Value: !Ref MyDatabase

  # !GetAtt returns: Connection endpoint
  DatabaseEndpoint:
    Value: !GetAtt MyDatabase.Endpoint.Address
    Description: "Database hostname for connection"

  DatabasePort:
    Value: !GetAtt MyDatabase.Endpoint.Port
    Description: "Database port number"

Connection String Example:
server=!GetAtt MyDatabase.Endpoint.Address
port=!GetAtt MyDatabase.Endpoint.Port
database=mydb
user=admin
```

### Real GetAtt Use Case: Availability Zone

**Creating Resources with GetAtt:**

```yaml
Resources:
  # Create EC2 instance
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      AvailabilityZone: us-east-1a

  # Create EBS volume in SAME AZ as instance
  DataVolume:
    Type: AWS::EC2::Volume
    Properties:
      Size: 100
      # Get the AZ from WebServer instance
      AvailabilityZone: !GetAtt WebServer.AvailabilityZone
      Tags:
        - Key: Name
          Value: data-volume

  # Attach volume to instance
  AttachVolume:
    Type: AWS::EC2::VolumeAttachment
    Properties:
      VolumeId: !Ref DataVolume
      InstanceId: !Ref WebServer
      Device: /dev/sdf

Benefits:
├─ No need to hardcode AZ
├─ Volume guaranteed to be in same AZ
├─ Portable template across regions
└─ Automatic zone selection
```

## Part 4: Other Critical Functions

### !FindInMap (Already Covered)

**Quick Reference:**

```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-0c55b159cbfafe1f0
    us-west-1:
      AMI: ami-11223344aabbccdd

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref AWS::Region, AMI]
      # Automatically selects correct AMI for region
```

### !ImportValue (Already Covered)

**Quick Reference:**

```yaml
# From another CloudFormation stack
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !ImportValue NetworkStack-PublicSubnet
      # Imports exported value from NetworkStack
```

### !Sub (String Substitution)

**What !Sub Does:**

```
!Sub performs string interpolation:
├─ Syntax: !Sub "String with ${Variable}"
├─ Replaces ${Variable} with actual value
├─ Variables can be: Parameters, Resources, Attributes
└─ Most useful for constructing strings
```

**Sub Examples:**

```yaml
Parameters:
  ProjectName:
    Type: String
    Default: "myapp"

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      Tags:
        - Key: Name
          # !Sub combines ProjectName with static text
          Value: !Sub "${ProjectName}-web-server"
          # Result: "myapp-web-server"

Outputs:
  InstanceInfo:
    Value: !Sub |
      Instance ID: ${WebServer}
      Region: ${AWS::Region}
      Account: ${AWS::AccountId}
    # Creates multi-line string with substitutions

  StackInfo:
    Value: !Sub "Stack ${AWS::StackName} in ${AWS::Region}"
    # Result: "Stack my-stack-name in us-east-1"
```

**Sub with Attributes:**

```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t2.micro
      Engine: mysql

  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      # Use Sub to pass database connection string
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          mysql -h ${MyDatabase.Endpoint.Address} -u admin -p password
          # Substitutes actual database endpoint
```

### !Join (String Concatenation)

**What !Join Does:**

```
!Join concatenates strings with delimiter:
├─ Syntax: !Join [Delimiter, [String1, String2, ...]]
├─ Delimiter: Character to put between strings
├─ Returns: Concatenated string
└─ Useful for: Creating lists, paths, etc.
```

**Join Examples:**

```yaml
Resources:
  # Create list of ports
  SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: "Web server SG"
      SecurityGroupIngress:
        - IpProtocol: tcp
          # Join port numbers
          FromPort: !Join ["", [8, 0]]  # Result: "80"
          ToPort: !Join ["", [8, 0]]    # Result: "80"
          CidrIp: 0.0.0.0/0

Outputs:
  # Create S3 bucket path
  BucketPath:
    Value: !Join ["/", ["s3://", !Ref MyBucket, "data"]]
    # Result: "s3://bucket-name/data"

  # Create list of subnets
  SubnetsInfo:
    Value: !Join [", ", [!Ref Subnet1, !Ref Subnet2, !Ref Subnet3]]
    # Result: "subnet-111, subnet-222, subnet-333"
```

### !Base64 (Encoding)

**What !Base64 Does:**

```
!Base64 encodes string to Base64:
├─ Syntax: !Base64 "String to encode"
├─ Returns: Base64-encoded version
├─ Used for: EC2 user data scripts
└─ CloudFormation automatically decodes for EC2
```

**Base64 Examples:**

```yaml
Resources:
  WebServer:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      InstanceType: t2.micro
      # UserData must be Base64 encoded
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          echo "Server name: ${AWS::StackName}" > /tmp/info.txt
          yum update -y
          yum install -y httpd
          systemctl start httpd
          # !Base64 automatically encodes this script
          # EC2 receives encoded data, decodes, and executes

Mechanism:
├─ Full script provided to !Base64
├─ !Base64 encodes entire script
├─ Encoded data sent to EC2
├─ EC2 automatically decodes
├─ Script executed at launch
└─ Result: Server fully configured at startup
```

## Part 5: Condition Functions

### Condition Functions Overview

**Condition-Related Intrinsic Functions:**

```
1. !If [Condition, ValueIfTrue, ValueIfFalse]
   ├─ Selects value based on condition
   ├─ Not for resource creation (that's Condition property)
   └─ Used in: Resource properties

2. !Equals [Value1, Value2]
   ├─ Compares two values
   ├─ Returns true/false
   └─ Used in: Conditions section

3. !And [Condition1, Condition2, ...]
   ├─ True if all conditions true
   └─ Used in: Conditions section

4. !Or [Condition1, Condition2, ...]
   ├─ True if any condition true
   └─ Used in: Conditions section

5. !Not [Condition]
   ├─ Inverts condition
   └─ Used in: Conditions section
```

### Condition Functions Examples

**Using Conditions Together:**

```yaml
Conditions:
  IsProduction:
    !Equals [!Ref Environment, production]

  IsStaging:
    !Equals [!Ref Environment, staging]

  IsProductionOrStaging:
    !Or
      - !Condition IsProduction
      - !Condition IsStaging

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-0c55b159cbfafe1f0
      # Use !If to select instance type
      InstanceType: !If [IsProduction, t2.medium, t2.micro]
      # If production: t2.medium
      # If not production: t2.micro
      
      # Use condition to control creation
      Condition: IsProductionOrStaging
      # Resource only created if Prod OR Staging
```

## Part 6: Lesser-Used Functions

### !GetAZs (Get Availability Zones)

**Purpose:**

```
!GetAZs returns list of AZs in region:
├─ Syntax: !GetAZs Region
├─ Region: AWS region name or empty for current
├─ Returns: List of AZ names (us-east-1a, us-east-1b, ...)
└─ Used for: Automatic AZ discovery
```

**Example:**

```yaml
Resources:
  Subnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVpc
      CidrBlock: 10.0.1.0/24
      # Get first AZ automatically
      AvailabilityZone: !Select [0, !GetAZs ""]
      # !GetAZs "" returns all AZs in current region
      # !Select [0, ...] picks first one

  Subnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVpc
      CidrBlock: 10.0.2.0/24
      # Get second AZ automatically
      AvailabilityZone: !Select [1, !GetAZs ""]
```

### !Select (List Selection)

**Purpose:**

```
!Select picks item from list:
├─ Syntax: !Select [Index, [Item1, Item2, Item3, ...]]
├─ Index: 0-based position (0 = first item)
├─ Returns: Selected item
└─ Used for: Picking specific items from lists
```

**Example:**

```yaml
# Select from GetAZs result
!Select [0, !GetAZs ""]  # First AZ: us-east-1a
!Select [1, !GetAZs ""]  # Second AZ: us-east-1b

# Select from user-provided list
Parameters:
  Subnets:
    Type: List<AWS::EC2::Subnet::Id>

Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      SubnetId: !Select [0, !Ref Subnets]
      # Uses first subnet from user's list
```

### !Split (String Splitting)

**Purpose:**

```
!Split creates list by splitting string:
├─ Syntax: !Split [Delimiter, String]
├─ Delimiter: Character to split on
├─ Returns: List of split parts
└─ Used for: Parsing delimited strings
```

**Example:**

```yaml
# Split CSV list into array
Outputs:
  SubnetsList:
    Value: !Join [", ", !Split [",", "subnet-1,subnet-2,subnet-3"]]
    # Gets subnets from comma-separated values
```

## Part 7: Best Practices and Exam Focus

### Function Best Practices

```
✓ Use !Ref for parameters and resources
  ├─ Simplest function
  ├─ Most commonly used
  ├─ Returns ID or parameter value
  └─ Always first choice for references

✓ Use !GetAtt for resource attributes
  ├─ When you need specific information
  ├─ Not just the ID
  ├─ Example: IP address, DNS name, endpoint
  └─ Check documentation for available attributes

✓ Use !Sub for string interpolation
  ├─ Building names, paths, tags
  ├─ More readable than !Join
  ├─ Supports multi-line strings
  └─ Preferred over !Join for most cases

✓ Use !FindInMap for lookups
  ├─ When values are in mappings
  ├─ Combine with !Ref AWS::Region for portability
  └─ Pre-defined values only

✓ Use !ImportValue for cross-stack
  ├─ References exported values
  ├─ Links stacks together
  ├─ Same region only
  └─ Creates dependencies

✓ Understand difference between functions
  ├─ !Ref: Get ID/value
  ├─ !GetAtt: Get attribute
  ├─ !Sub: String replacement
  ├─ !FindInMap: Mapping lookup
  └─ !ImportValue: Cross-stack reference
```

### SysOps Exam Focus

```
Likely Exam Questions:

Q1: "What does !Ref do for a resource?"
A) Gets a specific attribute
B) Returns physical ID of resource
C) Looks up value in mapping
D) Imports from another stack

Answer: B (!Ref returns physical ID)

Q2: "How to get EC2 instance's public IP?"
A) !Ref MyInstance
B) !GetAtt MyInstance.PublicIp
C) !FindInMap [Instances, MyInstance]
D) !ImportValue MyInstance.PublicIp

Answer: B (!GetAtt for specific attributes)

Q3: "Syntax for !Sub function?"
A) !Sub [Variable, Value]
B) !Sub "String with ${Variable}"
C) !Sub Variable from Value
D) !Sub { key: Value }

Answer: B (correct !Sub syntax)

Q4: "What does !FindInMap do?"
A) Maps parameters to values
B) Imports from other stacks
C) Looks up value in Mappings section
D) Maps attributes to resources

Answer: C (!FindInMap retrieves from Mappings)

Q5: "!Ref for pseudo parameter AWS::Region returns:"
A) Region name (us-east-1)
B) Region ID
C) All available regions
D) Region count

Answer: A (!Ref AWS::Region returns region name)

Key Points:
├─ !Ref for IDs and parameter values
├─ !GetAtt for resource attributes
├─ !Sub for string interpolation
├─ !FindInMap for mapping lookups
├─ !ImportValue for cross-stack references
└─ Condition functions (!If, !Equals, etc.)
```

### Key Concepts Summary

```
✓ Intrinsic functions enable dynamic templates
  ├─ Reference parameters and resources
  ├─ Extract attributes from resources
  ├─ Perform logic and string operations
  ├─ Create conditions
  └─ No manual hardcoding

✓ !Ref returns identifiers
  ├─ Parameter values
  ├─ Resource physical IDs
  ├─ Pseudo parameter values
  └─ Most frequently used

✓ !GetAtt returns resource attributes
  ├─ Specific information about resource
  ├─ IP addresses, DNS names, endpoints
  ├─ Zone information, environment details
  └─ Check docs for available attributes

✓ !Sub performs string substitution
  ├─ Replace variables in strings
  ├─ ${Variable} syntax
  ├─ More readable than concatenation
  └─ Supports multi-line strings

✓ Mapping and cross-stack functions
  ├─ !FindInMap for template-based lookups
  ├─ !ImportValue for external lookups
  ├─ Different scope and purpose
  └─ Choose based on value source

✓ Condition functions enable logic
  ├─ !If for conditional values
  ├─ !Equals, !And, !Or, !Not for logic
  ├─ Control resource creation
  └─ Declarative infrastructure

✓ Foundation of dynamic templates
  ├─ Functions make templates portable
  ├─ Reusable across regions, accounts
  ├─ Self-configuring infrastructure
  └─ Powerful infrastructure as code
```

---

**Total Words: ~9,500**  
**File Created: 9_CloudFormation_Intrinsic_Functions.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
