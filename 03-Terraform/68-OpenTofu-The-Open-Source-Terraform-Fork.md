# OpenTofu - The Open Source Terraform Fork

## Introduction

Terraform became one of the most popular:

```text
Infrastructure As Code (IaC)
```

tools in the cloud industry.

---

Organizations worldwide use Terraform for:

```text
AWS

Azure

GCP

Kubernetes

Multi-Cloud Infrastructure
```

---

For many years Terraform was:

```text
Fully Open Source
```

---

However, in 2023 HashiCorp changed Terraform's license.

---

This led to the creation of:

```text
OpenTofu
```

---

# What is OpenTofu?

OpenTofu is:

```text
An Open Source Fork Of Terraform
```

---

It was created to ensure that Terraform-like infrastructure automation remains:

```text
Open Source

Community Driven

Vendor Neutral
```

---

# Why Was OpenTofu Created?

Historically:

```text
Terraform Used MPL License
```

---

HashiCorp changed Terraform licensing to:

```text
Business Source License (BSL)
```

---

Many organizations preferred:

```text
Fully Open Source Solutions
```

---

As a result:

```text
OpenTofu Project Started
```

---

# Who Maintains OpenTofu?

OpenTofu is governed by:

```text
Linux Foundation
```

---

Benefits:

```text
Community Governance

Vendor Neutrality

Open Development
```

---

# Terraform vs OpenTofu

Terraform:

```text
Owned By HashiCorp
```

---

OpenTofu:

```text
Community Driven
```

---

Terraform:

```text
BSL License
```

---

OpenTofu:

```text
Open Source License
```

---

# Core Goal

OpenTofu aims to provide:

```text
Terraform Compatibility
```

---

Meaning:

```text
Existing Terraform Code
```

should work with:

```text
OpenTofu
```

---

# Architecture

```text
OpenTofu
      ↓
Providers
      ↓
Cloud Platforms
```

---

Very similar to:

```text
Terraform
      ↓
Providers
      ↓
Cloud Platforms
```

---

# Supported Platforms

OpenTofu supports:

```text
AWS

Azure

Google Cloud

Kubernetes

VMware

Many Other Providers
```

---

# Terraform Configuration Compatibility

Example:

```hcl
resource "aws_instance" "catalogue" {

  ami           = "ami-123456"

  instance_type = "t3.micro"

}
```

---

This configuration can generally run in:

```text
Terraform

OpenTofu
```

---

# Commands Comparison

Terraform:

```bash
terraform init

terraform plan

terraform apply

terraform destroy
```

---

OpenTofu:

```bash
tofu init

tofu plan

tofu apply

tofu destroy
```

---

# Example Workflow

Initialize:

```bash
tofu init
```

---

Preview:

```bash
tofu plan
```

---

Deploy:

```bash
tofu apply
```

---

Destroy:

```bash
tofu destroy
```

---

# State Files

OpenTofu uses:

```text
State Files
```

similar to Terraform.

---

Example:

```text
terraform.tfstate
```

---

State stores:

```text
Resources

Dependencies

Metadata
```

---

# Providers

OpenTofu uses:

```text
Providers
```

just like Terraform.

---

Examples:

```text
AWS Provider

Azure Provider

Google Provider

Kubernetes Provider
```

---

# Modules

OpenTofu supports:

```text
Terraform Modules
```

---

Example:

```text
VPC Module

EKS Module

ALB Module
```

---

No significant changes required.

---

# Remote State

OpenTofu supports:

```text
S3 Backend

Azure Storage

GCS

Terraform Cloud Compatible Backends
```

---

# Multi-Cloud Support

Example:

```text
AWS

Azure

GCP
```

---

Same Infrastructure-As-Code principles apply.

---

# Why Organizations Consider OpenTofu?

Reasons:

```text
Open Source Preference

Vendor Neutral Governance

Community Development

License Concerns

Long-Term Flexibility
```

---

# Production Example

Organization:

```text
Hundreds Of Terraform Repositories
```

---

Concern:

```text
Licensing Changes
```

---

Decision:

```text
Migrate To OpenTofu
```

---

Benefits:

```text
Minimal Code Changes

Continued Open Source Adoption
```

---

# Migration Example

Current:

```bash
terraform apply
```

---

New:

```bash
tofu apply
```

---

Most Terraform configurations continue working.

---

# OpenTofu Roadmap

Goals:

```text
Terraform Compatibility

Community Innovation

New Features

Open Governance
```

---

# OpenTofu Ecosystem

```text
OpenTofu
      ↓
Providers
      ↓
Modules
      ↓
Cloud Infrastructure
```

---

# Advantages

```text
Fully Open Source

Linux Foundation Governance

Terraform Compatibility

Community Driven

Multi Cloud Support
```

---

# Challenges

Organizations must evaluate:

```text
Tooling Compatibility

Provider Support

Migration Strategy

Operational Processes
```

---

# OpenTofu vs Terraform Cloud

OpenTofu:

```text
Infrastructure As Code Engine
```

---

Terraform Cloud:

```text
Terraform Management Platform
```

---

Different purposes.

---

# Industry Adoption

OpenTofu adoption is growing among organizations that:

```text
Prefer Open Source

Require Vendor Neutrality

Want Long-Term Community Governance
```

---

# Common Interview Questions

## What is OpenTofu?

### Short Answer

OpenTofu is an open-source fork of Terraform governed by the Linux Foundation.

### Detailed Explanation

It was created after HashiCorp changed Terraform's licensing model and aims to provide a community-driven alternative.

### Production Example

Organizations migrating Terraform projects to OpenTofu for open-source governance.

### Follow-Up Questions

- Why was OpenTofu created?
- Is OpenTofu compatible with Terraform?

---

## What is the Difference Between Terraform and OpenTofu?

### Short Answer

Terraform is maintained by HashiCorp, while OpenTofu is community governed under the Linux Foundation.

### Detailed Explanation

Both tools are very similar technically, but differ in governance and licensing.

### Production Example

Using the same Infrastructure as Code configuration with either Terraform or OpenTofu.

### Follow-Up Questions

- Do they use the same syntax?
- Are providers compatible?

---

## Is OpenTofu Compatible with Terraform?

### Short Answer

Yes, OpenTofu aims for high Terraform compatibility.

### Detailed Explanation

Most Terraform configurations, modules, and workflows work with OpenTofu with little or no modification.

### Production Example

Migrating existing AWS Terraform projects to OpenTofu.

### Follow-Up Questions

- Are state files compatible?
- Do Terraform modules work?

---

## Why Might an Organization Choose OpenTofu?

### Short Answer

For open-source governance, licensing preferences, and vendor neutrality.

### Detailed Explanation

Some organizations prefer community-controlled software and want to avoid vendor-specific licensing concerns.

### Production Example

Large enterprises adopting OpenTofu for long-term open-source strategy.

### Follow-Up Questions

- Who governs OpenTofu?
- Is OpenTofu production ready?

---

# Key Takeaways

```text
OpenTofu is an open-source fork of Terraform.

It is governed by the Linux Foundation.

OpenTofu was created after Terraform's licensing changes.

The goal is to maintain Terraform compatibility.

Most Terraform configurations work with OpenTofu.

OpenTofu supports providers, modules, and remote state.

Commands are similar to Terraform but use the tofu CLI.

Organizations choose OpenTofu for open-source governance and vendor neutrality.

OpenTofu is becoming an important part of the Infrastructure as Code ecosystem.

Understanding OpenTofu is valuable for modern DevOps and Platform Engineering roles.
```