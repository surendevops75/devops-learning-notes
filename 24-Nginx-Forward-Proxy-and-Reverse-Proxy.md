# Nginx Forward Proxy and Reverse Proxy

## Introduction

A Proxy acts on behalf of another system.

The word:

```text
Proxy = On Behalf Of Someone
```

There are two major types of proxies:

1. Forward Proxy
2. Reverse Proxy

Understanding the difference is important for DevOps, Networking, Security, and Cloud Engineering.

---

# What is a Proxy?

A Proxy is an intermediary between two systems.

Without Proxy:

```text
Client
   │
   ▼
Server
```

With Proxy:

```text
Client
   │
   ▼
Proxy
   │
   ▼
Server
```

The proxy receives requests and forwards them to the destination.

---

# Nginx Overview

Nginx is one of the most widely used software in production environments.

Nginx can function as:

1. HTTP/Web Server
2. Reverse Proxy
3. Load Balancer
4. SSL Termination Server
5. API Gateway (Basic Use Cases)

---

# What is a Forward Proxy?

A Forward Proxy works on behalf of the client.

Architecture:

```text
Client
   │
   ▼
Forward Proxy
   │
   ▼
Internet
   │
   ▼
Server
```

The destination server sees the proxy instead of the actual client.

---

# Forward Proxy Example

Without Proxy:

```text
Mobile
   │
   ▼
facebook.com
```

Facebook knows:

```text
Client IP Address
```

---

With Forward Proxy:

```text
Mobile
   │
   ▼
Forward Proxy
   │
   ▼
facebook.com
```

Facebook sees:

```text
Forward Proxy IP
```

instead of the mobile's IP.

---

# VPN Example

VPNs are common examples of Forward Proxies.

Example:

```text
Mobile
   │
   ▼
VPN Server (US)
   │
   ▼
facebook.com
```

Facebook sees:

```text
US VPN IP
```

instead of:

```text
Original Client IP
```

---

# Forward Proxy Use Cases

## Hide Client Identity

```text
Client → Proxy → Internet
```

Destination server cannot directly identify the original client.

---

## Bypass Geographic Restrictions

Example:

```text
India
   │
   ▼
VPN (US)
   │
   ▼
Website
```

Website believes the request originated from the US.

---

## Internet Access Control

Organizations may allow internet access only through approved proxies.

Example:

```text
Employee
   │
   ▼
Corporate Proxy
   │
   ▼
Internet
```

---

## Content Filtering

Organizations can block websites.

Example:

```text
YouTube
Facebook
Instagram
```

through a Forward Proxy.

---

## Logging and Monitoring

Companies monitor:

- Visited Websites
- Download Activity
- Internet Usage

using Forward Proxies.

---

# Forward Proxy Architecture

```text
Users
   │
   ▼
Forward Proxy
   │
   ▼
Internet
   │
   ▼
Destination Servers
```

---

# What is a Reverse Proxy?

A Reverse Proxy works on behalf of servers.

Architecture:

```text
Client
   │
   ▼
Reverse Proxy
   │
   ▼
Backend Servers
```

Clients communicate only with the Reverse Proxy.

Backend servers remain hidden.

---

# Reverse Proxy Example

Without Reverse Proxy:

```text
User
   │
   ▼
Backend Server
```

Backend server is directly exposed.

---

With Reverse Proxy:

```text
User
   │
   ▼
Nginx
   │
   ▼
Backend Server
```

User interacts only with Nginx.

---

# Why Reverse Proxy?

Reverse Proxy provides:

- Security
- Load Balancing
- SSL Offloading
- Request Routing
- Caching
- High Availability

---

# Hide Server Identity

Without Reverse Proxy:

```text
User
   │
   ▼
Backend Server Public IP
```

Backend becomes visible.

---

With Reverse Proxy:

```text
User
   │
   ▼
Nginx
   │
   ▼
Backend Server Private IP
```

Backend remains hidden.

---

# Reverse Proxy in Web Applications

Typical Architecture:

```text
User
   │
   ▼
Internet
   │
   ▼
Nginx
   │
   ▼
Backend Application
   │
   ▼
Database
```

---

# Reverse Proxy Routing Example

Frontend Request:

```text
http://daws86s.fun/api/transactions
```

Flow:

```text
Browser
   │
   ▼
Nginx
   │
   ▼
Backend:8080
```

Example Configuration:

```nginx
location /api/ {
    proxy_pass http://localhost:8080/;
}
```

---

# Reverse Proxy and Security

Benefits:

- Backend IP Hidden
- DDoS Protection
- SSL Termination
- Request Validation
- Access Control

---

# Load Balancing with Reverse Proxy

One Nginx server can distribute traffic across multiple backend servers.

Example:

```text
Users
   │
   ▼
Nginx
   │
   ├── Backend-1
   ├── Backend-2
   └── Backend-3
```

---

# SSL Termination

Without SSL Termination:

```text
User
   │ HTTPS
   ▼
Backend
```

Every backend server handles encryption.

---

With SSL Termination:

```text
User
   │ HTTPS
   ▼
Nginx
   │ HTTP
   ▼
Backend
```

Nginx handles encryption.

Backend handles application logic.

---

# Forward Proxy vs Reverse Proxy

| Forward Proxy | Reverse Proxy |
|--------------|---------------|
| Works for Client | Works for Server |
| Hides Client Identity | Hides Server Identity |
| Client Aware | Server Aware |
| Used by Users | Used by Organizations |
| VPN Example | Nginx Example |

---

# Real World Examples

## Forward Proxy

- VPN
- Corporate Proxy
- Content Filtering Systems

---

## Reverse Proxy

- Nginx
- HAProxy
- Traefik
- AWS Application Load Balancer

---

# Production Architecture

```text
Internet
   │
   ▼
Load Balancer
   │
   ▼
Nginx Reverse Proxy
   │
   ▼
Backend Servers
   │
   ▼
Database
```

---

# Nginx Commands

## Start

```bash
systemctl start nginx
```

---

## Stop

```bash
systemctl stop nginx
```

---

## Restart

```bash
systemctl restart nginx
```

---

## Reload

```bash
systemctl reload nginx
```

---

## Status

```bash
systemctl status nginx
```

---

## Verify Configuration

```bash
nginx -t
```

---

# Common Troubleshooting

## Reverse Proxy Not Working

Verify:

```bash
nginx -t
```

---

Check:

```bash
systemctl status nginx
```

---

Verify backend:

```bash
systemctl status backend
```

---

Verify backend port:

```bash
ss -lntp | grep 8080
```

---

Verify connectivity:

```bash
telnet backend-ip 8080
```

---

Check logs:

```bash
tail -f /var/log/nginx/error.log
```

---

# Frequently Asked Interview Questions

## What is a Proxy?

A system that acts on behalf of another system.

---

## What is a Forward Proxy?

A proxy that works on behalf of clients.

---

## What is a Reverse Proxy?

A proxy that works on behalf of servers.

---

## What is the Most Common Example of a Forward Proxy?

```text
VPN
```

---

## What is the Most Common Example of a Reverse Proxy?

```text
Nginx
```

---

## Why Use a Reverse Proxy?

- Security
- Load Balancing
- SSL Termination
- High Availability
- Backend Isolation

---

## Why Use a Forward Proxy?

- Hide Client Identity
- Bypass Restrictions
- Content Filtering
- Monitoring

---

## Which Proxy Hides the Client?

```text
Forward Proxy
```

---

## Which Proxy Hides the Server?

```text
Reverse Proxy
```

---

# Key Takeaways

- A proxy acts on behalf of another system.
- Forward proxies represent clients.
- Reverse proxies represent servers.
- VPNs are common examples of Forward Proxies.
- Nginx is a common example of a Reverse Proxy.
- Reverse proxies improve security by hiding backend servers.
- Reverse proxies can perform load balancing and SSL termination.
- Forward proxies are commonly used for filtering, monitoring, and bypassing restrictions.
- Understanding proxy architectures is essential for DevOps, Networking, and Cloud Engineering.