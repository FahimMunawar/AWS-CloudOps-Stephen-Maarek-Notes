# 14 — NAT Instances (Hands-On)

## Goal

Give private EC2 instances outbound internet access by routing traffic through a NAT instance in a public subnet.

---

## Step 1: Launch NAT Instance

**EC2 → Launch instance → Browse AMIs → Community AMIs → search "NAT"**

Select an AWS-published AMI: `amzn-ami-vpc-nat-*` — Architecture: x86_64

| Setting | Value |
|---|---|
| AMI | Community AMI: `amzn-ami-vpc-nat` (x86_64) |
| Instance type | t2.micro |
| Key pair | DemoKeyPair |
| VPC | DemoVPC |
| Subnet | **PublicSubnetA** (must be public) |
| Security group | `NATInstanceSG` |

**NATInstanceSG inbound rules:**

| Port/Protocol | Source |
|---|---|
| SSH (22) | Anywhere (`0.0.0.0/0`) |
| HTTP (80) | VPC CIDR (`10.0.0.0/16`) |
| HTTPS (443) | VPC CIDR (`10.0.0.0/16`) |
| All ICMP - IPv4 | VPC CIDR (`10.0.0.0/16`) — required for ping |

---

## Step 2: Disable Source/Destination Check

**EC2 → Select NAT instance → Actions → Networking → Change source/destination check → Stop**

> By default, EC2 drops packets where it is not the source or destination. NAT rewrites packet IPs, so this check **must be disabled**.

---

## Step 3: Update Private Route Table

**VPC → Route Tables → PrivateRouteTable → Routes → Edit routes → Add route**

| Destination | Target |
|---|---|
| `10.0.0.0/16` | local |
| `0.0.0.0/0` | NAT instance (EC2 instance ID) |

This tells all private subnet traffic destined for the internet to go through the NAT instance.

---

## Verification

```bash
# On Bastion Host → SSH to private EC2
ssh -i demo_key_pair.pem ec2-user@<private-instance-private-ip>

# On private EC2
ping google.com       # works after ICMP rule added to NATInstanceSG
curl example.com      # returns HTML → HTTP/HTTPS confirmed
```

---

## Key Observations

- Private EC2 still has **no public IP** — internet traffic goes out via the NAT instance's public IP only
- Bastion host is still required to SSH into the private instance
- If ICMP (ping) fails after adding the route → add **All ICMP - IPv4** inbound rule to NATInstanceSG from VPC CIDR

---

## Cleanup

Stop or terminate the NAT instance after the demo — NAT Gateway (next lecture) replaces it.

---

## Quick Reference

```
NAT Instance Hands-On Steps:
  1. Launch EC2 from community NAT AMI in PublicSubnetA
     NATInstanceSG: SSH (any), HTTP/HTTPS/ICMP from VPC CIDR (10.0.0.0/16)
  2. Disable source/destination check on the NAT instance
  3. PrivateRouteTable: add 0.0.0.0/0 → NAT instance

Verify: ping google.com + curl example.com from private EC2 (via bastion)
Private EC2 still has no public IP — NAT instance acts as the internet-facing exit point
```
