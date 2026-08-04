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

