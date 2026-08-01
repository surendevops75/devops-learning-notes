# AWS WAF and AWS Shield

---

# Introduction

AWS WAF (Web Application Firewall) and AWS Shield are AWS security services that protect web applications from common web attacks, bots, and Distributed Denial of Service (DDoS) attacks.

Modern internet-facing applications constantly face threats such as:

- SQL Injection
- Cross-Site Scripting (XSS)
- HTTP Floods
- SYN Floods
- UDP Floods
- Bot Attacks
- Credential Stuffing
- DDoS Attacks

AWS WAF protects applications at Layer 7 (Application Layer), while AWS Shield protects against Layer 3 and Layer 4 network attacks.

Together, they provide comprehensive protection for public-facing AWS applications.

---

# What is AWS WAF?

AWS WAF is a Web Application Firewall that monitors and filters HTTP and HTTPS requests reaching your applications.

It protects against

- SQL Injection (SQLi)
- Cross-Site Scripting (XSS)
- Malicious Bots
- IP Reputation Attacks
- Geographic Restrictions
- Rate-Based Attacks
- Custom Rules

---

# What is AWS Shield?

AWS Shield is a managed DDoS protection service.

It protects applications against

- Network Floods
- Transport Layer Attacks
- Infrastructure DDoS Attacks

AWS Shield is always running in the background.

---

# Why WAF and Shield?

Without Protection

```text
Internet

↓

Malicious Traffic

↓

Application

↓

Service Outage
```

With AWS WAF and Shield

```text
Internet

↓

AWS Shield

↓

AWS WAF

↓

Application Load Balancer

↓

Application
```

---

# Real World Problem

A banking application experiences

- SQL Injection Attempts
- Login Brute Force
- HTTP Flood Attacks
- Malicious Bots
- DDoS Traffic

Requirements

- Block Malicious Requests
- Detect Bots
- Prevent DDoS
- Monitor Security Events
- Maintain Availability

AWS WAF and Shield solve these problems.

---

# Enterprise Architecture

```text
Internet

↓

AWS Shield

↓

AWS WAF

↓

CloudFront

↓

Application Load Balancer

↓

Amazon ECS / EKS / EC2

↓

Amazon RDS
```

---

# AWS WAF Components

AWS WAF consists of

- Web ACL
- Rules
- Rule Groups
- Managed Rules
- Custom Rules
- IP Sets
- Regex Pattern Sets
- Rate-Based Rules

---

# Web ACL

A Web ACL (Access Control List) is the main container for WAF rules.

Architecture

```text
Web ACL

↓

Rule 1

↓

Rule 2

↓

Rule 3

↓

Allow / Block
```

Every protected application is associated with one Web ACL.

---

# Rules

Rules inspect incoming requests.

Possible Actions

- Allow
- Block
- Count
- CAPTCHA
- Challenge

---

# Rule Evaluation

Workflow

```text
Incoming Request

↓

Rule 1

↓

Rule 2

↓

Rule 3

↓

Decision
```

Rules are processed in priority order.

---

# Managed Rule Groups

AWS provides pre-built security rules.

Examples

- SQL Injection Protection
- Cross-Site Scripting Protection
- Linux Rules
- PHP Rules
- WordPress Rules
- Common Vulnerabilities

Benefits

- Regular Updates
- AWS Managed
- Enterprise Ready

---

# Custom Rules

Organizations create their own rules.

Examples

- Block specific IPs
- Restrict HTTP Methods
- Block User Agents
- Validate Headers
- Custom URI Filtering

---

# IP Sets

IP Sets contain

- Allowed IPs
- Blocked IPs

Example

```text
Corporate Office

↓

Allowed

------------------

Malicious IP

↓

Blocked
```

---

# Regex Pattern Sets

Regular Expressions match request patterns.

Examples

- URL Paths
- Query Parameters
- Headers
- Cookies

Useful for advanced filtering.

---

# Rate-Based Rules

Blocks excessive requests.

Example

```text
Client

↓

10,000 Requests

↓

WAF

↓

Block
```

Helps prevent

- HTTP Floods
- Login Abuse
- API Abuse

---

# Geographic Restrictions

Restrict access by country.

Example

```text
Allowed

India

USA

UK

Blocked

Other Countries
```

---

# Bot Control

AWS WAF Bot Control detects

- Web Crawlers
- Scrapers
- Automated Bots
- Credential Stuffing Bots

Reduces malicious automation.

---

# CAPTCHA

AWS WAF supports CAPTCHA.

Workflow

```text
Suspicious Request

↓

CAPTCHA

↓

Human Verified

↓

Allow
```

Prevents bot attacks.

---

# AWS Shield Standard

Included at no additional cost.

Protects against

- SYN Flood
- UDP Flood
- Reflection Attacks
- Network Layer DDoS

Automatically enabled.

---

# AWS Shield Advanced

Provides enhanced protection.

Features

- Advanced DDoS Detection
- DDoS Response Team (DRT)
- Cost Protection
- Detailed Metrics
- Enhanced Monitoring

Recommended for enterprise applications.

---

# WAF Integration

AWS WAF supports

- CloudFront
- Application Load Balancer
- API Gateway
- App Runner
- Verified Access

---

# CloudFront Integration

Architecture

```text
Internet

↓

CloudFront

↓

AWS WAF

↓

Origin
```

Protects applications globally.

---

# Application Load Balancer Integration

```text
Users

↓

AWS WAF

↓

ALB

↓

Application
```

---

# API Gateway Integration

```text
Users

↓

AWS WAF

↓

API Gateway

↓

Lambda
```

---

# Monitoring

CloudWatch Metrics

- Allowed Requests
- Blocked Requests
- CAPTCHA Requests
- Rate-Limited Requests

---

# Logging

AWS WAF logs can be sent to

- CloudWatch Logs
- Amazon S3
- Kinesis Data Firehose

Useful for

- Auditing
- Security Analysis
- Incident Response

---

# CloudWatch Alarms

Example

```text
Blocked Requests > 1000

↓

CloudWatch Alarm

↓

SNS Notification
```

---

# AWS CLI

Create Web ACL

```bash
aws wafv2 create-web-acl \
--name production-waf
```

List Web ACLs

```bash
aws wafv2 list-web-acls
```

List IP Sets

```bash
aws wafv2 list-ip-sets
```

---

# Terraform

```hcl
resource "aws_wafv2_web_acl" "production" {

  name = "production-waf"

  scope = "REGIONAL"

}
```

---

# CloudFormation

```yaml
Resources:

  WebACL:

    Type: AWS::WAFv2::WebACL

    Properties:

      Name: production-waf

      Scope: REGIONAL
```

---

# Python (Boto3)

```python
import boto3

waf = boto3.client("wafv2")

response = waf.list_web_acls(
    Scope="REGIONAL"
)

print(response)
```

---

# Enterprise Production Architecture

```text
                      Internet

                          │

                   AWS Shield Advanced

                          │

                      AWS WAF

                          │

                     CloudFront

                          │

               Application Load Balancer

                          │

        ┌─────────────────┼─────────────────┐

        │                 │                 │

      Amazon ECS      Amazon EKS        Amazon EC2

                          │

                     Amazon RDS

                          │

              CloudWatch • CloudTrail

                          │

                 Security Operations
```

---

# Best Practices

- Use AWS Managed Rule Groups
- Enable Rate-Based Rules
- Enable Bot Control
- Protect APIs with WAF
- Enable CloudWatch logging
- Monitor blocked requests
- Restrict unnecessary countries
- Enable AWS Shield Advanced for production
- Regularly review WAF logs
- Use least-privilege IAM policies

---

# Common Mistakes

- Creating WAF without rules
- Ignoring logs
- Not enabling rate limiting
- Blocking legitimate users
- No DDoS monitoring
- Not testing custom rules
- Exposing applications without WAF
- Using only IP filtering
- Ignoring bot traffic
- Missing CloudWatch alarms

---

# Troubleshooting

## Legitimate Users Blocked

Check

- Rule Priority
- IP Sets
- Regex Rules
- Rate Limits

---

## SQL Injection Not Blocked

Verify

- Managed Rule Group Enabled
- Rule Priority
- Logging
- Web ACL Association

---

## High False Positives

Review

- Custom Rules
- Regex Patterns
- Rate Limits

---

## DDoS Attack Continues

Check

- Shield Enabled
- CloudFront
- WAF Rules
- Shield Advanced Configuration

---

## WAF Not Protecting Application

Verify

- Web ACL Association
- Resource ARN
- Region
- Rule Actions

---

# Interview Questions

## Basic

1. What is AWS WAF?
2. What is AWS Shield?
3. WAF vs Shield?
4. What is a Web ACL?
5. What are WAF Rules?
6. What are Managed Rule Groups?
7. What are IP Sets?
8. What is Bot Control?
9. What is CAPTCHA?
10. What is Shield Standard?

---

## Intermediate

11. Shield Standard vs Shield Advanced?
12. Explain Rate-Based Rules.
13. Explain Rule Priority.
14. Explain Regex Pattern Sets.
15. WAF with API Gateway?
16. WAF with CloudFront?
17. WAF with ALB?
18. Explain Geographic Restrictions.
19. Explain WAF logging.
20. Explain CloudWatch integration.

---

## Advanced

21. Design a secure internet-facing architecture.
22. How would you mitigate an HTTP Flood attack?
23. Explain WAF request processing.
24. How would you protect login APIs from brute-force attacks?
25. Explain DDoS protection architecture.
26. How would you reduce false positives?
27. Design enterprise WAF rules.
28. Explain Shield Advanced benefits.
29. How would you monitor WAF effectiveness?
30. Best practices for production WAF deployments.

---

# Production Scenarios

### Scenario 1

A login API receives millions of requests from bots.

How would AWS WAF protect the application?

---

### Scenario 2

Attackers launch a SQL Injection attack against your production application.

Which WAF features would block the attack?

---

### Scenario 3

Your company launches a global application expected to receive DDoS attacks.

How would AWS Shield protect it?

---

### Scenario 4

Customers from specific countries should not access your application.

How would you configure AWS WAF?

---

### Scenario 5

A legitimate user is blocked by AWS WAF.

How would you troubleshoot the issue?

---

### Scenario 6

Security requires all internet-facing applications to use standardized WAF rules.

How would you implement this across multiple AWS accounts?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Web ACL | Main Rule Container |
| Rule | Inspect Requests |
| Rule Group | Collection of Rules |
| Managed Rules | AWS-Managed Security Rules |
| Custom Rules | Organization-Specific Rules |
| IP Set | Allow/Block IP Addresses |
| Regex Pattern Set | Pattern Matching |
| Rate-Based Rule | Prevent Request Flooding |
| CAPTCHA | Human Verification |
| Bot Control | Detect Automated Bots |
| Shield Standard | Basic DDoS Protection |
| Shield Advanced | Enterprise DDoS Protection |

---

# Summary

AWS WAF and AWS Shield together provide comprehensive protection for internet-facing AWS applications. AWS WAF secures Layer 7 traffic by filtering malicious HTTP/HTTPS requests using managed rules, custom rules, rate limiting, bot control, and CAPTCHA, while AWS Shield protects against Layer 3 and Layer 4 DDoS attacks. When integrated with CloudFront, Application Load Balancers, API Gateway, CloudWatch, and CloudTrail, they form a critical part of an enterprise security architecture, ensuring high availability, strong threat protection, and regulatory compliance.