# NLB Hands-On Practice: Creating and Testing a Network Load Balancer

This hands-on guide walks through creating a functional Network Load Balancer, configuring target groups, managing security groups, and verifying load balancing functionality in AWS.

---

## Part 1: Prerequisites

### Required Resources

**Before Starting**
```
✓ AWS account with appropriate permissions
✓ VPC with at least 2 subnets in different AZs
✓ 2+ EC2 instances running in the VPC
✓ Application running on instances (HTTP on port 80)
  └─ Example: Simple web server returning "Hello World"
✓ Existing security groups for EC2 instances
✓ Familiarity with AWS Console navigation
```

**Recommended Setup**
```
- VPC: Default VPC or custom VPC
- Subnets: 3 AZs enabled for high availability
- Instances: 2+ instances with:
  ├─ Running HTTP server (port 80)
  ├─ Security groups allowing SSH
  └─ EC2 Instance Connect or SSH access
- Security Groups: Instance SG already created
```

---

## Part 2: Create Network Load Balancer

### Step 1: Navigate to Load Balancers

**AWS Console Navigation**
```
1. Open AWS Management Console
2. Search or navigate: EC2 → Load Balancing → Load Balancers
3. Click "Create load balancer" button
```

### Step 2: Select NLB Type

**Load Balancer Selection**
```
Load Balancer Types Displayed:
├─ Application Load Balancer
├─ Network Load Balancer ← SELECT THIS
├─ Classic Load Balancer
└─ Gateway Load Balancer

Action: Click "Create" button under Network Load Balancer
```

### Step 3: Configure NLB Basic Settings

**Basic Configuration Page**

```
Field: Load balancer name
Input: DemoNLB
Purpose: Identifies your NLB in console

Field: Scheme
Options:
├─ Internet-facing ← SELECT (public access)
└─ Internal (VPC only)
Purpose: Internet-facing allows public internet access

Field: IP address type
Options:
├─ IPv4 ← SELECT
└─ Dualstack (IPv4 + IPv6)
Purpose: IPv4 for standard AWS setup
```

**Configuration Example**
```
Load balancer name: DemoNLB
Scheme: Internet-facing
IP address type: IPv4
```

### Step 4: Configure Network Mapping

**VPC and Subnet Selection**

```
Field: VPC
Action: Select your VPC from dropdown
Example: vpc-12345678 (default)
Purpose: Network where NLB operates

Field: Availability Zones
Instruction: Select subnets in multiple AZs
Example:
├─ us-east-1a: subnet-1a (Primary)
├─ us-east-1b: subnet-1b (Secondary)
└─ us-east-1c: subnet-1c (Tertiary)
Purpose: Spreads load across 3 availability zones
```

**Static IP Assignment**

```
AWS Automatic IP Assignment:
├─ AZ: us-east-1a
│  └─ IPv4 Address: 203.0.113.10 (automatically assigned)
│
├─ AZ: us-east-1b
│  └─ IPv4 Address: 203.0.113.20 (automatically assigned)
│
└─ AZ: us-east-1c
   └─ IPv4 Address: 203.0.113.30 (automatically assigned)

Key Point: Each AZ gets ONE static IPv4 address
This is the fundamental NLB feature for firewall whitelisting
```

**Optional: Assign Elastic IPs**

```
If you want predictable Elastic IPs instead:

1. Pre-allocate Elastic IPs (3 for 3 AZs)
   AWS Console → EC2 → Elastic IPs → Allocate

2. In NLB creation, click "Add/Edit" for each subnet

3. Select Elastic IP instead of AWS-assigned IP
   ├─ AZ: us-east-1a → EIP: eip-12345678
   ├─ AZ: us-east-1b → EIP: eip-87654321
   └─ AZ: us-east-1c → EIP: eip-11223344

Result: NLB uses your specific Elastic IPs
Benefit: Same IPs even if NLB is recreated
```

**Recommended Configuration**
```
Network mapping:
├─ VPC: vpc-12345678 (default)
├─ Subnets:
│  ├─ ☑ us-east-1a: subnet-1a (203.0.113.10)
│  ├─ ☑ us-east-1b: subnet-1b (203.0.113.20)
│  └─ ☑ us-east-1c: subnet-1c (203.0.113.30)
└─ Elastic IPs: (optional, leave as AWS-assigned for labs)
```

### Step 5: Configure Security Group

**Create Security Group (if needed)**

```
Option A: Use existing security group
├─ If you have NLB-specific SG: Select it
└─ Skip to Step 6

Option B: Create new security group
├─ Choose: No security groups attached
├─ Action: Click "Create a new security group"
│  ├─ Name: demo-sg-nlb
│  ├─ Description: Security group for Network Load Balancer
│  ├─ VPC: Select same VPC as NLB
│  └─ Create
│
└─ Configure inbound rules:
```

**Security Group Inbound Rules**

```
For Public HTTP Load Balancer:
├─ Rule 1:
│  ├─ Type: HTTP
│  ├─ Protocol: TCP
│  ├─ Port Range: 80
│  └─ Source: 0.0.0.0/0 (anywhere - public)
│
└─ Rule 2 (Optional for HTTPS):
   ├─ Type: HTTPS
   ├─ Protocol: TCP
   ├─ Port Range: 443
   └─ Source: 0.0.0.0/0 (anywhere - public)

For Health Checks:
└─ Already allowed if health check uses same port

Outbound Rules:
└─ Default: Allow all (to backends)
```

**Security Group Example**
```
demo-sg-nlb Inbound Rules:
├─ HTTP (80/tcp) from 0.0.0.0/0
├─ HTTPS (443/tcp) from 0.0.0.0/0
└─ (SSH 22/tcp optional for troubleshooting)

Outbound Rules:
└─ All traffic to 0.0.0.0/0
```

### Step 6: Attach Security Group to NLB

**In NLB Creation Wizard**

```
After security group creation:
1. Return to NLB creation page
2. Click "Refresh" button (if needed)
3. Select security group from dropdown
   ├─ Available: demo-sg-nlb
   └─ Remove: default (if listed)
4. Confirm selection
```

**Result**
```
Security groups:
├─ Selected: demo-sg-nlb
│  ├─ HTTP 80 from anywhere
│  └─ HTTPS 443 from anywhere
└─ Inbound traffic to NLB protected by this SG
```

---

## Part 3: Create Target Group

### Step 7: Configure Listener

**Listener Configuration**

```
In NLB creation wizard, "Listeners and routing" section:

Field: Protocol
Options:
├─ TCP ← SELECT for HTTP(port 80)
├─ UDP
├─ TCP_UDP
└─ TLS (HTTPS with encryption)

Field: Port
Input: 80
Purpose: NLB listens on port 80

Default action:
├─ Type: Forward to target group
└─ Target group: (need to create)
```

### Step 8: Create Target Group

**Target Group Creation**

```
Action: Click "Create a new target group" link
```

**Target Group Configuration Page**

```
Step 1: Basic Configuration

Field: Choose a target type
Options:
├─ Instances ← SELECT (for EC2 targets)
├─ IP addresses
├─ Lambda function
└─ Application Load Balancer

Field: Target group name
Input: demo-tg-nlb
Purpose: Identifies target group in console

Field: Protocol
Options:
├─ TCP ← SELECT (for NLB)
├─ UDP
└─ TCP_UDP

Field: Port
Input: 80
Purpose: Instances listen on port 80 for HTTP

Field: IP address type
Options:
├─ IPv4 ← SELECT (for standard setup)
└─ IPv6

Field: VPC
Select: Same VPC as NLB
Purpose: Ensure targets in same network

Field: Protocol version
Default: HTTP/1 (acceptable for HTTP)
```

### Step 9: Configure Health Checks

**Health Check Settings for NLB**

```
Health Check Configuration:

Field: Health check protocol
Options:
├─ TCP (simple connectivity check)
├─ HTTP ← SELECT (application-level check)
└─ HTTPS (encrypted check)

Field: Health check path
Input: / (or /health if endpoint exists)
Purpose: Path for HTTP request
Default: / (root, usually returns 200 OK)

Field: Port
Input: traffic port (shows "80")
Purpose: Same port as backend (port 80)

Field: Interval
Input: 5 seconds (aggressive)
Default: 30 seconds
Purpose: How often to check health
Note: 5 seconds for quick detection in lab, 30 in production

Field: Timeout
Input: 2 seconds (aggressive)
Default: 10 seconds
Purpose: Wait time for response
Note: 2 seconds for labs, higher in production

Field: Healthy threshold
Input: 2
Default: 3
Purpose: Consecutive successes to mark healthy
Aggressive setting: Marks healthy after 2 successful checks

Field: Unhealthy threshold
Input: 2
Default: 3
Purpose: Consecutive failures to mark unhealthy
Aggressive setting: Marks unhealthy after 2 failed checks
```

**Advanced Health Check Settings (Expanded)**

```
Matcher:
├─ Expected status code: ✓ 200
├─ Range: 200-299 (all 2xx responses = healthy)
└─ Won't show error (rarely needed)

Tags (optional):
├─ Key: Purpose
└─ Value: HTTP health check via ALB
```

**Health Check Summary**
```
Health Checks Configured:
├─ Protocol: HTTP
├─ Port: 80
├─ Path: /
├─ Interval: 5 seconds (check every 5 seconds)
├─ Timeout: 2 seconds (wait max 2 seconds)
├─ Healthy threshold: 2 (2 successes = healthy)
├─ Unhealthy threshold: 2 (2 failures = unhealthy)
└─ Expected status: 200 OK

Effect: Takes 10 seconds to mark healthy (2 checks × 5s interval)
```

### Step 10: Register Targets

**Select EC2 Instances**

```
In target group creation, "Register targets" section:

Available instances:
├─ i-1111111111 (10.0.1.10)
└─ i-2222222222 (10.0.1.20)

Action: Check both instances
Result: Both appear in "Selected targets"

Selected targets:
├─ i-1111111111 (port 80)
├─ i-2222222222 (port 80)
└─ Status: Expect Pending initially
```

**Create Target Group**

```
Click "Create target group" button
Result:
├─ Target group created: demo-tg-nlb
├─ Status: Active
├─ Targets: Registering (health checks starting)
└─ Redirect: Back to NLB creation wizard
```

### Step 11: Select Target Group in NLB

**Complete Listener Configuration**

```
Back in NLB creation wizard:

Field: Default action - Forward to target group
Action: Click dropdown and select demo-tg-nlb
Result: demo-tg-nlb selected for listener

Listener Configuration Summary:
├─ Protocol: TCP
├─ Port: 80
├─ Default action: Forward to → demo-tg-nlb
└─ Listener ready
```

---

## Part 4: Review and Create NLB

### Step 12: Review Configuration

**Review Page Summary**

```
Configuration Review:

1. Load balancer details:
   ├─ Name: DemoNLB
   ├─ Type: Network
   ├─ Scheme: Internet-facing
   └─ IP address type: IPv4

2. Network mapping:
   ├─ VPC: vpc-12345678
   ├─ Subnets: 3 selected
   └─ IPs: 203.0.113.10, 203.0.113.20, 203.0.113.30 (static)

3. Security groups:
   ├─ Selected: demo-sg-nlb
   ├─ Inbound: HTTP 80 from anywhere
   └─ Outbound: All traffic allowed

4. Listeners and routing:
   ├─ Listener: TCP port 80
   ├─ Target group: demo-tg-nlb
   └─ Action: Forward

5. Target group details:
   ├─ Name: demo-tg-nlb
   ├─ Protocol: TCP port 80
   ├─ Health check: HTTP / (interval 5s, timeout 2s)
   └─ Targets: i-1111, i-2222 (pending)
```

### Step 13: Create Load Balancer

**Final Creation**

```
Action: Click "Create load balancer" button
Process:
├─ NLB creation in progress
├─ Takes 30-60 seconds
└─ Status: Active

Automatic Redirect:
└─ NLB details page appears
```

---

## Part 5: Monitor NLB and Target Health

### Step 14: Check NLB Status

**NLB Details Page**

```
NLB Information Displayed:
├─ Name: DemoNLB
├─ State: Active ✓
├─ DNS name: network-load-balancer-123456.elb.amazonaws.com
├─ Scheme: internet-facing
├─ VPC: vpc-12345678
└─ Subnets: 3 AZs selected

Network Mapping Details:
├─ Availability Zone 1 (us-east-1a)
│  ├─ Subnet: subnet-1a
│  └─ IPv4 Address: 203.0.113.10 (static)
├─ Availability Zone 2 (us-east-1b)
│  ├─ Subnet: subnet-1b
│  └─ IPv4 Address: 203.0.113.20 (static)
└─ Availability Zone 3 (us-east-1c)
   ├─ Subnet: subnet-1c
   └─ IPv4 Address: 203.0.113.30 (static)
```

### Step 15: Check Target Group Health

**Navigate to Target Group**

```
From NLB page:
1. Menu: Load Balancing → Target Groups
2. Find: demo-tg-nlb
3. Click to open details
```

**Initial Status (First 30 seconds)**

```
Targets Status (Initially):
├─ i-1111111111 (10.0.1.10):
│  ├─ Status: initial (checking)
│  ├─ Description: "Elb.RegistrationInProgress"
│  └─ Health check: In progress
│
└─ i-2222222222 (10.0.1.20):
   ├─ Status: initial (checking)
   ├─ Description: "Elb.RegistrationInProgress"
   └─ Health check: In progress

Timeline:
- t=0s: Targets registered
- t=5s: First health check sent
- t=10s: Second health check sent (if first succeeded)
- t=10s: After 2 successes, marked HEALTHY
```

**Potential Issue: Unhealthy Status**

```
If targets show UNHEALTHY:
├─ Status: unhealthy
├─ Description: "Elb.RegistrationInProgress" or "Health checks failed"
├─ Cause: Security group blocking traffic
└─ Next step: Troubleshoot (see Part 6)
```

---

## Part 6: Troubleshoot Security Groups

### Issue: Targets Unhealthy After Health Checks

**Problem Diagnosis**

```
Symptoms:
├─ Target status: Unhealthy
├─ NLB ready to forward traffic
├─ But health checks failing
├─ Targets don't respond to health checks
└─ Result: NLB can't send traffic

Likely Cause: EC2 instance security group
├─ Blocks incoming traffic from NLB
├─ NLB SG: demo-sg-nlb
├─ Instance SG: launch-wizard-1 (or custom)
└─ No rule allowing NLB SG → Instance SG
```

**View EC2 Instance Security Group**

```
1. AWS Console: EC2 → Instances
2. Select one instance: i-1111111111
3. Details tab: Security groups section
4. Click security group: launch-wizard-1
5. Inbound rules displayed:
   ├─ SSH (22) from 0.0.0.0/0
   ├─ HTTP (80) from demo-sg-alb (ALB only)
   └─ Missing: HTTP (80) from demo-sg-nlb
```

**Current Rules (Insufficient)**

```
Inbound Rules on Instance SG (launch-wizard-1):

Rule 1: SSH
├─ Type: SSH (22)
├─ Protocol: TCP
├─ Port: 22
└─ Source: 0.0.0.0/0 (anywhere)

Rule 2: HTTP (from ALB) - CURRENT
├─ Type: HTTP (80)
├─ Protocol: TCP
├─ Port: 80
└─ Source: demo-sg-alb (ALB security group)

Rule 3: MISSING FOR NLB
├─ Type: HTTP (80)
├─ Protocol: TCP
├─ Port: 80
└─ Source: demo-sg-nlb (NLB security group) ← NEEDED

Problem: NLB health checks use demo-sg-nlb
         But instance SG only allows demo-sg-alb
         → Health checks BLOCKED
         → Instances marked UNHEALTHY
```

### Step 16: Add NLB to Instance Security Group

**Edit Inbound Rules**

```
On instance security group (launch-wizard-1):
1. Click "Edit inbound rules" button
2. Find HTTP rule from ALB
   ├─ Type: HTTP
   ├─ Port: 80
   └─ Source: demo-sg-alb ✓
```

**Add New Rule for NLB**

```
Click "Add rule" button

New Rule Configuration:
├─ Type: HTTP
├─ Protocol: TCP (auto-filled)
├─ Port Range: 80 (auto-filled)
├─ Source: (need to set)
│  └─ Click source dropdown
│     └─ Select: demo-sg-nlb (NLB security group)
├─ Description: "Allow from NLB" (optional)
└─ Result: Both ALB and NLB can access instances
```

**Updated Inbound Rules**

```
Inbound Rules on Instance SG (After Update):

Rule 1: SSH
├─ Type: SSH (22)
├─ Protocol: TCP
├─ Port: 22
└─ Source: 0.0.0.0/0

Rule 2: HTTP from ALB ✓
├─ Type: HTTP (80)
├─ Protocol: TCP
├─ Port: 80
└─ Source: demo-sg-alb

Rule 3: HTTP from NLB ✓ (NEW)
├─ Type: HTTP (80)
├─ Protocol: TCP
├─ Port: 80
└─ Source: demo-sg-nlb
```

**Save and Apply**

```
1. Click "Save rules" button
2. Inbound rules updated
3. Changes apply immediately
4. Health checks now allowed!
```

### Step 17: Wait for Targets to Become Healthy

**Check Target Health Status**

```
Navigate: Load Balancing → Target Groups → demo-tg-nlb

Timeline After Security Group Fix:
- t=0s: Security group rule added
- t=5s: Next health check sent (now allowed)
- t=5s: Response received: 200 OK ✓
- t=10s: Second health check sent
- t=10s: Response received: 200 OK ✓
- t=10s: After 2 successes, marked HEALTHY ✓

Result:
├─ i-1111111111: Status HEALTHY ✓
├─ i-2222222222: Status HEALTHY ✓
└─ NLB ready to forward traffic
```

**Verification**
```
Targets (Updated):
├─ i-1111111111 (10.0.1.10)
│  ├─ Status: healthy ✓
│  ├─ Description: "Ok"
│  └─ Port: 80
│
└─ i-2222222222 (10.0.1.20)
   ├─ Status: healthy ✓
   ├─ Description: "Ok"
   └─ Port: 80
```

---

## Part 7: Test Load Balancer

### Step 18: Access NLB via Static IP

**Get NLB Static IP**

```
From NLB details page:
├─ Network mapping section
├─ Find: "IPv4 Address" column
│  ├─ Availability Zone 1: 203.0.113.10
│  ├─ Availability Zone 2: 203.0.113.20
│  └─ Availability Zone 3: 203.0.113.30 ← Use this first
└─ Copy: 203.0.113.30
```

**Test via Browser**

```
Browser URL: http://203.0.113.30/

Expected Response:
├─ Status: 200 OK ✓
├─ Body: "Hello World from 10.0.1.10"
│        or
│        "Hello World from 10.0.1.20"
└─ Response depends on which instance served it
```

**Verify Load Balancing**

```
Refresh Browser Multiple Times:
Request 1: http://203.0.113.30/
└─ Response: "Hello World from 10.0.1.10" (instance 1)

Request 2: http://203.0.113.30/ (refresh)
└─ Response: "Hello World from 10.0.1.20" (instance 2)

Request 3: http://203.0.113.30/ (refresh)
└─ Response: "Hello World from 10.0.1.10" (instance 1)

Pattern:
├─ IP address in response changes
├─ Proves NLB is distributing traffic
├─ Load balancing working correctly ✓
└─ Each instance served alternate requests
```

**Test via Different AZ IP**

```
Browser URL: http://203.0.113.20/ (different static IP)

Expected Result:
├─ Also routes to instances 1 or 2
├─ Same load balancing behavior
├─ All 3 AZ IPs (203.0.113.10/20/30) work identically
└─ Confirms load balancing across regions
```

---

## Part 8: Cleanup

### Optional: Delete Resources

**To Prevent AWS Charges**

```
Delete in this order:
1. Delete Network Load Balancer (DemoNLB)
2. Delete Target Group (demo-tg-nlb)
3. Delete Security Groups (demo-sg-nlb)
```

### Step 19: Delete NLB

**Delete Load Balancer**

```
1. Navigate: EC2 → Load Balancing → Load Balancers
2. Find: DemoNLB
3. Click: Select checkbox
4. Click: "Actions" dropdown
5. Click: "Delete load balancer"
6. Confirm: Type "confirm" or click "Delete"
7. Status: Deleting... → Deleted

Timeline:
├─ t=0s: Delete initiated
├─ t=10s: Load balancer in "Deleting" state
├─ t=30s: Load balancer removed from list
└─ Targets no longer receive traffic
```

### Step 20: Delete Target Group (Optional)

**Delete Target Group**

```
1. Navigate: EC2 → Load Balancing → Target Groups
2. Find: demo-tg-nlb
3. Click: Select checkbox
4. Click: "Actions" dropdown
5. Click: "Delete target groups"
6. Confirm: Click "Delete target groups"
7. Result: Target group deleted

Note:
├─ NLB must be deleted first
├─ Target group otherwise shows "In use"
└─ Deletion frees up resources
```

### Step 21: Delete Security Groups (Optional)

**Delete NLB Security Group**

```
1. Navigate: EC2 → Security Groups
2. Find: demo-sg-nlb
3. Click: Select checkbox
4. Click: "Actions" dropdown
5. Click: "Delete security groups"
6. Confirm: Click "Delete"
7. Result: Security group deleted

Note:
├─ NLB must be deleted first
├─ Can't delete if in use
├─ Freeing SG frees reference
└─ Instance SG can stay (used by ALB)
```

---

## Part 9: Key Learnings from Hands-On

### Security Group Interaction

**Importance of Multi-Layer Security Groups**

```
Architecture:
Internet
   ↓
NLB (demo-sg-nlb):
├─ Allows HTTP from anywhere (0.0.0.0/0)
│
   ↓
EC2 Instances (launch-wizard-1):
├─ MUST allow from BOTH:
│  ├─ demo-sg-alb (for ALB health checks)
│  └─ demo-sg-nlb (for NLB health checks)
│
└─ If missing demo-sg-nlb rule:
   └─ NLB health checks BLOCKED
      → Targets marked UNHEALTHY
      → No traffic forwarded
      → Service appears DOWN

Lesson:
├─ Every load balancer needs SG rule
├─ Instance SG must reference load balancer SG
├─ Use security group names (not CIDRs)
└─ Dynamic and scalable approach
```

### Health Check Troubleshooting

**Common Mistakes and Fixes**

```
Issue 1: Targets Unhealthy After Creation
├─ Symptom: demo-tg-nlb shows "unhealthy"
├─ Cause 1: Security group not updated
├─ Cause 2: Application not running on instances
├─ Cause 3: Health check port mismatch
├─ Fix: Check instance SG, app status, port number

Issue 2: Health Checks Failing with Timeout
├─ Symptom: "Connection timeout" in health check
├─ Cause: Application not responding
├─ Fix: SSH to instance, verify HTTP server running
└─ Command: sudo systemctl status httpd (Linux)

Issue 3: Targets Healthy but Traffic Not Flowing
├─ Symptom: Targets show healthy, but no response
├─ Cause: NLB not receiving traffic (DNS/IP issue)
├─ Fix: Verify NLB IP address, DNS name
└─ Or: Verify client can reach NLB SG
```

### Static IP Benefits

**Why NLB Static IP is Important**

```
Use Case 1: Firewall Whitelisting
├─ Third-party service
├─ Only accepts specific IPs
├─ ALB: Dynamic IPs (doesn't work)
├─ NLB: Static IPs (perfect for this) ✓
└─ Enables integration with strict firewalls

Use Case 2: VPN/Direct Connect
├─ On-premises networks
├─ Need fixed public IP for routing
├─ ALB: IPs change (problematic)
├─ NLB: Stable IPs (ideal) ✓
└─ Hybrid cloud scenarios

Use Case 3: Compliance/Logging
├─ Regulatory requirement
├─ Must whitelist known IPs only
├─ ALB: Can't guarantee IPs
├─ NLB: Predictable IPs ✓
└─ Security and audit trails
```

---

## Part 10: Exam Focus Points

### NLB Hands-On Key Takeaways

**Configuration**
- NLB listens on Layer 4 (TCP/UDP), not HTTP
- One static IP per availability zone
- Can be AWS-assigned or Elastic IP
- Security group controls inbound traffic
- Similar to ALB but different protocols

**Target Groups**
- Register EC2 instances or IP addresses
- Health checks: TCP, HTTP, or HTTPS
- For instances with HTTP app: Use HTTP health check
- Health check interval: 30 seconds (default) or 5 seconds (aggressive)
- Targets must be in same VPC

**Security Groups**
- NLB SG: Controls traffic TO load balancer
- Instance SG: Controls traffic TO instances
- Instance SG MUST reference NLB SG
- Without rule: Health checks blocked → Targets unhealthy

**Testing & Verification**
- Use static IP to access NLB
- Refresh browser to see load balancing (IPs change)
- Check target health status in console
- Monitor health check failures for troubleshooting

**Cleanup**
- Delete NLB first (targets become unhealthy)
- Optionally delete target group and SG
- Prevents unwanted AWS charges