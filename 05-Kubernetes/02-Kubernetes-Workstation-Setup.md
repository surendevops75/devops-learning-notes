# Kubernetes Workstation Setup

## Introduction

Before creating or managing Kubernetes clusters, a workstation must be prepared with the required tools.

A Kubernetes administrator or DevOps engineer typically uses:

```text
Docker

AWS CLI

kubectl

eksctl
```

---

These tools allow us to:

```text
Create Clusters

Manage Clusters

Deploy Applications

Troubleshoot Workloads

Interact With AWS
```

---

# Workstation Architecture

```text
Developer Laptop
        │
        ├── Docker
        ├── AWS CLI
        ├── kubectl
        └── eksctl
                │
                ▼

             AWS EKS
                │
                ▼

         Kubernetes Cluster
```

---

# Why Do We Need Docker?

Docker is required because:

```text
eksctl Uses Containers

Build Container Images

Test Applications Locally

Container Runtime Knowledge
```

---

Verify Docker Installation

```bash
docker --version
```

---

Example Output

```text
Docker version 28.x.x
```

---

Verify Docker Service

```bash
systemctl status docker
```

---

Start Docker

```bash
systemctl start docker
```

---

Enable Docker

```bash
systemctl enable docker
```

---

# AWS CLI

## What is AWS CLI?

AWS CLI is a command-line tool used to interact with AWS services.

---

Examples

```bash
aws ec2 describe-instances

aws s3 ls

aws eks list-clusters
```

---

Without AWS CLI:

```text
Cannot Authenticate

Cannot Manage AWS Resources

Cannot Access EKS
```

---

# Install AWS CLI

Verify Installation

```bash
aws --version
```

---

Example Output

```text
aws-cli/2.x.x
```

---

# Configure AWS CLI

Command

```bash
aws configure
```

---

Provide

```text
AWS Access Key

AWS Secret Key

Region

Output Format
```

---

Example

```text
AWS Access Key ID:
****************

AWS Secret Access Key:
****************

Default region:
us-east-1

Default output:
json
```

---

# Verify AWS Access

Check Identity

```bash
aws sts get-caller-identity
```

---

Example Output

```json
{
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/admin"
}
```

---

This confirms:

```text
AWS CLI Working

Authentication Successful
```

---

# kubectl

## What is kubectl?

kubectl is the official Kubernetes command-line tool.

---

Used to:

```text
Create Resources

View Resources

Update Resources

Delete Resources

Troubleshoot Resources
```

---

Think of kubectl as:

```text
Client For Kubernetes API Server
```

---

# Common kubectl Commands

View Nodes

```bash
kubectl get nodes
```

---

View Pods

```bash
kubectl get pods
```

---

View Deployments

```bash
kubectl get deployments
```

---

Describe Pod

```bash
kubectl describe pod pod-name
```

---

View Logs

```bash
kubectl logs pod-name
```

---

# Install kubectl

Verify Installation

```bash
kubectl version --client
```

---

Example Output

```text
Client Version: v1.33.x
```

---

# kubectl Architecture

```text
kubectl
     │
     ▼

API Server
     │
     ▼

Kubernetes Cluster
```

---

Whenever you run:

```bash
kubectl get pods
```

kubectl sends request to:

```text
API Server
```

---

API Server returns response.

---

# eksctl

## What is eksctl?

eksctl is an official CLI tool used to create and manage EKS clusters.

---

Without eksctl:

```text
Many AWS Resources
Must Be Created Manually
```

---

Examples

```text
VPC

Subnets

Node Groups

Security Groups

IAM Roles
```

---

eksctl automates these tasks.

---

# Install eksctl

Verify Installation

```bash
eksctl version
```

---

Example Output

```text
0.2xx.x
```

---

# Create EKS Cluster

Example

```bash
eksctl create cluster \
--name roboshop \
--region us-east-1 \
--nodegroup-name workers \
--nodes 2
```

---

eksctl automatically creates:

```text
EKS Cluster

Node Group

Security Groups

IAM Roles

Networking Components
```

---

# List EKS Clusters

```bash
eksctl get cluster
```

---

Example Output

```text
roboshop
```

---

# Delete Cluster

```bash
eksctl delete cluster \
--name roboshop
```

---

Useful after practice sessions.

---

# Connecting kubectl to EKS

After cluster creation:

```bash
aws eks update-kubeconfig \
--name roboshop \
--region us-east-1
```

---

This updates:

```text
~/.kube/config
```

---

Verify

```bash
kubectl get nodes
```

---

Example Output

```text
ip-10-0-1-10

ip-10-0-2-20
```

---

Cluster connection successful.

---

# Kubernetes Configuration File

Location

```text
~/.kube/config
```

---

Contains

```text
Cluster Details

Authentication Information

Contexts
```

---

View Contexts

```bash
kubectl config get-contexts
```

---

Current Context

```bash
kubectl config current-context
```

---

# Common Verification Commands

Docker

```bash
docker --version
```

---

AWS CLI

```bash
aws --version
```

---

AWS Identity

```bash
aws sts get-caller-identity
```

---

kubectl

```bash
kubectl version --client
```

---

eksctl

```bash
eksctl version
```

---

Nodes

```bash
kubectl get nodes
```

---

# Troubleshooting

## AWS Authentication Failure

Error

```text
Unable To Locate Credentials
```

---

Fix

```bash
aws configure
```

---

Verify

```bash
aws sts get-caller-identity
```

---

## kubectl Not Connected

Error

```text
The connection to the server was refused
```

---

Fix

```bash
aws eks update-kubeconfig
```

---

Verify

```bash
kubectl get nodes
```

---

## eksctl Command Not Found

Error

```text
eksctl: command not found
```

---

Cause

```text
eksctl Not Installed

Path Not Configured
```

---

# Production Workflow

```text
Developer
      ↓

AWS CLI Authentication
      ↓

eksctl Create Cluster
      ↓

kubectl Connect
      ↓

Deploy Applications
      ↓

Manage Kubernetes Resources
```

---

# Common Interview Questions

## What is kubectl?

### Short Answer

kubectl is the command-line tool used to interact with Kubernetes clusters.

### Detailed Explanation

kubectl communicates with the Kubernetes API Server to manage cluster resources.

### Production Example

Viewing pods, deployments, and services in EKS.

### Follow-Up Questions

- What is the API Server?
- Where is kubeconfig stored?

---

## What is eksctl?

### Short Answer

eksctl is a CLI tool used to create and manage Amazon EKS clusters.

### Detailed Explanation

It automates creation of networking, IAM roles, node groups, and cluster resources.

### Production Example

Creating EKS clusters for microservices deployments.

### Follow-Up Questions

- What resources does eksctl create?
- How does it integrate with AWS?

---

## Why Do We Need AWS CLI?

### Short Answer

AWS CLI is required to authenticate and interact with AWS services.

### Detailed Explanation

It provides command-line access to EKS, EC2, IAM, S3, and other AWS services.

### Production Example

Updating kubeconfig for EKS access.

### Follow-Up Questions

- What is aws configure?
- How do you verify AWS credentials?

---

# Key Takeaways

```text
A Kubernetes workstation requires Docker, AWS CLI, kubectl, and eksctl.

AWS CLI provides access to AWS services.

kubectl is used to communicate with Kubernetes clusters.

eksctl simplifies EKS cluster creation and management.

AWS credentials are configured using aws configure.

kubectl uses ~/.kube/config to connect to clusters.

aws eks update-kubeconfig connects kubectl to EKS.

Verifying tools before cluster creation prevents setup issues.

These tools form the foundation of Kubernetes administration on AWS.
```