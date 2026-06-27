# Shell Scripting Logging, Colors and Redirections

## Introduction

In production environments, simply executing commands is not enough.

DevOps engineers must know:

- What happened?
- When did it happen?
- Why did it fail?
- Where are the logs?

This is where:

- Logging
- Colors
- Redirections

become important.

These concepts are used in almost every production-grade shell script.

---

# Why Logging?

Imagine running a deployment script.

```bash
dnf install nginx -y
systemctl start nginx
systemctl enable nginx
```

If the script fails:

```text
How do you know what happened?
```

Without logs:

```text
Troubleshooting becomes difficult.
```

---

# What is Logging?

Logging means recording script execution details into a file.

Logs help us understand:

```text
What Happened
When It Happened
Why It Happened
```

---

# Benefits of Logging

- Troubleshooting
- Auditing
- Monitoring
- Debugging
- Historical Records

---

# Log Storage Locations

Most Linux logs are stored under:

```text
/var/log
```

---

# Common Log Locations

System Logs:

```text
/var/log/messages
```

---

Authentication Logs:

```text
/var/log/secure
```

---

Application Logs:

```text
/var/log/<application-name>
```

---

# Shell Script Logs

A common practice is:

```text
/var/log/shellscript-logs
```

---

Example:

```text
/var/log/shellscript-logs/15-colors.log
```

---

# Creating Log Directory

```bash
mkdir -p /var/log/shellscript-logs
```

---

# Creating Log File

```bash
LOG_FILE=/var/log/shellscript-logs/install.log
```

---

# Why Store Logs?

Suppose a script runs at:

```text
2:00 AM
```

and fails.

Logs help identify:

```text
Failure Point
Error Message
Execution Flow
```

---

# Colors in Shell Scripts

Colors improve readability.

Instead of:

```text
SUCCESS
FAILED
SKIPPED
```

Use colors for easier identification.

---

# Common Colors

| Status | Color |
|----------|---------|
| Success | Green |
| Failure | Red |
| Skip | Yellow |

---

# Color Escape Codes

Red:

```bash
RED="\e[31m"
```

---

Green:

```bash
GREEN="\e[32m"
```

---

Yellow:

```bash
YELLOW="\e[33m"
```

---

Reset Color:

```bash
RESET="\e[0m"
```

---

# Success Example

```bash
echo -e "\e[32m SUCCESS \e[0m"
```

Output:

```text
SUCCESS (Green)
```

---

# Failure Example

```bash
echo -e "\e[31m FAILURE \e[0m"
```

Output:

```text
FAILURE (Red)
```

---

# Skip Example

```bash
echo -e "\e[33m SKIPPED \e[0m"
```

Output:

```text
SKIPPED (Yellow)
```

---

# Production Example

```bash
if [ $? -eq 0 ]; then
    echo -e "\e[32m SUCCESS \e[0m"
else
    echo -e "\e[31m FAILURE \e[0m"
fi
```

---

# File Descriptors

Linux treats everything as a file.

Every process automatically gets:

| Descriptor | Purpose |
|------------|----------|
| 0 | STDIN |
| 1 | STDOUT |
| 2 | STDERR |

---

# STDIN

Standard Input.

Usually:

```text
Keyboard Input
```

Descriptor:

```text
0
```

---

# STDOUT

Standard Output.

Normal command output.

Descriptor:

```text
1
```

---

Example:

```bash
echo "Hello"
```

Output:

```text
Hello
```

goes to:

```text
STDOUT
```

---

# STDERR

Standard Error.

Error messages.

Descriptor:

```text
2
```

---

Example:

```bash
cat file-does-not-exist
```

Output:

```text
No such file or directory
```

goes to:

```text
STDERR
```

---

# What is Redirection?

Redirection changes where output is sent.

Instead of displaying on screen:

```text
Store Output in File
```

---

# Redirect STDOUT

Syntax:

```bash
command > file.log
```

Equivalent:

```bash
command 1> file.log
```

---

# Example

```bash
date > output.log
```

Stores output in:

```text
output.log
```

---

# Overwrite Behavior

```bash
>
```

replaces existing content.

---

Example:

```bash
echo "Hello" > file.log
```

Again:

```bash
echo "World" > file.log
```

Result:

```text
World
```

Only latest content remains.

---

# Append Output

Use:

```bash
>>
```

---

Example:

```bash
echo "Hello" >> file.log
echo "World" >> file.log
```

Result:

```text
Hello
World
```

---

# Redirect STDERR

Syntax:

```bash
command 2> error.log
```

---

Example

```bash
cat missing-file.txt 2> error.log
```

Error stored in:

```text
error.log
```

---

# Redirect STDOUT and STDERR Separately

```bash
command 1>output.log 2>error.log
```

---

Example

```bash
ls existing-file missing-file \
1>output.log \
2>error.log
```

---

# Redirect Both Together

Syntax:

```bash
command &> output.log
```

---

Equivalent:

```bash
command > output.log 2>&1
```

---

Example

```bash
dnf install nginx -y &> install.log
```

Both success and error messages go into:

```text
install.log
```

---

# Production Logging Example

```bash
LOG_FILE=/var/log/shellscript-logs/install.log

dnf install nginx -y &>>$LOG_FILE
```

---

# Difference Between &> and &>>

| Operator | Action |
|------------|----------|
| &> | Overwrite |
| &>> | Append |

---

# Suppress Output

Sometimes output is unnecessary.

Use:

```bash
/dev/null
```

---

# What is /dev/null?

Special Linux device.

Anything sent here is discarded.

---

# Example

```bash
dnf install nginx -y > /dev/null
```

No output shown.

---

# Suppress Errors

```bash
command 2>/dev/null
```

---

# Suppress Everything

```bash
command >/dev/null 2>&1
```

or

```bash
command &>/dev/null
```

---

# Real DevOps Example

```bash
LOG_FILE=/var/log/shellscript-logs/backend.log

systemctl restart backend &>>$LOG_FILE
```

---

# Another Example

```bash
curl -s http://localhost:8080/health \
&>>$LOG_FILE
```

Store health-check results in logs.

---

# Logging Best Practices

## Always Create Logs

Avoid:

```text
No Logging
```

---

## Use Timestamps

Example:

```bash
DATE=$(date)
```

Include timestamps in logs.

---

## Separate Logs by Script

Good:

```text
backend.log
frontend.log
mysql.log
```

---

## Use Append Mode

Prefer:

```bash
>>
```

over:

```bash
>
```

to preserve history.

---

# Frequently Asked Interview Questions

## What is Logging?

Recording script execution details into files.

---

## Why Do We Need Logs?

For troubleshooting, auditing, and monitoring.

---

## Where Are Linux Logs Stored?

```text
/var/log
```

---

## What Are STDIN, STDOUT, and STDERR?

| Descriptor | Meaning |
|------------|----------|
| 0 | Input |
| 1 | Output |
| 2 | Error |

---

## Difference Between > and >> ?

| Operator | Action |
|-----------|---------|
| > | Overwrite |
| >> | Append |

---

## What Does 2> Mean?

Redirect error output.

---

## What Does &> Mean?

Redirect both output and errors.

---

## What is /dev/null?

A special device that discards data.

---

## Why Use Colors in Scripts?

To improve readability and quickly identify success, failure, and warning messages.

---

# Key Takeaways

- Logging is critical for troubleshooting and auditing.
- Linux logs are typically stored in `/var/log`.
- Colors improve script readability.
- Green commonly indicates success.
- Red commonly indicates failure.
- Yellow commonly indicates warnings or skipped operations.
- STDIN, STDOUT, and STDERR are Linux file descriptors.
- Redirection controls where output is sent.
- `>` overwrites files.
- `>>` appends to files.
- `2>` redirects errors.
- `&>` redirects both output and errors.
- `/dev/null` discards unwanted output.
- Logging and redirections are heavily used in production DevOps scripts.