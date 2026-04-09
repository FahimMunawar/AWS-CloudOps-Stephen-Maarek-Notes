# Launching EC2 Instance

This note summarizes the AWS EC2 instance launch process from the lecture transcript.
It focuses on Amazon Linux, key pair creation, security group setup, and connection methods.

## 1. Launching the Instance

- Name the instance: `MyFirstInstance`
- Choose an Amazon Linux AMI
- Select instance type: `t2.micro`
  - `t2.micro` is eligible for AWS Free Tier
- Review default settings and storage before launching

## 2. Create / Use a Key Pair

- Select or create a new key pair for SSH access
- In the transcript, the instructor created a new key pair called `DemoKeyPair`
- Choose `RSA` and download the `.pem` file
- Store the private key in a secure local directory, for example `~/Downloads`
- Set secure permissions:
  - `chmod 400 DemoKeyPair.pem`
- The private key is required for SSH access to the EC2 instance

## 3. Security Group Settings

- Create a new security group named `AWSSSH`
- Add an inbound rule for SSH:
  - Protocol: `TCP`
  - Port: `22`
  - Source: `0.0.0.0/0` (or better: your specific IP address)
- Security group rules control access to the instance
- Exam note: for SysOps, always prefer least privilege and restrict SSH access to known IPs when possible

## 4. Launch the Instance

- After verifying storage and network settings, launch the instance
- Wait until the instance state is `running`
- Copy the instance Public IPv4 address for connection

## 5. Connect via SSH

- Change to the directory containing the private key
  - Example: `cd ~/Downloads`
- Connect with SSH using the `ec2-user` account for Amazon Linux:
  - `ssh -i DemoKeyPair.pem ec2-user@<public-ipv4-address>`
- The first connection may prompt:
  - `Are you sure you want to continue connecting (yes/no/[fingerprint])?`
  - Type `yes`
- Once connected, run a test command:
  - `echo "hello world"`

## 6. Connect via EC2 Instance Connect

- EC2 Instance Connect is an alternate browser-based connection method
- Select `EC2 Instance Connect` in the console
- The suggested username for Amazon Linux is `ec2-user`
- Click `Connect` to open a shell session in the browser

## 7. Key Takeaways for SysOps Associate

- Understand how to launch an EC2 instance using the console
- Know how to create and use key pairs for SSH access
- Know which security group rule is required for SSH
- Be able to connect using both SSH and EC2 Instance Connect
- Remember that instance connect uses temporary SSH keys managed by AWS

