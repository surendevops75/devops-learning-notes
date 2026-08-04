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

# Amazon ECR

---

# Import Boto3

```python
import boto3

ecr = boto3.client("ecr")
```

---

# List Repositories

```python
response = ecr.describe_repositories()

for repo in response["repositories"]:
    print(repo["repositoryName"])
```

---

# Create Repository

```python
response = ecr.create_repository(
    repositoryName="my-app"
)

print(response["repository"]["repositoryUri"])
```

---

# Describe Repository

```python
response = ecr.describe_repositories(
    repositoryNames=["my-app"]
)

print(response["repositories"][0])
```

---

# Delete Repository

```python
ecr.delete_repository(
    repositoryName="my-app",
    force=True
)
```

---

# List Images

```python
response = ecr.list_images(
    repositoryName="my-app"
)

print(response["imageIds"])
```

---

# Describe Images

```python
response = ecr.describe_images(
    repositoryName="my-app"
)

for image in response["imageDetails"]:
    print(image["imageTags"])
```

---

# Start Image Scan

```python
ecr.start_image_scan(
    repositoryName="my-app",
    imageId={
        "imageTag": "latest"
    }
)
```

---

# Get Scan Findings

```python
response = ecr.describe_image_scan_findings(
    repositoryName="my-app",
    imageId={
        "imageTag": "latest"
    }
)

print(response["imageScanFindings"])
```

---

# Amazon ECS

---

# Import Client

```python
ecs = boto3.client("ecs")
```

---

# List Clusters

```python
response = ecs.list_clusters()

print(response["clusterArns"])
```

---

# Create Cluster

```python
response = ecs.create_cluster(
    clusterName="production"
)

print(response["cluster"]["clusterArn"])
```

---

# Delete Cluster

```python
ecs.delete_cluster(
    cluster="production"
)
```

---

# List Services

```python
response = ecs.list_services(
    cluster="production"
)

print(response["serviceArns"])
```

---

# Describe Services

```python
response = ecs.describe_services(
    cluster="production",
    services=["web"]
)

print(response["services"])
```

---

# Update Service

```python
ecs.update_service(
    cluster="production",
    service="web",
    desiredCount=3
)
```

---

# Force New Deployment

```python
ecs.update_service(
    cluster="production",
    service="web",
    forceNewDeployment=True
)
```

---

# List Tasks

```python
response = ecs.list_tasks(
    cluster="production"
)

print(response["taskArns"])
```

---

# Stop Task

```python
ecs.stop_task(
    cluster="production",
    task="task-id"
)
```

---

# Register Task Definition

```python
response = ecs.register_task_definition(
    family="web-app",
    networkMode="awsvpc",
    containerDefinitions=[]
)

print(response["taskDefinition"]["taskDefinitionArn"])
```

---

# Amazon EKS

---

# Import Client

```python
eks = boto3.client("eks")
```

---

# List Clusters

```python
response = eks.list_clusters()

print(response["clusters"])
```

---

# Describe Cluster

```python
response = eks.describe_cluster(
    name="production"
)

print(response["cluster"])
```

---

# Create Cluster

```python
eks.create_cluster(
    name="production",
    version="1.31",
    roleArn="arn:aws:iam::123456789012:role/EKSRole",
    resourcesVpcConfig={
        "subnetIds": [
            "subnet-1",
            "subnet-2"
        ]
    }
)
```

---

# Delete Cluster

```python
eks.delete_cluster(
    name="production"
)
```

---

# List Nodegroups

```python
response = eks.list_nodegroups(
    clusterName="production"
)

print(response["nodegroups"])
```

---

# Describe Nodegroup

```python
response = eks.describe_nodegroup(
    clusterName="production",
    nodegroupName="workers"
)

print(response["nodegroup"])
```

---

# Delete Nodegroup

```python
eks.delete_nodegroup(
    clusterName="production",
    nodegroupName="workers"
)
```

---

# AWS Lambda

---

# Import Client

```python
lambda_client = boto3.client("lambda")
```

---

# List Functions

```python
response = lambda_client.list_functions()

for function in response["Functions"]:
    print(function["FunctionName"])
```

---

# Get Function

```python
response = lambda_client.get_function(
    FunctionName="processOrders"
)

print(response["Configuration"])
```

---

# Invoke Function

```python
response = lambda_client.invoke(
    FunctionName="processOrders",
    Payload=b'{}'
)

print(response["StatusCode"])
```

---

# Update Function Code

```python
lambda_client.update_function_code(
    FunctionName="processOrders",
    ZipFile=open("lambda.zip", "rb").read()
)
```

---

# Delete Function

```python
lambda_client.delete_function(
    FunctionName="processOrders"
)
```

---

# Amazon API Gateway

---

# Import Client

```python
apigateway = boto3.client("apigateway")
```

---

# List APIs

```python
response = apigateway.get_rest_apis()

for api in response["items"]:
    print(api["name"])
```

---

# Get API

```python
response = apigateway.get_rest_api(
    restApiId="abc123"
)

print(response)
```

---

# Create REST API

```python
response = apigateway.create_rest_api(
    name="OrdersAPI"
)

print(response["id"])
```

---

# Delete REST API

```python
apigateway.delete_rest_api(
    restApiId="abc123"
)
```

---

# Amazon EventBridge

---

# Import Client

```python
events = boto3.client("events")
```

---

# List Rules

```python
response = events.list_rules()

for rule in response["Rules"]:
    print(rule["Name"])
```

---

# Create Rule

```python
events.put_rule(
    Name="DailySchedule",
    ScheduleExpression="rate(1 day)"
)
```

---

# Put Event

```python
events.put_events(
    Entries=[
        {
            "Source": "application",
            "DetailType": "OrderCreated",
            "Detail": "{}"
        }
    ]
)
```

---

# Delete Rule

```python
events.delete_rule(
    Name="DailySchedule"
)
```

---

# Amazon SNS

---

# Import Client

```python
sns = boto3.client("sns")
```

---

# List Topics

```python
response = sns.list_topics()

print(response["Topics"])
```

---

# Create Topic

```python
response = sns.create_topic(
    Name="alerts"
)

print(response["TopicArn"])
```

---

# Publish Message

```python
sns.publish(
    TopicArn="arn:aws:sns:...",
    Subject="Deployment",
    Message="Deployment completed successfully."
)
```

---

# Subscribe Endpoint

```python
sns.subscribe(
    TopicArn="arn:aws:sns:...",
    Protocol="email",
    Endpoint="admin@example.com"
)
```

---

# Delete Topic

```python
sns.delete_topic(
    TopicArn="arn:aws:sns:..."
)
```

---

# Amazon SQS

---

# Import Client

```python
sqs = boto3.client("sqs")
```

---

# List Queues

```python
response = sqs.list_queues()

print(response["QueueUrls"])
```

---

# Create Queue

```python
response = sqs.create_queue(
    QueueName="orders"
)

print(response["QueueUrl"])
```

---

# Send Message

```python
sqs.send_message(
    QueueUrl="https://sqs.ap-south-1.amazonaws.com/123456789012/orders",
    MessageBody="Order Created"
)
```

---

# Receive Message

```python
response = sqs.receive_message(
    QueueUrl="https://sqs.ap-south-1.amazonaws.com/123456789012/orders"
)

print(response.get("Messages"))
```

---

# Delete Message

```python
sqs.delete_message(
    QueueUrl="https://sqs.ap-south-1.amazonaws.com/123456789012/orders",
    ReceiptHandle="receipt-handle"
)
```

---

# Purge Queue

```python
sqs.purge_queue(
    QueueUrl="https://sqs.ap-south-1.amazonaws.com/123456789012/orders"
)
```

---

# AWS Step Functions

---

# Import Client

```python
stepfunctions = boto3.client("stepfunctions")
```

---

# List State Machines

```python
response = stepfunctions.list_state_machines()

for sm in response["stateMachines"]:
    print(sm["name"])
```

---

# Describe State Machine

```python
response = stepfunctions.describe_state_machine(
    stateMachineArn="arn:aws:states:..."
)

print(response)
```

---

# Start Execution

```python
response = stepfunctions.start_execution(
    stateMachineArn="arn:aws:states:...",
    input="{}"
)

print(response["executionArn"])
```

---

# List Executions

```python
response = stepfunctions.list_executions(
    stateMachineArn="arn:aws:states:..."
)

print(response["executions"])
```

---

# Stop Execution

```python
stepfunctions.stop_execution(
    executionArn="arn:aws:states:..."
)
```

---

# Waiter Example

```python
waiter = eks.get_waiter("cluster_active")

waiter.wait(
    name="production"
)
```

---

# Pagination Example

```python
paginator = lambda_client.get_paginator(
    "list_functions"
)

for page in paginator.paginate():
    for function in page["Functions"]:
        print(function["FunctionName"])
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = ecs.list_clusters()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Enable image scanning in Amazon ECR.
- Store container images with immutable tags.
- Use rolling updates for ECS and EKS deployments.
- Keep Lambda functions stateless.
- Use EventBridge instead of polling wherever possible.
- Configure SNS with Dead Letter Queues for reliable notifications.
- Use SQS visibility timeout appropriately.
- Implement retries and exponential backoff for Step Functions workflows.
- Use paginators for large result sets.
- Always catch and handle `ClientError` exceptions.

---

# Summary

This section covered Boto3 automation for Amazon ECR, ECS, EKS, Lambda, API Gateway, EventBridge, SNS, SQS, and Step Functions. These examples demonstrate practical automation patterns for container platforms, serverless applications, messaging systems, and event-driven architectures using Python.

---

# Amazon CloudWatch

---

# Import Boto3

```python
import boto3

cloudwatch = boto3.client("cloudwatch")
```

---

# List Metrics

```python
response = cloudwatch.list_metrics()

for metric in response["Metrics"]:
    print(metric["MetricName"])
```

---

# Get Metric Statistics

```python
response = cloudwatch.get_metric_statistics(
    Namespace="AWS/EC2",
    MetricName="CPUUtilization",
    StartTime=start_time,
    EndTime=end_time,
    Period=300,
    Statistics=["Average"]
)

print(response["Datapoints"])
```

---

# Put Custom Metric

```python
cloudwatch.put_metric_data(
    Namespace="Custom/Application",
    MetricData=[
        {
            "MetricName": "ActiveUsers",
            "Value": 100,
            "Unit": "Count"
        }
    ]
)
```

---

# Create Alarm

```python
cloudwatch.put_metric_alarm(
    AlarmName="HighCPU",
    MetricName="CPUUtilization",
    Namespace="AWS/EC2",
    Statistic="Average",
    Threshold=80,
    ComparisonOperator="GreaterThanThreshold",
    EvaluationPeriods=2,
    Period=300
)
```

---

# Describe Alarms

```python
response = cloudwatch.describe_alarms()

for alarm in response["MetricAlarms"]:
    print(alarm["AlarmName"])
```

---

# Delete Alarm

```python
cloudwatch.delete_alarms(
    AlarmNames=[
        "HighCPU"
    ]
)
```

---

# CloudWatch Logs

---

# Import Client

```python
logs = boto3.client("logs")
```

---

# List Log Groups

```python
response = logs.describe_log_groups()

for group in response["logGroups"]:
    print(group["logGroupName"])
```

---

# Create Log Group

```python
logs.create_log_group(
    logGroupName="application-logs"
)
```

---

# Delete Log Group

```python
logs.delete_log_group(
    logGroupName="application-logs"
)
```

---

# List Log Streams

```python
response = logs.describe_log_streams(
    logGroupName="application-logs"
)

print(response["logStreams"])
```

---

# Get Log Events

```python
response = logs.get_log_events(
    logGroupName="application-logs",
    logStreamName="stream-name"
)

print(response["events"])
```

---

# Filter Log Events

```python
response = logs.filter_log_events(
    logGroupName="application-logs",
    filterPattern="ERROR"
)

print(response["events"])
```

---

# AWS CloudTrail

---

# Import Client

```python
cloudtrail = boto3.client("cloudtrail")
```

---

# List Trails

```python
response = cloudtrail.describe_trails()

for trail in response["trailList"]:
    print(trail["Name"])
```

---

# Get Trail Status

```python
response = cloudtrail.get_trail_status(
    Name="organization-trail"
)

print(response)
```

---

# Create Trail

```python
cloudtrail.create_trail(
    Name="organization-trail",
    S3BucketName="audit-logs"
)
```

---

# Start Logging

```python
cloudtrail.start_logging(
    Name="organization-trail"
)
```

---

# Stop Logging

```python
cloudtrail.stop_logging(
    Name="organization-trail"
)
```

---

# Lookup Events

```python
response = cloudtrail.lookup_events()

print(response["Events"])
```

---

# AWS Config

---

# Import Client

```python
config = boto3.client("config")
```

---

# Describe Configuration Recorders

```python
response = config.describe_configuration_recorders()

print(response["ConfigurationRecorders"])
```

---

# Describe Config Rules

```python
response = config.describe_config_rules()

for rule in response["ConfigRules"]:
    print(rule["ConfigRuleName"])
```

---

# Get Compliance Summary

```python
response = config.get_compliance_summary_by_config_rule()

print(response)
```

---

# AWS CodeCommit

---

# Import Client

```python
codecommit = boto3.client("codecommit")
```

---

# List Repositories

```python
response = codecommit.list_repositories()

for repo in response["repositories"]:
    print(repo["repositoryName"])
```

---

# Create Repository

```python
response = codecommit.create_repository(
    repositoryName="devops-repo"
)

print(response["repositoryMetadata"])
```

---

# Get Repository

```python
response = codecommit.get_repository(
    repositoryName="devops-repo"
)

print(response["repositoryMetadata"])
```

---

# Delete Repository

```python
codecommit.delete_repository(
    repositoryName="devops-repo"
)
```

---

# AWS CodeBuild

---

# Import Client

```python
codebuild = boto3.client("codebuild")
```

---

# List Projects

```python
response = codebuild.list_projects()

print(response["projects"])
```

---

# Start Build

```python
response = codebuild.start_build(
    projectName="my-build"
)

print(response["build"]["id"])
```

---

# Batch Get Builds

```python
response = codebuild.batch_get_builds(
    ids=[
        "build-id"
    ]
)

print(response["builds"])
```

---

# Stop Build

```python
codebuild.stop_build(
    id="build-id"
)
```

---

# AWS CodeDeploy

---

# Import Client

```python
codedeploy = boto3.client("codedeploy")
```

---

# List Applications

```python
response = codedeploy.list_applications()

print(response["applications"])
```

---

# Create Deployment

```python
response = codedeploy.create_deployment(
    applicationName="web-app",
    deploymentGroupName="production"
)

print(response["deploymentId"])
```

---

# Get Deployment

```python
response = codedeploy.get_deployment(
    deploymentId="deployment-id"
)

print(response["deploymentInfo"])
```

---

# Stop Deployment

```python
codedeploy.stop_deployment(
    deploymentId="deployment-id"
)
```

---

# AWS CodePipeline

---

# Import Client

```python
pipeline = boto3.client("codepipeline")
```

---

# List Pipelines

```python
response = pipeline.list_pipelines()

for p in response["pipelines"]:
    print(p["name"])
```

---

# Get Pipeline

```python
response = pipeline.get_pipeline(
    name="production-pipeline"
)

print(response["pipeline"])
```

---

# Start Pipeline Execution

```python
response = pipeline.start_pipeline_execution(
    name="production-pipeline"
)

print(response["pipelineExecutionId"])
```

---

# Get Pipeline State

```python
response = pipeline.get_pipeline_state(
    name="production-pipeline"
)

print(response["stageStates"])
```

---

# AWS CodeArtifact

---

# Import Client

```python
codeartifact = boto3.client("codeartifact")
```

---

# List Domains

```python
response = codeartifact.list_domains()

print(response["domains"])
```

---

# List Repositories

```python
response = codeartifact.list_repositories()

print(response["repositories"])
```

---

# Get Authorization Token

```python
response = codeartifact.get_authorization_token(
    domain="my-domain"
)

print(response["authorizationToken"])
```

---

# AWS Systems Manager (SSM)

---

# Import Client

```python
ssm = boto3.client("ssm")
```

---

# Describe Managed Instances

```python
response = ssm.describe_instance_information()

for instance in response["InstanceInformationList"]:
    print(instance["InstanceId"])
```

---

# Start Session

```python
response = ssm.start_session(
    Target="i-0123456789abcdef0"
)

print(response["SessionId"])
```

---

# Send Command

```python
response = ssm.send_command(
    DocumentName="AWS-RunShellScript",
    InstanceIds=[
        "i-0123456789abcdef0"
    ],
    Parameters={
        "commands": [
            "uptime"
        ]
    }
)

print(response["Command"]["CommandId"])
```

---

# Get Command Invocation

```python
response = ssm.get_command_invocation(
    CommandId="command-id",
    InstanceId="i-0123456789abcdef0"
)

print(response["Status"])
```

---

# List Commands

```python
response = ssm.list_commands()

print(response["Commands"])
```

---

# Parameter Store

---

# Create Parameter

```python
ssm.put_parameter(
    Name="/prod/db/password",
    Value="Password123",
    Type="SecureString",
    Overwrite=True
)
```

---

# Get Parameter

```python
response = ssm.get_parameter(
    Name="/prod/db/password",
    WithDecryption=True
)

print(response["Parameter"]["Value"])
```

---

# Get Multiple Parameters

```python
response = ssm.get_parameters(
    Names=[
        "/prod/db/password",
        "/prod/api/url"
    ],
    WithDecryption=True
)

print(response["Parameters"])
```

---

# Delete Parameter

```python
ssm.delete_parameter(
    Name="/prod/db/password"
)
```

---

# OpsCenter

---

# List OpsItems

```python
response = ssm.list_ops_items()

print(response["OpsItemSummaries"])
```

---

# Get OpsItem

```python
response = ssm.get_ops_item(
    OpsItemId="oi-xxxxxxxx"
)

print(response["OpsItem"])
```

---

# Create OpsItem

```python
response = ssm.create_ops_item(
    Title="High CPU Alert",
    Source="CloudWatch"
)

print(response["OpsItemId"])
```

---

# Pagination Example

```python
paginator = logs.get_paginator(
    "describe_log_groups"
)

for page in paginator.paginate():
    for group in page["logGroups"]:
        print(group["logGroupName"])
```

---

# Waiter Example

```python
waiter = codebuild.get_waiter(
    "build_succeeded"
)

waiter.wait(
    ids=[
        "build-id"
    ]
)
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = cloudwatch.describe_alarms()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Create CloudWatch alarms for all production workloads.
- Use CloudWatch Logs retention policies to control costs.
- Enable CloudTrail organization-wide for auditing.
- Store application configuration in Parameter Store.
- Use Systems Manager Session Manager instead of SSH whenever possible.
- Automate deployments using CodePipeline and CodeBuild.
- Monitor deployment status before proceeding with dependent stages.
- Use paginators when retrieving large datasets.
- Catch and handle `ClientError` exceptions consistently.

---

# Summary

This section covered Boto3 automation for Amazon CloudWatch, CloudWatch Logs, CloudTrail, AWS Config, CodeCommit, CodeBuild, CodeDeploy, CodePipeline, CodeArtifact, Systems Manager (SSM), Parameter Store, and OpsCenter. These examples provide production-ready automation for monitoring, logging, CI/CD, infrastructure governance, and operational management using Python.

---

# Amazon Route 53

---

# Import Boto3

```python
import boto3

route53 = boto3.client("route53")
```

---

# List Hosted Zones

```python
response = route53.list_hosted_zones()

for zone in response["HostedZones"]:
    print(zone["Name"])
```

---

# Get Hosted Zone

```python
response = route53.get_hosted_zone(
    Id="Z123456789ABC"
)

print(response["HostedZone"])
```

---

# Create Hosted Zone

```python
import time

response = route53.create_hosted_zone(
    Name="example.com",
    CallerReference=str(time.time())
)

print(response["HostedZone"]["Id"])
```

---

# Delete Hosted Zone

```python
route53.delete_hosted_zone(
    Id="Z123456789ABC"
)
```

---

# List DNS Records

```python
response = route53.list_resource_record_sets(
    HostedZoneId="Z123456789ABC"
)

for record in response["ResourceRecordSets"]:
    print(record["Name"])
```

---

# Update DNS Record

```python
route53.change_resource_record_sets(
    HostedZoneId="Z123456789ABC",
    ChangeBatch={
        "Changes": []
    }
)
```

---

# CloudFront

---

# Import Client

```python
cloudfront = boto3.client("cloudfront")
```

---

# List Distributions

```python
response = cloudfront.list_distributions()

print(response["DistributionList"]["Items"])
```

---

# Get Distribution

```python
response = cloudfront.get_distribution(
    Id="E123456789ABC"
)

print(response["Distribution"])
```

---

# Create Invalidation

```python
response = cloudfront.create_invalidation(
    DistributionId="E123456789ABC",
    InvalidationBatch={
        "Paths": {
            "Quantity": 1,
            "Items": ["/*"]
        },
        "CallerReference": str(time.time())
    }
)

print(response["Invalidation"]["Id"])
```

---

# AWS WAF

---

# Import Client

```python
waf = boto3.client("wafv2")
```

---

# List Web ACLs

```python
response = waf.list_web_acls(
    Scope="REGIONAL"
)

for acl in response["WebACLs"]:
    print(acl["Name"])
```

---

# Get Web ACL

```python
response = waf.get_web_acl(
    Scope="REGIONAL",
    Id="acl-id",
    Name="ProductionACL"
)

print(response["WebACL"])
```

---

# Create Web ACL

```python
response = waf.create_web_acl(
    Name="ProductionACL",
    Scope="REGIONAL",
    DefaultAction={
        "Allow": {}
    },
    VisibilityConfig={
        "CloudWatchMetricsEnabled": True,
        "MetricName": "ProductionACL",
        "SampledRequestsEnabled": True
    },
    Rules=[]
)

print(response["Summary"]["Id"])
```

---

# Delete Web ACL

```python
waf.delete_web_acl(
    Scope="REGIONAL",
    Id="acl-id",
    Name="ProductionACL",
    LockToken="lock-token"
)
```

---

# AWS Shield

---

# Import Client

```python
shield = boto3.client("shield")
```

---

# List Protections

```python
response = shield.list_protections()

print(response["Protections"])
```

---

# Describe Protection

```python
response = shield.describe_protection(
    ProtectionId="protection-id"
)

print(response["Protection"])
```

---

# Amazon GuardDuty

---

# Import Client

```python
guardduty = boto3.client("guardduty")
```

---

# List Detectors

```python
response = guardduty.list_detectors()

print(response["DetectorIds"])
```

---

# List Findings

```python
response = guardduty.list_findings(
    DetectorId="detector-id"
)

print(response["FindingIds"])
```

---

# Get Findings

```python
response = guardduty.get_findings(
    DetectorId="detector-id",
    FindingIds=[
        "finding-id"
    ]
)

print(response["Findings"])
```

---

# AWS Security Hub

---

# Import Client

```python
securityhub = boto3.client("securityhub")
```

---

# Enable Security Hub

```python
securityhub.enable_security_hub()
```

---

# Get Findings

```python
response = securityhub.get_findings()

print(response["Findings"])
```

---

# List Standards

```python
response = securityhub.get_enabled_standards()

print(response["StandardsSubscriptions"])
```

---

# Amazon Inspector

---

# Import Client

```python
inspector = boto3.client("inspector2")
```

---

# List Findings

```python
response = inspector.list_findings()

print(response["findings"])
```

---

# List Coverage

```python
response = inspector.list_coverage()

print(response["coveredResources"])
```

---

# Amazon Macie

---

# Import Client

```python
macie = boto3.client("macie2")
```

---

# List Classification Jobs

```python
response = macie.list_classification_jobs()

print(response["items"])
```

---

# List Findings

```python
response = macie.list_findings()

print(response["findingIds"])
```

---

# Amazon Detective

---

# Import Client

```python
detective = boto3.client("detective")
```

---

# List Graphs

```python
response = detective.list_graphs()

print(response["GraphList"])
```

---

# List Members

```python
response = detective.list_members(
    GraphArn="graph-arn"
)

print(response["MemberDetails"])
```

---

# Amazon OpenSearch Service

---

# Import Client

```python
opensearch = boto3.client("opensearch")
```

---

# List Domains

```python
response = opensearch.list_domain_names()

print(response["DomainNames"])
```

---

# Describe Domain

```python
response = opensearch.describe_domain(
    DomainName="production"
)

print(response["DomainStatus"])
```

---

# Create Domain

```python
opensearch.create_domain(
    DomainName="production",
    EngineVersion="OpenSearch_2.17"
)
```

---

# Delete Domain

```python
opensearch.delete_domain(
    DomainName="production"
)
```

---

# Amazon Managed Prometheus

---

# Import Client

```python
amp = boto3.client("amp")
```

---

# List Workspaces

```python
response = amp.list_workspaces()

print(response["workspaces"])
```

---

# Create Workspace

```python
response = amp.create_workspace(
    alias="production-monitoring"
)

print(response["workspaceId"])
```

---

# Describe Workspace

```python
response = amp.describe_workspace(
    workspaceId="ws-xxxxxxxx"
)

print(response["workspace"])
```

---

# Delete Workspace

```python
amp.delete_workspace(
    workspaceId="ws-xxxxxxxx"
)
```

---

# Amazon Managed Grafana

---

# Import Client

```python
grafana = boto3.client("grafana")
```

---

# List Workspaces

```python
response = grafana.list_workspaces()

print(response["workspaces"])
```

---

# Describe Workspace

```python
response = grafana.describe_workspace(
    workspaceId="g-xxxxxxxx"
)

print(response["workspace"])
```

---

# Delete Workspace

```python
grafana.delete_workspace(
    workspaceId="g-xxxxxxxx"
)
```

---

# AWS X-Ray

---

# Import Client

```python
xray = boto3.client("xray")
```

---

# Get Trace Summaries

```python
response = xray.get_trace_summaries(
    StartTime=start_time,
    EndTime=end_time
)

print(response["TraceSummaries"])
```

---

# Batch Get Traces

```python
response = xray.batch_get_traces(
    TraceIds=[
        "trace-id"
    ]
)

print(response["Traces"])
```

---

# Get Service Graph

```python
response = xray.get_service_graph(
    StartTime=start_time,
    EndTime=end_time
)

print(response["Services"])
```

---

# Pagination Example

```python
paginator = securityhub.get_paginator(
    "get_findings"
)

for page in paginator.paginate():
    print(len(page["Findings"]))
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = guardduty.list_detectors()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Use Route 53 health checks with failover routing policies.
- Minimize CloudFront invalidations by versioning static assets.
- Protect internet-facing applications with AWS WAF and Shield.
- Enable GuardDuty, Security Hub, Inspector, and Macie in all production accounts.
- Review high-severity security findings regularly.
- Use Amazon Managed Prometheus and Grafana for Kubernetes monitoring.
- Monitor OpenSearch cluster health and storage utilization.
- Use pagination for large result sets.
- Handle `ClientError` exceptions for all security-related API calls.

---

# Summary

This section covered Boto3 automation for Amazon Route 53, CloudFront, AWS WAF, Shield, GuardDuty, Security Hub, Inspector, Macie, Detective, OpenSearch Service, Amazon Managed Prometheus, Amazon Managed Grafana, and AWS X-Ray. These examples demonstrate practical automation for networking, security, threat detection, observability, and search services in AWS.

---

# Amazon Bedrock

---

# Import Boto3

```python
import boto3

bedrock = boto3.client("bedrock")
```

---

# List Foundation Models

```python
response = bedrock.list_foundation_models()

for model in response["modelSummaries"]:
    print(model["modelId"])
```

---

# Get Foundation Model

```python
response = bedrock.get_foundation_model(
    modelIdentifier="amazon.titan-text-express-v1"
)

print(response["modelDetails"])
```

---

# Amazon Bedrock Runtime

---

# Import Runtime Client

```python
runtime = boto3.client("bedrock-runtime")
```

---

# Invoke Model

```python
import json

payload = {
    "inputText": "Explain Amazon EC2 in simple terms."
}

response = runtime.invoke_model(
    modelId="amazon.titan-text-express-v1",
    body=json.dumps(payload)
)

print(response["body"].read())
```

---

# Invoke Model with Streaming

```python
response = runtime.invoke_model_with_response_stream(
    modelId="amazon.titan-text-express-v1",
    body=json.dumps(payload)
)

for event in response["body"]:
    print(event)
```

---

# Amazon Q Developer

---

# Import Client

```python
qbusiness = boto3.client("qbusiness")
```

---

# List Applications

```python
response = qbusiness.list_applications()

for app in response["applications"]:
    print(app["displayName"])
```

---

# Get Application

```python
response = qbusiness.get_application(
    applicationId="application-id"
)

print(response["displayName"])
```

---

# List Indices

```python
response = qbusiness.list_indices(
    applicationId="application-id"
)

print(response["indices"])
```

---

# Amazon Textract

---

# Import Client

```python
textract = boto3.client("textract")
```

---

# Detect Document Text

```python
response = textract.detect_document_text(
    Document={
        "Bytes": open("invoice.png", "rb").read()
    }
)

for block in response["Blocks"]:
    if block["BlockType"] == "LINE":
        print(block["Text"])
```

---

# Analyze Document

```python
response = textract.analyze_document(
    Document={
        "Bytes": open("invoice.png", "rb").read()
    },
    FeatureTypes=[
        "TABLES",
        "FORMS"
    ]
)

print(response["Blocks"])
```

---

# Start Document Analysis

```python
response = textract.start_document_analysis(
    DocumentLocation={
        "S3Object": {
            "Bucket": "documents",
            "Name": "invoice.pdf"
        }
    },
    FeatureTypes=[
        "TABLES",
        "FORMS"
    ]
)

print(response["JobId"])
```

---

# Get Document Analysis

```python
response = textract.get_document_analysis(
    JobId="job-id"
)

print(response["Blocks"])
```

---

# Amazon Rekognition

---

# Import Client

```python
rekognition = boto3.client("rekognition")
```

---

# Detect Labels

```python
response = rekognition.detect_labels(
    Image={
        "S3Object": {
            "Bucket": "images",
            "Name": "car.jpg"
        }
    }
)

for label in response["Labels"]:
    print(label["Name"])
```

---

# Detect Faces

```python
response = rekognition.detect_faces(
    Image={
        "S3Object": {
            "Bucket": "images",
            "Name": "person.jpg"
        }
    }
)

print(response["FaceDetails"])
```

---

# Compare Faces

```python
response = rekognition.compare_faces(
    SourceImage={
        "S3Object": {
            "Bucket": "images",
            "Name": "source.jpg"
        }
    },
    TargetImage={
        "S3Object": {
            "Bucket": "images",
            "Name": "target.jpg"
        }
    }
)

print(response["FaceMatches"])
```

---

# Detect Text

```python
response = rekognition.detect_text(
    Image={
        "S3Object": {
            "Bucket": "images",
            "Name": "license.jpg"
        }
    }
)

print(response["TextDetections"])
```

---

# Amazon Comprehend

---

# Import Client

```python
comprehend = boto3.client("comprehend")
```

---

# Detect Sentiment

```python
response = comprehend.detect_sentiment(
    Text="AWS services are excellent.",
    LanguageCode="en"
)

print(response["Sentiment"])
```

---

# Detect Entities

```python
response = comprehend.detect_entities(
    Text="John works for Amazon.",
    LanguageCode="en"
)

print(response["Entities"])
```

---

# Detect Key Phrases

```python
response = comprehend.detect_key_phrases(
    Text="Cloud computing enables scalable applications.",
    LanguageCode="en"
)

print(response["KeyPhrases"])
```

---

# Detect Language

```python
response = comprehend.detect_dominant_language(
    Text="Bonjour tout le monde"
)

print(response["Languages"])
```

---

# Amazon Translate

---

# Import Client

```python
translate = boto3.client("translate")
```

---

# Translate Text

```python
response = translate.translate_text(
    Text="Hello World",
    SourceLanguageCode="en",
    TargetLanguageCode="es"
)

print(response["TranslatedText"])
```

---

# Amazon Polly

---

# Import Client

```python
polly = boto3.client("polly")
```

---

# Convert Text to Speech

```python
response = polly.synthesize_speech(
    Text="Welcome to AWS",
    OutputFormat="mp3",
    VoiceId="Joanna"
)

audio = response["AudioStream"].read()
```

---

# List Voices

```python
response = polly.describe_voices()

for voice in response["Voices"]:
    print(voice["Name"])
```

---

# Amazon Transcribe

---

# Import Client

```python
transcribe = boto3.client("transcribe")
```

---

# Start Transcription Job

```python
response = transcribe.start_transcription_job(
    TranscriptionJobName="meeting-audio",
    LanguageCode="en-US",
    Media={
        "MediaFileUri": "s3://media/audio.mp3"
    }
)

print(response["TranscriptionJob"])
```

---

# Get Transcription Job

```python
response = transcribe.get_transcription_job(
    TranscriptionJobName="meeting-audio"
)

print(response["TranscriptionJob"]["TranscriptionJobStatus"])
```

---

# List Transcription Jobs

```python
response = transcribe.list_transcription_jobs()

print(response["TranscriptionJobSummaries"])
```

---

# Amazon Personalize

---

# Import Client

```python
personalize = boto3.client("personalize")
```

---

# List Datasets

```python
response = personalize.list_datasets()

print(response["datasets"])
```

---

# List Solutions

```python
response = personalize.list_solutions()

print(response["solutions"])
```

---

# Describe Solution

```python
response = personalize.describe_solution(
    solutionArn="arn:aws:personalize:..."
)

print(response["solution"])
```

---

# Amazon Forecast

---

# Import Client

```python
forecast = boto3.client("forecast")
```

---

# List Predictors

```python
response = forecast.list_predictors()

print(response["Predictors"])
```

---

# Describe Predictor

```python
response = forecast.describe_predictor(
    PredictorArn="arn:aws:forecast:..."
)

print(response["PredictorName"])
```

---

# Delete Predictor

```python
forecast.delete_predictor(
    PredictorArn="arn:aws:forecast:..."
)
```

---

# Pagination Example

```python
paginator = rekognition.get_paginator(
    "detect_labels"
)
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = bedrock.list_foundation_models()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Keep prompts and model inputs free of sensitive information.
- Store large documents in Amazon S3 before processing with Textract.
- Validate confidence scores returned by Rekognition before taking automated actions.
- Use Comprehend language detection for multilingual applications.
- Cache frequently requested AI responses when appropriate.
- Use asynchronous APIs for long-running document processing jobs.
- Handle API throttling with retries and exponential backoff.
- Secure AI service access using IAM roles and least-privilege permissions.

---

# Summary

This section covered Boto3 automation for Amazon Bedrock, Bedrock Runtime, Amazon Q Developer, Textract, Rekognition, Comprehend, Translate, Polly, Transcribe, Personalize, and Forecast. These examples demonstrate practical AI/ML automation patterns for generative AI, document processing, computer vision, speech, translation, and natural language processing using Python.

---

# Amazon Bedrock

---

# Import Boto3

```python
import boto3

bedrock = boto3.client("bedrock")
```

---

# List Foundation Models

```python
response = bedrock.list_foundation_models()

for model in response["modelSummaries"]:
    print(model["modelId"])
```

---

# Get Foundation Model

```python
response = bedrock.get_foundation_model(
    modelIdentifier="amazon.titan-text-express-v1"
)

print(response["modelDetails"])
```

---

# Amazon Bedrock Runtime

---

# Import Runtime Client

```python
runtime = boto3.client("bedrock-runtime")
```

---

# Invoke Model

```python
import json

payload = {
    "inputText": "Explain Amazon EC2 in simple terms."
}

response = runtime.invoke_model(
    modelId="amazon.titan-text-express-v1",
    body=json.dumps(payload)
)

print(response["body"].read())
```

---

# Invoke Model with Streaming

```python
response = runtime.invoke_model_with_response_stream(
    modelId="amazon.titan-text-express-v1",
    body=json.dumps(payload)
)

for event in response["body"]:
    print(event)
```

---

# Amazon Q Developer

---

# Import Client

```python
qbusiness = boto3.client("qbusiness")
```

---

# List Applications

```python
response = qbusiness.list_applications()

for app in response["applications"]:
    print(app["displayName"])
```

---

# Get Application

```python
response = qbusiness.get_application(
    applicationId="application-id"
)

print(response["displayName"])
```

---

# List Indices

```python
response = qbusiness.list_indices(
    applicationId="application-id"
)

print(response["indices"])
```

---

# Amazon Textract

---

# Import Client

```python
textract = boto3.client("textract")
```

---

# Detect Document Text

```python
response = textract.detect_document_text(
    Document={
        "Bytes": open("invoice.png", "rb").read()
    }
)

for block in response["Blocks"]:
    if block["BlockType"] == "LINE":
        print(block["Text"])
```

---

# Analyze Document

```python
response = textract.analyze_document(
    Document={
        "Bytes": open("invoice.png", "rb").read()
    },
    FeatureTypes=[
        "TABLES",
        "FORMS"
    ]
)

print(response["Blocks"])
```

---

# Start Document Analysis

```python
response = textract.start_document_analysis(
    DocumentLocation={
        "S3Object": {
            "Bucket": "documents",
            "Name": "invoice.pdf"
        }
    },
    FeatureTypes=[
        "TABLES",
        "FORMS"
    ]
)

print(response["JobId"])
```

---

# Get Document Analysis

```python
response = textract.get_document_analysis(
    JobId="job-id"
)

print(response["Blocks"])
```

---

# Amazon Rekognition

---

# Import Client

```python
rekognition = boto3.client("rekognition")
```

---

# Detect Labels

```python
response = rekognition.detect_labels(
    Image={
        "S3Object": {
            "Bucket": "images",
            "Name": "car.jpg"
        }
    }
)

for label in response["Labels"]:
    print(label["Name"])
```

---

# Detect Faces

```python
response = rekognition.detect_faces(
    Image={
        "S3Object": {
            "Bucket": "images",
            "Name": "person.jpg"
        }
    }
)

print(response["FaceDetails"])
```

---

# Compare Faces

```python
response = rekognition.compare_faces(
    SourceImage={
        "S3Object": {
            "Bucket": "images",
            "Name": "source.jpg"
        }
    },
    TargetImage={
        "S3Object": {
            "Bucket": "images",
            "Name": "target.jpg"
        }
    }
)

print(response["FaceMatches"])
```

---

# Detect Text

```python
response = rekognition.detect_text(
    Image={
        "S3Object": {
            "Bucket": "images",
            "Name": "license.jpg"
        }
    }
)

print(response["TextDetections"])
```

---

# Amazon Comprehend

---

# Import Client

```python
comprehend = boto3.client("comprehend")
```

---

# Detect Sentiment

```python
response = comprehend.detect_sentiment(
    Text="AWS services are excellent.",
    LanguageCode="en"
)

print(response["Sentiment"])
```

---

# Detect Entities

```python
response = comprehend.detect_entities(
    Text="John works for Amazon.",
    LanguageCode="en"
)

print(response["Entities"])
```

---

# Detect Key Phrases

```python
response = comprehend.detect_key_phrases(
    Text="Cloud computing enables scalable applications.",
    LanguageCode="en"
)

print(response["KeyPhrases"])
```

---

# Detect Language

```python
response = comprehend.detect_dominant_language(
    Text="Bonjour tout le monde"
)

print(response["Languages"])
```

---

# Amazon Translate

---

# Import Client

```python
translate = boto3.client("translate")
```

---

# Translate Text

```python
response = translate.translate_text(
    Text="Hello World",
    SourceLanguageCode="en",
    TargetLanguageCode="es"
)

print(response["TranslatedText"])
```

---

# Amazon Polly

---

# Import Client

```python
polly = boto3.client("polly")
```

---

# Convert Text to Speech

```python
response = polly.synthesize_speech(
    Text="Welcome to AWS",
    OutputFormat="mp3",
    VoiceId="Joanna"
)

audio = response["AudioStream"].read()
```

---

# List Voices

```python
response = polly.describe_voices()

for voice in response["Voices"]:
    print(voice["Name"])
```

---

# Amazon Transcribe

---

# Import Client

```python
transcribe = boto3.client("transcribe")
```

---

# Start Transcription Job

```python
response = transcribe.start_transcription_job(
    TranscriptionJobName="meeting-audio",
    LanguageCode="en-US",
    Media={
        "MediaFileUri": "s3://media/audio.mp3"
    }
)

print(response["TranscriptionJob"])
```

---

# Get Transcription Job

```python
response = transcribe.get_transcription_job(
    TranscriptionJobName="meeting-audio"
)

print(response["TranscriptionJob"]["TranscriptionJobStatus"])
```

---

# List Transcription Jobs

```python
response = transcribe.list_transcription_jobs()

print(response["TranscriptionJobSummaries"])
```

---

# Amazon Personalize

---

# Import Client

```python
personalize = boto3.client("personalize")
```

---

# List Datasets

```python
response = personalize.list_datasets()

print(response["datasets"])
```

---

# List Solutions

```python
response = personalize.list_solutions()

print(response["solutions"])
```

---

# Describe Solution

```python
response = personalize.describe_solution(
    solutionArn="arn:aws:personalize:..."
)

print(response["solution"])
```

---

# Amazon Forecast

---

# Import Client

```python
forecast = boto3.client("forecast")
```

---

# List Predictors

```python
response = forecast.list_predictors()

print(response["Predictors"])
```

---

# Describe Predictor

```python
response = forecast.describe_predictor(
    PredictorArn="arn:aws:forecast:..."
)

print(response["PredictorName"])
```

---

# Delete Predictor

```python
forecast.delete_predictor(
    PredictorArn="arn:aws:forecast:..."
)
```

---

# Pagination Example

```python
paginator = rekognition.get_paginator(
    "detect_labels"
)
```

---

# Exception Handling

```python
from botocore.exceptions import ClientError

try:
    response = bedrock.list_foundation_models()

except ClientError as error:
    print(error.response["Error"]["Code"])
    print(error.response["Error"]["Message"])
```

---

# Best Practices

- Keep prompts and model inputs free of sensitive information.
- Store large documents in Amazon S3 before processing with Textract.
- Validate confidence scores returned by Rekognition before taking automated actions.
- Use Comprehend language detection for multilingual applications.
- Cache frequently requested AI responses when appropriate.
- Use asynchronous APIs for long-running document processing jobs.
- Handle API throttling with retries and exponential backoff.
- Secure AI service access using IAM roles and least-privilege permissions.

---

# Summary

This section covered Boto3 automation for Amazon Bedrock, Bedrock Runtime, Amazon Q Developer, Textract, Rekognition, Comprehend, Translate, Polly, Transcribe, Personalize, and Forecast. These examples demonstrate practical AI/ML automation patterns for generative AI, document processing, computer vision, speech, translation, and natural language processing using Python.

---

