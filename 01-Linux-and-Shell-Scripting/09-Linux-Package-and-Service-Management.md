# Linux Package and Service Management

## Introduction

Linux servers are usually installed with a minimal operating system.

Additional software such as:

- Nginx
- Apache
- MySQL
- Docker
- Git
- Jenkins

must be installed using package managers.

Package managers help:

- Install software
- Remove software
- Update software
- Manage dependencies
- Manage repositories

---

# What is a Package?

A package is a bundle containing:

- Application files
- Libraries
- Configuration files
- Documentation

Examples:

```text
nginx
git
docker
mysql
java
```

---

# Package Managers

A Package Manager is a tool used to manage software packages.

---

# Package Managers by Distribution

## RHEL / CentOS / Rocky Linux / Amazon Linux

```bash
dnf
```

Previously:

```bash
yum
```

---

## Ubuntu / Debian

```bash
apt
```

or

```bash
apt-get
```

---

# Why DNF Replaced YUM?

DNF (Dandified YUM) is the newer version.

Benefits:

- Faster dependency resolution
- Better performance
- Improved package handling
- Better memory management

---

# DNF Command

## Install Package

```bash
dnf install nginx
```

Install Nginx web server.

---

## Install Multiple Packages

```bash
dnf install nginx git wget
```

---

## Remove Package

```bash
dnf remove nginx
```

---

## Update Package

```bash
dnf update nginx
```

---

## Update Entire System

```bash
dnf update
```

---

## Package Information

```bash
dnf info nginx
```

---

## Search Package

```bash
dnf search nginx
```

---

## List Installed Packages

```bash
dnf list installed
```

Shows packages already installed.

---

## List Available Packages

```bash
dnf list available
```

Shows packages available from repositories.

---

## Reinstall Package

```bash
dnf reinstall nginx
```

---

## Check Installed Package

```bash
dnf list installed nginx
```

---

## Show Package Dependencies

```bash
dnf repoquery --requires nginx
```

---

# YUM Commands

Older systems may still use YUM.

Examples:

```bash
yum install nginx
yum remove nginx
yum update
yum list installed
```

Most commands are similar to DNF.

---

# APT Package Manager

Ubuntu and Debian use APT.

## Update Package Index

```bash
apt update
```

---

## Install Package

```bash
apt install nginx
```

---

## Remove Package

```bash
apt remove nginx
```

---

## Upgrade Packages

```bash
apt upgrade
```

---

## Search Package

```bash
apt search nginx
```

---

# RPM Package Manager

RPM stands for:

```text
Red Hat Package Manager
```

RPM files usually end with:

```text
.rpm
```

Example:

```text
nginx-1.24.rpm
```

---

# RPM Commands

## Install RPM

```bash
rpm -ivh package.rpm
```

### Options

| Option | Meaning |
|----------|----------|
| i | Install |
| v | Verbose |
| h | Progress Bar |

---

## Remove RPM

```bash
rpm -e package-name
```

---

## Query Installed Package

```bash
rpm -q nginx
```

---

## List All Packages

```bash
rpm -qa
```

---

# Package Repositories

Repositories are storage locations that contain software packages.

Linux downloads packages from repositories.

---

# Repository Location

RHEL-based systems:

```text
/etc/yum.repos.d/
```

Contains:

```text
*.repo
```

files.

Example:

```text
base.repo
epel.repo
custom.repo
```

---

# View Repository Files

```bash
ls /etc/yum.repos.d/
```

---

# Display Repository Configuration

```bash
cat /etc/yum.repos.d/*.repo
```

---

# Repository Example

```ini
[nginx]
name=Nginx Repository
baseurl=http://repo.example.com
enabled=1
gpgcheck=1
```

---

# What Happens During Installation?

Example:

```bash
dnf install nginx
```

Workflow:

```text
Server
   │
   ▼
Repository
   │
   ▼
Download Package
   │
   ▼
Install Dependencies
   │
   ▼
Install Application
```

---

# Nginx Web Server

Nginx is a popular HTTP web server.

Uses:

- Websites
- Reverse Proxy
- Load Balancer
- API Gateway

Default Port:

```text
80
```

---

# Install Nginx

```bash
dnf install nginx -y
```

### Option

```text
-y
```

Automatically answer yes.

---

# Verify Installation

```bash
rpm -q nginx
```

or

```bash
dnf list installed nginx
```

---

# Service Management

Linux uses services (daemons) to run applications in the background.

Examples:

```text
nginx
sshd
docker
jenkins
mysql
```

---

# systemd

Modern Linux distributions use:

```text
systemd
```

to manage services.

---

# systemctl Command

Used to manage services.

---

# Start Service

```bash
systemctl start nginx
```

Starts service immediately.

---

# Stop Service

```bash
systemctl stop nginx
```

Stops service.

---

# Restart Service

```bash
systemctl restart nginx
```

Stops and starts service.

Used after configuration changes.

---

# Reload Service

```bash
systemctl reload nginx
```

Reload configuration without full restart.

---

# Service Status

```bash
systemctl status nginx
```

Example Output:

```text
active (running)
```

---

# Enable Service

```bash
systemctl enable nginx
```

Meaning:

```text
Start automatically during boot
```

---

# Disable Service

```bash
systemctl disable nginx
```

Meaning:

```text
Do not start automatically during boot
```

---

# Check If Service Is Enabled

```bash
systemctl is-enabled nginx
```

Output:

```text
enabled
```

or

```text
disabled
```

---

# Check If Service Is Active

```bash
systemctl is-active nginx
```

Output:

```text
active
```

or

```text
inactive
```

---

# Reload systemd Configuration

```bash
systemctl daemon-reload
```

Used after:

- Creating service files
- Modifying service files

---

# List Running Services

```bash
systemctl list-units --type=service
```

---

# List Failed Services

```bash
systemctl --failed
```

---

# Nginx Service Workflow

Install:

```bash
dnf install nginx -y
```

Start:

```bash
systemctl start nginx
```

Verify:

```bash
systemctl status nginx
```

Enable:

```bash
systemctl enable nginx
```

Access:

```text
http://3.87.244.153
```

Port:

```text
80
```

---

# Service Logs

## journalctl

Used to view systemd logs.

---

## View All Logs

```bash
journalctl
```

---

## View Nginx Logs

```bash
journalctl -u nginx
```

---

## Recent Logs

```bash
journalctl -n 50
```

---

## Live Logs

```bash
journalctl -f
```

Similar to:

```bash
tail -f
```

---

# Verify Listening Ports

## ss Command

```bash
ss -tulpn
```

Shows:

- Open Ports
- Listening Services
- Process IDs

---

## Example

```text
tcp LISTEN 0 128 *:80
```

Means:

```text
Nginx listening on Port 80
```

---

# Common Service Troubleshooting

## Service Not Starting

Check:

```bash
systemctl status nginx
```

---

## View Logs

```bash
journalctl -u nginx
```

---

## Verify Configuration

Example:

```bash
nginx -t
```

Output:

```text
syntax is ok
```

---

## Verify Port Usage

```bash
ss -tulpn | grep 80
```

---

## Verify Firewall

```bash
firewall-cmd --list-all
```

---

# Common Production Services

| Service | Purpose |
|----------|----------|
| nginx | Web Server |
| httpd | Apache Web Server |
| sshd | SSH Service |
| docker | Container Runtime |
| mysqld | MySQL Database |
| jenkins | CI/CD Server |
| crond | Scheduler |
| rsyslog | Logging |

---

# Frequently Asked Interview Questions

## What is a Package Manager?

A tool used to install, update, remove and manage software.

Examples:

```text
dnf
yum
apt
```

---

## Difference Between YUM and DNF

| YUM | DNF |
|------|------|
| Older | Newer |
| Slower | Faster |
| Legacy Systems | Modern Systems |

---

## What is RPM?

Red Hat Package Manager.

Used to install and manage `.rpm` packages.

---

## Difference Between RPM and DNF

| RPM | DNF |
|------|------|
| Installs package directly | Handles dependencies |
| Manual dependency handling | Automatic dependency handling |

---

## Difference Between Start and Enable

### Start

```bash
systemctl start nginx
```

Runs service now.

---

### Enable

```bash
systemctl enable nginx
```

Starts service during boot.

---

## Difference Between Restart and Reload

### Restart

```bash
systemctl restart nginx
```

Stops and starts service.

---

### Reload

```bash
systemctl reload nginx
```

Reloads configuration without full restart.

---

## Where Are Repository Files Stored?

```text
/etc/yum.repos.d/
```

---

## How Do You Check Service Logs?

```bash
journalctl -u nginx
```

---

# Key Takeaways

- DNF is the modern package manager for RHEL-based systems.
- APT is used in Ubuntu and Debian.
- RPM manages `.rpm` packages.
- Repositories provide software packages.
- Repository configuration is stored in `/etc/yum.repos.d/`.
- Nginx is a popular web server running on Port 80.
- systemd manages services.
- systemctl is used for service operations.
- journalctl is used for service and system logs.
- Always verify service status after installation.
- Enable important services to start automatically after reboot.