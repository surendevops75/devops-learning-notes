# Terraform Providers and Provider Development

## Introduction

Terraform itself does not know how to create infrastructure.

For example:

```text
EC2

S3

VPC

Azure VM

GCP Compute Engine

Kubernetes Pods
```

---

Terraform needs a way to communicate with:

```text
AWS

Azure

Google Cloud

Kubernetes

VMware
```

---

This communication happens through:

```text
Providers
```

---

# What is a Provider?

A Provider is:

```text
A Plugin That Allows Terraform To Interact With A Platform
```

---

Provider acts as:

```text
Translator
```

between:

```text
Terraform

and

Cloud Platform APIs
```

---

# Architecture

```text
Terraform Code
        ↓
Provider
        ↓
Cloud API
        ↓
Infrastructure
```

---

# Example

Terraform Code:

```hcl
resource "aws_instance" "catalogue" {

}
```

---

Terraform itself cannot create EC2.

---

AWS Provider:

```text
Calls AWS APIs
```

---

AWS creates:

```text
EC2 Instance
```

---

# Why Providers Are Required?

Without Providers:

```text
Terraform Cannot Communicate
```

with any platform.

---

Terraform only understands:

```text
Desired State
```

---

Providers perform:

```text
API Calls

Resource Creation

Resource Updates

Resource Deletion
```

---

# Common Providers

## AWS

```hcl
provider "aws" {

}
```

---

Resources:

```text
EC2

S3

VPC

IAM

ALB
```

---

## Azure

```hcl
provider "azurerm" {

}
```

---

Resources:

```text
Virtual Machines

Storage Accounts

VNets
```

---

## Google Cloud

```hcl
provider "google" {

}
```

---

Resources:

```text
Compute Engine

Cloud Storage

VPC Networks
```

---

## Kubernetes

```hcl
provider "kubernetes" {

}
```

---

Resources:

```text
Deployments

Pods

Services

Ingress
```

---

# Provider Configuration

Example:

```hcl
provider "aws" {

  region = "us-east-1"

}
```

---

Purpose:

```text
Tell Terraform

Which Cloud

Which Region

Which Credentials
```

to use.

---

# Provider Initialization

When running:

```bash
terraform init
```

Terraform:

```text
Downloads Providers
```

---

Example:

```text
AWS Provider

Kubernetes Provider
```

---

Stored locally.

---

# Provider Registry

Terraform downloads providers from:

```text
Terraform Registry
```

---

Examples:

```text
hashicorp/aws

hashicorp/azurerm

hashicorp/google

hashicorp/kubernetes
```

---

# Provider Versioning

Example:

```hcl
terraform {

  required_providers {

    aws = {

      source  = "hashicorp/aws"

      version = "~> 5.0"

    }

  }

}
```

---

Why Version Providers?

```text
Stability

Predictability

Avoid Breaking Changes
```

---

# Multi Provider Example

Terraform can use:

```text
AWS

Kubernetes
```

simultaneously.

---

Architecture:

```text
Terraform
      ↓
 ┌───────────┐
 │ AWS       │
 └───────────┘
      ↓
 AWS Resources

 ┌───────────┐
 │Kubernetes │
 └───────────┘
      ↓
 K8s Resources
```

---

# Real Example

Create:

```text
EKS Cluster
```

using:

```text
AWS Provider
```

---

Deploy:

```text
Applications
```

using:

```text
Kubernetes Provider
```

---

Single Terraform Project.

---

# Provider Authentication

Providers require credentials.

---

AWS Example:

```text
Access Key

Secret Key

IAM Role
```

---

Azure Example:

```text
Service Principal
```

---

Google Example:

```text
Service Account
```

---

# Multi Account Providers

Example:

```text
DEV Account

PROD Account
```

---

Terraform supports:

```hcl
provider "aws" {

  alias = "dev"

}
```

---

```hcl
provider "aws" {

  alias = "prod"

}
```

---

Resources can use:

```hcl
provider = aws.dev
```

or

```hcl
provider = aws.prod
```

---

# Provider Lifecycle

Step 1

```text
Terraform Code Written
```

---

Step 2

```text
terraform init
```

downloads provider.

---

Step 3

```text
terraform plan
```

asks provider:

```text
What Exists?
```

---

Step 4

```text
terraform apply
```

asks provider:

```text
Create/Modify Resources
```

---

Step 5

```text
Provider Calls APIs
```

---

Step 6

```text
Infrastructure Updated
```

---

# How Providers Work Internally?

Provider receives:

```text
Terraform Configuration
```

---

Converts into:

```text
API Requests
```

---

Example:

```text
Create EC2
```

↓

```text
AWS API Request
```

↓

```text
EC2 Created
```

---

# What is Provider Development?

Sometimes organizations have systems that Terraform cannot manage.

Examples:

```text
Internal DNS System

Internal CMDB

Internal Ticketing Platform

Private Cloud
```

---

No provider available.

---

Solution:

```text
Build Custom Provider
```

---

# Provider Development Concept

Provider Developers create:

```text
Terraform Plugin
```

that communicates with:

```text
Custom APIs
```

---

Architecture:

```text
Terraform
      ↓
Custom Provider
      ↓
Company API
      ↓
Internal Platform
```

---

# Example

Company has:

```text
Internal DNS Platform
```

---

Terraform cannot manage it.

---

Engineering Team builds:

```text
DNS Provider
```

---

Now Terraform can:

```text
Create DNS Records

Update Records

Delete Records
```

---

# Provider SDK

HashiCorp provides:

```text
Terraform Provider SDK
```

---

Used to build:

```text
Custom Providers
```

---

Languages commonly used:

```text
Go
```

---

# Production Example

Platform Team manages:

```text
AWS

Kubernetes

Internal DNS

CMDB
```

---

Terraform uses:

```text
AWS Provider

Kubernetes Provider

Custom DNS Provider

Custom CMDB Provider
```

---

Single Infrastructure Workflow.

---

# Benefits of Providers

```text
Extensible

Multi Cloud

Reusable

Consistent

API Driven
```

---

# Common Interview Questions

## What is a Terraform Provider?

### Short Answer

A Provider is a plugin that enables Terraform to interact with external platforms and APIs.

### Detailed Explanation

Providers translate Terraform configurations into API calls that create, modify, or delete infrastructure resources.

### Production Example

AWS Provider creating EC2 instances and VPCs.

### Follow-Up Questions

- Why are providers required?
- What happens during terraform init?

---

## What Happens During terraform init?

### Short Answer

Terraform downloads and initializes required providers.

### Detailed Explanation

Terraform retrieves provider plugins from the Terraform Registry and prepares the working directory.

### Production Example

Downloading AWS and Kubernetes providers before deployment.

### Follow-Up Questions

- Where are providers downloaded from?
- Why is init required?

---

## Can Terraform Use Multiple Providers?

### Short Answer

Yes.

### Detailed Explanation

Terraform can manage resources across multiple providers within the same project.

### Production Example

AWS Provider managing EKS and Kubernetes Provider deploying applications.

### Follow-Up Questions

- Can AWS and Kubernetes providers be used together?
- How does Terraform handle provider authentication?

---

## What is Provider Development?

### Short Answer

Provider development is the process of creating custom Terraform providers for unsupported platforms.

### Detailed Explanation

Organizations can build providers that integrate Terraform with internal systems and APIs.

### Production Example

Building a provider for an internal DNS platform.

### Follow-Up Questions

- Which language is commonly used?
- What is the Terraform Provider SDK?

---

# Key Takeaways

```text
Providers allow Terraform to communicate with cloud platforms and APIs.

Terraform cannot manage infrastructure without providers.

terraform init downloads required providers.

Providers translate Terraform code into API calls.

AWS, Azure, Google Cloud, and Kubernetes all have providers.

Terraform supports multiple providers in a single project.

Provider versions should be pinned for stability.

Provider authentication depends on the target platform.

Organizations can build custom providers for internal systems.

Provider development is a key capability for advanced Terraform and platform engineering use cases.
```