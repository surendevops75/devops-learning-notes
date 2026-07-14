# AWS Load Balancer Controller Installation and IRSA

## Introduction

In previous chapters we learned:

```text
Ingress

Ingress Controller

AWS Load Balancer Controller

ALB Ingress
```

---

Now let's learn how to install:

```text
AWS Load Balancer Controller
```

in EKS.

---

This is one of the most common production tasks for DevOps Engineers.

---

# Why AWS Load Balancer Controller?

Suppose we create:

```yaml
kind: Ingress
```

---

Question

```text
Who Creates ALB?

Who Creates Target Groups?

Who Creates Listener Rules?
```

---

Answer

```text
AWS Load Balancer Controller
```

---

Without Controller

```text
Ingress Exists

ALB Does Not Exist
```

---

Traffic Will Not Work.

---

# Production Architecture

```text
Ingress Resource
        │

        ▼

AWS Load Balancer Controller
        │

        ▼

Service Account
        │

        ▼

IAM Role (IRSA)
        │

        ▼

AWS APIs
        │

        ▼

ALB

Listener

Rules

Target Groups
```

---

# Installation Prerequisites

Before installation ensure:

```text
EKS Cluster Exists

kubectl Configured

AWS CLI Installed

eksctl Installed

Helm Installed

OIDC Enabled
```

---

# Verify Cluster Access

```bash
kubectl get nodes
```

---

Expected Output

```text
Worker Nodes
```

---

# Verify AWS Access

```bash
aws sts get-caller-identity
```

---

Expected Output

```text
Account ID

User ARN
```

---

# Step 1 - Configure kubectl

Configure AWS CLI

```bash
aws configure
```

---

Provide

```text
Access Key

Secret Key

Region
```

---

Update kubeconfig

```bash
aws eks update-kubeconfig \
--region us-east-1 \
--name roboshop-dev
```

---

Verify

```bash
kubectl get nodes
```

---

# Step 2 - Verify OIDC Provider

AWS Load Balancer Controller uses:

```text
IRSA
```

---

IRSA requires:

```text
OIDC Provider
```

---

Check OIDC

```bash
aws eks describe-cluster \
--name roboshop-dev \
--query "cluster.identity.oidc.issuer"
```

---

Example Output

```text
https://oidc.eks.us-east-1.amazonaws.com/id/ABC123
```

---

# What Is OIDC?

OIDC stands for:

```text
OpenID Connect
```

---

Purpose

```text
Trust Relationship

Between

EKS

And

IAM
```

---

Without OIDC

```text
IRSA Cannot Work
```

---

# Step 3 - Associate OIDC Provider

Command

```bash
eksctl utils associate-iam-oidc-provider \
    --region us-east-1 \
    --cluster roboshop-dev \
    --approve
```

---

What Happens?

```text
OIDC Provider Created

IAM Trust Established
```

---

Verify

```bash
aws iam list-open-id-connect-providers
```

---

Expected

```text
OIDC Provider ARN
```

---

# Step 4 - Download IAM Policy

AWS provides official policy.

Documentation

```text
https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/
```

---

Purpose

```text
Allow Controller

To Create

ALBs

Target Groups

Listeners

Rules
```

---

# Why IAM Policy Required?

Controller calls:

```text
AWS APIs
```

---

Examples

```text
CreateLoadBalancer

CreateTargetGroup

RegisterTargets

ModifyListener
```

---

Without permissions

```text
Access Denied
```

---

# Step 5 - Create IAM Policy

Example

```bash
aws iam create-policy \
--policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy.json
```

---

Verify

```bash
aws iam list-policies
```

---

Expected

```text
AWSLoadBalancerControllerIAMPolicy
```

---

# Step 6 - Create Service Account

Important Concept

---

Controller runs inside:

```text
Pod
```

---

Pod needs AWS access.

---

Question

```text
How Does Pod Access AWS?
```

---

Answer

```text
IRSA
```

---

# Create IAM Service Account

```bash
eksctl create iamserviceaccount \
  --cluster roboshop-dev \
  --namespace kube-system \
  --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::<ACCOUNT-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

---

What Gets Created?

```text
Service Account

IAM Role

Trust Policy

Policy Attachment
```

---

# Verify Service Account

```bash
kubectl get sa \
-n kube-system
```

---

Expected

```text
aws-load-balancer-controller
```

---

# Architecture After IRSA

```text
Controller Pod
        │

        ▼

Service Account
        │

        ▼

IAM Role
        │

        ▼

AWS APIs
```

---

# Why Not Access Keys?

Old Method

```text
Access Key

Secret Key
```

inside Pod.

---

Problems

```text
Credential Leakage

Rotation Problems

Security Risks
```

---

Modern Method

```text
IRSA
```

---

# Step 7 - Add Helm Repository

Add Repository

```bash
helm repo add eks https://aws.github.io/eks-charts
```

---

Update

```bash
helm repo update
```

---

Verify

```bash
helm repo list
```

---

# Step 8 - Install Controller

Install

```bash
helm install aws-load-balancer-controller \
eks/aws-load-balancer-controller \
-n kube-system \
--set clusterName=roboshop-dev \
--set serviceAccount.create=false \
--set serviceAccount.name=aws-load-balancer-controller
```

---

Important

```text
Use Existing Service Account
```

---

Because IRSA already created it.

---

# Verify Installation

Check Pods

```bash
kubectl get pods \
-n kube-system
```

---

Expected

```text
aws-load-balancer-controller
```

---

Check Deployment

```bash
kubectl get deployment \
-n kube-system
```

---

Expected

```text
aws-load-balancer-controller
```

---

# Verify Logs

```bash
kubectl logs deployment/aws-load-balancer-controller \
-n kube-system
```

---

Healthy Logs

```text
Controller Started

Leader Elected

Watching Resources
```

---

# What Resources Are Watched?

Controller watches:

```text
Ingress

Service

TargetGroupBinding
```

---

Whenever changes occur:

```text
Reconciliation Starts
```

---

# Reconciliation Example

Desired State

```text
Ingress Created
```

---

Actual State

```text
No ALB Exists
```

---

Controller Action

```text
Create ALB
```

---

Result

```text
Desired State

=

Actual State
```

---

# Verify Controller Is Working

Create Ingress

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: test-ingress

  annotations:

    kubernetes.io/ingress.class: alb

spec:
  rules:
```

---

Apply

```bash
kubectl apply -f ingress.yaml
```

---

Check

```bash
kubectl get ingress
```

---

Expected

```text
ALB DNS Name
```

appears.

---

# Production Flow

Developer Creates

```text
Ingress
```

↓

Controller Detects

```text
Ingress Resource
```

↓

Controller Calls

```text
AWS APIs
```

↓

Creates

```text
ALB
```

↓

Creates

```text
Listener
```

↓

Creates

```text
Target Group
```

↓

Registers

```text
Pods
```

↓

Traffic Starts Flowing

---

# Common Problems

## ALB Not Created

Cause

```text
Controller Not Installed
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

# Access Denied

Cause

```text
IAM Policy Missing
```

---

Controller cannot call AWS APIs.

---

# OIDC Missing

Cause

```text
OIDC Provider Not Created
```

---

Verify

```bash
aws iam list-open-id-connect-providers
```

---

# Service Account Missing

Cause

```text
IRSA Not Configured
```

---

Verify

```bash
kubectl get sa -n kube-system
```

---

# Helm Installation Failure

Cause

```text
Wrong Cluster Name

Wrong Namespace

Repository Missing
```

---

Verify

```bash
helm list -A
```

---

# Ingress Not Creating ALB

Cause

```text
Missing Annotation
```

---

Required

```yaml
kubernetes.io/ingress.class: alb
```

---

# Troubleshooting Commands

Verify OIDC

```bash
aws iam list-open-id-connect-providers
```

---

Verify Service Account

```bash
kubectl get sa \
-n kube-system
```

---

Describe Service Account

```bash
kubectl describe sa aws-load-balancer-controller \
-n kube-system
```

---

Check Deployment

```bash
kubectl get deployment \
-n kube-system
```

---

Check Controller Logs

```bash
kubectl logs deployment/aws-load-balancer-controller \
-n kube-system
```

---

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

# Production Interview Questions

## How Do You Install AWS Load Balancer Controller?

### Short Answer

Enable OIDC, create IAM policy, create IRSA-based Service Account, and install the controller using Helm.

### Detailed Explanation

The controller requires AWS API access, which is provided through IRSA. After creating the IAM role and Service Account, Helm is used to deploy the controller.

### Production Example

We installed AWS Load Balancer Controller in EKS to expose Roboshop microservices through a shared ALB.

### Follow-Up Questions

- Why is OIDC required?
- Why is IRSA used?
- Which permissions are required?

---

## Why Is OIDC Required?

### Short Answer

OIDC allows IAM to trust Kubernetes Service Accounts.

### Detailed Explanation

IRSA depends on a trust relationship between EKS and IAM. OIDC establishes this trust.

### Production Example

The controller Service Account assumes an IAM Role using OIDC.

### Follow-Up Questions

- How do you verify OIDC?
- What happens if OIDC is missing?
- Can IRSA work without OIDC?

---

## Why Does AWS Load Balancer Controller Need IAM Permissions?

### Short Answer

It creates AWS resources.

### Detailed Explanation

The controller calls AWS APIs to create and manage ALBs, listeners, rules, target groups, and security groups.

### Production Example

Creating an Ingress automatically created an ALB using controller IAM permissions.

### Follow-Up Questions

- Which APIs are called?
- What happens if permissions are missing?
- How do you troubleshoot Access Denied errors?

---

## How Does AWS Load Balancer Controller Authenticate To AWS?

### Short Answer

Using IRSA.

### Detailed Explanation

The controller runs with a Service Account mapped to an IAM Role. AWS validates the OIDC token and issues temporary credentials.

### Production Example

The controller Pod assumed an IAM Role and created ALBs automatically.

### Follow-Up Questions

- Why not Access Keys?
- What role does OIDC play?
- What temporary credentials are generated?

---

# Key Takeaways

```text
AWS Load Balancer Controller is required for ALB-based ingress in EKS.

OIDC is mandatory for IRSA.

IRSA is used instead of Access Keys.

The controller needs IAM permissions to create AWS resources.

Helm is the recommended installation method.

The controller creates ALBs, listeners, target groups, and rules automatically.

Ingress resources alone cannot create ALBs.

Controller Pod → Service Account → IAM Role → AWS APIs is a critical production flow.

Understanding controller installation and troubleshooting is a common interview topic.
```