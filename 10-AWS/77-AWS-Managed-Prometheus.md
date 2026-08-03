# AWS Managed Prometheus

---

# Introduction

Amazon Managed Service for Prometheus (AMP) is a fully managed, Prometheus-compatible monitoring service that enables organizations to securely collect, store, query, and analyze metrics from applications, containers, Kubernetes clusters, and cloud infrastructure at scale.

Prometheus is the industry standard for collecting time-series metrics, especially in Kubernetes environments. Managing Prometheus servers at scale can be operationally complex due to storage, scaling, and availability requirements. Amazon Managed Service for Prometheus eliminates these operational challenges while remaining fully compatible with the open-source Prometheus ecosystem.

Amazon Managed Service for Prometheus integrates with

- Amazon EKS
- Kubernetes
- Amazon Managed Grafana
- Amazon CloudWatch
- AWS IAM
- AWS Distro for OpenTelemetry (ADOT)
- AWS Lambda
- Amazon EC2
- Amazon ECS
- AWS X-Ray

It provides scalable, highly available, and secure metrics storage for cloud-native applications.

---

# What is Amazon Managed Service for Prometheus?

Amazon Managed Service for Prometheus is a managed time-series metrics service.

It helps organizations

- Collect Metrics
- Store Time-Series Data
- Monitor Kubernetes
- Visualize Infrastructure
- Build Observability Platforms

Workflow

```text
Applications

↓

Metrics

↓

Prometheus

↓

Amazon Managed Service for Prometheus

↓

Grafana Dashboards

↓

Operations Team
```

---

# Why Amazon Managed Service for Prometheus?

Without Managed Prometheus

```text
Prometheus Server

↓

Storage Management

↓

Scaling Issues

↓

Operational Overhead
```

Problems

- Infrastructure Management
- Limited Scalability
- Storage Maintenance
- High Availability Challenges

With Managed Prometheus

```text
Metrics

↓

Managed Service

↓

Automatic Scaling

↓

Reliable Monitoring
```

---

# Real World Problem Statement

A company operates

- 300 Kubernetes Clusters
- Thousands of Containers
- Microservices
- APIs
- Databases

Requirements

- Centralized Metrics
- Long-Term Storage
- Kubernetes Monitoring
- High Availability
- Enterprise Observability

Amazon Managed Service for Prometheus provides scalable metrics management.

---

# Enterprise Architecture

```text
Applications

Containers

Amazon EKS

       │

Prometheus Metrics

       │

AWS Distro for OpenTelemetry

       │

Amazon Managed Service for Prometheus

       │

Amazon Managed Grafana

       │

Operations Team
```

---

# Core Components

Amazon Managed Service for Prometheus consists of

- Workspace
- Time-Series Database
- Metrics
- Labels
- PromQL
- Alert Manager Integration
- Remote Write API

---

# Workspace

A Workspace stores Prometheus metrics.

Each workspace contains

- Metrics
- Queries
- Permissions

Organizations can create multiple workspaces.

---

# Time-Series Metrics

Prometheus stores time-series data.

Example

```text
Timestamp

↓

CPU Utilization

↓

Memory Usage

↓

Request Rate
```

Each metric contains values over time.

---

# Metrics

Common metrics include

- CPU Usage
- Memory Usage
- Network Traffic
- Disk Utilization
- Request Count
- Response Time

Metrics are continuously collected.

---

# Labels

Labels provide metadata for metrics.

Example

```text
cpu_usage

environment=production

service=payment

region=ap-south-1
```

Labels enable efficient filtering.

---

# PromQL

PromQL is the Prometheus Query Language.

Example

```text
rate(http_requests_total[5m])
```

PromQL supports

- Aggregation
- Filtering
- Mathematical Operations
- Time-Based Analysis

---

# Remote Write

Prometheus servers send metrics using the Remote Write API.

Workflow

```text
Prometheus

↓

Remote Write

↓

Amazon Managed Service for Prometheus

↓

Metrics Storage
```

---

# AWS Distro for OpenTelemetry (ADOT)

ADOT collects metrics from

- Kubernetes
- Applications
- Containers
- Infrastructure

Workflow

```text
Applications

↓

ADOT Collector

↓

Managed Prometheus
```

---

# Grafana Integration

Amazon Managed Grafana connects directly to Managed Prometheus.

Workflow

```text
Metrics

↓

Managed Prometheus

↓

Grafana

↓

Dashboards
```

---

# Monitoring Kubernetes

Managed Prometheus is commonly used with

- Amazon EKS
- Kubernetes Nodes
- Pods
- Deployments
- Services
- Ingress Controllers

---

# Security

Security Features

- IAM Authentication
- Encryption at Rest
- Encryption in Transit
- CloudTrail Logging
- Fine-Grained Access Control

---

# AWS CLI

Create Workspace

```bash
aws amp create-workspace
```

List Workspaces

```bash
aws amp list-workspaces
```

Describe Workspace

```bash
aws amp describe-workspace \
--workspace-id <workspace-id>
```

---

# Terraform

```hcl
resource "aws_prometheus_workspace" "monitoring" {

  alias = "production-monitoring"

}
```

---

# CloudFormation

```yaml
Resources:

  PrometheusWorkspace:

    Type: AWS::APS::Workspace
```

---

# Python (Boto3)

```python
import boto3

amp = boto3.client("amp")

response = amp.list_workspaces()

print(response)
```

---

# Enterprise Production Architecture

```text
       Applications

      Amazon EKS

          │

 ADOT Collector

          │

Amazon Managed Service

   for Prometheus

          │

 Amazon Managed Grafana

          │

 Dashboards • Alerts

          │

   Operations Team
```

---

# Best Practices

- Use Managed Prometheus for Kubernetes monitoring
- Use AWS Distro for OpenTelemetry
- Organize metrics using labels
- Follow consistent metric naming conventions
- Integrate with Amazon Managed Grafana
- Enable IAM authentication
- Encrypt metrics in transit
- Enable CloudTrail auditing
- Monitor workspace usage
- Retain only required metrics
- Use meaningful PromQL queries
- Monitor ingestion rates

---

# Common Mistakes

- High-cardinality labels
- Poor metric naming
- Collecting unnecessary metrics
- Weak IAM permissions
- No Grafana dashboards
- Ignoring metric retention
- Missing alert rules
- Excessive PromQL queries
- No monitoring documentation
- Missing encryption

---

# Troubleshooting

## Metrics Not Appearing

Check

- Remote Write Configuration
- ADOT Collector
- Workspace ID
- IAM Permissions

---

## Grafana Cannot Query Metrics

Verify

- Workspace Connection
- IAM Permissions
- Data Source Configuration

---

## High Metric Cardinality

Check

- Labels
- Metric Design
- Collection Strategy

---

## PromQL Query Failed

Verify

- Metric Name
- Query Syntax
- Time Range
- Workspace

---

## High Ingestion Rate

Check

- Metric Volume
- Scrape Configuration
- Label Usage

---

# Interview Questions

## Basic

1. What is Amazon Managed Service for Prometheus?
2. Why use Managed Prometheus?
3. What is a workspace?
4. What are time-series metrics?
5. What are labels?
6. What is PromQL?
7. What is Remote Write?
8. Which AWS services integrate with Managed Prometheus?
9. What is ADOT?
10. How does Grafana integrate with Prometheus?

---

## Intermediate

11. Explain Prometheus architecture.
12. Explain Remote Write.
13. Explain labels.
14. Explain metric collection.
15. Explain Grafana integration.
16. Explain Kubernetes monitoring.
17. Explain ADOT.
18. Explain IAM security.
19. Explain workspace management.
20. Explain enterprise monitoring.

---

## Advanced

21. Design Kubernetes observability using Managed Prometheus.
22. Explain Managed Prometheus vs self-managed Prometheus.
23. Design enterprise metrics architecture.
24. Explain PromQL optimization.
25. Design highly available monitoring.
26. Explain operational best practices.
27. Design centralized observability.
28. Explain metric governance.
29. Design monitoring for microservices.
30. Best practices for Amazon Managed Service for Prometheus.

---

# Production Scenarios

### Scenario 1

Your Amazon EKS cluster hosts hundreds of microservices.

How would Amazon Managed Service for Prometheus help collect and store Kubernetes metrics?

---

### Scenario 2

An operations team wants dashboards showing CPU, memory, request rate, and latency for Kubernetes workloads.

How would Managed Prometheus integrate with Amazon Managed Grafana?

---

### Scenario 3

A monitoring system experiences extremely high metric cardinality.

What design changes would you recommend?

---

### Scenario 4

A company wants centralized metrics collection from multiple AWS accounts and Kubernetes clusters.

How would you design the monitoring architecture?

---

### Scenario 5

A Grafana dashboard displays no Prometheus metrics.

Which components would you troubleshoot first?

---

### Scenario 6

An enterprise requires scalable metrics collection, IAM authentication, encryption, CloudTrail auditing, and Kubernetes integration.

How does Amazon Managed Service for Prometheus satisfy these requirements?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Workspace | Metrics Storage |
| Time-Series Data | Historical Metrics |
| Labels | Metric Metadata |
| PromQL | Query Language |
| Remote Write | Metric Ingestion |
| ADOT | Metrics Collection |
| Amazon EKS | Kubernetes Monitoring |
| Amazon Managed Grafana | Visualization |
| CloudTrail | Audit Logging |
| IAM | Access Control |

---

# Summary

Amazon Managed Service for Prometheus (AMP) is a fully managed, Prometheus-compatible monitoring service that enables organizations to collect, store, query, and analyze time-series metrics at scale. Through managed workspaces, PromQL, Remote Write, AWS Distro for OpenTelemetry (ADOT), IAM security, CloudTrail auditing, and seamless integration with Amazon Managed Grafana and Amazon EKS, AMP provides a scalable and highly available foundation for enterprise observability and Kubernetes monitoring.