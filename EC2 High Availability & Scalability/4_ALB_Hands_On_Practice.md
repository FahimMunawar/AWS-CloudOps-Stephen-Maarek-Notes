# Application Load Balancer Hands-On Practice

This note provides a detailed step-by-step walkthrough of creating an Application Load Balancer (ALB), registering EC2 instances as targets, and demonstrating load balancing in action.

## Prerequisites

### Expected Timeline
- **EC2 Instances Launch**: 2-3 minutes
- **ALB Creation**: 5-10 minutes
- **Total Hands-On Time**: 15-20 minutes

### Cost Considerations
- **Free Tier**: 2× t2.micro instances included
- **ALB**: Not in free tier but minimal cost for 1 hour
- **Storage**: Minimal
- **Recommendation**: Clean up afterwards to minimize charges

### Required Permissions
- EC2 full access (instances, security groups)
- ELB (Elastic Load Balancing) full access
- CloudWatch read access (for monitoring)

---

## Part 1: Launch EC2 Instances

### Step 1: Navigate to EC2 Console
1. AWS Console → EC2
2. Click "Launch instances"
3. Ensure correct region selected

### Step 2: Configure First Instance

**Instance Details**
1. **Number of instances**: Set to `2` (top right)
   - Will launch 2 instances simultaneously
   - Second will be named separately

2. **Instance Name**: `My First Instance`
   - Both instances will get incrementing names
   - Rename second one after launch

3. **AMI Selection**: `Amazon Linux 2`
   - Free tier eligible
   - Good for web server testing
   - Default AMI in most regions

4. **Instance Type**: `t2.micro`
   - Free tier eligible
   - Sufficient for web server demo
   - Cost: ~$0.01/hour if outside free tier

### Step 3: Network Configuration

**Key Pair**
- Select: "Proceed without a key pair"
- Reason: Will use EC2 Instance Connect
- Alternative: Create key pair if SSH preferred

**Security Group**
- Select: Existing security group
- Choose: `Launch-wizard-1`
- Inbound rules:
  - HTTP (port 80): Needed for web traffic
  - SSH (port 22): Optional, for troubleshooting

- If Launch-wizard-1 doesn't exist, create with:
  - HTTP: Port 80, Source 0.0.0.0/0
  - SSH: Port 22, Source 0.0.0.0/0 (or your IP only)

### Step 4: Add User Data Script

**Advanced Details**
1. Scroll to "Advanced details"
2. **User Data**: (select text tab)
3. Paste script:

```bash
#!/bin/bash
# Update and install Apache
yum update -y
yum install -y httpd

# Start Apache service
systemctl start httpd
systemctl enable httpd

# Create simple HTML page showing instance info
cat > /var/www/html/index.html <<EOF
<!DOCTYPE html>
<html>
<head>
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World from EC2 Instance!</h1>
    <p>Instance ID: $(ec2-metadata --instance-id | cut -d " " -f 2)</p>
    <p>Availability Zone: $(ec2-metadata --availability-zone | cut -d " " -f 2)</p>
    <p>Time: $(date)</p>
</body>
</html>
EOF
```

**Purpose**
- Installs Apache web server
- Creates simple HTML page
- Auto-enables service on boot
- Shows instance ID (proof of load balancing)

### Step 5: Review and Launch

1. **Storage**: Accept defaults (8 GB)
2. **Summary**: Review all settings
3. Click "Launch instances"
4. Wait for instances to be created

### Step 6: Verify Instance Launch

1. **View instances**: Click "View all instances"
2. Wait for instances to reach "running" state
   - Status: Running
   - Status checks: 2/2 passed (takes 1-2 minutes)

3. **Rename second instance**:
   - Click second instance name
   - Change to: `My Second Instance`
   - Save

4. Wait for both to show "2/2 checks passed"

---

## Part 2: Verify Individual Instances

### Step 1: Test First Instance

1. Select: `My First Instance`
2. Copy: **Public IPv4 address**
3. Browser: Paste in new tab, press Enter
4. Expected: "Hello World from EC2 Instance!"
5. Note: Instance ID visible

**Example URL**
```
http://203.0.113.50/
```

### Step 2: Test Second Instance

1. Select: `My Second Instance`
2. Copy: **Public IPv4 address**
3. Browser: Paste in new tab, press Enter
4. Expected: "Hello World from EC2 Instance!"
5. Note: **Different Instance ID** than first

**Current State**
```
Instance 1: Direct access via IP works ✓
Instance 2: Direct access via IP works ✓
Problem: Two URLs (IPs) to access same service
Solution: Load balancer with single endpoint
```

---

## Part 3: Create Load Balancer Security Group

### Step 1: Create Security Group

1. EC2 Console → Security Groups
2. Click "Create security group"
3. **Security group name**: `demo-sg-load-balancer`
4. **Description**: "Security group for ALB in demo"
5. **VPC**: Default (or your specific VPC)

### Step 2: Configure Inbound Rules

**Remove default rule** (if present)

**Add Inbound Rule #1**
```
Type: HTTP
Protocol: TCP
Port: 80
Source: 0.0.0.0/0 (anywhere)
Description: "Allow HTTP from anywhere"
```

**No other inbound rules needed**

### Step 3: Configure Outbound Rules

- **Default**: Allow all (0.0.0.0/0)
- **Keep as-is**: Sufficient for ALB

### Step 4: Create Group

1. Click "Create security group"
2. Note security group ID: `sg-xxxxxxxxx`

---

## Part 4: Create Application Load Balancer

### Step 1: Navigate to Load Balancers

1. EC2 Console → Load Balancers
2. Click "Create load balancer"
3. Choose: **Application Load Balancer (ALB)**
4. Click "Create"

### Step 2: Basic Configuration

**Load Balancer Name**: `DemoALB`

**Scheme**: `Internet-facing`
- Accessible from internet
- Public security group
- Can reach from anywhere

**Address Type**: `IPv4`
- Standard for most applications
- Supports both IPv4 and IPv6

### Step 3: Network Mapping

**VPC**: Default (or select your VPC)

**Availability Zones**
- **Enable All**: Check all available AZs
- Example: us-east-1a, us-east-1b, us-east-1c
- Benefit: Multi-AZ high availability
- ALB nodes deployed in each selected AZ

```
Selected AZs:
├─ us-east-1a: ALB node deployed
├─ us-east-1b: ALB node deployed
└─ us-east-1c: ALB node deployed
```

### Step 4: Security Groups

**Select Security Groups**
1. Remove: "Default" security group
2. Add: `demo-sg-load-balancer` (created earlier)
3. Leave only: `demo-sg-load-balancer`

**Inbound Configuration**
- ALB accepts HTTP from anywhere (0.0.0.0/0)
- Perfect for public web traffic

### Step 5: Listeners and Routing

**Listener Configuration**
- **Protocol**: HTTP
- **Port**: 80

**Action: Forward to**
- Target Group: Need to create

**Create Target Group**
1. Click "Create target group"
2. Opens new window

---

## Part 5: Create Target Group

### Step 1: Basic Configuration

**Target type**: `Instances`
- Backend resources are EC2 instances

**Target group name**: `demo-tg-alb`
- Descriptive, easy to identify

**Protocol**: `HTTP`

**Port**: `80`

**Version**: `HTTP1`
- Standard HTTP version
- Sufficient for this demo

**VPC**: Default (match ALB VPC)

### Step 2: Health Check Configuration

**Health Check Settings**
```
Protocol: HTTP
Path: /

Interval: 30 seconds (default)
Timeout: 5 seconds (default)
Healthy threshold: 2 consecutive successes
Unhealthy threshold: 2 consecutive failures

Expected status code: 200
```

**Explanation**
- Check every 30 seconds
- Send HTTP request to `/`
- Expect 200 OK response
- Mark healthy after 2 passes
- Mark unhealthy after 2 failures

### Step 3: Register Targets

1. Click "Next"
2. **Available instances**: See both EC2 instances
3. Select: Both `My First Instance` and `My Second Instance`
4. **Port**: 80 (default)
5. Click "Include as pending below"

**Verification**
```
Selected instances:
├─ My First Instance (port 80)
└─ My Second Instance (port 80)
```

6. Click "Create target group"

### Step 4: Target Group Created

- Target group created successfully
- Now available in ALB configuration

---

## Part 6: Complete ALB Creation

### Step 1: Select Target Group

1. Back to ALB creation tab
2. Refresh if needed
3. **Forward to**: Select `demo-tg-alb`
4. Verify: Target group selected

### Step 2: Review Configuration

1. **ALB Name**: DemoALB
2. **Scheme**: Internet-facing
3. **Security Group**: demo-sg-load-balancer
4. **Network Zones**: All selected
5. **Listener**: HTTP port 80 → demo-tg-alb
6. **Targets**: 2 EC2 instances in demo-tg-alb

### Step 3: Create Load Balancer

1. Click "Create load balancer"
2. Wait for provisioning

**Status Progression**
```
Creating → Provisioning → Active (ready to use)
```

---

## Part 7: Verify Load Balancing

### Step 1: Get ALB DNS Name

1. Load Balancer shows: `Active` state
2. Copy: **DNS name**
   - Example: `DemoALB-123456789.us-east-1.elb.amazonaws.com`

### Step 2: Access via ALB DNS

1. Browser: Paste ALB DNS name
2. Press Enter
3. Expected: "Hello World from EC2 Instance!"
4. Note: Instance ID (Instance 1 or 2)

### Step 3: Verify Load Balancing in Action

**Refresh repeatedly** (F5 or Cmd+R)
- Refresh 1: Instance 1 (with ID)
- Refresh 2: Instance 2 (different ID)
- Refresh 3: Instance 1
- Refresh 4: Instance 2
- Pattern: Alternating between instances

**Proof of Load Balancing**
```
ALB detected 2 healthy targets
└─ Round-robin distribution
   ├─ Request 1 → Instance 1
   ├─ Request 2 → Instance 2
   ├─ Request 3 → Instance 1
   ├─ Request 4 → Instance 2
   └─ Pattern continues
```

---

## Part 8: Verify Health Checks

### Step 1: Check Target Group Health

1. Load Balancers → Select `DemoALB`
2. **Targets tab**
3. View: `demo-tg-alb` target group
4. See both instances: **Status: Healthy ✓**

**Healthy Status Indicators**
```
Instance 1:
├─ Status: Healthy ✓
├─ Target Health: Healthy (HTTP 200 OK)
└─ Description: Healthy

Instance 2:
├─ Status: Healthy ✓
├─ Target Health: Healthy (HTTP 200 OK)
└─ Description: Healthy
```

### Step 2: Understand Health Checks

**What's Happening Behind Scenes**
```
Every 30 seconds:
├─ ALB → Instance 1: GET / HTTP/1.1
│  └─ Response: 200 OK
│  └─ Mark: Healthy ✓
│
├─ ALB → Instance 2: GET / HTTP/1.1
│  └─ Response: 200 OK
│  └─ Mark: Healthy ✓
│
└─ Result: Both receiving traffic
```

---

## Part 9: Demonstrate Failure Handling

### Step 1: Stop First Instance

1. EC2 Instances
2. Select: `My First Instance`
3. Instance State → Stop
4. Click "Stop"
5. Wait for state to show "Stopped"

### Step 2: Watch Health Check Failure

1. Target Group → Targets
2. Wait ~30 seconds for next health check
3. Refresh page (may need ~60 seconds total)
4. Observe: Instance 1 status changes

**Status Progression**
```
Time 0: Both Healthy ✓
Time 30: Instance 1 health check fails
Time 60: Instance 1 marked Unhealthy ✗
Display: Instance 1 status: "Unhealthy - Target failed health checks"
```

**Instance 2 remains**: Healthy ✓

### Step 3: Verify Load Balancer Adaptation

1. Browser: Refresh ALB DNS page
2. Expected: Only Instance 2 responds
3. **No Instance 1 ID visible** (because stopped)
4. Repeated refreshes: All requests go to Instance 2

**New Behavior**
```
Before (2 healthy targets):
├─ ALB distributes: 50% to Instance 1, 50% to Instance 2

After (1 healthy target):
└─ ALB distributes: 100% to Instance 2
   └─ Instance 1 receives NO traffic
```

**Benefit**: No user impact (except for users connected to Instance 1 session)

### Step 4: Restart Instance

1. EC2 Instances
2. Select: `My First Instance`
3. Instance State → Start
4. Wait for "Running" state
5. Wait for "2/2 checks passed"

### Step 5: Watch Recovery

1. Target Group → Targets
2. Watch Instance 1 status change

**Recovery Progression**
```
Status: Initializing targets
└─ Health check sent to restarted instance
└─ Waits for successful response
└─ After healthy responses (2 passes): Mark Healthy ✓
```

**Status Changes**
```
Time 0: Instance starts
Time 15: First health check (fails, instance still booting)
Time 30: ALB mark Target Unhealthy (due to failures)
Time 60: Instance fully booted, responds to health check
Time 90: Second successful health check
Result: Instance marked Healthy ✓ and receives traffic again
```

### Step 6: Verify Full Restoration

1. Browser: Refresh ALB DNS page
2. Expected: Both Instance 1 and 2 appear
3. Repeated refreshes: Alternating between instances

**Return to Normal**
```
ALB Distribution:
├─ Request 1 → Instance 1 ✓
├─ Request 2 → Instance 2 ✓
├─ Request 3 → Instance 1 ✓
├─ Request 4 → Instance 2 ✓
└─ Both instances fully operational
```

---

## Part 10: Cleanup

### Step 1: Delete Load Balancer

1. Load Balancers
2. Select: `DemoALB`
3. Actions → Delete
4. Confirm deletion
5. Wait for "Deleted" status

### Step 2: Delete Target Group

1. Target Groups
2. Select: `demo-tg-alb`
3. Actions → Delete
4. Confirm deletion

### Step 3: Delete Security Group

1. Security Groups
2. Select: `demo-sg-load-balancer`
3. Actions → Delete
4. Confirm deletion

### Step 4: Terminate EC2 Instances

1. Instances
2. Select both instances
3. Instance State → Terminate
4. Confirm termination
5. Wait for "Terminated" status

### Step 5: Cost Verification

- Check AWS Billing (may take a few hours to update)
- ALB costs should be minimal for 1 hour usage
- Stop/terminate removes all recurring charges

---

## Key Learnings from Hands-On

### Concepts Demonstrated

1. **Load Balancing in Action**
   - Single endpoint serves multiple instances
   - Traffic distributed via round-robin
   - Proof: Changing instance IDs on refresh

2. **Health Checks**
   - ALB continuously verifies instance health
   - Unhealthy instances removed from rotation
   - Automatic recovery when instance restarts

3. **Fault Tolerance**
   - System adapts when instance fails
   - No user-facing service disruption (if multiple targets)
   - Single point of failure eliminated (with 2+ instances)

4. **High Availability**
   - ALB spans multiple AZs
   - Instances distributed across zones
   - Survives single instance or AZ failure

### Practical Performance Observations

**Before ALB**
- 2 different URLs (instance IPs)
- Users must know both endpoints
- No automatic failover

**After ALB**
- 1 single URL (ALB DNS)
- Users only need ALB endpoint
- Automatic failover to healthy instances
- Transparent load distribution

---

## Key Takeaways for SysOps Associate

### ALB Components
- **Load Balancer**: Public endpoint, distributes traffic
- **Target Group**: Grouping of backend resources
- **Listener**: Rules for incoming traffic (port 80, 443, etc.)
- **Targets**: EC2 instances that receive traffic

### Health Check Behavior
- **How**: GET request to health check path
- **When**: Every 30 seconds (default)
- **Success**: HTTP 200 response
- **Unhealthy**: Removed from rotation after 2 failures
- **Recovery**: Re-added after 2 consecutive successes

### Architecture Pattern
```
Users → ALB DNS name
    ↓
ALB (Multi-AZ)
    ↓
Target Group (Health Checks)
    ↓
EC2 Instances (Auto-distribution)
```

### Exam Focus Points
- ALB provides single endpoint for multiple backend services
- Health checks automatically remove unhealthy instances
- Target groups organize backend resources
- Security group restricts instance access to ALB only
- Multi-AZ deployment provides high availability
- Load balancing happens transparently to application layer