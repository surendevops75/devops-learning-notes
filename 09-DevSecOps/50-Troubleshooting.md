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



