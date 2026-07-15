# Pod Distribution Strategies: PDB, PodAntiAffinity and TopologySpreadConstraints

## Introduction

Running multiple replicas does not automatically guarantee:

```text
High Availability
```

---

Example

Deployment

```text
Catalogue

Replicas = 4
```

---

Scheduler May Place:

```text
Pod-1

Pod-2

Pod-3

Pod-4
```

on:

```text
Node-1
```

---

Result

```text
Single Point Of Failure
```

---

If Node-1 Fails

```text
All Pods Lost
```

---

Production Clusters Use:

```text
PodDisruptionBudget

PodAntiAffinity

TopologySpreadConstraints
```

to ensure:

```text
High Availability

Fault Tolerance

Safe Upgrades
```

---

# Production Problem

Cluster

```text
Node-1

Node-2
```

---

Application

```text
Catalogue

4 Replicas
```

---

Bad Distribution

```text
Node-1

Pod-1
Pod-2
Pod-3
Pod-4

Node-2

Empty
```

---

Node Failure

```text
Node-1 Down
```

---

Result

```text
Application Outage
```

---

# Solution

Use:

```text
PodAntiAffinity

TopologySpreadConstraints

PDB
```

---

# Understanding PodDisruptionBudget (PDB)

## What Is PDB?

PDB stands for:

```text
Pod Disruption Budget
```

---

It protects applications during:

```text
Node Upgrades

Node Maintenance

Drain Operations

Cluster Upgrades
```

---

PDB Does NOT Protect Against

```text
Node Crash

AZ Failure

Hardware Failure
```

---

PDB Controls

```text
Voluntary Disruptions
```

---

Examples

```text
kubectl drain

Node Upgrade

Cluster Upgrade
```

---

# Why PDB Is Required?

Suppose

```text
4 Replicas
```

---

Without PDB

Node Drain

↓

All Pods Evicted

↓

Downtime
```

---

With PDB

```text
Minimum Pods

Must Remain Running
```

---

# Example

Deployment

```text
4 Replicas
```

---

PDB

```yaml
minAvailable: 2
```

---

Meaning

```text
At Least

2 Pods

Must Stay Running
```

---

# PDB Example YAML

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget

metadata:
  name: catalogue-pdb

spec:

  minAvailable: 2

  selector:
    matchLabels:
      app: catalogue
```

---

# PDB Flow

Before Drain

```text
Pod-1

Pod-2

Pod-3

Pod-4
```

---

Allowed

```text
Only 2 Pods

Can Be Disrupted
```

---

# Important Limitation

PDB Assumes

```text
Replicas Exist
```

---

If All Pods Run On One Node

```text
Node Failure

↓

All Pods Lost
```

---

PDB Cannot Help.

---

# What Is PodAntiAffinity?

PodAntiAffinity controls:

```text
Where Pods

Should NOT Run
```

---

Goal

```text
Spread Pods

Across Nodes
```

---

# Example

Deployment

```text
Catalogue

4 Replicas
```

---

Desired

```text
Node-1

Pod-1
Pod-2

Node-2

Pod-3
Pod-4
```

---

Instead Of

```text
Node-1

Pod-1
Pod-2
Pod-3
Pod-4
```

---

# How PodAntiAffinity Works

Scheduler Checks

```text
Existing Pods
```

---

Rule

```text
Do Not Schedule

Near Similar Pods
```

---

# Example YAML

```yaml
podAntiAffinity:

  requiredDuringSchedulingIgnoredDuringExecution:

  - labelSelector:

      matchExpressions:

      - key: app

        operator: In

        values:
        - catalogue

    topologyKey: kubernetes.io/hostname
```

---

# What Does This Mean?

```text
Do Not Place

Catalogue Pods

On Same Node
```

---

Topology

```text
hostname
```

means:

```text
Node Level
```

---

# Result

```text
Node-1

Catalogue-1

Node-2

Catalogue-2

Node-3

Catalogue-3
```

---

# Benefits

```text
Node Failure Protection

Better Availability

Safer Upgrades
```

---

# Limitation

Requires

```text
Enough Nodes
```

---

Example

```text
3 Replicas

1 Node
```

---

Result

```text
Pods Pending
```

---

# What Is TopologySpreadConstraint?

PodAntiAffinity helps spread pods.

---

TopologySpreadConstraint ensures:

```text
Even Distribution
```

---

Across:

```text
Nodes

Availability Zones

Custom Topologies
```

---

# Why Needed?

Suppose

```text
6 Replicas

3 Nodes
```

---

Bad Distribution

```text
Node-1

4 Pods

Node-2

1 Pod

Node-3

1 Pod
```

---

Still Valid.

---

But Not Balanced.

---

TopologySpreadConstraint Fixes This.

---

# Example

Desired

```text
Node-1

2 Pods

Node-2

2 Pods

Node-3

2 Pods
```

---

# Example YAML

```yaml
topologySpreadConstraints:

- maxSkew: 1

  topologyKey: kubernetes.io/hostname

  whenUnsatisfiable: DoNotSchedule

  labelSelector:

    matchLabels:
      app: catalogue
```

---

# Understanding maxSkew

Controls

```text
Maximum Difference

Between Topologies
```

---

Example

```text
Node-1

2 Pods

Node-2

2 Pods

Node-3

1 Pod
```

---

Difference

```text
1
```

---

Allowed.

---

Example

```text
Node-1

4 Pods

Node-2

1 Pod

Node-3

1 Pod
```

---

Difference

```text
3
```

---

Rejected.

---

# Topology Key

Node Level

```text
kubernetes.io/hostname
```

---

AZ Level

```text
topology.kubernetes.io/zone
```

---

# Production Example

EKS

```text
us-east-1a

us-east-1b
```

---

Application

```text
Catalogue

4 Replicas
```

---

Desired

```text
1a

2 Pods

1b

2 Pods
```

---

Result

```text
AZ Failure

Application Still Running
```

---

# Combining All Three

Production Setup

```text
PDB

+

PodAntiAffinity

+

TopologySpreadConstraint
```

---

Example

```text
4 Replicas
```

---

PodAntiAffinity

```text
Spread Across Nodes
```

---

TopologySpreadConstraint

```text
Balance Across Nodes/AZs
```

---

PDB

```text
Protect During Drains
```

---

# Real Production Scenario

Application

```text
Catalogue

4 Replicas
```

---

Cluster

```text
Node-1

Node-2

Node-3

Node-4
```

---

Distribution

```text
Node-1

Pod-1

Node-2

Pod-2

Node-3

Pod-3

Node-4

Pod-4
```

---

Node Upgrade

```text
Drain Node-1
```

---

PDB

```text
Ensures

Minimum Pods

Remain Available
```

---

Scheduler

```text
Places New Pod

On Healthy Node
```

---

Application Remains Available.

---

# Production Architecture

```text
Deployment

↓

4 Replicas

↓

PodAntiAffinity

↓

Spread Pods

↓

TopologySpreadConstraint

↓

Balanced Distribution

↓

PDB

↓

Safe Node Maintenance
```

---

# Common Production Problems

## PDB Blocking Drain

Symptoms

```text
kubectl drain

Stuck
```

---

Reason

```text
minAvailable

Would Be Violated
```

---

Check

```bash
kubectl get pdb
```

---

# Pods Pending

Cause

```text
Strict PodAntiAffinity

Not Enough Nodes
```

---

Check

```bash
kubectl describe pod
```

---

# Uneven Distribution

Cause

```text
Missing TopologySpreadConstraint
```

---

Result

```text
Node Hotspots
```

---

# AZ Failure Impact

Cause

```text
All Pods

In Single AZ
```

---

Solution

```text
TopologySpreadConstraint

By Zone
```

---

# Troubleshooting Commands

Pods

```bash
kubectl get pods -o wide
```

---

PDB

```bash
kubectl get pdb
```

---

Describe PDB

```bash
kubectl describe pdb
```

---

Node Distribution

```bash
kubectl get pods -o wide
```

---

Pod Events

```bash
kubectl describe pod <pod>
```

---

# Common Interview Questions

## What Is A PodDisruptionBudget?

### Short Answer

A PDB protects application availability during voluntary disruptions such as node maintenance and drain operations.

### Detailed Explanation

PDB defines how many Pods must remain available during maintenance activities. Kubernetes prevents disruptions that would violate the PDB.

### Production Example

```text
Replicas = 4

minAvailable = 2
```

At least:

```text
2 Pods

Must Remain Running
```

### Follow-Up Questions

- Does PDB protect against node failures?
- What is a voluntary disruption?
- What happens during drain?
- How do you verify PDB status?

---

## What Is PodAntiAffinity?

### Short Answer

PodAntiAffinity prevents similar Pods from being scheduled on the same node.

### Detailed Explanation

It improves high availability by spreading replicas across multiple nodes.

### Production Example

Bad

```text
Node-1

Pod-1
Pod-2
Pod-3
```

Good

```text
Node-1

Pod-1

Node-2

Pod-2

Node-3

Pod-3
```

### Follow-Up Questions

- What is topologyKey?
- What happens if nodes are unavailable?
- Difference between Affinity and AntiAffinity?
- What causes Pods to remain Pending?

---

## What Is TopologySpreadConstraint?

### Short Answer

TopologySpreadConstraint ensures balanced Pod distribution across nodes or availability zones.

### Detailed Explanation

It prevents workloads from becoming concentrated on a small number of nodes or AZs.

### Production Example

```text
6 Pods

3 Nodes

↓

2 Pods Per Node
```

### Follow-Up Questions

- What is maxSkew?
- Difference from PodAntiAffinity?
- Why use AZ-level spreading?
- What happens if constraints cannot be satisfied?

---

## Why Use PDB, PodAntiAffinity and TopologySpreadConstraint Together?

### Short Answer

Together they provide availability, fault tolerance, and safe maintenance.

### Detailed Explanation

PodAntiAffinity spreads workloads, TopologySpreadConstraint balances them, and PDB protects them during maintenance.

### Production Example

```text
4 Replicas

↓

Spread Across Nodes

↓

Balance Across AZs

↓

Protected During Drain
```

### Follow-Up Questions

- Which one handles node failure?
- Which one handles maintenance?
- Which one handles AZ distribution?
- Which should be implemented first?

---

# Key Takeaways

```text
PDB protects applications during voluntary disruptions.

PDB does not protect against node crashes.

PodAntiAffinity spreads replicas across nodes.

TopologySpreadConstraint balances replicas across nodes or AZs.

topologyKey controls where spreading occurs.

hostname spreads across nodes.

zone spreads across Availability Zones.

PDB, PodAntiAffinity, and TopologySpreadConstraint should be used together in production.

These features improve availability, fault tolerance, and upgrade safety.

Proper pod distribution is critical for highly available Kubernetes applications.
```