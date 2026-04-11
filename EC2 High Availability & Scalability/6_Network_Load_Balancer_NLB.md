# Network Load Balancer (NLB): Layer 4 High-Performance Load Balancing

Network Load Balancer is an AWS load balancer optimized for extreme performance and ultra-low latency, operating at OSI Layer 4 (Transport Layer) to handle millions of requests per second with TCP and UDP traffic.

---

## Part 1: NLB Fundamentals

### What is NLB?

**Definition**
```
Network Load Balancer (NLB)
├─ Layer: Layer 4 (Transport Layer)
├─ Protocols: TCP and UDP traffic
├─ Performance: Millions of requests per second
├─ Latency: Ultra-low (microseconds)
├─ Use Cases: Not HTTP/HTTPS specific
└─ Best For: Extreme performance, non-HTTP protocols, static IPs
```

**OSI Model Context**
```
Layer 7: Application (HTTP, HTTPS)       ← ALB operates here
├─ URL routing, host headers, paths
├─ Request/response aware
└─ Good for web applications

Layer 4: Transport (TCP, UDP)            ← NLB operates here
├─ Connection-based (TCP) or datagram (UDP)
├─ Protocol agnostic (any TCP/UDP app)
├─ Ultra-high performance
└─ Good for gaming, IoT, extreme throughput

Network layer below Layer 4
└─ IP addresses, routing, etc.
```

### When to Use NLB

**Use NLB When:**
```
✓ UDP traffic required
  - Gaming (multiplayer real-time)
  - IoT sensor data streams
  - DNS services
  - VoIP

✓ TCP but extreme performance needed
  - Financial trading systems
  - Real-time analytics
  - Non-HTTP protocols (custom TCP apps)
  - Millions of requests per second

✓ Static IP requirement
  - Firewall whitelist (specific IPs only)
  - DNS TTL issues (need fixed IP)
  - Regulatory/compliance requirement
  - Third-party integration with IP restrictions

✓ Extreme low latency
  - High-frequency trading
  - Online gaming
  - Real-time applications
  - Sub-millisecond response times
```

**Use ALB When:**
```
✓ HTTP/HTTPS traffic
✓ Path-based or hostname-based routing needed
✓ Request-aware rules (query strings, headers)
✓ REST APIs, web applications
✓ Performance good enough (thousands of req/sec)
```

---

## Part 2: NLB Key Characteristics

### Static IP per Availability Zone

**What This Means**
```
Traditional ALB Architecture:
├─ ALB IP changes dynamically
├─ Traffic routed via DNS name
├─ DNS record: demoalb-123456.us-east-1.elb.amazonaws.com
└─ Behind scenes: IP changes as ALB scales/recovers

NLB Architecture:
├─ ONE static IP per Availability Zone
├─ IP never changes
├─ Example:
│  ├─ AZ-1 (us-east-1a): 203.0.113.10 (static)
│  ├─ AZ-2 (us-east-1b): 203.0.113.20 (static)
│  └─ AZ-3 (us-east-1c): 203.0.113.30 (static)
└─ Traffic distributed across AZs
```

**Elastic IP Assignment**
```
NLB Configuration:
├─ Create NLB with one subnet per AZ
├─ Specify Elastic IP for each subnet
└─ Result: NLB has predictable static IP addresses

Benefits:
├─ Client firewall can whitelist exact IPs
├─ Load balancer address never changes
├─ No DNS resolution delays (IP direct)
├─ Works with IP-based ACLs and security policies
└─ Ideal for strict network requirements
```

### Performance Characteristics

**Throughput Capacity**
```
NLB Performance:
├─ Requests per second: Millions (ultra-high)
├─ Connections per second: Millions (TCP)
├─ Latency: Microseconds (ultra-low)
├─ Bandwidth: Up to 100+ Gbps
└─ Zero latency penalty vs direct connection

ALB Performance:
├─ Requests per second: Thousands (very good)
├─ Latency: Milliseconds (acceptable)
└─ Suitable for most HTTP applications

Comparison:
- NLB: 1,000,000+ req/sec, microsecond latency
- ALB: 10,000-100,000 req/sec, millisecond latency
```

**Why NLB is Faster**
```
NLB speed factors:
├─ Layer 4 processing only (no HTTP parsing)
├─ No URL inspection or routing logic
├─ Simple connection forwarding
├─ Minimal CPU overhead
├─ Hardware accelerated
└─ Direct TCP/UDP passthrough

ALB processing:
├─ Must parse HTTP headers
├─ URL inspection and routing decisions
├─ Cookie analysis
├─ Header transformations
└─ More CPU intensive
```

---

## Part 3: NLB Target Groups

### Target Group Basics

**Similar to ALB**
```
NLB uses target groups:
├─ Register backends to receive traffic
├─ Health checks to verify availability
├─ Port configuration
├─ Protocol selection (TCP or UDP)
└─ Listener forwards traffic to target group
```

### Target Group Types

#### Target Type 1: EC2 Instances

**Configuration**
```
Target Group Type: Instance
├─ Target: EC2 instance (by instance ID)
├─ Backend Port: Specify which port on instance
├─ Protocol: TCP or UDP
└─ Example:
   ├─ i-0a1b2c3d4e5f6g7h8 on port 3000 (TCP)
   └─ i-0x9y8z7w6v5u4t3s2 on port 5000 (UDP)
```

**Use Case**
- Your application runs on EC2 instances
- NLB forwards traffic directly to instance
- Instance receives raw TCP/UDP packets
- Most common NLB backend type

#### Target Type 2: IP Addresses

**Configuration**
```
Target Group Type: IP Address
├─ Target: Hardcoded private IP addresses
├─ Protocol: TCP or UDP
├─ Port: Backend receiving port
└─ Must be PRIVATE IP addresses
```

**Key Constraints**
- Must specify CIDR block (VPC where IPs exist)
  ```
  Example CIDR: 10.0.0.0/8 (internal VPC)
              172.16.0.0/12 (another network)
  ```
- IPs must be private (RFC 1918)
  ```
  Valid: 10.x.x.x, 172.16.x.x, 192.168.x.x
  Invalid: Public IPs
  ```
- IPs hardcoded (not automatic discovery)

**Why Use IP Target Type?**

**Scenario 1: Hybrid Cloud**
```
Your Architecture:
├─ AWS VPC: 10.0.0.0/8
└─ On-premises datacenter: 192.168.0.0/16
   └─ Connected via Direct Connect or VPN

NLB Target Group:
├─ Target Type: IP Address
├─ Targets:
│  ├─ 10.0.1.10 (AWS EC2 instance) - TCP port 8080
│  ├─ 10.0.1.20 (AWS EC2 instance) - TCP port 8080
│  └─ 192.168.100.5 (On-prem server) - TCP port 8080
│
└─ Effect: Single NLB load balances across AWS + on-prem

Use Case:
- Gradual cloud migration
- Hybrid infrastructure
- On-prem and cloud backends together
```

**Scenario 2: Kubernetes/Container Orchestration**
```
Kubernetes Cluster:
├─ Pods run on EC2 instances
├─ Pod IPs: 10.0.x.y (internal pod network)
├─ Pod IPs are dynamic (created/destroyed)
└─ But can register pod IPs with NLB

Benefits:
- Direct pod-to-NLB connection
- Kubernetes controller auto-updates target group
- Ultra-low latency (no instance hop)
- Fine-grained load balancing
```

**Scenario 3: Private Services**
```
Your Architecture:
└─ Multiple services in VPC
   ├─ Database cluster: 10.0.20.5
   ├─ Cache layer: 10.0.20.10
   └─ Processing: 10.0.20.15

NLB with IP target group:
└─ Can distribute traffic across these services
   └─ When they're not on EC2 instances
   └─ When you need extreme performance
```

#### Target Type 3: ALB (NLB → ALB Stacking)

**Architecture**
```
Internet
   ↓
NLB (Layer 4)
├─ Static IPs per AZ
├─ Extreme performance
└─ Ultra-low latency
   ↓
ALB (Layer 7)
├─ HTTP/HTTPS routing
├─ Path-based rules
└─ Hostname-based rules
   ↓
EC2 Instances / Target Groups
```

**Why Stack NLB → ALB?**

```
Goal: Combine NLB and ALB benefits
├─ NLB benefits:
│  ├─ Static IP per AZ (firewall whitelisting)
│  ├─ Ultra-high performance
│  └─ Multi-protocol support
│
└─ ALB benefits:
   ├─ HTTP/HTTPS sophisticated routing
   ├─ Path-based rules
   ├─ Hostname-based rules
   └─ Request inspection

Combined Architecture:
NLB (static IPs + performance) 
  → ALB (routing logic)
    → Target groups (EC2, Lambda, etc.)
```

**Configuration**
```
NLB Configuration:
├─ NLB Listener: TCP port 80
├─ Target Group:
│  ├─ Type: Instance (or IP)
│  ├─ Targets: ALB instance IDs
│  ├─ Protocol: TCP
│  └─ Port: ALB listening port (80, 443)
│
└─ Result: NLB forwards TCP to ALB

ALB Configuration (as target of NLB):
├─ Security group: Allow TCP from NLB SG
├─ Listeners: HTTP 80, HTTPS 443
├─ Rules: Define routing as normal
└─ Target groups: Point to actual backends
```

**Use Cases for NLB → ALB**
```
1. Firewall requirement for static IPs
   - Client firewall needs exact IPs
   - NLB provides static IPs
   - ALB provides routing flexibility

2. Non-HTTP + HTTP hybrid
   - NLB layer: TCP for other protocols
   - ALB layer: HTTP for web traffic

3. Extreme performance requirement
   - NLB: First line (ultra-fast)
   - ALB: Second line (sophisticated routing)

4. High availability + intelligent routing
   - NLB distribution across AZs
   - ALB distribution across backends
```

---

## Part 4: NLB Health Checks

### Health Check Protocols

**Supported Protocols**
```
NLB Health Checks:
├─ TCP protocol
│  ├─ Connects to backend TCP port
│  ├─ If connection succeeds: Healthy ✓
│  ├─ If connection fails: Unhealthy ✗
│  ├─ Simplest check
│  └─ No application-level verification
│
├─ HTTP protocol
│  ├─ Sends HTTP GET request
│  ├─ Checks for specific status code (200)
│  ├─ Checks response body (optional)
│  ├─ Application-level verification
│  └─ Can verify service actually works
│
└─ HTTPS protocol
   ├─ Sends HTTPS GET request
   ├─ Verifies SSL certificate
   ├─ Secure communication with backend
   └─ Application-level over encrypted connection
```

### TCP Health Check

**Configuration**
```
Health Check Settings:
├─ Protocol: TCP
├─ Port: Backend port to check (e.g., 3000)
├─ Interval: 30 seconds (check frequency)
├─ Timeout: 10 seconds (wait for response)
└─ Healthy threshold: 3 (consecutive successes to mark healthy)

How It Works:
1. NLB initiates TCP connection to 10.0.1.10:3000
2. If connection succeeds (SYN-ACK received):
   ├─ Increment healthy counter
   └─ If counter reaches 3: Mark as HEALTHY
3. If connection fails:
   ├─ Reset healthy counter
   └─ If failures reach threshold: Mark as UNHEALTHY
```

**When to Use TCP Checks**
```
✓ Backend is custom TCP application
  - Telemetry server
  - Game server
  - Real-time protocol
  
✓ Don't need HTTP verification
  - Just need TCP connectivity
  - Application handles itself

✓ High performance check needed
  - TCP check is minimal overhead
  - No HTTP parsing
```

**Limitation**
- TCP connection alone doesn't ensure app is working
- App could be listening but returning errors
- Can't check response content

### HTTP Health Check

**Configuration**
```
Health Check Settings:
├─ Protocol: HTTP
├─ Port: Backend port (usually 80 or app port, e.g., 3000)
├─ Path: URL path to check (e.g., /health or /status)
├─ Interval: 30 seconds
├─ Timeout: 6 seconds
├─ Healthy threshold: 3 (consecutive 2xx responses)
└─ Unhealthy threshold: 3 (consecutive non-2xx responses)

Example Configuration:
├─ Protocol: HTTP
├─ Port: 3000
├─ Path: /health
└─ Expected response: Status 200 OK
```

**How It Works**
```
1. NLB sends: GET /health HTTP/1.1 to 10.0.1.10:3000
2. Backend responds: 
   ├─ 200 OK → Healthy ✓
   ├─ 500 Internal Server Error → Unhealthy ✗
   ├─ 503 Service Unavailable → Unhealthy ✗
   └─ Connection timeout → Unhealthy ✗
3. If 3 consecutive successes: Mark HEALTHY
```

**Response Body Checking**
```
Optional: Check response body contains specific text
├─ Expected response:
│  ├─ Status: 200 OK
│  ├─ Body: "{ \"status\": \"healthy\" }"
│  └─ Match text: "healthy"
│
└─ NLB verifies body contains "healthy"
   ├─ If match: Healthy ✓
   └─ If no match: Unhealthy ✗
```

**When to Use HTTP Checks**
```
✓ Backend is HTTP service
  - Node.js app on port 3000
  - Python Flask on port 5000
  - Java Spring Boot on port 8080

✓ Need to verify application state
  - /health endpoint
  - /status endpoint
  - Any application-level endpoint

✓ Check response content
  - Verify specific JSON response
  - Ensure database connectivity
  - Verify cache availability
```

### HTTPS Health Check

**Configuration**
```
Health Check Settings:
├─ Protocol: HTTPS
├─ Port: Secure backend port (usually 443 or 8443)
├─ Path: URL path to check (e.g., /health)
├─ Interval: 30 seconds
├─ Timeout: 6 seconds
└─ Healthy threshold: 3

Example:
├─ Protocol: HTTPS
├─ Port: 8443
├─ Path: /health
├─ Expected response: 200 OK
└─ Certificate verification: Enabled
```

**How It Works**
```
1. NLB establishes HTTPS to 10.0.1.10:8443
2. SSL/TLS handshake (certificate verified)
3. Sends: GET /health HTTP/1.1
4. Backend responds (over SSL)
5. Status code checked (200 = healthy)
6. Connection closed

Security:
├─ Encrypted communication
├─ Certificate validation
├─ Backend identity verified
└─ Secure health check for sensitive apps
```

**When to Use HTTPS Checks**
```
✓ Backend requires encrypted communication
  - Sensitive data (credentials, PII)
  - Compliance requirement (PCI-DSS, HIPAA)

✓ Backend has HTTPS only (no HTTP)
  - Force HTTPS application

✓ TLS certificate validation needed
  - Ensure certificate chain valid
  - Detect certificate expiration
```

---

## Part 5: NLB vs ALB vs CLB Comparison

### Load Balancer Comparison Table

```
Feature                  CLB              ALB              NLB
─────────────────────────────────────────────────────────────────
Layer                    4 & 7 (hybrid)   7 (Application)  4 (Transport)
Protocols                TCP, SSL/TLS     HTTP, HTTPS      TCP, UDP
HTTP Traffic             ✓ Limited        ✓✓ Excellent     ✗ No
Request Routing          Connectionless   ✓ Path/Host      ✗ No
Requests/sec             100,000s         100,000s         Millions
Latency                  Moderate         ~10ms            Microseconds
Performance Tier         Legacy           Good             Extreme
Static IP per AZ         ✗ No             ✗ No             ✓ Yes (Elastic)
Sticky Sessions          ✓ Yes            ✓ Yes            ✓ Yes
Connection Draining      ✓ Yes            ✓ Yes            ✓ Yes
WebSocket Support        ✓ Basic          ✓✓ Full          ✓ Yes (TCP)
Use Case                 Legacy VPC/EC2   HTTP APIs/Web    High-perf/Gaming/IoT
AWS Recommendation       Deprecated       Recommended      For NLB use cases
Target Groups            ✗ No             ✓ Yes            ✓ Yes
IP Target Support        ✗ No             ✓ Yes            ✓ Yes

Retirement Status        DEPRECATED       Active           Active
New Projects             Don't use        First choice     When needed
```

### Decision Tree

```
Do you need static IP per AZ?
├─ YES → NLB
└─ NO → Continue below

Do you need UDP protocol?
├─ YES → NLB (only option)
└─ NO → Continue below

Is it HTTP/HTTPS traffic?
├─ YES → ALB (recommended)
├─ NO → NLB (TCP app)
└─ NO but need extreme perf → NLB

Do you need URL routing (path/host)?
├─ YES → ALB
└─ NO → Both NLB and ALB work

Performance requirement?
├─ Millions req/sec → NLB
├─ 100,000s req/sec → ALB
├─ Less → ALB (lower cost)
└─ Unknown → ALB (safe choice)

Latency requirement?
├─ Microseconds → NLB
├─ Milliseconds OK → ALB
└─ ALB is fine → ALB

Result:
├─ HTTP web app: ALB
├─ Real-time gaming: NLB (UDP)
├─ Financial system: NLB (extreme perf)
├─ High-traffic API: ALB or NLB based on protocol
├─ Custom TCP protocol: NLB
├─ Needs static IPs: NLB
└─ On-prem + cloud: NLB with IP targets
```

### Use Case Examples

**Use ALB**
```
- REST API (HTTP/HTTPS)
- Web application (HTML/CSS/JS)
- Microservices with path routing
- Multi-tenant SaaS
- Mobile backend
- Typical enterprise web apps

Typical Setup:
├─ ALB on port 80/443
├─ Path-based rules (/api, /admin, /assets)
├─ Target groups (EC2, ECS, Lambda)
└─ Access log for audit
```

**Use NLB**
```
- Multiplayer online game (UDP)
- IoT sensor platform (UDP)
- Extreme throughput (millions req/sec)
- Financial trading system (low latency)
- Non-HTTP protocol (custom TCP)
- Static IP requirement (firewall whitelist)
- On-premises + AWS hybrid

Typical Setup:
├─ NLB with Elastic IPs
├─ TCP targets (game servers, IoT gateway)
├─ Health checks (TCP or HTTP)
└─ Ultra-low latency
```

**Use CLB (Legacy)**
```
- Existing applications (not new deployments)
- Classic VPC architecture (pre-2013)
- Very simple load balancing needs

AVOID for new projects
```

---

## Part 6: NLB Architecture Patterns

### Pattern 1: Simple NLB with EC2 Instances

```
Diagram:
Internet
   ↓
NLB (Static IPs: 203.0.113.10, 203.0.113.20, 203.0.113.30)
├─ Listener: TCP 3000
├─ Target Group: demo-nlb-tg
│  └─ Type: Instance
│     ├─ i-1111: 10.0.1.10:3000
│     ├─ i-2222: 10.0.1.20:3000
│     └─ i-3333: 10.0.1.30:3000
└─ Health Check: TCP port 3000 every 30 seconds

Traffic Flow:
1. Client connects: 203.0.113.10:3000
2. NLB selects target (round-robin or flow hash)
3. NLB connects backend: 10.0.1.10:3000
4. Data flows directly (TCP passthrough)

Benefits:
- Ultra-low latency
- Extremely high throughput
- Static IP for firewall whitelisting
```

### Pattern 2: Hybrid Architecture (NLB with IP Targets)

```
Diagram:
┌─ AWS Region (us-east-1)
│  │
│  ├─ NLB (Static IPs)
│  │  ├─ Target Group: hybrid-targets
│  │  │  ├─ 10.0.1.10 (AWS EC2)
│  │  │  ├─ 10.0.1.20 (AWS EC2)
│  │  │  └─ (Cloud targets)
│  │  │
│  │  └─ Connected via Target Group IP
│  │
│  └─ Target Group Type: IP Address
│
└─ On-Premises Datacenter
   │
   ├─ Direct Connect (AWS VPN)
   │  └─ Tunnel to VPC
   │
   └─ Servers: 192.168.100.x
      ├─ 192.168.100.5 (registered as NLB target)
      └─ 192.168.100.6 (registered as NLB target)

NLB Target Group:
├─ Type: IP Address
├─ CIDR: 10.0.0.0/8 (AWS) + 192.168.0.0/16 (On-prem)
└─ Targets:
   ├─ 10.0.1.10 (AWS EC2) ✓
   ├─ 10.0.1.20 (AWS EC2) ✓
   ├─ 192.168.100.5 (On-prem) ✓
   └─ 192.168.100.6 (On-prem) ✓

Traffic Distribution:
- NLB load balances across all 4 backends
- Seamless AWS + on-prem coordination
- Perfect for cloud migration scenarios
```

### Pattern 3: NLB → ALB Stacking

```
Diagram:
Internet (Fixed IPs via firewall whitelist)
   ↓
NLB (Layer 4 - Static IPs per AZ)
├─ IP AZ-1: 203.0.113.10 → ALB-1
├─ IP AZ-2: 203.0.113.20 → ALB-2
└─ IP AZ-3: 203.0.113.30 → ALB-3
   ↓
ALB (Layer 7 - HTTP/HTTPS with routing)
├─ Host Header Routing:
│  ├─ api.example.com → API Backend
│  ├─ admin.example.com → Admin Dashboard
│  └─ web.example.com → Web Servers
   ↓
Target Groups (EC2, ECS, Lambda)
├─ API Servers (3 instances)
├─ Admin Servers (2 instances)
└─ Web Servers (5 instances)

Configuration:
NLB:
├─ Listener: TCP 80
├─ Target Group:
│  ├─ Type: Instance or IP
│  └─ Targets: ALB instances (or ALB IP addresses)
└─ Health Check: HTTP GET /health

ALB:
├─ Listener: HTTP 80
├─ Rules: Host-based routing
└─ Target Groups: Actual backends

Benefits:
├─ Static IPs (NLB)
├─ HTTP routing (ALB)
├─ Ultimate flexibility
└─ Both performance and sophistication
```

---

## Part 7: NLB Health Check Best Practices

### Health Check Configuration

**Recommended Settings**
```
For Production Environment:

TCP Health Check:
├─ Interval: 30 seconds (standard)
├─ Timeout: 10 seconds (wait for response)
├─ Healthy threshold: 3 (mark healthy after 3 successes)
└─ Unhealthy threshold: 3 (mark unhealthy after 3 failures)

Effect: Takes ~90 seconds to detect failure (9 checks)

HTTP Health Check:
├─ Interval: 30 seconds
├─ Timeout: 6 seconds
├─ Healthy threshold: 3
├─ Unhealthy threshold: 3
├─ Path: /health (custom endpoint)
└─ Matcher: 200 (expected status code)
```

**Quick Detection (Aggressive)**
```
When you want fast failure detection:

TCP Health Check:
├─ Interval: 5 seconds
├─ Timeout: 3 seconds
├─ Healthy threshold: 1
└─ Unhealthy threshold: 1

Effect: ~5 seconds detection time
Downside: False positives possible

Use case: Critical systems where downtime detection is vital
```

### Health Endpoint Best Practices

**Implement /health Endpoint**
```
Backend Application (Node.js example):

app.get('/health', (req, res) => {
  // Check critical dependencies
  const checks = {
    database: await checkDatabase(),
    cache: await checkCache(),
    external_api: await checkExternalAPI(),
  };
  
  // Determine overall health
  const isHealthy = Object.values(checks).every(c => c === true);
  
  if (isHealthy) {
    res.status(200).json({ status: 'healthy', checks });
  } else {
    res.status(503).json({ status: 'unhealthy', checks });
  }
});
```

**What /health Should Check**
```
✓ Database connectivity
  - Can query database
  - Connection pool available
  
✓ Cache dependency
  - Redis/Memcached available
  - Cache hits working
  
✓ External API dependency
  - Third-party service responding
  - API not rate limited
  
✓ Local application state
  - Application threads alive
  - Memory usage normal
  - No stuck processes

✗ Don't check
  - Disk space (might be temporary)
  - CPU load (might be temporary spike)
  - Time of day (not relevant)
```

---

## Part 8: NLB Configuration Walkthrough

### Creating an NLB (High-Level Steps)

**Step 1: Access Load Balancing Console**
```
AWS Console → EC2 → Load Balancing → Load Balancers
```

**Step 2: Create Load Balancer**
```
Click "Create load balancer" → Select "Network Load Balancer"
```

**Step 3: Configure NLB**
```
Basic Configuration:
├─ Name: MyNLB
├─ Scheme: Internet-facing (public access)
└─ IP address type: IPv4

Network Mapping:
├─ VPC: (select your VPC)
├─ Subnets: (select AZs)
│  ├─ Availability Zone 1: subnet-1a
│  ├─ Availability Zone 2: subnet-1b
│  └─ Availability Zone 3: subnet-1c
```

**Step 4: Assign Elastic IPs**
```
For each AZ/Subnet:
├─ Allocate Elastic IP
├─ Assign to NLB subnet
└─ Result: One static IP per AZ

Example After Assignment:
├─ AZ-1 (us-east-1a): 203.0.113.10 (static)
├─ AZ-2 (us-east-1b): 203.0.113.20 (static)
└─ AZ-3 (us-east-1c): 203.0.113.30 (static)
```

**Step 5: Create Target Group**
```
Target Group Configuration:
├─ Name: demo-nlb-tg
├─ Protocol: TCP
├─ Port: 3000
├─ Target type: Instance
├─ VPC: (select VPC)

Health Check Settings:
├─ Protocol: TCP (or HTTP)
├─ Port: traffic port (3000)
├─ Interval: 30
├─ Timeout: 10
├─ Healthy threshold: 3
└─ Unhealthy threshold: 3

Register Targets:
├─ Select instances to add
├─ Instances added to target group
└─ Health checks begin automatically
```

**Step 6: Create Listener**
```
Listener Configuration:
├─ Listener Name: TCP-Listener
├─ Protocol: TCP
├─ Port: 3000
├─ Default Action: Forward to → demo-nlb-tg
└─ Create listener

Result:
- NLB listens on TCP 3000
- Forwards traffic to target group
- Distributes across healthy targets
```

**Step 7: Review and Create**
```
Review All Configuration:
├─ NLB name and network
├─ Subnets and Elastic IPs
├─ Target group and health checks
├─ Listeners and default actions
└─ Click "Create load balancer"

NLB Created:
├─ DNS name generated (but not needed - use IPs)
├─ Status: Active
├─ Targets: Initializing health checks
├─ Healthy targets: Updates over 30 seconds
└─ Ready to receive traffic
```

---

## Part 9: Key Takeaways for SysOps Associate

### When to Use NLB

**Use NLB for:**
```
✓ UDP traffic (think NLB instantly)
✓ TCP but extremely high performance
✓ Need static IP per availability zone
✓ Elastic IP requirement for firewall whitelisting
✓ Gaming applications (multiplayer real-time)
✓ IoT applications (massive throughput)
✓ Non-HTTP protocols (custom TCP apps)
```

**Characteristics to Remember**
```
- Layer 4 (TCP/UDP) not HTTP
- Millions of requests per second
- Microsecond latency
- One static IP per AZ
- Can assign Elastic IPs
- Target groups (EC2, IP, ALB)
- Health checks: TCP, HTTP, HTTPS
```

### Performance vs ALB

```
NLB:
├─ Extreme: Millions req/sec
├─ Ultra-low: Microsecond latency
├─ Limited: Non-HTTP only
└─ Best for: Performance-critical, non-HTTP

ALB:
├─ Excellent: 100,000s req/sec
├─ Acceptable: ~10ms latency
├─ Full: HTTP/HTTPS routing
└─ Best for: Web applications, APIs
```

### Hybrid Architecture

```
NLB → ALB is valid combination:
├─ Provides: Firewall-friendly static IPs (NLB)
├─ Provides: Sophisticated HTTP routing (ALB)
├─ Use when: Need both performance + routing control
```

### Health Check Protocols

```
TCP: Fastest, simplest
├─ Just checks connection
└─ Sufficient for custom TCP apps

HTTP: Application-level verification
├─ Can check /health endpoint
├─ Verifies app is functioning
└─ Good for HTTP-capable backends

HTTPS: Secure application verification
├─ Encrypted communication
├─ Certificate validation
└─ For sensitive/compliance-required apps
```

### Exam Focus Points

- **NLB at Layer 4**: Handles TCP/UDP, not HTTP/HTTPS
- **Static IPs crucial**: One per AZ, can be Elastic IPs
- **Performance metric**: Millions of requests per second
- **UDP = NLB**: Only NLB supports UDP
- **Target types**: EC2 instances, private IP addresses, ALB
- **Health checks**: TCP, HTTP, or HTTPS protocols
- **When exam mentions**: Firewall whitelist → Think NLB (static IPs)
- **When exam mentions**: Gaming/IoT → Think NLB
- **When exam mentions**: UDP → Think NLB (only option)
- **Static IP + HTTP routing**: NLB → ALB architecture