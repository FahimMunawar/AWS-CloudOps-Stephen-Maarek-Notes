# SSM Inventory and State Manager

This note covers SSM Inventory (collecting metadata from managed instances) and SSM State Manager (keeping instances in a defined configuration state), which work closely together.

## 1. SSM Inventory

### What it Does
- Collects **metadata** from managed EC2 and on-premises instances
- Data collected includes:
  - Installed software and applications
  - OS and drivers
  - Configuration settings
  - Installed updates
  - Running services

### Storage and Analysis Options

| Option | Purpose |
|---|---|
| AWS Console | View inventory data directly |
| S3 (Resource Data Sync) | Export all inventory data to a central bucket |
| Amazon Athena | Run SQL queries against S3 inventory data |
| Amazon QuickSight | Build dashboards and visualizations |

### Key Features
- **Collection interval**: Configurable — minutes, hours, or days
- **Multi-account**: Aggregate inventory data from multiple accounts into one S3 bucket for centralized querying
- **Custom inventory**: Define your own inventory types (e.g., track physical location of each managed server)

### Enabling Inventory (Console)

1. SSM Console → **Inventory**
2. Click **Set up inventory** → **Enable inventory on all instances**
3. This creates a **State Manager association** in the background that applies the inventory configuration to all managed instances
4. Check **State Manager** to monitor association execution status per instance

## 2. SSM State Manager

### What it Does
- Automates keeping managed instances in a **desired configuration state**
- Runs on a schedule to continuously enforce the defined state

### Use Cases
- Enable inventory collection on all instances (as shown above)
- Bootstrap instances with required software at launch
- Patch OS and software on a schedule
- Enforce security configurations (e.g., ensure port 22 is always closed)
- Ensure an antivirus is always installed and running

### How it Works
- You create an **Association** — a document + schedule + targets
- SSM applies the association on the defined schedule
- If an instance drifts from the desired state, the next execution corrects it

### Association Components

| Component | Description |
|---|---|
| Document | The SSM document defining what state to apply |
| Targets | Instances or resource groups to apply it to |
| Schedule | How often to enforce the state (cron or rate expression) |
| Parameters | Any input values the document needs |

## 3. Hands-On: Enable Inventory with Resource Data Sync

### Step 1: Enable Inventory
1. SSM Console → **Inventory** → **Set up inventory**
2. Click **Enable inventory on all instances**
3. Navigate to **State Manager** → verify the association is running on all target instances
4. Wait for all instances to show **Success** status

### Step 2: Create an S3 Bucket for Sync

```
Bucket name: demo-ssm-inventory-<your-account>
Region: same as SSM
Block public access: enabled (default)
```

### Step 3: Attach Bucket Policy

SSM needs permission to write to the bucket. Example bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ssm.amazonaws.com"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*",
      "Condition": {
        "StringEquals": {
          "aws:SourceAccount": "YOUR-ACCOUNT-ID"
        }
      }
    }
  ]
}
```

Replace `YOUR-BUCKET-NAME` and `YOUR-ACCOUNT-ID` before saving.

### Step 4: Create Resource Data Sync
1. SSM Console → **Inventory** → **Resource Data Sync** → **Create**
2. Name: `DemoSync`
3. S3 bucket: `demo-ssm-inventory-<your-account>`
4. Click **Create**

### Step 5: Query Inventory Data
- **Console**: SSM → Inventory → select inventory type (e.g., `AWS:Application`)
  - View installed applications, versions, publishers, architectures across all instances
- **Athena**: Click the Athena link in Inventory console to run custom SQL queries against the S3 data
  - Allow ~5 minutes for data to populate after sync creation

## 4. Key Takeaways for SysOps Associate

- **Inventory** collects software/OS/config metadata from managed instances — useful for compliance and auditing at scale
- Enabling inventory creates a **State Manager association** automatically — the two features are tightly linked
- Inventory data can be exported to **S3 → Athena → QuickSight** for advanced querying and dashboards
- **Resource Data Sync** aggregates inventory from multiple accounts into a single S3 bucket
- **State Manager** enforces a desired configuration state on a schedule — not a one-time action
- State Manager uses **SSM documents** + **associations** — know this terminology for the exam
- Common exam scenario: "Ensure all instances always have antivirus installed" → answer is **State Manager association**
- The S3 bucket for inventory sync requires a **bucket policy** granting `ssm.amazonaws.com` permission to `s3:PutObject`
