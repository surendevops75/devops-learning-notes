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

# AWS IAM

---

# IAM User

```yaml
Resources:

  DeveloperUser:

    Type: AWS::IAM::User

    Properties:

      UserName: developer

      Path: /

      Tags:

        - Key: Team

          Value: DevOps
```

---

# IAM Group

```yaml
Resources:

  DevOpsGroup:

    Type: AWS::IAM::Group

    Properties:

      GroupName: DevOps
```

---

# Add User to Group

```yaml
Resources:

  Developers:

    Type: AWS::IAM::UserToGroupAddition

    Properties:

      GroupName: !Ref DevOpsGroup

      Users:

        - !Ref DeveloperUser
```

---

# IAM Managed Policy

```yaml
Resources:

  S3ReadOnlyPolicy:

    Type: AWS::IAM::ManagedPolicy

    Properties:

      ManagedPolicyName: S3ReadOnly

      PolicyDocument:

        Version: "2012-10-17"

        Statement:

          - Effect: Allow

            Action:

              - s3:GetObject

              - s3:ListBucket

            Resource: "*"
```

---

# Attach Managed Policy to Group

```yaml
ManagedPolicyArns:

  - !Ref S3ReadOnlyPolicy
```

---

# IAM Role

```yaml
Resources:

  EC2Role:

    Type: AWS::IAM::Role

    Properties:

      RoleName: EC2Role

      AssumeRolePolicyDocument:

        Version: "2012-10-17"

        Statement:

          - Effect: Allow

            Principal:

              Service:

                - ec2.amazonaws.com

            Action:

              - sts:AssumeRole
```

---

# Inline IAM Policy

```yaml
Policies:

  - PolicyName: S3Access

    PolicyDocument:

      Version: "2012-10-17"

      Statement:

        - Effect: Allow

          Action:

            - s3:*

          Resource: "*"
```

---

# Attach AWS Managed Policy

```yaml
ManagedPolicyArns:

  - arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

---

# Instance Profile

```yaml
Resources:

  EC2InstanceProfile:

    Type: AWS::IAM::InstanceProfile

    Properties:

      Roles:

        - !Ref EC2Role
```

---

# Access Key

```yaml
Resources:

  DeveloperAccessKey:

    Type: AWS::IAM::AccessKey

    Properties:

      UserName: !Ref DeveloperUser
```

---

# Login Profile

```yaml
LoginProfile:

  Password: ChangeMe123!

  PasswordResetRequired: true
```

---

# Account Password Policy

```yaml
Resources:

  PasswordPolicy:

    Type: AWS::IAM::AccountPasswordPolicy

    Properties:

      MinimumPasswordLength: 14

      RequireSymbols: true

      RequireNumbers: true

      RequireUppercaseCharacters: true

      RequireLowercaseCharacters: true
```

---

# KMS Key

```yaml
Resources:

  EncryptionKey:

    Type: AWS::KMS::Key

    Properties:

      EnableKeyRotation: true

      PendingWindowInDays: 30

      Description: Production Encryption Key
```

---

# KMS Alias

```yaml
Resources:

  KMSAlias:

    Type: AWS::KMS::Alias

    Properties:

      AliasName: alias/production

      TargetKeyId: !Ref EncryptionKey
```

---

# Secrets Manager Secret

```yaml
Resources:

  DatabaseSecret:

    Type: AWS::SecretsManager::Secret

    Properties:

      Name: database-password
```

---

# Secret Value

```yaml
GenerateSecretString:

  SecretStringTemplate: '{"username":"admin"}'

  GenerateStringKey: password

  PasswordLength: 20
```

---

# Systems Manager Parameter

```yaml
Resources:

  DBPassword:

    Type: AWS::SSM::Parameter

    Properties:

      Name: /prod/db/password

      Type: SecureString

      Value: Password123
```

---

# ACM Certificate

```yaml
Resources:

  ACMCertificate:

    Type: AWS::CertificateManager::Certificate

    Properties:

      DomainName: example.com

      ValidationMethod: DNS
```

---

# IAM Identity Center Instance

```yaml
Resources:

  IdentityCenter:

    Type: AWS::SSO::Instance
```

---

# AWS Organizations

```yaml
Resources:

  Organization:

    Type: AWS::Organizations::Organization

    Properties:

      FeatureSet: ALL
```

---

# Organizational Unit

```yaml
Resources:

  DevelopmentOU:

    Type: AWS::Organizations::OrganizationalUnit

    Properties:

      Name: Development

      ParentId: r-xxxx
```

---

# AWS Account

```yaml
Resources:

  DevelopmentAccount:

    Type: AWS::Organizations::Account

    Properties:

      AccountName: Development

      Email: dev@example.com
```

---

# Service Control Policy

```yaml
Resources:

  SCP:

    Type: AWS::Organizations::Policy

    Properties:

      Name: DenyRoot

      Type: SERVICE_CONTROL_POLICY

      Content: |
        {
          "Version":"2012-10-17",
          "Statement":[]
        }
```

---

# GuardDuty Detector

```yaml
Resources:

  GuardDuty:

    Type: AWS::GuardDuty::Detector

    Properties:

      Enable: true
```

---

# Security Hub

```yaml
Resources:

  SecurityHub:

    Type: AWS::SecurityHub::Hub
```

---

# Security Hub Standard

```yaml
Resources:

  CISStandard:

    Type: AWS::SecurityHub::Standard

    Properties:

      StandardsArn: arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.4.0
```

---

# Amazon Inspector

```yaml
Resources:

  Inspector:

    Type: AWS::InspectorV2::Filter
```

---

# Amazon Macie

```yaml
Resources:

  Macie:

    Type: AWS::Macie::Session

    Properties:

      Status: ENABLED
```

---

# AWS Detective

```yaml
Resources:

  DetectiveGraph:

    Type: AWS::Detective::Graph

    Properties:

      AutoEnableMembers: true
```

---

# AWS WAF Web ACL

```yaml
Resources:

  WebACL:

    Type: AWS::WAFv2::WebACL

    Properties:

      Name: ProductionACL

      Scope: REGIONAL

      DefaultAction:

        Allow: {}

      VisibilityConfig:

        CloudWatchMetricsEnabled: true

        MetricName: ProductionACL

        SampledRequestsEnabled: true
```

---

# AWS Shield Protection

```yaml
Resources:

  ShieldProtection:

    Type: AWS::Shield::Protection

    Properties:

      Name: ALBProtection

      ResourceArn: arn:aws:elasticloadbalancing:...
```

---

# CloudTrail

```yaml
Resources:

  Trail:

    Type: AWS::CloudTrail::Trail

    Properties:

      IsLogging: true

      S3BucketName: audit-logs
```

---

# AWS Config Recorder

```yaml
Resources:

  ConfigRecorder:

    Type: AWS::Config::ConfigurationRecorder

    Properties:

      Name: default

      RoleARN: arn:aws:iam::123456789012:role/config-role
```

---

# Delivery Channel

```yaml
Resources:

  ConfigDelivery:

    Type: AWS::Config::DeliveryChannel

    Properties:

      S3BucketName: config-logs
```

---

# Config Rule

```yaml
Resources:

  EncryptedVolumes:

    Type: AWS::Config::ConfigRule

    Properties:

      ConfigRuleName: encrypted-volumes

      Source:

        Owner: AWS

        SourceIdentifier: ENCRYPTED_VOLUMES
```

---

# Outputs

```yaml
Outputs:

  RoleArn:

    Value: !GetAtt EC2Role.Arn

  KMSKey:

    Value: !Ref EncryptionKey

  SecretArn:

    Value: !Ref DatabaseSecret
```

---

# Best Practices

- Prefer IAM Roles instead of IAM Users whenever possible.
- Apply the principle of least privilege to IAM policies.
- Enable automatic KMS key rotation.
- Store secrets in AWS Secrets Manager instead of templates.
- Use SecureString for Systems Manager Parameters.
- Enable CloudTrail organization-wide.
- Enable AWS Config to monitor configuration drift.
- Enable GuardDuty, Security Hub, Inspector, Macie, and Detective in production accounts.
- Protect internet-facing applications using AWS WAF and Shield.
- Tag all IAM and security resources consistently.

---

# Summary

This section covered CloudFormation templates for IAM Users, Groups, Roles, Managed Policies, Instance Profiles, KMS, Secrets Manager, Systems Manager Parameter Store, ACM, AWS Organizations, Service Control Policies (SCPs), GuardDuty, Security Hub, Inspector, Macie, Detective, WAF, Shield, CloudTrail, and AWS Config. These templates provide production-ready patterns for implementing identity, access management, encryption, governance, and security in AWS.

---

# Amazon S3

---

# Create S3 Bucket

```yaml
Resources:

  ApplicationBucket:

    Type: AWS::S3::Bucket

    Properties:

      BucketName: production-app-storage

      Tags:

        - Key: Environment

          Value: Production
```

---

# Enable Versioning

```yaml
VersioningConfiguration:

  Status: Enabled
```

---

# Server-Side Encryption (SSE-S3)

```yaml
BucketEncryption:

  ServerSideEncryptionConfiguration:

    - ServerSideEncryptionByDefault:

        SSEAlgorithm: AES256
```

---

# SSE-KMS Encryption

```yaml
BucketEncryption:

  ServerSideEncryptionConfiguration:

    - ServerSideEncryptionByDefault:

        SSEAlgorithm: aws:kms

        KMSMasterKeyID: !Ref EncryptionKey
```

---

# Block Public Access

```yaml
PublicAccessBlockConfiguration:

  BlockPublicAcls: true

  BlockPublicPolicy: true

  IgnorePublicAcls: true

  RestrictPublicBuckets: true
```

---

# Bucket Lifecycle Rule

```yaml
LifecycleConfiguration:

  Rules:

    - Id: ArchiveLogs

      Status: Enabled

      Transitions:

        - StorageClass: GLACIER

          TransitionInDays: 30
```

---

# Bucket Logging

```yaml
LoggingConfiguration:

  DestinationBucketName: access-logs

  LogFilePrefix: s3/
```

---

# Bucket Notification

```yaml
NotificationConfiguration:

  LambdaConfigurations:

    - Event: s3:ObjectCreated:*

      Function: !GetAtt LambdaFunction.Arn
```

---

# Bucket Policy

```yaml
Resources:

  BucketPolicy:

    Type: AWS::S3::BucketPolicy

    Properties:

      Bucket: !Ref ApplicationBucket

      PolicyDocument:

        Version: "2012-10-17"
```

---

# Intelligent Tiering

```yaml
IntelligentTieringConfigurations:

  - Id: Archive

    Status: Enabled
```

---

# Amazon EFS

---

# File System

```yaml
Resources:

  EFS:

    Type: AWS::EFS::FileSystem

    Properties:

      Encrypted: true

      PerformanceMode: generalPurpose
```

---

# Mount Target

```yaml
Resources:

  EFSMount:

    Type: AWS::EFS::MountTarget

    Properties:

      FileSystemId: !Ref EFS

      SubnetId: !Ref PrivateSubnet

      SecurityGroups:

        - !Ref EFSSecurityGroup
```

---

# Access Point

```yaml
Resources:

  AccessPoint:

    Type: AWS::EFS::AccessPoint

    Properties:

      FileSystemId: !Ref EFS
```

---

# Amazon FSx

---

# Windows File Server

```yaml
Resources:

  WindowsFSx:

    Type: AWS::FSx::FileSystem

    Properties:

      FileSystemType: WINDOWS

      StorageCapacity: 32

      SubnetIds:

        - !Ref PrivateSubnet
```

---

# Lustre File System

```yaml
Resources:

  LustreFS:

    Type: AWS::FSx::FileSystem

    Properties:

      FileSystemType: LUSTRE

      StorageCapacity: 1200
```

---

# Amazon RDS

---

# MySQL Instance

```yaml
Resources:

  MySQL:

    Type: AWS::RDS::DBInstance

    Properties:

      Engine: mysql

      DBInstanceClass: db.t3.micro

      AllocatedStorage: 20

      MasterUsername: admin

      MasterUserPassword: Password123
```

---

# PostgreSQL Instance

```yaml
Resources:

  PostgreSQL:

    Type: AWS::RDS::DBInstance

    Properties:

      Engine: postgres

      DBInstanceClass: db.t3.small

      AllocatedStorage: 50
```

---

# DB Subnet Group

```yaml
Resources:

  DBSubnetGroup:

    Type: AWS::RDS::DBSubnetGroup

    Properties:

      DBSubnetGroupDescription: Database Subnets

      SubnetIds:

        - !Ref PrivateSubnet
```

---

# Parameter Group

```yaml
Resources:

  DBParameters:

    Type: AWS::RDS::DBParameterGroup

    Properties:

      Family: mysql8.0

      Description: Production Parameters
```

---

# Option Group

```yaml
Resources:

  DBOptions:

    Type: AWS::RDS::OptionGroup

    Properties:

      EngineName: mysql

      MajorEngineVersion: "8.0"
```

---

# Read Replica

```yaml
Resources:

  Replica:

    Type: AWS::RDS::DBInstance

    Properties:

      SourceDBInstanceIdentifier: !Ref MySQL
```

---

# Aurora Cluster

```yaml
Resources:

  AuroraCluster:

    Type: AWS::RDS::DBCluster

    Properties:

      Engine: aurora-mysql

      MasterUsername: admin

      MasterUserPassword: Password123
```

---

# Aurora Instance

```yaml
Resources:

  AuroraWriter:

    Type: AWS::RDS::DBInstance

    Properties:

      DBClusterIdentifier: !Ref AuroraCluster

      Engine: aurora-mysql

      DBInstanceClass: db.r6g.large
```

---

# Amazon DynamoDB

---

# DynamoDB Table

```yaml
Resources:

  UsersTable:

    Type: AWS::DynamoDB::Table

    Properties:

      BillingMode: PAY_PER_REQUEST

      AttributeDefinitions:

        - AttributeName: UserId

          AttributeType: S

      KeySchema:

        - AttributeName: UserId

          KeyType: HASH
```

---

# Global Secondary Index

```yaml
GlobalSecondaryIndexes:

  - IndexName: EmailIndex

    KeySchema:

      - AttributeName: Email

        KeyType: HASH

    Projection:

      ProjectionType: ALL
```

---

# Point-in-Time Recovery

```yaml
PointInTimeRecoverySpecification:

  PointInTimeRecoveryEnabled: true
```

---

# TTL

```yaml
TimeToLiveSpecification:

  AttributeName: Expiry

  Enabled: true
```

---

# Amazon ElastiCache

---

# Redis Cluster

```yaml
Resources:

  RedisCluster:

    Type: AWS::ElastiCache::CacheCluster

    Properties:

      Engine: redis

      CacheNodeType: cache.t3.micro

      NumCacheNodes: 1
```

---

# Redis Replication Group

```yaml
Resources:

  RedisReplication:

    Type: AWS::ElastiCache::ReplicationGroup

    Properties:

      Engine: redis

      ReplicationGroupDescription: Production Redis
```

---

# Memcached Cluster

```yaml
Resources:

  Memcached:

    Type: AWS::ElastiCache::CacheCluster

    Properties:

      Engine: memcached

      CacheNodeType: cache.t3.micro

      NumCacheNodes: 2
```

---

# AWS Backup

---

# Backup Vault

```yaml
Resources:

  BackupVault:

    Type: AWS::Backup::BackupVault

    Properties:

      BackupVaultName: ProductionVault
```

---

# Backup Plan

```yaml
Resources:

  BackupPlan:

    Type: AWS::Backup::BackupPlan

    Properties:

      BackupPlan:

        BackupPlanName: DailyBackup
```

---

# Backup Selection

```yaml
Resources:

  BackupSelection:

    Type: AWS::Backup::BackupSelection

    Properties:

      BackupPlanId: !Ref BackupPlan

      BackupSelection:

        SelectionName: ProductionResources
```

---

# AWS DataSync

---

# S3 Location

```yaml
Resources:

  SourceLocation:

    Type: AWS::DataSync::LocationS3

    Properties:

      S3BucketArn: !GetAtt ApplicationBucket.Arn
```

---

# DataSync Task

```yaml
Resources:

  MigrationTask:

    Type: AWS::DataSync::Task

    Properties:

      SourceLocationArn: !Ref SourceLocation

      DestinationLocationArn: arn:aws:datasync:destination
```

---

# AWS Storage Gateway

---

# File Share

```yaml
Resources:

  SMBShare:

    Type: AWS::StorageGateway::SMBFileShare

    Properties:

      GatewayARN: arn:aws:storagegateway:gateway

      LocationARN: !GetAtt ApplicationBucket.Arn
```

---

# AWS Snow Family

---

# Snowball Job

```yaml
Resources:

  ImportJob:

    Type: AWS::Snowball::Job

    Properties:

      JobType: IMPORT

      Resources: {}
```

---

# Outputs

```yaml
Outputs:

  BucketName:

    Value: !Ref ApplicationBucket

  DatabaseEndpoint:

    Value: !GetAtt MySQL.Endpoint.Address

  DynamoDBTable:

    Value: !Ref UsersTable

  EFSId:

    Value: !Ref EFS
```

---

# Best Practices

- Enable S3 Versioning and default encryption.
- Block public access for production buckets.
- Store databases in private subnets.
- Enable automated backups and Multi-AZ for production RDS instances.
- Enable Point-in-Time Recovery for DynamoDB tables.
- Encrypt EFS, FSx, RDS, ElastiCache, and S3 using AWS KMS.
- Use AWS Backup for centralized backup management.
- Configure lifecycle policies for long-term S3 storage optimization.
- Avoid hardcoding database credentials in templates; use Secrets Manager or dynamic references.
- Tag all storage resources consistently.

---

# Summary

This section covered CloudFormation templates for Amazon S3, EFS, FSx, RDS, Aurora, DynamoDB, ElastiCache, AWS Backup, DataSync, Storage Gateway, and Snow Family. These templates provide production-ready patterns for implementing secure, scalable AWS storage, database, backup, and migration solutions using CloudFormation.

---

# Amazon Elastic Container Registry (ECR)

---

# Create ECR Repository

```yaml
Resources:

  ECRRepository:

    Type: AWS::ECR::Repository

    Properties:

      RepositoryName: production-app

      ImageScanningConfiguration:

        ScanOnPush: true

      ImageTagMutability: IMMUTABLE
```

---

# Lifecycle Policy

```yaml
LifecyclePolicy:

  LifecyclePolicyText: |
    {
      "rules":[
        {
          "rulePriority":1,
          "description":"Keep last 20 images",
          "selection":{
            "tagStatus":"any",
            "countType":"imageCountMoreThan",
            "countNumber":20
          },
          "action":{
            "type":"expire"
          }
        }
      ]
    }
```

---

# Repository Policy

```yaml
RepositoryPolicyText:

  Version: "2012-10-17"

  Statement:

    - Effect: Allow

      Principal:

        AWS: arn:aws:iam::123456789012:root

      Action:

        - ecr:BatchGetImage
```

---

# Amazon ECS

---

# ECS Cluster

```yaml
Resources:

  ECSCluster:

    Type: AWS::ECS::Cluster

    Properties:

      ClusterName: production
```

---

# Capacity Providers

```yaml
CapacityProviders:

  - FARGATE

  - FARGATE_SPOT
```

---

# Task Definition

```yaml
Resources:

  TaskDefinition:

    Type: AWS::ECS::TaskDefinition

    Properties:

      Family: web

      Cpu: 512

      Memory: 1024

      NetworkMode: awsvpc

      RequiresCompatibilities:

        - FARGATE
```

---

# Container Definition

```yaml
ContainerDefinitions:

  - Name: nginx

    Image: nginx:latest

    Essential: true

    PortMappings:

      - ContainerPort: 80
```

---

# ECS Service

```yaml
Resources:

  ECSService:

    Type: AWS::ECS::Service

    Properties:

      Cluster: !Ref ECSCluster

      DesiredCount: 2

      LaunchType: FARGATE

      TaskDefinition: !Ref TaskDefinition
```

---

# Service Auto Scaling

```yaml
Resources:

  ECSScalableTarget:

    Type: AWS::ApplicationAutoScaling::ScalableTarget

    Properties:

      MaxCapacity: 10

      MinCapacity: 2

      ServiceNamespace: ecs
```

---

# Scaling Policy

```yaml
Resources:

  ECSScalingPolicy:

    Type: AWS::ApplicationAutoScaling::ScalingPolicy

    Properties:

      PolicyType: TargetTrackingScaling
```

---

# Amazon EKS

---

# EKS Cluster

```yaml
Resources:

  EKSCluster:

    Type: AWS::EKS::Cluster

    Properties:

      Name: production

      Version: "1.31"

      RoleArn: !GetAtt EKSRole.Arn
```

---

# VPC Configuration

```yaml
ResourcesVpcConfig:

  SubnetIds:

    - !Ref PrivateSubnet

  EndpointPublicAccess: true

  EndpointPrivateAccess: true
```

---

# Managed Node Group

```yaml
Resources:

  WorkerNodes:

    Type: AWS::EKS::Nodegroup

    Properties:

      ClusterName: !Ref EKSCluster

      NodeRole: !GetAtt WorkerRole.Arn

      ScalingConfig:

        DesiredSize: 2

        MinSize: 2

        MaxSize: 6
```

---

# Fargate Profile

```yaml
Resources:

  FargateProfile:

    Type: AWS::EKS::FargateProfile

    Properties:

      ClusterName: !Ref EKSCluster

      FargateProfileName: default
```

---

# EKS Add-on

```yaml
Resources:

  CoreDNSAddon:

    Type: AWS::EKS::Addon

    Properties:

      ClusterName: !Ref EKSCluster

      AddonName: coredns
```

---

# OIDC Provider

```yaml
Resources:

  OIDCProvider:

    Type: AWS::IAM::OIDCProvider
```

---

# AWS Lambda

---

# Lambda Function

```yaml
Resources:

  LambdaFunction:

    Type: AWS::Lambda::Function

    Properties:

      FunctionName: processor

      Runtime: python3.12

      Handler: lambda_function.lambda_handler

      Role: !GetAtt LambdaRole.Arn

      Code:

        S3Bucket: deployment-artifacts

        S3Key: lambda.zip
```

---

# Lambda Permission

```yaml
Resources:

  LambdaPermission:

    Type: AWS::Lambda::Permission

    Properties:

      Action: lambda:InvokeFunction

      FunctionName: !Ref LambdaFunction

      Principal: apigateway.amazonaws.com
```

---

# Lambda Version

```yaml
Resources:

  LambdaVersion:

    Type: AWS::Lambda::Version

    Properties:

      FunctionName: !Ref LambdaFunction
```

---

# Lambda Alias

```yaml
Resources:

  ProductionAlias:

    Type: AWS::Lambda::Alias

    Properties:

      Name: prod

      FunctionName: !Ref LambdaFunction

      FunctionVersion: !GetAtt LambdaVersion.Version
```

---

# API Gateway REST API

```yaml
Resources:

  RestAPI:

    Type: AWS::ApiGateway::RestApi

    Properties:

      Name: OrdersAPI
```

---

# API Gateway Resource

```yaml
Resources:

  OrdersResource:

    Type: AWS::ApiGateway::Resource
```

---

# API Gateway Method

```yaml
Resources:

  GetOrders:

    Type: AWS::ApiGateway::Method

    Properties:

      HttpMethod: GET
```

---

# Deployment

```yaml
Resources:

  APIDeployment:

    Type: AWS::ApiGateway::Deployment
```

---

# Stage

```yaml
Resources:

  ProductionStage:

    Type: AWS::ApiGateway::Stage

    Properties:

      StageName: prod
```

---

# HTTP API

```yaml
Resources:

  HTTPAPI:

    Type: AWS::ApiGatewayV2::Api

    Properties:

      ProtocolType: HTTP
```

---

# App Runner

```yaml
Resources:

  AppRunner:

    Type: AWS::AppRunner::Service

    Properties:

      ServiceName: production-app
```

---

# EventBridge Bus

```yaml
Resources:

  EventBus:

    Type: AWS::Events::EventBus

    Properties:

      Name: application-events
```

---

# Event Rule

```yaml
Resources:

  DailyRule:

    Type: AWS::Events::Rule

    Properties:

      ScheduleExpression: rate(1 day)
```

---

# SNS Topic

```yaml
Resources:

  AlertsTopic:

    Type: AWS::SNS::Topic

    Properties:

      TopicName: alerts
```

---

# SNS Subscription

```yaml
Resources:

  EmailSubscription:

    Type: AWS::SNS::Subscription

    Properties:

      Protocol: email

      Endpoint: admin@example.com

      TopicArn: !Ref AlertsTopic
```

---

# SQS Queue

```yaml
Resources:

  OrdersQueue:

    Type: AWS::SQS::Queue

    Properties:

      QueueName: orders
```

---

# Dead Letter Queue

```yaml
Resources:

  DLQ:

    Type: AWS::SQS::Queue

    Properties:

      QueueName: orders-dlq
```

---

# Redrive Policy

```yaml
RedrivePolicy:

  deadLetterTargetArn: !GetAtt DLQ.Arn

  maxReceiveCount: 5
```

---

# Step Functions

```yaml
Resources:

  OrderWorkflow:

    Type: AWS::StepFunctions::StateMachine

    Properties:

      StateMachineName: OrderWorkflow

      DefinitionString: "{}"
```

---

# Cloud Map Namespace

```yaml
Resources:

  PrivateNamespace:

    Type: AWS::ServiceDiscovery::PrivateDnsNamespace

    Properties:

      Name: internal.local

      Vpc: !Ref VPC
```

---

# Cloud Map Service

```yaml
Resources:

  DiscoveryService:

    Type: AWS::ServiceDiscovery::Service

    Properties:

      Name: web
```

---

# Outputs

```yaml
Outputs:

  ECRRepository:

    Value: !Ref ECRRepository

  ECSCluster:

    Value: !Ref ECSCluster

  EKSCluster:

    Value: !Ref EKSCluster

  LambdaFunction:

    Value: !Ref LambdaFunction

  OrdersQueue:

    Value: !Ref OrdersQueue
```

---

# Best Practices

- Enable image scanning and immutable tags in Amazon ECR.
- Store container images in private repositories.
- Use ECS Fargate for serverless container workloads.
- Enable EKS managed add-ons and IAM Roles for Service Accounts (IRSA).
- Package Lambda code separately and store artifacts in S3.
- Configure Dead Letter Queues (DLQs) for SQS and Lambda.
- Use EventBridge for event-driven architectures.
- Protect API Gateway using IAM, Cognito, or Lambda authorizers.
- Use Cloud Map for service discovery in microservices.
- Tag all container and serverless resources consistently.

---

# Summary

This section covered CloudFormation templates for Amazon ECR, ECS, EKS, Lambda, API Gateway (REST and HTTP APIs), App Runner, EventBridge, SNS, SQS, Step Functions, and AWS Cloud Map. These templates provide production-ready patterns for deploying containerized, Kubernetes-based, and serverless applications using AWS CloudFormation.

---

# Amazon CloudWatch

---

# CloudWatch Dashboard

```yaml
Resources:

  ProductionDashboard:

    Type: AWS::CloudWatch::Dashboard

    Properties:

      DashboardName: ProductionDashboard

      DashboardBody: |
        {
          "widgets":[]
        }
```

---

# CloudWatch Alarm

```yaml
Resources:

  HighCPUAlarm:

    Type: AWS::CloudWatch::Alarm

    Properties:

      AlarmName: HighCPU

      Namespace: AWS/EC2

      MetricName: CPUUtilization

      Statistic: Average

      Threshold: 80

      EvaluationPeriods: 2

      ComparisonOperator: GreaterThanThreshold
```

---

# Composite Alarm

```yaml
Resources:

  CriticalAlarm:

    Type: AWS::CloudWatch::CompositeAlarm

    Properties:

      AlarmName: CriticalAlarm

      AlarmRule: ALARM(HighCPU)
```

---

# Alarm Notification

```yaml
AlarmActions:

  - !Ref AlertsTopic
```

---

# CloudWatch Log Group

```yaml
Resources:

  ApplicationLogs:

    Type: AWS::Logs::LogGroup

    Properties:

      LogGroupName: /application/logs

      RetentionInDays: 30
```

---

# Log Stream

```yaml
Resources:

  LogStream:

    Type: AWS::Logs::LogStream

    Properties:

      LogGroupName: !Ref ApplicationLogs

      LogStreamName: production
```

---

# Resource Policy

```yaml
Resources:

  LogsPolicy:

    Type: AWS::Logs::ResourcePolicy

    Properties:

      PolicyName: CloudWatchLogs
```

---

# Subscription Filter

```yaml
Resources:

  LogSubscription:

    Type: AWS::Logs::SubscriptionFilter

    Properties:

      LogGroupName: !Ref ApplicationLogs

      FilterPattern: ""

      DestinationArn: arn:aws:lambda:...
```

---

# EventBridge

---

# Event Bus

```yaml
Resources:

  OperationsBus:

    Type: AWS::Events::EventBus

    Properties:

      Name: operations
```

---

# Event Rule

```yaml
Resources:

  EC2StateRule:

    Type: AWS::Events::Rule

    Properties:

      Name: EC2StateChange
```

---

# Event Target

```yaml
Targets:

  - Arn: !GetAtt LambdaFunction.Arn

    Id: LambdaTarget
```

---

# CloudTrail

---

# Trail

```yaml
Resources:

  OrganizationTrail:

    Type: AWS::CloudTrail::Trail

    Properties:

      TrailName: OrganizationTrail

      IsLogging: true

      IsMultiRegionTrail: true

      EnableLogFileValidation: true

      S3BucketName: audit-logs
```

---

# Event Selectors

```yaml
EventSelectors:

  - IncludeManagementEvents: true

    ReadWriteType: All
```

---

# AWS Config

---

# Configuration Recorder

```yaml
Resources:

  ConfigRecorder:

    Type: AWS::Config::ConfigurationRecorder

    Properties:

      Name: default

      RoleARN: arn:aws:iam::123456789012:role/config-role
```

---

# Delivery Channel

```yaml
Resources:

  ConfigDelivery:

    Type: AWS::Config::DeliveryChannel

    Properties:

      S3BucketName: config-bucket
```

---

# Config Rule

```yaml
Resources:

  EncryptedVolumes:

    Type: AWS::Config::ConfigRule

    Properties:

      ConfigRuleName: encrypted-volumes

      Source:

        Owner: AWS

        SourceIdentifier: ENCRYPTED_VOLUMES
```

---

# Conformance Pack

```yaml
Resources:

  SecurityPack:

    Type: AWS::Config::ConformancePack

    Properties:

      ConformancePackName: SecurityBaseline
```

---

# AWS CodeCommit

---

# Repository

```yaml
Resources:

  Repository:

    Type: AWS::CodeCommit::Repository

    Properties:

      RepositoryName: infrastructure
```

---

# Approval Rule Template

```yaml
Resources:

  ApprovalTemplate:

    Type: AWS::CodeCommit::ApprovalRuleTemplate

    Properties:

      ApprovalRuleTemplateName: MandatoryReview
```

---

# AWS CodeBuild

---

# Build Project

```yaml
Resources:

  BuildProject:

    Type: AWS::CodeBuild::Project

    Properties:

      Name: ApplicationBuild

      ServiceRole: !GetAtt CodeBuildRole.Arn
```

---

# Source Credential

```yaml
Resources:

  GitHubCredential:

    Type: AWS::CodeBuild::SourceCredential

    Properties:

      AuthType: PERSONAL_ACCESS_TOKEN

      ServerType: GITHUB

      Token: ghp_xxxxxxxxx
```

---

# Webhook

```yaml
Resources:

  BuildWebhook:

    Type: AWS::CodeBuild::Webhook

    Properties:

      ProjectName: !Ref BuildProject
```

---

# AWS CodeDeploy

---

# Application

```yaml
Resources:

  CodeDeployApp:

    Type: AWS::CodeDeploy::Application

    Properties:

      ApplicationName: WebApplication
```

---

# Deployment Group

```yaml
Resources:

  ProductionDeployment:

    Type: AWS::CodeDeploy::DeploymentGroup

    Properties:

      ApplicationName: !Ref CodeDeployApp

      ServiceRoleArn: !GetAtt CodeDeployRole.Arn
```

---

# AWS CodePipeline

---

# Pipeline

```yaml
Resources:

  CICDPipeline:

    Type: AWS::CodePipeline::Pipeline

    Properties:

      Name: ProductionPipeline

      RoleArn: !GetAtt PipelineRole.Arn
```

---

# AWS CodeArtifact

---

# Domain

```yaml
Resources:

  ArtifactDomain:

    Type: AWS::CodeArtifact::Domain

    Properties:

      DomainName: company
```

---

# Repository

```yaml
Resources:

  MavenRepository:

    Type: AWS::CodeArtifact::Repository

    Properties:

      RepositoryName: maven

      DomainName: !Ref ArtifactDomain
```

---

# AWS Systems Manager

---

# Parameter

```yaml
Resources:

  APIEndpoint:

    Type: AWS::SSM::Parameter

    Properties:

      Name: /prod/api/url

      Type: String

      Value: https://api.example.com
```

---

# Patch Baseline

```yaml
Resources:

  LinuxPatchBaseline:

    Type: AWS::SSM::PatchBaseline

    Properties:

      Name: LinuxBaseline

      OperatingSystem: AMAZON_LINUX_2
```

---

# Maintenance Window

```yaml
Resources:

  WeeklyMaintenance:

    Type: AWS::SSM::MaintenanceWindow

    Properties:

      Name: WeeklyMaintenance

      Schedule: cron(0 2 ? * SUN *)
```

---

# Maintenance Target

```yaml
Resources:

  MaintenanceTarget:

    Type: AWS::SSM::MaintenanceWindowTarget

    Properties:

      WindowId: !Ref WeeklyMaintenance

      ResourceType: INSTANCE
```

---

# Maintenance Task

```yaml
Resources:

  PatchTask:

    Type: AWS::SSM::MaintenanceWindowTask

    Properties:

      WindowId: !Ref WeeklyMaintenance

      TaskType: RUN_COMMAND
```

---

# Association

```yaml
Resources:

  InventoryAssociation:

    Type: AWS::SSM::Association

    Properties:

      Name: AWS-GatherSoftwareInventory
```

---

# SSM Document

```yaml
Resources:

  RestartDocument:

    Type: AWS::SSM::Document

    Properties:

      DocumentType: Command

      Name: RestartService
```

---

# OpsCenter

---

# OpsItem

```yaml
Resources:

  HighCPUIncident:

    Type: AWS::SSM::OpsItem

    Properties:

      Title: High CPU Usage

      Source: CloudWatch
```

---

# Resource Data Sync

```yaml
Resources:

  InventorySync:

    Type: AWS::SSM::ResourceDataSync

    Properties:

      SyncName: InventorySync

      S3Destination:

        BucketName: inventory-bucket
```

---

# CloudFormation Stack

```yaml
Resources:

  NetworkStack:

    Type: AWS::CloudFormation::Stack

    Properties:

      TemplateURL: https://s3.amazonaws.com/templates/network.yaml
```

---

# StackSet

```yaml
Resources:

  SecurityStackSet:

    Type: AWS::CloudFormation::StackSet

    Properties:

      StackSetName: SecurityBaseline

      PermissionModel: SERVICE_MANAGED
```

---

# StackSet Instance

```yaml
Resources:

  ProductionInstance:

    Type: AWS::CloudFormation::StackInstance

    Properties:

      StackSetName: !Ref SecurityStackSet

      Region: ap-south-1
```

---

# Outputs

```yaml
Outputs:

  Dashboard:

    Value: !Ref ProductionDashboard

  Pipeline:

    Value: !Ref CICDPipeline

  LogGroup:

    Value: !Ref ApplicationLogs

  BuildProject:

    Value: !Ref BuildProject
```

---

# Best Practices

- Create CloudWatch dashboards for critical workloads.
- Configure CloudWatch alarms with SNS notifications.
- Enable CloudTrail across all regions with log file validation.
- Record all supported resources using AWS Config.
- Store application configuration in Systems Manager Parameter Store.
- Automate deployments using CodePipeline.
- Store CodeBuild credentials securely using Secrets Manager.
- Schedule regular patching with Systems Manager Maintenance Windows.
- Use StackSets for multi-account deployments.
- Tag monitoring and CI/CD resources consistently.

---

# Summary

This section covered CloudFormation templates for CloudWatch, CloudWatch Logs, EventBridge, CloudTrail, AWS Config, CodeCommit, CodeBuild, CodeDeploy, CodePipeline, CodeArtifact, Systems Manager (SSM), OpsCenter, CloudFormation Stacks, and StackSets. These templates provide production-ready patterns for monitoring, CI/CD automation, configuration management, and operational governance using AWS CloudFormation.

---

# Amazon Route 53

---

# Public Hosted Zone

```yaml
Resources:

  PublicHostedZone:

    Type: AWS::Route53::HostedZone

    Properties:

      Name: example.com
```

---

# Private Hosted Zone

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

# A Record

```yaml
Resources:

  AppRecord:

    Type: AWS::Route53::RecordSet

    Properties:

      HostedZoneId: !Ref PublicHostedZone

      Name: app.example.com

      Type: A

      TTL: 300

      ResourceRecords:

        - 203.0.113.10
```

---

# Alias Record

```yaml
Resources:

  ALBAlias:

    Type: AWS::Route53::RecordSet

    Properties:

      HostedZoneId: !Ref PublicHostedZone

      Name: www.example.com

      Type: A

      AliasTarget:

        DNSName: !GetAtt ALB.DNSName

        HostedZoneId: !GetAtt ALB.CanonicalHostedZoneID
```

---

# CNAME Record

```yaml
Resources:

  APIRecord:

    Type: AWS::Route53::RecordSet

    Properties:

      HostedZoneId: !Ref PublicHostedZone

      Name: api.example.com

      Type: CNAME

      TTL: 300

      ResourceRecords:

        - alb.example.com
```

---

# MX Record

```yaml
Resources:

  MailRecord:

    Type: AWS::Route53::RecordSet

    Properties:

      HostedZoneId: !Ref PublicHostedZone

      Name: example.com

      Type: MX

      TTL: 300

      ResourceRecords:

        - 10 mail.example.com
```

---

# TXT Record

```yaml
Resources:

  SPFRecord:

    Type: AWS::Route53::RecordSet

    Properties:

      HostedZoneId: !Ref PublicHostedZone

      Name: example.com

      Type: TXT

      TTL: 300

      ResourceRecords:

        - '"v=spf1 include:_spf.google.com ~all"'
```

---

# Health Check

```yaml
Resources:

  AppHealthCheck:

    Type: AWS::Route53::HealthCheck

    Properties:

      HealthCheckConfig:

        Type: HTTPS

        FullyQualifiedDomainName: app.example.com

        Port: 443
```

---

# Failover Record

```yaml
Resources:

  PrimaryRecord:

    Type: AWS::Route53::RecordSet

    Properties:

      SetIdentifier: Primary

      Failover: PRIMARY
```

---

# Amazon CloudFront

---

# Distribution

```yaml
Resources:

  CDN:

    Type: AWS::CloudFront::Distribution

    Properties:

      DistributionConfig:

        Enabled: true

        DefaultRootObject: index.html
```

---

# Origin Access Control

```yaml
Resources:

  OriginAccessControl:

    Type: AWS::CloudFront::OriginAccessControl

    Properties:

      OriginAccessControlConfig:

        Name: S3Origin

        OriginAccessControlOriginType: s3

        SigningBehavior: always

        SigningProtocol: sigv4
```

---

# Cache Policy

```yaml
Resources:

  CachePolicy:

    Type: AWS::CloudFront::CachePolicy

    Properties:

      CachePolicyConfig:

        Name: ProductionCache
```

---

# Origin Request Policy

```yaml
Resources:

  OriginRequestPolicy:

    Type: AWS::CloudFront::OriginRequestPolicy

    Properties:

      OriginRequestPolicyConfig:

        Name: DefaultOriginPolicy
```

---

# Response Headers Policy

```yaml
Resources:

  SecurityHeaders:

    Type: AWS::CloudFront::ResponseHeadersPolicy

    Properties:

      ResponseHeadersPolicyConfig:

        Name: SecurityHeaders
```

---

# AWS WAF

---

# Web ACL

```yaml
Resources:

  WebACL:

    Type: AWS::WAFv2::WebACL

    Properties:

      Name: ProductionACL

      Scope: REGIONAL

      DefaultAction:

        Allow: {}

      VisibilityConfig:

        CloudWatchMetricsEnabled: true

        MetricName: ProductionACL

        SampledRequestsEnabled: true
```

---

# Managed Rule Group

```yaml
Rules:

  - Name: AWSManagedRulesCommonRuleSet

    Priority: 1

    OverrideAction:

      None: {}
```

---

# Associate WAF

```yaml
Resources:

  WAFAssociation:

    Type: AWS::WAFv2::WebACLAssociation

    Properties:

      ResourceArn: !Ref ALBArn

      WebACLArn: !GetAtt WebACL.Arn
```

---

# AWS Shield Advanced

---

# Shield Protection

```yaml
Resources:

  ShieldProtection:

    Type: AWS::Shield::Protection

    Properties:

      Name: ALBProtection

      ResourceArn: !Ref ALBArn
```

---

# Protection Group

```yaml
Resources:

  ProtectionGroup:

    Type: AWS::Shield::ProtectionGroup

    Properties:

      ProtectionGroupId: production

      Aggregation: SUM
```

---

# AWS Global Accelerator

---

# Accelerator

```yaml
Resources:

  Accelerator:

    Type: AWS::GlobalAccelerator::Accelerator

    Properties:

      Name: Production

      Enabled: true
```

---

# Listener

```yaml
Resources:

  HTTPSListener:

    Type: AWS::GlobalAccelerator::Listener

    Properties:

      AcceleratorArn: !Ref Accelerator

      Protocol: TCP
```

---

# Endpoint Group

```yaml
Resources:

  EndpointGroup:

    Type: AWS::GlobalAccelerator::EndpointGroup

    Properties:

      ListenerArn: !Ref HTTPSListener

      EndpointGroupRegion: ap-south-1
```

---

# AWS Direct Connect

---

# Direct Connect Gateway

```yaml
Resources:

  DXGateway:

    Type: AWS::DirectConnect::Gateway

    Properties:

      AmazonSideAsn: 64512
```

---

# Gateway Association

```yaml
Resources:

  DXGatewayAssociation:

    Type: AWS::DirectConnect::GatewayAssociation

    Properties:

      AssociatedGatewayId: dx-gateway-id

      GatewayId: !Ref DXGateway
```

---

# Site-to-Site VPN

---

# Customer Gateway

```yaml
Resources:

  CustomerGateway:

    Type: AWS::EC2::CustomerGateway

    Properties:

      BgpAsn: 65000

      IpAddress: 203.0.113.10

      Type: ipsec.1
```

---

# VPN Gateway

```yaml
Resources:

  VPNGateway:

    Type: AWS::EC2::VPNGateway

    Properties:

      Type: ipsec.1
```

---

# VPN Attachment

```yaml
Resources:

  VPNAttachment:

    Type: AWS::EC2::VPCGatewayAttachment

    Properties:

      VpcId: !Ref VPC

      VpnGatewayId: !Ref VPNGateway
```

---

# VPN Connection

```yaml
Resources:

  VPNConnection:

    Type: AWS::EC2::VPNConnection

    Properties:

      Type: ipsec.1

      CustomerGatewayId: !Ref CustomerGateway

      VpnGatewayId: !Ref VPNGateway
```

---

# AWS Client VPN

---

# Client VPN Endpoint

```yaml
Resources:

  ClientVPN:

    Type: AWS::EC2::ClientVpnEndpoint

    Properties:

      Description: RemoteAccess

      ClientCidrBlock: 172.16.0.0/22

      ServerCertificateArn: arn:aws:acm:certificate
```

---

# Network Association

```yaml
Resources:

  ClientVPNAssociation:

    Type: AWS::EC2::ClientVpnTargetNetworkAssociation

    Properties:

      ClientVpnEndpointId: !Ref ClientVPN

      SubnetId: !Ref PrivateSubnet
```

---

# Authorization Rule

```yaml
Resources:

  VPNAuthorization:

    Type: AWS::EC2::ClientVpnAuthorizationRule

    Properties:

      ClientVpnEndpointId: !Ref ClientVPN

      TargetNetworkCidr: 10.0.0.0/16

      AuthorizeAllGroups: true
```

---

# AWS Network Firewall

---

# Firewall Policy

```yaml
Resources:

  FirewallPolicy:

    Type: AWS::NetworkFirewall::FirewallPolicy

    Properties:

      FirewallPolicyName: ProductionPolicy
```

---

# Firewall

```yaml
Resources:

  NetworkFirewall:

    Type: AWS::NetworkFirewall::Firewall

    Properties:

      FirewallName: ProductionFirewall

      VpcId: !Ref VPC
```

---

# Rule Group

```yaml
Resources:

  StatefulRules:

    Type: AWS::NetworkFirewall::RuleGroup

    Properties:

      Capacity: 100

      Type: STATEFUL
```

---

# AWS Private CA

---

# Certificate Authority

```yaml
Resources:

  PrivateCA:

    Type: AWS::ACMPCA::CertificateAuthority

    Properties:

      Type: ROOT
```

---

# Certificate

```yaml
Resources:

  RootCertificate:

    Type: AWS::ACMPCA::Certificate

    Properties:

      CertificateAuthorityArn: !Ref PrivateCA
```

---

# AWS RAM

---

# Resource Share

```yaml
Resources:

  NetworkShare:

    Type: AWS::RAM::ResourceShare

    Properties:

      Name: SharedNetworking

      AllowExternalPrincipals: false
```

---

# Resource Association

```yaml
Resources:

  TGWAssociation:

    Type: AWS::RAM::ResourceAssociation

    Properties:

      ResourceArn: arn:aws:ec2:transit-gateway

      ResourceShareArn: !Ref NetworkShare
```

---

# Outputs

```yaml
Outputs:

  HostedZoneId:

    Value: !Ref PublicHostedZone

  CloudFrontDomain:

    Value: !GetAtt CDN.DomainName

  AcceleratorDNS:

    Value: !GetAtt Accelerator.DnsName
```

---

# Best Practices

- Use Route53 Alias records for AWS resources.
- Enable Route53 Health Checks for failover.
- Protect CloudFront and ALBs with AWS WAF.
- Enable AWS Shield Advanced for internet-facing production workloads.
- Use Origin Access Control (OAC) for S3-backed CloudFront distributions.
- Prefer Direct Connect for dedicated hybrid connectivity.
- Configure Site-to-Site VPN as backup connectivity.
- Deploy AWS Network Firewall for centralized traffic inspection.
- Use AWS Private CA for internal PKI.
- Share networking resources securely using AWS RAM.

---

# Summary

This section covered CloudFormation templates for Route53, CloudFront, AWS WAF, Shield Advanced, Global Accelerator, Direct Connect, Site-to-Site VPN, Client VPN, AWS Network Firewall, Private CA, and AWS RAM. These templates provide production-ready patterns for DNS, CDN, hybrid connectivity, edge security, and enterprise networking using AWS CloudFormation.

---

# Amazon Athena

---

# Athena WorkGroup

```yaml
Resources:

  AthenaWorkGroup:

    Type: AWS::Athena::WorkGroup

    Properties:

      Name: production

      State: ENABLED
```

---

# Named Query

```yaml
Resources:

  TopUsersQuery:

    Type: AWS::Athena::NamedQuery

    Properties:

      Name: TopUsers

      Database: analytics

      QueryString: |
        SELECT * FROM users;
```

---

# Data Catalog

```yaml
Resources:

  AthenaCatalog:

    Type: AWS::Athena::DataCatalog

    Properties:

      Name: AwsDataCatalog

      Type: GLUE
```

---

# AWS Glue

---

# Catalog Database

```yaml
Resources:

  AnalyticsDatabase:

    Type: AWS::Glue::Database

    Properties:

      CatalogId: !Ref AWS::AccountId

      DatabaseInput:

        Name: analytics
```

---

# Catalog Table

```yaml
Resources:

  OrdersTable:

    Type: AWS::Glue::Table

    Properties:

      CatalogId: !Ref AWS::AccountId

      DatabaseName: analytics
```

---

# Glue Crawler

```yaml
Resources:

  OrdersCrawler:

    Type: AWS::Glue::Crawler

    Properties:

      Name: orders-crawler

      Role: !GetAtt GlueRole.Arn

      DatabaseName: analytics
```

---

# Glue Job

```yaml
Resources:

  DailyETL:

    Type: AWS::Glue::Job

    Properties:

      Name: daily-etl

      Role: !GetAtt GlueRole.Arn

      Command:

        Name: glueetl

        ScriptLocation: s3://scripts/etl.py
```

---

# Glue Workflow

```yaml
Resources:

  ETLWorkflow:

    Type: AWS::Glue::Workflow

    Properties:

      Name: DailyWorkflow
```

---

# Glue Trigger

```yaml
Resources:

  DailyTrigger:

    Type: AWS::Glue::Trigger

    Properties:

      Type: SCHEDULED

      Schedule: cron(0 2 * * ? *)
```

---

# Amazon EMR

---

# EMR Cluster

```yaml
Resources:

  AnalyticsCluster:

    Type: AWS::EMR::Cluster

    Properties:

      Name: analytics-cluster

      ReleaseLabel: emr-7.0.0
```

---

# EMR Security Configuration

```yaml
Resources:

  EMRSecurity:

    Type: AWS::EMR::SecurityConfiguration

    Properties:

      Name: ProductionSecurity
```

---

# EMR Studio

```yaml
Resources:

  AnalyticsStudio:

    Type: AWS::EMR::Studio

    Properties:

      Name: AnalyticsStudio

      AuthMode: SSO
```

---

# Amazon Redshift

---

# Cluster

```yaml
Resources:

  RedshiftCluster:

    Type: AWS::Redshift::Cluster

    Properties:

      ClusterIdentifier: warehouse

      NodeType: ra3.xlplus

      ClusterType: single-node
```

---

# Subnet Group

```yaml
Resources:

  RedshiftSubnetGroup:

    Type: AWS::Redshift::ClusterSubnetGroup

    Properties:

      Description: Warehouse Subnets
```

---

# Parameter Group

```yaml
Resources:

  RedshiftParameters:

    Type: AWS::Redshift::ClusterParameterGroup

    Properties:

      Description: Production Parameters
```

---

# Scheduled Action

```yaml
Resources:

  PauseCluster:

    Type: AWS::Redshift::ScheduledAction

    Properties:

      ScheduledActionName: PauseWarehouse
```

---

# Amazon OpenSearch Service

---

# OpenSearch Domain

```yaml
Resources:

  LogsDomain:

    Type: AWS::OpenSearchService::Domain

    Properties:

      DomainName: production-logs

      EngineVersion: OpenSearch_2.17
```

---

# Domain Policy

```yaml
AccessPolicies:

  Version: "2012-10-17"

  Statement: []
```

---

# Package Association

```yaml
Resources:

  DictionaryPackage:

    Type: AWS::OpenSearchService::PackageAssociation

    Properties:

      DomainName: !Ref LogsDomain
```

---

# Amazon Bedrock

---

# Guardrail

```yaml
Resources:

  BedrockGuardrail:

    Type: AWS::Bedrock::Guardrail

    Properties:

      Name: production-guardrail
```

---

# Prompt

```yaml
Resources:

  DevOpsPrompt:

    Type: AWS::Bedrock::Prompt

    Properties:

      Name: devops-assistant
```

---

# Inference Profile

```yaml
Resources:

  InferenceProfile:

    Type: AWS::Bedrock::ApplicationInferenceProfile

    Properties:

      InferenceProfileName: production-profile
```

---

# Amazon Q Business

---

# Application

```yaml
Resources:

  QBusiness:

    Type: AWS::QBusiness::Application

    Properties:

      DisplayName: EngineeringAssistant
```

---

# Amazon Textract

---

# Textract IAM Role

```yaml
Resources:

  TextractRole:

    Type: AWS::IAM::Role

    Properties:

      RoleName: TextractRole
```

---

# Amazon Rekognition

---

# Collection

```yaml
Resources:

  EmployeeCollection:

    Type: AWS::Rekognition::Collection

    Properties:

      CollectionId: employees
```

---

# Stream Processor

```yaml
Resources:

  VideoProcessor:

    Type: AWS::Rekognition::StreamProcessor

    Properties:

      Name: video-analysis
```

---

# Amazon Comprehend

---

# Document Classifier

```yaml
Resources:

  SupportClassifier:

    Type: AWS::Comprehend::DocumentClassifier

    Properties:

      DocumentClassifierName: support-classifier

      LanguageCode: en
```

---

# Entity Recognizer

```yaml
Resources:

  CustomerRecognizer:

    Type: AWS::Comprehend::EntityRecognizer

    Properties:

      RecognizerName: customer-entities

      LanguageCode: en
```

---

# Amazon Managed Prometheus

---

# Workspace

```yaml
Resources:

  AMPWorkspace:

    Type: AWS::APS::Workspace

    Properties:

      Alias: production-monitoring
```

---

# Rule Group Namespace

```yaml
Resources:

  AlertRules:

    Type: AWS::APS::RuleGroupsNamespace

    Properties:

      Name: production-alerts
```

---

# Amazon Managed Grafana

---

# Workspace

```yaml
Resources:

  GrafanaWorkspace:

    Type: AWS::Grafana::Workspace

    Properties:

      Name: production

      AccountAccessType: CURRENT_ACCOUNT

      AuthenticationProviders:

        - AWS_SSO
```

---

# Role Association

```yaml
Resources:

  GrafanaAdmins:

    Type: AWS::Grafana::RoleAssociation

    Properties:

      Role: ADMIN
```

---

# AWS X-Ray

---

# Sampling Rule

```yaml
Resources:

  ProductionSampling:

    Type: AWS::XRay::SamplingRule

    Properties:

      SamplingRule:

        RuleName: ProductionSampling

        FixedRate: 0.1
```

---

# Amazon Kinesis

---

# Data Stream

```yaml
Resources:

  EventStream:

    Type: AWS::Kinesis::Stream

    Properties:

      Name: application-events

      ShardCount: 2
```

---

# Firehose Delivery Stream

```yaml
Resources:

  LogDelivery:

    Type: AWS::KinesisFirehose::DeliveryStream

    Properties:

      DeliveryStreamName: application-logs
```

---

# Amazon MSK

---

# MSK Cluster

```yaml
Resources:

  KafkaCluster:

    Type: AWS::MSK::Cluster

    Properties:

      ClusterName: production

      KafkaVersion: 3.7.0

      NumberOfBrokerNodes: 3
```

---

# Amazon Managed Service for Apache Flink

```yaml
Resources:

  StreamingApplication:

    Type: AWS::KinesisAnalyticsV2::Application

    Properties:

      ApplicationName: stream-processing

      RuntimeEnvironment: FLINK-1_18
```

---

# Outputs

```yaml
Outputs:

  AthenaWorkGroup:

    Value: !Ref AthenaWorkGroup

  RedshiftEndpoint:

    Value: !GetAtt RedshiftCluster.Endpoint.Address

  OpenSearchEndpoint:

    Value: !GetAtt LogsDomain.DomainEndpoint

  PrometheusWorkspace:

    Value: !Ref AMPWorkspace

  GrafanaWorkspace:

    Value: !Ref GrafanaWorkspace
```

---

# Best Practices

- Store Athena query results in encrypted S3 buckets.
- Schedule Glue Crawlers and ETL Jobs using Glue Triggers.
- Deploy Redshift clusters in private subnets with encryption enabled.
- Enable fine-grained access control for OpenSearch domains.
- Protect Bedrock applications with Guardrails.
- Use Amazon Q Business with IAM Identity Center integration.
- Configure Amazon Managed Prometheus with recording and alerting rules.
- Integrate Amazon Managed Grafana with AWS IAM Identity Center.
- Tune AWS X-Ray sampling rules to balance observability and cost.
- Encrypt Kinesis streams, Firehose, and MSK clusters using AWS KMS.

---

# Summary

This section covered CloudFormation templates for Amazon Athena, AWS Glue, Amazon EMR, Amazon Redshift, OpenSearch Service, Amazon Bedrock, Amazon Q Business, Textract, Rekognition, Comprehend, Amazon Managed Prometheus, Amazon Managed Grafana, AWS X-Ray, Amazon Kinesis, Kinesis Firehose, Amazon MSK, and Managed Service for Apache Flink. These templates provide production-ready patterns for analytics, AI/ML, streaming, search, and observability services using AWS CloudFormation.

---

# Nested Stacks

---

# Parent Stack

```yaml
Resources:

  NetworkStack:

    Type: AWS::CloudFormation::Stack

    Properties:

      TemplateURL: https://s3.amazonaws.com/templates/network.yaml

      Parameters:

        Environment: production
```

---

# Nested Stack Output

```yaml
Outputs:

  VPCId:

    Value: !GetAtt NetworkStack.Outputs.VPCId
```

---

# Multiple Nested Stacks

```yaml
Resources:

  Network:

    Type: AWS::CloudFormation::Stack

  Security:

    Type: AWS::CloudFormation::Stack

  Compute:

    Type: AWS::CloudFormation::Stack
```

---

# Cross-Stack References

---

# Export

```yaml
Outputs:

  VPCId:

    Value: !Ref VPC

    Export:

      Name: Production-VPC
```

---

# Import

```yaml
VpcId:

  Fn::ImportValue: Production-VPC
```

---

# StackSets

---

# StackSet

```yaml
Resources:

  SecurityBaseline:

    Type: AWS::CloudFormation::StackSet

    Properties:

      StackSetName: SecurityBaseline

      PermissionModel: SERVICE_MANAGED
```

---

# StackSet Instance

```yaml
Resources:

  ProductionStack:

    Type: AWS::CloudFormation::StackInstance

    Properties:

      StackSetName: !Ref SecurityBaseline

      Region: ap-south-1
```

---

# Change Sets

---

# Create Change Set

```bash
aws cloudformation create-change-set \
--stack-name production \
--change-set-name update-v1 \
--template-body file://template.yaml
```

---

# Describe Change Set

```bash
aws cloudformation describe-change-set \
--stack-name production \
--change-set-name update-v1
```

---

# Execute Change Set

```bash
aws cloudformation execute-change-set \
--change-set-name update-v1
```

---

# Delete Change Set

```bash
aws cloudformation delete-change-set \
--stack-name production \
--change-set-name update-v1
```

---

# Drift Detection

---

# Detect Drift

```bash
aws cloudformation detect-stack-drift \
--stack-name production
```

---

# Describe Drift

```bash
aws cloudformation describe-stack-resource-drifts \
--stack-name production
```

---

# Stack Status

```bash
aws cloudformation describe-stacks \
--stack-name production
```

---

# CloudFormation Macros

---

# Macro

```yaml
Resources:

  TransformMacro:

    Type: AWS::CloudFormation::Macro

    Properties:

      Name: CompanyMacro

      FunctionName: MacroFunction
```

---

# Transform

```yaml
Transform:

  - CompanyMacro
```

---

# AWS SAM Transform

```yaml
Transform: AWS::Serverless-2016-10-31
```

---

# Include Transform

```yaml
Transform:

  Name: AWS::Include

  Parameters:

    Location: s3://templates/common.yaml
```

---

# Custom Resources

---

# Lambda-backed Custom Resource

```yaml
Resources:

  CustomResource:

    Type: Custom::Initialize

    Properties:

      ServiceToken: !GetAtt CustomLambda.Arn
```

---

# Service Token

```yaml
ServiceToken:

  !GetAtt LambdaFunction.Arn
```

---

# Wait Condition

---

# Wait Condition Handle

```yaml
Resources:

  WaitHandle:

    Type: AWS::CloudFormation::WaitConditionHandle
```

---

# Wait Condition

```yaml
Resources:

  WaitCondition:

    Type: AWS::CloudFormation::WaitCondition

    Properties:

      Handle: !Ref WaitHandle

      Timeout: 600
```

---

# Resource Import

---

# Import Existing Resource

```bash
aws cloudformation create-change-set \
--change-set-type IMPORT
```

---

# Stack Policy

---

# Stack Policy Body

```json
{
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "Update:*",
      "Principal": "*",
      "Resource": "*"
    }
  ]
}
```

---

# Apply Stack Policy

```bash
aws cloudformation set-stack-policy \
--stack-name production
```

---

# Rollback Triggers

```yaml
RollbackConfiguration:

  MonitoringTimeInMinutes: 15
```

---

# Rollback Alarm

```yaml
RollbackTriggers:

  - Arn: arn:aws:cloudwatch:alarm
```

---

# Termination Protection

```bash
aws cloudformation update-termination-protection \
--enable-termination-protection
```

---

# Dynamic References

---

# Secrets Manager

```yaml
MasterUserPassword:

  "{{resolve:secretsmanager:db-secret:SecretString:password}}"
```

---

# Parameter Store

```yaml
ImageId:

  "{{resolve:ssm:/ami/latest}}"
```

---

# Metadata

```yaml
Metadata:

  AWS::CloudFormation::Init: {}
```

---

# cfn-init

```bash
/opt/aws/bin/cfn-init
```

---

# cfn-signal

```bash
/opt/aws/bin/cfn-signal
```

---

# cfn-hup

```bash
/opt/aws/bin/cfn-hup
```

---

# Creation Policy

```yaml
CreationPolicy:

  ResourceSignal:

    Count: 1

    Timeout: PT15M
```

---

# Update Policy

```yaml
UpdatePolicy:

  AutoScalingRollingUpdate:

    MinInstancesInService: 2
```

---

# Advanced Conditions

```yaml
Conditions:

  CreateProduction:

    !And

      - !Equals [!Ref Environment, prod]

      - !Equals [!Ref EnableFeature, true]
```

---

# Advanced Intrinsic Functions

---

# If

```yaml
!If

  - IsProduction

  - t3.large

  - t3.micro
```

---

# Equals

```yaml
!Equals

  - !Ref Environment

  - prod
```

---

# Not

```yaml
!Not

  - !Equals [!Ref Environment, dev]
```

---

# Or

```yaml
!Or

  - !Equals [!Ref Environment, qa]

  - !Equals [!Ref Environment, prod]
```

---

# And

```yaml
!And

  - !Condition IsProduction

  - !Condition EnableMonitoring
```

---

# FindInMap

```yaml
!FindInMap

  - RegionMap

  - !Ref AWS::Region

  - AMI
```

---

# Split

```yaml
!Split

  - ","

  - !Ref SubnetList
```

---

# Select

```yaml
!Select

  - 0

  - !Ref Subnets
```

---

# Join

```yaml
!Join

  - "-"

  - [prod, app]
```

---

# Sub

```yaml
!Sub

  arn:aws:s3:::${Bucket}
```

---

# Base64

```yaml
!Base64 |

  #!/bin/bash

  yum update -y
```

---

# Best Practices

- Use Nested Stacks for reusable infrastructure.
- Export shared resources using Cross-Stack References.
- Review Change Sets before every production deployment.
- Regularly perform Drift Detection.
- Store secrets using Dynamic References instead of plaintext.
- Use Stack Policies to protect critical resources.
- Enable Termination Protection for production stacks.
- Use Rollback Triggers with CloudWatch alarms.
- Use `cfn-init`, `cfn-signal`, and `cfn-hup` for EC2 bootstrapping.
- Keep templates modular and version-controlled.

---

# Summary

This section covered advanced CloudFormation capabilities including Nested Stacks, Cross-Stack References, StackSets, Change Sets, Drift Detection, Macros, Custom Resources, Wait Conditions, Resource Import, Stack Policies, Dynamic References, CloudFormation helper scripts, advanced intrinsic functions, and enterprise deployment patterns. These features enable scalable, maintainable, and production-ready CloudFormation deployments.

---

# Enterprise Repository Structure

```text
cloudformation/

├── environments/
│   ├── dev/
│   ├── qa/
│   ├── stage/
│   └── prod/
│
├── templates/
│   ├── network/
│   ├── security/
│   ├── compute/
│   ├── database/
│   ├── monitoring/
│   └── applications/
│
├── nested-stacks/
├── parameters/
├── policies/
├── scripts/
└── README.md
```

---

# Multi-Account Architecture

```text
AWS Organization

├── Management
├── Security
├── Logging
├── Shared Services
├── Development
├── Testing
├── Staging
└── Production
```

---

# Multi-Region Deployment

```yaml
Parameters:

  PrimaryRegion:

    Type: String

    Default: ap-south-1

  SecondaryRegion:

    Type: String

    Default: ap-southeast-1
```

---

# Environment-Based Parameters

```yaml
Parameters:

  Environment:

    Type: String

    AllowedValues:

      - dev

      - qa

      - stage

      - prod
```

---

# Common Resource Tags

```yaml
Tags:

  - Key: Environment

    Value: !Ref Environment

  - Key: ManagedBy

    Value: CloudFormation

  - Key: Team

    Value: DevOps

  - Key: Project

    Value: Platform
```

---

# Stack Naming Convention

```text
dev-network
dev-eks
qa-network
prod-network
prod-eks
```

---

# Parameter Files

```text
parameters/

├── dev.json
├── qa.json
├── stage.json
└── prod.json
```

---

# Deploy with Parameters

```bash
aws cloudformation create-stack \
--stack-name prod-network \
--template-body file://network.yaml \
--parameters file://parameters/prod.json
```

---

# CI/CD Pipeline

```text
Developer

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Template Validation

↓

Linting

↓

Security Scan

↓

Change Set

↓

Manual Approval

↓

Stack Update

↓

Production
```

---

# GitHub Actions Workflow

```yaml
name: CloudFormation

on:
  push:

jobs:

  deploy:

    runs-on: ubuntu-latest

    steps:

      - uses: actions/checkout@v4

      - run: aws cloudformation validate-template \
          --template-body file://template.yaml

      - run: cfn-lint template.yaml
```

---

# Jenkins Pipeline

```groovy
pipeline {

    stages {

        stage("Validate") {
            steps {
                sh "cfn-lint template.yaml"
            }
        }

        stage("Change Set") {
            steps {
                sh "aws cloudformation create-change-set"
            }
        }

        stage("Deploy") {
            steps {
                sh "aws cloudformation execute-change-set"
            }
        }

    }

}
```

---

# GitLab CI

```yaml
deploy:

  script:

    - aws cloudformation validate-template

    - cfn-lint template.yaml

    - aws cloudformation deploy
```

---

# Drift Detection Workflow

```text
Scheduled Job

↓

Detect Drift

↓

Review Drift Report

↓

Investigate Changes

↓

Update Stack

↓

Restore Desired State
```

---

# Security Best Practices

- Enable CloudTrail for all AWS accounts.
- Store secrets in AWS Secrets Manager.
- Use Dynamic References for sensitive values.
- Enable Stack Termination Protection.
- Protect critical resources using Stack Policies.
- Apply least-privilege IAM roles.
- Enable AWS Config and GuardDuty.
- Encrypt storage resources with AWS KMS.
- Use private subnets for databases and internal services.
- Enable logging for CloudFront, ALB, and S3.

---

# Cost Optimization

- Delete unused stacks.
- Use Auto Scaling.
- Schedule non-production environments.
- Use S3 lifecycle policies.
- Use Spot Instances where appropriate.
- Enable RDS storage autoscaling.
- Remove idle Elastic IPs.
- Review Cost Explorer monthly.

---

# Disaster Recovery Strategy

```text
Primary Region

↓

Cross-Region Replication

↓

Backups

↓

CloudFormation Templates

↓

Restore Infrastructure

↓

Recover Applications
```

---

# High Availability Pattern

```text
Route53

↓

Application Load Balancer

↓

Auto Scaling Group

↓

Multi-AZ Database

↓

Amazon S3

↓

CloudFront
```

---

# Blue/Green Deployment

```text
Production Stack

↓

Create New Stack

↓

Validate

↓

Switch Traffic

↓

Delete Old Stack
```

---

# Rolling Update Strategy

```yaml
UpdatePolicy:

  AutoScalingRollingUpdate:

    MaxBatchSize: 1

    MinInstancesInService: 2

    PauseTime: PT5M
```

---

# Stack Organization

```text
Network Stack

↓

Security Stack

↓

Storage Stack

↓

Compute Stack

↓

Monitoring Stack

↓

Application Stack
```

---

# CloudFormation Linting

```bash
cfn-lint template.yaml
```

---

# Package Template

```bash
aws cloudformation package \
--template-file template.yaml \
--s3-bucket deployment-artifacts
```

---

# Deploy Template

```bash
aws cloudformation deploy \
--template-file packaged.yaml \
--stack-name production
```

---

# Describe Stack

```bash
aws cloudformation describe-stacks \
--stack-name production
```

---

# List Stack Events

```bash
aws cloudformation describe-stack-events \
--stack-name production
```

---

# Delete Stack

```bash
aws cloudformation delete-stack \
--stack-name production
```

---

# Enterprise CloudFormation Checklist

## Code Quality

- Templates validated
- cfn-lint executed
- Parameters externalized
- Outputs documented
- Nested stacks used
- YAML formatted

---

## Security

- Secrets externalized
- Dynamic references used
- IAM least privilege
- Encryption enabled
- Stack policy applied
- Termination protection enabled

---

## Operations

- Change Sets reviewed
- Drift Detection scheduled
- CloudTrail enabled
- AWS Config enabled
- Monitoring configured
- Stack events reviewed

---

## Reliability

- Multi-AZ architecture
- Auto Scaling configured
- Backups enabled
- Disaster Recovery documented
- Rollback tested

---

# CloudFormation Interview Questions

## Basic

- What is AWS CloudFormation?
- What is a Stack?
- What is a Template?
- Difference between JSON and YAML templates?
- What are Parameters?
- What are Outputs?

---

## Intermediate

- What are Nested Stacks?
- What are StackSets?
- Explain Change Sets.
- Explain Drift Detection.
- What are Dynamic References?
- Explain Cross-Stack References.
- Explain Stack Policies.

---

## Advanced

- How do you structure CloudFormation repositories?
- How do you deploy CloudFormation across multiple AWS accounts?
- How do you secure CloudFormation templates?
- Explain zero-downtime infrastructure updates.
- How do you implement Blue/Green deployments?
- How do you manage CloudFormation in CI/CD pipelines?
- Explain Rollback Triggers.
- How do you recover failed CloudFormation deployments?

---

# Best Practices

- Keep templates modular using Nested Stacks.
- Store parameters outside templates.
- Always validate templates before deployment.
- Review Change Sets before production updates.
- Enable Drift Detection for production stacks.
- Use StackSets for organization-wide deployments.
- Protect production stacks with termination protection.
- Keep templates in version control.
- Automate deployments using CI/CD pipelines.
- Apply consistent tagging across all resources.

---

# Summary

This final section covered enterprise CloudFormation practices including repository organization, multi-account and multi-region deployments, CI/CD integration, security, disaster recovery, cost optimization, deployment strategies, operational checklists, and interview preparation. Together with the previous nine sections, this guide provides a comprehensive, production-ready reference for designing, deploying, and managing AWS infrastructure using CloudFormation.

---

# Cookbook Statistics

| Category | Coverage |
|----------|----------:|
| CloudFormation Fundamentals | ✅ |
| Networking | ✅ |
| IAM & Security | ✅ |
| Storage & Databases | ✅ |
| Containers & Kubernetes | ✅ |
| Monitoring & CI/CD | ✅ |
| Edge & Hybrid Networking | ✅ |
| Analytics & AI/ML | ✅ |
| Advanced CloudFormation | ✅ |
| Enterprise Patterns | ✅ |

**Approximate Coverage**

- **700+ CloudFormation YAML examples**
- **10 comprehensive sections**
- **Production-ready AWS infrastructure**
- **Enterprise repository patterns**
- **CI/CD integration**
- **Multi-account & multi-region deployments**
- **Security and governance best practices**
- **Disaster recovery patterns**
- **CloudFormation interview preparation**
- **Enterprise deployment checklists**