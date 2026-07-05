# Terraform Workspaces

## Introduction

Organizations typically maintain multiple environments.

Examples:

```text
DEV

QA

UAT

PROD
```

---

Each environment requires:

```text
Separate Infrastructure

Separate State

Separate Deployments
```

---

Without proper separation:

```text
DEV Changes

May Impact PROD
```

---

Terraform provides:

```text
Workspaces
```

to manage multiple environments using the same codebase.

---

# What is a Terraform Workspace?

A Workspace is:

```text
An Isolated State Environment
```

for the same Terraform configuration.

---

Each workspace maintains:

```text
Separate State File
```

while sharing:

```text
Same Terraform Code
```

---

# Example

Single Terraform Code:

```text
main.tf
```

---

Workspaces:

```text
DEV

QA

PROD
```

---

Result:

```text
Same Code

Different States
```

---

# Why Workspaces?

Without Workspaces:

```text
dev/
qa/
prod/
```

Need separate directories.

---

With Workspaces:

```text
One Codebase

Multiple States
```

---

# Workspace Architecture

```text
Terraform Code
        ↓
 ┌────────────┐
 │ DEV State  │
 ├────────────┤
 │ QA State   │
 ├────────────┤
 │ PROD State │
 └────────────┘
```

---

# Default Workspace

Terraform automatically creates:

```text
default
```

workspace.

---

Verify:

```bash
terraform workspace show
```

---

Output:

```text
default
```

---

# List Workspaces

Command:

```bash
terraform workspace list
```

---

Example Output:

```text
default
dev
qa
prod
```

---

# Create Workspace

Command:

```bash
terraform workspace new dev
```

---

Result:

```text
Workspace Created

Workspace Selected
```

---

# Create QA Workspace

```bash
terraform workspace new qa
```

---

# Create PROD Workspace

```bash
terraform workspace new prod
```

---

# Switch Workspace

Command:

```bash
terraform workspace select dev
```

---

Output:

```text
Switched To Workspace "dev"
```

---

# Verify Current Workspace

Command:

```bash
terraform workspace show
```

---

Output:

```text
dev
```

---

# State Isolation

Suppose:

```text
DEV Workspace
```

creates:

```text
VPC
```

---

State:

```text
DEV State File
```

---

Switch to:

```text
PROD Workspace
```

---

Terraform sees:

```text
No Resources
```

because:

```text
Different State
```

---

# Example Workflow

Create DEV Infrastructure

```bash
terraform workspace select dev

terraform apply
```

---

Creates:

```text
DEV Resources
```

---

Switch:

```bash
terraform workspace select prod
```

---

Run:

```bash
terraform apply
```

---

Creates:

```text
PROD Resources
```

---

Same code.

Different infrastructure.

---

# Using Workspace Name in Code

Terraform exposes:

```text
terraform.workspace
```

---

Example:

```hcl
resource "aws_instance" "app" {

  tags = {
    Environment = terraform.workspace
  }

}
```

---

DEV Result

```text
Environment = dev
```

---

PROD Result

```text
Environment = prod
```

---

# Dynamic Resource Naming

Example:

```hcl
Name = "catalogue-${terraform.workspace}"
```

---

DEV:

```text
catalogue-dev
```

---

QA:

```text
catalogue-qa
```

---

PROD:

```text
catalogue-prod
```

---

# Workspace-Based Instance Type

Example:

```hcl
instance_type = terraform.workspace == "prod" ? "t3.medium" : "t3.micro"
```

---

DEV:

```text
t3.micro
```

---

PROD:

```text
t3.medium
```

---

# Workspace State Storage

Local Example:

```text
terraform.tfstate.d/
```

---

Structure:

```text
terraform.tfstate.d
├── dev
├── qa
└── prod
```

---

Each workspace gets:

```text
Separate State
```

---

# Workspaces with S3 Backend

State Path:

```text
dev/terraform.tfstate

qa/terraform.tfstate

prod/terraform.tfstate
```

---

Single bucket.

Multiple workspace states.

---

# Production Example

Infrastructure:

```text
VPC

ALB

EC2

ASG
```

---

Developer wants:

```text
DEV Environment
```

---

Command:

```bash
terraform workspace select dev
terraform apply
```

---

Production Team:

```text
PROD Environment
```

---

Command:

```bash
terraform workspace select prod
terraform apply
```

---

Same codebase.

Different environments.

---

# Advantages

```text
State Isolation

Code Reuse

Environment Separation

Simple Management

Reduced Duplication
```

---

# Limitations

Workspaces are useful for:

```text
Small To Medium Projects
```

---

Large enterprises often prefer:

```text
Separate Repositories

Separate Backends

Terragrunt
```

---

because:

```text
Environment Complexity Increases
```

---

# Workspace vs TFVARS

TFVARS:

```text
Different Variable Files
```

---

Workspaces:

```text
Different State Files
```

---

Best Practice:

```text
Use Both Together
```

---

Example:

```bash
terraform workspace select prod

terraform apply -var-file=prod.tfvars
```

---

# Common Interview Questions

## What is a Terraform Workspace?

### Short Answer

A Workspace is an isolated state environment for the same Terraform configuration.

### Detailed Explanation

Workspaces allow multiple environments to share the same code while maintaining separate state files.

### Production Example

Managing DEV and PROD using the same Terraform codebase.

### Follow-Up Questions

- What is the default workspace?
- How do you switch workspaces?

---

## How Do You Create a Workspace?

### Short Answer

Using:

```bash
terraform workspace new <name>
```

### Detailed Explanation

Terraform creates a new workspace with its own state file.

### Production Example

Creating separate workspaces for DEV, QA, and PROD.

### Follow-Up Questions

- How do you list workspaces?
- How do you delete workspaces?

---

## What is terraform.workspace?

### Short Answer

A built-in variable containing the current workspace name.

### Detailed Explanation

It allows Terraform code to dynamically adjust configuration based on the active workspace.

### Production Example

Creating environment-specific resource names.

### Follow-Up Questions

- Can it be used in tags?
- Can it be used in conditional expressions?

---

## Workspace vs TFVARS?

### Short Answer

Workspaces separate state, while TFVARS separate variable values.

### Detailed Explanation

Workspaces provide isolated infrastructure tracking, while TFVARS provide environment-specific configuration values.

### Production Example

Using prod.tfvars with the prod workspace.

### Follow-Up Questions

- Can both be used together?
- Which one separates infrastructure state?

---

# Key Takeaways

```text
Terraform Workspaces provide isolated state environments.

The default workspace is called "default".

Each workspace maintains its own state file.

The same Terraform code can be used across multiple environments.

terraform.workspace provides the current workspace name.

Workspaces help manage DEV, QA, and PROD environments.

Workspaces separate state, not code.

TFVARS and Workspaces are often used together.

Large organizations may prefer separate repositories or Terragrunt for environment management.

Terraform Workspaces are a common interview and production topic.
```