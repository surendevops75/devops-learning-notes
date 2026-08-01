# Amazon Elastic Load Balancer (ELB)

---

# Introduction

Amazon Elastic Load Balancing (ELB) is a fully managed AWS service that automatically distributes incoming application traffic across multiple targets such as EC2 instances, containers, IP addresses, Lambda functions, and Kubernetes services.

Instead of sending all client requests to a single server, ELB intelligently routes traffic to healthy backend resources. This improves availability, scalability, fault tolerance, and application performance.

Elastic Load Balancing is one of the core services used in almost every production AWS architecture.

---

# What is Elastic Load Balancer?

Elastic Load Balancer acts as the entry point for client requests.

Instead of clients connecting directly to backend servers, they connect to the Load Balancer.

The Load Balancer then decides which backend resource should process each request.

```text
Users

↓

Elastic Load Balancer

↓

EC2-1

EC2-2

EC2-3
```

---

# Why Do We Need a Load Balancer?

Imagine an application with only one web server.

```text
Users

↓

EC2
```

Problems:

- Single Point of Failure
- Cannot Handle High Traffic
- Difficult Maintenance
- Poor Availability

Now imagine multiple servers.

```text
Users

↓

Load Balancer

↓

EC2-1

EC2-2

EC2-3
```

Advantages:

- High Availability
- Better Performance
- Automatic Distribution
- Fault Tolerance
- Zero Downtime Deployments

---

# Real-World Problem

An e-commerce company receives:

- 2 Million Users
- 40 Million API Requests
- Flash Sale Traffic
- Continuous Deployments

Requirements:

- No downtime
- High availability
- SSL termination
- Automatic scaling
- Secure traffic routing

Elastic Load Balancer solves these challenges.

---

# Complete Architecture

```text
                        Internet

                            │

                      Amazon Route53

                            │

                    AWS WAF (Optional)

                            │

                 Amazon Application Load Balancer

                            │

          ┌─────────────────┼─────────────────┐

          │                 │                 │

      EC2 Instance      EC2 Instance      EC2 Instance

          │                 │                 │

      Target Group      Target Group      Target Group

          │                 │                 │

          └─────────────────┼─────────────────┘

                    Amazon RDS

                         │

                  CloudWatch

                         │

                     Auto Scaling
```

---

# Internal Working

When a client sends a request:

```text
Browser

↓

DNS Resolution

↓

Route53

↓

Elastic Load Balancer

↓

Listener

↓

Listener Rule

↓

Target Group

↓

Healthy EC2

↓

Response
```

The client never communicates directly with backend servers.

---

# Benefits

Amazon ELB provides:

- High Availability
- Automatic Scaling
- Health Monitoring
- SSL Offloading
- Request Routing
- Cross-Zone Distribution
- Sticky Sessions
- Integration with Auto Scaling
- Kubernetes Support
- CloudWatch Monitoring

---

# Types of Load Balancers

AWS provides four types of Load Balancers.

| Type | Layer | Protocol |
|--------|-------|----------|
| Application Load Balancer | Layer 7 | HTTP / HTTPS |
| Network Load Balancer | Layer 4 | TCP / UDP / TLS |
| Gateway Load Balancer | Layer 3 | Network Appliances |
| Classic Load Balancer | Layer 4 & 7 | Legacy |

---

# Application Load Balancer (ALB)

Application Load Balancer operates at Layer 7 of the OSI Model.

Supports:

- HTTP
- HTTPS
- WebSocket
- HTTP/2

Best suited for:

- Web Applications
- REST APIs
- Kubernetes Ingress
- Microservices

---

# ALB Request Flow

```text
User

↓

HTTPS

↓

ALB

↓

Listener

↓

Listener Rule

↓

Target Group

↓

Application
```

---

# Features of ALB

- Host-Based Routing
- Path-Based Routing
- HTTP Header Routing
- Query Parameter Routing
- Redirects
- Authentication
- WebSocket Support
- HTTP/2 Support

---

# Host-Based Routing

Different domains can be routed to different applications.

Example

```text
shop.company.com

↓

Target Group A

------------------------

api.company.com

↓

Target Group B

------------------------

admin.company.com

↓

Target Group C
```

Useful for:

- Microservices
- Multi-domain applications

---

# Path-Based Routing

Routes requests based on URL paths.

Example

```text
/api/*

↓

API Servers

-------------------

/images/*

↓

Image Servers

-------------------

/admin/*

↓

Admin Application
```

Very common in enterprise applications.

---

# Network Load Balancer (NLB)

Network Load Balancer operates at Layer 4.

Supports:

- TCP
- UDP
- TLS

Best suited for:

- Gaming
- Financial Applications
- VoIP
- High-performance APIs

Advantages

- Ultra-low latency
- Millions of requests
- Static IP
- Elastic IP support

---

# Gateway Load Balancer (GWLB)

Gateway Load Balancer is used to deploy third-party virtual appliances.

Examples:

- Firewalls
- IDS
- IPS
- Network Monitoring

Architecture

```text
Internet

↓

Gateway Load Balancer

↓

Firewall Appliance

↓

Application
```

---

# Classic Load Balancer

Classic Load Balancer is the original AWS load balancer.

Supports:

- HTTP
- HTTPS
- TCP

Used only for legacy applications.

AWS recommends ALB or NLB for new deployments.

---

# Choosing the Right Load Balancer

```text
Need HTTP/HTTPS?

│

├── Yes

│      ↓

│  Application Load Balancer

│

└── No

       │

Need TCP/UDP?

│

├── Yes

│      ↓

│ Network Load Balancer

│

└── Need Firewall?

↓

Gateway Load Balancer
```

---

# Listeners

A Listener checks incoming requests and forwards them to appropriate Target Groups.

Example

| Protocol | Port |
|----------|------|
| HTTP | 80 |
| HTTPS | 443 |

Example

```text
HTTPS 443

↓

Listener

↓

Forward to Target Group
```

---

# Listener Rules

Listener Rules determine where traffic should go.

Rule Components

- Path
- Host
- Header
- Query String
- Source IP

Example

```text
/api/*

↓

API Target Group

-------------------

/images/*

↓

Image Target Group
```

---

# Target Groups

A Target Group contains backend resources.

Targets may include:

- EC2 Instances
- IP Addresses
- Lambda Functions
- ECS Tasks

Workflow

```text
ALB

↓

Target Group

↓

Healthy Targets
```

---

# Target Types

| Target Type | Use Case |
|--------------|----------|
| Instance | EC2 |
| IP | Containers |
| Lambda | Serverless |

---

# Health Checks

ELB continuously checks backend health.

Example

```
GET /health

↓

HTTP 200

↓

Healthy
```

If health checks fail:

```text
EC2

↓

Health Check Failed

↓

Removed from Rotation
```

Traffic automatically shifts to healthy targets.

---

# Health Check Parameters

| Parameter | Description |
|------------|-------------|
| Protocol | HTTP / HTTPS / TCP |
| Port | Health Check Port |
| Path | /health |
| Interval | Check Frequency |
| Timeout | Wait Time |
| Healthy Threshold | Success Count |
| Unhealthy Threshold | Failure Count |

---

# Cross-Zone Load Balancing

Normally

```text
AZ-A

↓

Traffic

↓

AZ-A Instances
```

With Cross-Zone enabled

```text
AZ-A

↓

Traffic

↓

AZ-A

↓

AZ-B

↓

AZ-C
```

Benefits

- Better utilization
- Even traffic distribution
- Higher availability

---

# Sticky Sessions

Sticky Sessions ensure the same client continues to communicate with the same backend server.

Workflow

```text
Client

↓

ALB Cookie

↓

EC2-2

↓

Next Request

↓

EC2-2
```

Useful for:

- Legacy Applications
- Shopping Carts
- Session-based Applications

Not recommended for stateless microservices.

---

# SSL/TLS Termination

Instead of decrypting HTTPS traffic on every EC2 instance:

```text
Client

↓

HTTPS

↓

ALB

↓

Decrypt

↓

HTTP

↓

EC2
```

Benefits

- Lower CPU usage
- Simpler certificate management
- Better security
- Centralized encryption

---

# AWS Certificate Manager (ACM)

ACM manages SSL/TLS certificates.

Benefits

- Free AWS certificates
- Automatic renewal
- Easy ALB integration
- No manual certificate installation

Production Recommendation:

Always use ACM certificates instead of manually installing certificates on EC2 instances.

---

# HTTP to HTTPS Redirection

Best practice:

```text
HTTP

↓

Redirect

↓

HTTPS
```

Ensures encrypted communication for all users.

---

# WebSocket Support

ALB supports:

- WebSocket
- HTTP/2

Suitable for:

- Chat Applications
- Live Notifications
- Gaming
- Real-time Dashboards

---

