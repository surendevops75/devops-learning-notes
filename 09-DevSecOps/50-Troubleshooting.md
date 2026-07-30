# Enterprise DevOps & DevSecOps Troubleshooting Guide

## Introduction

Troubleshooting is the systematic process of identifying, analyzing, isolating, and resolving issues affecting applications, infrastructure, networks, Kubernetes clusters, cloud services, and CI/CD pipelines.

A structured troubleshooting approach minimizes downtime, accelerates incident resolution, and improves production reliability.

---

# Why Troubleshooting Matters

Production environments are complex systems consisting of multiple interconnected components.

Effective troubleshooting helps organizations:

- Reduce Mean Time to Detect (MTTD)
- Reduce Mean Time to Recover (MTTR)
- Improve Service Availability
- Minimize Business Impact
- Improve Customer Experience
- Strengthen Reliability
- Enhance Operational Excellence

---

# Enterprise Troubleshooting Lifecycle

```text
Issue Reported

↓

Incident Created

↓

Detection

↓

Investigation

↓

Root Cause Analysis

↓

Resolution

↓

Validation

↓

Monitoring

↓

Postmortem

↓

Continuous Improvement
```

Every incident should follow a structured troubleshooting process.

---

# Enterprise Troubleshooting Architecture

```text
                Users
                  │
                  ▼
         Application Load Balancer
                  │
                  ▼
             Kubernetes Ingress
                  │
                  ▼
           Application Services
                  │
                  ▼
               Application Pods
                  │
      ┌───────────┼───────────┐
      ▼           ▼           ▼
 Database     Cache      Message Queue
      │
      ▼
 Monitoring Stack
      │
 ┌────┼─────────────┐
 ▼    ▼             ▼
Logs Metrics     Alerts
 │      │           │
 ▼      ▼           ▼
ELK Prometheus Alertmanager
```

Troubleshooting begins by identifying which layer is experiencing the failure.

---

# Troubleshooting Methodology

A consistent methodology improves incident resolution.

Steps:

1. Identify the issue
2. Collect evidence
3. Isolate the affected component
4. Identify the root cause
5. Apply the fix
6. Validate the solution
7. Document the incident

---

# Layer-Based Troubleshooting

```text
Users

↓

DNS

↓

Load Balancer

↓

Ingress

↓

Kubernetes

↓

Container

↓

Application

↓

Database

↓

Infrastructure
```

Always troubleshoot from the top layer downward or from the symptom toward the root cause.

---

# Production Troubleshooting Workflow

```text
Alert

↓

Dashboard Review

↓

Logs

↓

Metrics

↓

Events

↓

Configuration Review

↓

Root Cause

↓

Resolution

↓

Verification
```

Never apply changes before identifying the actual cause.

---

# Essential Troubleshooting Tools

| Tool | Purpose |
|------|---------|
| kubectl | Kubernetes Management |
| docker | Container Troubleshooting |
| terraform | Infrastructure Validation |
| aws cli | AWS Resource Management |
| helm | Kubernetes Packages |
| git | Source Code |
| curl | API Testing |
| nslookup | DNS Troubleshooting |
| dig | DNS Resolution |
| ping | Network Reachability |
| traceroute | Network Path |
| top | CPU Monitoring |
| htop | Resource Monitoring |
| ps | Process Inspection |
| journalctl | System Logs |
| systemctl | Service Management |

---

# Enterprise Observability Stack

```text
Application

↓

Metrics

↓

Prometheus

↓

Grafana

──────────────

Application

↓

Logs

↓

Fluent Bit

↓

ELK

──────────────

Application

↓

Security Events

↓

Falco

↓

SOC
```

Observability provides the evidence required during troubleshooting.

---

# Five Golden Signals

Production monitoring should include the five golden signals.

- Latency
- Traffic
- Errors
- Saturation
- Availability

These metrics help quickly identify application health.

---

# Troubleshooting Decision Tree

```text
Issue

↓

Infrastructure?

├── Yes → Infrastructure Investigation

└── No

↓

Application?

├── Yes → Application Investigation

└── No

↓

Network?

├── Yes → Network Investigation

└── No

↓

Security?

├── Yes → Security Investigation

└── No

↓

Configuration Investigation
```

Classifying the issue early reduces investigation time.

---

# Incident Severity Levels

| Severity | Description | Response |
|----------|-------------|----------|
| P1 | Complete Production Outage | Immediate |
| P2 | Major Service Degradation | High Priority |
| P3 | Partial Functionality Loss | Medium Priority |
| P4 | Minor Issue | Planned Resolution |

Organizations should define clear escalation procedures for each severity level.

---

# Information to Collect

Before making changes, collect:

- Incident Time
- Error Messages
- Application Logs
- Kubernetes Events
- System Metrics
- Recent Deployments
- Infrastructure Changes
- User Impact
- Monitoring Alerts
- Configuration Changes

Evidence collection should always precede remediation.

---

# Enterprise Best Practices

- Follow a documented troubleshooting process.
- Collect evidence before making changes.
- Use centralized monitoring and logging.
- Correlate logs, metrics, and events.
- Prioritize production incidents by severity.
- Validate fixes before closing incidents.
- Document root causes and resolutions.
- Conduct postmortems for major incidents.
- Automate repetitive diagnostics where possible.
- Continuously improve operational runbooks.

---

# Kubernetes Troubleshooting

Kubernetes problems can occur at the cluster, node, Pod, container, networking, or application level.

Always troubleshoot layer by layer instead of making assumptions.

---

# Kubernetes Troubleshooting Flow

```text
User Reports Issue

↓

Check Deployment

↓

Check Pods

↓

Check Events

↓

Check Logs

↓

Check Node

↓

Check Network

↓

Identify Root Cause

↓

Fix

↓

Validate
```

---

# Scenario 1

# Pod Status: CrashLoopBackOff

## Symptoms

```text
Pod

↓

Starts

↓

Crashes

↓

Restarts

↓

CrashLoopBackOff
```

---

## Investigation

```bash
kubectl get pods

kubectl describe pod <pod-name>

kubectl logs <pod-name>

kubectl logs <pod-name> --previous
```

---

## Possible Causes

- Application Crash
- Invalid Environment Variables
- Missing Secrets
- Missing ConfigMaps
- Database Connection Failure
- Resource Limits
- Startup Failure

---

## Resolution

- Review application logs.
- Verify Secrets and ConfigMaps.
- Validate environment variables.
- Check resource limits.
- Confirm dependent services are available.

---

# Scenario 2

# Pod Status: ImagePullBackOff

## Symptoms

```text
Pod

↓

Unable to Pull Image

↓

ImagePullBackOff
```

---

## Investigation

```bash
kubectl describe pod <pod-name>

kubectl get events
```

---

## Possible Causes

- Incorrect Image Name
- Invalid Image Tag
- Registry Authentication Failure
- Image Does Not Exist
- Network Connectivity Issue

---

## Resolution

- Verify repository name.
- Verify image tag.
- Check ImagePullSecrets.
- Confirm registry accessibility.
- Validate repository permissions.

---

# Scenario 3

# Pod Status: Pending

## Symptoms

```text
Pod

↓

Scheduler

↓

Pending
```

---

## Investigation

```bash
kubectl describe pod <pod-name>

kubectl get nodes
```

---

## Possible Causes

- Insufficient CPU
- Insufficient Memory
- Taints
- Missing Tolerations
- Node Selector Mismatch
- PVC Not Available

---

## Resolution

- Review scheduler events.
- Increase cluster capacity.
- Verify node labels.
- Check Persistent Volumes.
- Review taints and tolerations.

---

# Scenario 4

# Pod Status: OOMKilled

## Symptoms

```text
Application

↓

Memory Exhausted

↓

OOMKilled
```

---

## Investigation

```bash
kubectl describe pod <pod-name>

kubectl top pod

kubectl top node
```

---

## Possible Causes

- Memory Leak
- Low Memory Limit
- Large Workload
- Inefficient Queries

---

## Resolution

- Increase memory limits.
- Optimize application memory usage.
- Investigate memory leaks.
- Review heap configuration.

---

# Scenario 5

# Pod Status: ErrImagePull

## Symptoms

Container fails before startup.

---

## Investigation

```bash
kubectl describe pod <pod-name>
```

---

## Possible Causes

- Repository Not Found
- Authentication Failure
- Invalid Registry
- DNS Failure

---

## Resolution

- Verify registry URL.
- Check credentials.
- Validate image path.
- Test registry connectivity.

---

# Node Troubleshooting

Node failures affect multiple workloads.

---

# Investigation

```bash
kubectl get nodes

kubectl describe node <node>

kubectl top node
```

---

# Common Node Problems

- Node NotReady
- Disk Pressure
- Memory Pressure
- PID Pressure
- Network Failure
- Kubelet Failure

---

# Node Recovery Workflow

```text
Node Failure

↓

Check Status

↓

Check Kubelet

↓

Check Resources

↓

Restart Services

↓

Validate Node

↓

Schedule Pods
```

---

# Docker Troubleshooting

Containers may fail before reaching Kubernetes.

---

# Check Running Containers

```bash
docker ps

docker ps -a
```

---

# View Container Logs

```bash
docker logs <container-id>
```

---

# Inspect Container

```bash
docker inspect <container-id>
```

---

# Check Resource Usage

```bash
docker stats
```

---

# Restart Container

```bash
docker restart <container-id>
```

---

# Docker Troubleshooting Workflow

```text
Container

↓

Status

↓

Logs

↓

Inspect

↓

Network

↓

Volumes

↓

Restart

↓

Validate
```

---

# Container Networking Issues

## Symptoms

- Connection Timeout
- Connection Refused
- DNS Failure
- Service Unreachable

---

## Investigation

```bash
kubectl get svc

kubectl get endpoints

kubectl describe svc
```

---

## Resolution

- Verify Service selectors.
- Confirm Endpoints exist.
- Check Network Policies.
- Validate DNS resolution.
- Test connectivity between Pods.

---

# DNS Troubleshooting

DNS failures prevent service communication.

---

## Investigation

```bash
kubectl get pods -n kube-system

kubectl logs -n kube-system deployment/coredns
```

---

## Resolution

- Verify CoreDNS is healthy.
- Restart CoreDNS if required.
- Check Service definitions.
- Validate DNS policies.

---

# Enterprise Best Practices

- Investigate Events before making changes.
- Use `kubectl describe` before restarting Pods.
- Always review previous container logs.
- Monitor node health continuously.
- Configure resource requests and limits.
- Use readiness and liveness probes correctly.
- Verify DNS before investigating applications.
- Avoid deleting Pods before collecting logs.
- Document recurring issues.
- Create runbooks for common Kubernetes failures.

---

# Jenkins Troubleshooting

Jenkins is the heart of many CI/CD pipelines.

Pipeline failures should be investigated stage by stage instead of rerunning the job immediately.

---

# Jenkins Troubleshooting Workflow

```text
Pipeline Failed

↓

Build Logs

↓

Failed Stage

↓

Error Message

↓

Root Cause

↓

Fix

↓

Rebuild

↓

Validate
```

---

# Scenario 1

# Jenkins Build Failed

## Symptoms

```text
Build

↓

FAILED
```

---

## Investigation

```bash
Check:

Build Console Output

Pipeline Logs

Stage Logs
```

---

## Possible Causes

- Compilation Failure
- Test Failure
- Missing Dependency
- Invalid Environment Variable
- Build Script Error

---

## Resolution

- Review console logs.
- Fix compilation errors.
- Install missing dependencies.
- Verify pipeline configuration.
- Re-run after validation.

---

# Scenario 2

# Jenkins Agent Offline

## Symptoms

```text
Controller

↓

Agent

↓

Offline
```

---

## Investigation

```bash
systemctl status jenkins

systemctl status ssh

journalctl -u jenkins
```

---

## Possible Causes

- Network Issue
- SSH Failure
- Agent Service Down
- Authentication Failure

---

## Resolution

- Restart the agent.
- Verify SSH connectivity.
- Check firewall rules.
- Validate agent credentials.

---

# Scenario 3

# Jenkins Workspace Issues

## Symptoms

- Old files remain
- Unexpected build failures
- Incorrect artifacts

---

## Investigation

```bash
ls

pwd

du -sh .
```

---

## Resolution

Clean the workspace before each build.

```groovy
cleanWs()
```

---

# Scenario 4

# Pipeline Cannot Access Credentials

## Symptoms

```text
Access Denied

↓

Credential Not Found
```

---

## Investigation

Verify:

- Jenkins Credentials
- Credential ID
- Folder Permissions
- Pipeline Configuration

---

## Resolution

Use Jenkins Credentials instead of hardcoded secrets.

---

# Git Troubleshooting

Git issues are common during CI/CD execution.

---

# Scenario 1

# Merge Conflict

## Symptoms

```text
Git Merge

↓

Conflict
```

---

## Investigation

```bash
git status

git diff
```

---

## Resolution

Resolve conflicts manually.

```bash
git add .

git commit
```

---

# Scenario 2

# Authentication Failed

## Symptoms

```text
fatal:

Authentication failed
```

---

## Investigation

Verify:

- Personal Access Token
- SSH Keys
- Repository Permissions

---

## Resolution

Regenerate credentials and update repository access.

---

# Scenario 3

# Detached HEAD

## Investigation

```bash
git status

git branch
```

---

## Resolution

```bash
git checkout main
```

Return to the correct branch before making changes.

---

# Docker Registry Troubleshooting

Container images may fail during push or pull operations.

---

# Scenario 1

# Docker Push Failed

## Symptoms

```text
docker push

↓

Access Denied
```

---

## Investigation

```bash
docker login

docker images
```

---

## Possible Causes

- Authentication Failure
- Missing Permissions
- Incorrect Repository
- Expired Token

---

## Resolution

Authenticate again and verify repository permissions.

---

# Scenario 2

# Image Not Found

## Symptoms

```text
manifest unknown
```

---

## Investigation

```bash
docker images
```

---

## Resolution

Verify:

- Repository Name
- Image Tag
- Registry URL

---

# Terraform Troubleshooting

Infrastructure provisioning problems should be investigated before applying changes.

---

# Scenario 1

# Terraform Init Failed

## Investigation

```bash
terraform init
```

---

## Possible Causes

- Backend Configuration
- Provider Download Failure
- Network Issue

---

## Resolution

Validate backend configuration and provider versions.

---

# Scenario 2

# Terraform Plan Failed

## Investigation

```bash
terraform validate

terraform plan
```

---

## Possible Causes

- Syntax Error
- Missing Variable
- Invalid Resource
- Module Failure

---

## Resolution

Validate the configuration before planning.

---

# Scenario 3

# Terraform Apply Failed

## Investigation

```bash
terraform apply
```

Review:

- IAM Permissions
- Resource Limits
- Existing Resources
- Cloud API Errors

---

## Resolution

Correct the reported issue and re-run the deployment.

---

# AWS Troubleshooting

Cloud infrastructure issues require verification of IAM, networking, and service configuration.

---

# Scenario 1

# EC2 Instance Unreachable

## Investigation

```bash
ping <private-ip>

ssh ec2-user@<private-ip>
```

---

## Verify

- Security Groups
- Network ACLs
- Route Tables
- SSH Keys
- EC2 Status

---

## Resolution

Correct network configuration and verify SSH access.

---

# Scenario 2

# Amazon EKS Cluster Unreachable

## Investigation

```bash
kubectl cluster-info

kubectl get nodes
```

---

## Verify

- kubeconfig
- IAM Authentication
- Cluster Status
- Network Connectivity

---

## Resolution

Update kubeconfig and verify IAM permissions.

---

# Scenario 3

# Amazon ECR Authentication Failure

## Investigation

```bash
aws ecr get-login-password
```

---

## Verify

- IAM Permissions
- Repository
- AWS Region
- Login Token

---

## Resolution

Authenticate again and retry the image push or pull.

---

# Enterprise Troubleshooting Checklist

Before making production changes, verify:

- Application Health
- Infrastructure Status
- Kubernetes Events
- Container Logs
- Network Connectivity
- DNS Resolution
- IAM Permissions
- Recent Deployments
- Monitoring Alerts
- Configuration Changes

Always eliminate possible causes systematically.

---

# Enterprise Best Practices

- Read logs before restarting services.
- Investigate one layer at a time.
- Avoid changing multiple components simultaneously.
- Validate fixes in non-production environments when possible.
- Preserve logs for post-incident analysis.
- Maintain Infrastructure as Code for repeatability.
- Use centralized logging and monitoring.
- Automate common diagnostic checks.
- Document recurring production issues.
- Update operational runbooks after every major incident.

---

# Network Troubleshooting

Network problems are one of the most common causes of production incidents.

Always verify connectivity before investigating the application.

---

# Network Troubleshooting Workflow

```text
Application Error

↓

DNS

↓

Network

↓

Firewall

↓

Load Balancer

↓

Service

↓

Application

↓

Resolved
```

Investigate each network layer sequentially.

---

# Scenario 1

# DNS Resolution Failure

## Symptoms

```text
Application

↓

DNS Lookup

↓

Failed
```

---

## Investigation

```bash
nslookup application.example.com

dig application.example.com
```

---

## Possible Causes

- Incorrect DNS Record
- DNS Propagation Delay
- CoreDNS Failure
- Route53 Configuration Error

---

## Resolution

- Verify DNS records.
- Check DNS propagation.
- Restart CoreDNS if necessary.
- Validate Route53 configuration.

---

# Scenario 2

# Application Load Balancer Returns 502

## Symptoms

```text
Client

↓

ALB

↓

502 Bad Gateway
```

---

## Investigation

Verify:

- Target Group Health
- Application Health
- Security Groups
- Listener Rules
- Target Registration

---

## Resolution

- Restore healthy targets.
- Review application logs.
- Verify target ports.
- Confirm readiness probes.

---

# Scenario 3

# Service Connection Timeout

## Symptoms

```text
Client

↓

Service

↓

Timeout
```

---

## Investigation

```bash
curl http://service

kubectl get svc

kubectl get endpoints
```

---

## Resolution

- Verify Service selectors.
- Confirm Endpoints exist.
- Review Network Policies.
- Validate firewall configuration.

---

# Security Troubleshooting

Security controls may block legitimate application traffic.

Always review security policies before modifying them.

---

# Scenario 1

# Access Denied

## Symptoms

```text
User

↓

Authentication

↓

Access Denied
```

---

## Investigation

Verify:

- IAM Policy
- RBAC
- Security Groups
- Authentication Logs

---

## Resolution

Grant only the required permissions following the Principle of Least Privilege.

---

# Scenario 2

# Secret Not Available

## Symptoms

Application startup fails because required secrets cannot be loaded.

---

## Investigation

```bash
kubectl get secrets

kubectl describe secret <secret-name>
```

---

## Resolution

- Verify secret exists.
- Confirm namespace.
- Check application references.
- Validate secret permissions.

---

# Monitoring Troubleshooting

Monitoring systems must function correctly to detect production issues.

---

# Scenario 1

# Prometheus Target Down

## Symptoms

```text
Target

↓

DOWN
```

---

## Investigation

Verify:

- Exporter Status
- Service Discovery
- Network Connectivity
- Prometheus Configuration

---

## Resolution

Restart exporters if necessary and validate scrape configuration.

---

# Scenario 2

# Grafana Dashboard Shows No Data

## Symptoms

Charts display "No Data".

---

## Investigation

Verify:

- Prometheus Data Source
- Dashboard Variables
- Time Range
- Query Syntax

---

## Resolution

Reconnect the data source and validate PromQL queries.

---

# Scenario 3

# Alertmanager Not Sending Alerts

## Symptoms

Critical issues occur without notifications.

---

## Investigation

Verify:

- Alert Rules
- Notification Configuration
- SMTP or Webhook Settings
- Alertmanager Logs

---

## Resolution

Test notification channels and validate routing configuration.

---

# Database Troubleshooting

Databases are often the source of application failures.

Investigate connectivity before modifying application code.

---

# Scenario 1

# Database Connection Refused

## Symptoms

```text
Application

↓

Database

↓

Connection Refused
```

---

## Investigation

Verify:

- Database Availability
- Credentials
- Network Connectivity
- Security Groups
- Database Listener Port

---

## Resolution

Restore connectivity and validate authentication.

---

# Scenario 2

# Slow Database Queries

## Symptoms

Application response time increases significantly.

---

## Investigation

Review:

- Query Execution Plans
- Database CPU
- Locks
- Index Usage
- Connection Pool

---

## Resolution

Optimize queries, add indexes where appropriate, and review connection pool settings.

---

# Production Incident Management

Every production incident should follow a standardized process.

---

# Incident Workflow

```text
Alert

↓

Incident Created

↓

Assign Owner

↓

Investigation

↓

Root Cause

↓

Mitigation

↓

Recovery

↓

Validation

↓

Postmortem
```

Incident ownership should be assigned immediately.

---

# Root Cause Analysis Checklist

Investigate the following:

- What failed?
- When did it fail?
- Who was affected?
- What changed?
- How was the issue detected?
- What resolved the issue?
- How can recurrence be prevented?

Root Cause Analysis should focus on system improvements.

---

# Production Verification

After resolving an incident, verify:

- Application Availability
- API Response
- Database Connectivity
- Monitoring Dashboards
- Alert Status
- Logs
- User Transactions
- Error Rate

Production validation confirms successful recovery.

---

# Enterprise Incident Checklist

Before closing an incident, confirm:

- Root Cause Identified
- Permanent Fix Applied
- Monitoring Restored
- Alerts Operational
- Documentation Updated
- Stakeholders Informed
- Postmortem Completed
- Preventive Actions Assigned

Incident closure should occur only after verification.

---

# Enterprise Best Practices

- Troubleshoot one layer at a time.
- Correlate logs, metrics, and events.
- Verify infrastructure before modifying applications.
- Follow documented incident response procedures.
- Preserve evidence during investigations.
- Validate production health after every fix.
- Perform Root Cause Analysis for significant incidents.
- Update runbooks after recurring issues.
- Automate health checks and diagnostics where possible.
- Continuously improve operational processes based on production incidents.

---

# Common Mistakes

## Mistake 1

### Restarting Services Without Investigation

**Problem**

Administrators immediately restart Pods, containers, or servers.

```text
Issue

↓

Restart

↓

Issue Returns
```

**Impact**

- Root cause remains unknown
- Important logs are lost
- Incident duration increases

**Recommendation**

Always collect logs, events, and metrics before restarting any service.

---

## Mistake 2

### Ignoring Kubernetes Events

**Problem**

Only application logs are checked.

**Impact**

- Scheduler failures are missed
- Resource issues remain unnoticed
- Node problems are ignored

**Recommendation**

Always review Kubernetes Events.

```bash
kubectl get events --sort-by=.lastTimestamp
```

---

## Mistake 3

### Making Multiple Changes Simultaneously

**Problem**

Several configurations are modified during troubleshooting.

```text
Problem

↓

Multiple Changes

↓

Unknown Fix
```

**Impact**

- Difficult Root Cause Analysis
- Increased production risk
- Longer recovery

**Recommendation**

Apply one change at a time and validate the outcome.

---

## Mistake 4

### Ignoring Monitoring Alerts

**Problem**

Production alerts are acknowledged without investigation.

**Impact**

- Recurring incidents
- Hidden failures
- Reduced system reliability

**Recommendation**

Treat every critical alert as a potential production incident until verified.

---

## Mistake 5

### Closing Incidents Without Root Cause Analysis

**Problem**

The service is restored but no investigation is completed.

**Impact**

- Repeat incidents
- No preventive measures
- Poor operational maturity

**Recommendation**

Perform Root Cause Analysis (RCA) for all major incidents.

---

# End-to-End Production Troubleshooting Scenario

## Scenario

Users report that the application is unavailable immediately after a production deployment.

---

## Investigation Workflow

```text
User Complaint

↓

Monitoring Alert

↓

Ingress

↓

Service

↓

Pods

↓

Container Logs

↓

Application

↓

Database

↓

Root Cause

↓

Fix

↓

Validation
```

---

## Step 1

### Verify Deployment

```bash
kubectl get deployments

kubectl rollout status deployment payment
```

---

## Step 2

### Check Pods

```bash
kubectl get pods -o wide

kubectl describe pod <pod-name>
```

---

## Step 3

### Review Logs

```bash
kubectl logs <pod-name>

kubectl logs <pod-name> --previous
```

---

## Step 4

### Verify Services

```bash
kubectl get svc

kubectl get endpoints
```

---

## Step 5

### Verify Ingress

```bash
kubectl get ingress

kubectl describe ingress
```

---

## Step 6

### Check Node Health

```bash
kubectl get nodes

kubectl describe node <node-name>
```

---

## Step 7

### Validate Monitoring

Verify:

- Prometheus
- Grafana
- Alertmanager
- ELK Dashboards

Confirm that all application health indicators have returned to normal.

---

# Production Interview Questions

## Question 1

### What is your troubleshooting methodology?

**Answer**

Identify the issue, collect evidence, isolate the affected layer, determine the root cause, implement the fix, validate the solution, and document the incident.

---

## Question 2

### What should you check first when a Pod enters CrashLoopBackOff?

**Answer**

Review `kubectl describe pod`, Kubernetes Events, current logs, previous logs, Secrets, ConfigMaps, environment variables, and application dependencies.

---

## Question 3

### How do you troubleshoot ImagePullBackOff?

**Answer**

Verify the image name, tag, registry authentication, ImagePullSecrets, network connectivity, and repository permissions.

---

## Question 4

### How do you investigate a failed Jenkins pipeline?

**Answer**

Identify the failed stage, review console logs, inspect pipeline configuration, verify credentials, and resolve the reported error before rerunning the build.

---

## Question 5

### How do you troubleshoot Terraform failures?

**Answer**

Run `terraform validate`, `terraform plan`, review provider configuration, backend configuration, variables, IAM permissions, and cloud provider errors.

---

## Question 6

### What information should be collected before fixing a production issue?

**Answer**

Logs, metrics, Kubernetes Events, monitoring alerts, deployment history, configuration changes, infrastructure status, and user impact.

---

## Question 7

### Why should logs be collected before restarting services?

**Answer**

Restarting services may remove valuable diagnostic information needed to identify the actual root cause of the incident.

---

## Question 8

### How do you perform Root Cause Analysis?

**Answer**

Review the incident timeline, identify triggering changes, analyze logs and metrics, determine the underlying cause, implement preventive actions, and document the findings.

---

## Question 9

### Which tools are commonly used during DevOps troubleshooting?

**Answer**

kubectl, Docker, Terraform, AWS CLI, Helm, Git, curl, Prometheus, Grafana, ELK Stack, Alertmanager, Falco, journalctl, systemctl, and Linux diagnostic utilities.

---

## Question 10

### What are the characteristics of an effective production troubleshooting process?

**Answer**

Structured investigation, evidence collection, layer-by-layer analysis, minimal production impact, validated fixes, documented Root Cause Analysis, and continuous operational improvement.

---

# Key Takeaways

- Follow a structured troubleshooting methodology.
- Collect evidence before making changes.
- Investigate one infrastructure layer at a time.
- Correlate logs, metrics, events, and alerts.
- Review Kubernetes Events before restarting Pods.
- Validate infrastructure before modifying applications.
- Preserve diagnostic information during incidents.
- Verify application health after implementing fixes.
- Perform Root Cause Analysis for major incidents.
- Update operational runbooks using lessons learned.
- Automate diagnostics where practical.
- Continuously improve production operations through post-incident reviews.

---

# Enterprise Troubleshooting Workflow

```text
User Reports Issue

↓

Monitoring Alert

↓

Incident Created

↓

Severity Assessment

↓

Evidence Collection

├── Logs

├── Metrics

├── Events

├── Alerts

└── Recent Changes

↓

Layer-by-Layer Investigation

↓

Root Cause Analysis

↓

Implement Fix

↓

Validation

↓

Production Monitoring

↓

Incident Closure

↓

Postmortem

↓

Runbook Update

↓

Continuous Improvement
```

This workflow represents an enterprise-grade troubleshooting process that minimizes Mean Time to Detect (MTTD), reduces Mean Time to Recover (MTTR), improves service reliability, and drives continuous operational excellence.


