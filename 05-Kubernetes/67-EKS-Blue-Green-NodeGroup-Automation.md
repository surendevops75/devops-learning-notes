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

To upgrade worker nodes without downtime and provide a fast rollback mechanism.

### Detailed Explanation

Blue-Green nodegroup upgrades create a new nodegroup running the target Kubernetes version while keeping the current nodegroup active. After validating the new nodes, workloads are migrated using cordon and drain operations. Once validation is complete, the old nodegroup is removed.

### Production Example

```text
Blue NG 1.32

↓

Green NG 1.33

↓

Wait For Ready Nodes

↓

Drain Blue

↓

Move Workloads

↓

Delete Blue
```

### Follow-Up Questions

- Why not upgrade nodes in-place?
- How much extra capacity is required?
- How do you rollback?
- When do you delete the old nodegroup?

---

## Why Is The Control Plane Considered The Source Of Truth?

### Short Answer

New nodegroups should always match the currently running control plane version.

### Detailed Explanation

The script reads the cluster version directly from EKS and uses that version while creating the target nodegroup. This ensures worker nodes remain compatible with the upgraded control plane.

### Production Example

```text
Control Plane

1.33
```

Current NodeGroup

```text
1.32
```

Target NodeGroup

```text
1.33
```

### Follow-Up Questions

- What happens if node versions are ahead of the control plane?
- Can worker nodes be newer than the control plane?
- What is version skew?
- How do you retrieve cluster version?

---

## Why Detect Node Versions From Kubernetes Instead Of Terraform?

### Short Answer

Because running infrastructure is the source of truth.

### Detailed Explanation

Terraform state may become outdated because of manual changes, failed applies, or state drift. Querying Kubernetes directly ensures the script reads the actual kubelet version running on worker nodes.

### Production Example

Command:

```bash
kubectl get nodes \
-l nodegroup=blue \
-o jsonpath='{.items[0].status.nodeInfo.kubeletVersion}'
```

Output:

```text
v1.32.5-eks-xxxxx
```

Parsed Version:

```text
1.32
```

### Follow-Up Questions

- What is state drift?
- Can Terraform state become stale?
- Why is kubeletVersion important?
- How do you validate node versions?

---

## Why Create The New NodeGroup Before Draining The Old One?

### Short Answer

To ensure sufficient capacity exists before workload migration.

### Detailed Explanation

Pods cannot be safely migrated unless healthy target nodes already exist. The new nodegroup must be fully provisioned and validated before workloads are moved.

### Production Example

Before:

```text
Blue

5 Nodes
```

Create:

```text
Green

5 Nodes
```

Validate:

```text
All Nodes Ready
```

Only then:

```text
Drain Blue
```

### Follow-Up Questions

- What happens if target nodes are unavailable?
- How do you validate node readiness?
- What if workloads exceed available capacity?
- Why not drain first?

---

## Why Wait For Nodes To Become Ready?

### Short Answer

Workloads should only run on healthy nodes.

### Detailed Explanation

A node joining the cluster does not immediately mean it is ready to run production workloads. The script waits until Kubernetes reports the node condition as Ready before migration begins.

### Production Example

Command:

```bash
kubectl wait \
--for=condition=Ready
```

Timeout:

```text
30 Minutes
```

### Follow-Up Questions

- What conditions make a node Ready?
- What if timeout occurs?
- How do you troubleshoot NotReady nodes?
- Which command verifies node health?

---

## Why Use Upgrade Taints On New Nodes?

### Short Answer

To prevent workloads from scheduling before validation is completed.

### Detailed Explanation

New nodes are created with a temporary NoSchedule taint. This allows the platform team to verify node health, networking, storage, and cluster connectivity before workloads are allowed to run.

### Production Example

Taint:

```text
upgrade=true:NoSchedule
```

Flow:

```text
Create Nodes

↓

Validate Nodes

↓

Remove Taint

↓

Allow Scheduling
```

### Follow-Up Questions

- What is a taint?
- What is NoSchedule?
- Why not allow scheduling immediately?
- How do you remove taints?

---

## What Is The Difference Between Cordon And Drain?

### Short Answer

Cordon blocks new scheduling. Drain moves existing workloads.

### Detailed Explanation

Cordon marks nodes as unschedulable but leaves existing workloads running. Drain safely evicts workloads and allows Kubernetes to recreate them elsewhere.

### Production Example

Cordon:

```text
No New Pods
```

Drain:

```text
Existing Pods Migrated
```

### Follow-Up Questions

- Does cordon remove pods?
- What happens during drain?
- Can a drained node still run pods?
- Which step comes first?

---

## Why Add Human Approval Gates?

### Short Answer

Because production migrations and infrastructure deletion are high-risk operations.

### Detailed Explanation

The script intentionally pauses before creating or deleting infrastructure. Human approval ensures that the operator reviews the planned changes before proceeding.

### Production Example

Prompt:

```text
Type YES To Continue
```

Before:

```text
NodeGroup Creation

NodeGroup Deletion

Workload Migration
```

### Follow-Up Questions

- Why not fully automate?
- Which steps are most dangerous?
- What happens if someone makes a mistake?
- How do large enterprises handle approvals?

---

## How Do You Validate Workload Migration?

### Short Answer

Check application health after drain operations complete.

### Detailed Explanation

After workloads move to the target nodegroup, the platform team validates that pods are healthy and no scheduling issues exist.

### Production Example

Check For:

```text
Pending

CrashLoopBackOff

ImagePullBackOff
```

Command:

```bash
kubectl get pods -A
```

### Follow-Up Questions

- How do you validate application health?
- What causes Pending pods?
- What causes CrashLoopBackOff?
- What logs would you check?

---

## How Would You Rollback A NodeGroup Upgrade?

### Short Answer

Keep the old nodegroup until validation is complete and move workloads back if necessary.

### Detailed Explanation

Blue-Green architecture enables rapid rollback because the previous environment remains available until the upgrade is verified.

### Production Example

```text
Green Problem

↓

Cordon Green

↓

Drain Green

↓

Uncordon Blue

↓

Workloads Return
```

### Follow-Up Questions

- Why keep Blue alive?
- When is rollback triggered?
- How long should Blue remain available?
- What validations are required before deletion?

---

## How Would You Upgrade A 100-Node Production Cluster?

### Short Answer

Upgrade in batches instead of replacing all nodes at once.

### Detailed Explanation

Large clusters are upgraded gradually to reduce risk and avoid capacity issues. Small batches are migrated, validated, and then removed before moving to the next batch.

### Production Example

```text
100 Nodes

↓

Add 10 New Nodes

↓

Drain 10 Old Nodes

↓

Delete 10 Old Nodes

↓

Repeat
```

### Follow-Up Questions

- Why not upgrade all 100 nodes together?
- How do you determine batch size?
- What risks exist during migration?
- How do you monitor progress?

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