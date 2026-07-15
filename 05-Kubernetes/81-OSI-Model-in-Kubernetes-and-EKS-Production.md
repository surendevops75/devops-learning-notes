# OSI Model in Kubernetes and EKS Production

## Introduction

Most networking tutorials explain the OSI model using:

```text
Laptop

↓

Switch

↓

Router

↓

Server
```

---

However as a DevOps Engineer working on EKS, you troubleshoot:

```text
Route53

↓

ALB

↓

Ingress

↓

Service

↓

Pod
```

---

Understanding OSI layers in the context of Kubernetes helps answer questions like:

```text
Why is my application unreachable?

Why is DNS failing?

Why is TLS failing?

Why is traffic not reaching Pods?
```

---

# What Is OSI Model?

OSI stands for:

```text
Open Systems Interconnection
```

---

It divides networking into:

```text
Layer 7 - Application

Layer 6 - Presentation

Layer 5 - Session

Layer 4 - Transport

Layer 3 - Network

Layer 2 - Data Link

Layer 1 - Physical
```

---

# Why DevOps Engineers Need OSI?

When troubleshooting production issues:

```text
Application Failure

↓

TLS Failure

↓

Service Failure

↓

Networking Failure
```

---

OSI helps identify:

```text
Which Layer Is Broken?
```

---

# Typical EKS Traffic Flow

```text
User

↓

Route53

↓

ALB

↓

Ingress

↓

Service

↓

Pod
```

---

Let's map each component to OSI layers.

---

# Layer 7 - Application Layer

## Purpose

Responsible for:

```text
Application Communication

HTTP

HTTPS

REST APIs

DNS
```

---

This is the layer most DevOps Engineers interact with.

---

# EKS Components At Layer 7

```text
Ingress

Gateway API

HTTPRoute

CoreDNS

ALB Host Routing

ALB Path Routing

REST APIs
```

---

# Example

Request

```text
https://catalogue.roboshop.com
```

---

ALB Reads

```text
Host Header
```

---

Routes To

```text
Catalogue Service
```

---

This is:

```text
Layer 7 Routing
```

---

# Ingress Example

```yaml
rules:

- host: catalogue.roboshop.com
```

---

Ingress reads:

```text
HTTP Host Header
```

---

Layer 7 functionality.

---

# Gateway API Example

```text
HTTPRoute
```

---

Routes Based On

```text
Host

Path

Headers
```

---

Also Layer 7.

---

# CoreDNS

Application

```text
Catalogue
```

needs:

```text
MongoDB
```

---

Uses

```text
mongodb
```

instead of:

```text
10.0.1.25
```

---

CoreDNS resolves:

```text
mongodb

↓

ClusterIP
```

---

DNS belongs to:

```text
Layer 7
```

---

# Layer 6 - Presentation Layer

## Purpose

Responsible For

```text
Encryption

Decryption

SSL

TLS

Certificates
```

---

# EKS Components At Layer 6

```text
TLS Certificates

AWS ACM

HTTPS

SSL Handshake
```

---

# Example

User Opens

```text
https://catalogue.roboshop.com
```

---

Browser Starts

```text
TLS Handshake
```

---

ALB Presents

```text
Certificate
```

---

Certificate Issued By

```text
ACM
```

---

This is Layer 6.

---

# Production Example

Certificate Expired

↓

Browser Error

↓

Application Unreachable

---

Issue Exists At

```text
Layer 6
```

---

# Layer 5 - Session Layer

## Purpose

Responsible For

```text
Session Management

Connection Persistence

Session Tracking
```

---

# Examples

```text
Login Sessions

Shopping Cart Sessions

Sticky Sessions
```

---

# Production Example

User Logs In

↓

Receives Session

↓

Multiple API Calls

↓

Session Maintained

---

# ALB Sticky Sessions

ALB Can Maintain:

```text
User Session

↓

Same Backend
```

---

Layer 5 responsibility.

---

# Layer 4 - Transport Layer

## Purpose

Responsible For

```text
TCP

UDP

Ports
```

---

# EKS Components At Layer 4

```text
Services

NodePorts

NLB

Container Ports

Service Ports
```

---

# Service Example

```yaml
ports:

- port: 80

  targetPort: 8080
```

---

Traffic

```text
Port 80

↓

Port 8080
```

---

Layer 4 operation.

---

# TCP Communication

Example

```text
Catalogue

↓

MongoDB
```

---

Uses

```text
TCP Connection
```

---

Layer 4.

---

# NLB

Network Load Balancer

Operates At:

```text
Layer 4
```

---

Routes Based On

```text
TCP

UDP
```

---

Unlike ALB which operates at Layer 7.

---

# Layer 3 - Network Layer

## Purpose

Responsible For

```text
IP Addressing

Routing

Network Paths
```

---

# EKS Components At Layer 3

```text
VPC

Subnets

Route Tables

Security Groups

Pod IPs

ENIs

VPC CNI
```

---

# Pod Networking Example

Pod

```text
10.0.1.25
```

---

MongoDB

```text
10.0.2.14
```

---

Traffic

```text
10.0.1.25

↓

10.0.2.14
```

---

Layer 3 Routing.

---

# VPC CNI

Responsible For

```text
Assigning Pod IPs
```

---

Example

```text
VPC CIDR

10.0.0.0/16
```

---

Pod Receives

```text
10.0.1.25
```

---

Layer 3 operation.

---

# Security Groups

Control

```text
IP Based Access
```

---

Example

```text
ALB

↓

Node

Allowed
```

---

Layer 3.

---

# Route Tables

Responsible For

```text
Traffic Routing
```

---

Example

```text
Public Subnet

↓

Internet Gateway
```

---

Layer 3.

---

# Layer 2 - Data Link Layer

## Purpose

Responsible For

```text
MAC Address

Frame Delivery

Local Network Communication
```

---

# EKS Components

Mostly Managed By AWS.

---

Examples

```text
ENI

MAC Address

AWS Internal Networking
```

---

Rarely Troubleshooted Directly.

---

# ENI Example

Node

↓

ENI

↓

Secondary IPs

↓

Pods

---

ENI operates at Layer 2 and Layer 3.

---

# Layer 1 - Physical Layer

## Purpose

Responsible For

```text
Physical Hardware
```

---

Examples

```text
Servers

Switches

Fiber

Cables

Network Cards
```

---

In AWS

```text
Managed By AWS
```

---

Not Visible To EKS Users.

---

# Complete OSI Mapping For EKS

## Layer 7

```text
Ingress

Gateway API

HTTPRoute

CoreDNS

ALB Routing

REST APIs
```

---

## Layer 6

```text
TLS

SSL

Certificates

ACM
```

---

## Layer 5

```text
Sessions

Sticky Sessions

Authentication Sessions
```

---

## Layer 4

```text
TCP

UDP

Services

NodePort

NLB
```

---

## Layer 3

```text
VPC

Subnets

Pod IPs

ENIs

Security Groups

Route Tables

VPC CNI
```

---

## Layer 2

```text
MAC Addresses

ENI Internal Networking
```

---

## Layer 1

```text
AWS Physical Hardware
```

---

# Complete User To Pod Flow

```text
User

↓

Route53

↓

ALB

↓

Ingress

↓

Service

↓

Pod
```

---

OSI Mapping

```text
Route53      → Layer 7

ALB Routing  → Layer 7

TLS          → Layer 6

Session      → Layer 5

Service Port → Layer 4

Pod IP       → Layer 3

ENI          → Layer 2

AWS Hardware → Layer 1
```

---

# Production Troubleshooting Using OSI

## Problem

```text
https://catalogue.roboshop.com

Not Working
```

---

# Check Layer 7

```text
Ingress Correct?

Host Rule Correct?

Service Exists?
```

---

Commands

```bash
kubectl get ingress
kubectl get svc
```

---

# Check Layer 6

```text
Certificate Valid?

Certificate Expired?
```

---

Verify

```text
ACM Certificate
```

---

# Check Layer 5

```text
Session Issues?

Sticky Session Issues?
```

---

# Check Layer 4

```text
Service Port Correct?

Target Port Correct?
```

---

Commands

```bash
kubectl describe svc
```

---

# Check Layer 3

```text
Pod IP Reachable?

Security Groups Correct?

Subnet Routing Correct?
```

---

Commands

```bash
kubectl get pods -o wide
```

---

# Common Interview Questions

## Which OSI Layer Does Ingress Operate On?

### Answer

```text
Layer 7
```

Because it performs:

```text
Host Routing

Path Routing
```

---

## Which OSI Layer Does ALB Operate On?

### Answer

```text
Layer 7
```

Because it understands:

```text
HTTP

HTTPS

Headers

Paths
```

---

## Which OSI Layer Does NLB Operate On?

### Answer

```text
Layer 4
```

Because it routes:

```text
TCP

UDP
```

---

## Which OSI Layer Does Kubernetes Service Operate On?

### Answer

```text
Layer 4
```

Because it routes traffic based on:

```text
Ports

TCP

UDP
```

---

## Which OSI Layer Does VPC CNI Operate On?

### Answer

Primarily:

```text
Layer 3
```

because it manages:

```text
IP Addresses

Routing
```

---

# Key Takeaways

```text
OSI helps identify networking issues systematically.

Ingress and Gateway API operate at Layer 7.

TLS and ACM belong to Layer 6.

Sessions and sticky sessions belong to Layer 5.

Services and NLB operate at Layer 4.

VPC, Subnets, Pod IPs and VPC CNI operate at Layer 3.

ENIs primarily operate at Layer 2.

AWS hardware represents Layer 1.

Understanding OSI mapping makes Kubernetes troubleshooting much easier in production.
```