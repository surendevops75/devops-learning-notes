# Production Incident: SSH Failure and Package Dependency Management

## Introduction

One of the most common responsibilities of a DevOps Engineer is troubleshooting production incidents.

Not every issue causes:

```text
Application Failure
```

Sometimes:

```text
Application Works

Server Access Fails
```

This makes troubleshooting difficult because:

```text
Application Is Running

But Engineers Cannot Login
```

The following is a real-world type of incident involving:

```text
SSH

OpenSSL

Package Dependencies

AMI Management
```

---

# Incident Scenario

Application deployment was completed successfully using:

```text
Terraform

Ansible
```

---

Deployment Result:

```text
Application Running

Health Checks Passing

Users Accessing Application
```

Everything appeared normal.

---

# Problem Appears Later

After a few days:

```text
SSH Login Failed
```

---

Engineers observed:

```text
Cannot SSH Into Server
```

---

However:

```text
Application Still Running
```

---

# Strange Observation

Normally if a server is broken:

```text
Application Stops
```

---

But in this case:

```text
Application Running

SSH Not Working
```

---

# Initial Investigation

Questions:

```text
Network Issue?

Security Group Issue?

SSH Service Down?

Package Issue?
```

---

# Understanding SSH Dependency

SSH depends on several Linux packages.

Example:

```text
openssh

openssl

openssl-libs
```

---

Relationship:

```text
SSH
 ↓
OpenSSL
 ↓
OpenSSL Libraries
```

---

# Recent Change

While deploying the application:

```text
python3-devel
```

was installed.

---

# Why Was python3-devel Installed?

Many applications require:

```text
Python Development Libraries

Build Dependencies

Compilation Packages
```

---

# Package Installation Flow

```text
python3-devel
      ↓
Dependency Packages
      ↓
openssl-libs
```

---

# Hidden Dependency Upgrade

While installing:

```text
python3-devel
```

Linux automatically upgraded:

```text
openssl-libs
```

to a newer version.

---

# Existing Packages

Server already contained:

```text
openssl

openssl-server

openssl-libs
```

---

# Problem

Only:

```text
openssl-libs
```

was upgraded.

---

Other packages remained:

```text
Old Version
```

---

# Result

Version mismatch occurred.

Example:

```text
openssl-libs → New Version

openssl-server → Old Version
```

---

# What is Version Mismatch?

Two dependent packages expect:

```text
Compatible Versions
```

---

Example:

```text
Package A Version 5

Package B Version 5
```

Works correctly.

---

Example:

```text
Package A Version 8

Package B Version 5
```

May fail.

---

# SSH Dependency Failure

SSH relies on:

```text
OpenSSL Libraries
```

---

After upgrade:

```text
SSH
 ↓
Old OpenSSL Server
 ↓
New OpenSSL Libraries
```

---

Compatibility issue occurred.

---

# Final Result

```text
SSH Service Failed
```

---

But:

```text
Application Continued Running
```

because the application itself did not depend on SSH.

---

# Why Application Continued Working?

Application Process:

```text
Already Running
```

---

SSH Service:

```text
Separate Process
```

---

Application and SSH are independent services.

---

# Architecture

```text
Application
    ↓
Business Traffic
```

---

```text
SSH
    ↓
Administrative Access
```

---

Breaking SSH does not necessarily stop the application.

---

# Root Cause Analysis (RCA)

Problem:

```text
Cannot SSH Into Server
```

---

Impact:

```text
Operational Access Lost
```

---

Root Cause:

```text
OpenSSL Version Mismatch
```

---

Trigger:

```text
python3-devel Installation
```

---

# Immediate Fix

Upgrade all related OpenSSL packages.

Example:

```text
openssl

openssl-server

openssl-libs
```

must remain compatible.

---

# Long-Term Fix

Trainer Recommendation:

```text
Fix Base AMI
```

---

# Why Fix the AMI?

Current AMI contains:

```text
Outdated OpenSSL Packages
```

---

Every new server created from this AMI inherits:

```text
Same Problem
```

---

# Solution 1

Update:

```text
devops-practice AMI
```

with latest OpenSSL versions.

---

# Result

New servers launch with:

```text
Compatible Package Versions
```

---

# Solution 2

During deployment:

When installing:

```text
python3-devel
```

ensure related packages are upgraded.

---

Example

```text
openssl

openssl-server

openssl-libs
```

---

# Why Is AMI Important?

AMI acts as:

```text
Golden Image
```

---

Every EC2 instance originates from:

```text
AMI
```

---

If AMI contains problems:

```text
All Future Servers
```

inherit the problem.

---

# Golden Image Concept

Instead of fixing:

```text
100 Servers
```

fix:

```text
One AMI
```

---

Then launch:

```text
100 Correct Servers
```

---

# Production Best Practice

Before installing packages:

```text
Understand Dependencies
```

---

Verify:

```text
What Will Be Upgraded?
```

---

Example Commands

```bash
yum info package-name
```

---

```bash
yum deplist package-name
```

---

```bash
rpm -qa | grep openssl
```

---

# Production Troubleshooting Flow

Problem:

```text
Application Running

SSH Not Working
```

---

Step 1

Verify:

```text
Application Health
```

---

Step 2

Check:

```text
Security Groups
```

---

Step 3

Check:

```text
SSH Service
```

---

Step 4

Investigate:

```text
Recent Package Changes
```

---

Step 5

Validate:

```text
Package Versions
```

---

Step 6

Perform RCA

---

# Lessons Learned

```text
Package Dependencies Matter

Not Every Failure Stops Applications

Golden Images Must Be Maintained

Version Compatibility Is Important
```

---

# Production Example

Deployment:

```text
Terraform
 ↓
Ansible
 ↓
Install python3-devel
```

---

Hidden Action:

```text
openssl-libs Upgraded
```

---

Result:

```text
OpenSSL Version Mismatch
```

---

Impact:

```text
SSH Failure
```

---

Application:

```text
Still Running
```

---

# Common Interview Questions

## Application is Running but SSH is Not Working. What Would You Check?

### Short Answer

Check Security Groups, SSH service status, recent package changes, and dependency issues.

### Detailed Explanation

The application and SSH service are independent. Package upgrades, SSH daemon failures, or network changes can affect SSH without impacting application traffic.

### Production Example

OpenSSL version mismatch caused SSH failure while the application continued running.

### Follow-Up Questions

- Can an application run if SSH is broken?
- What Linux services does SSH depend on?

---

## What is a Package Dependency?

### Short Answer

A package dependency is another package required for a package to function correctly.

### Detailed Explanation

Many Linux packages rely on shared libraries and supporting packages. Upgrading one package may indirectly upgrade dependencies.

### Production Example

Installing python3-devel upgraded openssl-libs.

### Follow-Up Questions

- How can you check package dependencies?
- Why are dependencies important?

---

## What is Version Mismatch?

### Short Answer

A situation where related packages are running incompatible versions.

### Detailed Explanation

Dependent software expects compatible package versions. Mismatches may cause application or service failures.

### Production Example

openssl-libs upgraded while openssl-server remained on an older version.

### Follow-Up Questions

- How do you identify version mismatches?
- How do you prevent them?

---

## Why Should Golden Images Be Updated?

### Short Answer

To prevent repeated issues in future servers.

### Detailed Explanation

Updating the AMI ensures all newly launched servers inherit corrected package versions and configurations.

### Production Example

Updating devops-practice AMI with compatible OpenSSL packages.

### Follow-Up Questions

- What is a Golden Image?
- Why are AMIs important in production?

---

# Key Takeaways

```text
Applications and SSH services are independent.

An application can continue running even if SSH fails.

Package dependency upgrades can introduce unexpected issues.

OpenSSL version mismatches can break SSH.

Always understand package dependencies before upgrades.

Golden Images should be maintained and updated regularly.

Fixing the AMI prevents the same issue from appearing in future servers.

Root Cause Analysis is critical in production incident management.

Package compatibility is a key responsibility of DevOps Engineers.
```