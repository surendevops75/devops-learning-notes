# Linux Monitoring and Email Alerts

## Introduction

Monitoring is one of the most important responsibilities of a DevOps Engineer.

A healthy system should be continuously monitored for:

- CPU Usage
- Memory Usage
- Disk Usage
- Application Health
- Service Availability
- Network Connectivity

When problems occur, alerts should be sent automatically to the responsible team.

---

# What is Monitoring?

Monitoring is the process of continuously observing system resources and applications to detect issues before they impact users.

---

# Why Monitoring?

Without monitoring:

```text
Server Down
Application Failure
Disk Full
Memory Exhausted

Nobody Knows
```

---

With monitoring:

```text
Problem Detected
      │
      ▼
Alert Generated
      │
      ▼
Team Notified
      │
      ▼
Issue Resolved
```

---

# Key Resources to Monitor

## CPU

Measures processor utilization.

---

## Memory (RAM)

Measures memory consumption.

---

## Disk Usage

Measures storage utilization.

---

## Services

Checks whether applications are running.

Examples:

```text
nginx
mysql
shipping
catalogue
payment
```

---

# Disk Monitoring

One of the most common monitoring activities.

Command:

```bash
df -hT
```

---

# Understanding df

```text
df → Disk Filesystem
```

Displays filesystem usage.

---

# Options

```text
-h → Human Readable

-T → Filesystem Type
```

---

# Example Output

```text
Filesystem     Type  Size Used Avail Use%
/dev/xvda1     xfs   20G  10G   10G  50%
```

---

# Important Fields

```text
Filesystem
Size
Used
Available
Use%
```

---

# Why Monitor Disk Usage?

If disk becomes full:

```text
Applications Fail
Logs Stop Writing
Databases Crash
```

---

# Memory Monitoring

Check memory:

```bash
free -m
```

---

# Example Output

```text
Total
Used
Free
Available
```

---

# CPU Monitoring

Command:

```bash
top
```

or

```bash
top -b -n1
```

---

# CPU Idle Percentage

Example:

```text
99.7 id
```

---

# Meaning

```text
99.7% CPU Idle
```

CPU Usage:

```text
100 - 99.7
```

Result:

```text
0.3% Used
```

---

# Understanding CPU Idle

High Idle:

```text
Good
```

Example:

```text
95% Idle
```

---

Low Idle:

```text
Heavy CPU Usage
```

Example:

```text
5% Idle
```

---

# Alert Thresholds

Example:

```text
Disk Usage > 80%

CPU Usage > 85%

Memory Usage > 90%
```

Generate alerts.

---

# What is Alerting?

Alerting is the process of notifying teams when thresholds are crossed.

---

# Alert Channels

Common options:

```text
Email
SMS
Slack
Microsoft Teams
PagerDuty
```

---

# Email Alerts

One of the simplest alerting methods.

Example:

```text
Disk Usage Alert
CPU Alert
Memory Alert
```

---

# SMTP

SMTP stands for:

```text
Simple Mail Transfer Protocol
```

Used to send emails.

---

# Gmail SMTP Server

Example:

```text
smtp.gmail.com
```

---

# Port

```text
587
```

Used for TLS communication.

---

# Two-Step Verification

For Gmail SMTP:

```text
2-Step Verification
```

must be enabled.

---

# Why?

Google blocks direct password usage for applications.

Instead use:

```text
App Password
```

---

# App Password

Example:

```text
pcrxrsuhsorrcvnh
```

Generated from Google Account Security Settings.

---

# msmtp

A lightweight SMTP client for Linux.

Used for sending emails from scripts.

---

# Installation

```bash
dnf install msmtp -y
```

---

# Configuration File

Example:

```text
~/.msmtprc
```

---

# Basic Configuration

```text
defaults
auth on
tls on
tls_trust_file /etc/ssl/certs/ca-bundle.crt
logfile /var/log/msmtp.log
```

---

# Gmail Account Configuration

```text
account gmail

host smtp.gmail.com

port 587

from joindevops@gmail.com

user joindevops@gmail.com

password APP_PASSWORD
```

---

# Default Account

```text
account default : gmail
```

---

# Sending Email

Example:

```bash
{
echo "To: info@example.com"
echo "Subject: Test Mail"
echo ""
echo "This is test email"
} | msmtp "info@example.com"
```

---

# Email Components

```text
To
Subject
Body
```

---

# Example

```text
To: team@example.com

Subject: Disk Usage Alert

Body:
Disk Usage Exceeded Threshold
```

---

# HTML Emails

Plain text emails are simple.

HTML emails provide:

```text
Colors
Tables
Formatting
```

---

# HTML Conversion

Tools:

```text
Text to HTML Converter
```

can convert text into HTML format.

---

# Example Alert Template

```html
Hi Team,

There is an ALERT_TYPE in the system IP_ADDRESS.

Please find the details below:

MESSAGE

Regards,
Monitoring Team
```

---

# Dynamic Alerts

Variables can be replaced:

```text
ALERT_TYPE

IP_ADDRESS

MESSAGE
```

---

# Example

```text
ALERT_TYPE = Disk Usage

IP_ADDRESS = 172.31.10.15

MESSAGE = Usage exceeded 80%
```

---

# Monitoring Workflow

```text
Collect Metrics
        │
        ▼
Compare Thresholds
        │
        ▼
Generate Alert
        │
        ▼
Send Email
```

---

# Real DevOps Example

Monitor:

```text
Disk Usage
```

If:

```text
Usage > 80%
```

Then:

```text
Send Email Alert
```

to operations team.

---

# Common Mistakes

## No Monitoring

Problems discovered too late.

---

## No Alerting

Teams unaware of failures.

---

## Hardcoded Email Addresses

Difficult to maintain.

---

## Storing Passwords In Scripts

Security risk.

---

# Best Practices

## Use App Passwords

Instead of Gmail account password.

---

## Enable TLS

Secure communication.

---

## Monitor Critical Resources

CPU, RAM, Disk.

---

## Keep Alert Messages Clear

Include:

```text
Problem
Server
Timestamp
```

---

## Use Distribution Lists

Instead of individual email addresses.

---

# Frequently Asked Interview Questions

## What is Monitoring?

Continuous observation of systems and applications.

---

## Why is Monitoring Important?

To detect issues before users are impacted.

---

## Which Command Checks Disk Usage?

```bash
df -hT
```

---

## Which Command Checks Memory?

```bash
free -m
```

---

## Which Command Checks CPU Usage?

```bash
top
```

---

## What Does 99.7 id Mean?

99.7% CPU Idle.

---

## What is SMTP?

Simple Mail Transfer Protocol.

---

## Why is Gmail App Password Required?

For secure authentication when using applications.

---

## What is msmtp?

A lightweight Linux email client.

---

# Key Takeaways

- Monitoring is a core DevOps responsibility.
- CPU, RAM, and Disk are the most commonly monitored resources.
- `df -hT` is used for disk monitoring.
- `free -m` is used for memory monitoring.
- `top` is used for CPU monitoring.
- Email alerts notify teams about system issues.
- SMTP is used to send emails.
- Gmail requires App Passwords for application access.
- `msmtp` can be used to send emails from shell scripts.
- Monitoring combined with alerting helps prevent outages and improve system reliability.