# Sticky Sessions and Session Affinity: Maintaining Client State

Sticky sessions (session affinity) ensure that a client's requests are consistently routed to the same backend instance, preserving application session state and user context across multiple requests to the load balancer.

---

## Part 1: Understanding Sticky Sessions

### What Are Sticky Sessions?

**Definition**
```
Sticky Sessions (Session Affinity):
├─ Mechanism that binds a client to a specific backend instance
├─ All requests from same client go to same instance
├─ Persists for duration of session (or cookie lifetime)
├─ Implemented via cookies
└─ Available on CLB, ALB, and NLB
```

**Example: Without Sticky Sessions (Default Load Balancing)**
```
Request Flow (Normal Round-Robin):

Client A → ALB → Instance 1 → Response
Client A → ALB → Instance 2 → Response (different instance!)
Client A → ALB → Instance 3 → Response (different instance!)
Client A → ALB → Instance 1 → Response (back to instance 1)

Pattern:
├─ Requests distributed across instances
├─ Each request may go to different backend
├─ Load evenly spread
└─ Session data lost between requests
```

**Example: With Sticky Sessions (Affinity)**
```
Request Flow (Sticky):

Client A → ALB → Instance 1 → Response (cookie sent)
Client A → ALB → Instance 1 → Response (same instance, cookie matches)
Client A → ALB → Instance 1 → Response (same instance, cookie matches)
Client A → ALB → Instance 1 → Response (same instance, cookie matches)

Pattern:
├─ First request determines instance
├─ All subsequent requests go to same instance
├─ Sticky until cookie expires
└─ Session data maintained on instance

Simultaneously:
Client B → ALB → Instance 2 → Response (different instance)
Client C → ALB → Instance 3 → Response (different instance)
```

### Why Use Sticky Sessions?

**Use Cases**

```
✓ Session State Storage:
  ├─ User login information
  ├─ Shopping cart contents
  ├─ User preferences
  ├─ Temporary data
  └─ In-memory application state

✓ Legacy Applications:
  ├─ Written before distributed systems
  ├─ Store session on local filesystem
  ├─ Assume same server for requests
  └─ Can't easily refactor for distribution

✓ Video Streaming:
  ├─ Buffering position
  ├─ Streaming metrics
  └─ Connection state

✓ Database Connections:
  ├─ Persistent connections
  ├─ Connection pooling
  └─ Per-instance state

✓ File Uploads:
  ├─ Temporary file storage
  ├─ Multi-part upload tracking
  └─ Per-instance temporary dir
```

**Scenario Example**

```
E-commerce Application Without Sticky Sessions:

1. User logs in
   └─ Request → Instance 1: Session created, user ID stored
   └─ Response: Session ID returned

2. User browses products
   └─ Request + Session Cookie → Instance 2
   └─ Instance 2: "Who is this user? No session found!"
   └─ Response: "Please login again" (session lost)

Problem:
├─ User must re-login
├─ User experience broken
├─ Lost shopping cart
└─ Orders not saved

With Sticky Sessions:

1. User logs in
   └─ Request → Instance 1: Session created, user ID = 12345
   └─ Response: Cookie + Session ID

2. User browses products
   └─ Request + Cookie → ALB → Instance 1 (same!)
   └─ Instance 1: "Ah, session ID matches user 12345"
   └─ Response: User's cart and preferences loaded

Result:
├─ User stays logged in
├─ Cart persists
├─ Seamless experience
└─ Orders work correctly
```

### Potential Drawbacks

**Sticky Session Tradeoffs**

```
Disadvantages:

1. Load Imbalance
   ├─ Some instances may get "sticky" users
   ├─ Other instances may be idle
   ├─ CPU/Memory utilization uneven
   ├─ One "hot" user ties up instance
   └─ Overall throughput reduced

2. Session Loss on Instance Failure
   ├─ If instance 1 crashes
   ├─ Client stuck to instance 1
   ├─ Health check removes instance
   ├─ Client rerouted to new instance
   └─ BUT session data lost (was on instance 1)

3. Server Restarts
   ├─ When instance restarts
   ├─ In-memory sessions cleared
   ├─ Clients still try to connect
   ├─ Get 503 error initially
   └─ New instance assigned

4. Scaling Issues
   ├─ Adding new instances
   ├─ Old sticky clients still use old instances
   ├─ New instances don't get traffic
   ├─ Scaling benefits negated
   └─ Until cookies expire

5. Multi-Region Issues
   ├─ Sticky session only works within one ALB
   ├─ If failover to different region
   ├─ Session not replicated
   └─ User loses session
```

**Better Alternatives**

```
Instead of Sticky Sessions:

1. Distributed Session Store
   ├─ Use Redis or DynamoDB
   ├─ Store session data centrally
   ├─ Any instance can retrieve session
   ├─ Works with any routing
   ├─ Survives instance failure
   └─ Recommended modern approach

2. Database-Backed Sessions
   ├─ Store session in RDS
   ├─ Query session per request
   ├─ Works across all instances
   ├─ Scales with database
   └─ Good for persistent sessions

3. Stateless Architecture
   ├─ Use JWT tokens
   ├─ Client carries authentication
   ├─ No server-side session needed
   ├─ Any instance can handle request
   └─ Most scalable approach

When Sticky Sessions Acceptable:
├─ Short-lived sessions
├─ Failure is acceptable
├─ Temporary data only
├─ Legacy apps you can't refactor
└─ Testing/development environments
```

---

## Part 2: How Sticky Sessions Work

### Cookie Mechanism

**Cookie-Based Implementation**

```
How Stickiness Works:

Step 1: First Request (Client A)
├─ Client sends: GET /index.html
├─ ALB receives request
├─ ALB selects instance (e.g., Instance 1)
├─ ALB adds tracking cookie
├─ Instance 1 processes request
└─ Response sent with cookie

Step 2: Cookie in Response
└─ Response headers include:
   ├─ Set-Cookie: AWSALB=abc123def456; Path=/; Expires=...
   └─ Browser stores cookie

Step 3: Second Request (Client A)
├─ Browser recalls cookie: abc123def456
├─ Browser sends: GET /products
├─ Request includes: Cookie: AWSALB=abc123def456
├─ ALB receives request with cookie
├─ ALB checks: "Cookie abc123def456 → Instance 1"
├─ Request routed to Instance 1
└─ Session continues on same instance

Step 4: Cookie Expires
├─ Time passes: Cookie lifetime expires
├─ Browser deletes cookie
├─ Next request without old cookie
├─ ALB assigns new instance
└─ Session disconnected
```

**Cookie Flow Diagram**

```
┌────────────────────────────────────────────┐
│ Client Browser                             │
│ ├─ First request: No cookies               │
│ └─ Cookies stored: AWSALB=abc123...       │
└────────────────────┬───────────────────────┘
                     │
                     ├─ Request 1 (no cookie)
                     ↓
┌────────────────────────────────────────────┐
│ ALB (Load Balancer)                        │
│ ├─ Receives: GET /index.html               │
│ ├─ No cookie present                       │
│ ├─ Generates: AWSALB=abc123def456         │
│ ├─ Selects Instance 1                      │
│ ├─ Stores mapping: abc123... → Instance 1 │
│ └─ Routes to Instance 1                    │
└────────────────────┬───────────────────────┘
                     │
                     ├─ Response + Cookie
                     ↓
┌────────────────────────────────────────────┐
│ Instance 1 (Backend)                       │
│ ├─ Processes request                       │
│ ├─ Sessions stored in memory               │
│ └─ Returns response                        │
└────────────────────────────────────────────┘
                     │
                     ├─ Response (with Set-Cookie)
                     ↓
                   Browser stores cookie
                     │
                     ├─ Request 2 (with cookie)
                     ↓
┌────────────────────────────────────────────┐
│ ALB (Load Balancer)                        │
│ ├─ Receives: GET /products                │
│ ├─ Cookie: AWSALB=abc123def456            │
│ ├─ Looks up: abc123... → Instance 1       │
│ └─ Routes to Instance 1 (SAME)            │
└────────────────────┬───────────────────────┘
                     │
                     ├─ Request forwarded
                     ↓
┌────────────────────────────────────────────┐
│ Instance 1 (Backend)                       │
│ ├─ Session found in memory                │
│ ├─ User data retrieved                    │
│ └─ Returns personalized response          │
└────────────────────────────────────────────┘
```

### Cookie Lifetime and Expiration

**Cookie Expiration Process**

```
Timeline:

t=0:      User makes first request
          └─ ALB creates cookie: AWSALB=abc123
             └─ Expires: tomorrow (default 1 day)

t=10min:  User makes request 2
          └─ Cookie still valid
          └─ Routes to same instance

t=5 hours: User makes request 3
          └─ Cookie duration: 5 hours elapsed
          └─ Cookie still valid (1 day total)
          └─ Routes to same instance

t=24 hours: User makes request 4 (next day)
          └─ Cookie expired
          └─ Browser removes cookie
          └─ Request sent without cookie
          └─ ALB assigns new instance

t=24+ hours: User makes request 5
          └─ New cookie generated: AWSALB=xyz789
          └─ Routes to new instance
          └─ New session created
```

**Cookie Lifetime Options**

```
Duration-Based Cookie Lifetime:

Available durations:
├─ 1 second: Almost useless (expires immediately)
├─ 1 minute: Very short, good for heavy load balancing
├─ 1 hour: Short sessions
├─ 1 day: Default, common choice
├─ 7 days: Long sessions, stateful apps
└─ Customizable: 1 second to 7 days

Considerations:

Short Duration (1 minute):
├─ Load balances frequently
├─ Session data not preserved long
├─ Good for APIs, stateless apps
└─ Better scaling

Long Duration (7 days):
├─ User stays on same instance
├─ Session data persists
├─ Instance becomes "sticky"
├─ Potential load imbalance
└─ Better for legacy apps
```

---

## Part 3: Types of Cookies for Sticky Sessions

### Cookie Type 1: Duration-Based Cookies

**Load Balancer Generated Cookies**

```
Duration-Based Cookie (Automatic):

├─ Generated by: Load balancer (not application)
├─ Cookie name: AWSALB (for ALB)
│                AWSELB (for CLB)
│                AWSNLB (for NLB)
├─ Duration: Specified by administrator (1 sec - 7 days)
├─ Expiration: Fixed time from cookie creation
└─ Configuration: Set at target group level

How It Works:
├─ You don't need to modify application
├─ ALB automatically creates cookie
├─ ALB manages cookie lifetime
├─ Application unaware of stickiness
└─ Transparent to your code

When to Use:
├─ Don't have application-specific session ID
├─ Want simple automatic stickiness
├─ Application doesn't generate cookies
├─ Quick implementation needed
└─ Most common use case

Example Configuration:
├─ Duration: 1 day (86400 seconds)
├─ Cookie name: AWSALB (fixed)
└─ ALB handles everything
```

**Cookie Details: Duration-Based**

```
Response Cookie from ALB:
Set-Cookie: AWSALB=abc123def456ghijkl; 
            Path=/; 
            Expires=Wed, 12 Apr 2026 15:30:45 GMT; 
            HttpOnly

Breakdown:
├─ Name: AWSALB (ALB reserved name)
├─ Value: abc123def456ghijkl (unique identifier)
├─ Path: / (applies to entire site)
├─ Expires: Tomorrow at this time (1 day duration)
├─ HttpOnly: Cannot access via JavaScript
└─ Secure: (might be added for HTTPS)

Browser Request (Same User, Next Hour):
Cookie: AWSALB=abc123def456ghijkl

ALB Processing:
├─ Receives cookie: abc123def456ghijkl
├─ Looks up in mapping: abc123... → Instance 2
├─ Routes request to Instance 2
└─ Same instance as before (sticky)
```

### Cookie Type 2: Application-Based Cookies

**Application Generated Cookies**

```
Application-Based Cookie:

├─ Generated by: Your application (custom code)
├─ Cookie name: You choose (e.g., MYCUSTOMAPP)
├─ Duration: Can be specified by application
├─ Attributes: You can include custom data
├─ Configuration: ALB monitors for this cookie
└─ Caveat: Specific cookie name per target group

How It Works:
├─ Your application creates session cookie
├─ Cookie name: Whatever you specified (e.g., SESSION_ID)
├─ ALB reads this cookie
├─ ALB uses it for stickiness
├─ Application continues working normally

When to Use:
├─ Application already generates session cookies
├─ Want to use existing session IDs
├─ Need custom attributes in cookie
├─ Application manages session lifecycle
└─ Cookie already contains user/session info

Example:

Application Creates:
├─ Cookie: JSESSIONID=9A8B7C6D5E4F3G2H1I
├─ Used by: Your Java/Python/Node app
└─ Contains: Session ID from application

ALB Configuration:
├─ Enable stickiness: YES
├─ Type: Application-based
├─ Cookie name: JSESSIONID
└─ ALB now uses this cookie for stickiness

ALB Processing:
├─ Receives: Cookie: JSESSIONID=9A8B7C6D5E4F3G2H1I
├─ Look up: JSESSIONID → Instance 1
├─ Route to: Instance 1 (sticky based on app cookie)
└─ Result: Same behavior as duration-based
```

**Application Cookie vs Duration-Based Cookie**

```
Comparison:

Feature                  Duration-Based      Application-Based
─────────────────────────────────────────────────────────────
Generated by:            ALB                 Your application
Cookie name:             AWSALB (fixed)      You choose custom
Example name:            AWSALB              JSESSIONID, SID
Configuration location:  Target group SG     Application code
Duration control:        ALB (1s-7 days)     Application script
Reserved names:          AWSALB, AWSALBAPP   None (you choose)
Suitable for:            Generic stickiness  Existing sessions
Application changes:     None needed         Might need tweaks
Use case:                Quick setup         Integration with app
Common in exams:         More common         Less common

Reserved Cookie Names (Can't Use):
├─ AWSALB (ALB reserved)
├─ AWSALBAPP (ALB reserved for app-generated)
├─ AWSALBTG (ALB reserved for target groups)
├─ AWSELB (CLB reserved)
└─ AWSNLB (NLB reserved)

You can use:
├─ MYCUSTOMCOOKIE
├─ SESSION_ID
├─ JSESSIONID
├─ PHPSESSID
└─ Any name except reserved ones
```

### Cookie Type 3: Application Cookie (ALB Variant)

**AWSALBAPP Cookie**

```
Application Cookie (ALB Variant):

├─ Generated by: ALB (not your application)
├─ Cookie name: AWSALBAPP (fixed)
├─ Duration: Application-specified
├─ Use: When you want ALB to manage but app provides name
└─ Less common than duration-based

Difference from Duration-Based:
├─ Duration-Based: Simple, fixed name AWSALB, ALB controls all
├─ AWSALBAPP: ALB generated but uses app-like settings
│
└─ Rarely used in practice, mostly duration-based

Most Used in Order:
1. Duration-Based (AWSALB) - Most common, simplest
2. Application-Based (JSESSIONID etc.) - When app has cookies
3. AWSALBAPP - Rare, niche use case
```

---

## Part 4: Configuring Sticky Sessions

### Step-by-Step Configuration

**Step 1: Identify Target Group**

```
AWS Console Navigation:
1. EC2 → Load Balancing → Target Groups
2. Find: Your target group (e.g., demo-tg-alb)
3. Click: Target group name
4. Status: Active with healthy targets
```

**Step 2: Access Stickiness Settings**

```
In Target Group Details:
1. Tab: "Actions" dropdown
2. Click: "Edit attributes"
   OR
   Look for: "Edit target group attributes" button

Alternative Path:
1. Target Group tab
2. Scroll down: Attributes section
3. Click: "Edit" button next to Stickiness
```

**Step 3: Enable Stickiness**

```
Stickiness Configuration Screen:

Field: Stickiness
├─ Toggle: OFF → ON

Once Enabled, Select Cookie Type:
├─ Option 1: "Load balancer generated cookie"
│  ├─ Cookie name: AWSALB (automatic)
│  ├─ Duration: (specify 1 sec to 7 days)
│  └─ Recommended: 86400 (1 day) - DEFAULT
│
└─ Option 2: "Application-based cookie"
   ├─ Cookie name: (you must specify)
   ├─ Example: JSESSIONID
   └─ Requires: Your app generates this cookie
```

**Step 4: Configure Duration-Based Stickiness**

```
If Using Load Balancer Generated:

Field: Duration
├─ Default value: 86400 seconds (1 day)
├─ Minimum: 1 second
├─ Maximum: 604800 seconds (7 days)
├─ You can customize: Yes

Common Duration Values:
├─ 60 seconds (1 minute)
├─ 300 seconds (5 minutes)
├─ 3600 seconds (1 hour)
├─ 86400 seconds (1 day) ← DEFAULT
└─ 604800 seconds (7 days)

Entry Method:
├─ Type in: 3600 (for 1 hour)
└─ Click: Save
```

**Step 5: Configure Application-Based Stickiness**

```
If Using Application-Based Cookie:

Field: Cookie name
├─ Must specify cookie name
├─ Example values:
│  ├─ JSESSIONID (Java apps)
│  ├─ PHPSESSID (PHP apps)
│  ├─ SESSIONID (custom)
│  └─ MYCUSTOMCOOKIE (any name)
│
├─ Enter: JSESSIONID
└─ Save

ALB Now:
├─ Monitors for JSESSIONID cookie
├─ If present in request
├─ Uses it for stickiness routing
└─ Routes to same instance which originally set it
```

**Step 6: Save Configuration**

```
Final Step:
1. Click: "Save changes" OR "Save" button
2. Status: "Saving attributes..."
3. Confirmation: "Attributes updated successfully"
4. Stickiness: Now ENABLED
```

### Verifying Sticky Sessions Are Enabled

**Check Target Group Attributes**

```
In Target Group Details:

Scroll to: "Attributes" section

Stickiness Enabled Display:
├─ Stickiness: Enabled
├─ Load balancer generated cookie: ✓ (if selected)
│  ├─ Duration: 86400 seconds
│  └─ Cookie name: AWSALB
│
OR

├─ Application-based cookie: ✓ (if selected)
│  ├─ Cookie name: JSESSIONID
│  └─ Stickiness duration: (inherited from app)
```

---

## Part 5: Testing Sticky Sessions

### Browser Testing with Developer Tools

**Accessing Developer Tools**

```
Keyboard Shortcuts:
├─ Chrome/Firefox/Edge: F12
├─ Or: Right-click → Inspect
├─ Or: Ctrl+Shift+I (Windows) / Cmd+Option+I (Mac)

Opening Network Tab:
1. Open Developer Tools (F12)
2. Click: "Network" tab
3. Reload page: Ctrl+R or Cmd+R
4. Network requests shown in list
```

**Observing Cookies**

```
For Each Request:

1. Find request: GET / (your load balancer URL)
2. Click request to expand details
3. Right panel shows multiple tabs:
   ├─ Headers
   ├─ Response
   ├─ Cookies ← CLICK HERE
   └─ Others

Cookies Tab Shows:

Response Cookies (From Server):
├─ Set-Cookie: AWSALB=abc123def456
│  ├─ Value: abc123def456 (unique identifier)
│  ├─ Domain: your-alb.elb.amazonaws.com
│  ├─ Path: /
│  ├─ Expires: Tomorrow 3:45 PM (1 day from now)
│  ├─ HttpOnly: Yes (cannot access via JavaScript)
│  ├─ Secure: Yes (HTTPS only, if using HTTPS)
│  └─ SameSite: Lax (CSRF protection)

Request Cookies (From Browser):
├─ AWSALB=abc123def456
└─ Sent with: Every subsequent request
```

### Testing: Multiple Requests to Same Instance

**Without Sticky Sessions**

```
Requests Pattern:

Request 1: GET / → Instance 1 (10.0.1.10) → "Hello from 10.0.1.10"
Request 2: GET / → Instance 3 (10.0.1.30) → "Hello from 10.0.1.30"
Request 3: GET / → Instance 2 (10.0.1.20) → "Hello from 10.0.1.20"
Request 4: GET / → Instance 1 (10.0.1.10) → "Hello from 10.0.1.10"

Observation:
├─ Different IPs in response each time
├─ Round-robin distribution
├─ No cookie present
└─ No stickiness
```

**With Sticky Sessions Enabled**

```
Requests Pattern:

Request 1: GET / → ALB → Instance 2 (10.0.1.20) → Response + Cookie: AWSALB=xyz789
Request 2: GET / + Cookie: AWSALB=xyz789 → ALB → Instance 2 (10.0.1.20) → "Hello from 10.0.1.20"
Request 3: GET / + Cookie: AWSALB=xyz789 → ALB → Instance 2 (10.0.1.20) → "Hello from 10.0.1.20"
Request 4: GET / + Cookie: AWSALB=xyz789 → ALB → Instance 2 (10.0.1.20) → "Hello from 10.0.1.20"

Observation:
├─ Same IP (10.0.1.20) in all responses
├─ Cookie present in requests
├─ Routed to same instance
├─ Stickiness working correctly ✓
```

**Testing Multiple Clients**

```
Client A Testing:
├─ Request 1: → Instance 2 (gets cookie A)
├─ Request 2: → Instance 2 (uses cookie A)
└─ Request 3: → Instance 2 (uses cookie A)

Client B Testing (Different Browser/Device):
├─ Request 1: → Instance 3 (gets cookie B)
├─ Request 2: → Instance 3 (uses cookie B)
└─ Request 3: → Instance 3 (uses cookie B)

Client C Testing (Incognito/Private):
├─ Request 1: → Instance 1 (gets cookie C)
├─ Request 2: → Instance 1 (uses cookie C)
└─ Request 3: → Instance 1 (uses cookie C)

Result:
├─ Each client has own cookie
├─ Each client sticks to own instance
├─ Simultaneous independent sessions
└─ Stickiness per client (not global)
```

### Analyzing Cookie Information

**Detailed Cookie Analysis**

```
Cookie Details in Developer Tools:

Response Header Example:
Set-Cookie: AWSALB=aB1cD2eF3gH4iJ5kL6mN7oP8qR9sT0uV1wX2yZ3; 
            Path=/; 
            Expires=Wed, 12 Apr 2026 15:30:45 GMT; 
            HttpOnly; 
            Secure

Breakdown:

1. Name: AWSALB
   └─ Reserved for ALB
   └─ Automatically managed

2. Value: aB1cD2eF3gH4iJ5kL6mN7oP8qR9sT0uV1wX2yZ3
   └─ Unique identifier
   └─ ALB internally maps to instance
   └─ No meaningful structure (encrypted/hashed)

3. Path: /
   └─ Applies to entire domain
   └─ Not restricted to /api or /admin

4. Expires: Wed, 12 Apr 2026 15:30:45 GMT
   └─ Tomorrow (24 hours from now)
   └─ Matches: Duration 86400 seconds (1 day)
   └─ Browser deletes after expiration

5. HttpOnly: Yes
   └─ JavaScript cannot access this cookie
   └─ Prevents XSS attack theft
   └─ Browser sends automatically only

6. Secure: Yes
   └─ Only sent over HTTPS
   └─ Not sent over HTTP (security feature)
   └─ Protects from network sniffer theft

7. SameSite: Lax (often implicit)
   └─ CSRF protection
   └─ Cookie sent only for same-site requests
   └─ Not sent on cross-site requests
```

**Cookie Timeline Verification**

```
Request 1 (1:00 PM):
├─ Response includes: Expires=Wed, 12 Apr 2026 17:00:45 GMT
└─ Duration: 2 hours until expiration

Request 2 (1:30 PM):
├─ Cookie still valid (halfway through lifetime)
├─ Browser sends: Cookie: AWSALB=aB1cD2eF3gH4iJ5kL6mN7oP8qR9sT0uV1wX2yZ3
└─ ALB routes to: Same instance

Request 3 (Next day, 1:30 PM):
├─ Cookie expired (24 hours passed)
├─ Browser automatically deletes cookie
├─ Browser sends: No cookie
└─ ALB assigns: New instance (new session)
```

---

## Part 6: Disabling Sticky Sessions

### Reverting to Normal Load Balancing

**How to Disable Stickiness**

```
To Remove Sticky Sessions:

1. Navigate: EC2 → Load Balancing → Target Groups
2. Select: Your target group
3. Click: "Actions" dropdown
4. Select: "Edit attributes"
5. Find: Stickiness section
6. Change: Stickiness from "Enabled" to "Disabled"
7. Click: "Save changes"
8. Verify: "Attributes updated successfully"
```

**Result of Disabling**

```
After Disabling Sticky Sessions:

Request 1: GET / → ALB → Instance 1 → "Hello from Instance 1"
Request 2: GET / → ALB → Instance 3 → "Hello from Instance 3"
Request 3: GET / → ALB → Instance 2 → "Hello from Instance 2"

Changes:
├─ No SET-COOKIE header in response
├─ No cookie stored by browser
├─ Each request load-balanced independently
├─ Back to default round-robin
└─ Session data not preserved across requests
```

### Cookie Cleanup

**Browser Cookie Handling**

```
When Disabled:

Existing Cookie:
├─ Browser may still have old AWSALB cookie
├─ ALB will IGNORE this cookie
├─ Next request: Routed to new instance anyway
└─ System handles gracefully

Complete Cleanup:
1. Clear browser cache/cookies: Ctrl+Shift+Delete
   or Cmd+Shift+Delete (Mac)
2. Or: Use Incognito/Private window (no cookies persist)
3. Or: Simply wait (cookie expires after duration)
```

---

## Part 7: Sticky Sessions on Other Load Balancers

### CLB (Classic Load Balancer) Stickiness

**Configuration on CLB**

```
Classic Load Balancer:
├─ Also supports sticky sessions
├─ Configuration: At load balancer level (not target group)
├─ Cookie name: AWSELB (CLB reserved)
├─ Configuration similar: Duration and cookie type
└─ Largely deprecated (use ALB instead)
```

### NLB (Network Load Balancer) Stickiness

**NLB Stickiness Characteristics**

```
Network Load Balancer:
├─ Supports: Sticky sessions (TCP layer)
├─ Cookie: Not applicable (Layer 4, no cookies)
├─ Instead: Uses connection tracking
├─ Method: Source IP + port based
├─ Duration: Client connection lifetime
├─ Use case: TCP protocols, not HTTP

NLB Flow Stickiness:
├─ All packets from same IP:port
│  └─ Routed to same backend
├─ When connection closes
│  └─ New connection might go to different backend
└─ Works at transport layer (no cookies involved)

Configuration:
├─ Target group attributes
├─ Stickiness: Enabled/Disabled
├─ Duration: Configurable (similar to ALB)
├─ No cookie options (Layer 4 only)
```

---

## Part 8: Session Management Best Practices

### Sticky Sessions vs Distributed Sessions

**Comparison**

```
Sticky Sessions Approach:
├─ Pros:
│  ├─ Easy to implement
│  ├─ No application changes
│  ├─ Works with legacy apps
│  └─ No infrastructure overhead
│
└─ Cons:
   ├─ Load imbalance possible
   ├─ Session lost if instance fails
   ├─ Doesn't scale across regions
   └─ Temporary/unreliable

Distributed Session Store Approach:
├─ Pros:
│  ├─ Perfect load balancing
│  ├─ Survives instance failures
│  ├─ Works across regions
│  ├─ Works across multiple ALBs
│  └─ Truly scalable
│
└─ Cons:
   ├─ Additional infrastructure (Redis/DB)
   ├─ Application code changes needed
   ├─ Additional cost (cache layer)
   └─ More complex to setup

Recommendation Priority:
1. Distributed sessions (Redis/DynamoDB) - Best practice
2. Sticky sessions (if must keep state) - Acceptable
3. Stateless (JWT tokens) - Most scalable
```

**Implementation Pattern**

```
Modern Best Practice: Stateless + Optional Cache

Application with Distributed State:

1. User Authenticates
   └─ Application creates JWT token
   └─ Token contains user ID, permissions
   └─ No server-side session needed

2. Token Storage
   └─ Client stores in browser localStorage
   └─ Browser sends in Authorization header
   └─ Not cookie-dependent

3. Request Handling
   └─ Client sends: Authorization: Bearer eyJhbGc...
   └─ Any instance can validate token
   └─ No stickiness needed
   └─ Perfect load balancing

4. Optional Session Cache
   └─ For performance optimization
   └─ User details cached in Redis
   └─ Keyed by: user_id
   └─ Any instance can lookup cache
   └─ Cache miss: Query database

Benefits:
├─ Load balanced evenly
├─ Works across regions
├─ Works across multiple ALBs
├─ Works with Auto Scaling
└─ Enterprise scalability
```

### When to Use Sticky Sessions

**Appropriate Use Cases**

```
Use Sticky Sessions When:

✓ Legacy application you can't refactor
  └─ Old monolith with session per instance
  └─ No time/budget to upgrade
  └─ Temporary measure

✓ Session duration very short
  └─ Few minutes per session
  └─ Failure acceptable
  └─ Acceptable data loss risk

✓ Small-scale deployments
  └─ 2-3 instances only
  └─ Not expecting 1000s of users
  └─ Low traffic volume

✓ Testing/development environment
  └─ Learning/experimentation
  └─ Not production

✓ Specific use cases
  └─ File upload progression tracking
  └─ Streaming position on specific instance
  └─ Other temporary connection state
```

**Avoid Sticky Sessions When**

```
Don't Use Sticky Sessions:

✗ Building new applications
  └─ Use distributed session store instead
  └─ Build scalability in from start

✗ Mission-critical systems
  └─ Session loss = business impact
  └─ Use session replication instead

✗ Global/multi-region systems
  └─ Sticky sessions don't cross regions
  └─ Use distributed store instead

✗ Auto-scaling environments
  └─ New instances don't get traffic
  └─ Stickiness defeats scaling benefits

✗ High-availability requirements
  └─ Instance failure = session loss
  └─ Unacceptable downtime

✗ Microservices architecture
  └─ Cross-service communication
  └─ Sticky sessions don't help
  └─ Use distributed state instead
```

---

## Part 9: Key Takeaways for SysOps Associate

### Sticky Sessions Essentials

**What to Remember**

```
1. Definition:
   └─ Client requests routed to same backend instance
   └─ Implemented via cookies
   └─ Preserves session state on instance

2. How It Works:
   └─ First request: ALB generates cookie
   └─ Subsequent requests: Cookie identifies sticky instance
   └─ Cookie expires: Stickiness ends, new instance assigned

3. Cookie Types:
   └─ Duration-based (AWSALB): Simple, ALB-generated
   └─ Application-based: App-generated cookie name you specify

4. Reserved Cookie Names:
   └─ AWSALB (ALB duration-based)
   └─ AWSALBAPP (ALB app-based variant)
   └─ AWSALBTG (ALB target group)
   └─ AWSELB (CLB)
   └─ Cannot use these names for custom cookies

5. Configuration:
   └─ Target group level (not load balancer)
   └─ Enable stickiness on desired target group
   └─ Choose duration (1 sec to 7 days)
   └─ Default: 86400 seconds (1 day)

6. Tradeoffs:
   └─ Pro: Easy, preserves session
   └─ Con: Load imbalance, instance failure = session loss

7. When to Use:
   └─ Legacy applications
   └─ Short-lived sessions
   └─ No distributed state store available
   └─ Testing/development

8. Better Alternatives:
   └─ Distributed sessions (Redis/DynamoDB)
   └─ Stateless (JWT tokens)
   └─ Both more scalable for production
```

### Exam Likely Questions

**Question Types**

```
1. "How to preserve user session across multiple requests?"
   A) Add more instances
   B) Enable sticky sessions on target group
   C) Increase ALB capacity
   D) Use multiple ALBs
   
   Answer: B) Enable sticky sessions on target group
   └─ Sticky sessions maintain session on same instance

2. "Cookie names cannot use AWSALB. Which is valid?"
   A) AWSALBTG
   B) JSESSIONID
   C) AWSALBAPP
   D) AWSELB
   
   Answer: B) JSESSIONID
   └─ JSESSIONID is not reserved
   └─ Others are AWS reserved names

3. "What happens when sticky session cookie expires?"
   A) User is logged out
   B) Session data deleted
   C) ALB assigns new instance
   D) ALB creates new cookie
   
   Answer: C) ALB assigns new instance
   └─ Cookie expires, browser stops sending it
   └─ Next request: New instance assigned
   └─ Session data lost (was on old instance)

4. "Cookie is set by application. What to configure?"
   A) Duration-based cookie
   B) Application-based cookie
   C) AWSALB cookie
   D) AWSELB cookie
   
   Answer: B) Application-based cookie
   └─ Provider app cookie name
   └─ ALB monitors for that cookie

5. "Multiple sticky sessions impacting ALB performance?"
   A) Increase ALB capacity
   B) Use NLB instead
   C) Disable sticky sessions / use distributed sessions
   D) Add more target groups
   
   Answer: C) Disable sticky sessions / use distributed sessions
   └─ Sticky sessions cause load imbalance
   └─ Remove stickiness or refactor with cache layer
```

### Sticky Sessions Checklist

```
When implementing sticky sessions:

☑ Identify whether sticky sessions are needed
  └─ Or use distributed session store instead

☑ Choose cookie type
  └─ Duration-based (simpler) vs Application-based

☑ Set appropriate duration
  └─ Match application session timeout
  └─ Don't set too long or too short

☑ Enable at target group level
  └─ Not on load balancer level

☑ Test with browser dev tools
  └─ Verify cookies in Network tab
  └─ Confirm same instance across requests

☑ Monitor for load imbalance
  └─ Check CloudWatch metrics
  └─ Ensure even distribution

☑ Plan for failures
  └─ Document session loss risk
  └─ Plan recovery strategy

☑ Document in runbook
  └─ How to enable/disable
  └─ How to troubleshoot
  └─ When sticky sessions are in effect
```

---

## Part 10: Hands-On Summary

### What Was Demonstrated

```
1. Enabling sticky sessions:
   └─ Target group → Actions → Edit attributes
   └─ Stickiness: Enabled
   └─ Duration: 1 day (default)
   └─ Type: Load balancer generated (AWSALB)

2. Testing before stickiness:
   └─ Multiple requests to ALB
   └─ Different instance IPs in responses
   └─ Round-robin distribution observed

3. Testing after stickiness:
   └─ Multiple requests to ALB
   └─ Same instance IP in all responses
   └─ Cookie verified in developer tools

4. Observing cookies:
   └─ Open Browser Dev Tools (F12)
   └─ Network tab
   └─ HTTP request details
   └─ Cookies section:
      ├─ Response cookies (Set-Cookie)
      ├─ Request cookies (sent with each request)
      └─ Expiration date visible

5. Disabling stickiness:
   └─ Return to target group
   └─ Actions → Edit attributes
   └─ Stickiness: Disabled
   └─ Back to round-robin
```