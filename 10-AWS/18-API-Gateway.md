# Amazon API Gateway

---

# Introduction

Amazon API Gateway is a fully managed AWS service that enables developers to create, publish, secure, monitor, and manage APIs at any scale.

It acts as the front door for backend applications and services.

Instead of allowing clients to directly access backend resources, API Gateway provides a centralized entry point for:

- Authentication
- Authorization
- Rate Limiting
- Request Validation
- Monitoring
- Caching
- Traffic Management

API Gateway is widely used with:

- AWS Lambda
- ECS
- EKS
- EC2
- Elastic Load Balancer
- Private VPC Services

It is one of the core services used in modern serverless and microservices architectures.

---

# What is API Gateway?

API Gateway is a managed API management service.

It accepts client requests and forwards them to backend services.

Workflow

```text
Client

↓

API Gateway

↓

Backend Service

↓

Response
```

The backend can be

- Lambda
- ECS
- EKS
- EC2
- ALB
- HTTP APIs
- AWS Services

---

# Why API Gateway?

Without API Gateway

```text
Users

↓

Application

↓

EC2

↓

Database
```

Problems

- No Authentication
- No Rate Limiting
- No Monitoring
- Difficult Scaling
- No API Versioning

With API Gateway

```text
Users

↓

API Gateway

↓

Authentication

↓

Rate Limiting

↓

Backend

↓

Response
```

---

# Real World Problem

An enterprise exposes APIs for

- Mobile Apps
- Web Applications
- Third-Party Partners

Requirements

- Secure APIs
- API Keys
- Rate Limits
- Monitoring
- Logging
- Versioning
- HTTPS

Amazon API Gateway provides all these features.

---

# Enterprise Architecture

```text
Internet

↓

Route53

↓

Amazon API Gateway

↓

Authentication

↓

Lambda / ECS / EKS / ALB

↓

Amazon RDS
```

---

# API Types

Amazon API Gateway supports

- REST API
- HTTP API
- WebSocket API

---

# REST API

Supports

- Authentication
- API Keys
- Usage Plans
- Request Validation
- Transformation
- Caching

Best for

Enterprise APIs

---

# HTTP API

A lightweight API service.

Advantages

- Lower Cost
- Lower Latency
- Simpler Configuration

Ideal for

- Lambda
- Microservices

---

# WebSocket API

Supports

- Real-Time Chat
- Gaming
- Notifications
- Live Dashboards

Example

```text
Client

↔

WebSocket API

↔

Backend
```

---

# API Resources

Example

```
/users

/orders

/products

/payments
```

---

# HTTP Methods

Supported Methods

- GET
- POST
- PUT
- DELETE
- PATCH
- OPTIONS
- HEAD

---

# Stages

Stages represent deployment environments.

Examples

```text
Development

↓

Testing

↓

Staging

↓

Production
```

Each stage has a unique endpoint.

---

# Endpoint Types

API Gateway supports

### Edge Optimized

Uses CloudFront.

Best for

Global APIs.

---

### Regional

Available within one AWS Region.

Most common option.

---

### Private

Accessible only from a VPC.

Ideal for internal enterprise APIs.

---

# Authentication

Supported Methods

- IAM
- Cognito
- Lambda Authorizer
- JWT
- API Keys

---

# IAM Authentication

AWS IAM controls API access.

Example

```text
IAM User

↓

API Gateway

↓

Access Granted
```

---

# Cognito Authentication

Amazon Cognito authenticates end users.

Workflow

```text
User

↓

Login

↓

JWT Token

↓

API Gateway

↓

Backend
```

---

# Lambda Authorizer

Custom authentication logic.

Example

```text
API Request

↓

Lambda

↓

Validate Token

↓

Allow / Deny
```

---

# API Keys

API Keys identify clients.

Example

```text
Partner A

↓

API Key

↓

API Gateway
```

Often used with Usage Plans.

---

# Usage Plans

Control

- Requests Per Second
- Monthly Quotas
- Daily Limits

Example

```text
1000 Requests

↓

Per Day
```

---

# Throttling

Protects backend services.

Example

```
100 Requests/Second
```

Excess requests receive

```
429 Too Many Requests
```

---

# Request Validation

API Gateway validates

- Headers
- Query Parameters
- Request Body

Invalid requests are rejected before reaching the backend.

---

# Request Transformation

Modify requests before forwarding.

Example

```text
Client JSON

↓

API Gateway

↓

Backend JSON
```

---

# Response Transformation

Modify backend responses before returning them to clients.

---

# Caching

API Gateway supports response caching.

Workflow

```text
Request

↓

Cache

↓

Response

↓

Backend (If Needed)
```

Benefits

- Lower Latency
- Reduced Backend Load
- Lower Cost

---

# Logging

Logs are stored in CloudWatch.

Includes

- Requests
- Responses
- Errors
- Latency

---

# Monitoring

CloudWatch Metrics

- Request Count
- Latency
- 4XX Errors
- 5XX Errors
- Cache Hit Ratio

---

# Integration Types

API Gateway integrates with

- Lambda
- HTTP Services
- ALB
- AWS Services
- Mock APIs

---

# Lambda Integration

```text
Client

↓

API Gateway

↓

Lambda

↓

Response
```

---

# ECS / EKS Integration

```text
Users

↓

API Gateway

↓

ALB

↓

ECS / EKS
```

---

# Private Integration

Private APIs communicate through a VPC Link.

```text
API Gateway

↓

VPC Link

↓

Internal ALB

↓

Private Services
```

---

# AWS CLI

Create API

```bash
aws apigateway create-rest-api \
--name production-api
```

List APIs

```bash
aws apigateway get-rest-apis
```

---

# Terraform

```hcl
resource "aws_api_gateway_rest_api" "api" {

  name = "production-api"

}
```

---

# Best Practices

- Use HTTPS only
- Enable Authentication
- Configure Usage Plans
- Enable Throttling
- Enable CloudWatch Logs
- Use API Versioning
- Enable Caching
- Protect APIs using AWS WAF
- Use Least Privilege IAM
- Monitor API Metrics

---

# Common Mistakes

- Public APIs without authentication
- No Rate Limiting
- Missing Logging
- Hardcoding Secrets
- No API Versioning
- Ignoring 4XX/5XX Metrics

---

# Troubleshooting

## 403 Forbidden

Check

- IAM
- Cognito
- API Keys

---

## 429 Too Many Requests

Verify

- Usage Plan
- Throttling Limits

---

## High Latency

Check

- Backend
- Lambda Cold Starts
- Network
- Cache

---

## 5XX Errors

Verify

- Backend Service
- Lambda Logs
- Integration Configuration

---

# Interview Questions

1. What is Amazon API Gateway?
2. REST API vs HTTP API?
3. REST API vs WebSocket API?
4. What is API Gateway used for?
5. Explain API Keys.
6. Explain Usage Plans.
7. What is throttling?
8. What is request validation?
9. Explain Lambda Authorizer.
10. Explain Cognito integration.
11. Explain VPC Link.
12. API Gateway vs ALB?
13. How does API Gateway integrate with Lambda?
14. How do you secure APIs?
15. How do you monitor APIs?

---

# Scenario Questions

### Scenario 1

A public API receives millions of requests during a marketing campaign.

How would you protect backend services?

---

### Scenario 2

An internal microservice should only be accessible from within a VPC.

Which API Gateway endpoint type would you choose?

---

### Scenario 3

A mobile application requires user authentication before accessing APIs.

How would you design the authentication flow?

---

### Scenario 4

Customers report high API latency.

How would you troubleshoot the issue?

---

### Scenario 5

A partner exceeds their API quota every day.

How would Usage Plans help manage this?

---

# Summary

Amazon API Gateway is a fully managed API management service that provides secure, scalable, and monitored access to backend services. By integrating with Lambda, ECS, EKS, ALB, Cognito, IAM, CloudWatch, and AWS WAF, API Gateway enables organizations to build production-ready APIs with authentication, authorization, throttling, caching, and monitoring while protecting backend applications from excessive traffic and unauthorized access.