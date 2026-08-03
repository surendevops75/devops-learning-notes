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