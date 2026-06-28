# Production Monitoring Scripts with Email Alerts

## Introduction

Monitoring is one of the most important responsibilities of a DevOps Engineer.

The goal is:

```text
Detect Problems
      ↓
Generate Alerts
      ↓
Notify Team
      ↓
Resolve Issues
```

Common resources monitored:

- CPU
- Memory
- Disk
- Services
- Applications

In production environments, monitoring scripts are usually:

```text
Automated
Scheduled using Cron
Integrated with Email Alerts
```

---

# Monitoring Workflow

```text
Collect Metrics
        │
        ▼
Compare with Threshold
        │
        ▼
Generate Alert
        │
        ▼
Send Email
        │
        ▼
Log Activity
```

---

# Prerequisites

## Install msmtp

```bash
dnf install msmtp -y
```

---

## Configure Gmail SMTP

```text
~/.msmtprc
```

Example:

```text
defaults
auth on
tls on
tls_trust_file /etc/ssl/certs/ca-bundle.crt
logfile /var/log/msmtp.log

account gmail
host smtp.gmail.com
port 587

from joindevops@gmail.com
user joindevops@gmail.com
password APP_PASSWORD

account default : gmail
```

---

# Email Template

```text
Hi Team,

There is an ALERT_TYPE in the system IP_ADDRESS.

Please find the details below:

MESSAGE

Regards,
Monitoring Team
```

---

# Disk Usage Monitoring

## Why Monitor Disk?

If disk usage reaches:

```text
100%
```

Applications may fail.

Examples:

- MySQL stops writing
- Logs stop generating
- Deployments fail

---

# Check Disk Usage

Command:

```bash
df -hT
```

Example:

```text
Filesystem     Type  Size Used Avail Use%
/dev/xvda1     xfs   20G  18G   2G   90%
```

---

# Production Disk Monitoring Script

```bash
#!/bin/bash

THRESHOLD=80

TO_TEAM="devops-team@example.com"

HOST=$(hostname -I | awk '{print $1}')

MESSAGE=""

while read line
do

    PARTITION=$(echo $line | awk '{print $1}')

    USAGE=$(echo $line | awk '{print $6}' | sed 's/%//')

    if [ "$USAGE" -ge "$THRESHOLD" ]
    then

        MESSAGE+="Partition: $PARTITION Usage: ${USAGE}%<br>"

    fi

done < <(df -hT | grep -vE 'tmpfs|Filesystem')

if [ ! -z "$MESSAGE" ]
then

{
echo "To: $TO_TEAM"
echo "Subject: HIGH DISK USAGE ALERT"
echo "Content-Type: text/html"
echo ""
echo "Hi Team,<br><br>"
echo "High Disk Usage detected on server $HOST.<br><br>"
echo "$MESSAGE"
echo "<br>Regards,<br>Monitoring Team"
} | msmtp "$TO_TEAM"

fi
```

---

# Disk Monitoring Logic

```text
Run df -hT
      │
      ▼
Loop Through Partitions
      │
      ▼
Check Usage %
      │
      ▼
If Usage > Threshold
      │
      ▼
Send Alert
```

---

# CPU Monitoring

## Why Monitor CPU?

High CPU causes:

```text
Slow Applications
Timeouts
Poor User Experience
```

---

# CPU Usage

Linux provides:

```bash
top
```

Example:

```text
99.7 id
```

Meaning:

```text
99.7% Idle
```

CPU Usage:

```text
100 - Idle
```

---

# Production CPU Monitoring Script

```bash
#!/bin/bash

CPU_THRESHOLD=80

TO_TEAM="devops-team@example.com"

HOST=$(hostname -I | awk '{print $1}')

CPU_IDLE=$(top -bn1 | grep "Cpu(s)" | awk '{print $8}' | cut -d. -f1)

CPU_USAGE=$((100-CPU_IDLE))

if [ "$CPU_USAGE" -ge "$CPU_THRESHOLD" ]
then

{
echo "To: $TO_TEAM"
echo "Subject: HIGH CPU ALERT"
echo "Content-Type: text/html"
echo ""
echo "Hi Team,<br><br>"
echo "High CPU Usage detected.<br><br>"
echo "Server: $HOST<br>"
echo "CPU Usage: ${CPU_USAGE}%<br><br>"
echo "Regards,<br>Monitoring Team"
} | msmtp "$TO_TEAM"

fi
```

---

# CPU Monitoring Logic

```text
Get CPU Idle %
       │
       ▼
Calculate CPU Usage
       │
       ▼
Compare with Threshold
       │
       ▼
Generate Alert
```

---

# Memory Monitoring

## Why Monitor Memory?

High memory usage can cause:

```text
OOMKilled Containers
Application Crashes
Node Instability
```

---

# Check Memory

```bash
free -m
```

Example:

```text
Total
Used
Free
Available
```

---

# Production Memory Monitoring Script

```bash
#!/bin/bash

MEMORY_THRESHOLD=80

TO_TEAM="devops-team@example.com"

HOST=$(hostname -I | awk '{print $1}')

MEMORY_USAGE=$(free | awk '/Mem/ {printf("%.0f"), $3/$2 * 100}')

if [ "$MEMORY_USAGE" -ge "$MEMORY_THRESHOLD" ]
then

{
echo "To: $TO_TEAM"
echo "Subject: HIGH MEMORY ALERT"
echo "Content-Type: text/html"
echo ""
echo "Hi Team,<br><br>"
echo "Memory Usage exceeded threshold.<br><br>"
echo "Server: $HOST<br>"
echo "Memory Usage: ${MEMORY_USAGE}%<br><br>"
echo "Regards,<br>Monitoring Team"
} | msmtp "$TO_TEAM"

fi
```

---

# Service Monitoring

Applications may stop unexpectedly.

Examples:

```text
nginx
mysql
shipping
catalogue
payment
```

---

# Check Service Status

```bash
systemctl is-active nginx
```

---

# Production Service Monitoring Script

```bash
#!/bin/bash

SERVICE=nginx

STATUS=$(systemctl is-active $SERVICE)

if [ "$STATUS" != "active" ]
then

{
echo "To: devops-team@example.com"
echo "Subject: SERVICE DOWN ALERT"
echo ""
echo "$SERVICE service is DOWN"
} | msmtp devops-team@example.com

fi
```

---

# Logging

Monitoring scripts should maintain logs.

Example:

```bash
LOGFILE=/var/log/resource-monitor.log
```

---

# Log Entry

```bash
echo "$(date) : Alert Sent" >> $LOGFILE
```

---

# Better Logging Example

```bash
echo "$(date) : CPU Alert Sent" >> $LOGFILE
```

---

# HTML Email Formatting

Instead of:

```text
Plain Text
```

Use:

```html
<h3>Alert</h3>
<table border="1">
<tr>
<th>Resource</th>
<th>Usage</th>
</tr>
</table>
```

---

# Combined Monitoring Script

Production teams often create:

```text
monitor.sh
```

which checks:

- CPU
- RAM
- Disk
- Services

and sends a single alert email.

---

# Combined Workflow

```text
CPU Check
      │
      ▼
Memory Check
      │
      ▼
Disk Check
      │
      ▼
Service Check
      │
      ▼
Generate Alert
      │
      ▼
Send Email
```

---

# Cron Scheduling

## Run Every 5 Minutes

```cron
*/5 * * * * /home/ec2-user/disk-monitor.sh
```

---

## CPU Monitoring

```cron
*/5 * * * * /home/ec2-user/cpu-monitor.sh
```

---

## Memory Monitoring

```cron
*/5 * * * * /home/ec2-user/memory-monitor.sh
```

---

## Service Monitoring

```cron
*/5 * * * * /home/ec2-user/service-monitor.sh
```

---

# Production Improvements

## Alert Cooldown

Avoid:

```text
100 Emails in 5 Minutes
```

Send only one alert within a specific interval.

---

## Multiple Recipients

```text
devops@example.com
sre@example.com
manager@example.com
```

---

## Config File

Store thresholds in:

```text
monitor.conf
```

Example:

```text
CPU_THRESHOLD=80
MEMORY_THRESHOLD=80
DISK_THRESHOLD=80
```

---

## Use Functions

Avoid repeating email logic.

Example:

```bash
SEND_ALERT(){
    # email logic
}
```

---

# Common Mistakes

## No Thresholds

Alerts generated unnecessarily.

---

## No Logging

Hard to troubleshoot.

---

## No Cron Scheduling

Manual monitoring is unreliable.

---

## Hardcoded Email Addresses

Difficult to maintain.

---

# Best Practices

- Monitor CPU, Memory, Disk, and Services.
- Use thresholds to avoid false alerts.
- Store logs for auditing.
- Use HTML emails for readability.
- Schedule scripts using Cron.
- Use functions for reusable code.
- Avoid alert flooding.
- Test scripts before production deployment.

---

# Frequently Asked Interview Questions

## Why Monitor CPU?

To identify performance bottlenecks.

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

## Which Command Checks CPU?

```bash
top
```

---

## What is SMTP?

Simple Mail Transfer Protocol used for sending emails.

---

## Why Use msmtp?

To send emails from Linux scripts.

---

## Why Schedule Monitoring Scripts?

To automate health checks.

---

## Which Linux Scheduler is Commonly Used?

```text
Crontab
```

---

# Key Takeaways

- Monitoring is a critical DevOps responsibility.
- CPU, Memory, Disk, and Services should be monitored continuously.
- Email alerts help teams respond quickly.
- msmtp can be used for sending alerts from shell scripts.
- Cron automates monitoring execution.
- Logging improves troubleshooting.
- Production-grade monitoring requires thresholds, alerting, logging, and scheduling.