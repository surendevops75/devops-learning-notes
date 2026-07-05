# VPN, Bastion and Production Server Access

## Introduction

In cloud environments, direct access to production servers is considered a security risk.

Organizations implement multiple layers of security to ensure:

```text
Controlled Access

Secure Communication

Auditing

Compliance
```

A common production access model is:

```text
VPN
   ↓
Bastion Host
   ↓
Private Servers
```

---

# Why Not Expose Servers Directly?

Suppose a private server has:

```text
Public IP

Port 22 Open
```

Anyone on the internet can attempt:

```text
SSH Login

Password Attacks

Port Scanning

Brute Force Attacks
```

---

# Production Security Principle

```text
Private Servers Should Not Be Directly Accessible
```

---

# Environment Types

## DEV Environment

Development environments are less critical.

Access is usually simpler.

Example:

```text
VPN
   ↓
Server
```

or sometimes:

```text
Public SSH Access
```

depending on organizational policies.

---

# NON-PROD Environment

Examples:

```text
DEV

QA

UAT

Testing
```

Common Access:

```text
VPN
   ↓
Direct Server Access
```

---

# PROD Environment

Production environments require stricter controls.

Recommended Flow:

```text
VPN
   ↓
Bastion Host
   ↓
Private Servers
```

---

# What is a VPN?

VPN stands for:

```text
Virtual Private Network
```

---

# Purpose

Create a secure tunnel between:

```text
Engineer Laptop

Cloud Network
```

---

# VPN Architecture

```text
Laptop
   ↓
Encrypted Tunnel
   ↓
VPN Server
   ↓
AWS VPC
```

---

# Benefits

```text
Encryption

Authentication

Secure Access

Private Connectivity
```

---

# OpenVPN

A popular VPN solution is:

```text
OpenVPN
```

---

# OpenVPN Access Server

Example Service:

```text
OpenVPN Access Server
```

---

# Common Ports

## SSH

```text
22
```

Used for:

```text
Secure Shell Access
```

---

## HTTPS

```text
443
```

Used for:

```text
Secure Web Access
```

---

## OpenVPN Admin UI

```text
943
```

Used for:

```text
Administrative Portal
```

---

## OpenVPN Tunnel

```text
1194
```

Used for:

```text
VPN Traffic
```

---

# Example URLs

Admin Portal:

```text
https://44.203.237.102:943/admin
```

---

Client Portal:

```text
https://44.203.237.102:943/
```

---

Login Example:

```text
Username: openvpn
```

---

# OpenVPN Components

```text
VPN Server

Admin Portal

Client Portal

VPN Clients
```

---

# Access Flow

```text
Engineer Laptop
      ↓
OpenVPN Client
      ↓
VPN Tunnel
      ↓
AWS Network
```

---

# What is a Bastion Host?

A Bastion Host is:

```text
Jump Server

Gateway Server
```

used to access private infrastructure.

---

# Bastion Location

Usually deployed in:

```text
Public Subnet
```

---

# Why Public Subnet?

Engineers must first reach:

```text
Bastion
```

before accessing private servers.

---

# Architecture

```text
Internet
    ↓
VPN
    ↓
Bastion
    ↓
Private EC2
```

---

# Bastion Responsibilities

```text
SSH Gateway

Controlled Access

Auditing

Centralized Administration
```

---

# Example

Private Server:

```text
MongoDB
```

---

Private Server:

```text
Catalogue
```

---

Private Server:

```text
Cart
```

---

These servers have:

```text
No Public IP
```

---

# Access Flow

```text
Laptop
   ↓
VPN
   ↓
Bastion
   ↓
MongoDB
```

---

# Why Use Bastion?

Without Bastion:

```text
Every Server Needs Public Access
```

---

Problems:

```text
Security Risks

More Attack Surface

Difficult Auditing
```

---

# With Bastion

Only:

```text
Bastion
```

is exposed.

---

Benefits:

```text
Improved Security

Centralized Access

Reduced Exposure
```

---

# Security Group Example

Private Server Security Group:

```text
Allow SSH
```

from:

```text
Bastion Security Group
```

---

Instead of:

```text
0.0.0.0/0
```

---

# Best Practice

Use:

```text
Security Group Reference
```

instead of public IP ranges.

---

# Production Architecture

```text
Engineer Laptop
        ↓
VPN Connection
        ↓
OpenVPN Server
        ↓
Bastion Host
        ↓
Private EC2
        ↓
Applications
```

---

# Real Production Example

Developer needs access to:

```text
Catalogue Server
```

---

Flow:

```text
Connect VPN
       ↓
SSH Bastion
       ↓
SSH Catalogue
```

---

# Why VPN + Bastion?

Provides:

```text
Two Layers Of Security
```

---

Layer 1

```text
VPN Authentication
```

---

Layer 2

```text
Bastion Access Control
```

---

# Auditing Benefits

All access passes through:

```text
Bastion
```

---

Organizations can track:

```text
Who Connected

When Connected

Which Server Accessed
```

---

# Production Troubleshooting

Problem:

```text
Cannot SSH Into Server
```

---

Step 1

Verify:

```text
VPN Connected
```

---

Step 2

Verify:

```text
Can Reach Bastion
```

---

Step 3

Verify:

```text
Bastion Security Group
```

---

Step 4

Verify:

```text
Private Server Security Group
```

---

Step 5

Verify:

```text
SSH Service Running
```

---

# Common Interview Questions

## What is a VPN?

### Short Answer

A VPN creates an encrypted connection between a user and a private network.

### Detailed Explanation

VPNs allow secure remote access to internal infrastructure without exposing resources directly to the internet.

### Production Example

Engineers connecting to AWS infrastructure through OpenVPN.

### Follow-Up Questions

- Why is VPN required?
- What ports does OpenVPN use?

---

## What is a Bastion Host?

### Short Answer

A Bastion Host is a jump server used to access private infrastructure.

### Detailed Explanation

It acts as a gateway between administrators and private servers inside a VPC.

### Production Example

Using Bastion to SSH into MongoDB and Catalogue servers.

### Follow-Up Questions

- Why is Bastion placed in a public subnet?
- Can Bastion access private instances?

---

## Why Use VPN and Bastion Together?

### Short Answer

To provide multiple layers of security.

### Detailed Explanation

VPN authenticates users before they enter the network, while Bastion controls access to internal servers.

### Production Example

Production access model using OpenVPN and Bastion for all SSH access.

### Follow-Up Questions

- What are the security benefits?
- Is Bastion required in every environment?

---

## Why Are Production Servers Kept Private?

### Short Answer

To reduce security risks and prevent direct internet access.

### Detailed Explanation

Private servers are protected behind VPNs, Bastion Hosts, Load Balancers, and Security Groups.

### Production Example

MongoDB, Redis, and application servers deployed in private subnets.

### Follow-Up Questions

- Can private servers access the internet?
- How do engineers manage private servers?

---

# Key Takeaways

```text
Production servers should not be directly accessible from the internet.

VPN creates a secure encrypted connection into the cloud network.

OpenVPN is a commonly used VPN solution.

Bastion Hosts act as jump servers for private infrastructure.

Production access typically follows VPN → Bastion → Private Server.

Only Bastion should be exposed publicly.

Private servers should allow SSH only from Bastion.

Security Group references are preferred over IP-based rules.

VPN and Bastion together provide strong security and auditing capabilities.

This architecture is widely used in production AWS environments.
```