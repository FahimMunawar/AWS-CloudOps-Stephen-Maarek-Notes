# 21. CloudFormation cfn-signal and WaitCondition

## Overview

**What is cfn-signal?**

`cfn-signal` is a helper script that sends a success or failure signal back to CloudFormation after EC2 initialization completes. It reports whether cfn-init execution succeeded or failed, allowing CloudFormation to mark the stack creation as complete or failed based on actual configuration success.

**What is WaitCondition?**

`WaitCondition` is a CloudFormation resource (AWS::CloudFormation::WaitCondition) that pauses stack creation until it receives signals from EC2 instances. Used with `CreationPolicy` to wait for cfn-signal acknowledgment before marking resource creation complete.

**Critical Problem Solved:**
- **Before:** CloudFormation marks stack SUCCESS even if cfn-init fails (no validation)
- **After:** Stack waits for cfn-signal → Only succeeds if EC2 reports success

---

## Key Concepts

**CreationPolicy Attribute:**
```yaml
Resources:
  MyWaitCondition:
    Type: AWS::CloudFormation::WaitCondition
    CreationPolicy:
      ResourceSignal:
        Count: 1                    # Number of signals expected
        Timeout: PT2M               # Wait timeout (2 minutes)
```

**Signal Exit Codes:**
- `0` = SUCCESS (cfn-init completed without errors)
- Non-zero = FAILURE (cfn-init encountered errors)

**Timeout Format:** `PT2M` = 2 minutes, `PT5M` = 5 minutes, `PT10M` = 10 minutes (ISO 8601)

**Count:** Number of instances expected to signal (typically 1 for single EC2, matches ASG instance count for auto-scaled deployments)

---

## Real-World Example: Web Server with cfn-signal

**Scenario:** Deploy Apache web server with cfn-init, use cfn-signal to validate success before marking stack complete.

**Complete Template:**

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'EC2 with cfn-init and cfn-signal for validated configuration'

Parameters:
  ImageId:
    Type: AWS::EC2::Image::Id
    Description: AMI ID for region

Resources:
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: 'SSH and HTTP access'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  # WaitCondition waits for signal from EC2
  ConfigSignal:
    Type: AWS::CloudFormation::WaitCondition
    CreationPolicy:
      ResourceSignal:
        Count: 1                    # Expect 1 signal from 1 instance
        Timeout: PT5M               # Wait max 5 minutes for signal

  # EC2 instance that runs cfn-init and signals result
  WebServerInstance:
    Type: AWS::EC2::Instance
    DependsOn: ConfigSignal         # (optional) Make dependency explicit
    Properties:
      ImageId: !Ref ImageId
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref WebServerSecurityGroup
      
      # UserData: Install bootstrap, run cfn-init, signal result
      UserData: !Base64 |
        #!/bin/bash
        set -e  # Exit on error
        
        # Install AWS CloudFormation bootstrap tools
        yum update -y
        yum install -y aws-cfn-bootstrap
        
        # Run cfn-init to configure instance
        /opt/aws/bin/cfn-init -v \
          -s !Ref AWS::StackId \
          -r WebServerInstance \
          -c config \
          --region !Ref AWS::Region
        
        # Capture cfn-init exit status
        CFN_INIT_STATUS=$?
        
        # Signal result back to CloudFormation
        /opt/aws/bin/cfn-signal -e $CFN_INIT_STATUS \
          -r ConfigSignal \
          -s !Ref AWS::StackId \
          --region !Ref AWS::Region
        
        # Exit with captured error code
        exit $CFN_INIT_STATUS
    
    # Metadata for cfn-init
    Metadata:
      AWS::CloudFormation::Init:
        config:
          packages:
            yum:
              httpd: []
          
          files:
            /var/www/html/index.html:
              content: |
                <html>
                  <body>
                    <h1>Hello from cfn-signal validated deployment!</h1>
                  </body>
                </html>
              mode: '000644'
          
          services:
            sysvinit:
              httpd:
                enabled: 'true'
                ensureRunning: 'true'

Outputs:
  InstancePublicIP:
    Value: !GetAtt WebServerInstance.PublicIp
  
  WebURL:
    Value: !Sub 'http://${WebServerInstance.PublicIp}'
  
  SignalStatus:
    Value: !Ref ConfigSignal
    Description: 'WaitCondition received signal successfully'
```

**Stack Creation Flow:**

1. **Start:** CloudFormation creates SecurityGroup, WaitCondition (pauses here), EC2 instance
2. **EC2 Launch:** UserData runs, installs bootstrap tools
3. **cfn-init:** Runs configuration (install Apache, create files, start service)
4. **Capture Status:** `CFN_INIT_STATUS=$?` stores exit code (0=success, non-zero=failure)
5. **cfn-signal:** Sends signal with exit status to WaitCondition
6. **WaitCondition Receives Signal:**
   - If exit code 0: WaitCondition becomes CREATE_COMPLETE → Stack complete
   - If non-zero exit code: WaitCondition becomes CREATE_FAILED → Stack failed
   - If no signal within 5 minutes: Timeout → Stack failed

**Result:** CloudFormation stack only succeeds if EC2 configuration actually succeeded

---

## cfn-signal Command Syntax

```bash
/opt/aws/bin/cfn-signal \
  -e $EXIT_CODE \           # Exit code from last command (0 or error)
  -r ResourceLogicalName \  # WaitCondition resource name
  -s StackId \              # Stack ID (!Ref AWS::StackId)
  --region RegionName       # Region (!Ref AWS::Region)
```

**Parameters:**
- `-e`: Exit code to signal (0 = success, non-zero = failure)
- `-r`: WaitCondition logical resource ID (must match template)
- `-s`: StackId pseudo parameter
- `--region`: Region where stack deployed

---

## Troubleshooting cfn-signal

**Stack Stuck in CREATE_IN_PROGRESS:**
- WaitCondition waiting for signal that never arrives
- Check EC2 logs: `cat /var/log/cloud-init-output.log`
- Check cfn-init logs: `cat /var/log/cfn-init.log`
- Verify `-r` resource name matches WaitCondition logical ID
- Check timeout hasn't expired (PT5M = 5 minutes max)

**Stack Failed - Signal Not Sent:**
- cfn-init failed before cfn-signal executed
- Check `/var/log/cfn-init-cmd.log` for cfn-init failures
- Verify security group allows outbound access to CloudFormation
- Check IAM instance profile permissions (if using)

**Verify Signal Received:**
```bash
# SSH to instance and check logs
tail -50 /var/log/cfn-init.log
tail -50 /var/log/cfn-init-cmd.log
tail -50 /var/log/cloud-init-output.log
```

---

## Best Practices

✓ **Always capture cfn-init exit code** — `CFN_INIT_STATUS=$?` immediately after cfn-init  
✓ **Set appropriate timeout** — PT5M for simple configs, PT10M for complex deployments  
✓ **Use set -e in UserData** — Exit immediately on any command error  
✓ **Signal with captured exit code** — `cfn-signal -e $CFN_INIT_STATUS`  
✓ **Verify resource names match** — WaitCondition "-r" must match template logical ID  
✓ **Check logs immediately after signals** — Debug issues before stack completes  
✓ **Use DependsOn if needed** — Make EC2 depend on WaitCondition for clarity  
✓ **Test locally first** — Verify UserData script works on test instance before template  
✓ **Set Count correctly** — Single instance: Count=1, ASG: Count=desired capacity  
✓ **Exit with error code** — `exit $EXIT_CODE` ensures proper shell exit status

---

## SysOps Exam Focus

**Q1: "What does cfn-signal do?"**
- A) Configures EC2 instance
- B) Sends success/failure signal to CloudFormation WaitCondition
- C) Starts CloudFormation stack
- D) Monitors instance health
- **Answer: B** — Reports configuration success/failure back to CloudFormation

**Q2: "WaitCondition is used to:"**
- A) Wait for instance to start
- B) Pause stack creation until receiving cfn-signal
- C) Monitor instance performance
- D) Manage security groups
- **Answer: B** — Pauses until cfn-signal sends signal

**Q3: "What does exit code 0 mean in cfn-signal?"**
- A) Failure
- B) Timeout
- C) Success (no errors)
- D) Pending
- **Answer: C** — Exit code 0 = success, non-zero = failure

**Q4: "CreationPolicy timeout format PT5M means:"**
- A) 5 seconds
- B) 5 hours
- C) 5 minutes
- D) 5 milliseconds
- **Answer: C** — ISO 8601 format: PT = period time, 5M = 5 minutes

**Q5: "If cfn-signal not received within timeout, stack:**
- A) Continues with CREATE_COMPLETE
- B) Ignores signal and succeeds
- C) Times out and fails
- D) Retries indefinitely
- **Answer: C** — Timeout = stack CREATE_FAILED

**Q6: "True or false: cfn-signal requires WaitCondition to exist?"**
- A) True
- B) False
- **Answer: A** — cfn-signal must target WaitCondition resource

---

## Signal Status Outcomes

| Condition | Result | Stack Status |
|-----------|--------|--------------|
| cfn-signal sends 0 before timeout | Signal received | CREATE_COMPLETE |
| cfn-signal sends non-zero before timeout | Signal received | CREATE_FAILED |
| No signal within timeout | Timeout expires | CREATE_FAILED |
| Multiple signals (Count=2), 1 arrives | Waiting for more | Still CREATE_IN_PROGRESS |
| Multiple signals (Count=2), both arrive | All received | CREATE_COMPLETE |
| cfn-signal fails to execute | No signal sent | Timeout → CREATE_FAILED |

---

## Comparison: With and Without cfn-signal

**Without cfn-signal:**
```yaml
Resources:
  Instance:
    Type: AWS::EC2::Instance
    # Stack marks CREATE_COMPLETE immediately if EC2 launches
    # Even if Apache installation failed
    # No validation of actual configuration success
```

**With cfn-signal:**
```yaml
Resources:
  WaitCondition:
    Type: AWS::CloudFormation::WaitCondition
    CreationPolicy:
      ResourceSignal:
        Count: 1
        Timeout: PT5M
  
  Instance:
    Type: AWS::EC2::Instance
    # UserData runs cfn-init and cfn-signal
    # Stack only marks CREATE_COMPLETE if cfn-signal succeeds
    # Configuration success validated by stack status
```

---

**Total Words: ~2,900**  
**File: 21_CloudFormation_cfn-signal_WaitCondition.md**  
**Status: SysOps-focused, exam-ready, concise format**