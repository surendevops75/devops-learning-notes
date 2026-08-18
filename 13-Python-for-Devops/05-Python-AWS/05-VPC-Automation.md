# VPC-Automation

## Python for AWS DevOps — VPC Discovery, Subnets, Route Tables, Security Groups, Gateways, Endpoints & Production Network Automation

Amazon VPC is the networking foundation for most AWS DevOps environments.

Python/Boto3 can automate:

```text
VPC inventory
subnet discovery
route-table audits
security-group audits
internet/NAT gateway discovery
ENI inventory
VPC endpoint audits
DNS configuration checks
CIDR analysis
network tagging
environment reports
connectivity diagnostics
multi-account network inventory
```

The goal is not to replace Terraform for infrastructure provisioning.

A strong production design is usually:

```text
Terraform
   ↓
provision VPC infrastructure

Python/Boto3
   ↓
discover
audit
validate
report
diagnose
operate
```

---

# 1. VPC Mental Model

A typical AWS network:

```text
AWS Account
    |
    +-- VPC
          |
          +-- Public Subnet
          |      |
          |      +-- Internet Gateway
          |      +-- ALB
          |
          +-- Private Subnet
          |      |
          |      +-- EKS nodes
          |      +-- EC2
          |
          +-- Database Subnet
                 |
                 +-- RDS
```

Traffic is controlled by:

```text
route tables
security groups
network ACLs
gateways
endpoints
DNS
```

---

# 2. Boto3 EC2 Client

VPC APIs are available through the EC2 client:

```python
import boto3

ec2 = boto3.client(
    "ec2",
    region_name="ap-south-1",
)
```

Although the service is called EC2, the client exposes many VPC APIs.

---

# 3. Validate AWS Identity

Before network changes:

```python
sts = boto3.client("sts")

identity = sts.get_caller_identity()

print(
    identity["Account"]
)
print(
    identity["Arn"]
)
```

For production automation, validate the expected account before making changes.

---

# 4. List VPCs

```python
response = ec2.describe_vpcs()

for vpc in response.get(
    "Vpcs",
    []
):
    print(
        vpc["VpcId"],
        vpc["CidrBlock"]
    )
```

---

# 5. VPC Pagination

Use paginators for inventory APIs when supported:

```python
paginator = ec2.get_paginator(
    "describe_vpcs"
)

for page in paginator.paginate():

    for vpc in page.get(
        "Vpcs",
        []
    ):
        print(
            vpc["VpcId"]
        )
```

---

# 6. VPC Metadata

Useful fields include:

```text
VpcId
CidrBlock
State
IsDefault
DhcpOptionsId
Tags
```

---

# 7. Identify Default VPC

```python
response = ec2.describe_vpcs(
    Filters=[
        {
            "Name": "isDefault",
            "Values": ["true"],
        }
    ]
)
```

Do not automatically delete a default VPC just because it is unused.

Other workloads or account defaults may depend on it.

---

# 8. VPC Tags

```python
def get_tag(resource, key):
    for tag in resource.get(
        "Tags",
        []
    ):
        if tag.get("Key") == key:
            return tag.get("Value")

    return None
```

Example:

```python
environment = get_tag(
    vpc,
    "Environment"
)
```

---

# 9. Tagging a VPC

```python
ec2.create_tags(
    Resources=[vpc_id],
    Tags=[
        {
            "Key": "Environment",
            "Value": "dev",
        },
        {
            "Key": "ManagedBy",
            "Value": "Terraform",
        },
    ],
)
```

Do not overwrite authoritative Terraform-managed tags without understanding ownership.

---

# 10. VPC CIDR

Example:

```text
10.0.0.0/16
```

This provides:

```text
65,536 IPv4 addresses
```

approximately, before AWS-specific subnet/address reservations and architecture constraints.

---

# 11. CIDR Planning

A typical layout:

```text
VPC: 10.0.0.0/16

Public:
10.0.1.0/24
10.0.2.0/24

Private:
10.0.11.0/24
10.0.12.0/24

Database:
10.0.21.0/24
10.0.22.0/24
```

Actual sizing should be based on expected growth.

---

# 12. Python CIDR Analysis

Use the standard library:

```python
import ipaddress

network = ipaddress.ip_network(
    "10.0.0.0/16"
)

print(
    network.num_addresses
)
```

This is useful for validation before provisioning.

---

# 13. Validate CIDR

```python
network = ipaddress.ip_network(
    cidr,
    strict=False,
)

print(network)
```

Use this to normalize and validate user-provided CIDRs.

---

# 14. Detect Overlapping CIDRs

```python
net1 = ipaddress.ip_network(
    "10.0.0.0/16"
)

net2 = ipaddress.ip_network(
    "10.0.1.0/24"
)

print(
    net1.overlaps(net2)
)
```

This is useful for VPC/subnet planning and peering designs.

---

# 15. List Subnets

```python
paginator = ec2.get_paginator(
    "describe_subnets"
)

for page in paginator.paginate():

    for subnet in page.get(
        "Subnets",
        []
    ):
        print(
            subnet["SubnetId"],
            subnet["VpcId"],
            subnet["CidrBlock"],
        )
```

---

# 16. Filter Subnets by VPC

```python
response = ec2.describe_subnets(
    Filters=[
        {
            "Name": "vpc-id",
            "Values": [vpc_id],
        }
    ]
)
```

---

# 17. Subnet Metadata

Important fields:

```text
SubnetId
VpcId
AvailabilityZone
CidrBlock
State
AvailableIpAddressCount
MapPublicIpOnLaunch
Tags
```

---

# 18. Public vs Private Subnet

A subnet is not inherently "public" just because of its name.

The important relationship is:

```text
subnet
 ↓
route table
 ↓
Internet Gateway
```

A subnet with a route to an Internet Gateway can be public, assuming other controls permit Internet connectivity.

---

# 19. Do Not Trust Subnet Names

This is unreliable:

```text
subnet-public-1
```

The name alone does not prove public routing.

Always inspect:

```text
route table
routes
gateway target
```

---

# 20. Map Public IP on Launch

Check:

```python
subnet.get(
    "MapPublicIpOnLaunch"
)
```

This can be useful for identifying expected public-subnet behavior, but it does not alone determine Internet reachability.

---

# 21. Availability Zones

Production VPCs should normally span multiple Availability Zones for high availability.

Example:

```text
AZ-a
AZ-b
AZ-c
```

Python can validate subnet distribution.

---

# 22. AZ Distribution Audit

Build:

```text
subnet count per AZ
CIDR per AZ
subnet type per AZ
```

Example:

```text
Public:
ap-south-1a → 1
ap-south-1b → 1
ap-south-1c → 1
```

---

# 23. Subnet Capacity

```python
for subnet in subnets:
    print(
        subnet["SubnetId"],
        subnet.get(
            "AvailableIpAddressCount"
        )
    )
```

This can support capacity alerts.

---

# 24. Low IP Capacity

A production audit can flag:

```text
available IPs < threshold
```

But choose thresholds based on workload behavior.

For EKS, consider:

```text
pods
nodes
ENIs
prefix delegation
CNI configuration
```

---

# 25. List Route Tables

```python
paginator = ec2.get_paginator(
    "describe_route_tables"
)

for page in paginator.paginate():

    for rt in page.get(
        "RouteTables",
        []
    ):
        print(
            rt["RouteTableId"],
            rt["VpcId"]
        )
```

---

# 26. Route Table Associations

A route table can have subnet associations.

Inspect:

```python
rt.get(
    "Associations",
    []
)
```

This lets you determine which subnet uses which route table.

---

# 27. Main Route Table

A VPC has a main route table.

Subnets without an explicit association use the VPC's main route table.

A network audit should therefore account for:

```text
explicit association
+
main route table
```

---

# 28. Route Inspection

```python
for route in rt.get(
    "Routes",
    []
):
    print(
        route.get(
            "DestinationCidrBlock"
        ),
        route.get(
            "GatewayId"
        ),
        route.get(
            "NatGatewayId"
        )
    )
```

---

# 29. Internet Gateway

List Internet Gateways:

```python
response = ec2.describe_internet_gateways()

for igw in response.get(
    "InternetGateways",
    []
):
    print(
        igw["InternetGatewayId"]
    )
```

---

# 30. Internet Gateway Attachments

Check:

```python
igw.get(
    "Attachments",
    []
)
```

This identifies the VPC attached to the Internet Gateway.

---

# 31. Public Route Detection

Look for:

```text
Destination: 0.0.0.0/0
Target: Internet Gateway
```

Conceptually:

```text
0.0.0.0/0
    ↓
igw-xxxx
```

This is a key signal for public subnet routing.

---

# 32. IPv6 Public Routing

Also consider:

```text
::/0
```

A network audit that checks only IPv4 can miss IPv6 Internet routing.

---

# 33. NAT Gateway

List NAT gateways:

```python
paginator = ec2.get_paginator(
    "describe_nat_gateways"
)

for page in paginator.paginate(
    Filters=[
        {
            "Name": "state",
            "Values": [
                "available",
                "pending",
            ],
        }
    ]
):

    for nat in page.get(
        "NatGateways",
        []
    ):
        print(
            nat["NatGatewayId"],
            nat["SubnetId"]
        )
```

---

# 34. NAT Gateway Architecture

Typical design:

```text
Private Subnet
      |
      v
Private Route Table
      |
      v
NAT Gateway
      |
      v
Internet Gateway
      |
      v
Internet
```

NAT Gateway allows outbound Internet connectivity without assigning public IPv4 addresses to private resources.

---

# 35. NAT Gateway High Availability

For production:

```text
AZ-a private subnets
       ↓
NAT Gateway in AZ-a

AZ-b private subnets
       ↓
NAT Gateway in AZ-b
```

This reduces cross-AZ dependency and improves resilience.

---

# 36. NAT Gateway Cost

NAT Gateways have hourly and data-processing costs.

An automation audit can identify:

```text
NAT count
NAT AZ
private routes
traffic/cost signals
```

Do not delete NAT gateways solely because they appear expensive; validate dependencies first.

---

# 37. NAT Gateway State

Possible states include:

```text
pending
available
deleting
deleted
failed
```

Automation should handle state transitions carefully.

---

# 38. Elastic IPs

NAT Gateways commonly use Elastic IP addresses.

List addresses:

```python
response = ec2.describe_addresses()

for address in response.get(
    "Addresses",
    []
):
    print(
        address.get(
            "AllocationId"
        ),
        address.get(
            "PublicIp"
        )
    )
```

---

# 39. Unassociated Elastic IP Audit

Identify addresses without a resource association.

These may represent:

```text
unused allocation
```

But verify ownership and intended future use before releasing.

---

# 40. Release Elastic IP

```python
ec2.release_address(
    AllocationId=allocation_id
)
```

This is destructive.

Use:

```text
account guard
resource validation
dry-run
approval
```

before release automation.

---

# 41. Security Groups

Security groups are stateful virtual firewalls.

List them:

```python
paginator = ec2.get_paginator(
    "describe_security_groups"
)

for page in paginator.paginate():

    for sg in page.get(
        "SecurityGroups",
        []
    ):
        print(
            sg["GroupId"],
            sg["GroupName"]
        )
```

---

# 42. Security Group Inbound Rules

Inspect:

```python
for permission in sg.get(
    "IpPermissions",
    []
):
    print(
        permission.get(
            "IpProtocol"
        ),
        permission.get(
            "FromPort"
        ),
        permission.get(
            "ToPort"
        ),
    )
```

---

# 43. Security Group CIDR Rules

Look for:

```text
0.0.0.0/0
```

Example:

```text
TCP 22 → 0.0.0.0/0
```

This is a high-priority finding in many environments.

It should not automatically be treated as a vulnerability for every port/use case, but SSH/RDP exposure deserves particular attention.

---

# 44. IPv6 Security Rules

Also inspect:

```text
::/0
```

An audit that checks only IPv4 can miss broad IPv6 exposure.

---

# 45. Security Group Rule Audit

A report can classify:

```text
SSH public
RDP public
database public
HTTP public
HTTPS public
wide custom ports
```

---

# 46. Security Group Best Practice

Prefer:

```text
ALB SG
   ↓
application SG
   ↓
database SG
```

rather than:

```text
0.0.0.0/0
```

for every tier.

---

# 47. Security Group Referencing

Rules can reference another security group.

Concept:

```text
ALB-SG
   ↓
App-SG
```

This is generally better than hardcoding changing IP addresses for internal application tiers.

---

# 48. Security Group Rule Analysis

Inspect:

```python
for pair in permission.get(
    "UserIdGroupPairs",
    []
):
    print(
        pair.get(
            "GroupId"
        )
    )
```

---

# 49. Egress Rules

Do not audit only ingress.

Inspect:

```python
sg.get(
    "IpPermissionsEgress",
    []
)
```

Review broad egress according to organizational requirements.

---

# 50. Security Group Dependencies

Before deleting an SG, find:

```text
ENIs
EC2
ALB
RDS
EKS
Lambda
other resources
```

Never delete a security group solely because it appears unused by name.

---

# 51. Describe Network Interfaces

```python
paginator = ec2.get_paginator(
    "describe_network_interfaces"
)

for page in paginator.paginate():

    for eni in page.get(
        "NetworkInterfaces",
        []
    ):
        print(
            eni["NetworkInterfaceId"],
            eni.get(
                "PrivateIpAddress"
            )
        )
```

---

# 52. ENI Metadata

Useful fields:

```text
NetworkInterfaceId
SubnetId
VpcId
PrivateIpAddress
Groups
Description
InterfaceType
Status
Attachment
RequesterId
```

---

# 53. Why ENI Inventory Matters

ENIs can belong to:

```text
EC2
ALB
NAT
EKS
Lambda
VPC endpoints
RDS
other AWS services
```

They help diagnose network architecture and unexpected dependencies.

---

# 54. Find ENIs in a Subnet

```python
response = ec2.describe_network_interfaces(
    Filters=[
        {
            "Name": "subnet-id",
            "Values": [subnet_id],
        }
    ]
)
```

Useful during:

```text
subnet cleanup
IP exhaustion
security-group investigation
```

---

# 55. Find ENIs Using a Security Group

```python
response = ec2.describe_network_interfaces(
    Filters=[
        {
            "Name": "group-id",
            "Values": [sg_id],
        }
    ]
)
```

This helps determine whether an SG can safely be changed or removed.

---

# 56. VPC Endpoints

VPC endpoints provide private access to AWS services.

Types commonly encountered:

```text
Gateway endpoint
Interface endpoint
```

---

# 57. List VPC Endpoints

```python
paginator = ec2.get_paginator(
    "describe_vpc_endpoints"
)

for page in paginator.paginate():

    for endpoint in page.get(
        "VpcEndpoints",
        []
    ):
        print(
            endpoint["VpcEndpointId"],
            endpoint["VpcEndpointType"]
        )
```

---

# 58. Gateway Endpoint

Common example:

```text
S3
DynamoDB
```

Traffic can remain within AWS networking rather than requiring NAT for supported access patterns.

---

# 59. Interface Endpoint

Interface endpoints use:

```text
ENIs
private IPs
security groups
```

and can provide private connectivity to supported AWS services.

---

# 60. VPC Endpoint Security Groups

For interface endpoints, inspect:

```text
security groups
subnets
private DNS
```

Incorrect SG rules can cause application connectivity failures.

---

# 61. VPC Endpoint Audit

Check:

```text
endpoint type
service name
VPC
subnets
route tables
security groups
private DNS
policy
state
```

---

# 62. VPC Endpoint Policy

Where supported, endpoint policies can restrict access to resources/services.

Python can retrieve endpoint configuration for auditing.

Do not assume network reachability automatically means unrestricted service access.

---

# 63. VPC DNS

Check VPC DNS attributes:

```python
response = ec2.describe_vpc_attribute(
    VpcId=vpc_id,
    Attribute="enableDnsSupport",
)

print(
    response["EnableDnsSupport"][
        "Value"
    ]
)
```

And:

```python
response = ec2.describe_vpc_attribute(
    VpcId=vpc_id,
    Attribute="enableDnsHostnames",
)
```

---

# 64. Why DNS Matters

Applications often fail with symptoms like:

```text
connection timeout
name resolution failure
cannot resolve endpoint
```

VPC DNS configuration can be part of the investigation.

---

# 65. DHCP Options

VPC DHCP options influence network configuration such as:

```text
domain name
DNS servers
```

Inspect:

```python
response = ec2.describe_dhcp_options(
    DhcpOptionsIds=[
        dhcp_options_id
    ]
)
```

---

# 66. Route Audit Project

Build:

```bash
python vpcops.py routes
```

Report:

```text
VPC
route table
subnet
destination
target
```

Example:

```text
private-a
0.0.0.0/0
nat-1234

public-a
0.0.0.0/0
igw-1234
```

---

# 67. Detect Misclassified Private Subnet

A custom audit can flag:

```text
subnet tagged private
+
route to Internet Gateway
```

for review.

Tags are intent; routing is actual behavior.

---

# 68. Detect Missing NAT Route

For workloads expected to access the Internet:

```text
private subnet
 ↓
route table
 ↓
0.0.0.0/0
 ↓
NAT Gateway
```

Missing route can explain:

```text
yum/dnf failure
pip/npm download failure
Docker image pull failure
external API failure
```

---

# 69. EKS Network Troubleshooting

For EKS, inspect:

```text
VPC CIDR
subnets
route tables
NAT
security groups
ENIs
VPC endpoints
DNS
```

Python can automate the discovery portion of this investigation.

---

# 70. EKS Private Subnet Audit

A useful check:

```text
EKS node subnet
 ↓
route table
 ↓
NAT / required endpoint
```

Then verify required AWS service connectivity.

---

# 71. ECR Pull Troubleshooting

If private EKS nodes cannot pull images, investigate:

```text
NAT connectivity
ECR endpoints
S3 endpoint/connectivity
security groups
DNS
route tables
IAM
```

The issue is not always Kubernetes.

---

# 72. S3 Access From Private Subnets

Options can include:

```text
NAT Gateway
```

or:

```text
S3 Gateway VPC Endpoint
```

depending on architecture.

Python can audit whether expected endpoints exist.

---

# 73. Network Inventory

Build a normalized structure:

```python
{
    "vpc": "vpc-123",
    "cidr": "10.0.0.0/16",
    "subnets": [],
    "route_tables": [],
    "security_groups": [],
    "nat_gateways": [],
    "internet_gateways": [],
    "endpoints": [],
}
```

This becomes useful for reporting.

---

# 74. VPC Inventory Architecture

```text
STS
 ↓
Account validation
 ↓
VPCs
 ↓
Subnets
 ↓
Route tables
 ↓
Gateways
 ↓
Security groups
 ↓
ENIs
 ↓
Endpoints
 ↓
Normalize
 ↓
Report
```

---

# 75. Multi-Region VPC Inventory

Loop over approved regions:

```python
regions = [
    "ap-south-1",
    "ap-southeast-1",
]

for region in regions:

    ec2 = boto3.client(
        "ec2",
        region_name=region,
    )

    # inventory
```

Use an allowlist rather than blindly scanning every possible region.

---

# 76. Region Validation

For each region:

```text
create client
 ↓
collect account identity if needed
 ↓
inventory VPC
 ↓
record region
```

IAM is global, while VPC resources are regional.

---

# 77. Multi-Account Network Inventory

```text
Central automation
       |
       +-- Dev role
       |
       +-- Staging role
       |
       +-- Production role
       |
       +-- Security role
```

For every target:

```text
AssumeRole
 ↓
GetCallerIdentity
 ↓
validate account
 ↓
inventory region
```

---

# 78. Network Naming Standards

Example:

```text
vpc-prod-ap-south-1
subnet-prod-public-a
subnet-prod-private-a
rt-prod-private-a
sg-prod-alb
sg-prod-app
sg-prod-db
```

Naming should support operations and ownership.

---

# 79. VPC Tags

Useful tags:

```text
Environment
Project
Owner
CostCenter
ManagedBy
NetworkTier
```

Python can audit missing mandatory tags.

---

# 80. Mandatory Tag Audit

```python
required = [
    "Environment",
    "Owner",
    "ManagedBy",
]

missing = []

for key in required:

    if not get_tag(
        resource,
        key
    ):
        missing.append(key)
```

---

# 81. VPC Compliance Report

Example:

```text
VPC: vpc-prod

CIDR: PASS
AZ coverage: PASS
DNS: PASS
Public subnet routing: PASS
Private NAT routing: PASS
Security groups: REVIEW
Tags: PASS
Endpoints: REVIEW
```

---

# 82. Security Group Compliance Report

Example:

```text
SG: sg-app

Public SSH: FAIL
Public RDP: PASS
Database public access: PASS
Wide custom port: REVIEW
Egress: REVIEW
```

---

# 83. Network Health Report

Useful fields:

```text
VPC count
subnet count
AZ count
NAT count
IGW count
endpoint count
low-IP subnets
public SG findings
unassociated EIPs
```

---

# 84. VPC Connectivity Diagnostics

When an application cannot connect:

```text
source subnet
 ↓
route table
 ↓
destination
 ↓
security group
 ↓
NACL
 ↓
gateway/endpoint
 ↓
DNS
```

Python can automate evidence collection.

---

# 85. Security Group vs NACL

### Security Group

```text
stateful
resource-level
allow rules
```

### Network ACL

```text
subnet-level
stateless
allow + deny
ordered rules
```

Do not troubleshoot one while ignoring the other.

---

# 86. List Network ACLs

```python
paginator = ec2.get_paginator(
    "describe_network_acls"
)

for page in paginator.paginate():

    for acl in page.get(
        "NetworkAcls",
        []
    ):
        print(
            acl["NetworkAclId"]
        )
```

---

# 87. NACL Rules

Inspect:

```python
for entry in acl.get(
    "Entries",
    []
):
    print(
        entry.get("RuleNumber"),
        entry.get("Protocol"),
        entry.get("RuleAction"),
        entry.get("CidrBlock"),
    )
```

Also inspect IPv6 CIDRs where present.

---

# 88. NACL Troubleshooting

Because NACLs are stateless:

```text
inbound
+
outbound
```

must both permit the traffic.

This is different from security groups.

---

# 89. VPC Flow Logs

VPC Flow Logs can help investigate:

```text
ACCEPT
REJECT
source
destination
ports
protocol
```

Python can audit whether required flow logging exists, while analysis can be performed through the configured log destination.

---

# 90. Flow Log Inventory

List:

```python
response = ec2.describe_flow_logs()

for flow_log in response.get(
    "FlowLogs",
    []
):
    print(
        flow_log.get(
            "FlowLogId"
        ),
        flow_log.get(
            "ResourceId"
        )
    )
```

---

# 91. Flow Logs and ELK

Architecture:

```text
VPC Flow Logs
      ↓
CloudWatch Logs / S3
      ↓
log pipeline
      ↓
ELK
      ↓
Kibana
```

Python can automate configuration checks and reporting.

---

# 92. VPC Health Automation

Build:

```bash
python vpcops.py health
```

Checks:

```text
DNS
subnets
routes
NAT
IGW
security groups
NACLs
endpoints
flow logs
```

---

# 93. Dry-Run Network Changes

For any mutation:

```bash
python vpcops.py cleanup --dry-run
```

Example:

```text
Would release:
eipalloc-1234

Would remove:
unused SG rule

Would delete:
test subnet
```

Never make destructive networking changes without validation.

---

# 94. VPC Deletion Dependencies

Before deleting a VPC, consider:

```text
subnets
route tables
Internet Gateway
NAT Gateways
security groups
ENIs
VPC endpoints
network ACLs
DHCP options
load balancers
RDS
EKS
```

The dependency graph can be complex.

---

# 95. VPC Cleanup Workflow

```text
identify candidate
 ↓
validate environment
 ↓
check Terraform state
 ↓
check dependencies
 ↓
dry-run
 ↓
approval
 ↓
delete in dependency order
 ↓
verify
```

Terraform should generally own infrastructure destruction when the resources are Terraform-managed.

---

# 96. Do Not Mix IaC Ownership

Avoid:

```text
Terraform creates route
+
Python deletes route
```

This can create drift.

Better:

```text
Terraform → desired infrastructure
Python → audit/operations
```

unless a specific operational change is intentionally outside IaC ownership.

---

# 97. VPC + Terraform

Terraform commonly manages:

```text
VPC
subnets
route tables
IGW
NAT
security groups
endpoints
```

Python can validate:

```text
actual state
required state
```

---

# 98. VPC + Jenkins

```text
Jenkins
 ↓
Python audit
 ↓
VPC inventory
 ↓
report
 ↓
notification
```

For changes:

```text
Jenkins
 ↓
Terraform plan
 ↓
approval
 ↓
Terraform apply
```

Python can provide supporting diagnostics.

---

# 99. VPC + GitHub Actions

```text
GitHub Actions
 ↓
OIDC
 ↓
IAM Role
 ↓
Python
 ↓
network audit
```

Use read-only permissions for audits whenever possible.

---

# 100. VPC + EKS

A production EKS audit should correlate:

```text
EKS cluster
 ↓
VPC
 ↓
subnets
 ↓
route tables
 ↓
NAT/endpoints
 ↓
security groups
 ↓
ENIs
```

This makes troubleshooting much faster.

---

# 101. VPC + RDS

RDS networking depends on:

```text
DB subnet group
subnets
route behavior
security groups
DNS
```

Python can inventory these relationships.

---

# 102. VPC + ALB

ALB networking involves:

```text
subnets
security groups
ENIs
route behavior
target connectivity
```

An ALB that exists does not guarantee application reachability.

---

# 103. Network Troubleshooting Example

Problem:

```text
Application in private subnet
cannot reach external API
```

Check:

```text
1. subnet
2. route table
3. default route
4. NAT gateway
5. NAT state
6. IGW
7. security group egress
8. NACL
9. DNS
```

Python can collect the relevant configuration automatically.

---

# 104. Network Troubleshooting Example

Problem:

```text
EKS pods cannot pull image
```

Check:

```text
node subnet
route table
NAT
ECR connectivity
S3 connectivity
VPC endpoints
DNS
security groups
IAM
```

---

# 105. Network Troubleshooting Example

Problem:

```text
RDS connection timeout
```

Check:

```text
client subnet
route table
RDS subnet
RDS SG
client SG
NACL
DNS
port
```

---

# 106. Network Troubleshooting Example

Problem:

```text
ALB returns 502
```

Network investigation can include:

```text
ALB subnet
ALB SG
target subnet
target SG
target port
route
NACL
target health
```

The 502 itself is an application/load-balancer symptom, so also inspect target logs and health checks.

---

# 107. VPC CIDR Overlap Project

For multiple networks:

```python
networks = [
    ipaddress.ip_network(
        "10.0.0.0/16"
    ),
    ipaddress.ip_network(
        "10.1.0.0/16"
    ),
]
```

Compare:

```python
for i, left in enumerate(
    networks
):
    for right in networks[
        i + 1:
    ]:
        if left.overlaps(right):
            print(
                "Overlap detected"
            )
```

Useful before:

```text
VPC peering
Transit Gateway
VPN
hybrid connectivity
```

---

# 108. Transit Gateway Awareness

In larger AWS environments:

```text
VPC A
   \
VPC B → Transit Gateway
   /
VPC C
```

A VPC audit should consider Transit Gateway routes when troubleshooting cross-VPC connectivity.

---

# 109. VPC Peering Awareness

Peering creates:

```text
VPC A
  ↕
VPC B
```

Both sides need appropriate routes.

Security controls must also permit traffic.

---

# 110. Route Troubleshooting Principle

A route existing in one route table does not guarantee connectivity.

You need:

```text
source route
+
destination route
+
security rules
+
NACL
+
gateway/service configuration
```

---

# 111. VPC Endpoint vs NAT

For supported AWS service traffic:

```text
NAT
```

may be replaced or supplemented by:

```text
VPC Endpoint
```

Potential benefits:

```text
private path
reduced NAT dependency
security control
architecture simplification
```

Evaluate cost and service support before changing architecture.

---

# 112. VPC Endpoint Audit Project

Build:

```bash
python vpcops.py endpoints
```

Report:

```text
VPC
endpoint
service
type
state
subnets
security groups
private DNS
```

---

# 113. Unused NAT Audit

Potential report:

```text
NAT Gateway
VPC
AZ
private route count
```

Do not declare a NAT unused merely because the route table is not obvious; inspect all relevant route tables and dependencies.

---

# 114. Unused Security Group Audit

Potential candidates:

```text
SG
no ENI references
no resource references
```

Still verify service dependencies before deletion.

---

# 115. Unused EIP Audit

Potential candidates:

```text
EIP
no association
```

This is a simpler audit, but still confirm ownership and intended use before release.

---

# 116. Low Subnet IP Audit

```text
subnet
 ↓
available IP count
 ↓
threshold
 ↓
warning
```

For EKS, use workload-aware thresholds.

---

# 117. VPC Compliance Score

Example:

```text
VPC Security Score

DNS:                  PASS
Flow Logs:            PASS
Public SG rules:      FAIL
NAT HA:               WARN
Endpoint coverage:    PASS
Mandatory tags:       PASS
```

Do not hide individual findings behind a single score.

---

# 118. Structured JSON Output

```python
import json

print(
    json.dumps(
        report,
        indent=2,
        default=str,
    )
)
```

This makes the output easy to consume from:

```text
Jenkins
GitHub Actions
ELK
other automation
```

---

# 119. CSV Reporting

Python can produce:

```text
vpc_report.csv
```

Columns:

```text
account
region
vpc
cidr
subnet
az
route_table
nat
endpoint
```

Useful for network reviews.

---

# 120. VPC CLI Design

Example:

```bash
python vpcops.py vpcs
```

```bash
python vpcops.py subnets
```

```bash
python vpcops.py routes
```

```bash
python vpcops.py security-groups
```

```bash
python vpcops.py endpoints
```

```bash
python vpcops.py audit
```

```bash
python vpcops.py health
```

---

# 121. VPC CLI Arguments

Useful:

```text
--profile
--region
--vpc-id
--environment
--output
--format
--dry-run
```

For mutations:

```text
--confirm
```

can provide an additional explicit guard.

---

# 122. Production VPC Audit Pattern

```text
GetCallerIdentity
        ↓
Validate account
        ↓
Select region
        ↓
Discover VPC
        ↓
Discover subnets
        ↓
Map route tables
        ↓
Map gateways
        ↓
Audit SG/NACL
        ↓
Audit endpoints
        ↓
Audit DNS/flow logs
        ↓
Generate report
```

---

# 123. Error Handling

```python
from botocore.exceptions import ClientError

try:
    response = ec2.describe_vpcs()

except ClientError as exc:

    error = exc.response.get(
        "Error",
        {}
    )

    print(
        error.get("Code"),
        error.get("Message")
    )

    raise
```

---

# 124. Common VPC Errors

Examples:

```text
UnauthorizedOperation
InvalidVpcID.NotFound
InvalidSubnetID.NotFound
DependencyViolation
InvalidRouteTableID.NotFound
InvalidGroup.NotFound
```

---

# 125. DependencyViolation

This often means another AWS resource depends on the resource being deleted.

Investigate:

```text
ENIs
subnets
gateways
routes
load balancers
endpoints
```

before retrying deletion.

---

# 126. UnauthorizedOperation

Check:

```text
IAM policy
resource scope
region
account
SCP
permissions boundary
```

For some EC2 API errors, encoded authorization messages can provide additional diagnostic information when the caller has permission to decode them.

---

# 127. Retry Configuration

```python
from botocore.config import Config

config = Config(
    retries={
        "max_attempts": 5,
        "mode": "standard",
    }
)

ec2 = boto3.client(
    "ec2",
    config=config,
)
```

Do not retry permanent validation errors indefinitely.

---

# 128. VPC Automation Testing

Unit-test:

```text
CIDR overlap
tag extraction
route classification
public-subnet detection
SG risk classification
report generation
account guard
```

---

# 129. VPC Stub Testing

```python
from botocore.stub import Stubber

stubber = Stubber(ec2)

stubber.add_response(
    "describe_vpcs",
    {
        "Vpcs": [
            {
                "VpcId": "vpc-test",
                "CidrBlock": "10.0.0.0/16",
            }
        ]
    },
)

stubber.activate()
```

---

# 130. VPC Integration Testing

Use:

```text
dedicated AWS account
test VPC
test subnets
test security groups
test routes
```

Do not run cleanup logic against production VPCs.

---

# 131. Network Automation Security Checklist

```text
[ ] Account validated
[ ] Region validated
[ ] Least privilege
[ ] Read-only role for audits
[ ] No hardcoded credentials
[ ] No unrestricted SG mutations
[ ] No blind route changes
[ ] No automatic EIP release without validation
[ ] No production VPC deletion
[ ] Dry-run for destructive operations
```

---

# 132. Network Reliability Checklist

```text
[ ] Multi-AZ design
[ ] NAT HA where required
[ ] Route tables reviewed
[ ] DNS reviewed
[ ] Endpoint availability
[ ] Subnet capacity
[ ] Flow logs where required
[ ] Security group dependencies
[ ] NACL review
```

---

# 133. Network Cost Checklist

```text
[ ] NAT Gateway count
[ ] NAT data processing
[ ] Cross-AZ traffic
[ ] Unused EIPs
[ ] Unnecessary endpoints
[ ] VPC architecture
```

Never optimize solely for cost without considering availability and traffic paths.

---

# 134. Interview — What Is a VPC?

**Answer:**

> A VPC is an isolated virtual network in AWS where I define CIDR ranges, subnets, routing, security controls and connectivity to AWS services or external networks.

---

# 135. Interview — How Do You Identify a Public Subnet?

**Answer:**

> I inspect the subnet's route table and look for a default route to an Internet Gateway. I do not rely only on subnet names or the MapPublicIpOnLaunch setting.

---

# 136. Interview — Public vs Private Subnet?

**Answer:**

> A public subnet has a route toward an Internet Gateway, while a private subnet typically has no direct Internet Gateway route. A private subnet may use a NAT Gateway for outbound Internet access.

---

# 137. Interview — How Do You Find Which Route Table a Subnet Uses?

**Answer:**

> I inspect route-table associations. If the subnet has no explicit association, I determine the VPC's main route table because that becomes the effective route table.

---

# 138. Interview — How Do You Troubleshoot Private Subnet Internet Access?

**Answer:**

> I check the subnet route table for a default route to a NAT Gateway, verify NAT state and its public-subnet path through the Internet Gateway, then inspect security groups, NACLs and DNS.

---

# 139. Interview — Why Use Multiple NAT Gateways?

**Answer:**

> For production high availability, deploying NAT Gateways across Availability Zones can reduce dependency on a single AZ and avoid unnecessary cross-AZ traffic paths.

---

# 140. Interview — What Is a Security Group?

**Answer:**

> A security group is a stateful virtual firewall associated with resources such as ENIs. It controls allowed inbound and outbound traffic.

---

# 141. Interview — Security Group vs NACL?

**Answer:**

> Security groups are stateful and operate at the resource/ENI level. NACLs operate at the subnet level and are stateless, so return traffic must also be explicitly permitted.

---

# 142. Interview — How Do You Audit Public Security Groups?

**Answer:**

> I inspect ingress and egress rules for broad CIDRs such as `0.0.0.0/0` and `::/0`, identify sensitive ports, correlate rules with resource dependencies and classify findings for remediation.

---

# 143. Interview — Should Every `0.0.0.0/0` Rule Be Removed?

**Answer:**

> No. Some services such as public HTTPS endpoints intentionally need Internet access. I classify the rule based on port, resource role, architecture and business requirements rather than blindly deleting it.

---

# 144. Interview — How Do You Find Security Group Dependencies?

**Answer:**

> I query network interfaces using the security-group ID and then correlate the ENIs with their owning resources. I also consider managed AWS services that may create ENIs.

---

# 145. Interview — What Is a NAT Gateway?

**Answer:**

> A NAT Gateway provides outbound connectivity from private subnets to external destinations without requiring public IP addresses on the private resources.

---

# 146. Interview — What Is a VPC Endpoint?

**Answer:**

> A VPC endpoint provides private connectivity to supported AWS services. Gateway endpoints and interface endpoints use different networking models and should be evaluated according to the target service and architecture.

---

# 147. Interview — How Do You Troubleshoot EKS Network Problems?

**Answer:**

> I start with the pod/node subnet, route table and NAT or endpoint path, then inspect security groups, NACLs, DNS, ENIs and AWS service connectivity. I also verify IAM because network connectivity and authorization are separate concerns.

---

# 148. Interview — How Do You Find CIDR Overlaps?

**Answer:**

> I use Python's `ipaddress` module and compare network objects with `overlaps()`. This is useful before peering, Transit Gateway, VPN or hybrid-network integration.

---

# 149. Interview — How Would You Automate VPC Inventory?

**Answer:**

> I validate the AWS account, enumerate approved regions, discover VPCs and then correlate subnets, route tables, gateways, security groups, ENIs, endpoints and tags into a normalized report.

---

# 150. Interview — Would You Use Python to Create the Entire VPC?

**Answer:**

> For repeatable infrastructure I prefer Terraform as the source of truth. Python/Boto3 is valuable for operational automation, discovery, auditing, diagnostics and workflows that require custom logic.

---

# 151. Interview — How Do You Prevent Network Automation From Breaking Production?

**Answer:**

> I validate account and region, restrict permissions, verify resource ownership, use dry-run and change review, avoid broad mutations, check dependencies and verify the resulting state.

---

# 152. Interview — How Do You Handle Multi-Account VPC Auditing?

**Answer:**

> I use a central automation role to assume tightly scoped roles in each target account, validate the account identity, enumerate approved regions and aggregate normalized network findings.

---

# 153. Interview — What Would You Monitor in a VPC?

**Answer:**

> I would monitor network-related symptoms such as subnet IP pressure, NAT availability, rejected traffic where flow logs are enabled, endpoint health, load-balancer connectivity and application-level network failures.

---

# 154. Interview — How Do You Investigate a Connection Timeout?

**Answer:**

> I follow the traffic path from source to destination: DNS, route table, gateway or endpoint, security group, NACL and destination listener/service. I verify both configuration and observed traffic where telemetry is available.

---

# 155. Interview — What Is the Biggest Mistake in Network Automation?

**Answer:**

> Making assumptions from names or partial configuration. A subnet called `private` does not prove it is private, and an existing route does not prove end-to-end connectivity. Automation should correlate the complete network path.

---

# 156. Final VPC Automation Mental Model

```text
Validate Account
       ↓
Validate Region
       ↓
Discover VPC
       ↓
Discover Subnets
       ↓
Map Route Tables
       ↓
Map IGW/NAT
       ↓
Audit SG/NACL
       ↓
Audit ENIs
       ↓
Audit Endpoints
       ↓
Check DNS/Flow Logs
       ↓
Correlate Dependencies
       ↓
Generate Report
       ↓
Notify
```

The key DevOps principle is:

> **Don't troubleshoot networking by guessing. Map the actual traffic path from source to destination.**

Next:

```text
06-RDS-Automation.md
```

will cover RDS discovery, database status, snapshots, backups, subnet groups, security groups, parameter groups, monitoring configuration, maintenance windows, storage, Multi-AZ, failover-aware automation and production-safe database operations.
