# Sudo, Ports and Networking Basics

## Introduction

In Linux, normal users have limited privileges.

Administrative tasks such as:

- Installing software
- Creating users
- Modifying system files
- Restarting services

require elevated privileges.

Linux provides **sudo** to perform administrative operations securely.

---

# What is sudo?

**sudo** stands for:

```text
Super User DO
```

Allows a normal user to execute commands with root privileges.

Example:

```bash
sudo yum update
```

or

```bash
sudo dnf update
```

---

# Why sudo?

Instead of sharing the root password with everyone:

```text
User ---> sudo ---> Root Privileges
```

Benefits:

- Better security
- User accountability
- Audit trail
- Controlled access

---

# sudo Command

## Syntax

```bash
sudo <command>
```

Example:

```bash
sudo dnf install nginx
```

---

# Common sudo Examples

## Install Package

```bash
sudo dnf install nginx
```

---

## Edit Protected File

```bash
sudo vim /etc/hosts
```

---

## Restart Service

```bash
sudo systemctl restart nginx
```

---

## Become Root

```bash
sudo su
```

Remain in current directory.

---

## Become Root and Move to Root Home

```bash
sudo su -
```

Moves to:

```text
/root
```

---

# sudoers Configuration

Linux stores sudo permissions in:

```text
/etc/sudoers
```

This file controls:

- Who can use sudo
- What commands can be executed
- Whether passwords are required

---

# Why Not Edit sudoers Directly?

Avoid:

```bash
vim /etc/sudoers
```

A syntax error can break sudo access.

---

# visudo Command

Use:

```bash
visudo
```

### Benefits

- Syntax validation
- Prevents multiple edits
- Safe editing

---

# wheel Group

Most RHEL-based distributions use:

```text
wheel
```

group for sudo access.

Members of this group can execute administrative commands.

---

# Give Sudo Access to User

Suppose user:

```text
sivakumar
```

---

## Add User to wheel Group

```bash
usermod -aG wheel sivakumar
```

Verify:

```bash
id sivakumar
```

Output:

```text
groups=sivakumar,wheel
```

---

# Passwordless Sudo

Open:

```bash
visudo
```

Look for:

```text
%wheel ALL=(ALL) ALL
```

Normal behavior:

```text
Password Required
```

---

## Passwordless Sudo

Change to:

```text
%wheel ALL=(ALL) NOPASSWD: ALL
```

Now wheel group users can run:

```bash
sudo command
```

without entering password.

---

# Verify Sudo Access

```bash
sudo whoami
```

Output:

```text
root
```

---

# Networking Basics

Networking allows computers to communicate with each other.

Examples:

- Mobile to Server
- Browser to Website
- Laptop to Database
- API to API

---

# What is a Port?

A port is a logical communication endpoint.

Think of:

```text
Computer = Apartment Building

Port = Apartment Number
```

---

# Real World Example

Address:

```text
Gandhi Apartments
Ameerpet
Hyderabad
500082
Flat No: 102
```

Here:

```text
Building = Computer
Flat Number = Port
```

Courier reaches the correct apartment using flat number.

Similarly:

```text
IP Address = Computer
Port = Application
```

---

# Total Number of Ports

Ports range from:

```text
0 - 65535
```

Total:

```text
65,536 Ports
```

---

# Port Categories

## Well Known Ports

```text
0 - 1023
```

Examples:

| Port | Service |
|--------|----------|
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |

---

## Registered Ports

```text
1024 - 49151
```

Examples:

| Port | Service |
|--------|----------|
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 8080 | Tomcat |
| 6379 | Redis |

---

## Dynamic Ports

```text
49152 - 65535
```

Used temporarily by applications.

---

# Important Ports for DevOps

| Port | Service |
|--------|----------|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 8080 | Tomcat |
| 9090 | Prometheus |
| 3000 | Grafana |
| 5601 | Kibana |
| 9200 | Elasticsearch |
| 6443 | Kubernetes API Server |

---

# What is an IP Address?

IP Address uniquely identifies a device on a network.

Example:

```text
3.87.244.153
```

Without IP:

```text
No communication possible
```

---

# What is a Protocol?

A protocol is a set of rules for communication.

Examples:

```text
HTTP
HTTPS
SSH
FTP
SMTP
DNS
```

---

# HTTP Protocol

HTTP stands for:

```text
HyperText Transfer Protocol
```

Default Port:

```text
80
```

Example:

```text
http://3.87.244.153
```

Characteristics:

- Not encrypted
- Data visible in transit
- Faster but insecure

---

# HTTPS Protocol

HTTPS stands for:

```text
HyperText Transfer Protocol Secure
```

Default Port:

```text
443
```

Example:

```text
https://facebook.com
```

Characteristics:

- Encrypted
- Secure
- SSL/TLS enabled
- Industry standard

---

# Website Access Flow

Example:

```text
https://facebook.com
```

Step 1:

Browser requests:

```text
facebook.com
```

Step 2:

DNS converts:

```text
facebook.com
```

to:

```text
IP Address
```

Step 3:

Browser connects using:

```text
HTTPS
Port 443
```

Step 4:

Server validates request.

Step 5:

User enters:

```text
Username
Password
```

Step 6:

Server returns:

```text
Home Feed
```

---

# Client Server Architecture

## Client

Requests services.

Examples:

- Browser
- Mobile App
- Postman
- Curl

---

## Server

Provides services.

Examples:

- Facebook Server
- Web Server
- Database Server
- API Server

---

# Communication Flow

```text
Client
   │
   │ Request
   ▼
Server
   │
   │ Response
   ▼
Client
```

---

# Useful Networking Commands

## Check Connectivity

```bash
ping google.com
```

---

## DNS Lookup

```bash
nslookup google.com
```

---

## Show Listening Ports

```bash
ss -tulpn
```

or

```bash
netstat -tulpn
```

---

## Test HTTP Endpoint

```bash
curl http://server-ip
```

---

## Check Open Port

```bash
telnet server-ip 80
```

---

# Security Best Practices

## Avoid Direct Root Login

Prefer:

```bash
sudo
```

instead of:

```text
root login
```

---

## Use HTTPS

Prefer:

```text
HTTPS (443)
```

instead of:

```text
HTTP (80)
```

---

## Restrict SSH Access

Instead of:

```text
0.0.0.0/0
```

Use:

```text
Specific IP Range
```

whenever possible.

---

# Frequently Asked Interview Questions

## What is sudo?

Allows a normal user to execute commands as root.

---

## What is visudo?

Safe editor for `/etc/sudoers`.

Includes syntax validation.

---

## What is wheel Group?

Special Linux group used to grant sudo privileges.

---

## Difference Between sudo su and sudo su -

| Command | Result |
|----------|----------|
| sudo su | Become root, stay in current directory |
| sudo su - | Become root, move to /root |

---

## How Many Ports Exist?

```text
0 - 65535
```

Total:

```text
65,536
```

---

## Difference Between HTTP and HTTPS

| HTTP | HTTPS |
|--------|--------|
| Port 80 | Port 443 |
| Not Encrypted | Encrypted |
| Less Secure | More Secure |

---

## What is an IP Address?

Unique address assigned to a device on a network.

---

## What is a Protocol?

A set of rules used for communication between devices.

---

## What is Client Server Architecture?

A model where:

- Client requests services
- Server provides services

---

# Key Takeaways

- sudo provides controlled root access.
- sudo permissions are stored in `/etc/sudoers`.
- Use `visudo` instead of directly editing sudoers.
- wheel group is commonly used for sudo access.
- Ports identify applications running on a computer.
- Port range is 0–65535.
- HTTP uses Port 80.
- HTTPS uses Port 443.
- IP addresses identify devices.
- Protocols define communication rules.
- Most modern web applications use HTTPS.
- Client-server architecture is the foundation of modern applications.