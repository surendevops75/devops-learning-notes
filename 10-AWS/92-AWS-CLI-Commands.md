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

# Security Groups

---

# List Security Groups

```bash
aws ec2 describe-security-groups
```

---

# Describe Specific Security Group

```bash
aws ec2 describe-security-groups \
--group-ids sg-0123456789abcdef0
```

---

# Create Security Group

```bash
aws ec2 create-security-group \
--group-name web-sg \
--description "Web Security Group" \
--vpc-id vpc-xxxxxxxx
```

---

# Delete Security Group

```bash
aws ec2 delete-security-group \
--group-id sg-xxxxxxxx
```

---

# Add SSH Rule

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-xxxxxxxx \
--protocol tcp \
--port 22 \
--cidr 0.0.0.0/0
```

---

# Add HTTP Rule

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-xxxxxxxx \
--protocol tcp \
--port 80 \
--cidr 0.0.0.0/0
```

---

# Add HTTPS Rule

```bash
aws ec2 authorize-security-group-ingress \
--group-id sg-xxxxxxxx \
--protocol tcp \
--port 443 \
--cidr 0.0.0.0/0
```

---

# Remove Ingress Rule

```bash
aws ec2 revoke-security-group-ingress \
--group-id sg-xxxxxxxx \
--protocol tcp \
--port 22 \
--cidr 0.0.0.0/0
```

---

# Add Outbound Rule

```bash
aws ec2 authorize-security-group-egress \
--group-id sg-xxxxxxxx \
--protocol tcp \
--port 443 \
--cidr 0.0.0.0/0
```

---

# Remove Outbound Rule

```bash
aws ec2 revoke-security-group-egress \
--group-id sg-xxxxxxxx \
--protocol tcp \
--port 443 \
--cidr 0.0.0.0/0
```

---

# Key Pairs

---

# List Key Pairs

```bash
aws ec2 describe-key-pairs
```

---

# Create Key Pair

```bash
aws ec2 create-key-pair \
--key-name dev-key
```

---

# Save Private Key

```bash
aws ec2 create-key-pair \
--key-name dev-key \
--query "KeyMaterial" \
--output text > dev-key.pem
```

---

# Delete Key Pair

```bash
aws ec2 delete-key-pair \
--key-name dev-key
```

---

# Import Existing Public Key

```bash
aws ec2 import-key-pair \
--key-name office-key \
--public-key-material fileb://id_rsa.pub
```

---

# Elastic IP

---

# Allocate Elastic IP

```bash
aws ec2 allocate-address \
--domain vpc
```

---

# List Elastic IPs

```bash
aws ec2 describe-addresses
```

---

# Associate Elastic IP

```bash
aws ec2 associate-address \
--instance-id i-xxxxxxxx \
--allocation-id eipalloc-xxxxxxxx
```

---

# Disassociate Elastic IP

```bash
aws ec2 disassociate-address \
--association-id eipassoc-xxxxxxxx
```

---

# Release Elastic IP

```bash
aws ec2 release-address \
--allocation-id eipalloc-xxxxxxxx
```

---

# Launch Templates

---

# List Launch Templates

```bash
aws ec2 describe-launch-templates
```

---

# Create Launch Template

```bash
aws ec2 create-launch-template \
--launch-template-name web-template \
--version-description v1 \
--launch-template-data file://template.json
```

---

# Describe Launch Template

```bash
aws ec2 describe-launch-template-versions \
--launch-template-name web-template
```

---

# Delete Launch Template

```bash
aws ec2 delete-launch-template \
--launch-template-name web-template
```

---

# Auto Scaling

---

# List Auto Scaling Groups

```bash
aws autoscaling describe-auto-scaling-groups
```

---

# Describe Auto Scaling Group

```bash
aws autoscaling describe-auto-scaling-groups \
--auto-scaling-group-names web-asg
```

---

# Create Auto Scaling Group

```bash
aws autoscaling create-auto-scaling-group \
--auto-scaling-group-name web-asg \
--launch-template LaunchTemplateName=web-template \
--min-size 2 \
--max-size 5 \
--desired-capacity 2 \
--vpc-zone-identifier subnet-1,subnet-2
```

---

# Update Desired Capacity

```bash
aws autoscaling set-desired-capacity \
--auto-scaling-group-name web-asg \
--desired-capacity 3
```

---

# Update Auto Scaling Group

```bash
aws autoscaling update-auto-scaling-group \
--auto-scaling-group-name web-asg \
--max-size 10
```

---

# Delete Auto Scaling Group

```bash
aws autoscaling delete-auto-scaling-group \
--auto-scaling-group-name web-asg \
--force-delete
```

---

# Scaling Policies

---

# List Scaling Policies

```bash
aws autoscaling describe-policies
```

---

# Create Target Tracking Policy

```bash
aws autoscaling put-scaling-policy \
--auto-scaling-group-name web-asg \
--policy-name cpu-policy \
--policy-type TargetTrackingScaling
```

---

# Delete Scaling Policy

```bash
aws autoscaling delete-policy \
--auto-scaling-group-name web-asg \
--policy-name cpu-policy
```

---

# EC2 Tags

---

# List Tags

```bash
aws ec2 describe-tags
```

---

# Create Tag

```bash
aws ec2 create-tags \
--resources i-xxxxxxxx \
--tags Key=Environment,Value=Production
```

---

# Multiple Tags

```bash
aws ec2 create-tags \
--resources i-xxxxxxxx \
--tags \
Key=Environment,Value=Production \
Key=Owner,Value=DevOps
```

---

# Delete Tag

```bash
aws ec2 delete-tags \
--resources i-xxxxxxxx \
--tags Key=Owner
```

---

# Filter by Tag

```bash
aws ec2 describe-instances \
--filters Name=tag:Environment,Values=Production
```

---

# Placement Groups

---

# List Placement Groups

```bash
aws ec2 describe-placement-groups
```

---

# Create Placement Group

```bash
aws ec2 create-placement-group \
--group-name hpc-group \
--strategy cluster
```

---

# Delete Placement Group

```bash
aws ec2 delete-placement-group \
--group-name hpc-group
```

---

# Spot Instances

---

# Request Spot Instance

```bash
aws ec2 request-spot-instances \
--spot-price 0.02 \
--instance-count 1 \
--launch-specification file://spot.json
```

---

# Describe Spot Requests

```bash
aws ec2 describe-spot-instance-requests
```

---

# Cancel Spot Request

```bash
aws ec2 cancel-spot-instance-requests \
--spot-instance-request-ids sir-xxxxxxxx
```

---

# Capacity Reservations

---

# List Capacity Reservations

```bash
aws ec2 describe-capacity-reservations
```

---

# Create Capacity Reservation

```bash
aws ec2 create-capacity-reservation \
--instance-type t3.micro \
--instance-platform Linux/UNIX \
--availability-zone ap-south-1a \
--instance-count 2
```

---

# Cancel Capacity Reservation

```bash
aws ec2 cancel-capacity-reservation \
--capacity-reservation-id cr-xxxxxxxx
```

---

# Instance Metadata

---

# Get Instance Metadata (Inside EC2)

```bash
curl http://169.254.169.254/latest/meta-data/
```

---

# Get Instance ID

```bash
curl http://169.254.169.254/latest/meta-data/instance-id
```

---

# Get Availability Zone

```bash
curl http://169.254.169.254/latest/meta-data/placement/availability-zone
```

---

# Get IAM Role

```bash
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

---

# Get Public IPv4

```bash
curl http://169.254.169.254/latest/meta-data/public-ipv4
```

---

# Get Private IPv4

```bash
curl http://169.254.169.254/latest/meta-data/local-ipv4
```

---

# Best Practices

- Never allow SSH (22) from `0.0.0.0/0` in production unless absolutely necessary.
- Use IAM Roles instead of long-lived access keys.
- Prefer Launch Templates over Launch Configurations.
- Tag every AWS resource consistently.
- Use Auto Scaling for production workloads.
- Prefer Spot Instances for fault-tolerant workloads.
- Use Placement Groups only when required for specific performance needs.
- Use IMDSv2 for enhanced instance metadata security.

---

# Summary

This section covered advanced EC2 administration using the AWS CLI, including Security Groups, Key Pairs, Elastic IPs, Launch Templates, Auto Scaling Groups, Scaling Policies, Resource Tagging, Placement Groups, Spot Instances, Capacity Reservations, and EC2 Instance Metadata. These commands are commonly used for infrastructure automation, production operations, and DevOps workflows.

---

