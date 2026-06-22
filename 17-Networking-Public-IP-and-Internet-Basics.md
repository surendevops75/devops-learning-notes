# Networking, Public IP and Internet Basics

## Introduction

Modern applications communicate over networks.

Examples:

- Browser accessing a website
- Mobile application calling APIs
- Backend application connecting to a database
- Servers communicating with each other

Understanding networking fundamentals is essential for DevOps Engineers.

---

# What is an IP Address?

An IP Address (Internet Protocol Address) is a unique identifier assigned to a device connected to a network.

Example:

```text
49.204.160.153
```

Examples of devices having IP addresses:

- Laptop
- Mobile
- Server
- Router
- Printer

Without an IP address, devices cannot communicate over a network.

---

# How Internet Works?

The Internet is a network of networks.

When you open a website:

```text
https://amazon.in
```

The following happens:

```text
Browser
   │
   ▼
DNS Lookup
   │
   ▼
Public IP Address
   │
   ▼
Internet Routing
   │
   ▼
Load Balancer
   │
   ▼
Web Server
   │
   ▼
Application Server
   │
   ▼
Database
```

Response follows the reverse path back to the user.

---

# Example: Accessing Amazon

When a user opens:

```text
https://amazon.in
```

Browser performs:

### Step 1

Resolve domain name.

```text
amazon.in
```

to

```text
Public IP Address
```

using DNS.

---

### Step 2

Connect using:

```text
Protocol : HTTPS
Port     : 443
```

---

### Step 3

Request reaches Amazon infrastructure.

---

### Step 4

Application processes request.

---

### Step 5

Response returns to user.

---

# What is DNS?

DNS stands for:

```text
Domain Name System
```

DNS converts:

```text
amazon.in
```

into:

```text
IP Address
```

Humans remember:

```text
google.com
facebook.com
amazon.in
```

Computers communicate using:

```text
IP Addresses
```

---

# Public IP Address

A Public IP Address is reachable from the Internet.

Example:

```text
49.204.160.153
```

Characteristics:

- Globally unique
- Accessible over the Internet
- Assigned by ISP or Cloud Provider

Examples:

```text
AWS EC2 Public IP
Home Broadband Public IP
Office Public IP
```

---

# Private IP Address

A Private IP Address is used inside private networks.

Examples:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Example:

```text
172.31.30.95
```

Characteristics:

- Internal communication only
- Not accessible directly from the Internet
- Used inside VPCs and local networks

---

# Public IP vs Private IP

| Public IP | Private IP |
|------------|------------|
| Internet Accessible | Internal Network Only |
| Globally Unique | Reusable |
| Assigned by ISP/Cloud | Assigned Internally |
| Example: 49.204.160.153 | Example: 172.31.30.95 |

---

# How AWS Uses Public and Private IPs

Example:

```text
Internet User
      │
      ▼
Public IP
      │
      ▼
EC2 Instance
      │
      ▼
Private IP
```

Common Architecture:

```text
Frontend Server → Public IP

Backend Server → Private IP

Database Server → Private IP
```

This improves security.

---

# Wireless vs Wired Networks

Two common ways devices connect to a network:

## Wired Network

Examples:

- Ethernet Cable
- Fiber Cable

Advantages:

- Higher Speed
- Lower Latency
- More Stable
- Less Interference

---

## Wireless Network

Examples:

- WiFi
- Mobile Data

Advantages:

- Mobility
- Easy Connectivity

Disadvantages:

- Lower Speed
- Signal Interference
- Higher Latency

---

# Wired vs Wireless Comparison

| Wired | Wireless |
|---------|---------|
| Faster | Slower |
| Stable | Less Stable |
| Lower Latency | Higher Latency |
| Physical Cable Required | No Cable Required |

---

# Network Routing

Data travels through multiple networks before reaching destination.

Example:

```text
Chennai
   │
   ▼
Guntur
   │
   ▼
Vijayawada
   │
   ▼
Vizag
   │
   ▼
Kolkata
```

Data packets may travel through multiple routers and ISPs before reaching the final destination.

---

# What is a Protocol?

A Protocol is a set of rules used for communication.

Examples:

| Protocol | Port |
|-----------|------|
| HTTP | 80 |
| HTTPS | 443 |
| SSH | 22 |
| MySQL | 3306 |
| DNS | 53 |

---

# What is a Port?

A Port identifies a specific service running on a machine.

Example:

```text
IP Address = House

Port = Room Number
```

Examples:

```text
22    SSH
80    HTTP
443   HTTPS
3306  MySQL
8080  Backend Applications
```

---

# Connectivity Testing

## Ping

Verify network connectivity.

```bash
ping google.com
```

---

## DNS Lookup

```bash
nslookup google.com
```

---

## Advanced DNS Query

```bash
dig google.com
```

---

# Telnet

Used to verify whether a port is reachable.

Syntax:

```bash
telnet <IP> <PORT>
```

Example:

```bash
telnet 172.31.30.95 3306
```

Meaning:

```text
Can Backend Server Reach MySQL Server?
```

---

# Common Use Cases of Telnet

## Verify Database Connectivity

```bash
telnet 172.31.30.95 3306
```

---

## Verify Backend Connectivity

```bash
telnet backend-server-ip 8080
```

---

## Verify Web Server Connectivity

```bash
telnet webserver-ip 80
```

---

# Application Communication Example

Backend communicating with database:

```text
Backend Server
      │
      ▼
Database Server
      │
      ▼
Port 3306
```

Connectivity test:

```bash
telnet 172.31.30.95 3306
```

---

# Common Network Troubleshooting

## Website Not Accessible

Check:

```bash
ping <server>
```

---

Check:

```bash
nslookup domain.com
```

---

Check:

```bash
telnet <ip> <port>
```

---

Check:

```bash
curl http://server-ip
```

---

Check:

```bash
ss -lntp
```

---

# Real Production Architecture

```text
Internet
    │
    ▼
Load Balancer
    │
    ▼
Web Server
    │
    ▼
Backend Server
    │
    ▼
Database Server
```

Communication:

```text
User → Web → Backend → Database
```

---

# Frequently Asked Interview Questions

## What is an IP Address?

A unique identifier assigned to a device on a network.

---

## Difference Between Public and Private IP?

| Public IP | Private IP |
|------------|------------|
| Internet Accessible | Internal Use |
| Globally Unique | Reusable |
| Public Network | Private Network |

---

## What is DNS?

Domain Name System.

Converts domain names into IP addresses.

---

## What is a Port?

A logical endpoint used by applications for communication.

---

## What is Telnet Used For?

Used to verify network connectivity and port accessibility.

Example:

```bash
telnet 172.31.30.95 3306
```

---

## Why Are Databases Usually Assigned Private IPs?

For security reasons.

Databases should not be directly exposed to the Internet.

---

## Why is Wired Faster Than Wireless?

Because wired communication has:

- Less interference
- Lower latency
- Higher throughput

---

# Key Takeaways

- Every device on a network requires an IP address.
- Public IPs are Internet accessible.
- Private IPs are used within internal networks.
- DNS converts domain names into IP addresses.
- Ports identify services running on a machine.
- Wired networks are generally faster and more stable than wireless networks.
- Telnet is useful for testing port connectivity.
- Backend servers commonly connect to databases using private IP addresses.
- Most production architectures isolate backend and database servers from direct Internet access.
- Understanding networking is essential for DevOps and Cloud Engineers.