# Why Use SSM Run Command for Fleet Patching?

## The Scenario
You need to apply patches efficiently to a fleet of EC2 instances.

AWS Systems Manager (SSM) Run Command is a secure and automated way to remotely manage your servers at scale. Instead of logging into each instance individually via SSH or RDP, you can send commands to hundreds or thousands of instances simultaneously from a single interface. 

How it Works

Think of Run Command as a "push-button" administrative tool. You define a command or script (using an SSM Document), choose your targets, and AWS takes care of the rest. 
No SSH/RDP Required: You don’t need to open port 22 (Linux) or 3389 (Windows), nor do you need to manage SSH keys.

Security & Auditing: Every command executed is logged in AWS CloudTrail and Amazon CloudWatch, giving you a full audit trail of who did what and when.

Broad Support: It works on EC2 instances, on-premises servers, and even virtual machines in other cloud environments. 

## Why Run Command is the Answer

**SSM Run Command** is the correct choice because:

- Executes commands **across multiple EC2 instances simultaneously**
- **No SSH required** — the SSM agent on each instance runs the command
- No need to open port 22 or manage key pairs
- Supports **rate control** — patch instances progressively (e.g., 10 at a time) to avoid fleet-wide downtime
- Built-in **error threshold** — stops the rollout if too many instances fail
- Output logged to **CloudWatch Logs** or **S3** for auditing

## vs. Other Options

| Option | Why Not |
|---|---|
| SSH into each instance manually | Does not scale — one instance at a time |
| EC2 Instance Connect | Browser-based single session — not for bulk operations |
| SSM Session Manager | Interactive shell — not designed for scripted bulk execution |
| SSM Automation | Operates at AWS API level (e.g., restart, AMI) — not for running shell scripts inside instances |

## The Command to Use

To patch instances via Run Command, use the document:

```
AWS-RunPatchBaseline
```

This applies patches according to the assigned patch baseline for each instance.
