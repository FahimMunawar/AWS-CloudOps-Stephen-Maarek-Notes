# Systems Manager Overview

This note covers AWS Systems Manager (SSM), a suite of tools for managing EC2 instances and on-premises servers at scale.

## 1. What is Systems Manager?

- **AWS Systems Manager (SSM)**: A suite of tools to manage a fleet of EC2 instances and on-premises servers at scale
- **Purpose**: Provides insights into infrastructure state, detects problems, automates patching, and enforces compliance
- **Scope**: Works for both **Windows** and **Linux** operating systems
- **Integrations**: Fully integrated with CloudWatch metrics/dashboards and AWS Config
- **Cost**: Free service — you only pay for the underlying resources it creates or uses

## 2. Key Use Cases

- **Patching**: Automate OS and application patching across a fleet of instances
- **Automation**: Run automated operations at scale across many instances
- **Compliance**: Detect and enforce desired configuration state
- **Operations**: Centralized visibility and control of your infrastructure

## 3. SSM Feature Categories

### Node Tools
| Feature | Purpose |
|---|---|
| Fleet Manager | View and manage all nodes in one place |
| Compliance | Track patch and configuration compliance |
| Inventory | Collect metadata about instances and software |
| Hybrid Activations | Register on-premises servers with SSM |
| Session Manager | Browser-based shell without SSH or bastion hosts |
| Run Command | Execute commands remotely at scale |
| State Manager | Define and maintain desired configuration state |
| Patch Manager | Automate patching of instances |
| Distributor | Package and distribute software to instances |

### Change Management
- **Automation**: Run automated runbooks (pre-defined workflows)
- **Change Calendar**: Block or allow changes during specific time windows
- **Maintenance Windows**: Define scheduled windows for running tasks
- **Documents**: Define the actions SSM performs (SSM Documents / runbooks)
- **Quick Setup**: Simplified setup for common use cases

### Application Tools
- **Application Manager**: Manage application resources grouped logically
- **AppConfig**: Deploy application configurations safely with validation
- **Parameter Store**: Centralized, secure storage for configuration data and secrets

### Operations Tools
- **Explorer**: Aggregated view of operational data across AWS accounts and regions
- **OpsCenter**: Centralized location to view and resolve operational issues
- **CloudWatch Dashboard**: Integrated monitoring dashboards

## 4. How SSM Works

- **SSM Agent**: A lightweight agent that must be installed on every instance or server you want SSM to manage
- **Amazon Linux 2 AMI**: Comes with SSM agent **pre-installed** (this is why it is commonly used)
- **Some Ubuntu AMIs**: Also come with the SSM agent pre-installed
- **Other AMIs**: Agent can be manually installed with a few commands
- **IAM Role**: EC2 instances must have an IAM role with permissions to communicate with the SSM service

### Architecture

```
EC2 Instance (SSM Agent) ──┐
EC2 Instance (SSM Agent) ──┼──► AWS Systems Manager Service
On-Premises VM (SSM Agent) ┘         │
                                      ▼
                          Patching, Automation, Compliance,
                          Session Manager, Run Command, etc.
```

## 5. Troubleshooting SSM Registration

If an instance is not appearing in Systems Manager, check:

1. **SSM Agent not running**: Verify the agent is installed and the service is active
2. **Wrong or missing IAM role**: The EC2 instance must have an IAM role with `AmazonSSMManagedInstanceCore` policy (or equivalent permissions)
3. **Network connectivity**: Instance must be able to reach SSM service endpoints (via internet or VPC endpoints)

## 6. Key Takeaways for SysOps Associate

- SSM is a **free suite of tools** used to manage and automate operations across EC2 and on-premises servers
- The **SSM Agent** must be installed and running on every managed node
- **Amazon Linux 2** comes with the SSM agent pre-installed — prefer it when using SSM
- Two most common reasons an instance is not registered with SSM: **agent not running** or **missing IAM role**
- SSM is the go-to answer for exam questions about **patching at scale** and **running commands across a fleet**
- Key exam-relevant features: **Session Manager**, **Run Command**, **Patch Manager**, **State Manager**, **Parameter Store**, **Automation**
- Works for **both EC2 and on-premises** servers via the SSM agent
