# SSL/TLS Certificates and SNI: Secure Communication with Load Balancers

SSL/TLS certificates enable encrypted communication between clients and load balancers, with Server Name Indication (SNI) allowing a single load balancer to host multiple SSL certificates for different domains.

---

## Part 1: SSL and TLS Fundamentals

### What is SSL/TLS?

**SSL (Secure Sockets Layer)**
```
SSL Definition:
├─ Stands for: Secure Sockets Layer
├─ Purpose: Encrypt data in transit between client and server
├─ Version: Older protocol (now deprecated)
├─ Status: Replaced by TLS
└─ Terminology: Often called "SSL" colloquially (incorrect but common)
```

**TLS (Transport Layer Security)**
```
TLS Definition:
├─ Stands for: Transport Layer Security
├─ Purpose: Encrypt data in transit (same as SSL)
├─ Version: Newer protocol (current standard)
├─ Status: Active and supported
├─ Terminology: Technically correct name
└─ Versions: TLS 1.0, 1.1, 1.2, 1.3 (current best)
```

**Practical Distinction**

```
Modern Reality:
├─ Industry calls it "SSL certificates"
├─ But they're actually TLS certificates
├─ Both terms used interchangeably
├─ Technically: Should say "TLS"
├─ Practically: Everyone says "SSL"
└─ In this course: Will say both (common practice)

Why Still Called SSL?
├─ Historical: SSL was first
├─ Consumer familiar with "SSL"
├─ Vendors market as "SSL"
├─ Marketing: "SSL" more recognizable than TLS
└─ Actual: All modern certs use TLS
```

### In-Flight Encryption

**What Does Encryption Do?**

```
Encryption Process:

Unencrypted (HTTP):
Client sends: "Full name: John Doe, SSN: 123-45-6789"
Network sees: Plain text (anyone can read)
Server receives: Readable data

Encrypted (HTTPS):
Client sends: "@#$%^&*()_+{}|:"<>?"
Network sees: Garbled encrypted data
Server receives: "@#$%^&*()_+{}|:" → Decrypts → Readable

Key Points:
├─ Data is encrypted before sending
├─ Travels encrypted over network (in flight)
├─ Only sender and receiver can decrypt
├─ Network cannot read encrypted data
└─ Man-in-the-middle attacks prevented
```

**Why In-Flight Encryption Matters**

```
Vulnerable Information:
├─ Login credentials (username/password)
├─ Credit card numbers
├─ Social security numbers
├─ Personal identification
├─ Banking information
├─ Medical records
├─ Confidential business data
└─ Anything sensitive

Browser Warning Signs:

HTTP (Not Encrypted):
├─ URL: http://website.com
├─ Browser: Red warning/lock icon
├─ Warning: "Insecure connection"
├─ Message: "Don't enter sensitive data"

HTTPS (Encrypted):
├─ URL: https://website.com
├─ Browser: Green lock icon
├─ Indicates: Secure connection
├─ Message: "Safe to enter credentials"
```

---

## Part 2: SSL/TLS Certificate Basics

### Certificate Components

**X.509 Certificate**

```
Technical Name:
├─ Called: X.509 certificate
├─ Format: Standard structure
├─ Used for: SSL/TLS authentication
└─ Contains: Host info + public key
```

**Certificate Contents**

```
S.509 Certificate Includes:

1. Domain Name
   ├─ www.example.com (the hostname)
   ├─ Identifies: Who this cert is for
   └─ Matches: Browser's request domain

2. Issuing Authority
   ├─ Certificate Authority (CA)
   ├─ Example: Let's Encrypt, Simantec, GoDaddy
   └─ Signature: Proves legitimacy

3. Public Key
   ├─ Used for: Encryption setup
   ├─ Shared: Publicly available
   └─ With: Private key pair for decryption

4. Validity Period
   ├─ Valid From: Start date
   ├─ Valid Until: Expiration date
   ├─ Duration: Usually 1 year
   └─ Renewal: Must refresh before expiring

5. Digital Signature
   ├─ By: Certificate Authority
   ├─ Proves: Authenticity
   └─ Verifies: Not tampered with

6. Additional Fields
   ├─ Serial number
   ├─ Hash algorithm
   ├─ Subject alternative names (SANs)
   └─ Extended validation info
```

### Certificate Authorities (CAs)

**What is a CA?**

```
Certificate Authority:
├─ Organization: Issues SSL/TLS certificates
├─ Purpose: Verify domain ownership
├─ Trust: Sign certificates with private key
├─ Verification: Prove you own the domain
└─ Result: Browser trusts your certificate

Major CAs:

Public CAs:
├─ Comodo
├─ Symantec (now Digicert)
├─ GoDaddy
├─ GlobalSign
├─ DigiCert
├─ Entrust
└─ Let's Encrypt (free CA)

CA Requirements:
├─ Verify you own the domain
├─ Create cryptographic signature
├─ Return signed certificate
├─ Certificate trusted by browsers
└─ Valid for specified duration
```

**Certificate Chain**

```
Browser Trust Model:

Browser contains:
└─ Root CA certificates (built-in)
   ├─ Examples: Let's Encrypt root
   ├─ Symantec root
   └─ Other major CAs

When You Get Certificate:

1. You own domain: example.com
2. You request SSL cert from CA
3. CA verifies ownership:
   ├─ Email verification
   ├─ DNS record check
   └─ HTTP challenge
4. CA signs certificate with private key
5. You install cert on server

When Client Connects:

1. Server sends certificate
2. Browser receives certificate
3. Browser checks: "Is CA signature valid?"
4. Browser knows CA's public key (from root store)
5. Browser verifies signature
6. Browser trusts: Connection is secure
7. User sees: Green lock icon
```

### Expiration and Renewal

**Certificate Lifecycle**

```
Certificate Timeline:

Event 1: Issue Certificate
├─ CA signs certificate
├─ Valid From: Jan 1, 2024
├─ Valid Until: Dec 31, 2024
└─ Duration: 1 year

Monitoring Period (months 9-12):
├─ You monitor expiration date
├─ Send renewal request
├─ Re-verify domain ownership
└─ CA issues new certificate

Event 2: Renewal
├─ New certificate issued
├─ Valid From: Dec 15, 2024
├─ Valid Until: Dec 14, 2025
├─ Updated certificate installed
└─ No service interruption

Event 3: Expiration (if not renewed)
├─ Certificate expires: Dec 31, 2024
├─ Browser sees: Expired certificate
├─ Warning: Red warning/lock icon
├─ Message: "Expired certificate"
├─ User warns: "Your connection may not be secure"
└─ Users can't access site safely

Renewal Best Practice:
├─ Start renewal: 30 days before expiration
├─ Install new cert: Before expiration
├─ Automatic renewal: Enable if available
├─ Set reminders: Don't forget dates
└─ Monitor: AWS Certificate Manager can alert
```

---

## Part 3: SSL/TLS Termination

### What is SSL Termination?

**SSL Termination at Load Balancer**

```
SSL Termination Concept:

Traditional (Single Server):
Client → HTTPS → Server
        Encrypted    (decrypted on server)
        
Load Balancer with Termination:
Client → HTTPS → Load Balancer → HTTP → EC2 Backend
        Encrypted    (terminated)   (plain)
        
Terminology:
├─ "Termination": Where encryption ends
├─ Responsibility: Load balancer handles HTTPS
├─ Backend: Communicates via plain HTTP
└─ Benefit: Offloads encryption work to LB
```

**Architecture Diagram**

```
Internet Clients (HTTPS encrypted)
           ↓
    ┌─────────────────┐
    │  Elastic Load   │
    │   Balancer      │ ← SSL Termination happens here
    │  (HTTPS 443)    │   Decrypts traffic
    └────────┬────────┘
             │
             ├─ Backend Communication (HTTP 80, plain)
             │  (Private VPC, no encryption needed)
             ▼
    ┌─────────────────┐
    │   EC2 Instance  │ ← Receives plain HTTP
    │  (HTTP 80)      │
    │                 │ ← No HTTPS cert needed here
    └─────────────────┘

Key Points:
├─ Clients connect: Encrypted HTTPS
├─ LB terminates: Decrypts using certificate
├─ Backend receives: Plain HTTP
├─ Network: Private VPC (secure anyway)
└─ Benefit: EC2 doesn't need cert/HTTPS overhead
```

### Why Termination?

**Benefits of SSL Termination**

```
Performance:
├─ Encryption/decryption CPU-intensive
├─ Load balancer optimized for this
├─ EC2 freed for application logic
└─ Overall throughput improved

Simplification:
├─ Certificate installed: Only on LB
├─ EC2 instances: Don't need certs
├─ Renewal: Only at LB level
├─ Management: Centralized

Scaling:
├─ Add instances: No cert re-installation
├─ Instances: Can be identical
├─ No per-instance SSL overhead
└─ Easy horizontal scaling

Backend Security:
├─ Within VPC: Private network
├─ Doesn't need HTTPS (VPC private)
├─ HTTP sufficient for backend
└─ Reduces complexity
```

---

## Part 4: AWS Certificate Manager (ACM)

### What is ACM?

**AWS Certificate Manager**

```
ACM Overview:
├─ Service: Manage SSL/TLS certificates
├─ Purpose: Issue and renew certificates
├─ Integration: Works with load balancers
├─ Provisioned: Public or private certs
└─ Benefit: Free (AWS absorbs cost)

ACM Capabilities:

Issue Certificates:
├─ Public certificates (for internet-facing)
├─ Private certificates (for internal use)
├─ Free tier: No cost for public certs
└─ Automatic renewal: Before expiration

Manage Certificates:
├─ Dashboard: View all certificates
├─ Expiration: Track renewal dates
├─ Auto-renewal: Enable automatic
├─ Integration: Attach to load balancers
└─ Monitoring: Alert on issues

Import Certificates:
├─ Bring your own: From other CAs
├─ Upload: Existing certificates
├─ Manage: In ACM dashboard
└─ Use: Attach to LB
```

### Using ACM with Load Balancers

**HTTPS Listener Configuration**

```
When Creating HTTPS Listener:

You must specify:
├─ Certificate: Default certificate (required)
├─ Source: From ACM or uploaded
└─ Domain: Must match listener

Configuration Example:

Listener:
├─ Protocol: HTTPS
├─ Port: 443
├─ Default certificate: arn:aws:acm:region:account:certificate/id
└─ For domain: www.example.com

Additional Certificates:
├─ Optional: List of additional certs
├─ Purpose: Support multiple domains
├─ Used with: SNI (Server Name Indication)
└─ Stored: In ACM dashboard
```

---

## Part 5: Server Name Indication (SNI)

### The SNI Problem

**Problem: One Server, Multiple Domains**

```
Traditional Challenge:

Scenario: Single web server
├─ Must host: www.company.com AND www.company.org
├─ Both need: Different SSL certificates
├─ Limitation: SSL handshake happens before HTTP headers
├─ Dilemma: Which certificate to send?

Without SNI (Classic LB Era):

Client 1: Connects to 203.0.113.10 (IP address)
├─ Server: Doesn't know domain wanted
├─ Sends: Default certificate (company.com)
├─ Client: Tries to reach company.org
├─ Browser: Certificate mismatch! ✗ Error

Solution Before SNI:

Required: Multiple load balancers
├─ LB1 for www.company.com → 203.0.113.10
├─ LB2 for www.company.org → 203.0.113.20
└─ Expensive and wasteful
```

### How SNI Works

**SNI Solution**

```
SNI (Server Name Indication):

Definition:
├─ Technique: Client tells server hostname
├─ Timing: During SSL/TLS handshake
├─ Message: "I want to connect to www.company.com"
├─ Effect: Server knows which cert to use
└─ Result: Multiple certs on one server

SNI Process:

1. Client initiates connection
   └─ Client includes hostname in SSL handshake
   └─ Message: "I want www.company.com"

2. Server receives SNI
   └─ Server reads: hostname = www.company.com
   └─ Looks up: Which certificate for this hostname?
   └─ Found: www.company.com certificate in list

3. Server responds with correct certificate
   └─ Sends: Matching certificate
   └─ Client: "Certificate matches my request!" ✓
   └─ Connection: Established securely

4. Connection established
   └─ Both parties: Encrypted communication
   └─ With: Correct certificate
   └─ For: Correct domain

With SNI:

Client 1: Connects to 203.0.113.10
├─ SNI: "I want www.company.com"
├─ Server: Sends www.company.com certificate ✓
└─ Success: Connection works

Client 2: Connects to same 203.0.113.10
├─ SNI: "I want www.company.org"
├─ Server: Sends www.company.org certificate ✓
└─ Success: Different certificate served
```

### SNI Diagram

**Visual Representation**

```
Internet Clients with SNI

         │    Client wants     │    Client wants
         │  www.mycorp.com     │  domain1.example.com
         ▼                      ▼
    
    ┌──────────────────────────────────────┐
    │      Application Load Balancer       │
    │  (Single Public IP)                  │
    │                                      │
    │  SSL Certificates:                   │
    │  ├─ www.mycorp.com cert              │
    │  └─ domain1.example.com cert         │
    │                                      │
    │  Routing Rules:                      │
    │  ├─ Host = www.mycorp.com            │
    │  │  └─ → Target Group 1              │
    │  │  └─ Certificate: www.mycorp.com   │
    │  │                                   │
    │  └─ Host = domain1.example.com       │
    │     └─ → Target Group 2              │
    │     └─ Certificate: domain1.example  │
    └──────────┬──────────────────────────┬┘
               │                          │
               ▼                          ▼
        ┌────────────┐────────────┐────────────┐────────────┐
        │Target Gr1  │  Target Gr │  Target Gr2│  Target Gr │
        │ Group 1    │  (example) │            │  (example) │
        └────────────┘────────────┘────────────┘────────────┘

Flow 1 (www.mycorp.com):
1. Client sends SNI: "www.mycorp.com"
2. ALB reads SNI
3. ALB selects: www.mycorp.com certificate
4. ALB encrypts connection
5. ALB routes to: Target Group 1
6. Response sent back through same path

Flow 2 (domain1.example.com):
1. Different client sends SNI: "domain1.example.com"
2. ALB reads SNI
3. ALB selects: domain1.example.com certificate
4. ALB encrypts connection
5. ALB routes to: Target Group 2
6. Response sent back through same path
```

### SNI Support by Load Balancer

**Which Load Balancers Support SNI?**

```
ALB (Application Load Balancer):
├─ Supports: SNI ✓ YES
├─ Can serve: Multiple SSL certificates
├─ Method: Uses hostname-based routing + SNI
├─ Per domain: Can route to different target groups
├─ Versions: v2 (all ALBs after ~2015)
└─ Recommended: YES

NLB (Network Load Balancer):
├─ Supports: SNI ✓ YES
├─ Can serve: Multiple SSL certificates
├─ Method: Uses SNI for certificate selection
├─ Per domain: Can route to different target groups
├─ Versions: v2 (all NLBs after ~2017)
└─ Recommended: YES

CLB (Classic Load Balancer):
├─ Supports: SNI ✗ NO
├─ Can serve: One SSL certificate only
├─ Limitation: Older protocol, pre-SNI
├─ For multiple: Requires multiple CLBs
├─ Versions: Generation 1 (deprecated)
├─ Recommended: NO (use ALB instead)
└─ Status: Deprecated by AWS

Summary:
├─ ALB: Full SNI support
├─ NLB: Full SNI support
├─ CLB: No SNI support
└─ New deployments: Use ALB or NLB only
```

---

## Part 6: Multiple SSL Certificates on Load Balancers

### ALB with Multiple Certificates

**Configuration**

```
ALB Configuration for Multiple Domains:

Listener (HTTPS port 443):
├─ Default Certificate: www.mycorp.com (required)
├─ Additional Certificates:
│  ├─ domain1.example.com
│  ├─ domain2.example.com
│  └─ example.org
└─ Total: 1 default + N additional

Routing Rules:

Rule 1:
├─ Condition: Host header = www.mycorp.com
├─ Certificate: www.mycorp.com (default)
└─ Action: Forward to Target Group 1

Rule 2:
├─ Condition: Host header = domain1.example.com
├─ Certificate: domain1.example.com (additional)
└─ Action: Forward to Target Group 2

Rule 3:
├─ Condition: Host header = domain2.example.com
├─ Certificate: domain2.example.com (additional)
└─ Action: Forward to Target Group 3

How SNI Enables This:

Client 1: "I want www.mycorp.com"
├─ SNI reads: www.mycorp.com
├─ ALB selects: www.mycorp.com cert
└─ Routed based on: Matching rule

Client 2: "I want domain1.example.com"
├─ SNI reads: domain1.example.com
├─ ALB selects: domain1.example.com cert
└─ Routed based on: Matching rule

Benefits:
├─ Single ALB: Serves multiple domains
├─ Multiple certificates: No conflicts
├─ Clients see: Correct certificate
├─ Cost: One LB instead of many
└─ Management: Centralized
```

### NLB with Multiple Certificates

**Configuration**

```
NLB Configuration for Multiple Domains:

Listener 1 (HTTPS port 443):
├─ Certificate 1: www.mycorp.com
├─ Used for: www.mycorp.com connections
└─ Action: Forward to Target Group 1

Listener 2 (HTTPS port 443):
├─ Certificate 2: domain1.example.com
├─ Used for: domain1.example.com connections
└─ Action: Forward to Target Group 2

How It Works (NLB Approach):

NLB with SNI:
├─ Multiple listeners: Different certs
├─ Each listener: Specific hostname
├─ SNI: Directs to correct listener
└─ Listener: Uses appropriate certificate

Alternative (if needed):
├─ Multiple ports: 443, 8443, 9443
├─ Each port: Different certificate
├─ Clients: Use specific port
└─ Traditional: Less elegant than SNI

SNI Benefits for NLB:
├─ Multiple certs: Same port (443)
├─ No port changes: Standard HTTPS
├─ Cleaner: Single 443 port
└─ Modern: Uses SNI protocol
```

### CLB with Multiple Certificates

**Limitation**

```
Classic Load Balancer - No SNI Support:

CLB Limitation:
├─ Only one certificate: Per load balancer
├─ If multiple domains: Must create multiple CLBs
├─ Different IPs: Each CLB has own IP
├─ Expensive: Multiple load balancers
└─ Management: Complicated

For Multiple Domains with CLB:

Domain 1 (www.company.com):
├─ CLB 1: IP 203.0.113.1
├─ Certificate: www.company.com
└─ DNS: www.company.com → 203.0.113.1

Domain 2 (www.company.org):
├─ CLB 2: IP 203.0.113.2
├─ Certificate: www.company.org
└─ DNS: www.company.org → 203.0.113.2

Problem:
├─ Cost: 2 load balancers
├─ Management: Double the infrastructure
├─ Redundancy: Each CLB separate
└─ Not scalable

With ALB/NLB + SNI:

Both Domains:
├─ Single ALB: IP 203.0.113.1
├─ Cert 1: www.company.com
├─ Cert 2: www.company.org
├─ DNS: Both domains → 203.0.113.1 (same IP)
├─ SNI: Routes to correct certificate

Advantages:
├─ Cost: One load balancer
├─ Management: Simplified
├─ Redundancy: Shared
└─ Scalable

Why CLB Doesn't Support SNI:
├─ CLB: Pre-SNI era technology
├─ SNI: Newer protocol standard
├─ CLB: Doesn't read SNI from clients
└─ Cannot: Select certificate based on hostname
```

---

## Part 7: SSL/TLS Security Policies

### Security Policy Concept

**What is a Security Policy?**

```
Security Policy Purpose:

Control:
├─ Which SSL/TLS versions are supported
├─ Which encryption ciphers are allowed
├─ Strength of encryption
└─ Legacy client support level

Policy Types:

Legacy/Permissive:
├─ Support: SSL 3.0, TLS 1.0, 1.1, 1.2
├─ Ciphers: Older, weaker ciphers allowed
├─ Clients: Older browsers work
├─ Risk: Security vulnerabilities
└─ Use: Only if must support old clients

Modern:
├─ Support: TLS 1.2, 1.3 only
├─ Ciphers: Strong, modern ciphers only
├─ Clients: Modern browsers only
├─ Risk: Minimal
└─ Use: Recommended for all

Custom:
├─ You choose: Specific TLS versions
├─ You choose: Specific ciphers
├─ Balance: Security vs compatibility
└─ Use: If need specific requirements
```

**Why Security Policies Matter**

```
SSL Vulnerabilities:

SSLv3:
├─ POODLE attack (2014)
├─ Can be downgraded from TLS
└─ Should be: DISABLED

TLS 1.0, 1.1:
├─ BEAST attack
├─ CRIME attack
├─ Deprecated: By IETF
└─ Recommendation: Disable or limit

TLS 1.2:
├─ Secure: If configured properly
├─ Ciphers: Some weak ciphers exist
├─ Recommendation: Use strong ciphers only

TLS 1.3 (Latest):
├─ Most secure: Latest standard
├─ Improved: Over 1.2
├─ Modern clients: All support
└─ Recommendation: Prefer when possible
```

**AWS Policy Examples**

```
Common AWS Policies:

ELBSecurityPolicy-2016-08 (Legacy):
├─ Support: TLS 1.0, 1.1, 1.2
├─ Ciphers: Includes weak ciphers
├─ Status: Deprecated
└─ Compatibility: Old clients

ELBSecurityPolicy-2017-01 (Modern):
├─ Support: TLS 1.2 primarily
├─ Ciphers: Strong ciphers
├─ Status: Recommended
└─ Compatibility: Modern clients

ELBSecurityPolicy-FS-1-2-Res-2019-08 (Forward Secure):
├─ Support: TLS 1.2 and 1.3
├─ Ciphers: Forward secrecy enabled
├─ Status: Highly secure
└─ Compatibility: Modern clients

ELBSecurityPolicy-TLS-1-2-2017-01 (Strict):
├─ Support: TLS 1.2 only
├─ Ciphers: Limited to strong
├─ Status: Very secure
└─ Compatibility: May exclude old clients
```

**Applying Security Policy**

```
When Configuring HTTPS Listener:

You can set:
├─ Security policy: Choose from AWS options
├─ Support legacy: Usually a checkbox
├─ If enabled: Permits older TLS/ciphers
└─ If disabled: Only modern versions

Recommendation:
├─ Default: AWS recommended policy
├─ Modern: Disable legacy support
├─ Check: Application requirements
├─ Test: With your clients before deploying
└─ Monitor: For backward-compatibility issues
```

---

## Part 8: Integration Summary

### Load Balancer SSL/TLS Capabilities

**Comparison Table**

```
Feature                 ALB             NLB             CLB
─────────────────────────────────────────────────────────
HTTPS Support           ✓ YES           ✓ YES           ✓ YES
SNI Support             ✓ YES           ✓ YES           ✗ NO
Multiple Certs          ✓ YES           ✓ YES           ✗ NO (1 only)
ACM Integration         ✓ YES           ✓ YES           ✓ YES
Custom Certs Upload     ✓ YES           ✓ YES           ✓ YES
Security Policies       ✓ YES           ✓ YES           ✓ YES
Default Cert            Required        Required        Required
Additional Certs        Unlimited       Unlimited       N/A
SSL Termination         ✓ YES           ✓ YES           ✓ YES
Hostname Routing        ✓ YES           ✓ YES           ✗ NO
Recommended             ✓ YES           ✓ YES (perf)    ✗ NO (legacy)
```

### Practical Configuration Workflow

**Setting Up HTTPS on ALB**

```
Step 1: Prepare Certificates
├─ Option A: Use ACM-issued certificate
│  ├─ Request certificate in ACM
│  ├─ Verify domain ownership
│  └─ Receive certificate
│
└─ Option B: Upload existing certificate
   ├─ Have certificate from other CA
   ├─ Upload to ACM
   └─ Can then use with LB

Step 2: Create HTTPS Listener
├─ Protocol: HTTPS
├─ Port: 443
├─ Default certificate: Select from ACM
└─ Security policy: Choose AWS policy

Step 3: Add Additional Certs (if needed)
├─ For additional domains
├─ ALB can support multiple
└─ Client SNI determines which

Step 4: Configure Rules
├─ Hostname-based: www.company.com
├─ Route to: Target Group 1
│  └─ Certificate: Automatically selected via SNI
│
└─ Hostname-based: domain1.example.com
   └─ Route to: Target Group 2
      └─ Certificate: Automatically selected via SNI

Step 5: Configure Target Groups
├─ Backend: HTTP (unencrypted)
├─ Reason: Private VPC
└─ EC2: No certificate needed

Result:
├─ Client: HTTPS encrypted to ALB
├─ ALB: Terminates SSL
├─ Backend: Plain HTTP (safe in VPC)
└─ Perfect security setup
```

---

## Part 9: Key Concepts Summary

### SSL/TLS Essentials

```
Essential Knowledge:

1. What SSL/TLS Is:
   └─ Encryption protocol for internet traffic
   └─ Enables: https:// URLs
   └─ Provides: Confidentiality and authentication

2. In-Flight Encryption:
   └─ Data encrypted while traveling
   └─ Network cannot read data
   └─ Only endpoints can decrypt

3. Certificate Authorities:
   └─ Trustworthy organizations
   └─ Issue certificates after verification
   └─ Examples: Let's Encrypt, GoDaddy, etc.

4. SSL Termination:
   └─ Load balancer handles encryption
   └─ Backend gets plain HTTP
   └─ Offloads CPU work from instances

5. SNI (Server Name Indication):
   └─ Client tells server what domain wanted
   └─ Enables: Multiple certs per load balancer
   └─ Required: For hosting multiple domains

6. Certificate Components:
   └─ Domain name (what it's for)
   └─ Public key (for encryption setup)
   └─ CA signature (proof of authenticity)
   └─ Expiration date (renewal required)

7. AWS Certificate Manager:
   └─ Service to manage certificates
   └─ Free for public certificates
   └─ Automatic renewal available

8. Load Balancer Support:
   └─ ALB: Full support including SNI
   └─ NLB: Full support including SNI
   └─ CLB: Limited (no SNI, 1 cert only)
   └─ Recommendation: Use ALB or NLB
```

---

## Part 10: Exam Focus Points

### Common Exam Questions

```
Question Types:

1. "How many SSL certificates can ALB support?"
   A) 1 only
   B) Limited to 5
   C) Multiple (with SNI)
   D) Not supported
   
   Answer: C) Multiple (with SNI)
   └─ ALB supports multiple via SNI
   └─ CLB: Limited to 1

2. "What does SNI enable?"
   A) Faster encryption
   B) Multiple certificates on single LB
   C) Lower latency
   D) Better security
   
   Answer: B) Multiple certificates on single LB
   └─ SNI lets client tell server hostname
   └─ Server responds with matching cert

3. "Loading multiple domains requires what for ALB?"
   A) Multiple ALBs
   B) SNI support
   C) Separate load balancers per domain
   D) Special ALB version
   
   Answer: B) SNI support
   └─ ALB has SNI by default
   └─ SNI allows multiple certs per ALB

4. "Where does SSL termination occur?"
   A) EC2 instances
   B) Application layer
   C) Load balancer
   D) VPC router
   
   Answer: C) Load balancer
   └─ LB decrypts HTTPS
   └─ Backend receives HTTP

5. "Classic Load Balancer supports SNI?"
   A) Yes, full support
   B) Yes, partial support
   C) No, not supported
   D) Only with upgrade
   
   Answer: C) No, not supported
   └─ CLB predates SNI
   └─ Requires multiple CLBs for multiple domains

6. "ACM certificate cost for ALB?"
   A) $10/month per certificate
   B) $5/certificate yearly
   C) Free for public certificates
   D) Depends on domain length
   
   Answer: C) Free for public certificates
   └─ AWS doesn't charge for public ACM certs
   └─ Private certs have cost

7. "Which LB should you use for multiple domains?"
   A) CLB only
   B) ALB or NLB
   C) Need separate CLBs
   D) Custom solution required
   
   Answer: B) ALB or NLB
   └─ Both support SNI
   └─ CLB would need multiple instances

8. "Backend communication after SSL termination?"
   A) HTTPS (encrypted)
   B) HTTP (plain)
   C) Both options
   D) Custom protocol
   
   Answer: B) HTTP (plain)
   └─ LB terminates encryption
   └─ Backend uses HTTP
   └─ VPC is private (secure anyway)
```

### Key Takeaways

```
What to Remember:

1. SSL = TLS (terminology)
   └─ Technically TLS now
   └─ Everyone says SSL
   └─ Don't overthink it

2. SNI = Multiple Domains
   └─ One LB, many domains
   └─ Different certs per domain
   └─ Requires: ALB or NLB

3. Certificate Expiration
   └─ Must be renewed
   └─ ACM can auto-renew
   └─ Set reminders

4. SSL Termination Location
   └─ Happens at: Load balancer
   └─ Not needed: On EC2
   └─ Backend: Plain HTTP

5. CLB Limitation
   └─ No SNI support
   └─ One domain per CLB
   └─ Don't use for multi-domain

6. ACM is Free
   └─ Public certificates: No cost
   └─ Use it: Always recommended
   └─ Auto-renewal: Available

7. Backend Encryption
   └─ Not needed: Private VPC
   └─ Use HTTP: Between LB and EC2
   └─ Frontend: HTTPS (encrypted)

8. Security Policies
   └─ Choose: Modern versions
   └─ Support TLS 1.2+
   └─ Avoid: Legacy if possible
```

---

## Part 11: Hands-On Concepts

### What Was Demonstrated

```
1. ALB SSL Configuration:
   ├─ HTTPS listener creation
   ├─ Default certificate selection
   ├─ Additional certificates for multiple domains
   └─ SNI enabling automatic routing

2. Multiple Certificate Support:
   ├─ www.mycorp.com certificate
   ├─ domain1.example.com certificate
   ├─ Stored in ALB configuration
   └─ Selected automatically via SNI

3. Routing with SNI:
   ├─ Client specifies domain in SNI
   ├─ ALB selects matching certificate
   ├─ ALB routes to matching target group
   └─ Response encrypted with selected cert

4. Practical Setup:
   ├─ ALB attributes reviewed
   ├─ Certificate storage in ACM
   ├─ Listener configuration shown
   └─ No technical deep-dive executed
```

### Practical Workflow

```
To Deploy Multi-Domain HTTPS on ALB:

1. Use AWS Certificate Manager:
   ├─ Request certificates for:
   │  ├─ www.mycorp.com
   │  └─ domain1.example.com
   └─ Wait for validation

2. Create ALB:
   ├─ Internet-facing
   ├─ Multiple subnets
   └─ Some instance targets

3. Create HTTPS Listener:
   ├─ Port: 443
   ├─ Protocol: HTTPS
   ├─ Default cert: www.mycorp.com
   ├─ Add additional: domain1.example.com
   └─ Security policy: Recommended

4. Create Hostname Rules:
   ├─ Rule 1: www.mycorp.com → TG1
   ├─ Rule 2: domain1.example.com → TG2
   └─ SNI: Automatic

5. Test:
   ├─ https://www.mycorp.com
   │  └─ Uses: mycorp certificate
   │  └─ Routes to: TG1
   │
   └─ https://domain1.example.com
      └─ Uses: example certificate
      └─ Routes to: TG2

Result:
├─ Single ALB: Serves 2 domains
├─ Each domain: Own certificate
├─ Clients: Secure connection
└─ Perfect configuration
```