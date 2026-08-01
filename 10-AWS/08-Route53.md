# Amazon Route 53

---

# Introduction

Amazon Route 53 is AWS's highly available, scalable, and fully managed Domain Name System (DNS) web service.

It helps users access applications by translating human-readable domain names into IP addresses that computers understand.

Besides DNS resolution, Route 53 also provides:

- Domain Registration
- DNS Management
- Health Checks
- Traffic Routing
- Failover
- Load Balancing Integration

Route 53 is one of the core networking services used in almost every AWS production environment.

---

# What is DNS?

DNS (Domain Name System) is often called the **Phonebook of the Internet**.

Humans remember:

```
www.amazon.com
```

Computers understand:

```
54.239.28.85
```

DNS converts domain names into IP addresses.

Without DNS, users would need to remember numerical IP addresses for every website.

---

# Why Do We Need DNS?

Imagine opening Amazon.

Instead of typing

```
www.amazon.com
```

you would have to type

```
54.239.xxx.xxx
```

Now imagine Amazon changes its servers.

Every user would need to know the new IP.

DNS solves this by mapping a domain name to the current IP address.

---

# What is Amazon Route 53?

Amazon Route 53 is AWS's managed DNS service.

It provides:

- Domain Registration
- Public DNS
- Private DNS
- Health Monitoring
- Intelligent Routing
- High Availability

It integrates seamlessly with:

- EC2
- ALB
- CloudFront
- S3
- API Gateway
- Elastic Beanstalk
- Global Accelerator

---

# Why is it Called Route 53?

The name comes from:

```
Port 53
```

DNS uses:

- UDP Port 53
- TCP Port 53

Hence the name **Route 53**.

---

# Real-World Problem

A company owns:

```
company.com
```

Requirements:

- Website
- API
- Admin Portal
- Disaster Recovery
- Multiple AWS Regions

Users should always connect to the nearest healthy application.

Amazon Route 53 makes this possible.

---

# Enterprise Architecture

```text
                     Users

                       │

                DNS Query

                       │

                 Amazon Route53

        ┌──────────────┼──────────────┐

        │              │              │

    Primary ALB    Secondary ALB   CloudFront

        │              │              │

     EC2 / EKS      EC2 / EKS      Amazon S3

        │

    CloudWatch Health Check
```

---

# How DNS Resolution Works

When a user enters:

```
www.company.com
```

The following happens:

```text
Browser

↓

Local DNS Cache

↓

ISP DNS Resolver

↓

Root Name Server

↓

TLD Name Server (.com)

↓

Authoritative Name Server (Route53)

↓

IP Address

↓

Browser connects to Server
```

---

# Core Components

Amazon Route 53 consists of:

- Hosted Zones
- DNS Records
- Routing Policies
- Health Checks
- Resolver
- Domain Registration
- Traffic Flow

---

# Hosted Zones

A Hosted Zone stores DNS records for a domain.

Two types:

- Public Hosted Zone
- Private Hosted Zone

---

# Public Hosted Zone

Used for:

Internet-facing applications.

Example:

```
company.com

↓

Public Hosted Zone

↓

Internet Users
```

---

# Private Hosted Zone

Used for:

Internal AWS resources.

Example:

```
database.internal

↓

Private Hosted Zone

↓

Amazon VPC
```

Only resources inside associated VPCs can resolve these records.

---

# Domain Registration

Route 53 can register domains.

Example:

```
company.com

company.in

company.org
```

After registration:

- Route 53 becomes the registrar (for supported TLDs)
- Hosted Zone can be created automatically

---

# DNS Record Types

## A Record

Maps:

```
Domain

↓

IPv4 Address
```

Example

```
www.company.com

↓

192.168.10.10
```

---

## AAAA Record

Maps:

```
Domain

↓

IPv6 Address
```

---

## CNAME

Maps one domain to another domain.

Example

```
api.company.com

↓

loadbalancer.amazonaws.com
```

Cannot be used for the root domain.

---

## Alias Record

AWS-specific feature.

Maps AWS resources directly.

Supports:

- ALB
- NLB
- CloudFront
- S3 Website
- API Gateway
- Global Accelerator

Advantages:

- No additional query charges
- Supports root domain
- Automatically updates target IPs

Example

```
company.com

↓

Alias

↓

Application Load Balancer
```

---

## MX Record

Used for Mail Servers.

Example

```
company.com

↓

mail.company.com
```

---

## TXT Record

Stores text information.

Common Uses:

- SPF
- DKIM
- Domain Verification
- Google Verification

---

## NS Record

Specifies Name Servers.

Example

```
company.com

↓

ns-123.awsdns.com
```

---

## SOA Record

Contains:

- Primary Name Server
- Contact Email
- Serial Number
- Refresh Interval

Automatically created.

---

# TTL (Time To Live)

TTL determines how long DNS responses remain cached.

Example

```
TTL

300 Seconds

↓

Cache

↓

Next DNS Query
```

Lower TTL:

- Faster updates
- More DNS queries

Higher TTL:

- Better performance
- More caching

---

# Health Checks

Route 53 can monitor endpoints.

Checks:

- HTTP
- HTTPS
- TCP

Example

```text
Health Check

↓

HTTP 200

↓

Healthy

↓

Traffic Allowed
```

If unhealthy:

Traffic automatically shifts based on routing policy.

---

# Routing Policies

Amazon Route 53 supports multiple routing strategies.

- Simple
- Weighted
- Latency
- Failover
- Geolocation
- Geoproximity
- Multi-Value

---

# Simple Routing

Single resource.

```text
Users

↓

Route53

↓

Application Load Balancer
```

Suitable for:

- Small applications
- Development

---

# Weighted Routing

Traffic is distributed based on configured weights.

Example

```
Server A

80%

Server B

20%
```

Useful for:

- Blue-Green Deployment
- Canary Releases
- Traffic Testing

---

# Latency-Based Routing

Routes users to the AWS Region with the lowest network latency.

Example

```text
India User

↓

Mumbai Region

----------------

US User

↓

Virginia Region
```

Improves user experience.

---

# Failover Routing

Primary Region

↓

Healthy

↓

Traffic

OR

Primary Fails

↓

Secondary Region

↓

Traffic

Best suited for Disaster Recovery.

---

# Geolocation Routing

Routes traffic based on the user's geographic location.

Example

```
India

↓

Mumbai

--------------

USA

↓

Virginia

--------------

Europe

↓

Frankfurt
```

Useful for:

- Compliance
- Regional content
- Localized websites

---

# Geoproximity Routing

Routes traffic based on:

- User Location
- AWS Region
- Traffic Bias

Allows shifting traffic toward or away from a Region.

Useful during migrations or phased deployments.

---

# Multi-Value Routing

Returns multiple healthy IP addresses.

Example

```
IP1

IP2

IP3
```

If one endpoint becomes unhealthy, it is removed from DNS responses.

---

# DNSSEC

DNS Security Extensions protect against DNS spoofing and cache poisoning.

Benefits:

- Data integrity
- DNS authentication
- Improved security

Recommended for public production domains.

---

# Route 53 Resolver

The Route 53 Resolver enables DNS resolution between AWS and on-premises networks.

Supports:

- Inbound Endpoints
- Outbound Endpoints
- Hybrid DNS

Architecture

```text
On-Premises

↓

VPN / Direct Connect

↓

Route53 Resolver

↓

Amazon VPC
```

---

# Integration with AWS Services

| Service | Integration |
|----------|-------------|
| EC2 | DNS Resolution |
| ALB | Alias Records |
| CloudFront | CDN |
| S3 | Static Website |
| API Gateway | Custom Domains |
| Global Accelerator | Global Routing |
| EKS | Ingress DNS |

---

# AWS Console Walkthrough

1. Open Route 53 Console
2. Create Hosted Zone
3. Register Domain (optional)
4. Create DNS Record
5. Choose Routing Policy
6. Configure TTL
7. Save Record
8. Test DNS Resolution

---

# AWS CLI

Create Hosted Zone

```bash
aws route53 create-hosted-zone \
--name company.com \
--caller-reference demo123
```

List Hosted Zones

```bash
aws route53 list-hosted-zones
```

List Record Sets

```bash
aws route53 list-resource-record-sets \
--hosted-zone-id Z123456789
```

---

# Terraform

```hcl
resource "aws_route53_zone" "main" {

  name = "company.com"

}
```

A Record

```hcl
resource "aws_route53_record" "www" {

  zone_id = aws_route53_zone.main.zone_id

  name = "www"

  type = "A"

  ttl = 300

  records = [

    aws_instance.web.public_ip

  ]

}
```

---

# CloudFormation

```yaml
Resources:

  HostedZone:

    Type: AWS::Route53::HostedZone

    Properties:

      Name: company.com
```

---

# Python (Boto3)

```python
import boto3

client = boto3.client("route53")

response = client.list_hosted_zones()

print(response)
```

---

# Enterprise Production Architecture

```text
                     Internet Users

                           │

                      Amazon Route53

                           │

                 Health Check Enabled

                           │

        ┌──────────────────┼──────────────────┐

        │                                     │

   Primary Region                       Secondary Region

        │                                     │

 Application Load Balancer         Application Load Balancer

        │                                     │

     EC2 / EKS                           EC2 / EKS
```

---

# Best Practices

- Use Alias Records for AWS resources
- Enable Health Checks
- Use Failover Routing for DR
- Configure Low TTL during migrations
- Use DNSSEC for public domains
- Use Private Hosted Zones for internal services
- Monitor Health Checks
- Tag Hosted Zones

---

# Common Mistakes

- Using CNAME for root domain
- High TTL during migrations
- Missing Health Checks
- Public records for private services
- Incorrect Alias targets
- Ignoring DNS propagation time

---

# Troubleshooting

## DNS Not Resolving

Check:

- Hosted Zone
- NS Records
- TTL
- Domain Registrar
- Record Type

---

## Website Opens Intermittently

Verify:

- Health Checks
- Routing Policy
- ALB Health
- DNS Cache

---

## Wrong Server Responding

Check:

- Weighted Routing
- Geolocation Rules
- Alias Record
- DNS Cache

---

# Interview Questions

1. What is DNS?
2. What is Route 53?
3. Why is it called Route 53?
4. Difference between Public and Private Hosted Zones?
5. Difference between CNAME and Alias?
6. What is TTL?
7. Explain Weighted Routing.
8. Explain Latency Routing.
9. Explain Failover Routing.
10. Explain Geolocation Routing.
11. Explain Geoproximity Routing.
12. What is Multi-Value Routing?
13. What is DNSSEC?
14. What are Health Checks?
15. How does Route 53 integrate with ALB?
16. Can Route 53 register domains?
17. What is Route 53 Resolver?
18. How do you troubleshoot DNS issues?
19. What is an Alias Record?
20. Design a highly available DNS architecture.

---

# Scenario-Based Questions

### Scenario 1

Users report that your website is unreachable after changing DNS records.

How would you troubleshoot?

---

### Scenario 2

You need to gradually shift 20% of production traffic to a new application version.

Which routing policy would you use and why?

---

### Scenario 3

Your primary AWS Region is unavailable.

How can Route 53 redirect traffic automatically?

---

### Scenario 4

A company wants users from Europe and Asia to reach different application endpoints.

Which routing policy would you recommend?

---

### Scenario 5

An internal application hosted inside a VPC should not be accessible from the internet.

How would you configure Route 53?

---

# Cheat Sheet

| Feature | Amazon Route 53 |
|---------|-----------------|
| Service Type | Managed DNS |
| Port | UDP/TCP 53 |
| Public Hosted Zone | Internet DNS |
| Private Hosted Zone | VPC DNS |
| Alias Records | AWS Resources |
| Health Checks | Yes |
| DNSSEC | Supported |
| Domain Registration | Supported |
| Routing Policies | 7 Types |
| Hybrid DNS | Route 53 Resolver |

---

# Summary

Amazon Route 53 is a highly available and scalable DNS service that provides domain registration, DNS management, intelligent traffic routing, and health monitoring. It integrates deeply with AWS services such as ALB, CloudFront, API Gateway, and S3 to deliver resilient and performant applications.

In production environments, use Alias records for AWS resources, enable health checks for failover, choose routing policies based on business requirements, and leverage Private Hosted Zones for internal service discovery.