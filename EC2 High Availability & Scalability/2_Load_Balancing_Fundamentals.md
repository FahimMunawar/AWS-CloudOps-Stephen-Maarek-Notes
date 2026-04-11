# Load Balancing Fundamentals

This note covers the foundational concepts of load balancing, AWS Elastic Load Balancers, and their role in distributing traffic across multiple instances for high availability and scalability.

## Part 1: What is Load Balancing?

### Definition

**Load Balancer** = A server or set of servers that **forwards traffic** from clients to **multiple backend EC2 instances** or servers.

**Core Purpose**: Distribute incoming application traffic across multiple targets to optimize resource utilization, maximize throughput, minimize response time, and avoid overloading individual resources.

### Basic Load Balancing Architecture

```
User Requests
     ↓
  ┌──────────────────────────┐
  │ Elastic Load Balancer    │
  │ (Single Endpoint)        │
  └────────┬──────────────┬──┘
           │              │
      ┌────┴──┐      ┌────┴──┐
      ↓       ↓      ↓       ↓
   ┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
   │ EC2 │ │ EC2 │ │ EC2 │ │ EC2 │
   │ #1  │ │ #2  │ │ #3  │ │ #4  │
   └─────┘ └─────┘ └─────┘ └─────┘
```

### Load Distribution Example

**Scenario: Three Users Connect**

```
User 1 Connects:
└─ Load Balancer receives request
└─ Routes to EC2 Instance 1
└─ User gets single endpoint: load-balancer.aws.com

User 2 Connects:
└─ Load Balancer receives request
└─ Routes to EC2 Instance 2 (different from User 1)
└─ Same endpoint as User 1

User 3 Connects:
└─ Load Balancer receives request
└─ Routes to EC2 Instance 3
└─ Same endpoint as User 1 and User 2

Result:
├─ Single load balancer endpoint for all users
├─ Traffic distributed across 3 instances
├─ Instances hidden from direct user access
└─ Any instance can fail, load balancer routes to healthy ones
```

**Key Point**: Users connect to load balancer's single endpoint, not individual instances. Load balancer transparently distributes to backend instances.

## Part 2: Why Use Load Balancers?

### Benefits of Load Balancing

#### 1. Single Point of Access
- **Users connect to**: Load balancer's single endpoint (DNS name)
- **Users don't know**: Which backend instance they're connected to
- **Simplification**: One IP/DNS instead of multiple instance addresses
- **Benefit**: Can add/remove instances without changing user connection point

#### 2. Seamless Instance Failure Handling
- **Problem**: Individual instance failures would break user connections
- **Solution**: Load balancer detects failed instances
- **Result**: Failed instances automatically removed from distribution
- **Transparency**: Users don't experience connection loss (usually)

#### 3. Health Checks
- **Purpose**: Continuously verify instance health
- **Method**: Send periodic requests to instances
- **Detection**: Identify when instances become unhealthy
- **Action**: Automatically stop sending traffic to unhealthy instances

#### 4. SSL/TLS Termination
- **Use Case**: HTTPS encrypted traffic to websites
- **At Load Balancer**: Decrypts incoming HTTPS traffic
- **To Instances**: Sends HTTP traffic (decrypted)
- **Benefit**: Offloads encryption overhead from instances

#### 5. Enforce Stickiness with Cookies
- **Use Case**: Applications with session state
- **Method**: Load balancer remembers user→instance mapping
- **Implementation**: Uses cookies to maintain affinity
- **Benefit**: User requests go to same backend instance consistently

#### 6. High Availability Across Zones
- **Multi-AZ Support**: Load balancer spans multiple availability zones
- **Redundancy**: Survives single AZ failure
- **Active**: All backend instances active simultaneously
- **Result**: Zero-downtime failover

#### 7. Separate Public and Private Traffic
- **Public Tier**: Load balancer faces internet (public subnet)
- **Private Tier**: EC2 instances in private subnets
- **Security**: Instances never exposed to internet directly
- **Isolation**: Only load balancer handles public traffic

### Business Value
- **No Manual Intervention**: Automatic failover and recovery
- **Improved Reliability**: Handles instance/AZ failures
- **Cost Effective**: Pay per hour for service
- **Operational Simplicity**: AWS manages infrastructure

## Part 3: AWS Elastic Load Balancer (ELB)

### What is ELB?

**Elastic Load Balancer** = AWS's **managed load balancer** service that provides load balancing as a fully managed service.

### ELB Advantages

#### 1. Managed Service
- **AWS Responsibility**: Upgrades, maintenance, high availability
- **Your Responsibility**: Configuration, health check settings
- **No Downtime**: AWS handles updates transparently
- **Scalability**: Automatically scales to handle any traffic volume

#### 2. Cost Efficiency
- **Cheaper than DIY**: Less expensive than building your own load balancer
- **Operational Overhead**: No server management, no patching
- **Predictable Pricing**: Pay only for what you use
- **vs. Self-Managed**: Especially cost-effective at scale

#### 3. Nightmare Avoided
- **Self-Managed Problem**: Managing your own load balancer is complex
- **Scalability Issues**: Difficult to scale custom solution
- **Maintenance Burden**: Patches, updates, monitoring
- **AWS Solution**: Eliminates all these management tasks

#### 4. Deep AWS Integration
Load balancer integrates seamlessly with:
- **EC2 Instances**: Direct backend targets
- **Auto Scaling Groups**: Dynamic instance addition/removal
- **Amazon ECS**: Container orchestration
- **Certificate Manager**: SSL/TLS certificate management
- **CloudWatch**: Monitoring and metrics
- **Route 53**: DNS and health-aware routing
- **WAF**: Web Application Firewall protection
- **Global Accelerator**: Ultra-low latency routing
- **And more**: AWS services continuously added

## Part 4: Health Checks

### Purpose of Health Checks

**Health Checks** = Mechanism for load balancer to verify if backend instances are working properly.

**Critical Function**: Prevents sending traffic to broken or unhealthy instances.

### How Health Checks Work

```
Load Balancer Health Check Process:

Timer Trigger (Every 30 seconds)
    ↓
Send Health Check Request:
├─ Protocol: HTTP/TCP
├─ Port: Configured port (e.g., 80)
├─ Path: Health endpoint (e.g., /health)
└─ Request: GET /health HTTP/1.1

Instance Response:
├─ Success: HTTP 200 OK
│  └─ Instance marked: HEALTHY ✓
└─ Failure: Non-200 response or timeout
   └─ Instance marked: UNHEALTHY ✗

Traffic Routing:
├─ Healthy instances: Receive traffic
└─ Unhealthy instances: No traffic sent

Instance Recovery:
├─ Wait for next check
├─ If healthy again: Add back to rotation
└─ Automatic reintegration
```

### Health Check Configuration

**Example Configuration**
```
Protocol: HTTP
Port: 4567
Path: /health
Expected Status Code: 200
Interval: 30 seconds
Healthy: 2 consecutive successful checks
Unhealthy: 2 consecutive failed checks
```

**Interpretation**
- Check path `/health` on port 4567
- Expect HTTP 200 response
- Check every 30 seconds
- Require 2 passed checks before marking healthy
- Require 2 failed checks before marking unhealthy

### Application Health Endpoint Example

```python
# Flask application health endpoint
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/health', methods=['GET'])
def health():
    # Check application status
    if application_is_healthy():
        return jsonify({'status': 'healthy'}), 200
    else:
        return jsonify({'status': 'unhealthy'}), 500

def application_is_healthy():
    """Check if application is functioning correctly"""
    # Examples:
    # - Database connectivity
    # - Disk space available
    # - Memory usage acceptable
    # - Service dependencies running
    # - Configuration valid
    return True
```

### Health Check Best Practices

**Comprehensive Checks**
- Don't just return 200 OK
- Verify actual application functionality
- Check dependencies (database, cache, APIs)
- Validate configuration

**Avoid False Positives**
- Unhealthy instance returning 200 is dangerous
- Load balancer will keep sending traffic
- Causes cascading failures

**Timeout Configuration**
- Set reasonable timeout (3-5 seconds typical)
- Too short: False failures (slow responses)
- Too long: Delayed failure detection

---

## Part 5: Types of AWS Load Balancers

### Four Types of Managed Load Balancers

#### 1. Classic Load Balancer (CLB)

**Overview**
- **Generation**: V1 (2009, oldest)
- **Status**: Deprecated (still available but not recommended)
- **Name**: Classic Load Balancer

**Supported Protocols**
```
Layer 4 (Transport):
├─ TCP
└─ SSL/TLS (Secure TCP)

Layer 7 (Application):
├─ HTTP
└─ HTTPS (Encrypted HTTP)
```

**Characteristics**
- Ancient by cloud standards
- Limited features
- Still works but AWS discourages use
- Shown as deprecated in console

**When to Avoid**
- New deployments: Use ALB or NLB instead
- Feature Rich: CLB lacks modern capabilities
- Performance: Newer load balancers superior
- Support: AWS shifting away from CLB

#### 2. Application Load Balancer (ALB)

**Overview**
- **Generation**: V2 (2016, modern)
- **Name**: Application Load Balancer
- **Layer**: Layer 7 (Application)

**Supported Protocols**
```
Application Layer:
├─ HTTP
├─ HTTPS
└─ WebSocket (HTTP upgrade protocol)
```

**Best For**
- Web applications
- Microservices
- Container deployments
- API endpoints
- Modern web technologies

**Key Features**
- Host-based routing (different URL → different target)
- Path-based routing (/api → service A, /images → service B)
- Hostname routing
- Query string routing
- HTTP header routing
- Superior performance for HTTP/HTTPS

**Typical Use Case**
```
ALB Route Configuration:
├─ example.com/api → API Service (Target Group 1)
├─ example.com/web → Web Service (Target Group 2)
├─ images.example.com → Image Service (Target Group 3)
└─ api-v2.example.com → API v2 Service (Target Group 4)
```

#### 3. Network Load Balancer (NLB)

**Overview**
- **Generation**: V2 (2017, modern)
- **Name**: Network Load Balancer
- **Layer**: Layer 4 (Transport)

**Supported Protocols**
```
Transport Layer:
├─ TCP
├─ TLS (Secure TCP)
├─ UDP
└─ Security Composite Protocol (SCTP)
```

**Best For**
- Ultra-high performance applications
- Extreme throughput/low latency required
- Non-HTTP protocols
- Real-time applications
- Gaming servers
- IoT platforms
- Financial trading systems

**Key Features**
- Millions of requests per second
- Nanosecond latency
- Lower latency than ALB
- Can handle millions of concurrent connections
- Better throughput performance

**Typical Use Cases**
```
NLB Applications:
├─ Game servers (UDP traffic)
├─ MQTT brokers (custom TCP)
├─ Real-time data streams
├─ IP protocols other than HTTP
└─ Extreme performance requirements
```

#### 4. Gateway Load Balancer (GWLB)

**Overview**
- **Generation**: Latest (2020)
- **Name**: Gateway Load Balancer
- **Layer**: Layer 3 (Network)

**Supported Protocols**
```
Network Layer:
├─ IP (Internet Protocol)
└─ All IP-based protocols
```

**Purpose**
- Deploy, scale, manage network virtual appliances
- Centralized security/monitoring

**Use Cases**
- Firewalls
- Intrusion detection systems
- Deep packet inspection
- Network analytics
- Advanced security filtering

**Advanced Feature**: Typically not on basic SysOps exam

### Load Balancer Comparison

| Feature | CLB | ALB | NLB | GWLB |
|---------|-----|-----|-----|------|
| **Layer** | L4/L7 | L7 | L4 | L3 |
| **Protocols** | HTTP/HTTPS/TCP/SSL | HTTP/HTTPS/WebSocket | TCP/UDP/TLS/SCTP | IP |
| **Release Year** | 2009 | 2016 | 2017 | 2020 |
| **Status** | Deprecated | Recommended | Recommended | Advanced |
| **Use** | Legacy only | Web apps | High performance | Virtual appliances |
| **Latency** | Higher | Medium | Ultra-low | N/A |
| **Throughput** | Lower | Good | Extreme | N/A |
| **WebSocket** | No | Yes | No | No |
| **Advanced Routing** | Limited | Yes (path, host, header) | No | N/A |

**Recommendation**: Use ALB for most web applications, NLB for ultra-high performance needs.

## Part 6: Load Balancer Deployment Models

### External (Public) Load Balancer

**Use Case**: Public websites, internet-facing applications

```
Internet (Users at home, office, etc.)
        ↓
┌──────────────────────────────────┐
│ External Load Balancer           │
│ (IP: 203.0.113.50)              │
│ Internet-facing                 │
└──────────────────┬───────────────┘
        ↓          ↓          ↓
    [EC2-1]    [EC2-2]    [EC2-3]
    (Private)  (Private)  (Private)
```

**Characteristics**
- Accessible from internet
- Public IP address
- Public security group (0.0.0.0/0)
- Instances in private subnets (hidden)

### Internal (Private) Load Balancer

**Use Case**: Internal microservices, inter-tier communication

```
Public Tier (Web Servers)
    ↓  ↓  ↓
[EC2-Web-1] [EC2-Web-2] [EC2-Web-3]
    ↓        ↓        ↓
┌──────────────────────────────────┐
│ Internal Load Balancer           │
│ (IP: 10.0.1.50 - Private IP)    │
└──────────────────┬───────────────┘
        ↓          ↓          ↓
    [EC2-App-1] [EC2-App-2] [EC2-App-3]
    (Private)   (Private)   (Private)
```

**Characteristics**
- Only accessible within VPC
- Private IP address
- Private security group
- No internet access required

## Part 7: Security Configuration

### Security Group Strategy

#### Load Balancer Security Group

**Purpose**: Allow users to connect to load balancer publicly

**Configuration**
```
Inbound Rule #1:
├─ Protocol: TCP
├─ Port: 80 (HTTP)
├─ Source: 0.0.0.0/0 (anywhere)
└─ Allow public HTTP access

Inbound Rule #2:
├─ Protocol: TCP
├─ Port: 443 (HTTPS)
├─ Source: 0.0.0.0/0 (anywhere)
└─ Allow public HTTPS access

Outbound Rule:
└─ Destination: 0.0.0.0/0 (all)
```

**Security Implication**
- Load balancer accessible from anywhere
- This is intentional (for public applications)
- SSL/TLS encryption protects data in transit

#### Backend EC2 Instance Security Group

**Purpose**: Allow traffic ONLY from load balancer

**Configuration**
```
Inbound Rule #1:
├─ Protocol: TCP
├─ Port: 80 (HTTP)
├─ Source: sg-12345678 (Load Balancer SG)
└─ Allow traffic from LB only, NOT internet

Inbound Rule #2:
├─ Protocol: TCP
├─ Port: 443 (HTTPS)
├─ Source: sg-12345678 (Load Balancer SG)
└─ Allow traffic from LB only, NOT internet

Outbound Rule:
└─ Destination: 0.0.0.0/0 (all)
```

**Security Benefit**
- Instances not exposed to internet
- Only load balancer talks to instances
- Instances don't accept direct public connections
- Reference by security group = dynamic security

### Security Group Linking Example

```
┌─────────────────────────────────────────┐
│ Server Security Group (Load Balancer)   │
│ sg-alb12345                             │
└─────────────────────────────────────────┘

        Allow requests from:
        ↓
┌──────────────────────────────────────────┐
│ Instance Security Group (Backend EC2s)   │
│ sg-instance12345                         │
├──────────────────────────────────────────┤
│ Inbound: HTTP 80                         │
│ From: sg-alb12345 (the ALB's SG)         │
│                                          │
│ This means:                              │
│ "Only allow traffic if it comes FROM     │
│  something in the ALB security group"    │
└──────────────────────────────────────────┘
```

**Dynamic Security**
- If load balancer's source IP changes, rules still work
- Based on security group membership, not IP addresses
- Instances automatically adapt to load balancer changes

---

## Part 8: Load Balancer Integration Points

### AWS Services Integration

**EC2 Instances**
- Direct backend targets
- Primary use case
- Health checks via port/path

**Auto Scaling Groups**
- Automatic instance registration
- Dynamic target group updates
- Instances added/removed transparently

**Amazon ECS**
- Container orchestration
- Dynamic port mapping
- Service load balancing

**Certificate Manager**
- SSL/TLS certificate management
- HTTPS termination simplified
- Automatic certificate renewal

**CloudWatch**
- Metrics collection
- Performance monitoring
- Alarm triggers based on load

**Route 53**
- DNS health-aware routing
- Geolocation routing
- Failover to alternate endpoints

**WAF (Web Application Firewall)**
- Layer 7 attack protection
- SQL injection prevention
- DDoS mitigation

**Global Accelerator**
- Anycast routing
- Ultra-low latency globally
- Advanced traffic management

**More**: Integration points continue expanding

---

## Part 9: Key Takeaways for SysOps Associate

### Core Concepts
- **Load Balancer Purpose**: Distribute traffic across multiple backend instances
- **Single Endpoint**: Users connect to load balancer, not individual instances
- **Managed Service**: AWS handles maintenance, scaling, high availability
- **No Brainer**: Cost-effective vs. self-managed solution

### Health Checks
- **Purpose**: Verify instance health continuously
- **Method**: Send requests to configured port/path
- **Response**: Expect specific status code (usually 200)
- **Action**: Unhealthy instances automatically removed from distribution

### Load Balancer Types
1. **CLB**: Deprecated, avoid for new deployments
2. **ALB**: Layer 7, best for HTTP/HTTPS web applications (most common)
3. **NLB**: Layer 4, for extreme performance and non-HTTP protocols
4. **GWLB**: Network virtual appliances (advanced)

### Deployment Models
- **External/Public**: For internet-facing applications
- **Internal/Private**: For inter-tier communication within VPC

### Security Best Practice
- **Load Balancer SG**: Allow from 0.0.0.0/0 (ports 80, 443)
- **Instance SG**: Allow from load balancer SG only (restrictive)
- **Benefit**: Instances never exposed directly to internet

### Common Exam Questions
- Distinguish ALB (Layer 7, path/host routing) from NLB (Layer 4, extreme performance)
- Understand health check configuration and importance
- Know why load balancers are essential for scalability and HA
- Security group linking for restricting instance access
- Load balancer integration with Auto Scaling Groups

### Exam Focus Points
- Load balancers are essential for HA + Scalability combined
- ALB is default choice for typical web applications
- Health checks prevent traffic to unhealthy instances
- Security groups should restrict instance access to LB only
- Integration with AWS services is seamless