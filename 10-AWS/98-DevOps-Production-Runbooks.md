# DevOps Production Runbooks

---

# Introduction

A Production Runbook is a documented, step-by-step operational guide used to detect, investigate, mitigate, recover, and document production incidents consistently and safely.

Objectives

- Reduce downtime
- Standardize troubleshooting
- Minimize human error
- Improve recovery time
- Capture operational knowledge

---

# Incident Severity Levels

## Severity 1 (Critical)

Examples

- Production outage
- Complete service downtime
- Database unavailable
- Security breach
- Revenue impact

Target Response

```text
Immediate
```

---

## Severity 2 (High)

Examples

- Partial production outage
- API failures
- High latency
- Node failures

Target Response

```text
Within 15 Minutes
```

---

## Severity 3 (Medium)

Examples

- Performance degradation
- Failed deployments
- Minor service disruption

Target Response

```text
Within 1 Hour
```

---

## Severity 4 (Low)

Examples

- Cosmetic issues
- Minor alerts
- Documentation updates

Target Response

```text
Business Hours
```

---

# Incident Response Lifecycle

```text
Alert

↓

Acknowledge

↓

Investigate

↓

Mitigate

↓

Recover

↓

Monitor

↓

Root Cause Analysis

↓

Postmortem
```

---

# Incident Ownership

Assign

- Incident Commander
- Primary Engineer
- Secondary Engineer
- Communication Lead
- Business Stakeholder

---

# Initial Incident Checklist

Immediately verify

- What failed?
- When did it fail?
- Which environment?
- Business impact?
- Affected users?
- Recent deployments?
- Infrastructure changes?
- Security events?

---

# Production Investigation Workflow

```text
Alert Received

↓

Identify Service

↓

Check Monitoring

↓

Review Logs

↓

Check Infrastructure

↓

Identify Root Cause

↓

Mitigate

↓

Recover
```

---

# Golden Rule

Never assume.

Always verify using

- Metrics
- Logs
- Events
- Monitoring
- Configuration

---

# First Five Minutes

Review

- Monitoring dashboards
- Alerts
- Recent deployments
- Infrastructure health
- Application logs

---

# Recent Changes Checklist

Check

- Jenkins deployment
- Git commits
- Kubernetes rollout
- Terraform apply
- Security Group changes
- DNS updates
- Certificate renewal
- Database migration

---

# Communication Workflow

```text
Incident

↓

Engineering Team

↓

Management

↓

Support Team

↓

Customers (If Required)
```

---

# Production Dashboard

Review

- CPU
- Memory
- Disk
- Network
- Error Rate
- Latency
- Availability
- Request Rate

---

# Basic Linux Health Check

```bash
uptime

free -h

df -h

top

vmstat

iostat

sar
```

---

# Network Verification

```bash
ping

traceroute

nslookup

dig

curl

netstat

ss
```

---

# Process Verification

```bash
ps -ef

systemctl status

journalctl

top

htop
```

---

# Disk Verification

```bash
df -h

du -sh *

lsblk

mount
```

---

# Memory Verification

```bash
free -h

vmstat

top
```

---

# CPU Verification

```bash
top

mpstat

sar

lscpu
```

---

# Service Verification

```bash
systemctl status nginx

systemctl status docker

systemctl status kubelet
```

---

# Log Investigation

Review

```text
Application Logs

↓

System Logs

↓

Container Logs

↓

Kubernetes Events

↓

Cloud Logs
```

---

# Log Locations

```bash
/var/log/messages

/var/log/syslog

/var/log/secure

journalctl

kubectl logs
```

---

# Infrastructure Verification

Check

- EC2
- EKS
- RDS
- Load Balancer
- Auto Scaling
- Route53

---

# Monitoring Verification

Review

- Prometheus
- Grafana
- ELK
- CloudWatch
- AlertManager

---

# Database Verification

Check

- Connectivity
- CPU
- Connections
- Locks
- Replication
- Storage

---

# Rollback Decision

Questions

- Can rollback restore service?
- Is forward fix safer?
- Is data affected?
- Is customer impact increasing?

---

# Emergency Rollback Workflow

```text
Deployment Failed

↓

Rollback

↓

Validate

↓

Monitor

↓

Communicate
```

---

# Temporary Mitigation

Examples

- Scale application
- Restart pods
- Restart services
- Disable feature flag
- Increase replicas
- Redirect traffic

---

# Root Cause Categories

Examples

- Application Bug
- Infrastructure Failure
- Human Error
- Configuration Change
- Deployment Failure
- Third-party Failure
- Security Incident

---

# Root Cause Analysis (RCA)

Document

- Timeline
- Root Cause
- Impact
- Resolution
- Preventive Actions
- Lessons Learned

---

# Postmortem Template

```text
Incident

Timeline

Root Cause

Impact

Detection

Resolution

Action Items

Owner

Target Date
```

---

# Operational Metrics

Track

- MTTD
- MTTA
- MTTR
- Availability
- Error Rate
- SLA
- SLO

---

# Production Checklist

Before closing incident

- Service healthy
- Alerts cleared
- Monitoring stable
- Customers informed
- RCA started
- Logs preserved
- Action items created

---

# Common Production Mistakes

- Restarting without investigation
- Ignoring logs
- Making multiple changes simultaneously
- Skipping rollback plan
- Not documenting actions
- Poor communication
- Closing incident too early
- Not preserving evidence
- Missing RCA
- No follow-up actions

---

# Best Practices

- Follow documented runbooks.
- Assign a single Incident Commander.
- Verify assumptions using metrics and logs.
- Preserve logs before restarting services.
- Communicate status regularly during incidents.
- Roll back only after evaluating impact.
- Capture every action in the incident timeline.
- Complete a postmortem after major incidents.
- Track operational KPIs such as MTTD and MTTR.
- Continuously improve runbooks based on production experience.

---

# Summary

This section introduced production incident management, severity classification, investigation workflows, Linux health checks, communication, rollback strategy, Root Cause Analysis (RCA), postmortems, operational metrics, and production response best practices. These processes provide the operational foundation for handling production incidents consistently and effectively.

---

# Jenkins Production Runbooks

---

# Introduction

Jenkins is a critical component in DevOps environments. Production issues can stop software delivery, delay releases, and impact business operations.

This runbook covers the most common Jenkins production incidents and their recovery procedures.

---

# Jenkins Troubleshooting Workflow

```text
Alert

↓

Check Jenkins Service

↓

Check System Resources

↓

Review Jenkins Logs

↓

Identify Root Cause

↓

Fix Issue

↓

Validate Pipelines

↓

Monitor
```

---

# Jenkins Won't Start

## Symptoms

- Jenkins UI unavailable
- Service failed
- Port 8080 unreachable

---

## Investigation

```bash
systemctl status jenkins

journalctl -u jenkins

ps -ef | grep jenkins

ss -lntp | grep 8080
```

---

## Common Causes

- Java failure
- Disk full
- Corrupted plugin
- Invalid configuration
- Permission issues

---

## Resolution

```bash
systemctl restart jenkins
```

Check logs

```bash
journalctl -u jenkins -f
```

---

# Jenkins Service Failed

## Check

```bash
systemctl status jenkins

systemctl restart jenkins

systemctl enable jenkins
```

---

## Verify Java

```bash
java -version

echo $JAVA_HOME
```

---

# Jenkins UI Not Accessible

## Verify

```bash
curl localhost:8080

ss -lntp

netstat -lntp
```

---

## Check

- Firewall
- Security Groups
- Reverse Proxy
- Nginx
- Load Balancer

---

# Jenkins Agent Offline

## Symptoms

- Pipeline waiting
- Agent disconnected
- Builds queued

---

## Investigation

Master

```bash
Manage Jenkins

↓

Nodes
```

---

Agent

```bash
systemctl status jenkins-agent

journalctl -u jenkins-agent
```

---

## Network Check

```bash
ping

telnet

curl
```

---

## Resolution

Restart agent

```bash
systemctl restart jenkins-agent
```

Reconnect node.

---

# Pipeline Failure

## Investigation

Review

- Console Output
- Stage logs
- Environment variables
- Credentials
- Build history

---

# Build Failure

## Check

```text
Checkout

↓

Build

↓

Unit Tests

↓

Package

↓

Docker Build

↓

Deploy
```

Identify failing stage before making changes.

---

# Git Checkout Failure

## Symptoms

```text
Failed to clone repository
```

---

## Verify

- Git URL
- Credentials
- Branch
- Network
- SSH Key

---

## Commands

```bash
git clone

git ls-remote

ssh -T git@github.com
```

---

# Credential Failure

## Symptoms

- Authentication failed
- Permission denied
- Invalid credentials

---

## Verify

```text
Manage Jenkins

↓

Credentials
```

Check

- Username
- Password
- Token
- SSH Key

---

# Docker Build Failure

## Investigation

```bash
docker version

docker ps

docker images

docker system df
```

---

## Check

- Docker daemon
- Disk space
- Dockerfile
- Registry authentication

---

# Kubernetes Deployment Failure

## Verify

```bash
kubectl get pods

kubectl describe pod

kubectl logs

kubectl get events
```

---

## Review

- kubeconfig
- Cluster access
- Namespace
- Image
- Deployment status

---

# Artifact Upload Failure

## Check

- JFrog
- Nexus
- S3
- Network
- Credentials

---

## Verify

```bash
curl

aws s3 ls
```

---

# Plugin Failure

## Symptoms

- Jenkins startup failure
- Missing functionality
- UI errors

---

## Resolution

Review

```text
Manage Jenkins

↓

Plugin Manager
```

Disable or rollback problematic plugins.

---

# Disk Space Full

## Investigation

```bash
df -h

du -sh /var/lib/jenkins/*

docker system df
```

---

## Cleanup

- Old builds
- Workspaces
- Docker images
- Build artifacts
- Logs

---

# Workspace Cleanup

```bash
rm -rf workspace/*
```

Or use

```text
Workspace Cleanup Plugin
```

---

# Queue Growing

## Check

- Offline agents
- Executor count
- Long-running jobs
- Resource utilization

---

# Performance Issues

Review

- CPU
- Memory
- Disk IO
- Garbage Collection
- Thread Dumps

---

## Commands

```bash
top

free -h

vmstat

iostat
```

---

# Memory Issue

Symptoms

- Slow UI
- OutOfMemoryError
- Frequent GC

---

## Verify

```bash
jps

jcmd

jstat
```

Increase JVM heap if necessary.

---

# Backup Strategy

Backup

- JENKINS_HOME
- Jobs
- Credentials
- Plugins
- Configurations
- Shared Libraries

---

# Restore Jenkins

Workflow

```text
Stop Jenkins

↓

Restore Backup

↓

Verify Plugins

↓

Start Jenkins

↓

Validate Jobs
```

---

# Build Queue Stuck

Review

- Running jobs
- Dead executors
- Locks
- Agent connectivity

---

# Failed Deployment

Workflow

```text
Pipeline Failed

↓

Rollback

↓

Verify Previous Release

↓

Monitor

↓

Notify Team
```

---

# Security Incident

Examples

- Credential leak
- Unauthorized access
- Plugin vulnerability

Immediate Actions

- Rotate credentials
- Disable compromised accounts
- Review audit logs
- Patch Jenkins

---

# Log Locations

```bash
journalctl -u jenkins

/var/log/jenkins

JENKINS_HOME/logs
```

---

# Health Check

Verify

- Jenkins UI
- Agents
- Executors
- Queue
- Plugins
- Credentials
- Pipelines
- Disk Space

---

# Jenkins Production Checklist

Before closing incident

- Jenkins service healthy
- UI accessible
- Agents online
- Queue cleared
- Pipelines validated
- Deployments successful
- Monitoring normal
- Logs reviewed
- RCA documented

---

# Common Jenkins Production Issues

- Jenkins service stopped
- Java failure
- Agent offline
- Plugin conflicts
- Git authentication failure
- Docker daemon unavailable
- Kubernetes authentication issues
- Full disk
- High memory usage
- Corrupted workspace

---

# Best Practices

- Keep Jenkins and plugins updated.
- Backup `JENKINS_HOME` regularly.
- Use dedicated build agents.
- Monitor disk usage continuously.
- Clean old workspaces and artifacts automatically.
- Store credentials securely using Jenkins Credentials.
- Restrict administrative access with RBAC.
- Test plugin updates in non-production first.
- Monitor JVM health and resource utilization.
- Maintain rollback procedures for failed deployments.

---

# Summary

This section covered Jenkins production incident handling, service failures, agent troubleshooting, pipeline failures, Git checkout issues, credential problems, Docker and Kubernetes deployment failures, plugin management, performance tuning, backup and recovery, and operational best practices. These runbooks help restore Jenkins services quickly while minimizing deployment disruptions.

---

