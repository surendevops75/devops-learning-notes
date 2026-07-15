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

Using a shell script that validates versions, upgrades the control plane, waits for ACTIVE status, discovers addons dynamically, upgrades compatible addon versions, and validates the cluster.

### Detailed Explanation

Production EKS upgrades should be automated to reduce human errors and ensure consistency. The script validates the target version, verifies upgrade compatibility, upgrades the control plane, continuously monitors cluster status, discovers installed addons, upgrades only compatible addon versions, and validates the cluster after completion.

### Production Example

```text
Current Cluster Version

1.32

↓

Execute

sh upgrade-cluster.sh 1.33

↓

Version Validation

↓

Control Plane Upgrade

↓

Wait ACTIVE

↓

Addon Discovery

↓

Addon Upgrades

↓

Cluster Validation
```

### Follow-Up Questions

- Why automate cluster upgrades?
- What checks do you perform before upgrading?
- Why wait for ACTIVE status?
- How do you validate upgrade success?

---

## How Do You Prevent Unsupported Kubernetes Upgrades?

### Short Answer

By validating that the target version is exactly one minor version ahead of the current version.

### Detailed Explanation

Kubernetes supports sequential minor upgrades. Skipping versions can cause compatibility issues and upgrade failures. The script parses major and minor versions and ensures the target version is exactly one step ahead.

### Production Example

```text
1.32 → 1.33

Allowed
```

```text
1.33 → 1.34

Allowed
```

```text
1.32 → 1.35

Rejected
```

### Follow-Up Questions

- What is version skew?
- Why are version skips dangerous?
- How does AWS handle unsupported upgrades?
- Can major versions be changed directly?

---

## Why Upgrade The Control Plane Before Addons?

### Short Answer

Because addons must be compatible with the upgraded Kubernetes control plane.

### Detailed Explanation

The control plane is the core of Kubernetes. Addons such as CoreDNS, kube-proxy, and VPC CNI rely on APIs exposed by the control plane. Upgrading addons before the control plane can lead to compatibility issues.

### Production Example

```text
Control Plane

1.32

↓

1.33

↓

CoreDNS Upgrade

↓

kube-proxy Upgrade

↓

VPC CNI Upgrade
```

### Follow-Up Questions

- What happens if addons are upgraded first?
- Which addons are critical?
- How do you verify addon compatibility?
- What is the correct EKS upgrade order?

---

## Why Use Dynamic Addon Discovery?

### Short Answer

Different clusters may have different installed addons.

### Detailed Explanation

Hardcoding addon names makes automation less reusable. Dynamic discovery allows the same script to work across multiple environments regardless of which addons are installed.

### Production Example

Cluster A:

```text
CoreDNS

kube-proxy

VPC CNI
```

Cluster B:

```text
CoreDNS

kube-proxy

VPC CNI

EBS CSI

EFS CSI
```

Same script works for both.

### Follow-Up Questions

- Which command lists addons?
- What happens if an addon is missing?
- Can addon lists differ between environments?
- Why avoid hardcoding?

---

## How Do You Determine Which Addon Version To Upgrade To?

### Short Answer

The script selects the latest version compatible with the target Kubernetes version.

### Detailed Explanation

The latest addon version is not always compatible with the target cluster version. The script queries AWS addon compatibility information and chooses the newest supported version for the upgraded cluster.

### Production Example

```text
Target Cluster Version

1.33
```

Current CoreDNS:

```text
1.11
```

Latest Overall:

```text
1.15
```

Latest Compatible:

```text
1.12
```

Selected Version:

```text
1.12
```

### Follow-Up Questions

- Which AWS command provides compatibility information?
- Why not use the latest version?
- How do you handle compatibility issues?
- Can incompatible addons break the cluster?

---

## Why Not Upgrade Addons To The Latest Available Version?

### Short Answer

Because the latest version may not be compatible with the target Kubernetes version.

### Detailed Explanation

Compatibility is more important than version numbers. Production upgrades should always use supported versions rather than blindly upgrading to the newest release.

### Production Example

```text
Cluster Version

1.33
```

Latest CoreDNS:

```text
1.15
```

Compatible CoreDNS:

```text
1.12
```

Upgrade Target:

```text
1.12
```

### Follow-Up Questions

- How do you verify compatibility?
- What happens if compatibility is ignored?
- Which addons are most sensitive?
- How do you rollback addon upgrades?

---

## How Do You Monitor Control Plane Upgrade Progress?

### Short Answer

By continuously polling cluster status and version until the cluster becomes ACTIVE.

### Detailed Explanation

EKS upgrades are asynchronous. The upgrade request returns immediately, but the actual upgrade may take several minutes. The script repeatedly checks cluster status and version before continuing.

### Production Example

```text
UPDATING

↓

UPDATING

↓

ACTIVE
```

Version:

```text
1.32

↓

1.33
```

### Follow-Up Questions

- Why not proceed immediately?
- How often do you poll?
- What status values can appear?
- What happens if the cluster stays UPDATING?

---

## How Do You Handle Addon Upgrade Failures?

### Short Answer

The script monitors addon status and immediately aborts if a failure state is detected.

### Detailed Explanation

Continuing an upgrade after a failed addon can cause larger platform issues. The script checks addon health continuously and stops execution if any addon enters a failure state.

### Production Example

Monitored States:

```text
ACTIVE

FAILED

UPDATE_FAILED

DEGRADED
```

Abort On:

```text
FAILED

UPDATE_FAILED

DEGRADED
```

### Follow-Up Questions

- How do you troubleshoot DEGRADED addons?
- What causes addon failures?
- Which addon failures are most critical?
- How do you recover from a failed upgrade?

---

## How Do You Validate An EKS Upgrade?

### Short Answer

Validate nodes, pods, services, ingress, storage, and networking after the upgrade.

### Detailed Explanation

Version changes alone do not guarantee success. The entire platform must be validated to ensure workloads continue operating correctly.

### Production Example

Verify:

```text
Nodes Ready

Pods Running

PVCs Bound

Ingress Reachable

DNS Working

Monitoring Healthy
```

### Follow-Up Questions

- Which kubectl commands do you use?
- How do you validate CoreDNS?
- How do you validate networking?
- How do you validate storage?

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