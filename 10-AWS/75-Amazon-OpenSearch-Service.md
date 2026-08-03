# Amazon OpenSearch Service

---

# Introduction

Amazon OpenSearch Service is a fully managed search and analytics service that makes it easy to deploy, operate, and scale OpenSearch clusters for log analytics, application monitoring, real-time search, and observability workloads.

Modern applications generate massive volumes of logs, metrics, and events. Organizations need a scalable platform to search, analyze, visualize, and monitor this data in near real time. Amazon OpenSearch Service provides managed clusters with built-in integrations for analytics and observability.

Amazon OpenSearch Service integrates with

- Amazon CloudWatch
- AWS Lambda
- Amazon S3
- Amazon Kinesis Data Firehose
- Amazon OpenSearch Dashboards
- AWS IAM
- AWS CloudTrail
- Amazon VPC
- AWS WAF
- Amazon EventBridge

It provides enterprise-grade search, log analytics, and visualization capabilities.

---

# What is Amazon OpenSearch Service?

Amazon OpenSearch Service is a managed search and analytics platform.

It helps organizations

- Search Application Data
- Analyze Logs
- Monitor Applications
- Build Dashboards
- Perform Real-Time Analytics

Workflow

```text
Applications

↓

Logs & Events

↓

Amazon OpenSearch Service

↓

Search

↓

Dashboards

↓

Operations Team
```

---

# Why Amazon OpenSearch Service?

Without OpenSearch

```text
Application Logs

↓

Multiple Servers

↓

Manual Analysis

↓

Slow Troubleshooting
```

Problems

- Difficult Log Analysis
- Slow Searches
- Limited Scalability
- Manual Infrastructure Management

With OpenSearch

```text
Logs

↓

Amazon OpenSearch Service

↓

Fast Search

↓

Visualization

↓

Insights
```

---

# Real World Problem Statement

An enterprise operates

- 500 Microservices
- Kubernetes Clusters
- APIs
- Web Applications
- Security Monitoring

Requirements

- Centralized Logging
- Fast Search
- Real-Time Dashboards
- Operational Insights

Amazon OpenSearch Service provides centralized search and analytics.

---

# Enterprise Architecture

```text
Applications

        │

Logs & Metrics

        │

        ▼

Kinesis Firehose

        │

        ▼

Amazon OpenSearch Service

        │

────────┼──────────────

│        │             │

Search Dashboards Analytics
```

---

# Core Components

Amazon OpenSearch Service consists of

- OpenSearch Cluster
- Nodes
- Index
- Documents
- Shards
- Replicas
- OpenSearch Dashboards
- Snapshots

---

# Cluster

A cluster is a collection of OpenSearch nodes.

Responsibilities

- Store Data
- Execute Searches
- Process Analytics
- Ensure High Availability

---

# Nodes

Nodes perform search and indexing operations.

Common Node Types

- Data Nodes
- Master Nodes
- Coordinating Nodes

Each node has a specific responsibility.

---

# Index

An Index stores related documents.

Example

```text
Application Logs

↓

logs-2026

↓

Millions of Documents
```

Similar to a database table.

---

# Document

A document is the basic unit of stored data.

Example

```json
{
  "application": "payment-service",
  "status": "SUCCESS",
  "responseTime": 145
}
```

Documents are stored in JSON format.

---

# Shards

Indexes are divided into shards.

Benefits

- Parallel Processing
- Better Performance
- Horizontal Scaling

Architecture

```text
Index

↓

Shard 1

Shard 2

Shard 3
```

---

# Replicas

Replica shards improve availability.

Benefits

- High Availability
- Fault Tolerance
- Faster Searches

Workflow

```text
Primary Shard

↓

Replica Shard
```

---

# OpenSearch Dashboards

OpenSearch Dashboards provides visualization capabilities.

Features

- Search Interface
- Charts
- Graphs
- Dashboards
- Monitoring

Used for operational visibility.

---

# Data Ingestion

Data can be ingested using

- Kinesis Data Firehose
- Logstash
- Fluent Bit
- Fluentd
- Beats
- AWS Lambda

---

# Snapshot

Snapshots back up OpenSearch indexes.

Stored in

- Amazon S3

Benefits

- Disaster Recovery
- Backup
- Migration

---

# Security

Security Features

- IAM Authentication
- Fine-Grained Access Control
- Encryption at Rest
- Encryption in Transit
- VPC Deployment
- CloudTrail Auditing

---

# Monitoring

Monitor using

- Amazon CloudWatch
- OpenSearch Dashboards
- CloudTrail

Metrics include

- CPU Utilization
- JVM Memory
- Storage
- Cluster Health
- Search Latency

---

# AWS CLI

Create Domain

```bash
aws opensearch create-domain
```

List Domains

```bash
aws opensearch list-domain-names
```

Describe Domain

```bash
aws opensearch describe-domain \
--domain-name production-search
```

---

# Terraform

```hcl
resource "aws_opensearch_domain" "logs" {

  domain_name = "production-logs"

}
```

---

# CloudFormation

```yaml
Resources:

  OpenSearchDomain:

    Type: AWS::OpenSearchService::Domain
```

---

# Python (Boto3)

```python
import boto3

client = boto3.client("opensearch")

response = client.list_domain_names()

print(response)
```

---

# Enterprise Production Architecture

```text
         Applications

              │

     CloudWatch Logs

              │

 Kinesis Data Firehose

              │

 Amazon OpenSearch Service

              │

 ┌────────────┼────────────┐

 │            │            │

Search   Dashboards   Analytics

              │

      Operations Team
```

---

# Best Practices

- Deploy clusters inside a VPC
- Enable encryption at rest
- Enable encryption in transit
- Configure fine-grained access control
- Use dedicated master nodes for production
- Enable automated snapshots
- Monitor cluster health using CloudWatch
- Size shards appropriately
- Enable replica shards
- Rotate indexes using lifecycle policies
- Follow least-privilege IAM policies
- Monitor storage utilization

---

# Common Mistakes

- Oversized shards
- Too many small shards
- No replica shards
- Public cluster access
- Ignoring snapshots
- Weak IAM permissions
- No monitoring
- Running out of storage
- Ignoring JVM memory usage
- Missing encryption

---

# Troubleshooting

## Cluster Status Red

Check

- Node Availability
- Replica Shards
- Disk Space
- Cluster Health

---

## Slow Search Performance

Verify

- Shard Size
- Query Complexity
- JVM Memory
- CPU Utilization

---

## Indexing Failed

Check

- Mapping
- Storage
- Permissions
- Data Format

---

## High JVM Memory

Verify

- Heap Usage
- Query Load
- Index Size
- Cluster Scaling

---

## Snapshot Failed

Check

- S3 Permissions
- Repository Configuration
- Storage Availability

---

# Interview Questions

## Basic

1. What is Amazon OpenSearch Service?
2. Why use OpenSearch?
3. What is an index?
4. What is a document?
5. What is a shard?
6. What is a replica?
7. What is OpenSearch Dashboards?
8. Which AWS services integrate with OpenSearch?
9. Where are snapshots stored?
10. What data format does OpenSearch use?

---

## Intermediate

11. Explain OpenSearch cluster architecture.
12. Explain shards vs replicas.
13. Explain indexing.
14. Explain search performance.
15. Explain OpenSearch Dashboards.
16. Explain Kinesis integration.
17. Explain security features.
18. Explain snapshot strategy.
19. Explain monitoring.
20. Explain enterprise logging architecture.

---

## Advanced

21. Design centralized logging using Amazon OpenSearch Service.
22. Explain OpenSearch vs Elasticsearch.
23. Design highly available search clusters.
24. Explain OpenSearch security architecture.
25. Design log analytics for Kubernetes.
26. Explain shard optimization.
27. Design enterprise observability.
28. Explain operational best practices.
29. Design large-scale search architecture.
30. Best practices for Amazon OpenSearch Service.

---

# Production Scenarios

### Scenario 1

A Kubernetes platform generates millions of application logs every day.

How would Amazon OpenSearch Service help operations teams analyze and search these logs?

---

### Scenario 2

Your OpenSearch cluster reports a **Red** health status.

Which components would you investigate first?

---

### Scenario 3

A security team needs centralized search for CloudTrail logs.

How would OpenSearch Service support this requirement?

---

### Scenario 4

A development team wants dashboards showing API latency, error rates, and application logs.

Which OpenSearch component would you use?

---

### Scenario 5

An organization wants to protect OpenSearch from unauthorized access.

Which security features should be enabled?

---

### Scenario 6

A company requires a highly available logging platform with backups, encryption, monitoring, and visualization.

How would Amazon OpenSearch Service satisfy these requirements?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Cluster | Search Infrastructure |
| Node | Processing Unit |
| Index | Collection of Documents |
| Document | JSON Data Record |
| Shard | Horizontal Partition |
| Replica | High Availability |
| OpenSearch Dashboards | Visualization |
| Snapshot | Backup |
| CloudWatch | Monitoring |
| Kinesis Firehose | Data Ingestion |

---

# Summary

Amazon OpenSearch Service is a fully managed search and analytics service that enables organizations to index, search, analyze, and visualize large volumes of structured and unstructured data. By providing managed clusters, indexes, shards, replicas, OpenSearch Dashboards, snapshots, encryption, CloudWatch monitoring, and integrations with Amazon Kinesis Data Firehose, AWS Lambda, and Amazon S3, OpenSearch Service helps organizations build scalable log analytics, observability, and enterprise search solutions.