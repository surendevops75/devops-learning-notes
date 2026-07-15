# Kubernetes Deployment Strategies: Rolling Update vs Blue-Green

## Introduction

Deploying a new application version in production is not simply:

```text
Build

↓

Deploy

↓

Done
```

---

Production deployments require:

```text
High Availability

Zero Downtime

Fast Rollback

Risk Reduction
```

---

Kubernetes supports multiple deployment strategies.

The two most common are:

```text
Rolling Update

Blue-Green Deployment
```

---

# Why Deployment Strategies Matter?

Suppose:

```text
Catalogue v4.0

Running In Production
```

---

A new version is released:

```text
Catalogue v5.0
```

---

Questions:

```text
How To Deploy?

How To Avoid Downtime?

How To Rollback?
```

---

Deployment strategies answer these questions.

---

# Rolling Update

## Definition

Rolling Update gradually replaces old pods with new pods.

---

Instead of:

```text
Stop All Old Pods

↓

Start All New Pods
```

---

Kubernetes performs:

```text
Create New Pods

↓

Wait Ready

↓

Delete Old Pods

↓

Repeat
```

---

# Rolling Update Workflow

Current State

```text
4 Replicas

v4.0
```

---

Running

```text
Pod-1 v4

Pod-2 v4

Pod-3 v4

Pod-4 v4
```

---

Deploy

```text
v5.0
```

---

Step 1

```text
Create 2 Pods

v5.0
```

---

State

```text
Pod-1 v4

Pod-2 v4

Pod-3 v4

Pod-4 v4

Pod-5 v5

Pod-6 v5
```

---

Step 2

```text
Delete 2 Old Pods
```

---

Step 3

```text
Create 2 New Pods
```

---

Step 4

```text
Delete Remaining Old Pods
```

---

Final State

```text
Pod-5 v5

Pod-6 v5

Pod-7 v5

Pod-8 v5
```

---

# Important Characteristic

During deployment:

```text
Old Version

AND

New Version

Run Together
```

---

Example

```text
v4

+

v5
```

---

For a short period.

---

# Production Challenge

Application Teams Must Ensure:

```text
Backward Compatibility
```

---

Because:

```text
Old Version

And

New Version

Exist Simultaneously
```

---

# Example

API

```text
/api/v1
```

---

New Version

```text
/api/v2
```

---

If v5 removes support for:

```text
/api/v1
```

---

Problems may occur during rollout.

---

# Kubernetes Rolling Update Settings

Deployment Strategy

```yaml
strategy:
  type: RollingUpdate
```

---

Important Parameters

```yaml
maxUnavailable
maxSurge
```

---

# maxUnavailable

Controls:

```text
How Many Pods

Can Be Down
```

---

Example

```text
4 Replicas

maxUnavailable=1
```

---

At least:

```text
3 Pods

Remain Available
```

---

# maxSurge

Controls:

```text
Extra Pods

Created During Deployment
```

---

Example

```text
4 Replicas

maxSurge=1
```

---

Temporary Pods

```text
5
```

---

During rollout.

---

# Rolling Update Advantages

```text
No Downtime

Simple

Default Kubernetes Strategy

No Duplicate Infrastructure
```

---

# Rolling Update Disadvantages

```text
Old And New Versions Coexist

Rollback Takes Time

Application Compatibility Required
```

---

# Blue-Green Deployment

## Definition

Blue-Green creates:

```text
Two Independent Environments
```

---

Environment A

```text
Blue
```

---

Environment B

```text
Green
```

---

Only one receives production traffic.

---

# Blue-Green Workflow

Current

```text
Blue

v4.0
```

---

Create

```text
Green

v5.0
```

---

Validate

```text
Application

Database

API

Ingress
```

---

Switch Traffic

```text
Blue

↓

Green
```

---

Delete Blue Later.

---

# Key Difference

Rolling Update

```text
Old And New Versions

Run Together
```

---

Blue-Green

```text
Only One Version

Receives Traffic
```

---

# Kubernetes Blue-Green Example

Blue Pods

```text
app=v4

color=blue
```

---

Green Pods

```text
app=v5

color=green
```

---

Current Service

```yaml
selector:
  color: blue
```

---

Traffic

```text
Users

↓

Blue Pods
```

---

# Service Switch

Command

```bash
kubectl patch service nginx \
-p '{"spec":{"selector":{"color":"green"}}}'
```

---

Result

```text
Users

↓

Green Pods
```

---

Traffic Switch Happens Instantly.

---

# Production Example

Current

```text
Catalogue v4
```

---

Blue

```text
Serving Traffic
```

---

Green

```text
Catalogue v5
```

---

Validation Completed.

---

Switch Service Selector

```text
Blue

↓

Green
```

---

Users Start Using:

```text
v5
```

---

# Blue-Green Rollback

Problem Found

```text
v5 Error
```

---

Rollback

```text
Green

↓

Blue
```

---

Service Selector Changed Back.

---

Rollback Time

```text
Seconds
```

---

# Blue-Green Advantages

```text
Easy Rollback

No Mixed Versions

Safe Testing

Near Zero Downtime
```

---

# Blue-Green Disadvantages

```text
Double Infrastructure

Higher Cost

Operational Complexity
```

---

# VM-Based Blue-Green

Traditional Architecture

```text
Route53

↓

Load Balancer

↓

Target Group

↓

VMs
```

---

Current

```text
Target Group A

v4
```

---

Create

```text
Target Group B

v5
```

---

Validation

```text
Preview Load Balancer

Preview DNS
```

---

Switch

```text
Route53

↓

Target Group B
```

---

# Example

Current

```text
joindevops.com/api/v1
```

---

New Version

```text
joindevops.com/api/v2
```

---

Blue-Green Allows:

```text
Validation

Before Traffic Switch
```

---

# Rolling Update vs Blue-Green

| Feature | Rolling Update | Blue-Green |
|----------|----------|----------|
| Downtime | No | No |
| Infrastructure Cost | Low | High |
| Rollback Speed | Slow | Fast |
| Mixed Versions | Yes | No |
| Complexity | Low | Medium |
| Risk | Medium | Low |
| Extra Capacity | Not Required | Required |

---

# Which Strategy Should You Choose?

## Rolling Update

Use When

```text
Stateless Applications

Backward Compatible APIs

Cost Sensitive Workloads
```

---

Examples

```text
Frontend

Backend APIs

Microservices
```

---

## Blue-Green

Use When

```text
Critical Applications

Banking

Payments

E-Commerce
```

---

Examples

```text
Checkout

Payments

Authentication
```

---

# Real Production Example

Rolling Update

```text
Frontend

10 Replicas

v4 → v5
```

---

Reason

```text
Low Risk
```

---

Blue-Green

```text
Payment Service

v4 → v5
```

---

Reason

```text
Instant Rollback Needed
```

---

# Common Production Problems

## Rolling Update Failure

Symptoms

```text
Half Pods

v4

Half Pods

v5
```

---

Cause

```text
Application Incompatibility
```

---

# Blue-Green Failure

Cause

```text
Traffic Switched Too Early
```

---

Solution

```text
Validate Thoroughly
```

---

# Troubleshooting Commands

Deployment Status

```bash
kubectl rollout status deployment nginx
```

---

Deployment History

```bash
kubectl rollout history deployment nginx
```

---

Rollback

```bash
kubectl rollout undo deployment nginx
```

---

Pods

```bash
kubectl get pods
```

---

Services

```bash
kubectl get svc
```

---

# Common Interview Questions

## What Is A Rolling Update?

### Short Answer

A deployment strategy where Kubernetes gradually replaces old pods with new pods.

### Detailed Explanation

Instead of replacing all pods at once, Kubernetes creates new pods, waits until they become healthy, and then removes old pods in batches.

### Production Example

```text
4 Replicas

↓

Create 2 New

↓

Delete 2 Old

↓

Create 2 New

↓

Delete Remaining Old
```

### Follow-Up Questions

- What are maxSurge and maxUnavailable?
- Why are both versions running together?
- How does rollback work?
- What are the risks?

---

## What Is Blue-Green Deployment?

### Short Answer

A deployment strategy that uses two identical environments and switches traffic between them.

### Detailed Explanation

Blue serves production traffic while Green hosts the new version. After validation, traffic is switched to Green.

### Production Example

```text
Blue v4

↓

Green v5

↓

Validate

↓

Switch Traffic
```

### Follow-Up Questions

- How is traffic switched?
- How does rollback work?
- Why is rollback faster?
- Why does Blue-Green cost more?

---

## Difference Between Rolling Update And Blue-Green?

### Short Answer

Rolling Update gradually replaces pods while Blue-Green switches traffic between two environments.

### Detailed Explanation

Rolling Updates run both versions simultaneously during deployment. Blue-Green keeps environments separate and switches traffic only after validation.

### Production Example

Rolling:

```text
v4 + v5
```

Blue-Green:

```text
Only v4

or

Only v5
```

receives production traffic.

### Follow-Up Questions

- Which strategy is safer?
- Which strategy is cheaper?
- Which has faster rollback?
- Which is Kubernetes default?

---

## Why Does Rolling Update Require Backward Compatibility?

### Short Answer

Because old and new versions run together during deployment.

### Detailed Explanation

Requests may be served by either version until rollout completes. Both versions must understand the same APIs and data structures.

### Production Example

```text
/api/v1

and

/api/v2
```

running simultaneously.

### Follow-Up Questions

- What problems occur without compatibility?
- How do teams handle database changes?
- What are breaking changes?
- Why is API versioning important?

---

## Which Strategy Have You Used In Production?

### Short Answer

Both.

### Detailed Explanation

Rolling Updates are commonly used for stateless microservices, while Blue-Green is preferred for critical services requiring rapid rollback.

### Production Example

```text
Frontend

Rolling Update
```

```text
Payment Service

Blue-Green
```

### Follow-Up Questions

- Why different strategies?
- Which was more complex?
- How was rollback handled?
- How was traffic switched?