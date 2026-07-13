# Helm Versioning and Environment Management

## Introduction

In previous chapters we learned:

```text
Helm Charts

Templates

Values

Install

Upgrade

Rollback
```

---

In production environments, applications evolve continuously.

---

New releases introduce:

```text
Features

Bug Fixes

Security Patches
```

---

Helm provides:

```text
Version Management

Environment Management
```

to handle these changes safely.

---

# Helm Versioning

Helm uses:

```text
Semantic Versioning
```

---

Format

```text
x.y.z
```

---

Example

```text
1.0.0
```

---

# Semantic Versioning Components

## Major Version

```text
x.y.z
↑
```

---

Example

```text
1.0.0

↓

2.0.0
```

---

Represents

```text
Breaking Changes

Major Features

Architecture Changes
```

---

# Minor Version

```text
x.y.z
  ↑
```

---

Example

```text
1.0.0

↓

1.1.0
```

---

Represents

```text
New Features

Enhancements

Backward Compatible Changes
```

---

# Patch Version

```text
x.y.z
    ↑
```

---

Example

```text
1.1.0

↓

1.1.1
```

---

Represents

```text
Bug Fixes

Security Fixes

Minor Corrections
```

---

# Version Examples

```text
1.0.0
```

Initial release.

---

```text
1.1.0
```

Added new API.

---

```text
1.1.1
```

Fixed production bug.

---

```text
2.0.0
```

Breaking API change.

---

# Chart Version vs Application Version

Many beginners confuse these.

---

# Chart Version

Stored in:

```yaml
version:
```

---

Example

```yaml
version: 1.0.0
```

---

Represents

```text
Helm Chart Version
```

---

Used by Helm.

---

# Application Version

Stored in:

```yaml
appVersion:
```

---

Example

```yaml
appVersion: "4.0.0"
```

---

Represents

```text
Docker Image Version

Application Version
```

---

Used by application teams.

---

# Example Chart.yaml

```yaml
apiVersion: v2

name: catalogue

description: Catalogue Service

version: 1.0.0

appVersion: "4.0.0"
```

---

Explanation

```text
Chart Version = 1.0.0

Application Version = 4.0.0
```

---

# When Should Chart Version Change?

Suppose:

```text
Template Modified

Values Added

Chart Logic Changed
```

---

Increase

```text
Chart Version
```

---

Example

```text
1.0.0

↓

1.1.0
```

---

# When Should App Version Change?

Suppose:

```text
New Docker Image
```

---

Example

```text
catalogue:4.0.0

↓

catalogue:5.0.0
```

---

Update

```yaml
appVersion: "5.0.0"
```

---

# Environment Management

Production environments usually have:

```text
Development

QA

UAT

Production
```

---

Each environment requires:

```text
Different Replicas

Different Resources

Different Configurations
```

---

# Problem

Same chart.

---

Different requirements.

---

Development

```text
1 Replica
```

---

Production

```text
5 Replicas
```

---

How do we manage this?

---

Solution

```text
Environment Specific Values Files
```

---

# Default Values File

```text
values.yaml
```

---

Contains:

```text
Common Values

Default Values
```

---

Example

```yaml
deployment:

  replicas: 2

service:

  port: 8080
```

---

# Development Values

File

```text
values-dev.yaml
```

---

Example

```yaml
deployment:

  replicas: 1

resources:

  cpu: 100m

  memory: 128Mi
```

---

Purpose

```text
Low Cost

Low Resources
```

---

# Production Values

File

```text
values-prod.yaml
```

---

Example

```yaml
deployment:

  replicas: 5

resources:

  cpu: 1000m

  memory: 2Gi
```

---

Purpose

```text
High Availability

High Performance
```

---

# Environment Structure

```text
values.yaml

values-dev.yaml

values-prod.yaml
```

---

Single Chart

Multiple Environments

---

# Deploy Development Environment

```bash
helm install catalogue . \
-f values-dev.yaml
```

---

Helm combines:

```text
values.yaml

+

values-dev.yaml
```

---

Final values deployed.

---

# Deploy Production Environment

```bash
helm install catalogue . \
-f values-prod.yaml
```

---

Result

```text
Production Configuration
```

---

# Environment Override Priority

Priority

```text
Highest

↓

values-prod.yaml

↓

values.yaml

↓

Templates
```

---

Environment file overrides default values.

---

# Production Example

Development

```yaml
deployment:

  replicas: 1
```

---

Production

```yaml
deployment:

  replicas: 5
```

---

Same chart.

---

Different environments.

---

# Helm Install

Install application.

---

Command

```bash
helm install mongodb .
```

---

Verify

```bash
helm list
```

---

# Helm Status

Check release details.

---

Command

```bash
helm status mongodb
```

---

Displays

```text
Release Name

Namespace

Revision

Status
```

---

# Helm History

View deployment history.

---

Command

```bash
helm history mongodb
```

---

Displays

```text
Revision History
```

---

Useful for:

```text
Rollback

Auditing

Troubleshooting
```

---

# Helm Upgrade

Upgrade release.

---

Command

```bash
helm upgrade mongodb .
```

---

Creates

```text
New Revision
```

---

# Upgrade With Description

Command

```bash
helm upgrade mongodb \
--description "Upgraded to version 5.0.0" .
```

---

Useful for:

```text
Release Documentation
```

---

# Upgrade and Install Together

Very common in CI/CD.

---

Command

```bash
helm upgrade --install aws-ebs-csi-driver \
--namespace kube-system \
aws-ebs-csi-driver/aws-ebs-csi-driver
```

---

Meaning

```text
If Exists

↓

Upgrade

If Not Exists

↓

Install
```

---

# Why Use Upgrade --Install?

CI/CD pipelines do not know:

```text
Application Exists?

Or

Application Missing?
```

---

Single command handles both.

---

# CI/CD Example

Pipeline

```text
Build Image
       │

       ▼

Push To Registry
       │

       ▼

helm upgrade --install
       │

       ▼

Deployment
```

---

No manual intervention.

---

# Production Deployment Strategy

Repository

```text
helm-chart
```

---

Contains

```text
Chart.yaml

values.yaml

values-dev.yaml

values-prod.yaml

templates
```

---

Deployment Flow

```text
Developer
      │

      ▼

Git Push
      │

      ▼

CI/CD
      │

      ▼

Choose Environment File
      │

      ▼

helm upgrade --install
      │

      ▼

Deployment
```

---

# Common Problems

## Wrong Environment Values

Cause

```text
Wrong Values File
```

---

Example

```text
Production

Using

values-dev.yaml
```

---

Verify

```bash
helm get values release-name
```

---

## Version Mismatch

Cause

```text
Chart Version

App Version

Confusion
```

---

Remember

```text
version

↓

Chart

appVersion

↓

Application
```

---

## Upgrade Not Reflecting

Cause

```text
Old Values

Cached Values
```

---

Verify

```bash
helm get values release-name
```

---

# Troubleshooting

Check Release

```bash
helm status release-name
```

---

Check History

```bash
helm history release-name
```

---

Check Values

```bash
helm get values release-name -o yaml
```

---

Render Templates

```bash
helm template .
```

---

# Common Interview Questions

## What is Semantic Versioning?

### Short Answer

Semantic versioning uses the format Major.Minor.Patch.

### Detailed Explanation

Major versions represent breaking changes, minor versions represent new features, and patch versions represent bug fixes.

### Production Example

```text
1.0.0

↓

1.1.0

↓

1.1.1
```

### Follow-Up Questions

- What causes major version changes?
- What causes patch version changes?
- Why use semantic versioning?

---

## Difference Between Version and AppVersion?

### Short Answer

Version is the Helm chart version, while appVersion is the application version.

### Detailed Explanation

Helm tracks chart versions and application teams track image versions.

### Production Example

```yaml
version: 1.0.0

appVersion: "5.0.0"
```

### Follow-Up Questions

- Which one changes when templates change?
- Which one changes when Docker image changes?
- Does Helm use appVersion for upgrades?

---

## Why Use Multiple Values Files?

### Short Answer

To support different environments using the same chart.

### Detailed Explanation

Different environments require different replicas, resources, and configurations.

### Production Example

Development uses one replica while production uses five replicas.

### Follow-Up Questions

- How are values overridden?
- Which file has higher priority?
- Can multiple values files be combined?

---

## What Does helm upgrade --install Do?

### Short Answer

Installs the application if missing and upgrades it if already deployed.

### Detailed Explanation

This command is widely used in CI/CD because it handles both installation and upgrades.

### Production Example

AWS EBS CSI Driver deployments often use helm upgrade --install.

### Follow-Up Questions

- Why is it useful in automation?
- Does it create revisions?
- Can it be used for first deployment?

---

# Key Takeaways

```text
Helm uses semantic versioning.

version refers to chart version.

appVersion refers to application version.

values.yaml contains default values.

Environment-specific values files support multiple environments.

values-dev.yaml and values-prod.yaml are common patterns.

helm status shows release details.

helm history shows revisions.

helm upgrade creates new revisions.

helm upgrade --install is commonly used in CI/CD pipelines.

Environment management is one of Helm's most important production features.
```