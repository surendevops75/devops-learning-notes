# DNS and Domain Management

## Introduction

Humans prefer to remember names.

Computers communicate using numbers.

Example:

```text
Human Friendly:
google.com
amazon.in
facebook.com

Computer Friendly:
142.250.183.14
13.224.189.35
157.240.22.35
```

DNS bridges the gap between human-readable names and computer-readable IP addresses.

---

# What is DNS?

DNS stands for:

```text
Domain Name System
```

DNS converts:

```text
Domain Name
        │
        ▼
IP Address
```

Example:

```text
google.com
      │
      ▼
142.250.xxx.xxx
```

Without DNS, users would need to remember IP addresses for every website.

---

# Why DNS Exists

Consider a phonebook.

```text
Name        Number
--------------------
Ramesh      9876543210
Suresh      9876543211
Mahesh      9876543212
```

People remember names.

The phone system uses numbers.

DNS works in the same way:

```text
Domain Name  →  IP Address
```

---

# DNS Resolution Process

When a user opens:

```text
https://daws86s.fun
```

The following happens:

```text
Browser
   │
   ▼
DNS Resolver
   │
   ▼
DNS Records
   │
   ▼
IP Address
   │
   ▼
Web Server
```

The browser then connects to the returned IP address.

---

# DNS Resolver

A DNS Resolver is software that converts domain names into IP addresses.

Usually provided by:

- ISP
- Google DNS
- Cloudflare DNS
- Enterprise DNS Servers

Examples:

```text
Google DNS     : 8.8.8.8
Google DNS     : 8.8.4.4

Cloudflare DNS : 1.1.1.1
```

---

# DNS Lookup Example

User requests:

```text
daws86s.fun
```

DNS Resolver performs:

```text
daws86s.fun
      │
      ▼
Find IP Address
      │
      ▼
54.xx.xx.xx
```

Browser connects to:

```text
http://54.xx.xx.xx
```

---

# Internet Governance

The Internet requires centralized coordination.

This responsibility is handled by:

```text
ICANN
```

---

# What is ICANN?

ICANN stands for:

```text
Internet Corporation for Assigned Names and Numbers
```

ICANN is a non-profit organization responsible for:

- Domain Names
- IP Address Allocation
- DNS Root Management
- Internet Naming Standards

---

# Domain Name Structure

Example:

```text
www.daws86s.fun
```

Structure:

```text
www        -> Subdomain

daws86s    -> Domain Name

.fun       -> Top Level Domain (TLD)
```

---

# What is a TLD?

TLD stands for:

```text
Top Level Domain
```

Examples:

```text
.com
.in
.org
.net
.io
.edu
.us
.uk
.ai
.shop
.fun
.online
.tv
```

---

# Popular TLDs

## .com

Commercial websites.

Examples:

```text
google.com
amazon.com
```

---

## .org

Organizations.

Examples:

```text
wikipedia.org
```

---

## .edu

Educational institutions.

Examples:

```text
mit.edu
```

---

## .in

Country code TLD for India.

Managed under Indian domain administration.

Examples:

```text
amazon.in
```

---

## .us

Country code TLD for United States.

---

## .uk

Country code TLD for United Kingdom.

---

## .ai

Popular among AI companies.

Originally assigned to:

```text
Anguilla
```

---

## .tv

Originally assigned to:

```text
Tuvalu
```

Widely used for media and streaming services.

---

# Domain Registration

To use a domain name, it must be registered.

Example:

```text
daws86s.online
```

---

# Domain Registrars

Registrars are companies authorized to sell domain names.

Examples:

- GoDaddy
- Hostinger
- Namecheap
- Google Domains
- Route53

---

# Domain Purchase Process

Example:

```text
daws86s.online
```

Step 1:

Search domain availability.

---

Step 2:

Purchase through a registrar.

Example:

```text
GoDaddy
Hostinger
```

---

Step 3:

Provide ownership information.

Examples:

```text
Name
Email
Address
Phone Number
```

---

Step 4:

Registrar updates registration details.

---

Step 5:

Domain becomes active.

---

# Domain Ownership Flow

```text
ICANN
   │
   ▼
TLD Registry
   │
   ▼
Registrar
   │
   ▼
Customer
```

---

# TLD Registry

The TLD registry owns and manages the top-level domain.

Examples:

```text
.fun Registry

.com Registry

.in Registry
```

Responsibilities:

- Domain Registration Database
- Domain Ownership Records
- Nameserver Information

---

# Registrar Responsibilities

Registrars provide:

- Domain Purchase
- Domain Renewal
- DNS Management
- Nameserver Management

Examples:

```text
GoDaddy
Hostinger
```

---

# Domain Renewal

Domains are rented, not permanently owned.

Example:

```text
1 Year
2 Years
5 Years
```

If not renewed:

```text
Domain Expires
```

and may become available to others.

---

# Nameservers

Nameservers tell the Internet:

```text
Who manages the DNS records?
```

Example:

```text
daws86s.fun
```

may use:

```text
Hostinger Nameservers
```

or

```text
AWS Route53 Nameservers
```

---

# Why Change Nameservers?

Scenario:

Current DNS Management:

```text
Hostinger
```

Move DNS Management To:

```text
AWS Route53
```

Process:

```text
Update Nameservers
        │
        ▼
TLD Registry
        │
        ▼
Route53 becomes DNS Authority
```

---

# Domain Management Example

Domain:

```text
daws86s.fun
```

Frontend:

```text
daws86s.fun
      │
      ▼
Public IP
```

Backend Database:

```text
mysql.daws86s.fun
      │
      ▼
Private IP
```

DNS resolves these names to their configured destinations.

---

# Common DNS Tools

## nslookup

```bash
nslookup google.com
```

---

## dig

```bash
dig google.com
```

---

## host

```bash
host google.com
```

---

# Verify Domain Resolution

```bash
nslookup daws86s.fun
```

Output:

```text
Domain Name
IP Address
```

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
Nameserver
 │
 ▼
IP Address
 │
 ▼
Web Server
```

---

# Frequently Asked Interview Questions

## What is DNS?

Domain Name System.

Converts domain names into IP addresses.

---

## Why is DNS Required?

Humans remember names.

Computers communicate using IP addresses.

---

## What is ICANN?

Internet Corporation for Assigned Names and Numbers.

Responsible for Internet naming and numbering systems.

---

## What is a TLD?

Top Level Domain.

Examples:

```text
.com
.in
.org
.net
```

---

## What is a Domain Registrar?

A company authorized to sell and manage domain names.

Examples:

```text
GoDaddy
Hostinger
Namecheap
```

---

## What are Nameservers?

Servers responsible for managing DNS records for a domain.

---

## Why Change Nameservers?

To move DNS management from one provider to another.

Example:

```text
Hostinger → AWS Route53
```

---

## Do We Own a Domain Forever?

No.

Domains must be renewed periodically.

---

# Key Takeaways

- DNS stands for Domain Name System.
- DNS converts domain names into IP addresses.
- DNS makes the Internet user-friendly.
- ICANN governs Internet naming systems.
- TLD stands for Top Level Domain.
- Registrars sell and manage domain names.
- Domains are rented and require renewal.
- Nameservers determine who controls DNS records.
- DNS resolvers perform domain-to-IP lookups.
- Understanding DNS is essential for DevOps, Cloud, and Networking professionals.