# SSM Fleet Manager

This note covers SSM Fleet Manager — the central console for viewing and managing all SSM-registered nodes, and the IAM permissions that enable it.

## 1. What is Fleet Manager?

- **Purpose**: Centrally view and remotely manage all nodes registered with SSM
- **Supported node types**: EC2 instances, on-premises servers/VMs, Edge devices, IoT devices
- **Supported OS**: Windows and Linux
- **Requirement**: SSM agent must be installed and running on every node

## 2. Use Cases

- View node status, health, and performance across your entire fleet
- Perform troubleshooting and management tasks from a single console
- **Windows**: Connect via RDP directly through Fleet Manager
- **Linux**: Connect via Session Manager (browser-based CLI, no SSH needed)
- Acts as the **entry point** for applying other SSM features (Patch Manager, Run Command, etc.) at scale

## 3. IAM Permissions for Fleet Manager

### Required Policy: `AmazonSSMManagedInstanceCore`

This single policy grants EC2 instances everything they need to integrate with SSM:

| Capability | SSM Feature Enabled |
|---|---|
| Register with Systems Manager | Fleet Manager |
| Access session connections | Session Manager |
| Read SSM documents and parameters | Documents, Parameter Store |
| Receive and execute commands | Run Command |
| Apply patches | Patch Manager |
| Report inventory and compliance data | Inventory, Compliance |

### How to Attach

**Option 1: IAM Instance Role** (standard approach)
- Create an IAM role for EC2 with `AmazonSSMManagedInstanceCore` attached
- Assign the role to the EC2 instance at launch or after via **Modify IAM role**

**Option 2: Default Host Management Configuration**
- A Fleet Manager feature that automatically manages IAM permissions for you
- Removes the need to manually create and attach an instance role
- Simplifies setup at scale

## 4. Key Takeaways for SysOps Associate

- Fleet Manager is the **central hub** for viewing all managed nodes — EC2, on-premises, Edge, and IoT
- All nodes require the **SSM agent** installed and running
- `AmazonSSMManagedInstanceCore` is the **single policy** that unlocks all core SSM features for an EC2 instance
- If an instance is not appearing in Fleet Manager: check the **SSM agent** and the **IAM role**
- **Default Host Management Configuration** simplifies IAM setup and avoids manual role creation
- Fleet Manager is not just a viewer — it is the **foundation** that all other SSM features (Run Command, Patch Manager, Session Manager, etc.) build on
