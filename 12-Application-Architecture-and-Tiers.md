# Application Architecture and Tier Architecture

## Introduction

Application Architecture defines how different components of an application interact with each other.

As applications grow, responsibilities are separated into multiple layers (tiers) to improve:

- Scalability
- Maintainability
- Security
- Performance
- Availability

This concept is called:

```text
Decoupling
```

or

```text
Loose Coupling
```

---

# Desktop Applications vs Web Applications

## Desktop Applications

Applications installed directly on a user's machine.

Examples:

- Calculator
- MS Office
- Photoshop
- VLC Media Player

### Characteristics

- Installed on every machine
- Uses local CPU and RAM
- Data stored locally
- Limited accessibility

### Challenges

#### High Resource Consumption

```text
CPU
RAM
Storage
```

used locally.

---

#### Installation Required

Every system requires:

```text
Install
Upgrade
Maintenance
```

---

#### System Crashes

Application may stop working if:

```text
Operating System Failure
Hardware Failure
```

occurs.

---

#### Data Loss

Data is often stored locally.

```text
System Crash
      ↓
Possible Data Loss
```

---

#### Limited Accessibility

Can only be used on installed machines.

---

#### Security Issues

Data remains on local devices.

---

# Web Applications

Applications accessed through a browser.

Examples:

- Facebook
- Gmail
- LinkedIn
- Netflix
- Amazon

### Characteristics

- Centralized deployment
- Browser-based access
- Accessible anywhere
- Easier upgrades

---

### Advantages

#### No Installation

Users only need:

```text
Browser
Internet Connection
```

---

#### Centralized Management

Application updates happen on servers.

---

#### Better Security

Data resides in centralized infrastructure.

---

#### Accessible Anywhere

Can access from:

- Mobile
- Laptop
- Tablet
- Desktop

---

#### Easy Maintenance

Server update benefits all users.

---

# Evolution of Application Architectures

Applications evolved from:

```text
1-Tier
   ↓
2-Tier
   ↓
3-Tier
   ↓
Microservices
```

---

# 1-Tier Architecture

## Real World Example

Roadside Hotel

Responsibilities:

```text
Cooking
Order Management
Money Collection
Customer Interaction
```

handled by a single person.

---

### Software Equivalent

Everything runs on one machine.

```text
Application
Database
User Interface
```

all in same system.

---

### Architecture

```text
┌───────────────────┐
│ Application       │
│ Database          │
│ User Interface    │
└───────────────────┘
```

---

### Advantages

- Simple
- Cheap
- Easy Setup

---

### Disadvantages

- Poor scalability
- Difficult maintenance
- Single point of failure
- Limited users

---

# 2-Tier Architecture

## Real World Example

Small Hotel

Responsibilities divided.

```text
Owner
   ↓
Cook
```

Owner manages customers.

Cook prepares food.

---

### Software Equivalent

```text
Client
   │
   ▼
Database Server
```

or

```text
Database
Application + UI
```

---

### Architecture

```text
┌─────────────┐
│ Client      │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Database    │
└─────────────┘
```

---

### Advantages

- Better than 1-tier
- Easier management

---

### Disadvantages

- Database becomes bottleneck
- Limited scalability

---

# 3-Tier Architecture

## Real World Example

Restaurant

### Captain

```text
Welcomes Customer
Assigns Table
```

---

### Waiter

```text
Takes Orders
Delivers Food
Handles Communication
```

---

### Chef

```text
Converts Raw Ingredients
into Ready Food
```

---

### Flow

```text
Customer
    ↓
Captain
    ↓
Waiter
    ↓
Chef
```

---

# Software Equivalent

Raw data:

```text
SQL
NoSQL
```

Application Server:

```text
Java
Python
.NET
NodeJS
Go
PHP
```

Presentation Layer:

```text
Nginx
Apache
Web UI
```

Load Balancer:

```text
Nginx
HAProxy
AWS ALB
AWS ELB
```

---

# 3-Tier Architecture Components

```text
Client
   │
   ▼
Load Balancer
   │
   ▼
Web Tier
   │
   ▼
Application Tier
   │
   ▼
Database Tier
```

---

# Web Tier (Frontend Tier)

Responsible for:

- User Interface
- Static Content
- Request Routing

---

### Technologies

```text
HTML
CSS
JavaScript
React
Angular
Vue
```

---

### Web Servers

```text
Nginx
Apache HTTPD
```

---

### Responsibilities

- Serve static content
- Route requests
- SSL termination
- Reverse proxy

---

# Application Tier

Also called:

```text
Application Layer
Backend Layer
Middleware Layer
Business Layer
```

---

### Responsibilities

- Business Logic
- CRUD Operations
- Authentication
- Authorization
- API Processing

---

### Technologies

```text
Java
Python
NodeJS
.NET
Go
PHP
Ruby
```

---

### CRUD Operations

| Operation | Meaning |
|------------|----------|
| Create | Insert Data |
| Read | Retrieve Data |
| Update | Modify Data |
| Delete | Remove Data |

---

# Database Tier

Responsible for:

```text
Data Storage
Data Retrieval
Data Integrity
```

---

# SQL Databases

Structured data.

Examples:

- MySQL
- Oracle
- SQL Server
- PostgreSQL

---

### Characteristics

- Tables
- Rows
- Columns
- ACID Compliance

---

# NoSQL Databases

Flexible schema.

Examples:

- MongoDB
- Cassandra
- DynamoDB

---

### Characteristics

- High scalability
- Flexible structure
- Distributed architecture

---

# Redis

In-Memory Database.

Uses:

- Caching
- Session Storage
- Rate Limiting
- Fast Data Retrieval

---

### Benefits

```text
Very Fast
Low Latency
High Performance
```

---

# Message Queues

Used for asynchronous communication.

---

## Why Message Queues?

Without Queue:

```text
Application A
      │
      ▼
Application B
```

If B is down:

```text
Failure
```

---

With Queue:

```text
Application A
      │
      ▼
Message Queue
      │
      ▼
Application B
```

Messages are stored until processing.

---

# Popular Message Queues

## ActiveMQ

Apache messaging platform.

Used in:

- Java Applications
- Enterprise Systems

---

## WebSphere MQ

IBM Message Queue solution.

Used in:

- Banking
- Insurance
- Enterprise Systems

---

## Modern Alternatives

- RabbitMQ
- Apache Kafka
- AWS SQS
- Google Pub/Sub

---

# Load Balancer

Acts as traffic distributor.

---

## Example

1000 Requests

Without Load Balancer:

```text
Server 1
1000 Requests
```

---

With Load Balancer:

```text
           1000 Requests
                  │
                  ▼
          Load Balancer
             │      │
             ▼      ▼
        Server1  Server2
          500      500
```

---

## Benefits

- High Availability
- Scalability
- Fault Tolerance
- Better Performance

---

# Decoupling

Separating responsibilities into independent components.

Example:

```text
Database
Application
Web Server
```

separated into different servers.

---

## Benefits

- Independent Scaling
- Easier Maintenance
- Better Security
- Better Performance

---

# Loose Coupling

Components communicate through well-defined interfaces.

Example:

```text
Application
      │
      ▼
API
      │
      ▼
Database
```

Changes in one layer have minimal impact on others.

---

# Traditional Monolithic Architecture

```text
┌────────────────────────┐
│ UI                     │
│ Business Logic         │
│ Database Access        │
└────────────────────────┘
```

Everything deployed together.

---

## Challenges

- Difficult scaling
- Difficult deployments
- Large codebase
- Single point of failure

---

# Introduction to Microservices

Modern applications break functionality into smaller services.

Example:

```text
User Service
Product Service
Cart Service
Order Service
Payment Service
Notification Service
```

Each service:

- Independent
- Deployable
- Scalable

---

# Example: E-Commerce Architecture

```text
Users
   │
   ▼
Load Balancer
   │
   ▼
Web Tier (Nginx)
   │
   ▼
Application Tier
   │
   ├── User Service
   ├── Product Service
   ├── Cart Service
   ├── Order Service
   └── Payment Service
   │
   ▼
Database Tier
```

---

# Cloud-Native Architecture

Modern architectures often include:

```text
Load Balancer
     │
     ▼
Kubernetes
     │
     ▼
Microservices
     │
     ▼
Databases
     │
     ▼
Redis
     │
     ▼
Message Queues
```

---

# Frequently Asked Interview Questions

## What is 1-Tier Architecture?

Application, database and UI exist in same system.

---

## What is 2-Tier Architecture?

Client communicates directly with database.

---

## What is 3-Tier Architecture?

Application separated into:

- Web Tier
- Application Tier
- Database Tier

---

## What is Decoupling?

Separating components to reduce dependency.

---

## What is Loose Coupling?

Components interact through defined interfaces while remaining independent.

---

## What is Load Balancing?

Distributing traffic across multiple servers.

---

## Difference Between SQL and NoSQL

| SQL | NoSQL |
|------|--------|
| Structured | Flexible |
| Tables | Documents/Key-Value |
| ACID | High Scalability |
| MySQL | MongoDB |

---

## Why Use Redis?

- Caching
- Session Management
- Faster Data Access

---

## Why Use Message Queues?

To enable asynchronous communication and improve reliability.

---

# Key Takeaways

- Web applications are more scalable than desktop applications.
- 1-tier architecture is simple but limited.
- 2-tier separates clients and databases.
- 3-tier introduces Web, Application and Database layers.
- Load balancers distribute traffic.
- Web Tier handles user requests.
- Application Tier contains business logic.
- Database Tier stores data.
- Redis improves performance through caching.
- Message queues enable asynchronous processing.
- Decoupling and loose coupling improve scalability and maintainability.
- Modern cloud applications use microservices and distributed architectures.