# Boto3 Cookbook

---

# Introduction

Boto3 is the official AWS SDK for Python. It allows developers and DevOps engineers to automate AWS services programmatically.

Common Use Cases

- Infrastructure Automation
- Cloud Resource Management
- Serverless Applications
- CI/CD Pipelines
- Scheduled Automation
- Cloud Operations

---

# Install Boto3

```bash
pip install boto3
```

Verify Installation

```bash
python -c "import boto3; print(boto3.__version__)"
```

---

# Import Boto3

```python
import boto3
```

---

# Create Default Session

```python
import boto3

session = boto3.Session()
```

---

# Create Session Using Profile

```python
import boto3

session = boto3.Session(
    profile_name="dev"
)
```

---

# Create Session for Specific Region

```python
import boto3

session = boto3.Session(
    region_name="ap-south-1"
)
```

---

# Create EC2 Client

```python
import boto3

ec2 = boto3.client("ec2")
```

---

# Create EC2 Resource

```python
import boto3

ec2 = boto3.resource("ec2")
```

---

# Difference Between Client and Resource

| Client | Resource |
|---------|----------|
| Low-level API | High-level API |
| Faster | More Pythonic |
| Complete AWS API coverage | Simplified object interface |
| Returns dictionaries | Returns resource objects |

---

# List AWS Regions

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_regions()

for region in response["Regions"]:
    print(region["RegionName"])
```

---

# List Availability Zones

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_availability_zones()

for az in response["AvailabilityZones"]:
    print(az["ZoneName"])
```

---

# EC2 Examples

---

# List EC2 Instances

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_instances()

for reservation in response["Reservations"]:
    for instance in reservation["Instances"]:
        print(instance["InstanceId"])
```

---

# List Running Instances

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "instance-state-name",
            "Values": ["running"]
        }
    ]
)

for reservation in response["Reservations"]:
    for instance in reservation["Instances"]:
        print(instance["InstanceId"])
```

---

# List Stopped Instances

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "instance-state-name",
            "Values": ["stopped"]
        }
    ]
)
```

---

# Launch EC2 Instance

```python
response = ec2.run_instances(
    ImageId="ami-xxxxxxxx",
    InstanceType="t3.micro",
    MinCount=1,
    MaxCount=1,
    KeyName="my-key",
    SecurityGroupIds=["sg-xxxxxxxx"],
    SubnetId="subnet-xxxxxxxx"
)

print(response["Instances"][0]["InstanceId"])
```

---

# Stop EC2 Instance

```python
ec2.stop_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Start EC2 Instance

```python
ec2.start_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Reboot EC2 Instance

```python
ec2.reboot_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Terminate EC2 Instance

```python
ec2.terminate_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Describe Specific Instance

```python
response = ec2.describe_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)

print(response)
```

---

# Create AMI

```python
response = ec2.create_image(
    InstanceId="i-0123456789abcdef0",
    Name="Production-AMI"
)

print(response["ImageId"])
```

---

# List AMIs

```python
response = ec2.describe_images(
    Owners=["self"]
)

for image in response["Images"]:
    print(image["ImageId"])
```

---

# Deregister AMI

```python
ec2.deregister_image(
    ImageId="ami-xxxxxxxx"
)
```

---

# Create Snapshot

```python
response = ec2.create_snapshot(
    VolumeId="vol-xxxxxxxx",
    Description="Production Backup"
)

print(response["SnapshotId"])
```

---

# List Snapshots

```python
response = ec2.describe_snapshots(
    OwnerIds=["self"]
)

for snapshot in response["Snapshots"]:
    print(snapshot["SnapshotId"])
```

---

# Delete Snapshot

```python
ec2.delete_snapshot(
    SnapshotId="snap-xxxxxxxx"
)
```

---

# List Volumes

```python
response = ec2.describe_volumes()

for volume in response["Volumes"]:
    print(volume["VolumeId"])
```

---

# Create Volume

```python
response = ec2.create_volume(
    AvailabilityZone="ap-south-1a",
    Size=20,
    VolumeType="gp3"
)

print(response["VolumeId"])
```

---

# Attach Volume

```python
ec2.attach_volume(
    VolumeId="vol-xxxxxxxx",
    InstanceId="i-xxxxxxxx",
    Device="/dev/xvdf"
)
```

---

# Detach Volume

```python
ec2.detach_volume(
    VolumeId="vol-xxxxxxxx"
)
```

---

# Delete Volume

```python
ec2.delete_volume(
    VolumeId="vol-xxxxxxxx"
)
```

---

# Wait Until Instance is Running

```python
waiter = ec2.get_waiter("instance_running")

waiter.wait(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Wait Until Instance is Stopped

```python
waiter = ec2.get_waiter("instance_stopped")

waiter.wait(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Wait Until Snapshot Completes

```python
waiter = ec2.get_waiter("snapshot_completed")

waiter.wait(
    SnapshotIds=[
        "snap-xxxxxxxx"
    ]
)
```

---

# Error Handling

```python
import boto3
from botocore.exceptions import ClientError

ec2 = boto3.client("ec2")

try:
    response = ec2.describe_instances()
    print(response)

except ClientError as error:
    print(error)
```

---

# Best Practices

- Reuse Boto3 clients instead of creating them repeatedly.
- Use IAM Roles instead of embedding AWS credentials.
- Use waiters for long-running operations.
- Handle `ClientError` exceptions gracefully.
- Use pagination for large result sets.
- Prefer environment variables or named profiles for authentication.
- Avoid hardcoding resource IDs in production code.
- Log API responses for troubleshooting when appropriate.

---

# Summary

This section introduced Boto3 fundamentals, sessions, clients vs resources, EC2 instance management, AMIs, EBS volumes, snapshots, waiters, and exception handling. These examples provide the foundation for automating AWS infrastructure using Python.

---

# Amazon VPC

---

# Import Boto3

```python
import boto3

ec2 = boto3.client("ec2")
```

---

# List VPCs

```python
response = ec2.describe_vpcs()

for vpc in response["Vpcs"]:
    print(vpc["VpcId"])
```

---

# Describe Specific VPC

```python
response = ec2.describe_vpcs(
    VpcIds=[
        "vpc-0123456789abcdef0"
    ]
)

print(response["Vpcs"][0])
```

---

# Create VPC

```python
response = ec2.create_vpc(
    CidrBlock="10.0.0.0/16"
)

print(response["Vpc"]["VpcId"])
```

---

# Delete VPC

```python
ec2.delete_vpc(
    VpcId="vpc-xxxxxxxx"
)
```

---

# Enable DNS Support

```python
ec2.modify_vpc_attribute(
    VpcId="vpc-xxxxxxxx",
    EnableDnsSupport={
        "Value": True
    }
)
```

---

# Enable DNS Hostnames

```python
ec2.modify_vpc_attribute(
    VpcId="vpc-xxxxxxxx",
    EnableDnsHostnames={
        "Value": True
    }
)
```

---

# List Subnets

```python
response = ec2.describe_subnets()

for subnet in response["Subnets"]:
    print(subnet["SubnetId"])
```

---

# Create Subnet

```python
response = ec2.create_subnet(
    VpcId="vpc-xxxxxxxx",
    CidrBlock="10.0.1.0/24",
    AvailabilityZone="ap-south-1a"
)

print(response["Subnet"]["SubnetId"])
```

---

# Delete Subnet

```python
ec2.delete_subnet(
    SubnetId="subnet-xxxxxxxx"
)
```

---

# Enable Auto Assign Public IP

```python
ec2.modify_subnet_attribute(
    SubnetId="subnet-xxxxxxxx",
    MapPublicIpOnLaunch={
        "Value": True
    }
)
```

---

# Route Tables

---

# List Route Tables

```python
response = ec2.describe_route_tables()

for rt in response["RouteTables"]:
    print(rt["RouteTableId"])
```

---

# Create Route Table

```python
response = ec2.create_route_table(
    VpcId="vpc-xxxxxxxx"
)

print(response["RouteTable"]["RouteTableId"])
```

---

# Associate Route Table

```python
ec2.associate_route_table(
    RouteTableId="rtb-xxxxxxxx",
    SubnetId="subnet-xxxxxxxx"
)
```

---

# Create Internet Route

```python
ec2.create_route(
    RouteTableId="rtb-xxxxxxxx",
    DestinationCidrBlock="0.0.0.0/0",
    GatewayId="igw-xxxxxxxx"
)
```

---

# Delete Route

```python
ec2.delete_route(
    RouteTableId="rtb-xxxxxxxx",
    DestinationCidrBlock="0.0.0.0/0"
)
```

---

# Delete Route Table

```python
ec2.delete_route_table(
    RouteTableId="rtb-xxxxxxxx"
)
```

---

# Internet Gateway

---

# List Internet Gateways

```python
response = ec2.describe_internet_gateways()

for igw in response["InternetGateways"]:
    print(igw["InternetGatewayId"])
```

---

# Create Internet Gateway

```python
response = ec2.create_internet_gateway()

print(response["InternetGateway"]["InternetGatewayId"])
```

---

# Attach Internet Gateway

```python
ec2.attach_internet_gateway(
    InternetGatewayId="igw-xxxxxxxx",
    VpcId="vpc-xxxxxxxx"
)
```

---

# Detach Internet Gateway

```python
ec2.detach_internet_gateway(
    InternetGatewayId="igw-xxxxxxxx",
    VpcId="vpc-xxxxxxxx"
)
```

---

# Delete Internet Gateway

```python
ec2.delete_internet_gateway(
    InternetGatewayId="igw-xxxxxxxx"
)
```

---

# NAT Gateway

---

# List NAT Gateways

```python
response = ec2.describe_nat_gateways()

for nat in response["NatGateways"]:
    print(nat["NatGatewayId"])
```

---

# Create NAT Gateway

```python
response = ec2.create_nat_gateway(
    AllocationId="eipalloc-xxxxxxxx",
    SubnetId="subnet-xxxxxxxx"
)

print(response["NatGateway"]["NatGatewayId"])
```

---

# Delete NAT Gateway

```python
ec2.delete_nat_gateway(
    NatGatewayId="nat-xxxxxxxx"
)
```

---

# Security Groups

---

# List Security Groups

```python
response = ec2.describe_security_groups()

for sg in response["SecurityGroups"]:
    print(sg["GroupName"])
```

---

# Create Security Group

```python
response = ec2.create_security_group(
    GroupName="web-sg",
    Description="Web Security Group",
    VpcId="vpc-xxxxxxxx"
)

print(response["GroupId"])
```

---

# Add SSH Rule

```python
ec2.authorize_security_group_ingress(
    GroupId="sg-xxxxxxxx",
    IpPermissions=[
        {
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "IpRanges": [
                {
                    "CidrIp": "0.0.0.0/0"
                }
            ]
        }
    ]
)
```

---

# Add HTTP Rule

```python
ec2.authorize_security_group_ingress(
    GroupId="sg-xxxxxxxx",
    IpPermissions=[
        {
            "IpProtocol": "tcp",
            "FromPort": 80,
            "ToPort": 80,
            "IpRanges": [
                {
                    "CidrIp": "0.0.0.0/0"
                }
            ]
        }
    ]
)
```

---

# Remove Security Group Rule

```python
ec2.revoke_security_group_ingress(
    GroupId="sg-xxxxxxxx",
    IpPermissions=[
        {
            "IpProtocol": "tcp",
            "FromPort": 22,
            "ToPort": 22,
            "IpRanges": [
                {
                    "CidrIp": "0.0.0.0/0"
                }
            ]
        }
    ]
)
```

---

# Delete Security Group

```python
ec2.delete_security_group(
    GroupId="sg-xxxxxxxx"
)
```

---

# Elastic IP

---

# Allocate Elastic IP

```python
response = ec2.allocate_address(
    Domain="vpc"
)

print(response["AllocationId"])
```

---

# Associate Elastic IP

```python
ec2.associate_address(
    InstanceId="i-xxxxxxxx",
    AllocationId="eipalloc-xxxxxxxx"
)
```

---

# Disassociate Elastic IP

```python
ec2.disassociate_address(
    AssociationId="eipassoc-xxxxxxxx"
)
```

---

# Release Elastic IP

```python
ec2.release_address(
    AllocationId="eipalloc-xxxxxxxx"
)
```

---

# Network ACLs

---

# List Network ACLs

```python
response = ec2.describe_network_acls()

for acl in response["NetworkAcls"]:
    print(acl["NetworkAclId"])
```

---

# Create Network ACL

```python
response = ec2.create_network_acl(
    VpcId="vpc-xxxxxxxx"
)

print(response["NetworkAcl"]["NetworkAclId"])
```

---

# Delete Network ACL

```python
ec2.delete_network_acl(
    NetworkAclId="acl-xxxxxxxx"
)
```

---

# VPC Peering

---

# List Peering Connections

```python
response = ec2.describe_vpc_peering_connections()

for peer in response["VpcPeeringConnections"]:
    print(peer["VpcPeeringConnectionId"])
```

---

# Create Peering Connection

```python
response = ec2.create_vpc_peering_connection(
    VpcId="vpc-aaaa",
    PeerVpcId="vpc-bbbb"
)

print(response["VpcPeeringConnection"]["VpcPeeringConnectionId"])
```

---

# Accept Peering Request

```python
ec2.accept_vpc_peering_connection(
    VpcPeeringConnectionId="pcx-xxxxxxxx"
)
```

---

# Delete Peering Connection

```python
ec2.delete_vpc_peering_connection(
    VpcPeeringConnectionId="pcx-xxxxxxxx"
)
```

---

# Transit Gateway

---

# List Transit Gateways

```python
response = ec2.describe_transit_gateways()

for tgw in response["TransitGateways"]:
    print(tgw["TransitGatewayId"])
```

---

# Create Transit Gateway

```python
response = ec2.create_transit_gateway()

print(response["TransitGateway"]["TransitGatewayId"])
```

---

# Delete Transit Gateway

```python
ec2.delete_transit_gateway(
    TransitGatewayId="tgw-xxxxxxxx"
)
```

---

# Elastic Network Interfaces (ENIs)

---

# List ENIs

```python
response = ec2.describe_network_interfaces()

for eni in response["NetworkInterfaces"]:
    print(eni["NetworkInterfaceId"])
```

---

# Create ENI

```python
response = ec2.create_network_interface(
    SubnetId="subnet-xxxxxxxx"
)

print(response["NetworkInterface"]["NetworkInterfaceId"])
```

---

# Attach ENI

```python
ec2.attach_network_interface(
    NetworkInterfaceId="eni-xxxxxxxx",
    InstanceId="i-xxxxxxxx",
    DeviceIndex=1
)
```

---

# Delete ENI

```python
ec2.delete_network_interface(
    NetworkInterfaceId="eni-xxxxxxxx"
)
```

---

# VPC Endpoints

---

# List VPC Endpoints

```python
response = ec2.describe_vpc_endpoints()

for endpoint in response["VpcEndpoints"]:
    print(endpoint["VpcEndpointId"])
```

---

# Create Gateway Endpoint

```python
response = ec2.create_vpc_endpoint(
    VpcId="vpc-xxxxxxxx",
    ServiceName="com.amazonaws.ap-south-1.s3",
    VpcEndpointType="Gateway",
    RouteTableIds=["rtb-xxxxxxxx"]
)

print(response["VpcEndpoint"]["VpcEndpointId"])
```

---

# Delete VPC Endpoint

```python
ec2.delete_vpc_endpoints(
    VpcEndpointIds=[
        "vpce-xxxxxxxx"
    ]
)
```

---

# Wait Until VPC Available

```python
waiter = ec2.get_waiter("vpc_available")

waiter.wait(
    VpcIds=[
        "vpc-xxxxxxxx"
    ]
)
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = ec2.describe_vpcs()

except ClientError as error:
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Design VPC CIDR ranges carefully to avoid overlap.
- Keep databases in private subnets.
- Use Security Groups as the primary firewall.
- Prefer VPC Endpoints over NAT Gateway where applicable.
- Use Transit Gateway for hub-and-spoke architectures.
- Tag all networking resources.
- Avoid using the default VPC in production.
- Use waiters when creating networking resources.

---

# Summary

This section covered Boto3 examples for Amazon VPC, Subnets, Route Tables, Internet Gateways, NAT Gateways, Security Groups, Elastic IPs, Network ACLs, VPC Peering, Transit Gateway, ENIs, VPC Endpoints, waiters, and exception handling. These examples provide practical automation patterns for building and managing AWS networking infrastructure using Python.

---

# IAM

---

# Import Boto3

```python
import boto3

iam = boto3.client("iam")
```

---

# List IAM Users

```python
response = iam.list_users()

for user in response["Users"]:
    print(user["UserName"])
```

---

# Get IAM User

```python
response = iam.get_user(
    UserName="devuser"
)

print(response["User"])
```

---

# Create IAM User

```python
response = iam.create_user(
    UserName="devuser"
)

print(response["User"]["Arn"])
```

---

# Delete IAM User

```python
iam.delete_user(
    UserName="devuser"
)
```

---

# Update IAM User

```python
iam.update_user(
    UserName="devuser",
    NewUserName="developer"
)
```

---

# List IAM Groups

```python
response = iam.list_groups()

for group in response["Groups"]:
    print(group["GroupName"])
```

---

# Create IAM Group

```python
iam.create_group(
    GroupName="DevOps"
)
```

---

# Delete IAM Group

```python
iam.delete_group(
    GroupName="DevOps"
)
```

---

# Add User to Group

```python
iam.add_user_to_group(
    GroupName="DevOps",
    UserName="devuser"
)
```

---

# Remove User from Group

```python
iam.remove_user_from_group(
    GroupName="DevOps",
    UserName="devuser"
)
```

---

# List Users in Group

```python
response = iam.get_group(
    GroupName="DevOps"
)

for user in response["Users"]:
    print(user["UserName"])
```

---

# IAM Roles

---

# List Roles

```python
response = iam.list_roles()

for role in response["Roles"]:
    print(role["RoleName"])
```

---

# Get Role

```python
response = iam.get_role(
    RoleName="EC2Role"
)

print(response["Role"])
```

---

# Create Role

```python
import json

trust_policy = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "ec2.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}

response = iam.create_role(
    RoleName="EC2Role",
    AssumeRolePolicyDocument=json.dumps(trust_policy)
)

print(response["Role"]["Arn"])
```

---

# Delete Role

```python
iam.delete_role(
    RoleName="EC2Role"
)
```

---

# Attach Managed Policy

```python
iam.attach_role_policy(
    RoleName="EC2Role",
    PolicyArn="arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
)
```

---

# Detach Managed Policy

```python
iam.detach_role_policy(
    RoleName="EC2Role",
    PolicyArn="arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
)
```

---

# List Attached Policies

```python
response = iam.list_attached_role_policies(
    RoleName="EC2Role"
)

for policy in response["AttachedPolicies"]:
    print(policy["PolicyName"])
```

---

# IAM Policies

---

# List Policies

```python
response = iam.list_policies(
    Scope="Local"
)

for policy in response["Policies"]:
    print(policy["PolicyName"])
```

---

# Create Policy

```python
policy_document = {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": "*"
        }
    ]
}

response = iam.create_policy(
    PolicyName="S3ReadOnly",
    PolicyDocument=json.dumps(policy_document)
)

print(response["Policy"]["Arn"])
```

---

# Get Policy

```python
response = iam.get_policy(
    PolicyArn="arn:aws:iam::123456789012:policy/S3ReadOnly"
)

print(response["Policy"])
```

---

# Delete Policy

```python
iam.delete_policy(
    PolicyArn="arn:aws:iam::123456789012:policy/S3ReadOnly"
)
```

---

# Access Keys

---

# List Access Keys

```python
response = iam.list_access_keys(
    UserName="devuser"
)

print(response["AccessKeyMetadata"])
```

---

# Create Access Key

```python
response = iam.create_access_key(
    UserName="devuser"
)

print(response["AccessKey"]["AccessKeyId"])
```

---

# Delete Access Key

```python
iam.delete_access_key(
    UserName="devuser",
    AccessKeyId="AKIAxxxxxxxx"
)
```

---

# Update Access Key

```python
iam.update_access_key(
    UserName="devuser",
    AccessKeyId="AKIAxxxxxxxx",
    Status="Inactive"
)
```

---

# Login Profile

---

# Create Console Password

```python
iam.create_login_profile(
    UserName="devuser",
    Password="Password@123",
    PasswordResetRequired=True
)
```

---

# Delete Login Profile

```python
iam.delete_login_profile(
    UserName="devuser"
)
```

---

# AWS STS

---

# Import STS Client

```python
sts = boto3.client("sts")
```

---

# Get Caller Identity

```python
response = sts.get_caller_identity()

print(response["Arn"])
```

---

# Assume Role

```python
response = sts.assume_role(
    RoleArn="arn:aws:iam::123456789012:role/AdminRole",
    RoleSessionName="AutomationSession"
)

credentials = response["Credentials"]
```

---

# Get Session Token

```python
response = sts.get_session_token()

print(response["Credentials"])
```

---

# AWS KMS

---

# Import KMS Client

```python
kms = boto3.client("kms")
```

---

# List Keys

```python
response = kms.list_keys()

for key in response["Keys"]:
    print(key["KeyId"])
```

---

# Create KMS Key

```python
response = kms.create_key(
    Description="Production Key"
)

print(response["KeyMetadata"]["KeyId"])
```

---

# Describe Key

```python
response = kms.describe_key(
    KeyId="alias/aws/s3"
)

print(response["KeyMetadata"])
```

---

# Create Alias

```python
kms.create_alias(
    AliasName="alias/project-key",
    TargetKeyId="<key-id>"
)
```

---

# Delete Alias

```python
kms.delete_alias(
    AliasName="alias/project-key"
)
```

---

# Enable Key

```python
kms.enable_key(
    KeyId="<key-id>"
)
```

---

# Disable Key

```python
kms.disable_key(
    KeyId="<key-id>"
)
```

---

# Encrypt Data

```python
response = kms.encrypt(
    KeyId="alias/project-key",
    Plaintext=b"My Secret"
)

ciphertext = response["CiphertextBlob"]
```

---

# Decrypt Data

```python
response = kms.decrypt(
    CiphertextBlob=ciphertext
)

print(response["Plaintext"])
```

---

# AWS Secrets Manager

---

# Import Client

```python
secrets = boto3.client("secretsmanager")
```

---

# List Secrets

```python
response = secrets.list_secrets()

for secret in response["SecretList"]:
    print(secret["Name"])
```

---

# Create Secret

```python
secrets.create_secret(
    Name="db-password",
    SecretString="Password@123"
)
```

---

# Get Secret

```python
response = secrets.get_secret_value(
    SecretId="db-password"
)

print(response["SecretString"])
```

---

# Update Secret

```python
secrets.update_secret(
    SecretId="db-password",
    SecretString="NewPassword123"
)
```

---

# Delete Secret

```python
secrets.delete_secret(
    SecretId="db-password"
)
```

---

# AWS Certificate Manager (ACM)

---

# Import Client

```python
acm = boto3.client("acm")
```

---

# List Certificates

```python
response = acm.list_certificates()

for cert in response["CertificateSummaryList"]:
    print(cert["DomainName"])
```

---

# Describe Certificate

```python
response = acm.describe_certificate(
    CertificateArn="arn:aws:acm:..."
)

print(response["Certificate"])
```

---

# Request Certificate

```python
response = acm.request_certificate(
    DomainName="example.com",
    ValidationMethod="DNS"
)

print(response["CertificateArn"])
```

---

# Delete Certificate

```python
acm.delete_certificate(
    CertificateArn="arn:aws:acm:..."
)
```

---

# AWS Organizations

---

# Import Client

```python
org = boto3.client("organizations")
```

---

# List Accounts

```python
response = org.list_accounts()

for account in response["Accounts"]:
    print(account["Name"])
```

---

# Describe Organization

```python
response = org.describe_organization()

print(response["Organization"])
```

---

# List Policies

```python
response = org.list_policies(
    Filter="SERVICE_CONTROL_POLICY"
)

for policy in response["Policies"]:
    print(policy["Name"])
```

---

# IAM Identity Center (AWS SSO)

---

# Import Client

```python
sso = boto3.client("sso-admin")
```

---

# List Instances

```python
response = sso.list_instances()

for instance in response["Instances"]:
    print(instance["InstanceArn"])
```

---

# List Permission Sets

```python
response = sso.list_permission_sets(
    InstanceArn="<instance-arn>"
)

print(response["PermissionSets"])
```

---

# Pagination Example

```python
paginator = iam.get_paginator("list_users")

for page in paginator.paginate():
    for user in page["Users"]:
        print(user["UserName"])
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = iam.list_users()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Use IAM Roles instead of IAM Users for AWS workloads.
- Never hardcode AWS credentials in Python code.
- Rotate access keys regularly.
- Store application secrets in AWS Secrets Manager.
- Use customer-managed KMS keys for sensitive workloads.
- Apply the principle of least privilege to IAM policies.
- Use paginators for APIs that return large result sets.
- Catch and handle `ClientError` exceptions for all AWS API calls.

---

# Summary

This section covered Boto3 automation for IAM Users, Groups, Roles, Policies, Access Keys, Login Profiles, STS, KMS, Secrets Manager, ACM, AWS Organizations, IAM Identity Center, pagination, and exception handling. These examples demonstrate secure identity and access management automation using Python.

---

# Amazon S3

---

# Import Boto3

```python
import boto3

s3 = boto3.client("s3")
```

---

# Create Resource Object

```python
import boto3

s3_resource = boto3.resource("s3")
```

---

# List Buckets

```python
response = s3.list_buckets()

for bucket in response["Buckets"]:
    print(bucket["Name"])
```

---

# Create Bucket

```python
response = s3.create_bucket(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# Create Bucket in Specific Region

```python
response = s3.create_bucket(
    Bucket="my-demo-bucket",
    CreateBucketConfiguration={
        "LocationConstraint": "ap-south-1"
    }
)
```

---

# Delete Bucket

```python
s3.delete_bucket(
    Bucket="my-demo-bucket"
)
```

---

# Check Bucket Exists

```python
try:
    s3.head_bucket(
        Bucket="my-demo-bucket"
    )
    print("Bucket exists")

except Exception:
    print("Bucket not found")
```

---

# Upload File

```python
s3.upload_file(
    "app.log",
    "my-demo-bucket",
    "logs/app.log"
)
```

---

# Upload Bytes

```python
s3.put_object(
    Bucket="my-demo-bucket",
    Key="hello.txt",
    Body=b"Hello AWS"
)
```

---

# Upload JSON

```python
import json

data = {
    "name": "Surendra",
    "role": "DevOps"
}

s3.put_object(
    Bucket="my-demo-bucket",
    Key="employee.json",
    Body=json.dumps(data)
)
```

---

# Download File

```python
s3.download_file(
    "my-demo-bucket",
    "logs/app.log",
    "./app.log"
)
```

---

# Download Object

```python
response = s3.get_object(
    Bucket="my-demo-bucket",
    Key="hello.txt"
)

print(
    response["Body"].read().decode()
)
```

---

# Copy Object

```python
s3.copy_object(
    Bucket="destination-bucket",
    CopySource={
        "Bucket": "source-bucket",
        "Key": "app.log"
    },
    Key="backup/app.log"
)
```

---

# Delete Object

```python
s3.delete_object(
    Bucket="my-demo-bucket",
    Key="hello.txt"
)
```

---

# Delete Multiple Objects

```python
s3.delete_objects(
    Bucket="my-demo-bucket",
    Delete={
        "Objects": [
            {
                "Key": "a.txt"
            },
            {
                "Key": "b.txt"
            }
        ]
    }
)
```

---

# List Objects

```python
response = s3.list_objects_v2(
    Bucket="my-demo-bucket"
)

for obj in response.get("Contents", []):
    print(obj["Key"])
```

---

# List Objects by Prefix

```python
response = s3.list_objects_v2(
    Bucket="my-demo-bucket",
    Prefix="logs/"
)

for obj in response.get("Contents", []):
    print(obj["Key"])
```

---

# Pagination

```python
paginator = s3.get_paginator(
    "list_objects_v2"
)

for page in paginator.paginate(
    Bucket="my-demo-bucket"
):
    for obj in page.get("Contents", []):
        print(obj["Key"])
```

---

# Upload Directory

```python
import os

for root, dirs, files in os.walk("./website"):

    for file in files:

        local_path = os.path.join(root, file)

        s3.upload_file(
            local_path,
            "my-demo-bucket",
            local_path
        )
```

---

# Bucket Versioning

---

# Enable Versioning

```python
s3.put_bucket_versioning(
    Bucket="my-demo-bucket",
    VersioningConfiguration={
        "Status": "Enabled"
    }
)
```

---

# Check Versioning

```python
response = s3.get_bucket_versioning(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# List Object Versions

```python
response = s3.list_object_versions(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# Bucket Encryption

---

# Enable SSE-S3

```python
s3.put_bucket_encryption(
    Bucket="my-demo-bucket",
    ServerSideEncryptionConfiguration={
        "Rules": [
            {
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "AES256"
                }
            }
        ]
    }
)
```

---

# Enable SSE-KMS

```python
s3.put_bucket_encryption(
    Bucket="my-demo-bucket",
    ServerSideEncryptionConfiguration={
        "Rules": [
            {
                "ApplyServerSideEncryptionByDefault": {
                    "SSEAlgorithm": "aws:kms",
                    "KMSMasterKeyID": "alias/project-key"
                }
            }
        ]
    }
)
```

---

# Get Encryption

```python
response = s3.get_bucket_encryption(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# Bucket Policy

---

# Apply Bucket Policy

```python
import json

policy = {
    "Version": "2012-10-17",
    "Statement": []
}

s3.put_bucket_policy(
    Bucket="my-demo-bucket",
    Policy=json.dumps(policy)
)
```

---

# Get Bucket Policy

```python
response = s3.get_bucket_policy(
    Bucket="my-demo-bucket"
)

print(response["Policy"])
```

---

# Delete Bucket Policy

```python
s3.delete_bucket_policy(
    Bucket="my-demo-bucket"
)
```

---

# Public Access Block

---

# Enable Public Access Block

```python
s3.put_public_access_block(
    Bucket="my-demo-bucket",
    PublicAccessBlockConfiguration={
        "BlockPublicAcls": True,
        "IgnorePublicAcls": True,
        "BlockPublicPolicy": True,
        "RestrictPublicBuckets": True
    }
)
```

---

# Get Public Access Block

```python
response = s3.get_public_access_block(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# Lifecycle Rules

---

# Apply Lifecycle Policy

```python
s3.put_bucket_lifecycle_configuration(
    Bucket="my-demo-bucket",
    LifecycleConfiguration={
        "Rules": [
            {
                "ID": "ArchiveLogs",
                "Status": "Enabled",
                "Filter": {
                    "Prefix": "logs/"
                },
                "Transitions": [
                    {
                        "Days": 30,
                        "StorageClass": "GLACIER"
                    }
                ]
            }
        ]
    }
)
```

---

# Get Lifecycle Configuration

```python
response = s3.get_bucket_lifecycle_configuration(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# Replication

---

# Get Replication

```python
response = s3.get_bucket_replication(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# Presigned URL

---

# Generate Download URL

```python
url = s3.generate_presigned_url(
    "get_object",
    Params={
        "Bucket": "my-demo-bucket",
        "Key": "logs/app.log"
    },
    ExpiresIn=3600
)

print(url)
```

---

# Generate Upload URL

```python
url = s3.generate_presigned_url(
    "put_object",
    Params={
        "Bucket": "my-demo-bucket",
        "Key": "upload.txt"
    },
    ExpiresIn=1800
)
```

---

# Multipart Upload

---

# Create Multipart Upload

```python
response = s3.create_multipart_upload(
    Bucket="my-demo-bucket",
    Key="large.iso"
)

upload_id = response["UploadId"]
```

---

# List Multipart Uploads

```python
response = s3.list_multipart_uploads(
    Bucket="my-demo-bucket"
)

print(response)
```

---

# Abort Multipart Upload

```python
s3.abort_multipart_upload(
    Bucket="my-demo-bucket",
    Key="large.iso",
    UploadId=upload_id
)
```

---

# Object Metadata

---

# Head Object

```python
response = s3.head_object(
    Bucket="my-demo-bucket",
    Key="logs/app.log"
)

print(response)
```

---

# Upload Metadata

```python
s3.copy_object(
    Bucket="my-demo-bucket",
    CopySource={
        "Bucket": "my-demo-bucket",
        "Key": "logs/app.log"
    },
    Key="logs/app.log",
    Metadata={
        "Department": "DevOps"
    },
    MetadataDirective="REPLACE"
)
```

---

# Resource API

---

# List Buckets (Resource)

```python
for bucket in s3_resource.buckets.all():
    print(bucket.name)
```

---

# List Objects (Resource)

```python
bucket = s3_resource.Bucket(
    "my-demo-bucket"
)

for obj in bucket.objects.all():
    print(obj.key)
```

---

# Delete All Objects

```python
bucket = s3_resource.Bucket(
    "my-demo-bucket"
)

bucket.objects.all().delete()
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    s3.list_buckets()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Enable Versioning on production buckets.
- Enable default encryption using SSE-KMS.
- Block all public access unless required.
- Use multipart uploads for large files.
- Use paginators for buckets with millions of objects.
- Generate presigned URLs for temporary access instead of making objects public.
- Store application configuration separately from user-generated data.
- Enable lifecycle rules to optimize storage costs.

---

# Summary

This section covered Boto3 automation for Amazon S3 including bucket management, object operations, uploads, downloads, versioning, encryption, lifecycle policies, replication, presigned URLs, multipart uploads, metadata management, Resource API usage, pagination, and exception handling. These examples represent common production automation tasks for AWS storage management.

---

# Amazon RDS

---

# Import Boto3

```python
import boto3

rds = boto3.client("rds")
```

---

# List DB Instances

```python
response = rds.describe_db_instances()

for db in response["DBInstances"]:
    print(db["DBInstanceIdentifier"])
```

---

# Describe DB Instance

```python
response = rds.describe_db_instances(
    DBInstanceIdentifier="production-db"
)

print(response["DBInstances"][0])
```

---

# Create DB Instance

```python
response = rds.create_db_instance(
    DBInstanceIdentifier="production-db",
    Engine="mysql",
    DBInstanceClass="db.t3.micro",
    AllocatedStorage=20,
    MasterUsername="admin",
    MasterUserPassword="Password@123"
)

print(response["DBInstance"]["DBInstanceIdentifier"])
```

---

# Modify DB Instance

```python
rds.modify_db_instance(
    DBInstanceIdentifier="production-db",
    DBInstanceClass="db.t3.small",
    ApplyImmediately=True
)
```

---

# Start DB Instance

```python
rds.start_db_instance(
    DBInstanceIdentifier="production-db"
)
```

---

# Stop DB Instance

```python
rds.stop_db_instance(
    DBInstanceIdentifier="production-db"
)
```

---

# Reboot DB Instance

```python
rds.reboot_db_instance(
    DBInstanceIdentifier="production-db"
)
```

---

# Delete DB Instance

```python
rds.delete_db_instance(
    DBInstanceIdentifier="production-db",
    SkipFinalSnapshot=True
)
```

---

# Create Snapshot

```python
response = rds.create_db_snapshot(
    DBSnapshotIdentifier="production-db-backup",
    DBInstanceIdentifier="production-db"
)

print(response["DBSnapshot"]["DBSnapshotIdentifier"])
```

---

# Restore Snapshot

```python
rds.restore_db_instance_from_db_snapshot(
    DBInstanceIdentifier="restored-db",
    DBSnapshotIdentifier="production-db-backup"
)
```

---

# Delete Snapshot

```python
rds.delete_db_snapshot(
    DBSnapshotIdentifier="production-db-backup"
)
```

---

# Wait Until DB Available

```python
waiter = rds.get_waiter("db_instance_available")

waiter.wait(
    DBInstanceIdentifier="production-db"
)
```

---

# Amazon Aurora

---

# List Clusters

```python
response = rds.describe_db_clusters()

for cluster in response["DBClusters"]:
    print(cluster["DBClusterIdentifier"])
```

---

# Create Aurora Cluster

```python
response = rds.create_db_cluster(
    DBClusterIdentifier="aurora-prod",
    Engine="aurora-mysql",
    MasterUsername="admin",
    MasterUserPassword="Password@123"
)

print(response["DBCluster"]["DBClusterIdentifier"])
```

---

# Delete Aurora Cluster

```python
rds.delete_db_cluster(
    DBClusterIdentifier="aurora-prod",
    SkipFinalSnapshot=True
)
```

---

# Amazon DynamoDB

---

# Import Client

```python
dynamodb = boto3.client("dynamodb")
```

---

# List Tables

```python
response = dynamodb.list_tables()

for table in response["TableNames"]:
    print(table)
```

---

# Describe Table

```python
response = dynamodb.describe_table(
    TableName="Employees"
)

print(response["Table"])
```

---

# Create Table

```python
dynamodb.create_table(
    TableName="Employees",
    AttributeDefinitions=[
        {
            "AttributeName": "EmployeeId",
            "AttributeType": "S"
        }
    ],
    KeySchema=[
        {
            "AttributeName": "EmployeeId",
            "KeyType": "HASH"
        }
    ],
    BillingMode="PAY_PER_REQUEST"
)
```

---

# Put Item

```python
dynamodb.put_item(
    TableName="Employees",
    Item={
        "EmployeeId": {
            "S": "1001"
        },
        "Name": {
            "S": "Surendra"
        }
    }
)
```

---

# Get Item

```python
response = dynamodb.get_item(
    TableName="Employees",
    Key={
        "EmployeeId": {
            "S": "1001"
        }
    }
)

print(response["Item"])
```

---

# Scan Table

```python
response = dynamodb.scan(
    TableName="Employees"
)

print(response["Items"])
```

---

# Query Table

```python
response = dynamodb.query(
    TableName="Employees",
    KeyConditionExpression="EmployeeId = :id",
    ExpressionAttributeValues={
        ":id": {
            "S": "1001"
        }
    }
)
```

---

# Delete Item

```python
dynamodb.delete_item(
    TableName="Employees",
    Key={
        "EmployeeId": {
            "S": "1001"
        }
    }
)
```

---

# Delete Table

```python
dynamodb.delete_table(
    TableName="Employees"
)
```

---

# Amazon ElastiCache

---

# Import Client

```python
elasticache = boto3.client("elasticache")
```

---

# List Cache Clusters

```python
response = elasticache.describe_cache_clusters()

for cluster in response["CacheClusters"]:
    print(cluster["CacheClusterId"])
```

---

# Create Redis Cluster

```python
elasticache.create_cache_cluster(
    CacheClusterId="redis-prod",
    Engine="redis",
    CacheNodeType="cache.t3.micro",
    NumCacheNodes=1
)
```

---

# Delete Cache Cluster

```python
elasticache.delete_cache_cluster(
    CacheClusterId="redis-prod"
)
```

---

# Amazon EFS

---

# Import Client

```python
efs = boto3.client("efs")
```

---

# List File Systems

```python
response = efs.describe_file_systems()

for fs in response["FileSystems"]:
    print(fs["FileSystemId"])
```

---

# Create File System

```python
response = efs.create_file_system()

print(response["FileSystemId"])
```

---

# Create Mount Target

```python
efs.create_mount_target(
    FileSystemId="fs-xxxxxxxx",
    SubnetId="subnet-xxxxxxxx",
    SecurityGroups=[
        "sg-xxxxxxxx"
    ]
)
```

---

# Delete File System

```python
efs.delete_file_system(
    FileSystemId="fs-xxxxxxxx"
)
```

---

# Amazon FSx

---

# Import Client

```python
fsx = boto3.client("fsx")
```

---

# List File Systems

```python
response = fsx.describe_file_systems()

for fs in response["FileSystems"]:
    print(fs["FileSystemId"])
```

---

# Create File System

```python
fsx.create_file_system(
    FileSystemType="WINDOWS",
    StorageCapacity=32,
    SubnetIds=[
        "subnet-xxxxxxxx"
    ]
)
```

---

# Delete File System

```python
fsx.delete_file_system(
    FileSystemId="fs-xxxxxxxx"
)
```

---

# AWS Backup

---

# Import Client

```python
backup = boto3.client("backup")
```

---

# List Backup Vaults

```python
response = backup.list_backup_vaults()

for vault in response["BackupVaultList"]:
    print(vault["BackupVaultName"])
```

---

# Create Backup Vault

```python
backup.create_backup_vault(
    BackupVaultName="ProductionVault"
)
```

---

# Start Backup Job

```python
backup.start_backup_job(
    BackupVaultName="ProductionVault",
    ResourceArn="arn:aws:ec2:..."
)
```

---

# Start Restore Job

```python
backup.start_restore_job(
    RecoveryPointArn="arn:aws:backup:..."
)
```

---

# AWS DataSync

---

# Import Client

```python
datasync = boto3.client("datasync")
```

---

# List Tasks

```python
response = datasync.list_tasks()

for task in response["Tasks"]:
    print(task["TaskArn"])
```

---

# Start Task

```python
datasync.start_task_execution(
    TaskArn="arn:aws:datasync:..."
)
```

---

# Describe Task

```python
response = datasync.describe_task(
    TaskArn="arn:aws:datasync:..."
)

print(response)
```

---

# AWS Storage Gateway

---

# Import Client

```python
gateway = boto3.client("storagegateway")
```

---

# List Gateways

```python
response = gateway.list_gateways()

for gw in response["Gateways"]:
    print(gw["GatewayARN"])
```

---

# Describe Gateway

```python
response = gateway.describe_gateway_information(
    GatewayARN="arn:aws:storagegateway:..."
)

print(response)
```

---

# Refresh Cache

```python
gateway.refresh_cache(
    FileShareARN="arn:aws:storagegateway:..."
)
```

---

# AWS Snow Family

---

# Import Client

```python
snow = boto3.client("snowball")
```

---

# List Jobs

```python
response = snow.list_jobs()

for job in response["JobListEntries"]:
    print(job["JobId"])
```

---

# Describe Job

```python
response = snow.describe_job(
    JobId="JIDxxxxxxxx"
)

print(response)
```

---

# Create Import Job

```python
snow.create_job(
    JobType="IMPORT",
    AddressId="ADxxxxxxxx",
    Resources={}
)
```

---

# Cancel Job

```python
snow.cancel_job(
    JobId="JIDxxxxxxxx"
)
```

---

# Pagination Example

```python
paginator = dynamodb.get_paginator(
    "list_tables"
)

for page in paginator.paginate():
    print(page["TableNames"])
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = rds.describe_db_instances()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Enable automated backups for Amazon RDS.
- Use waiters for database provisioning and modifications.
- Prefer DynamoDB Query over Scan for better performance.
- Encrypt RDS, EFS, and FSx using AWS KMS.
- Centralize backups with AWS Backup.
- Monitor DataSync execution status for migrations.
- Validate Snow Family jobs before data transfer.
- Implement retry logic for long-running database operations.

---

# Summary

This section covered Boto3 automation for Amazon RDS, Aurora, DynamoDB, ElastiCache, EFS, FSx, AWS Backup, DataSync, Storage Gateway, and Snow Family. These examples demonstrate production-ready automation for database provisioning, storage management, backup, migration, and recovery using Python.

---

