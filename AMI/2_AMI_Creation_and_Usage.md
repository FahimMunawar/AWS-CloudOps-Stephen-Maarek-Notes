# AMI Creation and Usage

This note covers the practical process of creating custom AMIs from EC2 instances and launching new instances from those AMIs. It demonstrates the benefits of pre-configured environments for faster deployments.

## Overview

**AMI Creation Process**
- Start with a base instance
- Customize with software and configurations
- Create AMI (creates EBS snapshots)
- Launch new instances from the AMI
- **Benefits**: Faster boot times, consistent environments, pre-installed software

## 1. Preparing an Instance for AMI Creation

### Launch Base Instance
- **AMI**: Amazon Linux 2
- **Instance Type**: t2.micro (free tier eligible)
- **Key Pair**: Optional (can use EC2 Instance Connect)
- **Security Group**: Allow SSH and HTTP (ports 22 and 80)
- **Storage**: Default EBS configuration

### Customize with User Data Script
- **Purpose**: Install Apache web server during launch
- **Script Content**:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
```
- **Execution**: Runs automatically on first boot
- **Result**: Apache web server installed and running

### Verify Customization
- **Wait Time**: Allow 2-3 minutes for user data script completion
- **Test**: Access public IP via HTTP (http://public-ip)
- **Expected**: Default Apache test page
- **Note**: Even if instance shows "running", user data may still be executing

## 2. Creating an AMI

### Stop Instance (Recommended)
- **Why**: Ensures data integrity
- **Process**: Stop instance before AMI creation
- **State**: Instance must be stopped or running (running is acceptable but stopped is safer)

### Create AMI from Instance
1. **Select Instance**: Right-click running/stopped instance
2. **Actions** → **Image and templates** → **Create image**
3. **Image Name**: Descriptive name (e.g., "Demo-Image")
4. **Image Description**: Optional description
5. **Settings**: 
   - **No reboot**: Uncheck (allows instance to remain running)
   - **Instance volumes**: Include all attached EBS volumes
6. **Create Image**: Initiates AMI creation process

### AMI Creation Process
- **Status**: Initially "pending"
- **Duration**: 5-10 minutes depending on instance size
- **Behind Scenes**:
  - Creates EBS snapshots of all volumes
  - Registers AMI in your account
  - Generates unique AMI ID
- **Completion**: Status changes to "available"

### View Created AMI
- **Location**: EC2 Console → AMIs (left sidebar)
- **Ownership**: Filter by "Owned by me"
- **Details**: Shows AMI ID, creation date, architecture, etc.

## 3. Launching Instances from Custom AMI

### Using EC2 Launch Wizard
1. **Launch Instance**: Click "Launch instance"
2. **Choose AMI**: 
   - **My AMIs** tab
   - Select your custom AMI
   - Verify details (architecture, root device type)

3. **Configure Instance**:
   - **Instance Type**: Choose appropriate size
   - **Key Pair**: Optional
   - **Network Settings**: Select security group
   - **Storage**: Automatically configured from AMI

4. **Advanced Details**:
   - **User Data**: Add final customizations
   - **Example**: Create index.html file
   ```bash
   echo "Hello World" > /var/www/html/index.html
   ```

### Benefits Demonstration
- **Without AMI**: Install Apache each time (2-3 minutes)
- **With AMI**: Apache pre-installed, just add content (seconds)
- **Boot Time**: Significantly faster launches
- **Consistency**: Identical software stack across instances

## 4. AMI Management

### AMI Storage Details
- **EBS Snapshots**: Each volume becomes a snapshot
- **Cost**: Charged for snapshot storage
- **Cleanup**: Delete AMI and associated snapshots when no longer needed

### AMI Permissions
- **Default**: Private to your account
- **Sharing**: Can share with specific AWS accounts
- **Public**: Can make AMI public (not recommended for custom AMIs)

### AMI Lifecycle
- **Registration**: AMI registered in your account
- **Usage**: Can launch unlimited instances
- **Deregistration**: Remove AMI when no longer needed
- **Snapshots**: Associated snapshots remain until deleted

## 5. Practical Example: Web Server AMI

### Scenario
- Create AMI with pre-installed Apache
- Launch multiple web servers quickly
- Customize content per instance

### Step-by-Step Process

#### 1. Base Instance Setup
```bash
# User Data Script for base instance
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
```

#### 2. AMI Creation
- Stop instance
- Create image: "WebServer-AMI"
- Wait for completion

#### 3. Launch from AMI
```bash
# User Data for new instances
echo "Hello World from Instance 1" > /var/www/html/index.html
```

#### 4. Result
- Instance launches in seconds
- Apache already installed and running
- Only custom content needs to be added

## 6. Best Practices for AMI Creation

### Pre-Creation Checklist
- **Clean Instance**: Remove temporary files, logs, sensitive data
- **Stop Services**: Ensure no running processes that shouldn't persist
- **Update Software**: Apply latest security patches
- **Test Functionality**: Verify all customizations work correctly

### AMI Naming Convention
- **Format**: `AppName-Version-Date` (e.g., `WebServer-v1.0-2024-01-15`)
- **Versioning**: Increment version numbers for updates
- **Description**: Include what's included in the AMI

### Cost Optimization
- **Regular Cleanup**: Delete unused AMIs and snapshots
- **Region Planning**: Create AMIs in regions where needed
- **Storage Classes**: Consider moving old snapshots to cheaper storage

### Security Considerations
- **No Sensitive Data**: Remove passwords, keys, credentials
- **Encrypted Volumes**: Use encrypted EBS volumes
- **Minimal Attack Surface**: Only install required software
- **Regular Updates**: Keep base AMIs updated

## 7. Troubleshooting AMI Issues

### Common Problems
- **AMI Creation Fails**: Check instance state, permissions, EBS health
- **Launch Fails**: Verify AMI architecture matches instance type
- **Software Not Working**: Check if services start automatically
- **Permissions Issues**: Ensure AMI ownership and sharing settings

### Verification Steps
- **Test Launch**: Always test AMI by launching instance
- **Check Logs**: Review system logs for startup issues
- **Validate Software**: Ensure all pre-installed software works
- **Performance Test**: Verify boot times and functionality

## 8. Advanced AMI Use Cases

### Multi-Tier Applications
- **Web Tier AMI**: Pre-configured web servers
- **App Tier AMI**: Application servers with runtime
- **DB Tier AMI**: Database servers with optimized settings

### Development Environments
- **Dev AMI**: Development tools and IDEs
- **Test AMI**: Testing frameworks and mock services
- **Staging AMI**: Production-like configurations

### Compliance and Security
- **Hardened AMI**: Security tools and compliance configurations
- **Encrypted AMI**: All volumes encrypted by default
- **Audited AMI**: Pre-installed logging and monitoring

## 9. Key Takeaways for SysOps Associate

- **AMI Creation**: Stop instance → Create image → Wait for completion
- **Benefits**: Faster launches, consistent environments, pre-installed software
- **Process**: Customize instance → Create AMI → Launch from AMI
- **Storage**: EBS snapshots created automatically
- **User Data**: Use for final customizations, not full installations
- **Testing**: Always test AMIs by launching instances
- **Cleanup**: Deregister unused AMIs and delete snapshots
- **Security**: Remove sensitive data before AMI creation
- **Cost**: Charged for snapshot storage
- **Best Practice**: Version control AMIs, regular updates, documentation
