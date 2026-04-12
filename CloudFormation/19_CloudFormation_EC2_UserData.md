# 19. CloudFormation EC2 UserData

## Overview

**What is UserData in CloudFormation?**

UserData is a property on EC2 instances that runs a script during first launch. In CloudFormation, you pass this script via the `UserData` property using the `Base64()` function to encode the script as base64. The script executes as root on instance startup.

**Why Use It?**
- Automate EC2 instance configuration
- Install software/services on launch
- Set up web servers, databases, agents
- Initialize instance without manual steps
- Infrastructure as Code approach

**Critical Limitation:** ⚠️ **CloudFormation considers stack creation SUCCESS even if user data script FAILS**. Separate mechanisms needed to validate script execution.

---

## Key Concepts

**Base64 Function:**
```yaml
UserData: !Base64 |
  #!/bin/bash
  # Your script here
  command1
  command2
```

The `Base64` function encodes the multi-line script. CloudFormation passes encoded version to EC2, which decodes and executes it.

**Multi-Line String Syntax:**
```yaml
UserData: !Base64 |
  # The "|" means preserve newlines (literal multi-line block)
  #!/bin/bash
  line1
  line2
```

**Log Location:** `/var/log/cloud-init-output.log`
- Contains full execution output of user data script
- Useful for debugging why script failed
- Accessible via SSH to running instance

**Script Execution:**
- Runs as root user
- Runs once on first launch only
- Execution time varies (can take several minutes)
- Errors in script don't fail CloudFormation stack

---

## Real-World Example: Web Server Deployment

**Scenario:** Launch EC2 instance with Apache web server and custom webpage in a single CloudFormation operation.

**Complete Template:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'EC2 Instance with Apache Web Server via UserData'

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    Description: EC2 instance type
  
  ImageId:
    Type: AWS::EC2::Image::Id
    Description: AMI ID (pre-selected for your region)

Resources:
  # Security group allowing SSH and HTTP
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'Allow SSH and HTTP traffic'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
          Description: 'SSH from anywhere'
        
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          Description: 'HTTP from anywhere'

  # EC2 Instance with UserData
  WebServerInstance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref ImageId
      InstanceType: !Ref InstanceType
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      
      # UserData script runs on first launch
      UserData: !Base64 |
        #!/bin/bash
        set -e  # Exit on error (optional, helps debugging)
        
        # Update system packages
        dnf update -y
        
        # Install Apache web server
        dnf install -y httpd
        
        # Start Apache service
        systemctl start httpd
        
        # Enable Apache to start on reboot
        systemctl enable httpd
        
        # Create custom homepage
        cat > /var/www/html/index.html << 'EOF'
        <html>
          <head><title>Welcome</title></head>
          <body>
            <h1>Hello World from CloudFormation UserData!</h1>
            <p>Instance launched: $(date)</p>
          </body>
        </html>
        EOF

Outputs:
  InstancePublicIP:
    Value: !GetAtt WebServerInstance.PublicIp
    Description: 'Public IP of web server'
  
  WebURL:
    Value: !Sub 'http://${WebServerInstance.PublicIp}'
    Description: 'URL to access web server'
```

**Deployment & Verification:**

1. **Deploy stack:** CloudFormation creates security group, launches EC2 instance
2. **Script execution:** UserData script runs as instance starts (visible as "running" in EC2 console)
3. **Script completion:** Takes 30-60 seconds depending on what scripts do
4. **Verify:** Access public IP in browser → Apache homepage loads
5. **Debug:** SSH to instance, run `cat /var/log/cloud-init-output.log` to see execution output

**Execution Log Output Example:**
```
Cloud-init v. 0.98 running 'init' at Mon Jan 15 10:23:45 UTC 2024. Started.
...
+ dnf update -y
Updating Packages
Complete!

+ dnf install -y httpd
Installing packages
Complete!

+ systemctl start httpd
+ systemctl enable httpd

+ cat /var/www/html/index.html
...
Cloud-init v. 0.98 running 'final' at Mon Jan 15 10:24:12 UTC 2024. Finished.
```

---

## Critical Limitation: CloudFormation Success vs Script Success

**Important:** CloudFormation stack creation completes successfully EVEN IF user data script fails.

```yaml
Resources:
  Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-12345
      UserData: !Base64 |
        #!/bin/bash
        EXIT_CODE_COMMAND_THAT_FAILS=$(false)  # This fails!
        # But CloudFormation still marks stack as CREATE_COMPLETE
```

**Problem:** Stack shows "CREATE_COMPLETE" → User assumes everything works → Web server not running

**Solutions (covered in next lecture):**
1. Use CloudFormation `cfn-signal` to indicate success/failure
2. Use CloudFormation `WaitCondition` to wait for signal
3. Use external monitoring to verify

**For debugging:** Always check `/var/log/cloud-init-output.log` to verify script actually ran successfully

---

## Best Practices

✓ **Always use `#!/bin/bash` shebang** — Explicitly specify interpreter  
✓ **Use `set -e` flag** — Exit script on first error (helps catch failures)  
✓ **Update package manager first** — `dnf update -y` before installing packages  
✓ **Enable services for reboot** — `systemctl enable` so services restart after reboot  
✓ **Check `/var/log/cloud-init-output.log`** — Essential debugging when things fail  
✓ **Keep scripts simple** — Long complex scripts harder to debug  
✓ **Add descriptive comments** — Explain what each section does  
✓ **Test script locally first** — Run on test instance before adding to template  
✓ **Use proper quoting** — Quote variables like `"${VAR}"` to avoid expansion issues  
✓ **Redirect output if needed** — `2>&1` to capture stderr for log visibility

---

## SysOps Exam Focus

**Q1: "What function must wrap UserData script in CloudFormation?"**
- A) String()
- B) Sub()
- C) Base64()
- D) Encode()
- **Answer: C** — Base64() encodes script for CloudFormation EC2

**Q2: "Where can you view UserData script execution output?"**
- A) EC2 console
- B) `/var/log/cloud-init-output.log` on instance
- C) CloudFormation console
- D) AWS Systems Manager
- **Answer: B** — SSH to instance and check cloud-init log

**Q3: "Does CloudFormation stack fail if UserData script fails?"**
- A) Yes, stack creation fails
- B) No, but instance may not be configured correctly
- C) It depends on error type
- D) Only if using cfn-signal
- **Answer: B** — CloudFormation succeeds regardless of script outcome (critical exam point)

**Q4: "UserData script runs:"**
- A) Every time instance starts
- B) Only on first launch after creation
- C) When you manually trigger it
- D) Only if using cfn-signal
- **Answer: B** — Runs once on initial launch, not after reboots

**Q5: "To ensure UserData script success fails the CloudFormation stack, use:"**
- A) Base64 function
- B) cfn-signal with CloudFormation::WaitCondition
- C) UserDataValidation property
- D) AWS::CloudInitSuccess resource
- **Answer: B** — cfn-signal signals completion, WaitCondition waits for it (next lecture topic)

**Q6: "Multi-line UserData script in YAML uses:"**
- A) `>` (folded block)
- B) `|` (literal block)
- C) `"` quotes
- D) `\n` character
- **Answer: B** — Pipe `|` preserves newlines for multi-line scripts

---

## Common Scenarios

| Scenario | UserData Needed? | Considerations |
|----------|-----------------|-----------------|
| Install web server | Yes | Use Base64, enable service, open port 80 |
| Resize EBS volume | No | Can't be done in UserData, use different approach |
| Change instance settings | Depends | Simple settings in UserData, complex settings use Systems Manager |
| Pull secrets from Secrets Manager | Yes | Script can retrieve via AWS CLI |
| Set CloudWatch alarms | Depends | Can install agent in UserData |
| Join Windows domain | Yes | Requires appropriate UserData syntax |
| Join EC2 to Auto Scaling group | Not needed | Auto Scaling handles lifecycle |

---

## Next Topic Preview

The current CloudFormation limitation is that stack creation succeeds even if UserData fails. The next lecture covers:
- **cfn-signal:** Script signals completion status to CloudFormation
- **WaitCondition:** CloudFormation waits for signal before marking stack complete
- **Result:** Stack fails if UserData execution fails (integration with CloudFormation success)

---

**Total Words: ~2,600**  
**File: 19_CloudFormation_EC2_UserData.md**  
**Status: SysOps-focused, exam-ready, concise format**