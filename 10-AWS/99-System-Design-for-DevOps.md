# System Design Fundamentals

---

# Introduction

System Design is the process of designing scalable, reliable, secure, and highly available systems capable of serving millions of users while meeting business and technical requirements.

For DevOps Engineers, system design focuses on infrastructure, scalability, deployment architecture, automation, observability, disaster recovery, and operational excellence.

---

# Goals of System Design

Design systems that are

- Scalable
- Highly Available
- Fault Tolerant
- Reliable
- Secure
- Maintainable
- Cost Optimized
- Observable

---

# System Design Workflow

```text
Requirements

↓

Capacity Planning

↓

Architecture

↓

Database Design

↓

Caching

↓

Load Balancing

↓

Deployment

↓

Monitoring

↓

Scaling
```

---

# Functional Requirements

Describe

"What the system should do."

Examples

- User login
- Upload files
- Stream videos
- Place orders
- Send notifications

---

# Non-Functional Requirements

Describe

"How well the system should perform."

Examples

- High Availability
- Scalability
- Performance
- Reliability
- Security
- Low Latency
- Disaster Recovery

---

# Capacity Planning

Estimate

- Users
- Requests per second (RPS)
- Storage
- CPU
- Memory
- Network bandwidth

---

# Example Capacity Planning

Application

```text
Daily Users

10 Million
```

Average Requests

```text
5 Requests/User
```

Daily Requests

```text
50 Million Requests
```

---

# Scaling

Two approaches

```text
Vertical Scaling

OR

Horizontal Scaling
```

---

# Vertical Scaling

```text
Application

↓

More CPU

More Memory

More Storage
```

Advantages

- Simple
- No application changes

Disadvantages

- Hardware limits
- Single point of failure

---

# Horizontal Scaling

```text
Application

↓

Load Balancer

↓

Multiple Servers
```

Advantages

- High availability
- Fault tolerance
- Unlimited growth

Disadvantages

- More complex
- Requires load balancing

---

# Stateless Applications

Characteristics

- No session stored locally
- Easy scaling
- Easy replacement
- Ideal for Kubernetes

---

# Stateful Applications

Characteristics

- Local data
- Persistent storage
- Complex scaling
- Requires StatefulSets

Examples

- Databases
- Kafka
- Elasticsearch

---

# High Availability

Objective

Keep services available despite failures.

Methods

- Multiple Availability Zones
- Multiple Servers
- Load Balancers
- Automatic Failover

---

# Fault Tolerance

Definition

Continue operating even when one or more components fail.

Examples

- Multiple application replicas
- Multi-node databases
- Auto Scaling

---

# Reliability

Reliable systems

- Recover automatically
- Minimize downtime
- Handle failures gracefully

---

# Scalability vs Availability

Scalability

Ability to handle increasing load.

Availability

Ability to remain operational.

---

# Performance

Important Metrics

- Response Time
- Throughput
- Latency
- CPU Usage
- Memory Usage
- Error Rate

---

# Latency

Definition

Time required to complete one request.

Goal

Reduce latency wherever possible.

---

# Throughput

Definition

Number of requests processed per unit time.

Examples

```text
500 Requests/Second

1000 Requests/Second
```

---

# Bottleneck Analysis

Common Bottlenecks

- CPU
- Memory
- Database
- Network
- Disk
- Cache
- External APIs

---

# Single Point of Failure (SPOF)

Definition

One component whose failure causes complete system outage.

Examples

- Single database
- Single load balancer
- Single server

Avoid SPOFs whenever possible.

---

# Redundancy

Provide multiple copies of

- Servers
- Databases
- Load Balancers
- Network paths

---

# System Architecture

```text
Users

↓

DNS

↓

Load Balancer

↓

Application

↓

Cache

↓

Database
```

---

# Monolithic Architecture

Characteristics

- Single codebase
- Single deployment
- Easy initially
- Difficult to scale

---

# Microservices Architecture

Characteristics

- Independent services
- Independent deployments
- Better scalability
- Higher operational complexity

---

# Event-Driven Architecture

Workflow

```text
Producer

↓

Message Queue

↓

Consumer
```

Examples

- Kafka
- RabbitMQ
- Amazon SQS

---

# Distributed Systems

Characteristics

- Multiple nodes
- Shared workload
- Independent failures
- High scalability

Challenges

- Network latency
- Consistency
- Coordination

---

# CAP Theorem

Distributed systems can guarantee only two of

```text
Consistency

Availability

Partition Tolerance
```

---

# Consistency

All users see the same data after updates.

---

# Availability

Every request receives a response.

---

# Partition Tolerance

System continues despite network failures.

---

# Consistency Models

Strong Consistency

All reads return the latest value.

---

Eventual Consistency

Data becomes consistent over time.

Examples

- Amazon S3
- DNS

---

# Availability Zones

Purpose

Protect against data center failures.

Deploy production workloads across multiple Availability Zones.

---

# Multi-Region Architecture

Benefits

- Disaster Recovery
- Lower latency
- Business continuity

Trade-offs

- Higher cost
- More operational complexity

---

# Design Principles

- Simplicity
- Automation
- Loose coupling
- High cohesion
- Scalability
- Fault isolation

---

# Design Trade-offs

Examples

- Cost vs Performance
- Consistency vs Availability
- Simplicity vs Flexibility
- Latency vs Durability

---

# Failure Handling

Prepare for

- Server failures
- Network failures
- Database failures
- Region failures
- Human errors

---

# Design Validation Checklist

Verify

- No SPOF
- High availability
- Auto Scaling
- Monitoring
- Backup strategy
- Disaster Recovery
- Security
- Observability

---

# Common Design Mistakes

- Single server architecture
- No Auto Scaling
- No monitoring
- Shared databases
- Hardcoded configuration
- No caching
- Poor capacity planning
- No Disaster Recovery
- Ignoring security
- No redundancy

---

# Best Practices

- Design for failure from the beginning.
- Build stateless services whenever possible.
- Scale horizontally instead of vertically.
- Eliminate single points of failure.
- Deploy across multiple Availability Zones.
- Monitor every critical component.
- Automate deployments and recovery.
- Perform realistic capacity planning.
- Document architectural decisions.
- Continuously review and improve system design.

---

# Summary

This section introduced system design fundamentals, functional and non-functional requirements, scalability, high availability, fault tolerance, CAP theorem, distributed systems, architecture patterns, bottleneck analysis, capacity planning, and production design principles. These concepts form the foundation for designing enterprise-scale cloud-native systems.

---

# Load Balancing, Traffic Management & Scaling

---

# Introduction

Load balancing distributes incoming traffic across multiple servers or services to improve availability, scalability, fault tolerance, and performance.

Objectives

- Prevent server overload
- Improve availability
- Increase scalability
- Enable fault tolerance
- Support zero-downtime deployments

---

# Traffic Flow

```text
Users

↓

DNS

↓

Load Balancer

↓

Application Servers

↓

Database
```

---

# What is Load Balancing?

Definition

Distributes incoming client requests across multiple backend servers.

Benefits

- High Availability
- Scalability
- Fault Tolerance
- Better Resource Utilization
- Zero Downtime Maintenance

---

# Load Balancer Types

## Layer 4 (Transport Layer)

Works with

- TCP
- UDP

Routes traffic based on

- IP Address
- Port

Examples

- AWS Network Load Balancer (NLB)
- HAProxy (TCP Mode)

---

## Layer 7 (Application Layer)

Works with

- HTTP
- HTTPS
- gRPC

Routes traffic based on

- URL Path
- Host Header
- Cookies
- HTTP Headers

Examples

- AWS Application Load Balancer (ALB)
- NGINX
- Traefik

---

# Layer 4 vs Layer 7

| Feature | Layer 4 | Layer 7 |
|----------|---------|---------|
| Protocol | TCP/UDP | HTTP/HTTPS |
| Speed | Faster | Slightly Slower |
| Content Awareness | No | Yes |
| Path Routing | No | Yes |
| Host Routing | No | Yes |
| SSL Termination | Limited | Supported |

---

# AWS Load Balancers

Application Load Balancer (ALB)

Best For

- Web applications
- REST APIs
- Kubernetes Ingress
- Microservices

---

Network Load Balancer (NLB)

Best For

- TCP Applications
- Low latency
- High throughput
- Static IP

---

Gateway Load Balancer (GWLB)

Best For

- Firewalls
- IDS/IPS
- Security Appliances
- Traffic Inspection

---

# AWS Load Balancer Selection

```text
HTTP/HTTPS

↓

ALB

----------------------

TCP/UDP

↓

NLB

----------------------

Network Appliances

↓

GWLB
```

---

# Reverse Proxy

Definition

Receives client requests and forwards them to backend servers.

Examples

- NGINX
- HAProxy
- Traefik
- Envoy

---

# Reverse Proxy Architecture

```text
Client

↓

NGINX

↓

Application Servers
```

---

# Load Balancing Algorithms

Round Robin

```text
Server1

↓

Server2

↓

Server3

↓

Repeat
```

---

Least Connections

Route traffic to the server with the fewest active connections.

---

Least Response Time

Choose the backend with the fastest response time.

---

Weighted Round Robin

Assign more traffic to higher-capacity servers.

Example

```text
Server A

Weight 5

Server B

Weight 2

Server C

Weight 1
```

---

# Sticky Sessions

Definition

Routes a user's requests to the same backend server.

Advantages

- Session persistence

Disadvantages

- Uneven traffic distribution
- Harder horizontal scaling

Prefer stateless applications where possible.

---

# Health Checks

Purpose

Detect unhealthy backend servers automatically.

---

# Health Check Workflow

```text
Load Balancer

↓

Health Check

↓

Healthy?

↓

Yes → Route Traffic

No → Remove Target
```

---

# Health Check Parameters

Configure

- Protocol
- Port
- Path
- Interval
- Timeout
- Healthy Threshold
- Unhealthy Threshold

---

# Auto Scaling

Purpose

Automatically adjust infrastructure based on demand.

---

# Auto Scaling Workflow

```text
Traffic Increase

↓

CloudWatch Alarm

↓

Auto Scaling

↓

Launch Instance

↓

Register with Load Balancer
```

---

# Scale-In Workflow

```text
Low Utilization

↓

CloudWatch Alarm

↓

Auto Scaling

↓

Drain Connections

↓

Terminate Instance
```

---

# Connection Draining

Purpose

Allow existing requests to complete before removing an instance.

Benefits

- Zero downtime
- Graceful shutdown
- Better user experience

---

# Service Discovery

Purpose

Allow applications to dynamically discover backend services.

Examples

- Kubernetes DNS
- AWS Cloud Map
- Consul

---

# Kubernetes Service Discovery

```text
Pod

↓

Service

↓

CoreDNS

↓

Destination Pod
```

---

# DNS-Based Load Balancing

Methods

- Round Robin DNS
- Weighted Routing
- Latency-Based Routing
- Geolocation Routing
- Failover Routing

---

# Amazon Route 53 Routing Policies

Supports

- Simple Routing
- Weighted Routing
- Latency Routing
- Geolocation Routing
- Geoproximity Routing
- Failover Routing
- Multi-Value Answer Routing

---

# Global Traffic Management

Architecture

```text
Users

↓

Route 53

↓

Nearest Region

↓

Regional Load Balancer

↓

Application
```

---

# Content Delivery Network (CDN)

Purpose

Deliver static content from edge locations closer to users.

Benefits

- Lower latency
- Reduced origin load
- Faster page loads
- DDoS mitigation

---

# Amazon CloudFront

Supports

- Static Content
- Dynamic Content
- API Acceleration
- Video Streaming

---

# CloudFront Architecture

```text
Users

↓

Edge Location

↓

CloudFront

↓

Origin

↓

Application
```

---

# API Gateway

Purpose

Acts as a single entry point for APIs.

Features

- Authentication
- Authorization
- Rate Limiting
- Request Validation
- Monitoring
- API Versioning

---

# API Gateway Workflow

```text
Client

↓

API Gateway

↓

Authentication

↓

Microservice

↓

Response
```

---

# Rate Limiting

Purpose

Protect applications from excessive requests.

Methods

- Requests per second
- Burst limits
- User quotas
- API keys

---

# Circuit Breaker Pattern

Workflow

```text
Healthy

↓

Failures Increase

↓

Circuit Opens

↓

Requests Blocked

↓

Recovery Check

↓

Circuit Closes
```

Benefits

- Prevents cascading failures
- Improves system resilience

---

# Traffic Routing Strategies

Blue/Green

```text
Blue

↓

Green

↓

Switch Traffic
```

---

Canary

```text
5%

↓

20%

↓

50%

↓

100%
```

---

A/B Testing

Split traffic between different application versions for comparison.

---

# Session Management

Preferred

```text
External Session Store

↓

Redis
```

Avoid storing sessions on individual application servers.

---

# Load Balancing Metrics

Monitor

- Request Rate
- Response Time
- Active Connections
- Error Rate
- Healthy Targets
- Throughput
- Latency

---

# Common Traffic Bottlenecks

- Single Load Balancer
- Slow Database
- DNS latency
- Backend overload
- Network congestion
- SSL handshake delays
- Missing caching

---

# Production Traffic Checklist

Verify

- Load Balancer healthy
- Health checks passing
- Targets registered
- Auto Scaling active
- CloudFront operational
- Route 53 healthy
- DNS resolving
- SSL certificates valid
- Monitoring enabled

---

# Common Design Mistakes

- Single application server
- Missing health checks
- Sticky sessions for stateless apps
- No Auto Scaling
- No CDN
- Incorrect routing policy
- Missing connection draining
- No failover strategy
- Ignoring latency metrics
- No traffic testing

---

# Best Practices

- Use ALB for HTTP/HTTPS applications and NLB for TCP/UDP workloads.
- Configure meaningful health checks for every backend service.
- Enable Auto Scaling with CloudWatch alarms.
- Use CloudFront to reduce latency and protect origins.
- Prefer stateless services to simplify scaling.
- Implement connection draining during scale-in operations.
- Use Route 53 routing policies for global traffic management.
- Store user sessions in external systems such as Redis.
- Monitor traffic patterns continuously and plan capacity accordingly.
- Test failover, scaling, and deployment strategies regularly.

---

# Summary

This section covered load balancing fundamentals, Layer 4 vs Layer 7 load balancing, AWS ALB, NLB, and GWLB, reverse proxies, load balancing algorithms, health checks, Auto Scaling, service discovery, Route 53 routing policies, CloudFront, API Gateway, traffic routing strategies, and production traffic management. These concepts provide the foundation for building scalable, highly available cloud-native systems.

---

# Database Design, Storage & Caching

---

# Introduction

Databases are the backbone of modern distributed systems. Proper database architecture, caching, replication, and storage strategies are essential for building scalable, reliable, and high-performance applications.

Objectives

- High Performance
- Scalability
- High Availability
- Data Durability
- Low Latency
- Disaster Recovery

---

# Database Architecture

```text
Application

↓

Cache

↓

Primary Database

↓

Read Replica

↓

Backup
```

---

# Types of Databases

## Relational (SQL)

Characteristics

- Structured data
- ACID transactions
- Fixed schema
- Strong consistency

Examples

- PostgreSQL
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server

---

## NoSQL

Characteristics

- Flexible schema
- Horizontal scaling
- High throughput
- Eventual consistency (many implementations)

Examples

- DynamoDB
- MongoDB
- Cassandra
- Redis
- Couchbase

---

# SQL vs NoSQL

| Feature | SQL | NoSQL |
|----------|-----|--------|
| Schema | Fixed | Flexible |
| Transactions | ACID | Varies |
| Scaling | Vertical + Read Scaling | Horizontal |
| Joins | Supported | Limited/Depends |
| Best For | Structured Data | Large-scale Distributed Data |

---

# ACID Properties

Atomicity

All operations succeed or none succeed.

---

Consistency

Database remains in a valid state.

---

Isolation

Concurrent transactions do not interfere.

---

Durability

Committed data survives failures.

---

# BASE Model

Common in distributed NoSQL systems.

Components

- Basically Available
- Soft State
- Eventual Consistency

---

# Choosing a Database

Use SQL when

- Financial systems
- Banking
- Inventory
- Strong consistency

Use NoSQL when

- User profiles
- Product catalogs
- IoT
- Logging
- Large-scale metadata

---

# Database Scaling

Methods

```text
Vertical Scaling

↓

Bigger Server
```

OR

```text
Horizontal Scaling

↓

Multiple Servers
```

---

# Read Replicas

Purpose

Distribute read traffic.

Architecture

```text
Primary Database

↓

Replica 1

↓

Replica 2

↓

Replica 3
```

---

# Read/Write Separation

```text
Application

↓

Write

↓

Primary Database

---------------------

Read

↓

Read Replicas
```

Benefits

- Lower read latency
- Reduced primary load
- Better scalability

---

# Database Replication

Purpose

Maintain synchronized copies of data.

Benefits

- High Availability
- Disaster Recovery
- Read Scaling

---

# Synchronous Replication

Characteristics

- Strong consistency
- Higher latency
- Safer writes

---

# Asynchronous Replication

Characteristics

- Faster writes
- Possible replication lag
- Better performance

---

# Database Sharding

Purpose

Split data across multiple database servers.

Example

```text
Users A-H

↓

Shard 1

----------------

Users I-P

↓

Shard 2

----------------

Users Q-Z

↓

Shard 3
```

Benefits

- Horizontal scaling
- Better performance
- Higher capacity

---

# Database Partitioning

Types

- Range Partitioning
- Hash Partitioning
- List Partitioning
- Composite Partitioning

---

# Database Indexing

Purpose

Speed up query execution.

Common Index Types

- Primary Index
- Secondary Index
- Composite Index
- Unique Index

---

# Index Trade-offs

Advantages

- Faster reads

Disadvantages

- Slower writes
- More storage
- Maintenance overhead

---

# Database Connection Pooling

Purpose

Reuse existing database connections.

Benefits

- Reduced latency
- Better performance
- Lower database overhead

---

# Connection Pool Workflow

```text
Application

↓

Connection Pool

↓

Database
```

---

# Caching

Purpose

Store frequently accessed data in memory.

Benefits

- Lower latency
- Reduced database load
- Faster response times

---

# Cache Architecture

```text
Application

↓

Cache

↓

Database
```

---

# Redis

Best For

- Session storage
- Caching
- Rate limiting
- Queues
- Leaderboards

---

# Memcached

Best For

- Simple key-value caching
- Temporary cache
- Lightweight workloads

---

# Redis vs Memcached

| Feature | Redis | Memcached |
|----------|--------|-----------|
| Persistence | Yes | No |
| Data Types | Multiple | Key-Value |
| Replication | Supported | Limited |
| Pub/Sub | Yes | No |

---

# Cache-Aside Pattern

Workflow

```text
Application

↓

Cache

↓

Miss

↓

Database

↓

Update Cache
```

Advantages

- Simple
- Efficient

---

# Read-Through Cache

Workflow

```text
Application

↓

Cache

↓

Database (Automatic)

↓

Response
```

---

# Write-Through Cache

Workflow

```text
Application

↓

Cache

↓

Database
```

Advantages

- Cache always updated

Trade-off

- Higher write latency

---

# Write-Back Cache

Workflow

```text
Application

↓

Cache

↓

Response

↓

Database (Later)
```

Advantages

- Fast writes

Trade-off

- Risk of temporary data loss

---

# Cache Eviction Policies

Examples

- LRU (Least Recently Used)
- LFU (Least Frequently Used)
- FIFO (First In, First Out)
- TTL (Time To Live)

---

# Cache Invalidation

Strategies

- TTL expiration
- Event-driven invalidation
- Manual invalidation
- Versioned cache keys

---

# CDN Caching

Architecture

```text
Users

↓

CloudFront

↓

Origin

↓

Application
```

Cache

- Images
- CSS
- JavaScript
- Videos
- Static assets

---

# Data Consistency

Types

Strong Consistency

Latest data always returned.

---

Eventual Consistency

Data synchronizes over time.

---

# Backup Strategy

Implement

- Automated backups
- Point-in-Time Recovery (PITR)
- Cross-Region backups
- Backup validation

---

# Disaster Recovery

Protect

- Database backups
- Transaction logs
- Replicas
- Snapshots

---

# Storage Types

Object Storage

Examples

- Amazon S3

Use Cases

- Images
- Videos
- Backups
- Documents

---

Block Storage

Examples

- Amazon EBS

Use Cases

- EC2
- Databases
- Boot volumes

---

File Storage

Examples

- Amazon EFS

Use Cases

- Shared application storage
- Containers
- Web servers

---

# Large-Scale Storage Architecture

```text
Users

↓

Application

↓

Redis Cache

↓

Primary Database

↓

Read Replicas

↓

Amazon S3
```

---

# Database Monitoring

Monitor

- CPU
- Memory
- Connections
- Query latency
- Replication lag
- Slow queries
- Storage utilization
- Cache hit ratio

---

# Common Database Bottlenecks

- Missing indexes
- Slow queries
- High connection count
- Full table scans
- Cache misses
- Storage latency
- Replication lag
- Lock contention

---

# Production Database Checklist

Verify

- Backups successful
- Replication healthy
- Read replicas synchronized
- Indexes optimized
- Cache operational
- Connection pool healthy
- Monitoring enabled
- Disaster Recovery tested

---

# Common Design Mistakes

- No indexes
- Single database server
- No read replicas
- No caching
- Too many database connections
- Full table scans
- Missing backups
- No replication
- Large monolithic tables
- Ignoring query optimization

---

# Best Practices

- Choose SQL or NoSQL based on application requirements.
- Use read replicas to scale read-heavy workloads.
- Implement sharding only when vertical scaling is insufficient.
- Use Redis for distributed caching and session storage.
- Configure connection pooling to reduce database overhead.
- Apply appropriate indexing strategies based on query patterns.
- Monitor replication lag and slow queries continuously.
- Automate backups and regularly test recovery procedures.
- Store static assets in object storage such as Amazon S3.
- Design storage and caching layers to reduce database load and improve application performance.

---

# Summary

This section covered SQL vs NoSQL databases, ACID and BASE models, replication, read replicas, sharding, partitioning, indexing, connection pooling, Redis, Memcached, caching strategies, CDN caching, storage architecture, database monitoring, disaster recovery, and production database best practices. These concepts provide the foundation for designing scalable and resilient data platforms.