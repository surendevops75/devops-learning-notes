# AWS CloudFormation

---

# Introduction

AWS CloudFormation is an Infrastructure as Code (IaC) service that provisions and manages AWS resources using declarative templates.

Templates can be written in:

- YAML (Recommended)
- JSON

CloudFormation automatically manages:

- Resource creation
- Updates
- Rollbacks
- Dependency ordering
- Stack deletion

---

# Benefits

- Infrastructure as Code
- Repeatable deployments
- Version controlled infrastructure
- Automatic dependency resolution
- Rollback on failures
- Native AWS service

---

# CloudFormation Workflow

```text
Write Template
        ↓
Validate Template
        ↓
Create Stack
        ↓
Review Events
        ↓
Update Stack
        ↓
Delete Stack
```

---

# Template Structure

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: Sample Template

Metadata:

Parameters:

Mappings:

Conditions:

Resources:

Outputs:
```

---

# Minimal Template

```yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: Hello CloudFormation

Resources: {}
```

---

# Create Stack

```bash
aws cloudformation create-stack \
--stack-name production \
--template-body file://template.yaml
```

---

# Update Stack

```bash
aws cloudformation update-stack \
--stack-name production \
--template-body file://template.yaml
```

---

# Delete Stack

```bash
aws cloudformation delete-stack \
--stack-name production
```

---

# Validate Template

```bash
aws cloudformation validate-template \
--template-body file://template.yaml
```

---

# List Stacks

```bash
aws cloudformation list-stacks
```

---

# Describe Stack

```bash
aws cloudformation describe-stacks \
--stack-name production
```

---

# Describe Stack Events

```bash
aws cloudformation describe-stack-events \
--stack-name production
```

---

# List Stack Resources

```bash
aws cloudformation list-stack-resources \
--stack-name production
```

---

# Parameters

```yaml
Parameters:

  Environment:

    Type: String

    Default: dev
```

---

# Number Parameter

```yaml
Parameters:

  InstanceCount:

    Type: Number

    Default: 2
```

---

# CommaDelimitedList

```yaml
Parameters:

  Subnets:

    Type: CommaDelimitedList
```

---

# Allowed Values

```yaml
Parameters:

  Environment:

    Type: String

    AllowedValues:

      - dev

      - qa

      - prod
```

---

# Parameter Constraints

```yaml
Parameters:

  InstanceType:

    Type: String

    AllowedPattern: "^t3.*"
```

---

# NoEcho

```yaml
Parameters:

  DBPassword:

    Type: String

    NoEcho: true
```

---

# Metadata

```yaml
Metadata:

  AWS::CloudFormation::Interface:

    ParameterGroups:
      - Label:

          default: Environment
```

---

# Mappings

```yaml
Mappings:

  RegionMap:

    ap-south-1:

      AMI: ami-xxxxxxxx
```

---

# FindInMap

```yaml
ImageId:

  Fn::FindInMap:

    - RegionMap

    - !Ref AWS::Region

    - AMI
```

---

# Conditions

```yaml
Conditions:

  IsProduction:

    !Equals

      - !Ref Environment

      - prod
```

---

# Use Condition

```yaml
Condition: IsProduction
```

---

# Outputs

```yaml
Outputs:

  VpcId:

    Value: !Ref VPC
```

---

# Export Output

```yaml
Outputs:

  VpcId:

    Value: !Ref VPC

    Export:

      Name: Production-VPC
```

---

# Pseudo Parameters

```yaml
!Ref AWS::Region
```

---

```yaml
!Ref AWS::AccountId
```

---

```yaml
!Ref AWS::StackName
```

---

```yaml
!Ref AWS::Partition
```

---

```yaml
!Ref AWS::URLSuffix
```

---

# Resource Example

```yaml
Resources:

  VPC:

    Type: AWS::EC2::VPC

    Properties:

      CidrBlock: 10.0.0.0/16
```

---

# Resource Tags

```yaml
Tags:

  - Key: Name

    Value: Production
```

---

# DependsOn

```yaml
DependsOn:

  - InternetGateway
```

---

# Deletion Policy

```yaml
DeletionPolicy: Retain
```

---

# Update Replace Policy

```yaml
UpdateReplacePolicy: Retain
```

---

# Stack Policy

```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Principal": "*"
    }
  ]
}
```

---

# Wait Condition

```yaml
Type: AWS::CloudFormation::WaitCondition
```

---

# Wait Condition Handle

```yaml
Type: AWS::CloudFormation::WaitConditionHandle
```

---

# Resource Attributes

```yaml
!GetAtt VPC.CidrBlock
```

---

```yaml
!GetAtt Bucket.DomainName
```

---

# Resource Reference

```yaml
!Ref VPC
```

---

# Intrinsic Function

Join

```yaml
!Join

  - "-"

  - [dev, app]
```

---

Split

```yaml
!Split

  - ","

  - "a,b,c"
```

---

Select

```yaml
!Select

  - 0

  - !Ref Subnets
```

---

Sub

```yaml
!Sub

  arn:aws:s3:::${Bucket}
```

---

Base64

```yaml
!Base64 |
  #!/bin/bash
  yum update -y
```

---

# AWS-Specific Parameter

```yaml
Parameters:

  KeyPair:

    Type: AWS::EC2::KeyPair::KeyName
```

---

# VPC Parameter

```yaml
Type: AWS::EC2::VPC::Id
```

---

# Subnet Parameter

```yaml
Type: AWS::EC2::Subnet::Id
```

---

# Security Group Parameter

```yaml
Type: AWS::EC2::SecurityGroup::Id
```

---

# Best Practices

- Prefer YAML over JSON for readability.
- Use Parameters instead of hardcoded values.
- Export shared resources using Outputs.
- Use Mappings for region-specific values.
- Keep templates modular.
- Use Conditions to avoid duplicate templates.
- Validate templates before deployment.
- Tag all resources consistently.
- Protect critical resources using `DeletionPolicy: Retain`.

---

# Summary

This section covered CloudFormation fundamentals, template structure, parameters, mappings, conditions, outputs, intrinsic functions, pseudo parameters, stack operations, resource attributes, deletion policies, and best practices. These concepts form the foundation for building reusable and production-ready CloudFormation templates.

---

# Amazon VPC

---

# Create VPC

```yaml
Resources:

  VPC:

    Type: AWS::EC2::VPC

    Properties:

      CidrBlock: 10.0.0.0/16

      EnableDnsSupport: true

      EnableDnsHostnames: true

      Tags:

        - Key: Name

          Value: Production-VPC
```

---

# Internet Gateway

```yaml
Resources:

  InternetGateway:

    Type: AWS::EC2::InternetGateway
```

---

# Attach Internet Gateway

```yaml
Resources:

  GatewayAttachment:

    Type: AWS::EC2::VPCGatewayAttachment

    Properties:

      VpcId: !Ref VPC

      InternetGatewayId: !Ref InternetGateway
```

---

# Public Subnet

```yaml
Resources:

  PublicSubnet:

    Type: AWS::EC2::Subnet

    Properties:

      VpcId: !Ref VPC

      CidrBlock: 10.0.1.0/24

      AvailabilityZone: ap-south-1a

      MapPublicIpOnLaunch: true
```

---

# Private Subnet

```yaml
Resources:

  PrivateSubnet:

    Type: AWS::EC2::Subnet

    Properties:

      VpcId: !Ref VPC

      CidrBlock: 10.0.2.0/24

      AvailabilityZone: ap-south-1a
```

---

# Multiple Public Subnets

```yaml
Resources:

  PublicSubnet1:

    Type: AWS::EC2::Subnet

    Properties:

      VpcId: !Ref VPC

      CidrBlock: 10.0.1.0/24

      AvailabilityZone: ap-south-1a

      MapPublicIpOnLaunch: true

  PublicSubnet2:

    Type: AWS::EC2::Subnet

    Properties:

      VpcId: !Ref VPC

      CidrBlock: 10.0.2.0/24

      AvailabilityZone: ap-south-1b

      MapPublicIpOnLaunch: true
```

---

# Elastic IP

```yaml
Resources:

  NatEIP:

    Type: AWS::EC2::EIP

    Properties:

      Domain: vpc
```

---

# NAT Gateway

```yaml
Resources:

  NatGateway:

    Type: AWS::EC2::NatGateway

    Properties:

      AllocationId: !GetAtt NatEIP.AllocationId

      SubnetId: !Ref PublicSubnet
```

---

# Public Route Table

```yaml
Resources:

  PublicRouteTable:

    Type: AWS::EC2::RouteTable

    Properties:

      VpcId: !Ref VPC
```

---

# Internet Route

```yaml
Resources:

  DefaultRoute:

    Type: AWS::EC2::Route

    DependsOn: GatewayAttachment

    Properties:

      RouteTableId: !Ref PublicRouteTable

      DestinationCidrBlock: 0.0.0.0/0

      GatewayId: !Ref InternetGateway
```

---

# Route Table Association

```yaml
Resources:

  PublicAssociation:

    Type: AWS::EC2::SubnetRouteTableAssociation

    Properties:

      RouteTableId: !Ref PublicRouteTable

      SubnetId: !Ref PublicSubnet
```

---

# Private Route Table

```yaml
Resources:

  PrivateRouteTable:

    Type: AWS::EC2::RouteTable

    Properties:

      VpcId: !Ref VPC
```

---

# NAT Route

```yaml
Resources:

  PrivateRoute:

    Type: AWS::EC2::Route

    Properties:

      RouteTableId: !Ref PrivateRouteTable

      DestinationCidrBlock: 0.0.0.0/0

      NatGatewayId: !Ref NatGateway
```

---

# Associate Private Route Table

```yaml
Resources:

  PrivateAssociation:

    Type: AWS::EC2::SubnetRouteTableAssociation

    Properties:

      RouteTableId: !Ref PrivateRouteTable

      SubnetId: !Ref PrivateSubnet
```

---

# Security Group

```yaml
Resources:

  WebSecurityGroup:

    Type: AWS::EC2::SecurityGroup

    Properties:

      GroupDescription: Web Access

      VpcId: !Ref VPC
```

---

# SSH Rule

```yaml
SecurityGroupIngress:

  - IpProtocol: tcp

    FromPort: 22

    ToPort: 22

    CidrIp: 0.0.0.0/0
```

---

# HTTP Rule

```yaml
SecurityGroupIngress:

  - IpProtocol: tcp

    FromPort: 80

    ToPort: 80

    CidrIp: 0.0.0.0/0
```

---

# HTTPS Rule

```yaml
SecurityGroupIngress:

  - IpProtocol: tcp

    FromPort: 443

    ToPort: 443

    CidrIp: 0.0.0.0/0
```

---

# Egress Rule

```yaml
SecurityGroupEgress:

  - IpProtocol: -1

    CidrIp: 0.0.0.0/0
```

---

# Network ACL

```yaml
Resources:

  PublicACL:

    Type: AWS::EC2::NetworkAcl

    Properties:

      VpcId: !Ref VPC
```

---

# Inbound ACL Rule

```yaml
Resources:

  HttpInbound:

    Type: AWS::EC2::NetworkAclEntry

    Properties:

      NetworkAclId: !Ref PublicACL

      RuleNumber: 100

      Protocol: 6

      RuleAction: allow

      Egress: false

      CidrBlock: 0.0.0.0/0

      PortRange:

        From: 80

        To: 80
```

---

# Outbound ACL Rule

```yaml
Resources:

  OutboundRule:

    Type: AWS::EC2::NetworkAclEntry

    Properties:

      NetworkAclId: !Ref PublicACL

      RuleNumber: 100

      Protocol: -1

      RuleAction: allow

      Egress: true

      CidrBlock: 0.0.0.0/0
```

---

# VPC Peering

```yaml
Resources:

  VPCPeering:

    Type: AWS::EC2::VPCPeeringConnection

    Properties:

      VpcId: !Ref VPC

      PeerVpcId: vpc-xxxxxxxx
```

---

# Transit Gateway

```yaml
Resources:

  TransitGateway:

    Type: AWS::EC2::TransitGateway

    Properties:

      Description: Production TGW
```

---

# Transit Gateway Attachment

```yaml
Resources:

  TGWAttachment:

    Type: AWS::EC2::TransitGatewayAttachment

    Properties:

      TransitGatewayId: !Ref TransitGateway

      VpcId: !Ref VPC

      SubnetIds:

        - !Ref PrivateSubnet
```

---

# Elastic Network Interface

```yaml
Resources:

  ENI:

    Type: AWS::EC2::NetworkInterface

    Properties:

      SubnetId: !Ref PrivateSubnet
```

---

# Assign Elastic IP to ENI

```yaml
Resources:

  ENIAssociation:

    Type: AWS::EC2::EIPAssociation

    Properties:

      AllocationId: !GetAtt NatEIP.AllocationId

      NetworkInterfaceId: !Ref ENI
```

---

# Gateway VPC Endpoint (S3)

```yaml
Resources:

  S3Endpoint:

    Type: AWS::EC2::VPCEndpoint

    Properties:

      VpcId: !Ref VPC

      ServiceName: !Sub com.amazonaws.${AWS::Region}.s3

      VpcEndpointType: Gateway

      RouteTableIds:

        - !Ref PrivateRouteTable
```

---

# Interface Endpoint (SSM)

```yaml
Resources:

  SSMEndpoint:

    Type: AWS::EC2::VPCEndpoint

    Properties:

      VpcId: !Ref VPC

      ServiceName: !Sub com.amazonaws.${AWS::Region}.ssm

      VpcEndpointType: Interface

      PrivateDnsEnabled: true

      SecurityGroupIds:

        - !Ref WebSecurityGroup

      SubnetIds:

        - !Ref PrivateSubnet
```

---

# Route53 Hosted Zone

```yaml
Resources:

  PrivateHostedZone:

    Type: AWS::Route53::HostedZone

    Properties:

      Name: internal.local

      VPCs:

        - VPCId: !Ref VPC

          VPCRegion: !Ref AWS::Region
```

---

# Route53 Record

```yaml
Resources:

  AppRecord:

    Type: AWS::Route53::RecordSet

    Properties:

      HostedZoneName: internal.local.

      Name: app.internal.local.

      Type: A

      TTL: 300

      ResourceRecords:

        - 10.0.2.10
```

---

# DHCP Options

```yaml
Resources:

  DHCPOptions:

    Type: AWS::EC2::DHCPOptions

    Properties:

      DomainName: ec2.internal

      DomainNameServers:

        - AmazonProvidedDNS
```

---

# Associate DHCP Options

```yaml
Resources:

  DHCPAssociation:

    Type: AWS::EC2::VPCDHCPOptionsAssociation

    Properties:

      VpcId: !Ref VPC

      DhcpOptionsId: !Ref DHCPOptions
```

---

# Outputs

```yaml
Outputs:

  VPCId:

    Value: !Ref VPC

  PublicSubnet:

    Value: !Ref PublicSubnet

  PrivateSubnet:

    Value: !Ref PrivateSubnet

  InternetGateway:

    Value: !Ref InternetGateway
```

---

# Best Practices

- Enable DNS Support and DNS Hostnames.
- Separate public and private subnets.
- Deploy subnets across multiple Availability Zones.
- Use NAT Gateway only for private subnet internet access.
- Restrict Security Groups to required ports.
- Use VPC Endpoints to reduce NAT Gateway traffic.
- Use Route53 Private Hosted Zones for internal DNS.
- Avoid using the default VPC in production.
- Tag all networking resources consistently.
- Protect critical networking resources with stack policies.

---

# Summary

This section covered CloudFormation templates for Amazon VPC, Internet Gateway, NAT Gateway, Subnets, Route Tables, Security Groups, Network ACLs, VPC Peering, Transit Gateway, ENIs, VPC Endpoints, Route53 integration, DHCP options, and networking outputs. These templates provide production-ready patterns for building secure and scalable AWS networking infrastructure using CloudFormation.

---

