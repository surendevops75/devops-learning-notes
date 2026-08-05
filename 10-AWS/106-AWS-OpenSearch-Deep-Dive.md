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

# Chapter 3 - Indices, Documents, Fields, Shards & Replicas

Understanding **Indices, Documents, Shards, and Replicas** is the foundation of OpenSearch.

Nearly every production design and interview question revolves around these concepts.

---

# Data Organization

OpenSearch stores data in the following hierarchy.

```text
Cluster

↓

Index

↓

Primary Shard

↓

Document

↓

Fields
```

Unlike relational databases, OpenSearch stores data as JSON documents.

---

# What is an Index?

An Index is a logical collection of related documents.

Think of it as similar to a table in a relational database.

Example

```text
payment-logs

user-logs

nginx-access

application-events

audit-logs
```

Each index usually stores one type of data.

---

# Production Example

A banking application may create separate indices.

```text
payment-logs

↓

Payment Transactions

login-events

↓

Authentication Logs

audit-events

↓

Compliance Logs

application-logs

↓

Application Events
```

Separating data improves management and performance.

---

# Index Naming Best Practices

Large organizations follow consistent naming.

Example

```text
prod-payment-logs

prod-order-logs

prod-nginx-access

dev-payment-logs

test-payment-logs
```

This makes lifecycle management much easier.

---

# What is a Document?

A Document is the smallest unit stored in OpenSearch.

Documents are stored as JSON.

Example

```json
{
  "transactionId":"TX12345",
  "customer":"Surendra",
  "amount":5000,
  "currency":"INR",
  "status":"SUCCESS",
  "timestamp":"2026-08-05T09:20:00Z"
}
```

Every document belongs to exactly one index.

---

# Document Structure

```text
Document

├── transactionId

├── customer

├── amount

├── currency

├── status

└── timestamp
```

Each key is called a **Field**.

---

# What is a Field?

A Field represents one attribute inside a document.

Example

```json
{
 "service":"payment-api"
}
```

Field

```text
service
```

Value

```text
payment-api
```

---

# Common Field Types

OpenSearch supports many field types.

Examples

| Type | Example |
|------|----------|
| text | Payment Failed |
| keyword | SUCCESS |
| integer | 100 |
| long | 900000 |
| float | 95.5 |
| boolean | true |
| date | 2026-08-05 |
| object | Nested JSON |
| ip | 10.0.1.15 |

Choosing the correct field type is important for performance.

---

# Text vs Keyword

One of the most frequently asked interview topics.

Suppose

```text
SUCCESS
```

stored as

```text
text
```

OpenSearch analyzes it.

Suitable for

- Full-text search
- Natural language

---

Suppose

```text
SUCCESS
```

stored as

```text
keyword
```

Stored exactly as written.

Suitable for

- Filtering
- Sorting
- Aggregations

---

# Example

Search

```text
payment failed because database timeout
```

Use

```text
text
```

---

Status

```text
SUCCESS

FAILED
```

Use

```text
keyword
```

---

# What is a Shard?

OpenSearch cannot store billions of documents inside a single file.

Instead,

it splits an Index into multiple pieces called **Shards**.

Example

```text
payment-logs

↓

Shard-1

Shard-2

Shard-3
```

Each shard stores part of the data.

---

# Why Shards?

Suppose an index contains

```text
10 Billion Documents
```

One server cannot efficiently store or search everything.

Instead

```text
10 Billion

↓

Shard-1

↓

3.3 Billion

Shard-2

↓

3.3 Billion

Shard-3

↓

3.3 Billion
```

Now multiple servers work together.

---

# Primary Shards

Primary Shards contain the original data.

Example

```text
payment-index

↓

Primary-1

Primary-2

Primary-3
```

Every document is written to exactly one primary shard.

---

# Shard Distribution

Cluster

```text
Node-1

↓

Primary-1

Node-2

↓

Primary-2

Node-3

↓

Primary-3
```

Data is distributed automatically.

---

# Search Across Shards

Suppose a user searches

```text
Transaction Failed
```

Coordinator

↓

Query

↓

Primary-1

Primary-2

Primary-3

↓

Merge Results

↓

Return Response

All shards search simultaneously.

---

# What is a Replica?

A Replica is a copy of a Primary Shard.

Purpose

- High Availability
- Faster Searches
- Fault Tolerance

Example

```text
Primary-1

↓

Replica-1
```

---

# Replica Architecture

```text
Node-1

↓

Primary-1

──────────────

Node-2

↓

Replica-1
```

Primary and Replica are never placed on the same node.

This protects against node failures.

---

# Write Operation

Application

↓

Primary Shard

↓

Replica Shard

↓

Acknowledgement

↓

Success

Data is copied automatically.

---

# Read Operation

Search requests can be served by

- Primary
- Replica

This improves search throughput.

---

# Why Replicas Improve Performance

Without Replica

```text
100 Search Requests

↓

Primary
```

With Replica

```text
100 Requests

↓

Primary

↓

Replica

↓

Load Shared
```

Multiple copies increase query capacity.

---

# What Happens if a Node Fails?

Example

```text
Node-1

↓

Primary-1

↓

Failure
```

Replica

↓

Promoted

↓

New Primary

Applications continue operating.

---

# Replica Recovery

Failed node returns.

Workflow

```text
Node Returns

↓

Replica Rebuilt

↓

Cluster Balanced
```

Recovery happens automatically.

---

# Number of Shards

Choosing shard count is important.

Too Few

```text
Large Shards

↓

Slow Recovery
```

Too Many

```text
Thousands of Small Shards

↓

Memory Waste

↓

Slow Cluster
```

Both extremes reduce performance.

---

# Shard Sizing

A balanced shard size simplifies

- Recovery
- Search
- Storage
- Cluster Management

Enterprise clusters periodically review shard distribution as data grows.

---

# Routing

When indexing a document,

OpenSearch decides

```text
Document

↓

Hash Function

↓

Primary Shard
```

Applications do not choose the shard.

Routing is automatic unless custom routing is configured.

---

# Search Flow

```text
Client

↓

Coordinator

↓

Primary

↓

Replica

↓

Merge Results

↓

Return JSON
```

Searches are executed in parallel.

---

# Enterprise Example

A retail company generates

- Product Search
- Customer Logs
- Payment Logs
- Security Logs

Architecture

```text
Cluster

↓

Product Index

↓

5 Primary Shards

↓

1 Replica

↓

10 Total Shards
```

The platform continues serving queries even if one Data Node becomes unavailable.

---

# Best Practices

- Separate indices by workload.
- Choose appropriate field types.
- Use keyword for filtering.
- Use text for full-text search.
- Configure replicas in production.
- Monitor shard distribution.
- Avoid extremely small or extremely large shards.

---

# Common Mistakes

- Creating hundreds of tiny indices.
- Using text instead of keyword for exact matching.
- Running production without replicas.
- Creating excessive shard counts.
- Ignoring shard rebalancing after cluster growth.

---

# Interview Questions

## Basic

- What is an Index?
- What is a Document?
- What is a Field?
- What is a Shard?

## Intermediate

- Primary Shard vs Replica Shard.
- Text vs Keyword.
- Why does OpenSearch split data into shards?
- How are search requests processed across shards?

## Advanced

- Design shard and replica strategy for a logging platform storing 8 TB of logs per day.
- Explain what happens when a Data Node containing Primary Shards fails.
- How would you optimize shard sizing for a rapidly growing OpenSearch cluster?

---

# Chapter 4 - Mapping, Dynamic Mapping, Data Types & Index Templates

One of the biggest reasons OpenSearch clusters become slow or unstable is **poor mapping design**.

Mappings define **how OpenSearch stores and indexes every field** inside a document.

Choosing the correct mapping improves

- Search Performance
- Storage Efficiency
- Aggregation Speed
- Query Accuracy

Poor mappings lead to

- Large indices
- Slow searches
- High memory usage
- Incorrect search results

---

# What is Mapping?

A Mapping defines

- Field Name
- Data Type
- Indexing Behavior
- Search Behavior

Think of it as a schema for an Index.

Example

```text
Document

↓

Mapping

↓

Store Correctly

↓

Search Efficiently
```

---

# Mapping Architecture

```text
Index

↓

Mapping

├── customerName

├── amount

├── timestamp

├── status

└── service
```

Every field has a defined data type.

---

# Why Mapping is Important

Suppose

```text
Amount

500
```

If stored as

```text
Text
```

Sorting becomes inefficient.

If stored as

```text
Integer
```

Sorting and aggregations become much faster.

Choosing the correct data type directly impacts performance.

---

# Dynamic Mapping

By default,

OpenSearch automatically detects new fields.

Example

Application sends

```json
{
 "customer":"John",
 "amount":500
}
```

OpenSearch automatically creates

```text
customer → text

amount → long
```

No manual schema creation is required.

---

# Dynamic Mapping Workflow

```text
Application

↓

New Field

↓

OpenSearch Detects Type

↓

Creates Mapping

↓

Stores Document
```

Very convenient for development.

---

# Problems with Dynamic Mapping

Suppose different applications send

Application-A

```json
{
 "amount":500
}
```

Application-B

```json
{
 "amount":"500"
}
```

Now the same field has different types.

This may cause

- Mapping conflicts
- Failed indexing
- Query failures

Large enterprises usually avoid relying entirely on Dynamic Mapping.

---

# Explicit Mapping

Instead of automatic detection,

developers define mappings manually.

Example

```text
amount

↓

integer

status

↓

keyword

timestamp

↓

date
```

Benefits

- Predictable
- Better performance
- Easier maintenance
- No unexpected field types

---

# Mapping Comparison

| Dynamic Mapping | Explicit Mapping |
|-----------------|------------------|
| Automatic | Manual |
| Easy to start | Better for production |
| Less control | Full control |
| Risk of conflicts | Predictable schema |

---

# Common Data Types

OpenSearch supports many field types.

---

## Text

Used for

- Full-text search
- Articles
- Log Messages
- Product Descriptions

Example

```text
Payment failed because database timeout
```

Supports

- Tokenization
- Stemming
- Full-text search

---

## Keyword

Stores values exactly as received.

Examples

```text
SUCCESS

FAILED

PAYMENT

LOGIN
```

Suitable for

- Filtering
- Sorting
- Aggregations
- Exact Match

---

# Text vs Keyword

Example

Search

```text
database timeout
```

Use

```text
text
```

---

Filter

```text
status = SUCCESS
```

Use

```text
keyword
```

This is one of the most common OpenSearch interview questions.

---

# Numeric Types

Supported

```text
integer

long

float

double

short

byte
```

Examples

```text
Amount

↓

double

Age

↓

integer

Response Time

↓

float
```

---

# Date Type

Logs almost always contain timestamps.

Example

```text
2026-08-05T11:45:20Z
```

Stored as

```text
date
```

Benefits

- Time-based searches
- Sorting
- Dashboards
- Time filters

---

# Boolean Type

Stores

```text
true

false
```

Example

```json
{
 "success":true
}
```

---

# IP Type

Useful for

- Security Analytics
- Firewall Logs
- VPC Flow Logs

Example

```text
192.168.1.10

10.0.1.25
```

Stored efficiently for IP searches.

---

# Object Type

Example

```json
{
 "customer":{
    "id":101,
    "name":"John"
 }
}
```

The object contains nested fields.

---

# Nested Type

Suppose

```json
Order

↓

Items

↓

Product

↓

Quantity
```

Nested fields preserve relationships inside arrays of objects.

Frequently used in

- E-commerce
- Inventory
- Order Management

---

# Multi Fields

One field can be indexed in multiple ways.

Example

```text
customerName

↓

text

↓

keyword
```

Benefits

Search

```text
text
```

Filtering

```text
keyword
```

Best of both worlds.

---

# Indexing

Every indexed field becomes searchable.

Example

```text
service

↓

Indexed

↓

Search Possible
```

---

# Non-Indexed Fields

Sometimes fields are stored but not searchable.

Example

```text
Large Debug Payload

↓

Store Only

↓

No Index
```

Benefits

- Smaller index
- Faster indexing
- Reduced storage

---

# Doc Values

OpenSearch stores Doc Values for

- Sorting
- Aggregations
- Filtering

Example

```text
Sales Report

↓

Aggregation

↓

Doc Values
```

Doc Values improve query performance.

---

# Field Explosion

Suppose logs contain

```text
field1

field2

field3

...

field50000
```

Dynamic Mapping creates thousands of fields.

Problems

- High memory usage
- Slow cluster state
- Mapping explosion

Production systems often restrict the number of fields.

---

# Index Templates

Creating mappings manually for every index is inefficient.

Instead,

OpenSearch uses Index Templates.

Architecture

```text
Template

↓

New Index

↓

Mapping Applied

↓

Settings Applied
```

---

# Example

Template

```text
logs-*
```

Automatically applies to

```text
logs-2026-08

logs-2026-09

logs-2026-10
```

Every new log index receives identical mappings.

---

# What Can Templates Configure?

Templates can define

- Number of Shards
- Replica Count
- Mappings
- Index Settings
- Aliases
- Lifecycle Policies

Everything is applied automatically.

---

# Component Templates

Large organizations divide templates.

Example

```text
Common Mapping

↓

Security Settings

↓

Lifecycle Policy

↓

Index Template
```

Reusable templates reduce configuration duplication.

---

# Enterprise Logging Example

A company creates

```text
payment-logs

↓

Template Applied

↓

5 Shards

↓

1 Replica

↓

Timestamp Mapping

↓

Keyword Fields

↓

Lifecycle Policy
```

Every monthly index follows the same standards.

---

# Production Architecture

```text
Applications

↓

Fluent Bit

↓

OpenSearch

↓

Index Template

↓

Mapping

↓

Primary Shards

↓

Replica Shards
```

Every new index is created automatically using predefined templates.

---

# Best Practices

- Prefer explicit mappings for production.
- Use keyword for filtering.
- Use text for full-text search.
- Store timestamps as date fields.
- Use index templates.
- Prevent field explosion.
- Review mappings before large-scale ingestion.

---

# Common Mistakes

- Relying entirely on Dynamic Mapping.
- Using text instead of keyword.
- Creating thousands of unnecessary fields.
- Storing numbers as strings.
- Ignoring index templates.
- Allowing uncontrolled mapping growth.

---

# Interview Questions

## Basic

- What is Mapping?
- What is Dynamic Mapping?
- What is an Index Template?

## Intermediate

- Text vs Keyword.
- Dynamic Mapping vs Explicit Mapping.
- Why use Multi Fields?
- What are Doc Values?

## Advanced

- Design mappings for a centralized logging platform ingesting billions of log records.
- Explain how Index Templates simplify large enterprise deployments.
- How would you prevent mapping explosion in an OpenSearch cluster receiving logs from hundreds of microservices?

---

