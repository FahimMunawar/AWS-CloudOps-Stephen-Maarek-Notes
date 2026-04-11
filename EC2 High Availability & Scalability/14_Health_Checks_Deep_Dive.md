# Health Checks: Deep Dive and Configuration

Health checks continuously monitor target instance health, allowing load balancers to route traffic only to healthy instances. Configurable parameters enable fine-tuning for different application types and response characteristics.

---

## Part 1: Health Checks Fundamentals

### What Are Health Checks?

**Health Check Concept**

```
Health Check Purpose:

Definition:
├─ Periodic test: Send to backend instances
├─ Purpose: Verify instance is functioning
├─ Frequency: At regular intervals
├─ Action: Mark healthy or unhealthy based on response
└─ Effect: Route or don't route traffic accordingly

Why Health Checks Matter:

Problem Without Health Checks:
├─ Instance failure: LB doesn't know
├─ Traffic routes: To broken instances
├─ Users experience: Errors, timeouts
├─ Reliability: Poor
└─ Recovery: Manual intervention needed

Solution With Health Checks:
├─ Instance failure: Detected automatically
├─ Traffic routes: Only to healthy instances
├─ Users experience: Seamless (fail over to others)
├─ Reliability: High
└─ Recovery: Automatic

Flow Diagram:

Target Group contains:
├─ Instance 1 (Healthy ✓) → Receives traffic
├─ Instance 2 (Healthy ✓) → Receives traffic
├─ Instance 3 (Unhealthy ✗) → No traffic sent

Health Check Process:

Every [interval] seconds:
├─ ALB sends health check request
├─ To each: Instance in target group
├─ Instance responds: HTTP 200 (or configured code)
├─ ALB evaluates: Response and timing
├─ Decision: Mark as healthy or unhealthy
└─ Action: Update routing accordingly
```

**Key Components**

```
Health Check Configuration:

What it Tests:
├─ Connectivity: Can reach the instance
├─ Port: Is port open/listening
├─ Protocol: Does protocol speak correctly (HTTP/HTTPS/TCP)
├─ Application: Is app responding
└─ Business logic: Custom check path (if defined)

What it Doesn't Test:
├─ Database: May not check backend DB
├─ External dependencies: May not check APIs
├─ Full transaction: Usually simple request
├─ Performance: Usually not performance-based
└─ Memory usage: Does not check resource levels

Health Check Scope:

Typical Health Check:
├─ Type: Usually HTTP request to simple endpoint
├─ Path: /, /health, or custom path
├─ Response: Expected HTTP status code (usually 200)
├─ Time: Should respond within timeout
└─ Frequency: Every 30 seconds (default)

Custom Health Check Example:

Instead of: GET /
├─ Can use: GET /api/health
├─ Can check: /health/deep for dependencies
├─ Can use: POST /health-check with data
└─ Return: 200 if healthy, 503 if unhealthy
```

---

## Part 2: Health Check Configuration Parameters

### Health Check Protocol

**Protocol Selection**

```
Protocol Options:

HTTP:
├─ Default: Yes, most common
├─ Use case: Web application health endpoints
├─ Security: No encryption
├─ Port: Typically 80
├─ When to use: Most applications
└─ Example: GET http://instance-ip:80/health

HTTPS:
├─ Default: No, opt-in
├─ Use case: Encrypted health checks
├─ Security: Encrypted communication
├─ Port: Typically 443
├─ When to use: If instance expects HTTPS
├─ Example: GET https://instance-ip:443/health

TCP:
├─ Default: No, opt-in
├─ Use case: Generic TCP connectivity
├─ Security: No application protocol (raw TCP)
├─ Port: Custom
├─ When to use: Non-HTTP services (databases, etc.)
├─ Example: Open TCP connection to instance:5432

UDP:
├─ Default: No, opt-in
├─ Use case: UDP services
├─ Security: No connection (datagram based)
├─ Port: Custom
├─ When to use: UDP-based applications
└─ Example: Send UDP packet to instance:5353

Choosing Protocol:

For Web Apps (ALB):
├─ Use: HTTP or HTTPS
├─ Default: HTTP
├─ Reason: Application understands HTTP
└─ Most common: HTTP

For API Services:
├─ Use: HTTP or HTTPS
├─ Recommend: HTTPS if APIs require it
├─ Typical: HTTP (faster, internal VPC)
└─ Consider: Custom /health endpoint

For Databases (NLB):
├─ Use: TCP
├─ Reason: Databases speak TCP protocol
├─ Check: Connection possible
└─ Port: Database port (5432, 3306, etc.)

For Internal Services:
├─ Use: What service speaks
├─ Typical: HTTP for REST APIs
├─ Alternative: TCP for binary protocols
└─ Custom: Depends on service
```

**Protocol Performance Considerations**

```
Protocol Overhead:

HTTP:
├─ Overhead: Protocol headers ~250-500 bytes
├─ Parsing: Application must parse HTTP
├─ Latency: Usually 5-10ms for simple request
├─ CPU: Noticeable for high-frequency checks
└─ Recommendation: Default, fine for most

HTTPS:
├─ Overhead: Adds TLS handshake
├─ Latency: Significantly slower (50-100ms+)
├─ CPU: Much higher due to encryption
├─ When justified: Security requirement
└─ Recommendation: Use HTTP if possible (VPC is private)

TCP:
├─ Overhead: Minimal (connection only)
├─ Latency: Very fast (1-5ms)
├─ CPU: Minimal
├─ Resource: Very light
└─ Recommendation: Use for performance-critical checks

Recommendation:
├─ Default: Use HTTP
├─ Reason: Good balance of functionality and performance
├─ When HTTPS: Only if required for security or app limitation
├─ When TCP: Non-HTTP services or check frequency very high
└─ Performance: Usually not health checks' main concern
```

### Health Check Port

**Port Configuration**

```
Port Options:

Default Port:
├─ Port 80 (HTTP)
├─ Applies to: Most web applications
├─ Standard: HTTP protocol
└─ Assumption: App listening on 80

Custom Port:
├─ Any port: 1-65535
├─ When used: App on different port
├─ Examples: 8080, 8443, 3000, 5000
└─ Must match: What application listens on

Traffic Port Override:
├─ Special: Use same port as traffic
├─ Alternative: Different port for health checks
├─ When different: App runs services on multiple ports
└─ Example: Traffic on 80, health checks on 8080

Port Selection Scenarios:

Scenario 1: Standard Web App
├─ Traffic port: 80 (HTTP)
├─ Health check port: 80 (same)
├─ Configuration: Use default
└─ Simple: No override needed

Scenario 2: App on Non-Standard Port
├─ Traffic port: 3000 (Node.js)
├─ Health check port: 3000 (same)
├─ Configuration: Set port to 3000
└─ Still works: Both use same port

Scenario 3: Separate Health Port
├─ Traffic port: 80 (main app)
├─ Health check port: 8080 (dedicated health endpoint)
├─ Configuration: Set health port to 8080
├─ Benefit: Health check separate from user traffic
└─ Common in: High-traffic scenarios

Scenario 4: Multi-Port App
├─ Traffic port: 80 (HTTP) or 443 (HTTPS)
├─ Health check port: 8080 (independent service)
├─ Configuration: Enable traffic port override
└─ Strategy: Health checks don't compete with traffic

Configuration:

In AWS Console:
├─ AdvancedSettings: Traffic Port Override
├─ Option 1: Use listener port (default)
├─ Option 2: Use health check port
├─ Many apps: Default (same port) works fine
└─ Some apps: Override needed for separation

In CLI:
$ aws elbv2 modify-target-group \
  --target-group-arn <arn> \
  --health-check-port 8080
```

### Health Check Path

**Custom Health Check Endpoint**

```
Path Options:

Root Path:
├─ Path: /
├─ Request: GET / HTTP/1.1
├─ Response: Full home page
├─ Use: If app responds to /
├─ Issue: May be slow/heavy
└─ When: Simple apps without /health

Custom Health Endpoints:
├─ Path: /health
├─ Benefit: Lightweight response
├─ Best: App has dedicated endpoint
├─ Typical: Returns quick 200 OK
└─ Common: Recommended approach

Specific API Endpoints:
├─ Path: /api/health
├─ Path: /status
├─ Path: /healthz
├─ Path: /liveness
├─ Path: /readiness
└─ Use: Whatever app implements

Path Selection:

Best Practice - Dedicated Endpoint:

Application Code (Example):

// Simple health check endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy' });
});

// Return immediately
// No database queries
// No heavy computation
// Just: "I'm alive and ready"

Configuration in AWS:
├─ Path: /health
├─ Fast response: < 1 second
├─ Lightweight: Minimal resources
└─ Reliable: Predictable behavior

Alternative - Root Path:

If no /health endpoint:
├─ Path: /
├─ Returns: Full HTML home page
├─ Issues: Slower response
├─ Problem: More data transferred
├─ Impact: Higher resource usage
└─ Not ideal: But works

Kubernetes-Style Health:

If app follows Kubernetes conventions:
├─ Path: /healthz
├─ Return: 200 OK
├─ Liveness: /healthz (is app alive)
├─ Readiness: /readyz (can serve traffic)
├─ Use: Whatever your app implements
└─ Check: App documentation

Creating Effective Health Endpoints:

Guidelines:

1. Response Time:
   ├─ Should: Return in < 1 second
   ├─ Ideal: 100-200ms
   └─ Avoid: Any I/O operations

2. Resource Usage:
   ├─ Minimal: CPU usage
   ├─ Minimal: Memory usage
   ├─ No: Database queries
   └─ No: External API calls

3. Response Payload:
   ├─ Small: JSON response (100 bytes or less)
   ├─ Include: Necessary status only
   ├─ Example: {"status": "ok"}
   └─ Avoid: Verbose responses

4. HTTP Status Code:
   ├─ 200 OK: Instance is healthy
   ├─ 503 Service Unavailable: Instance unhealthy
   └─ Use: Appropriate codes for state

5. Dependencies:
   ├─ Local: Only checks within instance
   ├─ No: Database connectivity check (usually)
   ├─ No: External service dependencies
   └─ Why: Makes checks fail for wrong reasons

Example Code:

Node.js/Express:
```javascript
app.get('/health', (req, res) => {
  // Simple health check - just return OK
  res.status(200).json({ status: 'ok' });
});
```

Python/Flask:
```python
@app.route('/health')
def health():
    return {'status': 'ok'}, 200
```

Java/Spring:

```java
@GetMapping("/health")
public ResponseEntity<?> health() {
    return ResponseEntity.ok().body(
        Collections.singletonMap("status", "ok")
    );
}
```

Go/Gin:
```go
router.GET("/health", func(c *gin.Context) {
    c.JSON(200, gin.H{"status": "ok"})
})
```
```

### Health Check Timeout

**Response Timeout Parameter**

```
Timeout Definition:

What It Is:
├─ Maximum: Time to wait for response
├─ Unit: Seconds
├─ Range: 2-120 seconds
├─ Default: 5 seconds
└─ If exceeded: Health check marked failed

How It Works:

Timeline:

T=0 seconds: Health check request sent
├─ ALB sends: GET /health HTTP/1.1
├─ Message: Sent to instance
└─ Status: Waiting for response

T=1 second: Instance processing
├─ Instance: Receives request
├─ Instance: Processing begins
└─ Status: ALB still waiting

T=3 seconds: Instance responds
├─ Instance: Sends HTTP 200 response
├─ Message: Received by ALB
├─ Status: Response time = 3 seconds ✓ Success

T=5+ seconds (if no response):
├─ Default timeout reached
├─ ALB concludes: Instance not responding
├─ Status: Health check failed ✗

Timeout Scenarios:

Scenario 1: Fast Response (< Timeout)

Health Check Response: 100ms
├─ Timeout: 5 seconds
├─ Received: T=0.1 seconds
├─ Comparison: 0.1 < 5 ✓
├─ Result: Passed ✓
└─ Instance: Still healthy

Scenario 2: Slow Response (< Timeout)

Health Check Response: 4.5 seconds
├─ Timeout: 5 seconds
├─ Received: T=4.5 seconds
├─ Comparison: 4.5 < 5 ✓
├─ Result: Passed ✓
└─ Instance: Still healthy (but slow)

Scenario 3: Timeout Exceeded

Health Check Response: Hangs (7+ seconds)
├─ Timeout: 5 seconds
├─ Received: Never (hangs)
├─ Wait time: 5 seconds
├─ Action: Timeout fires
├─ Result: Failed ✗
└─ Instance: Marked unhealthy

Setting Appropriate Timeout:

For Fast Health Checks:
├─ Simple HTTP: /health endpoint
├─ Typical time: 100-500ms
├─ Recommended timeout: 2-3 seconds
├─ Logic: 5-10x actual response time
└─ Example: If avg 200ms, set 2-3 seconds

For Slower Applications:
├─ Database operations: May be slower
├─ Complex checks: Multiple steps
├─ Typical time: 1-2 seconds
├─ Recommended timeout: 3-5 seconds
└─ Example: If avg 2 seconds, set 5 seconds

For Very Slow Operations:
├─ Database queries: Slow backend
├─ External calls: To other services
├─ Typical time: 5+ seconds
├─ Recommended timeout: 10-30 seconds
└─ Example: If avg 8 seconds, set 15 seconds

Recommendation Strategy:

1. Measure actual response time
   ├─ During: Normal conditions
   ├─ Log: Response times to health endpoint
   └─ Calculate: 95th or 99th percentile

2. Set timeout:
   ├─ Formula: Timeout = (p99_response_time × 2) OR (p95_response_time × 3)
   ├─ Avoid: Too tight (false failures)
   ├─ Avoid: Too loose (slow failure detection)
   └─ Target: Balance

3. Test:
   ├─ Simulate: Instance under load
   ├─ Monitor: Response time variance
   ├─ Verify: Timeout works as expected
   └─ Adjust: If needed

Configuration Examples:

Default (Good for Most):
├─ Timeout: 5 seconds
├─ Use: Quick /health endpoint
└─ Works: For most web applications

Optimized for Speed:
├─ Timeout: 2 seconds
├─ Use: Very fast responses
└─ Suitable: High-frequency checks

Optimized for Reliable Services:
├─ Timeout: 10 seconds
├─ Use: More complex checks
└─ Suitable: Critical applications

Conservative (Avoid False Positives):
├─ Timeout: 30 seconds
├─ Use: Slow applications
└─ Suitable: Background workers, batch jobs

Note:
├─ Too short: False negatives (mark healthy as unhealthy)
├─ Too long: Slow failure detection
└─ Best: Find balance for your application
```

### Health Check Interval

**Frequency of Health Checks**

```
Interval Definition:

What It Is:
├─ Time between: Consecutive health checks
├─ Unit: Seconds
├─ Range: 5-300 seconds
├─ Default: 30 seconds
└─ If very low: May overwhelm application

How It Works:

Timeline Example (30 second interval):

T=0s:  Health check #1 sent to Instance A
       └─ Response: 200 OK ✓

T=30s: Health check #2 sent to Instance A
       └─ Response: 200 OK ✓

T=60s: Health check #3 sent to Instance A
       └─ Response: 200 OK ✓

T=90s: Health check #4 sent to Instance A
       └─ Response: 200 OK ✓

Pattern: Every 30 seconds, new check sent

Frequency Impact:

Low Interval (5-10 seconds):
├─ Frequency: Very often
├─ Failure detection: Very fast
├─ Response time: Quick to know about issues
├─ Trade-off: More health check traffic
├─ Resource: Higher CPU on instances
├─ When: Need rapid failure detection
└─ Example: Critical real-time services

Medium Interval (30 seconds - Default):
├─ Frequency: Regular
├─ Failure detection: Within 30 seconds
├─ Balance: Good balance
├─ Trade-off: Reasonable overhead
├─ Resource: Minimal impact
├─ When: Most applications
└─ Example: Web applications

Long Interval (60-300 seconds):
├─ Frequency: Infrequent
├─ Failure detection: May take 2-5 minutes
├─ Response time: Slower to detect issues
├─ Trade-off: Less overhead
├─ Resource: Minimal load
├─ When: Stable infrastructure
└─ Example: Background jobs

Interval Configuration Scenarios:

Scenario 1: Web Application (Default Good)

Application: eCommerce website
├─ Change frequency: Rare
├─ Stability: Usually stable
├─ Recovery needed: Minutes acceptable
├─ Config: 30 seconds (default)
└─ Rationale: Balanced approach

Scenario 2: Real-Time Service (Fast Detection)

Application: Stock trading platform
├─ Downtime: Very costly
├─ Recovery needed: Seconds matter
├─ Stability: Less stable
├─ Config: 5-10 seconds
└─ Rationale: Rapid failure detection

Scenario 3: Batch Processing (Slow OK)

Application: Data processing cluster
├─ Change frequency: Stable for hours
├─ Downtime: Not critical per-check
├─ Recovery needed: Many changes slow
├─ Config: 300 seconds (5 minutes)
└─ Rationale: Minimal overhead

Calculating Health Check Traffic:

Formula:
└─ Health checks per instance = (60 seconds / interval)
└─ Total checks = instances × (60 / interval)

Example 1 - 30 second interval:
├─ 10 instances target group
├─ Checks per minute: 60 / 30 = 2
├─ Total per minute: 10 × 2 = 20 checks/min
├─ Total per day: 20 × 1440 = 28,800 checks
└─ Impact: Minimal

Example 2 - 5 second interval (aggressive):
├─ 10 instances target group
├─ Checks per minute: 60 / 5 = 12
├─ Total per minute: 10 × 12 = 120 checks/min
├─ Total per day: 120 × 1440 = 172,800 checks
└─ Impact: Noticeable, but usually OK

Recommendation Strategy:

Define Objectives:
├─ How fast: Must detect failures?
├─ Acceptable downtime: During detection?
├─ Resource budget: For health checks?
└─ Answer: Determines interval

Choose Based on Use Case:
├─ Critical: 5-10 seconds
├─ Important: 15-30 seconds (default usually good)
├─ Standard: 30 seconds (most apps)
├─ Stable/Background: 60-300 seconds
└─ Document: Rationale for your choice

Test and Monitor:
├─ Monitor: Health check response times
├─ Track: Instance load during checks
├─ Verify: Interval appropriate
├─ Adjust: If needed
└─ Document: Final configuration

Default Recommendation:
├─ For most: Keep 30 seconds
├─ Reason: Good baseline
└─ Only change: If specific need
```

---

## Part 3: Health Check Thresholds

### Healthy Threshold Count

**Marking Instance as Healthy**

```
Healthy Threshold Definition:

What It Is:
├─ Number of: Consecutive successful health checks
├─ Required before: Marking instance as healthy
├─ Unit: Count/number
├─ Range: 2-10
├─ Default: 5
└─ Effect: How many passes until healthy

How It Works:

Timeline Example (Default: 5 passes needed):

Initial state: Instance started but unknown

T=0s:  Check #1 → Unhealthy ✗
T=30s: Check #2 → Healthy ✓ (1/5 passes)
T=60s: Check #3 → Healthy ✓ (2/5 passes)
T=90s: Check #4 → Healthy ✓ (3/5 passes)
T=120s: Check #5 → Healthy ✓ (4/5 passes)
T=150s: Check #6 → Healthy ✓ (5/5 passes) ← Now Healthy!

Total time: 150 seconds (5 minutes) to become healthy

Healthy Threshold Impact:

Low Count (2-3 passes):
├─ Time to healthy: Fast (1-2 minutes)
├─ Instances serving: Quickly after start
├─ Risk: May mark up before ready
├─ When: Less critical applications
└─ Benefit: Faster recovery/startup

Default Count (5 passes):
├─ Time to healthy: Medium (2.5 minutes)
├─ Instances serving: After reasonable wait
├─ Balance: Good balance of safety and speed
├─ When: Most applications
└─ Recommendation: Default usually good

High Count (8-10 passes):
├─ Time to healthy: Slow (4-5 minutes)
├─ Instances serving: After very careful verification
├─ Risk: Slower recovery/startup
├─ When: Critical applications
└─ Benefit: Very confident instance ready

Strategic Choices:

Strategy 1: Fast Activation (Low Threshold)

Scenario:
├─ Application: Boots quickly and is stable
├─ Examples: Stateless web server
├─ Needs: Quick recovery

Configuration:
├─ Healthy threshold: 2
├─ Interval: 30 seconds
├─ Time to healthy: ~60 seconds

Calculation:
├─ 2 successful checks × 30 sec interval = 60 seconds
└─ Instance: Ready to serve in 1 minute

Strategy 2: Balanced (Default)

Scenario:
├─ Application: Normal startup time
├─ Examples: Most web applications
├─ Needs: Good balance

Configuration:
├─ Healthy threshold: 5 (default)
├─ Interval: 30 seconds
├─ Time to healthy: ~150 seconds

Calculation:
├─ 5 successful checks × 30 sec interval = 150 seconds
└─ Instance: Ready to serve in 2.5 minutes

Strategy 3: Conservative (High Threshold)

Scenario:
├─ Application: Complex startup process
├─ Examples: Database-heavy applications
├─ Needs: Very confident before serving

Configuration:
├─ Healthy threshold: 10
├─ Interval: 30 seconds
├─ Time to healthy: ~300 seconds

Calculation:
├─ 10 successful checks × 30 sec interval = 300 seconds
└─ Instance: Ready to serve in 5 minutes

When to Adjust:

Increase Threshold If:
├─ Seeing: false positives (marked healthy too early)
├─ Problems: Instances crash/fail after becoming healthy
├─ Solution: Increase count (e.g., 5 → 8)
└─ Effect: Wait longer before allowing traffic

Decrease Threshold If:
├─ Seeing: Slow recovery after failures
├─ Problem: Auto Scaling too slow
├─ Seeing: Long startup delays
├─ Solution: Decrease count (e.g., 5 → 2)
└─ Effect: Faster traffic routing to new instances

Configuration:

Default (Recommended for Most):
├─ Threshold: 5
├─ Rationale: Good balance
└─ Continue: Unless specific need

Optimize for Speed:
├─ Threshold: 2
├─ When: Fast-starting apps
└─ Interval: Keep at 30 seconds

Optimize for Reliability:
├─ Threshold: 8
├─ When: Complex startups
└─ Interval: Maybe increase too (e.g., 60 sec)
```

### Unhealthy Threshold Count

**Marking Instance as Unhealthy**

```
Unhealthy Threshold Definition:

What It Is:
├─ Number of: Consecutive failed health checks
├─ Required before: Marking instance as unhealthy
├─ Unit: Count/number
├─ Range: 2-10
├─ Default: 3
└─ Effect: Rapid detection of failures

How It Works:

Timeline Example (Default: 3 failures needed):

Initial state: Instance healthy and serving traffic

T=0s:  Check #1 → Healthy ✓
T=30s: Check #2 → Healthy ✓
T=60s: Check #3 → Failed ✗ (1/3 failures)
       [Instance still receives traffic]

T=90s: Check #4 → Failed ✗ (2/3 failures)
       [Instance still receives traffic]

T=120s: Check #5 → Failed ✗ (3/3 failures) ← Now Unhealthy!
        [Traffic stops immediately]

Total time: 90 seconds (until marked unhealthy)

Unhealthy Threshold Impact:

Low Count (2-3 failures):
├─ Time to unhealthy: Very fast (1-2 minutes)
├─ Traffic stops: Quickly
├─ Risk: May be too sensitive (false positives)
├─ When: Detect problems immediately
└─ Example: 2 failures = 60 seconds with 30s interval

Medium Count (3-5 failures):
├─ Time to unhealthy: Medium (2-5 minutes)
├─ Traffic stops: After reasonable delay
├─ Balance: Tolerates some transient issues
├─ When: Most applications
└─ Example: 3 failures = 90 seconds with 30s interval

High Count (5-10 failures):
├─ Time to unhealthy: Slow (5-10 minutes)
├─ Traffic stops: Only after many failures
├─ Risk: Tolerates problems too long
├─ When: Stable but noisy environments
└─ Example: 5 failures = 150 seconds with 30s interval

Strategic Choices:

Strategy 1: Fast Detection (Low Threshold)

Scenario:
├─ Application: Must stop serving immediately on failure
├─ Examples: Payment processing, health apps
├─ Needs: Zero tolerance for issues

Configuration:
├─ Unhealthy threshold: 2
├─ Interval: 10 seconds (aggressive)
├─ Time to unhealthy: ~20 seconds

Impact:
├─ 2 failed checks × 10 sec interval = 20 seconds
├─ Traffic: Diverted very quickly
└─ Risk: May react to network blips too fast

Strategy 2: Balanced (Default)

Scenario:
├─ Application: Normal failure tolerance
├─ Examples: Most web applications
├─ Needs: Reasonable balance

Configuration:
├─ Unhealthy threshold: 3 (default)
├─ Interval: 30 seconds (default)
├─ Time to unhealthy: ~90 seconds

Impact:
├─ 3 failed checks × 30 sec interval = 90 seconds
├─ Traffic: Diverted after reasonable delay
└─ Tolerance: Some transient issues OK

Strategy 3: Tolerant (High Threshold)

Scenario:
├─ Application: Networking unstable/noisy
├─ Examples: External network connections
├─ Needs: Less sensitive to temporary issues

Configuration:
├─ Unhealthy threshold: 5
├─ Interval: 30 seconds
├─ Time to unhealthy: ~150 seconds

Impact:
├─ 5 failed checks × 30 sec interval = 150 seconds
├─ Traffic: Continues longer despite issues
└─ Rationale: May recover from temporary problems

When to Adjust:

Increase Threshold If:
├─ Seeing: False positives (mark unhealthy for temporary glitches)
├─ Problems: Network blips cause unnecessary failovers
├─ Noise: High variance in response times
├─ Solution: Increase count (e.g., 3 → 5)
└─ Effect: More tolerance for transient failures

Decrease Threshold If:
├─ Seeing: Slow failover to healthy instances
├─ Problems: Broken instances serve traffic too long
├─ Impact: User sees errors
├─ Solution: Decrease count (e.g., 3 → 2)
└─ Effect: Faster failover

Configuration:

Default (Recommended for Most):
├─ Threshold: 3
├─ Rationale: Good balance
└─ Continue: Unless specific need

Critical Application:
├─ Threshold: 2
├─ When: Need fast failover
└─ Interval: May decrease too (e.g., 10 sec)

Stable Network:
├─ Threshold: 3 or 4
├─ When: Traffic patterns stable
└─ Interval: Standard 30 seconds

Noisy Network:
├─ Threshold: 5
├─ When: Transient issues common
└─ Interval: May keep at 30 seconds

Comparison Table:

Threshold   Time to Unhealthy   Sensitivity   Use Case
─────────────────────────────────────────────────────
2           60 seconds          High/Fast     Critical apps
3           90 seconds          Balanced      Most apps (default)
5           150 seconds         Low/Tolerant  Stable/Noisy networks
10          300 seconds         Very Low      Rare/Experimental
```

---

## Part 4: Target Health Status States

### Understanding Target States

**Complete Health Status Lifecycle**

```
Target Status States:

A target can be in one of six states:

1. INITIAL:
   ├─ When: Just registered to target group
   ├─ Duration: First few seconds
   ├─ Status: Not yet evaluated
   ├─ Traffic: None routed
   ├─ Reason: Waiting for first health check
   └─ Next: Transitions to Healthy or Unhealthy

2. HEALTHY:
   ├─ When: Health checks passing
   ├─ Status: Ready to receive traffic
   ├─ Traffic: Receives traffic normally
   ├─ Healthy threshold: Met
   └─ Next: Stays Healthy (or goes Unhealthy)

3. UNHEALTHY:
   ├─ When: Health checks failing
   ├─ Status: Not ready for traffic
   ├─ Traffic: No traffic routed
   ├─ Reason: Exceeded unhealthy threshold
   └─ Next: Stays Unhealthy (or goes Healthy if checks pass again)

4. UNUSED:
   ├─ When: Not registered to any target group
   ├─ Status: Idle
   ├─ Traffic: None
   ├─ Reason: Instance exists but unused
   └─ Next: Register to target group → INITIAL

5. DRAINING:
   ├─ When: Being deregistered
   ├─ Status: Graceful shutdown in progress
   ├─ Traffic: No new traffic, existing completes
   ├─ Duration: Connection Draining timeout
   └─ Next: Deregistered completely

6. UNAVAILABLE:
   ├─ When: Health checks disabled
   ├─ Status: Not evaluated
   ├─ Traffic: May or may not receive (depends on LB)
   ├─ Reason: Health checks turned off
   └─ Next: Re-enable health checks

State Transition Diagram:

                    REGISTER
                       │
                       ▼
    UNUSED ◄────── INITIAL ────────► UNHEALTHY
             │                           │
             │                           ▼
             │                      More failures
             │                           │
             └─────► HEALTHY ◄──────────┘
                       │
                       │ Deregister or mark unhealthy
                       ▼
                    DRAINING
                       │
                       │ (Timeout expires)
                       ▼
                  (No longer shown)

Health Check Interaction:

INITIAL → HEALTHY (when health checks pass):
├─ Healthy threshold count: Met
├─ Example: 5 successful checks
└─ Result: Receives traffic

INITIAL → UNHEALTHY (if health checks fail):
├─ Unhealthy threshold count: Met
├─ Example: 3 failed checks
└─ Result: No traffic

HEALTHY → UNHEALTHY (when checks fail):
├─ Unhealthy threshold count: Met
├─ Example: 3 consecutive failures
└─ Result: Traffic diverted to other targets

UNHEALTHY → HEALTHY (when checks pass):
├─ Healthy threshold count: Met
├─ Example: 5 successful checks
└─ Result: Traffic restored
```

**Important Exam Fact: All Targets Unhealthy**

```
Critical Behavior:

What Happens If ALL Targets Unhealthy:

Scenario:
├─ Target group has: 3 instances
├─ Instance 1: Unhealthy ✗
├─ Instance 2: Unhealthy ✗
├─ Instance 3: Unhealthy ✗
├─ All targets: Failed health checks
└─ Question: What does LB do?

Standard Behavior (If Some Healthy):
├─ Route: To healthy instances only
├─ Unhealthy: Get no traffic
└─ Result: Clean failover

Edge Case Behavior (All Unhealthy):

Load Balancer Action:
├─ Doesn't: Simply drop all traffic
├─ Instead: Routes to ALL instances (even unhealthy)
├─ Reason: Best-effort attempt
└─ Logic: Assumes health check might be wrong

Why This Behavior?

Reasoning:
├─ Problem: If all targets unhealthy and drop traffic
├─ Result: Total service outage
├─ Alternative: Route to unhealthy instances
├─ Chance: Some might still work partially
└─ Philosophy: Better to try than fail completely

Analogy:
├─ Scenario: All instances are "sick"
├─ Option A: Turn away all customers (100% failed)
├─ Option B: Serve customers despite sickness (might work)
├─ LB chooses: Option B (best effort)
└─ Reasoning: Partial service > No service

Practical Implications:

Time to Investigate:
├─ If all targets unhealthy: Something wrong
├─ Usually: Health check configuration error
├─ Common causes:
│  ├─ Wrong port
│  ├─ Wrong path
│  ├─ Expected status code mismatch
│  ├─ All instances actually broken
│  └─ Security group blocking health checks
│
└─ Action: Don't panic, investigate health check config

This is Different From:

Expectations:
├─ Many: Expect load balancer to stop traffic
├─ When: All targets unhealthy
├─ Actually: LB still routes (best effort)
└─ Key learning: Remember for exam

Exam Question Template:

"If all targets in a target group are marked unhealthy,
what does the load balancer do?"

A) Stop all traffic immediately
B) Continue routing to all targets (best-effort)
C) Require manual intervention
D) Wait for targets to become healthy

Answer: B) Continue routing to all targets (best-effort)
└─ Key points:
   ├─ Not about: Waiting for recovery
   ├─ About: Best-effort attempt to serve
   └─ Reason: Prevents complete service outage
```

---

## Part 5: Hands-On Configuration

### Configuring Health Checks in AWS Console

**Step-by-Step Walkthrough**

```
Step 1: Navigate to Target Group

1. Open AWS Console
   └─ URL: https://console.aws.amazon.com

2. Go to: EC2 Dashboard
   ├─ Service: EC2
   └─ Region: Select your region

3. Find Load Balancing Section:
   ├─ Left sidebar: Load Balancing
   ├─ Option: Target Groups
   └─ Click: Target Groups

4. Select Your Target Group:
   ├─ Find: Your target group (e.g., "web-servers")
   ├─ Status: Should show instances
   └─ Click: To open details

Step 2: Access Health Check Settings

1. In Target Group Details:
   ├─ Tab or Section: Health Checks
   ├─ Look for: "Edit health check settings"
   └─ Click: To edit

2. Health Check Settings Page Opens:
   ├─ Shows: All configurable parameters
   └─ Ready: For modification

Step 3: Configure Protocol and Port

1. Find: Protocol dropdown
   ├─ Options: HTTP, HTTPS, TCP, UDP
   ├─ Select: Appropriate protocol (usually HTTP)
   └─ Applied: To all checks

2. Find: Port field
   ├─ Default: 80 (for HTTP)
   ├─ Change: To your application's port
   ├─ Example: 3000, 8080, custom port
   └─ Applied: To all checks

Step 4: Configure Health Check Path

1. Find: Health check path field
   ├─ Default: /
   ├─ Change: To custom path (e.g., /health)
   └─ Examples:
      ├─ / (root)
      ├─ /health
      ├─ /api/health
      ├─ /status
      └─ /liveness

2. Enter: Your custom path
   ├─ Consider: Fast, lightweight endpoint
   ├─ Avoid: Heavy operations
   └─ Save: Ready for testing

Step 5: Configure Advanced Settings

In "Advanced settings" section:

Timeout Setting:
├─ Find: Health check timeout
├─ Default: 5 seconds
├─ Adjust: Based on endpoint response time
├─ Typical: 2-10 seconds
├─ Enter: Your value

Interval Setting:
├─ Find: Health check interval
├─ Default: 30 seconds
├─ Adjust: Based on failure detection needs
├─ Typical: 10-300 seconds
├─ Enter: Your value

Healthy Threshold:
├─ Find: Healthy threshold count
├─ Default: 5
├─ Adjust: Based on startup time
├─ Typical: 2-10
├─ Enter: Your value

Unhealthy Threshold:
├─ Find: Unhealthy threshold count
├─ Default: 3
├─ Adjust: Based on sensitivity needs
├─ Typical: 2-10
├─ Enter: Your value

Success Status Code:
├─ Find: Expected status code
├─ Default: 200
├─ Alternative: 200-299 (any 2xx)
├─ Custom: Specific code your app uses
├─ Enter: Your value (usually 200)

Traffic Port Override:
├─ Option: Use listener port vs health check port
├─ Default: Use health check port (specified above)
├─ When different: Enable if needed
└─ Typical: Leave default

Step 6: Review Configuration

Before Saving:
├─ Protocol: Matches your app ✓
├─ Port: Correct for your app ✓
├─ Path: Valid on your app ✓
├─ Timeout: Reasonable for your endpoint ✓
├─ Interval: Appropriate for your needs ✓
├─ Thresholds: Set for your scenario ✓
└─ Status code: What your app returns ✓

Step 7: Save Configuration

1. Find: Save button (usually at bottom)
   ├─ Label: "Save" or "Apply"
   └─ Click: To save changes

2. Confirmation:
   ├─ Message: "Successfully updated"
   ├─ Status: Changes applied
   └─ Timing: Immediate effect

Step 8: Monitor Results

1. Return to Target Group Details:
   ├─ Targets section: Shows instances
   ├─ Status: Should show healthy/unhealthy
   └─ If healthy: Health checks working ✓

2. CloudWatch Monitoring:
   ├─ Metric: HealthyHostCount (should increase)
   ├─ Metric: UnHealthyHostCount (should decrease)
   └─ Timing: May take a few minutes

3. If Issues:
   ├─ Check: Still unhealthy?
   ├─ Debug: Security groups, port, path
   ├─ Try: Manual test of endpoint
   └─ Troubleshoot: As needed
```

### Testing Health Check Endpoint

**Verification Commands**

```
Manual Testing from Command Line:

Test 1: Using curl

Command:
$ curl -v http://instance-ip:port/health-path

Example (HTTP, Port 80, /health):
$ curl -v http://10.0.1.100:80/health

Output:
├─ Connection established: Shows success
├─ HTTP Status: Shows response code (should be 200)
├─ Response body: Shows returned data
└─ Timing: Shows response time

Successful Response Example:
```
< HTTP/1.1 200 OK
< Content-Type: application/json
< Content-Length: 15
<
{"status":"ok"}
```

Failed Response Example:
```
< HTTP/1.1 503 Service Unavailable
< Content-Type: application/json
<
{"status":"down"}
```

Test 2: Using wget

Command:
$ wget -O - http://instance-ip:port/health-path

Example:
$ wget -O - http://10.0.1.100:80/health

Output: Similar to curl, shows status code and body

Test 3: Using telnet (TCP only)

Command:
$ telnet instance-ip port

Example:
$ telnet 10.0.1.100 80

Output:
├─ Connection successful: Port is open
├─ Connection refused: Port not listening
└─ Timeout: No response (firewall?)

From EC2 Instance to Itself:

SSH into Instance:
$ ssh -i key.pem ec2-user@instance-ip

Then run curl:
$ curl localhost:80/health
$ curl http://127.0.0.1:3000/health
$ curl http://10.0.1.100:8080/health

From Load Balancer:

AWS CLI (local machine):
$ aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>

Output shows:
├─ TargetId: Instance ID
├─ Port: Port being checked
├─ State: Healthy/Unhealthy/Initial/etc
├─ Reason: Why (if unhealthy)
└─ Description: Additional details

Example Output:
```json
{
  "TargetHealthDescriptions": [
    {
      "Target": {
        "Id": "i-0123456789abcdef0",
        "Port": 80
      },
      "TargetHealth": {
        "State": "healthy",
        "Reason": "N/A",
        "Description": "N/A"
      }
    }
  ]
}
```

Troubleshooting with Detailed Info:

If Unhealthy - Check:

1. Security Group:
   $ aws ec2 describe-security-groups \
     --group-ids sg-xxxxx \
     --region us-east-1

   Verify: 
   ├─ Inbound rule for port?
   ├─ Protocol correct? (TCP for port 80)
   └─ Source: Allow from ALB security group?

2. Instance Status:
   $ aws ec2 describe-instance-status \
     --instance-ids i-xxxxx \
     --region us-east-1

   Verify:
   ├─ Instance state: running
   ├─ System status: OK
   └─ Instance status: OK

3. Application Log:
   $ tail -f /var/log/application.log
   $ tail -f /var/log/httpd/access_log

   Look for:
   ├─ Health check requests: /health
   ├─ Responses: Status codes
   └─ Errors: Application issues

4. Network Connectivity:
   $ netstat -tlnp | grep LISTEN

   Verify:
   ├─ Port is listening
   ├─ Correct port shown
   └─ Process running

5. Application Health:
   $ systemctl status application
   $ ps aux | grep application

   Verify:
   ├─ Process running
   ├─ No error state
   └─ Resource usage reasonable

Common Issues and Fixes:

Issue 1: Port Not Listening
├─ Problem: Application not running on port
├─ Test: netstat -tlnp
├─ Fix: Start application on correct port
└─ Verify: Retry health check

Issue 2: Security Group Blocking
├─ Problem: No inbound rule for health check port
├─ Test: telnet instance-ip port
├─ Fix: Add inbound rule for port from LB SG
└─ Verify: Retry health check

Issue 3: Wrong Health Path
├─ Problem: /health endpoint doesn't exist
├─ Test: curl http://localhost/health
├─ Fix: Use correct path or create /health endpoint
└─ Verify: Retry health check

Issue 4: Slow Health Endpoint
├─ Problem: Timeout exceeded
├─ Test: time curl http://localhost/health
├─ Fix: Increase timeout or optimize endpoint
└─ Verify: Should respond in < timeout
```

---

## Part 6: Common Issues and Troubleshooting

### All Targets Unhealthy

**Diagnosis and Resolution**

```
Symptom:
├─ All targets: Marked unhealthy
├─ Target group: Shows all red (unhealthy)
├─ Status: Initial or Unhealthy for all
└─ Traffic: Still routes (best-effort)

Root Causes (Most to Least Common):

Cause 1: Wrong Health Check Path (30% of cases)

Symptom:
├─ Path: Doesn't exist on application
├─ Example: /health but app only has /
├─ Result: 404 Not Found response
├─ Effect: Marked unhealthy

Diagnosis:
1. Check target group health check settings
   └─ Health check path: What is it?
   
2. SSH to instance, test path:
   $ curl http://localhost:80/health
   
   If 404:
   └─ Problem confirmed

3. What paths does app respond to?
   $ curl http://localhost:80/
   $ curl http://localhost:80/api
   $ curl http://localhost:80/status

Solution:
├─ Option A: Fix health check path
│  ├─ Go to target group
│  ├─ Edit: Health check path
│  ├─ Change to: Correct path
│  └─ Verify: Tests pass
│
└─ Option B: Add /health endpoint to app
   ├─ Code: Add simple /health route
   ├─ Returns: 200 OK with status
   └─ Deploy: Application with new endpoint

Cause 2: Wrong Port (25% of cases)

Symptom:
├─ Port: Set to wrong value
├─ Example: Health checks on 80 but app on 3000
├─ Result: Connection refused/timeout
├─ Effect: Marked unhealthy

Diagnosis:
1. Check target group settings
   └─ Port: What is it configured for?
   
2. Check application process
   $ netstat -tlnp | grep application
   $ ss -tlnp
   
   Look for:
   ├─ What port is app listening on?
   └─ Does it match target group?

Solution:
├─ Option A: Change health check port
│  ├─ Go to target group
│  ├─ Edit: Port number
│  ├─ Change to: Correct port
│  └─ Save and verify
│
└─ Option B: Change application port
   ├─ Stop: Application
   ├─ Config: Change port
   ├─ Start: Application
   └─ Verify: Listening on correct port

Cause 3: Security Group Blocking (20% of cases)

Symptom:
├─ Port: Should be reachable
├─ However: Connection timeout
├─ Reason: Security group denying traffic
├─ Effect: Marked unhealthy

Diagnosis:
1. From EC2 instance, test self:
   $ curl http://localhost:port/health
   
   If works:
   └─ Issue is external (security group)
   
2. Check instance security group:
   $ aws ec2 describe-security-groups \
     --group-ids sg-xxxxx

   Look for:
   ├─ Inbound rules
   ├─ Port: Is it open?
   ├─ Protocol: TCP?
   └─ Source: Is LB SG included?

3. Check if LB can reach instance:
   $ telnet instance-ip port
   
   If refuses:
   └─ Security group likely blocking

Solution:
├─ Add inbound rule to instance SG:
│  ├─ Protocol: TCP
│  ├─ Port: Port instance listens on
│  ├─ Source: Load Balancer security group
│  └─ Save and retry
│
└─ Alternative source:
   ├─ If reference SG not working: Use CIDR
   ├─ Source: 0.0.0.0/0 (any source, not recommended)
   └─ Better: Reference LB security group

Cause 4: Application Not Running (15% of cases)

Symptom:
├─ Process: Not running
├─ Port: Not listening
├─ Status: Connection refused
├─ Effect: All instances unhealthy

Diagnosis:
1. Check if process running:
   $ ps aux | grep application-name
   $ systemctl status application-name
   
2. Check logs:
   $ tail -50 /var/log/application.log
   
   Look for:
   ├─ Startup errors
   ├─ Crash messages
   └─ Configuration issues

3. Try to start manually:
   $ application-name start
   $ systemctl start application-name
   
   Check:
   └─ Does it start successfully?

Solution:
├─ Start application:
│  $ systemctl start application-name
│  $ service application-name start
│
├─ Enable on boot:
│  $ systemctl enable application-name
│
└─ Check logs:
   └─ Fix startup errors

Cause 5: Wrong Status Code (5% of cases)

Symptom:
├─ Endpoint: Responds successfully
├─ Status code: Not 200 (e.g., 503)
├─ Effect: Marked unhealthy
└─ Configuration: Status code mismatch

Diagnosis:
1. Test endpoint:
   $ curl -v http://localhost:port/health
   
   Output:
   ├─ Check: "HTTP/1.1 XXX"
   ├─ Example: "HTTP/1.1 200 OK"
   └─ Record: Actual status code

2. Check target group config:
   ├─ Expected status codes: What are they set to?
   ├─ Default: 200
   └─ Does endpoint return this?

Solution:
├─ Option A: Fix status code in app
│  ├─ Endpoint: Should return 200 OK
│  ├─ Fix code: Return correct status
│  └─ Deploy: Updated application
│
└─ Option B: Update expected status codes
   ├─ Target group settings
   ├─ Status codes: 200 or 200-299
   ├─ Save changes
   └─ Recheck targets

Generic Troubleshooting Checklist:

When All Targets Unhealthy - Check:

☐ Protocol:
  ├─ HTTP? For web app
  ├─ HTTPS? If encrypted needed
  └─ TCP? For non-HTTP services

☐ Port:
  ├─ Application listening on port?
  ├─ netstat -tlnp
  └─ Verify: Correct port

☐ Path:
  ├─ Endpoint exists on app?
  ├─ curl http://localhost:port/path
  └─ 404? Wrong path

☐ Status Code:
  ├─ App returns 200?
  ├─ Or other code?
  └─ Match target group config

☐ Security Group:
  ├─ Inbound rule exists?
  ├─ Port and protocol correct?
  └─ Source allows LB?

☐ Application:
  ├─ Process running?
  ├─ ps aux | grep
  ├─ systemctl status
  └─ Logs for errors?

☐ Network:
  ├─ Can reach instance?
  ├─ telnet instance port
  └─ From LB's VPC?

☐ Settings in Target Group:
  ├─ Timeout reasonable?
  ├─ Not less than response time?
  └─ Interval OK?

Once Fixed:
├─ Targets: Should become Healthy
├─ Time: May take healthy threshold × interval
├─ Traffic: Restored automatically
└─ Verify: All targets show healthy
```

### Slow Health Check Recovery

**Problem and Solution**

```
Symptom:
├─ Instance: Marked unhealthy due to issue
├─ Issue: Gets fixed/resolved
├─ Healthy threshold: Takes very long to activate
├─ Wait time: 3-5+ minutes
└─ Problem: Too slow

Causes:

High Healthy Threshold:
├─ Example: Set to 10
├─ Interval: 30 seconds
├─ Calculation: 10 × 30 = 300 seconds (5 min)
├─ Effect: Takes 5 minutes to become healthy
└─ Solution: Reduce threshold

Low Interval:
├─ Actually not: Interval too long
├─ Example: Interval 300 seconds (5 min)
├─ Between checks: 5 minutes wait
├─ Effect: May not detect health quickly
└─ Solution: Reduce interval

Both Combined:
├─ High threshold: 10
├─ Long interval: 60+ seconds
├─ Total: Could be 10 minutes or more!
└─ Solution: Reduce both

Solution Adjustments:

Before (Takes 5 minutes):
├─ Healthy threshold: 10
├─ Interval: 30 seconds
├─ Calculation: 10 × 30 = 300 seconds
└─ Wait: Very long

After (Takes 1 minute):
├─ Healthy threshold: 2
├─ Interval: 30 seconds
├─ Calculation: 2 × 30 = 60 seconds
└─ Wait: Much faster

Optimization:
├─ Reduce threshold: To 2-3 (from 10)
├─ Keep interval: At 30 seconds
└─ Result: 1-2 minutes recovery

Trade-off:
├─ Faster recovery: Yes ✓
├─ Risk of false positives: Higher
└─ Balance: Choose carefully

For Critical Services:

Extra Aggressive:
├─ Healthy threshold: 2
├─ Interval: 5-10 seconds
├─ Calculation: 2 × 10 = 20 seconds
└─ Recovery: Very fast

Risk:
├─ False positives: Higher chance
├─ Flapping: Instance going up/down
└─ Mitigation: Stable applications only

Configuration Steps:

1. Go to target group
2. Edit health check settings
3. Find: Healthy threshold
4. Reduce: From current to lower value
5. Find: Interval
6. Consider: Reducing interval too
7. Save: Changes
8. Test: Verify behavior
```

---

## Part 7: Exam Focus Points

### Key Exam Questions

```
1. "What protocol options for health checks?"
   A) HTTP only
   B) HTTP, HTTPS, TCP
   C) HTTP, TCP, UDP
   D) HTTPS, TLS only
   
   Answer: B) HTTP, HTTPS, TCP (also UDP)
   └─ Most common: HTTP (web apps)
   └─ Encrypted: HTTPS
   └─ Non-HTTP: TCP

2. "Default health check timeout?"
   A) 2 seconds
   B) 5 seconds
   C) 10 seconds
   D) 30 seconds
   
   Answer: B) 5 seconds
   └─ Range: 2-120 seconds
   └─ Can be customized per workload

3. "Default health check interval?"
   A) 10 seconds
   B) 15 seconds
   C) 30 seconds
   D) 60 seconds
   
   Answer: C) 30 seconds
   └─ Range: 5-300 seconds
   └─ Balance: Good default for most

4. "Default healthy threshold count?"
   A) 2
   B) 3
   C) 5
   D) 10
   
   Answer: C) 5
   └─ Range: 2-10
   └─ 5 attempts before marking healthy

5. "Default unhealthy threshold count?"
   A) 2
   B) 3
   C) 5
   D) 10
   
   Answer: B) 3
   └─ Range: 2-10
   └─ 3 failures before marking unhealthy

6. "What happens if ALL targets unhealthy?"
   A) Load balancer stops all traffic
   B) Routes to all targets (best-effort)
   C) Waits for any to become healthy
   D) Returns 503 error to clients
   
   Answer: B) Routes to all targets (best-effort)
   └─ Important: Doesn't drop all traffic
   └─ Reason: Better than queuing/timeout

7. "Healthy threshold count = 5, interval = 30s"
   A) Marks healthy in: 30 seconds
   B) Marks healthy in: 90 seconds
   C) Marks healthy in: 150 seconds
   D) Marks healthy in: 25 seconds
   
   Answer: C) Marks healthy in: 150 seconds
   └─ Calculation: 5 × 30 = 150 seconds
   └─ Time for 5 successful checks

8. "What is health check path used for?"
   A) Where health checks sent
   B) Routing to target
   C) URL path for health endpoint
   D) Log path on instance
   
   Answer: C) URL path for health endpoint
   └─ Example: /health, /api/health
   └─ Practice: Custom path is best
```

### Important Concepts

```
Key Terms:

1. Health Check Protocol:
   └─ HTTP, HTTPS, TCP, UDP
   └─ Determines: How instance contacted

2. Health Check Path:
   └─ /health or custom endpoint
   └─ Determines: Which endpoint queried

3. Timeout:
   └─ Max seconds waiting for response
   └─ Default: 5 seconds

4. Interval:
   └─ Seconds between checks
   └─ Default: 30 seconds

5. Healthy Threshold:
   └─ Consecutive passes before healthy
   └─ Default: 5

6. Unhealthy Threshold:
   └─ Consecutive failures before unhealthy
   └─ Default: 3

7. Target States:
   └─ Initial, Healthy, Unhealthy, Unused, Draining, Unavailable

8. Best-Effort Routing:
   └─ All targets unhealthy? Route anyway
   └─ Philosophy: Something > Nothing

Calculation Formulas:

Time to Healthy:
└─ Time = Healthy Threshold × Interval
└─ Example: 5 × 30 = 150 seconds

Time to Unhealthy:
└─ Time = Unhealthy Threshold × Interval
└─ Example: 3 × 30 = 90 seconds

Health Check Frequency:
└─ Frequency = 60 / Interval
└─ Example: 60 / 30 = 2 checks per minute
```

---

## Part 8: Best Practices

### Configuration Best Practices

```
1. Choose Appropriate Path:
   ├─ Best: Dedicated /health endpoint
   ├─ Quick: Responds in 100-200ms
   ├─ Lightweight: No database queries
   ├─ Reliable: Returns consistent result
   └─ Avoid: Heavy operations on / path

2. Set Realistic Timeout:
   ├─ Measure: Actual response time
   ├─ Add buffer: 2-3x of p99 time
   ├─ Too short: False failures
   ├─ Too long: Slow failure detection
   └─ Typical: 2-10 seconds for web apps

3. Choose Appropriate Interval:
   ├─ For most: 30 seconds is good
   ├─ Critical: 10-15 seconds
   ├─ Stable: 60-300 seconds
   ├─ Consider: Traffic generation
   └─ Test: Measure impact on instances

4. Set Thresholds Based on Requirements:
   ├─ Healthy threshold:
   │  ├─ Fast startup: 2-3
   │  └─ Careful startup: 5-10
   │
   └─ Unhealthy threshold:
      ├─ Quick failover: 2
      └─ Tolerant: 3-5

5. Use Meaningful Status Codes:
   ├─ 200: Instance is healthy
   ├─ 503: Instance unavailable (graceful degrades)
   ├─ Avoid: Using 404 or other misleading codes
   └─ Configure: Match your app's behavior

6. Monitor and Alert:
   ├─ CloudWatch: HealthyHostCount
   ├─ CloudWatch: UnHealthyHostCount
   ├─ Alerts: When all targets unhealthy
   ├─ Logs: Check health check patterns
   └─ Regular: Review and adjust

7. Security Considerations:
   ├─ VPC: Health checks within VPC (secure)
   ├─ Security group: Allow from LB SG
   ├─ Port: Use non-standard if security concern
   ├─ HTTPS: Not usually necessary (VPC private)
   └─ Data: Don't include sensitive info in response

8. Test Configuration:
   ├─ Staging: Test before production
   ├─ Simulate: Failures and recoveries
   ├─ Manual: Test via curl from LB
   ├─ Verify: All instances become healthy
   └─ Monitor: During traffic loads

Recommended Configuration by Workload:

Web Application (eCommerce, SaaS):
├─ Protocol: HTTP
├─ Path: /health (custom endpoint)
├─ Port: 80
├─ Timeout: 3 seconds
├─ Interval: 30 seconds
├─ Healthy threshold: 5
├─ Unhealthy threshold: 3
└─ Status code: 200

API Service:
├─ Protocol: HTTP
├─ Path: /api/health or /status
├─ Port: 443 (if HTTPS app) or 80
├─ Timeout: 2 seconds
├─ Interval: 15-30 seconds
├─ Healthy threshold: 3
├─ Unhealthy threshold: 2
└─ Status code: 200 or 200-299

Real-Time Service (Critical):
├─ Protocol: HTTP or HTTPS
├─ Path: /health (minimal check)
├─ Port: Application port
├─ Timeout: 2 seconds
├─ Interval: 5-10 seconds
├─ Healthy threshold: 2
├─ Unhealthy threshold: 2
└─ Status code: 200

Background/Batch Job:
├─ Protocol: TCP
├─ Path: N/A (TCP doesn't use path)
├─ Port: Listening port
├─ Timeout: 10 seconds
├─ Interval: 60-300 seconds
├─ Healthy threshold: 5
├─ Unhealthy threshold: 5
└─ Status code: N/A (just connection)

Database/Cache (NLB):
├─ Protocol: TCP
├─ Path: N/A
├─ Port: Database port
├─ Timeout: 5-10 seconds
├─ Interval: 30 seconds
├─ Healthy threshold: 3
├─ Unhealthy threshold: 3
└─ Status code: N/A
```

---

## Part 9: Summary

### Complete Health Check Checklist

```
Pre-Deployment Checklist:

Application Preparation:
☐ Health endpoint: Created and tested
☐ Response time: Measured (<1 second ideal)
☐ Status code: Returns 200 OK
☐ Payload: Small and lightweight
☐ No dependencies: Just checks if running

Target Group Configuration:
☐ Protocol: Selected (HTTP/HTTPS/TCP/UDP)
☐ Port: Correct for application
☐ Path: Valid endpoint on application
☐ Timeout: Set appropriately (2-10 seconds)
☐ Interval: Chosen for your workload
☐ Healthy threshold: Set for startup speed needed
☐ Unhealthy threshold: Set for sensitivity needed
☐ Status codes: Match application returns

Infrastructure Preparation:
☐ Security group: Inbound rule for health check port
☐ Security group: Source includes LB SG
☐ Application: Running and listening on port
☐ Application: Endpoint responding correctly

Testing Before Production:
☐ Manual test: curl from LB or client
☐ Verify: 200 OK response received
☐ Verify: Response time < timeout
☐ Verify: Targets become healthy
☐ Verify: Targets become unhealthy after stop
☐ Monitor: CloudWatch during test
☐ Calculate: Time to healthy (should match formula)
☐ Calculate: Time to unhealthy (should match formula)

Monitoring Setup:
☐ CloudWatch: HealthyHostCount alarm
☐ CloudWatch: UnHealthyHostCount alarm
☐ Logs: Reviewed and understood
☐ Alerts: Configured for anomalies
☐ Dashboard: Created for visibility

Production Launch:
☐ All checks: Passed in staging
☐ Configuration: Documented
☐ Runbooks: Written for troubleshooting
☐ Team: Trained on health check config
☐ Escalation: Contacts identified
☐ Monitor: First 24 hours closely

Post-Launch:
☐ Verify: All targets healthy
☐ Monitor: No unexpected failures
☐ Review: Logs for any issues
☐ Adjust: If needed based on data
☐ Document: Final configuration

60-Day Review:
☐ Check: Health check metrics over time
☐ Analyze: Any patterns or issues
☐ Optimize: Thresholds/intervals if needed
☐ Update: Documentation with learnings
☐ Plan: Next review or improvements
```

Health checks are now properly configured and monitored for production deployment!
