# 23. CloudFormation Nested Stacks

## Overview

**What are Nested Stacks?**

Nested stacks are CloudFormation stacks created as a resource within another stack using `Type: AWS::CloudFormation::Stack`. They allow you to isolate repeated patterns and common components into reusable stack templates.

**Why Use Them?**
- Reuse common infrastructure components (ALB config, security groups, RDS setup)
- Keep knowledge of one component isolated in one place
- Compose complex architectures from smaller, maintainable pieces
- AWS best practice for large CloudFormation deployments

**Key Rule:** Always update and delete via the **parent (root) stack** — never touch the nested stack directly.

---

## Nested Stacks vs Cross Stacks

This is a common exam distinction:

| Feature | Nested Stacks | Cross Stacks |
|---------|--------------|--------------|
| **Purpose** | Reuse components within a parent stack | Share values between independent stacks |
| **Lifecycle** | Same lifecycle as parent stack | Different lifecycles (VPC stack vs App stack) |
| **Sharing** | Internal to the parent — not shared broadly | Exported outputs consumed by multiple stacks |
| **Mechanism** | `AWS::CloudFormation::Stack` resource | `Outputs` + `Export` + `Fn::ImportValue` |
| **Update method** | Update the parent stack | Update each stack independently |
| **Use case** | Reusable ALB, RDS, ASG configuration blocks | Shared VPC, shared IAM roles |

**Decision Rule:**
- Same app, reusable component → **Nested Stack**
- Different apps sharing infrastructure output → **Cross Stack**

---

## Template Structure

**Parent Stack (calls nested stack):**

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Resources:
  # Regular resource in parent stack
  MyKeyPair:
    Type: AWS::EC2::KeyPair
    Properties:
      KeyName: my-key-pair

  # Nested stack resource
  MyStack:
    Type: AWS::CloudFormation::Stack
    Properties:
      # S3 URL to the nested stack template
      TemplateURL: https://s3.amazonaws.com/my-bucket/nested-template.yaml
      
      # Pass parameters to the nested stack
      Parameters:
        KeyName: !Ref MyKeyPair
        DBName: myDatabase
        DBUser: admin
        DBPassword: supersecret
        InstanceType: t2.micro

Outputs:
  # Reference to nested stack itself
  NestedStackRef:
    Value: !Ref MyStack

  # Access nested stack's outputs via GetAtt
  WebsiteURL:
    Value: !GetAtt MyStack.Outputs.WebsiteURL
    Description: 'Output pulled from the nested stack'
```

**Nested Stack Template (stored in S3):**

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Parameters:
  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
  DBName:
    Type: String
    MinLength: 1
    MaxLength: 64
  DBUser:
    Type: String
    NoEcho: true    # Sensitive — hides input in console
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t2.small

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Web server security group'

  WebServerInstance:
    Type: AWS::EC2::Instance
    CreationPolicy:
      ResourceSignal:
        Timeout: PT10M    # Wait for cfn-signal before marking complete
    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              mysql: []
              mysql-server: []
              httpd: []
              php: []
          files:
            /var/www/html/index.php:
              content: |
                <?php echo "Hello from nested stack!"; ?>
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'
              mysql:
                enabled: 'true'
                ensureRunning: 'true'
    Properties:
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      UserData: !Base64 |
        #!/bin/bash
        yum install -y aws-cfn-bootstrap
        /opt/aws/bin/cfn-init -v -s !Ref AWS::StackId -r WebServerInstance --region !Ref AWS::Region
        /opt/aws/bin/cfn-signal -e $? -r WebServerInstance -s !Ref AWS::StackId --region !Ref AWS::Region

Outputs:
  WebsiteURL:
    Value: !Sub 'http://${WebServerInstance.PublicIp}'
    Description: 'URL of deployed web server'
```

---

## Required IAM Capabilities

When creating a stack that contains nested stacks, CloudFormation requires two capabilities:

| Capability | Why Required |
|------------|-------------|
| `CAPABILITY_IAM` | Template creates IAM resources |
| `CAPABILITY_AUTO_EXPAND` | Template uses nested stacks (template is expanded at deploy time) |

> `CAPABILITY_AUTO_EXPAND` tells CloudFormation: *"I understand this template will be expanded by the content of nested templates, and I accept the risk."*

---

## Stack Lifecycle Rules

**Creation:**
```
Parent stack created
  ↓
Nested stack resources created (separate stack view in console)
  ↓
cfn-signal received by nested stack → nested stack CREATE_COMPLETE
  ↓
Parent stack CREATE_COMPLETE
```

**Update:**
```
Always update the PARENT stack with new template
  ↓
CloudFormation updates nested stack if template URL or parameters changed
  ↓
Never update the nested stack directly
```

**Deletion:**
```
Delete the PARENT stack only
  ↓
CloudFormation automatically deletes nested stacks
  ↓
Never delete nested stacks directly (will break parent)
```

---

## Accessing Nested Stack Outputs

Use `!GetAtt` with the `.Outputs.` pattern to pull outputs from a nested stack into the parent:

```yaml
Outputs:
  # Pull WebsiteURL output from MyStack nested stack
  WebURL:
    Value: !GetAtt MyStack.Outputs.WebsiteURL

  # Pull another output
  DBEndpoint:
    Value: !GetAtt MyStack.Outputs.DBEndpoint
```

---

## Console Behavior

- Nested stacks show a **NESTED** label in the stack list
- You can toggle "View nested" to see all nested stacks inline
- Each nested stack has its own **Events**, **Resources**, and **Outputs** tabs
- Stack events from nested stacks are visible separately, not mixed with parent events

---

## Best Practices

✓ **Always update via parent stack** — Direct nested stack updates cause drift and dependency issues  
✓ **Always delete via parent stack** — Deleting nested stack directly orphans parent stack resources  
✓ **Store nested templates in S3** — Required; TemplateURL must be an S3 URL  
✓ **Version your nested templates** — Use S3 versioning so updates are controlled  
✓ **Use for reusable components** — ALB, RDS, ASG, security groups used across multiple deployments  
✓ **Pass only necessary parameters** — Keep nested stack interfaces clean and minimal  
✓ **Use NoEcho for sensitive parameters** — Hides passwords/secrets in console and API responses  
✓ **Include cfn-signal in nested stacks** — CreationPolicy ensures nested stack only completes if config succeeds  

---

## SysOps Exam Focus

**Q1: "What is the primary use case for nested stacks?"**
- A) Sharing VPC outputs between application stacks
- B) Reusing common components (ALB, RDS) within a parent stack
- C) Deploying stacks across multiple regions
- D) Replacing cross-stack references
- **Answer: B** — Reusable components within the same parent stack

**Q2: "How do you update a nested stack?"**
- A) Update the nested stack directly in the console
- B) Update the parent stack with a new template
- C) Use AWS CLI to update nested stack ARN
- D) Delete and recreate the nested stack
- **Answer: B** — Always update via the parent; never touch nested stack directly

**Q3: "How do you access the output of a nested stack in the parent template?"**
- A) `Fn::ImportValue`
- B) `!Ref NestedStack`
- C) `!GetAtt NestedStack.Outputs.OutputKey`
- D) `!Sub '${NestedStack}'`
- **Answer: C** — Use GetAtt with the `.Outputs.` path

**Q4: "Which capability is required when deploying a template with nested stacks?"**
- A) CAPABILITY_NAMED_IAM
- B) CAPABILITY_AUTO_EXPAND
- C) CAPABILITY_NESTED
- D) CAPABILITY_STACK_SETS
- **Answer: B** — AUTO_EXPAND is required because the template is expanded at deploy time

**Q5: "What is the difference between nested stacks and cross stacks?"**
- A) No difference, they are the same concept
- B) Nested stacks share outputs using Fn::ImportValue; cross stacks use TemplateURL
- C) Nested stacks reuse components within a parent; cross stacks share values between independent stacks with different lifecycles
- D) Cross stacks are only for VPC resources
- **Answer: C** — Nested = reuse within parent; Cross = share values across independent stacks

**Q6: "Where must nested stack templates be stored?"**
- A) Local filesystem
- B) GitHub repository
- C) Amazon S3 (TemplateURL must be an S3 URL)
- D) AWS Systems Manager Parameter Store
- **Answer: C** — TemplateURL property requires an S3 URL

**Q7: "To delete a nested stack cleanly, you should:"**
- A) Delete the nested stack first, then the parent
- B) Delete only the parent stack — it automatically deletes nested stacks
- C) Delete all stacks simultaneously
- D) Use CloudFormation drift detection first
- **Answer: B** — Deleting the parent cascades deletion to all nested stacks

---

## Architecture Pattern

```
Root / Parent Stack
  ├─ KeyPair (direct resource)
  ├─ MyAppStack (AWS::CloudFormation::Stack)
  │     ├─ WebServerSecurityGroup
  │     ├─ WebServerInstance (with cfn-init + cfn-signal)
  │     └─ Outputs: WebsiteURL
  ├─ MyDBStack (AWS::CloudFormation::Stack)
  │     ├─ RDS Instance
  │     └─ Outputs: DBEndpoint
  └─ Outputs:
        ├─ !GetAtt MyAppStack.Outputs.WebsiteURL
        └─ !GetAtt MyDBStack.Outputs.DBEndpoint
```

Each nested stack can itself contain further nested stacks — there is no strict depth limit.

---

**File: 23_CloudFormation_Nested_Stacks.md**
**Status: SysOps-focused, exam-ready, concise format**
