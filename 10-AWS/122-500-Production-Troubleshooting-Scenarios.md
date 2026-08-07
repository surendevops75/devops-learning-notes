# 500 Production Troubleshooting Scenarios

# Chapter 1 - Production Troubleshooting Methodology

Production systems fail.

Servers crash.

Pods restart.

Deployments fail.

Applications slow down.

Databases become overloaded.

The difference between a junior engineer and a senior engineer is **not** whether failures occur—it is **how quickly and systematically they identify the root cause and restore service**.

This handbook teaches a structured troubleshooting methodology that can be applied to Linux, Kubernetes, Docker, AWS, Terraform, CI/CD, Networking, and Cloud Infrastructure.

---

# What is Production Troubleshooting?

Production troubleshooting is the process of:

- Detecting Issues
- Identifying Symptoms
- Finding Root Cause
- Applying Corrective Actions
- Verifying Recovery
- Preventing Recurrence

The objective is not just to restore service, but also to prevent the issue from happening again.

---

# Production Incident Lifecycle

Every production incident follows a similar lifecycle.

```text
Incident

↓

Detection

↓

Alert

↓

Investigation

↓

Diagnosis

↓

Mitigation

↓

Recovery

↓

Root Cause Analysis

↓

Preventive Actions
```

Never stop at recovery—always identify the root cause.

---

# Enterprise Troubleshooting Workflow

A structured troubleshooting workflow.

```text
Problem Reported

↓

Verify Problem

↓

Collect Information

↓

Identify Scope

↓

Check Recent Changes

↓

Analyze Logs

↓

Identify Root Cause

↓

Implement Fix

↓

Verify Resolution

↓

Document RCA
```

Avoid making assumptions before gathering evidence.

---

# Golden Rules of Production Troubleshooting

Always follow these principles:

- Do not panic.
- Confirm the issue before taking action.
- Gather evidence before making changes.
- Change only one variable at a time.
- Verify every fix.
- Record every action taken.
- Perform Root Cause Analysis.
- Update documentation after resolution.

---

# The 5W1H Method

Understand the incident before troubleshooting.

| Question | Purpose |
|-----------|---------|
| What happened? | Identify the issue |
| When did it start? | Determine timeline |
| Where is the issue? | Identify affected systems |
| Who is impacted? | Determine business impact |
| Why might it have happened? | Form hypotheses |
| How can it be reproduced? | Validate findings |

---

# Incident Severity Levels

Typical enterprise classification.

| Severity | Description |
|----------|-------------|
| Sev-1 | Complete outage affecting business-critical services |
| Sev-2 | Major functionality unavailable |
| Sev-3 | Partial degradation |
| Sev-4 | Minor issue or cosmetic defect |

Severity determines response priority.

---

# Troubleshooting Pyramid

Always troubleshoot from the bottom up.

```text
Application

↓

Container

↓

Operating System

↓

Network

↓

Infrastructure

↓

Cloud Platform
```

Many "application" problems originate from lower infrastructure layers.

---

# Data Collection Checklist

Before making changes, collect:

- Error Messages
- Logs
- Metrics
- Configuration
- Recent Deployments
- Alerts
- Screenshots
- Timeline

Evidence is essential for accurate diagnosis.

---

# Recent Change Analysis

Many production issues occur shortly after changes.

Investigate:

- New Deployment
- Configuration Changes
- Infrastructure Changes
- Security Updates
- Database Changes
- DNS Changes
- Certificate Updates

Always ask:

> **"What changed?"**

---

# Troubleshooting Decision Tree

```text
Issue Reported

↓

Can It Be Reproduced?

↓

Yes

↓

Collect Logs

↓

Identify Root Cause

↓

Apply Fix

↓

Verify

↓

Close Incident
```

---

# Root Cause Analysis (RCA)

An RCA should answer:

- What happened?
- Why did it happen?
- How was it detected?
- How was it resolved?
- How can it be prevented?

Never stop at "the server crashed."

Identify the underlying cause.

---

# Example RCA

Issue:

```text
Website returned HTTP 503
```

Immediate Cause:

```text
Application Pods Not Ready
```

Root Cause:

```text
Database credentials expired
```

Preventive Action:

```text
Automate credential rotation monitoring.
```

---

# Common Troubleshooting Categories

Most production incidents fall into one of these areas:

- Linux
- Networking
- Kubernetes
- Docker
- CI/CD
- Terraform
- AWS
- Databases
- Security
- Monitoring
- Storage
- DNS

Organizing incidents by category accelerates diagnosis.

---

# Escalation Guidelines

Escalate when:

- Business-critical services are impacted.
- Multiple systems are failing.
- Root cause cannot be identified.
- Security incidents are suspected.
- Disaster recovery procedures are required.

Early escalation reduces business impact.

---

# Communication During Incidents

Communicate clearly:

- Current Status
- Business Impact
- Actions Taken
- Next Steps
- Estimated Recovery Time

Avoid speculation.

Share verified information only.

---

# Incident Timeline

Maintain a timeline.

```text
09:00 Alert Triggered

↓

09:05 Investigation Started

↓

09:15 Root Cause Identified

↓

09:20 Fix Applied

↓

09:25 Service Restored

↓

10:00 RCA Completed
```

Accurate timelines improve post-incident reviews.

---

# Enterprise Best Practices

- Follow a structured troubleshooting process.
- Collect evidence before making changes.
- Validate assumptions using logs and metrics.
- Analyze recent changes first.
- Communicate clearly during incidents.
- Verify recovery before closing incidents.
- Perform Root Cause Analysis.
- Update operational documentation after every major incident.

---

# Common Mistakes

- Restarting services without investigation.
- Ignoring logs.
- Making multiple changes simultaneously.
- Assuming the first symptom is the root cause.
- Closing incidents without RCA.
- Poor communication during outages.
- Failing to document lessons learned.

---

# Interview Questions

## Basic

1. What is production troubleshooting?
2. Why is Root Cause Analysis important?
3. What information should you collect before troubleshooting?
4. What are incident severity levels?
5. Why should recent changes be investigated first?

---

## Intermediate

1. Explain a structured production troubleshooting workflow.
2. What is the purpose of an incident timeline?
3. Why should only one change be made at a time?
4. How do you determine business impact?
5. What should be included in an RCA?

---

## Advanced

1. Design an enterprise incident response process that covers detection, investigation, mitigation, communication, recovery, and Root Cause Analysis for a cloud-native platform.
2. Explain how logs, metrics, alerts, dashboards, and recent change analysis work together to reduce Mean Time to Resolution (MTTR).
3. A multinational SaaS platform experiences intermittent production outages across multiple AWS regions. Design a troubleshooting methodology that helps engineers quickly isolate failures, coordinate incident response, minimize downtime, and continuously improve operational reliability.

---

# Chapter 2 - Linux Production Troubleshooting Scenarios (1–50)

Linux is the operating system behind most enterprise infrastructure.

Before troubleshooting Kubernetes, Docker, Jenkins, or cloud platforms, engineers must verify the health of the underlying Linux server.

This chapter covers **50 real-world Linux production troubleshooting scenarios**, focusing on systematic diagnosis, root cause identification, resolution, and preventive measures.

---

# Scenario 1 - Linux Server Is Unreachable

### Symptoms

- SSH connection fails.
- Monitoring reports the server as down.
- Applications become inaccessible.

### Investigation

- Check cloud instance status.
- Verify network connectivity.
- Confirm Security Group or firewall rules.
- Review console logs.

### Root Cause

The instance is stopped, crashed, or blocked by network policies.

### Resolution

- Start or reboot the instance.
- Correct firewall or Security Group rules.
- Restore network connectivity.

### Prevention

Implement infrastructure monitoring and automated recovery.

---

# Scenario 2 - High CPU Usage

### Symptoms

- Slow application response.
- High load average.
- Monitoring alerts.

### Investigation

```bash
top

htop

ps -eo pid,ppid,cmd,%cpu --sort=-%cpu
```

### Root Cause

A process consumes excessive CPU resources.

### Resolution

- Optimize the application.
- Restart the affected process if required.
- Scale the workload.

### Prevention

Monitor CPU utilization and configure alerts.

---

# Scenario 3 - High Memory Usage

### Symptoms

- Applications become slow.
- Out Of Memory (OOM) events.
- Server instability.

### Investigation

```bash
free -h

top

vmstat
```

### Root Cause

Memory leak or insufficient RAM.

### Resolution

- Restart the leaking application.
- Increase available memory.
- Optimize memory usage.

### Prevention

Monitor memory trends and detect leaks early.

---

# Scenario 4 - OOM Killer Terminates Application

### Symptoms

Application exits unexpectedly.

### Investigation

```bash
dmesg | grep -i oom
```

### Root Cause

Kernel OOM Killer terminated the process.

### Resolution

Reduce memory consumption or increase available RAM.

### Prevention

Set appropriate memory limits and monitor usage.

---

# Scenario 5 - Disk Usage Reaches 100%

### Symptoms

- Applications fail to write data.
- Database errors.
- Deployment failures.

### Investigation

```bash
df -h

du -sh /*
```

### Root Cause

Filesystem completely full.

### Resolution

- Remove unnecessary files.
- Archive old logs.
- Extend storage if required.

### Prevention

Enable disk usage monitoring and log rotation.

---

# Scenario 6 - Inode Exhaustion

### Symptoms

Disk has free space but new files cannot be created.

### Investigation

```bash
df -i
```

### Root Cause

All available inodes are consumed.

### Resolution

Delete unnecessary small files.

### Prevention

Monitor inode usage regularly.

---

# Scenario 7 - SSH Login Fails

### Symptoms

Users cannot connect via SSH.

### Investigation

```bash
systemctl status sshd

journalctl -u sshd
```

### Root Cause

- SSH service stopped.
- Firewall issue.
- Invalid authentication.

### Resolution

Restart SSH service and verify configuration.

### Prevention

Continuously monitor SSH availability.

---

# Scenario 8 - DNS Resolution Failure

### Symptoms

Applications cannot reach external services.

### Investigation

```bash
nslookup google.com

dig google.com
```

### Root Cause

DNS server unavailable or misconfigured.

### Resolution

Correct DNS configuration.

### Prevention

Use redundant DNS servers.

---

# Scenario 9 - Time Synchronization Failure

### Symptoms

Authentication failures.

TLS certificate validation errors.

### Investigation

```bash
timedatectl
```

### Root Cause

Incorrect system time.

### Resolution

Synchronize using NTP.

### Prevention

Monitor time synchronization services.

---

# Scenario 10 - Service Fails After Reboot

### Symptoms

Application unavailable after server restart.

### Investigation

```bash
systemctl status service-name

systemctl is-enabled service-name
```

### Root Cause

Service not enabled.

### Resolution

```bash
systemctl enable service-name
```

### Prevention

Verify service startup configuration.

---

# Scenario 11 - Network Interface Down

### Symptoms

Server loses network connectivity.

### Investigation

```bash
ip addr

ip link
```

### Root Cause

Interface disabled or configuration issue.

### Resolution

Bring interface online and verify network configuration.

---

# Scenario 12 - Packet Loss

### Investigation

```bash
ping

mtr

traceroute
```

### Root Cause

Network congestion or hardware issue.

### Resolution

Identify faulty network component.

---

# Scenario 13 - High Load Average

### Investigation

```bash
uptime

top
```

### Root Cause

CPU saturation or blocked I/O.

### Resolution

Identify bottleneck and optimize workload.

---

# Scenario 14 - Swap Usage Extremely High

### Investigation

```bash
free -h

vmstat
```

### Root Cause

Insufficient memory.

### Resolution

Optimize applications or add RAM.

---

# Scenario 15 - Zombie Processes

### Investigation

```bash
ps -el | grep Z
```

### Root Cause

Parent process failed to clean child process.

### Resolution

Restart parent process.

---

# Scenario 16 - Defunct Process Accumulation

### Investigation

```bash
ps aux | grep defunct
```

### Root Cause

Improper process management.

### Resolution

Restart responsible service.

---

# Scenario 17 - Failed Systemd Service

### Investigation

```bash
systemctl status service

journalctl -xe
```

### Root Cause

Configuration or dependency issue.

### Resolution

Correct configuration and restart.

---

# Scenario 18 - Port Already in Use

### Investigation

```bash
ss -lntp

lsof -i :8080
```

### Root Cause

Another process owns the port.

### Resolution

Stop conflicting process or use another port.

---

# Scenario 19 - File Permission Denied

### Investigation

```bash
ls -l
```

### Root Cause

Incorrect ownership or permissions.

### Resolution

```bash
chmod

chown
```

---

# Scenario 20 - SELinux Blocking Application

### Investigation

```bash
getenforce

ausearch
```

### Root Cause

SELinux policy restriction.

### Resolution

Update policy or application configuration.

---

# Scenario 21 - Log Files Growing Rapidly

### Root Cause

Application generating excessive logs.

### Resolution

Enable log rotation.

---

# Scenario 22 - Log Rotation Not Working

### Investigation

```bash
logrotate -d
```

### Root Cause

Invalid logrotate configuration.

---

# Scenario 23 - Cron Job Not Running

### Investigation

```bash
crontab -l

systemctl status crond
```

### Root Cause

Cron service stopped or incorrect schedule.

---

# Scenario 24 - Incorrect File Ownership

### Investigation

```bash
ls -l
```

### Resolution

```bash
chown
```

---

# Scenario 25 - Package Installation Failure

### Investigation

```bash
dnf install

apt install
```

### Root Cause

Repository unavailable or dependency issue.

---

# Scenario 26 - Repository Unreachable

### Investigation

Verify internet connectivity and repository configuration.

---

# Scenario 27 - Filesystem Mounted Read-Only

### Investigation

```bash
mount
```

### Root Cause

Filesystem corruption.

---

# Scenario 28 - Filesystem Corruption

### Investigation

```bash
fsck
```

---

# Scenario 29 - High Disk I/O

### Investigation

```bash
iostat

iotop
```

---

# Scenario 30 - Slow File Operations

### Investigation

Check storage latency and disk utilization.

---

# Scenario 31 - Authentication Failures

Investigate:

```bash
/var/log/secure
```

---

# Scenario 32 - Excessive Failed SSH Attempts

Investigate authentication logs.

Implement Fail2Ban or firewall rules.

---

# Scenario 33 - User Account Locked

Verify PAM configuration and authentication logs.

---

# Scenario 34 - Password Expired

Use:

```bash
chage
```

---

# Scenario 35 - Server Boot Failure

Investigate GRUB configuration and boot logs.

---

# Scenario 36 - Kernel Panic

Review console logs and recent kernel updates.

---

# Scenario 37 - Kernel Upgrade Failure

Verify installed kernel versions.

Rollback if required.

---

# Scenario 38 - Network Routing Issue

Investigate:

```bash
ip route
```

---

# Scenario 39 - Firewall Blocking Traffic

Verify:

```bash
firewall-cmd

iptables
```

---

# Scenario 40 - DNS Configuration Incorrect

Review:

```text
/etc/resolv.conf
```

---

# Scenario 41 - SSL Certificate Expired

Verify certificate expiration.

Renew and reload services.

---

# Scenario 42 - System Clock Drift

Synchronize with NTP.

---

# Scenario 43 - Unexpected Server Reboot

Review:

```bash
last reboot

journalctl
```

---

# Scenario 44 - Excessive Process Creation

Investigate parent processes.

---

# Scenario 45 - Service Starts Slowly

Review dependencies and startup logs.

---

# Scenario 46 - Application Cannot Write Files

Check permissions, ownership, and available disk space.

---

# Scenario 47 - Backup Script Failure

Verify storage availability and permissions.

---

# Scenario 48 - NFS Mount Failure

Verify server availability and mount configuration.

---

# Scenario 49 - Performance Degradation After Patch

Compare:

- Kernel Version
- Packages
- Configuration Changes

Rollback if necessary.

---

# Scenario 50 - Random System Slowdowns

### Investigation

Review:

- CPU
- Memory
- Disk
- Network
- Logs
- Recent Changes

### Root Cause

Typically resource contention or configuration drift.

### Resolution

Identify the bottleneck using metrics and logs.

### Prevention

Continuous monitoring and capacity planning.

---

# Chapter 3 - Docker Production Troubleshooting Scenarios (51–100)

Docker is the foundation of modern containerized applications.

Most Kubernetes workloads, CI/CD pipelines, and microservices depend on Docker containers.

This chapter covers **50 real-world Docker production troubleshooting scenarios**, including image issues, container failures, networking, storage, security, and performance.

---

# Scenario 51 - Container Exits Immediately

### Symptoms

- Container starts and exits within seconds.
- `docker ps` shows no running container.

### Investigation

```bash
docker ps -a

docker logs <container-id>

docker inspect <container-id>
```

### Root Cause

The main application process exited.

### Resolution

Fix the application startup command or entrypoint.

### Prevention

Validate the Dockerfile and startup scripts before deployment.

---

# Scenario 52 - Container Keeps Restarting

### Symptoms

- Restart count continuously increases.
- Service unavailable.

### Investigation

```bash
docker ps

docker inspect <container>

docker logs <container>
```

### Root Cause

Application crash or incorrect restart policy.

### Resolution

Fix the application or configuration.

### Prevention

Implement proper health checks and startup validation.

---

# Scenario 53 - Image Pull Failure

### Symptoms

```
ImagePullBackOff

pull access denied
```

### Investigation

```bash
docker pull image-name

docker login
```

### Root Cause

- Invalid image name
- Authentication failure
- Missing repository

### Resolution

Correct image reference and registry credentials.

---

# Scenario 54 - Docker Build Failure

### Investigation

```bash
docker build .
```

Review:

- Dockerfile
- Build logs
- Missing dependencies

### Root Cause

Invalid Dockerfile or build errors.

---

# Scenario 55 - Container Cannot Reach Internet

### Investigation

```bash
docker exec -it container bash

ping google.com

curl google.com
```

### Root Cause

Docker network misconfiguration.

### Resolution

Verify bridge network and DNS.

---

# Scenario 56 - DNS Resolution Failure Inside Container

### Investigation

```bash
cat /etc/resolv.conf

nslookup google.com
```

### Root Cause

Incorrect Docker DNS configuration.

---

# Scenario 57 - Container Cannot Reach Database

### Investigation

- Verify database endpoint.
- Test connectivity.
- Check firewall.

### Root Cause

Network configuration or incorrect credentials.

---

# Scenario 58 - Port Mapping Incorrect

### Investigation

```bash
docker ps

docker port container
```

### Root Cause

Incorrect port publishing.

### Resolution

Expose the correct ports.

---

# Scenario 59 - Volume Not Mounted

### Investigation

```bash
docker inspect container
```

### Root Cause

Incorrect bind mount configuration.

---

# Scenario 60 - Data Lost After Restart

### Investigation

Verify whether persistent volumes are configured.

### Root Cause

Container filesystem used instead of volumes.

---

# Scenario 61 - High Container CPU Usage

### Investigation

```bash
docker stats
```

### Root Cause

Application consuming excessive CPU.

---

# Scenario 62 - High Memory Usage

### Investigation

```bash
docker stats
```

Review application memory behavior.

---

# Scenario 63 - Container OOMKilled

### Investigation

```bash
docker inspect container
```

Review OOM events.

### Resolution

Increase memory limit or optimize the application.

---

# Scenario 64 - Disk Space Consumed by Images

### Investigation

```bash
docker system df
```

### Resolution

```bash
docker image prune
```

---

# Scenario 65 - Too Many Stopped Containers

### Investigation

```bash
docker ps -a
```

### Resolution

```bash
docker container prune
```

---

# Scenario 66 - Dangling Images

### Investigation

```bash
docker images
```

### Resolution

```bash
docker image prune
```

---

# Scenario 67 - Container Logs Too Large

### Investigation

```bash
docker logs container
```

### Resolution

Configure Docker log rotation.

---

# Scenario 68 - Docker Daemon Not Running

### Investigation

```bash
systemctl status docker
```

### Resolution

Restart Docker service.

---

# Scenario 69 - Docker Service Fails to Start

Review:

```bash
journalctl -u docker
```

Check daemon configuration.

---

# Scenario 70 - Docker Socket Permission Denied

### Investigation

```bash
ls -l /var/run/docker.sock
```

Verify user permissions.

---

# Scenario 71 - Invalid Dockerfile

Review:

- FROM
- COPY
- CMD
- ENTRYPOINT

---

# Scenario 72 - COPY Command Failure

Verify source path and build context.

---

# Scenario 73 - ENTRYPOINT Failure

Inspect startup command.

---

# Scenario 74 - CMD Not Executed

Review interaction between CMD and ENTRYPOINT.

---

# Scenario 75 - Environment Variables Missing

### Investigation

```bash
docker inspect
```

Verify environment configuration.

---

# Scenario 76 - Secrets Exposed

Review:

- Dockerfile
- Environment Variables
- Image Layers

---

# Scenario 77 - Container Health Check Failing

### Investigation

```bash
docker inspect
```

Review health status.

---

# Scenario 78 - Registry Authentication Failure

Verify:

```bash
docker login
```

---

# Scenario 79 - Private Registry Unreachable

Check:

- DNS
- TLS
- Firewall
- Authentication

---

# Scenario 80 - Slow Docker Build

Review:

- Build Context
- Layer Cache
- Large Files

---

# Scenario 81 - Image Too Large

Analyze:

```bash
docker history image
```

---

# Scenario 82 - Build Cache Not Used

Verify Dockerfile ordering.

---

# Scenario 83 - Network Conflict

Review Docker bridge networks.

---

# Scenario 84 - Container Cannot Communicate with Another Container

Verify:

```bash
docker network ls

docker network inspect
```

---

# Scenario 85 - Overlay Network Failure

Review Docker Swarm networking.

---

# Scenario 86 - Container Time Incorrect

Verify host time synchronization.

---

# Scenario 87 - Certificate Error

Review:

- Certificates
- Expiration
- Trust Store

---

# Scenario 88 - Docker Compose Failure

Review:

```bash
docker compose config
```

---

# Scenario 89 - Compose Service Dependency Failure

Verify service startup order.

---

# Scenario 90 - BuildKit Failure

Disable temporarily for testing.

Review build configuration.

---

# Scenario 91 - Resource Limits Too Low

Review:

- CPU Limits
- Memory Limits

---

# Scenario 92 - Image Vulnerabilities

Scan image.

Review outdated packages.

---

# Scenario 93 - Container Running as Root

Review Dockerfile.

Configure non-root user.

---

# Scenario 94 - File Permission Errors

Verify mounted volume ownership.

---

# Scenario 95 - Slow Container Startup

Review:

- Initialization
- Database Connection
- External Dependencies

---

# Scenario 96 - Application Configuration Missing

Verify mounted configuration files.

---

# Scenario 97 - Docker Engine Upgrade Failure

Review:

- Compatibility
- Logs
- Configuration

---

# Scenario 98 - Image Tag Mismatch

Verify deployment references correct tag.

---

# Scenario 99 - Registry Rate Limiting

Review registry usage.

Implement authenticated pulls.

---

# Scenario 100 - Random Container Failures

### Investigation

Review:

- Logs
- Resource Usage
- Restart History
- Docker Events
- Recent Deployments

### Root Cause

Usually application instability, infrastructure issues, or configuration drift.

### Resolution

Collect evidence before restarting containers.

### Prevention

Implement monitoring, health checks, resource limits, and deployment validation.

---

# Chapter 4 - Kubernetes Production Troubleshooting Scenarios (101–150)

Kubernetes simplifies application deployment and scaling, but it also introduces new operational challenges.

Production issues can occur at multiple layers:

- Cluster
- Control Plane
- Worker Nodes
- Pods
- Deployments
- Services
- Networking
- Storage
- Security

This chapter covers **50 real-world Kubernetes production troubleshooting scenarios**, including diagnosis, root cause analysis, resolution, and prevention.

---

# Scenario 101 - Pod Stuck in Pending State

### Symptoms

- Pod never starts.
- Status remains **Pending**.

### Investigation

```bash
kubectl get pods

kubectl describe pod <pod-name>
```

### Root Cause

- Insufficient CPU or Memory
- Node Selector mismatch
- Taints & Tolerations
- PVC not bound

### Resolution

Resolve scheduling constraints.

### Prevention

Monitor cluster capacity and validate scheduling rules.

---

# Scenario 102 - CrashLoopBackOff

### Symptoms

Pod continuously restarts.

### Investigation

```bash
kubectl logs <pod> --previous

kubectl describe pod <pod>
```

### Root Cause

- Application crash
- Missing configuration
- Database connectivity failure
- Incorrect startup command

### Resolution

Fix application startup issue.

---

# Scenario 103 - ImagePullBackOff

### Investigation

```bash
kubectl describe pod
```

### Root Cause

- Wrong image name
- Missing image tag
- Registry authentication failure

### Resolution

Correct image reference and registry credentials.

---

# Scenario 104 - ErrImagePull

### Investigation

Verify:

- Image Repository
- Registry Access
- Network Connectivity

---

# Scenario 105 - Pod OOMKilled

### Investigation

```bash
kubectl describe pod
```

Review:

- Memory Limits
- Events

### Root Cause

Application exceeded memory limit.

### Resolution

Increase limits or optimize memory usage.

---

# Scenario 106 - Liveness Probe Failure

### Investigation

```bash
kubectl describe pod
```

### Root Cause

Health check endpoint failing.

### Resolution

Fix application or probe configuration.

---

# Scenario 107 - Readiness Probe Failure

### Symptoms

Pod Running but Service unavailable.

### Root Cause

Readiness probe fails.

### Resolution

Correct readiness configuration.

---

# Scenario 108 - Startup Probe Failure

Application initialization takes longer than configured.

Adjust startup probe thresholds.

---

# Scenario 109 - Pod Evicted

### Investigation

```bash
kubectl describe pod
```

### Root Cause

Node resource pressure.

---

# Scenario 110 - Node NotReady

### Investigation

```bash
kubectl get nodes

kubectl describe node
```

### Root Cause

- Kubelet failure
- Network issue
- Resource exhaustion

---

# Scenario 111 - Node Disk Pressure

Review:

```bash
df -h
```

Free disk space.

---

# Scenario 112 - Node Memory Pressure

Review node memory utilization.

Optimize workloads.

---

# Scenario 113 - Node PID Pressure

Too many running processes.

Review process count.

---

# Scenario 114 - Deployment Stuck

### Investigation

```bash
kubectl rollout status deployment
```

Review events.

---

# Scenario 115 - Rollout Failure

Investigate:

```bash
kubectl rollout history
```

Rollback if required.

---

# Scenario 116 - Deployment Rollback Failure

Review previous ReplicaSets.

Verify image availability.

---

# Scenario 117 - ReplicaSet Not Creating Pods

Review deployment events.

---

# Scenario 118 - DaemonSet Not Running

Verify:

- Node Labels
- Taints
- Scheduling Rules

---

# Scenario 119 - StatefulSet Pod Failure

Review:

- PVC
- Storage
- Pod Identity

---

# Scenario 120 - Job Not Completing

Review:

```bash
kubectl logs job-name
```

---

# Scenario 121 - CronJob Not Executing

Verify:

```bash
kubectl describe cronjob
```

---

# Scenario 122 - Service Unreachable

Review:

```bash
kubectl get svc

kubectl describe svc
```

---

# Scenario 123 - Endpoint Missing

Verify:

```bash
kubectl get endpoints
```

---

# Scenario 124 - ClusterIP Not Working

Review Service selector.

---

# Scenario 125 - NodePort Not Accessible

Verify firewall and node networking.

---

# Scenario 126 - LoadBalancer Pending

Cloud Load Balancer provisioning failed.

Verify cloud integration.

---

# Scenario 127 - Ingress Returning 404

Review:

- Host Rules
- Path Rules
- Backend Service

---

# Scenario 128 - Ingress TLS Failure

Verify certificate and secret.

---

# Scenario 129 - DNS Resolution Failure

Review CoreDNS.

```bash
kubectl get pods -n kube-system
```

---

# Scenario 130 - CoreDNS CrashLoopBackOff

Investigate logs.

Restart after configuration fix.

---

# Scenario 131 - PersistentVolume Pending

Verify StorageClass.

---

# Scenario 132 - PVC Not Bound

Review:

```bash
kubectl describe pvc
```

---

# Scenario 133 - StorageClass Missing

Verify StorageClass exists.

---

# Scenario 134 - Secret Missing

Review:

```bash
kubectl get secret
```

---

# Scenario 135 - ConfigMap Missing

Verify ConfigMap creation.

---

# Scenario 136 - Environment Variable Missing

Review Deployment manifest.

---

# Scenario 137 - Pod Cannot Reach Database

Verify:

- Service
- DNS
- Network Policy

---

# Scenario 138 - NetworkPolicy Blocking Traffic

Review ingress and egress rules.

---

# Scenario 139 - Pod-to-Pod Communication Failure

Verify CNI plugin.

---

# Scenario 140 - High API Server Latency

Review:

- Control Plane Metrics
- etcd Performance

---

# Scenario 141 - etcd Performance Degradation

Monitor:

- Disk Latency
- Storage
- CPU

---

# Scenario 142 - Control Plane Unavailable

Verify control plane components.

---

# Scenario 143 - Authentication Failure

Review RBAC configuration.

---

# Scenario 144 - Authorization Denied

Check RoleBindings and ClusterRoleBindings.

---

# Scenario 145 - HPA Not Scaling

Verify:

- Metrics Server
- Resource Requests
- CPU Metrics

---

# Scenario 146 - Metrics Server Failure

Check Metrics Server deployment.

---

# Scenario 147 - Container Cannot Pull Secret

Review ImagePullSecrets.

---

# Scenario 148 - Excessive Pod Restarts

Investigate:

- Application Logs
- Health Checks
- Resource Limits

---

# Scenario 149 - Slow Deployment

Review:

- Image Pull Time
- Scheduling
- Readiness Probes

---

# Scenario 150 - Random Kubernetes Failures

### Investigation

Review:

- Cluster Events
- Pod Logs
- Node Health
- Metrics
- Recent Deployments
- Resource Utilization

### Root Cause

Usually infrastructure, application, or configuration issues.

### Resolution

Use a systematic troubleshooting approach.

### Prevention

Implement monitoring, alerts, resource limits, health probes, and deployment validation.

---

# Chapter 5 - AWS Production Troubleshooting Scenarios (151–200)

AWS provides the infrastructure that powers many enterprise applications.

Production issues can occur in:

- EC2
- VPC
- IAM
- Load Balancers
- Auto Scaling
- EKS
- Route53
- S3
- RDS
- Security Groups

This chapter covers **50 real-world AWS production troubleshooting scenarios** commonly encountered by DevOps Engineers.

---

# Scenario 151 - EC2 Instance Unreachable

### Symptoms

- SSH timeout.
- Monitoring reports the instance as down.
- Application unavailable.

### Investigation

- Check EC2 instance state.
- Review Security Groups.
- Verify Network ACLs.
- Check system status checks.

### Root Cause

Instance stopped, crashed, or blocked by networking.

### Resolution

Restart instance or correct network configuration.

### Prevention

Enable CloudWatch alarms and Auto Recovery.

---

# Scenario 152 - EC2 Status Check Failed

### Investigation

Review:

- System Status Check
- Instance Status Check
- Console Logs

### Root Cause

Hardware issue or operating system failure.

### Resolution

Reboot or recover the instance.

---

# Scenario 153 - SSH Access Denied

### Investigation

Verify:

- Key Pair
- Security Group
- Network ACL
- SSH Service

### Root Cause

Authentication or firewall issue.

---

# Scenario 154 - Security Group Blocking Traffic

### Symptoms

Application inaccessible.

### Investigation

Review inbound and outbound rules.

### Resolution

Allow required ports.

---

# Scenario 155 - Network ACL Blocking Traffic

### Investigation

Verify subnet ACL rules.

### Root Cause

Explicit deny or missing allow rule.

---

# Scenario 156 - Internet Gateway Missing

### Symptoms

Public instances cannot reach the internet.

### Investigation

Verify Internet Gateway attachment.

---

# Scenario 157 - NAT Gateway Failure

### Symptoms

Private instances lose outbound internet access.

### Investigation

Verify:

- NAT Gateway
- Route Tables
- Elastic IP

---

# Scenario 158 - Route Table Misconfiguration

### Investigation

Review subnet associations.

### Resolution

Correct routing entries.

---

# Scenario 159 - VPC Peering Failure

Verify:

- Routes
- CIDR overlap
- Peering status

---

# Scenario 160 - DNS Resolution Failure

Review:

- Route53
- VPC DNS Settings
- Resolver Configuration

---

# Scenario 161 - Route53 Record Incorrect

Verify:

- Record Type
- Alias Target
- TTL

---

# Scenario 162 - Load Balancer Returns 503

### Investigation

Check:

- Target Group
- Health Checks
- Backend Instances

### Root Cause

No healthy targets.

---

# Scenario 163 - Load Balancer Health Check Failure

Review:

- Health Check Path
- Port
- Application Status

---

# Scenario 164 - Auto Scaling Not Launching Instances

Verify:

- Launch Template
- Scaling Policy
- Capacity Limits

---

# Scenario 165 - Auto Scaling Not Terminating Instances

Review cooldown periods and scaling policies.

---

# Scenario 166 - Launch Template Incorrect

Verify:

- AMI
- Security Groups
- IAM Role
- User Data

---

# Scenario 167 - IAM Permission Denied

### Symptoms

AWS API returns **AccessDenied**.

### Investigation

Review IAM policies and roles.

---

# Scenario 168 - IAM Role Missing

Verify EC2 instance profile attachment.

---

# Scenario 169 - STS AssumeRole Failure

Review trust relationship and permissions.

---

# Scenario 170 - S3 Access Denied

### Investigation

Review:

- Bucket Policy
- IAM Policy
- ACL
- Encryption Settings

---

# Scenario 171 - S3 Upload Failure

Verify:

- Permissions
- Storage Class
- Object Ownership

---

# Scenario 172 - S3 Lifecycle Not Working

Review lifecycle rules and prefixes.

---

# Scenario 173 - RDS Connection Failure

### Investigation

Verify:

- Endpoint
- Security Groups
- Database Availability
- Credentials

---

# Scenario 174 - RDS High CPU

Review:

- Slow Queries
- Connections
- Performance Insights

---

# Scenario 175 - RDS Storage Full

Increase storage or remove unnecessary data.

---

# Scenario 176 - Read Replica Lag

Monitor replication delay.

Investigate write workload.

---

# Scenario 177 - Multi-AZ Failover

Review failover events.

Verify application reconnection.

---

# Scenario 178 - EBS Volume Full

Review:

```bash
df -h
```

Extend volume if required.

---

# Scenario 179 - EBS Performance Degradation

Monitor:

- IOPS
- Throughput
- Queue Length

---

# Scenario 180 - Snapshot Failure

Verify IAM permissions and storage status.

---

# Scenario 181 - EKS Worker Node Not Joining

Review:

- Bootstrap Logs
- IAM Role
- Security Groups

---

# Scenario 182 - EKS API Unreachable

Verify cluster endpoint accessibility.

---

# Scenario 183 - CloudWatch Alarm Not Triggering

Review:

- Metric
- Threshold
- Evaluation Period

---

# Scenario 184 - CloudWatch Logs Missing

Verify log agent configuration.

---

# Scenario 185 - Lambda Timeout

Increase timeout or optimize function.

---

# Scenario 186 - Lambda Permission Failure

Review execution role.

---

# Scenario 187 - API Gateway Returns 502

Verify backend integration.

---

# Scenario 188 - ACM Certificate Validation Failure

Review DNS validation records.

---

# Scenario 189 - ALB SSL Certificate Error

Verify:

- ACM Certificate
- Listener Configuration

---

# Scenario 190 - Elastic IP Not Associated

Review instance association.

---

# Scenario 191 - VPC Endpoint Failure

Verify endpoint policy and routing.

---

# Scenario 192 - CloudFormation Stack Failure

Review Events tab.

Identify failed resource.

---

# Scenario 193 - AWS CLI Authentication Failure

Verify:

```bash
aws configure

aws sts get-caller-identity
```

---

# Scenario 194 - AWS Service Quota Exceeded

Review Service Quotas.

Request limit increase if required.

---

# Scenario 195 - Cross-Region Replication Failure

Verify replication configuration and permissions.

---

# Scenario 196 - KMS Key Access Denied

Review key policy and IAM permissions.

---

# Scenario 197 - Secrets Manager Access Failure

Verify IAM permissions and secret policy.

---

# Scenario 198 - Elastic Load Balancer High Latency

Review:

- Backend Health
- Target Response Time
- Network Performance

---

# Scenario 199 - Unexpected AWS Cost Increase

Investigate:

- Running Instances
- Storage
- Data Transfer
- Idle Resources

Implement cost optimization.

---

# Scenario 200 - Random AWS Production Failure

### Investigation

Review:

- CloudWatch Metrics
- AWS Health Dashboard
- Recent Deployments
- IAM Changes
- Networking
- Infrastructure Changes

### Root Cause

Usually infrastructure configuration, resource limits, deployment changes, or networking.

### Resolution

Follow a structured investigation process.

### Prevention

Implement monitoring, Infrastructure as Code, automated validation, and continuous operational reviews.

---

# Chapter 6 - Jenkins, GitHub Actions & CI/CD Production Troubleshooting Scenarios (201–250)

CI/CD pipelines are the backbone of modern software delivery.

When a pipeline fails, deployments stop, releases are delayed, and production incidents become more likely.

This chapter covers **50 real-world CI/CD troubleshooting scenarios** involving:

- Jenkins
- GitHub Actions
- Git
- Build Tools
- Artifact Management
- Deployment Automation
- Secrets Management
- Pipeline Performance

---

# Scenario 201 - Jenkins Server Unreachable

### Symptoms

- Jenkins UI inaccessible.
- Builds cannot start.

### Investigation

```bash
systemctl status jenkins

journalctl -u jenkins
```

### Root Cause

Jenkins service stopped or server unavailable.

### Resolution

Restart Jenkins and investigate logs.

### Prevention

Enable service monitoring and automatic recovery.

---

# Scenario 202 - Jenkins Service Fails to Start

### Investigation

```bash
journalctl -u jenkins
```

Review:

- Java version
- Configuration
- Plugin errors

### Root Cause

Configuration or dependency issue.

---

# Scenario 203 - Jenkins Job Stuck in Queue

### Symptoms

Build never starts.

### Investigation

Review:

- Available executors
- Offline agents
- Queue status

### Root Cause

No available build executor.

---

# Scenario 204 - Jenkins Agent Offline

### Investigation

Verify:

- Agent connectivity
- SSH connection
- JNLP connection
- Agent logs

### Root Cause

Network issue or agent service failure.

---

# Scenario 205 - Git Clone Failure

### Investigation

Review:

```bash
git clone
```

Check:

- Repository URL
- Credentials
- Network

### Root Cause

Authentication or repository access issue.

---

# Scenario 206 - Git Authentication Failed

### Investigation

Verify:

- SSH Keys
- Personal Access Token
- Credentials

---

# Scenario 207 - Git Merge Conflict

### Investigation

Review conflicting files.

### Resolution

Resolve conflicts and commit.

### Prevention

Frequent rebasing and smaller pull requests.

---

# Scenario 208 - Pipeline Syntax Error

### Investigation

Validate:

- Jenkinsfile
- YAML syntax
- Script formatting

---

# Scenario 209 - Jenkinsfile Not Found

Verify repository structure and pipeline configuration.

---

# Scenario 210 - GitHub Actions Workflow Not Triggered

### Investigation

Verify:

- Trigger event
- Branch
- Workflow location

### Root Cause

Workflow trigger misconfiguration.

---

# Scenario 211 - GitHub Actions Runner Offline

Review runner status.

Restart runner if required.

---

# Scenario 212 - Runner Permission Denied

Verify runner user permissions.

---

# Scenario 213 - Pipeline Environment Variables Missing

Review:

- Environment section
- Secrets
- Variable scope

---

# Scenario 214 - Secret Not Available

Verify:

- Jenkins Credentials
- GitHub Secrets
- Secret names

---

# Scenario 215 - Build Failure

### Investigation

Review build logs.

Check:

- Dependencies
- Build configuration
- Source changes

---

# Scenario 216 - Maven Build Failure

Verify:

- pom.xml
- Repository access
- Dependencies

---

# Scenario 217 - Gradle Build Failure

Review Gradle logs.

Check dependency resolution.

---

# Scenario 218 - npm Build Failure

Verify:

```bash
package.json

package-lock.json
```

---

# Scenario 219 - Docker Build Failure in Pipeline

Review Docker build logs.

Check Dockerfile.

---

# Scenario 220 - Docker Push Failure

Verify:

- Registry authentication
- Repository permissions

---

# Scenario 221 - Artifact Upload Failure

Review artifact repository connectivity.

---

# Scenario 222 - Artifact Download Failure

Verify:

- Repository
- Credentials
- Artifact version

---

# Scenario 223 - Pipeline Timeout

Review:

- Long-running stages
- External dependencies

Optimize execution.

---

# Scenario 224 - Pipeline Hanging

Review waiting stages.

Check blocked resources.

---

# Scenario 225 - Workspace Full

Review:

```bash
df -h
```

Clean old workspaces.

---

# Scenario 226 - Disk Full on Jenkins Server

Remove unused:

- Builds
- Logs
- Docker Images
- Workspaces

---

# Scenario 227 - Pipeline Slow

Investigate:

- Build Cache
- Dependency Downloads
- Parallelization

---

# Scenario 228 - Parallel Stage Failure

Review shared resource conflicts.

---

# Scenario 229 - Test Stage Failure

Investigate:

- Unit Tests
- Integration Tests
- Environment

---

# Scenario 230 - Flaky Tests

Identify unstable tests.

Stabilize before deployment.

---

# Scenario 231 - Deployment Stage Failure

Verify:

- Deployment scripts
- Kubernetes
- Infrastructure

---

# Scenario 232 - Rollback Failure

Review previous deployment artifacts.

---

# Scenario 233 - Deployment Succeeds but Application Fails

Review:

- Application Logs
- Health Checks
- Database Connectivity

---

# Scenario 234 - Kubernetes Deployment Failure

Verify:

- kubectl access
- Namespace
- Image
- Secrets

---

# Scenario 235 - Terraform Stage Failure

Review:

```bash
terraform plan

terraform apply
```

---

# Scenario 236 - Terraform State Lock

Verify lock status.

Release only after confirming no active operation.

---

# Scenario 237 - Pipeline Credential Expired

Update credentials.

Verify secret rotation.

---

# Scenario 238 - Expired Certificate in Pipeline

Renew certificate.

Restart affected services.

---

# Scenario 239 - SCM Polling Not Working

Verify:

- Webhook
- Poll schedule
- Repository access

---

# Scenario 240 - Webhook Failure

Check:

- Delivery logs
- Firewall
- Endpoint availability

---

# Scenario 241 - Plugin Compatibility Issue

Review plugin versions.

Update carefully after testing.

---

# Scenario 242 - Jenkins Upgrade Failure

Verify:

- Plugin compatibility
- Java version
- Backup

---

# Scenario 243 - GitHub API Rate Limit

Reduce unnecessary API requests.

Use authenticated access.

---

# Scenario 244 - Pipeline Permission Denied

Review:

- File permissions
- User permissions
- Execution rights

---

# Scenario 245 - Build Cache Corruption

Clean cache and rebuild.

---

# Scenario 246 - Pipeline Produces Different Results

Investigate:

- Environment differences
- Dependency versions
- Build reproducibility

---

# Scenario 247 - Notification Failure

Verify:

- Email server
- Slack webhook
- Teams webhook

---

# Scenario 248 - Scheduled Build Not Running

Review cron schedule and Jenkins scheduler.

---

# Scenario 249 - Pipeline Works Locally but Fails in CI

Compare:

- Environment variables
- Dependency versions
- Operating System
- File paths

---

# Scenario 250 - Random CI/CD Pipeline Failures

### Investigation

Review:

- Build Logs
- Agent Health
- Infrastructure
- Network
- Recent Changes
- Dependency Versions

### Root Cause

Usually environment inconsistency, infrastructure instability, dependency issues, or configuration drift.

### Resolution

Use systematic troubleshooting and compare successful vs failed executions.

### Prevention

Implement reproducible builds, standardized environments, monitoring, and pipeline validation.

---

# Chapter 7 - Terraform Production Troubleshooting Scenarios (251–300)

Terraform enables Infrastructure as Code (IaC), but production issues can occur during planning, provisioning, state management, module usage, or cloud resource creation.

This chapter covers **50 real-world Terraform production troubleshooting scenarios** commonly encountered in enterprise environments.

---

# Scenario 251 - Terraform Init Fails

### Symptoms

- Initialization fails.
- Providers cannot be downloaded.

### Investigation

```bash
terraform init
```

Review:

- Internet connectivity
- Provider source
- Backend configuration

### Root Cause

Invalid provider or backend configuration.

### Resolution

Correct configuration and re-run `terraform init`.

### Prevention

Pin provider versions and validate backend settings.

---

# Scenario 252 - Terraform Plan Fails

### Investigation

```bash
terraform plan
```

Review:

- Variable values
- Syntax errors
- Provider authentication

### Root Cause

Configuration or authentication issue.

---

# Scenario 253 - Terraform Apply Fails

### Symptoms

Resources are partially created.

### Investigation

Review apply output.

Identify the failed resource.

### Resolution

Fix the error and re-run `terraform apply`.

---

# Scenario 254 - State Lock Error

### Symptoms

```
Error acquiring the state lock
```

### Investigation

Verify whether another Terraform operation is running.

### Root Cause

Existing state lock.

### Resolution

Wait for the active operation or safely release the lock.

---

# Scenario 255 - State File Corruption

### Investigation

Verify remote backend.

Review backup state.

### Resolution

Restore a valid state backup.

---

# Scenario 256 - Backend Initialization Failure

Review:

- Backend bucket
- Access permissions
- Configuration

---

# Scenario 257 - Provider Authentication Failure

### Investigation

```bash
terraform providers
```

Verify cloud credentials.

---

# Scenario 258 - Invalid Variable Value

Review:

```bash
terraform validate
```

Verify variable definitions.

---

# Scenario 259 - Module Source Not Found

Review:

- Module path
- Git URL
- Version

---

# Scenario 260 - Module Version Conflict

Verify module version compatibility.

---

# Scenario 261 - Resource Already Exists

### Symptoms

Terraform attempts to create an existing resource.

### Resolution

Import the resource into Terraform state.

---

# Scenario 262 - Resource Drift

### Investigation

```bash
terraform plan
```

### Root Cause

Manual infrastructure changes.

### Resolution

Reconcile infrastructure with Terraform configuration.

---

# Scenario 263 - Import Failure

Review resource ID format.

Verify permissions.

---

# Scenario 264 - Destroy Failure

Investigate dependency errors.

Review resource relationships.

---

# Scenario 265 - Dependency Cycle

### Symptoms

```
Cycle detected
```

### Resolution

Refactor dependencies.

Remove circular references.

---

# Scenario 266 - Invalid Resource Reference

Verify resource names and outputs.

---

# Scenario 267 - Output Value Missing

Review output definitions.

Verify resource creation.

---

# Scenario 268 - Data Source Failure

Review:

- Data source configuration
- Permissions
- Resource availability

---

# Scenario 269 - Invalid Terraform Syntax

Run:

```bash
terraform fmt

terraform validate
```

---

# Scenario 270 - Unsupported Provider Version

Update provider constraints.

Reinitialize providers.

---

# Scenario 271 - Terraform Workspace Confusion

Verify active workspace.

```bash
terraform workspace list

terraform workspace show
```

---

# Scenario 272 - Wrong Environment Deployed

Verify:

- Workspace
- Variables
- Backend

---

# Scenario 273 - Sensitive Values Exposed

Review:

- Outputs
- Variables
- Logs

Mark sensitive values appropriately.

---

# Scenario 274 - Backend Permission Denied

Verify cloud IAM permissions.

---

# Scenario 275 - S3 Backend Unreachable

Review:

- Bucket
- Region
- Credentials
- Network

---

# Scenario 276 - State File Deleted

Restore from backup.

Verify backend recovery.

---

# Scenario 277 - Parallel Apply Conflict

Avoid concurrent Terraform executions.

---

# Scenario 278 - Resource Timeout

Increase timeout if appropriate.

Investigate cloud API delays.

---

# Scenario 279 - API Rate Limiting

Reduce parallelism.

Retry after delay.

---

# Scenario 280 - Cloud Resource Limit Reached

Review service quotas.

Request quota increase.

---

# Scenario 281 - Invalid AMI ID

Verify AMI availability in the selected region.

---

# Scenario 282 - Invalid Security Group

Verify Security Group ID.

Review dependencies.

---

# Scenario 283 - Invalid Subnet

Review subnet ID and Availability Zone.

---

# Scenario 284 - Invalid IAM Role

Verify role exists and trust policy.

---

# Scenario 285 - Invalid Route Table

Review VPC routing configuration.

---

# Scenario 286 - Resource Recreation Unexpected

Review lifecycle configuration.

Check changed arguments.

---

# Scenario 287 - Lifecycle Ignore Changes Not Working

Verify lifecycle block syntax.

---

# Scenario 288 - Count Index Error

Review:

- count
- references
- indexing

---

# Scenario 289 - for_each Key Changed

Changing keys causes resource recreation.

Use stable identifiers.

---

# Scenario 290 - Dynamic Block Failure

Review iterator and variable structure.

---

# Scenario 291 - Local Values Incorrect

Verify expressions and references.

---

# Scenario 292 - Remote State Not Accessible

Verify backend permissions and connectivity.

---

# Scenario 293 - Secret Rotation Breaks Apply

Update Terraform variables or secret references.

---

# Scenario 294 - Terraform Upgrade Failure

Review version compatibility.

Upgrade incrementally.

---

# Scenario 295 - CI/CD Terraform Failure

Verify:

- Credentials
- Backend
- Variables
- Network

---

# Scenario 296 - Terraform Plan Diff Unexpected

Investigate:

- Manual changes
- Provider upgrades
- Variable changes

---

# Scenario 297 - Infrastructure Created but Application Fails

Verify infrastructure first.

Then validate application configuration.

---

# Scenario 298 - Multiple Engineers Modify Same Infrastructure

Use:

- Remote Backend
- State Locking
- Code Reviews
- Version Control

---

# Scenario 299 - Drift After Manual Console Changes

Run:

```bash
terraform plan
```

Reconcile drift using Infrastructure as Code.

---

# Scenario 300 - Random Terraform Production Failure

### Investigation

Review:

- Terraform Logs
- Backend
- State
- Provider
- Cloud API
- Recent Changes
- Resource Limits

### Root Cause

Usually configuration errors, state inconsistencies, provider issues, cloud API failures, or manual infrastructure changes.

### Resolution

Use a structured troubleshooting process and avoid manual modifications during investigation.

### Prevention

Implement remote state management, state locking, version pinning, code reviews, automated validation, and Infrastructure as Code best practices.

---

# Chapter 8 - Networking & Load Balancer Production Troubleshooting Scenarios (301–350)

Networking issues are among the most challenging production problems because they affect multiple services simultaneously.

A single misconfigured route, firewall rule, or DNS record can bring down an entire application.

This chapter covers **50 real-world networking and load balancer troubleshooting scenarios** commonly encountered in enterprise environments.

---

# Scenario 301 - Application Not Reachable

### Symptoms

- Users cannot access the application.
- Browser times out.

### Investigation

Check:

- DNS
- Load Balancer
- Security Groups
- Application Health

### Root Cause

Network path interruption.

### Resolution

Restore connectivity.

### Prevention

Continuous monitoring and health checks.

---

# Scenario 302 - Ping Fails

### Investigation

```bash
ping <host>
```

Verify routing and firewall.

---

# Scenario 303 - Traceroute Stops Midway

### Investigation

```bash
traceroute <host>

mtr <host>
```

### Root Cause

Intermediate network failure.

---

# Scenario 304 - DNS Lookup Failure

### Investigation

```bash
dig example.com

nslookup example.com
```

### Root Cause

DNS server unavailable or incorrect record.

---

# Scenario 305 - Incorrect DNS Record

Verify:

- A Record
- CNAME
- Alias
- TTL

---

# Scenario 306 - Slow DNS Resolution

Review:

- DNS latency
- Resolver configuration
- Upstream DNS servers

---

# Scenario 307 - SSL Certificate Expired

### Symptoms

HTTPS connection fails.

### Investigation

```bash
openssl s_client -connect host:443
```

### Resolution

Renew certificate.

---

# Scenario 308 - TLS Handshake Failure

Review:

- Certificate
- Cipher Suites
- Protocol Versions

---

# Scenario 309 - HTTPS Redirect Loop

Review application and load balancer redirect rules.

---

# Scenario 310 - Firewall Blocking Traffic

Verify:

- Security Groups
- Firewall Rules
- Network ACLs

---

# Scenario 311 - Port Closed

### Investigation

```bash
ss -lnt

netstat -lnt
```

---

# Scenario 312 - Service Listening on Wrong Port

Review application configuration.

---

# Scenario 313 - Load Balancer Returns 502

### Investigation

Review:

- Backend Service
- Health Checks
- Application Logs

---

# Scenario 314 - Load Balancer Returns 503

### Root Cause

No healthy backend targets.

---

# Scenario 315 - Load Balancer Returns 504

### Root Cause

Backend timeout.

Review application latency.

---

# Scenario 316 - Backend Health Check Failure

Verify:

- Health Endpoint
- Port
- Timeout
- Expected Response

---

# Scenario 317 - Sticky Session Failure

Review session persistence configuration.

---

# Scenario 318 - Uneven Traffic Distribution

Review:

- Load Balancing Algorithm
- Health Status
- Target Registration

---

# Scenario 319 - Target Registration Failure

Verify target group membership.

---

# Scenario 320 - Reverse Proxy Misconfiguration

Review:

- Proxy Rules
- Headers
- Backend URLs

---

# Scenario 321 - API Gateway Returns 502

Review backend integration.

---

# Scenario 322 - Gateway Timeout

Review upstream response time.

---

# Scenario 323 - Proxy Connection Refused

Verify backend application status.

---

# Scenario 324 - Network Latency High

### Investigation

```bash
ping

mtr
```

Review network utilization.

---

# Scenario 325 - Packet Loss

Review:

- Network Hardware
- Routing
- Congestion

---

# Scenario 326 - Routing Loop

Verify routing tables.

---

# Scenario 327 - Incorrect Route Table

Review:

```bash
ip route
```

---

# Scenario 328 - NAT Gateway Failure

Private instances lose internet connectivity.

Verify NAT Gateway and routing.

---

# Scenario 329 - Internet Gateway Missing

Verify VPC internet connectivity.

---

# Scenario 330 - VPN Tunnel Down

Review:

- Tunnel Status
- Encryption
- Routing

---

# Scenario 331 - VPC Peering Failure

Verify:

- Route Tables
- CIDR Overlap
- Peering Status

---

# Scenario 332 - CIDR Conflict

Review network address allocation.

---

# Scenario 333 - Subnet Misconfiguration

Verify subnet association.

---

# Scenario 334 - Network ACL Deny Rule

Review explicit deny rules.

---

# Scenario 335 - Security Group Misconfiguration

Verify inbound and outbound rules.

---

# Scenario 336 - Kubernetes Ingress Failure

Review:

```bash
kubectl describe ingress
```

---

# Scenario 337 - Ingress 404 Error

Verify:

- Host Rules
- Path Rules
- Backend Service

---

# Scenario 338 - Ingress TLS Failure

Verify certificate secret.

---

# Scenario 339 - CoreDNS Failure

Review CoreDNS logs.

---

# Scenario 340 - Service Discovery Failure

Verify:

- DNS
- Service Registration
- Endpoints

---

# Scenario 341 - Pod Cannot Reach External Service

Review:

- Egress Rules
- DNS
- Firewall

---

# Scenario 342 - Pod-to-Pod Communication Failure

Verify:

- CNI Plugin
- Network Policies

---

# Scenario 343 - MTU Mismatch

Symptoms:

- Intermittent connectivity
- Packet fragmentation

Review interface MTU configuration.

---

# Scenario 344 - ARP Resolution Failure

Verify Layer-2 connectivity.

---

# Scenario 345 - Excessive Network Connections

### Investigation

```bash
ss -s

netstat -an
```

Review connection states.

---

# Scenario 346 - Ephemeral Port Exhaustion

Review:

```bash
cat /proc/sys/net/ipv4/ip_local_port_range
```

Monitor TIME_WAIT connections.

---

# Scenario 347 - SYN Flood Attack

Review firewall and load balancer protection.

Enable SYN cookies where appropriate.

---

# Scenario 348 - DDoS Traffic Spike

Review:

- Traffic Sources
- WAF Rules
- CDN Protection
- Rate Limiting

---

# Scenario 349 - Cross-Region Connectivity Failure

Verify:

- Peering
- Transit Gateway
- VPN
- DNS

---

# Scenario 350 - Random Network Production Failure

### Investigation

Review:

- DNS
- Routing
- Firewalls
- Load Balancer
- Security Groups
- Network ACLs
- Application Logs
- Recent Infrastructure Changes

### Root Cause

Usually routing issues, firewall rules, DNS problems, load balancer health, or infrastructure configuration changes.

### Resolution

Follow the complete network path from client to backend, validating each hop before making changes.

### Prevention

Implement end-to-end network monitoring, automated health checks, infrastructure as code, and periodic network validation.

---

# Chapter 9 - Database, Storage & Backup Production Troubleshooting Scenarios (351–400)

Enterprise applications depend on reliable databases and storage systems.

Even when the application is healthy, failures in storage or databases can cause:

- Slow response times
- Data corruption
- Service outages
- Backup failures
- Disaster recovery issues

This chapter covers **50 real-world production troubleshooting scenarios** involving databases, storage, persistence, backups, and disaster recovery.

---

# Scenario 351 - Database Connection Refused

### Symptoms

- Application cannot connect to database.
- Connection timeout.

### Investigation

Verify:

- Database status
- Network connectivity
- Security Groups
- Database endpoint

### Root Cause

Database unavailable or network blocked.

### Resolution

Restore database connectivity.

### Prevention

Enable database health monitoring.

---

# Scenario 352 - Authentication Failure

### Investigation

Verify:

- Username
- Password
- Secrets
- IAM authentication

### Root Cause

Invalid credentials.

---

# Scenario 353 - Database Endpoint Changed

Review application configuration.

Update endpoint.

---

# Scenario 354 - Database High CPU Usage

### Investigation

Review:

- Running Queries
- Slow Query Log
- Active Connections

### Root Cause

Expensive queries or excessive workload.

---

# Scenario 355 - Slow Database Queries

### Investigation

Use:

```sql
EXPLAIN
```

Review indexes.

Optimize queries.

---

# Scenario 356 - Database Deadlock

### Symptoms

Transactions waiting indefinitely.

### Investigation

Review database deadlock logs.

### Resolution

Optimize transaction order.

---

# Scenario 357 - Connection Pool Exhausted

Review:

- Maximum connections
- Connection leaks
- Idle connections

---

# Scenario 358 - Too Many Database Connections

Monitor active sessions.

Optimize connection pooling.

---

# Scenario 359 - Database Replication Lag

### Investigation

Review replication delay.

### Root Cause

Heavy write workload or network latency.

---

# Scenario 360 - Replica Not Synchronizing

Verify replication status.

Review replication logs.

---

# Scenario 361 - Database Storage Full

### Symptoms

Writes fail.

### Investigation

Check storage utilization.

### Resolution

Increase storage or archive old data.

---

# Scenario 362 - Database Backup Failure

Verify:

- Backup job
- Storage
- Permissions

---

# Scenario 363 - Backup Storage Full

Clean old backups or expand storage.

---

# Scenario 364 - Restore Failure

Review backup integrity.

Verify backup version compatibility.

---

# Scenario 365 - Corrupted Backup

Always validate backups after creation.

Restore from previous valid backup.

---

# Scenario 366 - Point-in-Time Recovery Failure

Verify:

- Transaction logs
- Backup chain

---

# Scenario 367 - Read Replica Unavailable

Review:

- Replica health
- Network
- Storage

---

# Scenario 368 - Database Failover Failure

Review automatic failover configuration.

Verify application reconnect logic.

---

# Scenario 369 - High Disk I/O

Monitor:

- IOPS
- Queue Depth
- Latency

---

# Scenario 370 - Storage Latency High

Investigate:

- Storage performance
- Cloud storage metrics
- Workload spikes

---

# Scenario 371 - Persistent Volume Full

### Investigation

```bash
df -h
```

Increase storage or clean unnecessary files.

---

# Scenario 372 - Persistent Volume Claim Pending

Review:

```bash
kubectl describe pvc
```

Verify StorageClass.

---

# Scenario 373 - Volume Mount Failure

Review Pod events.

Verify volume configuration.

---

# Scenario 374 - File System Read-Only

Review filesystem errors.

Check storage health.

---

# Scenario 375 - Snapshot Creation Failure

Verify:

- Storage permissions
- Snapshot limits

---

# Scenario 376 - Snapshot Restore Failure

Validate snapshot integrity.

Review restore process.

---

# Scenario 377 - Storage Class Misconfiguration

Review StorageClass parameters.

---

# Scenario 378 - Shared File System Unavailable

Verify:

- NFS/EFS availability
- Mount targets
- Network

---

# Scenario 379 - File Locking Issues

Investigate concurrent access.

Review application locking behavior.

---

# Scenario 380 - Object Storage Access Denied

Verify:

- IAM permissions
- Bucket policy
- Access keys

---

# Scenario 381 - Object Upload Failure

Review:

- Bucket
- Network
- Permissions

---

# Scenario 382 - Object Download Failure

Verify object existence and access rights.

---

# Scenario 383 - Lifecycle Policy Not Working

Review lifecycle configuration.

Verify object tags and prefixes.

---

# Scenario 384 - Cross-Region Replication Failure

Verify:

- Replication rules
- IAM permissions
- Destination storage

---

# Scenario 385 - Archive Retrieval Delay

Verify storage class.

Inform users about archive retrieval times.

---

# Scenario 386 - Encryption Key Access Failure

Review:

- KMS permissions
- Key status
- IAM policies

---

# Scenario 387 - Storage Encryption Failure

Verify encryption configuration.

---

# Scenario 388 - Storage Performance Drops After Migration

Compare:

- Storage type
- IOPS
- Throughput
- Latency

---

# Scenario 389 - Database Migration Failure

Review migration logs.

Validate schema compatibility.

---

# Scenario 390 - Schema Update Failure

Review migration scripts.

Rollback if required.

---

# Scenario 391 - Data Corruption Detected

Restore verified backup.

Perform integrity validation.

---

# Scenario 392 - Duplicate Records

Review application logic.

Check transaction retries.

---

# Scenario 393 - Missing Records

Review:

- Replication
- Transactions
- Backup
- Application Logs

---

# Scenario 394 - Database Lock Contention

Investigate long-running transactions.

Optimize locking strategy.

---

# Scenario 395 - Backup Schedule Not Running

Verify scheduler configuration.

Review cron or automation logs.

---

# Scenario 396 - Disaster Recovery Test Failure

Review:

- Recovery documentation
- Backup validity
- Failover procedures

---

# Scenario 397 - Recovery Takes Too Long

Review:

- Recovery Time Objective (RTO)
- Backup strategy
- Storage performance

Optimize recovery procedures.

---

# Scenario 398 - Unexpected Storage Cost Increase

Investigate:

- Snapshots
- Backups
- Object Storage
- Old Volumes

Apply lifecycle policies.

---

# Scenario 399 - Database Performance Degrades After Deployment

Compare:

- Application Version
- Database Queries
- Indexes
- Configuration Changes

Rollback if necessary.

---

# Scenario 400 - Random Database & Storage Production Failure

### Investigation

Review:

- Database Logs
- Storage Metrics
- Backup Status
- Replication
- Connection Pool
- Query Performance
- Recent Changes

### Root Cause

Usually caused by resource exhaustion, storage latency, replication issues, query inefficiencies, backup failures, or configuration changes.

### Resolution

Follow a structured troubleshooting process, verify data integrity before making changes, and restore from backups only when necessary.

### Prevention

Implement continuous monitoring, automated backups, regular restore testing, storage capacity planning, query optimization, replication health monitoring, and disaster recovery drills.

---

