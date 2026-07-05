# CloudFront CDN and Caching Architecture

## Introduction

As applications grow globally, users access them from different geographic locations.

Example:

```text
Users in India

Users in USA

Users in Europe

Users in Australia
```

If the application server exists only in one region:

```text
Higher Latency

Slower Response Times

Poor User Experience
```

To solve this problem, AWS provides:

```text
CloudFront
```

which is a:

```text
Content Delivery Network (CDN)
```

---

# What is a CDN?

CDN stands for:

```text
Content Delivery Network
```

---

A CDN stores frequently accessed content closer to users.

Instead of every request reaching the application server:

```text
User
   ↓
Nearest Edge Location
   ↓
Origin Server
```

---

# Real World Example

Suppose:

```text
Netflix Servers
```

are located in:

```text
United States
```

---

Users are accessing Netflix from:

```text
India
```

---

Without CDN:

```text
India
   ↓
USA
   ↓
USA
   ↓
India
```

Every request travels thousands of kilometers.

---

Result:

```text
High Latency

Slow Loading

Poor Performance
```

---

# With CDN

```text
India User
      ↓
India Edge Location
      ↓
USA Origin
```

---

First Request:

```text
Edge Location
```

downloads content from:

```text
Origin
```

---

Subsequent Requests:

```text
Served Directly From Cache
```

---

Result:

```text
Faster Response

Reduced Latency

Better User Experience
```

---

# What is CloudFront?

CloudFront is AWS's:

```text
Global CDN Service
```

---

Purpose:

```text
Content Caching

Performance Optimization

Global Delivery

HTTPS Acceleration
```

---

# CloudFront Components

```text
Distribution

Origin

Edge Locations

Behaviors

Cache

Invalidations
```

---

# Edge Locations

Edge Locations are AWS data centers distributed globally.

---

Purpose:

```text
Store Cached Content Closer To Users
```

---

Example

```text
India

Singapore

London

New York

Tokyo
```

---

# What is an Origin?

Origin is:

```text
Actual Source Of Content
```

---

Examples:

```text
Application Load Balancer

EC2

S3

API Gateway
```

---

# Roboshop Example

Origin:

```text
Frontend ALB
```

---

Request Flow

```text
User
   ↓
CloudFront
   ↓
Frontend ALB
```

---

# Existing Roboshop Architecture

```text
Route53
   ↓
Frontend ALB
   ↓
Frontend Service
```

---

# With CloudFront

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

# Why CloudFront?

Benefits:

```text
Lower Latency

Reduced Backend Load

Global Performance

HTTPS Support

Content Caching
```

---

# Application Architecture

Traditional:

```text
Application
   ↓
Database
```

---

Each request requires:

```text
Connect Database

Execute Query

Fetch Results

Process Results

Return Response
```

---

This consumes:

```text
CPU

Memory

Database Connections
```

---

# Caching Solution

CloudFront stores frequently accessed content.

---

Example:

```text
Images

CSS

JavaScript

Static Assets
```

---

Instead of:

```text
Application
   ↓
Database
```

every time,

CloudFront serves:

```text
Cached Content
```

---

# CloudFront HTTPS Support

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

# Common HTTP Methods

```text
GET

POST

PUT

DELETE
```

---

# Which Requests Are Cached?

Typically:

```text
GET
```

requests.

---

Because GET requests are:

```text
Read Operations
```

---

Examples:

```text
Images

CSS

JavaScript

Static Pages
```

---

# Which Requests Are Usually Not Cached?

```text
POST

PUT

DELETE
```

---

Because these operations:

```text
Modify Data
```

---

Examples:

```text
Add To Cart

Create User

Update Order

Delete Product
```

---

# Static Content Example

```text
https://roboshop-dev.daws86s.fun/media/graph.png

https://roboshop-dev.daws86s.fun/media/stan.png

https://roboshop-dev.daws86s.fun/media/instana_icon_square.png
```

---

These files rarely change.

Perfect candidates for:

```text
CloudFront Cache
```

---

# Request Flow Without CloudFront

```text
User
   ↓
Frontend ALB
   ↓
Frontend Service
```

---

Every request reaches:

```text
Application Servers
```

---

# Request Flow With CloudFront

```text
User
   ↓
CloudFront Edge Location
   ↓
Cache Hit
```

---

Response immediately returned.

---

# Cache Miss

If content is unavailable:

```text
Cache Miss
```

---

Flow:

```text
User
   ↓
CloudFront
   ↓
Origin
   ↓
Frontend ALB
```

---

CloudFront stores:

```text
Downloaded Content
```

for future requests.

---

# CloudFront Domain

CloudFront distributions provide:

```text
CloudFront URL
```

---

Example:

```text
xxxxxxxx.cloudfront.net
```

---

Usually mapped to:

```text
Custom Domain
```

---

Example:

```text
roboshop-dev.daws86s.fun
```

---

# CloudFront Requirements

Typically requires:

```text
Name

Domain

Certificate

Origin
```

---

# Production Example

Request:

```text
https://roboshop-dev.daws86s.fun/media/logo.png
```

---

Flow:

```text
User
   ↓
Nearest Edge Location
   ↓
Cache Hit
   ↓
Response
```

---

No request reaches:

```text
Frontend ALB
```

---

Result:

```text
Lower Cost

Lower Latency

Reduced Backend Load
```

---

# Benefits of CloudFront

```text
Global Content Delivery

Reduced Latency

Improved Performance

Lower Origin Load

HTTPS Support

DDoS Protection Integration

Global Edge Locations
```

---

# Production Architecture

```text
User
   ↓
CloudFront
   ↓
Edge Location
   ↓
Origin (Frontend ALB)
   ↓
Frontend Service
```

---

# Common Interview Questions

## What is a CDN?

### Short Answer

A CDN is a Content Delivery Network that caches content closer to users.

### Detailed Explanation

CDNs reduce latency by serving cached content from geographically distributed edge locations instead of always contacting the origin server.

### Production Example

CloudFront serving static images for Roboshop users from nearby edge locations.

### Follow-Up Questions

- What is an Edge Location?
- Why is caching useful?

---

## What is CloudFront?

### Short Answer

CloudFront is AWS's global CDN service.

### Detailed Explanation

CloudFront caches content at edge locations and improves application performance by reducing latency and backend load.

### Production Example

CloudFront sitting in front of a Frontend ALB.

### Follow-Up Questions

- What can be used as an origin?
- Does CloudFront support HTTPS?

---

## What is an Origin in CloudFront?

### Short Answer

An origin is the actual source of content.

### Detailed Explanation

When content is not available in cache, CloudFront retrieves it from the configured origin.

### Production Example

Frontend ALB acting as the origin for CloudFront.

### Follow-Up Questions

- Can S3 be an origin?
- Can ALB be an origin?

---

## Which HTTP Methods Are Commonly Cached?

### Short Answer

GET requests.

### Detailed Explanation

GET requests retrieve data and are generally safe to cache, whereas POST, PUT, and DELETE modify data and are usually not cached.

### Production Example

Caching images, CSS, and JavaScript files.

### Follow-Up Questions

- Why are POST requests not cached?
- Can cache policies be customized?

---

# Key Takeaways

```text
CloudFront is AWS's Content Delivery Network service.

CDNs reduce latency by serving content from edge locations.

Origins are the source systems that provide content.

Edge locations cache content closer to users.

GET requests are commonly cached.

POST, PUT, and DELETE requests are usually not cached.

CloudFront improves performance and reduces backend load.

Static assets are ideal candidates for caching.

CloudFront is commonly placed in front of ALBs in production environments.

CloudFront is widely used for global application delivery.
```