# Containerization and Application Evolution

## Introduction

Over the years, application architecture has evolved significantly.

Applications moved from:

```text
Monolithic Architecture
        ↓
Microservices Architecture
        ↓
Containerized Applications
```

---

Containerization emerged to solve deployment, scalability, and portability challenges faced by traditional applications.

---

# Evolution of Applications

## Traditional Enterprise Applications

Earlier applications were built as:

```text
Single Deployable Unit
```

Examples:

```text
Java J2EE Applications

.NET Applications
```

---

Technologies Used:

```text
Servlets

JSP (Java Server Pages)

Struts

Seam Framework

Spring Framework
```

---

Application Packaging:

```text
WAR Files

EAR Files
```

---

Examples:

```text
roboshop.war

roboshop.ear
```

---

Typical Size:

```text
150 MB – 200 MB
```

---

# What is a Monolithic Application?

A Monolithic Application is an application where:

```text
Frontend

Backend

Business Logic

Database Access
```

are packaged together.

---

Architecture

```text
Users
  ↓
Frontend
  ↓
Backend
  ↓
Database
```

---

Single Deployment Unit

```text
application.war
```

---

# Characteristics of Monolithic Applications

## Advantages

```text
Simple Initial Development

Simple Deployment

Easy To Test Initially
```

---

## Disadvantages

### Entire Application Must Be Rebuilt

Small change:

```text
Frontend Change
```

requires:

```text
Full Build

Full Deployment

Full Testing
```

---

### Technology Restriction

Entire application generally uses:

```text
Single Programming Language
```

Example:

```text
Java Frontend

Java Backend
```

---

### Slow Release Cycle

Deployment Flow:

```text
DEV
 ↓
SIT
 ↓
UAT
 ↓
PROD
```

---

A small change may take:

```text
Weeks
```

to reach production.

---

# Production Release Process

Typical Stakeholders

```text
Business Analyst

Delivery Manager

Architects

Technical Leads

Developers

Operations Team
```

---

Example Production Support Window

```text
9 PM – 6 AM
```

---

Incident Bridge Call

Participants:

```text
L1 Support

L2 Support

L3 Support

Developers

Monitoring Team

Service Desk

Client Teams
```

---

# Application Servers

Monolithic Java applications typically run on:

```text
Apache Tomcat

JBoss

WildFly

WebLogic

WebSphere
```

---

# What is Microservices Architecture?

Microservices break a large application into:

```text
Small Independent Services
```

---

Example Roboshop

```text
Catalogue

Cart

User

Shipping

Payment
```

---

Architecture

```text
Frontend
     ↓
API Calls
     ↓
Catalogue Service

Cart Service

User Service

Payment Service
```

---

# Characteristics of Microservices

Each service can have:

```text
Independent Code

Independent Deployment

Independent Scaling
```

---

Different services can use different technologies.

Example:

```text
Catalogue → NodeJS

Cart → Java

User → Python

Payment → Go
```

---

# Advantages of Microservices

## Independent Deployment

Deploy:

```text
Catalogue Only
```

without affecting:

```text
Cart

User

Payment
```

---

## Faster Releases

Smaller services:

```text
Faster Build

Faster Deployment

Faster Rollback
```

---

## Better Scalability

Scale only:

```text
Cart Service
```

during heavy traffic.

---

## Technology Freedom

Each team can choose:

```text
Best Programming Language
```

for their service.

---

# Challenges of Microservices

```text
Distributed Systems

Service Discovery

Networking

Monitoring

Logging

Security
```

---

# Why Containers Became Popular?

Microservices introduced:

```text
Many Small Services
```

---

Running each service on a VM becomes expensive.

---

Example

```text
20 Microservices
```

requires:

```text
20 VMs
```

---

Problems:

```text
High Cost

Large Resource Usage

Slow Boot Time
```

---

Solution:

```text
Containers
```

---

# Containerization

Containerization packages:

```text
Application

Runtime

Libraries

Dependencies
```

into a lightweight unit.

---

Benefits

```text
Fast Startup

Portability

Consistency

Resource Efficiency
```

---

# Real Production Example

Roboshop Microservices

```text
Catalogue

Cart

User

Shipping

Payment
```

---

Instead of:

```text
5 Virtual Machines
```

---

Run as:

```text
5 Containers
```

on a small number of servers.

---

Result:

```text
Lower Cost

Faster Deployment

Better Resource Utilization
```

---

# Common Interview Questions

## What is a Monolithic Application?

### Short Answer

A monolithic application packages all functionality into a single deployable unit.

### Detailed Explanation

Frontend, backend, business logic, and database access are tightly coupled and deployed together.

### Production Example

Traditional Java WAR applications running on Tomcat.

### Follow-Up Questions

- What are the limitations of monolithic applications?
- Why did microservices emerge?

---

## What are Microservices?

### Short Answer

Microservices are small, independently deployable services that work together to form an application.

### Detailed Explanation

Each service owns a specific business capability and can be developed, deployed, and scaled independently.

### Production Example

Roboshop services such as Catalogue, Cart, and User.

### Follow-Up Questions

- What are the benefits of microservices?
- What challenges do microservices introduce?

---

## Why Did Containerization Become Popular?

### Short Answer

Containers provide lightweight, portable environments for running microservices efficiently.

### Detailed Explanation

Containers reduce infrastructure costs, improve portability, and enable faster deployments.

### Production Example

Running multiple Roboshop services as containers instead of separate virtual machines.

### Follow-Up Questions

- How are containers different from VMs?
- Why are containers important for Kubernetes?

---

# Key Takeaways

```text
Monolithic applications package everything into one deployable unit.

Microservices split applications into independent services.

Microservices improve scalability and deployment flexibility.

Traditional enterprise applications commonly used WAR and EAR files.

Microservices increased the need for lightweight deployment units.

Containers provide portability, consistency, and efficient resource utilization.

Containerization is a foundational technology for modern DevOps and Kubernetes.
```