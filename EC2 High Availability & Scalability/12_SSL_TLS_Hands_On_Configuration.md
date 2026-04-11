# SSL/TLS Hands-On Configuration: ALB and NLB

Practical walkthrough for enabling SSL/TLS certificates on Application Load Balancers and Network Load Balancers using AWS Certificate Manager, IAM, or imported certificates.

---

## Part 1: Prerequisites for HTTPS Configuration

### What You Need

```
Before Configuring HTTPS:

1. Load Balancer
   ├─ ALB or NLB already created
   ├─ Has target groups
   └─ Instances registered as targets

2. Target Groups
   ├─ At least one target group
   ├─ Instances healthy or available
   └─ Backend configured for HTTP

3. SSL/TLS Certificate
   ├─ Option A: AWS Certificate Manager (recommended)
   │  └─ Issued by ACM (free)
   │
   ├─ Option B: Purchased certificate
   │  └─ From Certificate Authority
   │  └─ Uploaded to ACM or IAM
   │
   └─ Option C: Import certificate
      └─ Bring your own cert
      └─ Provide: Private key, Certificate body, Chain

4. Domain
   ├─ Domain name validated
   ├─ Certificate matches domain
   └─ DNS configured (if testing)

5. Security Groups
   ├─ Inbound: Port 443
   │  └─ From: Your clients/internet
   │
   ├─ Outbound: Port 80 to instances
   │  └─ For: Backend HTTP
   │
   └─ Instance SG: Allow from LB SG
```

### Certificate Sources in AWS

**Understanding the Options**

```
Certificate Storage Options:

ACM (AWS Certificate Manager) - RECOMMENDED:
├─ Where stored: AWS Certificate Manager service
├─ Cost: Free for public certificates
├─ Auto-renewal: Available (no expiration worry)
├─ Integration: Seamless with LBs
├─ Best for: New certificates or renewals
├─ Process: Request in ACM, verify domain, attach to LB
└─ Recommended: ✓ YES - Use this

IAM (Identity and Access Management) - NOT RECOMMENDED:
├─ Where stored: IAM certificate store
├─ Cost: No additional cost
├─ Auto-renewal: Not available
├─ Integration: Works but older method
├─ Best for: Legacy deployments
├─ Process: Import into IAM, attach to LB
├─ Note: AWS deprecated IAM cert storage
└─ Recommended: ✗ NO - Use ACM instead

Import Certificate - MANUAL:
├─ Where stored: ACM (after import)
├─ Cost: No cost to store in ACM
├─ Auto-renewal: You manage renewal
├─ Integration: Works after import
├─ Best for: Certificates from other CAs (GoDaddy, etc.)
├─ Process: Paste key/body/chain into ACM
└─ Recommended: ✓ If already have cert from CA
```

---

## Part 2: ALB HTTPS Listener Configuration

### Step 1: Navigate to ALB

**In AWS Console**

```
Navigation Path:

1. Open AWS Console
   └─ URL: https://console.aws.amazon.com

2. Go to EC2 Dashboard
   └─ Service: EC2
   └─ Region: Select your region

3. Find Load Balancers
   └─ Left sidebar: Load Balancers
   └─ Filter: Application Load Balancers

4. Select Your ALB
   ├─ Name: Your ALB name
   ├─ Tab: Click on your ALB
   └─ Details page opens
```

### Step 2: Add HTTPS Listener

**Creating the Listener**

```
In ALB Details Page:

1. Find Listeners Section
   ├─ Tab: "Listeners" or "Listeners and Rules"
   ├─ Shows: Existing listeners (HTTP 80, etc.)
   └─ Action: Click "Add listener"

2. New Listener Dialog Opens:
   
   Protocol Selection:
   ├─ Dropdown: "Select a protocol"
   ├─ Options: HTTP, HTTPS, gRPC
   ├─ Choose: HTTPS
   └─ Protocol set to: HTTPS

   Port Selection:
   ├─ Field: "Port"
   ├─ Value: 443 (default for HTTPS)
   ├─ Can change: To different port if needed
   │  ├─ 8443 (alternative HTTPS)
   │  └─ 9443 (another alternative)
   ├─ For standard: Leave as 443
   └─ Port set to: 443
```

### Step 3: Configure Secure Listener Settings

**Security Configuration**

```
After Protocol/Port Selection:

Section: "Secure listener settings"
├─ This section: Automatically appears
├─ Only for: HTTPS/TLS protocols
└─ Settings below

Setting 1: SSL Security Policy

Field: "Security policy"
├─ Dropdown: List of AWS policies
├─ Default: Usually "ELBSecurityPolicy-TLS-1-2-2017-01"
├─ Options shown:
│  ├─ ELBSecurityPolicy-TLS-1-0-2015-04 (Very old, not recommended)
│  ├─ ELBSecurityPolicy-2016-08 (Legacy with backward compat)
│  ├─ ELBSecurityPolicy-TLS-1-1-2017-01 (Older TLS 1.1/1.2)
│  ├─ ELBSecurityPolicy-TLS-1-2-2017-01 (Modern TLS 1.2 only)
│  ├─ ELBSecurityPolicy-FS-1-2-Res-2019-08 (Strong forward secrecy)
│  ├─ ELBSecurityPolicy-FS-1-2-2019-08 (Forward secrecy, balanced)
│  └─ Others: Specific cipher suites

Recommendation:
├─ Choose: "ELBSecurityPolicy-2016-08" or newer
├─ Modern: Prefer TLS 1.2+
├─ Legacy support: Only if needed
└─ Default: Usually fine

Selection:
├─ For typical setup: Leave on default (AWS recommended)
├─ Click: Select policy
└─ Applied to: This listener
```

### Step 4: Select or Import Certificate

**Certificate Configuration - Option A: From ACM (Recommended)**

```
Setting 2: Certificate Source

Field: "Choose a certificate"

Option A: FROM ACM (Recommended Path):

1. Dropdown appears: List of ACM certificates
   ├─ Shows: All certificates in current region
   ├─ Displays: Domain names
   ├─ Shows: Expiration dates
   └─ Examples:
      ├─ www.example.com (expires: Dec 31, 2024)
      ├─ example.org (expires: Jun 15, 2025)
      └─ *.example.net (expires: Mar 10, 2024)

2. Select Your Certificate:
   ├─ Look for: Certificate matching your domain
   ├─ Example: www.example.com
   ├─ Click: To select
   └─ Selected: Certificate now chosen

3. Additional Certificates (Optional):
   ├─ For: Multiple domains on same ALB
   ├─ SNI usage: Multiple certificates
   ├─ Button: "Add certificate" or similar
   ├─ Add: Additional certs as needed
   │  ├─ example.org
   │  ├─ Example.net
   │  └─ etc
   └─ Stored: All certs attached to listener

Result:
├─ Primary cert: Direct HTTPS connections
├─ Additional certs: Via SNI matching
└─ Ready for: Multiple domain hosting

Setup Example:

Listener 1 (HTTPS 443):
├─ Primary cert: www.example.com
├─ Additional: example.org, example.net
└─ SNI routes: To appropriate target groups
```

**Certificate Configuration - Option B: From IAM (Not Recommended)**

```
Option B: FROM IAM:

1. Certificate source: IAM store
   ├─ Legacy method (not recommended)
   ├─ AWS guidance: Use ACM instead
   └─ Process: Similar to ACM

2. Select from dropdown:
   ├─ Shows: Certs in IAM
   ├─ Displays: Domain names
   └─ Click: To select

3. Why not recommended:
   ├─ IAM: Older certificate storage
   ├─ ACM: Modern recommended service
   ├─ Auto-renewal: Only in ACM
   ├─ Management: Easier in ACM
   └─ AWS direction: Migrate to ACM

Recommendation:
├─ If using IAM certs: Migrate to ACM
└─ Use ACM: For all new deployments
```

**Certificate Configuration - Option C: Import Certificate (Manual)**

```
Option C: IMPORT NEW CERTIFICATE:

When to Use:
├─ Have cert from external CA: GoDaddy, Symantec, etc.
├─ Don't have cert in ACM: Not yet uploaded
├─ Want to import: Into ACM for storage
└─ Will then: Attach to listener

Import Process:

1. Click: "Import certificate" or similar button
   └─ Dialog: Certificate import appears

2. Provide Three Components:
   
   Component 1: Certificate Body
   ├─ Paste: The certificate body
   ├─ Format: PEM (text format)
   ├─ Looks like:
   │  ├─ -----BEGIN CERTIFICATE-----
   │  ├─ [Long base64 string]
   │  └─ -----END CERTIFICATE-----
   ├─ Source: From your CA
   └─ Paste: Into "Certificate body" field
   
   Component 2: Private Key
   ├─ Paste: The private key
   ├─ Format: PEM (text format)
   ├─ Looks like:
   │  ├─ -----BEGIN RSA PRIVATE KEY-----
   │  ├─ [Long base64 string]
   │  └─ -----END RSA PRIVATE KEY-----
   ├─ Source: From your CA or key generation
   ├─ Note: Keep secure, don't share
   └─ Paste: Into "Private key" field
   
   Component 3: Certificate Chain (Optional)
   ├─ Paste: Chain file if provided
   ├─ Format: PEM (text format)
   ├─ Looks like: Multiple certificates concatenated
   ├─ Source: From your CA
   ├─ When needed: CA usually provides this
   └─ Paste: Into "Certificate chain" field

3. Click: "Import" or "Save"
   ├─ Validation: AWS validates certificate
   ├─ Check: Domain match, validity
   ├─ Upload: To ACM service
   └─ Result: Certificate now in ACM

4. Certificate Now Available:
   ├─ Stored in: ACM
   ├─ Can be: Reused on multiple LBs
   ├─ Auto-renewal: You manage
   └─ Ready to: Attach to listener

When Needed:
├─ Purchased cert from CA: Must import
├─ Self-signed cert: Can import
├─ Cert from third-party: Must import first
└─ Then: Select from ACM dropdown
```

### Step 5: Configure Default Action

**Routing Configuration**

```
After Certificate Selection:

Section: "Default action"

Action Type: Forward
├─ Dropdown: "Select action"
├─ Options: Forward, Redirect, Fixed response
├─ Common: Forward to target group
├─ Choose: Forward

Target Group Selection:
├─ Dropdown: "Select a target group"
├─ Shows: All target groups for this ALB
├─ Choose: Where to send HTTPS traffic
├─ Example: TG-Web-Servers
│  ├─ Contains: Your backend EC2 instances
│  ├─ Health: Instances in healthy state
│  └─ Port: 80 (HTTP internally)

Selection:
├─ Pick: Your main/default target group
├─ Example: web-servers-tg
└─ Traffic: Will route to these targets
```

### Step 6: Complete Listener Creation

**Finalizing**

```
Final Steps:

1. Review Settings:
   ├─ Protocol: HTTPS ✓
   ├─ Port: 443 ✓
   ├─ Certificate: Selected ✓
   ├─ Security policy: Chosen ✓
   └─ Action: Forward to TG ✓

2. Click: "Add listener" or "Save"
   ├─ Validation: AWS validates
   ├─ Creation: Listener is created
   ├─ Time: Usually immediate
   └─ Status: Active

3. Listener Now Active:
   ├─ HTTPS 443: Now listening
   ├─ Accepts: HTTPS connections
   ├─ Certificate: Loaded and ready
   ├─ Routes: To target group
   └─ Clients: Can connect via HTTPS

Result:
├─ ALB listens on: 443 (HTTPS)
├─ Clients use: HTTPS protocol
├─ Certificate used: ACM certificate
├─ Encryption: Enabled
└─ Backend: HTTP (unencrypted in VPC)
```

---

## Part 3: ALB Additional Configurations

### Adding Additional Certificates (Multiple Domains)

**For Multi-Domain Hosting**

```
Scenario:
├─ One ALB: Serving multiple domains
├─ Domain 1: www.example.com
├─ Domain 2: example.org
├─ Domain 3: example.net
├─ Each domain: Own certificate via SNI

Configuration:

1. After Creating HTTPS Listener:
   └─ Listener: HTTPS 443 exists

2. Edit Listener or Add Certs:
   ├─ Method 1: During creation
   │  ├─ Click: "Add certificate"
   │  ├─ Select: Next certificate
   │  ├─ Repeat: For each additional
   │  └─ Add: All certs to listener
   │
   └─ Method 2: After creation (Edit Listener)
      ├─ Click: Edit listener
      ├─ Find: SSL certificates section
      ├─ Add: Additional certificates
      ├─ Save: Changes
      └─ Effect: Immediate

3. ALB Configuration Result:
   
   Listener (HTTPS 443):
   ├─ Certificate 1 (Default): www.example.com
   ├─ Certificate 2: example.org (via SNI)
   ├─ Certificate 3: example.net (via SNI)
   └─ Selected automatically: Based on SNI

4. How Clients Connect:

   Client A:
   ├─ Sends: SNI "www.example.com"
   ├─ ALB selects: www.example.com cert
   ├─ Routes: Based on hostname rule
   ├─ Target group: TG-Domain1
   └─ Response: Encrypted with domain1 cert

   Client B:
   ├─ Sends: SNI "example.org"
   ├─ ALB selects: example.org cert
   ├─ Routes: Based on hostname rule
   ├─ Target group: TG-Domain2
   └─ Response: Encrypted with domain2 cert

5. Listener Rules Configuration:
   ├─ Rule 1: Host = www.example.com
   │  │       → Forward to TG1
   │  │       (cert selected: www.example.com)
   │  │
   │  └─ Rule 2: Host = example.org
   │            → Forward to TG2
   │            (cert selected: example.org)

Result:
├─ One ALB: Handles all domains
├─ Multiple certs: Attached to listener
├─ SNI: Automatic certificate selection
├─ Clients: Each gets correct certificate
└─ Scalable: Add domains without new LBs
```

---

## Part 4: NLB TLS Listener Configuration

### Step 1: Navigate to NLB

**In AWS Console**

```
Navigation Path:

1. Open AWS Console
   └─ URL: https://console.aws.amazon.com

2. Go to EC2 Dashboard
   └─ Service: EC2
   └─ Region: Select your region

3. Find Load Balancers
   └─ Left sidebar: Load Balancers
   └─ Filter: Network Load Balancers
   └─ Note: NLB also listed as "Type: Network"

4. Select Your NLB
   ├─ Name: Your NLB name
   ├─ Click: To open details
   └─ Details page: Opens
```

### Step 2: Add TLS Listener

**Creating the Listener**

```
In NLB Details Page:

1. Find Listeners Section
   ├─ Tab: "Listeners" or "Listeners and Rules"
   ├─ Shows: Existing listeners (TCP 80, TCP 443, etc.)
   └─ Action: Click "Add listener"

2. New Listener Dialog Opens:
   
   Protocol Selection:
   ├─ Dropdown: "Select a protocol"
   ├─ Options: TCP, TLS, UDP
   │  ├─ TCP: Plain (no encryption)
   │  ├─ TLS: Encrypted (what we want)
   │  └─ UDP: Non-TCP protocol
   ├─ Choose: TLS (not TCP)
   └─ Protocol set to: TLS

   Port Selection:
   ├─ Field: "Port"
   ├─ Value: 443 (default for TLS/HTTPS)
   ├─ Can change: To different port if needed
   │  ├─ 8883 (alternative MQTT TLS)
   │  ├─ 9443 (alternative TLS port)
   │  └─ Other: As needed
   ├─ For standard: Leave as 443
   └─ Port set to: 443
```

### Step 3: Configure Security Settings

**TLS Security Options**

```
After Protocol/Port Selection:

Section: "Secure listener settings" or "TLS Configuration"
├─ This section: For TLS protocols
└─ Settings below

Setting 1: SSL Security Policy

Field: "Security policy"
├─ Dropdown: List of AWS TLS policies
├─ Default: Usually latest recommended
├─ Options: Same as ALB options
│  ├─ ELBSecurityPolicy-TLS-1-2-Ext-2018-06 (Extended)
│  ├─ ELBSecurityPolicy-TLS-1-2-2017-01 (Standard)
│  ├─ ELBSecurityPolicy-FS-1-2-Res-2019-08 (Forward Secure)
│  └─ Others: Specific policies

Recommendation:
├─ Choose: AWS recommended policy
├─ Or: "ELBSecurityPolicy-TLS-1-2-2017-01" or newer
└─ Default: Usually fine

Selection:
├─ For typical: Leave default
└─ Applied: To this listener
```

### Step 4: Select Certificate for NLB

**Certificate Configuration**

```
Setting 2: Certificate Source

Field: "Choose a certificate"

Available Options:

Option A: FROM ACM (RECOMMENDED):
├─ Dropdown: Shows ACM certificates
├─ Filter: By region
├─ Display: Domain names and expiration
├─ Select: Your certificate
└─ Example: www.example.com

Option B: FROM IAM (NOT RECOMMENDED):
├─ Dropdown: Shows IAM certificates
├─ Legacy: Use ACM instead
├─ Process: Select if available
└─ Recommendation: Move to ACM

Option C: IMPORT NEW:
├─ Click: Import certificate
├─ Process: Same as ALB process
├─ Provide: Private key, body, chain
├─ Result: stored in ACM
└─ Then: Available to select

Selection Process:
1. Click: Certificate dropdown
2. Choose: Your certificate
3. Verify: Correct domain
4. Select: Click to choose
5. Applied: To this listener
```

### Step 5: Configure Application Layer Protocol Negotiation (ALPN)

**Advanced TLS Setting**

```
Setting 3: ALPN Policies (Advanced/Optional)

What is ALPN?
├─ Stands for: Application Layer Protocol Negotiation
├─ Purpose: Negotiate protocol during TLS handshake
├─ Protocols: HTTP/1.1, HTTP/2, etc.
├─ Use case: Advanced TLS scenarios
└─ Typical: Leave as default

ALPN Field:
├─ Appearance: Usually auto-populated
├─ Values: Predefined by AWS
├─ Configuration: Usually not needed
├─ Leave as: Default value
├─ Advanced users: Can customize

Common ALPN Values:
├─ HTTP/1.1: Standard HTTP
├─ HTTP/2: HTTP version 2 (faster)
├─ TLS 1.3: Newer TLS protocol
└─ HTTP/1.0: Older HTTP

Default Recommendation:
├─ Most deployments: Use default
├─ Custom: Only if specific need
├─ Testing: Leave unchanged
└─ Effect: Usually transparent to users

When to Modify:
├─ HTTP/2 support: If clients need this
├─ Legacy clients: If need older versions
├─ Performance: Tuning for specific workload
└─ Rarely needed: For standard deployments

In Console:
├─ Field: Usually pre-filled
├─ Edit: Click if need changes
├─ Value: Usually correct as-is
└─ Leave: Default is fine
```

### Step 6: Configure Default Action

**Routing Configuration**

```
After Certificate and Policy Selection:

Section: "Default action"

Action Type: Forward
├─ Dropdown: "Select action"
├─ Options: Forward, Fixed response
├─ Choose: Forward

Target Group Selection:
├─ Dropdown: "Select a target group"
├─ Shows: All target groups for this NLB
├─ Choose: Where to send TLS traffic
├─ Example: TG-Database-Servers
│  ├─ Contains: Target EC2/IP instances
│  ├─ Health: Configured health checks
│  └─ Port: Target backend port

Selection:
├─ Pick: Your target group
├─ Example: database-servers-tg
└─ Traffic: Routes to these targets
```

### Step 7: Complete NLB Listener Creation

**Finalizing**

```
Final Steps:

1. Review Settings:
   ├─ Protocol: TLS ✓
   ├─ Port: 443 ✓
   ├─ Certificate: Selected ✓
   ├─ Security policy: Chosen ✓
   ├─ ALPN: Configured ✓
   └─ Action: Forward to TG ✓

2. Click: "Add listener" or "Save"
   ├─ Validation: AWS validates
   ├─ Creation: Listener is created
   ├─ Time: Usually immediate
   └─ Status: Active

3. Listener Now Active:
   ├─ TLS 443: Now listening
   ├─ Accepts: TLS/HTTPS connections
   ├─ Certificate: Loaded and ready
   ├─ Routes: To target group
   └─ Clients: Can connect via TLS

Result:
├─ NLB listens on: 443 (TLS)
├─ Clients use: TLS/HTTPS protocol
├─ Certificate used: ACM certificate
├─ Encryption: Enabled
└─ Backend: TCP (unencrypted in VPC)
```

---

## Part 5: NLB Additional Configurations

### Adding Multiple TLS Listeners

**For Multiple Certificates or Ports**

```
Scenario 1: Multiple Domains (Similar to ALB):

If NLB needs: Multiple domain certificates
├─ NLB approach: Multiple listeners
├─ Each listener: Different port typically
│  ├─ Listener 1: TLS 443 (www.example.com)
│  ├─ Listener 2: TLS 8443 (api.example.com)
│  └─ Each listener: Own certificate
│
└─ Or: Same port with SNI (newer approach)

Scenario 2: Multiple Services:

Different Backends:
├─ Service 1: TLS 443 → www.example.com → TG1
├─ Service 2: TLS 8443 → api.example.com → TG2
├─ Service 3: TLS 9443 → admin.example.com → TG3
└─ Each listener: Own certificate and port

Adding Additional Listener:

1. Click: "Add listener" again
   └─ Second listener dialog opens

2. Configure:
   ├─ Protocol: TLS
   ├─ Port: Different port (e.g., 8443)
   ├─ Certificate: Different cert if needed
   ├─ ALPN: Configure as needed
   └─ Target: Different target group

3. Result:
   ├─ NLB now listens: On both ports
   ├─ Each port: Own certificate
   ├─ Each service: Separate handling
   └─ Clients: Can access multiple services

Configuration Example:

NLB with Multiple TLS Listeners:

Listener 1:
├─ Protocol: TLS
├─ Port: 443
├─ Cert: www.example.com
└─ Forward to: TG1 (Web servers)

Listener 2:
├─ Protocol: TLS
├─ Port: 8443
├─ Cert: api.example.com
└─ Forward to: TG2 (API servers)

Listener 3:
├─ Protocol: TLS
├─ Port: 9443
├─ Cert: admin.example.com
└─ Forward to: TG3 (Admin servers)

Clients Connect:

Web Client:
├─ Connect to: example.com:443
├─ NLB receives: TLS on 443
├─ Certificate: www.example.com supplied
├─ Routed to: TG1
└─ Response: Web server response

API Client:
├─ Connect to: api.example.com:8443
├─ NLB receives: TLS on 8443
├─ Certificate: api.example.com supplied
├─ Routed to: TG2
└─ Response: API server response

Admin Client:
├─ Connect to: admin.example.com:9443
├─ NLB receives: TLS on 9443
├─ Certificate: admin.example.com supplied
├─ Routed to: TG3
└─ Response: Admin server response
```

---

## Part 6: Certificate Management

### Updating Certificates

**When Certificate Expires or Needs Renewal**

```
Certificate Lifecycle:

1. Certificate In Use:
   ├─ Expiration: 1 year (typical)
   ├─ Attached to: ALB/NLB listener
   └─ Status: Active

2. Before Expiration (30 days):
   ├─ Renewal needed: Get new certificate
   ├─ Option A: ACM auto-renews (if enabled)
   │  └─ New cert: Automatically available
   │
   └─ Option B: Manual renewal
      └─ Request: New cert from CA
      └─ Import: To ACM

3. Update Load Balancer:
   ├─ Edit: Existing listener
   ├─ Select: New certificate
   ├─ Save: Changes
   └─ Effect: Immediate

4. Old Certificate:
   ├─ Still available: In ACM
   ├─ No longer used: By this listener
   ├─ Can delete: Optional
   └─ Or keep: For reference

Update Process in Console:

1. Open Load Balancer
   └─ ALB or NLB

2. Go to: Listeners tab

3. Find: HTTPS/TLS listener

4. Click: Edit listener

5. Find: Certificate section

6. Select: New certificate

7. Save: Changes

8. Verify: Listener active with new cert

Result:
├─ New certificate: Now in use
├─ Clients: Get new certificate
├─ No downtime: Immediate switchover
└─ Old certificate: Unused
```

### Viewing Certificate Details

**Monitoring Certificate Expiration**

```
In ACM Dashboard:

1. Navigate to: AWS Certificate Manager
   └─ Services → Security → Certificate Manager

2. View Certificates:
   ├─ List: All certificates
   ├─ Column: Domain name
   ├─ Column: Expiration date
   ├─ Column: Status (Active/Expired/etc.)
   └─ In use certificates: Marked clearly

3. For Each Certificate:
   ├─ Click: Certificate details
   ├─ Shows: Associated load balancers
   ├─ Shows: Used on which listeners
   ├─ Shows: Renewal status (if auto-renew)
   └─ Shows: Validation status

4. Renewal Status:
   ├─ If auto-renew: Enabled
   │  ├─ Status: Will renew automatically
   │  ├─ New cert: Issued 30 days before expiration
   │  └─ No action: Usually needed
   │
   └─ If manual: You manage
      ├─ Status: Will expire on [date]
      ├─ Action needed: Renew before expiration
      └─ Reminder: Set calendar alert

Monitoring Best Practices:
├─ Enable: Auto-renewal when possible
├─ Remember: Manual expiration dates (if not auto)
├─ Set: Calendar alerts 30 days before expiration
├─ Monitor: ACM dashboard regularly
└─ Plan: Renewal process proactively
```

---

## Part 7: Testing HTTPS/TLS Connection

### Verifying Configuration

**Confirming Everything Works**

```
Test 1: Certificate is Loaded

Using Browser:

1. Open browser
   └─ Chrome, Firefox, Safari, etc.

2. Navigate to: https://your-domain.com
   └─ URL: Use HTTPS (not HTTP)

3. Check: Green lock icon
   ├─ If green: Certificate is valid ✓
   ├─ If red/warning: Problem with cert ✗
   └─ Click: Lock icon to see details

4. View Certificate:
   ├─ Click: Green lock icon
   ├─ Option: "Certificate" or "Connection is secure"
   ├─ Shows: Certificate details
   │  ├─ Issued to: Domain name
   │  ├─ Issued by: Certificate Authority (e.g., ACM)
   │  ├─ Valid from: Start date
   │  ├─ Valid until: Expiration date
   │  └─ Public key info: Displayed
   └─ Verify: Domain matches, not expired

5. Common Issues:

   Red Warning (Certificate Not Trusted):
   ├─ Reason: Cert not recognized
   ├─ Check: Domain name matches URL
   ├─ Check: Certificate validity date
   ├─ Check: Certificate is in ACM/attached
   └─ Solution: Verify cert selection

   Connection Refused:
   ├─ Reason: Port 443 not open
   ├─ Check: Security group allows 443 inbound
   ├─ Check: Load balancer listening on 443
   ├─ Check: Network ACLs allow traffic
   └─ Solution: Fix security group rules

   Timeout/No Response:
   ├─ Reason: Traffic not reaching LB
   ├─ Check: DNS resolves correctly
   ├─ Check: LB is running
   ├─ Check: Target group has healthy targets
   └─ Solution: Verify LB health
```

### Command-Line Testing

**Using Terminal Tools**

```
Test 2: Certificate Details via Command Line

Command 1: OpenSSL s_client (View Certificate)

Syntax:
$ openssl s_client -connect your-domain.com:443 -tls1_2

Example:
$ openssl s_client -connect www.example.com:443 -tls1_2

Output:
├─ TLS version: Shows TLS 1.2 negotiated
├─ Cipher: Shows encryption algorithm used
├─ Certificate: Full cert details displayed
├─ Issuer: Certificate Authority name
├─ Subject: Domain the cert is for
├─ Validity: Not before / Not after dates
└─ Public key: Size and algorithm

Check:
├─ Domain matches: ✓ Good
├─ Expiration: Not expired ✓ Good
├─ Issuer: AWS/ACM ✓ Good
└─ If error: Problem with configuration

Command 2: curl with Verbose

Syntax:
$ curl -v https://your-domain.com

Example:
$ curl -v https://www.example.com

Output:
├─ Connection: Shows TLS 1.2 established
├─ Certificate: Full details displayed
├─ HTTP response: HTTP/1.1 200 OK or similar
├─ Headers: Response headers shown
└─ If error: Shows why connection failed

Check:
├─ TLS established: ✓ Encryption working
├─ Status 200: ✓ Backend responding
├─ Content: Page loads successfully ✓ Good

Command 3: Check Expiration Only

Syntax:
$ openssl s_client -connect your-domain.com:443 2>/dev/null | \
  openssl x509 -noout -dates

Output:
├─ notBefore=Jan 1, 2024 00:00:00 GMT
└─ notAfter=Jan 1, 2025 00:00:00 GMT

Check:
├─ Not expired: ✓ Good
├─ Validity: Shows remaining time
└─ Act: Renew before notAfter date
```

### SNI Testing (Multi-Domain)

**Verifying SNI Works**

```
If Using Multiple Certificates with SNI:

Test: Confirm SNI Certificate Selection

Using OpenSSL:

Command 1: Connect without specifying hostname

$ openssl s_client -connect ALB-IP:443

Result:
├─ Gets: Default certificate
└─ May not: Be the one you want

Command 2: Connect WITH SNI (specifying hostname)

$ openssl s_client -connect ALB-IP:443 -servername www.example.com

Result:
├─ Gets: www.example.com certificate
└─ SNI: Correctly selects cert

Command 3: Connect to different domain on same ALB

$ openssl s_client -connect ALB-IP:443 -servername api.example.com

Result:
├─ Gets: api.example.com certificate
└─ SNI: Selects different cert

Verification:

For each domain tested:
├─ Check: Certificate domain matches request
├─ Verify: Correct certificate returned
├─ Confirm: SNI working properly
└─ Result: Multiple certs on one LB working

Success Indicators:
├─ Different domains: Get different certs
├─ Same IP: Different certificates returned
├─ SNI: Working correctly
└─ Configuration: Correct
```

---

## Part 8: Common Issues and Troubleshooting

### Certificate Issues

```
Issue 1: Certificate Mismatch Error

Symptom:
├─ Browser: Shows "Certificate doesn't match domain"
├─ Connection: Refused or warned
└─ User: Sees security warning

Causes:
├─ Domain mismatch: Cert for different domain
├─ SNI not working: Wrong cert selected
├─ Virtual hosting: Cert doesn't match hostname
└─ Typo: In certificate domain

Resolution:
1. Check certificate domain:
   └─ Verify cert matches domain in URL
   
2. Verify SNI:
   └─ Test: openssl s_client -servername
   └─ Check: Correct cert returned
   
3. Check listener configuration:
   └─ Go to: Listener settings
   └─ Verify: Certificate matches expected domain
   
4. For multiple domains:
   └─ Add: Correct certificates to listener
   └─ Configure: Routing rules match domains

5. If self-signed:
   └─ Browser: Always shows warning
   └─ Normal: For internal/testing
   └─ Solution: Use CA-signed cert

Prevention:
├─ Request certs: For exact domain used
├─ Test: Before deploying
├─ Monitor: Certificate expiration
└─ Automate: Use ACM auto-renewal
```

### Listener and Port Issues

```
Issue 2: HTTPS Listener Not Working

Symptom:
├─ Connection: Refused or timeout
├─ Browser: Can't connect
└─ Error: Connection timeout

Causes:
├─ Security group: Doesn't allow 443 inbound
├─ Listener: Not configured or not active
├─ Certificate: Not selected or invalid
├─ Backend: No healthy targets

Resolution:
1. Check security group:
   └─ Inbound rule: Must allow 443
   └─ From: Your client IP or 0.0.0.0/0
   
2. Verify listener:
   └─ Console: Shows listener active
   └─ Protocol: Must be HTTPS/TLS
   └─ Port: Must be 443 (or your chosen port)
   
3. Check certificate:
   └─ Certificate: Must be selected
   └─ Certificate: Must be valid
   └─ Expiration: Must be in future
   
4. Verify targets:
   └─ Health check: Show targets healthy
   └─ If unhealthy: Fix target health first
   └─ Then: HTTPS should work

5. Test connectivity:
   └─ From client: ping LB IP
   └─ From client: telnet LB-IP 443
   └─ Response: Shows if port accessible

Verification:
├─ telnet should work: Shows open port
├─ openssl should connect: Shows listening
├─ Browser should: Green lock
└─ All working: Configuration correct
```

### Performance Issues

```
Issue 3: Slow HTTPS Connection

Symptom:
├─ HTTPS: Slower than HTTP
├─ Time to first byte: High
├─ Clients: Report slowness

Causes:
├─ TLS negotiation: Takes extra handshake
├─ Security policy: Too strict/conservative
├─ Certificate: Large or complex
├─ Backend: Actually slow (not HTTPS)

Resolution:
1. Verify it's HTTPS related:
   └─ Compare: HTTP vs HTTPS timing
   └─ If both slow: Not HTTPS issue
   └─ If only HTTPS: Continue troubleshooting
   
2. Check TLS version:
   └─ OpenSSL: Show negotiated version
   └─ Should be: TLS 1.2 or 1.3
   └─ If TLS 1.0: Upgrade security policy
   
3. Review security policy:
   └─ Current: Which AWS policy in use
   └─ Option: Switch to FS (forward secure)
   └─ Then: Retest performance
   
4. Enable session resumption:
   └─ Reduces: TLS handshake overhead
   └─ Usually: Enabled by default
   └─ Check: In security policy details
   
5. Scale backend:
   └─ If slow: May be computation / capacity
   └─ Add instances: If needed
   └─ Monitor: Target CPU usage

Note:
├─ HTTPS overhead: Normal, expected
├─ Modern: Usually imperceptible to users
├─ Worth it: Security benefits outweigh
└─ Optimize: If truly problematic
```

---

## Part 9: Best Practices

### Configuration Best Practices

```
1. Certificate Management:
   ├─ Use: AWS Certificate Manager (ACM)
   ├─ Enable: Automatic renewal
   ├─ Monitor: Expiration dates
   ├─ Import: Existing certs if needed
   └─ Avoid: IAM for new deployments

2. Security Policy:
   ├─ Use: AWS recommended policies
   ├─ Support: TLS 1.2 minimum
   ├─ Avoid: Legacy TLS 1.0/1.1
   ├─ Test: With your clients
   └─ Prefer: TLS 1.3 when available

3. Multiple Domains:
   ├─ Use: ALB or NLB (not CLB)
   ├─ Prefer: SNI for multi-domain
   ├─ Add: All certs to listener
   ├─ Configure: Hostname routing rules
   └─ Verify: SNI working correctly

4. Listener Configuration:
   ├─ HTTPS listener: Port 443 standard
   ├─ Maintain: HTTP listener for redirect
   ├─ HTTP listener: Redirect to HTTPS
   ├─ All users: Go to HTTPS
   └─ Redirect: Automatic or manual

5. Backend Configuration:
   ├─ Backend: Use HTTP (VPC is secure)
   ├─ No certs: Needed on EC2 instances
   ├─ Simpler: Reduces complexity
   ├─ Performance: Offloads encryption work
   └─ Secure enough: Private VPC network

Recommended Architecture:

Internet
    ↓
Clients (HTTP and HTTPS)
    ↓
┌────────────────────────┐
│  Load Balancer (ALB)   │
├────────────────────────┤
│ Listener HTTP 80       │ → Redirect to HTTPS
│ Listener HTTPS 443     │ → SSL termination
│ Certificate loaded     │ → ACM certificate
└────────┬───────────────┘
         ↓
    Private VPC
         ↓
  EC2 Instances
  (HTTP 80 only)
  (No certs needed)
```

### Operational Best Practices

```
1. Pre-Deployment:
   ├─ Request certificate: 1 month before need
   ├─ Validate domain: Verify ownership
   ├─ Test connection: From staging client
   ├─ Verify certificate: Domain matches URL
   └─ Plan cutover: Minimal disruption

2. During Deployment:
   ├─ Update listener: Select new certificate
   ├─ Monitor: Check no errors in logs
   ├─ Test: From real clients
   ├─ Verify: Lock icon appears in browser
   └─ Announce: To users if relevant

3. Post-Deployment:
   ├─ Monitor: Connection logs
   ├─ Check: Error rates
   ├─ Verify: Performance acceptable
   ├─ Document: Configuration for future
   └─ Archive: Certificate for backup

4. Ongoing Management:
   ├─ Monitor: Certificate expiration
   ├─ Plan: Renewal 30 days before
   ├─ Update: New certificate on schedule
   ├─ Test: After each renewal
   └─ Automate: Use ACM auto-renewal

5. Incident Response:
   ├─ Alert: If cert expires unexpectedly
   ├─ Emergency: Have backup cert ready
   ├─ Fix: Replace expired cert quickly
   ├─ Communicate: To affected services
   └─ Review: How to prevent next time

Security Reminders:
├─ Encryption: Only between client and LB
├─ Backend: Additional security if needed
├─ VPC: Provides security for backend
├─ Never: Ignore certificate warnings
└─ Always: Use HTTPS for sensitive data
```

---

## Part 10: Exam Focus Points

### Key Exam Questions

```
1. "Where is SSL termination performed?"
   A) EC2 instances
   B) Load balancer
   C) Application layer
   D) VPC router
   
   Answer: B) Load balancer
   └─ LB handles HTTPS encryption
   └─ Backend communicates via HTTP
   └─ Offloads encryption work

2. "Which load balancers support SNI?"
   A) CLB only
   B) ALB and NLB
   C) NLB only
   D) All load balancers equally
   
   Answer: B) ALB and NLB
   └─ ALB: Full SNI support
   └─ NLB: Full SNI support
   └─ CLB: No SNI support

3. "How many SSL certificates can one ALB support?"
   A) 1 only
   B) 5 maximum
   C) 10 maximum
   D) Multiple via SNI
   
   Answer: D) Multiple via SNI
   └─ ALB: Multiple certificates
   └─ SNI enables certificate selection
   └─ No practical limit

4. "For HTTPS listener, certificate source?"
   A) Must use ACM only
   B) Must import from CA
   C) ACM, IAM, or import
   D) IAM is recommended
   
   Answer: C) ACM, IAM, or import
   └─ ACM: Recommended (free, auto-renew)
   └─ IAM: Legacy, not recommended
   └─ Import: Option if have existing cert

5. "Backend communication after SSL termination?"
   A) Must also be HTTPS
   B) Can be HTTP (VPC private)
   C) Must be encrypted
   D) Requires separate cert
   
   Answer: B) Can be HTTP (VPC private)
   └─ VPC: Private network
   └─ HTTP: Sufficient and standard
   └─ No certs: Needed on EC2

6. "Setting multiple domains on one ALB?"
   A) Requires multiple ALBs
   B) Use ALB with multiple listeners
   C) ALB with SNI and multiple certs
   D) Not possible
   
   Answer: C) ALB with SNI and multiple certs
   └─ SNI: Enables multi-domain
   └─ Multiple certificates: Attached to listener
   └─ Routing rules: Direct to target groups

7. "Certificate expiration handling?"
   A) Automatic renewal always
   B) Manual renewal always
   C) ACM enables auto-renewal
   D) Not possible to renew
   
   Answer: C) ACM enables auto-renewal
   └─ ACM: Can auto-renew before expiration
   └─ Monitoring: Recommended for all
   └─ Planning: Renew 30 days before

8. "NLB HTTPS configuration called what?"
   A) HTTP listener
   B) SSL listener
   C) TLS listener
   D) HTTPS listener
   
   Answer: C) TLS listener
   └─ NLB: Uses term "TLS"
   └─ ALB: Uses term "HTTPS"
   └─ Both: Same technology
   └─ Different: Terminology only
```

### Terms to Remember

```
Key Terms:

1. SSL Termination
   └─ Decryption of HTTPS at load balancer
   └─ Backend: Receives HTTP
   └─ Location: Happens at LB

2. SNI (Server Name Indication)
   └─ Protocol: Tells server hostname
   └─ Enables: Multiple certs per server
   └─ Requirement: For multi-domain LB

3. In-Flight Encryption
   └─ Encryption: While data travels
   └─ Network: Cannot read encrypted data
   └─ Endpoints: Can decrypt

4. ACM (AWS Certificate Manager)
   └─ Service: Manages certificates
   └─ Cost: Free for public certs
   └─ Renewal: Automatic available

5. Security Policy
   └─ Control: Which TLS versions allowed
   └─ Ciphers: Encryption algorithms
   └─ Strength: Security level

6. Certificate Chain
   └─ Multiple: Certificates in hierarchy
   └─ Root CA: Top of chain
   └─ Leaf cert: Your domain certificate

7. ALPN (Application Layer Protocol Negotiation)
   └─ Advanced: TLS feature
   └─ Protocol: Negotiated during handshake
   └─ Usually: Default works fine
```

---

## Part 11: Summary

### Complete Configuration Checklist

```
Before Going Live with HTTPS:

Preparation:
☐ Certificate obtained (from ACM or CA)
☐ Certificate domain validates your use case
☐ Certificate expiration date noted (30-day reminder set)
☐ Backend instances running healthily
☐ Target groups configured
☐ Security groups allow 443 inbound
☐ Security groups allow 80 (for HTTP redirect)

ALB Configuration (if using ALB):
☐ ALB created and running
☐ HTTPS listener added (port 443)
☐ Certificate selected from ACM
☐ Security policy chosen
☐ Target group selected
☐ HTTP listener configured for redirect
☐ Listener activated and showing "Active"

NLB Configuration (if using NLB):
☐ NLB created and running
☐ TLS listener added (port 443)
☐ Certificate selected from ACM
☐ Security policy chosen
☐ ALPN configured
☐ Target group selected
☐ Listener activated and showing "Active"

Multi-Domain Setup (if needed):
☐ Additional certificates added to listener
☐ Hostname-based routing rules created
☐ Each domain routes to correct target group
☐ SNI tested and working
☐ Each domain certificate selected correctly

Testing:
☐ HTTPS://yourdomain.com loads in browser
☐ Green lock icon appears
☐ Certificate details visible and correct
☐ HTTP redirects to HTTPS (if configured)
☐ Backend instances receive connections
☐ Response times acceptable
☐ Security policy negotiated correctly
☐ No error logs in CloudWatch

Monitoring:
☐ Certificate expiration: 30-day reminder set
☐ CloudWatch logs: Monitored for errors
☐ ACM auto-renewal: Enabled if applicable
☐ Target group health: Monitored
☐ Listener status: Showing Active

Documentation:
☐ Certificate ARN: Documented
☐ Security policy: Noted
☐ Routing rules: Documented (if complex)
☐ Target groups: Listed
☐ Renewal procedure: Documented
☐ Escalation contacts: Listed

Go-Live:
☐ All checks passed
☐ Team notified: Of configuration
☐ Users informed: If relevant
☐ Monitoring active
☐ On-call support: Available
☐ Rollback plan: In place (just in case)

Result:
├─ HTTPS: Active and working
├─ Clients: Encrypted connections
├─ Backend: HTTP still used
├─ Certificates: Managing automatically
└─ Success: HTTPS deployment complete
```

Your HTTPS/TLS configuration is now complete and ready for production!