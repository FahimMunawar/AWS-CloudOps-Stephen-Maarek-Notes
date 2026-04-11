# Advanced ALB Concepts: Security and Routing Rules

This note covers advanced Application Load Balancer concepts including network security hardening with security groups and creating sophisticated listener rules for routing and error handling.

## Part 1: Network Security with Security Groups

### Current Architecture (Before Hardening)

**Current Setup**
```
Internet Users
    ↓
Public IP 1: Direct to Instance 1 (HTTP allowed from anywhere)
Public IP 2: Direct to Instance 2 (HTTP allowed from anywhere)
    ↓
ALB DNS: Through Load Balancer (preferred path)
    ↓
Instances
```

**Problem**
- EC2 instances accessible directly via public IPs
- Security group allows HTTP from anywhere (0.0.0.0/0)
- Load balancer is optional, not required
- Direct instance access bypasses load balancer benefits
- Instance exposure increased

### Hardened Architecture (After Hardening)

**Desired Setup**
```
Internet Users
    ↓
ALB DNS: ONLY through Load Balancer (forced)
    ↓
ALB (Public SG: Allow HTTP from 0.0.0.0/0)
    ↓
Instances (Private SG: Allow HTTP from ALB SG only)
    ↓
Backend Services
```

**Benefit**
- Instances only accessible through load balancer
- Direct instance IP access blocked
- Security enforced at network level
- Single entry point for traffic

### Implementation: Restrict Instance Security Group

#### Step 1: Identify Load Balancer Security Group

**Where to Find**
1. EC2 Console → Load Balancers
2. Select: `DemoALB`
3. Details tab: Security groups
4. **Copy**: `demo-sg-load-balancer` (or note SG ID)
5. Example: `sg-0a1b2c3d4e5f6g7h8`

#### Step 2: Modify Instance Security Group

**Navigate to Security Groups**
1. EC2 Console → Security Groups
2. Select: `Launch-wizard-1` (instance security group)
3. Click "Edit inbound rules"

#### Step 3: Remove Public HTTP Rule

**Current Rule**
```
Type: HTTP
Protocol: TCP
Port: 80
Source: 0.0.0.0/0 (anywhere)
```

**Action**
1. Find HTTP rule
2. Click "Delete" button
3. Remove rule from inbound

**Result**: Instances no longer accept public HTTP

#### Step 4: Add Restricted HTTP Rule

**Add New Inbound Rule**
1. Click "Add rule"
2. Configure:
   ```
   Type: HTTP
   Protocol: TCP
   Port: 80
   Source: (Don't use CIDR)
   ```

3. **Source Type**: Security Group
   - Search: "load" or type SG ID
   - Select: `demo-sg-load-balancer`

**Rule Configuration**
```
Type: HTTP
Protocol: TCP
Port: 80
Source: sg-0a1b2c3d4e5f6g7h8 (demo-sg-load-balancer)
Description: "Allow from ALB only"
```

4. Click "Save rules"

**Result**
```
Inbound Rules After:
├─ HTTP 80
│  └─ From: Security Group (demo-sg-load-balancer)
│     └─ Effect: Allow traffic ONLY if source is ALB SG
└─ SSH 22
   └─ From: 0.0.0.0/0 (optional, for troubleshooting)
```

### Security Group Linking: How It Works

**Dynamic Security**
```
Request Flow:

1. User → ALB (from internet)
   ├─ Source: User's public IP
   ├─ ALB Security Group permits: ✓
   └─ Allowed

2. ALB → EC2 Instance
   ├─ Source: ALB's elastic network interface (in instance's VPC)
   ├─ Instance checks: "Is source in demo-sg-load-balancer?"
   ├─ Result: YES (ALB has that SG) ✓
   └─ Allowed

3. Direct user → EC2 Instance
   ├─ Source: User's public IP
   ├─ Instance checks: "Is source in demo-sg-load-balancer?"
   ├─ Result: NO (user doesn't have that SG) ✗
   └─ DENIED (connection timeout)
```

**Why This Works**
- Security group reference is **dynamic**
- Not IP address-specific
- Works regardless of ALB IP changes
- Any resource with referenced SG automatically permitted

### Verification: Test Security

#### Test 1: Direct Instance Access

**Before Hardening**
```
Client → Instance Public IP:80
├─ Security group allows HTTP from 0.0.0.0/0
└─ Response: 200 OK ✓
```

**After Hardening**
```
Client → Instance Public IP:80
├─ Security group allows HTTP from ALB SG only
├─ Client traffic not from ALB SG
└─ Response: Connection timeout ✗
```

**Practical Test**
1. Browser: `http://ec2-instance-public-ip`
2. Expected: **Connection timeout** (no response)
3. OR: **Page unable to load**
4. Reason: Security group now restricts to ALB SG only

#### Test 2: Load Balancer Access

**Still Works**
```
Client → ALB DNS:80
├─ ALB has demo-sg-load-balancer SG
├─ ALB forwards to Instance
├─ Instance permits traffic from demo-sg-load-balancer ✓
└─ Response: 200 OK ✓
```

**Practical Test**
1. Browser: `http://[ALB-DNS-name]`
2. Expected: **Hello World** response ✓
3. Works normally
4. Load balancing operates transparently

### Benefits of This Security Model

**Network Isolation**
- Instances hidden from direct internet access
- Only load balancer as entry point
- Single security control point

**Operational Benefits**
- SG reference is algorithm-aware
- No IP address maintenance needed
- Automatic scaling compatible
- Works with dynamic instance IPs

**Best Practice**
- Always restrict backend instance SGs to load balancer SG
- Front-facing tier: Public access
- Backend tier: Restricted to load balancer only
- Defense in depth and principle of least privilege

---

## Part 2: Advanced ALB Listener Rules

### Understanding Listener Rules

**Listeners**
- Entry point for incoming traffic
- Listen on specific port (e.g., 80, 443)
- Evaluate rules to determine where traffic goes

**Listener Rules**
- Conditions to match against incoming requests
- Actions to take when condition matched
- Priority/ordering to resolve conflicts
- Currently have default rule (matches everything)

### Default Rule

**Default Rule Behavior**
```
Name: "Default rule"
Condition: (none - matches all)
Action: Forward to → demo-tg-alb
Priority: Last (lowest)
```

**Role**
- Catches all requests not matching other rules
- Must exist (cannot be deleted)
- Typically simplest action (forward to main target group)

---

## Part 3: Creating Custom Listener Rules

### Concepts: Conditions and Actions

#### Conditions: What to Match

**Path-Based Condition**
```
Condition: Path is /error
Matches: 
- /error
- /error/page
- /error/123
Does NOT match:
- /errors (different path)
- /not-error
```

**Host Header Condition**
```
Condition: Host header is api.example.com
Matches:
- Requests to api.example.com
Does NOT match:
- Requests to www.example.com
- Requests to admin.example.com
```

**HTTP Method Condition**
```
Condition: HTTP method is GET
Matches:
- GET requests
Does NOT match:
- POST requests
- PUT requests
```

**Source IP Condition**
```
Condition: Source IP is 192.168.1.0/24
Matches:
- Requests from 192.168.1.0 - 192.168.1.255
Does NOT match:
- Requests from other IPs
```

**Query String Condition**
```
Condition: Query string contains debug=true
Matches:
- /page?debug=true
- /page?name=john&debug=true
Does NOT match:
- /page (no query string)
- /page?debug=false
```

**HTTP Header Condition**
```
Condition: Header X-API-Key equals secret123
Matches:
- Any request with header: X-API-Key: secret123
Does NOT match:
- Requests without header
- Different header value
```

#### Actions: What to Do

**Forward to Target Group**
- Route to specific target group
- Example: `/api/*` → api-servers-tg
- Most common action
- Can weight traffic between multiple target groups

**Redirect**
```
Redirect to:
├─ Protocol: HTTP → HTTPS
├─ Port: 80 → 443
├─ Host: old-domain.com → new-domain.com
├─ Path: /old → /new
├─ Query string: (keep or modify)
└─ Status code: 301 (moved permanently) or 302 (temporary)

Common Use Cases:
- HTTP to HTTPS redirect
- Domain migration
- URL restructuring
```

**Fixed Response**
```
Response Type: Fixed Response
├─ Status code: 200, 404, 403, 500, etc.
├─ Content-type: text/plain, application/json, text/html
└─ Message: Custom message to return

Example:
- 404 "Page not found"
- 403 "Access denied"
- 503 "Service maintenance"
```

---

## Part 4: Practical Example - Custom Error Rule

### Scenario: Custom /error Handler

**Goal**: Return custom response when users access `/error` path

### Step 1: Create New Listener Rule

**Navigate to Rules**
1. Load Balancers → `DemoALB`
2. Listeners tab
3. Port 80 listener → Click to edit
4. Under "Listener rules": Click "Add rule"

### Step 2: Configure Rule Name and Conditions

**Rule Name**
- Input: `DemoRule`
- Descriptive, identifies purpose

**Add Condition**
1. Click "Add condition"
2. Select: **Path** (from dropdown)

**Path Configuration**
```
Path is: (matches exactly)
Value: /error
```

Alternative path options:
```
Path is: /error (exact match)
Path is: /error* (starts with)
Path matches: /error/* (matches /error with subpath)
Path matches: /error.* (regex pattern)
```

3. Click "Confirm"

**Condition Added**
```
Condition: Path is /error
```

### Step 3: Configure Action

**Add Action**
1. Click "Add action"
2. Select: **Fixed response** (from "Do the following" options)

**Fixed Response Configuration**
```
Response status code: 404
Response content-type: text/plain
Response body: "Custom error"
```

**Alternative Options**
```
Status codes available:
- 200: OK
- 301: Moved permanently
- 302: Found (temporary redirect)
- 304: Not modified
- 400: Bad request
- 403: Forbidden
- 404: Not found
- 405: Method not allowed
- 500: Internal server error
- 503: Service unavailable
```

3. Click "Confirm"

### Step 4: Set Priority

**Rule Priority**
```
Current rules:
├─ Default rule: (no priority, last)
└─ DemoRule: (needs priority)

Set priority: 5
(Lower number = higher priority)
(1 is highest, 50,000 is lowest)
```

**Priority Logic**
- Request comes in matching `/error`
- ALB evaluates rules by priority
- Rule 5 (DemoRule): Condition matches → Use this rule ✓
- Action: Return fixed response 404
- Default rule: Never reached for `/error` requests

### Step 5: Create Rule

1. Review configuration
2. Click "Create rule" or "Save"

**Rules Now Configured**
```
Listener Rules (Port 80):
├─ Rule 5 (Priority): DemoRule
│  ├─ Condition: Path is /error
│  └─ Action: Fixed response (404, "Custom error")
│
└─ Default rule (Last/Lowest priority)
   ├─ Condition: (none - matches all)
   └─ Action: Forward to demo-tg-alb
```

---

## Part 5: Test Custom Rule

### Test 1: Normal Request (Not /error)

**Request**
```
Browser: http://[ALB-DNS]/
```

**ALB Processing**
```
1. Request arrives: GET /
2. Evaluate rules in priority order:
   ├─ Rule 5 (DemoRule): Path is /error?
   │  └─ NO - Condition not matched
   │
   └─ Default rule: Match all?
      └─ YES - Condition matched
      
3. Action for default rule: Forward to demo-tg-alb
4. Response: Target group → EC2 instance
5. Browser receives: "Hello World from EC2 Instance"
```

**Expected Result**: ✓ Normal response from EC2

### Test 2: /error Request (Matching Custom Rule)

**Request**
```
Browser: http://[ALB-DNS]/error
```

**ALB Processing**
```
1. Request arrives: GET /error
2. Evaluate rules in priority order:
   ├─ Rule 5 (DemoRule): Path is /error?
   │  └─ YES - Condition matched!
   │  └─ Action: Fixed response
   │  └─ Return: 404 "Custom error"
   │  └─ STOP - Rule matched, don't evaluate further
   
(Default rule not evaluated because higher priority rule matched)

3. Response: Fixed response from ALB
4. Browser receives: Error 404 "Custom error"
```

**Expected Result**: ✓ 404 error with custom message
- No connection to target group
- Fixed response returned directly by ALB
- Message: "Custom error" plain text

### Test 3: /error/detail Request

**Path Matching Behavior**
- Condition: `Path is /error` (exact)
- Request: `/error/detail`
- Match result: Depends on condition type

**With "Path is /error" (Exact Match)**
```
/error/detail → Does NOT match (different path)
→ Falls through to default rule
→ Forwards to target group
→ Returns Hello World
```

**With "Path matches /error/*" (Prefix Match)**
```
/error/detail → Matches
→ Returns 404 Fixed response
```

---

## Part 6: Advanced Listener Rule Examples

### Example 1: Path-Based Microservices Routing

```
Listener Rules:

Rule 1 (Priority 1):
├─ Condition: Path is /api/*
└─ Action: Forward to → api-services-tg

Rule 2 (Priority 2):
├─ Condition: Path is /images/*
└─ Action: Forward to → image-service-tg

Rule 3 (Priority 3):
├─ Condition: Path is /admin/*
└─ Action: Forward to → admin-dashboard-tg

Default Rule (Last):
├─ Condition: (all others)
└─ Action: Forward to → website-tg

Traffic Routing:
- /api/users → api-services-tg
- /images/photos → image-service-tg
- /admin/panel → admin-dashboard-tg
- / → website-tg
- /about → website-tg
```

### Example 2: HTTP to HTTPS Redirect

```
Listener Rules on Port 80:

Rule 1 (Priority 1):
├─ Condition: (none - all requests)
└─ Action: Redirect
   ├─ Protocol: HTTPS
   ├─ Port: 443
   ├─ Status code: 301 (Permanent)
   └─ Effect: All HTTP traffic redirects to HTTPS

URL Examples:
- http://example.com/page → https://example.com/page
- http://example.com:80/api → https://example.com:443/api
```

### Example 3: Host-Based Routing

```
Listener Rules:

Rule 1 (Priority 1):
├─ Condition: Host header is api.example.com
└─ Action: Forward to → api-backend-tg

Rule 2 (Priority 2):
├─ Condition: Host header is admin.example.com
└─ Action: Forward to → admin-dashboard-tg

Default Rule (Last):
├─ Condition: Host header is www.example.com
└─ Action: Forward to → website-tg

Traffic Routing:
- api.example.com/* → API Backend
- admin.example.com/* → Admin Dashboard
- www.example.com/* → Website
```

### Example 4: Query String-Based Routing (A/B Testing)

```
Listener Rules:

Rule 1 (Priority 1):
├─ Condition: Query string includes version=v2
└─ Action: Forward to → new-version-tg

Rule 2 (Priority 2):
├─ Condition: Query string includes version=v1
└─ Action: Forward to → legacy-version-tg

Default Rule (Last):
├─ Condition: (no version parameter)
└─ Action: Forward to → stable-version-tg

Traffic Routing:
- /app?version=v2 → New Version (2% traffic for testing)
- /app?version=v1 → Legacy Version
- /app → Stable Version (98% traffic)

A/B Testing Setup:
- Direct 2% to new version for testing
- Keep stable version for majority
- Easy to adjust percentages
```

### Example 5: Source IP Restriction

```
Listener Rules:

Rule 1 (Priority 1):
├─ Condition: Source IP is 203.0.113.0/24
└─ Action: Forward to → internal-dashboard-tg

Rule 2 (Priority 2):
├─ Condition: Source IP is NOT 203.0.113.0/24
└─ Action: Fixed response (403 Forbidden)

Effect:
- Internal network (203.0.113.x): Can access dashboard
- External networks: Access denied
- Use for internal tools/admin panels
```

---

## Part 7: Rule Priority and Evaluation

### Priority System

**How Priority Works**
```
Rules evaluated TOP to BOTTOM by priority:
├─ Priority 1: Evaluated first
├─ Priority 2: Evaluated second
├─ Priority 3: Evaluated third
└─ Default rule: Evaluated last (catches all)

First rule to MATCH its condition:
└─ Action executed, STOP evaluation
```

**Priority Examples**
```
Listener Rules:

Rule Priority 1:
├─ Condition: Path matches /admin/*
└─ Action: Forward to admin-tg
   └─ If matched: Send to admin-tg, STOP

Rule Priority 5:
├─ Condition: Path matches /a*
└─ Action: Forward to generic-tg
   └─ If reached and matched: Send to generic-tg, STOP

Request: /admin/users
├─ Check Priority 1: Path /admin/* matches ✓
├─ Action: Forward to admin-tg
├─ STOP evaluation (never reaches Priority 5)
└─ Result: Goes to admin-tg (specific rule wins)

Request: /api/data
├─ Check Priority 1: Path /admin/* matches? NO
├─ Check Priority 5: Path /a* matches? ✓
├─ Action: Forward to generic-tg
└─ Result: Goes to generic-tg
```

**Best Practice: Specific to General**
- Specific rules: Lower priority number (evaluated first)
- General rules: Higher priority number (evaluated later)
- Default rule: No priority (always last)

---

## Part 8: Best Practices for Rules

### Configuration Best Practices

**1. Organize by Specificity**
- Specific conditions: Priority 1-10
- General conditions: Priority 10+
- Default: Last (catches all)

**2. Use Descriptive Names**
- `RedirectHTTP` (clear purpose)
- `AdminPanelRouting` (specific functionality)
- Avoid: `Rule1`, `Temp`, `Test`

**3. Document Conditions**
- Clear condition requirements
- Update documentation when rules change
- Helps debugging and maintenance

**4. Monitor Rule Matches**
- CloudWatch metrics on listener
- Check access logs
- Verify rules are being hit as expected

### Common Rule Mistakes

**Mistake 1: Overlapping Conditions**
```
Rule 1: Path is /api/*
Rule 2: Path is /api/users/*

Result: /api/users/123 always matches Rule 1 first
Consequence: Rule 2 never executed
Fix: Put more specific rule (/api/users) at priority 1
```

**Mistake 2: Incorrect Priority**
```
Generic rule at priority 1
Specific rules at priority 10+

Result: Generic rule always matches first
Consequence: Specific rules never used
Fix: Reverse priorities (specific = lower number)
```

**Mistake 3: Mismatched Conditions**
```
Condition: Path is /error
Actual code: Throws error at /errors (different path)

Result: Wrong rule triggered or no rule match
Consequence: Unexpected behavior
Fix: Test exact path matching before deploying
```

---

## Part 9: Security Implications

### Security Best Practices

**Fixed Response for Sensitive Paths**
```
Certain paths should not reach backend:
├─ /admin/* → 403 Forbidden (if not authenticated)
├─ /.env → 404 (hide sensitive files)
├─ /wp-admin/* → 404 (WordPress security)
└─ /.git → 404 (hide version control)

Fixed response returns error WITHOUT backend access
→ Prevents resource exposure
→ Hides backend existence
```

**Rate Limiting via Rules**
```
Rule: If source IP is known attacker
└─ Action: Fixed response 429 (Too many requests)

Prevents:
- DDoS attacks
- Brute force attempts
- Resource exhaustion
```

**Security Validation at Edge**
```
Rule: If query string contains malicious pattern
└─ Action: Fixed response 400 (Bad request)

Benefits:
- Blocks attacks before backend sees them
- Reduces backend processing overhead
- Centralizes security policy
```

---

## Part 10: Key Takeaways for SysOps Associate

### Network Security
- **Restrict instance SGs**: Only allow traffic from ALB SG
- **Security group linking**: Dynamic, IP-agnostic security
- **Principle of least privilege**: Minimal required access
- **Defense in depth**: Multiple layers of security

### Listener Rules Components
- **Conditions**: What to match (path, host, method, IP, query, header)
- **Actions**: What to do (forward, redirect, fixed response)
- **Priority**: Order of evaluation (lower number = higher priority)
- **Default rule**: Catches all unmatched requests

### Common Use Cases
- Path-based routing for microservices
- Host-based routing for multi-tenant
- HTTP to HTTPS redirect
- Fixed response for error handling
- Query string-based A/B testing
- Source IP restrictions for internal access

### Exam Focus Points
- Security group linking references SG by ID, not IP
- Rules evaluated top-to-bottom by priority
- First matching rule's action is executed
- Default rule always evaluates last
- Fixed response useful for blocking/errors
- Redirect useful for URL transformation
- Forward to target group enables microservices routing