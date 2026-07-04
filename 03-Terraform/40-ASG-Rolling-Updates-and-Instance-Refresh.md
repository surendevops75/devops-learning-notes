# ASG Rolling Updates and Instance Refresh

## Introduction

Applications continuously receive:

```text
New Features

Bug Fixes

Security Patches

Performance Improvements
```

When a new version is released, production servers must be updated safely.

The challenge is:

```text
Update Application

Without Downtime
```

AWS Auto Scaling Groups provide:

```text
Instance Refresh

Rolling Updates
```

to solve this problem.

---

# Current Scenario

Suppose:

```text
Catalogue Service
```

is running on:

```text
Old AMI
```

Example:

```text
catalogue-v1
```

Current Servers:

```text
Catalogue-1

Catalogue-2

Catalogue-3

Catalogue-4
```

---

# New Application Release

Developers release:

```text
New Application Version
```

Terraform deployment starts.

Command:

```bash
terraform apply -auto-approve
```

---

# Deployment Workflow

```text
Create New Instance
        ↓
Configure Instance
        ↓
Pull Latest Code
        ↓
Stop Instance
        ↓
Create New AMI
        ↓
Update Launch Template
        ↓
ASG Refresh
```

---

# Step 1 - Create New Instance

Terraform creates a temporary build server.

Purpose:

```text
Install Latest Application

Configure Environment

Validate Application
```

---

# Step 2 - Configure Instance

Using:

```text
remote-exec

Shell Script

Ansible
```

---

# What Happens?

Ansible:

```text
Pulls Latest Code

Installs Dependencies

Configures Application

Starts Services
```

---

# Production Example

Catalogue Application:

```text
Install Java

Deploy Latest Catalogue Code

Configure Systemd

Start Application
```

---

# Step 3 - Stop Instance

After validation:

```text
Stop Instance
```

Purpose:

```text
Create Stable Image
```

---

# Step 4 - Create New AMI

AWS creates:

```text
New AMI
```

Example:

```text
catalogue-v2
```

---

# Important Point

Every deployment creates:

```text
New AMI ID
```

Example:

```text
AMI-1111

AMI-2222

AMI-3333
```

Each represents a different application version.

---

# Step 5 - Update Launch Template

Current Launch Template:

```text
AMI = catalogue-v1
```

---

New Launch Template Version:

```text
AMI = catalogue-v2
```

---

# Why New Version?

Existing servers:

```text
Still Running Old AMI
```

Launch Template must point to:

```text
Latest AMI
```

---

# Launch Template Versioning

Example:

```text
Version 1 → catalogue-v1

Version 2 → catalogue-v2

Version 3 → catalogue-v3
```

---

# Step 6 - Start ASG Refresh

Auto Scaling Group receives:

```text
New Launch Template Version
```

---

# What is ASG Refresh?

ASG Refresh replaces old instances with new instances created from the latest Launch Template.

---

# Why Not Replace Everything At Once?

If all servers are removed:

```text
Application Downtime
```

may occur.

---

# Solution

```text
Rolling Update
```

---

# What is Rolling Update?

Replace:

```text
One Server At A Time
```

instead of:

```text
All Servers At Once
```

---

# HR Analogy

Suppose:

```text
10 Employees
```

are working.

Need to replace them.

Wrong Approach:

```text
Fire All 10

Hire New 10
```

Result:

```text
Work Stops
```

---

Correct Approach:

```text
Recruit One Person

Give KT

Release One Old Employee

Repeat
```

Work continues normally.

---

# Same Concept in AWS

Current Instances:

```text
Catalogue-1

Catalogue-2

Catalogue-3

Catalogue-4
```

running:

```text
catalogue-v1
```

---

# Refresh Process

Step 1

Create:

```text
Catalogue-5
```

using:

```text
catalogue-v2
```

---

# Validation

Wait until:

```text
Instance Running

Health Check Passed
```

---

# Step 2

Terminate:

```text
Catalogue-1
```

---

# Step 3

Create:

```text
Catalogue-6
```

using:

```text
catalogue-v2
```

---

# Step 4

Terminate:

```text
Catalogue-2
```

---

# Process Continues

```text
Create New

Validate

Delete Old
```

until all instances use the new version.

---

# During Rolling Update

Old Instances:

```text
Catalogue-v1
```

New Instances:

```text
Catalogue-v2
```

Both versions may run simultaneously.

---

# Important Observation

At a given time:

```text
Old Version Running

New Version Running
```

---

# Why Is This Safe?

Traffic continues to be served.

Benefits:

```text
No Downtime

Gradual Rollout

Easy Monitoring
```

---

# Traffic Flow

```text
User
   ↓
ALB
   ↓
Target Group
   ↓
Old + New Instances
```

---

# Health Check Validation

Before deleting old instance:

AWS verifies:

```text
Instance Healthy
```

---

Example:

```bash
curl http://catalogue.backend-alb-dev.daws86s.fun/health
```

Response:

```text
Healthy
```

---

# If Health Check Fails

AWS:

```text
Stops Refresh

Keeps Old Instances
```

This prevents production outages.

---

# Complete Refresh Workflow

```text
terraform apply
        ↓
Create Build Instance
        ↓
Configure Application
        ↓
Create New AMI
        ↓
Update Launch Template
        ↓
Start ASG Refresh
        ↓
Create New Instance
        ↓
Health Check
        ↓
Delete Old Instance
        ↓
Repeat
```

---

# Production Architecture

```text
New Application Release
          ↓
Terraform Apply
          ↓
New AMI
          ↓
Launch Template Version
          ↓
ASG Refresh
          ↓
Rolling Update
          ↓
Production Updated
```

---

# Benefits of Rolling Updates

```text
No Downtime

Safer Deployments

Easy Rollback

Gradual Validation

Better User Experience
```

---

# Common Interview Questions

## What is ASG Instance Refresh?

### Short Answer

ASG Instance Refresh replaces existing instances using the latest Launch Template version.

### Detailed Explanation

When a new Launch Template version is created, Instance Refresh gradually replaces old instances with new instances launched from the updated template.

### Production Example

Replacing Catalogue servers running catalogue-v1 with catalogue-v2.

### Follow-Up Questions

- Does Instance Refresh cause downtime?
- What happens if health checks fail?

---

## What is a Rolling Update?

### Short Answer

A Rolling Update replaces instances gradually instead of all at once.

### Detailed Explanation

New instances are launched and validated before old instances are terminated, ensuring continuous availability.

### Production Example

Creating one Catalogue-v2 instance and deleting one Catalogue-v1 instance repeatedly until all servers are updated.

### Follow-Up Questions

- Why is rolling update preferred?
- What are alternatives to rolling updates?

---

## Why Create a New AMI for Every Release?

### Short Answer

To create immutable and versioned deployments.

### Detailed Explanation

Every application release is packaged into a new AMI, ensuring consistency across all instances.

### Production Example

catalogue-v1, catalogue-v2, and catalogue-v3 AMIs representing different releases.

### Follow-Up Questions

- What is immutable infrastructure?
- Why avoid updating production servers manually?

---

## Why Are Old and New Versions Running Together During Refresh?

### Short Answer

To avoid downtime during deployment.

### Detailed Explanation

New instances are validated before old instances are removed, ensuring uninterrupted service.

### Production Example

Catalogue-v1 and Catalogue-v2 serving traffic simultaneously during deployment.

### Follow-Up Questions

- Is this always safe?
- How does AWS control traffic during refresh?

---

# Key Takeaways

```text
Application updates usually create new AMIs.

Every release generates a new AMI ID.

Launch Templates are updated to reference the latest AMI.

ASG Refresh replaces old instances using the new Launch Template version.

Rolling Updates replace servers one at a time.

Health Checks validate new servers before old servers are removed.

Old and new application versions may run simultaneously.

Rolling Updates provide zero downtime deployments.

ASG Refresh is commonly used in production AWS environments.

This deployment strategy supports immutable infrastructure principles.
```