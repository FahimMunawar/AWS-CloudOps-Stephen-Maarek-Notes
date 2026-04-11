# 24. ASG Lifecycle Hooks and Advanced Features

## Part 1: Lifecycle Hooks Overview

### What Are Lifecycle Hooks?

Lifecycle hooks are a powerful ASG feature that allows you to interject custom logic and scripts into the lifecycle of EC2 instances as they are launched or terminated. Without lifecycle hooks, instances follow a straightforward state transition: **Pending → In Service** (on launch) or **In Service → Terminated** (on termination). With lifecycle hooks, you can insert wait states that allow you to perform custom initialization tasks during startup or cleanup tasks before termination.

**Key Definition**: Lifecycle hooks enable you to hook into the lifecycle of ASG instances, allowing you to perform custom actions before instances enter the In Service state or before they are terminated.

### The State Machine Without Lifecycle Hooks

When you launch an instance in an ASG without lifecycle hooks:
1. Instance starts (Pending state)
2. Instance immediately goes into service (In Service state)
3. Instance serves traffic

When you terminate an instance:
1. Instance begins termination process (In Service → Terminating)
2. Instance is terminated (Terminated state)
3. Instance becomes unavailable

This rapid transition doesn't give you time to perform custom operations like configuration, health verification, logging, or graceful shutdown procedures.

### The State Machine With Lifecycle Hooks

Lifecycle hooks introduce wait states that pause the normal transition:

**Launch Lifecycle Hook States:**
```
┌─────────────────────────────────────────────────────────┐
│                  Instance Lifecycle (Launch)             │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  Pending → Pending:Wait → [Custom Script] →              │
│  Pending:Proceed → In Service                            │
│                                                           │
│  Lifecycle Hook Pauses Here (Wait State)                 │
│  - Run initialization script                             │
│  - Install software                                      │
│  - Configure application                                 │
│  - Health verification                                   │
│  - Database connection initialization                    │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

**Termination Lifecycle Hook States:**
```
┌─────────────────────────────────────────────────────────┐
│                Instance Lifecycle (Termination)          │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  In Service → Terminating:Wait → [Custom Script] →      │
│  Terminating:Proceed → Terminated                        │
│                                                           │
│  Lifecycle Hook Pauses Here (Wait State)                 │
│  - Pause serving requests                                │
│  - Extract application logs                              │
│  - Drain connections                                     │
│  - Take instance backups                                 │
│  - Create EBS snapshots                                  │
│  - Troubleshoot and debug                                │
│                                                           │
└─────────────────────────────────────────────────────────┘
```

### Launch Lifecycle Hook States Explained

**1. Pending:Wait State**
- The instance launches but does not automatically move to In Service
- ASG holds the instance in a wait state indefinitely (or until you complete the action)
- Perfect for initial setup, configuration, and health verification
- Default timeout: 300 seconds (5 minutes) but configurable up to 86,400 seconds (24 hours)

**2. Pending:Proceed State**
- You explicitly signal the ASG that initialization is complete
- The instance transitions from Pending:Wait → Pending:Proceed → In Service
- Can occur automatically after timeout or when you send a completion signal
- Instance then begins serving traffic normally

### Termination Lifecycle Hook States Explained

**1. Terminating:Wait State**
- Instance remains running but is flagged for termination
- ASG holds the instance in a wait state before final termination
- Allows time for cleanup, logging, backup operations
- Default timeout: 300 seconds (5 minutes) but configurable
- Instance continues running during this period, can still drain connections

**2. Terminating:Proceed State**
- You signal that cleanup is complete
- The instance transitions from Terminating:Wait → Terminating:Proceed → Terminated
- Instance is then fully terminated and removed from ASG
- Resources are released

### Use Cases for Lifecycle Hooks

**Launch Lifecycle Hook Use Cases:**
1. **Custom Initialization**: Run scripts to install dependencies, configure services, pull configuration from parameter store
2. **Database Connection Warmup**: Initialize database connections, run cache pre-population queries
3. **Health Verification**: Run custom health checks before declaring the instance ready
4. **Application Specific Setup**: Deploy application code, activate features, register with service discovery
5. **Compliance and Security**: Run security scanning, validate instance against company policies, enable monitoring agents
6. **Performance Optimization**: Pre-warm caches, establish peer connections, compile JIT components

**Termination Lifecycle Hook Use Cases:**
1. **Log Extraction**: Copy application logs to S3 or centralized logging system before instance is gone
2. **Graceful Shutdown**: Allow in-flight requests to complete, drain persistent connections
3. **Debugging and Troubleshooting**: Capture memory dumps, create EBS snapshots of root volume for forensics
4. **State Backup**: Export instance state, database backups, session data to persistent storage
5. **Inventory Updates**: Update configuration management database to remove instance
6. **Metrics and Auditing**: Capture final performance metrics before instance terminates

## Part 2: Integration with EventBridge, SNS, and SQS

### How Lifecycle Hook Notifications Work

When a lifecycle hook event occurs, ASG sends a notification to target services. These target services can trigger automated processes:

**Notification Flow Architecture:**
```
┌──────────────────────────────────────────────────────┐
│                  ASG Instance Event                   │
│            (Launch or Termination Hook)               │
└──────────────────┬───────────────────────────────────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
    SNS Topic  EventBridge   SQS Queue
       │          │          │
       │          │          │
    ┌──┴──┐    ┌──┴───┐  ┌───┴──┐
    │     │    │      │  │      │
    ▼ Email App  Lambda  Worker
    Notification Processor  Application
    Handler     Function   Processor
```

### EventBridge Integration (Recommended)

**Advantages of Using EventBridge:**
- Most flexible targeting (multiple targets per rule)
- Supports Lambda functions, other EventBridge rules, SNS, SQS, and more
- Pattern-based matching allows complex event filtering
- Retry policies and Dead Letter Queues (DLQ) for reliability
- Native event transformation capabilities

**Typical Flow:**
```
ASG Lifecycle Hook Event
  ↓
EventBridge Rule Pattern Matching
  ↓
Matches Rule? Yes → EventBridge Routes to Target
  ↓
Lambda Function Triggered
  ↓
Lambda Executes Custom Script
  ↓
Lambda Completes Action
  ↓
Lambda Calls ASG API: complete-lifecycle-action
  ↓
Instance Transitions to Next State
```

**EventBridge Rule Example Pattern:**
```json
{
  "source": ["aws.autoscaling"],
  "detail-type": ["EC2 Instance Launch Successful"],
  "detail": {
    "AutoScalingGroupName": ["my-asg"],
    "Cause": ["LifecycleHookInitiated"]
  }
}
```

### SNS Integration

**SNS Workflow:**
- ASG sends notification to SNS topic
- SNS topic forwards to subscribers (email, SQS, Lambda, HTTP endpoints)
- Good for alerting, logging, multi-destination scenarios
- Can trigger email notifications about lifecycle events

**Best For:**
- Administrative notifications (escalations, alerts)
- Multiple notification destinations
- Simple pub-sub architecture
- Email integration for operators

### SQS Integration

**SQS Workflow:**
- ASG sends lifecycle event to SQS queue
- Worker application polls queue for messages
- Worker processes messages and calls ASG API to complete action
- Good for decoupled processing, handling volume spikes

**Best For:**
- Worker applications that need to process events
- Decoupling ASG from processing logic
- Handling variable load on processors
- Batch processing of lifecycle events

**SQS Advantages:**
- Messages persist in queue (15 minutes default)
- Worker can take time to process without timeout
- Retry capability if worker fails
- Can scale workers independently of ASG

### Practical Example: Lambda Function Processing Lifecycle Hook

```python
import json
import boto3
import logging

autoscaling = boto3.client('autoscaling')
logger = logging.getLogger()

def lambda_handler(event, context):
    """
    Process ASG lifecycle hook event
    """
    logger.info(f"Received event: {json.dumps(event)}")
    
    # Extract lifecycle hook details
    detail = event['detail']
    lifecycle_action_token = detail['LifecycleActionToken']
    lifecycle_hook_name = detail['LifecycleHookName']
    asg_name = detail['AutoScalingGroupName']
    instance_id = detail['EC2InstanceId']
    lifecycle_transition = detail['LifecycleTransition']
    
    try:
        if lifecycle_transition == 'autoscaling:EC2_INSTANCE_LAUNCHING':
            # Handle launch lifecycle hook
            logger.info(f"Instance {instance_id} launching - running init script")
            
            # Your custom initialization logic here
            # Example: Install software, configure application, etc.
            result = initialize_instance(instance_id)
            
            if result['success']:
                logger.info("Initialization successful")
                action_result = 'CONTINUE'
            else:
                logger.error(f"Initialization failed: {result['error']}")
                action_result = 'ABANDON'
        
        elif lifecycle_transition == 'autoscaling:EC2_INSTANCE_TERMINATING':
            # Handle termination lifecycle hook
            logger.info(f"Instance {instance_id} terminating - running cleanup")
            
            # Your custom termination logic here
            # Example: Extract logs, backup state, graceful shutdown
            result = cleanup_instance(instance_id)
            
            if result['success']:
                logger.info("Cleanup successful")
                action_result = 'CONTINUE'
            else:
                logger.warning(f"Cleanup completed with issues: {result['warning']}")
                action_result = 'CONTINUE'  # Still proceed even with issues
        
        # Complete the lifecycle action
        response = autoscaling.complete_lifecycle_action(
            LifecycleActionResult=action_result,
            LifecycleHookName=lifecycle_hook_name,
            AutoScalingGroupName=asg_name,
            LifecycleActionToken=lifecycle_action_token,
            InstanceId=instance_id
        )
        
        logger.info(f"Lifecycle action completed: {response}")
        return {
            'statusCode': 200,
            'body': json.dumps('Lifecycle hook processed successfully')
        }
        
    except Exception as e:
        logger.error(f"Error processing lifecycle hook: {str(e)}")
        return {
            'statusCode': 500,
            'body': json.dumps(f'Error: {str(e)}')
        }

def initialize_instance(instance_id):
    """
    Custom initialization function
    """
    ec2 = boto3.client('ec2')
    ssm = boto3.client('ssm')
    
    try:
        # Example: Get instance details
        response = ec2.describe_instances(InstanceIds=[instance_id])
        instance = response['Reservations'][0]['Instances'][0]
        logger.info(f"Instance config: {instance['InstanceType']}, {instance['ImageId']}")
        
        # Example: Run Systems Manager document on instance
        ssm.send_command(
            InstanceIds=[instance_id],
            DocumentName='AWS-RunShellScript',
            Parameters={
                'commands': [
                    'yum update -y',
                    'yum install -y nginx',
                    'systemctl start nginx'
                ]
            }
        )
        
        return {'success': True, 'message': 'Initialization started'}
        
    except Exception as e:
        return {'success': False, 'error': str(e)}

def cleanup_instance(instance_id):
    """
    Custom cleanup function
    """
    try:
        # Example: Extract logs
        logs_to_backup = '/var/log/app/application.log'
        s3 = boto3.client('s3')
        
        # Would copy logs to S3 in real implementation
        logger.info(f"Would backup logs from {instance_id} to S3")
        
        # Example: Graceful shutdown
        logger.info(f"Signaling graceful shutdown to {instance_id}")
        
        return {'success': True, 'message': 'Cleanup completed'}
        
    except Exception as e:
        return {'success': False, 'warning': str(e)}
```

### Key API Calls for Lifecycle Hooks

**AWS CLI: Complete Lifecycle Action**
```bash
aws autoscaling complete-lifecycle-action \
  --lifecycle-action-result CONTINUE \
  --lifecycle-hook-name my-launch-hook \
  --auto-scaling-group-name my-asg \
  --lifecycle-action-token ... \
  --instance-id i-1234567890abcdef0 \
  --region us-east-1
```

**Parameters:**
- `--lifecycle-action-result`: CONTINUE (proceed to next state) or ABANDON (terminate instance)
- `--lifecycle-hook-name`: Name of the lifecycle hook
- `--auto-scaling-group-name`: ASG name
- `--lifecycle-action-token`: Token received in the notification
- `--instance-id`: ID of the instance

**Record Lifecycle Action Heartbeat** (to extend timeout):
```bash
aws autoscaling record-lifecycle-action-heartbeat \
  --lifecycle-hook-name my-launch-hook \
  --auto-scaling-group-name my-asg \
  --lifecycle-action-token ... \
  --instance-id i-1234567890abcdef0 \
  --region us-east-1
```

This prevents timeout if your process takes longer than expected.

## Part 3: Launch Templates vs Launch Configuration

### What Do They Do?

Both launch templates and launch configurations define the specification for launching EC2 instances within an ASG. They encapsulate instance configuration including:
- **Amazon Machine Image (AMI)**: The OS and base software
- **Instance Type**: CPU, memory, network performance (t3.micro, t3.small, m5.large, etc.)
- **Key Pair**: For SSH authentication
- **Security Groups**: Firewall rules
- **User Data**: Scripts to run on instance startup
- **Storage**: Root volume type (gp2, gp3, io1) and size
- **Tags**: Metadata and resource identification
- **IAM Instance Profile**: Permissions for the instance
- **EBS Optimization**: Storage optimization settings
- **Monitoring**: Detailed CloudWatch monitoring
- **Capacity Reservations**: Reserve capacity in specific AZs

### Launch Configuration (Legacy)

**Characteristics:**
- Older AWS approach (deprecation pathway ongoing)
- **Cannot be edited** after creation - must be recreated for any change
- Single version - no version management
- Simpler feature set
- Less flexible for modern infrastructure patterns
- AWS recommends migrating away from launch configurations

**Limitations:**
- No parameter subsets or inheritance
- No spot instances mixed with on-demand
- No placement groups or capacity reservation support
- No T2 unlimited burst mode
- No multiple instance types in one configuration

**When You Might Still Use:**
- Legacy systems that haven't migrated
- Simple use cases with infrequent changes
- Existing infrastructure not being updated

**Editing a Launch Configuration:**
```bash
# Step 1: Create a NEW launch configuration (cannot edit existing)
aws ec2 create-launch-configuration \
  --launch-configuration-name my-lc-v2 \
  --image-id ami-12345678 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-groups sg-12345678

# Step 2: Update ASG to use new configuration
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-configuration-name my-lc-v2

# Step 3: Must then do instance refresh or wait for scale events
```

**Complete Replacement Workflow:**
1. Create new launch configuration with desired settings
2. Update ASG to reference new configuration
3. Perform instance refresh (instances start using new config on next scale event)
4. Wait for old instances to age out or manually terminate them

**Time Impact:** Days to weeks if no scaling events occur.

### Launch Templates (Current Standard)

**Characteristics:**
- Modern AWS recommended approach
- **Multiple versions** - each version is immutable, create new version to update
- Flexible feature set
- Supports advanced configurations (spot mix, placement groups, capacity reservations)
- Can create parameter subsets (inherit from base template)
- Actively maintained and enhanced by AWS

**Key Advantages:**
1. **Version Control**: Track all versions, easy rollback to previous version
2. **Inheritance**: Create child templates from parent for configuration reuse
3. **Mixed Fleets**: On-demand + spot instances in single template
4. **Advanced Features**:
   - Placement groups for cluster computing
   - Capacity reservations for guaranteed capacity
   - Dedicated hosts for licensing requirements
   - Multiple instance types for fleet optimization
   - T2 unlimited burst mode
   - Elastic GPU and FPGA acceleration

**AWS Recommendation:** "Launch templates are the recommended approach for launching instances with an Auto Scaling Group."

**Creating a Launch Template:**
```bash
aws ec2 create-launch-template \
  --launch-template-name my-launch-template \
  --version-description "Version 1" \
  --launch-template-data '{
    "ImageId": "ami-12345678",
    "InstanceType": "t3.micro",
    "KeyName": "my-key-pair",
    "SecurityGroupIds": ["sg-12345678"],
    "UserData": "IyEvYmluL2Jhc2gKZWNobyAnSGVsbG8gV29ybGQn",
    "Monitoring": {
      "Enabled": true
    },
    "TagSpecifications": [{
      "ResourceType": "instance",
      "Tags": [{
        "Key": "Name",
        "Value": "MyInstance"
      }]
    }]
  }'
```

**Creating a New Version (Updating):**
```bash
aws ec2 create-launch-template-version \
  --launch-template-name my-launch-template \
  --source-version 1 \
  --launch-template-data '{
    "ImageId": "ami-87654321",
    "InstanceType": "t3.small"
  }'
```

This creates Version 2 automatically incrementing the version number.

**Using Template Version in ASG:**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-launch-template,Version=\$Latest \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 5 \
  --vpc-zone-identifier "subnet-12345678,subnet-87654321"
```

**Template Parameters (Can Be Overridden):**
```bash
# Override parameters at ASG launch time
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-launch-template \
  --mixed-instances-policy '{
    "LaunchTemplate": {
      "LaunchTemplateSpecification": {
        "LaunchTemplateName": "my-launch-template",
        "Version": "$Latest"
      },
      "Overrides": [
        {"InstanceType": "t3.micro"},
        {"InstanceType": "t3.small"},
        {"InstanceType": "t2.micro"},
        {"InstanceType": "t2.small"}
      ]
    },
    "InstancesDistribution": {
      "OnDemandBaseCapacity": 2,
      "OnDemandPercentageAboveBaseCapacity": 50,
      "SpotInstancePools": 4
    }
  }' \
  --min-size 5 \
  --max-size 20 \
  --desired-capacity 10 \
  --vpc-zone-identifier "subnet-12345678,subnet-87654321"
```

### Launch Template Subsets and Inheritance

**Creating a Base Launch Template:**
```bash
aws ec2 create-launch-template \
  --launch-template-name base-template \
  --launch-template-data '{
    "ImageId": "ami-base123",
    "InstanceType": "t3.micro",
    "KeyName": "my-key",
    "SecurityGroupIds": ["sg-base"],
    "TagSpecifications": [{
      "ResourceType": "instance",
      "Tags": [
        {"Key": "Environment", "Value": "Production"},
        {"Key": "ManagedBy", "Value": "ASG"}
      ]
    }]
  }'
```

**Creating Child Template (Parameter Subset):**
```bash
aws ec2 create-launch-template \
  --launch-template-name production-web-template \
  --launch-template-data '{
    "LaunchTemplateId": "lt-base123",
    "VersionNumber": 1,
    "SecurityGroupIds": ["sg-production-web"],
    "TagSpecifications": [{
      "ResourceType": "instance",
      "Tags": [{"Key": "Role", "Value": "WebServer"}]
    }]
  }'
```

**Advantages of This Approach:**
- Central management of common settings (image, instance type, key)
- Override specific parameters per use case (security groups, tags, etc.)
- Consistent versioning and change management
- Easier to maintain at scale

### Direct Comparison Table

| Feature | Launch Configuration | Launch Template |
|---------|----------------------|-----------------|
| **Editability** | Cannot edit (recreate) | Multiple versions (new version) |
| **AWS Status** | Legacy (deprecation path) | Current standard (recommended) |
| **Versioning** | None (single config) | Full version control (v1, v2, v3...) |
| **Inheritance** | Not supported | Supported (parameter subsets) |
| **Spot Mix** | Not supported | Supported (on-demand + spot) |
| **Placement Groups** | Not supported | Supported |
| **Capacity Reservation** | Not supported | Supported |
| **Dedicated Hosts** | Not supported | Supported |
| **Multiple Instance Types** | Not supported | Supported |
| **T2 Unlimited** | Not supported | Supported |
| **Update Process** | Hours/days | Minutes (instance refresh v2) |
| **Backward Compatibility** | Still works | Moving forward standard |
| **New Features** | No longer enhanced | Actively enhanced |

## Part 4: Scaling Based on SQS Queue Depth

### The Use Case: Queue-Based Application

A common pattern couples Auto Scaling with message queue depth metrics. This allows your ASG to scale based on work that needs to be done rather than just system metrics.

**Architecture Example:**
```
┌──────────────────────────────────────────────────────────┐
│                                                            │
│  API Layer / Event Source                                 │
│         ↓                                                  │
│    ┌────────────────────┐                                 │
│    │    SQS Queue       │  (Messages waiting to process)  │
│    │  Current Length: 500 messages                        │
│    └────────────────────┘                                 │
│         ↑                                                  │
│    Tracked by CloudWatch Metric                          │
│         ↑                                                  │
│    ApproximateNumberOfMessages                           │
│         ↑                                                  │
│    CloudWatch Alarm (Threshold: 100 messages)            │
│         ↑                                                  │
│    Triggers Scaling Policy                               │
│         ↑                                                  │
│    ASG-EC2 Instances (Currently 3)                       │
│    Scale to 6 instances to process faster                │
│         ↓                                                  │
│    Instances poll SQS and process messages               │
│         ↓                                                  │
│    Queue length decreases                                │
│         ↓                                                  │
│    Alarm clears, scale-in policy activates               │
│         ↓                                                  │
│    ASG scales back to 3 instances                        │
│                                                            │
└──────────────────────────────────────────────────────────┘
```

### How SQS Queue Metrics Work

**SQS CloudWatch Metrics:**

1. **ApproximateNumberOfMessages** (Primary Metric for Scaling)
   - Current number of messages available in the queue
   - Exact count including visible and invisible messages
   - Updated approximately every 60 seconds
   - Useful for: Scaling workers based on backlog size

2. **ApproximateNumberOfMessagesDelayed**
   - Messages scheduled for delivery in future
   - Updated approximately every 60 seconds

3. **ApproximateNumberOfMessagesNotVisible**
   - Messages currently being processed (visibility timeout)
   - Indicates active processing

**Queue Metrics Flow:**
```
Message arrives in SQS Queue
          ↓
ApproximateNumberOfMessages increases by 1
          ↓
Worker polls queue every 5-10 seconds
          ↓
Worker receives message(visibility timeout = 30 seconds)
          ↓
ApproximateNumberOfMessagesNotVisible increases by 1
          ↓
ApproximateNumberOfMessages decreases by 1
          ↓
Worker deletes message after processing
          ↓
ApproximateNumberOfMessagesNotVisible decreases by 1
```

### Setting Up SQS-Based Auto Scaling

**Step 1: Create an SQS Queue**
```bash
aws sqs create-queue \
  --queue-name job-processing-queue \
  --attributes VisibilityTimeout=300,MessageRetentionPeriod=1209600
```

Output includes QueueUrl and QueueArn.

**Step 2: Create a CloudWatch Alarm on Queue Depth**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name job-queue-depth-high \
  --alarm-description "Scale out when job queue builds up" \
  --metric-name ApproximateNumberOfMessages \
  --namespace AWS/SQS \
  --statistic Average \
  --period 300 \
  --threshold 100 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 1 \
  --dimensions Name=QueueName,Value=job-processing-queue
```

This alarm triggers when queue has more than 100 messages for 5 minutes.

**Step 3: Create a Step Scaling Policy**
```bash
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name job-processing-asg \
  --policy-name scale-out-on-queue \
  --policy-type StepScaling \
  --adjustment-type PercentChangeInCapacity \
  --metric-aggregation-type Average \
  --step-adjustments \
    MetricIntervalLowerBound=0,MetricIntervalUpperBound=50,ScalingAdjustment=20 \
    MetricIntervalLowerBound=50,ScalingAdjustment=50
```

This policy scales out by 20% for 0-50 messages above threshold, 50% for more.

**Step 4: Link Alarm to Scaling Policy**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name job-queue-scale-out \
  --alarm-actions arn:aws:autoscaling:us-east-1:123456789012:...(scaling-policy-arn)
```

**Step 5: Create Scale-In Alarm and Policy**
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name job-queue-depth-low \
  --metric-name ApproximateNumberOfMessages \
  --namespace AWS/SQS \
  --statistic Average \
  --period 300 \
  --threshold 20 \
  --comparison-operator LessThanThreshold \
  --evaluation-periods 2 \
  --dimensions Name=QueueName,Value=job-processing-queue

aws autoscaling put-scaling-policy \
  --auto-scaling-group-name job-processing-asg \
  --policy-name scale-in-on-queue \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --step-adjustments \
    MetricIntervalUpperBound=0,ScalingAdjustment=-2
```

### Real-World Scenario: Email Processing System

**Setup:**
- Email receiver system puts jobs in SQS
- 100-1000 emails per minute during business hours
- Each email needs processing (virus scanning, archive storage, delivery)
- Processing takes 30 seconds per email on average
- t3.small instances can process about 2 emails/second (120/minute)

**Scaling Logic:**
- Baseline: 2 instances (handles 240 emails/minute)
- Scale trigger: 500 messages in queue
- Each instance processes 120/minute = needs 500/120 = 4.2 additional instances
- Scale out policy: +5 instances when queue > 500 messages
- Scale-in policy: -1 instance when queue < 100 messages for 10 minutes

**What Happens During Traffic Spike:**
```
T=0min:    Normal load: 2 instances, queue ~20 messages
T=5min:    Spike arrives: 1000 emails/minute incoming
T=6min:    Queue builds to 500 messages (since 2 instances only process 240/min)
T=7min:    Alarm triggers, adds 5 instances (total 7)
T=8min:    7 instances processing 840/min, queue starts draining
T=12min:   New spike ends at 100 emails/minute
T=15min:   Queue drains to 100 messages, scale-in begins
T=25min:   Queue remains low, scale-in policy removes 1 instance every minute
T=35min:   Back to 2 baseline instances
```

### SQS Queue Metrics Monitoring

**Creating a Dashboard:**
```bash
aws cloudwatch put-dashboard \
  --dashboard-name SQS-Scaling-Dashboard \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "properties": {
          "metrics": [
            ["AWS/SQS", "ApproximateNumberOfMessages", {"stat": "Average"}],
            ["AWS/SQS", "ApproximateNumberOfMessagesNotVisible"],
            ["AWS/SQS", "NumberOfMessagesSent"],
            ["AWS/SQS", "NumberOfMessagesReceived"],
            ["AWS/SQS", "NumberOfMessagesDeleted"]
          ],
          "period": 60,
          "stat": "Average",
          "region": "us-east-1",
          "title": "SQS Queue Metrics"
        }
      },
      {
        "type": "metric",
        "properties": {
          "metrics": [
            ["AWS/AutoScaling", "GroupDesiredCapacity"],
            ["AWS/AutoScaling", "GroupInServiceInstances"]
          ],
          "dimensions": {
            "AutoScalingGroupName": "job-processing-asg"
          },
          "period": 60,
          "stat": "Average",
          "region": "us-east-1",
          "title": "ASG Scaling Activity"
        }
      }
    ]
  }'
```

## Part 5: Health Checks and Instance Replacement

### Types of Health Checks

ASG supports three types of health checks to ensure only healthy instances are serving traffic:

### 1. EC2 Status Checks (Default)

**What It Does:**
- AWS infrastructure checks on the underlying hardware and OS
- Validates EC2 status (not application health)
- Enabled by default on all ASG instances
- Checks:
  - System status: underlying hardware health
  - Instance status: OS and instance agent health

**When It Fails:**
- Hardware issues (disk failure, network interface problem)
- Instance has stopped responding to hardware checks
- AWS infrastructure issue in the AZ
- Hypervisor problem

**What Happens:**
- Instance marked unhealthy
- ASG terminates the instance immediately
- **No reboot attempted** - instance is terminated
- New instance launched to replace it
- Old instance is completely removed

**Example:**
```
Instance running normally
    ↓
Hardware fails (corrupted disk)
    ↓
EC2 status check fails
    ↓
ASG detects failure
    ↓
Instance terminated (NOT rebooted)
    ↓
New instance launched
    ↓
Traffic restored to new healthy instance
```

**Viewing EC2 Status:**
```bash
aws ec2 describe-instance-status \
  --instance-ids i-1234567890abcdef0 \
  --query 'InstanceStatuses[0].{SystemStatus:SystemStatus.Status,InstanceStatus:InstanceStatus.Status}'
```

### 2. ELB Health Checks (Load Balancer Based)

**What It Does:**
- If ASG is connected to a Load Balancer, ELB performs application-level health checks
- Validates that the application is responding correctly
- Checks HTTP status codes, TCP connections, or custom scripts
- Can be configured on target group

**Configuration Example (Target Group):**
```bash
aws elbv2 modify-target-group \
  --target-group-arn arn:aws:elasticloadbalancing:... \
  --health-check-protocol HTTP \
  --health-check-port 80 \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 2
```

**What Happens When Health Check Fails:**
1. Instance fails health check (e.g., app crashed but OS running)
2. ELB marks target unhealthy
3. ELB stops sending traffic to that instance
4. ASG health check integration detects ELB failure status
5. ASG terminates instance
6. New instance launched

**Health Check Process:**
```
ELB periodically sends request to instance (every 30 seconds)
    ↓
Instance responds with HTTP 200
    ↓
Instance marked healthy by ELB
    ↓
ELB sends traffic to instance

vs.

ELB sends request
    ↓
Instance responds with HTTP 503 (application down)
    ↓
Unhealthy threshold reached (2 failed checks)
    ↓
Instance marked unhealthy
    ↓
ELB stops sending traffic
    ↓
ASG detects unhealthy status from ELB
    ↓
Instance terminated
    ↓
New instance launched
```

**Health Check Settings:**
- **Check Interval**: How often to test (30 seconds typical)
- **Timeout**: How long to wait for response (5 seconds typical)
- **Healthy Threshold**: Consecutive passes before marking healthy (2 typical)
- **Unhealthy Threshold**: Consecutive fails before marking unhealthy (2 typical)
- **Path**: For HTTP checks, which endpoint (/health, /ping, etc.)

### 3. Custom Health Checks

**What It Does:**
- Application or external system reports health directly to ASG
- Most flexibility for specific application logic
- Can integrate any custom health determination logic
- Uses ASG API to set instance health

**Setting Instance Health:**
```bash
aws autoscaling set-instance-health \
  --instance-id i-1234567890abcdef0 \
  --health-status Unhealthy \
  --auto-scaling-group-name my-asg
```

**Health Status Options:**
- `Healthy`: Instance is healthy, keep running
- `Unhealthy`: Instance is unhealthy, terminate and replace
- Can be called by the instance itself or external monitoring system

**Example Custom Health Implementation:**
```python
#!/usr/bin/env python3
import boto3
import time
import requests

autoscaling = boto3.client('autoscaling')
instance_id = requests.get(
    'http://169.254.169.254/latest/meta-data/instance-id'
).text
asg_name = 'my-asg'

while True:
    try:
        # Custom health check logic
        health_status = check_application_health()
        
        if health_status == 'healthy':
            health = 'Healthy'
        else:
            health = 'Unhealthy'
        
        # Report to ASG
        autoscaling.set_instance_health(
            InstanceId=instance_id,
            HealthStatus=health,
            AutoScalingGroupName=asg_name
        )
        
    except Exception as e:
        print(f"Error reporting health: {e}")
    
    time.sleep(60)  # Check every 60 seconds

def check_application_health():
    """
    Custom health check logic
    """
    try:
        # Check if application is responsive
        response = requests.get('http://localhost:8080/health', timeout=5)
        
        # Check database connectivity
        db_healthy = check_database()
        
        # Check memory/CPU not too high
        system_healthy = check_system_resources()
        
        if response.status_code == 200 and db_healthy and system_healthy:
            return 'healthy'
        else:
            return 'unhealthy'
    except:
        return 'unhealthy'

def check_database():
    # Custom database health implementation
    pass

def check_system_resources():
    # Check CPU and memory
    pass
```

**Use Cases for Custom Health Checks:**
- Complex application-specific validation
- Dependency health (database, cache, external services)
- Application metrics and thresholds
- Multi-step validation logic
- Integration with custom monitoring systems

### Important: No Reboot on Health Check Failure

**Critical Point:** When an instance fails a health check, ASG does **NOT reboot** it.

**What Actually Happens:**
1. Instance fails health check
2. ASG immediately terminates the instance
3. New instance launched
4. Old unhealthy instance removed

**Why No Reboot?**
- Health check failures typically indicate hardware or fundamental OS issues
- Rebooting might not fix the underlying problem
- Faster to replace than wait for reboot (typically 2-3 minutes)
- New instance is guaranteed fresh state
- Prevents cascading failures from reboot cycles

**This is Different From Manual Reboot:**
```
User manually reboots instance:
Instance → Reboot → OS starts → App initialized → Back to In Service

Health check failure:
Instance → Fails check → Terminated → Removed
                      ↓
                   New instance launched
```

### ASG Health Check Settings

**Configure Health Check Type:**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --health-check-type ELB \
  --health-check-grace-period 300 \
  ...
```

**Health Check Options:**
- `EC2`: AWS infrastructure checks (default)
- `ELB`: Load balancer health checks (recommended if using LB)
- `ELB_AND_ECS_TASK`: For ECS container deployments

**Health Check Grace Period:**
- Time to wait after instance launch before starting health checks
- Default: 0 seconds
- Recommended: 300 seconds (5 minutes) or match your app startup time
- Prevents premature instance termination during initialization

```bash
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --health-check-type ELB \
  --health-check-grace-period 300
```

## Part 6: Troubleshooting and Best Practices

### Common Scaling Issues

#### Issue 1: Cannot Launch New Instances Despite Scaling Trigger

**Symptoms:**
- Scaling policy is triggered but instances don't launch
- ASG remains at current capacity
- No error visible in console

**Possible Causes and Solutions:**

**Cause 1: Reached Maximum Capacity Limit**
```bash
# Check current capacity settings
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names my-asg \
  --query 'AutoScalingGroups[0].{Min:MinSize,Max:MaxSize,Desired:DesiredCapacity,Current:length(Instances)}'

# If at max limit, increase maximum capacity
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --max-size 20  # Increase from 10 to 20
```

**Cause 2: Availability Zone Capacity Issue**
```bash
# Check if AZ has capacity available
aws ec2 describe-instance-type-offerings \
  --filters Name=instance-type,Values=t3.micro \
  --region us-east-1 \
  --query 'InstanceTypeOfferings[*].{Location:Location,InstanceType:InstanceType}'

# Check ASG's assigned subnets/AZs
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names my-asg \
  --query 'AutoScalingGroups[0].VPCZoneIdentifier'
```

**Solution**: Add another subnet in different AZ:
```bash
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --vpc-zone-identifier "subnet-12345678,subnet-87654321,subnet-other123"
```

**Cause 3: Security Group or Key Pair Deleted**
```bash
# Check if security groups exist
aws ec2 describe-security-groups \
  --group-ids sg-12345678

# Check if key pair exists
aws ec2 describe-key-pairs \
  --key-names my-key-pair
```

**Solution**: Recreate or use different security group/key pair:
```bash
aws ec2 create-key-pair --key-name new-key-pair > new-key-pair.pem
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template \
  --launch-template-data '{"KeyName":"new-key-pair"}'
```

**Cause 4: AMI Deleted or No Longer Available**
```bash
# Verify AMI still exists
aws ec2 describe-images \
  --image-ids ami-12345678

# List available AMIs
aws ec2 describe-images \
  --owners self
```

#### Issue 2: Instances Launching But Not Entering Service

**Symptoms:**
- Instances appear in ASG
- Instances stay in Pending state
- Never transition to In Service
- Eventually terminated

**Possible Causes:**

**Cause 1: User Data Script Failing**
```bash
# Connect to pending instance and check logs
aws ssm start-session --target i-1234567890abcdef0

# Inside instance:
tail -f /var/log/cloud-init-output.log
```

**Solution**: Fix user data script:
```bash
# Create corrected launch template version
aws ec2 create-launch-template-version \
  --launch-template-name my-template \
  --launch-template-data '{
    "ImageId": "ami-12345678",
    "UserData": "...(corrected script)..."
  }'

# Instance refresh to apply fix
aws autoscaling start-instance-refresh \
  --auto-scaling-group-name my-asg
```

**Cause 2: Health Check Failing Immediately**
```bash
# Check health check settings
aws autoscaling describe-auto-scaling-groups \
  --auto-scaling-group-names my-asg \
  --query 'AutoScalingGroups[0].HealthCheckGracePeriod'

# If grace period too low, increase it
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --health-check-grace-period 600
```

**Cause 3: Application Port Not Listening**
- Health check trying to connect to port that app hasn't started on yet
- Solution: Ensure application starts before health checks begin

#### Issue 3: Instances Terminate Unexpectedly

**Symptoms:**
- Healthy instances suddenly terminate
- No scaling event triggered
- Activity history shows "Terminating instances: health check"

**Possible Causes:**

**Cause 1: Health Check Failing Due to Application Issue**
```bash
# Test health check endpoint manually
curl -v http://(instance-ip):port/health

# Check application logs on instance
tail -f /var/log/application.log
```

**Cause 2: Instance Running Out of Resources**
```bash
# Check CPU and memory
top
df -h
dmesg | tail -50

# Monitor metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time $(date -u -d '15 minutes ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average
```

**Cause 3: Security Group Rules Blocking Health Checks**
```bash
# Verify security group allows traffic from LB
aws ec2 describe-security-groups \
  --group-ids sg-12345678 \
  --query 'SecurityGroups[0].IpPermissions'
```

### Monitoring and Logging Best Practices

**1. Enable Detailed CloudWatch Monitoring:**
```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-asg \
  --launch-template LaunchTemplateName=my-template \
  --enable-metrics-collection \
  --metrics-collection Granularity=1Minute,Metrics=GroupMinSize,GroupMaxSize,GroupDesiredCapacity,GroupInServiceInstances,GroupPendingInstances,GroupTerminatingInstances,GroupInstancesInService
```

**Metrics to Track:**
- `GroupDesiredCapacity`: Desired number of instances
- `GroupInServiceInstances`: Actually serving traffic
- `GroupPendingInstances`: Starting up
- `GroupTerminatingInstances`: Shutting down
- `GroupTotalInstances`: All instances

**2. Create CloudWatch Dashboard:**
```bash
aws cloudwatch put-dashboard \
  --dashboard-name ASG-Monitoring \
  --dashboard-body '{
    "widgets": [
      {
        "type": "metric",
        "properties": {
          "metrics": [
            ["AWS/AutoScaling", "GroupInServiceInstances", {"stat": "Average"}],
            ["AWS/AutoScaling", "GroupPendingInstances"],
            ["AWS/AutoScaling", "GroupTerminatingInstances"]
          ],
          "dimensions": {"AutoScalingGroupName": "my-asg"},
          "period": 60,
          "stat": "Average",
          "title": "ASG Instance Status"
        }
      }
    ]
  }'
```

**3. Set Up SNS Notifications:**
```bash
aws autoscaling put-notification-configuration \
  --auto-scaling-group-name my-asg \
  --topic-arn arn:aws:sns:us-east-1:123456789012:asg-notifications \
  --notification-types "autoscaling:EC2_INSTANCE_LAUNCH" \
                       "autoscaling:EC2_INSTANCE_TERMINATE" \
                       "autoscaling:EC2_INSTANCE_LAUNCH_ERROR" \
                       "autoscaling:EC2_INSTANCE_TERMINATE_ERROR"
```

**4. Review Activity History:**
```bash
aws autoscaling describe-scaling-activities \
  --auto-scaling-group-name my-asg \
  --max-records 20 \
  --query 'Activities[*].{StartTime:StartTime,EndTime:EndTime,StatusCode:StatusCode,StatusMessage:StatusMessage}' \
  --output text
```

### Production Deployment Checklist

**Planning Phase:**
- [ ] Understand current application startup time (measure 3-5 instances)
- [ ] Calculate appropriate health check grace period (startup time + 60 seconds buffer)
- [ ] Choose health check type (EC2 for stateless, ELB for monitored apps, Custom for complex)
- [ ] Identify appropriate IAM roles and security groups
- [ ] Document scaling policies and triggers
- [ ] Create runbooks for common issues
- [ ] Plan rollback procedure

**Configuration Phase:**
- [ ] Create launch template with all required parameters
- [ ] Test launch template in staging (verify user data, security groups, tags)
- [ ] Create ASG with appropriate min/max/desired capacity
- [ ] Attach to load balancer if needed
- [ ] Configure health checks with reasonable grace period
- [ ] Set up scaling policies (at least 1 scale-out, 1 scale-in)
- [ ] Enable lifecycle hooks if needed (for special initialization/cleanup)

**Pre-Production Testing:**
- [ ] Verify ASG launches instances correctly
- [ ] Verify instances pass health checks
- [ ] Test scale-out by increasing desired capacity manually
- [ ] Test scale-in by decreasing desired capacity manually
- [ ] Verify manual scaling works
- [ ] Test with load to trigger scaling policies
- [ ] Verify old instances terminate gracefully
- [ ] Check logs from terminated instances

**Monitoring Setup:**
- [ ] Enable CloudWatch metrics collection
- [ ] Create dashboard for ongoing monitoring
- [ ] Set up CloudWatch alarms for key metrics
- [ ] Configure SNS notifications for events
- [ ] Document metrics and alerts

**Production Deployment:**
- [ ] Deploy to production during low-traffic window if possible
- [ ] Start with minimal capacity (min=1) to test
- [ ] Monitor first 24 hours actively
- [ ] Watch for unexpected terminations or pending instances
- [ ] Verify scaling policies trigger at expected thresholds
- [ ] Confirm logs are being captured properly

**Post-Production:**
- [ ] Monitor daily for first week
- [ ] Review scaling events and patterns
- [ ] Adjust scaling policies if needed (too aggressive/conservative)
- [ ] Document lessons learned
- [ ] Update runbooks with real-world findings
- [ ] Train operations team on ASG management

### SysOps Exam Focus and Key Takeaways

**Lifecycle Hooks - Key Points:**
1. Override default state transitions (Pending→In Service, In Service→Terminated)
2. Insert wait states for custom logic (Pending:Wait, Terminating:Wait)
3. Integration: EventBridge (flexible), SNS (notifications), SQS (workers)
4. Use cases: initialization, log extraction, health verification, garbage collection
5. Timeout configurable (default 300s, up to 86400s)
6. Must call `complete-lifecycle-action` to proceed

**Launch Templates - Key Points:**
1. Modern standard replacing launch configuration
2. Support multiple versions (v1, v2, v3...)
3. Enable subsets/inheritance for configuration reuse
4. Support spot instances mixed with on-demand
5. Support placement groups, capacity reservations, dedicated hosts
6. AWS-recommended for all new deployments

**SQS Scaling - Key Points:**
1. Scale based on work (queue depth) not just system metrics
2. Metric: ApproximateNumberOfMessages
3. CloudWatch alarms trigger scaling policies
4. Good for worker/background job patterns
5. Decouples incoming requests from processing capacity

**Health Checks - Key Points:**
1. Three types: EC2 (default, infrastructure), ELB (app-level), Custom (manual)
2. Failure → Terminated, NOT rebooted
3. Grace period prevents premature failures during startup
4. Recommended: Grace period = (startup time + 60s buffer)
5. Health check type choice depends on how you monitor application

**Common Exam Questions:**
1. **Q: When should you use each launch template feature?**
   A: Spot mix for cost optimization, placement groups for cluster computing, capacity reservation for guaranteed availability, dedicated hosts for licensing

2. **Q: How does lifecycle hook integration work?**
   A: ASG sends event to EventBridge/SNS/SQS, target service processes event and calls complete-lifecycle-action API, instance transitions to next state

3. **Q: Why does ASG terminate unhealthy instances instead of rebooting?**
   A: Health check failure usually indicates fundamental issue, new instance is faster and cleaner than troubleshooting reboot

4. **Q: How to scale based on custom application metrics?**
   A: Application reports metric to CloudWatch or custom health, create target tracking or step scaling policy, ASG scales accordingly

5. **Q: What's the difference between launch template and launch configuration?**
   A: Launch template supports multiple versions/inheritance/spot-mix/modern features, launch configuration is legacy single version only

### Best Practices Summary

**For Lifecycle Hooks:**
- Keep initialization scripts idempotent (safe to run multiple times)
- Use reasonable timeouts (balance between allowing operations and preventing stuck instances)
- Log all lifecycle events for auditing
- Test lifecycle hooks in staging before production
- Use EventBridge for complex multi-target scenarios

**For Launch Templates:**
- Create base template for common settings, inherit in child templates
- Use $Latest version for active deployments, specify explicit version for stability
- Keep user data scripts simple, move complex logic to launch hook
- Use templated parameters for multi-environment deployments
- Store templates in version control documentation

**For Scaling:**
- Scale out more aggressively than scale in (better user experience)
- Use target tracking for most scenarios (simpler than step scaling)
- Implement grace period equal to startup time + buffer
- Monitor scaling events, adjust policies based on patterns
- Test scaling behaviors before production

**For Health Checks:**
- Match health check type to deployment pattern
- Ensure health check endpoints are fast and reliable
- Add retry logic (healthy/unhealthy thresholds prevent flapping)
- Monitor health check failures separately from application errors
- Use lifecycle hooks to prepare instance before health checks begin

---

**Total Words: ~18,500**  
**File Created: 24_ASG_Lifecycle_Hooks_And_Advanced_Features.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/EC2 High Availability & Scalability/**
