# EC2 Status Checks and Automated Recovery

This note covers EC2 status checks, which are automated health checks performed by AWS, and automated recovery options for failed instances.

## Overview

**EC2 Status Checks**
- Automated checks performed by AWS to identify hardware and software issues
- Three types: System Status Check, Instance Status Check, Attached EBS Status Check
- Critical for monitoring EC2 instance health and enabling automated recovery

## 1. System Status Check

### What It Monitors
- Problems with AWS infrastructure (physical host where your instance runs)
- Hardware and software issues on the underlying host

### Common Failure Causes
- Hardware failure on the physical host
- Loss of system power
- Software issues on the physical host
- Network connectivity loss to the host
- Loss of network connectivity from the host

### Resolution Method
- **Stop and Start** the instance (not reboot)
- This migrates the instance to a different physical host within AWS

### Stop/Start Behavior
- Instance moves to a different physical host in the same AZ
- **Public IP changes** (unless using Elastic IP)
- **Private IP remains the same**
- **EBS data is preserved** (for EBS-backed instances)
- Instance ID remains the same

### Monitoring System Issues
- Check **AWS Personal Health Dashboard** for:
  - Scheduled maintenance affecting your host
  - Critical infrastructure issues
  - Recommended actions and timelines

## 2. Instance Status Check

### What It Monitors
- Software and network configuration issues on your instance
- Problems within your EC2 instance itself

### Common Failure Causes
- Invalid network configuration
- Exhausted memory (out of RAM)
- Corrupted file system
- Incompatible kernel
- Misconfigured services

### Resolution Method
- **Reboot** the instance
- Fix the underlying configuration issue
- May require instance reconfiguration

## 3. Attached EBS Status Check

### What It Monitors
- Health of EBS volumes attached to your instance
- Volume reachability and I/O operation capability

### Common Failure Causes
- EBS volume connectivity issues
- I/O operation failures
- Volume corruption or hardware issues

### Resolution Method
- Reboot the instance
- Replace the affected EBS volume(s)
- Check volume health and attachments

## 4. CloudWatch Metrics for Status Checks

### Available Metrics
- **StatusCheckFailed_System**: System status check (0 = healthy, 1 = failed)
- **StatusCheckFailed_Instance**: Instance status check (0 = healthy, 1 = failed)
- **StatusCheckFailed_AttachedEBS**: EBS status check (0 = healthy, 1 = failed)
- **StatusCheckFailed**: Overall status check (combines all three)

### Metric Values
- **0**: Status check passed (healthy)
- **1**: Status check failed (unhealthy)

### Monitoring in Console
- View in EC2 instance **Monitoring** tab
- Create CloudWatch dashboards
- Set up alarms for automated response

## 5. Automated Recovery Options

### Option 1: CloudWatch Alarm with Recovery Action (Recommended)

#### Setup Process
1. Create CloudWatch alarm on status check metric
2. Set threshold to 1 (failed)
3. Configure alarm actions:
   - **Recover Instance**: Automatic recovery
   - **SNS Notification**: Send alerts

#### Recovery Behavior
- AWS automatically recovers the failed instance
- **Preserves**:
  - Same private IP address
  - Same public IP address (or Elastic IP)
  - Same instance metadata
  - Same placement group
  - All attached EBS volumes and data
- **Minimal disruption** to applications

#### Use Cases
- Critical instances requiring high availability
- When preserving instance identity is important
- Production workloads needing automated failover

### Option 2: Auto Scaling Group

#### Setup Process
1. Create Auto Scaling Group (ASG)
2. Set min/max/desired capacity to 1
3. Configure health checks to monitor EC2 status checks
4. Enable automatic instance replacement

#### Recovery Behavior
- When instance fails health check:
  - ASG terminates the unhealthy instance
  - ASG launches a new instance to maintain desired capacity
- **New instance characteristics**:
  - Different private IP
  - Different instance ID
  - New EBS volumes (unless using launch templates with specific volumes)
  - Requires automation to restore application state

#### Use Cases
- Stateless applications
- When automation can restore instance configuration
- Cost-effective replacement over recovery
- Development/testing environments

## 6. Choosing the Right Recovery Method

### When to Use CloudWatch Recovery
- **Single critical instances** where preservation matters
- **Stateful applications** requiring same IP addresses
- **Databases or applications** with specific EBS volume requirements
- **Production workloads** needing minimal disruption

### When to Use Auto Scaling Group
- **Stateless applications** that can be recreated
- **Web servers** or application servers
- **Microservices** with external configuration management
- **Development environments** where replacement is acceptable

### Comparison Table

| Aspect | CloudWatch Recovery | Auto Scaling Replacement |
|--------|-------------------|-------------------------|
| **IP Preservation** | Yes (private & public) | No |
| **EBS Volumes** | Same volumes preserved | New volumes (template-based) |
| **Instance ID** | Same | Different |
| **Placement Group** | Same | May change |
| **Data Preservation** | Full | Requires backup/automation |
| **Cost** | Recovery action cost | New instance launch cost |
| **Use Case** | Critical stateful instances | Stateless scalable apps |

## 6. Practical CloudWatch Alarm Setup

### Creating Status Check Alarms

#### From EC2 Console
1. Go to EC2 instance → **Status Checks** tab
2. Click **Actions** → **Create status check alarm**
3. Configure alarm settings:
   - **Alarm Notification**: Send to SNS topic (optional)
   - **Alarm Action**: Choose **Recover** (for system issues) or **Reboot** (for instance issues)
   - **Alarm Threshold**: Status check failed metric
   - **Consecutive Periods**: Number of 5-minute periods (e.g., 1)
4. **Important**: Can only create recovery alarms for **StatusCheckFailed_System**
5. Name the alarm and create

#### Manual CloudWatch Alarm Creation
1. Go to CloudWatch → Alarms → Create Alarm
2. Select metric: EC2 → Status Checks → StatusCheckFailed_System
3. Set threshold: > 0 (failed)
4. Configure actions: **EC2 Action** → **Recover this instance**
5. Add SNS notification (optional)

### Testing Recovery Alarms

#### Simulate Alarm Trigger
- Use AWS CLI or CloudShell to manually trigger alarm:
```bash
aws cloudwatch set-alarm-state \
  --alarm-name "Your-Alarm-Name" \
  --state-value ALARM \
  --state-reason "Testing recovery action"
```

#### What Happens During Recovery
1. Alarm enters ALARM state
2. SNS notification sent (if configured)
3. EC2 recovery action executes
4. Instance is recovered on the same hardware (if possible) or migrated
5. Alarm history shows: "Successfully executed action: EC2 Recover"

#### Verification Steps
- Check CloudWatch → Alarms for alarm state changes
- Monitor EC2 instance status during recovery
- Review alarm history for executed actions
- Verify instance remains accessible after recovery

### Alarm Configuration Options

#### Recovery vs Reboot Actions
- **Recover**: For system status check failures (AWS infrastructure issues)
- **Reboot**: For instance status check failures (your configuration issues)
- **Note**: Cannot mix recovery and reboot in same alarm

#### Threshold Settings
- **Metric**: StatusCheckFailed_System (for recovery) or StatusCheckFailed_Instance (for reboot)
- **Threshold**: > 0 (any failure triggers alarm)
- **Period**: 5 minutes (standard monitoring interval)
- **Consecutive Periods**: 1 (immediate action) or higher for stability

#### Notification Setup
- Create SNS topic for alarm notifications
- Subscribe email addresses or other endpoints
- Include alarm details in notification messages

### Troubleshooting Alarm Issues

#### Common Problems
- **"Can only be done on status check failed system"**: Recovery actions only work with system status checks
- **Alarm not triggering**: Check metric selection and threshold settings
- **Recovery not executing**: Verify IAM permissions for CloudWatch actions
- **Instance not recovering**: Check if instance is in supported state

#### Permissions Required
- CloudWatch needs permission to perform EC2 recovery actions
- Ensure alarm has proper IAM role or the account has necessary permissions

## 7. Best Practices

### Monitoring Setup
- Enable **detailed monitoring** for faster detection
- Create CloudWatch dashboards for status check visibility
- Set up SNS notifications for all status check failures

### Recovery Configuration
- Test recovery procedures in non-production environments
- Document recovery processes and expected behavior
- Monitor recovery success rates and adjust thresholds

### High Availability Design
- Use **multiple AZs** to reduce single-host failure impact
- Implement **health checks** at application level
- Consider **read replicas** for databases to minimize downtime

## 8. Key Takeaways for SysOps Associate

- **Three status check types**: System (AWS hardware), Instance (your config), EBS (volumes)
- **System failures**: Stop/start instance to migrate to healthy host
- **Instance failures**: Reboot or fix configuration
- **EBS failures**: Reboot or replace volumes
- **CloudWatch Recovery**: Preserves instance identity, best for critical instances
- **Auto Scaling**: Replaces instance, best for stateless applications
- Status check metrics enable automated monitoring and recovery
- Personal Health Dashboard shows AWS-scheduled maintenance affecting your instances
- Stop/start (not reboot) migrates instances to different physical hosts
