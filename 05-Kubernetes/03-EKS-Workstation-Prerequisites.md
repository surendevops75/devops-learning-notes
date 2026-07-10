# EKS Workstation Prerequisites

## Introduction

Before creating an EKS cluster, the workstation must be prepared properly.

A typical DevOps workstation should have:

```text
Sufficient Disk Space

Docker

AWS CLI

kubectl

eksctl

AWS Credentials
```

---

Without proper preparation, common issues include:

```text
Disk Full Errors

Docker Image Pull Failures

Cluster Creation Failures

Authentication Errors

Tool Compatibility Issues
```

---

# Why Workstation Preparation Matters?

When creating EKS clusters:

```text
Docker Stores Images

eksctl Creates Resources

kubectl Connects To Clusters

AWS CLI Authenticates Requests
```

---

All these tools consume:

```text
Storage

Memory

CPU
```

---

# Expanding /var Partition

## Why Expand /var?

Docker stores data in:

```text
/var/lib/docker
```

---

Contents:

```text
Docker Images

Containers

Volumes

Build Cache

Logs
```

---

Default space may not be sufficient.

---

Production Preparation

```bash
growpart /dev/nvme0n1 4
```

Expands disk partition.

---

Extend Logical Volume

```bash
lvextend -L +30G /dev/mapper/RootVG-varVol
```

---

Grow Filesystem

```bash
xfs_growfs /var
```

---

Result

```text
Additional Storage Available For Docker
```

---

# Verify Disk Space

Check Filesystem Usage

```bash
df -h
```

---

Example

```text
Filesystem      Size Used Avail Use%

/var             50G  10G   40G
```

---

# Docker Installation

Docker is required for:

```text
Container Operations

Image Building

Testing Applications

Container Runtime Knowledge
```

---

Install Docker Repository

```bash
dnf -y install dnf-plugins-core
```

---

Add Docker Repository

```bash
dnf config-manager \
--add-repo \
https://download.docker.com/linux/rhel/docker-ce.repo
```

---

Install Docker

```bash
dnf install docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin -y
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

Verify Docker

```bash
docker --version
```

---

Example

```text
Docker version 28.x.x
```

---

# Docker Permissions

By default:

```text
Only Root User
```

can execute Docker commands.

---

Add User To Docker Group

```bash
usermod -aG docker ec2-user
```

---

Logout and Login Again.

---

Verify

```bash
docker ps
```

---

# kubectl Installation

## What is kubectl?

kubectl is the Kubernetes command-line tool.

---

Used to:

```text
Create Resources

Manage Resources

Troubleshoot Clusters
```

---

Download kubectl

```bash
curl -O \
https://s3.us-west-2.amazonaws.com/amazon-eks/1.34.2/2025-11-13/bin/linux/amd64/kubectl
```

---

Make Executable

```bash
chmod +x ./kubectl
```

---

Create Binary Directory

```bash
mkdir -p $HOME/bin
```

---

Copy Binary

```bash
cp ./kubectl $HOME/bin/kubectl
```

---

Update PATH

```bash
export PATH=$HOME/bin:$PATH
```

---

Verify Installation

```bash
kubectl version --client
```

---

Expected Output

```text
Client Version: v1.34.x
```

---

# eksctl Installation

## What is eksctl?

eksctl is a CLI tool used to create and manage EKS clusters.

---

Benefits

```text
Automates Cluster Creation

Creates Node Groups

Creates IAM Roles

Creates Networking Components
```

---

Set Architecture Variable

```bash
ARCH=amd64
```

---

Set Platform Variable

```bash
PLATFORM=$(uname -s)_$ARCH
```

---

Download eksctl

```bash
curl -sLO \
"https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
```

---

Extract Archive

```bash
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp
```

---

Remove Archive

```bash
rm eksctl_$PLATFORM.tar.gz
```

---

Install Binary

```bash
sudo install -m 0755 \
/tmp/eksctl \
/usr/local/bin
```

---

Cleanup

```bash
rm /tmp/eksctl
```

---

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

# AWS CLI Verification

Verify AWS CLI

```bash
aws --version
```

---

Configure Credentials

```bash
aws configure
```

---

Verify Access

```bash
aws sts get-caller-identity
```

---

Successful Output

```json
{
  "Account": "123456789012"
}
```

---

# Tool Verification Checklist

Verify Docker

```bash
docker --version
```

---

Verify kubectl

```bash
kubectl version --client
```

---

Verify eksctl

```bash
eksctl version
```

---

Verify AWS CLI

```bash
aws --version
```

---

Verify AWS Access

```bash
aws sts get-caller-identity
```

---

# Common Setup Problems

## Docker Service Not Running

Error

```text
Cannot Connect To Docker Daemon
```

---

Fix

```bash
systemctl start docker
```

---

## kubectl Not Found

Error

```text
kubectl: command not found
```

---

Fix

```bash
export PATH=$HOME/bin:$PATH
```

---

## eksctl Not Found

Error

```text
eksctl: command not found
```

---

Verify

```bash
which eksctl
```

---

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

# EKS Workstation Preparation Flow

```text
Increase /var Storage
         ↓

Install Docker
         ↓

Install kubectl
         ↓

Install eksctl
         ↓

Configure AWS CLI
         ↓

Verify Tools
         ↓

Ready For EKS Cluster Creation
```

---

# Common Interview Questions

## Why Do We Expand /var Before Working With Docker?

### Short Answer

Docker stores images, containers, and volumes under /var/lib/docker.

### Detailed Explanation

Insufficient disk space can lead to image pull failures, build failures, and container creation issues.

### Production Example

Large container images filling up the Docker storage directory.

### Follow-Up Questions

- Where does Docker store images?
- How do you check disk usage?

---

## Why Is kubectl Required?

### Short Answer

kubectl is used to communicate with Kubernetes clusters.

### Detailed Explanation

It acts as a client that sends requests to the Kubernetes API Server.

### Production Example

Managing pods, deployments, and services in EKS.

### Follow-Up Questions

- What is kubeconfig?
- What is API Server?

---

## Why Is eksctl Popular?

### Short Answer

It simplifies EKS cluster creation.

### Detailed Explanation

eksctl automatically creates networking, IAM roles, node groups, and Kubernetes control plane resources.

### Production Example

Creating an EKS cluster with managed node groups.

### Follow-Up Questions

- What AWS resources does eksctl create?
- How does eksctl differ from Terraform?

---

# Key Takeaways

```text
A properly prepared workstation is required before creating EKS clusters.

Docker stores data under /var/lib/docker.

Expanding /var prevents storage-related failures.

kubectl is the Kubernetes command-line client.

eksctl simplifies EKS cluster creation and management.

AWS CLI provides authentication and AWS access.

All tools should be verified before creating clusters.

Proper workstation preparation reduces troubleshooting effort later.
```