# AWS CLI Commands

---

# Introduction

The AWS Command Line Interface (AWS CLI) is a unified tool for managing AWS services from the command line.

It allows you to

- Create Resources
- Update Resources
- Delete Resources
- Automate Infrastructure
- Script Operations
- Troubleshoot Production Systems

AWS CLI is widely used by DevOps Engineers, Cloud Engineers, Platform Engineers, and Site Reliability Engineers.

---

# Check AWS CLI Version

```bash
aws --version
```

Example Output

```text
aws-cli/2.15.10 Python/3.x Linux
```

---

# Configure AWS CLI

```bash
aws configure
```

Prompts

```text
AWS Access Key ID

AWS Secret Access Key

Default Region

Output Format
```

---

# Configure Named Profile

```bash
aws configure --profile dev
```

---

# List Configured Profiles

```bash
aws configure list-profiles
```

---

# Show Current Configuration

```bash
aws configure list
```

---

# View Configuration File

Linux

```bash
cat ~/.aws/config
```

Windows

```text
C:\Users\<username>\.aws\config
```

---

# View Credentials File

Linux

```bash
cat ~/.aws/credentials
```

---

# Change Default Region

```bash
aws configure set region ap-south-1
```

---

# Use Specific Profile

```bash
aws s3 ls --profile dev
```

---

# Get Current IAM Identity

```bash
aws sts get-caller-identity
```

Example Output

```json
{
  "UserId": "...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/admin"
}
```

---

# Assume IAM Role

```bash
aws sts assume-role \
--role-arn arn:aws:iam::123456789012:role/AdminRole \
--role-session-name DevSession
```

---

# List AWS Regions

```bash
aws ec2 describe-regions
```

Only Region Names

```bash
aws ec2 describe-regions \
--query "Regions[].RegionName"
```

---

# List Availability Zones

```bash
aws ec2 describe-availability-zones
```

---

# Output Formats

JSON

```bash
--output json
```

Table

```bash
--output table
```

Text

```bash
--output text
```

Example

```bash
aws ec2 describe-instances --output table
```

---

# Filtering Results

Example

```bash
aws ec2 describe-instances \
--filters Name=instance-state-name,Values=running
```

---

# Query Output

Example

```bash
aws ec2 describe-instances \
--query "Reservations[].Instances[].InstanceId"
```

---

# Pagination

```bash
--max-items 10
```

---

# Help

```bash
aws help
```

Service Help

```bash
aws ec2 help
```

Command Help

```bash
aws ec2 describe-instances help
```

---

# EC2 Commands

---

# List EC2 Instances

```bash
aws ec2 describe-instances
```

---

# List Running Instances

```bash
aws ec2 describe-instances \
--filters Name=instance-state-name,Values=running
```

---

# List Stopped Instances

```bash
aws ec2 describe-instances \
--filters Name=instance-state-name,Values=stopped
```

---

# Show Instance IDs

```bash
aws ec2 describe-instances \
--query "Reservations[].Instances[].InstanceId"
```

---

# Show Public IPs

```bash
aws ec2 describe-instances \
--query "Reservations[].Instances[].PublicIpAddress"
```

---

# Show Private IPs

```bash
aws ec2 describe-instances \
--query "Reservations[].Instances[].PrivateIpAddress"
```

---

# Show Instance Names

```bash
aws ec2 describe-instances \
--query "Reservations[].Instances[].Tags[?Key=='Name'].Value"
```

---

# Describe Specific Instance

```bash
aws ec2 describe-instances \
--instance-ids i-0123456789abcdef0
```

---

# Launch EC2 Instance

```bash
aws ec2 run-instances \
--image-id ami-xxxxxxxx \
--instance-type t3.micro \
--key-name my-key \
--security-group-ids sg-xxxxxxxx \
--subnet-id subnet-xxxxxxxx
```

---

# Stop Instance

```bash
aws ec2 stop-instances \
--instance-ids i-0123456789abcdef0
```

---

# Start Instance

```bash
aws ec2 start-instances \
--instance-ids i-0123456789abcdef0
```

---

# Reboot Instance

```bash
aws ec2 reboot-instances \
--instance-ids i-0123456789abcdef0
```

---

# Terminate Instance

```bash
aws ec2 terminate-instances \
--instance-ids i-0123456789abcdef0
```

---

# Describe Instance Status

```bash
aws ec2 describe-instance-status
```

---

# Get Console Output

```bash
aws ec2 get-console-output \
--instance-id i-0123456789abcdef0
```

---

# Get System Logs

```bash
aws ec2 get-console-output \
--instance-id i-0123456789abcdef0 \
--latest
```

---

# Monitor Instance

```bash
aws ec2 monitor-instances \
--instance-ids i-0123456789abcdef0
```

---

# Disable Detailed Monitoring

```bash
aws ec2 unmonitor-instances \
--instance-ids i-0123456789abcdef0
```

---

# Modify Instance Type

```bash
aws ec2 modify-instance-attribute \
--instance-id i-0123456789abcdef0 \
--instance-type "{\"Value\":\"t3.large\"}"
```

---

# Create AMI

```bash
aws ec2 create-image \
--instance-id i-0123456789abcdef0 \
--name Production-AMI
```

---

# List AMIs

```bash
aws ec2 describe-images --owners self
```

---

# Deregister AMI

```bash
aws ec2 deregister-image \
--image-id ami-xxxxxxxx
```

---

# Create Snapshot

```bash
aws ec2 create-snapshot \
--volume-id vol-xxxxxxxx
```

---

# List Snapshots

```bash
aws ec2 describe-snapshots \
--owner-ids self
```

---

# Delete Snapshot

```bash
aws ec2 delete-snapshot \
--snapshot-id snap-xxxxxxxx
```

---

# List Volumes

```bash
aws ec2 describe-volumes
```

---

# Attach Volume

```bash
aws ec2 attach-volume \
--volume-id vol-xxxxxxxx \
--instance-id i-xxxxxxxx \
--device /dev/xvdf
```

---

# Detach Volume

```bash
aws ec2 detach-volume \
--volume-id vol-xxxxxxxx
```

---

# Create Volume

```bash
aws ec2 create-volume \
--availability-zone ap-south-1a \
--size 20 \
--volume-type gp3
```

---

# Delete Volume

```bash
aws ec2 delete-volume \
--volume-id vol-xxxxxxxx
```

---

# Best Practices

- Use named profiles instead of sharing credentials.
- Prefer IAM Roles over Access Keys on EC2.
- Use `--query` to filter large outputs.
- Use `--output table` for readability.
- Avoid hardcoding credentials in scripts.
- Validate commands in non-production environments first.
- Use least-privilege IAM permissions for CLI users.
- Enable MFA for privileged accounts.

---

# Summary

This section covered AWS CLI fundamentals, configuration, authentication, AWS STS, output formatting, filtering, querying, and essential EC2 operations including instance management, AMIs, EBS volumes, snapshots, and monitoring. These commands form the foundation for automating and managing AWS resources from the command line.

---

