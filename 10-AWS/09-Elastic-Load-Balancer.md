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

---

# Access Logs

Elastic Load Balancer can store access logs in an Amazon S3 bucket.

Access logs help you analyze:

- Client IP Address
- Request Path
- Request Time
- HTTP Method
- Response Code
- Backend Target
- Processing Time
- User Agent
- TLS Version

Architecture

```text
Client

↓

Application Load Balancer

↓

Access Log

↓

Amazon S3

↓

Athena / CloudWatch / SIEM
```

Enterprise Use Cases

- Security Auditing
- Request Analysis
- Performance Analysis
- Compliance
- Incident Investigation

---

# CloudWatch Monitoring

Elastic Load Balancer automatically publishes metrics to Amazon CloudWatch.

Important Metrics

| Metric | Description |
|---------|-------------|
| RequestCount | Total Requests |
| TargetResponseTime | Backend Response Time |
| HealthyHostCount | Healthy Targets |
| UnHealthyHostCount | Unhealthy Targets |
| HTTPCode_ELB_4XX | Client Errors |
| HTTPCode_ELB_5XX | Load Balancer Errors |
| HTTPCode_Target_5XX | Backend Errors |
| ActiveConnectionCount | Active Connections |
| NewConnectionCount | New Connections |

Recommended CloudWatch Alarms

- High Response Time
- High 5XX Errors
- Low Healthy Host Count
- Sudden Traffic Spike
- Target Failures

---

# Security Groups

Application Load Balancer supports Security Groups.

Example

Inbound

| Source | Port |
|----------|------|
| Internet | 443 |

Outbound

| Destination | Port |
|-------------|------|
| Application SG | 8080 |

Best Practice

Never allow backend EC2 instances to accept traffic directly from the Internet.

Instead:

```text
Internet

↓

ALB Security Group

↓

Application Security Group

↓

EC2
```

This ensures that only the ALB communicates with backend instances.

---

# Integration with EC2

Application Load Balancer distributes traffic across multiple EC2 instances.

Architecture

```text
Users

↓

ALB

↓

Target Group

↓

EC2-1

EC2-2

EC2-3
```

Benefits

- Automatic Load Distribution
- High Availability
- Health Monitoring

---

# Integration with Auto Scaling

One of the biggest advantages of ELB is its integration with Auto Scaling Groups.

Workflow

```text
Users

↓

Application Load Balancer

↓

Auto Scaling Group

↓

EC2 Instances
```

Scaling Out

```text
CPU > 70%

↓

Auto Scaling

↓

Launch EC2

↓

Register Target

↓

Receive Traffic
```

Scaling In

```text
Low CPU

↓

Terminate EC2

↓

Deregister Target

↓

Traffic Redistributed
```

This process is automatic.

---

# Connection Draining (Deregistration Delay)

Before terminating an instance,

ALB waits until existing requests finish.

Workflow

```text
Instance Selected

↓

Stop New Requests

↓

Complete Existing Requests

↓

Remove Target
```

Prevents user disruption.

---

# Integration with ECS

ALB integrates directly with Amazon ECS.

Architecture

```text
Users

↓

ALB

↓

Target Group

↓

ECS Tasks

↓

Containers
```

Useful for:

- Microservices
- Docker Containers
- API Services

---

# Integration with Amazon EKS

Amazon EKS uses the AWS Load Balancer Controller.

Architecture

```text
Users

↓

Route53

↓

Application Load Balancer

↓

Ingress

↓

Service

↓

Pods
```

The controller automatically:

- Creates ALB
- Creates Target Groups
- Registers Pods
- Updates Rules

Supported Features

- Path Routing
- Host Routing
- SSL
- WAF
- ACM
- Sticky Sessions

---

# Ingress Example

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: application

spec:

  ingressClassName: alb

  rules:

  - host: app.company.com

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: application

            port:

              number: 80
```

---

# Enterprise Production Architecture

```text
                           Internet

                               │

                           Route53

                               │

                          AWS WAF

                               │

                 Application Load Balancer

                               │

        ┌──────────────────────┼──────────────────────┐

        │                      │                      │

   EC2 ASG (AZ-A)        EC2 ASG (AZ-B)        EC2 ASG (AZ-C)

        │                      │                      │

        └──────────────────────┼──────────────────────┘

                        Amazon RDS

                             │

                    CloudWatch

                             │

                      Amazon SNS
```

---

# AWS Console Walkthrough

1. Open EC2 Console

2. Select **Load Balancers**

3. Click **Create Load Balancer**

4. Choose

- Application
- Network
- Gateway

5. Configure

- VPC
- Subnets
- Security Group
- Listeners

6. Create Target Group

7. Register Targets

8. Configure Health Check

9. Review

10. Create

---

# AWS CLI

Create Load Balancer

```bash
aws elbv2 create-load-balancer \
--name production-alb \
--subnets subnet-123 subnet-456
```

Create Target Group

```bash
aws elbv2 create-target-group \
--name app-targets \
--protocol HTTP \
--port 80 \
--vpc-id vpc-123
```

Register Targets

```bash
aws elbv2 register-targets \
--target-group-arn arn:aws:... \
--targets Id=i-123456
```

Describe Load Balancers

```bash
aws elbv2 describe-load-balancers
```

Describe Target Groups

```bash
aws elbv2 describe-target-groups
```

Describe Target Health

```bash
aws elbv2 describe-target-health \
--target-group-arn arn:aws:...
```

Delete Load Balancer

```bash
aws elbv2 delete-load-balancer \
--load-balancer-arn arn:aws:...
```

---

# Terraform

Create ALB

```hcl
resource "aws_lb" "application" {

  name = "production-alb"

  internal = false

  load_balancer_type = "application"

  security_groups = [

    aws_security_group.alb.id

  ]

  subnets = [

    aws_subnet.public_a.id,

    aws_subnet.public_b.id

  ]

}
```

Target Group

```hcl
resource "aws_lb_target_group" "application" {

  name = "app"

  port = 80

  protocol = "HTTP"

  vpc_id = aws_vpc.main.id

}
```

Listener

```hcl
resource "aws_lb_listener" "https" {

  load_balancer_arn = aws_lb.application.arn

  port = 443

  protocol = "HTTPS"

  certificate_arn = aws_acm_certificate.main.arn

  default_action {

    type = "forward"

    target_group_arn = aws_lb_target_group.application.arn

  }

}
```

---

# CloudFormation

```yaml
Resources:

  ApplicationLoadBalancer:

    Type: AWS::ElasticLoadBalancingV2::LoadBalancer

    Properties:

      Name: production-alb

      Scheme: internet-facing

      Type: application
```

---

# Python (Boto3)

Create Load Balancer

```python
import boto3

elb = boto3.client("elbv2")

response = elb.create_load_balancer(

    Name="production-alb",

    Subnets=[

        "subnet-123",

        "subnet-456"

    ]

)

print(response)
```

---

# Best Practices

- Deploy ALBs across at least two Availability Zones
- Use HTTPS only in production
- Store certificates in AWS Certificate Manager
- Enable Access Logs
- Enable CloudWatch alarms
- Configure Health Checks correctly
- Use Path-Based Routing for microservices
- Place EC2 instances in private subnets
- Integrate with Auto Scaling
- Protect public ALBs with AWS WAF

---

# Common Mistakes

- Single Availability Zone deployment
- Incorrect Health Check path
- Allowing direct internet access to EC2
- Missing HTTPS redirection
- Sticky Sessions for stateless applications
- Wrong Target Group port
- Missing Security Group rules
- Ignoring CloudWatch alarms

---

# Troubleshooting

## Health Checks Failing

Verify:

- Application running
- Health endpoint
- Security Group
- NACL
- Port
- Response Code

---

## 502 Bad Gateway

Check:

- Target application
- Listener
- Target Group
- Backend logs

---

## 503 Service Unavailable

Possible causes:

- No healthy targets
- Health check failure
- Empty Target Group

---

## SSL Certificate Error

Verify:

- ACM certificate
- Domain validation
- Listener configuration
- Expiration date

---

## Traffic Not Reaching EC2

Check:

- Security Groups
- Target Registration
- Route Tables
- Listener Rules
- Health Status

---

# Interview Questions

### Basic

1. What is Elastic Load Balancer?
2. Why do we use a Load Balancer?
3. What are the types of ELB?
4. Difference between ALB and NLB?
5. What is a Target Group?
6. What is a Listener?
7. What is a Health Check?
8. What is SSL Termination?
9. What is Cross-Zone Load Balancing?
10. What are Sticky Sessions?

### Intermediate

11. Explain Host-Based Routing.
12. Explain Path-Based Routing.
13. What is Deregistration Delay?
14. How does ALB integrate with Auto Scaling?
15. What metrics does ELB publish?
16. How do Health Checks work?
17. Difference between ALB and Classic Load Balancer?
18. Why use ACM?
19. Explain WebSocket support.
20. How do Listener Rules work?

### Advanced

21. Why is ALB Layer 7?
22. When would you choose NLB instead of ALB?
23. Explain Gateway Load Balancer.
24. How does ALB integrate with Kubernetes?
25. Design a highly available ALB architecture.
26. How would you troubleshoot 502 errors?
27. How would you secure a public ALB?
28. Explain Cross-Zone Load Balancing behavior.
29. How does ALB support microservices?
30. Explain an end-to-end request flow through ALB.

---

# Scenario-Based Questions

### Scenario 1

Users receive **503 Service Unavailable**.

How would you troubleshoot?

---

### Scenario 2

Health checks suddenly fail after deployment.

What would you verify?

---

### Scenario 3

A new EC2 instance launches but does not receive traffic.

How would you investigate?

---

### Scenario 4

SSL certificate has expired.

How would you replace it without downtime?

---

### Scenario 5

One Availability Zone becomes unavailable.

How does ALB maintain application availability?

---

### Scenario 6

A microservices application needs `/api/*` and `/admin/*` routed to different services.

Which ALB feature would you use?

---

### Scenario 7

Traffic increased from 5,000 to 100,000 requests per minute during a flash sale.

How do ALB and Auto Scaling work together?

---

### Scenario 8

Your application must inspect all incoming traffic through a third-party firewall appliance.

Which AWS Load Balancer would you choose and why?

---

# Cheat Sheet

| Feature | Application LB | Network LB | Gateway LB |
|---------|----------------|------------|------------|
| OSI Layer | Layer 7 | Layer 4 | Layer 3 |
| Protocols | HTTP, HTTPS | TCP, UDP, TLS | IP |
| Host-Based Routing | ✅ | ❌ | ❌ |
| Path-Based Routing | ✅ | ❌ | ❌ |
| Static IP | ❌ | ✅ | ❌ |
| WebSocket | ✅ | ❌ | ❌ |
| Kubernetes Ingress | ✅ | Limited | ❌ |
| Best For | Web Apps & APIs | High Performance TCP/UDP | Virtual Appliances |

---

# Summary

Amazon Elastic Load Balancing distributes incoming traffic across multiple healthy targets, improving availability, scalability, and fault tolerance. By supporting different load balancer types—Application, Network, and Gateway—AWS provides solutions for web applications, high-performance TCP/UDP workloads, and network security appliances.

In production, combine ELB with Auto Scaling, ACM, AWS WAF, CloudWatch, and private subnets to build secure, resilient, and highly available applications. Proper listener configuration, health checks, and routing rules are essential for reliable traffic management.