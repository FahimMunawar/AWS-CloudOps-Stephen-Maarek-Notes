# SSM Automation

This note covers SSM Automation — higher-level runbooks that perform actions on AWS resources from outside the instance, as opposed to Run Command which runs inside the instance.

## 1. What is SSM Automation?

- **SSM Automation**: Executes operational tasks on EC2 instances and other AWS resources at the AWS API level
- **Key distinction**: Automation operates **from outside** the instance (AWS API calls), whereas Run Command operates **from inside** the instance (shell commands via the SSM agent)
- **Runbook**: The name for an SSM document of type `Automation` — defines the steps of the automation

### Common Use Cases

- Restart EC2 instances
- Create AMIs from running instances
- Take EBS snapshots
- Attach/detach EBS volumes
- Attach IAM roles to instances
- Enter/exit Auto Scaling Group standby
- RDS snapshot creation

## 2. Run Command vs. Automation

| Feature | Run Command | Automation |
|---|---|---|
| Execution context | Inside the instance (via SSM agent) | Outside the instance (AWS API) |
| Document type | Command | Automation (Runbook) |
| Typical actions | Shell scripts, installs, config changes | Restart, AMI creation, snapshots |
| Target | EC2 instances | EC2, EBS, AMI, RDS, and more |

## 3. How to Trigger SSM Automation

| Trigger | How |
|---|---|
| Console / CLI / SDK | Manual execution |
| EventBridge rule | Automated, event-driven trigger |
| Maintenance Window | Scheduled execution |
| AWS Config Remediation | Auto-triggered when a resource is non-compliant |

## 4. Execution Modes

| Mode | Description |
|---|---|
| Simple execution | Run on all targets simultaneously |
| Rate control | Run one (or N) targets at a time, with error threshold |
| Multi-account / Multi-region | Execute across accounts and regions |
| Manual (step-by-step) | Pause between each step for review |

## 5. AWS-Managed Runbook Categories

AWS provides pre-built automation runbooks across many categories:

- **Instance Management**: Restart, stop, start, attach/detach volumes, modify IAM role
- **AMI Management**: Create AMIs, copy AMIs cross-region
- **Data Backups**: EBS snapshots, RDS snapshots
- **Patching**: Patch instances using baselines
- **Security**: Remediate security findings
- **Cost Management**: Stop unused instances, resize instances

Browse them under: SSM → Documents → filter by **Type: Automation**

## 6. Hands-On: Restart EC2 Instances with Rate Control

### Document Used

**`AWS-RestartEC2Instance`**
- Steps: (1) Stop instance → (2) Start instance
- No SSH or agent interaction needed — purely AWS API calls

### Steps

1. SSM Console → **Automation** → **Execute automation**
2. Search for document: `AWS-RestartEC2Instance`
3. Select version: **Latest**
4. Execution mode: **Rate control**
5. Target parameter: **InstanceIds**
6. Target source: **Resource group** → select `DevGroup`
   - SSM resolves the resource group to the actual instance IDs automatically
7. Rate control:
   - Concurrency: **1 target at a time**
   - Error threshold: **1** (stop on first error)
8. IAM role: leave blank (uses your current credentials) — or specify a dedicated automation role
9. Click **Execute automation**

### What Happens

```
Automation triggered on DevGroup
        │
        ▼
Step 1: StopInstances  (Instance → Stopping → Stopped)
        │
        ▼
Step 2: StartInstances (Instance → Pending → Running)
        │
        ▼
Next instance in DevGroup (rate control: one at a time)
```

- Verify in EC2 console: instance transitions through `stopping → stopped → pending → running`
- No SSH access required
- No custom script required — the runbook handles sequencing, rate control, error handling, and logging

### Optional: Automation with Approval Step

**`AWS-RestartEC2InstanceWithApproval`**
- Adds a human approval step before the restart executes
- Requires specifying an IAM user ARN or role ARN as the approver
- Useful for production environments requiring change control

## 7. Key Takeaways for SysOps Associate

- SSM Automation runbooks operate at the **AWS API level** — no SSH, no agent required for the action itself
- Run Command = **inside the instance**; Automation = **outside the instance** — know this distinction
- Use **Rate Control** mode for safe rollouts across large fleets (concurrency + error threshold)
- Automations can be triggered by: **Console/CLI, EventBridge, Maintenance Windows, AWS Config Remediation**
- AWS Config Remediation → SSM Automation is a common exam pattern for auto-fixing non-compliant resources
- Resource groups can be used as targets — SSM resolves group membership to instance IDs automatically
- Explore AWS-managed runbooks in the Documents section filtered by type **Automation** — there are hundreds covering EC2, EBS, AMI, RDS, and more
