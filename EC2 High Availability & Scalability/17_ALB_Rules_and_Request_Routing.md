# ALB Rules and Request Routing

Complete guide to Application Load Balancer rules, routing conditions, weighted target groups, and blue-green deployments for sophisticated traffic distribution and gradual application updates.

---

## Part 1: ALB Rules Overview

### Complete Rule System

**Rule Architecture and Processing**

```
ALB Rules Concept:

Definition:
├─ Rules: Set on Application Load Balancer
├─ Quantity: Multiple rules per ALB (or just one)
├─ Processing: Rules evaluated in order
├─ Default: Last rule is the "default rule"
└─ Processing order: Critical to understand

Rule Components:

Each Rule Contains:
1. Conditions:
   ├─ Specify when rule applies
   ├─ Multiple conditions possible
   ├─ All conditions must match (AND logic)
   └─ Example: Host is example.com AND path is /api/*

2. Actions:
   ├─ What happens when rule matches
   ├─ One or more actions per rule
   ├─ Examples: Forward, redirect, fixed response
   └─ Primary action: Usually forward to target group

3. Priority:
   ├─ Order of evaluation
   ├─ Lower number = higher priority
   ├─ Rule 1, Rule 2, Rule 3...Default
   └─ First match wins

Rule Types:

Standard Rules:
├─ Name: Custom rule (not default)
├─ Properties: Specific conditions
├─ Priority: 1, 2, 3, etc.
├─ Actions: Forward, redirect, fixed response
└─ Can match: Never all requests (has conditions)

Default Rule:
├─ Name: Default rule
├─ Priority: Last (infinity/highest number)
├─ Conditions: None defined
├─ Matches: ALL requests not matching other rules
├─ Purpose: Catch-all for unmatched requests
└─ Required: Every ALB must have one

Processing Model:

Request Arrives:
├─ Step 1: Check Rule 1 conditions
│  ├─ Match: Execute Rule 1 actions
│  └─ No match: Continue
├─ Step 2: Check Rule 2 conditions
│  ├─ Match: Execute Rule 2 actions
│  └─ No match: Continue
├─ Step 3: Check Rule 3 conditions
│  ├─ Match: Execute Rule 3 actions
│  └─ No match: Continue
├─ Step 4: If no rules matched
│  └─ Execute: Default rule actions

First Match Wins:
├─ Implication: Rule order matters
├─ Important: More specific rules should come first
├─ Example: /api/* should be before /api/users/*
└─ Strategy: Order from most to least specific

Example Processing:

Setup: 3 Rules + Default

Rule 1: Host = example.com, Path = /api/*
Rule 2: Host = api.example.com, Path = /*
Rule 3: Source IP = 10.0.1.0/24, Path = /internal/*
Default: All other traffic

Request 1: example.com + /api/users

Processing:
├─ Check Rule 1: Host = example.com? YES, Path = /api/*? YES
├─ MATCH! Execute Rule 1 actions
└─ Stop processing (first match)

Request 2: api.example.com + /data

Processing:
├─ Check Rule 1: Host = example.com? NO
├─ Check Rule 2: Host = api.example.com? YES, Path = /*? YES
├─ MATCH! Execute Rule 2 actions
└─ Stop processing

Request 3: example.com + /public

Processing:
├─ Check Rule 1: Host = example.com? YES, Path = /api/*? NO
├─ Check Rule 2: Host = api.example.com? NO
├─ Check Rule 3: Source IP = 10.0.1.0/24? NO
├─ No rules matched
├─ Execute Default rule actions
└─ Stop processing

Architecture:

Internet Clients
├─ HTTP/HTTPS Request
└─ ↓ ALB (Application Load Balancer)
   ├─ Rule Evaluation
   │  ├─ Rule 1: Check conditions → Execute actions
   │  ├─ Rule 2: Check conditions → Execute actions
   │  ├─ Rule 3: Check conditions → Execute actions
   │  └─ Default: Execute if no match
   │
   └─ Actions
      ├─ Forward to Target Group
      ├─ Redirect to URL
      └─ Return Fixed Response

Multiple Rules Benefits:

Path-Based Routing:
├─ /api/* → API target group
├─ /static/* → Static content target group
├─ /admin/* → Admin target group
└─ /* → Default target group

Host-Based Routing:
├─ api.example.com → API servers
├─ app.example.com → Web application
├─ admin.example.com → Admin panel
└─ example.com → Marketing site

Complex Routing:
├─ Host + Path: api.example.com + /v2/* → API v2 targets
├─ Source IP + Path: 10.0.0.0/8 + /internal/* → Internal targets
├─ Method + Path: POST to /api/upload/* → Upload targets
└─ Custom: Any combination

Single Rule Model:

Configuration:
├─ Default Rule: Single rule (default)
├─ Target Group: Entire ALB → same target group
├─ Routing: No conditions, all traffic same destination
└─ Limitation: Can't route based on path/host

When Appropriate:
├─ Simple applications: Single backend
├─ Monolithic architecture: One target group suffices
├─ Testing: Simple ALB setup
└─ Not recommended for: Production with multiple services

Summary:
├─ Rules: Core ALB feature
├─ Flexibility: Sophisticated routing possible
├─ Order: Matters (first match wins)
├─ Default: Always processed last
└─ Power: Multiple target groups per ALB
```

---

## Part 2: Rule Conditions

### Request Matching Criteria

**Routing Condition Types**

```
Condition Overview:

What Are Conditions:

Definition:
├─ Criteria: Against which request evaluated
├─ Match behavior: If condition met → rule can execute
├─ Multiple: Can combine many conditions
├─ Logic: AND (all must match)
├─ Not OR: Cannot use OR logic directly
└─ Purpose: Determine which requests match rule

Six Standard Conditions:

1. Host Header
2. HTTP Request Method
3. Path Pattern
4. Source IP
5. HTTP Header
6. Query String Parameter

Each Condition:
├─ Multiple values: Supported (OR within condition)
├─ Optional: At least one typically used
├─ Combined: Multiple conditions = AND logic
└─ Powerful: Create very specific routing

Condition 1: Host Header

What It Matches:
├─ Field: HTTP Host header in request
├─ Value: Domain name from client request
├─ Format: example.com OR subdomain.example.com
└─ Source: Comes from client HTTP request

Common Uses:

Single Domain:
├─ Host: example.com
├─ Effect: Only requests to example.com match
├─ Use: Route to specific backend

Subdomain Routing:
├─ Host: api.example.com
├─ Effect: Only api subdomain requests match
├─ Use: Route API to separate target group

Multiple Domains:
├─ Host: example.com OR www.example.com
├─ Effect: Both domains matched
├─ Use: Route to same target group

Wildcard Domain:
├─ Host: *.example.com
├─ Effect: Any subdomain of example.com
├─ Use: Route all subdomains to same backend

Example Rules:

Rule 1:
├─ Host: api.example.com
├─ Action: Forward to API target group

Rule 2:
├─ Host: *.example.com (except api)
├─ Action: Forward to default target group

Rule 3 (Default):
├─ Host: N/A
├─ Action: Forward to default target group

Client Request:
├─ Host: api.example.com → Rule 1
├─ Host: web.example.com → Rule 2
├─ Host: other.com → Rule 3 (default)

Condition 2: HTTP Request Method

What It Matches:
├─ Field: HTTP method in request
├─ Values: GET, POST, PUT, DELETE, HEAD, OPTIONS, PATCH
├─ Purpose: Route based on operation type
└─ Use case: Different backends for different operations

Common Uses:

Read vs Write:
├─ GET/HEAD: Read operations
├─ POST/PUT/DELETE: Write operations
└─ Strategy: Route to different backends

API Versioning:
├─ GET /api/users → v1
├─ POST /api/users → v2
└─ Effect: Different backends per method

File Operations:
├─ GET /files/* → Download backend
├─ POST /files/* → Upload backend
└─ Optimization: Separate backends by operation

Example Rules:

Rule 1:
├─ Method: GET OR HEAD
├─ Path: /api/*
├─ Action: Forward to read-optimized targets

Rule 2:
├─ Method: POST OR PUT
├─ Path: /api/*
├─ Action: Forward to write-optimized targets

HTTP Methods:

GET:
├─ Purpose: Retrieve data
├─ Safe: Yes (doesn't change state)
├─ Body: No
└─ Common: Most requests

POST:
├─ Purpose: Create/submit data
├─ Safe: No
├─ Body: Yes (data sent)
└─ Common: Form submissions, API creates

PUT:
├─ Purpose: Replace entire resource
├─ Safe: No
├─ Body: Yes (resource data)
└─ Use: Update operations

DELETE:
├─ Purpose: Delete resource
├─ Safe: No
├─ Body: Usually no
└─ Use: Delete operations

HEAD:
├─ Purpose: Like GET but no body returned
├─ Safe: Yes
├─ Body: No
└─ Use: Checking if resource exists

PATCH:
├─ Purpose: Partial resource update
├─ Safe: No
├─ Body: Yes (patch data)
└─ Use: Partial updates

Condition 3: Path Pattern

What It Matches:
├─ Field: Path portion of URL
├─ Format: /path or /path/* or /path/to/resource
├─ Wildcards: * supported for matching
├─ Case: Sensitive
└─ Purpose: Route to target based on URL path

Path Pattern Syntax:

Exact Match:
├─ Pattern: /api/users
├─ Matches: Exactly /api/users
├─ Doesn't match: /api/users/123 or /api/user

Prefix Match:
├─ Pattern: /api/*
├─ Matches: /api/anything and /api/users/123
├─ Doesn't match: /api or /application

Wildcard:
├─ * = Any characters (including /)
├─ Example: /images/*.jpg
├─ Matches: /images/photo.jpg or /images/subfolder/photo.jpg

Common Uses:

API Versioning:
├─ Rule 1: /api/v1/* → API v1 targets
├─ Rule 2: /api/v2/* → API v2 targets
├─ Rule 3: /api/* → Default API targets

Service Separation:
├─ Rule 1: /auth/* → Authentication service
├─ Rule 2: /payment/* → Payment service
├─ Rule 3: /users/* → User service

Static vs Dynamic:
├─ Rule 1: /static/* → Static content (S3, cache)
├─ Rule 2: /* → Dynamic application

Admin Panel:
├─ Rule: /admin/* → Admin target group

Example Routing:

Request /api/v1/users:
├─ Check /api/v1/* → MATCH (specific rule)
├─ Check /api/* → Would match but rule processed first
├─ Execute: First matching rule

Request /static/image.png:
├─ Check /api/v1/* → NO
├─ Check /api/v2/* → NO
├─ Check /static/* → MATCH
├─ Execute: Static content rule

Request /public:
├─ Check all specific rules → NO MATCH
├─ Execute: Default rule

Condition 4: Source IP

What It Matches:
├─ Field: Client IP address making request
├─ Value: Single IP or CIDR range
├─ Format: 203.0.113.0/24 or 203.0.113.100
├─ Purpose: Route based on where request originates
└─ Use: Restrict internal vs external

Practical Uses:

Internal Access:
├─ Source: 10.0.0.0/8 (private network)
├─ Route: /internal/* to internal targets
├─ Effect: Only company network access

Geo-Based Routing:
├─ Source: Specific country IP ranges
├─ Route: To regional targets
├─ Effect: Route to nearest region

VPN Access:
├─ Source: 192.168.1.0/24 (VPN subnet)
├─ Route: To VPN-only resources
├─ Effect: Secure access control

Partner Access:
├─ Source: Partner company IP ranges
├─ Route: To partner-specific API endpoints
├─ Effect: Controlled partner access

Example Rules:

Rule 1:
├─ Source IP: 10.0.0.0/8
├─ Path: /admin/*
├─ Action: Forward to admin targets

Rule 2:
├─ Source IP: 203.0.113.0/24
├─ Path: /api/*
├─ Action: Forward to partner API targets

Default:
├─ Action: Return 403 Forbidden for /admin/*

IP Notation:

Single IP:
├─ Format: 203.0.113.100
├─ Meaning: Exactly this IP
└─ Use: Specific server access

CIDR Range:
├─ Format: 203.0.113.0/24
├─ Meaning: 203.0.113.0 to 203.0.113.255 (256 IPs)
├─ /8 = large range (millions)
├─ /16 = medium range (thousands)
├─ /24 = smaller range (256)
├─ /32 = single IP
└─ Use: Network ranges

Common Ranges:

Private IP (RFC 1918):
├─ 10.0.0.0/8 (Class A private)
├─ 172.16.0.0/12 (Class B private)
├─ 192.168.0.0/16 (Class C private)
└─ Use: Internal networks

Public IP:
├─ Anything not private
├─ 8.8.8.8 example
└─ Use: Internet-facing

Condition 5: HTTP Header

What It Matches:
├─ Field: Any HTTP header in request
├─ Examples: User-Agent, Authorization, X-Custom-Header
├─ Values: String matching
├─ Case-insensitive: Header names
├─ Purpose: Route based on header information
└─ Use: Advanced routing (authorization, user agent)

Common Uses:

Authorization Token:
├─ Header: Authorization
├─ Value: Contains "Bearer"
├─ Route: To authenticated endpoint list
├─ Effect: Token check at ALB level

User Agent:
├─ Header: User-Agent
├─ Value: Contains "Mobile"
├─ Route: To mobile-optimized targets
├─ Effect: Route based on device type

Custom Headers:
├─ Header: X-API-Version
├─ Value: v2
├─ Route: To API v2 targets
├─ Effect: Client-specified routing

Content Type:
├─ Header: Content-Type
├─ Value: application/json
├─ Route: To JSON processing targets
├─ Effect: Different backends by format

Example Rules:

Rule 1:
├─ Header: User-Agent contains "Mobile"
├─ Action: Forward to mobile-optimized targets

Rule 2:
├─ Header: X-Debug-Mode = true
├─ Action: Forward to debug targets

Default:
├─ Action: Forward to production targets

Condition 6: Query String Parameter

What It Matches:
├─ Field: Query parameters in URL
├─ Format: ?param=value or ?param=value&other=value
├─ Location: After ? in URL
├─ Purpose: Route based on URL parameters
└─ Use: A/B testing, versioning, feature flags

Query String Syntax:

Single Parameter:
├─ URL: /page?version=2
├─ Parameter: version
├─ Value: 2
├─ Route if match: Forward to v2 targets

Multiple Parameters:
├─ URL: /search?q=books&category=fiction
├─ Parameters: q, category
├─ Values: books, fiction

Using Query Parameters:

A/B Testing:
├─ URL: /app?variant=new
├─ Rule: If variant=new → Route to new app targets
├─ Effect: Small % of users test new version

Feature Flags:
├─ URL: /app?feature=beta-ui
├─ Rule: If feature=beta-ui → Route to beta targets
├─ Effect: Test new features with specific users

API Versioning:
├─ URL: /api?version=2
├─ Rule: If version=2 → Route to API v2
├─ Effect: Client chooses API version via URL

Example Rules:

Rule 1:
├─ Query param: channel = beta
├─ Action: Forward to beta targets (5% traffic)

Rule 2:
├─ Query param: channel = stable
├─ Action: Forward to stable targets

Default:
├─ Action: Forward to stable targets (if no param)

Combining Conditions:

AND Logic (All Must Match):
├─ Example: Host=api.example.com AND Path=/v2/*
├─ Effect: Only requests matching BOTH conditions
├─ Matching: api.example.com/v2/users → YES
├─ Matching: api.example.com/v1/users → NO (path wrong)
├─ Matching: app.example.com/v2/users → NO (host wrong)

OR Within Condition:
├─ Example: Host = api.example.com OR www.example.com
├─ Within: Same condition type
├─ Effect: Either value matches
├─ Result: Both hosts route to same rule

Complex Example:

Rule Setup:

Rule 1: Host=api.example.com AND Path=/v2/* AND Method=POST
├─ Matches: POST to api.example.com/v2/users
└─ Action: Forward to API v2 write targets

Rule 2: Host=api.example.com AND Path=/v2/*
├─ Matches: Any method to api.example.com/v2/*
├─ Doesn't match: If Rule 1 matches first
└─ Action: Forward to API v2 read targets

Rule 3: Host=api.example.com
├─ Matches: Any request to api.example.com
├─ Only if Rules 1-2 don't match
└─ Action: Forward to API targets

Rule 4 (Default):
├─ Matches: Anything not matched above
└─ Action: Forward to web app targets

Request Processing:

Request: POST api.example.com/v2/users
├─ Check Rule 1: Host=api? YES, Path=/v2/*? YES, Method=POST? YES
├─ MATCH! Execute Rule 1 (API v2 write)

Request: GET api.example.com/v2/users
├─ Check Rule 1: Host=api? YES, Path=/v2/*? YES, Method=POST? NO
├─ Check Rule 2: Host=api? YES, Path=/v2/*? YES
├─ MATCH! Execute Rule 2 (API v2 read)

Request: GET api.example.com
├─ Check Rule 1: Host=api? YES, Path=/v2/*? NO
├─ Check Rule 2: Host=api? YES, Path=/v2/*? NO
├─ Check Rule 3: Host=api? YES
├─ MATCH! Execute Rule 3 (API)

Request: GET example.com/home
├─ Check all rules: NO MATCH
├─ Execute: Default rule (web app)
```

---

## Part 3: Rule Actions

### What Happens When Rule Matches

**Action Types and Configuration**

```
Actions Overview:

Definition:
├─ Actions: What happens when rule conditions match
├─ One or more: Can have multiple actions per rule
├─ Sequence: Executed in order
├─ Terminating: Some stop processing, others continue
└─ Types: Forward, redirect, fixed response

Three Main Action Types:

1. Forward
├─ Destination: Target group or weighted targets
├─ Effect: Send request to backend targets
├─ Terminating: Yes (stops further actions)
└─ Primary: Most common action

2. Redirect
├─ Destination: Different URL/protocol
├─ Effect: Client redirected to new URL
├─ Terminating: Yes
└─ Use: URL changes, protocol upgrades

3. Fixed Response
├─ Response: Static response body
├─ Effect: Client got response from ALB
├─ Terminating: Yes
└─ Use: Errors, maintenance, blocks

Action 1: Forward

What It Does:
├─ Request: Forwarded to target group (or groups)
├─ Backend: Processes request normally
├─ Response: Returned to client
├─ Effect: Normal request-response flow

Forwarding To:
├─ Single Target Group
│  └─ All requests matching rule → same target group
├─ Multiple Target Groups (with weights)
│  └─ Requests distributed by weight
└─ Determining Factor: Weight %

Single Target Group Example:

Rule: Host=api.example.com, Path=/users/*
├─ Action: Forward to "API-targets"
├─ Effect: All matching requests → API target group
└─ Result: Normal load balancing within target group

Multiple Target Groups (Weighted):

Rule: Path=/api/*
├─ Action: Forward with weights
│  ├─ "API-v1-targets": Weight 8 (80%)
│  ├─ "API-v2-targets": Weight 2 (20%)
│  └─ Total weight: 10
├─ Effect: 80% traffic to v1, 20% to v2
└─ Use: Blue-green deployments, canary deployments

Weight Calculation:

Formula:
├─ Percentage = (Target Weight / Total Weight) × 100
├─ Example: Weight 2 out of 10 = 2/10 = 20%

Example Setup:

Target Groups and Weights:
├─ Production: Weight 90
├─ Canary: Weight 10
├─ Total: 100
├─ Result: 90% production, 10% canary traffic

Distribution:
├─ 100 requests
├─ Production: 90 requests (90%)
├─ Canary: 10 requests (10%)

Distribution Method:
├─ Per-request: Each request independently evaluated
├─ Random: Based on weights
├─ Result: Smooth distribution

Forward Target Configuration:

Single Target Group:
```
Rule: Host=api.example.com
├─ Action: Forward
├─ Target: api-targets (single)
└─ Configuration: Simple

Weighted Targets:
```
Rule: Path=/api/*
├─ Action: Forward
├─ Targets:
│  ├─ api-v1: Weight 8
│  ├─ api-v2: Weight 2
│  └─ Total: 10 (auto-calculated)
└─ Weights: 80% v1, 20% v2
```

Action 2: Redirect

What It Does:
├─ Request: Not forwarded to target
├─ Response: HTTP redirect response
├─ Client: Follows redirect (new request)
├─ New URL: Client goes to redirected location
└─ Effect: URL change at client level

Redirect Capabilities:

Protocol Change:
├─ From: HTTP
├─ To: HTTPS
├─ Effect: Force HTTPS (common security practice)
└─ Status code: 301 (permanent)

Host Change:
├─ From: www.example.com
├─ To: example.com (or vice versa)
├─ Effect: Canonical URL enforcement
└─ Status code: 301

Path Change:
├─ From: /old-path
├─ To: /new-path
├─ Effect: URL restructuring
└─ Status code: 301 or 307

Query Parameter Change:
├─ From: /page?sortby=old
├─ To: /page?sortby=new
├─ Effect: Parameter standardization
└─ Flexibility: Can modify any part

Redirect Use Cases:

HTTP to HTTPS:
├─ Rule: Protocol = HTTP
├─ Action: Redirect to HTTPS
├─ Effect: All HTTP traffic forced to HTTPS
└─ Security: Industry standard

Domain Canonicalization:
├─ Rule: Host = www.example.com
├─ Action: Redirect to example.com
├─ Effect: Canonical URL (no www)
└─ SEO: Prevent duplicate content

Moved Endpoints:
├─ Rule: Path = /old-api/*
├─ Action: Redirect to /new-api/*
├─ Effect: Old API paths redirect to new
└─ Migration: Gradual user transition

Temporary to Permanent:
├─ Rule: Host = temporary.example.com
├─ Action: Redirect to permanent.example.com
├─ Effect: Temporary domain → permanent
└─ Status: 301 (permanent)

Example Rules:

Rule 1 (HTTP → HTTPS):
├─ Condition: Protocol = HTTP
├─ Action: Redirect to HTTPS://#{host}#{path}
└─ Result: Any HTTP request → HTTPS automatically

Rule 2 (www → no www):
├─ Condition: Host = www.example.com
├─ Action: Redirect to example.com
└─ Result: www.example.com → example.com

Rule 3 (Old path → new):
├─ Condition: Path = /api/v1/*
├─ Action: Redirect to /api/v2/#{path_post}
└─ Result: /api/v1/users → /api/v2/users

Status Codes:

301 Moved Permanently:
├─ Use: Permanent redirect
├─ Caching: Browser caches forever
├─ Effect: Old URL replaced

302 Found (Temporary):
├─ Use: Temporary redirect
├─ Caching: Not cached
├─ Effect: Try new, original still valid

307 Temporary Redirect:
├─ Use: Temporary, method preserved
├─ Note: POST remains POST (unlike 302)
└─ Effect: Try new, original still valid

Redirect Variables:

Substitution in Redirect URL:

#{protocol}
├─ Replaced: With request protocol
├─ Example: http or https
└─ Use: In redirect target URL

#{host}
├─ Replaced: With request host
├─ Example: example.com
└─ Use: Keep same host in redirect

#{port}
├─ Replaced: With request port
├─ Example: 8080
└─ Use: Preserve port in redirect

#{path}
├─ Replaced: With entire path
├─ Example: /api/users/123
└─ Use: Keep same path

#{path_post}
├─ Replaced: With path after matched part
├─ Example: If rule matches /api/v1/*, value = /users/123
└─ Use: Strip matched part

#{query}
├─ Replaced: With query string
├─ Example: ?id=123&sort=name
└─ Use: Keep query parameters

Action 3: Fixed Response

What It Does:
├─ Request: Not forwarded
├─ Response: Static response from ALB
├─ Content: Fixed body
├─ Status code: Specified by you
└─ Effect: Response generated by ALB itself

Fixed Response Use Cases:

Maintenance Mode:
├─ Rule: All requests
├─ Action: Fixed response (503 Service Unavailable)
├─ Body: "Service under maintenance"
└─ Effect: Temporary downtime

Access Denied:
├─ Rule: Path = /admin/*
├─ Action: Fixed response (403 Forbidden)
├─ Body: "Access Denied"
└─ Effect: Block access at ALB level

Health Check Response:
├─ Rule: Path = /health
├─ Action: Fixed response (200 OK)
├─ Body: "OK"
└─ Effect: Lightweight health check

Rate Limit Response:
├─ Rule: Too many requests from IP
├─ Action: Fixed response (429 Too Many Requests)
├─ Body: "Rate limited"
└─ Effect: Reject excessive traffic

API Error Response:
├─ Rule: Path = /api/* AND method = POST
├─ Action: Fixed response (400 Bad Request)
├─ Body: JSON error message
└─ Effect: Reject invalid requests at ALB

Example Rules:

Rule 1 (Maintenance):
├─ Rule: Any request
├─ Action: Fixed response (503)
├─ Content-Type: text/plain
├─ Body: "Site under maintenance"

Rule 2 (Admin Blocked):
├─ Rule: Path = /admin/*
├─ Action: Fixed response (403)
├─ Content-Type: text/html
├─ Body: "<h1>Access Denied</h1>"

Rule 3 (Health Check):
├─ Rule: Path = /health
├─ Action: Fixed response (200)
├─ Content-Type: text/plain
├─ Body: "OK"

Status Codes for Fixed Response:

200-299 Success:
├─ 200 OK: Normal success response
├─ 202 Accepted: Request accepted but not processed
└─ Use: Health checks, success messages

400-499 Client Error:
├─ 400 Bad Request: Invalid request
├─ 403 Forbidden: Access denied
├─ 404 Not Found: Resource missing
├─ 429 Too Many Requests: Rate limiting
└─ Use: Errors, rejections, denials

500-599 Server Error:
├─ 500 Internal Server Error: Server error
├─ 503 Service Unavailable: Maintenance/overload
└─ Use: Maintenance, temporary issues

Response Content:

Text Format:
├─ Content-Type: text/plain
├─ Example: "Service maintenance"
└─ Use: Simple messages

HTML Format:
├─ Content-Type: text/html
├─ Example: "<h1>Error</h1><p>Access denied</p>"
└─ Use: Formatted pages with styling

JSON Format:
├─ Content-Type: application/json
├─ Example: {"error": "Invalid API key"}
└─ Use: API responses

Character Limits:
├─ Maximum: 1024 characters typically
├─ Limitation: ALB constraint
└─ Strategy: Keep messages concise

Multiple Actions:

Can Have Multiple:
├─ Authenticate (custom rule)
├─ Validate (custom rule)
├─ Then: Forward (standard)
└─ Effect: Multi-step processing

Execution Order:
├─ Custom actions: First (if any)
├─ Standard actions: Then (forward/redirect/fixed)
├─ Terminating: First terminating action stops chain
└─ Result: Remaining actions skipped

Action Selection Guide:

Use Forward When:
├─ Normal request flow
├─ Route to backend targets
├─ Let targets process request
└─ Standard behavior

Use Redirect When:
├─ URL needs to change
├─ Protocol upgrade (HTTP → HTTPS)
├─ Canonical URL enforcement
├─ Permanent URL migrations
└─ Client driven to new URL

Use Fixed Response When:
├─ No backend processing needed
├─ Maintenance mode needed
├─ Access control at ALB level
├─ Health checks needed
├─ ALB responds directly
└─ Reduce backend load
```

---

## Part 4: Weighted Target Groups

### Blue-Green and Canary Deployments

**Gradual Traffic Shifting**

```
Weighted Target Groups Overview:

Purpose:
├─ Goal: Route requests to multiple target groups
├─ Within: Single rule
├─ Distribution: Controlled by weights
├─ Effect: Percentage-based traffic split
└─ Use case: Gradual application updates, testing

Blue-Green Deployment:

Concept:
├─ Blue: Current production (stable)
├─ Green: New version (being tested)
├─ Gradual: Shift traffic from blue to green
├─ Validation: Monitor green before full switch
└─ Rollback: Easy if green has issues

Setup:

Initial State:
├─ ALB: Single ALB used by all clients
├─ Rule: Forward to target groups
├─ Blue targets: Weight 100 (all traffic)
├─ Green targets: Weight 0 (no traffic)

Phase 1: Testing
├─ Action: Not yet deployed
└─ Status: Waiting for new version ready

Phase 2: Deployment
├─ Action: Deploy new version to green targets
├─ Server: Mark unhealthy via health checks
├─ Status: Waiting for green to become healthy

Phase 3: Initial Traffic Shift (5%)
├─ Blue: Weight 95
├─ Green: Weight 5
├─ Traffic: 95% old version, 5% new version
├─ Monitor: Green metrics closely
└─ Duration: 5-10 minutes

Phase 4: Increased Traffic (25%)
├─ Blue: Weight 75
├─ Green: Weight 25
├─ Traffic: 75% old, 25% new
├─ Monitor: Green still performing well
└─ Duration: 10-20 minutes

Phase 5: Majority Traffic (75%)
├─ Blue: Weight 25
├─ Green: Weight 75
├─ Traffic: 25% old, 75% new
├─ Monitor: Green is primary now
└─ Duration: 20-30 minutes

Phase 6: Full Traffic (100%)
├─ Blue: Weight 0
├─ Green: Weight 100
├─ Traffic: All new version
├─ Monitor: Stable, no issues
└─ Duration: Permanent (or until next version)

End State:
├─ Green: Now becomes the new "blue" (current)
├─ Blue: Old version (holds in case rollback needed)
├─ Duration: Minutes to hours
└─ Rollback: Available if needed

Rollback Scenario:

Issue Detected:
├─ Green performance: Degraded
├─ Errors: Spiking on green
├─ Alert: Team notified
└─ Action: Begin rollback

Rollback Steps:
├─ Step 1: Blue 50%, Green 50% (quick)
├─ Step 2: Blue 100%, Green 0% (back to original)
├─ Time: <1 minute typically
├─ Impact: Previous version restored
└─ Clients: Unaffected (transparent)

Post-Rollback:
├─ Investigation: Why did green fail?
├─ Fix: Resolve issue
├─ Testing: More thorough testing
├─ Redeployment: Try again

Weight Configuration:

How Weights Work:

Setup:
├─ Blue target group: Weight 8
├─ Green target group: Weight 2
├─ Total: 10 (often normalized)
├─ Percentage: 8/10 = 80%, 2/10 = 20%

Per-Request Distribution:
├─ Request 1: Random → Weight 8? → Blue (80% chance)
├─ Request 2: Random → Weight 8? → Blue
├─ Request 3: Random → Weight 2? → Green
├─ ...
├─ Over 100 requests:
│  ├─ Blue: ~80 requests
│  ├─ Green: ~20 requests
│  └─ Ratio: 80:20 (matches weights)

Smooth Distribution:
├─ Not serial: (Blue, Blue, Blue, Blue, Blue, Blue, Blue, Blue, Green, Green)
├─ Random: (Blue, Green, Blue, Blue, Green, Blue, Blue, Blue, Blue, Green)
├─ Result: Gradual, statistically correct ratio

Weight Values:

Range:
├─ Minimum: 0 (no traffic)
├─ Maximum: 999 (not really, but high numbers acceptable)
├─ Typical: 0-100 for simplicity

Examples:

Weights 100, 0:
├─ Distribution: 100% to first, 0% to second
├─ Use: Before canary starts
└─ Effect: All traffic to stable version

Weights 95, 5:
├─ Distribution: 95% to first, 5% to second
├─ Use: Initial canary
└─ Effect: Small test with new version

Weights 50, 50:
├─ Distribution: 50% to each
├─ Use: Major version testing
└─ Effect: Half on each version

Weights 10, 90:
├─ Distribution: 10% to first, 90% to second
├─ Use: Final transition
└─ Effect: Almost all on new version

Weights 0, 100:
├─ Distribution: 0% to first, 100% to second
├─ Use: Final state
└─ Effect: Full cutover, ready for cleanup

Canary Deployment:

Concept:
├─ Named after: Canary in coal mine (early warning)
├─ Goal: Small percentage tests new code
├─ Safety: Major changes risky, test small first
├─ Duration: Hours or days
└─ Scales: From 1% to 100% slowly

Comparison to Blue-Green:

Blue-Green Deployment:
├─ Strategy: 100% to new after validation
├─ Time: Faster (minutes to hours)
├─ Risk: Tested in staging first, then deployed
├─ Typical: When confident in new version

Canary Deployment:
├─ Strategy: Gradual traffic increase
├─ Time: Slower (hours to days)
├─ Risk: Monitor in production, catch issues live
├─ Typical: When very risk-averse or large changes

Both Use:
├─ Weighted routing: Core mechanism
├─ Gradual: Shift traffic over time
├─ Monitor: Watch metrics
└─ Rollback: Available every step

Monitoring During Weighted Routing:

Key Metrics to Watch:

Error Rate:
├─ Metric: HTTPCode_Target_5XX on green
├─ Baseline: Should match blue
├─ Alert: If significantly higher
└─ Action: Investigate or rollback

Latency:
├─ Metric: TargetResponseTime
├─ Green target: p99 latency
├─ Comparison: vs Blue p99 latency
├─ Alert: If green significantly slower
└─ Action: Investigate or rollback

Throughput:
├─ Metric: RequestCountPerTarget
├─ Green targets: Receiving traffic as expected
├─ Check: Not overwhelmed or underutilized
└─ Alert: If distribution wrong

Custom Metrics:
├─ Application-specific: Metrics from your app
├─ Examples: Cache hit rate, DB query time, API response
├─ Monitor: Any custom metrics from green
└─ Alert: If degraded compared to blue

Logs:
├─ Check: Application logs from green
├─ Look for: Errors, exceptions
├─ Count: How many errors?
├─ Compare: To blue logs
└─ Alert: If error rate higher

AWS Console Monitoring:

Dashboard:
├─ Create: Custom dashboard
├─ Add: Blue metrics (HealthyHostCount, latency)
├─ Add: Green metrics (same metrics)
├─ Side-by-side: Compare behavior
└─ Live: Watch during deployment

Alarms:

Alarm 1 (Green Error Spike):
├─ Metric: HTTPCode_Target_5XX (green)
├─ Threshold: Alert if > baseline × 2
├─ Response: Page on-call, may need rollback

Alarm 2 (Green Latency High):
├─ Metric: TargetResponseTime (green p99)
├─ Threshold: Alert if > blue p99 × 1.5
├─ Response: Investigate performance

Alarm 3 (Green Unhealthy):
├─ Metric: HealthyHostCount (green)
├─ Threshold: Alert if drops below X
├─ Response: Check application health

Practical Example:

Scenario: E-commerce site deploying new search feature

Setup:
├─ Blue targets: 5 instances, current stable version
├─ Green targets: 5 instances, new version with better search
├─ Rule: Forward with weights

Phase 1: Blue 100%, Green 0%
├─ Time: New instances deployed, warming up
├─ Status: Green becoming healthy
├─ Monitoring: Green startup times
└─ Duration: 5 minutes

Phase 2: Blue 95%, Green 5%
├─ Time: 10 AM
├─ Traffic: 200 req/sec × 5% = 10 requests to green
├─ Monitor: 
│  ├─ Error rate on green: 0%? (baseline 0%)
│  ├─ Latency p99 on green: 200ms? (baseline 200ms)
│  ├─ Search quality: Any bad results?
│  └─ Logs: Any exceptions visible?
├─ Duration: 10 minutes
└─ Decision: Continue or rollback?

Phase 3: Blue 75%, Green 25%
├─ Time: 10:10 AM
├─ Traffic: 200 req/sec × 25% = 50 requests to green
├─ Monitor:
│  ├─ Error rate: Still 0% on green? ✓
│  ├─ Latency p99: Around 200ms? ✓ (very good)
│  ├─ Search quality: Results accurate? ✓ (anecdotal)
│  └─ Database: Query patterns normal? ✓
├─ Duration: 15 minutes
└─ Decision: Continue to higher

Phase 4: Blue 50%, Green 50%
├─ Time: 10:25 AM
├─ Traffic: 200 req/sec × 50% = 100 to each
├─ Monitor:
│  ├─ Error rate: Still good on green? ✓
│  ├─ Latency: Green latency trending? Stable ✓
│  ├─ Search accuracy: No complaints? ✓
│  └─ Backend load: Distributed evenly? ✓
├─ Duration: 20 minutes
└─ Decision: Continue to majority

Phase 5: Blue 25%, Green 75%
├─ Time: 10:45 AM
├─ Traffic: 200 req/sec × 75% = 150 to green
├─ Monitor:
│  ├─ Error rate: Staying low? ✓
│  ├─ Latency: Good? ✓
│  ├─ Feature adoption: Positive? ✓
│  └─ Database: Handling load? ✓
├─ Duration: 30 minutes
└─ Decision: Continue to full

Phase 6: Blue 0%, Green 100%
├─ Time: 11:15 AM
├─ Traffic: 200 req/sec, all to green
├─ Verify:
│  ├─ All traffic green: ✓
│  ├─ Performance: Stable? ✓
│  ├─ Errors: Still low? ✓
│  ├─ Feature: Working as expected? ✓
│  └─ No incidents: Deployment successful ✓
├─ Post-deployment:
│  ├─ Remove: Old blue instances (or keep for emergency)
│  ├─ Clean up: ALB rule (remove weights if only 100%)
│  ├─ Documentation: Update runbooks
│  └─ Learning: Post-mortem if any issues
└─ Total time: ~1 hour 15 minutes

Rollback Example:

Scenario: Issue detected in phase 4

Time: 10:40 AM, Phase 4 (50/50)
├─ Issue: Cache hit rate dropped 50% → 20% (green) 
├─ Impact: Database suddenly overloaded
├─ Alert: Database latency spiked through roof
├─ Investigation: Quickly confirm green is culprit

Rollback Actions:

Step 1: Quick reduction (Blue 75%, Green 25%)
├─ Time: 10:41 AM (1 minute)
├─ Effect: Reduce green traffic immediately
├─ Observation: 
│  ├─ Database latency: Still high
│  └─ Analysis: Need more data

Step 2: Heavy reduction (Blue 95%, Green 5%)
├─ Time: 10:42 AM (2 minutes)
├─ Effect: Most traffic back to blue
├─ Observation:
│  ├─ Database latency: Dropping
│  ├─ Trend: Looking better
│  └─ Analysis: Confirms green issue

Step 3: Full rollback (Blue 100%, Green 0%)
├─ Time: 10:43 AM (3 minutes)
├─ Effect: Completely back to previous
├─ Status: Site stable, database recovered
├─ Customer impact: Minimal (few minutes of issues)

Investigation:
├─ Root cause: New query pattern in search was inefficient
├─ Cache: Green's implementation not caching properly
├─ Database: N+1 queries problem introduced
├─ Fix: Optimize cache implementation, add query batching
├─ Next attempt: After fixes verified in staging

Time to Rollback: ~3 minutes (automated possible for faster)
Original deployment time lost: ~40 minutes (phases 1-3 ok)
New version: Can be deployed again after fixes

Implementation with AWS:

AWS Console:

1. Set up two target groups:
   ├─ Target group: api-prod-v1 (blue)
   └─ Target group: api-prod-v2 (green)

2. Create rule with weights:
   ├─ Rule: Path=/api/*
   ├─ Action: Forward
   ├─ Target 1: api-prod-v1, Weight=100
   ├─ Target 2: api-prod-v2, Weight=0
   └─ Status: Ready to deploy green

3. Deploy to green:
   ├─ Register instances: In api-prod-v2 target group
   ├─ Wait: Health checks pass
   ├─ Status: All instances healthy
   └─ Ready: For traffic

4. Shift traffic (edit rule):
   ├─ Change weights: 95, 5
   ├─ Save: Changes applied
   ├─ Effect: Immediate, smooth transition
   └─ Monitor: CloudWatch dashboard

5. Continue shifting:
   ├─ Edit rule repeatedly
   ├─ Percentages: Increase gradually
   ├─ Monitor: Each step
   └─ Decide: Continue or rollback

AWS CLI:

Modify target group weights:
```bash
aws elbv2 modify-rule \
  --rule-arn <rule-arn> \
  --actions Type=forward,ForwardConfig="{
    TargetGroups=[
      {TargetGroupArn=<v1-arn>,Weight=8},
      {TargetGroupArn=<v2-arn>,Weight=2}
    ]
  }"
```

Script for Gradual Shift:

```bash
#!/bin/bash

# Blue-green deployment script
# Gradually increase green traffic

RULE_ARN="arn:aws:elasticloadbalancing:..."
BLUE_ARN="arn:aws:elasticloadbalancing:...api-v1"
GREEN_ARN="arn:aws:elasticloadbalancing:...api-v2"

percentages=(5 10 25 50 75 90 100)

for percent in "${percentages[@]}"; do
  blue_weight=$((100 - percent))
  green_weight=$percent
  
  aws elbv2 modify-rule \
    --rule-arn $RULE_ARN \
    --actions Type=forward,ForwardConfig="{
      TargetGroups=[
        {TargetGroupArn=$BLUE_ARN,Weight=$blue_weight},
        {TargetGroupArn=$GREEN_ARN,Weight=$green_weight}
      ]
    }"
  
  echo "Updated weights: Blue=$blue_weight, Green=$green_weight"
  
  # Wait and monitor
  sleep 300  # 5 minutes
  
  # Check metrics (example)
  error_rate=$(aws cloudwatch get-metric-statistics \
    --metric-name HTTPCode_Target_5XX \
    --statistics Average \
    --period 60 \
    --start-time ... \
    --end-time ...)
  
  echo "Error rate: $error_rate"
  
  # If error rate too high, rollback
  if (( $(echo "$error_rate > 1.0" | bc -l) )); then
    echo "Error rate too high! Rolling back..."
    # Revert to previous weight
    break
  fi
done

echo "Deployment complete!"
```

Benefits of Weighted Routing:

Reduced Risk:
├─ Small testing: Catch issues early
├─ Large changes: Test before full deployment
└─ Confidence: Build trust in new version

Gradual Rollout:
├─ Smooth: No sudden 100% traffic shift
├─ Monitoring: Continuous observation
├─ Flexibility: Pause or rollback at any time
└─ Safety: Rollback available at each step

User Experience:
├─ Transparent: Most users don't notice
├─ Stability: If issues, few users affected
├─ Continuity: Seamless for most users
└─ Performance: Both versions actively considered

Operational Excellence:
├─ Automation: Can automate progression
├─ Observability: Continuous monitoring
├─ Confidence: High trust in deployments
└─ Efficiency: Quick rollback if needed

Common Mistakes:

Mistake 1: Jump to 100%
├─ Problem: No intermediate validation
├─ Risk: New version has issues → all users affected
├─ Solution: Always stage with small percentage first

Mistake 2: Too Fast Progression
├─ Problem: Don't have time to monitor/analyze
├─ Risk: Issues not detected before larger rollout
├─ Solution: Wait 5-10 minutes between steps

Mistake 3: Wrong Metrics
├─ Problem: Not watching right things
├─ Risk: In-depth metrics not analyzed
├─ Solution: Define metrics before deployment

Mistake 4: No Rollback Plan
├─ Problem: If issues, don't know what to do
├─ Risk: Stuck with bad version
├─ Solution: Always have rollback procedure ready

Best Practices:

1. Plan before deployment:
   ├─ Stages: Define progression (5%, 25%, 50%, 100%)
   ├─ Metrics: Know what to monitor
   ├─ Thresholds: Define alert levels
   └─ Rollback: Document how to quickly revert

2. Monitor actively:
   ├─ Dashboard: Live monitoring during deployment
   ├─ Metrics: Watch every stage
   ├─ Logs: Check for errors
   └─ Team: Dedicated person watching

3. Automate if possible:
   ├─ Script: Automate percentage changes
   ├─ Monitoring: Automated checks
   ├─ Alerts: Triggered automatically
   └─ Rollback: Automated if thresholds exceeded

4. Communicate:
   ├─ Team: Notify about deployment
   ├─ Status: Update as phases complete
   ├─ Issues: Alert if problems detected
   └─ Completion: Announce when done

5. Post-deployment:
   ├─ Monitoring: Continue for 24 hours
   ├─ Logs: Final analysis of deployment
   ├─ Learning: Document what worked/what didn't
   └─ Cleanup: Remove old version when confident
```

---

## Part 5: AWS Console Configuration

### Hands-On Rule Setup

**Creating and Managing Rules**

```
Accessing ALB Rules:

Step 1: Open AWS Console
├─ Service: EC2
├─ Region: Select your region
└─ Locate: Your ALB

Step 2: Find Listeners
├─ ALB Details: Open load balancer
├─ Section: Listeners and rules
├─ Find: Your listener (HTTP or HTTPS)
└─ Click: Rules link for that listener

Step 3: View Existing Rules
├─ List: All rules in order
├─ Default: Always last
├─ Priorities: 1, 2, 3... then Default
└─ Edit: Click rule to modify

Creating New Rule:

Step 1: Add Rule
├─ Button: "Add rule" (usually at top)
├─ Location: In rules list
└─ Page: Rule creation form opened

Step 2: Set Conditions

Add Conditions:
├─ Click: Add condition
├─ Select: Condition type
│  ├─ Host header
│  ├─ HTTP request method
│  ├─ Path pattern
│  ├─ Source IP
│  ├─ HTTP header
│  ├─ Query string parameter
│  └─ Select appropriate
│
├─ Enter: Condition value
│  ├─ Example (host): api.example.com
│  ├─ Example (path): /api/v2/*
│  ├─ Example (method): POST
│  └─ Be specific
│
├─ Add more: If needed (AND logic)
│  ├─ Multiple conditions: All must match
│  └─ Result: Very specific rule

Condition Examples in Console:

Host Header:
├─ Condition type: Host header
├─ Value: api.example.com (exact)
├─ Value: *.example.com (wildcard)
└─ Multiple values: Separate with commas

Path Pattern:
├─ Condition type: Path pattern
├─ Value: /api/* (matches /api/anything)
├─ Value: /api/v2/users (exact match)
└─ Careful: Order matters, most specific first

HTTP Method:
├─ Condition type: HTTP request method
├─ Values available: GET, HEAD, POST, PUT, DELETE, PATCH, OPTIONS
├─ Select: One or more (OR within condition)
└─ Example: POST OR PUT for write operations

Step 3: Set Actions

Add Actions:
├─ Click: Add action (usually after conditions)
├─ Select: Action type
│  ├─ Forward to target group
│  ├─ Redirect to URL
│  └─ Return fixed response
│
└─ Configure: Based on type selected

Forward Action:

Option 1: Single Target Group
├─ Select: Target group from dropdown
├─ Action: Forward all matching → that target group
└─ Simplest: Single destination

Option 2: Weighted Target Groups
├─ Add target group 1:
│  ├─ Select: Target group
│  └─ Weight: Enter number (e.g., 8)
├─ Add target group 2:
│  ├─ Select: Target group
│  └─ Weight: Enter number (e.g., 2)
├─ Result: Weights shown as percentage (80%, 20%)
└─ Use: Blue-green deployments

Redirect Action:

Configure Redirect:
├─ Protocol: HTTP, HTTPS, #{protocol}
├─ Host: example.com, www.example.com, #{host}
├─ Port: 80, 443, #{port}
├─ Path: /api/*, /api/v2/*, #{path}
├─ Query: #{query}
├─ Status code: 301, 302, 307

Common Redirects:

HTTP → HTTPS:
├─ Protocol: HTTPS
├─ Host: #{host} (keep same)
├─ Port: 443 (HTTPS default)
├─ Path: #{path} (keep same)
└─ Status: 301 (permanent)

www → no www:
├─ Protocol: #{protocol} (keep same)
├─ Host: example.com (remove www)
├─ Port: #{port} (keep same)
├─ Path: #{path} (keep same)
└─ Status: 301 (permanent)

/old-api/* → /new-api/*:
├─ Protocol: #{protocol}
├─ Host: #{host}
├─ Port: #{port}
├─ Path: /new-api#{path_post}
└─ Status: 301 or 307

Fixed Response Action:

Configure Response:
├─ Status code: 200, 404, 403, 503, etc.
├─ Content type: text/plain, text/html, application/json
├─ Response body: Type your message (up to 1024 chars)

Example Messages:

Maintenance:
├─ Status: 503
├─ Content type: text/html
├─ Body: <h1>Under Maintenance</h1><p>We'll be back soon</p>

Access Denied:
├─ Status: 403
├─ Content type: text/html
├─ Body: <h1>Access Denied</h1>

Health Check:
├─ Status: 200
├─ Content type: text/plain
├─ Body: OK

Step 4: Set Priority

Priority Assignment:
├─ Automatic: Usually auto-assigned (1, 2, 3, etc.)
├─ Custom: Can manually set
├─ Default: Automatically last
├─ Important: Lower number = higher priority

Rules Order:

Existing Rules:
├─ Rule 1: Priority 1
├─ Rule 2: Priority 2
├─ New Rule: Will be Priority 3
├─ Default: Always last

Changing Priority:
├─ Edit: Existing rule
├─ Priority field: Change number
├─ Effect: Reorder (most specific first)
└─ Save: Changes applied

Best Practice:
├─ More specific: Lower priority number
├─ Less specific: Higher priority number
├─ Default: Always highest number (last)

Step 5: Save & Create

Save Rule:
├─ Button: "Create" or "Save"
├─ Status: Rule added to listener
├─ Verification: Appears in rules list
└─ Effect: Active immediately

Editing Existing Rule:

Step 1: Open Rule
├─ Rules list: Find rule to edit
├─ Click: Edit (pencil icon usually)
└─ Page: Rule editing form

Step 2: Modify
├─ Conditions: Can add/remove/change
├─ Actions: Can modify target groups, weights
├─ Priority: Can reorder
└─ Changes: Any of above

Step 3: Save
├─ Button: "Save" or "Update"
├─ Status: Rule updated
├─ Effect: Active immediately
└─ Behavior: New for matching requests

Deleting Rule:

Step 1: Find Rule
├─ Rules list: Locate rule to delete
├─ Note: Cannot delete default rule
└─ Action: Only custom rules

Step 2: Delete
├─ Click: Delete (trash icon usually)
├─ Confirmation: Confirm deletion
└─ Effect: Rule removed

Step 3: Verify
├─ List: Rule no longer present
├─ Traffic: No longer uses this rule
└─ Default: Catches unmatched requests

Managing Priorities:

View Current Order:
├─ Rules tab: Lists all in order
├─ Priority: Shows number
├─ Default: Shows as "Default"

Reorder Rules:

Option 1: Edit Priority Field
├─ Open rule: Click edit
├─ Priority: Change number
├─ Save: Applied
└─ Effect: Reordered in list

Option 2: Drag-Drop (if available)
├─ Click-hold: Rule
├─ Drag: To new position
├─ Release: Drop in new position
└─ Effect: Priority auto-adjusted

Important: Order Matters
├─ First matching: Rule wins
├─ Example: If /api/* matches but /api/v1/* also matches
├─ Result: Whichever listed first wins
└─ Strategy: Most specific first

Rules List Example:

Priority 1: Host=api.example.com, Path=/v2/*
Priority 2: Host=api.example.com, Path=/v1/*
Priority 3: Host=api.example.com
Priority 4 (Default): All others

Request routing:
├─ api.example.com/v2/users → Rule 1
├─ api.example.com/v1/users → Rule 2
├─ api.example.com/other → Rule 3
├─ other.example.com → Rule 4

Weighted Targets Management:

Viewing Current Weights:

From Rules List:
├─ Action column: Shows "Forward to..."
├─ Click: See target group details
├─ If weighted: Shows percentage split
└─ Example: "80% api-prod, 20% api-canary"

Edit Weights:

Option 1: From Rules List
├─ Click: Edit rules action/icon
├─ Targets: Edit weights field
├─ Weight 1: New value
├─ Weight 2: New value
├─ Save: Changes applied

Option 2: From Rule Edit
├─ Open: Rule for editing
├─ Actions: Find weighted targets action
├─ Modify: Change percentages
├─ Save: Changes applied

Percentage Verification:

Display:
├─ Weights shown: As percentages
├─ Example: 80%, 20%
├─ Total: Always 100%
├─ Verification: Math automatically correct

Manual Calculation:
├─ Weight 1: 8
├─ Weight 2: 2
├─ Total: 10
├─ Percentage 1: 8/10 = 80%
├─ Percentage 2: 2/10 = 20%

Blue-Green in Console:

Initial Setup:

1. Create target groups:
   ├─ Blue TG: api-prod-v1
   ├─ Green TG: api-prod-v2
   └─ Ready: Both configured

2. Create rule:
   ├─ Condition: Path=/api/*
   ├─ Action: Forward with weights
   │  ├─ api-prod-v1 (blue): Weight 100
   │  └─ api-prod-v2 (green): Weight 0
   └─ Status: Rule ready

3. Deploy to green:
   ├─ Register instances: In api-prod-v2
   ├─ Wait: Health checks pass
   └─ Status: Green targets healthy

4. Shift traffic (edit rule):
   ├─ Open: Rule for edit
   ├─ Change weights:
   │  ├─ Blue: 95
   │  └─ Green: 5
   ├─ Save: Changes applied immediately
   └─ Effect: 5% traffic to green

5. Continue shifting:
   ├─ Monitor: Green metrics
   ├─ Repeat: Every 5-10 minutes
   ├─ Weights: 95→5, 75→25, 50→50, 25→75, 0→100
   └─ Vigilance: Watch alerts

Rollback in Console:

Quick Rollback:

1. Open rule: That has weighted targets
2. Edit: Weights back to previous
   ├─ Blue: 95 (or 100 for full rollback)
   ├─ Green: 5 (or 0 for full rollback)
3. Save: Immediate effect
4. Result: Traffic reduced/removed from green

Full Rollback:
├─ Blue: Set to 100
├─ Green: Set to 0
├─ Save: Complete rollback
└─ Effect: No traffic to green (as before)

Testing/Validation:

Before Production:

1. Create test rule:
   ├─ Condition: Host=test.example.com
   ├─ Action: Forward to blue TG
   └─ Purpose: Testing path

2. Test condition:
   ├─ Curl: curl http://test.example.com/api/test
   ├─ Verify: Request reaches backend
   └─ Confirm: Rule working

3. Add weight:
   ├─ Edit: Rule
   ├─ Add: Green TG, weight 25%
   ├─ Save changes

4. Monitor:
   ├─ Send requests: Multiple times
   ├─ Check: Both backends receiving
   ├─ Verify: Approximately 75%/25% split
   └─ Confirm: Weighting working

AWS CLI Management:

List All Rules:

```bash
aws elbv2 describe-rules \
  --listener-arn <listener-arn> \
  --region us-east-1
```

Output: All rules with conditions and actions

Describe Specific Rule:

```bash
aws elbv2 describe-rules \
  --rule-arns <rule-arn> \
  --region us-east-1
```

Create New Rule:

```bash
aws elbv2 create-rule \
  --listener-arn <listener-arn> \
  --priority 1 \
  --conditions Field=path-pattern,Values=/api/* \
  --actions Type=forward,TargetGroupArn=<tg-arn>
```

Modify Rule (Update Weights):

```bash
aws elbv2 modify-rule \
  --rule-arn <rule-arn> \
  --priority 2 \
  --actions Type=forward,ForwardConfig="{
    TargetGroups=[
      {TargetGroupArn=<blue-arn>,Weight=80},
      {TargetGroupArn=<green-arn>,Weight=20}
    ]
  }"
```

Delete Rule:

```bash
aws elbv2 delete-rule \
  --rule-arn <rule-arn>
```

Common Configuration Tasks:

Change Listener Port:

Not in rule: In listener itself
├─ ALB → Listeners
├─ Edit listener: Click edit
├─ Port: Change port number
├─ Protocol: Change HTTP/HTTPS
└─ Save: Applied

Add HTTPS/SSL:

1. Listener: Add new HTTPS listener
2. Port: 443
3. Certificate: Select from ACM
4. Rules: Create rules for new listener (same rules)
5. HTTP → HTTPS redirect: Also set up

Set Target Group Stickiness:

Not in rule: In target group
├─ Target group: Select
├─ Attributes: Edit
├─ Stickiness: Enable/configure
└─ Save: Applied

Best Practices:

1. Order rules correctly:
   ├─ Most specific: First (lower priority number)
   ├─ Least specific: Later (higher priority number)
   ├─ Default: Always last
   └─ Example: /api/v2/users before /api/*

2. Use descriptive names:
   ├─ Rule names: Clear purpose
   ├─ Example: "API-v2-requests" is better than "Rule-1"
   ├─ Helps: Understanding rules later
   └─ Documentation: Inline clarity

3. Test before production:
   ├─ Staging: Test all new rules
   ├─ Verify: Matching works correctly
   ├─ Check: All conditions function
   └─ Validate: Actions produce expected results

4. Monitor after changes:
   ├─ CloudWatch: Watch metrics
   ├─ Logs: Check for unusual activity
   ├─ Errors: Any new errors appearing?
   └─ Performance: Any latency changes?

5. Document rules:
   ├─ Purpose: Why does this rule exist?
   ├─ Conditions: What does it match?
   ├─ Action: What does it do?
   ├─ When: Date created, last modified
   └─ Owner: Who created/maintains it?
```

---

## Part 6: Exam Focus and Summary

### Test-Relevant Information

**Critical Concepts for Certification**

```
Essential Facts:

ALB Rules Core:
✓ Rules: Set on ALB (not target group)
✓ Quantity: Multiple rules per ALB allowed
✓ Order: Processed sequentially 1st to last
✓ Default: Last rule, catches unmatched
✓ First match: Wins (stops evaluation)

Condition Types:
✓ Host header: Domain routing
✓ HTTP method: GET/POST/etc
✓ Path pattern: /api/*, /users, etc
✓ Source IP: Client IP address
✓ HTTP header: Any header (custom too)
✓ Query string: URL parameters
✓ Logic: AND between different types
✓ Within: OR logic available

Action Types:
✓ Forward: To target group(s)
✓ Redirect: To different URL
✓ Fixed response: ALB-generated 200/400/500

Weighted Routing:
✓ Multiple TGs: Per single rule
✓ Weights: Control percentage split
✓ Distribution: Per-request random-ish
✓ Use case: Blue-green deployments

Exam Questions:

1. "What processes first when a request arrives?"
   A) Default rule
   B) Rule with lowest priority number
   C) Highest numbered rule
   D) Random rule
   
   Answer: B) Rule with lowest priority number
   └─ Know: Lower number = higher priority

2. "Can you have multiple target groups in one rule?"
   A) No, one rule = one target group
   B) Yes, with weights
   C) Yes, but only two
   D) No, requires multiple rules
   
   Answer: B) Yes, with weights
   └─ Know: Weighted routing for blue-green

3. "What does Host header condition match?"
   A) Backend server hostname
   B) HTTP Host header from client
   C) ALB hostname
   D) Domain name of ALB
   
   Answer: B) HTTP Host header from client
   └─ Know: Matches what client sends

4. "If no rules match, what happens?"
   A) Request dropped
   B) Error returned
   C) Default rule processes
   D) Random rule selected
   
   Answer: C) Default rule processes
   └─ Know: Default rule is catch-all

5. "How are weights calculated?"
   A) Fixed percentages 0-100%
   B) Ratio from total: weight/total weight
   C) Priority-based
   D) Load-based
   
   Answer: B) Ratio from total: weight/total weight
   └─ Know: Percentages auto-calculated

6. "What's a blue-green deployment?"
   A) Deploy binary and backup
   B) Current vs new version with gradual switch
   C) Backup to second AZ
   D) Blue=failed, green=working
   
   Answer: B) Current vs new version with gradual switch
   └─ Know: Weighted routing enables this

7. "Which condition allows routing based on ?param=value?"
   A) Query string parameter
   B) HTTP header
   C) Path pattern
   D) Source IP
   
   Answer: A) Query string parameter
   └─ Know: URL query strings specifically

8. "Can a rule redirect HTTP to HTTPS?"
   A) No, ALB only forwards
   B) Yes, via redirect action
   C) No, needs separate listener
   D) Only with fixed response
   
   Answer: B) Yes, via redirect action
   └─ Know: Redirect action type available

9. "What's the max character length for fixed response body?"
   A) 256 characters
   B) 512 characters
   C) 1024 characters
   D) Unlimited
   
   Answer: C) 1024 characters
   └─ Know: ALB constraint on fixed responses

10. "With weights 8:2, what's the traffic split?"
    A) 40%:60%
    B) 80%:20%
    C) 50%:50%
    D) 60%:40%
    
    Answer: B) 80%:20%
    └─ Know: 8/(8+2) = 80%, 2/(8+2) = 20%

11. "How many default rules can an ALB have?"
    A) One per listener
    B) Multiple
    C) Zero
    D) One per ALB
    
    Answer: A) One per listener
    └─ Know: Each listener has one default rule

12. "What enables gradual traffic shifting between versions?"
    A) Target group attributes
    B) Weighted target groups
    C) Health checks
    D) Deregistration delay
    
    Answer: B) Weighted target groups
    └─ Know: Core mechanism for gradual deployments

Key Concepts Summary:

Rules Basics:
├─ What: Routing logic on ALB
├─ Where: Per listener on ALB
├─ How: Conditions → Actions
├─ Important: Order matters

Conditions (6 types):
├─ Host, Method, Path, IP, Header, Query
├─ All conditions must match (AND)
├─ Multiple values within condition (OR)

Actions (3 types):
├─ Forward (to target group(s))
├─ Redirect (to URL)
├─ Fixed Response (from ALB)

Weighted Target Groups:
├─ Multiple targets: Per rule
├─ Weights: Percentages distributed
├─ Use: Blue-green, canary deployments

Common Scenario:

"You need to gradually roll out API v2 while keeping v1 running"

Solution:
├─ Create: Target group for v2
├─ Deploy: Instances to v2 target group
├─ Create rule: With weighted forward
│  ├─ v1 targets: Weight 100 initially
│  └─ v2 targets: Weight 0 initially
├─ Gradually: Increase v2 weight
│  ├─ Step 1: v1 95%, v2 5%
│  ├─ Step 2: v1 75%, v2 25%
│  ├─ Step 3: v1 50%, v2 50%
│  ├─ Step 4: v1 25%, v2 75%
│  ├─ Step 5: v1 0%, v2 100%
│  └─ Duration: 1+ hours with monitoring
├─ Monitor: Error rates, latency, health
├─ Verify: v2 performing as expected
└─ Standby: Ready to rollback at any point

Key Differences:

vs Target Group Routes:
├─ ALB rules: At load balancer level
└─ Target group: At instance level

vs Redirect vs Forward:
├─ Redirect: Client sent to new URL (GET request)
├─ Forward: Request sent to backend
└─ Fixed: Response sent by ALB

vs Auto Scaling:
├─ Rules: Static routing to target groups
├─ ASG: Dynamic instance count changes
└─ Together: ASG + weighted rules = powerful

Scenario Mapping:

Scenario 1: API versioning (v1, v2)
├─ Condition: Path = /api/v1/* or /api/v2/*
├─ Action: Forward to respective version targets
└─ Rule count: One per version or single with weights

Scenario 2: Host-based multitenant
├─ Condition: Host = company1.app.com, company2.app.com, etc
├─ Action: Forward to company-specific target group
└─ Rule count: One per company (or one with multiple hosts)

Scenario 3: Method-based routing
├─ Condition: Method = GET (read), POST (write)
├─ Action: Forward to read-optimized vs write-optimized
└─ Rule count: Two rules (or combined with path)

Scenario 4: Feature flags (A/B testing)
├─ Condition: Query = variant=new
├─ Action: Forward to new feature targets or old
└─ Rule count: One or two depending on granularity

Scenario 5: Admin access restriction
├─ Condition: Path = /admin/*, Source IP = 10.0.0.0/8
├─ Action: Forward to admin targets or fixed response (403)
└─ Rule count: One with restriction

Scenario 6: Blue-green deployment
├─ Condition: Path = /* (or path-specific)
├─ Action: Forward with weights
│  ├─ Current version: Weight 90
│  └─ New version: Weight 10
└─ Gradual: Increase new version weight over time
```

---

## Conclusion

ALB Rules and Request Routing provide sophisticated traffic control capabilities, enabling complex application architectures, gradual deployments, and advanced traffic management strategies. Master these concepts for both AWS SysOps certification and production deployment success!
