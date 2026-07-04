# HTTPS, SSL and Certificates

## Introduction

When users access websites, sensitive information is transmitted over the internet.

Examples:

```text
Usernames

Passwords

Bank Details

Credit Card Information

Personal Data
```

If data travels as plain text:

```text
Anyone Can Read It
```

This creates a major security risk.

To solve this problem, we use:

```text
HTTPS
```

---

# What is HTTP?

HTTP stands for:

```text
HyperText Transfer Protocol
```

HTTP is used for communication between:

```text
Browser

Server
```

---

# Problem with HTTP

HTTP traffic is:

```text
Plain Text
```

Example:

```text
Username = surendra

Password = Password123
```

Anyone intercepting the traffic can read it.

---

# Solution

Use:

```text
HTTPS
```

---

# What is HTTPS?

HTTPS stands for:

```text
HyperText Transfer Protocol Secure
```

HTTPS is:

```text
HTTP + Encryption
```

---

# Default Ports

HTTP:

```text
80
```

HTTPS:

```text
443
```

---

# Examples

HTTP:

```text
http://daws86s.fun
```

---

HTTPS:

```text
https://daws86s.fun
```

---

HTTPS with Explicit Port:

```text
https://daws86s.fun:443
```

---

Custom Port Example:

```text
https://daws86s.fun:445
```

---

# What is SSL?

SSL stands for:

```text
Secure Socket Layer
```

SSL was originally developed to secure internet communication.

Today SSL has been replaced by:

```text
TLS
```

but people still commonly say:

```text
SSL Certificate
```

---

# Goal of SSL/TLS

Provide:

```text
Encryption

Authentication

Data Integrity
```

---

# Why Encryption?

Suppose you send:

```text
My Password Is Password123
```

Without encryption:

```text
Readable By Anyone
```

---

With encryption:

```text
Random Unreadable Text
```

Example:

```text
A7X9#K2@M8
```

Only the intended recipient can understand it.

---

# Public Key Cryptography

HTTPS relies on:

```text
Public Key

Private Key
```

---

# What is a Public Key?

Public Key is:

```text
Shared With Everyone
```

Example:

```text
Stored On Server

Included In Certificate
```

---

# What is a Private Key?

Private Key is:

```text
Secret

Never Shared
```

Only the server possesses it.

---

# Important Rule

```text
Public Key Encrypts

Private Key Decrypts
```

---

# Real World Analogy

Think of a mailbox.

Public Key:

```text
Mailbox Slot
```

Anyone can drop a letter.

---

Private Key:

```text
Mailbox Key
```

Only the owner can open it.

---

# Example

Suppose:

```text
Client
```

has:

```text
Server Public Key
```

---

Client sends:

```text
Hello Server
```

encrypted with:

```text
Public Key
```

---

Only:

```text
Private Key
```

can decrypt it.

---

# Domain Requirement

HTTPS requires:

```text
A Valid Domain
```

Example:

```text
daws86s.fun
```

---

Environment Examples

Development:

```text
roboshop-dev.daws86s.fun
```

---

Production:

```text
daws86s.fun
```

---

# Why Domain is Required?

Certificates are issued for:

```text
Domains
```

not IP addresses in most production scenarios.

---

# What is a Certificate?

A certificate contains:

```text
Domain Name

Public Key

Certificate Information

Digital Signature
```

---

# Example

Certificate for:

```text
daws86s.fun
```

contains:

```text
Public Key
```

associated with that domain.

---

# Problem

Anyone can create:

```text
Public Key

Private Key
```

pair.

How do users know the website is genuine?

---

# Certificate Authority (CA)

A trusted third-party organization called:

```text
Certificate Authority
```

validates ownership.

---

# Popular Certificate Authorities

Examples:

```text
DigiCert

GlobalSign

Sectigo

Let's Encrypt
```

---

# Certificate Issuance Process

Step 1

```text
Purchase Or Own Domain
```

---

Step 2

Generate:

```text
Public Key

Private Key
```

---

Step 3

Send:

```text
Public Key
```

to CA.

---

Step 4

CA validates:

```text
Domain Ownership
```

---

Step 5

CA signs the certificate.

---

Result:

```text
Trusted Certificate
```

---

# What Does Signing Mean?

CA uses its reputation to certify:

```text
This Public Key Belongs To This Domain
```

---

# Browser Trust

Browsers already trust:

```text
Popular Certificate Authorities
```

---

When a certificate is presented:

Browser verifies:

```text
CA Signature
```

---

If valid:

```text
Secure Connection
```

---

# HTTPS Connection Flow

```text
Browser
     ↓
Website
     ↓
Certificate Sent
     ↓
Certificate Verified
     ↓
Secure Session Established
```

---

# Simplified HTTPS Handshake

Step 1

Client connects:

```text
https://daws86s.fun
```

---

Step 2

Server sends:

```text
Certificate
```

containing:

```text
Public Key
```

---

Step 3

Browser validates:

```text
Certificate

CA Signature

Domain Name
```

---

Step 4

Browser generates:

```text
Random Session Key
```

---

Step 5

Session Key encrypted using:

```text
Server Public Key
```

and sent to server.

---

Step 6

Server decrypts using:

```text
Private Key
```

---

Step 7

Both sides now possess:

```text
Same Session Key
```

---

Step 8

All future communication uses:

```text
Session Key Encryption
```

---

# Why Use Session Keys?

Public Key encryption is:

```text
Slow
```

Session key encryption is:

```text
Fast
```

---

Therefore:

```text
Public Key

↓

Exchange Session Key

↓

Session Key Handles Communication
```

---

# Trainer Explanation

Server contains:

```text
Public Key

Private Key
```

---

Client downloads:

```text
Certificate
```

---

Client creates:

```text
Random Message
```

and encrypts it using:

```text
Public Key
```

---

Server decrypts it using:

```text
Private Key
```

---

If both sides match:

```text
Server Identity Confirmed
```

---

Future communication uses:

```text
Session Key
```

---

# Benefits of HTTPS

```text
Encryption

Authentication

Integrity

Trust

Security
```

---

# What Happens Without HTTPS?

Risks:

```text
Password Theft

Data Theft

Man-in-the-Middle Attacks

Session Hijacking
```

---

# Production Example

User accesses:

```text
https://daws86s.fun
```

---

Flow:

```text
Browser
     ↓
Certificate Validation
     ↓
Secure Session
     ↓
Encrypted Communication
```

---

# Common Interview Questions

## What is HTTPS?

### Short Answer

HTTPS is HTTP secured using SSL/TLS encryption.

### Detailed Explanation

HTTPS encrypts communication between clients and servers, protecting data from interception and tampering.

### Production Example

Users securely accessing roboshop-dev.daws86s.fun.

### Follow-Up Questions

- What port does HTTPS use?
- How is HTTPS different from HTTP?

---

## What is SSL?

### Short Answer

SSL is a security protocol used to encrypt internet communication.

### Detailed Explanation

Although modern systems use TLS, SSL remains a commonly used term for secure web communication.

### Production Example

SSL certificates installed on production load balancers.

### Follow-Up Questions

- What replaced SSL?
- Why do people still call them SSL certificates?

---

## Difference Between Public Key and Private Key?

### Short Answer

Public keys encrypt data, private keys decrypt data.

### Detailed Explanation

Public keys are shared openly, while private keys remain secret and are used to decrypt information.

### Production Example

Browser encrypts session information using the website's public key.

### Follow-Up Questions

- Can private keys be shared?
- What happens if a private key is compromised?

---

## Why Do We Need a Certificate Authority?

### Short Answer

Certificate Authorities validate domain ownership and establish trust.

### Detailed Explanation

Without a CA, users cannot verify whether a public key actually belongs to the intended website.

### Production Example

Let's Encrypt validating ownership of daws86s.fun before issuing a certificate.

### Follow-Up Questions

- What happens if a certificate is self-signed?
- How do browsers trust CAs?

---

## Explain HTTPS Handshake.

### Short Answer

The browser verifies the certificate, exchanges a session key securely, and uses it for encrypted communication.

### Detailed Explanation

Public key cryptography establishes trust and securely exchanges a session key, which is then used for fast symmetric encryption.

### Production Example

User opening https://daws86s.fun from a browser.

### Follow-Up Questions

- Why not use public key encryption for all communication?
- What is a session key?

---

# Key Takeaways

```text
HTTPS is HTTP secured using SSL/TLS.

HTTPS protects sensitive data.

SSL has largely been replaced by TLS.

HTTPS uses port 443.

Public keys are shared openly.

Private keys remain secret.

Certificates contain public keys and domain information.

Certificate Authorities validate domain ownership.

Browsers trust certificates signed by trusted CAs.

HTTPS uses a TLS handshake to establish secure communication.

Session keys are used for efficient encrypted communication.

HTTPS is mandatory for modern production applications.
```