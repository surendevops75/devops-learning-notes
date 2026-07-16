# ETCD - The Brain Of Kubernetes

## Introduction

When people learn Kubernetes, they usually focus on:

```text
Pods

Deployments

Services

Ingress

StatefulSets
```

---

But very few engineers ask:

```text
Where Does Kubernetes Store Everything?
```

---

Suppose you create:

```bash
kubectl apply -f deployment.yaml
```

---

Question:

```text
Where Is This Deployment Stored?
```

---

Answer:

```text
ETCD
```

---

Similarly:

```text
Pods

Deployments

Services

ConfigMaps

Secrets

Namespaces

RBAC

Nodes
```

are all stored inside:

```text
ETCD
```

---

# What Is ETCD?

ETCD is a:

```text
Distributed Key-Value Database
```

used by Kubernetes as its:

```text
Source Of Truth
```

---

Think Of It Like:

```text
MySQL

For Kubernetes
```

---

Everything Kubernetes knows about the cluster exists inside ETCD.

---

# Why Is ETCD Called The Brain Of Kubernetes?

Imagine a cluster:

```text
50 Nodes

500 Pods

100 Services
```

---

API Server knows about them because:

```text
ETCD Stores Them
```

---

If ETCD disappears:

```text
API Server Loses Cluster State
```

---

Result:

```text
Control Plane Stops Working
```

---

Therefore:

```text
ETCD = Brain Of Kubernetes
```

---

# Kubernetes Architecture With ETCD

```text
kubectl apply

↓

API Server

↓

ETCD

↓

Persist State
```

---

Example

Create Deployment

```bash
kubectl create deployment nginx \
--image=nginx
```

---

Flow

```text
kubectl

↓

API Server

↓

Validate Request

↓

Store In ETCD

↓

Success Response
```

---

# What Data Is Stored In ETCD?

## Workloads

```text
Pods

Deployments

ReplicaSets

DaemonSets

StatefulSets

Jobs

CronJobs
```

---

## Networking

```text
Services

Endpoints

Ingress

Gateway API Objects
```

---

## Configuration

```text
ConfigMaps

Secrets
```

---

## Security

```text
Users

Roles

RoleBindings

ClusterRoles

ClusterRoleBindings

Service Accounts
```

---

## Cluster Information

```text
Nodes

Namespaces

Events
```

---

# What Is NOT Stored In ETCD?

Many Engineers Get This Wrong.

---

Not Stored

```text
Container Images

Application Logs

Database Data

Files Inside Containers

EBS Data

EFS Data
```

---

Example

```text
MongoDB Data

Stored In EBS

Not ETCD
```

---

# ETCD Data Structure

ETCD stores data as:

```text
Key

Value
```

---

Example

```text
Key

/registry/pods/default/nginx
```

---

Value

```text
Pod Definition
```

stored as JSON.

---

# How Scheduler Uses ETCD

Pod Created

```text
Pending
```

---

Stored In

```text
ETCD
```

---

Scheduler Reads

```text
Pending Pod
```

---

Selects Node

---

Updates ETCD

---

Pod Becomes

```text
Running
```

---

# How Controller Manager Uses ETCD

Deployment

```text
Replicas = 3
```

---

Actual Pods

```text
Replicas = 2
```

---

Controller Detects Difference.

---

Updates Cluster.

---

Stores New State In

```text
ETCD
```

---

# ETCD High Availability

Single ETCD Node

```text
Dangerous
```

---

If ETCD Fails

```text
Cluster Fails
```

---

Production Uses

```text
3 ETCD Nodes

or

5 ETCD Nodes
```

---

# ETCD Cluster

Example

```text
ETCD-1

ETCD-2

ETCD-3
```

---

One Node Becomes

```text
Leader
```

---

Others Become

```text
Followers
```

---

# Leader Election

Example

```text
ETCD-1 Leader

ETCD-2 Follower

ETCD-3 Follower
```

---

Writes Go Through

```text
Leader
```

---

Followers Replicate Data.

---

# What Is Quorum?

Quorum means:

```text
Majority Of Members
```

must be available.

---

# 3 Node ETCD

Need

```text
2 Nodes
```

alive.

---

Example

```text
Node-1 Up

Node-2 Up

Node-3 Down
```

---

Cluster Works.

---

Example

```text
Node-1 Up

Node-2 Down

Node-3 Down
```

---

No Quorum.

---

Cluster Stops.

---

# 5 Node ETCD

Need

```text
3 Nodes
```

alive.

---

Example

```text
3 Up

2 Down
```

---

Cluster Works.

---

# Why Not 4 Nodes?

Because Quorum

```text
3
```

---

Same As

```text
5 Nodes
```

without extra benefit.

---

Production Recommendation

```text
3

or

5
```

only.

---

# ETCD Security

ETCD Contains

```text
Secrets

Certificates

Cluster Configuration
```

---

Need Strong Protection.

---

# TLS Encryption

Communication Between

```text
API Server

↓

ETCD
```

is encrypted.

---

Certificates

```text
ca.crt

server.crt

server.key
```

---

Used For Authentication.

---

# Encryption At Rest

Problem

```text
Secrets Stored In Plain Text
```

inside ETCD.

---

Solution

```text
Encryption At Rest
```

---

Example

```yaml
kind: EncryptionConfiguration
```

---

Secrets Become Encrypted Before Storage.

---

# ETCD Backup

One Of The Most Important Kubernetes Operations.

---

Why Backup?

Because

```text
ETCD Corruption

Accidental Deletion

Disk Failure

Cluster Recovery
```

can happen.

---

# Backup Command

```bash
ETCDCTL_API=3 etcdctl snapshot save backup.db \
--endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key
```

---

Output

```text
backup.db
```

---

# Verify Backup

```bash
etcdctl snapshot status backup.db
```

---

Example Output

```text
Revision

Total Keys

Database Size
```

---

# Production Backup Strategy

Daily

```text
ETCD Snapshot
```

---

Store In

```text
S3

Backup Server

DR Site
```

---

# ETCD Restore

Disaster Happens.

---

Example

```text
ETCD Corrupted
```

---

Need Recovery.

---

# Restore Command

```bash
ETCDCTL_API=3 etcdctl snapshot restore backup.db
```

---

Creates

```text
Recovered ETCD Data
```

---

# Recovery Flow

```text
Stop ETCD

↓

Restore Snapshot

↓

Replace Data Directory

↓

Start ETCD

↓

Cluster Recovered
```

---

# ETCD Disaster Recovery Example

Accidentally Delete

```text
Namespace

Services

Deployments
```

---

ETCD Snapshot Exists.

---

Restore Snapshot.

---

Cluster Returns To Previous State.

---

# EKS vs Self Managed Kubernetes

Very Important Interview Topic.

---

# Self Managed Kubernetes

You Manage

```text
ETCD

Backups

Restore

Certificates
```

---

# EKS

AWS Manages

```text
ETCD

Control Plane

High Availability
```

---

You Cannot Directly Access

```text
ETCD
```

---

# How To Backup ETCD In EKS?

Answer:

```text
You Do Not Directly Backup ETCD.

AWS Manages ETCD Internally.

You Backup Kubernetes Resources Separately If Needed.
```

---

# Common ETCD Failure Scenarios

## Disk Full

Problem

```text
No More Writes
```

---

Result

```text
API Requests Fail
```

---

## Certificate Expired

Problem

```text
API Server Cannot Talk To ETCD
```

---

Result

```text
Control Plane Failure
```

---

## Quorum Lost

Example

```text
3 Node Cluster

2 Nodes Lost
```

---

Result

```text
ETCD Stops Accepting Writes
```

---

## Database Corruption

Solution

```text
Restore Snapshot
```

---

# ETCD Monitoring

Monitor

```text
Disk Usage

Database Size

Leader Changes

Latency

Snapshot Status
```

---

# Useful Commands

Check Health

```bash
etcdctl endpoint health
```

---

Check Status

```bash
etcdctl endpoint status
```

---

List Members

```bash
etcdctl member list
```

---

Check Alarms

```bash
etcdctl alarm list
```

---

# Production Scenario

Application Team Reports

```text
Pods Not Creating
```

---

Investigation

```text
API Server Errors
```

---

Further Investigation

```text
ETCD Disk Full
```

---

Result

```text
Control Plane Cannot Persist Data
```

---

Fix

```text
Free Disk Space

Restart ETCD
```

---

Cluster Recovers.

---

# Common Interview Questions

## Why Is ETCD Called The Brain Of Kubernetes?

### Short Answer

ETCD is called the brain of Kubernetes because it stores the complete cluster state.

### Detailed Explanation

Every Kubernetes object such as Pods, Deployments, Services, Secrets, ConfigMaps, Nodes, and RBAC resources are stored inside ETCD. The API Server, Scheduler, and Controller Manager continuously read and write data from ETCD. Without ETCD, Kubernetes has no source of truth.

### Production Example

```text
Deployment Created

↓

Stored In ETCD

↓

Scheduler Reads

↓

Pods Created
```

### Follow-Up Questions

- What data is stored in ETCD?
- What data is not stored in ETCD?
- Does Scheduler use ETCD?
- Does Controller Manager use ETCD?

---

## What Happens If ETCD Goes Down?

### Short Answer

The Kubernetes control plane becomes unavailable.

### Detailed Explanation

The API Server depends on ETCD for storing and retrieving cluster state. If ETCD becomes unavailable, Kubernetes cannot process updates, create new resources, or maintain cluster consistency.

### Production Example

```text
ETCD Down

↓

API Server Errors

↓

Pods Cannot Be Created

↓

Control Plane Failure
```

### Follow-Up Questions

- Will existing Pods continue running?
- Can workloads serve traffic?
- What happens to new deployments?
- How is recovery performed?

---

## What Is Quorum?

### Short Answer

Quorum is the minimum number of ETCD members required for the cluster to operate.

### Detailed Explanation

ETCD uses the Raft consensus algorithm. More than half of the ETCD members must be available to process writes.

### Production Example

```text
3 Node ETCD

↓

Need 2 Nodes Alive

↓

Quorum Achieved
```

### Follow-Up Questions

- Why use odd numbers of nodes?
- Why not use 4 nodes?
- What happens when quorum is lost?
- How does leader election work?

---

## How Do You Backup ETCD?

### Short Answer

Using ETCD snapshots.

### Detailed Explanation

ETCD provides snapshot functionality that creates a consistent backup of the entire cluster state.

### Production Example

```bash
etcdctl snapshot save backup.db
```

### Follow-Up Questions

- How often should backups run?
- Where should backups be stored?
- How do you verify backups?
- What is contained inside a snapshot?

---

## How Do You Restore ETCD?

### Short Answer

Restore a snapshot and replace the ETCD data directory.

### Detailed Explanation

A previously taken snapshot is restored and ETCD is restarted using the recovered data.

### Production Example

```text
Stop ETCD

↓

Restore Snapshot

↓

Replace Data Directory

↓

Start ETCD
```

### Follow-Up Questions

- What happens after restore?
- How do you validate recovery?
- Can restore be performed on a new cluster?
- How long does recovery take?

---

## What Is The Difference Between EKS ETCD And Self Managed ETCD?

### Short Answer

In self-managed Kubernetes you manage ETCD, while in EKS AWS manages ETCD.

### Detailed Explanation

Self-managed clusters require ETCD installation, backup, restore, monitoring, and security management. In EKS these responsibilities are handled by AWS.

### Production Example

```text
kubeadm Cluster

↓

Manage ETCD Yourself
```

```text
EKS

↓

AWS Manages ETCD
```

### Follow-Up Questions

- Can you access ETCD in EKS?
- How is backup handled in EKS?
- Who manages ETCD certificates?
- How is ETCD upgraded in EKS?

---

# Key Takeaways

```text
ETCD is the source of truth for Kubernetes.

All cluster state is stored in ETCD.

API Server, Scheduler, and Controller Manager depend on ETCD.

Production ETCD uses 3 or 5 nodes for high availability.

Quorum is required for ETCD operations.

Secrets should be protected using encryption at rest.

Regular ETCD snapshots are critical for disaster recovery.

ETCD backup and restore is one of the most important Kubernetes administration skills.

In self-managed Kubernetes you manage ETCD.

In EKS AWS manages ETCD internally.
```