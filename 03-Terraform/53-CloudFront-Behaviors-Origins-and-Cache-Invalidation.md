# CloudFront Behaviors, Origins and Cache Invalidation

## Introduction

In the previous topic, we learned:

```text
CloudFront

CDN

Edge Locations

Caching

Origins
```

However, CloudFront is not just a caching service.

It also provides:

```text
Path-Based Routing

Multiple Origins

Cache Behaviors

HTTPS Enforcement

Cache Invalidation
```

These features allow CloudFront to intelligently route and cache content.

---

# What is an Origin?

An Origin is:

```text
The Actual Source Of Content
```

When content is not available in cache:

```text
CloudFront
    ↓
Origin
```

retrieves the content.

---

# Common Origins

```text
S3 Bucket

Application Load Balancer

EC2 Instance

API Gateway
```

---

# Roboshop Example

Origin:

```text
Frontend ALB
```

---

Architecture:

```text
User
   ↓
CloudFront
   ↓
Frontend ALB
   ↓
Frontend Service
```

---

# Multiple Origin Architecture

CloudFront can use:

```text
One Origin

Multiple Origins
```

---

Example:

```text
Origin-1 → Roboshop

Origin-2 → Corporate Banking
```

---

Architecture

```text
CloudFront
     ↓
 ┌───────────────┐
 │ Origin-1      │
 │ Origin-2      │
 └───────────────┘
```

---

# What is a Behavior?

A Behavior determines:

```text
Which Request Goes To Which Origin

What To Cache

What Not To Cache
```

---

Behavior is similar to:

```text
ALB Listener Rules
```

---

# Example

Request:

```text
/roboshop
```

↓

```text
Roboshop Origin
```

---

Request:

```text
/corporatebanking
```

↓

```text
Corporate Banking Origin
```

---

# Real Example

Website:

```text
icici.com
```

---

Request:

```text
icici.com/netbanking
```

↓

```text
Net Banking Application
```

---

Request:

```text
icici.com/corporatebanking
```

↓

```text
Corporate Banking Application
```

---

# CloudFront Behavior Example

```text
/roboshop/*
```

↓

```text
roboshop-dev.daws86s.fun
```

---

```text
/corporatebanking/*
```

↓

```text
corporatebanking.daws86s.fun
```

---

# Path-Based Routing

CloudFront evaluates:

```text
Request Path
```

and chooses:

```text
Correct Origin
```

---

Flow

```text
Request
   ↓
Behavior Evaluation
   ↓
Origin Selection
```

---

# Cache Behavior

CloudFront allows us to define:

```text
Which Content Should Be Cached
```

---

Examples:

```text
Images

CSS

JavaScript

Fonts
```

---

# Static Content Example

```text
/media/graph.png

/media/stan.png

/media/instana_icon_square.png
```

---

These files rarely change.

Ideal for:

```text
Caching
```

---

# Dynamic Content

Examples:

```text
Add To Cart

Login

Place Order

Payment
```

---

Requests:

```text
POST

PUT

DELETE
```

---

Usually:

```text
Not Cached
```

---

# Why Not Cache Dynamic Requests?

Because data changes frequently.

Example:

```text
Cart Items

Orders

Payments
```

must always be current.

---

# Cache Hit

Content already exists at Edge Location.

Flow:

```text
User
   ↓
CloudFront
   ↓
Cache Hit
   ↓
Response
```

---

# Cache Miss

Content not available in cache.

Flow:

```text
User
   ↓
CloudFront
   ↓
Origin
   ↓
Response
```

---

CloudFront stores:

```text
Downloaded Content
```

for future requests.

---

# What is Cache Invalidation?

CloudFront caches content for a period of time.

---

Problem:

Application releases new content.

Example:

```text
New Logo

Updated CSS

New JavaScript
```

---

Users may still receive:

```text
Old Cached Version
```

---

# Solution

CloudFront provides:

```text
Cache Invalidation
```

---

# What Happens During Invalidation?

CloudFront removes:

```text
Old Cached Content
```

from Edge Locations.

---

Next Request:

```text
CloudFront
   ↓
Origin
```

downloads fresh content.

---

# Release Deployment Example

Old File:

```text
logo.png
```

---

New Deployment:

```text
logo.png
```

updated.

---

Without Invalidation:

```text
Users See Old Logo
```

---

With Invalidation:

```text
Users See New Logo
```

---

# Common Invalidation Paths

Invalidate Specific File

```text
/media/logo.png
```

---

Invalidate Folder

```text
/media/*
```

---

Invalidate Everything

```text
/*
```

---

# Production Deployment Flow

```text
Deploy New Version
      ↓
Invalidate Cache
      ↓
Users Request Content
      ↓
CloudFront Downloads Fresh Content
      ↓
Updated Content Served
```

---

# HTTPS Enforcement

CloudFront can automatically redirect:

```text
HTTP
```

to:

```text
HTTPS
```

---

Example

```text
http://dev.daws86s.fun
```

↓

```text
https://dev.daws86s.fun
```

---

# SSL Certificates

CloudFront distributions usually require:

```text
Domain

Certificate

Origin
```

---

Example

```text
roboshop-dev.daws86s.fun
```

---

Certificate:

```text
ACM Certificate
```

---

Origin:

```text
Frontend ALB
```

---

# CloudFront and ALB

Architecture:

```text
User
   ↓
CloudFront
   ↓
Frontend ALB
   ↓
Frontend Service
```

---

Benefits:

```text
Caching

HTTPS

Global Delivery

Reduced Backend Load
```

---

# Production Architecture

```text
User
   ↓
CloudFront
   ↓
Behavior Evaluation
   ↓
Origin Selection
   ↓
Frontend ALB
   ↓
Application
```

---

# Production Example

Request:

```text
https://roboshop-dev.daws86s.fun/media/graph.png
```

---

Behavior:

```text
/media/*
```

↓

```text
Cache Content
```

---

Request:

```text
POST /api/cart
```

↓

```text
Do Not Cache
```

---

# Common Interview Questions

## What is a CloudFront Behavior?

### Short Answer

A Behavior defines how CloudFront handles requests and selects origins.

### Detailed Explanation

Behaviors control path-based routing, caching rules, HTTPS enforcement, and origin selection.

### Production Example

/roboshop/* routed to Roboshop Origin.

### Follow-Up Questions

- Can multiple behaviors exist?
- What is path-based routing?

---

## What is an Origin?

### Short Answer

An Origin is the source location from which CloudFront retrieves content.

### Detailed Explanation

CloudFront fetches content from origins whenever a cache miss occurs.

### Production Example

Frontend ALB configured as the origin.

### Follow-Up Questions

- Can ALB be an origin?
- Can S3 be an origin?

---

## What is Cache Invalidation?

### Short Answer

Cache invalidation removes cached content from CloudFront edge locations.

### Detailed Explanation

After new deployments, invalidation forces CloudFront to retrieve fresh content from the origin.

### Production Example

Invalidating /media/* after a frontend release.

### Follow-Up Questions

- Why is invalidation needed?
- What happens without invalidation?

---

## What is the Difference Between Cache Hit and Cache Miss?

### Short Answer

Cache Hit serves content from the edge location, while Cache Miss retrieves content from the origin.

### Detailed Explanation

Hits provide faster responses, while misses require communication with the origin server.

### Production Example

Serving graph.png directly from an edge location.

### Follow-Up Questions

- Which is faster?
- What causes cache misses?

---

# Key Takeaways

```text
Origins are the source systems for CloudFront content.

Behaviors determine routing and caching decisions.

Path-based routing can direct requests to different origins.

Static content is ideal for caching.

Dynamic content is usually not cached.

Cache Hits improve performance.

Cache Misses retrieve content from origins.

Cache Invalidation removes outdated content from edge locations.

CloudFront commonly sits in front of ALBs.

CloudFront behaviors are similar to ALB listener rules.
```