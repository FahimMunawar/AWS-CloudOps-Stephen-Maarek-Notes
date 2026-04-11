# Target Group Attributes and Configuration

Complete configuration reference for target group attributes including deregistration delay, slow start, routing algorithms, and stickiness settings, essential for optimizing request distribution and managing instance lifecycle.

---

## Part 1: Target Group Attributes Overview

### All Configurable Attributes

**Target Group Settings**

```
Target Group Attributes:

1. Deregistration Delay:
   ├─ Type: Timeout in seconds
   ├─ Purpose: Graceful instance shutdown
   ├─ Range: 1-3600 seconds
   ├─ Default: 300 seconds (5 minutes)
   ├─ Terminology: 
   │  ├─ CLB: Called "Connection Draining"
   │  └─ ALB/NLB: Called "Deregistration Delay"
   └─ Impact: Allows in-flight requests to complete

2. Slow Start:
   ├─ Type: Duration in seconds
   ├─ Purpose: Gradual traffic ramp-up
   ├─ Range: 0-900 seconds
   ├─ Default: 0 (disabled)
   ├─ When enabled: Gradually increase traffic to new target
   └─ Effect: Prevents overwhelming fresh instances

3. Routing Algorithm:
   ├─ Options Available:
   │  ├─ Round Robin (default)
   │  ├─ Least Outstanding Request
   │  └─ Flow Hash (NLB only)
   ├─ ALB supports: Round Robin, Least Outstanding Request
   ├─ CLB supports: Round Robin, Least Outstanding Request
   ├─ NLB supports: Flow Hash (for TCP/UDP)
   └─ Impact: How requests distributed among targets

4. Stickiness:
   ├─ Enable/Disable: Boolean
   ├─ Types:
   │  ├─ Application-based (cookie)
   │  ├─ Duration-based (cookie)
   │  └─ Flow-based (NLB)
   ├─ Cookie option: Custom name or ALB-generated
   ├─ Duration: Seconds cookie remains valid
   └─ Impact: Session persistence across requests

5. Cross-Zone Load Balancing:
   ├─ Enable/Disable: Boolean
   ├─ Purpose: Route across availability zones
   ├─ Supported: All load balancer types
   └─ Cost: Additional data transfer charges

6. Preserve Client IP:
   ├─ Type: Boolean
   ├─ Meaning: Pass client IP to target
   ├─ Supported: ALB/NLB (not CLB)
   ├─ How: Via X-Forwarded-For header or direct
   └─ Important: For security group rules, logging

7. Connection Settings (NLB specific):
   ├─ TCP Keep-Alive: Enable/disable
   ├─ Timeout: Seconds for idle connection
   └─ Purpose: Long-lived connection management

8. Attributes for Protocol-Specific:
   ├─ HTTP Support: For HTTP/HTTPS listeners
   ├─ SSL: For HTTPS listeners
   └─ Options: Depend on protocol

Configuration Scope:

Where Set:
├─ ALB: Target group level
├─ NLB: Target group level
├─ CLB: Load balancer level (legacy)
└─ Application: AWS Console or CLI

Exam Important:
├─ Know: What each attribute does
├─ Know: Valid ranges
├─ Know: Default values
├─ Know: Which LB types support which
└─ Know: How they affect behavior
```

---

## Part 2: Deregistration Delay Revisited

### Complete Timeout Management

**Graceful Instance Shutdown**

```
Deregistration Delay (Formerly Connection Draining):

Definition:
├─ Type: Timeout duration
├─ Unit: Seconds
├─ Range: 1-3600 seconds
├─ Default: 300 seconds (5 minutes)
├─ Can disable: Set to 0 for immediate termination

Purpose:

Scenario Without Deregistration Delay:
├─ Instance marked for deregistration
├─ LB: Immediately stops sending new requests
├─ Existing connections: Closed abruptly
├─ User experience: Requests fail mid-processing
├─ Result: Data loss, error messages
└─ Problem: Not graceful

Scenario With Deregistration Delay:
├─ Instance marked for deregistration
├─ LB: Stops sending NEW requests
├─ Existing connections: Allowed to complete
├─ Time window: Timeout value (default 300s)
├─ After timeout: Force close remaining
├─ Result: In-flight requests complete
└─ Benefit: Graceful, no request loss

Behavior Timeline:

T=0s: Deregister initiated
├─ Terminal state: "Draining"
├─ New requests: Routed elsewhere
└─ Existing: Still being handled

T=0-300s: Graceful phase
├─ Instance: Processing existing requests
├─ LB: Waiting for requests to complete
├─ New traffic: Not sent to instance
└─ Monitor: TargetInUseConnectionCount

T=300s: Timeout reached
├─ Remaining connections: Force-closed
├─ Instance: Removed from rotation
├─ Status: "Deregistered"
└─ Any in-flight: Lost

Configuration:

Short Timeout (30 seconds):
├─ Best for: Web applications (HTTP requests)
├─ Typical: GET requests < 1 second
├─ Setup: Response time very quick
├─ Risk: Long requests may fail
└─ Use case: Standard web apps

Medium Timeout (300 seconds - default):
├─ Best for: Balanced workloads
├─ Typical: Mix of request types
├─ Description: Works for most cases
├─ Coverage: 99% of normal requests
└─ Use case: General purpose

Long Timeout (900+ seconds):
├─ Best for: Long-running requests
├─ Typical: File uploads, batch jobs
├─ Example: Video file uploads (5+ minutes)
├─ Risk: Extended downtime during updates
└─ Use case: Data processing applications

Formula for Sizing:

Calculate Timeout:
├─ Measurement: p95 request duration
├─ Formula: timeout = (p95_duration × 1.5) to (p95_duration × 2)
├─ Example: p95 = 5 seconds → timeout = 7.5-10 seconds
├─ Conservative: timeout = max_observed_duration × 2
└─ Safe default: 300 seconds (works for most)

Scenario Examples:

Web Application:
├─ Type: REST API
├─ Request time: Mostly < 100ms
├─ Recommended: 30-60 seconds
├─ Why: Quick requests, no long operations

E-commerce Site:
├─ Type: Product search, checkout
├─ Request time: 100-500ms typically
├─ Recommended: 60-120 seconds
├─ Why: Most requests fast, some slower

Video Processing:
├─ Type: Upload, transcode, download
├─ Request time: 1-10+ minutes
├─ Recommended: 300-900 seconds
├─ Why: Long-running operations common

Database Application:
├─ Type: Analytics, reporting
├─ Request time: Variable (seconds to minutes)
├─ Recommended: 300 seconds default, or higher
├─ Why: Query complexity varies

Monitoring During Draining:

Metric: TargetInUseConnectionCount
├─ Shows: Open connections to target
├─ Tracks: While draining
├─ Expected: Declining over time
└─ When 0: Instance can be removed

CloudWatch Metrics:
├─ Track: During deregistration process
├─ Monitor: Connection counts declining
├─ Alert: If stuck (connections not closing)
└─ Action: Investigate if timeout exceeded

Best Practices:

1. Set appropriately:
   ├─ Know your application
   ├─ Measure: Actual request durations
   └─ Set: Based on measurements + buffer

2. Monitor carefully:
   ├─ Watch: Connection counts during drain
   ├─ Alert: If timeout exceeded
   └─ Investigate: Why connections not closing

3. Test before production:
   ├─ Deregister during low traffic
   ├─ Monitor: Actual behavior
   ├─ Verify: No request failures
   └─ Adjust: If needed

4. Document timeout:
   ├─ Reason: Why that value chosen
   ├─ History: When/how often triggered
   ├─ Adjustment: Process for changing
   └─ Team: Everyone aware
```

---

## Part 3: Slow Start Feature

### Gradual Traffic Ramp-Up

**New Instance Warm-Up**

```
Slow Start Overview:

Purpose:
├─ Problem: New instance immediately overwhelmed
├─ Solution: Gradually increase traffic
├─ Implementation: Linear ramp over time
└─ Benefit: Prevents overload, better stability

How Slow Start Works:

Without Slow Start:

Timeline:
├─ T=0: Instance passes health check
├─ T=0.1: Instance gets FULL share of traffic
├─ T=0.5: Instance receiving 100% of traffic
├─ Result: Potential overload
└─ Issues: Errors, slowness, cascading failures

With Slow Start (30 second example):

Timeline:
├─ T=0s: Instance passes health check
│ └─ Traffic: 0 requests
│
├─ T=5s: Quarter duration elapsed
│ └─ Traffic: ~25% of normal share
│
├─ T=10s: Half duration elapsed
│ └─ Traffic: ~50% of normal share
│
├─ T=15s: 3/4 duration elapsed
│ └─ Traffic: ~75% of normal share
│
├─ T=30s: Full duration elapsed
│ └─ Traffic: 100% of normal share (FULL)
│
└─ Result: Smooth ramp, no overload

Real-World Scenario:

Setup:
├─ Target group: 4 healthy instances
├─ Traffic: 1000 requests/sec
├─ Per instance: 250 requests/sec normally
└─ New instance: Just became healthy

Without Slow Start:
├─ T=0: New instance added
├─ Requests immediately: 250/sec
├─ Instance state: CPU spikes, GC pauses
├─ Effect: Response time increases
├─ Other requests: May timeout
└─ Cascading: Other instances slow too

With Slow Start (60 seconds):
├─ T=0: New instance added
├─ Requests: 0/sec (ramp begins)
├─ T=10s: ~40 requests/sec
├─ T=20s: ~85 requests/sec
├─ T=30s: ~145 requests/sec
├─ T=40s: ~190 requests/sec
├─ T=50s: ~220 requests/sec
├─ T=60s: Full 250/sec
├─ Effect: Smooth load increase
└─ Result: Better performance

Configuration:

Duration Options:
├─ Disabled: 0 seconds (default)
├─ Minimal: 30 seconds
├─ Standard: 60 seconds
├─ Generous: 120 seconds
├─ Maximum: 900 seconds
└─ Custom: Any value between

Choosing Duration:

Short Duration (30s):
├─ Use for: Fast-starting applications
├─ Example: Stateless services
├─ Warmup needs: Minimal
├─ Risk: Still some overload potential

Medium Duration (60-120s):
├─ Use for: Most applications
├─ Example: Web apps, APIs
├─ Warmup needs: Moderate
├─ Sweet spot: Good balance

Long Duration (300-900s):
├─ Use for: Complex applications
├─ Example: Database services, ML workloads
├─ Warmup needs: Significant
├─ Reason: Cache priming, initialization

When Slow Start Exits:

Slow start ends when:
1. Duration expires: Time runs out
2. Target becomes unhealthy: Health check fails
3. Target is deregistered: Manual removal
4. Load balancer scales down: Capacity reduced

Effect When Target Unhealthy During Slow Start:
├─ Immediately: Exit slow start mode
├─ Status: Transition to "Unhealthy"
├─ Traffic: Routed away
└─ Recovery: Requires manual re-registration

Practical Example:

Application Setup:
├─ Language: Java application
├─ Startup: 30 seconds to startup
├─ Warmup: 30 seconds for JVM warmup
├─ Cache: 30 seconds to warm caches
├─ Total: ~90 seconds before ready

Without Slow Start:
├─ T=0: Instance starts
├─ T=30s: Health check passes
├─ T=30.1s: IMMEDIATELY receives 250 req/sec
├─ Issue: JVM still warming up
├─ Result: High latency, potential errors
└─ Duration: 60+ seconds of bad performance

With Slow Start (120 seconds):
├─ T=0: Instance starts
├─ T=30s: Health check passes
├─ T=30-150s: Traffic gradually increases
├─ Effect: Matches application warmup timing
├─ Result: By 150s, app fully warmed, receiving full traffic
└─ Outcome: Smooth, high-quality requests

Monitoring Slow Start:

CloudWatch Metrics:
├─ TargetResponseTime: Watch for improvement
├─ RequestCount: Verify gradual increase
├─ HTTPCode_Target_5XX: Should stay low
└─ Latency: Should be highest initially, improving

What To Look For:
├─ Response improving: Over slow start duration
├─ No sudden spikes: In errors or latency
├─ Smooth ramp: Not jarring transitions
└─ Stabilization: After slow start completes

Consideration with Routing Algorithm:

Round Robin:
├─ Effect: Requests cycle through instances
├─ Slow start: Still applies to new instance
├─ Sequence: Requests to new instance gradual
└─ Behavior: Works well with slow start

Least Outstanding Request:
├─ Effect: New instance has zero pending
├─ Slow start: Gradual increase still applies
├─ Sequence: More requests to new instance initially
├─ Benefit: Better balancing with slow start
└─ Behavior: Very good combination

Best Practices:

1. Enable for all applications:
   ├─ Safest: Always enable
   ├─ Cost: Minimal to none
   ├─ Risk: Zero downside
   └─ Recommendation: Enable by default

2. Size duration appropriately:
   ├─ Measure: Application warmup time
   ├─ Add buffer: +20-30%
   ├─ Set conservatively: Better to err long
   └─ Example: 60-120s for most apps

3. Monitor first deployment:
   ├─ Watch: Application metrics
   ├─ Verify: Behavior matches expectations
   ├─ Adjust: If issues observed
   └─ Document: Final settings

4. Combine with routing:
   ├─ Consider: Least Outstanding Request
   ├─ Better distribution: With slow start
   ├─ Result: Smoother ramps
   └─ Recommendation: Use together

5. Test scenarios:
   ├─ Scale out: Add new instance under load
   ├─ Verify: No cascading failures
   ├─ Monitor: Response time changes
   └─ Adjust: If needed
```

---

## Part 4: Routing Algorithms Overview

### Request Distribution Strategies

**Load Distribution Methods**

```
Three Main Routing Algorithms:

Algorithm 1: Round Robin
├─ Availability: ALB, CLB
├─ Transport: HTTP/HTTPS
├─ Method: Cycle through targets sequentially
├─ Default: Yes, this is the default
└─ Behavior: 1→2→3→1→2→3...

Algorithm 2: Least Outstanding Request
├─ Availability: ALB, CLB
├─ Transport: HTTP/HTTPS
├─ Method: Send to least busy target
├─ Default: No (must enable)
└─ Behavior: Route to target with fewest pending

Algorithm 3: Flow Hash
├─ Availability: NLB only
├─ Transport: TCP, UDP
├─ Method: Hash protocol/IPs/ports/sequence
├─ Default: No (NLB specific)
└─ Behavior: Single connection → single target

Comparison Table:

╔════════════════════╦═══════════════════════════════╦════════════════════════════╗
║ Characteristic     ║ Round Robin                   ║ Least Outstanding Request  ║
╠════════════════════╬═══════════════════════════════╬════════════════════════════╣
║ Algorithm          ║ Cycle through targets         ║ Route to least busy        ║
║ Best for           ║ Similar response times        ║ Variable response times    ║
║ Distribution       ║ Even splits                   ║ Load balancing             ║
║ Complexity         ║ Very simple                   ║ More sophisticated         ║
║ Latency            ║ Predictable                   ║ Optimized                  ║
║ Fairness           ║ Round-robin fair              ║ Performance fair           ║
║ CPU overhead       ║ Very low                      ║ Slightly higher            ║
║ When to use        ║ Static requests               ║ Dynamic workloads          ║
╚════════════════════╩═══════════════════════════════╩════════════════════════════╝

When to Choose Each:

Choose Round Robin When:
├─ Consistent: Request processing times
├─ Simple: Stateless services
├─ Example: Read-only APIs
├─ Reason: Simple, predictable
└─ Test: All targets equal capacity

Choose Least Outstanding Request When:
├─ Variable: Request processing times
├─ Complex: Some requests slow, some fast
├─ Example: Video encoding, search
├─ Reason: Better performance
└─ Test: 10-40% response time variation

Choose Flow Hash (NLB) When:
├─ Long-lived: Connection-based
├─ Stateful: Protocols like gRPC
├─ Example: Messaging, gaming
├─ Reason: Connection affinity built-in
└─ Test: Persistent connection model
```

---

## Part 5: Least Outstanding Request Algorithm

### Intelligent Load Balancing

**Route to Least Busy Target**

```
Least Outstanding Request Definition:

Core Concept:
├─ Request: Incoming at load balancer
├─ Check: Each target's queue
├─ Count: Outstanding (pending/unfinished) requests
├─ Select: Target with LOWEST count
├─ Rationale: Least busy has most capacity
└─ Result: More balanced load distribution

Outstanding Request:

Definition:
├─ Request: Received by target
├─ Status: Waiting for processing or processing
├─ Duration: From arrival to response sent
├─ Counted: While active
└─ Not counted: After response

Example Timeline:

Request 1:
├─ T=0ms: Arrives at target
├─ T=0-100ms: Outstanding
├─ T=100ms: Response sent
└─ After: Not counted

Request 2:
├─ T=10ms: Arrives at target
├─ T=10-50ms: Outstanding
├─ T=50ms: Response sent
└─ After: Not counted

Scenario Example:

Setup:
├─ Targets: 3 instances
├─ Response time: Variable (50-200ms)
├─ Requests/sec: 100 total
└─ Distribution desired: Intelligent

With Round Robin:

Sequence of requests:
├─ Request 1 → Target A
├─ Request 2 → Target B
├─ Request 3 → Target C
├─ Request 4 → Target A
├─ Request 5 → Target B
├─ Request 6 → Target C
└─ Pattern: 1:1:1 regardless of load

Instance State (at same time):
├─ Target A: 50 outstanding requests (50ms each = 2.5s total)
├─ Target B: 8 outstanding requests (200ms each = 1.6s total)
├─ Target C: 42 outstanding requests (50ms each = 2.1s total)
└─ Result: Unfair distribution (A and C overwhelmed)

With Least Outstanding Request:

Scenario after 100 requests:
├─ Target A outstanding: 3 requests
├─ Target B outstanding: 2 requests
├─ Target C outstanding: 4 requests
└─ Next request goes to: Target B (lowest)

Distribution over time:
├─ Requests follow: Least busy pattern
├─ Result: More balance
├─ A gets: Fewer requests (already busy)
├─ B gets: More requests (fastest)
├─ C gets: Medium requests
└─ Effect: Better utilization

Advantages:

Adaptive Behavior:
├─ Automatic: Adapts without configuration
├─ Fair: Performance-based, not round-robin
├─ Dynamic: Responds to changing conditions
├─ Smart: Reduces wait times
└─ Result: Better overall latency

Example with Slow Target:

Scenario:
├─ Request A: 100ms to process
├─ Request B: 50ms to process
├─ Request C: 100ms to process
├─ Request D: 50ms to process

Round Robin (sequential):
├─ T=0-50ms: A processing on Target 1, B on Target 2
├─ T=0-50ms: C processing on Target 3, D on Target 1
├─ Issues: Target 1 still busy when D arrives
└─ Wait: D queues, must wait for A

Least Outstanding Request:
├─ When D arrives: Check outstanding
│  ├─ Target 1: 1 outstanding (A)
│  ├─ Target 2: 1 outstanding (B)
│  ├─ Target 3: 1 outstanding (C)
│  └─ Choose: Any (all equal)
├─ If Target 2 chosen: D routed there
├─ (B finished before D arrives) → D goes to Target 2
└─ Result: More optimal

Real-World Benefits:

Scenario: E-commerce Site
├─ Request type 1: Search (100ms)
├─ Request type 2: View product (50ms)
├─ Request type 3: Checkout (500ms)
├─ Mix: Random of above

Distribution Problem:
├─ If sequential: Some targets busy with checkout
├─ Other targets: Waiting for search/product views
├─ Effect: Some fast targets idle, slow targets overloaded
└─ Result: Inconsistent response times

With Least Outstanding Request:
├─ Fast requests: Go to fast-processing targets
├─ Slow requests: Get their own target
├─ Result: More even distribution of effort
├─ Experience: Consistent response times
└─ Satisfaction: Better user outcomes

When Outstanding Request Shines:

Microservices:
├─ Services: Variable processing time
├─ Distribution: Least Outstanding better
├─ Reason: Responsive to current load
└─ Result: Better overall throughput

Batch Processing:
├─ Jobs: Different lengths (1s to 10s)
├─ Distribution: Round Robin causes queueing
├─ With LOR: Better utilization
└─ Result: Higher efficiency

Media Encoding:
├─ Files: Different sizes, different encode times
├─ Distribution: Least Outstanding optimal
├─ Effect: Pipeline efficiently used
└─ Result: Higher throughput

API Gateway:
├─ Requests: Mix of simple/complex
├─ Distribution: Least Outstanding better
├─ Impact: More consistent latency
└─ Result: Better API response

Implementing Least Outstanding Request:

AWS Console:
1. Go to target group
2. Edit: Attributes
3. Routing algorithm: Select "Least Outstanding Request"
4. Save: Changes
5. Effect: Immediate (new requests use new algorithm)

CLI (AWS):
```bash
aws elbv2 modify-load-balancer-attributes \
  --load-balancer-arn <arn> \
  --attributes \
    Key=routing.http.drop_invalid_header_fields.enabled,Value=false \
    Key=routing.http2.enabled,Value=true \
    Key=routing.http.xff_client_port.enabled,Value=false

# For target group routing algorithm (ALB):
# Note: ALB routing algorithm is set at load balancer level, not target group
# CLI: Use modify-load-balancer-attributes for routing algorithm configuration
```

Monitoring Impact:

Metrics to Track:
├─ TargetResponseTime: Should improve
├─ RequestCount: Per target (should balance)
├─ ActiveConnectionCount: More even
├─ HTTPCode_5XX: Might decrease
└─ Overall latency: Usually improves

Comparison Query:

Before switching:
├─ Baseline: Capture metrics for 1 hour
├─ p99 latency: Record baseline
├─ Server errors: Baseline count

After switching:
├─ Compare: Same metrics
├─ p99 latency: Should decrease/improve
├─ Server errors: Should decrease
└─ Assessment: Is improvement worth CPU cost?

Considerations:

CPU Overhead:
├─ Amount: Slightly higher
├─ Reason: Must track outstanding requests
├─ Impact: <1% additional CPU usually
├─ Worth it: Yes, for most workloads
└─ Trade-off: Better performance vs minimal CPU

Predictability:
├─ Round Robin: Completely predictable
├─ Least Outstanding: Less predictable
├─ Issue: Testing harder with randomness
├─ Solution: Test with load testing tools
└─ Practice: Simulate variable request times

Best Practices:

1. Test before production:
   ├─ Load test: With both algorithms
   ├─ Compare: Results side-by-side
   ├─ Decision: Based on data
   └─ Document: Reasoning

2. Monitor after change:
   ├─ Track: Key performance metrics
   ├─ Compare: To previous baseline
   ├─ Alert: If degradation
   └─ Rollback: If needed (within minutes)

3. Combine with Slow Start:
   ├─ New instance: Gets fewer requests initially
   ├─ Slow Start: Ramps up gradually
   ├─ Least Outstanding: Ensures smooth ramp
   └─ Result: Perfect combination

4. Not silver bullet:
   ├─ Better than round-robin: Usually yes
   ├─ Solves all problems: No
   ├─ Still need: Proper sizing, monitoring
   └─ Part of: Complete strategy
```

---

## Part 6: Round Robin Algorithm

### Sequential Request Distribution

**Default Load Distribution**

```
Round Robin Definition:

Core Concept:
├─ Algorithm: Cycle through targets sequentially
├─ Pattern: 1 → 2 → 3 → 1 → 2 → 3 → ...
├─ Selection: Next in sequence, regardless of load
├─ Default: Yes, for ALB/CLB
└─ Predictable: Always same pattern

How Round Robin Works:

Setup:
├─ Targets: Target1, Target2, Target3
├─ Queue: Round robin position
└─ Initial: Point to Target1

Request Sequence:

Request 1:
├─ Current position: Target1
├─ Destination: Target1
└─ Next position: Target2

Request 2:
├─ Current position: Target2
├─ Destination: Target2
└─ Next position: Target3

Request 3:
├─ Current position: Target3
├─ Destination: Target3
└─ Next position: Target1

Request 4:
├─ Current position: Target1
├─ Destination: Target1
└─ Next position: Target2

Pattern Continues:
├─ Requests cycle: Through targets
├─ Order: Predictable and consistent
└─ Result: Even distribution

Advantages:

Simplicity:
├─ Algorithm: Very simple
├─ Implementation: Minimal CPU
├─ Predictable: Consistent pattern
├─ CPU cost: Essentially free
└─ Reason: Default algorithm

Predictability:
├─ Testing: Can predict exact flow
├─ Debugging: Easy to trace
├─ Documentation: Clear behavior
└─ Reason: Deterministic pattern

Even Distribution:
├─ Assumption: Similar request times
├─ Result: Even load split
├─ Example: 3 targets, 300 req/sec = 100/sec each
└─ Fairness: Mathematically equal

Low Overhead:
├─ Tracking: Just increment counter
├─ Memory: One integer
├─ CPU: Negligible
└─ Efficiency: Highest possible

When Round Robin Works Best:

Similar Response Times:
├─ All requests: Take similar time
├─ Targets: Similar capacity
├─ Example: Simple GET requests
├─ Result: Works perfectly

Stateless Services:
├─ Requests: Independent
├─ State: Not stored in target
├─ Example: Read-only APIs
└─ Benefit: Simple distribution works

Read-Only APIs:
├─ Cache hits: Always same time
├─ Queries: Pre-calculated
├─ Example: Product catalog API
└─ Distribution: Even works well

Static Content:
├─ Response: Always same size
├─ Time: Always similar
├─ Example: Image serving
└─ Distribution: Perfect candidate

Disadvantages:

No Load Awareness:
├─ Problem: Ignores actual load
├─ Scenario: One target processing slow request
├─ Result: Next request queues on same target
├─ Effect: Uneven experienced latency
└─ Issue: Not optimal

Variable Processing Times:
├─ Problem: Doesn't adapt to request type
├─ Scenario: Mix of 10ms and 1000ms requests
├─ Distribution: Even split causes issues
├─ Effect: Some targets overloaded, others idle
└─ Result: Higher p99 latency

Potential Imbalance:
├─ Under load: Can create queueing
├─ Fast targets: May be idle
├─ Slow targets: May have queue
├─ Perception: Unfair (though mathematically fair)
└─ Experience: Variable response times

Example Problem Scenario:

Setup:
├─ Targets: 3 web servers
├─ Request type A: 50ms (database query)
├─ Request type B: 10ms (cache hit)
├─ Mix: Alternating A, B, A, B, ...

Timeline with Round Robin:

T=0ms:
├─ Request 1 (A, 50ms) → Target 1
├─ Outstanding: Target1 has 1 (50ms queue)

T=10ms:
├─ Request 2 (B, 10ms) → Target 2
├─ Outstanding: Target1 still has 1, Target2 has 1

T=20ms:
├─ Request 3 (A, 50ms) → Target 3
├─ Outstanding: All have 1

T=30ms:
├─ Request 2 finishes on Target 2
├─ Request 4 (B, 10ms) → Target 1
├─ Outstanding: Target1 now has 2, Target2 has 0, Target3 has 1

T=40ms:
├─ Request 3 finishes on Target 3
├─ Request 5 (A, 50ms) → Target 2
├─ Outstanding: Target1 has 2, Target2 has 1, Target3 has 0

Result:
├─ Target 1: Overloaded (2 requests)
├─ Target 2: Medium (1 request)
├─ Target 3: Empty
├─ Problem: Not optimal distribution
└─ Effect: Users on Target 1 see higher latency

With Least Outstanding Request:

Same scenario:
├─ Request 4 would NOT go to Target 1
├─ Instead: Goes to Target 2 (0 outstanding)
├─ Request 5 would NOT go to Target 2
├─ Instead: Goes to Target 3 (0 outstanding)
└─ Result: Better balancing

Implementation:

Round Robin is Default:
├─ Action: Nothing required
├─ Default: Automatically used
├─ Config: No configuration needed
└─ Status: Just works

To Confirm Round Robin:

AWS Console:
1. Go to: Target group
2. Look: Attributes
3. Routing algorithm: Should say "Round Robin"
4. Or: May not show (defaults to Round Robin)

CLI Check:
```bash
aws elbv2 describe-target-groups \
  --target-group-arn <arn>
# Look for: routing HTTP settings
```

Switching Away (To Least Outstanding):

AWS Console:
1. Target group
2. Edit attributes
3. Routing algorithm: Change to "Least Outstanding Request"
4. Save

Monitoring Round Robin:

Metrics to Track:
├─ RequestCountPerTarget: Should be equal
├─ TargetResponseTime: May vary by target
├─ Latency perception: Depends on request mix
└─ Errors: Shouldn't correlate with algorithm

Typical Distribution:

With Round Robin:
├─ Target 1: 33.3% of requests
├─ Target 2: 33.3% of requests
├─ Target 3: 33.3% of requests
├─ Variance: Near zero
└─ Fairness: Perfect mathematical fairness

Comparison with Other Algorithms:

Round Robin vs Least Outstanding Request:

Round Robin:
├─ + Simple, efficient, predictable
├─ + Works well for similar-duration requests
├─ - Poor with variable request times
├─ - Creates queuing with mixed load

Least Outstanding Request:
├─ + Adapts to request load
├─ + Better for variable workloads
├─ - More CPU overhead
├─ - Less predictable behavior

Choice Decision:

Use Round Robin If:
├─ Requests: Similar processing time
├─ Targets: Equal capacity
├─ Workload: Predictable
├─ Simple: Desired over sophisticated
└─ Example: Standard web farm

Use Least Outstanding If:
├─ Requests: Variable processing time
├─ Targets: Different capacity
├─ Workload: Unpredictable mix
├─ Performance: Critical concern
└─ Example: Complex microservices

Best Practices:

1. Understand your workload:
   ├─ Measure: Request processing times
   ├─ Analyze: Distribution of times
   ├─ Decide: Based on data

2. Test both:
   ├─ Baseline: Current (Round Robin)
   ├─ Compare: With Least Outstanding
   ├─ Measure: Impact on latency

3. Monitor distribution:
   ├─ Verify: Equal split (should see 1/N per target)
   ├─ Alert: If imbalanced
   ├─ Investigate: Why uneven

4. Document rationale:
   ├─ Record: Why Round Robin chosen
   ├─ Decision: Based on what?
   ├─ Revisit: Periodically (quarterly)
   └─ Change: If workload changes
```

---

## Part 7: Flow Hash Algorithm (NLB Specific)

### Connection-Based Routing

**Long-Lived Connection Management**

```
Flow Hash Overview:

Definition:
├─ Algorithm: Hash-based routing
├─ Scope: NLB only (Network Load Balancer)
├─ Transport: TCP and UDP protocols
├─ Basis: Protocol, IPs, ports, TCP sequence
├─ Duration: Entire connection lifetime
└─ Effect: Connection affinity (same target always)

When Available:

Load Balancer Types:
├─ ALB: NOT available
├─ CLB: NOT available
├─ NLB: YES, this is NLB specific
└─ GW LB: NOT applicable

Protocol Support:
├─ TCP: YES, full support
├─ UDP: YES, full support
├─ HTTP/HTTPS: NOT applicable (ALB feature)
└─ Used for: Layer 4 (transport layer) workloads

Flow Hash Calculation:

Hash Inputs:

Components Hashed:
├─ Protocol: TCP or UDP
├─ Source IP: Client IP address
├─ Source Port: Client port
├─ Destination IP: Backend IP
├─ Destination Port: Backend port
├─ TCP Sequence Number: TCP flow identifier

Hash Algorithm:
├─ Input: All above components combined
├─ Processing: Cryptographic hash function
├─ Output: Hash value
└─ Selection: Hash % (number of targets) = target index

Result:
├─ Calculation: Consistent
├─ Deterministic: Same input = same output
├─ Connection: Routed to same target
└─ Duration: For entire connection lifetime

Example:

Connection Details:
├─ Client: 203.0.113.100:52341
├─ Server: 10.0.1.50:443 (backend)
├─ Protocol: TCP
├─ Sequence: 1234567890

Hash Process:
├─ Input: TCP + 203.0.113.100 + 52341 + 10.0.1.50 + 443 + sequence
├─ Hash function: Apply algorithm
├─ Result: 42739521
├─ Targets available: 3
├─ Calculation: 42739521 % 3 = 2
└─ Destination: Target 3 (index 2)

Guarantee:
├─ Same connection: Always Target 3
├─ Until: Connection closes
├─ Other clients: May get different targets
└─ Result: Consistent routing

How Flow Hash Works in Practice:

Scenario: gRPC Service

Setup:
├─ Protocol: gRPC over HTTP/2
├─ Connections: Long-lived bidirectional streaming
├─ Transport: TCP
├─ Targets: 3 backend servers

Connection 1:
├─ Client: 203.0.113.100:52341
├─ Hash: → Target A
├─ Duration: Open for 10 minutes
├─ Routing: ALL requests on this connection → Target A
└─ Flow: Multiplexed over same connection

Connection 2:
├─ Client: 203.0.113.101:52342
├─ Hash: → Target B
├─ Duration: Open for 5 minutes
├─ Routing: ALL requests on this connection → Target B
└─ Flow: Multiplexed over same connection

Connection 3:
├─ Client: 203.0.113.102:52343
├─ Hash: → Target A
├─ Duration: Open for 8 minutes
├─ Routing: ALL requests on this connection → Target A
└─ Flow: Multiplexed over same connection

Result:
├─ Target A: 2 active connections
├─ Target B: 1 active connection
├─ Target C: 0 active connections
├─ Future: New connections hash independently
└─ Rebalancing: Only on new connections

Why Connection Affinity Matters:

Stateful Services:

Example: Session-based Message Queue
├─ Connection: Client connects
├─ State: Queue position, offsets, cursors
├─ Storage: In memory on Target A
├─ Need: Same target for whole session
└─ Benefit: No state sync needed

Without Connection Affinity:
├─ Connection 1: Data sent to Target A
├─ Connection 2: Data sent to Target B
├─ Problem: Target B doesn't have connection state
├─ Error: Offset not available, re-sync needed
└─ Impact: Higher latency, complexity

With Flow Hash:
├─ Connection: Always routed to same target
├─ State: Never lost
├─ Performance: Optimal
└─ Sync: No need for distributed state

Examples of Flow Hash Usage:

Use Case 1: WebSocket Service
├─ Protocol: TCP/WebSocket
├─ Connections: Long-lived bidirectional
├─ State: User session data
├─ Benefit: Same target → same state
└─ Effect: Seamless messaging

Use Case 2: gRPC Services
├─ Protocol: TCP/HTTP2
├─ Connections: Streaming, long-lived
├─ State: Channel state, multiplexed requests
├─ Benefit: All requests to same backend
└─ Effect: Efficient multiplexing

Use Case 3: Game Servers
├─ Protocol: TCP/UDP
├─ Connections: Player sessions
├─ State: Game world, player position
├─ Benefit: Same server entire game
└─ Effect: Consistent experience

Use Case 4: SSH/Telnet
├─ Protocol: TCP
├─ Connections: Interactive session
├─ State: Session data
├─ Benefit: Same server for session duration
└─ Effect: Seamless interactive use

Advantages:

Built-In Affinity:
├─ No configuration: Automatic
├─ No cookies: No tracking needed
├─ Transparent: Clients unaware
├─ Efficient: No extra metadata
└─ Benefit: Simple and clean

Perfect Affinity:
├─ Guarantee: Same connection = same target
├─ Durability: Until connection ends
├─ Reliability: No exceptions
└─ Consistency: Deterministic

Connection Distribution:
├─ Per-connection: Not per-request
├─ Load: Spreads to multiple connections
├─ Flexibility: New connections can go anywhere
└─ Rebalancing: Automatic with new connections

No Overhead:
├─ State: Minimal tracking
├─ CPU: Very efficient
├─ Memory: Minimal impact
└─ Latency: No additional delay

Disadvantages:

Imbalanced Load:

Scenario:
├─ Client A: 10-second connection (light traffic)
├─ Client B: 600-second connection (heavy traffic)
├─ Hash: Both go to different targets
├─ Load: Heavy client loads one target heavily
└─ Effect: Possible imbalance

Example:
├─ Connection 1 (heavy): 1000 req/sec → Target A
├─ Connection 2 (light): 10 req/sec → Target B
├─ Connection 3 (heavy): 1000 req/sec → Target A
├─ Result: Target A getting much more load
└─ Imbalance: Due to hash distribution of heavy connections

Long Connection Sessions:

Problem:
├─ Client: Connects once
├─ Duration: Days/weeks open
├─ Target: Always same
├─ Issue: Can't scale away that connection
└─ Result: Fixed routing, inflexible

Testing Complexity:

Unpredictability:
├─ Behavior: Depends on client source IP/port
├─ Production: Different from test environment
├─ Debugging: Hard to reproduce specific routing
└─ Testing: Can't control hash output easily

Implementing Flow Hash:

NLB Configuration:

Note: Flow hash is DEFAULT for NLB!
├─ TCP/UDP: Automatically uses Flow Hash
├─ Configuration: Usually no action needed
├─ Change: Can only use ALB for per-request routing
└─ Alternative: If need round robin-like behavior

To Verify Flow Hash:
```bash
# Check NLB target group attributes
aws elbv2 describe-target-groups \
  --target-group-arn <nlb-target-group-arn>
# Note: Flow hash for NLB is default, not explicitly listed
# Look for: TargetType, Protocol (TCP/UDP)
```

Cannot Disable:
├─ NLB: Always uses flow hash
├─ Cannot change: To round robin
├─ When needed: Different routing, use ALB
└─ Alternative: If must have per-request routing

Monitoring Flow Hash:

Key Metrics:

ActiveConnectionCount:
├─ What: Number of active connections per target
├─ Track: Should be somewhat balanced
├─ Alert: If heavily skewed
└─ Reason: Indicates hash distribution

ConnectionCount Per Target:
├─ What: Total connections per target
├─ Track: Over time
├─ Pattern: Should grow proportionally
└─ Alert: If one target stalled

NetworkIn/NetworkOut:
├─ What: Data transferred per target
├─ Track: Should be proportional to connections
├─ Alert: If severely imbalanced
└─ Reason: Indicates uneven load

NewFlowCount:
├─ What: New connections per target per interval
├─ Track: Should arrive regularly
├─ Alert: If uneven arrival
└─ Reason: Load testing, client patterns

Healthy vs Unhealthy:
├─ Target becomes unhealthy: Existing connections?
├─ Answer: Keep routing to it (connections sticky)
├─ Policy: Depends on NLB configuration
├─ Action: May route to unhealthy target

Interaction with Health Checks:

Healthy Target Lost:

Scenario:
├─ Target A: Healthy, has connections
├─ Event: Target A fails health check
├─ New connections: Routed to Target B/C
├─ Existing connections: Still flowmitted?
└─ Behavior: Configuration dependent

Behavior Setting:
├─ Default: Drain when unhealthy
├─ Alternative: Immediate close
└─ Configurable: Via target group attributes

Connection Replacement:

When to Start New:
├─ Connection closes: Naturally (client disconnect)
├─ Time-out: Long-lived connections end
├─ Rebalance: Happens automatically
└─ Trigger: Client initiates new connection

Balance Over Time:
├─ Short-lived connections: Balance quickly
├─ Long-lived: May take hours/days
├─ New deployment: Gradual rebalancing
└─ Strategy: Plan for natural churn

Best Practices:

1. Understand connection patterns:
   ├─ Know: Average connection duration
   ├─ Measure: Connection count per target
   ├─ Monitor: For imbalance
   └─ Alert: If skewed

2. Monitor load distribution:
   ├─ Track: Bytes transferred per target
   ├─ Watch: For proportionality
   ├─ Alert: If > 10% deviation
   └─ Investigate: Why imbalanced

3. Plan for long connections:
   ├─ Expect: Some targets may be busier
   ├─ Accept: Smaller imbalance acceptable
   ├─ Monitor: Overall system health
   └─ Act: Scale up if needed

4. Test scalability:
   ├─ Add targets: With active connections
   ├─ Verify: New connections go to new targets
   ├─ Check: Existing connections stay
   └─ Confirm: Gradual rebalancing

5. Connection lifecycle:
   ├─ Short connections: Good for load balance
   ├─ Long connections: Accept some imbalance
   ├─ Mix: Monitor and optimize
   └─ Design: Consider connection duration

Comparison with Session Stickiness:

Flow Hash:
├─ Scope: Entire connection lifetime
├─ Mechanism: Protocol-based hash
├─ Configuration: Not needed (automatic)
├─ Overhead: None
└─ Reliability: Perfect

Session Stickiness (ALB):
├─ Scope: Configured cookie duration
├─ Mechanism: Cookie-based tracking
├─ Configuration: Needed
├─ Overhead: Cookie management
└─ Reliability: Cookie-dependent

Use Cases:

Use Flow Hash For:
├─ Long-lived connections
├─ Protocol-level affinity needed
├─ Stateful TCP/UDP services
├─ gRPC, WebSocket, game servers

Use Session Stickiness For:
├─ HTTP/HTTPS applications
├─ Session-based state
├─ Per-request routing needed
├─ Web application servers
```

---

## Part 8: Stickiness Settings

### Session Persistence Configuration

**Maintaining Session State**

```
Stickiness Overview:

Purpose:
├─ Goal: Route requests from same client to same target
├─ Mechanism: Cookie-based or connection-based
├─ Duration: Configured timeout
└─ Use case: Maintain session state

Types of Stickiness:

1. Application-Based (Cookie)
├─ How: Custom application cookie
├─ ALB: Reads existing app cookie
├─ Example: SESSIONID or similar
├─ Duration: Cookie expiration
└─ Control: Application sets expiry

2. Duration-Based (Cookie)
├─ How: ALB-generated cookie
├─ ALB: Creates/manages cookie
├─ Default: 86400 seconds (1 day)
├─ Expires: After configured duration
└─ Control: Target group setting

3. Flow-Based (NLB)
├─ How: Connection-based routing
├─ NLB: Always active for TCP/UDP
├─ Duration: Connection lifetime
└─ Control: Not configurable

Stickiness States:

Enabled:
├─ Status: Active
├─ Requests: Routed to same target
├─ Cookie: Present or connection routed
└─ Effect: Session persistence

Disabled:
├─ Status: Inactive (default)
├─ Requests: Routed by algorithm (Round Robin/LOR)
├─ Cookie: Not managed by ALB
└─ Effect: No session persistence

Application-Based Stickiness:

How It Works:

Step 1: Client Makes Request
├─ Request: First time from client
├─ ALB: Routes to Target A
├─ Target A: Processes, creates session
└─ Response: Returns with app cookie (SESSIONID=ABC123)

Step 2: Client Receives Response
├─ Browser: Stores cookie
├─ Cookie: SESSIONID=ABC123 from Target A
└─ Expiry: Application defined

Step 3: Client Makes Follow-up Request
├─ Request: Includes SESSIONID=ABC123
├─ ALB: Reads cookie, recognizes session
├─ Routing: Routes to Target A (same as cookie)
└─ Target A: Finds session data, continues

Step 4: Connection Throughout Session
├─ All requests: Include SESSIONID cookie
├─ ALB: Routes to Target A consistently
├─ Session: Maintained continuously
└─ Until: Cookie expires

Configuration:

AWS Console:
1. Go to: Target group → Attributes
2. Stickiness: Enable
3. Type: Application-based
4. Cookie name: Enter app cookie name (e.g., "SESSIONID")
5. Save: Changes
6. Effect: Immediate for new connections

Cookie Name:
├─ Format: Must match app cookie name exactly
├─ Case-sensitive: Important
├─ Common examples:
│  ├─ SESSIONID (Java/JSP)
│  ├─ JSESSIONID (Java standard)
│  ├─ PHPSESSID (PHP)
│  ├─ ASP.NET_SessionId (C# ASP.NET)
│  └─ session_id (custom)

Duration-Based Stickiness:

How It Works:

Step 1: Client Makes Request
├─ Request: First time from client
├─ ALB: Routes to Target A
├─ Response: ALB creates cookie (AWSALB=XYZ789)
└─ Expiry: 1 day by default

Step 2: ALB Cookie Generated
├─ Mechanism: ALB-managed cookie
├─ Format: Encoded with target info
├─ Contains: Which target to route to
├─ Visible: In browser cookie storage

Step 3: Subsequent Requests
├─ Client: Sends AWSALB cookie
├─ ALB: Reads cookie, routes to Target A
├─ Effect: All requests to same target
└─ Until: Cookie expires or deleted

Step 4: After Expiration
├─ Cookie: Expires after duration
├─ Next request: No sticky cookie
├─ Routing: Uses default algorithm
└─ Result: May go to different target

Configuration:

AWS Console:
1. Go to: Target group → Attributes
2. Stickiness: Enable
3. Type: Duration-based
4. Duration: Set seconds (default 86400)
5. Cookie name: Defaults to AWSALB (optional: can customize)
6. Save: Changes

Duration Options:

Common Values:
├─ 60 seconds: Very short sessions
├─ 300 seconds: 5 minutes (short)
├─ 3600 seconds: 1 hour (medium)
├─ 86400 seconds: 24 hours (long, default)
├─ Custom: Any value up to 604800 (1 week)

Choosing Duration:

Session Length:
├─ Short lived (< 5 min): Set 300-600 seconds
├─ Medium (5-60 min): Set 1800-3600 seconds
├─ Long lived (hours): Set 86400 seconds
└─ Matches: Expected user session length

Safety Margin:
├─ Formula: (expected_session_length × 1.2) to 1.5
├─ Example: 30 min session → 2160-2700 seconds
├─ Reason: Buffer for long requests
└─ Compromise: Between stickiness and flexibility

NLB Flow-Based Stickiness:

Automatic for TCP/UDP:
├─ Enabled: By default
├─ Configuration: No settings to change
├─ Duration: Connection lifetime
├─ Mechanism: Hash-based (as discussed)
└─ Persistence: Perfect (connection → target)

Not for HTTP/HTTPS:
├─ Note: NLB can handle HTTP
├─ But: HTTP is per-request, not flow
├─ Behavior: Each HTTP request is separate
└─ Recommendation: Use ALB for HTTP with stickiness

Enabling/Disabling Stickiness:

Completely Disable:

When:
├─ Stateless services: No need
├─ Microservices: Should be stateless
├─ APIs: Usually stateless
└─ Recommendation: Default state

How:
├─ AWS Console: Leave disabled (default)
├─ Effect: Requests routed by algorithm only
└─ Result: Maximum flexibility

Enable Application-Based:

When:
├─ App manages sessions: Via cookies
├─ Need: Route to same server
├─ Requirement: App already has cookie
└─ Advantage: No double-cookie overhead

Enable Duration-Based:

When:
├─ App not session-aware: No app cookie
├─ Need: Session persistence
├─ Requirement: ALB manages sessions
└─ Advantage: Simple, automatic

Practical Scenarios:

Scenario 1: E-Commerce Site

Setup:
├─ Session: Shopping cart
├─ Stored: In-memory on web server
├─ Need: Route to same server
├─ Application: Creates SESSIONID cookie

Config:
├─ Stickiness: Enable
├─ Type: Application-based
├─ Cookie name: SESSIONID
├─ Result: Same server for entire shopping session

Scenario 2: API Gateway

Setup:
├─ Stateless: Each request independent
├─ Need: No persistence
├─ Application: No session concept
├─ Routing: Can go anywhere

Config:
├─ Stickiness: Disable (leave default)
├─ Type: N/A
├─ Result: Best performance, requests routed optimally

Scenario 3: Legacy App with Local Cache

Setup:
├─ Session: User data cached locally
├─ Stored: In-memory dictionary
├─ Need: Same server for consistency
├─ Application: No cookie management

Config:
├─ Stickiness: Enable
├─ Type: Duration-based
├─ Duration: 3600 (1 hour)
├─ Result: Routes to same server for hour

Monitoring Stickiness:

CloudWatch Metrics:

Cookie-based:
├─ Metric: Not directly tracked
├─ Indication: Same target received multiple requests
├─ Verify: CloudWatch Logs analysis
└─ Check: Access logs for cookie patterns

Access Logs:

Enable:
├─ Stored: In S3
├─ Query: With Athena
├─ Find: Stickiness behavior

Query Example:
```sql
SELECT 
  client_ip,
  target_ip,
  COUNT(*) as requests,
  COUNT(DISTINCT target_ip) as unique_targets
FROM alb_logs
WHERE date = '2023-12-15'
GROUP BY client_ip, target_ip
ORDER BY requests DESC;

-- Should show: Same client_ip mostly to single target_ip
```

Session Affinity:
├─ Check: Verify routing consistency
├─ Tool: Load testing with session cookies
├─ Confirm: Same target for session duration
└─ Alert: If different targets in same session

Issues and Troubleshooting:

Stickiness Not Working:

Problem: Requests going to different targets
├─ Reason 1: Stickiness disabled (check settings)
├─ Reason 2: Cookie missing/expired
├─ Reason 3: Wrong cookie name configured
├─ Reason 4: Application not creating cookie

Troubleshooting:
├─ Step 1: Verify stickiness enabled in console
├─ Step 2: Check cookie present in requests
├─ Step 3: Compare cookie name with config
├─ Step 4: Verify app creating cookie
├─ Step 5: Check cookie expiration matching

Cookie Name Mismatch:

Problem: Application uses SESSIONID but config says SESSION
├─ Effect: ALB doesn't recognize cookie
├─ Result: Routes randomly, not sticky
├─ Solution: Fix cookie name in config

Debug:
├─ Browser: Inspect cookies (F12)
├─ Look for: Exact cookie name
├─ Copy: Exact name (case-sensitive)
├─ Update: Config with exact name

Session Timeout vs Cookie Timeout:

Issue: Session expires before cookie
├─ Scenario: Cookie set to 1 hour, session 30 min
├─ Problem: Sticky routing continues, but session gone
├─ Result: User sees logged-out state
├─ Solution: Match cookie duration to session tolerance

Issue: Cookie expires before session
├─ Scenario: Cookie 30 min, session 1 hour
├─ Problem: Cookie expires, route to different server
├─ Result: Session lost (different server)
├─ Solution: Cookie duration ≥ expected session length

Target Replacement Impact:

When Target Removed:

Scenario:
├─ Target A: Has sticky sessions
├─ Action: Deregister Target A
├─ Question: What happens to sessions?

Behavior:
├─ Existing connections: Deregistration delay applies
├─ New connections: Routed elsewhere
├─ Sticky sessions: "Broken" after deregistration
├─ Result: Users routed to new server

Replacement Strategy:

Graceful:
1. Set long deregistration delay
2. Enable connection draining
3. Let existing sessions complete
4. New sessions go to new target
5. Old target eventually removed

Quick Replacement:
1. Short deregistration delay
2. Existing sessions terminate
3. Users reconnect automatically
4. Route to new target
5. Quick complete replacement

Best Practices:

1. Understand your application:
   ├─ Know: If stateful or stateless
   ├─ Check: How application manages state
   └─ Decide: If persistence needed

2. Choose correctly:
   ├─ Stateless: Leave stickiness disabled
   ├─ Stateful with app cookie: Application-based
   ├─ Stateful no app cookie: Duration-based
   └─ Never: Unnecessary stickiness

3. Size appropriately:
   ├─ Duration: Match expected session length
   ├─ Buffer: Add 20-50% margin
   ├─ Balance: Short enough for rebalancing
   └─ Test: Verify timing works

4. Monitor closely:
   ├─ Verify: Stickiness working (via logs)
   ├─ Alert: If behavior changes
   ├─ Check: Cookie header in requests
   └─ Adjust: If issues found

5. Plan for scaling:
   ├─ New targets: Added as capacity needed
   ├─ Existing sessions: Remain on old targets
   ├─ Rebalance: Happens over time as cookies expire
   └─ Expect: Gradual balancing, not immediate

6. Combine with slow start:
   ├─ New target: Gets new sessions
   ├─ Slow start: Gradually increases load
   ├─ Combined: Smooth ramp-up
   └─ Benefit: No overload
```

---

## Part 9: AWS Console Configuration

### Practical Setup Walkthrough

**Hands-On Configuration**

```
Accessing Target Group Attributes:

Step 1: Open AWS Console
├─ Service: EC2
├─ Region: Select your region
└─ Console: Ready

Step 2: Go to Target Groups
├─ Navigation: Left menu
├─ Section: Load Balancing
├─ Option: Target Groups
└─ View: List of all target groups

Step 3: Select Target Group
├─ Click: On target group to edit
├─ Details: Open target group page
└─ Location: Now viewing target group

Step 4: Edit Attributes
├─ Button: "Edit Attributes" or just "Attributes"
├─ Location: Usually on target group details page
├─ Or: Actions menu → Edit Attributes
└─ Page: Now showing editable attributes

Deregistration Delay Configuration:

Location:
├─ Page: Attributes
├─ Section: Find "Deregistration delay"
└─ Field: Timeout in seconds

Setting Value:
├─ Default: 300 (will be shown)
├─ Input: Click field, enter new value
├─ Range: 1-3600 seconds
├─ Input examples:
│  ├─ 30 for short timeouts
│  ├─ 60 for medium
│  ├─ 300 for default
│  └─ 900 for long operations

Disable:
├─ To disable: Enter 0
├─ Effect: Immediate deregistration
└─ Warning: Connections terminated immediately

Save:
├─ Button: "Save changes"
├─ Confirmation: Usually appears
└─ Effect: Applied immediately to new deregistrations

Slow Start Configuration:

Location:
├─ Page: Attributes
├─ Section: Find "Slow start mode" or "Slow start duration"
└─ Field: Duration in seconds

Setting Value:
├─ Default: 0 (disabled)
├─ To enable: Enter seconds (30-900)
├─ Common values:
│  ├─ 0 = disabled (default)
│  ├─ 30 = short warm-up
│  ├─ 60 = moderate warm-up
│  └─ 120+ = extended warm-up

Recommendation:
├─ Most apps: 60 seconds
├─ Fast apps: 30 seconds
├─ Complex apps: 120+ seconds
└─ Test: Start here, adjust based on results

Save:
├─ Button: "Save changes"
├─ Effect: Next new healthy target uses this
└─ Existing: Current targets unaffected

Routing Algorithm Configuration:

Location:
├─ Page: Attributes (or may be separate)
├─ Section: "Routing algorithm" or "Load balancing algorithm"
└─ Options: Dropdown menu

ALB Options:

If ALB:
├─ Option 1: Round Robin (default)
├─ Option 2: Least outstanding request
└─ Choose: Based on workload

Round Robin:
├─ Selection: Should already be selected
├─ Change: Only if needed
├─ Effect: Cycles through targets

Least Outstanding Request:
├─ Selection: Click option
├─ Effect: Routes to least busy target
├─ Benefit: Better for variable workloads
└─ Cost: Minimal CPU overhead

NLB Options:

If NLB:
├─ Note: Flow Hash is only option for TCP/UDP
├─ Automatic: Cannot change
├─ Display: May not show option (uses flow hash always)
└─ Verification: Confirm in description

Save:
├─ Button: "Save changes" (if available)
├─ Effect: Applied to new connections
└─ Existing: Current connections continue with old setting

Stickiness Configuration:

Location:
├─ Page: Attributes
├─ Section: "Stickiness" or "Load balancer stickiness"
└─ Setting: Enable/disable toggle

Enabling Stickiness:

Step 1: Check Box
├─ Find: "Enable stickiness" checkbox
├─ Click: To enable
└─ Checkmark: Now showing

Step 2: Select Type

If Application-Based:
├─ Choose: "Application-based" (likely default if available)
├─ Effect: Uses existing app cookie
├─ Next: Enter cookie name

Cookie Name Field:
├─ Required: If application-based
├─ Example name: SESSIONID
├─ Case-sensitive: Yes
├─ Match: Must match app cookie exactly
└─ Input: Type name carefully

If Duration-Based:
├─ Choose: "Duration-based"
├─ Effect: ALB generates/manages cookie
├─ Next: Set duration

Duration Field:
├─ Default: 86400 (24 hours)
├─ Input: Change if needed
├─ Range: 1 to 604800 seconds
├─ Calculate: Your expected session length

Cookie Name (Optional):
├─ Default: AWSALB if not specified
├─ Optional: Can customize
└─ Rarely needed: Default fine for most

Save:
├─ Button: "Save changes"
├─ Effect: Applied to new requests
└─ Existing: Requests already in-flight continue

Disabling Stickiness:

Step 1: Find Stickiness
├─ Section: Stickiness settings
└─ Current: Enabled (showing checkmark)

Step 2: Uncheck
├─ Click: Enable checkbox to uncheck
├─ Effect: Stickiness disabled
└─ Confirmation: Checkbox now empty

Step 3: Save
├─ Button: "Save changes"
├─ Effect: Future requests routed normally
└─ Existing: Current sticky sessions continue until cookie expiry

Cross-Zone Load Balancing:

Location:
├─ Page: May be Attributes or separate setting
├─ Section: "Cross-zone load balancing" or similar
└─ Option: Enable/disable

Setting:

Enable:
├─ Purpose: Balance across AZs
├─ Effect: Additional data transfer charges
├─ Benefit: More even distribution
└─ Check: If instances spread across AZs

Disable:
├─ Purpose: Save data transfer costs
├─ Effect: Routes within same AZ preferred
├─ Trade-off: Less even distribution
└─ Use: If single AZ or cost-sensitive

Preserve Client IP:

Location:
├─ Page: Attributes
├─ Section: "Preserve client IP" or "Client IP"
└─ Option: Enable/disable toggle

Setting:

Enable:
├─ Method: passes client IP to target
├─ For ALB: Via X-Forwarded-For header
├─ For NLB: Direct IP visible to target
├─ Use: Logging, security decisions
└─ Recommendation: Usually enable

Disable:
├─ Effect: Target sees LB IP instead
├─ Problem: Can't identify real client
├─ Use: If specifically needed otherwise
└─ Rare: Usually keep enabled

Verify All Settings:

Checklist:

Before Saving:
┌─ Deregistration delay: Correct timeout value?
├─ Slow start: Enabled with appropriate duration?
├─ Routing algorithm: Correct for workload?
├─ Stickiness: Enabled/disabled appropriately?
│  ├─ If enabled: Type selected?
│  ├─ If app-based: Cookie name correct?
│  └─ If duration-based: Duration appropriate?
├─ Cross-zone: Enabled as expected?
├─ Preserve client IP: Enabled as expected?
└─ Ready: To save?

Save Changes:

Button Location:
├─ Usually: Bottom of page
├─ Label: "Save changes" or "Update attributes"
└─ Click: To apply

Confirmation:
├─ Message: Usually appears
├─ Status: "Changes saved successfully"
└─ Effect: Applied immediately

Verification:

After Save:
1. Refresh page (F5)
2. Verify: Settings still showing as set
3. Check: No error messages
4. Monitor: Target behavior for next 5-10 minutes
5. Confirm: Working as expected

Testing New Settings:

Slow Start Test:

Verify Working:
1. Add new healthy target
2. Monitor: TargetResponseTime in CloudWatch
3. Observe: Gradual increase expected over duration
4. Confirm: After duration, normal response time
5. If not: Check configuration

Routing Algorithm Test:

Round Robin:
├─ Send 30 requests from client
├─ Monitor: Request distribution across targets
├─ Expected: Approximately equal distribution
├─ Verify: Each target gets ~10 requests

Least Outstanding Request:
├─ Send mixed request types (fast and slow)
├─ Monitor: Response times
├─ Expected: Better balance than round robin
├─ Verify: No obvious queuing

Stickiness Test:

Enable & Test:
1. Make first request (gets routed to target A)
2. Check: Response includes cookie header
3. Make second request (with cookie)
4. Verify: Routed to same target A
5. Monitor: Access logs for confirmation

CLI Configuration (Advanced):

Modify Target Group Attributes:

Command:
```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn <target-group-arn> \
  --attributes \
    Key=deregistration_delay.timeout_seconds,Value=300 \
    Key=slow_start.duration_seconds,Value=60 \
    Key=stickiness.enabled,Value=true \
    Key=stickiness.type,Value=lb_cookie
```

For Application-Based Stickiness:
```bash
aws elbv2 modify-target-group-attributes \
  --target-group-arn <target-group-arn> \
  --attributes \
    Key=stickiness.enabled,Value=true \
    Key=stickiness.type,Value=app_cookie \
    Key=stickiness.app_cookie.cookie_name,Value=SESSIONID
```

Verify Settings:

```bash
aws elbv2 describe-target-group-attributes \
  --target-group-arn <target-group-arn>
```

Output will show all current settings.
```

---

## Part 10: Choosing Right Attributes

### Decision Framework

**Configuration Best Practices**

```
Decision Tree for Target Group Configuration:

Step 1: Deregistration Delay Setting

Question: How long are typical requests?
├─ Answer A: < 5 seconds → Set to 30-60 seconds
├─ Answer B: 5-60 seconds → Set to 60-300 seconds
├─ Answer C: > 60 seconds → Set to 300-900 seconds
└─ Calculation: (max_request_time × 1.5) to 2.0

Step 2: Slow Start Decision

Question: Is application complex/stateful?
├─ Answer A: Simple/stateless (APIs) → Set 30-60 seconds
├─ Answer B: Moderate complexity → Set 60-120 seconds
├─ Answer C: Very complex (databases) → Set 120-300 seconds
└─ Default: Enable for all unless zero benefit

Question: How often add new instances?
├─ Answer A: Rarely (static setup) → Still enable (no downside)
├─ Answer B: Often (auto-scaling) → Definitely enable
├─ Answer C: Continuous (serverless) → Extra important

Step 3: Routing Algorithm Selection

Question: Are request processing times similar?
├─ Answer A: Yes, very similar times → Use Round Robin
├─ Answer B: Variable (10ms to 1s) → Use Least Outstanding Request
├─ Answer C: Highly variable (1s to 1000s) → Use Least Outstanding Request

Question: What load balancer type?
├─ Answer A: ALB → Either Round Robin OR Least Outstanding Request
├─ Answer B: CLB → Either Round Robin OR Least Outstanding Request
├─ Answer C: NLB → Only Flow Hash available (automatic)

Question: Performance critical?
├─ Answer A: No, standard web app → Round Robin fine
├─ Answer B: Yes, latency matters → Least Outstanding Request recommended
├─ Answer C: Extreme, real-time → Least Outstanding Request likely better

Selection Guide:

For Web Applications: Round Robin
├─ Reason: Requests similar, simple logic
├─ Example: Blog, informational site
└─ Result: Good balance of simplicity and performance

For APIs: Round Robin
├─ Reason: Standardized requests
├─ Example: REST APIs, GraphQL
└─ Result: Even distribution works

For Microservices: Least Outstanding Request
├─ Reason: Variable processing times
├─ Example: Multiple service types
└─ Result: Better load balancing

For Batch Processing: Least Outstanding Request
├─ Reason: Job times vary significantly
├─ Example: Image processing, encoding
└─ Result: Prevents bottlenecking

Step 4: Stickiness Decision

Question: Is application stateful?
├─ Answer A: Completely stateless → Disable stickiness
├─ Answer B: Mostly stateless, some sessions → Choose based on need
├─ Answer C: Stateful (in-memory sessions) → Enable stickiness

Question: Does application create cookies?
├─ Answer A: Yes, explicit session cookies → Use Application-based
├─ Answer B: No, need ALB to manage → Use Duration-based
└─ Answer C: Connection-based state (NLB) → Uses Flow Hash (automatic)

Question: Session lifetime?
├─ Answer: If duration-based, set to match expected session length

Step 5: Cross-Zone Load Balancing

Question: Instances across multiple AZs?
├─ Answer A: Single AZ only → Can disable (saves cost)
├─ Answer B: Multiple AZs, cost-sensitive → Disable to save
├─ Answer C: Multiple AZs, performance critical → Enable
├─ Answer D: Multiple AZs, any → Depends on workload

Question: Even distribution important?
├─ Answer A: Not critical → Disable (cost optimization)
├─ Answer B: Important → Enable (accept extra cost)
└─ Answer C: Critical → Enable at all costs

Configuration Templates:

Template 1: Standard Web Application

Setup:
├─ Application type: Traditional web app
├─ Sessions: Cookie-based
├─ Instances: Multiple AZs
└─ Priority: Balance of simplicity and performance

Configuration:
├─ Deregistration delay: 300 seconds
├─ Slow start: 60 seconds
├─ Routing algorithm: Round Robin
├─ Stickiness: Duration-based, 86400 seconds
├─ Cross-zone: Enabled
└─ Preserve client IP: Enabled

Reasoning:
├─ Delay: Default works for most
├─ Slow start: Helps with new deployments
├─ Round robin: Sufficient for similar-time requests
├─ Stickiness: Sessions need persistence
├─ Cross-zone: Ensure even distribution
└─ Preserve IP: For logging and security

Template 2: RESTful API

Setup:
├─ Application type: Microservice API
├─ Sessions: None (stateless)
├─ Instances: Multiple AZs
└─ Priority: Performance and efficiency

Configuration:
├─ Deregistration delay: 60 seconds
├─ Slow start: 60 seconds
├─ Routing algorithm: Least Outstanding Request
├─ Stickiness: Disabled
├─ Cross-zone: Enabled
└─ Preserve client IP: Enabled

Reasoning:
├─ Delay: Shorter for quick requests
├─ Slow start: Helps API performance at startup
├─ LOR: Variable API response times
├─ No stickiness: Stateless by design
├─ Cross-zone: Performance focused
└─ Preserve IP: For rate limiting, logging

Template 3: Long-Running Batch Processing

Setup:
├─ Application type: Video processing, analytics
├─ Sessions: Connection-based state
├─ Instances: EC2 with significant compute
└─ Priority: Throughput and reliability

Configuration:
├─ Deregistration delay: 900 seconds (15 min)
├─ Slow start: 120 seconds
├─ Routing algorithm: Least Outstanding Request
├─ Stickiness: Duration-based, 3600 seconds
├─ Cross-zone: Enabled
└─ Preserve client IP: Enabled

Reasoning:
├─ Delay: Long for in-flight jobs to complete
├─ Slow start: Extended warm-up for compute jobs
├─ LOR: Highly variable processing times
├─ Stickiness: Some request types need continuity
├─ Cross-zone: Distribute compute load
└─ Preserve IP: For tracking large uploads

Template 4: Real-Time Communication (NLB)

Setup:
├─ Application type: WebSocket, gRPC, messaging
├─ Sessions: Long-lived connections
├─ Instances: Multiple AZs
└─ Priority: Low latency, perfect affinity

Configuration (NLB):
├─ Deregistration delay: 300 seconds
├─ Slow start: 60 seconds
├─ Routing algorithm: Flow Hash (automatic, NLB)
├─ Stickiness: Not applicable (built-in via flow hash)
├─ Cross-zone: Enabled
└─ Preserve client IP: Enabled

Reasoning:
├─ Delay: Graceful shutdown for connections
├─ Slow start: Helps server initialization
├─ Flow hash: Perfect for long-lived connections
├─ Affinity: Built-in at protocol level
├─ Cross-zone: Distribute connections
└─ Preserve IP: For security and logging

Common Mistakes to Avoid:

Mistake 1: Too Short Deregistration Delay
├─ Problem: Requests fail during shutdown
├─ Effect: Users see errors
├─ Solution: Size appropriately
└─ Formula: (max_request_duration × 1.5) + safety_margin

Mistake 2: Stickiness without Reason
├─ Problem: Reduces flexibility, limits scaling
├─ Effect: Bottlenecks, uneven load
├─ Solution: Only enable if truly needed

Mistake 3: Wrong Routing Algorithm
├─ Problem: Poor performance, wasted capacity
├─ Effect: Higher latency, reduced throughput
├─ Solution: Test and choose based on workload

Mistake 4: Mismatched Timeouts
├─ Problem: Session expires or timeout wrong
├─ Effect: User experience issues
├─ Solution: Align deregistration and stickiness durations

Mistake 5: Ignoring Application Requirements
├─ Problem: Configuration doesn't match app design
├─ Effect: Unexpected behavior
├─ Solution: Understand app architecture first

Testing Recommendations:

Load Testing:

Before Production:
1. Use load testing tool (Apache JMeter, Gatling)
2. Test with current configuration
3. Measure: Latency p50, p95, p99
4. Record: Error rates, throughput
5. Document: Baseline metrics

Compare Configurations:
1. Change one setting
2. Run same load test
3. Compare: New vs baseline
4. Measure: Impact
5. Decide: Keep change or revert

Gradual Rollout:

In Production:
1. Deploy: To small percentage of traffic
2. Monitor: For 15-30 minutes
3. Check: Errors, latency changes
4. If good: Increase traffic percentage
5. Escalate: To 100% after validation
6. If issues: Rollback immediately
```

---

## Part 11: Exam Focus Points

### Key Test Questions

```
1. "What is the default deregistration delay setting?"
   A) 30 seconds
   B) 60 seconds
   C) 300 seconds
   D) 900 seconds
   
   Answer: C) 300 seconds (5 minutes)
   └─ Know: This is the default
   └─ Remember: Can be changed 1-3600

2. "Which attribute prevents new instances from being overwhelmed?"
   A) Deregistration delay
   B) Slow start
   C) Cross-zone load balancing
   D) Stickiness
   
   Answer: B) Slow start
   └─ Know: Ramps traffic gradually
   └─ Duration: 0-900 seconds

3. "What does Least Outstanding Request algorithm do?"
   A) Routes to first available target
   B) Cycles through targets sequentially
   C) Routes to target with fewer pending requests
   D) Uses protocol-based hashing
   
   Answer: C) Routes to target with fewer pending requests
   └─ Know: Better for variable workloads
   └─ Available: ALB, CLB (not NLB)

4. "Which routing algorithm is NLB-specific?"
   A) Round Robin
   B) Flow Hash
   C) Least Outstanding Request
   D) Random
   
   Answer: B) Flow Hash
   └─ Know: NLB only, for TCP/UDP
   └─ Benefit: Connection affinity

5. "What type of stickiness uses ALB-generated cookie?"
   A) Application-based
   B) Duration-based
   C) Connection-based
   D) Flow-based
   
   Answer: B) Duration-based
   └─ Know: ALB manages the cookie
   └─ Default duration: 86400 seconds (1 day)

6. "When should you enable stickiness?"
   A) Always, for all applications
   B) Only if application is stateful
   C) For all web applications
   D) Never, reduces performance
   
   Answer: B) Only if application is stateful
   └─ Know: Stateless apps don't need it
   └─ Examples: Stateless APIs don't need stickiness

7. "What is Flow Hash based on?"
   A) Request URL path
   B) Client HTTP header
   C) Protocol, IP addresses, ports, TCP sequence
   D) Application-specific data
   
   Answer: C) Protocol, IP addresses, ports, TCP sequence
   └─ Know: Complete connection details
   └─ Result: Same connection always same target

8. "How long should deregistration delay be for a video upload service?"
   A) 30 seconds
   B) 60 seconds
   C) 300 seconds
   D) 900+ seconds
   
   Answer: D) 900+ seconds
   └─ Know: Long requests need long timeout
   └─ Consider: Video uploads (5+ min) need protection

9. "Can you change routing algorithm for NLB?"
   A) Yes, to Round Robin
   B) Yes, to Least Outstanding Request
   C) No, must use Flow Hash
   D) Yes, to any algorithm
   
   Answer: C) No, must use Flow Hash
   └─ Know: NLB locked to Flow Hash
   └─ Reason: TCP/UDP connection model

10. "What is the purpose of slow start?"
    A) Prevent target from becoming unhealthy
    B) Gradually increase traffic to warm-up target
    C) Reduce ALB processing time
    D) Enable session persistence
    
    Answer: B) Gradually increase traffic to warm-up target
    └─ Know: From 0 to full over duration
    └─ Default: Disabled (0 seconds)

11. "Which combo works best for stateful applications?"
    A) Round Robin + No stickiness
    B) Least Outstanding Request + Application stickiness
    C) Flow Hash + Duration stickiness
    D) Any algorithm + No stickiness
    
    Answer: B) Least Outstanding Request + Application stickiness
    └─ Know: Must handle state + balance load
    └─ Reason: Stateful needs affinity, variable load needs LOR

12. "What is NOT a valid deregistration delay?"
    A) 0 seconds
    B) 300 seconds
    C) 3600 seconds
    D) 7200 seconds
    
    Answer: D) 7200 seconds (2 hours)
    └─ Know: Max is 3600 seconds (1 hour)
    └─ Range: 0-3600 seconds
```

---

## Part 12: Best Practices and Summary

### Optimal Configuration Strategy

**Comprehensive Checklist**

```
Pre-Implementation Checklist:

1. Understand Application:
   ☐ Stateful or stateless?
   ☐ Average request duration?
   ☐ Request duration range (min-max)?
   ☐ Session model (if any)?
   ☐ Does app create cookies?
   ☐ Connection model (request vs long-lived)?
   ☐ Multi-AZ instances?

2. Determine Attributes:
   ☐ Deregistration delay: Calculated?
   ☐ Slow start: Needed (duration)?
   ☐ Routing algorithm: Chosen?
   ☐ Stickiness: Needed (type)?
   ☐ Cross-zone: Needed?
   ☐ Preserve IP: Needed?

3. Test Plan:
   ☐ Load test script: Created?
   ☐ Baseline metrics: Recorded?
   ☐ Test environment: Set up?
   ☐ Monitoring enabled: CloudWatch?
   ☐ Alarms configured: For key metrics?

Configuration Checklist:

ALB Configuration:

Attributes Tab:
☐ Deregistration delay: Set correctly
☐ Slow start duration: Enabled if needed
☐ Slow start applies to: New healthy targets
☐ Routing algorithm: Selected (Round Robin or LOR)
☐ Stickiness: Enabled/disabled as needed
  ☐ Type: Application-based or Duration-based
  ☐ Cookie name: Correct (if app-based)
  ☐ Duration: Set to match session length
☐ Cross-zone: Load balancing enabled if multi-AZ
☐ Preserve client IP: Enabled for logging

NLB Configuration:

Attributes Tab (if using for HTTP):
☐ Deregistration delay: Set correctly
☐ Slow start duration: Enabled if needed
☐ Flow Hash: Already selected (default, TCP/UDP)
☐ Connection settings: Configured if needed
☐ Cross-zone: Load balancing enabled if multi-AZ
☐ Preserve client IP: Enabled for logging

Monitoring Setup:

CloudWatch Metrics:
☐ Metrics: Enabled (should be automatic)
☐ Dashboard: Created with key metrics
☐ TargetResponseTime: Tracked for performance
☐ RequestCountPerTarget: Verified for balance
☐ ErrorCodes: Monitored (4XX, 5XX)
☐ HealthyHostCount: Alert if drops

Alarms:
☐ High Priority Alarms:
  ☐ HealthyHostCount = 0 (critical!)
  ☐ HTTPCode_ELB_5XX > 0 (critical!)
  ☐ Spillover > 0 (critical!)
☐ Medium Priority Alarms:
  ☐ UnHealthyHostCount > 0 (warning)
  ☐ SurgeQueueLength > threshold (warning)
☐ Alarm Actions:
  ☐ SNS topic: Configured
  ☐ Recipients: Set (email, Slack, PagerDuty)
  ☐ Thresholds: Appropriate to your workload

Access Logs:

Setup:
☐ S3 bucket: Created for logs
☐ Bucket policy: Allows ALB to write
☐ ALB enable: Access logs enabled
☐ Prefix: Set (optional but recommended)
☐ Athena: Table created for querying

Testing Checklist:

Load Testing:

Baseline Test:
☐ Request rate: Sustainable from client
☐ Duration: 30-60 minutes
☐ Mix: Representative of actual traffic
☐ Results captured:
  ☐ Latency (p50, p95, p99)
  ☐ Throughput (requests/sec)
  ☐ Error rate
  ☐ CPU/Memory on targets

Slow Start Validation:

Test Procedure:
1. ☐ Add new healthy target
2. ☐ Monitor: TargetResponseTime
3. ☐ Verify: Ramp-up over configured duration
4. ☐ Check: After duration, normal response time
5. ☐ Confirm: No spike in errors during ramp

Routing Algorithm Validation:

Round Robin Test:
1. ☐ Send exactly 30 sequential requests
2. ☐ Verify: 10 requests to each of 3 targets
3. ☐ Confirm: Pattern is perfectly cyclic

Least Outstanding Request Test:
1. ☐ Load test with variable response times
2. ☐ Monitor: Response time distribution
3. ☐ Verify: Better balance than round robin
4. ☐ Confirm: No specific target overloaded

Stickiness Validation:

Stickiness Test:
1. ☐ Make first request, capture returned cookie
2. ☐ Make second request with cookie included
3. ☐ Verify: Same target used for both
4. ☐ Check: Access logs show routing consistency
5. ☐ Confirm: Until cookie expiry or timeout

Deregistration Delay Test:

Graceful Shutdown Test:
1. ☐ Deregister target during moderate load
2. ☐ Monitor: Existing connections declining
3. ☐ Check: New connections routed elsewhere
4. ☐ Verify: No connection drops during drain time
5. ☐ Confirm: All requests complete gracefully

Production Rollout:

Staged Deployment:

Phase 1: Testing Environment
☐ Full configuration applied
☐ Load testing completed
☐ Metrics baseline established
☐ Alarms verified working
☐ Team trained

Phase 2: Production (Canary):
☐ Deploy: Small percentage of traffic (5-10%)
☐ Monitor: 15-30 minutes
☐ Check:
  ☐ Error rate normal?
  ☐ Latency p99 acceptable?
  ☐ Alarms not firing?
  ☐ Targets healthy?
☐ Proceed: Start with 25% traffic
☐ Repeat: Monitor 15 min, gradually increase

Phase 3: Production (Full):
☐ 100% traffic reached
☐ Continued monitoring: 1 hour+
☐ Documentation: Updated with actual production settings
☐ Team: Notified of new configuration
☐ Runbook: Ensure team knows how to troubleshoot

Ongoing Management:

Weekly:
☐ Review: CloudWatch dashboard
☐ Check: Error trends, latency trends
☐ Verify: No anomalies
☐ Performance: Within expected ranges

Monthly:
☐ Baseline review: Compare to baseline from month ago
☐ Trends: Growing, stable, or declining?
☐ Optimization: Can any settings be tuned?
☐ Capacity: Enough headroom for growth?

Quarterly:
☐ Full review: All attributes
☐ Decision: Any changes needed?
☐ Workload: Has application changed?
☐ Traffic: Growing or shifting?
☐ Adjustments: Plan and test if needed

Troubleshooting Decision Tree:

Problem: High error rate (500s)

Investigate:
├─ Check: HealthyHostCount metric
├─ If 0: All targets unhealthy
│  └─ Action: Check target logs, security groups
├─ If > 0: Some targets up
│  ├─ Check: TargetResponseTime
│  ├─ If high: Backend slow
│  │  └─ Action: Optimize app, add capacity
│  └─ If normal: Connectivity issue
│     └─ Action: Check ports, firewalls

Problem: Increased latency

Investigate:
├─ Check: RequestCountPerTarget
├─ If high: Requests overloading targets
│  └─ Action: Scale out (add targets)
├─ Check: SurgeQueueLength
├─ If high: Requests queuing
│  └─ Action: Scale immediately
├─ Check: TargetResponseTime
├─ If backend slow: App issue
│  └─ Action: Debug application

Problem: Uneven request distribution

Investigate:
├─ Check: Deployment state (slow start?)
├─ Check: Routing algorithm settings
├─ Verify: Targets have similar capacity
├─ Check: No targets marked unhealthy
├─ If all ok: Request handling time difference
│  └─ Action: May be normal, monitor trend

Problem: Session loses state

Investigate:
├─ Check: Stickiness enabled?
├─ If disabled: Enable if needed
├─ If enabled: Check type
│  ├─ App-based: Verify cookie name matches app
│  └─ Duration-based: Increase duration if needed
├─ Verify: Cookie present in requests
├─ Check: Application session timeout
│  └─ Align: With stickiness duration

Key Metrics Summary:

Always Monitor:
├─ HealthyHostCount: Are targets up?
├─ UnHealthyHostCount: Any problems?
├─ RequestCountPerTarget: Balanced distribution?
├─ TargetResponseTime: Performance acceptable?
├─ HTTPCode_5XX: Server errors present?

Sometimes Monitor:
├─ SurgeQueueLength: Queue forming?
├─ Spillover: Traffic being dropped?
├─ BackendConnectionErrors: Connection problems?
├─ ActiveConnectionCount: Long-lived service?

Best Practices Summary:

1. Match configuration to application:
   ├─ Stateless: No stickiness needed
   ├─ Stateful: Stickiness required
   ├─ Variable workload: Least Outstanding Request
   └─ Constant workload: Round Robin sufficient

2. Size timeouts appropriately:
   ├─ Deregistration delay: Based on max request time
   ├─ Slow start: Match app warmup time
   ├─ Stickiness duration: Match session lifetime
   └─ All: Conservative, err on longer side

3. Test thoroughly:
   ├─ Lab environment: Before production
   ├─ Load testing: Validate behavior
   ├─ Monitoring: Verify production behavior
   └─ Gradual rollout: Stage to production safely

4. Monitor continuously:
   ├─ Dashboard: View key metrics
   ├─ Alarms: Immediate notification of problems
   ├─ Logs: Access logs for deeper analysis
   └─ Trends: Monthly review for optimization

5. Document everything:
   ├─ Configuration: Why each setting chosen
   ├─ Baseline: Expected normal metrics
   ├─ Procedures: How to troubleshoot
   └─ Changes: Track what changed and when

Configuration Template Summary:

┌─────────────────────────────────────────────────────────────────┐
│ Deregistration Delay: (max_request_duration × 1.5)             │
│ Short requests (< 5s): 30-60 seconds                           │
│ Medium requests (5-60s): 60-300 seconds                        │
│ Long requests (> 60s): 300-900 seconds                         │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ Slow Start: Enable for all, set based on warmup time           │
│ Simple apps: 30-60 seconds                                     │
│ Moderate: 60-120 seconds                                       │
│ Complex: 120-300 seconds                                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ Routing Algorithm:                                              │
│ ALB: Round Robin (default) or Least Outstanding Request        │
│ CLB: Round Robin (default) or Least Outstanding Request        │
│ NLB: Flow Hash (only option for TCP/UDP)                       │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│ Stickiness:                                                     │
│ Stateless (APIs): Disable                                      │
│ Stateful (web app): Enable (app-based if possible)             │
│ Duration: Set to session_length × 1.2 to 1.5                  │
└─────────────────────────────────────────────────────────────────┘

Perfect for Exam Preparation:

Know These Facts:
✓ Default deregistration delay: 300 seconds
✓ Max deregistration delay: 3600 seconds
✓ Default slow start: 0 (disabled)
✓ Max slow start: 900 seconds
✓ Default stickiness: Disabled
✓ Default stickiness duration: 86400 seconds
✓ Default routing algorithm: Round Robin
✓ NLB routing: Flow Hash only
✓ ALB routing: Round Robin or Least Outstanding Request

Know These Algorithms:
✓ Round Robin: Cycles through targets
✓ Least Outstanding Request: Routes to least busy target
✓ Flow Hash: NLB-specific, uses protocol/IP/port/TCP seq

Know These Benefits:
✓ Slow start: Prevents overwhelming new instances
✓ Stickiness: Maintains session state
✓ Deregistration delay: Allows graceful shutdown
✓ Least Outstanding Request: Better for variable workloads
```

---

## Conclusion

Target group attributes provide comprehensive control over request routing behavior, instance lifecycle management, and session persistence. Proper configuration aligned with application requirements ensures optimal performance, reliability, and user experience. Master these settings for both AWS SysOps exams and production deployment success!
