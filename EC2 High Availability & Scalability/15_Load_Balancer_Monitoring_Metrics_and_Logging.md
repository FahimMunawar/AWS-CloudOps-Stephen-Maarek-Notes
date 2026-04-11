# Load Balancer Monitoring, Metrics, and Logging

Comprehensive monitoring of load balancers requires understanding HTTP status codes, CloudWatch metrics, access logs, and request tracing to identify performance issues, errors, and capacity constraints.

---

## Part 1: HTTP Status Codes from Load Balancers

### Understanding Status Code Categories

**Status Code Ranges**

```
HTTP Status Code Structure:

Status codes: 100-599
├─ 1XX: Informational (100-199) - Temporary responses
├─ 2XX: Success (200-299) - Request succeeded
├─ 3XX: Redirect (300-399) - Further action needed
├─ 4XX: Client Error (400-499) - Client problem
└─ 5XX: Server Error (500-599) - Server problem

For Load Balancer Monitoring:

2XX (Success):
├─ Range: 200-299
├─ Meaning: Request succeeded
├─ Action: Track for baseline metrics
├─ Monitoring: Normal operation
└─ Example: 200 OK

4XX (Client Error):
├─ Range: 400-499
├─ Meaning: Client's fault
├─ Action: May indicate bad requests
├─ Monitoring: Should be rare (< 1%)
└─ Root: Client side, not server

5XX (Server Error):
├─ Range: 500-599
├─ Meaning: Server's fault
├─ Action: Indicates infrastructure problem
├─ Monitoring: Track closely
└─ Root: Load balancer or backend issue

Exam Tip:
├─ Remember: 4XX = Client problem
├─ Remember: 5XX = Server problem
└─ Important: For troubleshooting
```

### Common 2XX Status Codes

**Successful Responses**

```
200 OK:
├─ Meaning: Request succeeded, response returned
├─ Typical: Normal web requests
├─ Expected: Most traffic should be this
├─ Metrics: Count 2XX codes in CloudWatch
└─ Action: No intervention needed

201 Created:
├─ Meaning: Resource created successfully
├─ Typical: POST requests creating resources
├─ Expected: API-based applications
├─ Monitoring: Part of success rate
└─ Action: Normal operation

204 No Content:
├─ Meaning: Request succeeded, no content returned
├─ Typical: DELETE requests, updates
├─ Expected: API applications
├─ Monitoring: Part of success rate
└─ Action: Normal operation
```

### Common 4XX Status Codes

**Client-Side Errors**

```
400 Bad Request:
├─ Meaning: Client sent malformed request
├─ Causes: 
│  ├─ Malformed JSON/XML
│  ├─ Missing required parameters
│  ├─ Invalid header format
│  └─ Protocol violation from client
├─ Typical: Happens regularly
├─ Root cause: Client application bug
├─ Monitoring: Track percentage (should be < 1%)
└─ Action: Investigate client application

401 Unauthorized:
├─ Meaning: Authentication required/failed
├─ Causes:
│  ├─ Missing credentials
│  ├─ Invalid API key
│  ├─ Expired token
│  └─ Invalid username/password
├─ Typical: Secure APIs
├─ Root cause: Client authentication issue
├─ Monitoring: Track login failures
└─ Action: Check authentication system

403 Forbidden:
├─ Meaning: Authenticated but not authorized
├─ Causes:
│  ├─ Insufficient permissions
│  ├─ IP blocked/whitelisted
│  ├─ Rate limit exceeded
│  └─ Geographical restriction
├─ Typical: Security policies
├─ Root cause: Client authorization issue
├─ Monitoring: Track permission denials
└─ Action: Check access control policies

460 Client Closed Connection:
├─ Meaning: Client closed connection before response
├─ Causes:
│  ├─ Client timeout
│  ├─ User cancelled request
│  ├─ Network disconnection
│  └─ Client application crash
├─ Typical: Network issues, impatient clients
├─ Root cause: Client side
├─ Monitoring: Occasional events normal
└─ Action: Reduce timeout or optimize response time

463 Ambiguous Header X-Forwarded-For:
├─ Meaning: X-Forwarded-For header malformed
├─ Causes:
│  ├─ Invalid IP addresses in header
│  ├─ Incorrectly formatted proxies
│  └─ Proxy chain issue
├─ Typical: Multi-tier proxy scenarios
├─ Root cause: Proxy/client configuration
├─ Monitoring: Should be rare
└─ Action: Verify proxy configuration

404 Not Found:
├─ Meaning: Resource doesn't exist
├─ Causes:
│  ├─ Wrong URL path
│  ├─ Resource deleted
│  ├─ Typo in client request
│  └─ Health check on wrong path
├─ Typical: Occasional (typos, etc.)
├─ Root cause: Client error
├─ Monitoring: High 404s may indicate health check issue
└─ Action: Check health check configuration if spike
```

### Common 5XX Status Codes

**Server-Side Errors**

```
500 Internal Server Error:
├─ Meaning: Application error occurred
├─ Causes:
│  ├─ Unhandled exception
│  ├─ Application crash
│  ├─ Database query failed
│  ├─ Resource exhaustion
│  └─ Code bug/logic error
├─ Root cause: Backend application
├─ Monitoring: Alert if > 0.1%
├─ Severity: High
└─ Action: Check application logs immediately

502 Bad Gateway:
├─ Meaning: ALB can't connect to backend
├─ Causes:
│  ├─ Backend instance crashed
│  ├─ Port not listening
│  ├─ Network connectivity issue
│  ├─ Security group blocking
│  └─ Backend unable to accept connections
├─ Root cause: Backend or network
├─ Monitoring: Alert if any occurrence
├─ Severity: Critical
└─ Action: Check backend health, security groups

503 Service Unavailable:
├─ Meaning: Service temporarily unavailable
├─ Causes:
│  ├─ No healthy backend instances
│  ├─ Load balancer overloaded
│  ├─ All backends down
│  ├─ Maintenance mode
│  └─ Rate limiting triggered
├─ Root cause: Backend or LB capacity
├─ Monitoring: Alert immediately
├─ Severity: Critical
└─ Action: Check HealthyHostCount metric, scale up

504 Gateway Timeout:
├─ Meaning: Backend took too long to respond
├─ Causes:
│  ├─ Backend processing slow
│  ├─ Long-running requests
│  ├─ Network latency high
│  ├─ LB idle timeout reached
│  └─ Backend hung/unresponsive
├─ Root cause: Backend performance
├─ Monitoring: Investigate if spike
├─ Severity: Medium-High
└─ Action: Check backend performance, tune timeouts

561 Unauthorized (AWS specific):
├─ Meaning: ALB cannot authorize request
├─ Causes:
│  ├─ Authentication failed at ALB
│  ├─ Token validation failed
│  ├─ Cognito/OIDC provider issue
│  └─ ALB authenticator misconfigured
├─ Root cause: ALB authentication rules
├─ Monitoring: Check authentication logs
├─ Severity: Medium
└─ Action: Verify ALB authentication config
```

---

## Part 2: CloudWatch Metrics for Load Balancers

### Key Metrics Overview

**Essential Load Balancer Metrics**

```
Metric Categories:

1. Host Health Metrics:
   ├─ HealthyHostCount
   ├─ UnHealthyHostCount
   └─ Health check status

2. Error Metrics:
   ├─ 2XX successful requests
   ├─ 3XX redirect responses
   ├─ 4XX client errors
   ├─ 5XX server errors
   └─ Exception counts

3. Performance Metrics:
   ├─ TargetResponseTime
   ├─ RequestCount
   ├─ RequestCountPerTarget
   └─ Latency measurements

4. Capacity Metrics:
   ├─ SurgeQueueLength
   ├─ Spillover count
   ├─ ActiveConnectionCount
   └─ Consumed LB Capacity Units

5. Backend Metrics:
   ├─ BackendConnectionErrors
   ├─ TargetTLSNegotiationErrorCount
   └─ HTTP/TCP connection metrics

Metric Sources:

CloudWatch Metrics:
├─ Published by: ALB/NLB directly
├─ Namespace: AWS/ApplicationELB or AWS/NetworkELB
├─ Resolution: 1-minute or 5-minute granularity
├─ Retention: 15 months
└─ Cost: No charge for standard metrics

Access Logs (Additional Detail):
├─ Published by: ALB/NLB to S3
├─ Content: Request-level details
├─ Granularity: Every request
├─ Retention: Depends on S3 lifecycle
└─ Cost: S3 storage + request charges
```

### HealthyHostCount and UnHealthyHostCount

**Host Health Metrics**

```
HealthyHostCount:

Definition:
├─ Count: Number of healthy target instances
├─ Source: Health check results
├─ Updated: After health check transitions
├─ Granularity: Per target group
└─ Range: 0 to N (N = total instances)

What It Shows:

Example Setup:
├─ Target group: 6 total instances
├─ Healthy: 4 instances
├─ Unhealthy: 2 instances
├─ HealthyHostCount metric: 4
├─ UnHealthyHostCount metric: 2

Timeline:

T=10:00: 6 instances healthy
└─ HealthyHostCount = 6

T=10:05: Instance 1 fails health check
├─ Transitioning...
├─ After unhealthy threshold reached
└─ HealthyHostCount = 5

T=10:10: Instance 2 crashes
├─ Health check fails
├─ Transitions to unhealthy
└─ HealthyHostCount = 4

T=10:15: New instance launched
├─ Initially unhealthy
├─ Passes healthy threshold
└─ HealthyHostCount = 5

Monitoring Importance:

Why Track:
├─ Capacity indicator: How many instances ready
├─ Scale trigger: When to add/remove instances
├─ Health indicator: System health status
├─ Alerting: Know when instances go down
└─ Troubleshooting: Diagnose failures

Alerts to Set:

Alert 1: HealthyHostCount = 0
├─ Severity: Critical
├─ Meaning: No healthy instances!
├─ Action: Page on-call engineer
└─ Response: Investigate immediately

Alert 2: HealthyHostCount < Expected
├─ Severity: Warning
├─ Example: Expected 5, got 3
├─ Meaning: Some instances unhealthy
├─ Action: Investigate why

Alert 3: UnHealthyHostCount > 0
├─ Severity: Warning
├─ Meaning: Instances going unhealthy
├─ Action: Check application, security groups
└─ Threshold: Usually 0 or very low

CloudWatch Query Example:

Metric: AWS/ApplicationELB | HealthyHostCount
├─ Dimension: TargetGroup = my-targets
├─ Statistic: Average (or Min)
├─ Period: 1 minute
└─ Alert: < 3 for 2 minutes = critical

Configuration Example:

CLI:
$ aws cloudwatch put-metric-alarm \
  --alarm-name healthy-host-count-alert \
  --alarm-description "Alert if healthy hosts < 3" \
  --metric-name HealthyHostCount \
  --namespace AWS/ApplicationELB \
  --statistic Average \
  --period 60 \
  --threshold 3 \
  --comparison-operator LessThanThreshold \
  --evaluation-periods 2
```

### Request Count and Error Metrics

**Traffic and Error Tracking**

```
RequestCount:

Definition:
├─ Count: Total HTTP requests received
├─ Granularity: Per time period (minute/5-min)
├─ Scope: All traffic to load balancer
├─ Dimension: Optional per target group
└─ Use: Measure traffic load

What It Shows:

Baseline Tracking:
├─ Peak traffic: When do requests spike?
├─ Off-peak: Low traffic times
├─ Trends: Growing or decreasing traffic
├─ Patterns: Daily, weekly patterns
└─ Forecasting: Plan capacity based on growth

Example:

Monday 9:00 AM: 10,000 requests/min
├─ Normal business day start
├─ Traffic rising

Monday 12:00 PM: 50,000 requests/min
├─ Peak traffic (lunch time browsing)
├─ High demand

Monday 6:00 PM: 15,000 requests/min
├─ Evening traffic
├─ Declining

RequestCountPerTarget:

Definition:
├─ Count: Average requests per target
├─ Calculation: TotalRequests / HealthyHostCount
├─ Use: Scale on instance load
├─ Dimension: Per target group
└─ Important: For AutoScaling Group policy

What It Shows:

Example:

Total requests: 100,000/min
Healthy targets: 10
RequestCountPerTarget: 10,000 requests/min per instance

Use Case:
├─ Alert: If > 10,000 requests/target/min
├─ Action: Scale out (add instances)
├─ Rationale: Each instance handling too much load
└─ Target: 5,000-8,000 requests/target/min

Error Code Metrics:

HTTPCode_Target_2XX_Count:
├─ Meaning: Successful responses from target
├─ Status codes: 200-299
├─ What's included: All success responses
├─ Monitoring: Should be >99% of traffic

HTTPCode_Target_4XX_Count:
├─ Meaning: Client error (from backend)
├─ Status codes: 400-499
├─ What's included: Client request issues
├─ Monitoring: Should be < 1%

HTTPCode_Target_5XX_Count:
├─ Meaning: Server error (from backend)
├─ Status codes: 500-599
├─ What's included: Backend failures
├─ Monitoring: Should be 0 or very low

HTTPCode_ELB_5XX_Count:
├─ Meaning: LB originated errors
├─ Status codes: 500-599
├─ What's included: LB-specific failures
├─ Examples: 502, 503, 504 (from ALB, not backend)
├─ Monitoring: Alert on any occurrence

Error Rate Calculation:

Formula:
ErrorRate = (4XX + 5XX) / (2XX + 3XX + 4XX + 5XX) × 100

Example:

Requests in past 5 min:
├─ 2XX: 49,500 successful
├─ 4XX: 300 client errors
├─ 5XX: 200 server errors
├─ Total: 50,000

Error rate:
├─ (300 + 200) / 50,000 × 100 = 1%
└─ This is high, should investigate

Alerting on Errors:

Alert 1: HTTPCode_ELB_5XX_Count > 0
├─ Severity: Critical
├─ Meaning: ALB itself generating errors
├─ Cause: No healthy backends, overload
├─ Action: Immediate investigation

Alert 2: HTTPCode_Target_5XX_Count spike
├─ Severity: High
├─ Meaning: Backends returning errors
├─ Cause: Application issue, backend crash
├─ Action: Check backend logs

Alert 3: Error rate > 1%
├─ Severity: Medium
├─ Meaning: Elevated error rate
├─ Investigation: Check what errors
└─ Action: Trending analysis
```

### Target Response Time

**Performance Metrics**

```
TargetResponseTime:

Definition:
├─ Measure: Time for backend to respond
├─ Unit: Seconds
├─ Scope: Response time at target
├─ Does NOT include: ALB processing time
├─ Does NOT include: Network latency (usually)
└─ Represents: Backend application latency

What It Shows:

Response Time = Time from ALB sending request to receiving full response

Example:
├─ T=0s: ALB sends request to backend
├─ T=0.1s: Backend receives request
├─ T=0.2s: Backend processes
├─ T=0.3s: Backend sends response
├─ T=0.35s: ALB receives full response
└─ Target Response Time: 0.35 seconds

Monitoring Importance:

Baseline Tracking:
├─ Measure: Normal response time for your app
├─ Example: Expect 100-200ms
├─ Track: p50, p90, p99 percentiles
└─ Alert: When exceeds threshold

Performance Degradation:

Alert Trigger:
├─ Threshold: 500ms (example)
├─ Current: 2 seconds
├─ Cause: Database slow, backend overloaded
├─ Action: Investigate performance

CloudWatch Statistics:

Available:
├─ Average: Mean response time
├─ Maximum: Worst case response time
├─ Minimum: Best case response time
├─ p50 (Median): 50th percentile
├─ p90: 90th percentile (good metric)
├─ p99: 99th percentile (tail latency)
└─ p99.9: Extreme outliers

Recommended Alerts:

Alert 1: p99 response time > 1 second
├─ Severity: Warning
├─ Meaning: Some slow requests
├─ Action: Monitor trend

Alert 2: p99 response time > 5 seconds
├─ Severity: High
├─ Meaning: Significant latency
├─ Action: Investigate performance

Alert 3: Average response time > normal + 50%
├─ Severity: Medium
├─ Example: Normally 200ms, now 300ms
├─ Action: Check load, capacity
```

### SurgeQueueLength and Spillover

**Capacity and Queuing Metrics**

```
SurgeQueueLength:

Definition:
├─ Count: Pending requests in queue
├─ Scope: Waiting for healthy target to become available
├─ Limit: Maximum 1000 requests
├─ Unit: Number of requests
└─ Duration: Time spent waiting

How It Works:

Scenario 1: Healthy Capacity
├─ Incoming requests: 100/sec
├─ Available targets: 10 healthy
├─ Requests per target: ~10
├─ Queue: Empty (0 queued)
├─ SurgeQueueLength: 0

Scenario 2: Approaching Capacity
├─ Incoming requests: 500/sec
├─ Available targets: 10 healthy
├─ Requests per target: ~50
├─ Queue: Forming (100 queued)
├─ SurgeQueueLength: 100

Scenario 3: Overloaded
├─ Incoming requests: 1000/sec
├─ Available targets: 10 healthy
├─ Requests per target: Would be 100 each
├─ Queue: Long queue building
├─ SurgeQueueLength: 500+

Monitoring Importance:

Why Track:
├─ Capacity indicator: When near limit
├─ Scale trigger: When to add capacity
├─ User experience: Queue growing = slower responses
├─ Cost: Each queued request consumes LB resources
└─ Scaling: Best metric for ASG policies

Alerts to Set:

Alert 1: SurgeQueueLength > 100
├─ Severity: Warning
├─ Meaning: Requests backing up
├─ Action: Investigate, monitor trend

Alert 2: SurgeQueueLength > 500
├─ Severity: High
├─ Meaning: Significant queue
├─ Action: Scale out immediately

Alert 3: Persistent SurgeQueueLength > 0
├─ Severity: Medium
├─ Meaning: Sustained overload
├─ Action: Increase baseline capacity

Spillover Count:

Definition:
├─ Count: Requests rejected when queue full
├─ Trigger: SurgeQueueLength = 1000 (max limit)
├─ New request arrives: Can't queue
├─ Result: Request dropped/rejected
├─ Response to client: Usually 503

What It Means:

If Spillover > 0:
├─ Severity: CRITICAL
├─ Meaning: Customers losing requests
├─ Impact: Lost transactions, poor UX
├─ Reason: LB at absolute limit
└─ Action: IMMEDIATE scale-out needed

Alert:

Alert: Spillover > 0
├─ Severity: Critical
├─ Response: Page on-call
├─ Action: Emergency scaling
└─ Investigation: Why so overloaded?

Prevention:

Best Practice:
├─ Monitor: RequestCountPerTarget
├─ Scale policy: Before SurgeQueueLength grows
├─ Target: Keep queue near 0
├─ Never reach: Spillover condition
└─ Strategy: Proactive scaling

Scaling Based on Queue:

Example Policy:
├─ Scale out: If RequestCountPerTarget > 70%
├─ Don't wait: For SurgeQueueLength to appear
├─ Reason: Preventive, not reactive
├─ Result: Smooth capacity handling
└─ Customer impact: Zero
```

### Active Connection Count

**Connection Metrics**

```
ActiveConnectionCount:

Definition:
├─ Count: Active TCP connections
├─ Scope: Between clients and load balancer
├─ Duration: Open and active
├─ Unit: Number of connections
└─ Typical: For long-lived connections

What It Shows:

Scenario 1: Web Traffic (Short Connections)
├─ Request: Comes in, handled, closed
├─ Duration: 100ms-1 second
├─ ActiveConnectionCount: Usually low
├─ Pattern: Spiky, follows request pattern

Scenario 2: WebSocket or gRPC (Long Connections)
├─ Connection: Established, stays open
├─ Duration: Minutes to hours
├─ ActiveConnectionCount: Large, stable
├─ Pattern: Ramps up as users connect

Monitoring Use:

Web Applications:
├─ Metric: Not as important
├─ Reason: Connections very short-lived
└─ Better metric: RequestCount

Long-Lived Services:
├─ Metric: Very important
├─ Reason: Track active connections
├─ Use: Scale based on connections
└─ Example: WebSocket applications

Practical Use:

Use Case:
├─ Service: Real-time chat application
├─ Connections: WebSocket, persistent
├─ Goal: Each user has 1 connection
├─ Capacity: Know max users per instance
├─ Scaling: Scale when connections high

Example:

Instance capacity: 10,000 concurrent connections
Instances: 3 healthy
Total capacity: 30,000 connections

Monitor:
├─ ActiveConnectionCount: Per instance
├─ When > 10,000: Add more instances
└─ Scale policy: Add when approaching limit

Consuming LB Capacity Units:

Definition:
├─ Measure: How much of LB capacity used
├─ Unit: LCU (Load Balancer Capacity Unit)
├─ Components: 
│  ├─ New connections per second
│  ├─ Active connections per minute
│  ├─ Processed bytes
│  ├─ Rule evaluations per second
│  └─ Processed bytes by ALB
│
├─ Pricing: Pay for consumed LCUs
└─ Monitoring: Track utilization

CloudWatch Metric:
├─ Metric: ConsumedLBCapacityUnits
├─ Shows: How much paying per hour
├─ Useful: Cost tracking
└─ Alert: If unexpectedly high
```

---

## Part 3: Troubleshooting with Metrics

### Common Issues and Metric Investigation

**Using Metrics to Diagnose Problems**

```
Issue 1: Spike in 503 Errors

Observable:
├─ Metric: HTTPCode_ELB_5XX_Count spiking
├─ Response: 503 Service Unavailable
├─ Timing: Sudden spike
└─ User impact: Services unavailable

Investigation:

Step 1: Check Host Health
├─ Metric: HealthyHostCount
├─ Question: Is it 0 or very low?
├─ If 0: No healthy backends!
└─ Conclusion: Backend issue

Step 2: Check Individual Targets
├─ Action: Look at target group
├─ Find: Which targets unhealthy?
├─ Check: Health check status
├─ Reason: Why marked unhealthy?

Step 3: Check Target Response Time
├─ If slow: Backends hung/overloaded
├─ If normal: Something else
├─ Metric: TargetResponseTime
└─ Conclusion: Performance issue

Root Cause:

Option A: All Backends Down
├─ HealthyHostCount = 0
├─ Solution: Start/recover instances
└─ Prevention: Better health checks

Option B: No Capacity
├─ SurgeQueueLength = 1000 (max)
├─ Spillover > 0
├─ Solution: Scale out
└─ Metric: RequestCount too high

Option C: Broken Instances
├─ HealthyHostCount very low
├─ Some healthy, some not
├─ Solution: fix unhealthy ones
└─ Investigation: logs/config

Issue 2: Elevated 4XX Errors

Observable:
├─ Metric: HTTPCode_Target_4XX_Count elevated
├─ Rate: Significantly above baseline
├─ Type: Client errors (400, 401, 403, etc.)
└─ User impact: Bad requests failing

Investigation:

Step 1: Which 4XX Codes?
├─ Access logs: Check specific codes
├─ Or: Parse logs for patterns
├─ Example: Mostly 400? Malformed requests
├─ Example: Mostly 401? Authentication issue

Step 2: Time Correlation
├─ When: Did errors start?
├─ What changed: Around that time?
├─ Deployment: New code?
├─ Configuration: ALB rules changed?
└─ Timeframe: Narrows down cause

Step 3: Check Specific Targets
├─ Are all targets: Returning 4XX?
├─ Or: Just specific instances?
├─ If all: Application/API issue
├─ If specific: Target-level issue (config, data)

Root Cause:

Option A: Client Bug
├─ Malformed requests from client
├─ Code change: Introduced bug
├─ Solution: Fix client application

Option B: API Change
├─ Endpoint: Changed or removed
├─ Version: API versioning issue
├─ Solution: Update client or rollback

Option C: Authentication Issue
├─ Surge in 401: Auth system issue
├─ Expired: Tokens or keys
├─ Solution: Fix auth system

Issue 3: High Response Latency

Observable:
├─ Metric: TargetResponseTime increasing
├─ p99: Rising significantly
├─ User: Complaints about slowness
└─ Trend: Gradual or sudden

Investigation:

Step 1: Check Load
├─ Metric: RequestCount or RequestCountPerTarget
├─ Question: More traffic than usual?
├─ If yes: Overload likely cause
└─ If no: Something else

Step 2: Check Backend Performance
├─ SSH to instance: Check top, free, iostat
├─ CPU: High usage?
├─ Memory: Running low?
├─ Disk I/O: High latency?
└─ Analysis: Where's the bottleneck?

Step 3: Check Dependencies
├─ Database: Performance?
├─ External APIs: Slow responses?
├─ Cache: Hits vs misses?
└─ Investigation: Trace specific requests

Step 4: Check Health Checks
├─ Timeout: Set too tight?
├─ Path: Expensive operation?
├─ Frequency: Too often?
└─ Effect: Health checks consuming resources?

Root Cause:

Option A: Traffic Spike
├─ Sudden increase in RequestCount
├─ Backends can't keep up
├─ Solution: Scale out or optimize app

Option B: Resource Exhaustion
├─ CPU maxed out
├─ Memory low
├─ Disk read/write slow
├─ Solution: Scale up (bigger instances) or out

Option C: Slow Dependency
├─ Database queries slow
├─ External API slow
├─ Cache misses high
├─ Solution: Fix dependency, not ALB

Option D: Code Issue
├─ Memory leak: Gradual slowdown
├─ Connection pool: Exhausted
├─ Lock contention: Multiple requests competing
├─ Solution: Debug and fix code

Issue 4: Unhealthy Hosts Not Recovering

Observable:
├─ Instance: Marked unhealthy
├─ Time: Remains unhealthy for long period
├─ Status: Never transitions to healthy
├─ Effect: Capacity reduced

Investigation:

Step 1: SSH to Instance
├─ Is it actually running?
├─ Is application process running?
├─ $ ps aux | grep application

Step 2: Test Health Check Path
├─ $ curl http://localhost:port/health-path -v
├─ Does endpoint exist?
├─ What status code returned?
├─ How long does it take?

Step 3: Check Logs
├─ Application logs: Any errors?
├─ System logs: Startup issues?
├─ Security: Errors or blocks?
└─ Diagnosis: Why failing health check?

Step 4: Verify Networking
├─ Security group: Allows port?
├─ Network ACLs: Allows traffic?
├─ Routing: Can reach instance?
└─ Test: telnet from LB

Root Cause:

Option A: Health Check Configuration Wrong
├─ Path doesn't exist
├─ Port incorrect
├─ Status code mismatch
├─ Solution: Fix health check config

Option B: Application Not Providing
├─ Endpoint not implemented
├─ Returns wrong status code
├─ Throws exception
├─ Solution: Fix application

Option C: Instance Issue
├─ Disk full: Can't write logs
├─ Memory exhausted
├─ Process crashed on startup
├─ Solution: Fix instance issue or terminate/replace
```

---

## Part 4: Access Logs and Analysis

### Enabling and Using Access Logs

**Access Log Configuration**

```
Access Logs Overview:

What They Are:
├─ Log type: Request-level logs
├─ Source: Every request to ALB
├─ Destination: Amazon S3
├─ Content: Request metadata
├─ Format: Text (tab or space delimited)
└─ Cost: Only S3 storage (no ALB cost)

What They Include:

Request Information:
├─ Timestamp: When request received
├─ Client IP address: Who made request
├─ Client port: Which port used
├─ Backend IP: Which instance processed
├─ Backend port: Which backend port
├─ Request path: What was requested
├─ HTTP method: GET, POST, etc.
└─ HTTP version: HTTP/1.1, HTTP/2, etc.

Response Information:
├─ Status code: 200, 404, 502, etc.
├─ Response size: Bytes returned
├─ Response time: When response sent
└─ Response data: Bytes sent to client

ALB-Specific Information:
├─ Trace ID: X-Amzn-Trace-Id
├─ SSL cipher: Encryption cipher used
├─ SSL protocol: TLS version used
├─ User agent: Client browser/application
└─ Referrer: Where request came from

Example Log Entry:

http 2023-12-15T10:30:45.123456Z app-alb-1 
  203.0.113.10:12345 203.0.113.50:80 0.100 0.200 0.050 
  200 200 34 1024 "GET http://example.com/api/users HTTP/1.1" 
  "Mozilla/5.0" - "-" arn:aws:elasticloadbalancing:... 
  "Root=1-12345678-abcdefghijklmnop" "-" "-" 0 2023-12-15T10:30:45.123456Z 
  "http" "10" "-" "-"

Log Fields (In Order):
1. type: http or https
2. timestamp: Request received time
3. elb: Load balancer name
4. client:port: Client IP and port
5. target:port: Backend IP and port
6. request_processing_time: Time ALB took
7. target_processing_time: Time backend took
8. response_processing_time: Time to send response
9. elb_status_code: ALB status (if error)
10. target_status_code: Backend status
11. received_bytes: Bytes received from client
12. sent_bytes: Bytes sent to client
... and more fields

Why Access Logs Useful:

Compliance:
├─ Audit: Track all access
├─ Logging: Required by regulations
├─ Data: Kept long-term in S3
├─ Protection: Encrypted by default
└─ Inquiry: Can answer "who accessed what"

Debugging:
├─ Specific requests: Trace single request
├─ Error analysis: Why did request fail
├─ Performance: Which requests slow
├─ Patterns: Identify trends
└─ Forensics: Post-mortem analysis

Capacity Planning:
├─ Traffic analysis: Request patterns
├─ Peak times: When busiest
├─ Client types: Browser vs API
├─ Bandwidth: How much data transferred
└─ Growth: Trend over time
```

### Enabling Access Logs

**Configuration Steps**

```
Step 1: Set Up S3 Bucket

Create Bucket:
├─ AWS Console: S3 service
├─ Create bucket: With unique name
├─ Example: my-alb-logs
├─ Region: Same as ALB (optional)
└─ Properties: Note bucket name/ARN

Enable ALB Account Access:

Different: By region
┌─ us-east-1: 127311923021
├─ us-east-2: 033677994240
├─ us-west-1: 027434742980
├─ us-west-2: 797873946194
├─ eu-west-1: 156460612806
├─ eu-central-1: 054676820928
├─ ap-southeast-1: 114774131450
└─ ap-northeast-1: 582318560864

Bucket Policy (Allow ALB to write):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::ACCOUNT-ID:root"
      },
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::my-alb-logs/*"
    }
  ]
}
```

Step 2: Enable Access Logs on ALB

In AWS Console:

1. Go to: Load Balancers
   ├─ Service: EC2 → Load Balancers
   ├─ Find: Your ALB
   └─ Click: To open details

2. Find: Attributes
   ├─ Section: Or "Edit Attributes"
   └─ Click: To edit

3. Find: Access logs
   ├─ Setting: Access logs
   ├─ Status: Probably disabled
   └─ Click: Enable

4. Configure S3 Destination:
   ├─ Bucket name: my-alb-logs
   ├─ Prefix: (optional) logs/
   │  └─ Example: logs/application/
   └─ Save: Changes

5. Save Configuration:
   ├─ Click: Save or Update
   ├─ Status: Processing
   └─ Wait: Few minutes

Logs Start Appearing:

Timeline:
├─ Configuration applied: Immediately
├─ First logs: May take 5-10 minutes
├─ Logs path: s3://my-alb-logs/AWSLogs/account-id/elasticloadbalancing/region/
├─ Subdirs: By date, hour
└─ Files: GZ compressed

File Structure:

S3 Path Example:
```
s3://my-alb-logs/AWSLogs/123456789/elasticloadbalancing/us-east-1/2023/12/15/
  123456789_elasticloadbalancing_us-east-1_app-alb-1_1234567890abcdef_1234567890.log.gz
```

Step 3: Query Logs with Athena

Set Up Athena:

1. Go to: Athena service
   └─ AWS Console: Athena

2. Create table:
   ├─ Define schema: For ALB logs
   ├─ SQL: Create table statement
   └─ Run: Creates table

3. Query logs:
   ├─ SQL: SELECT queries
   └─ Explore: Access log data

Example Query:

Count requests by status code:
```sql
SELECT 
  target_status_code,
  COUNT(*) as count
FROM alb_logs
WHERE Date >= '2023-12-15'
GROUP BY target_status_code
ORDER BY count DESC;
```

Find slow requests:
```sql
SELECT 
  timestamp,
  client_ip,
  request_url,
  target_processing_time
FROM alb_logs
WHERE target_processing_time > 1.0
  AND Date >= '2023-12-15'
ORDER BY target_processing_time DESC
LIMIT 100;
```

Find requests to specific path:
```sql
SELECT 
  timestamp,
  client_ip,
  request_url,
  target_status_code,
  sent_bytes
FROM alb_logs
WHERE request_url LIKE '%/api/users%'
  AND Date >= '2023-12-15'
LIMIT 1000;
```
```

---

## Part 5: Request Tracing

### X-Amzn-Trace-Id Header

**Distributed Request Tracing**

```
Request Tracing Overview:

What It Is:
├─ Header: X-Amzn-Trace-Id
├─ Added by: ALB automatically
├─ Purpose: Track request across systems
├─ Contains: Unique trace ID and timestamp
├─ Useful: Distributed tracing, debugging
└─ Format: Root=1-1234567890-abcdefghij...

How It Works:

Request Flow:

Client Request:
└─ No header initially

ALB Receives:
├─ ALB adds: X-Amzn-Trace-Id header
├─ Value: Unique ID for this request
└─ Forwards: To backend with header

Backend Apps Can:
├─ Extract: X-Amzn-Trace-Id
├─ Log: With this ID
├─ Pass: To downstream services
└─ Collect: Related requests

Benefits:

Tracking Single Request:
├─ Across: Multiple services
├─ Through: Different systems
├─ Logging: Find all related logs
├─ Debug: Reconstruct request flow
└─ Troubleshoot: Complex issues

Example:

User Request:
1. Client → ALB
   └─ ALB adds X-Amzn-Trace-Id: Root=1-ABC123...

2. ALB → Backend App
   └─ Header sent along

3. Backend calls Database
   └─ App logs: X-Amzn-Trace-Id

4. Backend calls Email Service
   └─ App logs: X-Amzn-Trace-Id

5. Troubleshooting:
   └─ Search logs: For X-Amzn-Trace-Id
   └─ Find: All related operations
   └─ Reconstruct: Complete flow

Using in Code:

Extract Header:

```python
# Flask example
from flask import request

trace_id = request.headers.get('X-Amzn-Trace-Id')
print(f"Trace ID: {trace_id}")
```

```javascript
// Node.js example
const traceId = req.headers['x-amzn-trace-id'];
console.log(`Trace ID: ${traceId}`);
```

Log with Trace ID:

```python
import logging
logger = logging.getLogger(__name__)

trace_id = request.headers.get('X-Amzn-Trace-Id', 'N/A')
logger.info(f"User login [TraceID: {trace_id}]")
```

Pass to Downstream:

```python
# When calling other service
import requests
headers = {
    'X-Amzn-Trace-Id': trace_id
}
response = requests.get('http://internal-api/users', headers=headers)
```

Integrations:

AWS X-Ray:
├─ Limited: ALB not fully integrated yet
├─ As of now: Can't directly see ALB segments in X-Ray
├─ But: Can use trace ID manually
└─ Future: More integration expected

Third-Party Tracing:

Can Use:
├─ Jaeger: Extract and use trace ID
├─ Zipkin: Extract and use trace ID
├─ DataDog: Can capture X-Amzn-Trace-Id
├─ New Relic: Can capture X-Amzn-Trace-Id
├─ Splunk: Parse and track
└─ Custom: Parse and log

Implementation:

Automatic Addition:
├─ ALB: Automatically adds header
├─ Always: Present in every request
├─ Format: Consistent
└─ No action: Needed for this

Manual Usage:

1. Expect: Header in all requests
2. Extract: In application code
3. Log: Include in all logs
4. Pass: To dependent services
5. Correlate: Find requests by ID

Example Log Analysis:

Given: X-Amzn-Trace-Id: Root=1-639D36FF-62C1E5D02E01E11A
├─ Root=: Always present
├─ 1-: Version/format
├─ 639D36FF: Epoch timestamp (hex)
├─ 62C1E5D02E01E11A: Random data
└─ Together: Unique request ID

Search ALL Logs:
├─ Query: All systems
├─ For: Root=1-639D36FF-62C1E5D02E01E11A
└─ Find: All operations for this request

Benefits:

Versus Without:
├─ Without: Hard to track single request
├─ Through: Multiple systems
├─ With: Easy correlation
├─ Find: All related data
└─ Debug: Much faster resolution

Note on X-Ray:

Current Status:
├─ ALB: Not yet deeply integrated
├─ X-Ray: Can't see ALB segments natively
├─ But: Trace ID available for manual use
├─ Future: AWS likely to add more
└─ Action: Check AWS docs for updates
```

---

## Part 6: Monitoring Dashboard Setup

### Creating Monitoring Dashboard

**CloudWatch Dashboard**

```
Dashboard Purpose:

Consolidate:
├─ Key metrics: From load balancer
├─ Health: Targets, instances
├─ Errors: Error codes, rates
├─ Performance: Response times, latency
├─ Capacity: Queue, connections
└─ At-a-glance: What's happening

Creating Dashboard:

Steps:

1. Go to CloudWatch:
   ├─ AWS Console: CloudWatch
   ├─ Service: CloudWatch
   └─ Region: Same as ALB

2. Create Dashboard:
   ├─ Dashboards: In left menu
   ├─ Click: Create Dashboard
   ├─ Name: My-ALB-Dashboard
   └─ Create: Dashboard

3. Add Widgets:

   Widget 1: HealthyHostCount
   ├─ Type: Number widget
   ├─ Metric: HealthyHostCount
   ├─ Dimension: Target group
   └─ Display: Current count

   Widget 2: UnHealthyHostCount
   ├─ Type: Number widget
   ├─ Metric: UnHealthyHostCount
   ├─ Dimension: Target group
   └─ Alert: Show if > 0

   Widget 3: Error Rate
   ├─ Type: Line chart
   ├─ Metrics: 4XX, 5XX counts
   ├─ Period: 5 minutes
   └─ Compare: Trends

   Widget 4: Response Time
   ├─ Type: Line chart
   ├─ Metric: TargetResponseTime
   ├─ Statistic: p99
   ├─ Period: 1 minute
   └─ Threshold: Add alarm line

   Widget 5: Request Count
   ├─ Type: Line chart
   ├─ Metric: RequestCount
   ├─ Period: 5 minutes
   └─ Show: Traffic load

   Widget 6: SurgeQueueLength
   ├─ Type: Line chart
   ├─ Metric: SurgeQueueLength
   ├─ Period: 1 minute
   ├─ Threshold: Alert at 500
   └─ Spillover: Alert at > 0

4. Save Dashboard:
   ├─ Name: Save dashboard
   ├─ View: Ready for monitoring
   └─ Refresh: Auto-refresh every minute (optional)

Recommended Metrics Summary:

Always Include:
├─ HealthyHostCount
├─ UnHealthyHostCount
├─ RequestCount
├─ HTTPCode_Target_5XX_Count
├─ TargetResponseTime (p99)
└─ SurgeQueueLength

Optional but Useful:
├─ ActiveConnectionCount
├─ BackendConnectionErrors
├─ ConsumedLBCapacityUnits
├─ RequestCountPerTarget
├─ HTTPCode_ELB_5XX_Count
└─ Spillover

Alarms to Set:

High Priority:
├─ HealthyHostCount = 0 (CRITICAL)
├─ HTTPCode_ELB_5XX_Count > 0 (CRITICAL)
├─ Spillover > 0 (CRITICAL)
└─ Response: Page on-call immediately

Medium Priority:
├─ UnHealthyHostCount > 0 (WARNING)
├─ SurgeQueueLength > 500 (WARNING)
├─ RequestCountPerTarget > threshold (WARNING)
└─ Response: Investigate within minutes

Low Priority:
├─ Error rate > 1% (INFO)
├─ Response time > baseline + 50% (INFO)
└─ Response: Monitor trend, adjust if growing
```

---

## Part 7: Exam Focus Points

### Key Exam Questions

```
1. "What does 502 Bad Gateway mean?"
   A) Client sent bad request
   B) Backend unreachable/unavailable
   C) Gateway timeout
   D) Client closed connection
   
   Answer: B) Backend unreachable/unavailable
   └─ 502: ALB can't reach backend
   └─ Usually: Port error, crashed app

2. "What does 503 Service Unavailable mean?"
   A) Backend timeout
   B) No healthy instances available
   C) Client error
   D) Gateway timeout
   
   Answer: B) No healthy instances available
   └─ 503: All targets unhealthy
   └─ Or: LB at capacity

3. "What does 504 Gateway Timeout mean?"
   A) Client took too long
   B) Health check timeout
   C) Backend took too long
   D) Connection refused
   
   Answer: C) Backend took too long
   └─ 504: Backend slow response
   └─ Check: Idle timeout, app performance

4. "Which metric tracks healthy instances?"
   A) HealthyHostCount
   B) InstanceHealth
   C) TargetCount
   D) ActiveConnections
   
   Answer: A) HealthyHostCount
   └─ Direct metric: Number of healthy targets
   └─ Important: Alert if 0

5. "If all targets unhealthy, what does ALB do?"
   A) Stop all traffic
   B) Return 503 to all requests
   C) Route to all targets (best-effort)
   D) Wait for recovery
   
   Answer: C) Route to all targets (best-effort)
   └─ Reason: Better than complete outage
   └─ Important: Exam trick question

6. "What does SurgeQueueLength represent?"
   A) Failed requests
   B) Pending requests in queue
   C) Total requests processed
   D) Error requests
   
   Answer: B) Pending requests in queue
   └─ Max: 1000 requests in queue
   └─ Spillover: When queue full

7. "What is X-Amzn-Trace-Id used for?"
   A) Security authentication
   B) Request tracing across services
   C) Load balancer identification
   D) Error tracking
   
   Answer: B) Request tracing across services
   └─ Header: Added automatically by ALB
   └─ Use: Distributed tracing, debugging

8. "Where are access logs stored?"
   A) CloudWatch Logs
   B) S3 bucket
   C) Application Insights
   D) EC2 instance
   
   Answer: B) S3 bucket
   └─ Cost: S3 storage only
   └─ Query: Use Athena for analysis
```

### Important Metrics Summary

```
Key Metrics to Remember:

Health:
├─ HealthyHostCount: Number of healthy targets
├─ UnHealthyHostCount: Number of unhealthy targets
└─ Alert on: Health count drops

Errors:
├─ 4XX errors: Client problems
├─ 5XX errors: Server problems
├─ Focus on: 502, 503, 504 from ALB
└─ Alert on: Any ALB-sourced 5XX

Performance:
├─ TargetResponseTime: Backend latency
├─ p99: Worst 1% of requests
└─ Investigate: If spikes

Capacity:
├─ RequestCountPerTarget: Avg requests per instance
├─ SurgeQueueLength: Pending requests
├─ Spillover: Rejected requests
└─ Action: Scale when queue builds

Related:
├─ BackendConnectionErrors: Connection issues
├─ ActiveConnectionCount: Live connections
└─ ConsumedLBCapacityUnits: Cost tracking
```

---

## Part 8: Best Practices

### Monitoring Best Practices

```
1. Set Up Dashboards:
   ├─ Centralized: View key metrics
   ├─ Real-time: 1-5 minute updates
   ├─ At-a-glance: Know system health
   └─ Share: With team for visibility

2. Create Meaningful Alarms:
   ├─ Critical: Page immediately
   ├─ Warning: Investigate within minutes
   ├─ Info: Monitor for trends
   └─ Routing: Send to appropriate team/person

3. Monitor Right Metrics:
   ├─ Start: Health and errors
   ├─ Add: Performance metrics
   ├─ Include: Capacity metrics
   └─ Evolve: Based on experience

4. Baseline Your Metrics:
   ├─ Understand: Normal values for your app
   ├─ Peak: What's normal peak?
   ├─ Off-peak: What's normal baseline?
   └─ Thresholds: Set based on baseline + 50%

5. Use Access Logs:
   ├─ Enable: For all production ALBs
   ├─ Retention: Keep 90+ days
   ├─ Analysis: Use Athena for queries
   └─ Cost: Worth investment for diagnostics

6. Implement Request Tracing:
   ├─ Extract: X-Amzn-Trace-Id in apps
   ├─ Log: Include in all logs
   ├─ Pass: To downstream services
   └─ Benefit: Faster debugging

7. Alert on Right Things:
   ├─ Don't: Alert on everything
   ├─ Do: Alert on actual problems
   ├─ Critical: "Cannot serve traffic"
   ├─ Warning: "Degradation starting"
   └─ Info: "Worth monitoring"

8. Test Alerting:
   ├─ Verify: Alerts actually fire
   ├─ Verify: Notification reaches team
   ├─ Verify: Team responds appropriately
   └─ Practice: Before incident occurs
```

---

## Part 9: Summary

### Complete Monitoring Checklist

```
Pre-Production Monitoring Setup:

Metrics Collection:
☐ CloudWatch metrics: Enabled
☐ Access logs: Enabled to S3
☐ S3 bucket: Created and configured
☐ IAM permissions: Correct for ALB
☐ Log retention: Policy set (90+ days)

CloudWatch Dashboard:
☐ Created: Custom dashboard
☐ Metrics included:
  ☐ HealthyHostCount
  ☐ UnHealthyHostCount
  ☐ RequestCount
  ☐ TargetResponseTime
  ☐ Error codes (4XX, 5XX)
  ☐ SurgeQueueLength
  ☐ RequestCountPerTarget
☐ Refresh: Set to auto-refresh
☐ Shared: With team

Alarms:
☐ Critical alarms: Created
  ☐ HealthyHostCount = 0
  ☐ HTTPCode_ELB_5XX_Count > 0
  ☐ Spillover > 0
☐ Warning alarms: Created
  ☐ UnHealthyHostCount > 0
  ☐ SurgeQueueLength > 500
  ☐ Response time spike
☐ Notifications: SNS topics configured
☐ Escalation: Routing defined
☐ Testing: Alarms tested

Access Logs:
☐ Enabled: On load balancer
☐ S3 bucket: Created
☐ Retention policy: Set
☐ Athena: Table created
☐ Sample queries: Tested

Request Tracing:
☐ Documented: X-Amzn-Trace-Id usage
☐ Applications: Extract header in code
☐ Logging: Include trace ID in logs
☐ Downstream: Pass header if calling others
☐ Process: Documented for troubleshooting

Training:
☐ Team: Trained on metrics
☐ Dashboards: Know how to navigate
☐ Alarms: Know what means what
☐ Access logs: Know how to query
☐ Troubleshooting: Follow documented procedures

Documentation:
☐ Dashboard: Screenshot and documented
☐ Baseline: Normal metrics documented
☐ Thresholds: Alert thresholds explained
☐ Escalation: Process documented
☐ Runbook: Troubleshooting procedures written

Post-Production:
☐ Monitor: First week closely
☐ Adjust: Alarms as needed
☐ Baseline: Refine based on actual data
☐ Review: Monthly metrics review
☐ Optimize: Improve over time

30-Day Review:
☐ Analyze: Month of metrics data
☐ Trending: Any concerning patterns?
☐ False alarms: Any? Tune them
☐ Missing alarms: Anything we didn't catch?
☐ Improvements: Process updates needed?
```

Load Balancer monitoring is now fully configured and ready for production operation!
