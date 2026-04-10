# SSM Distributor

This note covers SSM Distributor — a feature for packaging and deploying software to managed instances at scale.

## 1. What is SSM Distributor?

- Packages and deploys software to managed EC2 instances and on-premises servers
- Supports multiple platforms: **Windows and Linux**
- Package types available:
  - **Your own custom packages**
  - **AWS-provided packages** (e.g., CloudWatch agent, SSM agent)
  - **Third-party packages**

## 2. Distributor Package Structure

A package is an **SSM document** whose contents are stored in **Amazon S3**.

One zip file is created **per target OS/platform**, containing:

| File | Purpose |
|---|---|
| `install.sh` / `install.ps1` | Script to install the software |
| `uninstall.sh` / `uninstall.ps1` | Script to remove the software |
| Executable / binary | The software to be installed |
| `manifest.json` | Describes package contents, versions, and target platforms |

## 3. Deployment Options

| Method | Use Case | SSM Document Used |
|---|---|---|
| **Run Command** | One-time installation | `AWS-ConfigureAWSPackage` |
| **State Manager** | Scheduled / recurring installation | `AWS-ConfigureAWSPackage` |

- **Run Command**: Install a package immediately on target instances — a one-off operation
- **State Manager association**: Install (and keep installed) a package on a schedule — ensures instances always have the latest version

## 4. Key Takeaways for SysOps Associate

- Distributor packages are **SSM documents** — their contents (zip files) are stored in **S3**
- Create **one zip per OS platform** (e.g., one for Windows, one for Amazon Linux)
- The document used to install a Distributor package is **`AWS-ConfigureAWSPackage`**
- Use **Run Command** for one-time installs; use **State Manager** for recurring/enforced installs
- Distributor is the answer for exam scenarios asking how to **deploy and maintain software packages** across a fleet of managed instances
