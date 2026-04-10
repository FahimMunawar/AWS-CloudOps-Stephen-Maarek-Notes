# SSM Setup Hands-On

This note walks through registering EC2 instances with AWS Systems Manager by creating the required IAM role and launching instances with the SSM agent.

## Overview

- Goal: Launch EC2 instances that automatically appear as managed nodes in SSM Fleet Manager
- Key insight: The **instance reaches out to SSM** — SSM does not reach into the instance
- No inbound connectivity (SSH, HTTP, HTTPS) is required

## 1. Create the IAM Role

The EC2 instance must have an IAM role with permission to communicate with SSM.

### Steps

1. Navigate to **IAM → Roles → Create role**
2. Select trusted entity: **AWS service → EC2**
3. Search for and attach the policy: `AmazonSSMManagedInstanceCore`
   - This policy contains the minimum permissions needed for SSM to manage the instance
4. Name the role: `AmazonEC2RoleForSSM`
5. Click **Create role**

### Why `AmazonSSMManagedInstanceCore`?
- Grants the instance permission to call SSM APIs
- Enables Fleet Manager, Session Manager, Run Command, Patch Manager, and more
- This is the **minimum required policy** for SSM management

## 2. Launch EC2 Instances

### Configuration

| Setting | Value | Reason |
|---|---|---|
| AMI | Amazon Linux 2 | SSM agent is pre-installed |
| Instance Type | t2.micro | Free tier eligible |
| Key Pair | None (proceed without) | Not needed — SSM handles access |
| Security Group | New group, **no inbound rules** | Demonstrates outbound-only model |
| IAM Instance Profile | `AmazonEC2RoleForSSM` | Required for SSM registration |
| Number of instances | 3 | To demonstrate fleet management |

### Steps

1. EC2 Console → **Launch Instances**
2. Select **Amazon Linux 2** AMI
3. Choose **t2.micro**
4. Key pair → **Proceed without a key pair**
5. Security group → Create new → **remove all inbound rules** (leave empty)
6. Expand **Advanced details** → IAM instance profile → select `AmazonEC2RoleForSSM`
7. Set number of instances to **3**
8. Click **Launch instances**

## 3. Verify in Fleet Manager

1. Navigate to **Systems Manager → Fleet Manager**
2. Refresh the page after instances reach `running` state
3. All three instances appear as **managed nodes**
4. Confirm the following columns:
   - **Platform**: Amazon Linux
   - **Source type**: EC2
   - **SSM Agent online**: Yes
   - **Agent version**: displayed

## 4. Key Concept: Outbound-Only Architecture

```
EC2 Instance (SSM Agent)
        │
        │  outbound HTTPS (port 443) to SSM endpoints
        ▼
AWS Systems Manager Service
```

- The **SSM agent on the instance initiates** the connection to the SSM service
- The SSM service does **not** connect inbound to the instance
- This is why a security group with **zero inbound rules** still works
- Instances do not need a public IP or SSH access to be managed by SSM

## 5. Troubleshooting Registration Failures

| Symptom | Likely Cause | Fix |
|---|---|---|
| Instance not appearing in Fleet Manager | Missing or wrong IAM role | Attach `AmazonSSMManagedInstanceCore` policy |
| Instance not appearing in Fleet Manager | SSM agent not running | Verify agent is installed and active |
| Instance not appearing in Fleet Manager | No outbound internet access | Add VPC endpoint for SSM or allow outbound HTTPS |

## 6. Key Takeaways for SysOps Associate

- **Amazon Linux 2** has the SSM agent pre-installed — no manual setup needed
- The EC2 instance **must have an IAM role** with `AmazonSSMManagedInstanceCore`
- SSM uses an **outbound connection model** — the agent on the instance calls SSM, not the other way around
- **No SSH, no key pair, no inbound rules** are required for SSM management
- Fleet Manager is where you view and manage all registered nodes
- Two failure causes to know: **agent not running** or **IAM role missing/incorrect**
