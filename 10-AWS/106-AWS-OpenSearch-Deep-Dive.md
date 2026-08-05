# AWS OpenSearch (Deep Dive)

# Chapter 1 - AWS OpenSearch Fundamentals

## What is AWS OpenSearch?

Amazon OpenSearch Service is a fully managed search and analytics service that allows organizations to index, search, analyze, and visualize massive amounts of structured and unstructured data in near real time.

It is commonly used for

- Log Analytics
- Application Search
- Security Analytics
- Observability
- Business Intelligence
- Full-Text Search

AWS manages the infrastructure while engineers focus on indexing and querying data.

---

# Why Was OpenSearch Created?

Traditional relational databases are excellent for transactional workloads.

Example

```text
Customer

↓

MySQL

↓

Find Customer ID = 1001
```

However, searching millions of log records or performing full-text search is inefficient in relational databases.

Example

```text
Search

"Payment Timeout"

Across

5 Billion Logs
```

A relational database would require expensive table scans.

OpenSearch solves this problem by building specialized indexes optimized for search.

---

# Evolution

Originally

```text
Elasticsearch

+

Kibana
```

AWS later introduced

```text
OpenSearch

+

OpenSearch Dashboards
```

OpenSearch continues as the open-source search and analytics engine supported by AWS.

---

# Typical Enterprise Use Cases

OpenSearch is widely used for

- Centralized Logging
- Kubernetes Logs
- Application Logs
- Security Event Analysis
- SIEM Platforms
- E-commerce Product Search
- Website Search
- Infrastructure Monitoring
- Cloud Operations
- DevOps Observability

---

# Real Production Example

A company operates

- 250 Kubernetes Nodes
- 1,200 Containers
- 40 Microservices

Every service produces logs.

Instead of logging locally,

all logs are centralized.

```text
Applications

↓

Fluent Bit

↓

OpenSearch

↓

Dashboards

↓

DevOps Team
```

Engineers can search logs from every application in seconds.

---

# OpenSearch Architecture

```text
Applications

↓

Log Collector

↓

Amazon OpenSearch

↓

Indexes

↓

Search Queries

↓

OpenSearch Dashboards

↓

Users
```

---

# Why Not Store Logs in Files?

Example

```text
Server-1

↓

/var/log/app.log

Server-2

↓

/var/log/app.log

Server-3

↓

/var/log/app.log
```

Problems

- Distributed logs
- Difficult searching
- No centralized visibility
- Difficult troubleshooting

---

Instead

```text
All Servers

↓

OpenSearch

↓

Single Search Platform
```

---

# Core Components

An OpenSearch deployment consists of

```text
OpenSearch Cluster

├── Nodes

├── Indices

├── Documents

├── Shards

├── Replicas

└── OpenSearch Dashboards
```

---

# What is a Cluster?

A Cluster is the complete OpenSearch environment.

Example

```text
Production Cluster

↓

Payment Logs

↓

Application Logs

↓

Security Logs
```

Every node belongs to one cluster.

---

# What is a Node?

A Node is a server participating in the OpenSearch cluster.

```text
Cluster

├── Node-1

├── Node-2

└── Node-3
```

Each node stores data and participates in search operations.

---

# Types of Nodes

Common node roles include

- Cluster Manager Node
- Data Node
- Coordinating Node
- Ingest Node

Large enterprise clusters may dedicate separate nodes for different responsibilities.

---

# What is an Index?

An Index is similar to a table in a relational database.

Example

```text
payment-logs

user-logs

security-events

nginx-access
```

Each Index stores related documents.

---

# What is a Document?

A Document is a single record stored in JSON format.

Example

```json
{
  "timestamp":"2026-08-05T10:15:00Z",
  "service":"payment-api",
  "status":"SUCCESS",
  "responseTime":145
}
```

Every document belongs to an Index.

---

# OpenSearch Data Hierarchy

```text
Cluster

↓

Index

↓

Shard

↓

Document

↓

Fields
```

This hierarchy is fundamental to understanding OpenSearch.

---

# OpenSearch vs Relational Database

| Relational Database | OpenSearch |
|----------------------|------------|
| Database | Cluster |
| Table | Index |
| Row | Document |
| Column | Field |
| SQL Query | Search Query |

This comparison is frequently asked in interviews.

---

# Why JSON?

OpenSearch stores data as JSON documents.

Benefits

- Flexible schema
- Easy integration
- Nested objects
- Fast indexing
- API friendly

---

# Search Workflow

```text
Application

↓

Index Document

↓

OpenSearch

↓

Create Index

↓

Store Document

↓

Search Request

↓

Search Results
```

---

# Enterprise Example

A banking platform generates

- Login Events
- Payment Logs
- Fraud Detection Logs
- API Gateway Logs
- Kubernetes Logs

Architecture

```text
Applications

↓

Fluent Bit

↓

Amazon OpenSearch

↓

Dashboards

↓

Operations Team
```

Instead of searching hundreds of servers,

engineers search one centralized platform.

---

# Advantages

- Near real-time search
- Full-text search
- Scalable architecture
- High availability
- REST APIs
- JSON support
- Centralized analytics
- Visualization support

---

# Limitations

- Higher memory usage
- Requires shard planning
- Storage intensive
- Improper mappings can impact performance
- Large clusters require careful capacity planning

---

# When Should You Use OpenSearch?

Use OpenSearch when

- Searching logs
- Centralized observability
- Full-text search
- Security analytics
- Large datasets
- Fast filtering and aggregation

Do not use OpenSearch as a replacement for transactional databases.

It complements—not replaces—databases like PostgreSQL, MySQL, or Amazon RDS.

---

# Best Practices

- Plan index strategy before deployment.
- Use structured JSON logs.
- Enable replication.
- Monitor cluster health.
- Secure access with IAM and VPC.
- Use index lifecycle management.
- Separate production and non-production clusters.

---

# Common Mistakes

- Treating OpenSearch like a relational database.
- Creating too many indices.
- Ignoring shard sizing.
- Storing duplicate data unnecessarily.
- Running without monitoring.

---

# Interview Questions

## Basic

- What is Amazon OpenSearch?
- Why is OpenSearch used?
- What is an Index?
- What is a Document?

## Intermediate

- OpenSearch vs Elasticsearch.
- OpenSearch vs MySQL.
- Explain Cluster, Node, Index, and Document.

## Advanced

- Design a centralized logging platform using OpenSearch.
- Explain why OpenSearch is suitable for observability.
- When would you choose OpenSearch instead of a relational database?

---

