# EC2 Hibernate

This note covers EC2 Hibernate, a feature that allows instances to be suspended and resumed while preserving RAM state for faster startup times.

## Overview

**EC2 Hibernate**
- Allows instances to be suspended with RAM state preserved
- Much faster startup compared to normal boot process
- RAM contents are saved to encrypted EBS root volume
- Instance appears to "wake up" rather than reboot

## 1. How Hibernate Works

### Normal Stop/Start Process
- **Stop**: Instance shuts down, RAM contents lost
- **Start**: Full OS boot process, user data scripts run, applications initialize
- **Time**: Can take several minutes for complex applications

### Hibernate Process
1. **Hibernation Triggered**: Instance enters stopping state
2. **RAM Dump**: RAM contents written to encrypted file on root EBS volume
3. **Instance Stops**: Physical RAM cleared, but data preserved on EBS
4. **Resume**: RAM contents loaded back from EBS to memory
5. **Result**: Instance "wakes up" in exact same state as when hibernated

### Key Differences
- **Normal Boot**: OS kernel loads, services start, caches warm up
- **Hibernate Resume**: OS frozen state restored, applications continue running
- **Speed**: Hibernate resume is much faster (seconds vs minutes)

## 2. Prerequisites and Limitations

### Required Conditions
- **Root Volume**: Must be EBS (not instance store)
- **Encryption**: Root EBS volume must be encrypted
- **Space**: Root volume must have sufficient space for RAM dump
- **RAM Size**: Instance RAM must be ≤ 150 GB (limit may change)
- **Instance Types**: Not supported on bare metal instances

### Supported Platforms
- **Operating Systems**: Linux and Windows
- **Instance Purchasing**: On-Demand, Reserved, and Spot instances
- **Instance Families**: Most modern instance families supported

### Time Limits
- **Maximum Hibernation**: 60 days (may change)
- After 60 days, instance cannot be resumed from hibernation

## 3. Use Cases

### Long-Running Processes
- Applications that take time to initialize
- Services that maintain in-memory state
- Databases with large in-memory caches
- Applications that shouldn't restart from scratch

### Fast Resume Requirements
- Development environments that need quick start/stop cycles
- Applications sensitive to initialization time
- Workloads where preserving exact state is critical
- Cost optimization through frequent stop/start

### When NOT to Use Hibernate
- Applications that can tolerate normal boot times
- Instances that don't need to preserve RAM state
- Workloads that require fresh starts
- Bare metal instances (not supported)

## 4. Technical Details

### RAM Preservation
- **What Gets Saved**: All RAM contents, including:
  - Running processes
  - Open file handles
  - Network connections (may be lost)
  - Application state
- **What Doesn't Get Saved**: 
  - Network connections (typically reset)
  - Temporary files in memory
  - Some system state that requires fresh initialization

### Storage Requirements
- **Space Needed**: At least RAM size + some overhead
- **Location**: Encrypted file on root EBS volume
- **Security**: Data encrypted at rest using EBS encryption

### Performance Impact
- **Hibernation Time**: Slightly longer than normal stop (due to RAM dump)
- **Resume Time**: Significantly faster than normal start
- **Storage I/O**: Increased I/O during hibernation/resume

## 5. Enabling Hibernate

### Instance Launch
- **AMI Support**: Must use hibernation-enabled AMI
- **Instance Type**: Choose supported instance type
- **Root Volume**: Ensure EBS encrypted with adequate space
- **Launch**: Enable hibernation in advanced options

### Console Steps
1. Launch instance wizard
2. Choose hibernation-compatible AMI
3. Select supported instance type
4. Configure storage: EBS root volume, encrypted, sufficient space
5. Advanced details: Enable hibernation
6. Launch instance

### Programmatic Enablement
- Use AWS CLI or SDK with hibernation configuration
- Specify hibernation options in launch template

## 6. Practical Hibernate Demo

### Launching a Hibernate-Enabled Instance

#### Step 1: Choose AMI and Instance Type
- Select **Amazon Linux 2** AMI
- Choose supported instance type (e.g., **t2.micro**)
- Select or create key pair

#### Step 2: Configure Storage
- **Root Volume**: Must be EBS (default)
- **Encryption**: Enable encryption on root EBS volume
- **Key**: Use default AWS managed key or custom KMS key
- **Size**: Ensure sufficient space (RAM size + overhead)
  - Example: t2.micro has 1 GB RAM → 8 GB EBS sufficient

#### Step 3: Enable Hibernation
- In launch wizard, scroll to **Advanced details**
- Find **Stop - Hibernate behaviour**
- Check **Enable hibernation as an additional stop behavior**
- **Warning**: Ensure root volume has enough space and is encrypted

#### Step 4: Launch Instance
- Configure security groups as needed
- Launch the instance

### Testing Hibernation

#### Connect and Check Uptime
```bash
# Connect via EC2 Instance Connect or SSH
uptime
# Output: Shows how long instance has been running
# Example: "up 1 min"
```

#### Hibernate the Instance
1. Select instance in EC2 console
2. **Actions** → **Instance State** → **Stop - Hibernate**
3. Wait for instance to enter **stopped** state

#### Resume from Hibernation
1. Select stopped instance
2. **Actions** → **Instance State** → **Start**
3. Wait for instance to enter **running** state

#### Verify Hibernation Worked
```bash
# Connect again and check uptime
uptime
# Output should show CONTINUOUS uptime from before hibernation
# Example: "up 3 min" (not reset to 0)
```

### What the Uptime Command Proves
- **Normal Stop/Start**: Uptime resets to 0 (fresh boot)
- **Hibernate Resume**: Uptime continues from before hibernation
- **Proof**: OS was never actually stopped - it was frozen and resumed

### Key Observations
- Instance appears to "wake up" rather than reboot
- All processes and state preserved
- Much faster than normal boot process
- RAM contents restored from EBS volume

### Cleanup
- Terminate the test instance when done
- Hibernation data stored on EBS is automatically cleaned up

## 7. Hibernate vs Other States

### Comparison Table

| Feature | Stop/Start | Hibernate | Terminate |
|---------|------------|-----------|-----------|
| **RAM State** | Lost | Preserved | Lost |
| **EBS Data** | Preserved | Preserved | Preserved (if not deleted) |
| **Boot Time** | Normal | Fast | N/A (new instance) |
| **Cost** | Storage only | Storage only | No cost |
| **Max Duration** | Unlimited | 60 days | N/A |
| **Use Case** | Normal pause | Quick resume | Permanent removal |

### When to Choose Each
- **Stop/Start**: Standard pause/resume, acceptable boot times
- **Hibernate**: Need fast resume, preserve exact state
- **Terminate**: No longer need instance, clean slate

## 8. Cost Considerations

### Storage Costs
- **EBS Storage**: Pay for root volume storage during hibernation
- **No Instance Cost**: Stopped instances don't incur compute charges
- **Data Transfer**: No charges for hibernation process

### Optimization Tips
- Use appropriate instance sizes (RAM ≤ 150 GB)
- Ensure adequate but not excessive EBS storage
- Consider hibernation duration limits (60 days)

## 9. Best Practices

### Configuration
- Always use encrypted EBS root volumes
- Ensure sufficient storage space (RAM size + 10-20% overhead)
- Test hibernation in development environments first

### Monitoring
- Monitor hibernation success/failure
- Track resume times vs normal boot times
- Set up alerts for hibernation limit approaching (60 days)

### Security
- Use encrypted root volumes (required)
- Follow least privilege IAM policies
- Regularly rotate hibernation-enabled instances

## 10. Key Takeaways for SysOps Associate

- **EC2 Hibernate** preserves RAM state for faster startup
- **Requirements**: EBS root volume, encrypted, sufficient space, RAM ≤ 150 GB
- **Process**: RAM dumped to EBS → instance stops → RAM loaded on resume
- **Use cases**: Long-running processes, fast resume needs, state preservation
- **Limits**: 60-day maximum hibernation, no bare metal support
- **Benefits**: Much faster than normal boot, preserves application state
- **Not for**: Applications that can tolerate normal boot times
- Hibernate is faster and preserves more state than normal stop/start
