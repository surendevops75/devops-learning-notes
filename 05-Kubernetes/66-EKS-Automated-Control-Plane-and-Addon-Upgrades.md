# EKS Automated Control Plane and Addon Upgrades

## Introduction

In production environments, Kubernetes upgrades should not rely on manual execution.

Manual upgrades can lead to:

```text
Human Errors

Version Mismatches

Missed Addon Upgrades

Extended Downtime

Inconsistent Processes
```

---

To solve this problem, Platform Teams automate:

```text
Control Plane Upgrades

Addon Upgrades

Version Validation

Health Checks

Status Monitoring
```

---

# Production Scenario

Current Cluster

```text
roboshop-dev

Version: 1.32
```

---

Target Version

```text
1.33
```

---

Execution

```bash
sh upgrade-cluster.sh 1.33
```

---

Goal

```text
Upgrade Control Plane

Upgrade Compatible Addons

Validate Cluster Health
```

---

# High Level Workflow

```text
Validate Input

↓

Fetch Current Version

↓

Validate Upgrade Path

↓

Upgrade Control Plane

↓

Wait For ACTIVE

↓

Discover Installed Addons

↓

Find Compatible Versions

↓

Upgrade Addons

↓

Wait For ACTIVE

↓

Validate Cluster
```

---

# Script Input Validation

The script requires exactly one argument.

Example

```bash
sh upgrade-cluster.sh 1.33
```

---

Invalid

```bash
sh upgrade-cluster.sh
```

---

Validation

```bash
if [ "$#" -ne 1 ]
```

---

Purpose

```text
Prevent Accidental Execution
```

---

# Fetch Current Cluster Version

Command

```bash
aws eks describe-cluster \
--name roboshop-dev \
--query cluster.version
```

---

Example Output

```text
1.32
```

---

Stored As

```text
CURRENT_CP_VERSION
```

---

# Target Version

Provided By User

```text
1.33
```

---

Stored As

```text
EKS_TARGET_VERSION
```

---

# Version Parsing

Current Version

```text
1.32
```

---

Target Version

```text
1.33
```

---

Script Extracts

```text
Major Version

Minor Version
```

---

Example

```text
Current

Major = 1
Minor = 32
```

---

Target

```text
Major = 1
Minor = 33
```

---

# Production Safety Check

The script enforces:

```text
Same Major Version

AND

Target Minor = Current Minor + 1
```

---

Allowed

```text
1.32 → 1.33

1.33 → 1.34

1.34 → 1.35
```

---

Rejected

```text
1.32 → 1.35

1.32 → 1.36

1.33 → 2.0
```

---

Validation Logic

```text
Current Major = Target Major

AND

(Target Minor - Current Minor) = 1
```

---

Why?

```text
Kubernetes Supports

Sequential Minor Upgrades
```

---

# Example

Valid

```bash
sh upgrade-cluster.sh 1.33
```

Current

```text
1.32
```

---

Result

```text
Version Check Passed
```

---

Invalid

```bash
sh upgrade-cluster.sh 1.35
```

---

Result

```text
ABORT

Target Version Must Be Exactly One Minor Step Ahead
```

---

# Control Plane Upgrade

Command

```bash
aws eks update-cluster-version \
--name roboshop-dev \
--kubernetes-version 1.33
```

---

AWS Upgrades

```text
API Server

Scheduler

Controller Manager

etcd
```

---

# Why Upgrade Control Plane First?

Worker Nodes communicate with:

```text
API Server
```

---

Therefore:

```text
Control Plane

↓

Addons

↓

Node Groups
```

---

is the only supported order.

---

# Wait For Upgrade Completion

Triggering an upgrade does not mean:

```text
Upgrade Completed
```

---

Production automation must wait.

---

Script Checks

```text
Cluster Status

Cluster Version
```

---

Every

```text
60 Seconds
```

---

Expected

```text
Status = ACTIVE

Version = Target Version
```

---

# Example

Current State

```text
UPDATING

Version 1.32
```

---

Wait

```text
60 Seconds
```

---

Check Again

```text
UPDATING

Version 1.33
```

---

Wait Again

---

Success

```text
ACTIVE

Version 1.33
```

---

# Failure Detection

If Status Becomes

```text
FAILED
```

---

Script Immediately

```text
Aborts Execution
```

---

Reason

```text
Prevent Further Damage
```

---

# Dynamic Addon Discovery

Most scripts hardcode addons.

Example

```text
CoreDNS

kube-proxy

VPC CNI
```

---

This script does NOT.

---

Command

```bash
aws eks list-addons
```

---

Example Output

```text
coredns

kube-proxy

vpc-cni

aws-ebs-csi-driver
```

---

Benefit

```text
Works Across Multiple Clusters
```

---

# Why Dynamic Discovery?

Cluster A

```text
CoreDNS

kube-proxy

VPC CNI
```

---

Cluster B

```text
CoreDNS

kube-proxy

VPC CNI

EBS CSI

EFS CSI
```

---

Same Script Works Everywhere.

---

# Addon Upgrade Workflow

For Every Addon

```text
Get Current Version

↓

Find Compatible Version

↓

Compare

↓

Upgrade If Required

↓

Wait Until ACTIVE
```

---

# Current Addon Version

Command

```bash
aws eks describe-addon
```

---

Example

```text
CoreDNS

1.11
```

---

# Finding Compatible Versions

Command

```bash
aws eks describe-addon-versions
```

---

Purpose

```text
Find Versions Compatible

With Target Kubernetes Version
```

---

# Important Concept

The script does NOT choose:

```text
Latest Overall Version
```

---

It chooses:

```text
Latest Compatible Version
```

---

# Example

Target Cluster

```text
1.33
```

---

Current CoreDNS

```text
1.11
```

---

Latest Overall

```text
1.15
```

---

Latest Compatible

```text
1.12
```

---

Script Selects

```text
1.12
```

---

Why?

```text
Compatibility Matters

More Than Latest Version
```

---

# Version Comparison

Current

```text
1.11
```

---

Compatible

```text
1.12
```

---

Action

```text
Upgrade
```

---

Current

```text
1.12
```

---

Compatible

```text
1.12
```

---

Action

```text
Skip
```

---

# Addon Upgrade

Command

```bash
aws eks update-addon
```

---

Example

```text
CoreDNS

1.11

↓

1.12
```

---

# Addon Status Monitoring

The script continuously monitors:

```text
Addon Status

Addon Version
```

---

Expected

```text
ACTIVE
```

---

# Healthy States

```text
ACTIVE
```

---

# Failure States

```text
FAILED

UPDATE_FAILED

DEGRADED
```

---

If Any Failure State Appears

```text
Abort Upgrade
```

---

Reason

```text
Cluster Stability
```

---

# Why Wait For Addons?

Without Waiting

```text
Multiple Upgrades

May Overlap
```

---

Result

```text
Difficult Troubleshooting
```

---

Therefore

```text
Upgrade One Addon

↓

Wait ACTIVE

↓

Upgrade Next Addon
```

---

# Real Production Example

Current

```text
Cluster Version

1.32
```

---

Installed Addons

```text
CoreDNS 1.11

kube-proxy 1.32

VPC CNI 1.18
```

---

Upgrade

```text
1.33
```

---

Script Flow

```text
Upgrade Control Plane

↓

Wait ACTIVE

↓

Discover Addons

↓

Upgrade CoreDNS

↓

Wait ACTIVE

↓

Upgrade kube-proxy

↓

Wait ACTIVE

↓

Upgrade VPC CNI

↓

Wait ACTIVE
```

---

# Post Upgrade Validation

After Upgrade Verify

```text
Nodes

Pods

Services

Ingress

Storage

Networking
```

---

# Verify Nodes

```bash
kubectl get nodes
```

---

Expected

```text
Ready
```

---

# Verify Pods

```bash
kubectl get pods -A
```

---

Expected

```text
Running
```

---

# Verify Ingress

```bash
kubectl get ingress
```

---

Expected

```text
ALB Reachable
```

---

# Verify Storage

```bash
kubectl get pvc -A
```

---

Expected

```text
Bound
```

---

# Common Production Failures

## Invalid Upgrade Path

Example

```text
1.32 → 1.35
```

---

Result

```text
Upgrade Rejected
```

---

## Cluster Upgrade Failure

Status

```text
FAILED
```

---

Action

```text
Abort Execution
```

---

## Compatible Version Not Found

Cause

```text
Addon Not Supported
```

---

Action

```text
Stop Upgrade
```

---

## Addon Stuck

Status

```text
DEGRADED
```

---

Action

```text
Investigate Before Continuing
```

---

## CoreDNS Failure

Symptoms

```text
DNS Resolution Failure
```

---

Impact

```text
Service Discovery Problems
```

---

# Production Architecture

```text
upgrade-cluster.sh

↓

Validate Parameters

↓

Get Current Version

↓

Parse Major/Minor

↓

Validate Upgrade Path

↓

Upgrade Control Plane

↓

Wait ACTIVE

↓

Discover Installed Addons

↓

Find Compatible Versions

↓

Upgrade Addons

↓

Wait ACTIVE

↓

Platform Validation
```

---

# Common Interview Questions

## How Do You Automate EKS Control Plane Upgrades?

### Short Answer

Using a shell script that validates versions, upgrades the control plane, waits for ACTIVE status, discovers addons, upgrades compatible versions, and validates the cluster.

### Production Example

Automated upgrade of roboshop-dev from 1.32 to 1.33.

---

## How Do You Prevent Unsupported Upgrades?

### Short Answer

Validate that the target version is exactly one minor version ahead.

### Example

```text
1.32 → 1.33

Allowed
```

```text
1.32 → 1.35

Rejected
```

---

## Why Not Upgrade Addons To The Latest Version?

### Short Answer

Latest version may not be compatible.

### Correct Approach

```text
Latest Compatible Version
```

---

## Why Use Dynamic Addon Discovery?

### Short Answer

Different clusters have different addons.

### Benefit

One script works across all environments.

---

## How Do You Handle Addon Upgrade Failures?

### Short Answer

Monitor status continuously and abort if addon becomes:

```text
FAILED

UPDATE_FAILED

DEGRADED
```

---

## Why Wait For ACTIVE Before Continuing?

### Short Answer

The upgrade request is asynchronous.

### Reason

The cluster must be fully upgraded before starting addon upgrades.

---

# Key Takeaways

```text
Production EKS upgrades should be automated.

Always validate upgrade paths.

Only upgrade one minor version at a time.

Wait for ACTIVE status before continuing.

Discover addons dynamically.

Upgrade only compatible addon versions.

Monitor addon health continuously.

Abort on FAILED, UPDATE_FAILED, or DEGRADED states.

Validate the entire platform after upgrade.

Automation improves consistency, reliability, and safety.
```