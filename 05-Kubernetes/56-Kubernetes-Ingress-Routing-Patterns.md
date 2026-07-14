# Kubernetes Ingress Routing Patterns

## Introduction

In the previous chapter we learned:

```text
Ingress

Ingress Controller

AWS Load Balancer Controller

Traffic Flow
```

---

Ingress becomes powerful because of:

```text
Routing Rules
```

---

Ingress can route traffic based on:

```text
Host Name

Path
```

---

These are the two most common routing patterns used in production.

---

# Why Routing Is Required?

Suppose we have:

```text
Cart Service

Catalogue Service

User Service

Payment Service
```

---

Question

```text
How Can One Load Balancer

Serve Multiple Applications?
```

---

Answer

```text
Routing Rules
```

---

# Routing Types

Ingress supports:

```text
Host Based Routing

Path Based Routing
```

---

# Host Based Routing

Routing happens based on:

```text
Domain Name
```

---

Example

```text
catalogue.roboshop.com

cart.roboshop.com

user.roboshop.com
```

---

Different host names.

---

Different applications.

---

# Host Based Routing Architecture

```text
                    ALB
                     │
     ┌───────────────┼───────────────┐
     │               │               │
     ▼               ▼               ▼

catalogue      cart          user
 Service      Service      Service
```

---

# Production Example

User opens:

```text
catalogue.roboshop.com
```

---

Ingress checks:

```text
Host Header
```

---

Routes to:

```text
Catalogue Service
```

---

User opens:

```text
cart.roboshop.com
```

---

Routes to:

```text
Cart Service
```

---

# Host Based Routing YAML

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: roboshop

spec:

  rules:

  - host: catalogue.roboshop.com

    http:

      paths:

      - path: /

        pathType: Prefix

        backend:

          service:

            name: catalogue

            port:

              number: 8080

  - host: cart.roboshop.com

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

Traffic Flow

```text
catalogue.roboshop.com

↓

Catalogue Service

↓

Catalogue Pods
```

---

Traffic Flow

```text
cart.roboshop.com

↓

Cart Service

↓

Cart Pods
```

---

# Real Production Example

E-Commerce Platform

Applications

```text
Cart

Catalogue

Payment

User
```

---

Domains

```text
cart.company.com

catalogue.company.com

payment.company.com

user.company.com
```

---

One ALB

---

Many Applications

---

# Banking Example

Domains

```text
netbanking.icici.com

corporate.icici.com

loans.icici.com
```

---

Routing

```text
netbanking.icici.com

↓

NetBanking Service
```

---

Routing

```text
corporate.icici.com

↓

Corporate Service
```

---

# Advantages

```text
Easy To Understand

Application Isolation

Independent Teams
```

---

# Path Based Routing

Routing happens using:

```text
URL Path
```

---

Example

```text
roboshop.com/cart

roboshop.com/catalogue

roboshop.com/payment
```

---

Single Domain

---

Different Paths

---

# Path Based Routing Architecture

```text
                ALB
                 │

      roboshop.com
                 │

 ┌───────────────┼───────────────┐
 │               │               │

 ▼               ▼               ▼

/cart      /catalogue      /user
```

---

# Production Example

User opens

```text
roboshop.com/cart
```

---

Ingress routes

```text
Cart Service
```

---

User opens

```text
roboshop.com/catalogue
```

---

Ingress routes

```text
Catalogue Service
```

---

# Path Based Routing YAML

```yaml
apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

  name: roboshop

spec:

  rules:

  - host: roboshop.com

    http:

      paths:

      - path: /cart

        pathType: Prefix

        backend:

          service:

            name: cart

            port:

              number: 8080

      - path: /catalogue

        pathType: Prefix

        backend:

          service:

            name: catalogue

            port:

              number: 8080
```

---

Traffic Flow

```text
roboshop.com/cart

↓

Cart Service

↓

Cart Pods
```

---

Traffic Flow

```text
roboshop.com/catalogue

↓

Catalogue Service

↓

Catalogue Pods
```

---

# Host Based vs Path Based Routing

| Feature | Host Based | Path Based |
|----------|----------|----------|
| Decision | Domain Name | URL Path |
| Example | cart.company.com | company.com/cart |
| Isolation | Better | Moderate |
| Common Usage | Microservices | Small Applications |

---

# Which One Is Used More?

Most enterprises prefer:

```text
Host Based Routing
```

---

Reason

```text
Clear Separation

Independent SSL

Independent DNS
```

---

# PathType

Ingress requires:

```yaml
pathType:
```

---

Options

```text
Prefix

Exact

ImplementationSpecific
```

---

# Prefix

Most common.

---

Example

```yaml
path: /cart

pathType: Prefix
```

---

Matches

```text
/cart

/cart/1

/cart/items
```

---

# Exact

Matches only:

```text
Exact Path
```

---

Example

```yaml
path: /cart

pathType: Exact
```

---

Matches

```text
/cart
```

---

Does Not Match

```text
/cart/items
```

---

# Default Backend

Question

```text
What Happens

If No Rule Matches?
```

---

Answer

```text
Default Backend
```

---

Example

User Opens

```text
unknown.roboshop.com
```

---

No Rule Found.

---

Forward To

```text
Default Backend
```

---

# 404 Page Example

Default Backend returns:

```text
404

Application Not Found
```

---

# Multiple Rules Example

```text
catalogue.roboshop.com

↓

Catalogue Service
```

---

```text
cart.roboshop.com

↓

Cart Service
```

---

```text
user.roboshop.com

↓

User Service
```

---

Single ALB.

---

# Ingress Evaluation Flow

```text
Request

↓

Host Match?

↓

Path Match?

↓

Backend Service

↓

Pods
```

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
api.company.com/catalogue

↓

Catalogue
```

---

```text
api.company.com/cart

↓

Cart
```

---

# Production Banking Example

```text
netbanking.bank.com

↓

NetBanking
```

---

```text
corporate.bank.com

↓

Corporate Banking
```

---

```text
loans.bank.com

↓

Loan Service
```

---

Single ALB.

---

# Common Problems

## Wrong Host

Example

```text
catalog.roboshop.com
```

instead of

```text
catalogue.roboshop.com
```

---

Result

```text
404
```

---

# Wrong Path

Example

```text
/catalog

instead of

/catalogue
```

---

No Match.

---

# Service Missing

Cause

```text
Backend Service

Not Found
```

---

Verify

```bash
kubectl get svc
```

---

# Endpoint Missing

Cause

```text
No Pods

Behind Service
```

---

Verify

```bash
kubectl get endpoints
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
kubectl describe ingress ingress-name
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

Check Pods

```bash
kubectl get pods
```

---

# Common Interview Questions

## What is Host Based Routing?

### Short Answer

Routing based on domain name.

### Detailed Explanation

Ingress examines the host header and routes traffic to the corresponding backend service.

### Production Example

catalogue.roboshop.com routes to Catalogue Service.

### Follow-Up Questions

- Where is the host defined?
- Which routing type is more common?
- Can multiple hosts use one ALB?

---

## What is Path Based Routing?

### Short Answer

Routing based on URL path.

### Detailed Explanation

Ingress evaluates the URL path and forwards traffic to the appropriate service.

### Production Example

roboshop.com/cart routes to Cart Service.

### Follow-Up Questions

- What is pathType?
- What does Prefix mean?
- What is Exact path matching?

---

## Difference Between Host Based and Path Based Routing?

### Short Answer

Host-based routing uses domains while path-based routing uses URL paths.

### Detailed Explanation

Host routing separates applications using DNS names, while path routing separates applications using URL paths.

### Production Example

cart.company.com versus company.com/cart.

### Follow-Up Questions

- Which is preferred in production?
- Which provides better isolation?
- Which is easier for microservices?

---

## What is a Default Backend?

### Short Answer

It handles requests that do not match any ingress rule.

### Detailed Explanation

When no host or path rule matches, traffic is forwarded to the default backend.

### Production Example

Unknown domains return a custom 404 page.

### Follow-Up Questions

- What happens if no default backend exists?
- Can we customize the response?
- How is it configured?

---

# Key Takeaways

```text
Ingress supports host-based and path-based routing.

Host-based routing uses domain names.

Path-based routing uses URL paths.

Host-based routing is most common in enterprise environments.

Ingress routes traffic to Services, not Pods.

PathType controls matching behavior.

Prefix is the most commonly used pathType.

Default backend handles unmatched requests.

One ALB can serve multiple applications.

Routing patterns are fundamental for production Kubernetes networking.
```