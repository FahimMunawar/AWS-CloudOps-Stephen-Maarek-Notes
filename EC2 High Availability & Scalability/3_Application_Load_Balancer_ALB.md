# Application Load Balancer (ALB)

This note covers the Application Load Balancer, a Layer 7 load balancer designed for modern web applications, microservices, and containers. ALBs are the recommended choice for most HTTP/HTTPS applications on AWS.

## Part 1: Application Load Balancer Overview

### Key Characteristics

**Layer 7 Only**
- **OSI Layer**: Application layer (HTTP/HTTPS only)
- **Protocols Supported**:
  - HTTP/1.1, HTTP/2
  - HTTPS (encrypted HTTP)
  - WebSocket (HTTP upgrade)
- **Not Supported**: TCP, UDP, or any non-HTTP protocols

**Intelligent Routing**
- Routes to **multiple HTTP applications** across different machines
- Applications grouped in **Target Groups**
- Routing decisions based on Layer 7 data (URLs, hostnames, headers)
- Can route multiple applications on same EC2 instance

**Modern Application Support**
- Microservices architecture friendly
- Container-based applications (Docker, ECS)
- Lambda functions backend
- On-premises servers via private IPs

### Advantages Over CLB

**Single ALB vs. Multiple CLBs**

```
Classic Load Balancer Approach (Outdated):
├─ CLB #1: Application 1
├─ CLB #2: Application 2
├─ CLB #3: Application 3
└─ Problem: Multiple load balancers, higher cost, complex management

Application Load Balancer Approach (Modern):
└─ ALB: All applications
   ├─ Route to Application 1 via /app1
   ├─ Route to Application 2 via /app2
   └─ Route to Application 3 via /app3
   └─ Benefit: Single load balancer, intelligent routing
```

---

## Part 2: ALB Routing Capabilities

### Overview of Routing Types

ALBs support advanced routing based on Layer 7 data:

```
Incoming Request to ALB
    ↓
Evaluate Routing Rules:
├─ Path-based routing
├─ Hostname-based routing
├─ Query string/parameter routing
├─ HTTP header routing
└─ Custom conditions
    ↓
Route to appropriate Target Group
    ↓
Forward to backend instances in that group
```

### 1. Path-Based Routing

**Use Case**: Different endpoints serve different applications

**Example Setup**
```
ALB receives: example.com

Routing Rules:
├─ URL path /users → Target Group 1 (User Service)
├─ URL path /posts → Target Group 2 (Post Service)
├─ URL path /search → Target Group 3 (Search Service)
└─ Default path / → Target Group 4 (Home Service)

Examples:
├─ example.com/users → User Service
├─ example.com/posts → Post Service
├─ example.com/search → Search Service
└─ example.com/ → Home Service
```

**Real-World Scenario**
```
ALB: my-app.example.com

Target Group Rules:
├─ /api/v1/* → API Backend Services
├─ /images/* → Image Processing Service
├─ /admin/* → Admin Dashboard Service
└─ /* → Web Frontend Service
```

**Configuration**
```
Listener Rule 1:
├─ If path is /users
└─ Then forward to → UserService-TG

Listener Rule 2:
├─ If path is /posts
└─ Then forward to → PostService-TG

Listener Rule 3:
├─ Default (no match)
└─ Then forward to → DefaultService-TG
```

### 2. Hostname-Based Routing

**Use Case**: Different hostnames serve different applications

**Example Setup**
```
ALB: Receives all requests for *.example.com

Routing Rules:
├─ Host: one.example.com → Target Group 1
├─ Host: other.example.com → Target Group 2
├─ Host: api.example.com → Target Group 3
└─ Host: www.example.com → Target Group 4

Examples:
├─ one.example.com → Application 1
├─ other.example.com → Application 2
├─ api.example.com → API Service
└─ www.example.com → Website
```

**Real-World Scenario: Multi-Tenant SaaS**
```
Single ALB, Multiple Customers:
├─ customer1.app.example.com → Tenant 1 Service
├─ customer2.app.example.com → Tenant 2 Service
└─ customer3.app.example.com → Tenant 3 Service

Benefit: One ALB serves all tenants, routes to appropriate backend
```

**Configuration**
```
Listener Rule 1:
├─ If hostname is one.example.com
└─ Then forward to → App1-TG

Listener Rule 2:
├─ If hostname is other.example.com
└─ Then forward to → App2-TG
```

### 3. Query String and Parameter Routing

**Use Case**: Route based on URL query parameters

**Example Setup**
```
ALB: my-app.example.com

Routing Rules:
├─ URL contains ?Platform=Mobile → Target Group 1 (Mobile App)
├─ URL contains ?Platform=Desktop → Target Group 2 (Desktop App)
├─ URL contains ?version=v2 → Target Group 3 (New Version)
└─ URL contains ?version=v1 → Target Group 4 (Legacy Version)

Examples:
├─ my-app.example.com/app?Platform=Mobile → Mobile Service
├─ my-app.example.com/app?Platform=Desktop → Desktop Service
├─ my-app.example.com/api?version=v2 → API v2
└─ my-app.example.com/api?version=v1 → API v1
```

**Real-World Scenario**
```
AB Testing with ALB:
├─ example.com/product?test_group=A → Version A
├─ example.com/product?test_group=B → Version B
└─ example.com/product → Control/Default

Traffic split intelligently based on query parameters
```

**Configuration**
```
Listener Rule 1:
├─ If query parameter Platform=Mobile
└─ Then forward to → MobileService-TG

Listener Rule 2:
├─ If query parameter Platform=Desktop
└─ Then forward to → DesktopService-TG
```

### 4. HTTP Header-Based Routing

**Use Case**: Route based on HTTP headers in request

**Example Setup**
```
ALB: api.example.com

Routing Rules:
├─ Header: X-Client-Type: Premium → Target Group 1 (Premium API)
├─ Header: X-Client-Type: Free → Target Group 2 (Free API)
├─ Header: User-Agent: *Mobile* → Target Group 3 (Mobile)
└─ Header: X-API-Version: v2 → Target Group 4 (API v2)

Examples:
├─ Request with X-Client-Type: Premium → Premium API Service
├─ Request with X-Client-Type: Free → Free API Service
└─ Mobile User-Agent → Mobile-optimized Service
```

**Real-World Scenario: API Rate Limiting**
```
Different SLAs based on customer tier:
├─ Header: X-Tier: Gold → Premium Service (no limits)
├─ Header: X-Tier: Silver → Standard Service (rate limited)
└─ No header → Free Service (strict limits)
```

**Configuration**
```
Listener Rule 1:
├─ If header X-Client-Type equals Premium
└─ Then forward to → PremiumAPI-TG

Listener Rule 2:
├─ If header X-Client-Type equals Free
└─ Then forward to → FreeAPI-TG
```

### Combining Multiple Routing Rules

**Advanced Example: Microservices Architecture**
```
Single ALB: myapp.example.com

Rule 1: Mobile API v2
├─ Path: /api/v2/*
├─ AND Header: User-Agent contains Mobile
└─ → Target Group: Mobile-API-v2-TG

Rule 2: Desktop API v2
├─ Path: /api/v2/*
├─ AND Header: User-Agent contains Windows/Mac
└─ → Target Group: Desktop-API-v2-TG

Rule 3: Legacy API v1
├─ Path: /api/v1/*
└─ → Target Group: API-v1-TG

Rule 4: Admin Panel
├─ Hostname: admin.myapp.example.com
└─ → Target Group: Admin-Dashboard-TG

Rule 5: Website
├─ Default (all other)
└─ → Target Group: Website-TG
```

---

## Part 3: Target Groups

### What is a Target Group?

**Target Group** = A logical grouping of backend resources that an ALB can route traffic to.

**Characteristics**
- Multiple resources can be in one target group
- Resources share same configuration
- Independent health checks per target group
- Load balancing within group (round-robin by default)

### Supported Target Types

#### 1. EC2 Instances

**Configuration**
```
Target Group: WebServer-TG
├─ Instance 1: i-0123456789abcdef0
├─ Instance 2: i-1123456789abcdef0
├─ Instance 3: i-2123456789abcdef0
├─ Health Check: Port 80, path /health
└─ Load Balancing: Round-robin across instances
```

**Use Case**: Traditional web servers on EC2

#### 2. Auto Scaling Group (ASG)

**Configuration**
```
Target Group: API-Backend-TG
└─ Associated with: Auto Scaling Group (api-asg)
   ├─ Min instances: 2
   ├─ Max instances: 10
   └─ Instances automatically registered/deregistered
```

**Benefit**: Instances dynamically added/removed as ASG scales

#### 3. ECS Tasks

**Configuration**
```
Target Group: ECS-Service-TG
└─ Associated with: ECS Service
   ├─ Task Definition: my-app-v1
   ├─ Desired Count: 5
   ├─ Launch Type: FARGATE
   └─ Dynamic port mapping: ALB routes to random ports
```

**Use Case**: Container deployments, microservices

#### 4. Lambda Functions

**Configuration**
```
Target Group: Lambda-Service-TG
├─ Target Type: Lambda
├─ Lambda Function: myapp-api-handler
├─ Payload Format: AWS_PROXY or REST
└─ Timeout: 30 seconds
```

**Use Case**: Serverless APIs, event-driven functions

#### 5. IP Addresses (Private)

**Configuration**
```
Target Group: OnPrem-Service-TG
├─ Target Type: IP
├─ IP Address 1: 192.168.1.10 (DC datacenter)
├─ IP Address 2: 192.168.1.11 (DC datacenter)
└─ Port: 8080
```

**Use Case**: On-premises servers, hybrid cloud, private networks

**Important**: Must be **private IP addresses** (not public)

### Target Group Example: Hybrid Setup

**Scenario**: Mix of AWS and On-Premises Resources

```
ALB: integration.company.com

Target Group 1: Cloud-Services-TG
├─ EC2 Instance 1 (AWS): i-xyz123
├─ EC2 Instance 2 (AWS): i-abc456
├─ ECS Task 1 (AWS): on port 32768
└─ Health Check: /health

Target Group 2: OnPrem-Services-TG
├─ On-Prem Server 1: 192.168.1.100:8080
├─ On-Prem Server 2: 192.168.1.101:8080
└─ Health Check: /status

Routing Rules:
├─ Path /cloud/* → Cloud-Services-TG
└─ Path /onprem/* → OnPrem-Services-TG
```

---

## Part 4: Practical Routing Example

### Real-World Scenario: Platform-Based Routing

**Setup**: Single E-commerce Application

```
Requirement: Route mobile and desktop users to different backends

ALB: shop.example.com

Request 1: shop.example.com/app?Platform=Mobile
├─ Query Parameter: Platform=Mobile detected
├─ ALB Rule: If Platform=Mobile → Mobile-Backend-TG
├─ Route to: Mobile-optimized Application
└─ Service: Shows mobile UI, optimized performance

Request 2: shop.example.com/app?Platform=Desktop
├─ Query Parameter: Platform=Desktop detected
├─ ALB Rule: If Platform=Desktop → Desktop-Backend-TG
├─ Route to: Desktop Application
└─ Service: Shows desktop UI, full features

Request 3: shop.example.com/admin
├─ Path: /admin detected
├─ ALB Rule: If path=/admin → Admin-Backend-TG
├─ Route to: Admin Dashboard
└─ Service: Admin panel (different instance group)
```

**Benefits**
- Single ALB manages all routing
- Different backend services optimized for each platform
- Flexible without any code changes on backend
- Easy to add new routing rules

---

## Part 5: ALB Fixed Hostname

### Fixed Hostname Feature

**Concept**: ALB provides a consistent DNS name

**Example**
```
ALB Name: my-app-alb
AWS Generated Hostname: my-app-alb-123456789.us-east-1.elb.amazonaws.com
```

**Characteristics**
- **Auto-Generated**: AWS provides unique hostname
- **Fixed**: Does not change unless ALB recreated
- **DNS Name**: Publicly resolvable (for external ALBs)
- **CNAME**: Can create custom CNAME in Route 53
- **Users Access**: Via this hostname, not instance IPs

**Usage Example**
```
DNS Configuration:
└─ Create CNAME: shop.example.com
   └─ Points to: my-app-alb-123456789.us-east-1.elb.amazonaws.com

Users access: shop.example.com
└─ Resolves to: ALB's fixed hostname
└─ Load balancer handles distribution
```

**Benefit**: Users access via friendly domain, internal routing handled by ALB

---

## Part 6: Client IP Handling

### Problem: Client IP Visibility

**Direct Connection** (Without Load Balancer)
```
Client: 12.34.56.78
    ↓
EC2 Instance
└─ Request from: 12.34.56.78 ✓
└─ Direct client IP visible to application
```

**With Load Balancer** (Connection Termination)
```
Client: 12.34.56.78
    ↓
Load Balancer: Terminates connection
    ├─ Receives from: 12.34.56.78
    └─ Sends to instance: From LB's private IP (e.g., 10.0.1.50)
    ↓
EC2 Instance
└─ Request appears from: 10.0.1.50 (LB IP, not client!) ✗
```

**Issue**: Application only sees load balancer IP, not actual client IP

### Solution: X-Forwarded Headers

**ALB inserts HTTP headers** with original client information:

#### 1. X-Forwarded-For
```
Header: X-Forwarded-For
Value: 12.34.56.78

Meaning: "This request came from client 12.34.56.78"
```

#### 2. X-Forwarded-Port
```
Header: X-Forwarded-Port
Value: 443

Meaning: "User accessed via port 443"
```

#### 3. X-Forwarded-Proto
```
Header: X-Forwarded-Proto
Value: https

Meaning: "User used HTTPS (not HTTP)"
```

### Real Request Flow

**Incoming Request**
```
GET / HTTP/1.1
Host: shop.example.com
User-Agent: Mozilla/5.0...

From client: 12.34.56.78
Over protocol: HTTPS (port 443)
```

**After ALB Processing**
```
GET / HTTP/1.1
Host: shop.example.com
User-Agent: Mozilla/5.0...
X-Forwarded-For: 12.34.56.78
X-Forwarded-Port: 443
X-Forwarded-Proto: https

From ALB to EC2: Using ALB's private IP
```

**Application Code Usage**
```python
# Flask example
from flask import Flask, request

app = Flask(__name__)

@app.route('/request-info')
def request_info():
    # Get original client IP (behind ALB)
    client_ip = request.headers.get('X-Forwarded-For', 
                                    request.remote_addr)
    
    # Get original protocol
    protocol = request.headers.get('X-Forwarded-Proto', 'http')
    
    # Get original port
    port = request.headers.get('X-Forwarded-Port', '80')
    
    return {
        'client_ip': client_ip,
        'protocol': protocol,
        'port': port,
        'alb_ip': request.remote_addr  # This is ALB's IP
    }
```

### Connection Termination Details

**Full Flow Visualization**
```
Step 1: Client connects to ALB
Client IP: 12.34.56.78
    ↓ (TCP connection)
ALB Port 443 (HTTPS)

Step 2: ALB processes HTTPS
└─ Decrypts SSL/TLS
└─ Reads HTTP request
└─ Adds X-Forwarded-* headers
└─ Connection terminated between ALB and client

Step 3: ALB connects to EC2
From ALB IP: 10.0.1.50 (private IP)
    ↓ (new HTTP connection)
EC2 Port 80 (HTTP, not encrypted)

Step 4: Application reads headers
X-Forwarded-For: 12.34.56.78
X-Forwarded-Port: 443
X-Forwarded-Proto: https
```

**Key Points**
- **Connection Termination**: ALB ends client SSL/TLS connection
- **New Connection**: ALB creates separate HTTP connection to instance
- **Header Injection**: ALB adds headers with original client details
- **Security**: HTTPS encryption offloaded at ALB, instance connection can be HTTP (or HTTPS for defense-in-depth)

---

## Part 7: Use Cases for ALB

### Primary Use Cases

**1. Microservices Architecture**
```
Single ALB
├─ User Service (instances or containers)
├─ Product Service (instances or containers)
├─ Order Service (instances or containers)
└─ Notification Service (instances or containers)

Routing:
├─ /users → User Service
├─ /products → Product Service
├─ /orders → Order Service
└─ /notifications → Notification Service
```

**2. Container-Based Applications (Docker/ECS)**
```
ALB
└─ ECS Cluster with dynamic port mapping
   └─ ALB intelligently routes to containers
   └─ As containers start/stop, routing updates
```

**3. Multi-Tenant SaaS**
```
ALB
├─ tenant1.saas.com → Tenant 1 Backend
├─ tenant2.saas.com → Tenant 2 Backend
└─ tenant3.saas.com → Tenant 3 Backend
```

**4. A/B Testing / Blue-Green Deployment**
```
ALB
├─ 50% traffic → Blue (stable version)
├─ 50% traffic → Green (new version)
└─ Can adjust split via query parameters or headers
```

**5. Different SLAs**
```
ALB routing based on customer tier
├─ Premium customers → Premium backend service
├─ Standard customers → Standard backend service
└─ Free customers → Limited backend service
```

### Comparison with CLB

| Aspect | CLB | ALB |
|--------|-----|-----|
| **OSI Layer** | 4 & 7 mix | 7 only |
| **Applications per LB** | 1 CLB per app | Many apps per ALB |
| **Routing** | Limited | Advanced (path, host, query) |
| **Containers** | Poor fit | Excellent fit |
| **Modern Apps** | Outdated | Recommended |

---

## Part 8: Best Practices for ALB

### Configuration Best Practices

**1. Organize by Target Groups**
- Logical grouping of backend services
- Separate health check configuration
- Clear management and monitoring

**2. Health Checks**
- Implement proper health check endpoints
- Endpoints should reflect actual service health
- Not just "endpoint exists" but "service works"

**3. Routing Rule Priority**
- Rules evaluated in order of priority
- Put specific rules before general ones
- Example: `/api/v2` before `/api`

**4. Multiple Listeners**
```
Listener 1: Port 80 (HTTP)
├─ Redirect all to HTTPS
└─ Rule: If HTTP, then redirect to HTTPS

Listener 2: Port 443 (HTTPS)
├─ All actual routing rules
└─ Route to target groups
```

**5. Security Groups**
- ALB SG: Open to public (0.0.0.0/0)
- Instance SG: Only from ALB SG
- Restrict instances from direct internet

---

## Part 9: Key Takeaways for SysOps Associate

### ALB Fundamentals
- **Layer 7**: Application layer (HTTP/HTTPS only)
- **Target Groups**: Logical grouping of backend resources
- **Intelligent Routing**: Content-based routing decisions
- **Modern Apps**: Designed for microservices and containers

### Routing Capabilities
- **Path-based**: Different URLs → different services
- **Hostname-based**: Different domains → different services
- **Query string/Header**: Parameter-based routing
- **Combination**: Multiple conditions in single rule

### Target Types
- EC2 instances
- Auto Scaling Groups
- ECS tasks (dynamic port mapping)
- Lambda functions
- Private IP addresses (on-premises)

### Client Information
- **X-Forwarded-For**: Original client IP
- **X-Forwarded-Port**: Original port (80, 443)
- **X-Forwarded-Proto**: Protocol (HTTP, HTTPS)
- **Always check**: Application must read these headers

### When to Use ALB
- Web applications and APIs
- Microservices architecture
- Container applications (especially ECS)
- Multiple applications on same load balancer
- Modern applications with advanced routing

### When NOT to Use ALB
- Extreme performance required → Use NLB
- Non-HTTP protocols needed → Use NLB or CLB
- Legacy applications expecting simple LB → Consider CLB (deprecated)

### Exam Focus Points
- ALB is Layer 7, supports only HTTP/HTTPS
- Path and hostname routing are core ALB features
- Target groups make ALB flexible and scalable
- X-Forwarded headers required to get original client IP
- ALB recommended over CLB for all new deployments
- Container (ECS) support is major ALB advantage