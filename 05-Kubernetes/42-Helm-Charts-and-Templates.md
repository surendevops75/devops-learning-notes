# Helm Charts and Templates

## Introduction

In the previous chapter we learned:

```text
Helm

Charts

Repositories

Releases

Values
```

---

Now let's understand:

```text
How Helm Charts Work

How Templates Work

How Values Are Injected
```

---

This is the most important Helm topic for:

```text
DevOps Engineers

Platform Engineers

Kubernetes Administrators
```

---

# Problem Without Helm

Suppose we have:

```text
Development

QA

Production
```

---

Application Image

```text
catalogue:4.0.0
```

---

Deployment YAML

```yaml
image: catalogue:4.0.0
```

---

New Release

```text
catalogue:5.0.0
```

---

Without Helm

```text
Open YAML

Modify YAML

Commit YAML

Deploy YAML
```

---

For multiple environments:

```text
More Complexity

More Duplication

More Errors
```

---

# Helm Solution

Helm allows:

```text
Templating

Parameterization

Reusable Deployments
```

---

Instead of hardcoding:

```yaml
image: catalogue:4.0.0
```

---

Use:

```yaml
image: catalogue:{{ .Values.deployment.imageVersion }}
```

---

Now image version comes from:

```text
values.yaml
```

---

# What is a Helm Chart?

A Helm Chart is:

```text
Collection Of Files

Used To Deploy

Kubernetes Applications
```

---

Think of it as:

```text
Application Package
```

---

Contains

```text
Deployment

Service

ConfigMap

Secret

Ingress

HPA

PVC
```

---

# Chart Structure

Example

```text
catalogue-chart/

├── Chart.yaml

├── values.yaml

├── templates/

│   ├── deployment.yaml

│   ├── service.yaml

│   ├── configmap.yaml

│   └── ingress.yaml
```

---

# Chart.yaml

Chart metadata file.

---

Example

```yaml
apiVersion: v2

name: catalogue

description: Catalogue Service

version: 1.0.0

appVersion: "4.0.0"
```

---

Purpose

```text
Chart Information

Version Information
```

---

# values.yaml

Stores configurable values.

---

Example

```yaml
deployment:

  imageVersion: "4.0.0"

  replicas: 2

service:

  port: 8080
```

---

Purpose

```text
Environment Specific Configuration
```

---

# templates Directory

Contains:

```text
Deployment

Service

ConfigMap

Ingress

Secrets
```

---

These files contain:

```text
Helm Variables

Helm Functions

Helm Templates
```

---

# Template Example

Normal Kubernetes Deployment

```yaml
containers:

- image: catalogue:4.0.0
```

---

Helm Template

```yaml
containers:

- image: catalogue:{{ .Values.deployment.imageVersion }}
```

---

Meaning

```text
Get Image Version

From values.yaml
```

---

# How Helm Rendering Works

Input

```text
Template File

+

values.yaml
```

---

Output

```text
Valid Kubernetes YAML
```

---

Flow

```text
Template
      │

      ▼

Values

      │

      ▼

Helm Rendering

      │

      ▼

Final Manifest

      │

      ▼

Kubernetes API Server
```

---

# Example

values.yaml

```yaml
deployment:

  imageVersion: "5.0.0"
```

---

Template

```yaml
image: catalogue:{{ .Values.deployment.imageVersion }}
```

---

Rendered Output

```yaml
image: catalogue:5.0.0
```

---

# Common Template Variables

## Release Name

```yaml
{{ .Release.Name }}
```

---

Example

```text
catalogue
```

---

Used for

```text
Resource Naming
```

---

## Namespace

```yaml
{{ .Release.Namespace }}
```

---

Example

```text
roboshop
```

---

## Chart Name

```yaml
{{ .Chart.Name }}
```

---

Example

```text
catalogue
```

---

## Values

```yaml
{{ .Values }}
```

---

Most commonly used.

---

# Deployment Template Example

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: {{ .Release.Name }}

spec:

  replicas: {{ .Values.deployment.replicas }}

  selector:

    matchLabels:

      app: catalogue

  template:

    metadata:

      labels:

        app: catalogue

    spec:

      containers:

      - name: catalogue

        image: catalogue:{{ .Values.deployment.imageVersion }}
```

---

# Corresponding Values File

```yaml
deployment:

  replicas: 2

  imageVersion: "4.0.0"
```

---

Rendered Output

```yaml
replicas: 2

image: catalogue:4.0.0
```

---

# ConfigMap Templating

values.yaml

```yaml
config:

  env: dev

  logLevel: debug
```

---

Template

```yaml
data:

  ENV: {{ .Values.config.env }}

  LOG_LEVEL: {{ .Values.config.logLevel }}
```

---

Rendered Output

```yaml
ENV: dev

LOG_LEVEL: debug
```

---

# Environment Management

One Chart

```text
Multiple Environments
```

---

Development

```yaml
imageVersion: 4.0.0

replicas: 1
```

---

Production

```yaml
imageVersion: 5.0.0

replicas: 5
```

---

Same Chart.

---

Different Values.

---

# Why Templates Are Powerful

Without Templates

```text
Multiple YAML Files

Duplicate Code

Manual Updates
```

---

With Templates

```text
Reusable

Maintainable

Scalable
```

---

# Helm Template Validation

Render templates locally.

---

Command

```bash
helm template .
```

---

Purpose

```text
Preview Generated YAML
```

before deployment.

---

# Validate Chart

Command

```bash
helm lint .
```

---

Purpose

```text
Check Chart Errors
```

---

# Production Example

Current Version

```text
catalogue:4.0.0
```

---

New Release

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

No Deployment YAML modification required.

---

# Helm Template Best Practices

## Keep Values In values.yaml

Good

```yaml
replicas: 3
```

---

Bad

```yaml
replicas: 3
```

hardcoded inside template.

---

## Reuse Templates

Avoid duplication.

---

## Parameterize Environment Values

Examples

```text
Replicas

Images

Ports

CPU

Memory
```

---

# Common Problems

## Variable Not Found

Cause

```text
Wrong Values Path
```

---

Example

```yaml
.Values.imageVersion
```

instead of

```yaml
.Values.deployment.imageVersion
```

---

## Invalid YAML

Cause

```text
Template Syntax Error
```

---

Check

```bash
helm lint .
```

---

## Wrong Rendered Output

Cause

```text
Incorrect Values
```

---

Verify

```bash
helm template .
```

---

# Troubleshooting

Render Templates

```bash
helm template .
```

---

Validate Chart

```bash
helm lint .
```

---

Check Values

```bash
cat values.yaml
```

---

Inspect Chart

```bash
tree .
```

---

# Common Interview Questions

## What is a Helm Chart?

### Short Answer

A Helm Chart is a package containing Kubernetes resources and templates.

### Detailed Explanation

Charts package Deployments, Services, ConfigMaps, Secrets, and other Kubernetes resources into reusable units.

### Production Example

The Catalogue service is deployed using a chart containing Deployment, Service, and ConfigMap templates.

### Follow-Up Questions

- What files exist inside a chart?
- What is Chart.yaml?
- What is values.yaml?

---

## What is values.yaml?

### Short Answer

values.yaml stores configurable values used by Helm templates.

### Detailed Explanation

Instead of hardcoding values, Helm templates read values from values.yaml during rendering.

### Production Example

Image versions and replica counts are managed through values.yaml.

### Follow-Up Questions

- Can values be overridden?
- What happens if a value is missing?
- Why use values.yaml?

---

## What is Helm Templating?

### Short Answer

Helm templating dynamically generates Kubernetes manifests using values.

### Detailed Explanation

Templates contain placeholders that Helm replaces using values from values.yaml during chart rendering.

### Production Example

Image version changes from 4.0.0 to 5.0.0 without modifying Deployment YAML.

### Follow-Up Questions

- How does Helm render templates?
- What command shows rendered output?
- Which file provides values?

---

## Difference Between Chart.yaml and values.yaml?

### Short Answer

Chart.yaml contains chart metadata, while values.yaml contains configurable values.

### Detailed Explanation

Chart.yaml describes the chart itself, whereas values.yaml controls deployment behavior.

### Production Example

Chart version is stored in Chart.yaml and image version is stored in values.yaml.

### Follow-Up Questions

- Which file stores replicas?
- Which file stores chart version?
- Can multiple values files exist?

---

## What Does helm template Do?

### Short Answer

It renders templates locally without deploying resources.

### Detailed Explanation

Helm combines templates and values to generate final Kubernetes manifests.

### Production Example

Engineers verify generated manifests before production deployments.

### Follow-Up Questions

- Does helm template deploy resources?
- How is it used for debugging?
- What is helm lint?

---

# Key Takeaways

```text
Helm Charts package Kubernetes applications.

Chart.yaml contains metadata.

values.yaml contains configurable values.

Templates generate Kubernetes manifests.

Helm replaces variables during rendering.

One chart can support multiple environments.

helm template previews generated manifests.

helm lint validates chart structure.

Templating reduces duplication and improves maintainability.

Helm templating is one of the most important production Helm concepts.
```