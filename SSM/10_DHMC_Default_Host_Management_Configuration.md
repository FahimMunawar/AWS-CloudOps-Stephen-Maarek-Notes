# Default Host Management Configuration (DHMC)

This note covers DHMC — a Systems Manager feature that automatically registers EC2 instances without requiring a manually attached IAM instance profile.

## 1. The Problem DHMC Solves

Normally, EC2 instances need an **IAM instance profile** with `AmazonSSMManagedInstanceCore` to be managed by SSM. DHMC removes this requirement.

## 2. How DHMC Works

Every EC2 instance has an **instance identity role** — a built-in IAM role with no permissions that you do not control. Its only purpose is to identify the EC2 instance to certain AWS services.

```
EC2 Instance
  │
  │  (uses instance identity role — no permissions, auto-assigned by AWS)
  ▼
Systems Manager
  │
  │  (SSM passes an IAM role back to the instance)
  ▼
EC2 Instance is now a managed node
(no EC2 instance profile required)
```

- The instance identity role is **separate** from the EC2 instance profile
- You have no control over it — AWS assigns it automatically to every EC2 instance
- SSM uses it to identify the instance, then grants it the permissions needed for management

## 3. Requirements

| Requirement | Detail |
|---|---|
| IMDSv2 | Must be enabled on the instance (not IMDSv1) |
| SSM agent version | Must be recent enough to support IMDSv2 |
| DHMC enablement | Must be enabled **per region** |

## 4. What DHMC Automatically Enables

Once active, instances managed via DHMC get full SSM feature access:

- **Session Manager** — browser-based CLI access
- **Patch Manager** — automated patching
- **Inventory** — software and config data collection
- **SSM agent auto-update** — agent is kept current automatically

## 5. DHMC vs. IAM Instance Profile

| | IAM Instance Profile | DHMC |
|---|---|---|
| Setup required | Create role, attach to instance | Enable DHMC per region |
| Works without profile | No | Yes |
| IMDSv2 required | No | Yes |
| SSM features enabled | All (with correct policy) | All core features |
| Agent auto-update | No | Yes |

## 6. Key Takeaways for SysOps Associate

- DHMC lets EC2 instances be managed by SSM **without an IAM instance profile**
- It works via the **instance identity role** — a zero-permission role automatically present on every EC2 instance
- SSM receives the identity signal and **passes an IAM role back** to enable management
- Requires **IMDSv2** to be enabled and a sufficiently recent SSM agent
- Must be **enabled per region** — it is not global
- Automatically keeps the **SSM agent up to date** on managed instances
- Use DHMC to simplify fleet onboarding at scale without touching IAM roles per instance
