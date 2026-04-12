# 24. CloudFormation DependsOn

## Overview

**What is DependsOn?**

`DependsOn` is a resource attribute that explicitly controls the creation order of resources. When specified, CloudFormation will not create a resource until its dependency has been successfully created.

**Why Use It?**
- Enforce creation order when no `!Ref` or `!GetAtt` link exists between resources
- Ensure dependent services (e.g. RDS) are ready before dependent resources (e.g. EC2) start
- Mirror deletion order — DependsOn reverses automatically on delete

---

## How CloudFormation Determines Order

CloudFormation uses two mechanisms to determine resource creation order:

| Mechanism | How It Works | Example |
|-----------|-------------|---------|
| **Implicit** (automatic) | `!Ref` or `!GetAtt` between resources tells CloudFormation one depends on the other | EC2 referencing a SecurityGroup via `!Ref` |
| **Explicit** (`DependsOn`) | You manually declare the dependency when no `!Ref`/`!GetAtt` link exists | EC2 depending on RDS with no direct property link |

> If two resources have no `!Ref` or `!GetAtt` between them and no `DependsOn`, CloudFormation creates them **in parallel**.

---

## Syntax

**Single dependency:**
```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    DependsOn: MyDBInstance       # Wait for RDS before creating EC2
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.micro

  MyDBInstance:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: mysql
      MasterUsername: admin
      MasterUserPassword: password
      AllocatedStorage: 20
```

**Multiple dependencies:**
```yaml
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    DependsOn:
      - MyDBInstance
      - MyBucket
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.micro
```

---

## Creation vs Deletion Order

`DependsOn` affects **both** creation and deletion — but in **reverse**:

```
Creation Order (A DependsOn B):
  B created first → B CREATE_COMPLETE → A created

Deletion Order (A DependsOn B):
  A deleted first → A DELETE_COMPLETE → B deleted
```

**Example from lecture:**

```
MyBucket DependsOn MyEC2Instance

Create:   MyEC2Instance → CREATE_COMPLETE → MyBucket created
Delete:   MyBucket → DELETE_COMPLETE → MyEC2Instance terminated
```

This ensures nothing is deleted out of order, preventing dependency errors during stack teardown.

---

## Real-World Example: EC2 Waits for RDS

**Scenario:** Application server should only launch after the database is ready.

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Resources:
  AppDatabase:
    Type: AWS::RDS::DBInstance
    Properties:
      DBInstanceClass: db.t3.micro
      Engine: mysql
      DBName: appdb
      MasterUsername: admin
      MasterUserPassword: '{{resolve:secretsmanager:prod/db:password}}'
      AllocatedStorage: 20

  AppServer:
    Type: AWS::EC2::Instance
    DependsOn: AppDatabase        # EC2 waits for RDS to be fully created
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.micro
      UserData: !Base64 |
        #!/bin/bash
        # RDS is guaranteed to exist by the time this runs
        echo "Connecting to database..."
```

Without `DependsOn`, both resources would start creation in parallel and the EC2 UserData script might run before RDS is available.

---

## Implicit vs Explicit Dependency — Comparison

```yaml
# IMPLICIT — no DependsOn needed
# CloudFormation sees !Ref and knows EC2 depends on SG
Resources:
  MySecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Web SG'

  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      SecurityGroups:
        - !Ref MySecurityGroup    # CloudFormation infers dependency automatically


# EXPLICIT — DependsOn required
# No !Ref or !GetAtt links EC2 to S3, so no implicit dependency
Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345678
      InstanceType: t2.micro

  MyBucket:
    Type: AWS::S3::Bucket
    DependsOn: MyEC2Instance      # Must be explicit — no property link exists
```

---

## Best Practices

✓ **Prefer implicit dependencies** — Use `!Ref` / `!GetAtt` where possible; CloudFormation handles order automatically  
✓ **Use DependsOn only when necessary** — Over-using it adds unnecessary serialization and slows stack creation  
✓ **Remember deletion reversal** — DependsOn reverses on delete; design with teardown in mind  
✓ **Use for service readiness** — When EC2 UserData needs a resource (RDS, ElastiCache) to exist before running  
✓ **Can depend on multiple resources** — Use list syntax for multiple dependencies  

---

## SysOps Exam Focus

**Q1: "Two resources have no !Ref or !GetAtt relationship. How does CloudFormation create them?"**
- A) It fails with a dependency error
- B) It creates them sequentially in template order
- C) It creates them in parallel
- D) It asks the user for creation order
- **Answer: C** — Without any dependency signal, CloudFormation parallelizes creation

**Q2: "When should you use DependsOn?"**
- A) Always, for every resource
- B) When two resources have no !Ref or !GetAtt link but one must be created before the other
- C) Only for EC2 instances
- D) Only when using nested stacks
- **Answer: B** — Use it only when no implicit dependency exists but ordering matters

**Q3: "Resource A has DependsOn: B. What happens during stack deletion?"**
- A) B is deleted first, then A
- B) A and B are deleted in parallel
- C) A is deleted first, then B
- D) Only A is deleted; B is preserved
- **Answer: C** — Deletion order is the reverse of creation order

**Q4: "When is DependsOn NOT needed between two resources?"**
- A) When both are EC2 instances
- B) When one resource uses !Ref or !GetAtt to reference the other
- C) When both are in the same stack
- D) When using nested stacks
- **Answer: B** — !Ref and !GetAtt create implicit dependencies automatically

**Q5: "What is the effect of adding DependsOn: MyDBInstance to an EC2 resource?"**
- A) EC2 and RDS are created in parallel
- B) RDS is created only after EC2 is ready
- C) EC2 is created only after RDS reaches CREATE_COMPLETE
- D) EC2 is deleted when RDS is deleted
- **Answer: C** — EC2 waits for RDS CREATE_COMPLETE before starting

---

## Quick Reference

```
No relationship between resources → Created in PARALLEL
!Ref or !GetAtt link exists       → Implicit order (automatic)
DependsOn specified               → Explicit order (manual)

DependsOn: B  →  Create: B first, then A
              →  Delete: A first, then B
```

---

**File: 24_CloudFormation_DependsOn.md**
**Status: SysOps-focused, exam-ready, concise format**
