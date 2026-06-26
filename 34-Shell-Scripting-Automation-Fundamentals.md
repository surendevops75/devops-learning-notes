# Shell Scripting and Automation Fundamentals

## Introduction

In real-world environments, DevOps Engineers perform many repetitive tasks.

Examples:

- Creating Servers
- Installing Software
- Configuring Applications
- Deploying Code
- Restarting Services
- Monitoring Applications

Performing these tasks manually every time is inefficient.

Automation helps execute tasks consistently and quickly.

---

# Manual Process

Consider application deployment.

Steps:

```text
Create Server
      │
      ▼
Install Programming Language
      │
      ▼
Create System User
      │
      ▼
Create Application Directory
      │
      ▼
Download Application Code
      │
      ▼
Extract Application Files
      │
      ▼
Install Dependencies
      │
      ▼
Create Systemd Service
      │
      ▼
Reload Systemd
      │
      ▼
Start Service
```

All these steps are Linux commands executed one after another.

---

# Problems with Manual Process

## Too Much Time

Every server requires the same steps.

Example:

```text
Server-1
Server-2
Server-3
Server-4
```

Repeating commands manually consumes time.

---

## Repeated Work

The same process is performed repeatedly.

Example:

```text
Install NodeJS
Install Python
Install Java
```

every time a new server is created.

---

## Human Errors

Example:

```bash
dnf install nginx -y
```

Correct.

But:

```bash
dnf install nginxxx -y
```

Incorrect.

Small mistakes can cause failures.

---

## Difficult Troubleshooting

Manual execution makes it difficult to identify where the failure occurred.

---

## Continuous Network Connection Required

If the session disconnects during execution:

```text
Deployment Stops
```

---

# What is Automation?

Automation means:

```text
Execute Tasks Automatically
```

without manually running every command.

---

# What is Shell Scripting?

A Shell Script is a file containing Linux commands executed sequentially.

Instead of:

```bash
command1
command2
command3
command4
```

running manually,

store them in a file:

```bash
deploy.sh
```

and execute once.

---

# Real World Example

Suppose you have:

```text
100 Log Files
```

Option 1:

```text
Carry One Log at a Time
```

Option 2:

```text
Put All Logs in a Cart
Move Together
```

Shell scripting works like the second approach.

---

# Why Shell Scripting?

Benefits:

- Faster Execution
- Consistency
- Reusability
- Less Human Error
- Easier Troubleshooting

---

# Deployment Process Example

Manual:

```bash
rm -rf /app/*
curl -o /tmp/app.zip URL
unzip /tmp/app.zip
systemctl restart app
```

Automation:

```bash
./deploy.sh
```

---

# Deployment Activities

## Remove Old Code

```bash
rm -rf /app/*
```

---

## Download New Code

```bash
curl -o /tmp/app.zip URL
```

---

## Extract Application

```bash
unzip /tmp/app.zip
```

---

## Restart Service

```bash
systemctl restart app
```

---

# Structure of a Shell Script

Example:

```bash
#!/bin/bash

echo "Hello World"
```

---

# Shebang

First line:

```bash
#!/bin/bash
```

tells Linux:

```text
Execute using Bash Shell
```

---

# Variables

Variables store data.

Example:

```bash
NAME="Ramesh"
```

Use:

```bash
echo $NAME
```

Output:

```text
Ramesh
```

---

# Why Variables?

Avoid hardcoding values.

Example:

```bash
APP_NAME="catalogue"
```

instead of repeatedly writing:

```text
catalogue
```

---

# Data Types

Shell scripting commonly works with:

```text
Strings
Numbers
Arrays
```

Examples:

```bash
NAME="DevOps"
AGE=25
```

---

# Conditions

Conditions allow decision-making.

Example:

```bash
if [ $? -eq 0 ]
then
    echo "Success"
else
    echo "Failure"
fi
```

---

# Why Conditions?

Example:

```bash
Install Package
```

If installation succeeds:

```text
Proceed
```

Else:

```text
Stop
```

---

# Example

```bash
dnf install nginx -y
```

If successful:

```text
Continue Deployment
```

Otherwise:

```text
Exit Script
```

---

# Loops

Loops execute tasks repeatedly.

Example:

```bash
for i in 1 2 3 4 5
do
    echo $i
done
```

Output:

```text
1
2
3
4
5
```

---

# Why Loops?

Example:

```text
100 Servers
```

Instead of writing commands 100 times:

```bash
for SERVER in server1 server2 server3
do
    echo $SERVER
done
```

---

# Functions

Functions group reusable logic.

Example:

```bash
greet() {
    echo "Hello"
}
```

Call:

```bash
greet
```

---

# Why Functions?

Without Functions:

```text
Duplicate Code
```

With Functions:

```text
Reusable Logic
```

---

# Error Handling

One of the most important concepts in automation.

Goal:

```text
If Success → Continue

If Failure → Stop
```

---

# Exit Status

Every Linux command returns an exit code.

```text
0  → Success
Non-Zero → Failure
```

Example:

```bash
echo $?
```

---

# Checking Command Status

```bash
dnf install nginx -y

if [ $? -eq 0 ]
then
    echo "Installation Successful"
else
    echo "Installation Failed"
fi
```

---

# Common Automation Workflow

```text
Create Server
      │
      ▼
Install Software
      │
      ▼
Configure Application
      │
      ▼
Deploy Code
      │
      ▼
Start Service
      │
      ▼
Verify Health
```

---

# Shell Script Example

```bash
#!/bin/bash

dnf install nginx -y

if [ $? -ne 0 ]
then
    echo "Installation Failed"
    exit 1
fi

systemctl start nginx

echo "Deployment Completed"
```

---

# Common DevOps Use Cases

## Application Deployment

```text
Deploy Applications Automatically
```

---

## Server Configuration

```text
Install Packages
Configure Services
```

---

## Log Management

```text
Collect Logs
Archive Logs
Cleanup Logs
```

---

## Backup Operations

```text
Database Backups
File Backups
```

---

## Health Checks

```text
Verify Services
Verify Ports
Verify APIs
```

---

# Advantages of Automation

- Faster Deployments
- Reduced Errors
- Better Consistency
- Improved Productivity
- Easier Maintenance
- Repeatable Processes

---

# Frequently Asked Interview Questions

## What is Shell Scripting?

A method of automating Linux commands using scripts.

---

## Why Use Shell Scripts?

To automate repetitive tasks and reduce manual effort.

---

## What is a Variable?

A container used to store data.

---

## What is a Condition?

A decision-making statement used to control script flow.

---

## What is a Loop?

A mechanism to repeat tasks multiple times.

---

## What is a Function?

A reusable block of code.

---

## What is Error Handling?

Detecting failures and responding appropriately.

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

# Key Takeaways

- Manual processes are slow and error-prone.
- Automation improves consistency and speed.
- Shell scripts allow Linux commands to run automatically.
- Variables store reusable values.
- Conditions help make decisions.
- Loops automate repetitive tasks.
- Functions promote code reuse.
- Error handling improves reliability.
- Shell scripting is one of the foundational skills for DevOps Engineers...