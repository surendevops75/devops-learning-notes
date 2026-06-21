# Network Management and Architecture

## Introduction

Networking is the backbone of modern applications.

In Cloud environments such as AWS:

- AWS manages physical networking
- AWS manages power and data centers
- AWS manages hardware failures

Customers manage:

- EC2 Instances
- Security Groups
- VPC
- Routing
- Applications

---

# Network Management

Network management involves:

- Monitoring network connections
- Verifying service accessibility
- Checking open ports
- Troubleshooting connectivity issues
- Monitoring traffic

---

# Application Accessibility Check

Suppose Nginx is installed.

Check service status:

```bash
systemctl status nginx
```

Check process:

```bash
ps -ef | grep nginx
```

Access application:

```text
http://34.228.167.238:80
```

If application is inaccessible:

Check:

- Service status
- Process status
- Listening port
- Security Groups
- Firewall
- Application logs

---

# Network Diagnostic Commands

## netstat Command

Used to display:

- Network connections
- Routing tables
- Listening ports
- Interface statistics

### Syntax

```bash
netstat [options]
```

---

## Common netstat Options

### Show Listening TCP Ports

```bash
netstat -ltn
```

Where:

```text
-l = Listening
-t = TCP
-n = Numeric
```

---

### Show Listening Ports with Process Information

```bash
netstat -ltnp
```

Example:

```text
tcp   0   0 0.0.0.0:80   0.0.0.0:*   LISTEN   1234/nginx
```

Meaning:

```text
Nginx listening on Port 80
PID = 1234
```

---

### Show All Connections

```bash
netstat -an
```

---

### Routing Table

```bash
netstat -rn
```

---

# ss Command

Modern replacement for netstat.

Faster and more efficient.

### Syntax

```bash
ss [options]
```

---

## Common ss Commands

### Show Listening Ports

```bash
ss -ltn
```

---

### Show Listening Ports with Process Information

```bash
ss -ltnp
```

---

### Show TCP Connections

```bash
ss -t
```

---

### Show UDP Connections

```bash
ss -u
```

---

### Show All Connections

```bash
ss -a
```

---

### Show Statistics

```bash
ss -s
```

---

# Example

```bash
ss -ltnp
```

Output:

```text
State   Recv-Q Send-Q Local Address:Port
LISTEN  0      128    0.0.0.0:80
```

Meaning:

```text
Application listening on Port 80
```

---

# Service Verification Workflow

Suppose:

```text
Website Not Accessible
```

### Step 1

Check service.

```bash
systemctl status nginx
```

---

### Step 2

Check process.

```bash
ps -ef | grep nginx
```

---

### Step 3

Check listening port.

```bash
ss -ltnp | grep 80
```

or

```bash
netstat -ltnp | grep 80
```

---

### Step 4

Check Security Group.

```text
Port 80 Open?
```

---

### Step 5

Check Browser.

```text
http://IP:80
```

---

# Linux File System Hierarchy

Linux follows a hierarchical directory structure.

Starting point:

```text
/
```

Root directory.

---

# Important Linux Directories

## /

Root directory.

Top-level directory.

---

## /bin

Essential commands.

Examples:

```text
ls
cp
mv
cat
```

---

## /sbin

System administration commands.

Examples:

```text
fdisk
reboot
shutdown
```

---

## /etc

Configuration files.

Examples:

```text
/etc/passwd
/etc/group
/etc/ssh/sshd_config
```

---

## /home

User home directories.

Example:

```text
/home/ec2-user
/home/suresh
```

---

## /root

Root user's home directory.

```text
/root
```

---

## /var

Variable data.

Examples:

```text
Logs
Spool files
Cache
```

Common:

```text
/var/log
```

---

## /tmp

Temporary files.

```text
/tmp
```

---

## /opt

Optional applications.

Examples:

```text
Jenkins
Custom Software
```

---

## /usr

User applications and libraries.

---

# Linux Links

Linux supports:

- Hard Links
- Soft Links (Symbolic Links)

---

# Inode

Every file in Linux has an inode.

Inode stores:

- File permissions
- Owner
- Group
- File size
- Timestamps

It does NOT store:

```text
File Name
```

---

# View Inode Number

```bash
ls -i
```

Example:

```text
12345 notes.txt
```

Where:

```text
12345 = inode number
```

---

# Hard Link

Creates another reference to same inode.

### Create Hard Link

```bash
ln source.txt hardlink.txt
```

---

## Verify Inode

```bash
ls -li
```

Example:

```text
12345 source.txt
12345 hardlink.txt
```

Same inode.

---

# Hard Link Characteristics

✓ Same inode

✓ Same data

✓ Changes reflect in both files

✓ Works only within same filesystem

✗ Cannot link directories

---

# Soft Link (Symbolic Link)

Pointer to original file.

### Create Soft Link

```bash
ln -s source.txt softlink.txt
```

---

## Verify

```bash
ls -l
```

Output:

```text
softlink.txt -> source.txt
```

---

# Soft Link Characteristics

✓ Different inode

✓ Can cross filesystems

✓ Can link directories

✓ Similar to Windows Shortcut

---

## Broken Soft Link

If original file is deleted:

```text
Soft Link Becomes Invalid
```

---

# Hard Link vs Soft Link

| Hard Link | Soft Link |
|------------|------------|
| Same inode | Different inode |
| Cannot link directories | Can link directories |
| Cannot cross filesystem | Can cross filesystem |
| Survives original deletion | Breaks if original deleted |

---

# Proxy Servers

A Proxy acts as an intermediary between client and server.

---

# Forward Proxy

Client knows proxy.

Server does not know actual client.

Architecture:

```text
Client
   │
   ▼
Forward Proxy
   │
   ▼
Internet
```

Examples:

- Corporate Proxy
- Content Filtering
- Internet Access Control

---

# Forward Proxy Use Cases

- Hide client identity
- Restrict websites
- Content filtering
- Security controls

---

# Reverse Proxy

Client does not know actual backend server.

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

Examples:

- Nginx
- HAProxy
- Traefik

---

# Reverse Proxy Benefits

- Load Balancing
- SSL Termination
- Security
- Caching
- High Availability

---

# Example: Nginx Reverse Proxy

```text
Internet Users
        │
        ▼
      Nginx
        │
 ┌──────┴──────┐
 ▼             ▼
App1         App2
```

---

# Load Balancing Introduction

A Reverse Proxy can also distribute traffic.

Example:

```text
1000 Requests
```

Distributed as:

```text
App1 = 500
App2 = 500
```

Benefits:

- Better performance
- High availability
- Scalability

---

# Common Networking Commands

## ping

Check connectivity.

```bash
ping google.com
```

---

## nslookup

DNS lookup.

```bash
nslookup google.com
```

---

## dig

Advanced DNS lookup.

```bash
dig google.com
```

---

## traceroute

Trace network path.

```bash
traceroute google.com
```

---

## curl

Test HTTP endpoint.

```bash
curl http://server-ip
```

---

## wget

Download files.

```bash
wget https://example.com/file.txt
```

---

# Production Troubleshooting

## Website Not Loading

Check:

```bash
systemctl status nginx
```

---

```bash
ps -ef | grep nginx
```

---

```bash
ss -ltnp | grep 80
```

---

```bash
curl http://localhost
```

---

Check:

- Security Groups
- Firewall Rules
- DNS Configuration
- Application Logs

---

# Frequently Asked Interview Questions

## What is an inode?

Metadata structure that stores file information.

---

## Difference Between Hard Link and Soft Link?

| Hard Link | Soft Link |
|------------|------------|
| Same inode | Different inode |
| Survives deletion | Breaks on deletion |

---

## What is netstat?

Tool used to display network connections and listening ports.

---

## What is ss?

Modern replacement for netstat.

---

## What is a Reverse Proxy?

A server that sits between clients and backend servers.

Examples:

- Nginx
- HAProxy

---

## What is a Forward Proxy?

A server that represents clients to the internet.

---

## Why Use Reverse Proxy?

- Security
- Load Balancing
- SSL Termination
- High Availability

---

# Key Takeaways

- AWS manages physical networking infrastructure.
- netstat and ss are used to inspect network connections.
- Linux follows a hierarchical filesystem.
- Every file has an inode.
- Hard links share the same inode.
- Soft links point to another file.
- Forward proxy protects clients.
- Reverse proxy protects servers.
- Nginx is commonly used as a reverse proxy.
- Reverse proxies often perform load balancing.
- Networking troubleshooting is a critical DevOps skill.