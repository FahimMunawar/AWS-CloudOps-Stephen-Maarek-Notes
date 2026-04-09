# AMI Overview

This note covers Amazon Machine Images (AMIs), which are the foundational templates that power EC2 instances. AMIs contain the software configuration, operating system, and settings needed to launch EC2 instances.

## Overview

**What is an AMI?**
- **AMI**: Amazon Machine Image
- **Purpose**: Template containing software configuration for EC2 instances
- **Components**: Operating system, application software, monitoring tools, custom configurations
- **Benefits**: Faster boot times, pre-configured environments, consistent deployments

## 1. AMI Types

### AWS-Provided AMIs (Public AMIs)
- **Source**: Created and maintained by AWS
- **Examples**: Amazon Linux 2, Ubuntu, Windows Server, Red Hat Enterprise Linux
- **Cost**: Free to use
- **Updates**: Regularly updated by AWS
- **Use Case**: Quick starts, development, testing

### Custom AMIs
- **Source**: Created by users/AWS customers
- **Process**: Build from existing EC2 instances
- **Maintenance**: User responsibility
- **Benefits**: Pre-installed software, custom configurations, faster launches
- **Use Case**: Production environments, standardized deployments

### AWS Marketplace AMIs
- **Source**: Created by third-party vendors
- **Business Model**: Can be free or paid
- **Examples**: Specialized software, security tools, development stacks
- **Verification**: AWS-reviewed for security and functionality
- **Use Case**: Commercial software, pre-configured solutions

## 2. AMI Components

### Operating System
- Base OS (Linux distributions, Windows)
- Kernel version and patches
- System libraries and dependencies

### Software Configuration
- Pre-installed applications
- Custom scripts and configurations
- Monitoring and logging tools
- Security settings and policies

### Instance Configuration
- Default user accounts
- Network settings
- Storage configurations
- Boot parameters

### Metadata
- AMI ID and name
- Architecture (32-bit/64-bit)
- Virtualization type (HVM/PV)
- Root device type (EBS/Instance Store)

## 3. AMI Lifecycle

### Creating a Custom AMI

#### Step 1: Launch Base Instance
- Start with AWS-provided or Marketplace AMI
- Choose appropriate instance type and configuration

#### Step 2: Customize Instance
- Install required software and applications
- Configure system settings and security
- Set up monitoring and logging
- Test configurations

#### Step 3: Prepare for AMI Creation
- **Stop Instance**: Ensures data integrity
- Clean up temporary files and logs
- Remove sensitive data (passwords, keys)
- Verify instance is in desired state

#### Step 4: Create AMI
- Use EC2 console or CLI to create AMI
- **Behind the Scenes**: Creates EBS snapshots of all volumes
- Generates new AMI with unique ID
- Can be used immediately after creation

#### Step 5: Launch Instances from AMI
- Use new AMI as template for new instances
- All customizations preserved
- Faster launch times than installing from scratch

### AMI Storage
- **EBS Snapshots**: Root volume and data volumes stored as snapshots
- **S3**: AMI metadata stored in S3
- **Regional**: AMIs are region-specific by default
- **Copying**: Can copy AMIs across regions for global deployment

## 4. AMI Management

### Regional Considerations
- **Region-Specific**: AMIs created in one region, available in that region
- **Cross-Region Copy**: Use "Copy AMI" to replicate across regions
- **Global Infrastructure**: Leverage AWS global footprint

### Cost Considerations
- **Storage Costs**: EBS snapshots incur storage charges
- **Transfer Costs**: Cross-region copying has data transfer fees
- **Instance Launch**: No additional cost for using custom AMIs

### Best Practices
- **Version Control**: Use descriptive names and versioning
- **Regular Updates**: Keep AMIs updated with security patches
- **Testing**: Test AMIs before production deployment
- **Cleanup**: Deregister unused AMIs and delete associated snapshots

## 5. Practical Example: Cross-Region Deployment

### Scenario
- Create customized instance in us-east-1a
- Build AMI from customized instance
- Launch identical instance in us-east-1b using the AMI

### Process
1. **Launch in us-east-1a**: Start with base AMI, customize
2. **Create AMI**: Stop instance, build AMI (creates EBS snapshots)
3. **Copy AMI**: Copy to us-east-1b (if needed)
4. **Launch in us-east-1b**: Use AMI to launch identical instance

### Benefits
- **Consistency**: Identical environments across regions
- **Disaster Recovery**: Quick recovery in different regions
- **Load Distribution**: Distribute workloads across regions
- **Compliance**: Meet data residency requirements

## 6. AMI vs Other AWS Services

### AMI vs Launch Templates
- **AMI**: Contains OS and software configuration
- **Launch Template**: Contains instance configuration (type, network, storage)
- **Combined Use**: Use AMI for software, launch template for infrastructure

### AMI vs Docker Images
- **AMI**: Full VM image with OS kernel
- **Docker**: Container image, shares host OS kernel
- **Use Cases**: AMI for full isolation, Docker for microservices

### AMI vs Snapshots
- **AMI**: Complete bootable image
- **Snapshot**: Point-in-time backup of EBS volume
- **Relationship**: AMI creation uses snapshots behind the scenes

## 7. Security Considerations

### AMI Security
- **Source Verification**: Only use trusted AMIs
- **Vulnerability Scanning**: Scan custom AMIs for vulnerabilities
- **Encryption**: Use encrypted EBS volumes
- **Access Control**: Control AMI sharing permissions

### Marketplace AMIs
- **Vendor Reputation**: Research marketplace vendors
- **Pricing**: Understand licensing costs
- **Support**: Check vendor support offerings
- **Updates**: Plan for vendor-provided updates

## 8. Key Takeaways for SysOps Associate

- **AMI Types**: AWS-provided (free), Custom (user-created), Marketplace (paid)
- **Components**: OS, software, configurations, metadata
- **Creation Process**: Launch → Customize → Stop → Create AMI → Launch new instances
- **Storage**: EBS snapshots + S3 metadata
- **Regional**: AMIs are region-specific, can be copied across regions
- **Benefits**: Faster boot times, consistent deployments, pre-configured environments
- **Use Cases**: Standardized deployments, disaster recovery, marketplace solutions
- **Cost**: Storage charges for snapshots, no cost for AMI usage
- **Security**: Verify sources, scan for vulnerabilities, control access
- **Best Practice**: Version control, regular updates, testing before production
