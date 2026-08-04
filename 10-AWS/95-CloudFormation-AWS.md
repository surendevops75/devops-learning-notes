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

