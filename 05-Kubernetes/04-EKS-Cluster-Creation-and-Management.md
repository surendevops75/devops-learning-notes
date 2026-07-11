# EKS Cluster Creation and Management

## Introduction

Before deploying applications to Kubernetes, we need a cluster.

In AWS, Kubernetes is provided as:

```text
Amazon EKS
(Elastic Kubernetes Service)
```

---

Amazon EKS provides:

```text
Managed Control Plane

High Availability

Security

Scalability

AWS Integration
```

---

Instead of creating resources manually, we commonly use:

```text
eksctl
```

---

# What is eksctl?

eksctl is the official CLI tool for Amazon EKS.

---

Purpose

```text
Create EKS Clusters

Manage Node Groups

Upgrade Clusters

Delete Clusters
```

---

Think of eksctl as:

```text
Terraform For EKS
```

specialized for Kubernetes cluster provisioning.

---

# Why Use eksctl?

Without eksctl:

```text
Create VPC

Create IAM Roles

Create EKS Cluster

Create Node Groups

Configure Networking

Configure Security
```

must be done manually.

---

With eksctl:

```text
Single Configuration File
```

can create everything.

---

Benefits

```text
Fast Setup

Repeatable

Version Controlled

Easy Management
```

---

# EKS Architecture

```text
AWS Account
      │
      ▼

EKS Cluster
      │
      ├── Control Plane
      │
      └── Worker Nodes
```

---

Control Plane

```text
Managed By AWS
```

---

Worker Nodes

```text
Managed By Customer
```

---

# Cluster Configuration File

Most production environments use:

```text
eks.yaml
```

---

Instead of long commands.

---

Benefits

```text
Infrastructure As Code

Version Control

Repeatability

Automation
```

---

# Sample eks.yaml

```yaml
apiVersion: eksctl.io/v1alpha5

kind: ClusterConfig

metadata:
  name: roboshop
  region: us-east-1

managedNodeGroups:
- name: workers

  instanceType: t3.medium

  desiredCapacity: 2
```

---

Main Sections

```text
apiVersion

kind

metadata

managedNodeGroups
```

---

# Cluster Metadata

Example

```yaml
metadata:
  name: roboshop
  region: us-east-1
```

---

Purpose

```text
Cluster Name

AWS Region
```

---

# Managed Node Groups

Example

```yaml
managedNodeGroups:
- name: workers

  instanceType: t3.medium

  desiredCapacity: 2
```

---

Defines

```text
Worker Nodes

Instance Type

Node Count
```

---

# Creating EKS Cluster

Command

```bash
eksctl create cluster \
--config-file=eks.yaml
```

---

Flow

```text
eksctl
     ↓

AWS APIs
     ↓

Create VPC
     ↓

Create EKS Control Plane
     ↓

Create Node Groups
     ↓

Configure Networking
     ↓

Cluster Ready
```

---

Cluster creation typically takes:

```text
10-20 Minutes
```

depending on region and resources.

---

# Verify Cluster Creation

Check Nodes

```bash
kubectl get nodes
```

---

Example Output

```text
ip-10-0-1-10

ip-10-0-2-15
```

---

Verify Cluster

```bash
kubectl cluster-info
```

---

Output

```text
Kubernetes Control Plane
```

information.

---

# View Cluster Details

List Clusters

```bash
eksctl get cluster
```

---

Output

```text
roboshop
```

---

View Node Groups

```bash
eksctl get nodegroup \
--cluster roboshop
```

---

# Kubernetes Access Configuration

After cluster creation:

```text
kubectl
```

must know how to connect.

---

Update kubeconfig

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name roboshop
```

---

Verify

```bash
kubectl get nodes
```

---

# Cluster Deletion

When cluster is no longer required:

```bash
eksctl delete cluster \
--config-file=eks.yaml
```

---

Flow

```text
Delete Node Groups
      ↓

Delete EKS Cluster
      ↓

Delete Related Resources
```

---

Benefits

```text
Cost Savings

Environment Cleanup

Lab Reset
```

---

# Why Delete Unused Clusters?

EKS resources cost money.

---

Resources include:

```text
Control Plane

EC2 Instances

EBS Volumes

Load Balancers
```

---

Unused clusters increase:

```text
AWS Costs
```

---

# Common EKS Resources Created

During cluster creation:

```text
VPC

Subnets

Security Groups

IAM Roles

EKS Control Plane

Worker Nodes
```

---

Understanding this helps troubleshooting.

---

# Production Example

Roboshop Environment

```text
EKS Cluster
```

contains:

```text
Frontend

Catalogue

User

Cart

Shipping

Payment
```

---

Worker Nodes

```text
t3.medium
```

---

Managed using:

```text
eksctl
```

---

# Common Cluster Verification Commands

Check Nodes

```bash
kubectl get nodes
```

---

Check Namespaces

```bash
kubectl get namespaces
```

---

Check Pods

```bash
kubectl get pods -A
```

---

Check Cluster Info

```bash
kubectl cluster-info
```

---

# Common Problems

## kubectl Cannot Connect

Error

```text
Unable To Connect To Server
```

---

Check

```bash
aws eks update-kubeconfig
```

---

Verify AWS credentials.

---

## No Nodes Available

Command

```bash
kubectl get nodes
```

---

Output

```text
No Resources Found
```

---

Possible Causes

```text
Node Group Creation Failed

IAM Issues

AWS Limits
```

---

## Cluster Creation Failed

Check

```bash
eksctl utils describe-stacks \
--cluster roboshop
```

---

Verify

```text
CloudFormation Events
```

---

# Production Best Practices

## Use Configuration Files

Good

```bash
eksctl create cluster \
--config-file=eks.yaml
```

---

Avoid long manual commands.

---

## Store YAML In Git

Benefits

```text
Version Control

Auditing

Reusability
```

---

## Use Managed Node Groups

Benefits

```text
Simpler Upgrades

Reduced Maintenance

AWS Managed
```

---

## Delete Unused Clusters

Benefits

```text
Cost Optimization
```

---

# EKS Cluster Lifecycle

```text
Create eks.yaml
        ↓

eksctl create cluster
        ↓

Configure kubectl
        ↓

Deploy Applications
        ↓

Manage Cluster
        ↓

Delete Cluster
```

---

# Common Interview Questions

## What is eksctl?

### Short Answer

eksctl is the official CLI tool used to create and manage EKS clusters.

### Detailed Explanation

It automates creation of EKS control planes, node groups, networking, and IAM resources.

### Production Example

Creating a Roboshop EKS cluster using a configuration file.

### Follow-Up Questions

- Why use eksctl?
- How does it differ from Terraform?

---

## How Do You Create an EKS Cluster?

### Short Answer

Using a ClusterConfig YAML file and eksctl.

### Example

```bash
eksctl create cluster \
--config-file=eks.yaml
```

### Follow-Up Questions

- What is inside eks.yaml?
- How long does cluster creation take?

---

## How Do You Delete an EKS Cluster?

### Short Answer

Using eksctl delete cluster.

### Example

```bash
eksctl delete cluster \
--config-file=eks.yaml
```

### Follow-Up Questions

- What resources get deleted?
- Why should unused clusters be removed?

---

## How Do You Verify Cluster Creation?

### Short Answer

Using kubectl and eksctl commands.

### Commands

```bash
kubectl get nodes

kubectl cluster-info
```

### Follow-Up Questions

- How do you verify worker nodes?
- How do you update kubeconfig?

---

# Key Takeaways

```text
Amazon EKS is AWS's managed Kubernetes service.

eksctl is the most common tool for EKS cluster creation.

Cluster configuration is typically stored in eks.yaml.

eksctl create cluster provisions the cluster.

kubectl is used to verify and manage the cluster.

aws eks update-kubeconfig configures kubectl access.

Managed Node Groups simplify worker node management.

eksctl delete cluster removes the cluster and associated resources.

Using configuration files enables Infrastructure as Code.

Understanding cluster lifecycle is essential for EKS administration.
```