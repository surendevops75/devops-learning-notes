# ALB Ingress in EKS Production Setup

## Introduction

In previous chapters we learned:

```text
Ingress

Ingress Controller

AWS Load Balancer Controller

Host Based Routing

Path Based Routing
```

---

Now let's see how ALB Ingress is implemented in a real EKS production environment.

---

# Production Requirement

Applications

```text
Catalogue

Cart

User

Shipping

Payment
```

---

Users should access:

```text
catalogue.daws86s.fun

cart.daws86s.fun

user.daws86s.fun

payment.daws86s.fun
```

---

Requirement

```text
Single ALB

Multiple Applications

HTTPS Enabled

High Availability
```

---

# Production Architecture

```text
Internet
    │

    ▼

Route53
    │

    ▼

Application Load Balancer
    │

    ▼

Listener (80/443)
    │

    ▼

Rules
    │

    ▼

Target Groups
    │

    ▼

Kubernetes Services
    │

    ▼

Pods
```

---

# ALB Request Flow

Request

```text
catalogue.daws86s.fun
```

---

Flow

```text
User

↓

Route53

↓

ALB

↓

Listener

↓

Rule

↓

Catalogue Target Group

↓

Catalogue Service

↓

Catalogue Pods
```

---

# Another Example

Request

```text
cart.daws86s.fun
```

---

Flow

```text
User

↓

ALB

↓

Host Rule

↓

Cart Target Group

↓

Cart Service

↓

Cart Pods
```

---

# Banking Example

Request

```text
netbanking.icici.com
```

---

Flow

```text
ALB

↓

Listener

↓

Rule

↓

NetBanking TG

↓

Pods
```

---

Request

```text
corporate.icici.com
```

---

Flow

```text
ALB

↓

Listener

↓

Rule

↓

Corporate TG

↓

Pods
```

---

# Prerequisites

Before creating ALB Ingress

Ensure:

```text
EKS Cluster Exists

AWS Load Balancer Controller Installed

OIDC Configured

IAM Permissions Configured

Services Created

Pods Running
```

---

# Verify Controller

```bash
kubectl get pods -n kube-system
```

---

Expected

```text
aws-load-balancer-controller
```

---

# Verify Services

```bash
kubectl get svc
```

---

Example

```text
catalogue

cart

user

payment
```

---

# Basic ALB Ingress

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: roboshop

  annotations:

    kubernetes.io/ingress.class: alb

spec:

  rules:

  - host: catalogue.daws86s.fun

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: catalogue

            port:

              number: 8080
```

---

Important Annotation

```yaml
kubernetes.io/ingress.class: alb
```

---

Meaning

```text
Use AWS Load Balancer Controller
```

---

# Host Based Routing Example

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: roboshop

  annotations:

    kubernetes.io/ingress.class: alb

spec:

  rules:

  - host: catalogue.daws86s.fun

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: catalogue

            port:

              number: 8080

  - host: cart.daws86s.fun

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: cart

            port:

              number: 8080
```

---

Result

```text
catalogue.daws86s.fun

↓

Catalogue Service
```

---

```text
cart.daws86s.fun

↓

Cart Service
```

---

Single ALB

---

Multiple Applications

---

# Path Based Routing Example

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: roboshop

  annotations:

    kubernetes.io/ingress.class: alb

spec:

  rules:

  - host: daws86s.fun

    http:

      paths:

      - path: /catalogue

        pathType: Prefix

        backend:

          service:

            name: catalogue

            port:

              number: 8080

      - path: /cart

        pathType: Prefix

        backend:

          service:

            name: cart

            port:

              number: 8080
```

---

Traffic

```text
daws86s.fun/catalogue

↓

Catalogue Service
```

---

Traffic

```text
daws86s.fun/cart

↓

Cart Service
```

---

# Internet Facing ALB

Purpose

```text
Public Access
```

---

Annotation

```yaml
alb.ingress.kubernetes.io/scheme: internet-facing
```

---

Use Cases

```text
E-Commerce

Banking

Public APIs

Customer Portals
```

---

# Internal ALB

Purpose

```text
Private Access
```

---

Annotation

```yaml
alb.ingress.kubernetes.io/scheme: internal
```

---

Use Cases

```text
Internal Applications

Backoffice Systems

Private APIs
```

---

# SSL Configuration

Production applications should use:

```text
HTTPS
```

---

Not

```text
HTTP
```

---

# ACM Certificate

Create certificate in:

```text
AWS Certificate Manager
```

---

Example

```text
*.daws86s.fun
```

---

# Attach Certificate

Annotation

```yaml
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:123456789012:certificate/xxxx
```

---

Purpose

```text
Enable HTTPS
```

---

# HTTPS Listener

Annotation

```yaml
alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
```

---

Result

```text
Port 80

Port 443
```

---

# HTTP to HTTPS Redirect

Annotation

```yaml
alb.ingress.kubernetes.io/ssl-redirect: '443'
```

---

Flow

```text
HTTP

↓

HTTPS
```

---

Automatically.

---

# Route53 Integration

Create Record

```text
catalogue.daws86s.fun
```

---

Points To

```text
ALB DNS Name
```

---

Example

```text
internal-abc.us-east-1.elb.amazonaws.com
```

---

# Health Checks

ALB continuously checks:

```text
Pod Health
```

---

Healthy Pods

```text
Receive Traffic
```

---

Unhealthy Pods

```text
Removed From Target Group
```

---

# Target Group Health Check

Example

```text
/health
```

---

Response

```text
200 OK
```

---

Pod considered healthy.

---

# Production E-Commerce Example

Applications

```text
Frontend

Catalogue

Cart

Payment

User
```

---

Routing

```text
shop.company.com

↓

Frontend
```

---

```text
catalogue.company.com

↓

Catalogue
```

---

```text
cart.company.com

↓

Cart
```

---

Single ALB.

---

# Production Banking Example

```text
netbanking.bank.com

↓

NetBanking Service
```

---

```text
corporate.bank.com

↓

Corporate Banking Service
```

---

```text
loans.bank.com

↓

Loan Service
```

---

One ALB.

---

Many Applications.

---

# Common Problems

## ALB Not Created

Cause

```text
AWS Load Balancer Controller Missing
```

---

Verify

```bash
kubectl get pods -n kube-system
```

---

# DNS Not Resolving

Cause

```text
Route53 Record Missing
```

---

Verify

```text
DNS Configuration
```

---

# Target Group Empty

Cause

```text
Pods Not Ready
```

---

Verify

```bash
kubectl get endpoints
```

---

# SSL Not Working

Cause

```text
Certificate ARN Wrong
```

---

Verify

```text
ACM Certificate
```

---

# 503 Error

Cause

```text
No Healthy Targets
```

---

Verify

```text
Target Group Health
```

---

# Troubleshooting

View Ingress

```bash
kubectl get ingress
```

---

Describe Ingress

```bash
kubectl describe ingress roboshop
```

---

Check Controller Logs

```bash
kubectl logs deployment/aws-load-balancer-controller \
-n kube-system
```

---

Check Services

```bash
kubectl get svc
```

---

Check Endpoints

```bash
kubectl get endpoints
```

---

Check ALBs

```bash
aws elbv2 describe-load-balancers
```

---

Check Target Groups

```bash
aws elbv2 describe-target-groups
```

---

# Common Interview Questions

## How Have You Implemented Ingress In EKS?

### Short Answer

We used AWS Load Balancer Controller to create an ALB and expose multiple microservices through host-based routing.

### Detailed Explanation

A single ALB was shared across services such as Catalogue, Cart, User, and Payment. Route53 DNS records pointed to the ALB and HTTPS was enabled using ACM certificates.

### Production Example

catalogue.daws86s.fun → Catalogue Service → Catalogue Pods.

### Follow-Up Questions

- Why ALB instead of multiple LoadBalancers?
- How was SSL configured?
- How was DNS configured?

---

## What Is The Difference Between Internal And Internet Facing ALB?

### Short Answer

Internet-facing ALBs are public, while internal ALBs are private.

### Detailed Explanation

Internet-facing ALBs receive traffic from the internet. Internal ALBs are accessible only within the VPC.

### Production Example

Customer portals use internet-facing ALBs while internal APIs use internal ALBs.

### Follow-Up Questions

- Which annotation controls this?
- Can it be changed later?
- What are common use cases?

---

## How Is SSL Configured On ALB?

### Short Answer

Using ACM certificates.

### Detailed Explanation

Certificates are created in AWS Certificate Manager and attached to the ALB through ingress annotations.

### Production Example

*.daws86s.fun wildcard certificate secures all Roboshop subdomains.

### Follow-Up Questions

- What is ACM?
- What is SSL redirect?
- Can one certificate secure multiple domains?

---

## How Does ALB Route Traffic?

### Short Answer

Using listeners, rules, and target groups.

### Detailed Explanation

Requests arrive at listeners, rules determine the destination, and target groups forward traffic to Pods.

### Production Example

catalogue.daws86s.fun → Listener → Rule → Catalogue TG → Pods.

### Follow-Up Questions

- What is a listener?
- What is a target group?
- How are health checks performed?

---

# Key Takeaways

```text
AWS Load Balancer Controller creates ALBs from Ingress resources.

ALB supports host-based and path-based routing.

Internet-facing ALBs are public.

Internal ALBs are private.

Route53 integrates DNS with ALB.

ACM provides SSL certificates.

Health checks determine healthy targets.

A single ALB can expose multiple applications.

ALB → Listener → Rule → Target Group → Service → Pod is the complete traffic flow.

ALB Ingress is the standard production ingress pattern in EKS.
```