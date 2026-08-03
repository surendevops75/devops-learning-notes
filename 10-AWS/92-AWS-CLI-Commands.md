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

# Amazon VPC

---

# List VPCs

```bash
aws ec2 describe-vpcs
```

---

# Describe Specific VPC

```bash
aws ec2 describe-vpcs \
--vpc-ids vpc-0123456789abcdef0
```

---

# Create VPC

```bash
aws ec2 create-vpc \
--cidr-block 10.0.0.0/16
```

---

# Delete VPC

```bash
aws ec2 delete-vpc \
--vpc-id vpc-xxxxxxxx
```

---

# Modify DNS Support

```bash
aws ec2 modify-vpc-attribute \
--vpc-id vpc-xxxxxxxx \
--enable-dns-support
```

---

# Enable DNS Hostnames

```bash
aws ec2 modify-vpc-attribute \
--vpc-id vpc-xxxxxxxx \
--enable-dns-hostnames
```

---

# Subnets

---

# List Subnets

```bash
aws ec2 describe-subnets
```

---

# Describe Specific Subnet

```bash
aws ec2 describe-subnets \
--subnet-ids subnet-xxxxxxxx
```

---

# Create Subnet

```bash
aws ec2 create-subnet \
--vpc-id vpc-xxxxxxxx \
--cidr-block 10.0.1.0/24 \
--availability-zone ap-south-1a
```

---

# Delete Subnet

```bash
aws ec2 delete-subnet \
--subnet-id subnet-xxxxxxxx
```

---

# Enable Auto Assign Public IP

```bash
aws ec2 modify-subnet-attribute \
--subnet-id subnet-xxxxxxxx \
--map-public-ip-on-launch
```

---

# Route Tables

---

# List Route Tables

```bash
aws ec2 describe-route-tables
```

---

# Create Route Table

```bash
aws ec2 create-route-table \
--vpc-id vpc-xxxxxxxx
```

---

# Associate Route Table

```bash
aws ec2 associate-route-table \
--subnet-id subnet-xxxxxxxx \
--route-table-id rtb-xxxxxxxx
```

---

# Create Internet Route

```bash
aws ec2 create-route \
--route-table-id rtb-xxxxxxxx \
--destination-cidr-block 0.0.0.0/0 \
--gateway-id igw-xxxxxxxx
```

---

# Delete Route

```bash
aws ec2 delete-route \
--route-table-id rtb-xxxxxxxx \
--destination-cidr-block 0.0.0.0/0
```

---

# Delete Route Table

```bash
aws ec2 delete-route-table \
--route-table-id rtb-xxxxxxxx
```

---

# Internet Gateway

---

# List Internet Gateways

```bash
aws ec2 describe-internet-gateways
```

---

# Create Internet Gateway

```bash
aws ec2 create-internet-gateway
```

---

# Attach Internet Gateway

```bash
aws ec2 attach-internet-gateway \
--internet-gateway-id igw-xxxxxxxx \
--vpc-id vpc-xxxxxxxx
```

---

# Detach Internet Gateway

```bash
aws ec2 detach-internet-gateway \
--internet-gateway-id igw-xxxxxxxx \
--vpc-id vpc-xxxxxxxx
```

---

# Delete Internet Gateway

```bash
aws ec2 delete-internet-gateway \
--internet-gateway-id igw-xxxxxxxx
```

---

# NAT Gateway

---

# List NAT Gateways

```bash
aws ec2 describe-nat-gateways
```

---

# Create NAT Gateway

```bash
aws ec2 create-nat-gateway \
--subnet-id subnet-xxxxxxxx \
--allocation-id eipalloc-xxxxxxxx
```

---

# Delete NAT Gateway

```bash
aws ec2 delete-nat-gateway \
--nat-gateway-id nat-xxxxxxxx
```

---

# Network ACL

---

# List Network ACLs

```bash
aws ec2 describe-network-acls
```

---

# Create Network ACL

```bash
aws ec2 create-network-acl \
--vpc-id vpc-xxxxxxxx
```

---

# Create Inbound Rule

```bash
aws ec2 create-network-acl-entry \
--network-acl-id acl-xxxxxxxx \
--rule-number 100 \
--protocol tcp \
--port-range From=80,To=80 \
--cidr-block 0.0.0.0/0 \
--rule-action allow \
--ingress
```

---

# Delete NACL Rule

```bash
aws ec2 delete-network-acl-entry \
--network-acl-id acl-xxxxxxxx \
--rule-number 100 \
--ingress
```

---

# Delete Network ACL

```bash
aws ec2 delete-network-acl \
--network-acl-id acl-xxxxxxxx
```

---

# VPC Peering

---

# List Peering Connections

```bash
aws ec2 describe-vpc-peering-connections
```

---

# Create Peering Connection

```bash
aws ec2 create-vpc-peering-connection \
--vpc-id vpc-aaaa \
--peer-vpc-id vpc-bbbb
```

---

# Accept Peering Request

```bash
aws ec2 accept-vpc-peering-connection \
--vpc-peering-connection-id pcx-xxxxxxxx
```

---

# Delete Peering Connection

```bash
aws ec2 delete-vpc-peering-connection \
--vpc-peering-connection-id pcx-xxxxxxxx
```

---

# Transit Gateway

---

# List Transit Gateways

```bash
aws ec2 describe-transit-gateways
```

---

# Create Transit Gateway

```bash
aws ec2 create-transit-gateway
```

---

# Delete Transit Gateway

```bash
aws ec2 delete-transit-gateway \
--transit-gateway-id tgw-xxxxxxxx
```

---

# List Attachments

```bash
aws ec2 describe-transit-gateway-attachments
```

---

# Attach VPC

```bash
aws ec2 create-transit-gateway-vpc-attachment \
--transit-gateway-id tgw-xxxxxxxx \
--vpc-id vpc-xxxxxxxx \
--subnet-ids subnet-1 subnet-2
```

---

# Elastic Network Interface (ENI)

---

# List Network Interfaces

```bash
aws ec2 describe-network-interfaces
```

---

# Create ENI

```bash
aws ec2 create-network-interface \
--subnet-id subnet-xxxxxxxx
```

---

# Attach ENI

```bash
aws ec2 attach-network-interface \
--network-interface-id eni-xxxxxxxx \
--instance-id i-xxxxxxxx \
--device-index 1
```

---

# Detach ENI

```bash
aws ec2 detach-network-interface \
--attachment-id eni-attach-xxxxxxxx
```

---

# Delete ENI

```bash
aws ec2 delete-network-interface \
--network-interface-id eni-xxxxxxxx
```

---

# VPC Endpoints

---

# List VPC Endpoints

```bash
aws ec2 describe-vpc-endpoints
```

---

# Create Gateway Endpoint (S3)

```bash
aws ec2 create-vpc-endpoint \
--vpc-id vpc-xxxxxxxx \
--service-name com.amazonaws.ap-south-1.s3 \
--route-table-ids rtb-xxxxxxxx \
--vpc-endpoint-type Gateway
```

---

# Create Interface Endpoint

```bash
aws ec2 create-vpc-endpoint \
--vpc-id vpc-xxxxxxxx \
--service-name com.amazonaws.ap-south-1.ec2 \
--subnet-ids subnet-xxxxxxxx \
--security-group-ids sg-xxxxxxxx \
--vpc-endpoint-type Interface
```

---

# Delete VPC Endpoint

```bash
aws ec2 delete-vpc-endpoints \
--vpc-endpoint-ids vpce-xxxxxxxx
```

---

# VPN

---

# List VPN Connections

```bash
aws ec2 describe-vpn-connections
```

---

# List Customer Gateways

```bash
aws ec2 describe-customer-gateways
```

---

# List Virtual Private Gateways

```bash
aws ec2 describe-vpn-gateways
```

---

# Direct Connect

---

# List Direct Connect Connections

```bash
aws directconnect describe-connections
```

---

# List Virtual Interfaces

```bash
aws directconnect describe-virtual-interfaces
```

---

# DHCP Options

---

# List DHCP Option Sets

```bash
aws ec2 describe-dhcp-options
```

---

# Create DHCP Option Set

```bash
aws ec2 create-dhcp-options \
--dhcp-configurations Key=domain-name,Values=example.internal
```

---

# Associate DHCP Option Set

```bash
aws ec2 associate-dhcp-options \
--dhcp-options-id dopt-xxxxxxxx \
--vpc-id vpc-xxxxxxxx
```

---

# Resource Queries

---

# List VPC IDs

```bash
aws ec2 describe-vpcs \
--query "Vpcs[].VpcId"
```

---

# List Public Subnets

```bash
aws ec2 describe-subnets \
--filters Name=map-public-ip-on-launch,Values=true
```

---

# List Route Table IDs

```bash
aws ec2 describe-route-tables \
--query "RouteTables[].RouteTableId"
```

---

# List Internet Gateway IDs

```bash
aws ec2 describe-internet-gateways \
--query "InternetGateways[].InternetGatewayId"
```

---

# Best Practices

- Use private subnets for application and database tiers.
- Enable DNS support and DNS hostnames in production VPCs.
- Use NAT Gateways for outbound internet access from private subnets.
- Prefer Gateway Endpoints for Amazon S3 and DynamoDB to reduce NAT Gateway costs.
- Use Interface Endpoints (PrivateLink) for private access to AWS services.
- Keep Network ACLs simple; rely primarily on Security Groups.
- Use Transit Gateway instead of complex VPC peering for large environments.
- Tag all networking resources consistently.

---

# Summary

This section covered AWS CLI commands for Amazon VPC, Subnets, Route Tables, Internet Gateways, NAT Gateways, Network ACLs, VPC Peering, Transit Gateway, Elastic Network Interfaces (ENIs), VPC Endpoints, VPN, Direct Connect, DHCP Options, and common resource queries. These commands are essential for automating AWS networking in production environments.

---

# IAM (Identity and Access Management)

---

# Get Current Identity

```bash
aws sts get-caller-identity
```

---

# List IAM Users

```bash
aws iam list-users
```

---

# Get User Details

```bash
aws iam get-user \
--user-name devuser
```

---

# Create IAM User

```bash
aws iam create-user \
--user-name devuser
```

---

# Delete IAM User

```bash
aws iam delete-user \
--user-name devuser
```

---

# Update IAM User

```bash
aws iam update-user \
--user-name devuser \
--new-user-name developer
```

---

# List IAM Groups

```bash
aws iam list-groups
```

---

# Create IAM Group

```bash
aws iam create-group \
--group-name DevOps
```

---

# Delete IAM Group

```bash
aws iam delete-group \
--group-name DevOps
```

---

# Add User to Group

```bash
aws iam add-user-to-group \
--group-name DevOps \
--user-name devuser
```

---

# Remove User from Group

```bash
aws iam remove-user-from-group \
--group-name DevOps \
--user-name devuser
```

---

# List Users in Group

```bash
aws iam get-group \
--group-name DevOps
```

---

# IAM Roles

---

# List Roles

```bash
aws iam list-roles
```

---

# Get Role Details

```bash
aws iam get-role \
--role-name EC2Role
```

---

# Create IAM Role

```bash
aws iam create-role \
--role-name EC2Role \
--assume-role-policy-document file://trust-policy.json
```

---

# Delete IAM Role

```bash
aws iam delete-role \
--role-name EC2Role
```

---

# Attach Managed Policy to Role

```bash
aws iam attach-role-policy \
--role-name EC2Role \
--policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

---

# Detach Policy from Role

```bash
aws iam detach-role-policy \
--role-name EC2Role \
--policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

---

# List Attached Role Policies

```bash
aws iam list-attached-role-policies \
--role-name EC2Role
```

---

# Policies

---

# List Managed Policies

```bash
aws iam list-policies
```

---

# Get Policy

```bash
aws iam get-policy \
--policy-arn arn:aws:iam::123456789012:policy/MyPolicy
```

---

# Get Policy Version

```bash
aws iam get-policy-version \
--policy-arn arn:aws:iam::123456789012:policy/MyPolicy \
--version-id v1
```

---

# Create Policy

```bash
aws iam create-policy \
--policy-name S3ReadOnly \
--policy-document file://policy.json
```

---

# Delete Policy

```bash
aws iam delete-policy \
--policy-arn arn:aws:iam::123456789012:policy/S3ReadOnly
```

---

# Attach Policy to User

```bash
aws iam attach-user-policy \
--user-name devuser \
--policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

---

# Detach Policy from User

```bash
aws iam detach-user-policy \
--user-name devuser \
--policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

---

# Attach Policy to Group

```bash
aws iam attach-group-policy \
--group-name DevOps \
--policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

---

# Detach Policy from Group

```bash
aws iam detach-group-policy \
--group-name DevOps \
--policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

---

# Access Keys

---

# List Access Keys

```bash
aws iam list-access-keys \
--user-name devuser
```

---

# Create Access Key

```bash
aws iam create-access-key \
--user-name devuser
```

---

# Update Access Key Status

```bash
aws iam update-access-key \
--user-name devuser \
--access-key-id AKIAxxxxxxxx \
--status Inactive
```

---

# Delete Access Key

```bash
aws iam delete-access-key \
--user-name devuser \
--access-key-id AKIAxxxxxxxx
```

---

# Login Profile

---

# Create Console Login

```bash
aws iam create-login-profile \
--user-name devuser \
--password 'TempPassword@123'
```

---

# Delete Console Login

```bash
aws iam delete-login-profile \
--user-name devuser
```

---

# STS

---

# Get Caller Identity

```bash
aws sts get-caller-identity
```

---

# Assume Role

```bash
aws sts assume-role \
--role-arn arn:aws:iam::123456789012:role/AdminRole \
--role-session-name AdminSession
```

---

# Get Session Token

```bash
aws sts get-session-token
```

---

# Decode Authorization Message

```bash
aws sts decode-authorization-message \
--encoded-message <message>
```

---

# KMS

---

# List KMS Keys

```bash
aws kms list-keys
```

---

# Describe KMS Key

```bash
aws kms describe-key \
--key-id alias/aws/s3
```

---

# Create KMS Key

```bash
aws kms create-key
```

---

# List Key Aliases

```bash
aws kms list-aliases
```

---

# Create Alias

```bash
aws kms create-alias \
--alias-name alias/project-key \
--target-key-id <key-id>
```

---

# Delete Alias

```bash
aws kms delete-alias \
--alias-name alias/project-key
```

---

# Enable Key

```bash
aws kms enable-key \
--key-id <key-id>
```

---

# Disable Key

```bash
aws kms disable-key \
--key-id <key-id>
```

---

# Schedule Key Deletion

```bash
aws kms schedule-key-deletion \
--key-id <key-id> \
--pending-window-in-days 30
```

---

# Cancel Key Deletion

```bash
aws kms cancel-key-deletion \
--key-id <key-id>
```

---

# Secrets Manager

---

# List Secrets

```bash
aws secretsmanager list-secrets
```

---

# Create Secret

```bash
aws secretsmanager create-secret \
--name db-password \
--secret-string "MySecurePassword"
```

---

# Get Secret Value

```bash
aws secretsmanager get-secret-value \
--secret-id db-password
```

---

# Update Secret

```bash
aws secretsmanager update-secret \
--secret-id db-password \
--secret-string "NewPassword123"
```

---

# Rotate Secret

```bash
aws secretsmanager rotate-secret \
--secret-id db-password
```

---

# Delete Secret

```bash
aws secretsmanager delete-secret \
--secret-id db-password
```

---

# AWS Certificate Manager (ACM)

---

# List Certificates

```bash
aws acm list-certificates
```

---

# Describe Certificate

```bash
aws acm describe-certificate \
--certificate-arn arn:aws:acm:...
```

---

# Request Certificate

```bash
aws acm request-certificate \
--domain-name example.com \
--validation-method DNS
```

---

# Delete Certificate

```bash
aws acm delete-certificate \
--certificate-arn arn:aws:acm:...
```

---

# AWS Organizations

---

# Describe Organization

```bash
aws organizations describe-organization
```

---

# List Accounts

```bash
aws organizations list-accounts
```

---

# List Organizational Units

```bash
aws organizations list-organizational-units-for-parent \
--parent-id r-xxxx
```

---

# List Policies

```bash
aws organizations list-policies \
--filter SERVICE_CONTROL_POLICY
```

---

# Describe Policy

```bash
aws organizations describe-policy \
--policy-id p-xxxxxxxx
```

---

# IAM Identity Center (AWS SSO)

---

# List Instances

```bash
aws sso-admin list-instances
```

---

# List Permission Sets

```bash
aws sso-admin list-permission-sets \
--instance-arn <instance-arn>
```

---

# Describe Permission Set

```bash
aws sso-admin describe-permission-set \
--instance-arn <instance-arn> \
--permission-set-arn <permission-set-arn>
```

---

# Resource Queries

---

# List User Names

```bash
aws iam list-users \
--query "Users[].UserName"
```

---

# List Role Names

```bash
aws iam list-roles \
--query "Roles[].RoleName"
```

---

# List Group Names

```bash
aws iam list-groups \
--query "Groups[].GroupName"
```

---

# List Policy Names

```bash
aws iam list-policies \
--query "Policies[].PolicyName"
```

---

# Best Practices

- Use IAM Roles instead of IAM Users whenever possible.
- Enable MFA for privileged users.
- Rotate access keys regularly.
- Follow the Principle of Least Privilege.
- Store secrets in AWS Secrets Manager instead of configuration files.
- Use Customer Managed KMS keys for sensitive workloads.
- Use IAM Identity Center for centralized workforce access.
- Apply Service Control Policies (SCPs) to enforce organization-wide guardrails.

---

# Summary

This section covered AWS CLI commands for IAM Users, Groups, Roles, Policies, Access Keys, Login Profiles, STS, KMS, Secrets Manager, AWS Certificate Manager (ACM), AWS Organizations, IAM Identity Center, and common resource queries. These commands are essential for identity management, security, encryption, and governance in AWS environments.

---

# Amazon S3

---

# List All Buckets

```bash
aws s3 ls
```

---

# List Bucket Contents

```bash
aws s3 ls s3://my-bucket
```

---

# List Bucket Recursively

```bash
aws s3 ls s3://my-bucket --recursive
```

---

# Show Bucket Size (Summary)

```bash
aws s3 ls s3://my-bucket \
--recursive \
--human-readable \
--summarize
```

---

# Create Bucket

```bash
aws s3 mb s3://my-bucket
```

---

# Create Bucket in Specific Region

```bash
aws s3api create-bucket \
--bucket my-bucket \
--region ap-south-1 \
--create-bucket-configuration LocationConstraint=ap-south-1
```

---

# Delete Empty Bucket

```bash
aws s3 rb s3://my-bucket
```

---

# Delete Bucket with Objects

```bash
aws s3 rb s3://my-bucket --force
```

---

# Upload Single File

```bash
aws s3 cp test.txt s3://my-bucket/
```

---

# Upload with Different Name

```bash
aws s3 cp app.log s3://my-bucket/logs/application.log
```

---

# Upload Entire Directory

```bash
aws s3 cp ./images s3://my-bucket/images \
--recursive
```

---

# Download File

```bash
aws s3 cp s3://my-bucket/test.txt .
```

---

# Download Entire Folder

```bash
aws s3 cp s3://my-bucket/images ./images \
--recursive
```

---

# Copy Between Buckets

```bash
aws s3 cp \
s3://bucket-a/file.txt \
s3://bucket-b/file.txt
```

---

# Move Object

```bash
aws s3 mv \
s3://bucket-a/file.txt \
s3://bucket-a/archive/file.txt
```

---

# Rename Object

```bash
aws s3 mv \
s3://bucket/file.txt \
s3://bucket/newfile.txt
```

---

# Delete File

```bash
aws s3 rm s3://my-bucket/test.txt
```

---

# Delete Folder

```bash
aws s3 rm s3://my-bucket/logs \
--recursive
```

---

# Sync Local → S3

```bash
aws s3 sync \
./website \
s3://my-bucket
```

---

# Sync S3 → Local

```bash
aws s3 sync \
s3://my-bucket \
./backup
```

---

# Sync Between Buckets

```bash
aws s3 sync \
s3://bucket-a \
s3://bucket-b
```

---

# Sync with Delete

```bash
aws s3 sync \
./website \
s3://my-bucket \
--delete
```

---

# Include Specific Files

```bash
aws s3 sync \
./logs \
s3://my-bucket \
--exclude "*" \
--include "*.log"
```

---

# Exclude Files

```bash
aws s3 sync \
./website \
s3://my-bucket \
--exclude "*.tmp"
```

---

# Bucket Versioning

---

# Enable Versioning

```bash
aws s3api put-bucket-versioning \
--bucket my-bucket \
--versioning-configuration Status=Enabled
```

---

# Check Versioning

```bash
aws s3api get-bucket-versioning \
--bucket my-bucket
```

---

# List Object Versions

```bash
aws s3api list-object-versions \
--bucket my-bucket
```

---

# Delete Specific Version

```bash
aws s3api delete-object \
--bucket my-bucket \
--key test.txt \
--version-id <version-id>
```

---

# Bucket Encryption

---

# Enable SSE-S3

```bash
aws s3api put-bucket-encryption \
--bucket my-bucket \
--server-side-encryption-configuration \
'{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

---

# Enable SSE-KMS

```bash
aws s3api put-bucket-encryption \
--bucket my-bucket \
--server-side-encryption-configuration \
'{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms","KMSMasterKeyID":"alias/project-key"}}]}'
```

---

# Check Encryption

```bash
aws s3api get-bucket-encryption \
--bucket my-bucket
```

---

# Bucket Policy

---

# Get Bucket Policy

```bash
aws s3api get-bucket-policy \
--bucket my-bucket
```

---

# Apply Bucket Policy

```bash
aws s3api put-bucket-policy \
--bucket my-bucket \
--policy file://policy.json
```

---

# Delete Bucket Policy

```bash
aws s3api delete-bucket-policy \
--bucket my-bucket
```

---

# Public Access Block

---

# Enable Public Access Block

```bash
aws s3api put-public-access-block \
--bucket my-bucket \
--public-access-block-configuration \
BlockPublicAcls=true,\
IgnorePublicAcls=true,\
BlockPublicPolicy=true,\
RestrictPublicBuckets=true
```

---

# Check Public Access Block

```bash
aws s3api get-public-access-block \
--bucket my-bucket
```

---

# Lifecycle Rules

---

# Apply Lifecycle Policy

```bash
aws s3api put-bucket-lifecycle-configuration \
--bucket my-bucket \
--lifecycle-configuration file://lifecycle.json
```

---

# Get Lifecycle Policy

```bash
aws s3api get-bucket-lifecycle-configuration \
--bucket my-bucket
```

---

# Delete Lifecycle Policy

```bash
aws s3api delete-bucket-lifecycle \
--bucket my-bucket
```

---

# Replication

---

# Configure Replication

```bash
aws s3api put-bucket-replication \
--bucket source-bucket \
--replication-configuration file://replication.json
```

---

# View Replication

```bash
aws s3api get-bucket-replication \
--bucket source-bucket
```

---

# Glacier Restore

---

# Restore Glacier Object

```bash
aws s3api restore-object \
--bucket archive-bucket \
--key backup.zip \
--restore-request Days=7
```

---

# Check Restore Status

```bash
aws s3api head-object \
--bucket archive-bucket \
--key backup.zip
```

---

# Presigned URL

---

# Generate Presigned URL

```bash
aws s3 presign \
s3://my-bucket/file.txt
```

---

# Presigned URL (Valid 1 Hour)

```bash
aws s3 presign \
s3://my-bucket/file.txt \
--expires-in 3600
```

---

# Access Points

---

# List Access Points

```bash
aws s3control list-access-points \
--account-id 123456789012
```

---

# Multipart Upload

---

# Start Multipart Upload

```bash
aws s3api create-multipart-upload \
--bucket my-bucket \
--key large-file.iso
```

---

# List Multipart Uploads

```bash
aws s3api list-multipart-uploads \
--bucket my-bucket
```

---

# Abort Multipart Upload

```bash
aws s3api abort-multipart-upload \
--bucket my-bucket \
--key large-file.iso \
--upload-id <upload-id>
```

---

# Object Metadata

---

# View Object Metadata

```bash
aws s3api head-object \
--bucket my-bucket \
--key app.log
```

---

# Copy with New Metadata

```bash
aws s3 cp \
s3://my-bucket/app.log \
s3://my-bucket/app.log \
--metadata Department=DevOps \
--metadata-directive REPLACE
```

---

# Resource Queries

---

# List Bucket Names

```bash
aws s3api list-buckets \
--query "Buckets[].Name"
```

---

# List Objects Only

```bash
aws s3api list-objects-v2 \
--bucket my-bucket \
--query "Contents[].Key"
```

---

# List Objects Larger Than 100 MB

```bash
aws s3api list-objects-v2 \
--bucket my-bucket \
--query "Contents[?Size > \`104857600\`]"
```

---

# Best Practices

- Enable bucket versioning for production buckets.
- Block all public access unless explicitly required.
- Enable default server-side encryption.
- Use lifecycle rules to move infrequently accessed data to lower-cost storage classes.
- Prefer `aws s3 sync` over repeated `cp` operations for directories.
- Use multipart uploads for large files.
- Enable cross-region replication for critical data.
- Use presigned URLs for temporary object access instead of making buckets public.

---

# Summary

This section covered AWS CLI commands for Amazon S3, including bucket management, object operations, synchronization, versioning, encryption, bucket policies, public access controls, lifecycle rules, replication, Glacier restores, multipart uploads, access points, metadata, presigned URLs, and common resource queries. These commands are essential for storage automation, backups, migrations, and production data management.

---

# Amazon RDS

---

# List DB Instances

```bash
aws rds describe-db-instances
```

---

# Describe Specific DB Instance

```bash
aws rds describe-db-instances \
--db-instance-identifier prod-db
```

---

# Create RDS Instance

```bash
aws rds create-db-instance \
--db-instance-identifier prod-db \
--engine mysql \
--db-instance-class db.t3.micro \
--allocated-storage 20 \
--master-username admin \
--master-user-password Password@123
```

---

# Start DB Instance

```bash
aws rds start-db-instance \
--db-instance-identifier prod-db
```

---

# Stop DB Instance

```bash
aws rds stop-db-instance \
--db-instance-identifier prod-db
```

---

# Reboot DB Instance

```bash
aws rds reboot-db-instance \
--db-instance-identifier prod-db
```

---

# Modify DB Instance

```bash
aws rds modify-db-instance \
--db-instance-identifier prod-db \
--db-instance-class db.t3.small \
--apply-immediately
```

---

# Delete DB Instance

```bash
aws rds delete-db-instance \
--db-instance-identifier prod-db \
--skip-final-snapshot
```

---

# Create Snapshot

```bash
aws rds create-db-snapshot \
--db-instance-identifier prod-db \
--db-snapshot-identifier prod-db-snapshot
```

---

# List Snapshots

```bash
aws rds describe-db-snapshots
```

---

# Restore from Snapshot

```bash
aws rds restore-db-instance-from-db-snapshot \
--db-instance-identifier restored-db \
--db-snapshot-identifier prod-db-snapshot
```

---

# Delete Snapshot

```bash
aws rds delete-db-snapshot \
--db-snapshot-identifier prod-db-snapshot
```

---

# List Read Replicas

```bash
aws rds describe-db-instances \
--query "DBInstances[?ReadReplicaSourceDBInstanceIdentifier!=null]"
```

---

# Aurora

---

# List Aurora Clusters

```bash
aws rds describe-db-clusters
```

---

# Describe Aurora Cluster

```bash
aws rds describe-db-clusters \
--db-cluster-identifier aurora-cluster
```

---

# Create Aurora Cluster

```bash
aws rds create-db-cluster \
--db-cluster-identifier aurora-cluster \
--engine aurora-mysql \
--master-username admin \
--master-user-password Password@123
```

---

# Delete Aurora Cluster

```bash
aws rds delete-db-cluster \
--db-cluster-identifier aurora-cluster \
--skip-final-snapshot
```

---

# DynamoDB

---

# List Tables

```bash
aws dynamodb list-tables
```

---

# Describe Table

```bash
aws dynamodb describe-table \
--table-name Employees
```

---

# Create Table

```bash
aws dynamodb create-table \
--cli-input-json file://table.json
```

---

# Delete Table

```bash
aws dynamodb delete-table \
--table-name Employees
```

---

# Put Item

```bash
aws dynamodb put-item \
--table-name Employees \
--item file://employee.json
```

---

# Get Item

```bash
aws dynamodb get-item \
--table-name Employees \
--key file://key.json
```

---

# Scan Table

```bash
aws dynamodb scan \
--table-name Employees
```

---

# Query Table

```bash
aws dynamodb query \
--table-name Employees \
--key-condition-expression "EmployeeId = :id"
```

---

# Delete Item

```bash
aws dynamodb delete-item \
--table-name Employees \
--key file://key.json
```

---

# ElastiCache

---

# List Cache Clusters

```bash
aws elasticache describe-cache-clusters
```

---

# Create Redis Cluster

```bash
aws elasticache create-cache-cluster \
--cache-cluster-id redis-prod \
--engine redis \
--cache-node-type cache.t3.micro \
--num-cache-nodes 1
```

---

# Delete Cache Cluster

```bash
aws elasticache delete-cache-cluster \
--cache-cluster-id redis-prod
```

---

# EFS

---

# List File Systems

```bash
aws efs describe-file-systems
```

---

# Create File System

```bash
aws efs create-file-system
```

---

# Describe Mount Targets

```bash
aws efs describe-mount-targets
```

---

# Create Mount Target

```bash
aws efs create-mount-target \
--file-system-id fs-xxxxxxxx \
--subnet-id subnet-xxxxxxxx \
--security-groups sg-xxxxxxxx
```

---

# Delete Mount Target

```bash
aws efs delete-mount-target \
--mount-target-id fsmt-xxxxxxxx
```

---

# Delete File System

```bash
aws efs delete-file-system \
--file-system-id fs-xxxxxxxx
```

---

# Amazon FSx

---

# List File Systems

```bash
aws fsx describe-file-systems
```

---

# Create Windows FSx

```bash
aws fsx create-file-system \
--file-system-type WINDOWS \
--storage-capacity 32
```

---

# Delete File System

```bash
aws fsx delete-file-system \
--file-system-id fs-xxxxxxxx
```

---

# AWS Backup

---

# List Backup Vaults

```bash
aws backup list-backup-vaults
```

---

# Create Backup Vault

```bash
aws backup create-backup-vault \
--backup-vault-name ProductionVault
```

---

# List Recovery Points

```bash
aws backup list-recovery-points-by-backup-vault \
--backup-vault-name ProductionVault
```

---

# Start Backup Job

```bash
aws backup start-backup-job \
--backup-vault-name ProductionVault \
--resource-arn arn:aws:ec2:...
```

---

# Start Restore Job

```bash
aws backup start-restore-job \
--recovery-point-arn arn:aws:backup:...
```

---

# Delete Backup Vault

```bash
aws backup delete-backup-vault \
--backup-vault-name ProductionVault
```

---

# AWS DataSync

---

# List Agents

```bash
aws datasync list-agents
```

---

# List Locations

```bash
aws datasync list-locations
```

---

# List Tasks

```bash
aws datasync list-tasks
```

---

# Start Task

```bash
aws datasync start-task-execution \
--task-arn arn:aws:datasync:...
```

---

# Describe Task

```bash
aws datasync describe-task \
--task-arn arn:aws:datasync:...
```

---

# AWS Storage Gateway

---

# List Gateways

```bash
aws storagegateway list-gateways
```

---

# Describe Gateway

```bash
aws storagegateway describe-gateway-information \
--gateway-arn arn:aws:storagegateway:...
```

---

# List Volumes

```bash
aws storagegateway list-volumes
```

---

# Refresh Cache

```bash
aws storagegateway refresh-cache \
--file-share-arn arn:aws:storagegateway:...
```

---

# AWS Snow Family

---

# List Snow Jobs

```bash
aws snowball list-jobs
```

---

# Describe Snow Job

```bash
aws snowball describe-job \
--job-id JIDxxxxxxxx
```

---

# Create Snow Job

```bash
aws snowball create-job \
--job-type IMPORT \
--resources file://resources.json \
--address-id ADxxxxxxxx
```

---

# Cancel Snow Job

```bash
aws snowball cancel-job \
--job-id JIDxxxxxxxx
```

---

# Resource Queries

---

# List RDS Identifiers

```bash
aws rds describe-db-instances \
--query "DBInstances[].DBInstanceIdentifier"
```

---

# List DynamoDB Tables

```bash
aws dynamodb list-tables \
--query "TableNames"
```

---

# List EFS IDs

```bash
aws efs describe-file-systems \
--query "FileSystems[].FileSystemId"
```

---

# List Backup Vault Names

```bash
aws backup list-backup-vaults \
--query "BackupVaultList[].BackupVaultName"
```

---

# Best Practices

- Enable Multi-AZ for production RDS databases.
- Automate RDS snapshots and backups.
- Use DynamoDB Query instead of Scan whenever possible.
- Enable encryption for EFS, FSx, and RDS.
- Use AWS Backup to centralize backup policies.
- Monitor DataSync task executions for migration jobs.
- Validate Snow Family jobs before shipping devices.
- Tag all storage resources consistently for cost allocation.

---

# Summary

This section covered AWS CLI commands for Amazon RDS, Aurora, DynamoDB, ElastiCache, EFS, FSx, AWS Backup, DataSync, Storage Gateway, and Snow Family. These commands are commonly used for provisioning, managing, backing up, and migrating AWS database and storage resources.

---

# Amazon ECR

---

# List Repositories

```bash
aws ecr describe-repositories
```

---

# Create Repository

```bash
aws ecr create-repository \
--repository-name my-app
```

---

# Delete Repository

```bash
aws ecr delete-repository \
--repository-name my-app \
--force
```

---

# List Images

```bash
aws ecr list-images \
--repository-name my-app
```

---

# Describe Images

```bash
aws ecr describe-images \
--repository-name my-app
```

---

# Delete Image

```bash
aws ecr batch-delete-image \
--repository-name my-app \
--image-ids imageTag=latest
```

---

# Get Login Password

```bash
aws ecr get-login-password
```

---

# Docker Login to ECR

```bash
aws ecr get-login-password | docker login \
--username AWS \
--password-stdin \
123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

---

# Start Image Scan

```bash
aws ecr start-image-scan \
--repository-name my-app \
--image-id imageTag=latest
```

---

# Get Image Scan Results

```bash
aws ecr describe-image-scan-findings \
--repository-name my-app \
--image-id imageTag=latest
```

---

# Amazon ECS

---

# List Clusters

```bash
aws ecs list-clusters
```

---

# Describe Cluster

```bash
aws ecs describe-clusters \
--clusters production
```

---

# Create Cluster

```bash
aws ecs create-cluster \
--cluster-name production
```

---

# Delete Cluster

```bash
aws ecs delete-cluster \
--cluster production
```

---

# List Services

```bash
aws ecs list-services \
--cluster production
```

---

# Describe Service

```bash
aws ecs describe-services \
--cluster production \
--services web
```

---

# Update Service

```bash
aws ecs update-service \
--cluster production \
--service web \
--desired-count 3
```

---

# Force New Deployment

```bash
aws ecs update-service \
--cluster production \
--service web \
--force-new-deployment
```

---

# List Tasks

```bash
aws ecs list-tasks \
--cluster production
```

---

# Stop Task

```bash
aws ecs stop-task \
--cluster production \
--task <task-id>
```

---

# Register Task Definition

```bash
aws ecs register-task-definition \
--cli-input-json file://task-definition.json
```

---

# List Task Definitions

```bash
aws ecs list-task-definitions
```

---

# Amazon EKS

---

# List Clusters

```bash
aws eks list-clusters
```

---

# Describe Cluster

```bash
aws eks describe-cluster \
--name production
```

---

# Create Cluster

```bash
aws eks create-cluster \
--cli-input-json file://cluster.json
```

---

# Delete Cluster

```bash
aws eks delete-cluster \
--name production
```

---

# Update kubeconfig

```bash
aws eks update-kubeconfig \
--region ap-south-1 \
--name production
```

---

# List Node Groups

```bash
aws eks list-nodegroups \
--cluster-name production
```

---

# Describe Node Group

```bash
aws eks describe-nodegroup \
--cluster-name production \
--nodegroup-name workers
```

---

# Create Node Group

```bash
aws eks create-nodegroup \
--cli-input-json file://nodegroup.json
```

---

# Delete Node Group

```bash
aws eks delete-nodegroup \
--cluster-name production \
--nodegroup-name workers
```

---

# AWS Lambda

---

# List Functions

```bash
aws lambda list-functions
```

---

# Get Function

```bash
aws lambda get-function \
--function-name processOrders
```

---

# Create Function

```bash
aws lambda create-function \
--cli-input-json file://lambda.json
```

---

# Update Function Code

```bash
aws lambda update-function-code \
--function-name processOrders \
--zip-file fileb://lambda.zip
```

---

# Invoke Function

```bash
aws lambda invoke \
--function-name processOrders \
output.json
```

---

# Delete Function

```bash
aws lambda delete-function \
--function-name processOrders
```

---

# List Event Source Mappings

```bash
aws lambda list-event-source-mappings
```

---

# API Gateway

---

# List REST APIs

```bash
aws apigateway get-rest-apis
```

---

# Get API Details

```bash
aws apigateway get-rest-api \
--rest-api-id abc123
```

---

# Create REST API

```bash
aws apigateway create-rest-api \
--name OrdersAPI
```

---

# Delete REST API

```bash
aws apigateway delete-rest-api \
--rest-api-id abc123
```

---

# List Stages

```bash
aws apigateway get-stages \
--rest-api-id abc123
```

---

# Amazon EventBridge

---

# List Event Buses

```bash
aws events list-event-buses
```

---

# List Rules

```bash
aws events list-rules
```

---

# Create Rule

```bash
aws events put-rule \
--name OrderEvents
```

---

# Delete Rule

```bash
aws events delete-rule \
--name OrderEvents
```

---

# Put Event

```bash
aws events put-events \
--entries file://events.json
```

---

# Amazon SNS

---

# List Topics

```bash
aws sns list-topics
```

---

# Create Topic

```bash
aws sns create-topic \
--name alerts
```

---

# Publish Message

```bash
aws sns publish \
--topic-arn arn:aws:sns:... \
--message "Deployment Successful"
```

---

# Subscribe Endpoint

```bash
aws sns subscribe \
--topic-arn arn:aws:sns:... \
--protocol email \
--notification-endpoint admin@example.com
```

---

# Delete Topic

```bash
aws sns delete-topic \
--topic-arn arn:aws:sns:...
```

---

# Amazon SQS

---

# List Queues

```bash
aws sqs list-queues
```

---

# Create Queue

```bash
aws sqs create-queue \
--queue-name orders
```

---

# Get Queue URL

```bash
aws sqs get-queue-url \
--queue-name orders
```

---

# Send Message

```bash
aws sqs send-message \
--queue-url https://sqs.ap-south-1.amazonaws.com/... \
--message-body "Order Created"
```

---

# Receive Message

```bash
aws sqs receive-message \
--queue-url https://sqs.ap-south-1.amazonaws.com/...
```

---

# Delete Message

```bash
aws sqs delete-message \
--queue-url https://sqs.ap-south-1.amazonaws.com/... \
--receipt-handle <receipt-handle>
```

---

# Purge Queue

```bash
aws sqs purge-queue \
--queue-url https://sqs.ap-south-1.amazonaws.com/...
```

---

# AWS Step Functions

---

# List State Machines

```bash
aws stepfunctions list-state-machines
```

---

# Describe State Machine

```bash
aws stepfunctions describe-state-machine \
--state-machine-arn arn:aws:states:...
```

---

# Create State Machine

```bash
aws stepfunctions create-state-machine \
--cli-input-json file://state-machine.json
```

---

# Start Execution

```bash
aws stepfunctions start-execution \
--state-machine-arn arn:aws:states:... \
--input file://input.json
```

---

# List Executions

```bash
aws stepfunctions list-executions \
--state-machine-arn arn:aws:states:...
```

---

# CloudWatch

---

# List Metrics

```bash
aws cloudwatch list-metrics
```

---

# Get Metric Statistics

```bash
aws cloudwatch get-metric-statistics \
--namespace AWS/EC2 \
--metric-name CPUUtilization
```

---

# Put Metric Alarm

```bash
aws cloudwatch put-metric-alarm \
--alarm-name HighCPU
```

---

# Describe Alarms

```bash
aws cloudwatch describe-alarms
```

---

# Delete Alarm

```bash
aws cloudwatch delete-alarms \
--alarm-names HighCPU
```

---

# CloudWatch Logs

---

# List Log Groups

```bash
aws logs describe-log-groups
```

---

# List Log Streams

```bash
aws logs describe-log-streams \
--log-group-name /aws/lambda/processOrders
```

---

# Get Log Events

```bash
aws logs get-log-events \
--log-group-name /aws/lambda/processOrders \
--log-stream-name stream-name
```

---

# Tail Logs

```bash
aws logs tail \
/aws/lambda/processOrders \
--follow
```

---

# Create Log Group

```bash
aws logs create-log-group \
--log-group-name application-logs
```

---

# Delete Log Group

```bash
aws logs delete-log-group \
--log-group-name application-logs
```

---

# AWS App Runner

---

# List Services

```bash
aws apprunner list-services
```

---

# Describe Service

```bash
aws apprunner describe-service \
--service-arn arn:aws:apprunner:...
```

---

# Create Service

```bash
aws apprunner create-service \
--cli-input-json file://service.json
```

---

# Delete Service

```bash
aws apprunner delete-service \
--service-arn arn:aws:apprunner:...
```

---

# Resource Queries

---

# List EKS Cluster Names

```bash
aws eks list-clusters \
--query "clusters"
```

---

# List Lambda Function Names

```bash
aws lambda list-functions \
--query "Functions[].FunctionName"
```

---

# List ECS Cluster ARNs

```bash
aws ecs list-clusters \
--query "clusterArns"
```

---

# List SNS Topic ARNs

```bash
aws sns list-topics \
--query "Topics[].TopicArn"
```

---

# Best Practices

- Use Amazon ECR image scanning before deployments.
- Update kubeconfig using `aws eks update-kubeconfig` instead of editing kubeconfig manually.
- Use CloudWatch alarms with SNS notifications for production monitoring.
- Configure Dead Letter Queues (DLQs) for Lambda and SQS where appropriate.
- Use EventBridge for event-driven integrations instead of polling.
- Force ECS deployments only when required to minimize unnecessary restarts.
- Store Lambda deployment packages in version-controlled artifacts.

---

# Summary

This section covered AWS CLI commands for Amazon ECR, ECS, EKS, Lambda, API Gateway, EventBridge, SNS, SQS, Step Functions, CloudWatch, CloudWatch Logs, and App Runner. These commands are frequently used by DevOps engineers for container orchestration, serverless deployments, messaging, event processing, monitoring, and production operations.

---

# AWS CloudFormation

---

# List Stacks

```bash
aws cloudformation list-stacks
```

---

# Describe Stacks

```bash
aws cloudformation describe-stacks
```

---

# Describe Specific Stack

```bash
aws cloudformation describe-stacks \
--stack-name production-stack
```

---

# Create Stack

```bash
aws cloudformation create-stack \
--stack-name production-stack \
--template-body file://template.yaml
```

---

# Update Stack

```bash
aws cloudformation update-stack \
--stack-name production-stack \
--template-body file://template.yaml
```

---

# Delete Stack

```bash
aws cloudformation delete-stack \
--stack-name production-stack
```

---

# Validate Template

```bash
aws cloudformation validate-template \
--template-body file://template.yaml
```

---

# Estimate Template Cost

```bash
aws cloudformation estimate-template-cost \
--template-body file://template.yaml
```

---

# List Stack Resources

```bash
aws cloudformation list-stack-resources \
--stack-name production-stack
```

---

# Get Stack Events

```bash
aws cloudformation describe-stack-events \
--stack-name production-stack
```

---

# Detect Stack Drift

```bash
aws cloudformation detect-stack-drift \
--stack-name production-stack
```

---

# Describe Drift Detection Status

```bash
aws cloudformation describe-stack-drift-detection-status \
--stack-drift-detection-id <drift-id>
```

---

# AWS CodeCommit

---

# List Repositories

```bash
aws codecommit list-repositories
```

---

# Create Repository

```bash
aws codecommit create-repository \
--repository-name devops-repo
```

---

# Get Repository

```bash
aws codecommit get-repository \
--repository-name devops-repo
```

---

# Delete Repository

```bash
aws codecommit delete-repository \
--repository-name devops-repo
```

---

# AWS CodeBuild

---

# List Projects

```bash
aws codebuild list-projects
```

---

# Describe Project

```bash
aws codebuild batch-get-projects \
--names my-build
```

---

# Create Build Project

```bash
aws codebuild create-project \
--cli-input-json file://project.json
```

---

# Start Build

```bash
aws codebuild start-build \
--project-name my-build
```

---

# Stop Build

```bash
aws codebuild stop-build \
--id <build-id>
```

---

# List Build IDs

```bash
aws codebuild list-builds
```

---

# Batch Get Builds

```bash
aws codebuild batch-get-builds \
--ids <build-id>
```

---

# AWS CodeDeploy

---

# List Applications

```bash
aws deploy list-applications
```

---

# Create Application

```bash
aws deploy create-application \
--application-name web-app
```

---

# List Deployment Groups

```bash
aws deploy list-deployment-groups \
--application-name web-app
```

---

# Create Deployment

```bash
aws deploy create-deployment \
--application-name web-app \
--deployment-group-name production
```

---

# Get Deployment

```bash
aws deploy get-deployment \
--deployment-id d-XXXXXXXX
```

---

# Stop Deployment

```bash
aws deploy stop-deployment \
--deployment-id d-XXXXXXXX
```

---

# AWS CodePipeline

---

# List Pipelines

```bash
aws codepipeline list-pipelines
```

---

# Get Pipeline

```bash
aws codepipeline get-pipeline \
--name production-pipeline
```

---

# Create Pipeline

```bash
aws codepipeline create-pipeline \
--cli-input-json file://pipeline.json
```

---

# Start Pipeline

```bash
aws codepipeline start-pipeline-execution \
--name production-pipeline
```

---

# Get Pipeline State

```bash
aws codepipeline get-pipeline-state \
--name production-pipeline
```

---

# Delete Pipeline

```bash
aws codepipeline delete-pipeline \
--name production-pipeline
```

---

# AWS CodeArtifact

---

# List Domains

```bash
aws codeartifact list-domains
```

---

# List Repositories

```bash
aws codeartifact list-repositories
```

---

# Create Repository

```bash
aws codeartifact create-repository \
--domain my-domain \
--repository devops-packages
```

---

# Get Authorization Token

```bash
aws codeartifact get-authorization-token \
--domain my-domain
```

---

# Delete Repository

```bash
aws codeartifact delete-repository \
--domain my-domain \
--repository devops-packages
```

---

# AWS CloudTrail

---

# List Trails

```bash
aws cloudtrail describe-trails
```

---

# Get Trail Status

```bash
aws cloudtrail get-trail-status \
--name organization-trail
```

---

# Create Trail

```bash
aws cloudtrail create-trail \
--name organization-trail \
--s3-bucket-name audit-logs
```

---

# Start Logging

```bash
aws cloudtrail start-logging \
--name organization-trail
```

---

# Stop Logging

```bash
aws cloudtrail stop-logging \
--name organization-trail
```

---

# Lookup Events

```bash
aws cloudtrail lookup-events
```

---

# AWS Config

---

# List Configuration Recorders

```bash
aws configservice describe-configuration-recorders
```

---

# List Config Rules

```bash
aws configservice describe-config-rules
```

---

# Get Compliance

```bash
aws configservice get-compliance-summary-by-config-rule
```

---

# AWS Systems Manager (SSM)

---

# Describe Instances

```bash
aws ssm describe-instance-information
```

---

# Start Session

```bash
aws ssm start-session \
--target i-0123456789abcdef0
```

---

# Send Command

```bash
aws ssm send-command \
--document-name AWS-RunShellScript \
--targets Key=instanceids,Values=i-0123456789abcdef0 \
--parameters commands="uptime"
```

---

# List Commands

```bash
aws ssm list-commands
```

---

# Get Command Invocation

```bash
aws ssm get-command-invocation \
--command-id <command-id> \
--instance-id i-0123456789abcdef0
```

---

# AWS Parameter Store

---

# List Parameters

```bash
aws ssm describe-parameters
```

---

# Create Parameter

```bash
aws ssm put-parameter \
--name /prod/db/password \
--value Password123 \
--type SecureString
```

---

# Get Parameter

```bash
aws ssm get-parameter \
--name /prod/db/password \
--with-decryption
```

---

# Get Multiple Parameters

```bash
aws ssm get-parameters \
--names /prod/db/password /prod/api/url
```

---

# Delete Parameter

```bash
aws ssm delete-parameter \
--name /prod/db/password
```

---

# OpsCenter

---

# List OpsItems

```bash
aws ssm list-ops-items
```

---

# Get OpsItem

```bash
aws ssm get-ops-item \
--ops-item-id oi-xxxxxxxx
```

---

# Create OpsItem

```bash
aws ssm create-ops-item \
--title "Production Alert"
```

---

# Resource Explorer

---

# Search Resources

```bash
aws resource-explorer-2 search \
--query-string "region:ap-south-1"
```

---

# Resource Groups

---

# List Resource Groups

```bash
aws resource-groups list-groups
```

---

# Create Resource Group

```bash
aws resource-groups create-group \
--name Production
```

---

# Tagging API

---

# Get Tagged Resources

```bash
aws resourcegroupstaggingapi get-resources
```

---

# Resource Queries

---

# List CloudFormation Stacks

```bash
aws cloudformation list-stacks \
--query "StackSummaries[].StackName"
```

---

# List CodeBuild Projects

```bash
aws codebuild list-projects \
--query "projects"
```

---

# List CodePipeline Names

```bash
aws codepipeline list-pipelines \
--query "pipelines[].name"
```

---

# List SSM Managed Instances

```bash
aws ssm describe-instance-information \
--query "InstanceInformationList[].InstanceId"
```

---

# Best Practices

- Validate CloudFormation templates before deployment.
- Enable CloudTrail organization-wide for auditing.
- Use Systems Manager Session Manager instead of SSH where possible.
- Store configuration values in Parameter Store or Secrets Manager.
- Use CodePipeline for repeatable CI/CD workflows.
- Monitor build failures in CodeBuild.
- Enable AWS Config rules for compliance monitoring.
- Use tagging consistently for governance and cost allocation.

---

# Summary

This section covered AWS CLI commands for CloudFormation, CodeCommit, CodeBuild, CodeDeploy, CodePipeline, CodeArtifact, CloudTrail, AWS Config, Systems Manager (SSM), Parameter Store, OpsCenter, Resource Groups, and Tagging APIs. These services are fundamental for Infrastructure as Code, CI/CD automation, operational management, auditing, and governance in AWS.

---

