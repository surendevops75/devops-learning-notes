# Horizontal Pod Autoscaler (HPA)

## Introduction

Applications experience varying traffic patterns.

---

Examples

```text
Morning Traffic

Weekend Traffic

Festival Traffic

Sale Events

Peak Business Hours
```

---

Running a fixed number of Pods may cause:

```text
Resource Wastage

Performance Issues

Application Downtime
```

---

Kubernetes provides:

```text
Horizontal Pod Autoscaler (HPA)
```

to automatically scale applications.

---

# What is Scaling?

Scaling means:

```text
Increasing

Or

Decreasing

Resources
```

based on demand.

---

There are two types:

```text
Vertical Scaling

Horizontal Scaling
```

---

# Vertical Scaling

Vertical scaling means:

```text
Increase Server Resources
```

---

Example

Before

```text
1 GB RAM

20 GB Disk
```

---

After

```text
10 GB RAM

200 GB Disk
```

---

Resources are increased on:

```text
Same Server
```

---

# Advantages of Vertical Scaling

```text
Simple

Easy To Understand

Less Infrastructure
```

---

# Disadvantages of Vertical Scaling

```text
Downtime Required

Single Point Of Failure

Hardware Limitations
```

---

Example

```text
One Large Server
```

---

If server fails:

```text
Application Fails
```

---

# Horizontal Scaling

Horizontal scaling means:

```text
Increase Number Of Instances
```

---

Example

Before

```text
1 Pod
```

---

After

```text
5 Pods
```

---

Instead of increasing server size:

```text
Increase Replica Count
```

---

# Advantages of Horizontal Scaling

```text
No Downtime

Better Availability

Fault Tolerance

Cloud Friendly
```

---

Example

```text
Frontend-1

Frontend-2

Frontend-3
```

---

Traffic is distributed.

---

# Disadvantages of Horizontal Scaling

```text
More Infrastructure

Application Design Complexity
```

---

Applications should be:

```text
Stateless
```

for best results.

---

# What is HPA?

HPA stands for:

```text
Horizontal Pod Autoscaler
```

---

It automatically:

```text
Scale Out

Scale In
```

Pods based on resource utilization.

---

Common Metrics

```text
CPU

Memory

Custom Metrics
```

---

# HPA Architecture

```text
Application Pods
        │

        ▼

Metrics Server
        │

        ▼

Horizontal Pod Autoscaler
        │

        ▼

Deployment
```

---

HPA continuously monitors:

```text
Resource Usage
```

and adjusts replicas.

---

# How HPA Works

Example

```text
Target CPU = 70%
```

---

Current CPU

```text
90%
```

---

Result

```text
Add More Pods
```

---

Current CPU

```text
20%
```

---

Result

```text
Reduce Pods
```

---

# HPA Components

## Deployment

HPA works on:

```text
Deployment

StatefulSet
```

---

Most commonly:

```text
Deployment
```

---

## Metrics Server

Metrics Server provides:

```text
CPU Usage

Memory Usage
```

to Kubernetes.

---

Without Metrics Server:

```text
HPA Does Not Work
```

---

# Metrics Server

Metrics Server is an:

```text
EKS Add-On
```

---

Responsibilities

```text
Collect Metrics

Expose Metrics

Support Autoscaling
```

---

# Verify Metrics Server

Check

```bash
kubectl get deployment metrics-server -n kube-system
```

---

View Metrics

```bash
kubectl top nodes
```

---

View Pod Metrics

```bash
kubectl top pods
```

---

Example Output

```text
NAME          CPU   MEMORY

frontend-1    50m   120Mi

frontend-2    80m   130Mi
```

---

# Resource Requests and Limits

HPA requires:

```text
Resource Requests
```

---

Without requests:

```text
HPA Cannot Calculate Utilization
```

---

# Why Requests Matter?

Example

```text
CPU Request = 500m
```

---

Current Usage

```text
200m
```

---

Utilization

```text
200 / 500

40%
```

---

This percentage is used by HPA.

---

# Example Resource Configuration

```yaml
resources:

  requests:

    cpu: "500m"

    memory: "512Mi"

  limits:

    cpu: "1000m"

    memory: "1Gi"
```

---

# HPA Parameters

Common Parameters

```text
minReplicas

maxReplicas

targetCPUUtilizationPercentage
```

---

# minReplicas

Minimum number of Pods.

---

Example

```yaml
minReplicas: 2
```

---

Never scales below:

```text
2 Pods
```

---

# maxReplicas

Maximum number of Pods.

---

Example

```yaml
maxReplicas: 10
```

---

Never scales above:

```text
10 Pods
```

---

# Target CPU

Desired CPU utilization.

---

Example

```yaml
averageUtilization: 70
```

---

Meaning

```text
Maintain 70% CPU Usage
```

---

# HPA Example

```yaml
apiVersion: autoscaling/v2

kind: HorizontalPodAutoscaler

metadata:

  name: frontend-hpa

spec:

  scaleTargetRef:

    apiVersion: apps/v1

    kind: Deployment

    name: frontend

  minReplicas: 2

  maxReplicas: 10

  metrics:

  - type: Resource

    resource:

      name: cpu

      target:

        type: Utilization

        averageUtilization: 70
```

---

Create HPA

```bash
kubectl apply -f hpa.yaml
```

---

Verify

```bash
kubectl get hpa
```

---

Output

```text
NAME

frontend-hpa
```

---

Describe

```bash
kubectl describe hpa frontend-hpa
```

---

# Scaling Example

Initial State

```text
Frontend Pods = 2

CPU = 80%
```

---

Target

```text
70%
```

---

HPA Action

```text
Scale To 3 Pods
```

---

Traffic Distributed.

---

CPU Drops.

---

# Scale In Example

Initial State

```text
Frontend Pods = 5

CPU = 15%
```

---

Target

```text
70%
```

---

HPA Action

```text
Scale Down
```

---

Result

```text
2 Pods
```

---

# Production Example

Roboshop Frontend

Deployment

```text
Frontend
```

---

Traffic suddenly increases.

---

CPU rises to:

```text
85%
```

---

HPA detects:

```text
High CPU Usage
```

---

HPA scales

```text
2 Pods

To

6 Pods
```

---

Traffic handled automatically.

---

# HPA Workflow

```text
User Traffic
      ↓

CPU Increases
      ↓

Metrics Server
      ↓

HPA Detects
      ↓

Deployment Scaled
      ↓

New Pods Created
```

---

# HPA vs ReplicaSet

ReplicaSet

```text
Fixed Replica Count
```

---

Example

```text
3 Pods Always
```

---

HPA

```text
Dynamic Replica Count
```

---

Example

```text
2 To 10 Pods
```

based on demand.

---

# Common Problems

## HPA Not Scaling

Cause

```text
Metrics Server Missing
```

---

Check

```bash
kubectl top pods
```

---

If error appears:

```text
Install Metrics Server
```

---

## CPU Shows Unknown

Cause

```text
Resource Requests Missing
```

---

Verify

```yaml
resources:
  requests:
```

---

## HPA Stuck

Cause

```text
Wrong Metrics

Incorrect Configuration
```

---

Check

```bash
kubectl describe hpa
```

---

# Troubleshooting

Check HPA

```bash
kubectl get hpa
```

---

Describe

```bash
kubectl describe hpa
```

---

Check Metrics

```bash
kubectl top nodes
```

---

Check Pod Metrics

```bash
kubectl top pods
```

---

Verify Requests

```bash
kubectl describe deployment frontend
```

---

# Common Interview Questions

## What is HPA?

### Short Answer

HPA automatically scales Pods based on resource utilization.

### Detailed Explanation

Horizontal Pod Autoscaler monitors metrics such as CPU and Memory and adjusts replica counts automatically.

### Production Example

Frontend Deployment scales from 2 Pods to 6 Pods when traffic increases.

### Follow-Up Questions

- Which metrics can HPA use?
- Does HPA require Metrics Server?
- Can HPA scale StatefulSets?

---

## Difference Between Horizontal and Vertical Scaling?

### Short Answer

Horizontal scaling adds more instances, while vertical scaling increases resources on the same instance.

### Detailed Explanation

Horizontal scaling improves availability and fault tolerance, whereas vertical scaling increases server capacity but can introduce downtime.

### Production Example

Adding more frontend Pods is horizontal scaling. Increasing EC2 size from 2 vCPU to 8 vCPU is vertical scaling.

### Follow-Up Questions

- Which scaling method causes downtime?
- Which scaling method is cloud-native?
- Which scaling method avoids single points of failure?

---

## Why Is Metrics Server Required?

### Short Answer

Metrics Server provides CPU and Memory metrics used by HPA.

### Detailed Explanation

HPA relies on Metrics Server to determine whether Pods should scale up or down.

### Production Example

Without Metrics Server, kubectl top pods fails and HPA cannot make scaling decisions.

### Follow-Up Questions

- How do you verify Metrics Server?
- What command shows pod metrics?
- Is Metrics Server installed by default?

---

## Why Are Resource Requests Mandatory For HPA?

### Short Answer

HPA calculates utilization percentages using resource requests.

### Detailed Explanation

Without requests, Kubernetes cannot determine the percentage utilization needed for scaling decisions.

### Production Example

CPU Request = 500m and Current Usage = 200m results in 40% utilization.

### Follow-Up Questions

- What happens if requests are missing?
- Does HPA use limits?
- How is CPU utilization calculated?

---

## What are minReplicas and maxReplicas?

### Short Answer

They define the minimum and maximum number of Pods HPA can maintain.

### Detailed Explanation

HPA scales within the configured range and never exceeds those limits.

### Production Example

minReplicas=2 and maxReplicas=10 allows scaling between 2 and 10 Pods.

### Follow-Up Questions

- Can HPA scale to zero?
- What happens if traffic exceeds maxReplicas?
- How should maxReplicas be chosen?

---

# Key Takeaways

```text
HPA stands for Horizontal Pod Autoscaler.

HPA automatically scales Pods.

Horizontal scaling adds instances.

Vertical scaling increases server resources.

Metrics Server is mandatory for HPA.

Resource requests are required for utilization calculations.

HPA commonly scales Deployments.

minReplicas defines the minimum Pod count.

maxReplicas defines the maximum Pod count.

HPA is one of the most important Kubernetes production features.
```