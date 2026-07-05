# Terraform State Commands

## Introduction

Terraform stores infrastructure information in:

```text
terraform.tfstate
```

---

This state file acts as:

```text
Source Of Truth
```

for Terraform.

---

Terraform uses state to track:

```text
Resource IDs

Dependencies

Attributes

Relationships
```

---

Sometimes engineers need to:

```text
Inspect State

Modify State

Move Resources

Remove Resources
```

without affecting actual infrastructure.

---

Terraform provides:

```text
terraform state
```

commands for this purpose.

---

# Why State Commands?

Suppose:

```text
Infrastructure Exists

Terraform State Exists
```

---

But:

```text
State Incorrect

Resource Renamed

Resource Imported Wrongly
```

---

State commands help fix these situations.

---

# Common State Commands

```text
terraform state list

terraform state show

terraform state rm

terraform state mv

terraform state pull

terraform state push
```

---

# Terraform State List

Purpose:

```text
Show All Resources
```

currently tracked by Terraform.

---

Command:

```bash
terraform state list
```

---

Example Output

```text
aws_vpc.main

aws_subnet.public

aws_subnet.private

aws_security_group.catalogue

aws_instance.catalogue
```

---

# When to Use?

```text
Check Managed Resources

Verify Imports

Troubleshooting
```

---

# Production Example

After importing:

```text
VPC

Subnets

Security Groups
```

---

Verify resources:

```bash
terraform state list
```

---

# Terraform State Show

Purpose:

```text
Display Resource Details
```

from state.

---

Command:

```bash
terraform state show RESOURCE
```

---

Example

```bash
terraform state show aws_instance.catalogue
```

---

Output

```text
AMI

Instance Type

Tags

Private IP

Subnet ID
```

---

# Why Use State Show?

To inspect:

```text
Current Resource Values
```

---

Useful after:

```text
Import

Troubleshooting

State Investigation
```

---

# Production Example

Imported EC2.

Need to identify:

```text
AMI

Subnet

Security Group
```

---

Command:

```bash
terraform state show aws_instance.catalogue
```

---

# Terraform State RM

Purpose:

```text
Remove Resource From State
```

without deleting infrastructure.

---

# Important

Removes:

```text
State Entry
```

---

Does NOT Remove:

```text
Actual AWS Resource
```

---

Command:

```bash
terraform state rm RESOURCE
```

---

Example

```bash
terraform state rm aws_instance.catalogue
```

---

Result

Terraform:

```text
Stops Managing Resource
```

---

AWS:

```text
Resource Still Exists
```

---

# Production Use Case

Suppose:

```text
EC2 Managed By Terraform
```

---

Need to hand over management to:

```text
Another Team
```

---

Remove:

```text
State Entry
```

---

Leave:

```text
Infrastructure Running
```

---

# Before State RM

```text
Terraform
    ↓
State
    ↓
EC2
```

---

# After State RM

```text
Terraform

(No Relationship)

EC2 Still Exists
```

---

# Terraform State MV

Purpose:

```text
Move State
```

from one resource address to another.

---

Used during:

```text
Refactoring

Renaming Resources

Module Migration
```

---

Command

```bash
terraform state mv SOURCE DESTINATION
```

---

Example

Current:

```text
aws_instance.catalogue
```

---

New Name:

```text
aws_instance.product_catalogue
```

---

Command:

```bash
terraform state mv \
aws_instance.catalogue \
aws_instance.product_catalogue
```

---

Result

No infrastructure changes.

Only:

```text
State Updated
```

---

# Why Use State MV?

Without it:

```text
Terraform Thinks

Old Resource Deleted

New Resource Created
```

---

May trigger:

```text
Destroy

Recreate
```

---

State MV avoids this.

---

# Production Example

Renaming:

```text
catalogue
```

to:

```text
product_catalogue
```

---

Without State MV:

```text
Resource Recreation
```

possible.

---

With State MV:

```text
No Downtime
```

---

# Terraform State Pull

Purpose:

```text
Download Current State
```

---

Command:

```bash
terraform state pull
```

---

Output:

```text
JSON State File
```

---

Useful for:

```text
Auditing

Backups

Debugging
```

---

# Terraform State Push

Purpose:

```text
Upload State
```

to backend.

---

Command:

```bash
terraform state push terraform.tfstate
```

---

Used rarely.

Typically during:

```text
State Recovery

State Migration
```

---

# Common Troubleshooting Scenario

Problem:

```text
Terraform State Corrupted
```

---

Solution:

```text
Pull Backup

Restore State

Push Correct State
```

---

# State Commands and Remote Backends

Works with:

```text
Local State

S3 Backend

Terraform Cloud
```

---

Terraform automatically interacts with:

```text
Configured Backend
```

---

# Production Scenario

Engineer accidentally imports:

```text
Wrong Security Group
```

---

Need to remove it from state.

---

Command:

```bash
terraform state rm aws_security_group.catalogue
```

---

Result:

```text
AWS Resource Unchanged

State Corrected
```

---

# Another Production Scenario

Resource renamed:

Old:

```text
aws_instance.catalogue
```

---

New:

```text
aws_instance.product_catalogue
```

---

Use:

```bash
terraform state mv
```

---

Avoids:

```text
Destroy/Recreate
```

operations.

---

# State Command Flow

```text
Infrastructure
       ↓
Terraform State
       ↓
State Commands
       ↓
Manage State Safely
```

---

# Common Interview Questions

## What Does terraform state list Do?

### Short Answer

Lists all resources tracked in Terraform state.

### Detailed Explanation

It displays every resource currently managed by Terraform.

### Production Example

Verifying imported resources.

### Follow-Up Questions

- Does it query AWS?
- Does it modify resources?

---

## What Does terraform state show Do?

### Short Answer

Displays detailed information about a resource stored in state.

### Detailed Explanation

Shows resource attributes tracked by Terraform.

### Production Example

Inspecting imported EC2 configuration.

### Follow-Up Questions

- Is AWS queried?
- Can sensitive values appear?

---

## What Does terraform state rm Do?

### Short Answer

Removes a resource from state without deleting the actual resource.

### Detailed Explanation

Terraform stops managing the resource but infrastructure remains intact.

### Production Example

Handing over EC2 management to another team.

### Follow-Up Questions

- Does it delete AWS resources?
- When is it useful?

---

## What Does terraform state mv Do?

### Short Answer

Moves a resource from one state address to another.

### Detailed Explanation

Used during refactoring and renaming resources without recreating infrastructure.

### Production Example

Renaming aws_instance.catalogue.

### Follow-Up Questions

- Why not just rename the resource block?
- What happens without state mv?

---

# Key Takeaways

```text
Terraform state commands manage Terraform state safely.

terraform state list shows all managed resources.

terraform state show displays resource details.

terraform state rm removes resources from state without deleting infrastructure.

terraform state mv renames or relocates resources within state.

terraform state pull downloads current state.

terraform state push uploads state to a backend.

State commands are useful during imports, refactoring, migrations, and troubleshooting.

Improper use of state commands can affect Terraform's understanding of infrastructure.

State management is a critical skill for production Terraform environments.
```