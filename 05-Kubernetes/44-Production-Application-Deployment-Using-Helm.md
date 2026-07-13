# Production Application Deployment Using Helm

## Introduction

In production environments, Helm is not used only for:

```text
Install

Upgrade

Rollback
```

---

It is used to package and deploy:

```text
Microservices

Platform Components

Monitoring Stack

Ingress Controllers

Storage Drivers
```

---

A typical application contains:

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

Helm helps package all of them into:

```text
A Single Deployable Unit
```

called a:

```text
Chart
```

---

# Production Scenario

Application

```text
Catalogue Service
```

---

Dependencies

```text
Deployment

Service

ConfigMap
```

---

Image

```text
catalogue:4.0.0
```

---

Requirement

```text
Deploy

Upgrade

Rollback

Manage Versions
```

using Helm.

---

# Chart Structure

```text
catalogue/

├── Chart.yaml

├── values.yaml

└── templates

    ├── deployment.yaml

    ├── service.yaml

    └── configmap.yaml
```

---

# Chart.yaml

Stores chart metadata.

---

```yaml
apiVersion: v2

name: catalogue

description: Roboshop Catalogue Service

type: application

version: 1.0.0

appVersion: "4.0.0"
```

---

Fields

```text
name

description

chart version

application version
```

---

# values.yaml

Stores configurable values.

---

```yaml
deployment:

  replicas: 2

  image:

    repository: 123456789.dkr.ecr.us-east-1.amazonaws.com/catalogue

    tag: "4.0.0"

service:

  port: 8080

config:

  MONGO_URL: mongodb://mongodb:27017/catalogue
```

---

Purpose

```text
Environment Specific Configuration
```

---

# ConfigMap Template

File

```text
templates/configmap.yaml
```

---

```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: {{ .Release.Name }}

data:

  MONGO_URL: {{ .Values.config.MONGO_URL | quote }}
```

---

Rendered Output

```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: catalogue

data:

  MONGO_URL: mongodb://mongodb:27017/catalogue
```

---

# Service Template

File

```text
templates/service.yaml
```

---

```yaml
apiVersion: v1

kind: Service

metadata:

  name: {{ .Release.Name }}

spec:

  selector:

    app: {{ .Release.Name }}

  ports:

  - port: {{ .Values.service.port }}

    targetPort: 8080

  type: ClusterIP
```

---

Rendered Output

```yaml
kind: Service

metadata:

  name: catalogue

spec:

  selector:

    app: catalogue

  ports:

  - port: 8080

    targetPort: 8080

  type: ClusterIP
```

---

# Deployment Template

File

```text
templates/deployment.yaml
```

---

```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: {{ .Release.Name }}

spec:

  replicas: {{ .Values.deployment.replicas }}

  selector:

    matchLabels:

      app: {{ .Release.Name }}

  template:

    metadata:

      labels:

        app: {{ .Release.Name }}

    spec:

      containers:

      - name: catalogue

        image: "{{ .Values.deployment.image.repository }}:{{ .Values.deployment.image.tag }}"

        ports:

        - containerPort: 8080

        envFrom:

        - configMapRef:

            name: {{ .Release.Name }}
```

---

# Deployment Rendering

Values

```yaml
deployment:

  replicas: 2

  image:

    tag: "4.0.0"
```

---

Rendered Output

```yaml
replicas: 2

image: 123456789.dkr.ecr.us-east-1.amazonaws.com/catalogue:4.0.0
```

---

# Validate Chart

Before deployment validate chart.

---

Command

```bash
helm lint .
```

---

Purpose

```text
Validate Chart Structure

Find Template Errors
```

---

# Render Templates

Preview generated manifests.

---

Command

```bash
helm template .
```

---

Purpose

```text
Verify Generated YAML
```

---

# Install Application

Deploy application.

---

Command

```bash
helm install catalogue .
```

---

Result

```text
Release Created

Deployment Created

Service Created

ConfigMap Created
```

---

# Verify Installation

Check Release

```bash
helm list
```

---

Check Resources

```bash
kubectl get all
```

---

Check Pods

```bash
kubectl get pods
```

---

# Production Upgrade

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

```bash
helm upgrade catalogue . \
--set deployment.image.tag=5.0.0
```

---

Helm Creates

```text
New Revision
```

---

Deployment Performs

```text
Rolling Update
```

---

# Upgrade Flow

```text
Build New Image
        │

        ▼

Push To ECR
        │

        ▼

Update Helm Value
        │

        ▼

helm upgrade
        │

        ▼

Rolling Deployment
```

---

# Verify Upgrade

Check History

```bash
helm history catalogue
```

---

Example

```text
REVISION

1

2
```

---

Check Running Image

```bash
kubectl describe deployment catalogue
```

---

# Rollback Scenario

Suppose

```text
catalogue:5.0.0
```

contains a bug.

---

Users report

```text
API Failures

Checkout Issues

Application Errors
```

---

Check History

```bash
helm history catalogue
```

---

Output

```text
REVISION

1

2
```

---

Rollback

```bash
helm rollback catalogue 1
```

---

Result

```text
catalogue:4.0.0
```

restored.

---

# Uninstall Application

Remove release.

---

Command

```bash
helm uninstall catalogue
```

---

Helm removes

```text
Deployment

Service

ConfigMap
```

created by release.

---

# CI/CD Integration

Typical Production Flow

```text
Developer
      │

      ▼

Git Push
      │

      ▼

GitHub Actions / Jenkins
      │

      ▼

Build Docker Image
      │

      ▼

Push To ECR
      │

      ▼

helm upgrade
      │

      ▼

Kubernetes Deployment
```

---

# Real Production Example

Developer fixes a bug.

---

New Image

```text
catalogue:5.0.0
```

---

Pipeline Builds

```text
Docker Image
```

---

Pipeline Pushes

```text
ECR Repository
```

---

Pipeline Executes

```bash
helm upgrade catalogue . \
--set deployment.image.tag=5.0.0
```

---

Application Updated.

---

If issue occurs

```bash
helm rollback catalogue 1
```

---

Application Restored.

---

# Helm Deployment Lifecycle

```text
Chart.yaml
      │

values.yaml
      │

Templates
      │

helm install
      │

Release
      │

Deployment
      │

Running Application
      │

helm upgrade
      │

New Revision
      │

helm rollback
      │

Previous Revision
```

---

# Common Problems

## Upgrade Failed

Cause

```text
Invalid Values

Template Errors
```

---

Check

```bash
helm lint .
```

---

## Wrong Image Version

Cause

```text
Incorrect Values
```

---

Verify

```bash
helm get values catalogue -o yaml
```

---

## Rollback Failed

Cause

```text
Revision Does Not Exist
```

---

Verify

```bash
helm history catalogue
```

---

# Troubleshooting

Check Releases

```bash
helm list
```

---

Check History

```bash
helm history catalogue
```

---

Check Values

```bash
helm get values catalogue -o yaml
```

---

Check Deployment

```bash
kubectl describe deployment catalogue
```

---

Check Pods

```bash
kubectl get pods
```

---

# Common Interview Questions

## How is Helm used in production?

### Short Answer

Helm packages Kubernetes resources into charts and manages deployments through install, upgrade, rollback, and uninstall operations.

### Detailed Explanation

Applications are packaged as charts containing Deployment, Service, ConfigMap, Secret, Ingress, and other resources. CI/CD pipelines deploy new versions using helm upgrade.

### Production Example

Catalogue service is deployed using a Helm chart and upgraded from 4.0.0 to 5.0.0 using helm upgrade.

### Follow-Up Questions

- What is a chart?
- What is a release?
- How does rollback work?

---

## Why use values.yaml?

### Short Answer

To separate configuration from templates.

### Detailed Explanation

Values allow different environments to use the same chart with different settings.

### Production Example

Development uses one replica while production uses five replicas.

### Follow-Up Questions

- Can values be overridden?
- What is --set?
- Can multiple values files exist?

---

## How do CI/CD pipelines deploy applications using Helm?

### Short Answer

Pipelines build images, push them to a registry, and execute helm upgrade.

### Detailed Explanation

Helm templates remain unchanged while image tags and environment values are updated dynamically.

### Production Example

GitHub Actions builds catalogue:5.0.0 and deploys it using helm upgrade.

### Follow-Up Questions

- Why use Helm in CI/CD?
- What is rolling update?
- How is rollback performed?

---

## What happens during helm upgrade?

### Short Answer

Helm creates a new release revision and updates Kubernetes resources.

### Detailed Explanation

Helm renders templates with new values and applies changes to the cluster.

### Production Example

Catalogue version changes from 4.0.0 to 5.0.0.

### Follow-Up Questions

- Does upgrade create a revision?
- How do you verify upgrades?
- Can upgrades be rolled back?

---

# Key Takeaways

```text
Helm Charts package Kubernetes applications.

Chart.yaml stores metadata.

values.yaml stores configuration.

Templates generate Kubernetes manifests.

helm install deploys applications.

helm upgrade updates applications.

helm rollback restores previous revisions.

helm uninstall removes releases.

CI/CD pipelines commonly use helm upgrade.

Helm is one of the most widely used deployment mechanisms in Kubernetes.
```