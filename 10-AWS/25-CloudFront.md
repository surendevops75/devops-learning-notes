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

# AWS CLI

Create Distribution

```bash
aws cloudfront create-distribution \
--distribution-config file://distribution.json
```

List Distributions

```bash
aws cloudfront list-distributions
```

Get Distribution

```bash
aws cloudfront get-distribution \
--id E123456789ABC
```

Create Invalidation

```bash
aws cloudfront create-invalidation \
--distribution-id E123456789ABC \
--paths "/*"
```

Get Invalidation

```bash
aws cloudfront get-invalidation \
--distribution-id E123456789ABC \
--id I123456789ABC
```

Delete Distribution

```bash
aws cloudfront delete-distribution \
--id E123456789ABC
```

---

# Terraform

Create CloudFront Distribution

```hcl
resource "aws_cloudfront_distribution" "production" {

  enabled = true

  origin {

    domain_name = aws_s3_bucket.website.bucket_regional_domain_name

    origin_id = "s3-origin"

  }

  default_cache_behavior {

    target_origin_id = "s3-origin"

    viewer_protocol_policy = "redirect-to-https"

    allowed_methods = ["GET", "HEAD"]

    cached_methods = ["GET", "HEAD"]

  }

}
```

---

# CloudFormation

```yaml
Resources:

  CloudFrontDistribution:

    Type: AWS::CloudFront::Distribution

    Properties:

      DistributionConfig:

        Enabled: true
```

---

# Python (Boto3)

Create Distribution

```python
import boto3

cf = boto3.client("cloudfront")

response = cf.create_distribution(
    DistributionConfig={
        "CallerReference": "production",
        "Enabled": True
    }
)
```

List Distributions

```python
response = cf.list_distributions()

print(response)
```

---

# CloudWatch Integration

CloudFront publishes metrics to CloudWatch.

Common Metrics

- Requests
- Bytes Downloaded
- Bytes Uploaded
- Cache Hit Rate
- Error Rate
- Origin Latency

Architecture

```text
CloudFront

↓

CloudWatch

↓

Dashboards

↓

CloudWatch Alarms
```

---

# CloudFront Logs

CloudFront provides

- Standard Logs
- Real-Time Logs

Logs can be delivered to

- Amazon S3
- Kinesis Data Streams

Useful for

- Security Analysis
- Performance Analysis
- Troubleshooting

---

# Standard Logging

Contains

- Client IP
- URL
- User Agent
- Response Code
- Bytes Sent
- Edge Location

Useful for historical analysis.

---

# Real-Time Logs

Real-Time Logs provide immediate visibility.

Use Cases

- Security Monitoring
- Live Analytics
- Fraud Detection
- API Monitoring

---

# CloudWatch Alarms

Example

```text
5XX Errors > Threshold

↓

CloudWatch Alarm

↓

SNS

↓

Operations Team
```

---

# Origin Failover

CloudFront supports multiple origins.

Example

```text
Primary Origin

↓

Failure

↓

Secondary Origin

↓

Serve Content
```

Improves availability.

---

# Origin Groups

Origin Groups define failover behavior.

Architecture

```text
CloudFront

↓

Primary Origin

↓

Failure

↓

Backup Origin
```

---

# Multi-Origin Distribution

One distribution can serve multiple origins.

Example

```text
/images/*

↓

Amazon S3

-----------------

/api/*

↓

Application Load Balancer

-----------------

/videos/*

↓

Media Server
```

---

# Viewer Protocol Policy

Controls HTTP/HTTPS behavior.

Options

- HTTP Only
- HTTPS Only
- Redirect HTTP to HTTPS

Production Recommendation

Always redirect HTTP to HTTPS.

---

# Price Classes

CloudFront offers pricing options.

- Price Class 100
- Price Class 200
- Price Class All

Lower price classes reduce Edge Location coverage to lower costs.

---

# Geo Restriction

Restrict content delivery by country.

Example

```text
Allowed

India

USA

Germany

Blocked

Others
```

Useful for licensing requirements.

---

# Origin Shield

Origin Shield adds an additional caching layer.

Architecture

```text
Users

↓

Edge Location

↓

Origin Shield

↓

Origin
```

Benefits

- Lower Origin Load
- Better Cache Hit Ratio
- Reduced Latency

---

# Security Best Practices

- Enable HTTPS only
- Use AWS WAF
- Enable AWS Shield
- Use Origin Access Control (OAC)
- Use Signed URLs for premium content
- Enable CloudWatch metrics
- Enable Standard Logs
- Enable Real-Time Logs for critical applications
- Compress content
- Configure appropriate TTL values
- Use custom domain with ACM certificates
- Protect S3 buckets from public access

---

# Common Mistakes

- Public S3 buckets behind CloudFront
- Using HTTP instead of HTTPS
- Very low TTL values
- Never invalidating cache
- Ignoring cache policies
- No WAF protection
- Not enabling logging
- Using one origin for every workload
- Missing Origin Failover
- Not monitoring cache hit ratio

---

# Troubleshooting

## Content Not Updating

Check

- Cache TTL
- Cache Invalidation
- Browser Cache
- Origin Cache-Control Headers

---

## High Origin Load

Verify

- Cache Hit Ratio
- TTL Settings
- Cache Policy
- Origin Shield

---

## HTTPS Certificate Error

Check

- ACM Certificate
- Domain Name
- Alternate Domain Names (CNAMEs)
- Certificate Region

---

## 403 Forbidden

Verify

- OAC Configuration
- S3 Bucket Policy
- WAF Rules
- Signed URL Expiration

---

## High Latency

Check

- Origin Response Time
- Cache Hit Ratio
- Edge Location
- Compression
- Large Objects

---

# Interview Questions

## Basic

1. What is Amazon CloudFront?
2. What is a CDN?
3. What is an Edge Location?
4. What is an Origin?
5. What is Cache Hit?
6. What is Cache Miss?
7. What is TTL?
8. What is Cache Invalidation?
9. What is Origin Access Control (OAC)?
10. CloudFront vs S3 Static Website?

---

## Intermediate

11. Explain Cache Behaviors.
12. Explain Cache Policies.
13. What is Origin Shield?
14. Explain Signed URLs.
15. Explain Signed Cookies.
16. CloudFront vs ALB?
17. Lambda@Edge vs CloudFront Functions?
18. Explain Real-Time Logs.
19. Explain Geo Restriction.
20. Explain Multi-Origin Distribution.

---

## Advanced

21. Design a global content delivery architecture.
22. How would you improve cache hit ratio?
23. Explain CloudFront request flow.
24. How would you secure private S3 content?
25. Explain OAC vs OAI.
26. Design a multi-origin CloudFront architecture.
27. How would you troubleshoot high latency?
28. Explain CloudFront caching strategy.
29. How would you optimize CloudFront costs?
30. Best practices for production CloudFront deployments.

---

# Production Scenarios

### Scenario 1

Users in Australia report slow website performance.

How would CloudFront improve response times?

---

### Scenario 2

Your application serves premium videos to paying customers only.

How would Signed URLs or Signed Cookies secure access?

---

### Scenario 3

A new application logo is uploaded but users still see the old version.

How would you resolve the issue?

---

### Scenario 4

Your S3 bucket must remain private while serving static website assets.

How would Origin Access Control help?

---

### Scenario 5

The primary application server becomes unavailable.

How would Origin Failover maintain availability?

---

### Scenario 6

An application serves static content from S3 and APIs from EKS.

How would you configure multiple origins within CloudFront?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Distribution | CloudFront Configuration |
| Origin | Source of Content |
| Edge Location | Global Cache |
| Regional Edge Cache | Intermediate Cache |
| Cache Policy | Control Cache Behavior |
| Origin Request Policy | Control Origin Requests |
| TTL | Cache Duration |
| Cache Invalidation | Remove Cached Content |
| OAC | Secure S3 Access |
| Signed URL | Temporary Access |
| Signed Cookie | Secure Multiple Files |
| Lambda@Edge | Edge Compute |
| CloudFront Function | Lightweight Edge Logic |
| Origin Shield | Additional Cache Layer |

---

# Summary

Amazon CloudFront is AWS's global Content Delivery Network (CDN) that accelerates content delivery by caching data at Edge Locations worldwide. Features such as cache policies, TTL, Origin Access Control (OAC), Signed URLs, Signed Cookies, Origin Shield, Lambda@Edge, CloudFront Functions, and multi-origin distributions enable organizations to build secure, high-performance, and globally scalable applications. When integrated with AWS WAF, AWS Shield, CloudWatch, Route 53, ACM, and S3, CloudFront becomes a key component of enterprise-grade web architectures, delivering low latency, high availability, and strong security.