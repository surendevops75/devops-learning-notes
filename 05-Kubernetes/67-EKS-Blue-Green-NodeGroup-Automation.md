# EKS Blue-Green NodeGroup Automation

## Introduction

After upgrading:

```text
Control Plane

Addons
```

---

The next step is:

```text
Worker Node Upgrade
```

---

In production environments:

```text
Worker Nodes Run Applications
```

---

Therefore:

```text
Node Upgrade

Must Be Performed

Without Downtime
```

---

# Traditional Node Upgrade Problem

Suppose:

```text
Control Plane

1.33
```

---

Current Nodes

```text
1.32
```

---

If nodes are upgraded directly:

```text
Workloads May Be Impacted
```

---

Production teams prefer:

```text
Blue-Green NodeGroup Strategy
```

---

# What Is Blue-Green NodeGroup Upgrade?

Current Environment

```text
Blue NodeGroup

1.32
```

---

Create

```text
Green NodeGroup

1.33
```

---

Move Workloads

```text
Blue

↓

Green
```

---

Delete

```text
Blue
```

---

# High Level Architecture

Before Upgrade

```text
Control Plane 1.33

↓

Blue NG

1.32
```

---

After Upgrade

```text
Control Plane 1.33

↓

Green NG

1.33
```

---

# Production Automation Goal

Execute

```bash
sh upgrade-nodegroup.sh blue
```

---

Meaning

```text
Blue Is Active

Green Will Be Created

Workloads Will Move

Blue Will Be Deleted
```

---

# Dynamic NodeGroup Selection

The script does not assume:

```text
Blue Always Active
```

---

Input Can Be

```bash
sh upgrade-nodegroup.sh blue
```

or

```bash
sh upgrade-nodegroup.sh green
```

---

Logic

```text
Current = Blue

Target = Green
```

---

OR

```text
Current = Green

Target = Blue
```

---

# Why Is This Useful?

After multiple upgrades:

```text
Blue

↓

Green

↓

Blue

↓

Green
```

---

The script always works.

---

# Upgrade Workflow

```text
Identify Active NodeGroup

↓

Get Control Plane Version

↓

Get Current Node Version

↓

Create Target NodeGroup

↓

Wait For Ready Nodes

↓

Remove Upgrade Taints

↓

Cordon Current Nodes

↓

Drain Current Nodes

↓

Validate Workloads

↓

Delete Old NodeGroup
```

---

# Step 1 - Validate Inputs

Expected

```bash
sh upgrade-nodegroup.sh blue
```

---

Allowed Values

```text
blue

green
```

---

Invalid

```bash
sh upgrade-nodegroup.sh prod
```

---

Result

```text
Abort
```

---

# Step 2 - Determine Target NodeGroup

Current

```text
blue
```

---

Target

```text
green
```

---

Current

```text
green
```

---

Target

```text
blue
```

---

# Step 3 - Get Control Plane Version

Command

```bash
aws eks describe-cluster
```

---

Example

```text
1.33
```

---

Why?

```text
Control Plane

Is Source Of Truth
```

---

# Step 4 - Detect Current Node Version

Instead of trusting:

```text
Terraform State

Documentation

Variables
```

---

The script asks Kubernetes.

---

Command

```bash
kubectl get nodes \
-l nodegroup=blue
```

---

Extract

```text
kubeletVersion
```

---

Example

```text
v1.32.5-eks-xxxxx
```

---

Parsed Version

```text
1.32
```

---

# Why Read Actual Nodes?

Production Rule

```text
Running Infrastructure

Is Source Of Truth
```

---

# Example

Control Plane

```text
1.33
```

---

Current Blue

```text
1.32
```

---

Target Green

```text
1.33
```

---

# Terraform Driven Architecture

NodeGroups Are Created Through:

```text
Terraform
```

---

Variables

```text
enable_blue

enable_green
```

---

# Existing Environment

```text
enable_blue=true

enable_green=false
```

---

Meaning

```text
Only Blue Exists
```

---

# Create Target Environment

Terraform Variables

```text
enable_blue=true

enable_green=true
```

---

Meaning

```text
Blue Exists

Green Exists
```

---

# Version Assignment Logic

Example

Current

```text
Blue 1.32
```

---

Control Plane

```text
1.33
```

---

Terraform Plan

```text
Blue = 1.32

Green = 1.33
```

---

# Step 5 - Terraform Plan

Command

```bash
terraform plan
```

---

Purpose

```text
Review Infrastructure Changes
```

---

# Production Safety Gate

The script pauses.

---

Prompt

```text
Type YES To Continue
```

---

# Why?

Creating Infrastructure Is Safe.

---

Deleting Infrastructure Is Risky.

---

Human Approval Required.

---

# Step 6 - Create Target NodeGroup

Command

```bash
terraform apply
```

---

Result

```text
Green NodeGroup Created
```

---

Example

Before

```text
Blue

5 Nodes
```

---

After

```text
Blue

5 Nodes

+

Green

5 Nodes
```

---

# Why Same Capacity?

To handle workload migration.

---

# Step 7 - Wait For Target Nodes

Command

```bash
kubectl wait
```

---

Condition

```text
Ready
```

---

Timeout

```text
30 Minutes
```

---

# Why Wait?

Never move workloads to:

```text
Unhealthy Nodes
```

---

# Step 8 - Upgrade Taints

Target Nodes Usually Created With

```text
upgrade=true:NoSchedule
```

---

Purpose

```text
Prevent Scheduling
```

---

During Validation.

---

# Validate New Nodes

Check

```text
Node Status

Networking

Storage

Cluster Join
```

---

# Remove Taints

Command

```bash
kubectl taint node \
upgrade=true:NoSchedule-
```

---

Result

```text
Scheduler Can Use New Nodes
```

---

# Step 9 - Cordon Current Nodes

Command

```bash
kubectl cordon
```

---

Purpose

```text
Prevent New Scheduling
```

---

Example

```text
Blue Nodes

Scheduling Disabled
```

---

Existing Pods Continue Running.

---

# Cordon Does Not

```text
Delete Pods

Move Pods
```

---

Only

```text
Block New Scheduling
```

---

# Step 10 - Drain Current Nodes

Command

```bash
kubectl drain
```

---

Purpose

```text
Evict Existing Pods
```

---

Kubernetes Will

```text
Reschedule Pods

To Green Nodes
```

---

# Workload Migration

Before

```text
Blue Nodes

Running Pods
```

---

After

```text
Green Nodes

Running Pods
```

---

# Why Drain?

To migrate applications safely.

---

# Production Validation

After migration:

Check

```text
Pending Pods

CrashLoopBackOff

ImagePullBackOff
```

---

Command

```bash
kubectl get pods -A
```

---

Expected

```text
Healthy Workloads
```

---

# Step 11 - Delete Old NodeGroup

Terraform Variables

```text
enable_blue=false

enable_green=true
```

---

Meaning

```text
Delete Blue

Keep Green
```

---

Terraform Plan

```bash
terraform plan
```

---

Human Approval

```text
Type YES To Continue
```

---

Terraform Apply

```bash
terraform apply
```

---

Result

```text
Blue Removed
```

---

# Final State

```text
Control Plane 1.33

↓

Green NodeGroup 1.33
```

---

Workloads Running Successfully.

---

# Rollback Strategy

Suppose

```text
Green Fails
```

---

Do Not Delete Blue.

---

Rollback

```text
Cordon Green

↓

Drain Green

↓

Uncordon Blue

↓

Workloads Return
```

---

# Why Blue-Green Is Safe?

Old Environment Remains Available.

---

Rollback Time

```text
Minutes
```

---

# Large Scale Production Example

Cluster

```text
100 Nodes
```

---

Instead Of

```text
Replace 100 Nodes
```

---

Use

```text
Add 10 New Nodes

↓

Drain 10 Old Nodes

↓

Delete 10 Old Nodes

↓

Repeat
```

---

Benefits

```text
Lower Risk

Controlled Migration

Easier Troubleshooting
```

---

# Production Architecture

```text
upgrade-nodegroup.sh

↓

Detect Active NodeGroup

↓

Read Control Plane Version

↓

Read Node Version

↓

Terraform Plan

↓

Human Approval

↓

Create Target NodeGroup

↓

Wait Ready

↓

Remove Taints

↓

Cordon Old Nodes

↓

Drain Old Nodes

↓

Validate Pods

↓

Terraform Plan

↓

Human Approval

↓

Delete Old NodeGroup
```

---

# Common Production Problems

## Target Nodes Not Ready

Symptoms

```text
kubectl wait Timeout
```

---

Verify

```bash
kubectl get nodes
```

---

# Pods Stuck Pending

Cause

```text
Resource Shortage

Node Selectors

Taints
```

---

Verify

```bash
kubectl describe pod
```

---

# Workloads Not Moving

Cause

```text
Target Nodes Still Tainted
```

---

Verify

```bash
kubectl describe node
```

---

# Drain Failure

Cause

```text
Pod Disruption Budget
```

---

Verify

```bash
kubectl get pdb -A
```

---

# Terraform Failure

Verify

```bash
terraform plan
```

---

Review

```text
IAM

Subnets

NodeGroup Configuration
```

---

# Common Interview Questions

## Why Use Blue-Green NodeGroup Upgrades?

### Short Answer

To upgrade worker nodes without downtime and with fast rollback capability.

### Production Example

Blue 1.32 → Green 1.33 → Move Workloads → Delete Blue.

---

## Why Detect Node Version From Kubernetes Instead Of Terraform?

### Short Answer

Running infrastructure is the source of truth.

### Explanation

Terraform state may not always reflect the actual cluster state.

---

## Why Wait For Ready Nodes Before Draining?

### Short Answer

Workloads need healthy capacity before migration starts.

---

## Why Add Human Approval Gates?

### Short Answer

Infrastructure deletion and workload migration are high-risk actions.

### Explanation

Human approval prevents accidental production outages.

---

## Difference Between Cordon And Drain?

### Cordon

```text
Stop New Scheduling
```

### Drain

```text
Move Existing Workloads
```

---

## How Would You Rollback?

### Short Answer

Keep old nodegroup available until validation completes.

### Flow

```text
Cordon Green

↓

Drain Green

↓

Enable Blue

↓

Workloads Return
```

---

# Key Takeaways

```text
NodeGroup upgrades should follow Blue-Green principles.

Control Plane is the source of truth.

Always detect actual node versions.

Use Terraform to manage nodegroups.

Wait for nodes to become Ready.

Remove upgrade taints before migration.

Cordon blocks scheduling.

Drain moves workloads.

Validate workloads after migration.

Use human approval gates before destructive actions.

Keep rollback options available until validation is complete.
```