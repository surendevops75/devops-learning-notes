# Custom Providers and Plugin Architecture

## Introduction

Terraform supports hundreds of providers.

Examples:

```text
AWS

Azure

Google Cloud

Kubernetes

VMware

GitHub

Datadog
```

---

However, many organizations have:

```text
Internal Platforms

Private APIs

Custom Applications

Enterprise Systems
```

---

Terraform cannot manage them unless a provider exists.

---

Solution:

```text
Custom Providers
```

---

# What is a Custom Provider?

A Custom Provider is:

```text
A Terraform Plugin Built To Communicate With A Specific API
```

---

It extends Terraform's capabilities.

---

Instead of managing:

```text
AWS

Azure

GCP
```

---

Terraform can also manage:

```text
Internal DNS

CMDB

Ticketing Systems

Private Cloud Platforms
```

---

# Why Custom Providers?

Suppose a company has:

```text
Internal DNS Platform
```

---

Engineers currently create DNS records manually.

---

Problems:

```text
Manual Work

Human Errors

No Automation

No Version Control
```

---

Solution:

```text
Create Custom Terraform Provider
```

---

Now Terraform can:

```text
Create Records

Update Records

Delete Records
```

---

# Plugin Architecture

Terraform follows a:

```text
Plugin Based Architecture
```

---

Core Terraform Engine:

```text
Does Not Know AWS

Does Not Know Azure

Does Not Know Kubernetes
```

---

Everything is delegated to:

```text
Providers
```

---

# Architecture

```text
Terraform Core
        ↓
Provider Plugin
        ↓
API
        ↓
Infrastructure
```

---

# AWS Example

```text
Terraform Core
       ↓
AWS Provider
       ↓
AWS API
       ↓
EC2 Created
```

---

# Custom Provider Example

```text
Terraform Core
       ↓
Custom DNS Provider
       ↓
DNS API
       ↓
DNS Record Created
```

---

# How Terraform Communicates

Terraform Core communicates with providers using:

```text
RPC Protocol
```

---

Provider acts as a separate process.

---

Architecture:

```text
Terraform
      ↓
RPC
      ↓
Provider Plugin
      ↓
API
```

---

# Provider Responsibilities

Providers handle:

```text
Authentication

Resource Creation

Resource Updates

Resource Deletion

State Synchronization
```

---

# Example Resource

Custom DNS Record:

```hcl
resource "company_dns_record" "app" {

  name = "catalogue"

  ip   = "10.0.1.10"

}
```

---

Terraform:

```text
Reads Configuration
```

---

Provider:

```text
Calls DNS API
```

---

Result:

```text
DNS Record Created
```

---

# Resource Lifecycle

Every provider typically supports:

## Create

```text
Create Resource
```

---

## Read

```text
Read Current State
```

---

## Update

```text
Modify Resource
```

---

## Delete

```text
Remove Resource
```

---

This is commonly known as:

```text
CRUD Operations
```

---

# CRUD Flow

```text
Create

Read

Update

Delete
```

---

# Provider Development Workflow

Step 1

```text
Identify API
```

---

Step 2

```text
Design Resources
```

---

Step 3

```text
Implement Provider Logic
```

---

Step 4

```text
Compile Provider
```

---

Step 5

```text
Use In Terraform
```

---

# Example Organization

Company Services:

```text
Internal DNS

CMDB

Inventory System
```

---

Goal:

```text
Manage Everything Through Terraform
```

---

Custom Providers Created:

```text
DNS Provider

CMDB Provider

Inventory Provider
```

---

# Terraform Provider SDK

HashiCorp provides:

```text
Terraform Provider SDK
```

---

Purpose:

```text
Build Custom Providers
```

---

Benefits:

```text
Standard Development Model

Plugin Framework

Testing Tools
```

---

# Common Language

Most Terraform providers are written in:

```text
Go
```

---

Reason:

```text
Terraform Core Is Written In Go
```

---

# Provider Framework

Modern provider development often uses:

```text
Terraform Plugin Framework
```

---

Benefits:

```text
Simpler Development

Better Testing

Future Compatibility
```

---

# Authentication Example

Custom Provider:

```text
Internal DNS Provider
```

---

Authentication:

```text
API Token

Username

Password

OAuth
```

---

Provider handles authentication automatically.

---

# State Management

Providers are responsible for:

```text
Reading Current State

Updating State

Returning Resource Data
```

---

Example:

```text
DNS Record Updated
```

---

Provider informs Terraform.

---

Terraform updates:

```text
terraform.tfstate
```

---

# Multi-Provider Environment

Production Example:

```text
AWS Provider

Kubernetes Provider

GitHub Provider

Custom DNS Provider
```

---

Architecture:

```text
Terraform
      ↓
AWS Provider
      ↓
AWS

Terraform
      ↓
Kubernetes Provider
      ↓
Kubernetes

Terraform
      ↓
DNS Provider
      ↓
Internal DNS
```

---

# Enterprise Use Cases

Custom Providers commonly manage:

```text
CMDB Systems

Internal DNS

Private Clouds

Service Catalogs

Ticketing Systems

Asset Inventories
```

---

# Example

Developer creates:

```text
New Application
```

---

Terraform:

```text
Creates EC2

Creates ALB

Creates DNS Record

Updates CMDB
```

---

All through:

```text
Multiple Providers
```

---

# Provider Registry

Public providers are typically distributed through:

```text
Terraform Registry
```

---

Internal providers may be distributed through:

```text
Private Registries

Internal Repositories
```

---

# Advantages

```text
Unlimited Extensibility

Infrastructure Standardization

Automation

API Integration

Single Workflow
```

---

# Challenges

```text
Development Complexity

Maintenance

API Compatibility

Testing Requirements
```

---

# Real Production Example

Organization has:

```text
AWS

Kubernetes

Internal DNS

Internal CMDB
```

---

Terraform manages:

```text
Cloud Infrastructure

Application Deployment

DNS Records

Configuration Management
```

---

using:

```text
AWS Provider

Kubernetes Provider

DNS Provider

CMDB Provider
```

---

# Common Interview Questions

## What is a Custom Terraform Provider?

### Short Answer

A Custom Provider is a Terraform plugin that integrates Terraform with a specific API or platform.

### Detailed Explanation

Custom providers allow Terraform to manage systems that are not supported by existing providers.

### Production Example

A provider for an organization's internal DNS platform.

### Follow-Up Questions

- Why build a custom provider?
- How does Terraform communicate with providers?

---

## What is Terraform's Plugin Architecture?

### Short Answer

Terraform uses providers as plugins to communicate with external systems.

### Detailed Explanation

Terraform Core delegates infrastructure operations to provider plugins through RPC communication.

### Production Example

AWS Provider managing AWS resources and Kubernetes Provider managing Kubernetes resources.

### Follow-Up Questions

- Is AWS functionality built into Terraform?
- What role do providers play?

---

## Which Language is Commonly Used for Provider Development?

### Short Answer

Go.

### Detailed Explanation

Terraform Core and the Provider SDK are written in Go, making it the standard language for provider development.

### Production Example

Developing an internal CMDB provider using Go.

### Follow-Up Questions

- What SDK is used?
- Can providers be written in other languages?

---

## What Problems Do Custom Providers Solve?

### Short Answer

They allow Terraform to manage internal or unsupported platforms.

### Detailed Explanation

Organizations can integrate Terraform with private APIs and enterprise systems using custom providers.

### Production Example

Managing internal DNS and CMDB platforms through Infrastructure as Code.

### Follow-Up Questions

- Can Terraform manage internal applications?
- What enterprise systems commonly use custom providers?

---

# Key Takeaways

```text
Terraform uses a plugin-based architecture.

Providers extend Terraform's capabilities.

Custom providers integrate Terraform with unsupported platforms.

Terraform communicates with providers using RPC.

Providers handle CRUD operations for resources.

Most providers are developed in Go.

HashiCorp provides SDKs and frameworks for provider development.

Custom providers are common in large enterprises.

Internal DNS, CMDB, and ticketing systems are common provider use cases.

Custom providers enable organizations to manage all infrastructure through a single Terraform workflow.
```