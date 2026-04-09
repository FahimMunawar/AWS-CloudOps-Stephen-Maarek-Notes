# EC2 Image Builder

This note covers EC2 Image Builder, a fully managed service that automates the creation, maintenance, validation, and testing of AMIs and container images. It's increasingly appearing on the SysOps exam.

## Overview

**EC2 Image Builder Purpose**
- **Automation**: Automates entire AMI creation workflow
- **Consistency**: Ensures uniform images across deployments
- **Validation**: Automated testing of created images
- **Distribution**: Multi-region image deployment
- **Maintenance**: Scheduled updates and patches
- **Cost**: Free service (pay only for resources used)

**Key Benefits**
- Eliminates manual AMI creation
- Reduces human error and inconsistency
- Continuous image updates through scheduling
- Built-in testing and validation
- Multi-region distribution capability
- Security scanning and compliance checks

## 1. EC2 Image Builder Workflow

### Complete Workflow Process

```
1. Build Phase
   └─ Create Builder EC2 instance
   └─ Install components and customize
   └─ Create AMI from Builder instance

2. Test Phase
   └─ Create Test EC2 instance from AMI
   └─ Run validation tests
   └─ Verify security and functionality

3. Distribution Phase
   └─ Distribute AMI to specified regions
   └─ Make available for production use
   └─ Terminate temporary instances
```

### Detailed Steps

#### Phase 1: Build
1. **Trigger Event**: Manual, scheduled, or on package update
2. **Infrastructure Creation**: EC2 Image Builder provisions Builder EC2 instance
3. **Component Installation**: Defined components installed
   - Install Java, Python, or other runtimes
   - Update OS packages and security patches
   - Install firewalls and security tools
   - Install custom applications
   - Configure system settings
4. **Builder Instance State**: Fully customized with all software
5. **AMI Creation**: 
   - Create snapshot of Builder instance volume
   - Register as new AMI
   - Apply metadata and tags
6. **Builder Cleanup**: Builder EC2 instance terminated to reduce costs

#### Phase 2: Test
1. **Test Instance Provisioning**: EC2 Image Builder creates Test EC2 instance from newly created AMI
2. **Test Suite Execution**: Predefined tests run automatically
   - Application functionality tests
   - Security scanning
   - Performance validation
   - System configuration verification
3. **Test Types**:
   - **Functional Tests**: Verify applications run correctly
   - **Security Tests**: Scan for vulnerabilities
   - **Compliance Tests**: Check compliance standards
   - **Custom Tests**: User-defined validation scripts
4. **Test Results**:
   - Pass: AMI approved for distribution
   - Fail: AMI marked as failed, issue investigation needed
5. **Test Instance Cleanup**: Terminated after tests complete

#### Phase 3: Distribution
1. **Region Selection**: Specify target regions for AMI distribution
2. **AMI Copy**: AMI replicated to selected regions
   - Can copy to same account regions
   - Can distribute to other AWS accounts
   - Can copy with encryption changes
3. **Availability**: AMI available in all specified regions
4. **Launch Ready**: Instances can be launched in any distribution region
5. **Storage Costs**: Incurred in each region where AMI resides

## 2. Key Components

### Recipe (Configuration)
- **Definition**: Specifies what software and components to install
- **Components**: List of pre-built or custom components
- **Base Image**: Starting point (Amazon Linux, Ubuntu, Windows, etc.)
- **Version Control**: Recipe can be versioned
- **Reusability**: Recipes can be used multiple times

**Recipe Components Structure**
```yaml
Recipe:
  - Name: MyWebServerRecipe
  - Base Image: amzn2-ami-hvm-*
  - Components:
    - Apache Web Server
    - PHP/Python Runtime
    - Custom Application Component
    - Security Hardening Component
  - Output: AMI for web deployment
```

### Components
- **Pre-built Components**: AWS maintains library of common components
  - Package managers (yum, apt)
  - Web servers (Apache, Nginx)
  - Languages (Java, Python, Node.js)
  - Monitoring agents
  - Security tools

- **Custom Components**: User-defined components
  - Custom installation scripts
  - Application-specific configurations
  - Organization standards and policies
  - Compliance requirements

### Infrastructure Configuration
- **Instance Type**: EC2 instance type for Builder instance
  - Default: t2.micro or t3.micro (cost-effective)
  - Can be increased for faster builds
- **Security Group**: Network configuration
- **VPC/Subnet**: Network placement
- **IAM Role**: Permissions for Image Builder service

### Distribution Settings
- **Target Regions**: AWS regions for AMI distribution
- **Target Accounts**: AWS accounts to share with
- **Encryption**: KMS keys for encrypted copy
- **Launch Permissions**: Public/private/account-specific

### Testing Configuration
- **Test Components**: Predefined test suites
- **Infrastructure Config**: Test instance specifications
- **Timeout Settings**: Maximum test execution time
- **Skip Tests**: Option to skip testing entirely

## 3. Scheduling and Execution

### Execution Triggers

#### Manual Execution
- **Initiation**: User manually starts pipeline
- **Use Case**: One-time builds, testing new recipes
- **Control**: Full user control over timing

#### Scheduled Execution
- **Cron Schedule**: Define frequency
  - Hourly: `0 * * * *`
  - Daily: `0 2 * * *` (2 AM UTC)
  - Weekly: `0 2 ? * SUN` (Sunday 2 AM)
- **Use Case**: Regular security updates, patch management
- **Frequency**: Flexible scheduling from hourly to yearly

#### Event-Based Execution
- **Trigger Type**: On package or component update
- **AWS Systems Manager**: Integration with package updates
- **Automatic Rebuild**: Triggered when source components updated
- **Use Case**: Continuous image updates, security patches
- **Zero Manual Effort**: Automatic response to updates

### Scheduling Best Practices
- **Off-Peak Times**: Schedule builds during low-usage windows
- **Maintenance Windows**: Align with organization maintenance periods
- **Testing Duration**: Allow sufficient time for full pipeline
- **Frequency**: Balance freshness vs. build costs

## 4. Multi-Region Distribution

### Distribution Architecture
```
Source Region (Build)
├─ Builder EC2 instance
├─ AMI created
├─ AMI tested
└─ AMI distributed to:
   ├─ us-east-1
   ├─ eu-west-1
   ├─ ap-southeast-1
   └─ Other target regions
```

### Distribution Workflow
1. **Recipe Definition**: Specify target regions
2. **Build and Test**: Occur in source region
3. **Copy Process**: AMI copied to each target region
   - Automated by Image Builder
   - Can take several minutes depending on size
4. **Regional Availability**: AMI available in all regions
5. **Independent Lifecycle**: Each regional AMI can be managed separately

### Multi-Region Benefits
- **Global Deployment**: Launch instances anywhere
- **Disaster Recovery**: Backup capabilities across regions
- **Local Compliance**: Store images in compliant regions
- **Low Latency**: Images available locally
- **Failover Ready**: Quick launch in alternate regions

### Cost Implications
- **EBS Snapshot Storage**: Charged in each region
- **More Regions = More Cost**: Cumulative storage charges
- **Alternative**: Create once, copy as needed vs. distribute all

## 5. Pricing and Cost Management

### What You Pay For

#### EC2 Instances
- **Builder Instance**: Running during build phase
  - Typically short-lived (minutes to hours)
  - Instance type affects cost
- **Test Instance**: Running during test phase
  - Similar duration as build
- **No Charges**: For Image Builder service itself

#### Storage Costs
- **EBS Snapshots**: Stored in each region
  - Primary region: Always charged
  - Additional regions: Charged per region
  - Size determines monthly cost
- **AMI Registration**: Free, but snapshots cost money

#### Data Transfer
- **Cross-Region Copy**: 
  - Data transfer charges apply
  - Bandwidth varies by region pair
  - Included in AWS data transfer pricing
- **To Internet**: Charged if accessing outside AWS

### Cost Optimization Strategies
- **Selective Regions**: Only distribute to necessary regions
- **Cleanup Old AMIs**: Regular deregistration removes snapshot costs
- **Smaller Base Images**: Reduces storage charges
- **Efficient Components**: Minimize additional software
- **Compression**: Images are compressed during storage
- **Scheduled Builds**: Build during off-peak to save on compute

### Cost Calculation Example
```
Scenario: Weekly AMI build, 3 regions, 20 GB base image

Weekly Cost:
- Builder t2.micro (15 min): $0.015
- Test t2.micro (10 min): $0.010
- EBS Snapshots (3 regions × 20 GB): ~$4.50/month
- Data transfer (regional copy): ~$0.50

Monthly Cost: ~$20-30 (depending on actual resource usage)
```

## 6. Security and Compliance Features

### Built-in Security Scanning
- **Vulnerability Detection**: Scans created images
- **Compliance Validation**: Checks against standards
- **Security Best Practices**: Applies AWS recommendations
- **Results**: Reports included in build output

### Compliance Components
- **CIS Benchmarks**: Center for Internet Security standards
- **HIPAA**: Healthcare compliance checks
- **PCI DSS**: Payment card standards
- **FedRAMP**: Federal compliance
- **Custom Policies**: Organization-specific requirements

### Security Enhancements
- **Encrypt Images**: Optional encryption during distribution
- **IAM Integration**: Role-based access control
- **Encryption Keys**: Support for customer-managed CMKs
- **Audit Logging**: CloudTrail integration for compliance

## 7. Practical Use Cases

### Web Server Fleet
```
Recipe: WebServer
├─ Base: Amazon Linux 2
├─ Components:
│  ├─ Apache/Nginx
│  ├─ PHP/Python/Node.js
│  ├─ Monitoring agent
│  └─ Log aggregation
├─ Schedule: Weekly
├─ Regions: us-east-1, us-west-2, eu-west-1
└─ Benefit: Consistent web servers globally
```

### Security Hardened AMI
```
Recipe: HardenedBase
├─ Base: Amazon Linux 2
├─ Components:
│  ├─ Security patches (auto-update)
│  ├─ Firewall configuration
│  ├─ Audit logging
│  ├─ Intrusion detection
│  └─ Compliance scanning
├─ Schedule: Daily or on package update
├─ Distribution: All corporate regions
└─ Benefit: Continuous security baseline
```

### Development Environment
```
Recipe: DevEnvironment
├─ Base: Ubuntu 20.04
├─ Components:
│  ├─ Docker
│  ├─ Git and version control
│  ├─ Development tools (GCC, Make)
│  ├─ IDEs and editors
│  └─ Test frameworks
├─ Schedule: Manual trigger
├─ Distribution: Shared AWS account region
└─ Benefit: Reproducible dev environment
```

### Compliance-Required AMI
```
Recipe: ComplianceImage
├─ Base: CIS-hardened Base
├─ Components:
│  ├─ Compliance monitoring
│  ├─ Audit logging
│  ├─ Encryption at rest
│  ├─ Certificate installation
│  └─ FIPS validation
├─ Schedule: Triggered by compliance tool
├─ Distribution: Compliance-required regions
└─ Benefit: Always compliant images
```

## 8. Comparison with Alternatives

| Method | Automation | Testing | Updates | Cost | Complexity |
|--------|-----------|---------|---------|------|------------|
| **Manual AMI** | None | Manual | Manual | Low | Low |
| **EC2 Image Builder** | Full | Automated | Scheduled | Low-Medium | High |
| **Packer** | Full (3rd party) | External | Manual | Low | High |
| **CloudFormation** | Partial | Manual | Manual | Low-Medium | Medium |
| **AWS Systems Manager** | Partial | Manual | Manual | Medium | Medium |

**When to Use EC2 Image Builder**
- Frequent AMI updates needed
- Multi-region deployment required
- Automated testing essential
- Compliance requirements exist
- AWS-native solution preferred

## 9. Best Practices

### Recipe Development
- **Version Control**: Track recipe changes
- **Component Testing**: Test components individually before combining
- **Documentation**: Document each component's purpose
- **Reusability**: Design components for multiple recipes

### Scheduling Strategy
- **Frequency**: Balance freshness vs. build cost
- **Timing**: Schedule during maintenance windows
- **Monitoring**: Alert on build failures
- **Retention**: Define old AMI retention policy

### Distribution Strategy
- **Region Selection**: Only distribute where needed
- **Account Sharing**: Use efficiently across accounts
- **Cleanup**: Regular deregistration of old AMIs
- **Tagging**: Consistent tagging for Easy identification

### Testing Configuration
- **Comprehensive Tests**: Test critical functionality
- **Security Scanning**: Always enable security tests
- **Performance Validation**: Include performance checks
- **Custom Tests**: Add organization-specific validations

### Cost Management
- **Monitor Costs**: Track snapshot storage charges
- **Optimize Sizes**: Minimize image sizes
- **Cleanup Schedule**: Regular old AMI removal
- **Region Strategy**: Distribute only to needed regions

## 10. Key Takeaways for SysOps Associate

### Core Concepts
- **Service Purpose**: Automates AMI creation, testing, and distribution
- **Fully Managed**: AWS manages entire process
- **Three Phases**: Build → Test → Distribute
- **Free Service**: Only pay for resources (EC2, storage)

### Build Process
- **Builder Instance**: Temporary EC2 instance for customization
- **Recipe**: Defines what software to install
- **Components**: Pre-built or custom installation packages
- **Automated**: No manual steps required

### Testing Phase
- **Automatic Test Instance**: Created from AMI
- **Predefined Tests**: Security, functionality, compliance
- **Pass/Fail**: Only approved AMIs distributed
- **Optional**: Can skip testing if desired

### Distribution
- **Multi-Region**: Single command distributes to multiple regions
- **Account Sharing**: Can share with other AWS accounts
- **Encryption**: Support for KMS encryption during distribution
- **Global Ready**: Makes applications truly global

### Scheduling
- **Manual**: Trigger on demand
- **Scheduled**: Cron-based regular builds
- **Event-Based**: Trigger on package updates
- **Flexibility**: Choose appropriate trigger for use case

### Exam Focus Points
- **Automation**: Eliminates manual image creation
- **Workflow**: Build → Test → Distribute
- **Scheduling**: Supports cron and event-based triggers
- **Cost**: Free service, pay for resources
- **Testing**: Built-in validation before distribution
- **Distribution**: Multi-region capability
- **Use Cases**: Security baselines, compliance images, global deployment