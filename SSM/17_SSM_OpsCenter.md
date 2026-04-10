# SSM OpsCenter

This note covers SSM OpsCenter — a centralized dashboard for viewing, investigating, and remediating operational issues across AWS resources.

## 1. What is OpsCenter?

- **Purpose**: Centralize operational issues from multiple AWS services into one place for investigation and remediation
- **Goal**: Reduce **mean time to resolve (MTTR)** operational issues
- Supports EC2 instances and on-premises managed nodes

## 2. OpsItems

The core unit in OpsCenter is an **OpsItem** — a record representing an operational issue or interruption that needs investigation.

OpsItems can originate from:
- **EventBridge** events
- **CloudWatch** alarms
- **AWS Config** rule violations
- **Security Hub** findings
- **CloudTrail** logs
- **DevOps Guru** anomalies
- **SSM Incident Manager**
- **Application Insights**
- Custom sources (e.g., Lambda functions)

### Example Issue Types
- Security findings (Security Hub)
- DynamoDB table throttling (performance)
- EC2 instance failed to launch from ASG (failure)
- Orphaned EBS volumes (cost/hygiene)

## 3. How OpsCenter Works

```
Data Sources
(CloudWatch, EventBridge, Config,
Security Hub, DevOps Guru, etc.)
          │
          ▼
    OpsCenter (OpsItems)
          │
          ├── View and investigate issue
          ├── Recommended Runbooks / Automations
          └── Execute SSM Automation to remediate
```

- OpsCenter aggregates issues and surfaces **recommended SSM Runbooks** for remediation
- Remediations are executed as **SSM Automations** directly from OpsCenter

## 4. Real-World Example: Orphaned EBS Volume Cleanup

```
EventBridge scheduled rule (daily)
          │
          ▼
Lambda function scans EBS volumes
> 45 days old with no attachment
          │
          ▼
Creates OpsItem in OpsCenter
          │
          ▼
OpsCenter surfaces recommended actions:
  - Create snapshot
  - Delete snapshot
  - Delete volume
          │
          ▼
SSM Automation executes the chosen action
```

## 5. Key Takeaways for SysOps Associate

- OpsCenter centralizes operational issues as **OpsItems** from many AWS services into one place
- Data sources include: **CloudWatch, EventBridge, Config, Security Hub, DevOps Guru, SSM Incident Manager**
- OpsCenter recommends **SSM Runbooks (Automations)** to remediate each OpsItem
- The goal is to reduce **MTTR** — mean time to resolve
- Custom OpsItems can be created programmatically (e.g., from Lambda) for custom operational workflows
- Remediation actions in OpsCenter are executed as **SSM Automations** — tying OpsCenter back to the broader SSM toolset
