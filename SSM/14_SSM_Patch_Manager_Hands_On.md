# SSM Patch Manager Hands-On

This note walks through creating a patch policy, exploring the Patch Manager dashboard, and setting up a Maintenance Window with a registered Run Command task.

## 1. Creating a Patch Policy

### Steps

1. SSM Console → **Patch Manager** → **Create patch policy**

### Patch Policy Configuration

| Setting | Options | Notes |
|---|---|---|
| Policy name | `DemoPolicy` | Any descriptive name |
| Scanning operation | Scan only or **Scan and install** | Scan-only generates compliance reports; Scan and install applies patches |
| Scan schedule | Daily at 1:00 AM (customizable) | Cron or rate expression |
| Install schedule | Weekly on Sunday (customizable) | Separate from scan schedule |
| Instance reboot | Allow / Do not allow / If needed | Controls post-patch reboot behaviour |
| Patch baseline | AWS recommended defaults or custom per OS | Select per operating system |
| Log output | Optional S3 bucket | Stores patch operation logs |
| Target region | Current region or multiple regions | Patch policy can cover multiple regions from one place |
| Target instances | All managed nodes or specific instances | Can filter by tags or instance IDs |
| Concurrency | e.g., 10% of nodes at a time | Gradual rollout to avoid fleet-wide impact |
| Error threshold | e.g., stop after N% errors | Prevents cascading failures |
| IAM instance profile | Optional — attach SSM role if not already present | Required for instances without an existing SSM role |

### Patch Baseline Selection
- **Default**: AWS-recommended baselines per OS (Amazon Linux, Ubuntu, Windows, etc.)
- **Custom**: Create your own baseline with specific approval rules, exceptions, and patch repositories — then reference it in the policy

## 2. Patch Manager Dashboard

Navigate: **Patch Manager** → **Start with an overview**

| Section | Contents |
|---|---|
| Dashboard | Instance management summary, compliance overview, reports |
| Node patching details | Per-instance patch status |
| Patch baselines | One default baseline per OS; create custom baselines here |
| Patches | Browse all patches by OS, release date, severity |
| Settings | Security Hub integration with Patch Manager |

### Creating a Custom Patch Baseline (Console)
1. Patch Manager → **Patch baselines** → **Create patch baseline**
2. Select OS
3. Define **approval rules**: auto-approve patches N days after release, filter by severity/classification
4. Define **exceptions**: explicitly approved or rejected patch lists
5. Optionally set a custom patch repository
6. Assign to a patch group via the patch policy

## 3. Creating a Maintenance Window

### Steps

1. SSM Console → **Maintenance Windows** → **Create maintenance window**

### Maintenance Window Configuration

| Setting | Example Value | Notes |
|---|---|---|
| Name | `DemoMaintenanceWindow` | Descriptive name |
| Allow unregistered targets | Yes | Targets can be registered after window creation |
| Schedule type | Cron expression | Or use rate expression |
| Cron schedule | `0 3 * * ? *` (daily at 3:00 AM) | Runs at 3 AM every day |
| Duration | 2 hours | How long the window stays open |
| Stop initiating tasks | 1 hour before close | Prevents tasks from starting too close to window end |

### Register a Run Command Task in the Window

1. Select the maintenance window → **Register tasks** → **Register Run command task**
2. Task name: `patch`
3. Document: `AWS-RunPatchBaseline`
4. Targets: select registered managed nodes (e.g., all three instances)
5. Concurrency: `1` (one target at a time)
6. Error threshold: `0`
7. Click **Register Run command**

The `AWS-RunPatchBaseline` document will now execute automatically during the maintenance window according to the defined schedule.

## 4. Key Takeaways for SysOps Associate

- A **patch policy** is the high-level configuration combining baseline + schedule + targets + rate control in one place
- **Scan** generates compliance reports; **Scan and install** applies the patches — know the difference
- Custom patch baselines are created in Patch Manager and then referenced in the patch policy
- **Maintenance Windows** run on a cron schedule and can contain multiple registered tasks (Run Command, Automation, etc.)
- The task registered inside a maintenance window to apply patches uses the document **`AWS-RunPatchBaseline`**
- **Concurrency + error threshold** on the maintenance window task provides safe rollout — same controls as Run Command
- Patch logs can be sent to **S3** for auditing
- A single patch policy can target **multiple regions** — useful for centralized patch management
