# SSM Automation Real-World Example

This note covers a real-world SSM Automation use case: automatically patching an AMI and rolling it out to an Auto Scaling Group without manual intervention.

## 1. The Problem

- An Auto Scaling Group (ASG) is running EC2 instances based on a **source AMI**
- The source AMI is outdated and needs OS patches applied
- All new and existing instances in the ASG should use the **patched AMI**
- Goal: Automate the entire process end-to-end using SSM Automation

## 2. Solution: Multi-Step SSM Automation

The automation orchestrates multiple AWS services in sequence, all without manual steps.

### Full Automation Flow

```
Step 1: Launch EC2 instance from source AMI
        │
        ▼
Step 2: Run Command → AWS-RunPatchBaseline
        (installs OS patches on the instance)
        │
        ▼
Step 3: Stop the patched instance
        │
        ▼
Step 4: Create AMI from stopped instance → Patched AMI
        │
        ▼
Step 5: Terminate the temporary instance
        │
        ▼
Step 6: Run Python script → Update Launch Template
        (sets new Patched AMI as the base image)
        │
        ▼
Step 7: Update ASG to use the new Launch Template version
        │
        ▼
Step 8: Trigger EC2 Instance Refresh on the ASG
        (replaces existing instances with new patched AMI)
```

## 3. Key Components Used

| Component | Role in the Automation |
|---|---|
| SSM Automation | Orchestrates all steps |
| EC2 | Temporary instance for patching |
| Run Command (`AWS-RunPatchBaseline`) | Applies patches inside the instance |
| EC2 AMI | Captures patched state as a new image |
| Python script (inline in runbook) | Updates the Launch Template via API |
| EC2 Launch Template | Defines the AMI used by the ASG |
| Auto Scaling Group | Fleet of instances to be updated |
| EC2 Instance Refresh | Replaces running instances with new AMI |

## 4. Why Not Do This Manually?

Each step above is a separate AWS API call. Doing this manually means:
- Coordinating EC2, SSM, AMI, Launch Template, ASG, and Instance Refresh APIs
- Handling errors and retries at each step
- No rate control or audit trail

SSM Automation handles all of this declaratively in a single runbook with built-in error handling, rate control, and CloudTrail logging.

## 5. IAM Role Requirement

The automation needs an **IAM role** (Automation Assume Role) with permissions to:
- Launch and terminate EC2 instances
- Create AMIs and snapshots
- Update Launch Templates
- Modify Auto Scaling Groups
- Trigger Instance Refresh
- Invoke SSM Run Command

## 6. Key Takeaways for SysOps Associate

- SSM Automation can **orchestrate multiple AWS services** in a single runbook — EC2, AMI, ASG, Launch Templates, and more
- **Run Command can be called as a step within an Automation** — they are complementary, not alternatives
- The pattern **patch AMI → update launch template → instance refresh** is the standard way to roll out a new base image to an ASG
- **EC2 Instance Refresh** is the mechanism that replaces existing ASG instances with ones using the new launch template
- Inline scripts (e.g., Python) can be embedded in automation runbooks to call AWS APIs not covered by built-in actions
- This pattern is fully **hands-off** once triggered — no SSH, no manual AMI creation, no console clicks required
