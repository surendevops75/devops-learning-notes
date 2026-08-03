# AWS Production Scenarios

---

# Introduction

Production environments are significantly different from development environments. While AWS services are important to understand, organizations primarily evaluate engineers on their ability to troubleshoot incidents, identify root causes, restore services quickly, and prevent future failures.

This guide contains real-world AWS and DevOps production scenarios commonly encountered in enterprise environments and technical interviews.

Each scenario includes

- Problem Statement
- Symptoms
- Possible Causes
- Investigation Steps
- Root Cause
- Resolution
- Prevention
- Interview Tips

---

# Production Incident Workflow

Always follow a structured troubleshooting approach.

```text
Incident Report

↓

Identify Impact

↓

Collect Logs

↓

Check Monitoring

↓

Identify Root Cause

↓

Fix Issue

↓

Validate Solution

↓

Document RCA

↓

Prevent Recurrence
```

---

# Scenario 1

# EC2 Instance Not Reachable

---

## Problem

Users cannot access the application hosted on an EC2 instance.

---

## Symptoms

- Website unavailable
- SSH connection timeout
- Health checks failing
- ALB marks instance unhealthy

---

## Possible Causes

- EC2 stopped
- Security Group blocked
- NACL blocked
- Route Table issue
- Instance crashed
- Disk full
- High CPU
- SSH daemon stopped

---

## Investigation

```bash
aws ec2 describe-instance-status
```

Check

- Instance State
- System Status Check
- Instance Status Check

Verify

```bash
ping

ssh

curl
```

CloudWatch

- CPU
- Memory
- Network
- Disk

---

## Root Cause Example

Root filesystem reached 100%.

Applications stopped responding.

---

## Resolution

```bash
df -h

du -sh *

rm unnecessary files
```

Restart services.

---

## Prevention

- CloudWatch Alarms
- Disk Monitoring
- Log Rotation
- Auto Recovery

---

## Interview Tip

Always investigate

- EC2
- Security Group
- Route Table
- NACL
- Application
- Operating System

Never assume the issue is with AWS first.

---

# Scenario 2

# Application Load Balancer Returns 503

---

## Symptoms

- HTTP 503
- Health checks failing
- Application unavailable

---

## Possible Causes

- No healthy targets
- Health check failure
- Wrong Target Group
- Application crash
- Wrong port

---

## Investigation

Check

```text
Target Group

↓

Registered Targets

↓

Health Checks

↓

Application Logs
```

Commands

```bash
curl localhost:8080

systemctl status nginx

systemctl status application
```

---

## Root Cause

Application listening on port 8081

Target Group configured for 8080

---

## Resolution

Correct Target Group

OR

Modify application port

---

## Prevention

- Standard Port Configuration
- Health Check Validation
- Deployment Testing

---

# Scenario 3

# Auto Scaling Not Launching New Instances

---

## Symptoms

CPU above 90%

Only existing instances running

Application slow

---

## Possible Causes

- Scaling Policy
- Launch Template
- Instance Limits
- Capacity Issues

---

## Investigation

Check

```text
Auto Scaling Group

↓

Desired Capacity

↓

Launch Template

↓

Scaling Policy

↓

Activity History
```

---

## Root Cause

Desired Capacity = Max Capacity

Scaling impossible

---

## Resolution

Increase

```text
Max Size

Desired Capacity
```

---

## Prevention

Review scaling policies regularly.

---

# Scenario 4

# RDS Database Connection Timeout

---

## Symptoms

Application

↓

Database Timeout

↓

API Failure

---

## Possible Causes

- Security Group
- Database Down
- Max Connections
- Slow Queries

---

## Investigation

CloudWatch

- CPU
- Connections
- Free Storage

Check

```sql
SHOW PROCESSLIST;
```

---

## Root Cause

Connection pool exhausted.

---

## Resolution

- Restart application
- Tune connection pool
- Kill idle sessions

---

## Prevention

Use

- Connection Pooling
- Read Replicas
- Performance Insights

---

# Scenario 5

# S3 Access Denied

---

## Symptoms

Application cannot upload files.

---

## Possible Causes

- IAM Policy
- Bucket Policy
- KMS Permissions
- ACL
- SCP

---

## Investigation

Verify

```bash
aws s3 ls
```

Review

- IAM
- Bucket Policy
- KMS
- CloudTrail

---

## Root Cause

Bucket policy denied PutObject.

---

## Resolution

Update Bucket Policy.

---

## Prevention

Use IAM least privilege.

---

# Scenario 6

# Lambda Function Timing Out

---

## Symptoms

Lambda

↓

Timeout

↓

No Response

---

## Possible Causes

- Infinite Loop
- Database Delay
- External API Delay
- Memory Too Low

---

## Investigation

CloudWatch Logs

Duration

Timeout

Memory Usage

---

## Root Cause

Database query taking 40 seconds.

Lambda timeout

15 seconds.

---

## Resolution

Increase timeout.

Optimize database query.

---

## Prevention

Monitor duration.

Optimize dependencies.

---

# Scenario 7

# EBS Volume Full

---

## Symptoms

Application crashes

Cannot write logs

Filesystem full

---

## Investigation

```bash
df -h

du -sh *

lsblk
```

---

## Root Cause

Application logs consumed all storage.

---

## Resolution

Delete logs.

Expand EBS.

Extend filesystem.

---

## Prevention

Log rotation.

Disk alarms.

---

# Scenario 8

# Route53 DNS Not Resolving

---

## Symptoms

Website unavailable.

---

## Investigation

Check

- Hosted Zone
- Record
- TTL
- Name Servers

Commands

```bash
nslookup

dig
```

---

## Root Cause

Wrong Alias Record.

---

## Resolution

Correct Route53 record.

---

## Prevention

DNS validation before deployment.

---

# Scenario 9

# CloudFront Serving Old Content

---

## Symptoms

Updated website

Old content displayed

---

## Root Cause

Cached objects.

---

## Resolution

Invalidate

```text
/*
```

---

## Prevention

Use versioned static assets.

---

# Scenario 10

# IAM Permission Denied

---

## Symptoms

Application

↓

AccessDeniedException

---

## Investigation

Review

- IAM Policy
- SCP
- Permission Boundary
- Resource Policy

---

## Root Cause

Missing

```json
s3:GetObject
```

Permission.

---

## Resolution

Update IAM policy.

---

## Prevention

Follow least privilege with proper testing.

---

# Summary

These first ten scenarios represent some of the most common production issues involving EC2, ALB, Auto Scaling, RDS, S3, Lambda, EBS, Route 53, CloudFront, and IAM. A structured troubleshooting approach—checking monitoring data, logs, networking, permissions, and application health—is essential for quickly identifying root causes and restoring services.

---

# Scenario 11

# VPC Connectivity Issue

---

## Problem

EC2 instances cannot communicate with other AWS resources.

---

## Symptoms

- SSH timeout
- API timeout
- Database unreachable
- Internet inaccessible

---

## Possible Causes

- Route Table
- Security Group
- NACL
- Internet Gateway
- NAT Gateway
- DNS

---

## Investigation

Check

```text
VPC

↓

Subnet

↓

Route Table

↓

Security Group

↓

NACL

↓

Internet Gateway
```

Commands

```bash
ping

traceroute

curl
```

---

## Root Cause

Private subnet missing default route.

---

## Resolution

Add

```text
0.0.0.0/0

↓

NAT Gateway
```

---

## Prevention

Review VPC architecture before deployment.

---

# Scenario 12

# Internet Gateway Not Working

---

## Symptoms

Public EC2 cannot access Internet.

---

## Investigation

Check

- Internet Gateway attached
- Public Route Table
- Public IP
- Security Group

---

## Root Cause

No Internet Gateway attached to VPC.

---

## Resolution

Attach Internet Gateway.

Update Route Table.

---

## Prevention

Validate network architecture.

---

# Scenario 13

# NAT Gateway Failure

---

## Symptoms

Private instances cannot download packages.

```bash
yum update

apt update

docker pull
```

fail.

---

## Possible Causes

- NAT Gateway deleted
- Elastic IP missing
- Route Table
- Availability Zone issue

---

## Investigation

Check

```text
Private Route

↓

NAT Gateway

↓

Elastic IP

↓

Internet Gateway
```

---

## Root Cause

Route pointed to deleted NAT Gateway.

---

## Resolution

Create new NAT Gateway.

Update Route Table.

---

## Prevention

CloudWatch alarms.

Infrastructure as Code.

---

# Scenario 14

# Security Group Blocking Traffic

---

## Symptoms

Application unavailable.

SSH fails.

Database unreachable.

---

## Investigation

Review

Inbound Rules

Outbound Rules

Source

Destination

---

## Root Cause

Port

```text
8080
```

blocked.

---

## Resolution

Allow required ports.

---

## Prevention

Maintain standard Security Group templates.

---

# Scenario 15

# Network ACL Blocking Application

---

## Symptoms

Security Groups appear correct.

Traffic still blocked.

---

## Investigation

Review

Inbound Rule

Outbound Rule

Rule Priority

---

## Root Cause

NACL denied ephemeral ports.

---

## Resolution

Allow

```text
1024-65535
```

---

## Prevention

Understand stateless firewall behavior.

---

# Scenario 16

# Docker Container Keeps Restarting

---

## Symptoms

```bash
docker ps
```

shows

Restarting

---

## Investigation

```bash
docker logs

docker inspect

docker stats
```

---

## Possible Causes

- Application Crash
- Missing Environment Variable
- Port Conflict
- Out of Memory

---

## Root Cause

Database environment variable missing.

---

## Resolution

Update

```yaml
environment:
```

Restart container.

---

## Prevention

Validate environment variables before deployment.

---

# Scenario 17

# Docker Image Pull Failed

---

## Symptoms

```bash
docker pull
```

fails.

---

## Possible Causes

- Wrong Image Tag
- Authentication
- Network
- Repository Missing

---

## Investigation

```bash
docker login

docker images
```

---

## Root Cause

Image tag incorrect.

---

## Resolution

Use correct version.

---

## Prevention

Use immutable image tags.

---

# Scenario 18

# Amazon ECR Authentication Failed

---

## Symptoms

```bash
docker push
```

returns

Unauthorized

---

## Investigation

Verify

IAM

Repository

Login

---

## Resolution

```bash
aws ecr get-login-password

docker login
```

Retry push.

---

## Prevention

Automate authentication inside CI/CD.

---

# Scenario 19

# ECS Tasks Continuously Stopping

---

## Symptoms

Tasks

↓

Running

↓

Stopped

↓

Restart

---

## Investigation

Review

Task Definition

Container Logs

CloudWatch

Memory

CPU

---

## Root Cause

Container exceeded memory limit.

---

## Resolution

Increase task memory.

Optimize application.

---

## Prevention

Monitor resource utilization.

---

# Scenario 20

# EKS Pods Not Starting

---

## Symptoms

Pods remain

```text
Pending
```

---

## Investigation

```bash
kubectl get pods

kubectl describe pod
```

Check

- Scheduler
- Node Resources
- Taints
- PVC
- Events

---

## Possible Causes

- No Nodes
- Insufficient CPU
- Memory
- Taints
- Storage

---

## Root Cause

No worker nodes available.

---

## Resolution

Scale Node Group.

---

## Prevention

Cluster Autoscaler.

Node monitoring.

---

# Scenario 21

# Kubernetes CrashLoopBackOff

---

## Symptoms

```bash
kubectl get pods
```

shows

CrashLoopBackOff

---

## Investigation

```bash
kubectl logs

kubectl logs --previous

kubectl describe pod
```

---

## Possible Causes

- Wrong Environment Variables
- Database Down
- Application Bug
- Missing Secret
- Missing ConfigMap

---

## Root Cause

Secret not mounted.

---

## Resolution

Create Secret.

Restart deployment.

---

## Prevention

Validate manifests before deployment.

---

# Scenario 22

# ImagePullBackOff

---

## Symptoms

Pod cannot download image.

---

## Investigation

```bash
kubectl describe pod
```

Review

- Image Name
- Registry
- Pull Secret

---

## Root Cause

Incorrect image tag.

---

## Resolution

Update Deployment.

---

## Prevention

CI/CD validation.

---

# Scenario 23

# Kubernetes Service Not Accessible

---

## Symptoms

Pod running.

Application unreachable.

---

## Investigation

```bash
kubectl get svc

kubectl describe svc

kubectl get endpoints
```

---

## Root Cause

Wrong selector.

---

## Resolution

Update labels.

---

## Prevention

Follow consistent labeling strategy.

---

# Scenario 24

# Load Balancer Not Created in EKS

---

## Symptoms

Service

```yaml
type: LoadBalancer
```

No ELB created.

---

## Investigation

Check

AWS Load Balancer Controller

IAM Role

Subnet Tags

---

## Root Cause

Controller missing.

---

## Resolution

Install AWS Load Balancer Controller.

---

## Prevention

Validate EKS prerequisites.

---

# Scenario 25

# ALB Health Checks Failing

---

## Symptoms

Targets unhealthy.

---

## Investigation

Check

Application

Health Endpoint

Target Group

Security Group

---

## Root Cause

Health endpoint

```text
/health
```

returned

500.

---

## Resolution

Fix application health endpoint.

---

## Prevention

Standardize readiness endpoints.

---

# Summary

These production scenarios cover VPC networking, Internet Gateway, NAT Gateway, Security Groups, NACLs, Docker, Amazon ECR, Amazon ECS, Amazon EKS, Kubernetes Pods, Services, Load Balancers, and ALB health checks. These are among the most frequently encountered operational issues in enterprise AWS and Kubernetes environments and are commonly discussed in DevOps interviews.

---

# Scenario 26

# Jenkins Pipeline Failed

---

## Problem

CI/CD pipeline fails during execution.

---

## Symptoms

- Build Failed
- Pipeline Stopped
- Deployment Not Triggered
- Artifact Missing

---

## Possible Causes

- Git Authentication
- Build Error
- Missing Credentials
- Agent Offline
- Jenkins Plugin Issue

---

## Investigation

Check

```text
Pipeline Console Output

↓

Jenkins Agent

↓

Credentials

↓

Workspace

↓

Build Logs
```

Commands

```bash
systemctl status jenkins

journalctl -u jenkins

docker ps
```

---

## Root Cause

Expired Git credentials.

---

## Resolution

Update Jenkins credentials.

Re-run pipeline.

---

## Prevention

- Rotate credentials
- Use Secrets Manager
- Monitor Jenkins health

---

# Scenario 27

# GitHub Actions Workflow Failed

---

## Symptoms

Workflow stops at build stage.

---

## Investigation

Review

- Workflow Logs
- Secrets
- Permissions
- Runner Status

---

## Root Cause

Missing repository secret.

---

## Resolution

Add required secret.

Restart workflow.

---

## Prevention

Validate workflows before merging.

---

# Scenario 28

# Terraform Apply Failed

---

## Symptoms

```bash
terraform apply
```

returns error.

---

## Possible Causes

- State Lock
- IAM
- Syntax Error
- Resource Conflict
- API Limits

---

## Investigation

```bash
terraform validate

terraform plan

terraform state list
```

---

## Root Cause

State file locked.

---

## Resolution

```bash
terraform force-unlock
```

Retry apply.

---

## Prevention

Use remote backend with locking.

---

# Scenario 29

# Terraform Drift Detected

---

## Symptoms

Resources changed manually.

Terraform wants unexpected updates.

---

## Investigation

```bash
terraform plan
```

Compare

AWS

↓

Terraform State

---

## Root Cause

Manual AWS Console changes.

---

## Resolution

- Import resources
- Update code
- Remove manual changes

---

## Prevention

Infrastructure as Code only.

---

# Scenario 30

# CloudFormation Stack Failed

---

## Symptoms

Stack status

```text
ROLLBACK_COMPLETE
```

---

## Investigation

Review

Events

↓

Failed Resource

↓

CloudFormation Logs

---

## Root Cause

IAM permission missing.

---

## Resolution

Correct permissions.

Delete failed stack.

Redeploy.

---

## Prevention

Validate templates using

```bash
aws cloudformation validate-template
```

---

# Scenario 31

# Argo CD Application OutOfSync

---

## Symptoms

Application status

```text
OutOfSync
```

---

## Investigation

Check

Git Repository

↓

Manifest

↓

Cluster State

↓

ArgoCD Logs

---

## Root Cause

Manual kubectl changes.

---

## Resolution

```text
Sync Application
```

or

Restore Git state.

---

## Prevention

Never modify production manually.

Use GitOps.

---

# Scenario 32

# Helm Upgrade Failed

---

## Symptoms

```bash
helm upgrade
```

fails.

---

## Investigation

```bash
helm history

helm status

kubectl describe pods
```

---

## Root Cause

Invalid values.yaml

---

## Resolution

Fix configuration.

Rollback

```bash
helm rollback
```

---

## Prevention

Validate values before deployment.

---

# Scenario 33

# Prometheus Not Scraping Metrics

---

## Symptoms

No metrics.

Grafana dashboards empty.

---

## Investigation

Check

Targets

↓

ServiceMonitor

↓

Pod Labels

↓

Prometheus Logs

---

## Root Cause

Incorrect Service selector.

---

## Resolution

Update labels.

Restart Prometheus.

---

## Prevention

Standardize Kubernetes labels.

---

# Scenario 34

# Grafana Dashboard Shows No Data

---

## Symptoms

Dashboard empty.

---

## Investigation

Verify

Datasource

↓

Prometheus

↓

Query

↓

Time Range

---

## Root Cause

Datasource disconnected.

---

## Resolution

Reconnect datasource.

Test query.

---

## Prevention

Monitor datasource health.

---

# Scenario 35

# ELK Stack Not Receiving Logs

---

## Symptoms

Logs missing.

---

## Investigation

Check

Fluent Bit

↓

Logstash

↓

Elasticsearch

↓

Kibana

---

## Root Cause

Fluent Bit daemonset failed.

---

## Resolution

Restart Fluent Bit.

Verify configuration.

---

## Prevention

Monitor logging pipeline.

---

# Scenario 36

# Elasticsearch Cluster Red

---

## Symptoms

Cluster Health

```text
RED
```

---

## Investigation

Check

- Disk Space
- Shards
- Nodes
- Replicas

---

## Root Cause

Disk full.

---

## Resolution

Increase storage.

Delete old indices.

---

## Prevention

Enable Index Lifecycle Management.

---

# Scenario 37

# CloudWatch Alarm Never Triggers

---

## Symptoms

High CPU.

Alarm remains OK.

---

## Investigation

Check

Metric

↓

Namespace

↓

Threshold

↓

Evaluation Period

---

## Root Cause

Wrong metric selected.

---

## Resolution

Use correct CloudWatch metric.

---

## Prevention

Validate alarms after creation.

---

# Scenario 38

# CloudWatch Logs Missing

---

## Symptoms

No application logs.

---

## Investigation

Check

CloudWatch Agent

↓

IAM Role

↓

Log Group

↓

Network

---

## Root Cause

CloudWatch Agent stopped.

---

## Resolution

```bash
systemctl restart amazon-cloudwatch-agent
```

---

## Prevention

Monitor CloudWatch Agent.

---

# Scenario 39

# Route53 Failover Not Working

---

## Symptoms

Primary Region down.

Traffic not switching.

---

## Investigation

Review

Health Check

↓

DNS Record

↓

Routing Policy

---

## Root Cause

Health check misconfigured.

---

## Resolution

Correct health check.

Verify failover policy.

---

## Prevention

Regular DR testing.

---

# Scenario 40

# RDS Multi-AZ Failover

---

## Symptoms

Short database interruption.

---

## Investigation

CloudWatch

↓

RDS Events

↓

Application Logs

---

## Root Cause

Primary database unavailable.

Automatic failover initiated.

---

## Resolution

Wait for failover.

Reconnect application.

---

## Prevention

Implement retry logic.

Use connection pooling.

---

# Summary

These scenarios focus on CI/CD pipelines, Infrastructure as Code, GitOps, observability, logging, monitoring, Route 53 failover, and RDS high availability. They reflect common production incidents in enterprise DevOps environments and demonstrate how structured troubleshooting, automation, and monitoring reduce downtime and improve operational reliability.

---

