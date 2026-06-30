**File Name**

```text id="w3f9bt"
52-Backup-Automation-Script.md
```

````markdown id="n4y1qe"
# Backup Automation Script

## Introduction

In production environments, deleting old files immediately is not always a good practice.

A safer approach is:

```text
Find Old Files
      │
      ▼
Take Backup
      │
      ▼
Move Backup to Storage
      │
      ▼
Delete Original Files
```

This process is called:

```text
Backup Automation
```

---

# Why Backups?

Imagine:

```text
Application Logs Deleted
Database Files Deleted
Configuration Files Deleted
```

If no backup exists:

```text
Data Loss
```

Backups help recover files when problems occur.

---

# Real World Scenario

Source Directory:

```text
/var/log/roboshop-logs
```

Destination Directory:

```text
/backup
```

Requirement:

```text
Find files older than 14 days

Compress them

Move backup archive

Delete original files
```

---

# Backup Workflow

```text
Source Directory
       │
       ▼
Find Old Files
       │
       ▼
Create Archive
       │
       ▼
Move Archive
       │
       ▼
Delete Source Files
```

---

# Script Inputs

The script should be flexible.

Instead of hardcoding paths:

```text
Accept Arguments
```

---

# Example

```bash
sh backup.sh <source-dir> <destination-dir>
```

---

# Advanced Example

```bash
sh backup.sh <source-dir> <destination-dir> <days>
```

---

# Examples

Default:

```bash
sh backup.sh /app/logs /backup
```

---

Custom Days:

```bash
sh backup.sh /app/logs /backup 30
```

---

# Understanding Arguments

```bash
$1
```

Source Directory

---

```bash
$2
```

Destination Directory

---

```bash
$3
```

Days (Optional)

---

# Step 1: Validate Arguments

Requirement:

```text
Minimum 2 Arguments
```

---

# Example

```bash
if [ $# -lt 2 ]
then
    echo "Usage: backup.sh source-dir destination-dir"
    exit 1
fi
```

---

# Understanding $# 

Represents:

```text
Total Number of Arguments
```

---

# Example

```bash
sh backup.sh a b
```

Output:

```text
$# = 2
```

---

# Step 2: Store Arguments

```bash
SOURCE_DIR=$1

DEST_DIR=$2
```

---

# Optional Days Argument

```bash
DAYS=${3:-14}
```

---

# Understanding

If third argument exists:

```text
Use It
```

Otherwise:

```text
Use 14
```

---

# Examples

Command:

```bash
sh backup.sh logs backup
```

Result:

```text
DAYS=14
```

---

Command:

```bash
sh backup.sh logs backup 30
```

Result:

```text
DAYS=30
```

---

# Step 3: Validate Source Directory

Check:

```bash
if [ ! -d "$SOURCE_DIR" ]
then
    echo "Source Directory Not Found"
    exit 1
fi
```

---

# Step 4: Validate Destination Directory

Check:

```bash
if [ ! -d "$DEST_DIR" ]
then
    echo "Destination Directory Not Found"
    exit 1
fi
```

---

# Why Validate?

Without validation:

```text
Backup Fails
Archive Lost
Unexpected Errors
```

---

# Finding Old Files

Requirement:

```text
Older Than N Days
```

---

# Command

```bash
find "$SOURCE_DIR" \
-name "*.log" \
-mtime +$DAYS
```

---

# Example

```bash
find /app/logs \
-name "*.log" \
-mtime +14
```

---

# Why Use find?

It allows:

- Pattern Matching
- Date Filtering
- Recursive Search

---

# Compression

Instead of moving hundreds of files:

```text
Compress Into Single Archive
```

---

# Popular Compression Tools

```text
zip
tar
gzip
```

---

# Common Linux Approach

```bash
tar
```

---

# What is tar?

TAR stands for:

```text
Tape Archive
```

Used to bundle multiple files.

---

# Example Archive Name

```text
logs-2026-06-28.tar.gz
```

---

# Create Timestamp

```bash
TIMESTAMP=$(date +%F)
```

---

# Example Output

```text
2026-06-28
```

---

# Archive File Name

```bash
BACKUP_FILE=$DEST_DIR/logs-$TIMESTAMP.tar.gz
```

---

# Create Archive

```bash
tar -czvf $BACKUP_FILE files
```

---

# Understanding Options

```text
c → Create

z → Compress

v → Verbose

f → File Name
```

---

# Complete Logic

```text
Find Files
      │
      ▼
Create Archive
      │
      ▼
Store In Destination
```

---

# Delete Original Files

After successful backup:

```bash
rm -f file-name
```

---

# Important Rule

Never delete files before:

```text
Backup Success Validation
```

---

# Backup Script Algorithm

```text
Receive Arguments
        │
        ▼
Validate Arguments
        │
        ▼
Validate Source Directory
        │
        ▼
Validate Destination Directory
        │
        ▼
Find Old Files
        │
        ▼
Create Archive
        │
        ▼
Validate Archive
        │
        ▼
Delete Source Files
```

---

# Logging

Store backup activity.

Example:

```text
/var/log/shellscript-logs/backup.log
```

---

# Example

```bash
echo "Backup Created: $BACKUP_FILE" >> backup.log
```

---

# Sample Script Structure

```bash
#!/bin/bash

SOURCE_DIR=$1
DEST_DIR=$2
DAYS=${3:-14}

Validate Inputs

Validate Directories

Find Files

Create Archive

Delete Files

Log Results
```

---

# Production Use Cases

## Application Logs

```text
Backup Old Logs
```

---

## Audit Logs

```text
Compliance Requirements
```

---

## Configuration Files

```text
Backup Before Changes
```

---

## Database Dumps

```text
Archive Historical Dumps
```

---

# Cron Scheduling

Backups are usually automated using:

```bash
crontab -e
```

---

# Example

Run Daily:

```bash
0 1 * * * /home/ec2-user/backup.sh
```

---

# Meaning

```text
1:00 AM Every Day
```

---

# Common Mistakes

## No Validation

Directories may not exist.

---

## Delete Before Backup

Risk of data loss.

---

## No Logging

Difficult troubleshooting.

---

## Hardcoded Paths

Less reusable.

---

# Best Practices

## Accept Arguments

Avoid hardcoding.

---

## Validate Directories

Before processing.

---

## Use Compression

Save storage.

---

## Log Every Operation

Maintain history.

---

## Verify Backup Success

Before deletion.

---

## Schedule with Cron

Avoid manual execution.

---

# Frequently Asked Interview Questions

## Why Do We Need Backups?

To prevent data loss.

---

## What Does $# Represent?

Total arguments passed to script.

---

## What Does ${3:-14} Mean?

Use third argument if provided, otherwise use 14.

---

## Which Command Finds Old Files?

```bash
find
```

---

## Which Command Creates Archives?

```bash
tar
```

---

## What Does tar -czvf Do?

Creates compressed archive.

---

## Why Validate Directories?

To avoid backup failures.

---

## Why Log Backup Operations?

For auditing and troubleshooting.

---

## Why Use Cron?

To automate scheduled backups.

---

# Key Takeaways

- Backup automation is a common DevOps task.
- Scripts should accept source and destination directories as arguments.
- Always validate input arguments.
- Validate source and destination directories.
- Use `find` to locate old files.
- Use `tar` to create compressed archives.
- Verify backup success before deleting files.
- Log backup activities.
- Schedule backups using cron jobs.
- Automated backups reduce operational risk and prevent data loss.
````