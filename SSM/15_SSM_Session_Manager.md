# SSM Session Manager

This note covers SSM Session Manager — a secure, auditable shell access method for EC2 instances and on-premises servers that requires no SSH, no key pairs, and no open inbound ports.

## 1. What is Session Manager?

- Provides a **secure interactive shell** to EC2 instances and on-premises servers
- Access via: **AWS Console, CLI, or Session Manager SDK**
- Supports: **Linux, macOS, and Windows**
- No SSH keys, no bastion hosts, no open inbound security group rules required

## 2. How it Works

```
User (with IAM permission)
        │
        ▼
Session Manager Service
        │
        │  (same outbound agent mechanism as Run Command)
        ▼
SSM Agent on EC2 Instance
(instance must have SSM IAM role)
```

- The user connects to the **Session Manager service** (not directly to the instance)
- Session Manager relays the shell session through the SSM agent
- The instance needs **no inbound rules** — the agent initiates the outbound connection

## 3. Session Manager vs. SSH vs. EC2 Instance Connect

| Feature | SSH | EC2 Instance Connect | Session Manager |
|---|---|---|---|
| Requires open port 22 | Yes | Yes (backend uses SSH) | No |
| Requires key pair | Yes | No (temporary key) | No |
| Requires bastion host | Sometimes | No | No |
| Session logging | No | No | Yes (S3 / CloudWatch) |
| IAM-based access control | No | Partial | Yes |
| Command restriction | No | No | Optional |
| Audit trail (CloudTrail) | No | No | Yes (`StartSession`) |

## 4. Logging and Auditing

All session activity can be logged to:

| Destination | What is Captured |
|---|---|
| **Amazon S3** | Full session transcript |
| **CloudWatch Logs** | Commands and output, searchable |
| **CloudTrail** | `StartSession` API events — who started a session, on which instance, when |

This gives compliance-grade visibility not available with traditional SSH.

## 5. IAM Permissions

### Instance Side
- EC2 instance must have `AmazonSSMManagedInstanceCore` (or equivalent) attached

### User Side
- IAM policy granting `ssm:StartSession` on target instances
- Optionally restrict by tag — example: only allow sessions on instances tagged `Environment = dev`

```json
{
  "Effect": "Allow",
  "Action": "ssm:StartSession",
  "Resource": "arn:aws:ec2:*:*:instance/*",
  "Condition": {
    "StringEquals": {
      "ssm:resourceTag/Environment": "dev"
    }
  }
}
```

- Additional permissions needed if logging: `s3:PutObject` and `logs:CreateLogStream` / `logs:PutLogEvents`
- **Optional**: Restrict which commands a user can run inside a session (via session document)

## 6. Key Takeaways for SysOps Associate

- Session Manager = **interactive shell with no SSH, no key pair, no open inbound ports**
- EC2 Instance Connect still uses SSH in the backend — Session Manager does not
- Session activity is **fully logged** to S3 and/or CloudWatch Logs — this is the main security advantage over SSH
- **CloudTrail** captures `StartSession` events for auditing who accessed which instance
- Use **IAM conditions on resource tags** to restrict which instances a user can start sessions on
- Session Manager is the preferred answer for any exam scenario asking for **secure, auditable, keyless** EC2 access
- Works for both **EC2 and on-premises** servers, across Linux, macOS, and Windows
