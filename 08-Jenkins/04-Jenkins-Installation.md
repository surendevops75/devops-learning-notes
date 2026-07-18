# Jenkins Installation

# Introduction

Before Jenkins can automate CI/CD pipelines, it must be installed and configured properly.

Jenkins is a Java-based application, so Java must be installed before installing Jenkins.

In production environments, Jenkins is usually installed on:

- RHEL
- CentOS
- Rocky Linux
- Ubuntu
- Amazon Linux

Most organizations also provision Jenkins using Infrastructure as Code (Terraform/Ansible) instead of installing it manually.

---

# Jenkins Prerequisites

Before installing Jenkins, ensure the following requirements are met.

## Operating System

```text
Linux

Windows

macOS
```

Linux is the most commonly used platform in production.

---

## Java

Jenkins requires Java.

Current Long-Term Support (LTS) versions of Jenkins support Java 17 and Java 21.

Verify Java:

```bash
java -version
```

---

## Minimum Hardware

Development Environment

```text
CPU : 2 Cores

RAM : 4 GB

Disk : 20 GB
```

Production Environment

```text
CPU : 4+ Cores

RAM : 8 GB or More

Disk : 100+ GB
```

Disk usage increases over time because Jenkins stores:

- Build History
- Console Logs
- Workspace
- Plugins
- Artifacts

---

# Installing Jenkins on RHEL/CentOS/Rocky Linux

## Step 1 - Add Jenkins Repository

```bash
sudo curl -o /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

---

## Step 2 - Import Repository Key

```bash
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

---

## Step 3 - Install Java

```bash
sudo yum install fontconfig java-21-openjdk -y
```

Verify Installation

```bash
java -version
```

---

## Step 4 - Install Jenkins

```bash
sudo yum install jenkins -y
```

---

## Step 5 - Reload Systemd

```bash
sudo systemctl daemon-reload
```

---

## Step 6 - Start Jenkins

```bash
sudo systemctl start jenkins
```

---

## Step 7 - Enable Jenkins

```bash
sudo systemctl enable jenkins
```

---

## Step 8 - Verify Jenkins Status

```bash
sudo systemctl status jenkins
```

Expected Output

```text
Active (running)
```

---

# Jenkins Default Port

Jenkins runs on

```text
8080
```

Access Jenkins

```text
http://<server-ip>:8080
```

Example

```text
http://192.168.1.100:8080
```

---

# Firewall Configuration

If the firewall is enabled, allow port 8080.

```bash
sudo firewall-cmd --permanent --add-port=8080/tcp

sudo firewall-cmd --reload
```

---

# Initial Jenkins Unlock

During the first login, Jenkins asks for an administrator password.

Location

```text
/var/lib/jenkins/secrets/initialAdminPassword
```

Display Password

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password and paste it into the Jenkins UI.

---

# Install Suggested Plugins

After unlocking Jenkins

Select

```text
Install Suggested Plugins
```

Jenkins automatically installs commonly used plugins such as:

- Git
- Pipeline
- SSH
- Credentials
- Workspace Cleanup

Additional plugins can be installed later.

---

# Create Administrator User

Provide

- Username
- Password
- Email
- Full Name

This becomes the Jenkins administrator account.

---

# Jenkins Home Directory

One of the most important interview questions.

Default Location

```text
/var/lib/jenkins
```

Everything Jenkins needs is stored here.

Example Structure

```text
/var/lib/jenkins/

jobs/

workspace/

plugins/

secrets/

users/

logs/

config.xml
```

---

## jobs/

Stores all Jenkins jobs.

---

## workspace/

Temporary working directory.

When Jenkins clones a repository, builds code, or runs tests, it uses this directory.

---

## plugins/

Contains installed Jenkins plugins.

---

## secrets/

Stores encrypted credentials and the initial administrator password.

---

## users/

Stores Jenkins user information.

---

## config.xml

Main Jenkins configuration file.

---

# Useful Jenkins Commands

Check Version

```bash
java -jar jenkins.war --version
```

Restart Jenkins

```bash
sudo systemctl restart jenkins
```

Stop Jenkins

```bash
sudo systemctl stop jenkins
```

Start Jenkins

```bash
sudo systemctl start jenkins
```

Check Logs

```bash
sudo journalctl -u jenkins
```

Follow Logs

```bash
sudo journalctl -u jenkins -f
```

---

# Increasing Disk Space

Your trainer demonstrated extending logical volumes because Jenkins stores builds, logs, workspaces, and artifacts, which can quickly consume disk space.

Extend Home Volume

```bash
sudo lvextend -L +20G /dev/mapper/RootVG-homeVol
```

Extend Root Volume

```bash
sudo lvextend -L +10G /dev/mapper/RootVG-rootVol
```

> **Note:** After extending a logical volume, you also need to grow the filesystem using tools such as `xfs_growfs` (for XFS) or `resize2fs` (for ext4), depending on the filesystem in use.

---

# Real Production Example

Production Jenkins Server

```text
AWS EC2

↓

Rocky Linux

↓

Java 21

↓

Jenkins

↓

Git Plugin

↓

Docker Plugin

↓

SonarQube Plugin

↓

Pipeline Plugin

↓

AWS Plugin

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS
```

---

# Best Practices

- Use the latest Jenkins LTS version.
- Use Java LTS versions.
- Keep plugins updated.
- Store Jenkins Home on a dedicated volume.
- Perform regular backups of `/var/lib/jenkins`.
- Restrict access using RBAC.
- Secure Jenkins with HTTPS in production.
- Regularly remove unused plugins.

---

# Common Interview Questions

## Why is Java required for Jenkins?

### Short Answer

Jenkins is a Java-based application and runs on the Java Virtual Machine (JVM).

### Detailed Explanation

Jenkins is packaged as a WAR application. The JVM provides the runtime environment required to execute Jenkins, making it platform-independent.

### Production Example

A Jenkins controller running on Rocky Linux uses OpenJDK 21 to execute all CI/CD jobs.

### Follow-up Questions

- Which Java versions does Jenkins support?
- Can Jenkins run without Java?
- Why is Java LTS recommended?

---

## Where is Jenkins Home located?

### Short Answer

The default Jenkins Home directory is:

```text
/var/lib/jenkins
```

### Detailed Explanation

This directory contains jobs, plugins, user information, workspaces, secrets, logs, and the main configuration file. Backing up this directory preserves most Jenkins configuration and job data.

### Production Example

During a server migration, the team backed up `/var/lib/jenkins` and restored it on a new Jenkins controller.

### Follow-up Questions

- What is stored in the workspace directory?
- Where are plugins stored?
- Where is the initial admin password located?

---

## Why do Jenkins servers require large disks?

### Short Answer

Because Jenkins stores workspaces, logs, plugins, build history, artifacts, and reports.

### Detailed Explanation

As projects and build frequency increase, storage requirements grow significantly. Organizations often dedicate separate storage volumes for Jenkins Home.

### Production Example

A Jenkins server running hundreds of daily pipelines required extending its logical volume to prevent disk space issues.

### Follow-up Questions

- How do you monitor disk usage?
- What happens when Jenkins runs out of disk space?
- How can old build data be cleaned up?

---

# Production Scenarios

## Scenario 1

Jenkins service is not starting.

### Solution

```bash
sudo systemctl status jenkins

sudo journalctl -u jenkins
```

Review logs for Java, permission, or port-related errors.

---

## Scenario 2

Cannot access the Jenkins UI.

### Solution

- Verify the Jenkins service is running.
- Check firewall rules for port 8080.
- Confirm the server's security group allows inbound traffic.

---

## Scenario 3

Disk usage reaches 100%.

### Solution

- Remove old workspaces and builds.
- Configure build retention policies.
- Extend the logical volume if additional storage is required.

---

# Key Takeaways

```text
Jenkins is a Java-based application.

Java must be installed before Jenkins.

The default Jenkins Home directory is /var/lib/jenkins.

Jenkins listens on port 8080 by default.

The initial administrator password is stored in the secrets directory.

In production, monitor disk usage and regularly back up Jenkins Home.

Keep Jenkins and plugins updated for security and stability.
```