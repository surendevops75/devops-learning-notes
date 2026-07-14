# EKS ALB Target Types and Traffic Flow

## Introduction

When working with Ingress in EKS, one of the most important topics is understanding:

```text
How Traffic Reaches Pods
```

---

Many engineers know:

```text
Ingress

ALB

Service

Pod
```

---

But they do not understand:

```text
How ALB Reaches Pods

How Target Groups Work

How Security Groups Affect Traffic

Instance Mode vs IP Mode
```

---

These are very common production and interview topics.

---

# Complete Traffic Flow

User opens:

```text
https://catalogue.daws86s.fun
```

---

Traffic Flow

```text
User

↓

Route53

↓

Application Load Balancer

↓

Listener

↓

Rule

↓

Target Group

↓

Service

↓

Pod
```

---

Question

```text
How Does ALB Know

Where Pods Are?
```

---

Answer

```text
Target Groups
```

---

# What is a Target Group?

Target Group is an AWS resource.

---

Purpose

```text
Maintain Backend Targets

Receive Traffic From ALB
```

---

Target Group contains:

```text
EC2 Instances

OR

Pod IPs
```

---

ALB sends traffic to:

```text
Target Group
```

---

Target Group sends traffic to:

```text
Backend Targets
```

---

# ALB Architecture

```text
User
 │

 ▼

ALB
 │

 ▼

Listener
 │

 ▼

Rule
 │

 ▼

Target Group
 │

 ▼

Application
```

---

# Listener

Listener waits for requests.

---

Examples

```text
80

443
```

---

# Rules

Rules determine:

```text
Where Traffic Goes
```

---

Examples

```text
Host

Path
```

---

# Target Group

Target Group determines:

```text
Which Backend Receives Traffic
```

---

# Target Types

AWS supports:

```text
Instance

IP
```

---

Very Important Interview Topic.

---

# Instance Target Type

Target Group contains:

```text
Worker Nodes
```

---

Example

```text
TG

↓

Node-1

Node-2

Node-3
```

---

Traffic Flow

```text
User

↓

ALB

↓

Target Group

↓

Worker Node

↓

Service

↓

Pod
```

---

Example

```text
User

↓

ALB

↓

Node-1

↓

Kube Proxy

↓

Catalogue Pod
```

---

# Instance Mode Architecture

```text
User
 │

 ▼

ALB
 │

 ▼

Target Group
 │

 ▼

Worker Node
 │

 ▼

Service
 │

 ▼

Pod
```

---

# How Does It Work?

Controller registers:

```text
Worker Nodes
```

inside Target Group.

---

ALB forwards traffic to:

```text
Worker Nodes
```

---

Kubernetes forwards traffic internally.

---

# Advantages

```text
Simple

Easy To Understand
```

---

# Disadvantages

Extra Hop

```text
ALB

↓

Node

↓

Pod
```

---

Additional latency.

---

# IP Target Type

Target Group contains:

```text
Pod IPs
```

---

Example

```text
TG

↓

10.0.1.10

10.0.2.15

10.0.3.20
```

---

Traffic Flow

```text
User

↓

ALB

↓

Target Group

↓

Pod
```

---

Direct Routing.

---

# IP Mode Architecture

```text
User
 │

 ▼

ALB
 │

 ▼

Target Group
 │

 ▼

Pod
```

---

No Node Hop.

---

# How Does It Work?

AWS Load Balancer Controller registers:

```text
Pod IPs
```

directly inside:

```text
Target Group
```

---

ALB forwards traffic directly to Pods.

---

# Advantages

```text
Less Latency

Better Performance

Direct Routing
```

---

# Why EKS Prefers IP Mode?

Because:

```text
Pods Are Real Targets
```

---

Traffic directly reaches:

```text
Application Pods
```

---

Most modern EKS deployments use:

```text
IP Target Type
```

---

# Instance vs IP Mode

| Feature | Instance | IP |
|----------|----------|----------|
| Target Group Contains | Nodes | Pod IPs |
| Extra Hop | Yes | No |
| Performance | Good | Better |
| Traffic Path | ALB → Node → Pod | ALB → Pod |
| Modern EKS | Less Common | Preferred |

---

# Security Group Flow

Question

```text
How Does ALB Reach Nodes?
```

---

Answer

```text
Security Groups
```

---

# Security Group Architecture

```text
ALB SG

↓

Node SG

↓

Pod
```

---

# Example

ALB Security Group

```text
Allows

80

443
```

---

Node Security Group

```text
Allows Traffic

From ALB SG
```

---

Result

```text
Traffic Flows Successfully
```

---

# VPC CIDR Example

Suppose

```text
VPC

10.0.0.0/16
```

---

Node Security Group allows:

```text
10.0.0.0/16
```

---

Traffic from ALB reaches:

```text
Worker Nodes
```

---

Successfully.

---

# Common Production Issue

ALB Healthy

```text
✓
```

---

Target Group Healthy

```text
✓
```

---

Application Not Reachable

```text
✗
```

---

Cause

```text
Security Group Issue
```

---

# Security Group Troubleshooting

Verify

```text
ALB Security Group
```

---

Verify

```text
Node Security Group
```

---

Verify

```text
Inbound Rules
```

---

# Production Example

Catalogue Service

Domain

```text
catalogue.daws86s.fun
```

---

Flow

```text
User

↓

ALB

↓

Listener

↓

Host Rule

↓

Catalogue TG

↓

Catalogue Pod
```

---

# Cart Example

```text
cart.daws86s.fun
```

---

Flow

```text
User

↓

ALB

↓

Listener

↓

Cart Rule

↓

Cart TG

↓

Cart Pod
```

---

# Banking Example

```text
netbanking.icici.com
```

---

Flow

```text
ALB

↓

Listener

↓

Rule

↓

NetBanking TG

↓

Pods
```

---

```text
corporate.icici.com
```

---

Flow

```text
ALB

↓

Listener

↓

Rule

↓

Corporate TG

↓

Pods
```

---

# Common Misconception

## Can Ingress Create Ingress Controller?

Answer

```text
No
```

---

Ingress is:

```text
Configuration
```

---

Ingress Controller is:

```text
Software
```

---

Correct Flow

```text
Install AWS Load Balancer Controller

↓

Create Ingress

↓

Controller Reads Ingress

↓

Controller Creates ALB
```

---

# Common Misconception

## Is ALB an Ingress Controller?

Answer

```text
No
```

---

ALB is:

```text
Load Balancer
```

---

Ingress Controller is:

```text
AWS Load Balancer Controller
```

---

Correct Architecture

```text
Ingress Resource

↓

AWS Load Balancer Controller

↓

ALB

↓

Target Groups

↓

Pods
```

---

# Common Production Problems

## ALB Created

But

```text
No Traffic
```

---

Cause

```text
Security Group Blocked
```

---

## Target Group Empty

Cause

```text
Pods Not Registered
```

---

Verify

```bash
kubectl get pods
```

---

## Unhealthy Targets

Cause

```text
Health Check Failure
```

---

Verify

```text
/health Endpoint
```

---

## Wrong Target Type

Cause

```text
Instance Instead Of IP
```

---

Verify

```text
Target Group Settings
```

---

# Troubleshooting Commands

Check Ingress

```bash
kubectl get ingress
```

---

Describe Ingress

```bash
kubectl describe ingress ingress-name
```

---

Check Services

```bash
kubectl get svc
```

---

Check Endpoints

```bash
kubectl get endpoints
```

---

Check ALB

```bash
aws elbv2 describe-load-balancers
```

---

Check Target Groups

```bash
aws elbv2 describe-target-groups
```

---

Check Target Health

```bash
aws elbv2 describe-target-health \
--target-group-arn <arn>
```

---

# Common Interview Questions

## What is a Target Group?

### Short Answer

A Target Group is an AWS resource that contains backend targets and receives traffic from an ALB.

### Detailed Explanation

ALB forwards traffic to Target Groups, which then deliver traffic to worker nodes or pod IPs.

### Production Example

Catalogue Target Group forwards requests to Catalogue Pods.

### Follow-Up Questions

- Can Target Groups contain Pods?
- Can Target Groups contain EC2 instances?
- Who creates Target Groups?

---

## What is Instance Target Type?

### Short Answer

The Target Group contains worker nodes.

### Detailed Explanation

ALB forwards traffic to worker nodes, and Kubernetes routes traffic to Pods.

### Production Example

ALB → Node → Catalogue Pod.

### Follow-Up Questions

- Why is there an extra hop?
- What are the disadvantages?
- Is it commonly used today?

---

## What is IP Target Type?

### Short Answer

The Target Group contains Pod IPs.

### Detailed Explanation

ALB directly forwards traffic to Pods without passing through worker nodes.

### Production Example

ALB → Catalogue Pod.

### Follow-Up Questions

- Why is IP mode preferred?
- How are Pod IPs registered?
- Does Kubernetes Service still exist?

---

## Is ALB an Ingress Controller?

### Short Answer

No.

### Detailed Explanation

AWS Load Balancer Controller is the Ingress Controller. ALB is the load balancer created by the controller.

### Production Example

Ingress → AWS Load Balancer Controller → ALB.

### Follow-Up Questions

- What creates the ALB?
- What watches Ingress resources?
- Can ALB exist without Ingress?

---

## Can Ingress Create an Ingress Controller?

### Short Answer

No.

### Detailed Explanation

Ingress is only a Kubernetes resource. The controller must already be installed.

### Production Example

Creating an Ingress without AWS Load Balancer Controller will not create an ALB.

### Follow-Up Questions

- What is the role of an Ingress Controller?
- What happens if the controller is missing?
- How do you verify controller installation?

---

# Key Takeaways

```text
Target Groups are critical components of ALB architecture.

Target Groups can use Instance or IP target types.

Instance Mode:
ALB → Node → Pod

IP Mode:
ALB → Pod

Most EKS production environments prefer IP mode.

Security Groups control traffic flow between ALB and worker nodes.

Ingress does not create an Ingress Controller.

AWS Load Balancer Controller creates and manages ALBs.

ALB is not an Ingress Controller.

Understanding traffic flow is essential for production troubleshooting and interviews.
```