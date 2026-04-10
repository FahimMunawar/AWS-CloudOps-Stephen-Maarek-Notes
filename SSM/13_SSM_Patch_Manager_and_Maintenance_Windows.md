# SSM Patch Manager and Maintenance Windows

This note covers SSM Patch Manager — automated patching for managed instances — and Maintenance Windows, which schedule when patching and other tasks run.

## 1. What is Patch Manager?

- Automates patching of **OS updates, application updates, and security updates**
- Supports: **EC2 instances and on-premises servers**, Linux / MacOS / Windows
- Two modes:
  - **On-demand**: Run patch manager manually at any time
  - **Scheduled**: Run within a **Maintenance Window**
- Generates a **patch compliance report** — lists missing/installed patches per instance
  - Report can be sent to **S3** for further analysis

## 2. Core Components

### Patch Baseline
Defines **which patches should and should not** be installed.

| Type | Description |
|---|---|
| **Predefined (AWS-managed)** | One per OS (Windows, Amazon Linux, Ubuntu, etc.) — cannot be modified |
| **Custom** | You define approved, auto-approved, and rejected patches; choose patch repository |

- **Default behavior**: Install only **critical** and **security** patches
- **Auto-approval**: Patches can be automatically approved N days after release (useful when no manual approver is available)
- **Custom repository**: You can point Patch Manager at an internal corporate patch repository

### Patch Group
Associates a set of instances with a specific patch baseline.

- Tag instances with key `Patch Group` and a value (e.g., `dev`, `test`, `prod`)
- One instance can belong to **only one patch group**
- One patch group can be registered with **only one patch baseline**
- Instances with **no patch group tag** use the **default patch baseline**

## 3. How Patch Baselines and Groups Work Together

```
Instance Tags                    Patch Baseline Assignment
─────────────────────────────────────────────────────────
Patch Group = dev         ──►   Custom Baseline (PB-dev)
Patch Group = prod        ──►   Custom Baseline (PB-prod)
(no patch group tag)      ──►   Default Baseline (AWS-managed)
```

### Example with Three Instances

| Instance | Tag | Patch Baseline Assigned |
|---|---|---|
| Instance 1 | `Patch Group = dev` | PB-dev (custom) |
| Instance 2 | *(no patch group tag)* | Default AWS baseline |
| Instance 3 | *(no patch group tag)* | Default AWS baseline |

## 4. Running Patches: AWS-RunPatchBaseline Document

The SSM document used to apply patches is **`AWS-RunPatchBaseline`**.

```
Console / SDK / Maintenance Window
          │
          ▼
    Run Command
  (AWS-RunPatchBaseline)
          │
          ▼
  SSM Agent on instance
  queries Patch Manager
  for applicable patches
  based on assigned baseline
          │
          ▼
  Patches installed on instance
```

- Supports Linux, MacOS, and Windows Server
- Initiated from: console, CLI/SDK, or a Maintenance Window
- Supports **rate control** (concurrency + error threshold) like all Run Command operations

## 5. Maintenance Windows

### What it Does
Defines a **scheduled time window** for performing tasks on instances — patching, driver updates, software installs, etc.

### Components

| Component | Description |
|---|---|
| **Schedule** | Cron or rate expression (e.g., nightly at 3:00–5:00 AM) |
| **Duration** | How long the window is open (e.g., 2 hours) |
| **Registered targets** | Instances or resource groups to operate on |
| **Tasks** | What to run during the window (e.g., Run Command with `AWS-RunPatchBaseline`) |

### Typical Use
- Schedule patch runs during off-peak hours (e.g., 3:00–5:00 AM)
- Combine with rate control to patch instances progressively without downtime

## 6. Patch Compliance Reporting

- After patching, Patch Manager generates a **compliance report**
- Report shows: which patches are installed, missing, or failed per instance
- Can be exported to **S3** and queried with Athena
- Visible in the SSM console under **Compliance**

## 7. Key Takeaways for SysOps Associate

- Patch Manager automates OS and application patching for **EC2 and on-premises**, across Linux, MacOS, and Windows
- Two key components: **Patch Baseline** (what to patch) and **Patch Group** (which instances get which baseline)
- Tag instances with `Patch Group = <value>` to assign a custom baseline — untagged instances get the **default baseline**
- One instance → one patch group; one patch group → one patch baseline
- The SSM document that applies patches is **`AWS-RunPatchBaseline`**
- Use **Maintenance Windows** to schedule patches at safe times (e.g., nightly) with rate control
- Patch compliance reports can be sent to **S3** for auditing and analysis
- Auto-approval lets patches be approved automatically N days after release — useful for hands-off environments
