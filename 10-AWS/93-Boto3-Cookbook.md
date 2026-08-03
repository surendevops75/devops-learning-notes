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

