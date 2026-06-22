# DNS Records and TTL

## Introduction

DNS works by storing different types of records.

Each record has a specific purpose such as:

- Mapping a domain to an IP address
- Redirecting one domain to another
- Configuring mail servers
- Domain ownership verification
- Defining authoritative nameservers

DNS records are stored in a DNS Zone.

---

# What is a DNS Record?

A DNS Record is an entry inside a DNS Zone that provides information about a domain.

Example:

```text
daws86s.fun → 54.88.122.243
```

DNS Record:

```text
Type : A
Name : daws86s.fun
Value: 54.88.122.243
```

---

# Common DNS Records

## A Record

A Record maps a domain name to an IPv4 address.

Example:

```text
daws86s.fun
        │
        ▼
54.88.122.243
```

DNS Record:

```text
Type  : A
Name  : daws86s.fun
Value : 54.88.122.243
```

---

## Use Cases

Frontend Website

```text
daws86s.fun
```

Backend API

```text
api.daws86s.fun
```

Database

```text
mysql.daws86s.fun
```

---

# AAAA Record

AAAA Record maps a domain name to an IPv6 address.

Example:

```text
Type  : AAAA
Name  : daws86s.fun
Value : 2404:6800:4007:80a::200e
```

---

# IPv4 vs IPv6

## IPv4

```text
54.88.122.243
```

---

## IPv6

```text
2404:6800:4007:80a::200e
```

---

# CNAME Record

CNAME stands for:

```text
Canonical Name
```

Used to point one domain to another domain.

Example:

```text
www.daws86s.fun
        │
        ▼
daws86s.fun
```

DNS Record:

```text
Type  : CNAME
Name  : www
Value : daws86s.fun
```

---

# CNAME Use Cases

Redirect:

```text
www.example.com
```

to

```text
example.com
```

---

Point:

```text
app.company.com
```

to

```text
loadbalancer.amazonaws.com
```

---

# MX Record

MX stands for:

```text
Mail Exchange
```

Used to define mail servers for a domain.

Example:

```text
company.com
```

Email:

```text
user@company.com
```

DNS Record:

```text
Type  : MX
Value : mail.company.com
```

---

# MX Record Example

```text
company.com
        │
        ▼
Google Workspace Mail Server
```

or

```text
Microsoft 365 Mail Server
```

---

# TXT Record

TXT Record stores text information.

Common Uses:

- Domain Verification
- SPF Records
- DKIM Records
- Application Verification

---

## Example

Google Verification:

```text
google-site-verification=abc123xyz
```

DNS Record:

```text
Type : TXT
```

---

# TXT Record Use Cases

Verify ownership for:

- Google Workspace
- Microsoft 365
- GitHub
- AWS
- Cloudflare

---

# NS Record

NS stands for:

```text
Name Server
```

NS records identify which nameservers are authoritative for a domain.

Example:

```text
daws86s.fun
```

Nameservers:

```text
ns-123.awsdns-10.com
ns-456.awsdns-20.net
ns-789.awsdns-30.org
ns-321.awsdns-40.co.uk
```

---

# Purpose of NS Records

```text
Domain
    │
    ▼
Which DNS Server Controls This Domain?
```

Answer is provided by NS Records.

---

# SOA Record

SOA stands for:

```text
Start Of Authority
```

Every DNS Zone contains one SOA Record.

Provides:

- Zone Owner Information
- Serial Number
- Refresh Interval
- Retry Interval
- Expiry Information

---

# Example SOA Record

```text
Primary Nameserver
Admin Contact
Serial Number
Refresh Timer
Retry Timer
Expiry Timer
```

---

# Record Summary

| Record | Purpose |
|----------|----------|
| A | Domain → IPv4 |
| AAAA | Domain → IPv6 |
| CNAME | Domain → Domain |
| MX | Mail Server |
| TXT | Verification/Text Data |
| NS | Nameserver Information |
| SOA | Zone Authority Information |

---

# What is TTL?

TTL stands for:

```text
Time To Live
```

TTL determines how long a DNS response can be cached before a fresh lookup is required.

Example:

```text
TTL = 300 seconds
```

Meaning:

```text
Cache for 5 minutes
```

After 5 minutes:

```text
Query DNS Again
```

---

# Why TTL Exists

Without caching:

```text
Every Request
       │
       ▼
DNS Server
```

This creates unnecessary load.

TTL reduces DNS traffic.

---

# DNS Caching

Example:

```text
daws86s.fun
      │
      ▼
54.88.122.243
```

TTL:

```text
3600 seconds
```

DNS Resolver stores:

```text
daws86s.fun → 54.88.122.243
```

for one hour.

Future requests use cache instead of querying DNS again.

---

# High TTL

Example:

```text
86400 seconds
```

(24 Hours)

Advantages:

- Reduced DNS Queries
- Better Performance
- Less Load on DNS Infrastructure

Best For:

- Stable Websites
- Static Infrastructure

---

# Low TTL

Example:

```text
60 seconds
```

Advantages:

- Faster DNS Updates
- Faster Migration

Best For:

- Server Migrations
- Load Balancer Changes
- Disaster Recovery Activities

---

# TTL Examples

## Stable Production Website

```text
TTL = 3600
TTL = 86400
```

---

## Migration Activity

Before migration:

```text
TTL = 60
```

Change server.

Wait for propagation.

Increase TTL later.

---

# DNS Resolution Flow

```text
User
 │
 ▼
Browser
 │
 ▼
DNS Resolver
 │
 ▼
DNS Cache
 │
 ├── Cache Hit
 │      │
 │      ▼
 │   Return IP
 │
 └── Cache Miss
        │
        ▼
     Nameserver
        │
        ▼
      DNS Record
```

---

# Route53 Record Examples

## Frontend

```text
Type  : A
Name  : daws86s.fun
Value : 54.88.122.243
```

---

## Database

```text
Type  : A
Name  : mysql.daws86s.fun
Value : 172.31.30.95
```

---

## WWW Redirect

```text
Type  : CNAME
Name  : www
Value : daws86s.fun
```

---

# Common DNS Troubleshooting

## Verify Domain Resolution

```bash
nslookup daws86s.fun
```

---

## Detailed Lookup

```bash
dig daws86s.fun
```

---

## Check Nameservers

```bash
dig NS daws86s.fun
```

---

## Check Mail Records

```bash
dig MX company.com
```

---

## Check TXT Records

```bash
dig TXT company.com
```

---

# Frequently Asked Interview Questions

## What is an A Record?

Maps a domain name to an IPv4 address.

---

## What is a CNAME Record?

Maps one domain name to another domain name.

---

## What is an MX Record?

Specifies mail servers responsible for receiving emails.

---

## What is a TXT Record?

Stores text information used for verification and security.

---

## What is an NS Record?

Specifies authoritative nameservers for a domain.

---

## What is an SOA Record?

Provides authoritative information about a DNS Zone.

---

## What is TTL?

Time To Live.

Determines how long DNS responses remain cached.

---

## Why Reduce TTL Before Migration?

To ensure DNS changes propagate quickly and users reach the new infrastructure sooner.

---

# Key Takeaways

- DNS records control how domains resolve and function.
- A records map domains to IPv4 addresses.
- AAAA records map domains to IPv6 addresses.
- CNAME records map one domain to another domain.
- MX records configure email routing.
- TXT records are commonly used for verification and security.
- NS records define authoritative nameservers.
- SOA records define zone authority information.
- TTL controls DNS caching behavior.
- High TTL reduces DNS traffic.
- Low TTL is useful during migrations and infrastructure changes.