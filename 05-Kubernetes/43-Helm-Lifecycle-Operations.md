# Helm Lifecycle Operations

## Introduction

After creating Helm Charts, the next step is managing the application lifecycle.

---

Common operations include:

```text
Install

Upgrade

History

Rollback

Get Values

Uninstall
```

---

These operations make Helm one of the most powerful tools in Kubernetes.

---

# Helm Release Lifecycle

```text
Chart
  │

  ▼

Install
  │

  ▼

Release Created
  │

  ▼

Upgrade
  │

  ▼

New Revision
  │

  ▼

Rollback
  │

  ▼

Previous Revision
  │

  ▼

Uninstall
```

---

# What is a Release?

When a chart is installed:

```text
Helm Creates

A Release
```

---

Example

```bash
helm install catalogue .
```

---

Release Name

```text
catalogue
```

---

A release is:

```text
Running Instance

Of A Chart
```

---

# Install Chart

Command

```bash
helm install <release-name> .
```

---

Example

```bash
helm install catalogue .
```

---

Helm will:

```text
Render Templates

Generate YAML

Deploy Resources

Create Release
```

---

# Verify Installation

List Releases

```bash
helm list
```

---

Example

```text
NAME

catalogue
```

---

View Kubernetes Resources

```bash
kubectl get all
```

---

# Install In Namespace

```bash
helm install catalogue . \
-n roboshop
```

---

If namespace does not exist:

```bash
kubectl create namespace roboshop
```

---

# Upgrade Release

One of Helm's most powerful features.

---

Current Version

```text
catalogue:4.0.0
```

---

New Version

```text
catalogue:5.0.0
```

---

Update

```yaml
deployment:

  imageVersion: "5.0.0"
```

---

Deploy

```bash
helm upgrade catalogue .
```

---

Helm creates:

```text
New Revision
```

---

# Upgrade With Values

Example

```bash
helm upgrade catalogue . \
--set deployment.imageVersion=5.0.0
```

---

Helm overrides:

```yaml
values.yaml
```

temporarily.

---

# Production Upgrade Example

Current State

```text
catalogue:4.0.0
```

---

CI/CD Pipeline Builds

```text
catalogue:5.0.0
```

---

Pipeline Executes

```bash
helm upgrade catalogue . \
--set deployment.imageVersion=5.0.0
```

---

Application Updated

Without Changing Templates.

---

# View Release History

Command

```bash
helm history catalogue
```

---

Example Output

```text
REVISION

1

2

3
```

---

Shows

```text
Release Changes

Deployment History
```

---

# Why History Is Important?

Allows:

```text
Auditing

Troubleshooting

Rollback
```

---

# View Release Details

Command

```bash
helm status catalogue
```

---

Displays

```text
Current Revision

Namespace

Resources

Status
```

---

# View Installed Values

Command

```bash
helm get values catalogue
```

---

YAML Format

```bash
helm get values catalogue -o yaml
```

---

Useful for:

```text
Debugging

Verification

Auditing
```

---

# Example Output

```yaml
deployment:

  imageVersion: 5.0.0

  replicas: 3
```

---

# Rollback

One of Helm's most important production features.

---

Suppose

```text
catalogue:5.0.0
```

contains bugs.

---

Users report:

```text
Application Errors
```

---

Solution

```text
Rollback
```

---

# View History

```bash
helm history catalogue
```

---

Example

```text
REVISION

1

2

3
```

---

Current Revision

```text
3
```

---

Previous Stable Revision

```text
2
```

---

# Rollback Command

```bash
helm rollback catalogue 2
```

---

Meaning

```text
Restore Revision 2
```

---

# Rollback Flow

```text
Revision 1

catalogue:4.0.0
```

---

```text
Revision 2

catalogue:4.1.0
```

---

```text
Revision 3

catalogue:5.0.0
```

---

Issue Found

---

Rollback

```bash
helm rollback catalogue 2
```

---

Result

```text
catalogue:4.1.0
```

---

Restored.

---

# Why Rollback Is Powerful?

Without Helm

```text
Find Old YAML

Restore YAML

Deploy YAML
```

---

With Helm

```bash
helm rollback
```

---

Single Command.

---

# Uninstall Release

Remove Application

```bash
helm uninstall catalogue
```

---

Helm removes:

```text
Deployment

Service

ConfigMap

Secret

Ingress
```

created by release.

---

# Verify Removal

```bash
helm list
```

---

Release disappears.

---

Check Resources

```bash
kubectl get all
```

---

# Upgrade vs Rollback

| Operation | Purpose |
|------------|------------|
| Upgrade | Move Forward |
| Rollback | Move Backward |

---

Example

```text
4.0.0 → 5.0.0
```

---

Upgrade

---

Example

```text
5.0.0 → 4.0.0
```

---

Rollback

---

# Helm Commands Summary

Install

```bash
helm install catalogue .
```

---

List Releases

```bash
helm list
```

---

Upgrade

```bash
helm upgrade catalogue .
```

---

History

```bash
helm history catalogue
```

---

Status

```bash
helm status catalogue
```

---

Get Values

```bash
helm get values catalogue -o yaml
```

---

Rollback

```bash
helm rollback catalogue 2
```

---

Uninstall

```bash
helm uninstall catalogue
```

---

# CI/CD Integration

Typical Pipeline

```text
Build Image
      │

      ▼

Push To Registry
      │

      ▼

Update Image Tag
      │

      ▼

helm upgrade
      │

      ▼

Deployment
```

---

# Production Example

Application

```text
Catalogue Service
```

---

Current Version

```text
4.0.0
```

---

New Deployment

```text
5.0.0
```

---

Issue Detected

```text
Checkout Failures

API Errors
```

---

Action

```bash
helm rollback catalogue 1
```

---

Application Restored

Within Seconds.

---

# Common Problems

## Upgrade Failed

Cause

```text
Invalid Values

Template Errors
```

---

Verify

```bash
helm template .
```

---

## Rollback Failed

Cause

```text
Revision Not Found
```

---

Check

```bash
helm history release-name
```

---

## Release Not Visible

Cause

```text
Wrong Namespace
```

---

Check

```bash
helm list -A
```

---

# Troubleshooting

List Releases

```bash
helm list -A
```

---

Check Status

```bash
helm status release-name
```

---

Check History

```bash
helm history release-name
```

---

View Values

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

## What is a Helm Release?

### Short Answer

A Helm Release is a deployed instance of a Helm Chart.

### Detailed Explanation

Every Helm installation creates a release that Helm tracks and manages.

### Production Example

Installing the catalogue chart creates a release named catalogue.

### Follow-Up Questions

- Can multiple releases use the same chart?
- How does Helm track releases?
- What happens during uninstall?

---

## Difference Between Install and Upgrade?

### Short Answer

Install creates a new release, while Upgrade updates an existing release.

### Detailed Explanation

Install deploys an application for the first time, whereas Upgrade modifies an existing deployment.

### Production Example

Catalogue version 4.0.0 is upgraded to 5.0.0 using helm upgrade.

### Follow-Up Questions

- Does upgrade create a revision?
- Can upgrade change values?
- Can upgrade change image versions?

---

## What is Helm Rollback?

### Short Answer

Rollback restores a previous release revision.

### Detailed Explanation

Helm maintains release history and allows reverting to a known working version.

### Production Example

Catalogue 5.0.0 is rolled back to 4.0.0 after production issues.

### Follow-Up Questions

- How do you check revisions?
- What command performs rollback?
- Why is rollback important?

---

## What Does helm history Show?

### Short Answer

It displays release revisions and deployment history.

### Detailed Explanation

Helm stores every release revision, allowing auditing and rollback.

### Production Example

A DevOps engineer identifies the last stable release before performing a rollback.

### Follow-Up Questions

- What is a revision?
- How do you rollback to a revision?
- Is history preserved after upgrades?

---

## What Does helm get values Do?

### Short Answer

It displays values currently used by a release.

### Detailed Explanation

Helm retrieves the configuration that was used during deployment.

### Production Example

An engineer verifies the deployed image version before troubleshooting.

### Follow-Up Questions

- Can values be exported?
- How do you display YAML output?
- Why is this useful in production?

---

# Key Takeaways

```text
Helm Install creates a release.

Helm Upgrade updates an existing release.

Every upgrade creates a new revision.

Helm History tracks release revisions.

Helm Rollback restores previous revisions.

Helm Get Values displays deployment configuration.

Helm Status shows release health.

Helm Uninstall removes releases and resources.

Rollback is one of Helm's most important production features.

Helm lifecycle operations are heavily used in CI/CD pipelines.
```