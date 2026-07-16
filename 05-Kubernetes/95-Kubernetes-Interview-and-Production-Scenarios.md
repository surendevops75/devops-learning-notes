# Kubernetes Interview Questions & Production Scenarios

## Introduction

Most Kubernetes interviews do not focus on:

```text
What is a Pod?

What is a Service?

What is a Deployment?
```

---

Instead interviewers focus on:

```text
Architecture

Production Issues

Troubleshooting

Design Decisions

Scaling

Security

High Availability
```

---

This file covers the most commonly asked Kubernetes interview questions and production scenarios.

---

# Section 1: Kubernetes Architecture

## What Happens When You Run kubectl apply?

### Short Answer

The request goes through API Server, Authentication, Authorization, Admission Controllers, ETCD, Controller Manager, Scheduler, and finally Kubelet.

### Complete Flow

```text
kubectl apply

↓

API Server

↓

Authentication

↓

Authorization

↓

Admission Controllers

↓

ETCD

↓

Controller Manager

↓

Scheduler

↓

Node

↓

Kubelet

↓

Container Runtime

↓

Pod Running
```

### Follow-Up Questions

- What component stores the Deployment?
- When does Scheduler run?
- When is ETCD updated?
- Where does Kubelet fit?

---

## What Happens When A Pod Is Created?

### Flow

```text
Deployment Created

↓

Stored In ETCD

↓

Deployment Controller Creates ReplicaSet

↓

ReplicaSet Creates Pod

↓

Scheduler Selects Node

↓

Kubelet Starts Containers

↓

Pod Running
```

### Follow-Up Questions

- What if Scheduler cannot find a node?
- What if image pull fails?
- What if resources are unavailable?

---

## What Happens When A Service Is Created?

### Flow

```text
Service Created

↓

Stored In ETCD

↓

EndpointSlice Controller Finds Matching Pods

↓

Endpoints Created

↓

kube-proxy Updates Rules

↓

Traffic Starts Routing
```

---

## What Happens When User Accesses Application Through Ingress?

### Flow

```text
User

↓

ALB / NLB

↓

Ingress Controller

↓

Service

↓

Pod
```

---

## What Happens If API Server Goes Down?

### Answer

New operations fail.

Examples:

```text
kubectl apply

kubectl delete

kubectl scale
```

---

Existing Applications Continue Running.

---

## What Happens If Scheduler Goes Down?

### Answer

Existing Pods Continue Running.

---

New Pods Remain:

```text
Pending
```

---

Until Scheduler Recovers.

---

## What Happens If Controller Manager Goes Down?

### Answer

Controllers Stop Reconciling.

Examples:

```text
ReplicaSets

Deployments

Jobs
```

---

Existing Pods Continue Running.

---

## What Happens If ETCD Goes Down?

### Answer

Control Plane Becomes Unavailable.

---

No New State Can Be Stored.

---

Cluster Operations Fail.

---

# Section 2: ETCD Questions

## Why Is ETCD Called The Brain Of Kubernetes?

Because ETCD stores:

```text
Pods

Deployments

Services

Secrets

ConfigMaps

RBAC
```

---

Without ETCD Kubernetes loses cluster state.

---

## What Data Is Stored In ETCD?

```text
Pods

Deployments

Services

Secrets

ConfigMaps

Nodes

Namespaces

RBAC
```

---

## What Is NOT Stored In ETCD?

```text
Container Logs

Container Images

Database Data

Application Files
```

---

## What Is Quorum?

Majority Of ETCD Nodes Must Be Available.

---

Example

```text
3 Nodes → Need 2

5 Nodes → Need 3
```

---

## What Happens If Quorum Is Lost?

ETCD Stops Accepting Writes.

---

Control Plane Stops Functioning.

---

## How Do You Backup ETCD?

```bash
etcdctl snapshot save backup.db
```

---

## How Do You Restore ETCD?

```bash
etcdctl snapshot restore backup.db
```

---

# Section 3: Scheduling Questions

## Difference Between Node Selector And Node Affinity?

### Node Selector

Simple Matching.

```yaml
nodeSelector:
  environment: prod
```

---

### Node Affinity

Advanced Matching.

```yaml
nodeAffinity
```

Supports:

```text
Required

Preferred
```

---

## Difference Between Pod Affinity And Pod Anti-Affinity?

### Pod Affinity

Pods Stay Together.

---

### Pod Anti-Affinity

Pods Stay Apart.

---

## Difference Between Taints/Tolerations And Node Affinity?

### Affinity

Pods Choose Nodes.

---

### Taints

Nodes Reject Pods.

---

## Why Does Pod Remain Pending?

Possible Reasons:

```text
Insufficient CPU

Insufficient Memory

Taints

Affinity Rules

No Nodes Available

PDB Constraints
```

---

# Production Scenario

## Scenario: Pod Stuck In Pending

### Investigation

```bash
kubectl describe pod nginx
```

### Check

```text
Events

Resources

Taints

Affinity Rules
```

### Expected Interview Answer

Always start with:

```bash
kubectl describe pod
```

---

# Section 4: Networking Questions

## Explain Kubernetes Networking

### Rules

```text
Pod To Pod Communication

Service To Pod Communication

External To Service Communication
```

---

## What Does kube-proxy Do?

Manages:

```text
Service Routing
```

---

Creates:

```text
iptables

IPVS
```

rules.

---

## What Does CoreDNS Do?

Provides:

```text
Service Discovery
```

---

Example

```text
catalogue.default.svc.cluster.local
```

---

## What Does VPC CNI Do?

Assigns VPC IPs To Pods.

---

EKS Networking Component.

---

# Production Scenario

## Service Exists But Traffic Not Reaching Pods

### Check Service

```bash
kubectl describe svc catalogue
```

### Verify

```text
Selectors

Endpoints

Labels
```

---

## DNS Resolution Failing

### Check

```bash
kubectl get pods -n kube-system
```

Verify:

```text
CoreDNS Running
```

---

# Section 5: Storage Questions

## Difference Between PV And PVC?

### PV

Actual Storage.

---

### PVC

Storage Request.

---

## Difference Between EBS And EFS?

### EBS

```text
Single Node

High Performance
```

---

### EFS

```text
Shared Across Nodes
```

---

## Why Use StatefulSet For Databases?

Provides:

```text
Stable Network Identity

Persistent Storage
```

---

# Production Scenario

## PVC Stuck In Pending

### Check

```bash
kubectl get pvc
```

### Verify

```text
StorageClass

Provisioner

Capacity
```

---

# Section 6: Security Questions

## Difference Between Authentication And Authorization?

### Authentication

Who Are You?

---

### Authorization

What Can You Do?

---

## What Is RBAC?

Role Based Access Control.

---

## Difference Between Role And ClusterRole?

### Role

Namespace Scope.

---

### ClusterRole

Cluster Scope.

---

## What Is IRSA?

IAM Roles For Service Accounts.

---

Provides:

```text
Pod Level AWS Permissions
```

---

# Production Scenario

## Application Cannot Access S3

### Verify

```text
OIDC

IAM Role

Service Account

Trust Relationship
```

---

# Section 7: Deployment Questions

## Difference Between Deployment And StatefulSet?

### Deployment

Stateless Applications.

---

### StatefulSet

Stateful Applications.

---

## Difference Between Rolling Update And Blue-Green?

### Rolling Update

Old And New Versions Run Together.

---

### Blue-Green

Only One Version Receives Traffic.

---

## Difference Between Canary And Blue-Green?

### Canary

Traffic Split.

```text
90% → Old

10% → New
```

---

### Blue-Green

Traffic Switch.

```text
100% → Blue

100% → Green
```

---

# Production Scenario

## Deployment Rollout Stuck

### Check

```bash
kubectl rollout status deployment
```

### Verify

```text
Readiness Probe

Resources

Image

Application Errors
```

---

## Rollback Deployment

```bash
kubectl rollout undo deployment
```

---

# Section 8: Troubleshooting Questions

## What Causes CrashLoopBackOff?

```text
Application Crash

Missing ConfigMap

Missing Secret

Database Connectivity

OOM
```

---

## What Causes OOMKilled?

Container Exceeds Memory Limit.

---

## What Causes ImagePullBackOff?

Image Cannot Be Pulled.

---

Reasons

```text
Wrong Image

Private Registry

Authentication Failure
```

---

# Production Scenario

## Pod In CrashLoopBackOff

### Commands

```bash
kubectl describe pod
```

```bash
kubectl logs pod-name
```

```bash
kubectl logs pod-name --previous
```

---

## Pod Restarting Continuously

Check:

```text
OOM

Probe Failures

Application Errors
```

---

# Section 9: Scaling Questions

## Difference Between HPA And Cluster Autoscaler?

### HPA

Scales Pods.

---

### Cluster Autoscaler

Scales Nodes.

---

## Difference Between Cluster Autoscaler And Karpenter?

### Cluster Autoscaler

Uses Existing Node Groups.

---

### Karpenter

Creates Best EC2 Instance Dynamically.

---

## What Problem Does KEDA Solve?

Event Driven Scaling.

---

Examples

```text
Kafka

RabbitMQ

SQS
```

---

# Production Scenario

## HPA Scaled Pods But Pods Remain Pending

### Reason

No Nodes Available.

---

Need:

```text
Cluster Autoscaler

or

Karpenter
```

---

## SQS Queue Has 10000 Messages

CPU = 10%

### Solution

```text
KEDA
```

---

# Section 10: EKS Questions

## Difference Between EKS And Self Managed Kubernetes?

### EKS

AWS Manages Control Plane.

---

### Self Managed

You Manage Everything.

---

## How Do You Upgrade EKS?

### Control Plane

```bash
aws eks update-cluster-version
```

---

### Addons

```bash
aws eks update-addon
```

---

### Node Groups

Blue-Green Upgrade.

---

# Production Scenario

## Control Plane Upgraded To 1.33

Node Group Still 1.32

Supported?

### Answer

Yes.

Worker Nodes Can Be One Minor Version Behind.

---

# Section 11: Istio Questions

## Why Service Mesh?

Provides

```text
mTLS

Retries

Traffic Splitting

Observability
```

---

## What Is Envoy?

Sidecar Proxy Used By Istio.

---

## What Is mTLS?

Mutual TLS.

---

Authenticates Both Sides.

---

# Production Scenario

Need

```text
90% → v1

10% → v2
```

### Solution

```text
Istio Canary

Gateway API
```

---

# Section 12: eBPF & Cilium Questions

## What Is eBPF?

Kernel-Level Programmability.

---

## What Is Cilium?

Modern Kubernetes Networking Solution Using eBPF.

---

## Can Cilium Replace kube-proxy?

Yes.

---

# Section 13: Kafka Questions

## What Is Producer?

Produces Messages.

---

## What Is Consumer?

Consumes Messages.

---

## What Is Topic?

Logical Category For Messages.

---

## What Is Partition?

Provides Scalability.

---

## What Is Consumer Group?

Allows Parallel Consumption.

---

## What Is Offset?

Tracks Consumer Progress.

---

# Production Scenario

Consumer Crashed.

### What Happens?

Messages Stay In Kafka.

---

Consumer Resumes From Last Offset.

---

# Section 14: OPA / Kyverno Questions

## Why OPA?

Policy As Code.

---

## Why Kyverno?

Kubernetes Native Policy Management.

---

## Difference Between OPA And Kyverno?

### OPA

Uses Rego.

---

### Kyverno

Uses YAML.

---

# Section 15: Operators Questions

## What Is An Operator?

Automated Administrator For Kubernetes Applications.

---

## What Is CRD?

Custom Resource Definition.

---

## What Is Reconciliation?

Matching Desired State With Actual State.

---

## Difference Between Controller And Operator?

Operator = Controller + Domain Knowledge + CRDs

---

# Top Production Scenarios Asked In Interviews

## Scenario 1

```text
Pod Pending
```

---

## Scenario 2

```text
CrashLoopBackOff
```

---

## Scenario 3

```text
OOMKilled
```

---

## Scenario 4

```text
ImagePullBackOff
```

---

## Scenario 5

```text
Service Not Reachable
```

---

## Scenario 6

```text
Ingress Not Working
```

---

## Scenario 7

```text
DNS Resolution Failing
```

---

## Scenario 8

```text
Node NotReady
```

---

## Scenario 9

```text
ETCD Full
```

---

## Scenario 10

```text
Control Plane Upgrade Failure
```

---

## Scenario 11

```text
HPA Not Scaling
```

---

## Scenario 12

```text
Karpenter Not Creating Nodes
```

---

## Scenario 13

```text
IRSA Not Working
```

---

## Scenario 14

```text
ALB Not Created
```

---

## Scenario 15

```text
Application Cannot Reach Database
```

---

# Key Takeaways

```text
Most Kubernetes interviews focus on production troubleshooting and architecture.

Always explain the complete control plane flow.

Know how API Server, ETCD, Scheduler, Controller Manager and Kubelet work together.

Be comfortable troubleshooting Pending Pods, CrashLoopBackOff, OOMKilled and networking issues.

Understand scaling using HPA, KEDA, Cluster Autoscaler and Karpenter.

Know EKS upgrades, IRSA, Gateway API, Istio, Kafka, OPA, Kyverno and Operators.

Production scenarios are more important than theoretical definitions in senior-level interviews.
```