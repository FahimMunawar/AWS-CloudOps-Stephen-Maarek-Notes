# 12 — Bastion Hosts (Hands-On)

## Goal

SSH into a private EC2 instance via a bastion host in a public subnet.

---

## Step 1: Create a Key Pair

**EC2 → Key Pairs → Create key pair** → name: `DemoKeyPair` → PEM format → save the `.pem` file locally.

> Required for SSH into both the bastion host and the private instance.

---

## Step 2: Launch Private EC2 Instance

| Setting | Value |
|---|---|
| AMI | Amazon Linux 2 |
| Instance type | t2.micro |
| Key pair | DemoKeyPair |
| VPC | DemoVPC |
| Subnet | PrivateSubnetA |
| Security group | `PrivateSG` |

**PrivateSG inbound rule:**

| Port | Source |
|---|---|
| 22 (SSH) | Bastion Host Security Group (NOT `0.0.0.0/0`) |

> EC2 Instance Connect will NOT work for private instances — no IGW route, no public IP.

---

## Step 3: Connect to Bastion Host

Use EC2 Instance Connect (or terminal SSH) to connect to the bastion host in the public subnet.

---

## Step 4: Copy Key to Bastion Host

The bastion host needs the private key to SSH onward to the private instance:

```bash
# On bastion host — create the key file
vi demo_key_pair.pem
# Paste contents of DemoKeyPair.pem → save

# Fix permissions (required — SSH rejects unprotected key files)
chmod 400 demo_key_pair.pem
```

> If the key file has bad formatting (line endings), SSH will fail with "bad permissions" or passphrase errors — recreate with `vi` if needed.

---

## Step 5: SSH into Private Instance

```bash
# From bastion host
ssh -i demo_key_pair.pem ec2-user@<private-instance-private-ip>
```

Traffic path:
```
Your computer → SSH → Bastion Host (public subnet) → SSH → Private EC2 (private subnet)
```

---

## Verification

```bash
# On private EC2 instance
ping google.com   # FAILS — private subnet has no outbound internet access
```

Private EC2 has no internet access (no NAT Gateway yet) — covered in next lecture.

---

## Quick Reference

```
Bastion Hands-On Summary:
  1. Create key pair (PEM) → save locally
  2. Launch private EC2 in PrivateSubnetA → PrivateSG allows SSH from bastion SG only
  3. Connect to bastion via EC2 Instance Connect or SSH
  4. Copy .pem key to bastion → chmod 400
  5. SSH from bastion → private EC2 using private IP

Key file troubleshooting:
  - chmod 400 required — SSH rejects world-readable keys
  - Use vi for clean formatting if paste introduces encoding issues

Private EC2 has no internet (no NAT) → ping google.com fails
```
