# EC2 CloudWatch Metrics

This note covers CloudWatch metrics for EC2 instances, which are critical for monitoring and troubleshooting.
It includes AWS-provided metrics, custom metrics, and how to enable detailed monitoring.

## Overview

**CloudWatch Integration with EC2**
- AWS automatically pushes metrics for your EC2 instances
- Metrics are used for monitoring, alerting, and troubleshooting
- Essential for SysOps Associate exam and real-world operations

## 1. Basic vs Detailed Monitoring

### Basic Monitoring
- **Default**: Enabled automatically for all EC2 instances
- **Interval**: Metrics collected every 5 minutes
- **Cost**: Free
- **Retention**: 15 months

### Detailed Monitoring
- **Optional**: Must be enabled manually
- **Interval**: Metrics collected every 1 minute
- **Cost**: Additional charges apply
- **Retention**: 15 months
- **How to Enable**:
  - Go to EC2 instance → Monitoring tab
  - Click "Manage detailed monitoring"
  - Enable detailed monitoring

## 2. AWS-Provided EC2 Metrics

### CPU Metrics
- **CPUUtilization**: Percentage of CPU usage (0-100%)
- **CPUCreditUsage** (T2/T3 instances): Credits consumed during burst
- **CPUCreditBalance** (T2/T3 instances): Available burst credits
- Most important metric for performance monitoring

### Network Metrics
- **NetworkIn**: Bytes received by the instance
- **NetworkOut**: Bytes sent by the instance
- **NetworkPacketsIn**: Number of packets received
- **NetworkPacketsOut**: Number of packets sent
- Measures network traffic in and out of the instance

### Status Check Metrics
- **StatusCheckFailed**: Overall status check (0 = passed, 1 = failed)
- **StatusCheckFailed_Instance**: Instance-level health check
- **StatusCheckFailed_System**: System/hardware-level health check
- **StatusCheckFailed_AttachedEBS**: EBS volume health check

### Disk Metrics (Instance Store Only)
- **DiskReadOps**: Read operations per second
- **DiskWriteOps**: Write operations per second
- **DiskReadBytes**: Bytes read per second
- **DiskWriteBytes**: Bytes written per second
- **Note**: For EBS-backed instances, disk metrics are on the EBS volume, not the instance

### Important: RAM is NOT Included
- AWS does not provide RAM/memory metrics for EC2 instances
- You must push custom RAM metrics if needed
- Common exam question: "Can you get RAM usage from CloudWatch? No"

## 3. Status Checks Explained

### Instance Status Check
- **What it checks**: EC2 virtual machine health
- **Causes of failure**:
  - Incorrect network configuration
  - Incorrect firewall rules
  - Exhausted memory
  - Corrupted file system
  - Incompatible kernel
- **Recovery**: Usually requires instance restart or reconfiguration

### System Status Check
- **What it checks**: Underlying AWS hardware health
- **Causes of failure**:
  - Loss of network connectivity
  - Loss of system power
  - Software issues on the physical host
  - Hardware issues on the physical host
- **Recovery**: AWS automatically migrates to healthy hardware
- **Note**: You cannot fix system status check failures yourself

### EBS Status Check
- **What it checks**: Health of attached EBS volumes
- **Causes of failure**: EBS volume issues
- **Recovery**: May require volume replacement or repair

## 4. Custom Metrics

### What are Custom Metrics?
- Metrics you define and push to CloudWatch
- Not automatically provided by AWS
- Examples: RAM usage, application-specific metrics

### Resolution Options
- **Standard Resolution**: 1-minute intervals
- **High Resolution**: Up to 1-second intervals
- High resolution incurs additional costs

### Pushing Custom Metrics
- Requires IAM role on EC2 instance with CloudWatch permissions
- Use AWS CLI, SDK, or CloudWatch agent
- Example: Push RAM usage metrics

### Common Custom Metrics for EC2
- **Memory Utilization**: RAM usage percentage
- **Disk Space**: Available disk space
- **Application Metrics**: Request count, error rates, response times
- **Custom Application Logs**: Converted to metrics

## 5. Viewing Metrics in Console

### EC2 Instance Monitoring Tab
- Go to EC2 instance → Monitoring tab
- View basic metrics with 5-minute intervals
- Enable detailed monitoring for 1-minute intervals

### CloudWatch Dashboard
- Create a custom dashboard for EC2 metrics
- Add multiple metrics to one view
- Visualize CPU, network, status checks together
- Set up alarms and notifications

### Key Metrics to Monitor
- CPUUtilization > 80% (potential bottleneck)
- StatusCheckFailed = 1 (health issues)
- CPUCreditBalance (for burstable instances)
- NetworkIn/NetworkOut (traffic patterns)

## 6. Cost Considerations

### Basic Monitoring
- Free for all EC2 instances
- Included in EC2 pricing

### Detailed Monitoring
- Additional cost per instance per month
- $0.015 per instance (approximate, check current pricing)

### Custom Metrics
- $0.30 per metric per month (standard resolution)
- $0.003 per 1,000 metrics (high resolution)

## 7. CloudWatch Unified Agent

### Overview
- **Purpose**: Collect additional system-level metrics and logs from EC2 instances
- **Supported Platforms**: EC2 instances and on-premises servers
- **Key Features**:
  - Collects RAM usage, disk space, processes, etc.
  - Sends logs to CloudWatch Logs
  - Enables detailed monitoring not available by default

### Why Use the Unified Agent?
- **Default EC2**: No logs or additional metrics sent to CloudWatch
- **Unified Agent**: Required to collect RAM, detailed system metrics, and logs
- **Essential for**: Memory monitoring, log aggregation, process monitoring

### Configuration Methods
1. **SSM Parameter Store**: Store configuration centrally
2. **Configuration File**: Local configuration file on the instance

### Required Permissions
- **EC2 Instances**: IAM role with permissions for:
  - CloudWatch metrics
  - CloudWatch logs
  - SSM Parameter Store (if used)
- **On-Premises Servers**: IAM access keys with appropriate permissions

### Metrics Namespace
- **Default Namespace**: `CWAgent`
- **Customizable**: Can be changed in configuration
- All agent-pushed metrics appear under this namespace

### Procstat Plugin
- **Purpose**: Monitor individual processes on Linux/Windows servers
- **Metrics Collected**:
  - CPU usage per process
  - Memory usage per process
  - Process statistics
- **Process Selection Methods**:
  - **PID**: Process ID number
  - **Process Name**: Exact name match
  - **Pattern**: Regex pattern matching
- **Metric Prefix**: `procstat_`
  - Examples: `procstat_cpu_usage`, `procstat_memory_used`, `procstat_time`

### Use Cases
- **Memory Monitoring**: Track RAM usage (not provided by AWS by default)
- **Process Monitoring**: Monitor specific application processes
- **Log Aggregation**: Send system and application logs to CloudWatch Logs
- **System Metrics**: Disk space, network statistics, etc.

### Practical Installation and Configuration

#### Step 1: Create IAM Role
1. Go to IAM → Roles → Create Role
2. Select "AWS service" → "EC2"
3. Attach `CloudWatchAgentServerPolicy`
4. Name: `AmazonEC2RoleForCloudWatch`
5. Attach role to your EC2 instance (Security → Modify IAM role)

#### Step 2: Launch EC2 Instance with Role
- Use Amazon Linux 2 AMI
- Attach the IAM role created above
- Configure security group for SSH and HTTP access
- Launch instance

#### Step 3: Install CloudWatch Unified Agent
```bash
sudo yum install amazon-cloudwatch-agent
```

#### Step 4: Run Configuration Wizard
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```
- Select Linux → EC2 → Root user
- Configure StatsD daemon (optional)
- Enable host metrics collection (CPU, Memory, etc.)
- Configure log file monitoring
- Store configuration in SSM Parameter Store

#### Step 5: Start Agent from SSM Configuration
```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl \
  -a fetch-config \
  -m ec2 \
  -c ssm:AmazonCloudWatch-Linux \
  -s
```

#### Step 6: Verify Installation
- **Logs**: Check CloudWatch → Log Groups for your configured log files
- **Metrics**: Check CloudWatch → Metrics → CWAgent namespace
- **Common Metrics**:
  - `mem_used_percent`: RAM usage
  - `disk_used_percent`: Disk space usage
  - `cpu_usage_idle`: CPU idle time
  - `cpu_usage_user`: CPU user time

#### Configuration Storage Options
1. **SSM Parameter Store** (Recommended for multiple instances)
   - Centralized configuration
   - Requires SSM permissions
   - Retrieved at runtime

2. **Local Configuration File**
   - Stored on instance
   - Good for single instances
   - Static configuration

#### Troubleshooting
- **Permission Issues**: Ensure IAM role has `CloudWatchAgentServerPolicy`
- **SSM Access**: For SSM config storage, temporarily add `CloudWatchAgentAdminPolicy`
- **Agent Not Starting**: Check configuration syntax and required dependencies
- **Missing Metrics**: Verify agent is running with `sudo systemctl status amazon-cloudwatch-agent`

## 7. Key Takeaways for SysOps Associate

- Know the four main AWS-provided metric categories: CPU, Network, Status Checks, Disk (instance store only)
- RAM usage is NOT provided by AWS - you must push it as a custom metric
- Status checks differentiate between instance-level (your config) and system-level (AWS hardware) issues
- Basic monitoring = 5-minute intervals (free), Detailed monitoring = 1-minute intervals (paid)
- EBS disk metrics appear on the EBS volume, not the EC2 instance
- T2/T3 instances have CPU credit metrics for burst monitoring
- Custom metrics require IAM permissions to push to CloudWatch
- Status check failures trigger automatic recovery for system issues