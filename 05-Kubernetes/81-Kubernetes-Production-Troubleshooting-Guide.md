# Kubernetes Production Troubleshooting Guide

## Introduction

In production, Kubernetes issues usually fall into one of these categories:

```text
Pod Issues

Node Issues

Networking Issues

Storage Issues

DNS Issues

Application Issues

Resource Issues
```

---

A good Kubernetes engineer should be able to answer:

```text
What Is Broken?

Why Is It Broken?

How To Verify?

How To Fix?
```

---

# Kubernetes Troubleshooting Flow

Always follow:

```text
Observe

↓

Identify

↓

Verify

↓

Fix

↓

Validate
```

---

Example

```text
Pod Not Running

↓

Check Status

↓

Check Events

↓

Check Logs

↓

Fix

↓

Verify
```

---

# Scenario 1: Pod Stuck In Pending

## Symptoms

```bash
kubectl get pods
```

Output

```text
STATUS

Pending
```

---

## Common Causes

```text
Insufficient CPU

Insufficient Memory

Node Selector Mismatch

Taints

Affinity Rules

No Available Nodes

PVC Pending
```

---

## Investigation

Describe Pod

```bash
kubectl describe pod <pod-name>
```

---

Check Events

```text
FailedScheduling
```

---

Example

```text
0/3 nodes available
```

---

## Production Example

Node Resources

```text
CPU Fully Utilized
```

---

Scheduler Cannot Place Pod

---

## Fix

```text
Add Nodes

Increase Resources

Modify Scheduling Rules
```

---

# Scenario 2: CrashLoopBackOff

## Symptoms

```bash
kubectl get pods
```

Output

```text
CrashLoopBackOff
```

---

## Meaning

Container Starts

↓

Crashes

↓

Restarts

↓

Crashes Again

---

## Investigation

Current Logs

```bash
kubectl logs <pod>
```

---

Previous Crash Logs

```bash
kubectl logs <pod> --previous
```

---

Describe Pod

```bash
kubectl describe pod <pod>
```

---

## Common Causes

```text
Application Bug

Wrong Environment Variables

Missing Secret

Missing ConfigMap

Database Connectivity Issue
```

---

## Production Example

Catalogue Service

↓

Missing MongoDB Password

↓

Application Crash

↓

CrashLoopBackOff

---

## Fix

```text
Check Logs

Check Secrets

Check ConfigMaps

Validate Dependencies
```

---

# Scenario 3: ImagePullBackOff

## Symptoms

```text
ImagePullBackOff
```

---

## Meaning

Container Runtime Cannot Pull Image.

---

## Investigation

```bash
kubectl describe pod <pod>
```

---

Events

```text
Failed To Pull Image
```

---

## Common Causes

```text
Wrong Image Tag

Image Does Not Exist

ECR Permission Issue

Docker Hub Rate Limit

Registry Connectivity Issue
```

---

## Production Example

Deployment

```text
catalogue:v100
```

---

Actual Version

```text
catalogue:v10
```

---

Result

```text
ImagePullBackOff
```

---

## Fix

```text
Correct Image Tag

Verify Registry Access

Check IAM Permissions
```

---

# Scenario 4: OOMKilled

## Symptoms

```bash
kubectl describe pod
```

---

Output

```text
OOMKilled
```

---

## Meaning

Container Exceeded Memory Limit.

---

## Investigation

```bash
kubectl describe pod
```

---

Check

```text
Last State
```

---

## Example

Container Limit

```text
512Mi
```

---

Application Uses

```text
800Mi
```

---

Result

```text
OOMKilled
```

---

## Fix

```text
Increase Memory Limits

Optimize Application
```

---

# Scenario 5: Service Not Working

## Symptoms

```text
Application Cannot Reach Service
```

---

## Investigation

Check Service

```bash
kubectl get svc
```

---

Check Endpoints

```bash
kubectl get endpointslices
```

---

## Common Causes

```text
Wrong Labels

Wrong Selectors

Pods Not Ready

kube-proxy Issues
```

---

## Production Example

Service Selector

```yaml
app: catalogue
```

---

Pod Label

```yaml
app: catalog
```

---

Result

```text
No Endpoints
```

---

## Fix

Correct Labels.

---

# Scenario 6: DNS Resolution Failure

## Symptoms

Application Logs

```text
Unknown Host

DNS Lookup Failed
```

---

## Investigation

Test DNS

```bash
kubectl exec -it <pod> -- nslookup kubernetes.default
```

---

Check CoreDNS

```bash
kubectl get pods -n kube-system
```

---

## Common Causes

```text
CoreDNS Down

Network Policy Blocking DNS

Node Networking Issue
```

---

## Production Example

Default Deny Policy

↓

Blocks CoreDNS

↓

Application Cannot Resolve Services

---

## Fix

Allow DNS Traffic

Restore CoreDNS

---

# Scenario 7: PVC Pending

## Symptoms

```bash
kubectl get pvc
```

Output

```text
Pending
```

---

## Investigation

```bash
kubectl describe pvc
```

---

## Common Causes

```text
Missing StorageClass

CSI Driver Failure

Volume Provisioning Failure

Wrong Access Mode
```

---

## Production Example

EBS CSI Not Installed

↓

PVC Pending

↓

Pod Pending

---

## Fix

```text
Install CSI Driver

Correct StorageClass
```

---

# Scenario 8: Node NotReady

## Symptoms

```bash
kubectl get nodes
```

Output

```text
NotReady
```

---

## Investigation

```bash
kubectl describe node
```

---

## Common Causes

```text
kubelet Down

Network Failure

Disk Pressure

Memory Pressure
```

---

## Production Example

Node Disk Full

↓

kubelet Stops Reporting

↓

Node NotReady

---

## Fix

```text
Restore Node Health

Restart kubelet

Free Resources
```

---

# Scenario 9: Pod Cannot Access AWS Services

## Symptoms

```text
AccessDenied
```

---

## Investigation

Check Service Account

```bash
kubectl get sa
```

---

Check Annotation

```bash
kubectl describe sa
```

---

## Common Causes

```text
Missing IRSA

Wrong IAM Policy

Incorrect Trust Relationship
```

---

## Production Example

Pod Reads Secret

↓

No IAM Permission

↓

AccessDenied

---

## Fix

```text
Verify IRSA

Verify IAM Policy
```

---

# Scenario 10: Deployment Rollout Failure

## Symptoms

```bash
kubectl rollout status deployment/catalogue
```

---

Output

```text
Waiting For Rollout
```

---

## Investigation

```bash
kubectl describe deployment
```

---

Check Pods

```bash
kubectl get pods
```

---

## Common Causes

```text
Readiness Probe Failure

CrashLoopBackOff

ImagePullBackOff

Insufficient Resources
```

---

## Fix

Identify Pod Failure

Fix Root Cause

Retry Rollout

---

# Production Troubleshooting Workflow

## Pod Problem

```text
kubectl get pods

↓

kubectl describe pod

↓

kubectl logs
```

---

## Service Problem

```text
kubectl get svc

↓

kubectl get endpointslices

↓

Verify Labels
```

---

## Node Problem

```text
kubectl get nodes

↓

kubectl describe node
```

---

## Storage Problem

```text
kubectl get pvc

↓

kubectl describe pvc
```

---

## DNS Problem

```text
nslookup

↓

CoreDNS

↓

Network Policies
```

---

# Most Used Production Commands

Pods

```bash
kubectl get pods -A
```

---

Pod Details

```bash
kubectl describe pod <pod>
```

---

Logs

```bash
kubectl logs <pod>
```

---

Previous Logs

```bash
kubectl logs <pod> --previous
```

---

Nodes

```bash
kubectl get nodes
```

---

Services

```bash
kubectl get svc
```

---

EndpointSlices

```bash
kubectl get endpointslices
```

---

Events

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

---

Resource Usage

```bash
kubectl top nodes

kubectl top pods
```

---

# Production Troubleshooting Mindset

Never Start With:

```text
Fixing
```

---

Always Start With:

```text
Understanding
```

---

Follow:

```text
Symptoms

↓

Events

↓

Logs

↓

Root Cause

↓

Fix

↓

Validation
```

---

# Common Interview Questions

## A Pod Is Stuck In Pending. What Will You Do?

### Short Answer

Describe the Pod and check scheduling events.

### Investigation

```bash
kubectl describe pod <pod>
```

Check:

```text
FailedScheduling
```

### Follow-Up Questions

- Insufficient CPU?
- Taints?
- Affinity Rules?
- PVC Pending?

---

## What Is CrashLoopBackOff?

### Short Answer

The container repeatedly starts and crashes.

### Investigation

```bash
kubectl logs <pod>

kubectl logs <pod> --previous
```

### Follow-Up Questions

- Missing Secrets?
- Application Crash?
- Database Connectivity?
- Wrong ConfigMap?

---

## How Do You Troubleshoot Service Issues?

### Short Answer

Verify Service, EndpointSlice, Labels, and Pod readiness.

### Investigation

```bash
kubectl get svc

kubectl get endpointslices
```

### Follow-Up Questions

- Wrong Selector?
- No Endpoints?
- kube-proxy Issue?
- Readiness Probe Failure?

---

## How Do You Troubleshoot DNS Issues?

### Short Answer

Check CoreDNS and DNS resolution inside Pods.

### Investigation

```bash
nslookup kubernetes.default
```

### Follow-Up Questions

- CoreDNS Running?
- Network Policy Blocking?
- Service Name Correct?
- DNS Configuration Issue?

---

# Key Takeaways

```text
Most Kubernetes issues fall into Pods, Nodes, Networking, DNS, Storage, or Resources.

Always start with Events and Logs.

kubectl describe is one of the most important troubleshooting commands.

CrashLoopBackOff usually requires log analysis.

Pending Pods usually indicate scheduling problems.

Service issues often involve selectors and endpoints.

DNS issues typically involve CoreDNS or Network Policies.

PVC Pending usually points to storage configuration issues.

Node NotReady often indicates kubelet or infrastructure problems.

A structured troubleshooting approach is critical in production environments.
```