# EC2 Image Builder Hands-On Practice

This note provides a detailed step-by-step walkthrough of using EC2 Image Builder to create a custom AMI with pre-installed components and test it in a production workflow.

## Prerequisites

### Required Permissions
- EC2 full access (for instance and AMI management)
- IAM access (to create and manage roles)
- Systems Manager access (for Image Builder service)

### Expected Timeline
- **Pipeline Creation**: 5-10 minutes
- **Build + Test + Distribution**: 15-30 minutes total
- **Total Hands-On Time**: 30-45 minutes

### Cost Considerations
- **Free Tier**: t2.micro instances included
- **Components**: Free to use
- **Storage**: Minimal charge for AMI snapshot
- **Recommendation**: Use t2.micro to stay within free tier

## Part 1: Create the IAM Role

### Step 1: Manual IAM Role Creation
**Why Manual?** Automatic role creation sometimes misses policies. Manual creation ensures correct configuration.

1. **Navigate to IAM Console**
   - Services → IAM → Roles
   - Click "Create role"

2. **Select Trusted Entity**
   - **Trusted entity type**: AWS service
   - **Service or use case**: EC2
   - Click "Next"

3. **Attach Policies**
   Search for and attach these three required policies:
   
   **Policy 1: EC2InstanceProfileForImageBuilder**
   - Allows Image Builder to manage builder instances
   - Full permissions for Image Builder operations
   - Search: "EC2InstanceProfileForImageBuilder"
   
   **Policy 2: AmazonSSMManagedInstanceCore**
   - Required for Systems Manager access
   - Enables EC2 Instance Connect
   - Enables SSM Session Manager
   - Search: "AmazonSSMManagedInstanceCore"
   
   **Policy 3: ECRContainerBuilds** (Optional)
   - Only if building Docker container images
   - Skip if only building AMIs
   - Search: "ECRContainerBuilds"

4. **Add Tags (Optional)**
   - Key: `Purpose` → Value: `ImageBuilder`
   - Key: `Environment` → Value: `Lab`

5. **Name the Role**
   - **Role Name**: `EC2InstanceProfileForImageBuilder`
   - **Description**: "Role for EC2 Image Builder service"
   - Click "Create role"

### Verification
- Role created successfully
- Three policies attached
- EC2 service trust relationship established

## Part 2: Create Image Pipeline

### Step 1: Navigate to EC2 Image Builder
1. AWS Console → Services → EC2 Image Builder
2. Click "Create image pipeline" or "Get started"

### Step 2: Configure Pipeline Details
1. **Pipeline Name**: `MyDemoPipeline`
   - Use descriptive names
   - Common pattern: `{AppName}-Pipeline`

2. **Build Schedule** (Choose one)
   
   **Option A: Manual (Recommended for Lab)**
   - Only runs when you manually trigger it
   - No automatic schedule
   - Best for learning and testing

   **Option B: Scheduled**
   - Weekly: Monday 9:00 AM UTC
   - Customize cron expression for other frequencies
   - Example: `0 2 ? * SUN` (Sundays 2 AM)

   **Option C: Conditional**
   - "Run at scheduled time only on dependency updates"
   - Automatically rebuilds when components update
   - Good for security patching

3. **Click "Next"**

### Step 3: Create Recipe

#### Recipe Selection/Creation
1. **Recipe Option**: "Create a new recipe"
   - We don't have a pre-existing recipe for this lab

2. **Image Type**: Select "AMI"
   - Alternative: Docker (for container images)
   - We're building an EC2 AMI

3. **Recipe Details**
   - **Name**: `MyDemoRecipe`
   - **Version**: `1.0.0`
     - Format: `major.minor.patch`
     - Increment when updating components
   - Click "Next"

#### Select Source Image

1. **Managed Images** (Select)
   - Pre-built AWS images
   - Regularly updated with patches
   - Free to use

2. **Choose Base Image**: `Amazon Linux 2`
   - Other options: Ubuntu, RHEL, Windows Server
   - Amazon Linux 2: Good for AWS-focused labs

3. **Architecture**: 
   - **Select**: x86_64
   - **DO NOT select**: ARM64
     - Reason: t2.micro not available for ARM64
     - Will cause launch errors later

4. **Image Origin**: `Quick Start`
   - Pre-configured AWS-managed images
   - Tested and maintained by AWS

5. **Select Image**
   - **Amazon Linux 2 x86**
     - Look for: `amzn2-ami-minimal-hvm-*-x86_64-ebs`
     - Use latest available version

6. **OS Version**
   - Select: "Use the latest available OS version"
   - Always gets latest patches on build

7. **Click "Next"**

#### Add Components and Configure Build

1. **Add Build Components**
   - Components customize the base image
   - Install software and configure settings

   **Component 1: AWS CLI v2** (If upgrading CLI)
   - Search: `aws-cli-version-2-linux`
   - Default Amazon Linux 2 includes CLI v1
   - This component upgrades to v2
   - Click "Add"

   **Component 2: Java** (If installing Java)
   - Search: `amazon-corretto-11-headless`
   - Installs OpenJDK 11 from Amazon Corretto
   - Headless: No GUI components
   - Click "Add"

2. **Component Ordering**
   - Reorder if necessary (drag to reorder)
   - **Recommended Order**:
     1. Install AWS CLI v2 first
     2. Install Java after
   - Order can affect dependencies

3. **Create Custom Component** (Optional for advanced use)
   - Write custom buildspec YAML
   - Install custom applications
   - Run custom scripts
   - Not needed for this lab

4. **Review Selected Components**
   - Verify all desired software included
   - Each component shows in build logs

5. **Click "Next"**

### Step 4: Configure Test Components

1. **Testing Options**
   - Test Instance: Optional phase after build
   - Validates AMI before distribution

2. **Available Tests**
   - Pre-built test components
   - Custom test components
   - Skip testing option

3. **For This Lab**: Skip Testing
   - Checkbox: "Skip"
   - Reason: Hands-on validation will be manual
   - In production: Always enable tests

4. **Click "Next"**

### Step 5: Configure Infrastructure

1. **Infrastructure Configuration**
   - Defines EC2 instance for building
   - Specifies IAM role and instance type

2. **Option Selection**: "Create new infrastructure configuration"
   - Name: `MyDemoInfra`
   - More control than defaults

3. **IAM Role Selection**
   - **Search/Select**: `EC2InstanceProfileForImageBuilder`
   - Role we created earlier
   - Must have required policies

4. **Instance Type**
   - **Select**: `t2.micro`
   - **Reason**: Free tier eligible, saves cost
   - **Default**: m5.large (would incur charges)
   - **Alternatives**: t2.small, t3.micro

5. **Additional Settings** (Optional)
   - **VPC/Subnet**: Default options OK
   - **Security Group**: Default options OK
   - **Key Pair**: Not needed (using Systems Manager)

6. **Architecture Verification**
   - Confirm: t2.micro only supports x86_64
   - Matches our AMI selection (x86, not ARM64)

7. **Click "Next"**

### Step 6: Configure Distribution

1. **Distribution Settings**
   - Where to distribute created AMI
   - Can be same or multiple regions

2. **Option A: Service Defaults** (For Lab)
   - Distributes to current region only
   - Where pipeline was created
   - Simplest option

3. **Option B: Create New Distribution Configuration** (Advanced)
   ```
   Region 1: eu-west-2 (Europe - London)
   Region 2: us-east-1 (N. Virginia)
   Region 3: ap-southeast-1 (Singapore)
   ```
   - Each region: Separate AMI copy
   - Each region: Separate storage costs
   - Useful for global applications

4. **For This Lab**: Use Service Defaults
   - Single region distribution
   - Only eu-west-2 (or your current region)

5. **Click "Next"**

### Step 7: Review and Create

1. **Review Configuration**
   - Pipeline name: `MyDemoPipeline`
   - Recipe: `MyDemoRecipe v1.0.0`
   - Infrastructure: `MyDemoInfra` with t2.micro
   - Components: AWS CLI v2, Java 11
   - Distribution: Service default

2. **No Changes Needed**: Click "Create pipeline"

3. **Success Message**
   - Pipeline created successfully
   - Ready to execute

## Part 3: Execute Pipeline

### Step 1: Run the Pipeline

1. **Navigate to Pipelines**
   - EC2 Image Builder → Image pipelines
   - Select: `MyDemoPipeline`

2. **Execute Pipeline**
   - Click "Run pipeline"
   - Confirmation: "Start execution"
   - Click "Run"

### Step 2: Monitor Pipeline Execution

1. **Execution Screen**
   - **Status**: Shows current phase
   - **Output Image**: The AMI being created
   - **Output Image Status**: pending/building/testing/distributing/available

2. **Build Phase** (5-15 minutes)
   - EC2 builder instance launches
   - Components installed
   - AMI created from instance
   - Builder instance terminated

   **Verification**: Check EC2 Instances
   - Console → EC2 → Instances
   - See `Builder instance for MyDemoRecipe`
   - Instance Name tag visible
   - Image Builder tag visible
   - Temporary instance only

3. **Test Phase** (2-5 minutes)
   - Test instance launches
   - AMI mounted on test instance
   - Tests execute (if configured)
   - Test instance terminates

   **Verification**: Check Instances Again
   - `Test instance for MyDemoRecipe` visible
   - Short-lived temporary instance
   - Instance from new AMI

4. **Distribution Phase** (2-5 minutes)
   - AMI distributed to target regions
   - For single region: Minimal work
   - For multi-region: Copy to each region

5. **Completed**
   - Status: "Available"
   - AMI ready for use

## Part 4: Verify Components Installation

### Step 1: Launch Instance from New AMI

1. **Locate the AMI**
   - EC2 Console → AMIs
   - Filter: "Owned by me"
   - Find: `MyDemoRecipe` (with timestamp)
   - Name pattern: `MyDemoRecipe-{timestamp}`

2. **Verify AMI Details**
   - **Created by**: EC2 Image Builder
   - **Architecture**: x86_64
   - **Root Device**: EBS

3. **Launch Instance**
   - Click AMI → "Launch instance from AMI"
   - **Instance Name**: `Test-From-AMI`
   - **Instance Type**: t2.micro
   - **Key Pair**: Proceed without key pair (optional)
   - **Security Group**: Choose existing or create new
   - **Scroll down** → "Advanced details" (optional)
   - Click "Launch instance"

### Step 2: Connect to Instance

1. **Wait for Running State**
   - Status check: "2/2 checks passed"
   - Takes 1-2 minutes

2. **Connect Options**

   **Option A: EC2 Instance Connect** (Recommended)
   - Select instance → Connect
   - EC2 Instance Connect tab
   - Browser-based terminal
   - No key pair needed
   - Username: `ec2-user`

   **Option B: SSH** (Requires key pair)
   - SSH from local machine
   - Requires inbound SSH (port 22)
   - Username: `ec2-user`

3. **Connection Command**
   ```bash
   Using EC2 Instance Connect:
   - No command needed, use browser terminal
   
   Using SSH:
   ssh -i your-key.pem ec2-user@public-ip
   ```

### Step 3: Verify AWS CLI v2 Installation

```bash
# Check AWS CLI version
aws --version

# Expected output:
# aws-cli/2.7.0 Python/3.9.11 Linux/5.10.xxx-generic x86_64.amzn2

# Version should be 2.x.x
```

### Step 4: Verify Java 11 Installation

```bash
# Check Java version
java -version

# Expected output:
# openjdk version "11.0.xx" 2021-xx-xx
# OpenJDK Runtime Environment 18.3 (build 11.0.xx+9-post-Amazon-Linux release)
# OpenJDK 64-Bit Server VM 18.3 (build 11.0.xx+9-post-Amazon-Linux, mixed mode, sharing)

# Should show Java 11 (openjdk 11)
```

### Step 5: Additional Verification Commands

```bash
# List installed packages
yum list installed | grep java

# Verify Java executable location
which java

# Display Amazon Linux version
cat /etc/os-release

# Check installed AWS CLI plugins
aws plugin list
```

## Part 5: Cleanup Process

### Step 1: Terminate Test Instance
1. EC2 Console → Instances
2. Select: `Test-From-AMI`
3. State → Terminate instance
4. Confirm termination

### Step 2: Deregister AMI
1. EC2 Console → AMIs
2. Select: `MyDemoRecipe-{timestamp}`
3. Actions → Deregister AMI
4. Confirm (snapshots will remain)

### Step 3: Delete Associated Snapshots
1. EC2 Console → Snapshots
2. Filter: Snapshots created by Image Builder
3. Select snapshots from `MyDemoRecipe`
4. Actions → Delete snapshots
5. Confirm deletion
6. Frees up storage costs

### Step 4: Clean Infrastructure (Optional)
1. **Keep Pipeline**: Manual pipeline won't run automatically
2. **Delete if Needed**:
   - Image Builder → Image pipelines
   - Select `MyDemoPipeline`
   - Delete pipeline (recipes remain for reuse)

3. **Keep IAM Role**: Can be reused for future Image Builder work
4. **Delete if Not Needed**:
   - IAM → Roles
   - Select `EC2InstanceProfileForImageBuilder`
   - Delete role

## Part 6: Troubleshooting Common Issues

### Issue: Build Fails with Architecture Mismatch
**Problem**: ARM64 selected instead of x86_64
```
Error: No compatible instances available for t2.micro in ARM64
```
**Solution**:
- Use x86_64 architecture for base image
- Select: "Amazon Linux 2 x86" (not ARM64)
- t2.micro only supports x86_64

### Issue: IAM Role Missing Required Policies
**Problem**: Build instance cannot be created
```
AccessDenied: User is not authorized to perform: ec2:RunInstances
```
**Solution**:
- Verify three policies attached to role:
  1. EC2InstanceProfileForImageBuilder
  2. AmazonSSMManagedInstanceCore
  3. ECRContainerBuilds (if using)
- Recreate role with all policies

### Issue: EC2 Instance Connect Not Available
**Problem**: Cannot access instance via browser terminal
```
Error: Instance Connect not available
```
**Solution**:
- Verify security group allows Systems Manager
- Ensure IAM role has AmazonSSMManagedInstanceCore
- Use SSH with key pair instead
- Check instance status checks passing

### Issue: Build Phase Timeout
**Problem**: Pipeline runs for >30 minutes
```
Build phase still running after 30 minutes
```
**Solution**:
- Check instance in EC2 console
- Monitor Image Builder execution logs
- May be installing large components
- Reduce component complexity for faster builds

### Issue: Test Phase Fails
**Problem**: Test instance created but tests fail
```
Test phase failed: Custom tests returned error code 1
```
**Solution**:
- If automatic tests enabled, review test logs
- Verify components compatible with base image
- Test components individually first
- Skip tests and manually verify components

## Part 7: Best Practices for Hands-On Learning

### Documentation
- Record pipeline configuration
- Note component versions used
- Document customizations made
- Keep reference for future builds

### Incremental Testing
- Start with single component
- Build and test
- Add additional components
- Verify each addition

### Cost Management
- Always use t2.micro for learning
- Use single-region distribution
- Clean up immediately after testing
- Monitor AMI storage costs

### Security Practice
- Use security groups appropriately
- Don't share AMIs publicly in lab
- Review component contents before adding
- Delete test AMIs after validation

### Iteration Pattern
1. Create simple recipe
2. Build and verify
3. Add new components
4. Build and verify again
5. Document changes
6. Keep versioned recipes

## Key Exam Points from Hands-On

1. **Three Policies Required**: EC2InstanceProfileForImageBuilder, AmazonSSMManagedInstanceCore, ECRContainerBuilds
2. **Architecture Matters**: x86 vs. ARM64 affects instance type compatibility
3. **Temporary Instances**: Builder and test instances are short-lived
4. **Three Phases**: Build → Test → Distribute is mandatory workflow
5. **AMI Storage**: Snapshots continue incurring costs until deleted
6. **Manual Execution**: Pipeline won't auto-run unless scheduled
7. **Component Ordering**: Order of component installation can matter
8. **Verification Methods**: Console tags and manual CLI verification confirm setup
9. **Cost Optimization**: t2.micro saves money, manual execution on-demand
10. **Cleanup Importance**: Deregister AMIs and delete snapshots for cost control