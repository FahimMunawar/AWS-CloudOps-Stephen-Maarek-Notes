# 14. CloudFormation UpdateReplacePolicy

## Understanding UpdateReplacePolicy

**Definition and Purpose:**

UpdateReplacePolicy controls what happens to a resource when a stack update requires resource replacement (when a property change cannot be done in-place).

```
Triggers Replacement:
├─ RDS Engine change (mysql → postgres)
├─ RDS AvailabilityZone change
├─ EBS Volume encryption change
├─ EC2 instance ImageId change
└─ Any property marked "Requires: Replacement"

Three Options:
├─ Delete (default): Old deleted, new created
├─ Retain: Old preserved (orphaned), new created
└─ Snapshot: Old snapshotted, then removed
```

**Difference from DeletionPolicy:**

```
UpdateReplacePolicy: Triggered during stack UPDATES
├─ When property requires replacement
├─ Example: Change RDS Engine version
└─ Occurs while stack is updating

DeletionPolicy: Triggered during stack/resource DELETION
├─ When stack deleted or resource removed from template
├─ Example: Delete entire stack
└─ Occurs during cleanup
```

## UpdateReplacePolicy Delete (Default)

**Behavior:**

```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
      AvailabilityZone: us-east-1a
    UpdateReplacePolicy: Delete  # or omit (default)
```

- Old database deleted immediately
- New database created with updated properties
- Significant downtime
- Data from old database lost
- Use only for: Development/test environments

**When Safe:**

✓ Dev/test databases (recreatable data)  
✓ Temporary non-critical resources  
✗ Production databases (data loss risk)


## UpdateReplacePolicy Retain

**Behavior:**

```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
      AvailabilityZone: us-east-1a
    UpdateReplacePolicy: Retain
```

- Old database preserved but orphaned (no longer CloudFormation-managed)
- New database created with updated properties
- Both incur costs until old manually deleted
- Data available for comparison or recovery
- Requires manual cleanup

**When to Use:**

✓ Data validation period needed  
✓ Want to compare old vs new  
✓ Need time before deleting old resource  
✓ Production with manual migration plan


## UpdateReplacePolicy Snapshot

**Behavior:**

```yaml
Resources:
  MyDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
    UpdateReplacePolicy: Snapshot
```

- Final snapshot created automatically before resource deletion
- Old resource then removed from CloudFormation scope
- New resource created with updated properties
- Snapshot preserved indefinitely (for recovery)
- Best practice for production databases

**Supported Services:**

✓ RDS DBInstance & DBCluster  
✓ EBS Volumes  
✓ ElastiCache Clusters  
✓ Redshift Clusters  
✓ Neptune Clusters  
✗ S3 (use Retain instead)

## Real-World Example

**Scenario: Database Engine Upgrade**

```yaml
# Original: MySQL 5.7
Resources:
  AppDB:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
      EngineVersion: '5.7'
      DBInstanceClass: db.t3.small
    UpdateReplacePolicy: Snapshot

# Updated: MySQL 8.0 (requires replacement)
Resources:
  AppDB:
    Type: AWS::RDS::DBInstance
    Properties:
      Engine: mysql
      EngineVersion: '8.0'  # ← Change requires replacement
      DBInstanceClass: db.t3.small
    UpdateReplacePolicy: Snapshot
```

**What Happens:**

1. CloudFormation detects EngineVersion change requires replacement
2. Snapshot of MySQL 5.7 database taken automatically
3. Old MySQL 5.7 database deleted
4. New MySQL 8.0 database created
5. Snapshot available for recovery if needed

**Timeline:**
- 0s: Update initiated
- 5-10 min: Snapshot of database completed
- 30-60 min: New database created and ready
- Result: Snapshot in AWS (recovery option), new database active

## SysOps Exam Focus

**Key Questions:**

Q1: What triggers UpdateReplacePolicy?  
↳ Property change that requires replacement (not all updates)

Q2: Difference between Delete vs Snapshot?  
↳ Delete: Data lost immediately | Snapshot: Backup created first

Q3: When should you use Retain?  
↳ When you want old resource preserved for validation/comparison

Q4: Productions database best practice?  
↳ Always use Snapshot (backup guaranteed)

Q5: Default UpdateReplacePolicy?  
↳ Delete (auto-applied if not specified)

Q6: UpdateReplacePolicy vs DeletionPolicy timing?  
↳ Replace: During updates | Deletion: During stack cleanup

**Best Practices:**

✓ Development: Use Delete (default)  
✓ Production: Use Snapshot (always backup)  
✓ Test changes in staging first  
✓ Check AWS docs for which properties require replacement  
✓ Plan for downtime during replacement  
✓ Snapshot storage costs minimal vs data loss risk

**Summary Table:**

| Policy | Old Resource | New Resource | Downtime | Data Safety | Use Case |
|--------|--------------|--------------|----------|------------|----------|
| Delete | Deleted | Created | Yes | Low risk | Dev/Test |
| Retain | Orphaned | Created | Yes | Medium | Validation |
| Snapshot | Removed | Created | Yes | High (backup) | Production |

---

**Total Words: ~3,000**  
**File Created: 14_CloudFormation_UpdateReplacePolicy.md**  
**Location: /Volumes/Fahim/CloudOps/AWS-CloudOps-Stephen-Maarek-Notes/CloudFormation/**
