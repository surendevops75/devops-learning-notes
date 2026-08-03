# AWS Certificate Manager (ACM)

---

# Introduction

AWS Certificate Manager (ACM) is a fully managed service that simplifies the provisioning, management, deployment, and renewal of SSL/TLS certificates used to secure AWS applications and websites.

SSL/TLS certificates encrypt data transmitted between clients and servers, ensuring confidentiality, integrity, and authentication. Managing certificates manually can be complex due to expiration, renewal, and deployment processes. AWS ACM automates these tasks, reducing operational overhead and improving security.

AWS Certificate Manager integrates with

- Elastic Load Balancer (ALB/NLB)
- Amazon CloudFront
- Amazon API Gateway
- AWS Elastic Beanstalk
- Amazon CloudFront
- AWS App Runner
- AWS Private Certificate Authority (Private CA)
- Amazon Route 53

ACM is the recommended service for managing public SSL/TLS certificates on AWS.

---

# What is AWS Certificate Manager?

AWS Certificate Manager provisions and manages SSL/TLS certificates.

It helps organizations

- Secure Websites
- Encrypt Network Traffic
- Automate Certificate Renewal
- Simplify Certificate Deployment
- Improve Application Security

Workflow

```text
Client

↓

HTTPS Request

↓

SSL/TLS Certificate

↓

AWS Certificate Manager

↓

Secure AWS Application
```

---

# Why AWS ACM?

Without ACM

```text
Purchase Certificate

↓

Manual Installation

↓

Track Expiration

↓

Manual Renewal

↓

Service Outage Risk
```

Problems

- Manual Certificate Management
- Expired Certificates
- Service Downtime
- Operational Overhead

With ACM

```text
Request Certificate

↓

Automatic Validation

↓

Automatic Deployment

↓

Automatic Renewal

↓

Secure Application
```

---

# Real World Problem Statement

An enterprise hosts

- Customer Portal
- Banking APIs
- E-Commerce Website
- Internal Applications

Requirements

- HTTPS Everywhere
- Automatic Certificate Renewal
- Central Certificate Management
- Secure Communications

AWS ACM provides automated SSL/TLS certificate lifecycle management.

---

# Enterprise Architecture

```text
Internet Users

        │

HTTPS Request

        │

        ▼

Amazon CloudFront

        │

Application Load Balancer

        │

AWS Certificate Manager

        │

EC2 / ECS / EKS / Lambda
```

---

# Core Components

AWS ACM consists of

- Public Certificates
- Imported Certificates
- Certificate Validation
- Automatic Renewal
- Certificate Deployment
- AWS Integrations
- Private Certificates (via Private CA)

---

# Public Certificates

AWS ACM can issue trusted public SSL/TLS certificates.

Used for

- Websites
- APIs
- Load Balancers
- CloudFront Distributions

Benefits

- Free of Cost
- Automatic Renewal
- AWS Managed

---

# Imported Certificates

Organizations can import third-party certificates into ACM.

Examples

- DigiCert
- Sectigo
- GlobalSign
- Entrust

Imported certificates require manual renewal.

---

# Domain Validation

Before issuing a certificate, ACM validates domain ownership.

Validation Methods

- DNS Validation
- Email Validation

DNS validation is recommended because it supports automatic renewal.

---

# DNS Validation

Workflow

```text
Request Certificate

↓

Create DNS Record

↓

Route 53 Validation

↓

Certificate Issued
```

Benefits

- Automatic Renewal
- Easy Management
- Highly Recommended

---

# Email Validation

Workflow

```text
Request Certificate

↓

Validation Email

↓

Approve Request

↓

Certificate Issued
```

Requires manual approval.

---

# Automatic Renewal

AWS automatically renews eligible ACM-issued certificates before expiration.

Requirements

- ACM-Issued Certificate
- DNS Validation
- Certificate In Use

Imported certificates are **not** automatically renewed.

---

# Wildcard Certificates

Wildcard certificates secure multiple subdomains.

Example

```text
*.example.com

↓

api.example.com

www.example.com

dev.example.com
```

Reduces certificate management overhead.

---

# Multi-Domain Certificates

One certificate can secure multiple domains.

Example

```text
example.com

api.example.com

shop.example.com

example.org
```

Uses Subject Alternative Names (SANs).

---

# Subject Alternative Name (SAN)

SAN allows multiple domain names within a single certificate.

Benefits

- Fewer Certificates
- Easier Management
- Lower Complexity

---

# ACM Integrations

AWS ACM integrates with

- Application Load Balancer
- Network Load Balancer (TLS)
- CloudFront
- API Gateway
- Elastic Beanstalk
- App Runner

Certificates can be attached directly without exporting private keys.

---

# Certificate Lifecycle

```text
Request Certificate

↓

Domain Validation

↓

Certificate Issued

↓

Deploy to AWS Service

↓

Automatic Renewal

↓

Expiration Prevention
```

---

# AWS CLI

Request Certificate

```bash
aws acm request-certificate \
--domain-name example.com \
--validation-method DNS
```

List Certificates

```bash
aws acm list-certificates
```

Describe Certificate

```bash
aws acm describe-certificate \
--certificate-arn <certificate-arn>
```

---

# Terraform

```hcl
resource "aws_acm_certificate" "website" {

  domain_name       = "example.com"

  validation_method = "DNS"

}
```

---

# CloudFormation

```yaml
Resources:

  WebsiteCertificate:

    Type: AWS::CertificateManager::Certificate

    Properties:

      DomainName: example.com

      ValidationMethod: DNS
```

---

# Python (Boto3)

```python
import boto3

acm = boto3.client("acm")

response = acm.list_certificates()

print(response)
```

---

# Enterprise Production Architecture

```text
                  Internet

                     │

               Amazon Route 53

                     │

                Amazon CloudFront

                     │

          AWS Certificate Manager

                     │

       Application Load Balancer

                     │

      EC2 • ECS • EKS • Lambda
```

---

# Best Practices

- Use DNS validation whenever possible
- Enable HTTPS for all public applications
- Use ACM-issued certificates instead of self-signed certificates
- Monitor certificate expiration
- Use wildcard certificates where appropriate
- Use SAN certificates to reduce certificate sprawl
- Store private certificates in AWS Private CA
- Use CloudFront with ACM for global applications
- Regularly review certificate inventory
- Remove unused certificates
- Use strong TLS policies
- Follow least-privilege IAM permissions for ACM

---

# Common Mistakes

- Using expired certificates
- Choosing email validation when DNS validation is available
- Forgetting to renew imported certificates
- Using self-signed certificates in production
- Deploying HTTP instead of HTTPS
- Leaving unused certificates in ACM
- Weak TLS security policies
- Incorrect domain validation records
- Using multiple certificates unnecessarily
- Missing certificate monitoring

---

# Troubleshooting

## Certificate Pending Validation

Check

- DNS Record
- Route 53 Configuration
- Domain Ownership
- Validation Status

---

## Certificate Not Renewing

Verify

- DNS Validation
- Certificate In Use
- ACM-Issued Certificate

---

## HTTPS Not Working

Check

- Load Balancer Listener
- ACM Certificate Attachment
- Security Groups
- DNS Configuration

---

## Domain Validation Failed

Verify

- Correct CNAME Record
- DNS Propagation
- Hosted Zone

---

## Imported Certificate Expired

Check

- Manual Renewal
- Import Updated Certificate
- Deployment Status

---

# Interview Questions

## Basic

1. What is AWS Certificate Manager?
2. Why use ACM?
3. What is SSL/TLS?
4. What is DNS validation?
5. What is email validation?
6. What is automatic certificate renewal?
7. What is a wildcard certificate?
8. What is a SAN certificate?
9. Which AWS services integrate with ACM?
10. Can ACM export private keys?

---

## Intermediate

11. Explain ACM certificate lifecycle.
12. Explain DNS vs Email validation.
13. Explain wildcard certificates.
14. Explain SAN certificates.
15. Explain ACM integration with ALB.
16. Explain ACM integration with CloudFront.
17. Explain certificate renewal.
18. Explain imported certificates.
19. Explain TLS policies.
20. Explain ACM best practices.

---

## Advanced

21. Design enterprise certificate management using ACM.
22. Explain ACM vs AWS Private CA.
23. Design secure HTTPS architecture.
24. Explain certificate automation.
25. Design global HTTPS infrastructure.
26. Explain certificate governance.
27. Design multi-domain certificate strategy.
28. Explain operational best practices.
29. Design enterprise certificate lifecycle management.
30. Best practices for AWS Certificate Manager.

---

# Production Scenarios

### Scenario 1

Your Application Load Balancer must serve HTTPS traffic for **www.example.com**.

How would ACM simplify certificate management?

---

### Scenario 2

A certificate expires every year, causing production outages.

How would ACM prevent this issue?

---

### Scenario 3

A company hosts **api.example.com**, **shop.example.com**, and **www.example.com**.

Would you use multiple certificates or a SAN certificate? Explain.

---

### Scenario 4

Your organization uses CloudFront for global content delivery.

How does ACM integrate with CloudFront to provide HTTPS?

---

### Scenario 5

A company purchases SSL certificates from DigiCert.

Can these certificates be used with ACM? What limitations apply?

---

### Scenario 6

An enterprise requires automatic certificate renewal with minimal administrative effort.

Which validation method should be selected, and why?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Public Certificate | Free AWS-Issued SSL/TLS Certificate |
| Imported Certificate | Third-Party Certificate |
| DNS Validation | Automatic Validation & Renewal |
| Email Validation | Manual Validation |
| Wildcard Certificate | Secure Multiple Subdomains |
| SAN Certificate | Secure Multiple Domains |
| Automatic Renewal | AWS-Managed Renewal |
| CloudFront | Global HTTPS |
| ALB | HTTPS Load Balancing |
| Route 53 | DNS Validation |

---

# Summary

AWS Certificate Manager (ACM) is a managed service that simplifies the provisioning, deployment, and lifecycle management of SSL/TLS certificates for AWS applications. By supporting free public certificates, DNS validation, automatic renewal, wildcard and SAN certificates, and seamless integration with services such as Application Load Balancer, CloudFront, API Gateway, and App Runner, ACM enables organizations to secure applications with HTTPS while minimizing operational effort and reducing the risk of certificate-related outages.