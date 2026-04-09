# AWS Instance Scheduler

This note covers the AWS Instance Scheduler solution, a CloudFormation-deployed tool for automatically starting and stopping AWS resources to reduce costs.

## Overview

**AWS Instance Scheduler**
- **Not a native AWS service**: Deployed via CloudFormation
- **Purpose**: Automatically start/stop resources to reduce costs (up to 70%)
- **Supported Services**: EC2 instances, EC2 Auto Scaling Groups, RDS instances
- **Architecture**: Lambda functions + DynamoDB for scheduling
- **Use Case**: Stop non-production resources outside business hours

## 1. How It Works

### Architecture Components
1. **DynamoDB Table**: Stores schedule configurations
2. **Main Lambda Function**: Reads schedules and triggers actions
3. **Service-Specific Lambdas**: Handle EC2, RDS, ASG operations
4. **CloudWatch Events**: Trigger scheduling checks (every 5 minutes by default)

### Process Flow
1. Lambda function checks DynamoDB for schedules
2. Evaluates current time against defined schedules
3. Triggers appropriate start/stop actions via service-specific lambdas
4. Resources are started/stopped based on schedule rules

### Key Features
- **Cross-Account Support**: Can manage resources across multiple AWS accounts
- **Cross-Region Support**: Works across different AWS regions
- **Production Ready**: Complete solution with monitoring and logging
- **Tagging**: Automatically tags resources with scheduler actions

## 2. Supported Services

### EC2 Instances
- Individual EC2 instances
- Can start/stop based on schedules
- Supports all EC2 instance types

### EC2 Auto Scaling Groups
- Manages ASG desired capacity
- Can set capacity to 0 during off-hours
- Scales back up during business hours

### RDS Instances & Clusters
- RDS DB instances
- RDS Aurora clusters
- Supports start/stop operations
- Optional snapshot creation before stopping

### Additional Services
- **Neptune**: Graph database clusters
- **DocumentDB**: MongoDB-compatible clusters
- **Redshift**: Data warehouse clusters (if supported)

## 3. Schedule Configuration

### DynamoDB Table Structure
- **Config Table**: Main configuration table
- **Schedule Items**: Define when to start/stop resources
- **Parameters**:
  - `name`: Schedule identifier
  - `begintime`: Start time (HH:MM format)
  - `endtime`: End time (HH:MM format)
  - `description`: Schedule description

### Schedule Types
- **Office Hours**: Business hours only
- **Weekdays**: Monday-Friday
- **Working Days**: Custom weekday definitions
- **Monthly**: First Monday of each quarter
- **Custom**: Flexible time-based rules

### Time Zone Support
- Configurable default time zone
- Supports different time zones per schedule
- UTC-based calculations

## 4. Deployment Process

### CloudFormation Template
- **Source**: AWS Solutions library
- **Template URL**: Pre-hosted CloudFormation template
- **One-Click Deployment**: Launch directly from AWS console

### Key Parameters
- **Frequency**: How often to check schedules (default: 5 minutes)
- **Time Zone**: Default time zone for schedules
- **Service Enablement**: Which services to enable (EC2, RDS, etc.)
- **Tagging**: Enable automatic tagging of actions
- **Snapshots**: Configure RDS snapshot behavior

### Stack Resources Created
- **Lambda Functions**: Multiple functions for different services
- **DynamoDB Tables**: Configuration and state tables
- **IAM Roles**: Permissions for Lambda functions
- **CloudWatch Events**: Scheduled triggers
- **CloudWatch Logs**: Logging and monitoring

## 5. Cost Optimization

### Savings Potential
- **Up to 70% cost reduction** for non-production resources
- **Example**: Stop dev/test instances outside business hours
- **Business Hours**: 40 hours/week savings on 168-hour week

### When to Use
- **Development/Test Environments**: Stop when not in use
- **Non-Critical Workloads**: Batch processing, staging environments
- **Cost-Conscious Organizations**: Maximize cloud cost efficiency
- **Compliance Requirements**: Ensure resources only run when needed

### Cost Considerations
- **CloudFormation Stack**: Minimal cost for infrastructure
- **Lambda Invocations**: Pay per schedule check
- **DynamoDB**: Small storage costs for schedules
- **Net Savings**: Usually significant compared to running costs

## 6. Practical Implementation

### Accessing the Solution
1. Search "Instance Scheduler AWS" in AWS documentation
2. Navigate to AWS Solutions page
3. Click "Launch Solution" → redirects to CloudFormation
4. Template URL auto-populated

### Configuration Steps
1. **Stack Name**: Choose descriptive name (e.g., "Instance-Scheduler")
2. **Parameters**:
   - Enable desired services (EC2, RDS, ASG)
   - Set default time zone
   - Configure tagging and snapshots
3. **Create Stack**: Deploy all resources

### Managing Schedules
1. **DynamoDB Console**: Access config table
2. **Add Items**: Create schedule entries
3. **Define Rules**: Set start/stop times and days
4. **Tag Resources**: Apply schedule tags to target resources

### Resource Tagging
- **Schedule Tag**: Applied to resources to be scheduled
- **Action Tags**: Automatically added when actions performed
- **Format**: Includes timestamp and action type

## 7. Monitoring and Troubleshooting

### CloudWatch Integration
- **Logs**: All Lambda functions log to CloudWatch Logs
- **Metrics**: Custom metrics for scheduler performance
- **Alarms**: Set up alerts for scheduling failures

### Common Issues
- **Time Zone Mismatches**: Ensure consistent time zone settings
- **IAM Permissions**: Verify Lambda has necessary service permissions
- **Resource Tagging**: Ensure resources have correct schedule tags
- **Schedule Conflicts**: Multiple schedules may conflict

### Best Practices
- **Test Schedules**: Start with test resources before production
- **Monitor Logs**: Regularly check CloudWatch logs for issues
- **Backup Configurations**: Export DynamoDB schedules regularly
- **Documentation**: Document all schedules and their purposes

## 8. Limitations and Considerations

### Service Limitations
- **Not Real-Time**: Checks schedules every 5 minutes (configurable)
- **No Sub-Minute Precision**: Minimum 5-minute intervals
- **Manual Override**: Can manually start/stop resources outside schedules

### Operational Considerations
- **State Management**: Resources maintain state between start/stop cycles
- **Data Persistence**: EBS volumes and RDS data preserved
- **Application Impact**: Ensure applications can handle start/stop cycles

### Security Considerations
- **IAM Permissions**: Lambda functions need appropriate service permissions
- **Cross-Account**: Requires trust relationships for cross-account scheduling
- **Audit Trail**: All actions logged for compliance

## 9. Key Takeaways for SysOps Associate

- **AWS Instance Scheduler**: CloudFormation solution for automated start/stop
- **Purpose**: Cost optimization by stopping resources outside business hours
- **Supported Services**: EC2, RDS, ASG, Neptune, DocumentDB
- **Architecture**: Lambda functions + DynamoDB schedules
- **Deployment**: One-click CloudFormation deployment
- **Configuration**: DynamoDB table for schedule definitions
- **Savings**: Up to 70% cost reduction for non-production resources
- **Use Cases**: Dev/test environments, batch processing, compliance
- **Not Real-Time**: 5-minute check intervals (configurable)
- **Cross-Account/Region**: Supports managing resources across accounts/regions
