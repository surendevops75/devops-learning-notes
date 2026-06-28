# Crontab and Scheduled Automation

## Introduction

Many administrative tasks need to run automatically without human intervention.

Examples:

- Backups
- Log Cleanup
- Monitoring Scripts
- Health Checks
- Database Maintenance
- Report Generation

Instead of manually running scripts every day, Linux provides a scheduler called:

```text
Crontab
```

---

# What is Crontab?

Crontab is a Linux scheduler used to execute commands or scripts automatically at specified times.

Think of it as:

```text
Alarm Clock for Scripts
```

---

# Why Do We Need Crontab?

Without Crontab:

```text
Login Daily
Run Script
Logout
```

With Crontab:

```text
Schedule Once

Runs Automatically Forever
```

---

# Real World Example

Backup Script:

```bash
backup.sh
```

Requirement:

```text
Run Every Day at 2:00 AM
```

No manual work required.

---

# Why Run Jobs at Night?

Most organizations schedule maintenance during:

```text
Low Traffic Hours
```

Example:

```text
12:00 AM - 06:00 AM
```

Reasons:

- Fewer Users
- Lower CPU Usage
- Lower Memory Usage
- Less Business Impact

---

# Common Scheduled Tasks

## Backup

```text
Daily
Weekly
Monthly
```

---

## Log Cleanup

```text
Delete Old Logs
```

---

## Health Checks

```text
Disk Usage
CPU Usage
Memory Usage
```

---

## Application Monitoring

```text
Check Service Status
```

---

# Cron Service

Linux runs a background service responsible for scheduling.

Service Name:

```text
crond
```

Check Status:

```bash
systemctl status crond
```

---

# Start Cron Service

```bash
systemctl start crond
```

---

# Enable Cron Service

```bash
systemctl enable crond
```

---

# What is a Crontab Entry?

A crontab entry defines:

```text
When To Run
What To Run
```

---

# Crontab Syntax

```text
* * * * * command
```

---

# Understanding Fields

```text
* * * * *
│ │ │ │ │
│ │ │ │ └── Day of Week
│ │ │ └──── Month
│ │ └────── Day of Month
│ └──────── Hour
└────────── Minute
```

---

# Cron Format

```text
Minute Hour Day Month DayOfWeek Command
```

---

# Example

```text
0 2 * * * command
```

Meaning:

```text
Run Daily at 2:00 AM
```

---

# Special Characters

## Asterisk (*)

Means:

```text
Every
```

---

Example:

```text
* * * * *
```

Meaning:

```text
Every Minute
```

---

# Example

```text
0 * * * *
```

Meaning:

```text
Every Hour
```

---

# Example

```text
0 0 * * *
```

Meaning:

```text
Every Day at Midnight
```

---

# Editing Crontab

Open:

```bash
crontab -e
```

---

# View Existing Jobs

```bash
crontab -l
```

---

# Remove Crontab

```bash
crontab -r
```

---

# Backup Script Example

Script:

```bash
backup.sh
```

Requirement:

```text
Run Daily at 2 AM
```

Entry:

```text
0 2 * * * /home/ec2-user/backup.sh
```

---

# Real Production Example

```text
50 Servers
```

You develop:

```text
backup.sh
```

---

Process:

```text
Create Script
       │
       ▼
Test on Sample Server
       │
       ▼
Validate Results
       │
       ▼
Deploy to All Servers
       │
       ▼
Add Crontab Entry
```

---

# Production Issue Example

Suppose script works in:

```text
Development
Testing
```

---

Later fails in:

```text
Production
```

Reason:

```text
zip command not installed
```

---

Lesson:

```text
Validate Dependencies
```

inside scripts.

---

# Better Backup Example

Command:

```text
0 2 * * * sudo backup.sh source_dir dest_dir
```

Meaning:

```text
Run Daily at 2:00 AM
```

using sudo privileges.

---

# Why Use sudo?

Some scripts require:

- Root Access
- System Directories
- Service Management

---

# Cron Environment

Cron runs with a limited environment.

Commands working manually may fail in cron.

---

# Common Problem

Works:

```bash
zip file.zip logs
```

Manually.

---

Fails:

```text
Command Not Found
```

inside cron.

---

# Solution

Use absolute paths.

Example:

```bash
/usr/bin/zip
```

instead of:

```bash
zip
```

---

# Logging Cron Jobs

Always capture output.

Bad:

```text
No Logs
```

---

Good:

```text
Store Output
```

---

Example:

```text
0 2 * * * backup.sh >> /var/log/backup.log 2>&1
```

---

# Understanding

```text
>>      Append Output

2>&1    Redirect Errors
```

---

# Verify Cron Execution

Check:

```bash
grep CRON /var/log/cron
```

---

# Cron Workflow

```text
Write Script
      │
      ▼
Test Script
      │
      ▼
Deploy Script
      │
      ▼
Add Cron Entry
      │
      ▼
Monitor Logs
```

---

# Common Scheduling Examples

## Every Minute

```text
* * * * *
```

---

## Every Hour

```text
0 * * * *
```

---

## Daily at Midnight

```text
0 0 * * *
```

---

## Daily at 2 AM

```text
0 2 * * *
```

---

## Every Sunday

```text
0 2 * * 0
```

---

## Monthly Backup

```text
0 2 1 * *
```

Meaning:

```text
First Day of Every Month
```

---

# Common Mistakes

## Script Not Executable

Fix:

```bash
chmod +x script.sh
```

---

## Wrong Path

Cron may not find scripts.

Use full paths.

---

## Missing Dependencies

Example:

```text
zip
tar
aws
mysql
```

not installed.

---

## No Logging

Difficult troubleshooting.

---

# Best Practices

## Test Manually First

Before scheduling.

---

## Use Absolute Paths

Example:

```bash
/usr/bin/zip
```

---

## Log Everything

Store output and errors.

---

## Schedule During Low Traffic Hours

Example:

```text
12 AM - 6 AM
```

---

## Validate Dependencies

Check required tools before execution.

---

# Frequently Asked Interview Questions

## What is Crontab?

A Linux scheduler used to run commands automatically.

---

## How Do You Edit Crontab?

```bash
crontab -e
```

---

## How Do You View Cron Jobs?

```bash
crontab -l
```

---

## What Does * Mean?

```text
Every
```

---

## What Does 0 2 * * * Mean?

Run daily at 2:00 AM.

---

## Why Schedule Jobs at Night?

Low traffic and minimal impact.

---

## Why Use Absolute Paths in Cron?

Cron has a limited environment.

---

## How Do You Log Cron Output?

```text
>> logfile 2>&1
```

---

# Key Takeaways

- Crontab is Linux's built-in scheduler.
- Cron jobs automate repetitive tasks.
- Common uses include backups, monitoring, and cleanup.
- Cron entries define when and what to execute.
- Jobs are usually scheduled during low-traffic hours.
- Always test scripts before scheduling.
- Use absolute paths inside cron jobs.
- Log output and errors for troubleshooting.
- Validate dependencies before production deployment.
- Crontab is one of the most commonly used automation tools in Linux administration and DevOps.