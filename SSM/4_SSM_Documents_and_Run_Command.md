# SSM Documents and Run Command

This note covers SSM Documents — the core building blocks of Systems Manager — and how to use Run Command to execute them across a fleet of EC2 instances without SSH.

## 1. What are SSM Documents?

- **SSM Documents**: JSON or YAML files that define parameters and actions for SSM to execute
- **Analogy**: Similar in spirit to CloudFormation, but for operational tasks rather than infrastructure provisioning
- **Pre-built**: AWS provides hundreds of managed documents out of the box
- **Custom**: You can write and version your own documents

### Document Structure

```yaml
description: "Description of what the document does"
parameters:
  ParameterName:
    type: String
    default: "default value"
    description: "What this parameter does"
mainSteps:
  - action: aws:runShellScript
    name: StepName
    inputs:
      runCommand:
        - command1
        - command2
```

### Document Types

| Type | Used By |
|---|---|
| Command | Run Command, State Manager |
| Session | Session Manager |
| Automation | Automation (runbooks) |
| Policy | State Manager |

### Document Ownership

| Tab | Contents |
|---|---|
| Owned by Amazon | AWS-managed documents, auto-maintained |
| Owned by me | Custom documents you create |
| Shared with me | Documents shared by other AWS accounts |

### Documents and Other SSM Features

Documents are used across SSM features:
- **Run Command**: Execute a document across instances
- **State Manager**: Apply a document on a schedule to maintain desired state
- **Patch Manager**: Uses `AWS-ApplyPatchBaseline` document
- **Automation**: Uses automation-type documents (runbooks)
- **Parameter Store integration**: Documents can pull values from Parameter Store at runtime for dynamic behavior

## 2. Example: AWS-Managed Document

**`AWS-ApplyPatchBaseline`**
- Purpose: Scan or install patches from a patch baseline
- Platform: Windows
- Parameters: `Operation` (Scan/Install), `SnapshotId`
- Maintained by AWS — no ownership or versioning required on your part

## 3. Creating a Custom Document (Hands-On)

### Example: InstallApache Document

```yaml
schemaVersion: "2.2"
description: "Install Apache HTTP server"
parameters:
  Message:
    type: String
    default: "Hello World"
    description: "Welcome message for the web server"
mainSteps:
  - action: aws:runShellScript
    name: installApache
    inputs:
      runCommand:
        - sudo yum update -y
        - sudo yum install -y httpd
        - sudo systemctl start httpd
        - sudo systemctl enable httpd
        - echo "{{Message}} from $(hostname)" > /var/www/html/index.html
```

### Steps to Create

1. SSM Console → **Documents** → **Create document**
2. Select type: **Command or Session**
3. Name: `InstallApache`
4. Target type: `AWS::EC2::Instance`
5. Document type: **Command document**
6. Content format: **YAML**
7. Paste document content → **Create document**

## 4. SSM Run Command

### What is Run Command?

- Executes a document (script) or a single command **across a fleet of EC2 instances**
- **No SSH required** — the SSM agent on the instance executes the command
- Instances do not need inbound port 22 (or any inbound rule) open
- Fully integrated with **IAM** (who can run commands) and **CloudTrail** (audit log)

### Targeting Options

| Method | When to Use |
|---|---|
| Instance tags | Target by tag key/value (e.g., `Environment=prod`) |
| Resource groups | Use pre-built groups (from previous lecture) |
| Manual selection | Choose specific instances |

### Rate Control

Controls how aggressively the command rolls out across targets:

- **Concurrency**: How many instances to run the command on simultaneously
  - Fixed number: e.g., `10 targets at a time`
  - Percentage: e.g., `33% at a time`
- **Error threshold**: When to stop the entire run if errors occur
  - Fixed: Stop after `1` error
  - Percentage: Stop if more than `5%` of instances fail

### Output Options

| Destination | Use Case |
|---|---|
| Console (default) | Quick inspection, up to 48,000 characters |
| S3 bucket | Long-term storage, large outputs |
| CloudWatch Logs | Ongoing monitoring, searchable logs |
| SNS notification | Alerts on `InProgress`, `Success`, `Failed` status |

### Triggering Run Command

- Manually via console or AWS CLI
- **EventBridge rules** (CloudWatch Events) can trigger Run Command automatically
- **Automation runbooks** can invoke Run Command as a step

## 5. Hands-On: Install Apache on 3 Instances

### Prerequisites

- Three EC2 instances (Amazon Linux 2) with `AmazonEC2RoleForSSM` IAM role
- Security group with **HTTP port 80 inbound** open (to verify the web server afterwards)
- No SSH required — port 22 does not need to be open

### Steps

1. SSM Console → **Run Command** → **Run a command**
2. Search documents → **Owned by me** → select `InstallApache`
3. Set parameter: `Message` = `Custom Hello World`
4. Targets: select the three instances manually
5. Rate control:
   - Concurrency: **1 target at a time**
   - Error threshold: **0** (stop on first error)
6. Output: enable **CloudWatch Logs** → log group `runCommandOutput`
7. Click **Run**

### Execution Flow

```
Run Command triggered
       │
       ▼
Instance 1: In Progress → Success
       │
       ▼
Instance 2: In Progress → Success
       │
       ▼
Instance 3: In Progress → Success
```

### Verifying the Result

- Navigate to each instance's public IP in a browser
- Expected response: `Custom Hello World from <hostname>`
- Each instance returns a different hostname, confirming independent execution
- CloudWatch Logs → log group `runCommandOutput` → streams for `stdout` and `stderr` per instance

## 6. Key Takeaways for SysOps Associate

- SSM Documents are **JSON or YAML** files defining parameters and actions — they are the core unit of SSM
- AWS provides many **pre-built documents** (`AWS-ApplyPatchBaseline`, etc.) — prefer these when available
- Custom documents can be **versioned** and **shared** across accounts
- **Run Command** executes documents across a fleet with **no SSH or inbound ports needed**
- Use **rate control** (concurrency + error threshold) to safely roll out commands at scale
- Command output can go to the **console, S3, CloudWatch Logs**, and status notifications to **SNS**
- **EventBridge** can trigger Run Command automatically based on events
- Documents integrate with: Run Command, State Manager, Patch Manager, Automation, and Parameter Store
