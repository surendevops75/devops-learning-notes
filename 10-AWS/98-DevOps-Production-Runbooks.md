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

# Kubernetes & Amazon EKS Production Runbooks

---

# Introduction

Kubernetes production incidents can affect application availability, scalability, networking, and deployments. A structured troubleshooting approach minimizes downtime and accelerates recovery.

---

# Kubernetes Troubleshooting Workflow

```text
Alert

↓

Check Cluster Health

↓

Check Nodes

↓

Check Pods

↓

Review Events

↓

Review Logs

↓

Identify Root Cause

↓

Mitigate

↓

Recover
```

---

# Cluster Health Check

```bash
kubectl cluster-info

kubectl get componentstatuses

kubectl version

kubectl get nodes
```

---

# Node Health

```bash
kubectl get nodes -o wide

kubectl describe node <node-name>

kubectl top nodes
```

---

# Pod Health

```bash
kubectl get pods -A

kubectl get pods -o wide

kubectl describe pod <pod>

kubectl logs <pod>

kubectl logs <pod> --previous
```

---

# Kubernetes Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp

kubectl get events -A
```

Always review Events before restarting workloads.

---

# Pods Stuck in Pending

## Symptoms

```text
STATUS

Pending
```

---

## Investigation

```bash
kubectl describe pod <pod>
```

Check

- Insufficient CPU
- Insufficient Memory
- Node Selector
- Taints
- Affinity Rules
- PVC binding
- Scheduler Events

---

## Resolution

- Add cluster capacity
- Scale worker nodes
- Fix scheduling rules
- Verify Persistent Volumes

---

# CrashLoopBackOff

## Symptoms

```text
STATUS

CrashLoopBackOff
```

---

## Investigation

```bash
kubectl logs <pod> --previous

kubectl describe pod <pod>

kubectl get events
```

Review

- Application logs
- Environment variables
- Secrets
- ConfigMaps
- Startup scripts
- Health probes

---

## Common Causes

- Application crash
- Missing configuration
- Invalid Secrets
- Database unavailable
- Incorrect image

---

# ImagePullBackOff

## Symptoms

```text
Failed to pull image
```

---

## Investigation

```bash
kubectl describe pod

kubectl get events
```

Verify

- Image name
- Image tag
- Amazon ECR access
- ImagePullSecrets
- IAM permissions

---

## Resolution

- Push image
- Correct image tag
- Fix registry credentials
- Verify repository permissions

---

# ErrImagePull

Common Causes

- Typo in image name
- Repository missing
- Authentication failure
- Network connectivity issue

---

# OOMKilled

## Symptoms

```text
Reason

OOMKilled
```

---

## Investigation

```bash
kubectl describe pod

kubectl top pod
```

Review

- Memory limits
- Memory requests
- Application memory usage

---

## Resolution

- Increase memory limit
- Optimize application
- Fix memory leak
- Right-size resources

---

# Node NotReady

## Symptoms

```text
STATUS

NotReady
```

---

## Investigation

```bash
kubectl describe node

systemctl status kubelet
```

Check

- kubelet
- Disk pressure
- Memory pressure
- Network
- Container runtime

---

## Resolution

```bash
systemctl restart kubelet
```

Investigate underlying infrastructure before restarting.

---

# Pod Evicted

Common Reasons

- Disk Pressure
- Memory Pressure
- Node Maintenance

---

## Investigation

```bash
kubectl describe pod
```

---

# Deployment Failure

## Check

```bash
kubectl rollout status deployment/<deployment>

kubectl rollout history deployment/<deployment>

kubectl describe deployment
```

---

## Rollback

```bash
kubectl rollout undo deployment/<deployment>
```

---

# ReplicaSet Issues

```bash
kubectl get rs

kubectl describe rs
```

Verify desired and available replicas.

---

# Service Issues

## Investigation

```bash
kubectl get svc

kubectl describe svc

kubectl get endpoints
```

Verify

- Labels
- Selectors
- TargetPort
- Endpoints

---

# DNS Resolution Failure

## Investigation

```bash
kubectl exec -it <pod> -- nslookup kubernetes.default

kubectl get pods -n kube-system
```

Check

- CoreDNS
- Network
- DNS Config

---

# Ingress Issues

## Verify

```bash
kubectl get ingress

kubectl describe ingress
```

Check

- Rules
- Backend services
- ALB status
- Certificates

---

# AWS Load Balancer Controller

Verify

```bash
kubectl get pods -n kube-system

kubectl logs deployment/aws-load-balancer-controller -n kube-system
```

---

# Persistent Volume Issues

## Investigation

```bash
kubectl get pv

kubectl get pvc

kubectl describe pvc
```

Check

- StorageClass
- Volume binding
- Access modes
- Capacity

---

# ConfigMap Issues

```bash
kubectl describe configmap

kubectl get configmap
```

Verify

- Mounted correctly
- Updated values
- Namespace

---

# Secret Issues

```bash
kubectl get secrets

kubectl describe secret
```

Verify

- Namespace
- Mount
- Environment variables

---

# HPA Issues

```bash
kubectl get hpa

kubectl describe hpa
```

Check

- Metrics Server
- CPU metrics
- Memory metrics
- Scaling events

---

# Cluster Autoscaler

Verify

```bash
kubectl logs deployment/cluster-autoscaler -n kube-system
```

Review

- Scale-up failures
- Scale-down events
- AWS permissions

---

# Amazon EKS Control Plane

Review

- API Server availability
- IAM authentication
- Cluster endpoint
- Node registration

---

# Worker Node Issues

Check

```bash
kubectl get nodes

systemctl status kubelet

journalctl -u kubelet
```

---

# Container Runtime

Verify

```bash
systemctl status containerd

crictl ps
```

---

# Namespace Issues

```bash
kubectl get ns

kubectl describe ns
```

---

# Network Policies

Review

```bash
kubectl get networkpolicy

kubectl describe networkpolicy
```

---

# Resource Quotas

```bash
kubectl get resourcequota

kubectl describe resourcequota
```

---

# Production Health Checklist

Verify

- Nodes Ready
- Pods Running
- Services Healthy
- Ingress Working
- Storage Mounted
- DNS Working
- Metrics Available
- HPA Healthy

---

# Common Kubernetes Production Issues

- Pending Pods
- CrashLoopBackOff
- OOMKilled
- ImagePullBackOff
- Node NotReady
- Failed Deployment
- DNS Failure
- PVC Pending
- ALB Ingress Failure
- Cluster Autoscaler Failure

---

# Recovery Workflow

```text
Incident

↓

Identify Component

↓

Collect Logs

↓

Review Events

↓

Mitigate

↓

Validate

↓

Monitor

↓

Document RCA
```

---

# Best Practices

- Always review `kubectl describe` before restarting Pods.
- Check Events before making configuration changes.
- Preserve logs using `--previous` for restarted containers.
- Configure resource requests and limits for all workloads.
- Monitor cluster health continuously using Prometheus and Grafana.
- Enable Cluster Autoscaler or Karpenter for Amazon EKS.
- Test deployment rollbacks regularly.
- Use readiness and liveness probes correctly.
- Verify Ingress and DNS after every deployment.
- Document production incidents and update runbooks after every RCA.

---

# Summary

This section covered production troubleshooting for Kubernetes and Amazon EKS, including Pending Pods, CrashLoopBackOff, ImagePullBackOff, OOMKilled, Node NotReady, Deployments, Services, Ingress, DNS, Persistent Volumes, HPA, Cluster Autoscaler, and recovery workflows. These runbooks provide a structured approach to diagnosing and resolving the most common Kubernetes production incidents.

---

# Docker & Container Production Runbooks

---

# Introduction

Docker is the foundation for modern containerized applications. Production issues can impact deployments, Kubernetes workloads, CI/CD pipelines, and application availability.

This runbook provides a structured approach to troubleshooting Docker and container runtime issues.

---

# Docker Troubleshooting Workflow

```text
Alert

↓

Check Docker Service

↓

Check Containers

↓

Review Logs

↓

Verify Images

↓

Check Storage

↓

Check Network

↓

Identify Root Cause

↓

Recover
```

---

# Docker Daemon Not Running

## Symptoms

- Docker commands fail
- Containers unavailable
- CI/CD pipelines fail

---

## Investigation

```bash
systemctl status docker

journalctl -u docker

docker version

docker info
```

---

## Common Causes

- Docker daemon stopped
- Invalid daemon configuration
- Low disk space
- Permission issues
- Container runtime failure

---

## Resolution

```bash
systemctl restart docker
```

Verify

```bash
docker ps
```

---

# Docker Service Failed

## Investigation

```bash
systemctl status docker

journalctl -xe

journalctl -u docker -f
```

---

## Verify

- daemon.json
- Storage driver
- Network configuration
- Docker socket

---

# Container Not Starting

## Symptoms

```text
Exited

Created

Restarting
```

---

## Investigation

```bash
docker ps -a

docker inspect <container>

docker logs <container>
```

---

## Common Causes

- Invalid command
- Missing environment variables
- Volume mount failure
- Permission issues
- Missing dependencies

---

# Container Exits Immediately

## Investigation

```bash
docker logs <container>

docker inspect <container>
```

Review

- Entry point
- CMD
- Exit code
- Application startup

---

## Common Exit Codes

| Exit Code | Meaning |
|-----------|---------|
| 0 | Successful exit |
| 1 | General application error |
| 125 | Docker execution error |
| 126 | Command cannot execute |
| 127 | Command not found |
| 137 | Killed (OOM or SIGKILL) |
| 143 | Graceful termination (SIGTERM) |

---

# Restart Loop

## Symptoms

```text
Restarting

Restart Count Increasing
```

---

## Investigation

```bash
docker logs

docker inspect

docker events
```

Review

- Application crash
- Health checks
- Startup scripts
- Resource limits

---

# Image Pull Failure

## Symptoms

```text
pull access denied

image not found
```

---

## Investigation

```bash
docker pull <image>

docker login

docker images
```

Verify

- Image name
- Image tag
- Registry authentication
- Network connectivity

---

# Amazon ECR Authentication

Verify

```bash
aws ecr get-login-password

docker login

docker pull
```

---

# Volume Mount Failure

## Investigation

```bash
docker inspect

docker volume ls

docker volume inspect
```

Verify

- Mount path
- Permissions
- Disk availability
- Host path

---

# Bind Mount Issues

Check

```bash
ls -ld

mount

df -h
```

---

# Container Network Failure

## Investigation

```bash
docker network ls

docker network inspect

docker inspect
```

---

## Connectivity Tests

```bash
ping

curl

nslookup

telnet
```

---

# DNS Failure

Verify

```bash
cat /etc/resolv.conf

docker exec -it <container> nslookup google.com
```

---

# High CPU Usage

## Investigation

```bash
docker stats

top

htop
```

Review

- Application loops
- Infinite retries
- Heavy workloads

---

# High Memory Usage

## Investigation

```bash
docker stats

free -h

top
```

Review

- Memory leaks
- Cache growth
- JVM heap usage

---

# OOMKilled Container

## Symptoms

```text
Exit Code

137
```

---

## Investigation

```bash
docker inspect

docker logs

docker stats
```

---

## Resolution

- Increase memory
- Optimize application
- Tune JVM
- Fix memory leak

---

# Disk Full

## Investigation

```bash
df -h

docker system df

du -sh /var/lib/docker/*
```

---

## Cleanup

```bash
docker image prune

docker container prune

docker volume prune

docker network prune
```

---

## Full Cleanup

```bash
docker system prune -a
```

Use with caution in production.

---

# Too Many Images

Verify

```bash
docker images
```

Remove

```bash
docker rmi <image>
```

---

# Too Many Containers

Verify

```bash
docker ps -a
```

Remove

```bash
docker rm <container>
```

---

# Docker Logs Too Large

Verify

```bash
du -sh /var/lib/docker/containers/*
```

Configure

```text
Log Rotation

max-size

max-file
```

---

# Docker Compose Failure

## Investigation

```bash
docker compose ps

docker compose logs

docker compose config
```

---

## Verify

- Compose file
- Environment variables
- Network
- Volumes

---

# Docker Socket Issues

Verify

```bash
ls -l /var/run/docker.sock
```

Review

- Ownership
- Group permissions
- Docker service

---

# Registry Authentication Failure

Review

- Username
- Password
- Token
- Repository permissions

---

# Container Health Check Failure

Inspect

```bash
docker inspect <container>
```

Review

- Health command
- Timeout
- Interval
- Retries

---

# Docker Storage Driver

Verify

```bash
docker info
```

Review

- overlay2
- Storage utilization
- Filesystem support

---

# Production Recovery Workflow

```text
Incident

↓

Docker Service

↓

Container Logs

↓

Images

↓

Network

↓

Storage

↓

Restart

↓

Validation

↓

Monitoring
```

---

# Docker Monitoring

Monitor

- Container status
- CPU usage
- Memory usage
- Disk usage
- Restart count
- Network traffic
- Health checks

---

# Production Health Checklist

Verify

- Docker service running
- Required containers running
- Images available
- Volumes mounted
- Networks healthy
- Disk space available
- Logs reviewed
- Monitoring healthy

---

# Common Docker Production Issues

- Docker daemon stopped
- Container restart loops
- Image pull failures
- ECR authentication failure
- Volume mount issues
- High CPU usage
- Memory leaks
- OOMKilled containers
- Full disk
- Network failures

---

# Best Practices

- Keep Docker Engine updated with supported versions.
- Configure log rotation to prevent disk exhaustion.
- Use health checks for production containers.
- Monitor CPU, memory, and restart counts continuously.
- Clean unused images, containers, and volumes regularly.
- Store images in secure registries such as Amazon ECR.
- Avoid running containers as the root user.
- Use resource limits for CPU and memory.
- Back up persistent volumes before maintenance.
- Test recovery procedures periodically.

---

# Summary

This section covered Docker production troubleshooting, including daemon failures, container startup issues, restart loops, image pull failures, Amazon ECR authentication, volume mounts, networking, DNS, resource utilization, storage cleanup, Docker Compose issues, health checks, and recovery workflows. These runbooks provide a structured process for diagnosing and restoring containerized workloads in production environments.

---

# Terraform & Infrastructure as Code Production Runbooks

---

# Introduction

Terraform production failures can impact infrastructure provisioning, deployments, networking, security, and cloud resources. This runbook provides structured troubleshooting and recovery procedures for Infrastructure as Code (IaC) incidents.

---

# Terraform Troubleshooting Workflow

```text
Alert

↓

Identify Command

↓

Review Error

↓

Check Authentication

↓

Check State

↓

Validate Configuration

↓

Recover

↓

Verify Infrastructure
```

---

# Terraform Version Check

```bash
terraform version

terraform providers
```

Verify

- Terraform version
- Provider versions
- Module compatibility

---

# terraform init Failure

## Symptoms

```text
Initialization failed
```

---

## Investigation

```bash
terraform init

terraform version
```

Verify

- Backend configuration
- Internet connectivity
- Provider versions
- Registry availability

---

## Common Causes

- Invalid backend
- Provider download failure
- Authentication issue
- Version mismatch
- Corrupted plugin cache

---

## Resolution

```bash
terraform init -upgrade

terraform init -reconfigure
```

---

# Provider Authentication Failure

## AWS Example

Verify

```bash
aws sts get-caller-identity

aws configure list
```

Review

- IAM permissions
- AWS profile
- Environment variables
- Temporary credentials

---

# terraform validate Failure

## Investigation

```bash
terraform validate
```

Common Causes

- Syntax errors
- Invalid variables
- Incorrect module references
- Unsupported arguments

---

# terraform plan Failure

## Investigation

```bash
terraform plan
```

Review

- Variable values
- Backend access
- Provider authentication
- State file
- Resource dependencies

---

# terraform apply Failure

## Investigation

```bash
terraform apply

terraform show
```

Review

- AWS API errors
- Quota limits
- Dependency failures
- IAM permissions
- Existing resources

---

# Common Apply Errors

Examples

- AccessDenied
- ResourceAlreadyExists
- LimitExceeded
- InvalidParameter
- DependencyViolation

---

# State File

Purpose

Stores the current infrastructure state managed by Terraform.

---

# State File Location

Examples

```text
Local

terraform.tfstate
```

```text
Remote

Amazon S3 Backend
```

---

# Remote State Verification

```bash
aws s3 ls

terraform state list
```

---

# State Lock

Purpose

Prevent concurrent Terraform operations.

---

# Lock Detection

Example

```text
Error acquiring state lock
```

---

## Investigation

Verify

- Active Terraform jobs
- CI/CD pipelines
- Previous failed execution

---

## Resolution

```bash
terraform force-unlock LOCK_ID
```

Only unlock after confirming no active Terraform execution.

---

# State Drift

Definition

Infrastructure changed outside Terraform.

---

## Detection

```bash
terraform plan
```

Review unexpected changes.

---

## Resolution

- Import resources
- Update configuration
- Remove manual changes

---

# Resource Import

```bash
terraform import
```

Use when

- Existing AWS resources
- Manual resource creation
- State recovery

---

# State Recovery

Commands

```bash
terraform state list

terraform state show

terraform state mv

terraform state rm
```

---

# Module Failure

## Investigation

```bash
terraform get

terraform providers

terraform validate
```

Verify

- Module source
- Version
- Variables
- Outputs

---

# Variable Issues

Verify

```bash
terraform.tfvars

variables.tf
```

Common Problems

- Missing values
- Wrong types
- Invalid defaults

---

# Backend Failure

Review

- S3 bucket
- Bucket permissions
- Backend configuration
- Region
- IAM access

---

# Resource Already Exists

Example

```text
EntityAlreadyExists

BucketAlreadyExists
```

---

## Resolution

- Import existing resource
- Rename resource
- Remove duplicate configuration

---

# Dependency Failure

Review

```text
depends_on

Resource References

Output Values
```

---

# Destroy Failure

## Investigation

```bash
terraform destroy
```

Check

- Resource dependencies
- Protection settings
- IAM permissions

---

# Output Verification

```bash
terraform output

terraform show
```

Verify infrastructure after successful deployment.

---

# Production Rollback

Workflow

```text
Failed Apply

↓

Identify Resources

↓

Rollback Configuration

↓

terraform apply

↓

Validate Infrastructure
```

---

# Safe Rollback Strategy

Never edit the state file manually unless absolutely necessary.

Preferred

- Restore previous code
- Review plan
- Apply changes
- Validate resources

---

# Infrastructure Verification

Verify

- EC2
- VPC
- Security Groups
- EKS
- ALB
- RDS
- IAM
- Route53

---

# Drift Prevention

Recommendations

- Prevent manual changes
- Restrict console access
- Use CI/CD deployments
- Enable change approvals

---

# CI/CD Pipeline Failure

Review

- Backend access
- AWS credentials
- Terraform version
- Variables
- Workspace
- State lock

---

# Workspace Issues

Verify

```bash
terraform workspace list

terraform workspace show
```

Ensure correct workspace is selected.

---

# Debug Logging

Enable

```bash
export TF_LOG=DEBUG

terraform apply
```

Disable after troubleshooting.

---

# Backup Strategy

Back up

- Terraform code
- Remote state
- Variable files
- Module versions

---

# Production Recovery Workflow

```text
Incident

↓

Terraform Error

↓

Review State

↓

Review Plan

↓

Fix Configuration

↓

Apply

↓

Verify AWS Resources

↓

Monitor
```

---

# Terraform Health Checklist

Verify

- Backend accessible
- State healthy
- No active locks
- Provider authenticated
- Variables correct
- Modules available
- Plan reviewed
- Infrastructure validated

---

# Common Terraform Production Issues

- init failure
- plan failure
- apply failure
- state lock
- state drift
- authentication failure
- backend failure
- module errors
- variable issues
- provider version mismatch

---

# Best Practices

- Store state remotely using an Amazon S3 backend.
- Enable state locking to prevent concurrent changes.
- Review every `terraform plan` before applying.
- Never modify the state file manually unless absolutely necessary.
- Version-control Terraform modules and provider versions.
- Restrict manual infrastructure changes outside Terraform.
- Use CI/CD pipelines for infrastructure deployments.
- Regularly back up Terraform state.
- Validate infrastructure after every apply operation.
- Document recovery procedures and update runbooks after incidents.

---

# Summary

This section covered Terraform production troubleshooting, including initialization failures, provider authentication, plan and apply failures, remote state management, state locking, drift detection, module issues, backend failures, rollback strategies, and production recovery workflows. These runbooks provide a structured process for safely managing and recovering Infrastructure as Code deployments in production environments.

---

# CI/CD Pipeline Production Runbooks

---

# Introduction

CI/CD pipeline failures can interrupt software delivery, delay releases, and impact production deployments. This runbook provides a structured approach for diagnosing, recovering, and preventing common CI/CD issues.

---

# CI/CD Troubleshooting Workflow

```text
Pipeline Failed

↓

Identify Failed Stage

↓

Review Logs

↓

Verify Credentials

↓

Check Infrastructure

↓

Fix Issue

↓

Re-run Pipeline

↓

Validate Deployment
```

---

# Pipeline Stuck

## Symptoms

- Pipeline queued indefinitely
- No stage execution
- Jobs pending

---

## Investigation

Review

- Available runners/agents
- Executor availability
- Pipeline queue
- Resource utilization
- Scheduler status

---

## Resolution

- Restart runner/agent
- Increase executors
- Free system resources
- Retry pipeline

---

# Git Checkout Failure

## Symptoms

```text
Repository clone failed
```

---

## Investigation

Verify

- Repository URL
- Branch name
- Git credentials
- SSH keys
- Network connectivity

---

## Commands

```bash
git clone <repo>

git ls-remote <repo>

ssh -T git@github.com
```

---

# Build Failure

## Investigation

Review

- Build logs
- Compiler errors
- Dependency versions
- Environment variables
- Build tools

---

## Common Causes

- Syntax errors
- Missing dependencies
- Incorrect environment variables
- Version mismatch
- Build script failure

---

# Unit Test Failure

## Investigation

Review

- Failed test cases
- Recent code changes
- Test reports
- Mock configurations

---

## Resolution

- Fix application code
- Update test cases
- Resolve flaky tests
- Re-run pipeline

---

# Static Code Analysis Failure

Review

- SonarQube Quality Gate
- Code coverage
- Code smells
- Security hotspots
- Critical vulnerabilities

---

## Resolution

Fix quality issues before deployment.

---

# Dependency Scan Failure

Review

- Critical CVEs
- High-risk libraries
- License violations
- Dependency versions

---

## Resolution

Update vulnerable dependencies.

---

# Container Image Build Failure

## Investigation

```bash
docker build

docker images

docker logs
```

Review

- Dockerfile
- Base image
- Build context
- Registry authentication

---

# Container Scan Failure

Review

- Critical vulnerabilities
- High vulnerabilities
- Base image age
- Operating system packages

---

## Resolution

- Update base image
- Patch dependencies
- Rebuild image

---

# Artifact Publishing Failure

## Symptoms

```text
Upload failed
```

---

## Verify

- Artifact repository
- Credentials
- Network
- Storage capacity

---

## Common Targets

- Amazon S3
- JFrog Artifactory
- Nexus Repository

---

# Deployment Failure

## Investigation

Review

- Deployment logs
- Kubernetes events
- Helm output
- Infrastructure health

---

# Kubernetes Deployment Failure

```bash
kubectl rollout status deployment/<deployment>

kubectl describe deployment

kubectl get events

kubectl logs
```

---

# Helm Deployment Failure

```bash
helm list

helm status

helm history

helm rollback
```

---

# GitHub Actions Failure

## Investigation

Review

- Workflow logs
- GitHub Secrets
- Runner status
- Action versions
- Repository permissions

---

## Verify

- Workflow syntax
- Branch protection
- Required approvals

---

# GitLab CI Failure

## Investigation

Review

- Pipeline logs
- Runner availability
- Protected variables
- CI configuration

---

## Commands

```bash
gitlab-runner status
```

---

# Jenkins Pipeline Failure

Review

- Console output
- Jenkins agents
- Credentials
- Workspace
- Plugins

---

# Runner/Agent Offline

## Investigation

Check

- CPU
- Memory
- Disk
- Network
- Service status

---

## Resolution

Restart runner service.

---

# Secret Management Failure

Verify

- AWS Secrets Manager
- Parameter Store
- Jenkins Credentials
- GitHub Secrets
- GitLab Variables

Never expose secrets in logs.

---

# Environment Variable Issues

Review

- Missing variables
- Incorrect values
- Scope
- Environment mapping

---

# Webhook Failure

Verify

- Webhook URL
- Authentication
- Firewall rules
- Repository settings
- Network connectivity

---

# Release Failure

Workflow

```text
Release

↓

Deployment Failure

↓

Rollback

↓

Validate Previous Version

↓

Monitor
```

---

# Rollback Strategy

Preferred

```text
Previous Stable Release

↓

Deploy

↓

Health Check

↓

Traffic Validation
```

---

# Canary Deployment Failure

Actions

- Stop rollout
- Route traffic back
- Investigate logs
- Fix issue
- Restart deployment

---

# Blue/Green Failure

Workflow

```text
Green Deployment

↓

Validation Failed

↓

Switch Traffic

↓

Blue Environment

↓

Recovery
```

---

# Pipeline Performance Issues

Review

- Queue time
- Build duration
- Test duration
- Image build time
- Deployment time

---

## Optimization

- Parallel stages
- Build cache
- Dependency cache
- Incremental builds
- Faster runners

---

# Release Validation

Verify

- Application health
- API responses
- Logs
- Metrics
- Alerts
- User access

---

# Deployment Verification

Check

- Pods running
- Services healthy
- Database connectivity
- ALB/Ingress
- DNS resolution
- Monitoring dashboards

---

# CI/CD Health Checklist

Verify

- Source repository reachable
- Runners healthy
- Credentials valid
- Secrets available
- Artifact repository accessible
- Deployment successful
- Monitoring healthy
- Rollback tested

---

# Production Recovery Workflow

```text
Pipeline Failure

↓

Identify Failed Stage

↓

Collect Logs

↓

Resolve Issue

↓

Re-run Pipeline

↓

Validate Deployment

↓

Monitor Production

↓

Document RCA
```

---

# Common CI/CD Production Issues

- Repository authentication failure
- Pipeline stuck
- Build failures
- Unit test failures
- SonarQube Quality Gate failure
- Dependency scan failure
- Docker build failure
- Artifact upload failure
- Deployment failure
- Runner offline

---

# Best Practices

- Protect production branches with mandatory reviews.
- Store secrets securely using dedicated secret management services.
- Validate every deployment with automated health checks.
- Use canary or blue/green deployment strategies for production.
- Automate rollback for failed deployments where possible.
- Keep CI/CD runners updated and monitored.
- Cache dependencies to improve pipeline performance.
- Fail pipelines on critical security vulnerabilities.
- Monitor deployment metrics after every release.
- Update runbooks after every production incident.

---

# Summary

This section covered CI/CD production troubleshooting, including pipeline failures, Git checkout issues, build and test failures, SonarQube quality gates, dependency scanning, Docker image builds, artifact publishing, GitHub Actions, GitLab CI, Jenkins pipelines, deployment failures, rollback strategies, runner issues, and production recovery workflows. These runbooks provide a systematic approach to restoring CI/CD pipelines and ensuring reliable software delivery.

---

