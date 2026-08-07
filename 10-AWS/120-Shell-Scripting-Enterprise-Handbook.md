# Shell Scripting Enterprise Handbook

# Chapter 1 - Shell Scripting Fundamentals & Enterprise Architecture

Shell scripting is one of the most valuable skills for a DevOps Engineer.

Almost every DevOps tool relies on shell scripts for automation, including:

- Jenkins
- GitHub Actions
- Kubernetes
- Docker
- Terraform
- Ansible
- AWS CLI
- Azure CLI
- Google Cloud CLI

Instead of performing repetitive tasks manually, shell scripts automate them consistently and reliably.

---

# Why Shell Scripting?

A DevOps Engineer performs hundreds of repetitive tasks every day, such as:

- Deploying applications
- Restarting services
- Creating backups
- Rotating logs
- Monitoring servers
- Managing users
- Installing packages
- Validating deployments

Without automation:

```text
Administrator

↓

Manual Commands

↓

Human Errors

↓

Slow Operations
```

With shell scripting:

```text
Administrator

↓

Shell Script

↓

Automated Tasks

↓

Consistent Results
```

---

# What is a Shell?

A shell is a command-line interpreter that acts as an interface between the user and the Linux kernel.

```text
User

↓

Shell

↓

Kernel

↓

Hardware
```

The shell receives commands, interprets them, and requests the Linux kernel to execute them.

---

# What is a Shell Script?

A shell script is a text file containing a sequence of Linux commands executed automatically.

Instead of typing commands one by one:

```text
Command 1

↓

Command 2

↓

Command 3
```

A script executes them in order.

---

# Why DevOps Engineers Use Shell Scripts

Shell scripting is commonly used for:

- Infrastructure Automation
- Deployment Automation
- Health Checks
- Server Configuration
- Backup Automation
- Monitoring
- CI/CD Pipelines
- Kubernetes Administration

Automation reduces manual effort and improves consistency.

---

# Shell Architecture

```text
User

↓

Shell Script

↓

Shell Interpreter

↓

Linux Kernel

↓

Operating System
```

The shell interpreter processes each command before passing it to the kernel.

---

# Common Linux Shells

Linux supports multiple shells.

| Shell | Description |
|--------|-------------|
| Bash | Default shell on most Linux systems |
| sh | Original Unix shell |
| zsh | Extended interactive shell |
| ksh | Korn Shell |
| fish | User-friendly interactive shell |

For DevOps, **Bash** is the most widely used shell.

---

# Bash (Bourne Again Shell)

Bash is the default shell on most enterprise Linux distributions.

Features include:

- Variables
- Functions
- Loops
- Conditional Statements
- Arrays
- Arithmetic Operations
- Command Substitution

Most CI/CD pipelines execute Bash scripts.

---

# Verify Your Current Shell

Display the current shell:

```bash
echo $SHELL
```

Example output:

```text
/bin/bash
```

---

# Check Bash Version

```bash
bash --version
```

Knowing the Bash version is important because some features are version-specific.

---

# Script File Extension

Shell scripts commonly use the `.sh` extension.

Examples:

```text
backup.sh

deploy.sh

health-check.sh

cleanup.sh
```

The extension is a convention and improves readability.

---

# Creating Your First Script

Create a new script:

```bash
touch hello.sh
```

Open it with your preferred editor:

```bash
vi hello.sh
```

or

```bash
nano hello.sh
```

---

# Script Structure

A basic shell script contains:

```bash
#!/bin/bash

echo "Hello, DevOps!"
```

The first line is called the **Shebang**.

---

# What is Shebang?

The Shebang tells Linux which interpreter should execute the script.

Example:

```bash
#!/bin/bash
```

Other examples:

```bash
#!/bin/sh

#!/usr/bin/env bash
```

Without a valid interpreter, the script may not execute correctly.

---

# Execute Permission

A script must have execute permission before it can run.

Grant execute permission:

```bash
chmod +x hello.sh
```

Verify permissions:

```bash
ls -l hello.sh
```

---

# Running a Script

Execute the script directly:

```bash
./hello.sh
```

Or run it using Bash:

```bash
bash hello.sh
```

Both methods execute the script, but the first requires execute permission.

---

# Comments

Comments improve script readability.

Single-line comment:

```bash
# Install Docker
```

Use comments to explain:

- Complex logic
- Configuration
- Assumptions
- Important warnings

---

# Exit Status

Every Linux command returns an exit status.

```text
0

↓

Success

────────────

Non-zero

↓

Failure
```

Scripts use exit codes to determine whether commands succeeded.

---

# Check Exit Status

Display the exit status of the previous command:

```bash
echo $?
```

Example:

```text
0
```

This indicates successful execution.

---

# Script Execution Flow

```text
Start

↓

Read Script

↓

Execute Commands

↓

Return Exit Code

↓

End
```

If a command fails, the script can decide how to proceed.

---

# Common Enterprise Scripts

Examples include:

- Backup Scripts
- Deployment Scripts
- Health Check Scripts
- User Provisioning Scripts
- Log Cleanup Scripts
- Disk Monitoring Scripts
- Kubernetes Automation Scripts
- AWS Automation Scripts

---

# Enterprise Example

A deployment pipeline might execute:

```text
GitHub

↓

Jenkins

↓

Build

↓

Shell Script

↓

Docker Build

↓

Push Image

↓

Update GitOps Repository
```

The shell script coordinates multiple automation steps.

---

# Kubernetes Example

A health-check script:

```text
Check Pods

↓

Check Nodes

↓

Check Services

↓

Generate Report
```

This script can run automatically through Cron or a CI/CD pipeline.

---

# Enterprise Best Practices

- Use Bash for Linux automation.
- Always include a Shebang.
- Write meaningful comments.
- Check exit codes after critical commands.
- Keep scripts modular and reusable.
- Store scripts in version control.
- Test scripts in non-production environments first.
- Use descriptive file names.

---

# Common Mistakes

- Forgetting the execute permission.
- Omitting the Shebang line.
- Hardcoding environment-specific values.
- Ignoring exit codes.
- Writing long scripts without comments.
- Running untested scripts in production.
- Mixing unrelated tasks into one script.

---

# Interview Questions

## Basic

1. What is a shell?
2. What is Bash?
3. What is a shell script?
4. What is a Shebang?
5. How do you execute a shell script?

## Intermediate

1. Difference between `bash script.sh` and `./script.sh`.
2. Why are exit codes important?
3. How do you make a script executable?
4. Explain the purpose of comments in scripts.
5. What are common enterprise uses of shell scripting?

## Advanced

1. Design an automation workflow using shell scripts for deploying Docker containers to Amazon EKS through a CI/CD pipeline.
2. Explain how shell scripting integrates with Jenkins, GitHub Actions, Terraform, Kubernetes, and AWS to automate enterprise infrastructure.
3. A financial organization manages hundreds of Linux servers. Design a shell scripting strategy for backups, monitoring, deployments, health checks, log management, and operational automation while ensuring maintainability and reliability.

---

