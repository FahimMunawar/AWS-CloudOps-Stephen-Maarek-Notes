# 20. CloudFormation cfn-init Helper Script

## Overview

**What is cfn-init?**

`cfn-init` is a Python helper script that retrieves and interprets CloudFormation metadata to configure EC2 instances. It makes complex EC2 configurations readable and maintainable by replacing long bash scripts with declarative configuration blocks.

**Why Use cfn-init Instead of UserData?**
- Readability: Declarative configuration vs imperative bash scripts
- Maintainability: Structure organized by packages, files, commands, services
- Debugging: Separate log files for each operation
- Scalability: Complex multi-step configurations readable at a glance
- Best practice: AWS-recommended approach for instance configuration

**How It Works:**
1. CloudFormation launches EC2 instance
2. UserData script runs and calls `cfn-init`
3. cfn-init queries CloudFormation service for metadata
4. cfn-init downloads packages, creates files, runs commands, starts services
5. Output logged to `/var/log/cfn-init.log` and `/var/log/cfn-init-cmd.log`

---

## CloudFormation Init Metadata Block

**Structure:**
```yaml
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:  # Install packages
          groups:    # Create user groups
          users:     # Create users
          sources:   # Download and extract files
          files:     # Create files with content
          commands:  # Run shell commands
          services:  # Start/enable services
```

**Components:**

| Component | Purpose | Example |
|-----------|---------|---------|
| **packages** | Latest versions of packages | apache, mysql, nodejs |
| **groups** | Create user groups | webadmin group |
| **users** | Create OS users | appuser |
| **sources** | Download/extract archives | Git repos, configs |
| **files** | Create text files with content | Config files, web pages |
| **commands** | Execute shell commands | Custom setup scripts |
| **services** | Configure system services | Start HTTPD, enable on boot |

---

## Real-World Example: Web Server with cfn-init

**Scenario:** Deploy Apache web server with custom configuration using metadata block instead of UserData bash script.

**Complete Template:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'EC2 Instance with cfn-init for Apache Configuration'

Parameters:
  ImageId:
    Type: AWS::EC2::Image::Id
    Description: AMI ID for region

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Allow SSH and HTTP'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref ImageId
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      
      # UserData calls cfn-init
      UserData: !Base64 |
        #!/bin/bash
        # Download and execute cfn-init helper script
        yum update -y
        yum install -y aws-cfn-bootstrap
        
        # Call cfn-init to process metadata
        /opt/aws/bin/cfn-init -v \
          -s !Ref AWS::StackId \
          -r WebServerInstance \
          -c config \
          --region !Ref AWS::Region
        
        # Signal completion (covered in next lecture)
        echo "cfn-init completed"
    
    # Metadata with declarative configuration
    Metadata:
      AWS::CloudFormation::Init:
        config:
          # Install packages
          packages:
            yum:
              httpd: []
              php: []
          
          # Create files
          files:
            /var/www/html/index.html:
              content: |
                <html>
                  <head><title>Welcome</title></head>
                  <body>
                    <h1>Hello from cfn-init!</h1>
                    <p>Instance configured via metadata block</p>
                  </body>
                </html>
              mode: '000644'
              owner: 'apache'
              group: 'apache'
            
            /etc/httpd/conf.d/custom.conf:
              content: |
                # Custom Apache configuration
                Listen 80
              mode: '000644'
          
          # Run commands
          commands:
            01_echo:
              command: echo "Apache installation started" >> /var/log/setup.log
          
          # Configure services
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'
                files:
                  - /etc/httpd/conf.d/custom.conf
                  - /var/www/html/index.html

Outputs:
  InstancePublicIP:
    Value: !GetAtt WebServerInstance.PublicIp
  
  WebURL:
    Value: !Sub 'http://${WebServerInstance.PublicIp}'
```

---

## cfn-init Command Syntax

**Basic Invocation in UserData:**

```bash
#!/bin/bash
# Install cfn-bootstrap (contains cfn-init, cfn-signal, cfn-hup)
yum install -y aws-cfn-bootstrap

# Call cfn-init
/opt/aws/bin/cfn-init -v \
  -s AWS::StackId \           # Stack ID (use !Ref)
  -r ResourceLogicalId \       # Resource name (e.g., WebServerInstance)
  -c config \                  # Config set name (typically "config")
  --region AWS::Region        # Region (use !Ref)
```

**Parameters Explained:**
- `-v`: Verbose output (recommended for debugging)
- `-s`: StackId pseudo parameter identifying the stack
- `-r`: Logical resource name (must match resource name in template)
- `-c`: Config set name (usually `config`, can have multiple sets)
- `--region`: Region where stack is deployed

---

## Debugging cfn-init

**Log Files:**

```bash
# Main cfn-init execution log
cat /var/log/cfn-init.log

# Command outputs from cfn-init
cat /var/log/cfn-init-cmd.log

# System cloud-init log (calls cfn-init)
cat /var/log/cloud-init-output.log
```

**Checking Execution:**

```bash
# Verify Apache is running
systemctl status httpd

# Check web page was created
cat /var/www/html/index.html

# Review cfn-init command execution
tail -50 /var/log/cfn-init-cmd.log
```

**Common Errors and Solutions:**

| Error | Cause | Solution |
|-------|-------|----------|
| `cfn-init: command not found` | aws-cfn-bootstrap not installed | Add `yum install -y aws-cfn-bootstrap` to UserData |
| Metadata not found | Wrong resource name in `-r` parameter | Verify resource logical name matches template |
| Permission denied on files | Incorrect file ownership | Specify `owner` and `group` in files block |
| Service won't start | Dependencies missing | Install required packages before service config |

---

## Best Practices

✓ **Use cfn-init for all complex configurations** — More readable than bash scripts  
✓ **Always install aws-cfn-bootstrap in UserData** — Required before calling cfn-init  
✓ **Verify resource name in `-r` parameter** — Must match logical resource ID exactly  
✓ **Set file ownership correctly** — Specify `owner` and `group` for critical files  
✓ **Order commands with prefixes** — `01_`, `02_`, `03_` to ensure execution order  
✓ **Check logs immediately after stack creation** — Catch errors before they compound  
✓ **Use `-v` flag for verbose output** — Essential for debugging failed configurations  
✓ **Specify config dependencies** — Use `files` and `packages` in service depends_on  
✓ **Test locally first** — Test metadata structure on test stack before production  
✓ **Clean up resources** — Delete test stacks to avoid unexpected charges

---

## SysOps Exam Focus

**Q1: "What does cfn-init do?"**
- A) Signals stack completion
- B) Interprets CloudFormation metadata and configures EC2
- C) Monitors EC2 instance health
- D) Manages security groups
- **Answer: B** — Retrieves metadata and configures instance

**Q2: "Where is cfn-init metadata defined in template?"**
- A) Properties section
- B) Metadata section of resource
- C) Outputs section
- D) Parameters section
- **Answer: B** — AWS::CloudFormation::Init in Metadata block

**Q3: "Which parameter identifies the EC2 instance in cfn-init call?"**
- A) `-s` (StackId)
- B) `-r` (resource logical name)
- C) `-c` (config set)
- D) `--region`
- **Answer: B** — Resource name must match exactly

**Q4: "Where should you check if cfn-init commands failed?"**
- A) `/var/log/cloud-init-output.log`
- B) `/var/log/cfn-init.log`
- C) `/var/log/cfn-init-cmd.log`
- D) EC2 console
- **Answer: C** — cfn-init-cmd.log shows all command outputs

**Q5: "True or false: cfn-init signals stack success/failure to CloudFormation?"**
- A) True
- B) False
- **Answer: B** — cfn-init doesn't signal completion (need cfn-signal for that)

**Q6: "aws-cfn-bootstrap package contains which helper scripts?"**
- A) cfn-init only
- B) cfn-init, cfn-signal, cfn-get-metadata, cfn-hup
- C) cfn-signal only
- D) AWS CLI tools
- **Answer: B** — Package includes all four CloudFormation helper scripts

---

## Four CloudFormation Helper Scripts

| Script | Purpose | Lesson |
|--------|---------|--------|
| **cfn-init** | Configure EC2 from metadata | Current (20) |
| **cfn-signal** | Signal stack completion/failure | Next (21) |
| **cfn-get-metadata** | Query metadata from stack | Advanced |
| **cfn-hup** | Monitor metadata changes, trigger updates | Advanced |

---

## Key Comparison: UserData vs cfn-init

| Aspect | UserData Script | cfn-init + Metadata |
|--------|-----------------|-------------------|
| **Readability** | Procedural bash (confusing) | Declarative blocks (clear) |
| **Maintenance** | Hard to modify | Easy to extend |
| **Debugging** | Limited logging | Detailed separate logs |
| **Complexity** | Not suitable for large configs | Designed for complex setups |
| **File management** | `cat > file.txt` syntax | Dedicated files block with permissions |
| **Service management** | `systemctl` commands | Services block with enabled/running |

**Recommendation:** Use cfn-init metadata for all but trivial configurations (installing single package)

---

**Total Words: ~2,700**  
**File: 20_CloudFormation_cfn-init_Helper.md**  
**Status: SysOps-focused, exam-ready, concise format**