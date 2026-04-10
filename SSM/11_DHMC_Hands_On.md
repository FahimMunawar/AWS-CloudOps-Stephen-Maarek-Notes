# DHMC Hands-On

This note walks through enabling Default Host Management Configuration and verifying that an EC2 instance registers with SSM without an IAM instance profile.

## 1. Enable DHMC in Fleet Manager

1. SSM Console → **Fleet Manager**
2. Click **Configure Default Host Management Configuration**
3. Select **Create a new IAM role** using the recommended settings
4. Click **Configure**

> DHMC is now enabled for the region. SSM will automatically pass an IAM role to qualifying instances.

## 2. Launch a Test Instance (No IAM Role)

| Setting | Value |
|---|---|
| Name | `DemoInstance` |
| AMI | Amazon Linux 2023 |
| Architecture | x86_64 (64-bit) |
| Key pair | None |
| IAM instance profile | **None** — intentionally omitted |
| Metadata (IMDSv2) | Enabled, V2 only (default on AL2023) |

- Under **Security** tab after launch: confirm **no IAM role** is attached
- Amazon Linux 2023 has IMDSv2-only enabled by default — no manual change needed

## 3. SSM Agent Version Requirement

DHMC requires SSM agent version **≥ 3.2.x**.

Amazon Linux 2023 may ship with an older version. Verify and upgrade if needed:

```bash
# Check if agent is running
sudo systemctl status amazon-ssm-agent

# Stop agent before upgrading
sudo systemctl stop amazon-ssm-agent

# Install latest agent (x86_64)
sudo yum install -y \
  https://s3.amazonaws.com/ec2-downloads-windows/SSMAgent/latest/linux_amd64/amazon-ssm-agent.rpm

# Verify new version
amazon-ssm-agent --version
# Expected: 3.2.923 or higher

# Confirm agent is running
sudo systemctl status amazon-ssm-agent
```

> Connect to the instance using **EC2 Instance Connect** — no key pair or SSH needed.

## 4. Verify Registration in Fleet Manager

1. SSM Console → **Fleet Manager** → refresh
2. `DemoInstance` appears as a managed node
3. SSM agent version is displayed
4. Click the instance → **Security** tab → confirm **no IAM role attached**

This confirms DHMC is working: the instance is managed by SSM purely via the instance identity role handshake.

## 5. Cleanup

1. SSM Console → **Fleet Manager** → **Configure Default Host Management Configuration** → **Disable**
2. EC2 Console → select `DemoInstance` → **Terminate instance**

## 6. Key Takeaways for SysOps Associate

- DHMC is enabled with a **single setting** in Fleet Manager — it creates and manages the required IAM role automatically
- The test instance must have **no IAM instance profile** to prove DHMC is doing the work
- **IMDSv2** is required — Amazon Linux 2023 enables this by default; older AMIs may need it configured manually
- SSM agent must be **≥ 3.2.x** — upgrade manually if the bundled version is older
- After the agent upgrade, the instance appears in Fleet Manager **with no IAM role attached** — this is the proof of DHMC
- DHMC must be **disabled per region** when no longer needed; it does not disable automatically
