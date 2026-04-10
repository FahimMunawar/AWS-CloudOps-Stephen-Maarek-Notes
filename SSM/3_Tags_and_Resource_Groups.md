# Tags and Resource Groups

This note covers AWS resource tags and how to use them to create SSM Resource Groups for operating on logical collections of resources.

## 1. What are Tags?

- **Tags**: Key-value pairs attached to AWS resources
- **Common on**: EC2, S3, DynamoDB, Lambda, and most other AWS services
- **Naming**: Free-form — no strict rules, but consistency matters
- **Common tag keys**: `Environment`, `Team`, `Project`, `Owner`, `CostCenter`
- **Best practice**: More tags is better than fewer — they have no cost

### Use Cases for Tags

| Use Case | Example |
|---|---|
| Resource grouping | Group all `dev` instances together |
| Automation | Run SSM Patch Manager on all `Environment=prod` instances |
| Security | Apply SCPs or IAM policies based on tags |
| Cost allocation | Track spend per team or environment using Cost Explorer |

## 2. What are Resource Groups?

- **Resource Groups**: Logical collections of AWS resources that share the same tags
- **Scope**: Regional — resource groups are created per region
- **Purpose**: Enable you to operate on a group of resources as a unit (e.g., patch all dev instances at once)
- **Backed by**: AWS Resource Groups service, accessible via SSM console

### What You Can Group

- Amazon EC2 instances
- S3 buckets
- DynamoDB tables
- Lambda functions
- Any taggable AWS resource

## 3. Hands-On Example

### Instance Tags Applied

| Instance Name | Environment | Team |
|---|---|---|
| MyDevInstance | dev | finance |
| MyProdInstance | prod | finance |
| MyOtherDevInstance | dev | operations |

### How to Tag an Instance

1. EC2 Console → Select instance → **Manage tags**
2. Add key-value pairs (e.g., `Environment` = `dev`, `Team` = `finance`)
3. Save

### Creating a Resource Group

1. Navigate to **Systems Manager → Resource Groups** (or search "Resource Groups")
2. Click **Create a resource group**
3. Select type: **Tag based**
4. Choose resource types (e.g., `AWS::EC2::Instance`) or use **All resource types**
5. Add tag filter (e.g., `Environment` = `dev`)
6. Click **Preview group resources** to verify the correct resources are matched
7. Name the group and click **Create group**

### Groups Created in This Example

| Group Name | Tag Filter | Matched Instances |
|---|---|---|
| DevGroup | Environment = dev | MyDevInstance, MyOtherDevInstance |
| ProdGroup | Environment = prod | MyProdInstance |
| FinanceGroup | Team = finance | MyDevInstance, MyProdInstance |

> Note: A single instance can belong to multiple resource groups (MyDevInstance is in both DevGroup and FinanceGroup).

## 4. Why Resource Groups Matter for SSM

Resource groups are a **prerequisite** for targeting SSM operations at scale:

- **Patch Manager**: Patch all instances in `ProdGroup` on a schedule
- **Run Command**: Execute a script across all instances in `FinanceGroup`
- **State Manager**: Apply a configuration to all `DevGroup` instances
- **Maintenance Windows**: Define a window for a specific resource group

Without resource groups, you would need to target individual instances one by one.

## 5. Key Takeaways for SysOps Associate

- Tags are **key-value pairs** — free-form naming, works on most AWS resources
- **More tags = better** — they enable grouping, automation, security, and cost tracking
- Resource groups are **regional** and **tag-based**
- A resource group matches all resources that satisfy its tag filter(s)
- One resource can belong to **multiple resource groups** simultaneously
- Resource groups are the **target mechanism** for SSM bulk operations (patching, run command, etc.)
- Common exam scenario: "How do you patch all production EC2 instances?" → Tag them `Environment=prod`, create a resource group, use SSM Patch Manager targeting that group
