# Deployments and Rolling Updates

## Introduction

ReplicaSets ensure:

```text
Desired Number Of Pods
```

are running.

---

However modern applications need more than:

```text
Pod Creation

Self Healing

Scaling
```

---

Applications also require:

```text
Version Upgrades

Rollbacks

Zero Downtime Deployments
```

---

Kubernetes provides:

```text
Deployments
```

for this purpose.

---

# What is a Deployment?

A Deployment is a higher-level Kubernetes resource that manages:

```text
ReplicaSets

Pods
```

---

Architecture

```text
Deployment
      ↓

ReplicaSet
      ↓

Pods
```

---

Think of Deployment as:

```text
Application Release Manager
```

---

# Why Do We Need Deployments?

Suppose:

```text
frontend:v1
```

is running.

---

New version available:

```text
frontend:v2
```

---

Without Deployment:

```text
Delete Old Pods

Create New Pods
```

---

Problems

```text
Downtime

Manual Effort

No Rollback
```

---

Deployments solve these issues.

---

# Deployment Architecture

```text
Deployment
      ↓

ReplicaSet
      ↓

Pods
```

---

Example

```text
Frontend Deployment
          ↓

Frontend ReplicaSet
          ↓

Frontend Pods
```

---

# Deployment YAML

Example

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:
  name: frontend

spec:

  replicas: 2

  selector:
    matchLabels:
      app: frontend

  template:

    metadata:
      labels:
        app: frontend

    spec:

      containers:

      - name: frontend

        image: nginx:1.25
```

---

Create Deployment

```bash
kubectl apply -f deployment.yaml
```

---

Verify

```bash
kubectl get deployments
```

---

Output

```text
frontend
```

---

# Deployment Creates ReplicaSet

When Deployment is created:

```text
Deployment
      ↓

Creates ReplicaSet
      ↓

Creates Pods
```

---

Verify

```bash
kubectl get rs
```

---

Example

```text
frontend-7f6c9f
```

---

Verify Pods

```bash
kubectl get pods
```

---

# Scaling Deployments

Deployment supports scaling.

---

Example

```yaml
replicas: 5
```

---

Result

```text
5 Pods Running
```

---

Scale Using Command

```bash
kubectl scale deployment frontend \
--replicas=5
```

---

Verify

```bash
kubectl get deployment
```

---

# Updating Applications

Current Version

```text
frontend:v1
```

---

Requirement

```text
frontend:v2
```

---

Update Deployment

```yaml
image: frontend:v2
```

---

Apply

```bash
kubectl apply -f deployment.yaml
```

---

Deployment starts:

```text
Rolling Update
```

---

# What is a Rolling Update?

Rolling Update means:

```text
Update Pods Gradually
```

without downtime.

---

Example

Current State

```text
frontend:v1

Pod-1

Pod-2
```

---

Target State

```text
frontend:v2

Pod-1

Pod-2
```

---

# Rolling Update Process

Step 1

```text
Create New Pod

frontend:v2
```

---

Step 2

```text
Wait Until Healthy
```

---

Step 3

```text
Delete One Old Pod

frontend:v1
```

---

Step 4

```text
Create Another v2 Pod
```

---

Step 5

```text
Delete Remaining v1 Pod
```

---

Final State

```text
frontend:v2

Pod-1

Pod-2
```

---

# Trainer Example

Current

```text
frontend:v1

2 Replicas
```

---

Update To

```text
frontend:v2
```

---

Deployment Process

```text
Create One v2 Pod
      ↓

Delete One v1 Pod
      ↓

Create Another v2 Pod
      ↓

Delete Remaining v1 Pod
```

---

Result

```text
Zero Downtime Upgrade
```

---

# Rolling Update Architecture

```text
Deployment
      ↓

Old ReplicaSet (v1)
      ↓

Pod-1
Pod-2

      ↓

New ReplicaSet (v2)
      ↓

Pod-3
Pod-4
```

---

During update:

```text
Both ReplicaSets Exist
```

temporarily.

---

# Deployment Revision History

Every update creates:

```text
New ReplicaSet
```

---

Example

```text
Revision 1

frontend:v1
```

---

Update

```text
Revision 2

frontend:v2
```

---

Update Again

```text
Revision 3

frontend:v3
```

---

Deployment keeps history.

---

# Viewing Rollout Status

Check Update Progress

```bash
kubectl rollout status deployment/frontend
```

---

Output

```text
deployment "frontend"
successfully rolled out
```

---

# Viewing Rollout History

Command

```bash
kubectl rollout history deployment/frontend
```

---

Output

```text
REVISION

1

2

3
```

---

Useful for troubleshooting.

---

# What is Rollback?

Rollback means:

```text
Return To Previous Version
```

---

Example

Current

```text
frontend:v3
```

---

Problem Found

```text
Application Failure
```

---

Rollback

```text
frontend:v2
```

---

# Rollback Command

```bash
kubectl rollout undo deployment/frontend
```

---

Result

```text
Previous Version Restored
```

---

# Rollback To Specific Revision

View History

```bash
kubectl rollout history deployment/frontend
```

---

Rollback

```bash
kubectl rollout undo deployment/frontend \
--to-revision=2
```

---

Result

```text
frontend:v2
```

---

# Deployment Strategies

Kubernetes supports multiple deployment strategies.

---

Most Common

```text
Rolling Update
```

---

Other Strategy

```text
Recreate
```

---

# Recreate Strategy

Process

```text
Delete All Old Pods
      ↓

Create New Pods
```

---

Advantage

```text
Simple
```

---

Disadvantage

```text
Downtime
```

---

Not preferred for production.

---

# Rolling Update Strategy

Process

```text
Gradually Replace Pods
```

---

Advantages

```text
Zero Downtime

Safer Deployments

Easy Rollback
```

---

Most commonly used strategy.

---

# Deployment vs ReplicaSet

| Feature | ReplicaSet | Deployment |
|----------|------------|------------|
| Pod Management | Yes | Yes |
| Self Healing | Yes | Yes |
| Scaling | Yes | Yes |
| Rolling Updates | No | Yes |
| Rollbacks | No | Yes |
| Revision History | No | Yes |
| Production Usage | Rare | Common |

---

# Real Production Example

Roboshop Frontend

Current Version

```text
frontend:v1
```

---

New Release

```text
frontend:v2
```

---

Deployment performs:

```text
Rolling Update
```

---

Users continue accessing application.

---

Benefits

```text
No Downtime

Easy Rollback

Safe Upgrades
```

---

# Deployment Flow

```text
Developer Pushes Code
         ↓

CI Pipeline Builds Image
         ↓

New Image Version
         ↓

Deployment Updated
         ↓

New ReplicaSet Created
         ↓

Rolling Update Starts
         ↓

Pods Updated
         ↓

Application Upgraded
```

---

# Troubleshooting Deployments

View Deployments

```bash
kubectl get deployments
```

---

Describe Deployment

```bash
kubectl describe deployment frontend
```

---

Check ReplicaSets

```bash
kubectl get rs
```

---

Check Pods

```bash
kubectl get pods
```

---

Check Rollout Status

```bash
kubectl rollout status deployment/frontend
```

---

# Common Problems

## ImagePullBackOff

Cause

```text
Wrong Image Name
```

---

Check

```bash
kubectl describe pod pod-name
```

---

## CrashLoopBackOff

Cause

```text
Application Configuration Error
```

---

Check

```bash
kubectl logs pod-name
```

---

## Stuck Rollout

Cause

```text
Failed Health Checks

Resource Issues
```

---

Check

```bash
kubectl rollout status deployment/frontend
```

---

# Common Interview Questions

## What is a Deployment?

### Short Answer

A Deployment manages ReplicaSets and Pods while providing rolling updates and rollbacks.

### Detailed Explanation

Deployments automate application updates and maintain application availability.

### Production Example

Frontend deployment running multiple replicas.

### Follow-Up Questions

- Does Deployment create ReplicaSets?
- Why not use ReplicaSets directly?

---

## What is a Rolling Update?

### Short Answer

A rolling update gradually replaces old Pods with new Pods.

### Detailed Explanation

Kubernetes updates applications without downtime by replacing Pods one at a time.

### Production Example

Upgrading frontend:v1 to frontend:v2.

### Follow-Up Questions

- How does Kubernetes avoid downtime?
- What happens if update fails?

---

## What is Rollback?

### Short Answer

Rollback restores a previous deployment version.

### Detailed Explanation

Deployments maintain revision history, allowing quick recovery from bad releases.

### Production Example

Rolling back frontend:v3 to frontend:v2.

### Follow-Up Questions

- How do you view deployment history?
- How do you rollback to a specific revision?

---

## Why Are Deployments Used In Production?

### Short Answer

Deployments provide scaling, self-healing, rolling updates, and rollbacks.

### Detailed Explanation

They simplify application lifecycle management and reduce deployment risk.

### Production Example

Roboshop frontend upgrades without downtime.

### Follow-Up Questions

- What resources does Deployment manage?
- What deployment strategies are available?

---

# Key Takeaways

```text
Deployments manage ReplicaSets and Pods.

Deployments support scaling and self-healing.

Deployments provide rolling updates.

Deployments provide rollbacks.

Rolling updates enable zero downtime deployments.

Each deployment update creates a new ReplicaSet.

Deployment history enables quick rollback.

Rolling Update is the most common production strategy.

Deployments are the standard way to run stateless applications in Kubernetes.
```