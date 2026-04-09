# Changing EC2 Instance Type

This note covers how to resize (change) an EC2 instance type.
It is critical for scaling applications vertically and for SysOps operations.

## 1. Prerequisites

- The instance must be **EBS-backed** (not instance store)
  - EBS volumes are persistent block storage
  - Instance store volumes are temporary and lost on stop/start
- The instance must be in a **stopped** state before changing instance type

## 2. Steps to Change Instance Type

### Step 1: Stop the Instance
- Select your running instance in the EC2 console
- Stop the instance and wait for state to become `stopped`
- You cannot change instance type while the instance is running

### Step 2: Change Instance Type
- Right-click the stopped instance
- Select **Instance Settings** → **Change Instance Type**
- Choose the new instance type from the dropdown list
  - Example: upgrade from `t2.micro` to `t2.small`
  - Example: downgrade to `t2.nano`
  - Can also switch to completely different types (e.g., `r5dn.4xlarge`)

### Step 3: Start the Instance
- Once the instance type is changed, start the instance
- Wait for the instance state to become `running`

## 3. Important Concepts

### EBS-Optimized Instances
- **EBS-Optimized**: provides dedicated throughput to EBS volumes
  - Enabled by default on newer instance type generations (e.g., `t3`)
  - Not available on older generations (e.g., `t2.micro`, `t2.small`)
- Better throughput to EBS volumes for I/O intensive workloads
- Newer generation instance types support this feature

### Stop and Start Behavior
- When you stop and start (not reboot) an instance:
  - The instance is **moved to a different physical host** within the AWS data center
  - Public IP address may change (if not using Elastic IP)
  - Private IP address remains the same
  - Instance ID remains the same
- **Data Persistence**: Because storage is EBS-backed:
  - All files and data on the EBS volume remain intact
  - Previous data is accessible immediately after restart

## 4. Practical Example from the Lecture

**Before:**
- Instance type: `t2.micro`
- Memory: 993 MB
- File: `hello.txt` (containing "hello")

**Steps:**
1. Stopped the `t2.micro` instance
2. Changed instance type to `t2.small`
3. Started the instance

**After:**
- Instance type: `t2.small`
- Memory: 1,991 MB (approximately doubled)
- File: `hello.txt` still exists and contains "hello"
- Proof that data persistence works with EBS

## 5. Free Tier Considerations

- `t2.micro` is part of the AWS Free Tier (750 hours/month for new accounts)
- `t2.small` and larger are **not** free tier eligible
- Changing to `t2.small` or larger will incur charges
- After testing, resize back to `t2.micro` to avoid charges

## 6. How to Verify Instance After Resize

- Connect to the instance via SSH or EC2 Instance Connect
- Check file system:
  - `ls -la` to list files
  - Previous files will be present
- Check memory to verify resize:
  - `free -m` to display memory in MB
  - Compare values before and after resize
- Run any commands to verify application state

## 7. Key Takeaways for SysOps Associate

- Know the three-step process: **Stop → Change Type → Start**
- Understand that only EBS-backed instances can be resized
- Know what happens during stop/start (move to different host)
- Understand EBS persistence and why data survives a resize
- Know free tier eligibility for instance types
- Be able to verify instance changes via SSH commands like `free -m`
- Understand EBS-optimized feature and its availability on newer instance types
- Recognize that public IP may change, but instance ID and private IP remain the same
