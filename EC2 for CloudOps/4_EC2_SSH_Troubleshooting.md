# EC2 SSH Troubleshooting

This note summarizes common SSH and EC2 Instance Connect issues for EC2 instances.
It focuses on permissions, usernames, networking, and how EC2 Instance Connect differs from direct SSH.

## 1. Common SSH Issues

### 1.1 Unprotected Private Key File
- If your key file permissions are too open, SSH returns:
  - `unprotected private key file`
- Fix the permissions before connecting:
  - `chmod 400 DemoKeyPair.pem`
- SSH requires private keys to be protected from other users on your system.

### 1.2 Wrong SSH Username
- You must use the correct username for the AMI/OS:
  - Amazon Linux: `ec2-user`
  - Ubuntu: `ubuntu`
  - RHEL: `ec2-user` or `root`
  - CentOS: `centos`
  - Debian: `admin` or `debian`
- If the username is incorrect, you may see:
  - `permission denied`
  - `too many authentication failures`
  - `connection closed by [instance IP] port 22`
  - `host key not found`
- The username is tied to the AMI, not the key pair.

### 1.3 SSH Connection Timeout
- A timeout is usually a networking issue, not a key issue.
- Possible causes:
  - Security group inbound rule not allowing SSH (TCP port 22)
  - Route table misconfiguration for the subnet
  - Network ACL (NACL) blocking traffic
  - Instance does not have a Public IPv4 address
  - The instance is CPU-bound or unresponsive
- Check networking first when SSH times out.

## 2. SSH vs EC2 Instance Connect

### 2.1 Direct SSH
- Direct SSH uses your local private key file and the EC2 instance inbound SSH rule.
- The source IP must match the allowed SSH rule in the security group.
- Example inbound rule:
  - Type: `SSH`
  - Protocol: `TCP`
  - Port Range: `22`
  - Source: `your-ip-address/32`
- Only allowed IPs can connect.

### 2.2 EC2 Instance Connect
- EC2 Instance Connect uses the AWS API and a one-time public SSH key.
- The public key is pushed to the instance and is valid for 60 seconds.
- You do not provide your private SSH key directly when using Instance Connect.
- The service connects from a specific AWS IP prefix range.

### 2.3 Security Group Requirements for EC2 Instance Connect
- The security group must allow SSH from the EC2 Instance Connect IP range, not just your own IP.
- AWS provides a JSON file of IP ranges for services and regions.
- Example workflow:
  1. Download the AWS IP address ranges JSON file
  2. Filter entries for `EC2 Instance Connect`
  3. Find your region prefix (e.g. `eu-central-1`)
  4. Add the resulting CIDR block to the SSH inbound rule
- Without the correct AWS IP prefix, EC2 Instance Connect will fail.

### 2.4 Why EC2 Instance Connect Works Differently
- With SSH, the connection originates from your local IP.
- With Instance Connect, the connection originates from AWS-managed IP ranges.
- AWS pushes a temporary public key to the instance on your behalf.
- Your client simply requests the connection through the EC2 Instance Connect service.

## 3. Troubleshooting Steps

### 3.1 Verify PEM File Permissions
- Run:
  - `ls -l DemoKeyPair.pem`
  - `chmod 400 DemoKeyPair.pem`

### 3.2 Verify the Correct Username
- Check the AMI documentation or use the appropriate username for the OS
- Example:
  - `ssh -i DemoKeyPair.pem ec2-user@<public-ip>`

### 3.3 Verify Security Group Settings
- Ensure an inbound rule exists for SSH on port 22
- If using direct SSH, source should be your IP or allowed CIDR
- If using EC2 Instance Connect, source should be the EC2 Instance Connect IP prefix for your region

### 3.4 Verify Public IP and Network Path
- Confirm the instance has a public IPv4 address
- Check that the subnet route table allows internet access
- Review network ACLs for inbound/outbound SSH traffic

### 3.5 Verify Instance Health
- If the instance is under heavy CPU load, it may not respond to SSH
- Consider using the EC2 console or CloudWatch metrics to check instance health

## 4. Practical Examples

### 4.1 Fixing Permissions
```bash
chmod 400 DemoKeyPair.pem
ssh -i DemoKeyPair.pem ec2-user@<public-ip>
```

### 4.2 Wrong Username Example
- Incorrect:
  - `ssh -i DemoKeyPair.pem ubuntu@<public-ip>`
- Correct for Amazon Linux:
  - `ssh -i DemoKeyPair.pem ec2-user@<public-ip>`

### 4.3 EC2 Instance Connect Security Group Example
- Allow SSH from the AWS EC2 Instance Connect IP prefix for your region
- Example security group rule:
  - Type: `SSH`
  - Protocol: `TCP`
  - Port: `22`
  - Source: `x.x.x.x/yy` (EC2 Instance Connect CIDR range)

## 5. Key Takeaways for SysOps Associate

- Know the most common SSH failures: bad PEM permissions, wrong username, and network issues
- Distinguish between direct SSH access and EC2 Instance Connect
- Understand that EC2 Instance Connect uses temporary keys and AWS IP ranges
- Be able to diagnose timeout errors by checking security groups, route tables, NACLs, and public IP assignment
- Know how to find the correct SSH username from the AMI
