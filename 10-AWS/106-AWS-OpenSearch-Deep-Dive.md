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

# Chapter 2 - OpenSearch Cluster Architecture & Internal Working

Understanding the internal architecture of OpenSearch is essential for designing scalable and highly available production environments.

A search request passes through multiple components before returning results.

Knowing how these components interact helps in designing, scaling, and troubleshooting enterprise OpenSearch clusters.

---

# High-Level Architecture

```text
Applications

↓

REST API

↓

OpenSearch Cluster

↓

Coordinator Node

↓

Data Nodes

↓

Shards

↓

Documents

↓

Search Results
```

Every request enters through the cluster and is routed to the appropriate nodes.

---

# Cluster Architecture

A production cluster typically consists of multiple nodes.

```text
               OpenSearch Cluster

        ┌──────────┼──────────┐

   Manager Node   Data Node   Data Node

                     │

                 Replica Node

                     │

              OpenSearch Dashboards
```

Each node performs a different responsibility.

---

# What is a Cluster Manager Node?

Earlier versions used the term **Master Node**.

OpenSearch now uses the term **Cluster Manager Node**.

Its responsibilities include

- Managing cluster state
- Creating indices
- Allocating shards
- Node discovery
- Cluster health
- Leader election

It does **not** process large volumes of search data.

---

# Cluster Manager Workflow

```text
New Node

↓

Join Cluster

↓

Cluster Manager

↓

Update Cluster State

↓

Notify Other Nodes
```

The Cluster Manager coordinates the cluster.

---

# What is a Data Node?

Data Nodes store

- Documents
- Shards
- Replicas

They perform

- Search
- Indexing
- Aggregations
- Filtering

Most cluster resources are consumed by Data Nodes.

---

# Data Node Architecture

```text
Data Node

├── Primary Shards

├── Replica Shards

├── Search Engine

└── Storage
```

Adding more Data Nodes increases cluster capacity.

---

# Coordinating Node

Every node can act as a coordinating node.

Responsibilities

- Receive client request
- Forward request
- Merge responses
- Return final result

Workflow

```text
Client

↓

Coordinator

↓

Shard 1

Shard 2

Shard 3

↓

Merge Results

↓

Response
```

The coordinating node does not permanently store data unless it also has the data role.

---

# Ingest Node

Some organizations preprocess documents before indexing.

Example

```text
Application

↓

Ingest Pipeline

↓

Convert Timestamp

↓

Remove Fields

↓

GeoIP Lookup

↓

Index Document
```

This work is performed by an Ingest Node.

---

# Dedicated Node Roles

Large production clusters separate responsibilities.

```text
Cluster

├── Manager Nodes

├── Data Nodes

├── Ingest Nodes

├── Coordinating Nodes

└── Dashboards
```

Benefits

- Better scalability
- Easier troubleshooting
- Independent scaling
- Improved stability

---

# Small Cluster Architecture

Development

```text
Single Node

↓

Manager

↓

Data

↓

Coordinator

↓

Ingest
```

Everything runs on one node.

Suitable only for development.

---

# Enterprise Architecture

```text
              Load Balancer

                    │

          Coordinating Nodes

                    │

      ┌─────────────┼─────────────┐

  Manager-1     Manager-2     Manager-3

      │

──────────────────────────────────────────

Data-1   Data-2   Data-3   Data-4   Data-5

      │

──────────────────────────────────────────

Dedicated Ingest Nodes
```

This architecture is commonly used in production.

---

# Cluster Discovery

When a node starts

```text
New Node

↓

Discover Manager

↓

Join Cluster

↓

Receive Cluster State

↓

Start Processing
```

Cluster discovery is automatic once configured.

---

# Cluster State

Cluster State contains

- Nodes
- Indices
- Shards
- Replicas
- Routing Information
- Mappings
- Templates

Every node keeps an updated copy.

---

# Leader Election

Suppose

```text
Manager Node

↓

Failure
```

Remaining manager-eligible nodes perform an election.

```text
Manager-1

↓

Failure

↓

Election

↓

Manager-2

↓

New Cluster Manager
```

This minimizes downtime.

---

# Why Three Manager Nodes?

Production recommendation

```text
Manager-1

Manager-2

Manager-3
```

Why not two?

With only two nodes,

network issues can cause a split-brain scenario where both nodes believe they are the leader.

Three manager nodes provide quorum-based decision making.

---

# Quorum

Example

```text
3 Manager Nodes

↓

2 Available

↓

Cluster Continues
```

But

```text
3 Manager Nodes

↓

Only 1 Available

↓

Cluster Cannot Elect Leader
```

Majority is required.

---

# Node Communication

Nodes constantly exchange

- Cluster State
- Health Information
- Shard Allocation
- Replication Updates

Architecture

```text
Node A

↔

Node B

↔

Node C
```

Communication is continuous.

---

# REST API

Applications communicate with OpenSearch using REST APIs.

Example

```text
Application

↓

HTTPS

↓

OpenSearch

↓

JSON Response
```

This makes integration simple across different programming languages.

---

# Search Request Flow

Suppose a user searches

```text
payment timeout
```

Flow

```text
Application

↓

Coordinator Node

↓

Shard Search

↓

Merge Results

↓

Sort Results

↓

Return JSON
```

The application receives only the final merged response.

---

# Index Request Flow

Adding a new document

```text
Application

↓

Coordinator

↓

Primary Shard

↓

Replica Shard

↓

Acknowledgement

↓

Success
```

Replication occurs automatically.

---

# Scaling the Cluster

Need more storage?

```text
Add Data Nodes
```

Need better coordination?

```text
Add Coordinating Nodes
```

Need more ingestion throughput?

```text
Add Ingest Nodes
```

Each role scales independently.

---

# High Availability

Production clusters should use multiple Availability Zones.

```text
Availability Zone A

↓

Manager

↓

Data

────────────────

Availability Zone B

↓

Manager

↓

Data

────────────────

Availability Zone C

↓

Manager

↓

Data
```

A single Availability Zone failure should not stop the cluster.

---

# Enterprise Production Example

A retail platform generates

- 900 GB logs/day
- 15 billion documents
- 400 searches/second

Architecture

```text
Applications

↓

Fluent Bit

↓

Load Balancer

↓

Coordinating Nodes

↓

Data Nodes

↓

OpenSearch Dashboards

↓

Operations Team
```

The platform continues operating even if individual nodes fail.

---

# Best Practices

- Use three dedicated Cluster Manager nodes in production.
- Separate Manager and Data Nodes for large clusters.
- Deploy across multiple Availability Zones.
- Scale Data Nodes independently.
- Monitor cluster health continuously.
- Avoid running production on a single node.

---

# Common Mistakes

- Single-node production clusters.
- Using only one Manager node.
- Running Manager and Data roles together in very large clusters.
- Ignoring cluster health warnings.
- Not distributing nodes across Availability Zones.

---

# Interview Questions

## Basic

- What is an OpenSearch Cluster?
- What is a Data Node?
- What is a Cluster Manager Node?

## Intermediate

- Explain Coordinating Nodes.
- Explain Ingest Nodes.
- Why are three Manager nodes recommended?

## Advanced

- Design a highly available OpenSearch cluster for processing 5 TB of logs per day.
- Explain the complete search request flow inside an OpenSearch cluster.
- How does OpenSearch maintain availability when a Manager node fails?

---

