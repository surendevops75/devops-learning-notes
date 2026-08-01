# Amazon CloudFront

---

# Introduction

Amazon CloudFront is AWS's global Content Delivery Network (CDN) service that securely delivers web applications, APIs, videos, images, and static content to users with low latency and high transfer speeds.

Instead of serving every request from the origin server, CloudFront caches content at Edge Locations located around the world.

CloudFront integrates with:

- Amazon S3
- Application Load Balancer
- EC2
- Amazon ECS
- Amazon EKS
- API Gateway
- AWS Lambda
- Lambda@Edge
- AWS WAF
- AWS Shield
- Route53
- AWS Certificate Manager (ACM)

CloudFront is one of the most important AWS networking services used in production architectures.

---

# What is Amazon CloudFront?

Amazon CloudFront is a Content Delivery Network (CDN).

A CDN caches content closer to users.

Instead of

```text
User (India)

↓

USA Server

↓

Response
```

CloudFront serves cached content from the nearest Edge Location.

```text
User

↓

Nearest Edge Location

↓

Cached Content
```

---

# Why CloudFront?

Without CloudFront

```text
Users

↓

Origin Server

↓

Every Request

↓

Higher Latency
```

Problems

- Slow Response
- Higher Origin Load
- More Bandwidth
- Poor User Experience

With CloudFront

```text
Users

↓

Edge Location

↓

Cached Response

↓

Origin (Only If Needed)
```

Benefits

- Lower Latency
- Reduced Origin Load
- Faster Downloads
- Global Availability

---

# Real World Problem

An online shopping platform serves

- Images
- CSS
- JavaScript
- Product Videos
- APIs

Customers are located in

- India
- USA
- Europe
- Australia
- Japan

Without a CDN, every request reaches the origin.

CloudFront caches content globally.

---

# Enterprise Architecture

```text
Internet Users

↓

Route53

↓

CloudFront

↓

AWS WAF

↓

Application Load Balancer

↓

Amazon EKS / ECS / EC2

↓

Amazon RDS
```

---

# CDN Concept

Content Delivery Network

```text
Origin

↓

Edge Locations

↓

Users
```

Edge Locations store cached copies.

---

# Origin

An Origin is where CloudFront retrieves content.

Supported Origins

- Amazon S3
- Application Load Balancer
- EC2
- API Gateway
- Media Servers
- Custom HTTP Servers

---

# Edge Locations

Edge Locations cache content.

Workflow

```text
User

↓

Nearest Edge

↓

Cached Content

↓

Response
```

AWS has hundreds of Edge Locations worldwide.

---

# Regional Edge Cache

Between the Origin and Edge Locations,

CloudFront uses Regional Edge Caches.

Architecture

```text
Origin

↓

Regional Edge Cache

↓

Edge Location

↓

User
```

Benefits

- Better Cache Efficiency
- Reduced Origin Requests

---

# Cache Behavior

Cache Behavior controls

- Path Patterns
- HTTP Methods
- Origin Selection
- Cache Policy
- Compression

Example

```text
/images/*

↓

S3

----------------

/api/*

↓

ALB
```

---

# Cache Hit

If content exists in cache,

CloudFront serves it immediately.

Workflow

```text
Request

↓

Edge Cache

↓

Cache Hit

↓

Response
```

---

# Cache Miss

If content is unavailable,

CloudFront contacts the origin.

Workflow

```text
Request

↓

Cache Miss

↓

Origin

↓

Store in Cache

↓

Response
```

---

# Time To Live (TTL)

TTL controls cache duration.

Types

- Minimum TTL
- Default TTL
- Maximum TTL

Example

```
3600 Seconds
```

Content stays cached for one hour.

---

# Cache Invalidation

Removes cached content before TTL expires.

Example

```text
Updated Logo

↓

Invalidate Cache

↓

Users Receive Latest Version
```

---

# Cache Policies

Cache Policies determine

- Headers
- Cookies
- Query Strings
- Compression

Policies improve cache efficiency.

---

# Origin Request Policies

Controls what CloudFront forwards to the origin.

Examples

- Headers
- Cookies
- Query Parameters

---

# Compression

CloudFront automatically compresses

- HTML
- CSS
- JavaScript

Supported Formats

- Gzip
- Brotli

Benefits

- Faster Downloads
- Reduced Bandwidth

---

# HTTPS

CloudFront supports HTTPS using ACM certificates.

Workflow

```text
User

↓

HTTPS

↓

CloudFront

↓

Origin
```

---

# Custom Domain

Instead of

```text
d123abc.cloudfront.net
```

Use

```text
www.company.com
```

Requires

- Route53
- ACM Certificate

---

# Signed URLs

Signed URLs provide temporary access.

Example

```text
Private Video

↓

Signed URL

↓

Authorized User
```

Useful for premium content.

---

# Signed Cookies

Signed Cookies provide access to multiple protected files.

Example

```text
Login

↓

Signed Cookie

↓

Download Multiple Files
```

---

# Origin Access Control (OAC)

OAC securely connects CloudFront to S3.

Benefits

- Private S3 Bucket
- Secure Authentication
- Recommended by AWS

Replaces the older Origin Access Identity (OAI) model.

---

# Field-Level Encryption

Encrypts sensitive request fields.

Examples

- Credit Card Number
- SSN
- Password

Provides end-to-end protection.

---

# Lambda@Edge

Lambda@Edge executes code at Edge Locations.

Use Cases

- Header Modification
- Authentication
- URL Rewrites
- Redirects

Workflow

```text
User

↓

Edge Location

↓

Lambda@Edge

↓

Response
```

---

# CloudFront Functions

CloudFront Functions execute lightweight JavaScript.

Best For

- Header Changes
- URL Redirects
- Authentication Checks

Lower latency than Lambda@Edge.

---

# Security Integration

CloudFront integrates with

- AWS WAF
- AWS Shield
- ACM
- IAM
- CloudTrail

Provides enterprise-grade security.

---

# Summary

Amazon CloudFront is AWS's global Content Delivery Network (CDN) that improves application performance by caching content at Edge Locations worldwide. Core concepts such as Origins, Edge Locations, Regional Edge Caches, Cache Behaviors, TTL, Cache Policies, Signed URLs, Origin Access Control, HTTPS, Lambda@Edge, and CloudFront Functions form the foundation for building fast, scalable, secure, and globally distributed web applications.

---

