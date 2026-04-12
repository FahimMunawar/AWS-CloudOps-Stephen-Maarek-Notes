# 22. CloudFormation cfn-signal Failure & Debugging

## Overview

**The Problem: WaitCondition Did Not Receive Required Signals**

A common exam scenario: stack creation hangs or fails because the `WaitCondition` never received the expected number of signals from the EC2 instance. This can happen for several reasons and requires a specific debugging approach.

---

## Common Causes of Missing Signals

| Cause | Explanation | Fix |
|-------|-------------|-----|
| **Helper scripts not installed** | AMI doesn't include `aws-cfn-bootstrap` | Install via UserData: `yum install -y aws-cfn-bootstrap` |
| **cfn-init failure** | Configuration error causes non-zero exit code → cfn-signal sends failure | Check `/var/log/cfn-init.log` and `/var/log/cfn-init-cmd.log` |
| **cfn-signal failure** | cfn-signal itself fails to execute or send | Check `/var/log/cloud-init-output.log` |
| **No internet access** | EC2 in private subnet can't reach CloudFormation service endpoint | Add NAT Gateway or VPC endpoint; test with `curl amazon.com` |
| **Timeout exceeded** | Script takes longer than `Timeout` in `CreationPolicy` | Increase timeout value (e.g. `PT10M`) |
| **Wrong resource name** | `-r` flag in cfn-signal doesn't match WaitCondition logical ID | Verify exact logical ID in template |

---

## How a Failure Propagates

```
cfn-init command exits with code 1 (non-zero)
        ↓
INIT_STATUS=$? captures the error code (1)
        ↓
cfn-signal -e $INIT_STATUS sends exit code 1 to WaitCondition
        ↓
WaitCondition receives FAILURE signal
        ↓
WaitCondition → CREATE_FAILED
        ↓
CloudFormation stack → ROLLBACK_IN_PROGRESS
        ↓
All resources deleted (EC2, SecurityGroup, WaitCondition)
```

**Example: Intentional failure in cfn-init metadata:**
```yaml
commands:
  01_fail:
    command: "echo boom && exit 1"
    # exit 1 = non-zero = failure
    # cfn-init exits with code 1
    # cfn-signal receives code 1 → stack fails
```

---

## The Debugging Problem

**Default Behavior (Rollback on Failure):**
- Stack fails → CloudFormation immediately deletes all resources including the EC2 instance
- You lose access to the instance and its logs before you can investigate

**Solution: Disable Rollback on Failure**

When creating the stack, under **Stack failure options**, change from:

> ~~Roll back all stack resources~~ (default)

to:

> **Preserve successfully provisioned resources**

This keeps the EC2 instance alive after failure so you can SSH in and inspect logs.

---

## Debugging Workflow

```
1. Create stack with "Preserve successfully provisioned resources"
        ↓
2. Stack fails → EC2 instance remains running
        ↓
3. SSH into instance (or use EC2 Instance Connect)
        ↓
4. Check logs:
   - /var/log/cloud-init-output.log  → UserData + cfn-init call output
   - /var/log/cfn-init.log           → cfn-init execution log
   - /var/log/cfn-init-cmd.log       → command outputs from cfn-init
        ↓
5. Identify root cause and fix template
        ↓
6. Delete preserved stack manually, redeploy fixed version
```

**Connectivity Test (for private subnet issues):**
```bash
# Run on EC2 instance to verify outbound internet access
curl -I https://www.amazon.com
# If this fails, EC2 cannot reach CloudFormation service endpoint
```

---

## Stack Failure Options — Console Setting

| Setting | Behavior | Use When |
|---------|----------|----------|
| **Roll back all stack resources** | Deletes everything on failure (default) | Production deployments |
| **Preserve successfully provisioned resources** | Keeps created resources intact | Development / debugging |

> Set to **Preserve** during development cycles, revert to **Roll back** for production.

---

## Best Practices

✓ **Always install aws-cfn-bootstrap explicitly** — Don't assume the AMI has it  
✓ **Verify AMI includes helper scripts** — Especially for custom or community AMIs  
✓ **Test internet connectivity first** — Private subnets need NAT Gateway or VPC endpoint  
✓ **Use Preserve on failure during dev** — Keep EC2 alive for log inspection  
✓ **Check all three log files** — Each reveals a different layer of failure  
✓ **Match resource name exactly** — `-r` in cfn-signal must match WaitCondition logical ID  
✓ **Set generous timeouts in dev** — Short timeouts hide root causes; use `PT15M` while debugging  
✓ **Revert to rollback for production** — Preserve is for debugging only, not live stacks  

---

## SysOps Exam Focus

**Q1: "WaitCondition did not receive the required number of signals. What should you check first?"**
- A) Security group inbound rules
- B) Whether the AMI has CloudFormation helper scripts installed and EC2 has internet access
- C) The CloudFormation service quotas
- D) The IAM role attached to CloudFormation
- **Answer: B** — Missing helper scripts and no internet access are the two most common root causes

**Q2: "To debug a failed EC2 cfn-init, you need to SSH into the instance. What must you do when creating the stack?"**
- A) Enable termination protection
- B) Set stack failure option to "Preserve successfully provisioned resources"
- C) Disable deletion policy
- D) Use a public subnet
- **Answer: B** — Without this, CloudFormation deletes the instance on failure before you can inspect it

**Q3: "Your EC2 is in a private subnet and cfn-signal is not being received. What is the likely cause?"**
- A) cfn-init is not installed
- B) The security group blocks port 443
- C) EC2 has no outbound internet access to reach the CloudFormation service endpoint
- D) The WaitCondition timeout is too short
- **Answer: C** — Private subnet instances need NAT Gateway or VPC endpoint to reach CloudFormation

**Q4: "Which log file shows the output of individual commands run by cfn-init?"**
- A) `/var/log/cfn-init.log`
- B) `/var/log/cfn-init-cmd.log`
- C) `/var/log/cloud-init-output.log`
- D) `/var/log/syslog`
- **Answer: B** — `cfn-init-cmd.log` contains output from each command block

**Q5: "A cfn-init command exits with code 1. What happens to the stack?"**
- A) Stack succeeds, cfn-signal is ignored
- B) cfn-signal sends exit code 1 → WaitCondition fails → Stack rolls back
- C) Stack waits for timeout then succeeds
- D) Only the failed resource is replaced
- **Answer: B** — Non-zero exit code propagates through cfn-signal to fail the WaitCondition and roll back the stack

**Q6: "Default CloudFormation behavior when stack creation fails is:"**
- A) Preserve all resources for debugging
- B) Pause and wait for manual input
- C) Roll back and delete all stack resources
- D) Retry stack creation automatically
- **Answer: C** — Default is full rollback; must explicitly choose to preserve resources

---

## Three Log Files — Quick Reference

| Log File | What It Shows |
|----------|---------------|
| `/var/log/cloud-init-output.log` | Full UserData script output including cfn-init invocation |
| `/var/log/cfn-init.log` | cfn-init execution details (packages, files, services) |
| `/var/log/cfn-init-cmd.log` | Output of each individual `commands` block entry |

---

## Relationship to Previous Topics

```
19 — EC2 UserData
  └─ Problem: Stack succeeds even if script fails

20 — cfn-init
  └─ Solution: Declarative config via metadata block

21 — cfn-signal + WaitCondition
  └─ Solution: Stack fails if configuration fails

22 — cfn-signal Failure Debugging  ← YOU ARE HERE
  └─ Solution: How to investigate WHY cfn-signal failed
```

---

**File: 22_CloudFormation_cfn-signal_Failure_Debugging.md**
**Status: SysOps-focused, exam-ready, concise format**
