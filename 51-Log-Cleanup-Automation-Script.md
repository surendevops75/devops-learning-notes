# Log Cleanup Automation Script

## Introduction

In production environments, applications continuously generate logs.

Examples:

```text
Application Logs
Nginx Logs
Apache Logs
System Logs
Database Logs
```

If logs are not cleaned regularly:

```text
Disk Space Fills Up
Performance Issues
Server Outages
```

To avoid this, DevOps engineers automate log cleanup.

---

# Real World Scenario

Suppose logs are stored in:

```text
/var/log/roboshop-logs
```

Example files:

```text
cart-2025-06-01.log
cart-2025-06-05.log
cart-2025-06-10.log
cart-2025-06-25.log
```

Requirement:

```text
Delete all .log files older than 14 days
```

---

# Problem Statement

Tasks:

```text
1. Check source directory exists
2. Find .log files
3. Find files older than 14 days
4. Log files being deleted
5. Delete files
```

---

# Service Health Check Example

Before cleanup, we may verify application health.

Example:

```text
Check Shipping Service
```

Commands:

```bash
systemctl status shipping
```

or

```bash
ps -ef | grep shipping
```

or

```bash
curl http://localhost:8080/health
```

---

# Finding Files

Linux provides:

```bash
find
```

command.

---

# Syntax

```bash
find <path> <conditions>
```

---

# Example

Find all log files:

```bash
find /var/log/roboshop-logs -name "*.log"
```

---

# Understanding

```text
find                     → Search Utility

/var/log/roboshop-logs   → Search Location

-name                    → File Name Filter

*.log                    → Log Files
```

---

# Finding Old Files

Requirement:

```text
Older than 14 Days
```

Use:

```bash
-mtime +14
```

---

# Example

```bash
find /var/log/roboshop-logs \
-name "*.log" \
-mtime +14
```

---

# Understanding mtime

mtime means:

```text
Modification Time
```

---

Examples:

```text
-mtime 7     → Exactly 7 Days

-mtime -7    → Less Than 7 Days

-mtime +7    → Greater Than 7 Days
```

---

# Find Command Output

Example:

```text
/var/log/roboshop-logs/cart.log
/var/log/roboshop-logs/shipping.log
/var/log/roboshop-logs/payment.log
```

---

# Why Not Delete Immediately?

Good DevOps practice:

```text
Log Files Being Deleted
```

before deletion.

This helps:

```text
Auditing
Troubleshooting
Verification
```

---

# While Loop

The `find` command output can be processed using:

```bash
while
```

loop.

---

# Syntax

```bash
while [ condition ]
do
    commands
done
```

---

# File Processing Pattern

```bash
find ... | while read FILE
do
    commands
done
```

---

# Example

```bash
find /var/log/roboshop-logs \
-name "*.log" \
-mtime +14 | while read FILE
do
    echo "$FILE"
done
```

---

# Output

```text
/var/log/roboshop-logs/cart.log
/var/log/roboshop-logs/shipping.log
```

---

# Logging Files Before Deletion

Example:

```bash
echo "Deleting file: $FILE"
```

---

# Deleting Files

Command:

```bash
rm -f "$FILE"
```

---

# Complete Logic

```text
Find File
    │
    ▼
Log File Name
    │
    ▼
Delete File
```

---

# Script Algorithm

Step 1:

```text
Check Directory Exists
```

---

Step 2:

```text
Find .log Files
```

---

Step 3:

```text
Filter Older Than 14 Days
```

---

Step 4:

```text
Log Files
```

---

Step 5:

```text
Delete Files
```

---

# Directory Validation

Before cleanup:

```bash
if [ ! -d "$SOURCE_DIR" ]
then
    echo "Directory Not Found"
    exit 1
fi
```

---

# Understanding -d

```bash
-d
```

checks:

```text
Directory Exists
```

---

# Cleanup Script Structure

```text
Validate Directory
        │
        ▼
Find Old Logs
        │
        ▼
Display Files
        │
        ▼
Delete Files
        │
        ▼
Generate Log Output
```

---

# Sample Script

```bash
#!/bin/bash

SOURCE_DIR="/var/log/roboshop-logs"

if [ ! -d "$SOURCE_DIR" ]
then
    echo "Directory Not Found"
    exit 1
fi

find "$SOURCE_DIR" \
-name "*.log" \
-mtime +14 | while read FILE
do
    echo "Deleting: $FILE"
    rm -f "$FILE"
done
```

---

# Production Enhancements

Instead of:

```text
Display on Screen
```

Store logs:

```text
/var/log/shellscript-logs/cleanup.log
```

---

# Example

```bash
LOG_FILE=/var/log/shellscript-logs/cleanup.log
```

---

# Append Log Entries

```bash
echo "Deleting $FILE" >> $LOG_FILE
```

---

# Real DevOps Use Cases

## Application Logs

```text
Roboshop Logs
```

---

## Nginx Logs

```text
/var/log/nginx
```

---

## Apache Logs

```text
/var/log/httpd
```

---

## Custom Application Logs

```text
/app/logs
```

---

# Common Mistakes

## Using rm Without Validation

Dangerous:

```bash
rm -rf *
```

---

## Wrong Directory

Example:

```text
Production Data Deleted
```

Always verify path.

---

## Not Logging Deleted Files

Makes troubleshooting difficult.

---

# Best Practices

## Validate Directory

Before running cleanup.

---

## Log All Deleted Files

Maintain audit history.

---

## Test Using echo

Before deleting:

```bash
echo "$FILE"
```

Verify output.

---

## Restrict Search Pattern

Use:

```bash
*.log
```

instead of deleting everything.

---

# Frequently Asked Interview Questions

## Why Do We Need Log Cleanup?

To prevent disk space issues.

---

## Which Command Is Used to Find Files?

```bash
find
```

---

## What Does -name Do?

Filters files by name.

---

## What Does -mtime +14 Mean?

Files older than 14 days.

---

## What Does -d Check?

Directory existence.

---

## Why Use While Loop?

To process files one by one.

---

## Why Log Deleted Files?

For auditing and troubleshooting.

---

# Key Takeaways

- Production applications generate large numbers of logs.
- Old logs should be cleaned regularly.
- `find` is commonly used to locate files.
- `-name "*.log"` filters log files.
- `-mtime +14` finds files older than 14 days.
- Always validate directories before cleanup.
- Use loops to process files individually.
- Log file deletions before removing them.
- Automated cleanup scripts help maintain server health and disk space.