# Shell Scripting Error Handling

## Introduction

A script should not blindly execute commands.

In production environments, failures happen frequently:

- Package installation failures
- Network issues
- Permission problems
- Authentication failures
- Missing files
- Invalid inputs

A good script should identify failures and take appropriate action.

This process is called:

```text
Error Handling
```

---

# What is Error Handling?

Error handling is the process of:

```text
Detecting Errors
Handling Errors
Taking Correct Action
```

instead of allowing the script to continue blindly.

---

# Why Error Handling?

Without error handling:

```text
Command Fails
      │
      ▼
Script Continues
      │
      ▼
More Failures
```

This can create bigger problems.

---

# Real World Example

Suppose:

```bash
dnf install mysql -y
```

fails.

If the script still executes:

```bash
systemctl start mysqld
```

it will fail because MySQL is not installed.

---

# Main Question

Whenever an error occurs:

```text
Can We Continue?

OR

Should We Stop?
```

This is the core idea behind error handling.

---

# Types of Errors

There are two major categories.

---

# Expected Errors

Errors that we know may occur.

Examples:

- Package already installed
- File already exists
- User already exists
- DNS record already exists

These should be handled properly.

---

# Example

Create User:

```bash
useradd expense
```

Possible Result:

```text
user already exists
```

This is expected.

---

# Handling Expected Errors

Instead of failing:

```text
Check First
Then Perform Action
```

Example:

```bash
id expense
```

If user exists:

```text
Skip Creation
```

---

# Unexpected Errors

Errors we do not anticipate.

Examples:

```text
Disk Full
Memory Issue
OS Corruption
Network Failure
AWS Outage
```

These usually require investigation.

---

# Example

```bash
dnf install mysql -y
```

Suddenly:

```text
No Internet Connection
```

This is an unexpected error.

---

# Exit Status

Every Linux command returns an exit code.

Example:

```bash
pwd
```

Check status:

```bash
echo $?
```

---

# Meaning of Exit Codes

```text
0       → Success

1-127   → Failure
```

---

# Successful Example

```bash
ls
```

Check:

```bash
echo $?
```

Output:

```text
0
```

---

# Failed Example

```bash
abcd
```

Check:

```bash
echo $?
```

Output:

```text
127
```

Meaning:

```text
Command Not Found
```

---

# Why Exit Status Matters?

Scripts should decide what to do based on:

```text
Success

OR

Failure
```

---

# Basic Error Handling

Example:

```bash
dnf install mysql -y

if [ $? -eq 0 ]; then
    echo "Installation Successful"
else
    echo "Installation Failed"
fi
```

---

# Production Approach

Algorithm:

```text
Execute Command

Check Exit Status

If Success
    Continue

Else
    Stop Script
```

---

# Example

```bash
dnf install nginx -y

if [ $? -ne 0 ]; then
    echo "Installation Failed"
    exit 1
fi
```

---

# What is exit 1?

```bash
exit 1
```

means:

```text
Stop Script
Return Failure
```

---

# Common Validation Checks

Before executing operations, validate prerequisites.

---

# Root Access Validation

Many Linux operations require root access.

Examples:

```bash
dnf install
useradd
systemctl restart
```

---

# Validation

```bash
id -u
```

Output:

```text
0
```

means:

```text
Root User
```

---

# Root Check

```bash
if [ $(id -u) -ne 0 ]; then
    echo "Please run as root"
    exit 1
fi
```

---

# Why Validate Root?

Without root:

```text
Package Installation Fails
User Creation Fails
Service Operations Fail
```

---

# AWS Authentication Example

Before creating AWS resources:

```text
Authenticate First
```

---

# Configure AWS CLI

```bash
aws configure
```

---

# Verify Authentication

```bash
aws s3 ls
```

---

# Understanding Result

No output:

```text
Okay
```

Maybe bucket list is empty.

---

# Error Output

Example:

```text
Unable to locate credentials
```

or

```text
Access Denied
```

Means:

```text
Authentication Failed
```

---

# Validation Logic

```text
Run aws s3 ls

If Success
    Continue

Else
    Stop
```

---

# Example Script

```bash
aws s3 ls >/dev/null

if [ $? -ne 0 ]; then
    echo "AWS Authentication Failed"
    exit 1
fi
```

---

# Package Installation Validation

Example:

```bash
dnf install mysql -y
```

Check:

```bash
if [ $? -eq 0 ]; then
    echo "Installed Successfully"
else
    echo "Installation Failed"
    exit 1
fi
```

---

# Function-Based Error Handling

Instead of repeating logic.

Create reusable function.

---

# Example

```bash
VALIDATE(){

    if [ $1 -ne 0 ]; then
        echo "$2 Failed"
        exit 1
    else
        echo "$2 Success"
    fi

}
```

---

# Usage

```bash
dnf install nginx -y

VALIDATE $? "Nginx Installation"
```

---

# Why Functions Help?

Without function:

```text
Repeated Validation Code
```

With function:

```text
Reusable Validation Logic
```

---

# Error Handling Flow

```text
Execute Command
       │
       ▼
Check Exit Status
       │
       ▼
Success?
  │         │
 Yes        No
  │         │
Continue   Stop
```

---

# Real DevOps Examples

## Package Installation

```bash
dnf install nginx -y
```

Validate installation.

---

## User Creation

```bash
useradd expense
```

Validate user creation.

---

## Service Startup

```bash
systemctl start nginx
```

Validate startup.

---

## Database Connectivity

```bash
mysql -h DB_HOST
```

Validate connection.

---

## AWS Authentication

```bash
aws s3 ls
```

Validate credentials.

---

# Common Mistakes

## Ignoring Errors

Bad:

```bash
dnf install mysql -y

systemctl start mysqld
```

No validation.

---

## Continuing After Failure

Bad:

```text
Failure Occurred

Still Continue
```

---

## No Exit Codes

Scripts should clearly return success or failure.

---

# Best Practices

## Validate Critical Operations

Examples:

- Package Installation
- Database Connection
- Service Startup
- AWS Authentication

---

## Stop on Critical Failures

Do not continue when required steps fail.

---

## Handle Expected Errors

Example:

```text
User Already Exists
```

should not crash the script.

---

## Use Reusable Functions

Avoid duplicating validation logic.

---

# Frequently Asked Interview Questions

## What is Error Handling?

Detecting and responding to failures in scripts.

---

## Why Is Error Handling Important?

To prevent scripts from continuing after failures.

---

## What is an Expected Error?

An anticipated error such as:

```text
User Already Exists
```

---

## What is an Unexpected Error?

An unplanned failure such as:

```text
Network Failure
Disk Full
```

---

## What Does Exit Code 0 Mean?

```text
Success
```

---

## What Does Non-Zero Exit Code Mean?

```text
Failure
```

---

## How Do You Check Previous Command Status?

```bash
echo $?
```

---

## How Do You Stop a Script?

```bash
exit 1
```

---

## How Do You Validate AWS Authentication?

```bash
aws s3 ls
```

---

# Key Takeaways

- Error handling is critical in production automation.
- Every Linux command returns an exit status.
- Exit code `0` means success.
- Non-zero exit codes indicate failures.
- Expected and unexpected errors should be treated differently.
- Validate important operations before continuing.
- AWS authentication should be verified before resource creation.
- Functions help centralize error handling logic.
- Good error handling makes scripts reliable and production-ready.