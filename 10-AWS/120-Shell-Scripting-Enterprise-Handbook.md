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

# Chapter 2 - Variables, User Input, Data Types & Operators

Variables are the foundation of every shell script.

Almost every automation script uses variables to store:

- File Names
- User Names
- Server Names
- IP Addresses
- Environment Names
- Database Names
- AWS Resources

Without variables, scripts become difficult to maintain and reuse.

---

# What is a Variable?

A variable stores a value that can be used throughout a script.

Instead of writing the same value repeatedly:

```text
Production

Production

Production
```

Store it once:

```bash
ENV=Production
```

Now the script can reference the variable whenever needed.

---

# Declaring Variables

Variables are assigned using the following syntax:

```bash
NAME="Surendra"

AGE=28

CITY="Hyderabad"
```

**Note:** There should be **no spaces** around the `=` operator.

Correct:

```bash
NAME="DevOps"
```

Incorrect:

```bash
NAME = "DevOps"
```

---

# Accessing Variables

Use the `$` symbol to access a variable.

Example:

```bash
NAME="DevOps"

echo $NAME
```

Output:

```text
DevOps
```

Variables can be reused throughout the script.

---

# Using Curly Braces

Variables can also be referenced using curly braces.

Example:

```bash
echo ${NAME}
```

This is useful when combining variables with text.

Example:

```bash
PROJECT="payment"

echo "${PROJECT}_service"
```

Output:

```text
payment_service
```

---

# Variable Naming Rules

Variable names should:

- Start with a letter or underscore
- Contain letters, numbers, and underscores
- Avoid spaces
- Be descriptive

Examples:

Correct:

```bash
SERVER_NAME

APP_VERSION

DB_HOST
```

Incorrect:

```bash
2SERVER

SERVER-NAME

APP VERSION
```

---

# Environment Variables

Environment variables are available system-wide.

View all environment variables:

```bash
printenv
```

or

```bash
env
```

Common environment variables include:

```bash
HOME

PATH

USER

HOSTNAME

SHELL

PWD
```

---

# Display Environment Variables

Examples:

```bash
echo $HOME

echo $USER

echo $HOSTNAME

echo $PATH
```

These variables are frequently used in automation scripts.

---

# Local Variables vs Environment Variables

| Local Variable | Environment Variable |
|----------------|----------------------|
| Available only within the current shell | Available to child processes |
| Created inside a script | Exported using `export` |
| Temporary | Shared with subprocesses |

---

# Export Variables

Make a variable available to child processes.

Example:

```bash
export ENV=Production
```

Verify:

```bash
echo $ENV
```

---

# Read User Input

Accept user input during script execution.

Example:

```bash
read NAME

echo "Hello $NAME"
```

The script waits until the user enters a value.

---

# Prompt User Input

Provide a prompt while reading input.

```bash
read -p "Enter your name: " NAME

echo "Welcome $NAME"
```

This improves the user experience.

---

# Secure Input

Hide sensitive input such as passwords.

```bash
read -s PASSWORD
```

Example:

```bash
read -sp "Enter Password: " PASSWORD
```

Characters are not displayed while typing.

---

# Command Substitution

Store command output in a variable.

Modern syntax:

```bash
DATE=$(date)
```

Older syntax:

```bash
DATE=`date`
```

The `$(...)` format is recommended.

---

# Practical Examples

Store the hostname:

```bash
HOST=$(hostname)
```

Store the current directory:

```bash
DIR=$(pwd)
```

Store the current date:

```bash
TODAY=$(date)
```

Command substitution is widely used in automation.

---

# Special Shell Variables

Shell provides predefined variables.

| Variable | Description |
|----------|-------------|
| `$0` | Script name |
| `$1` | First argument |
| `$2` | Second argument |
| `$#` | Number of arguments |
| `$@` | All arguments |
| `$*` | All arguments as one string |
| `$$` | Current process ID |
| `$?` | Exit status of previous command |

---

# Positional Parameters

Example:

```bash
./deploy.sh production v1.2
```

Inside the script:

```bash
echo $1
```

Output:

```text
production
```

```bash
echo $2
```

Output:

```text
v1.2
```

---

# Arithmetic Operations

Use double parentheses for arithmetic.

Example:

```bash
A=10

B=20

TOTAL=$((A+B))

echo $TOTAL
```

Output:

```text
30
```

---

# Common Arithmetic Operators

| Operator | Description |
|----------|-------------|
| `+` | Addition |
| `-` | Subtraction |
| `*` | Multiplication |
| `/` | Division |
| `%` | Modulus |

---

# String Operations

Example:

```bash
FIRST="Dev"

SECOND="Ops"

echo "$FIRST$SECOND"
```

Output:

```text
DevOps
```

Determine string length:

```bash
echo ${#FIRST}
```

---

# Read-Only Variables

Prevent variables from being modified.

Example:

```bash
readonly ENV=Production
```

Attempting to change the value results in an error.

---

# Unset Variables

Remove a variable.

Example:

```bash
unset ENV
```

The variable no longer exists.

---

# Quoting

Shell supports three types of quoting.

### Double Quotes

Variables are expanded.

```bash
NAME="DevOps"

echo "$NAME"
```

Output:

```text
DevOps
```

---

### Single Quotes

Variables are not expanded.

```bash
echo '$NAME'
```

Output:

```text
$NAME
```

---

### Backslash

Escapes special characters.

Example:

```bash
echo \$HOME
```

Output:

```text
$HOME
```

---

# Enterprise Example

Deployment script:

```bash
APP="payment"

VERSION="1.4.2"

NAMESPACE="production"

echo "Deploying $APP version $VERSION to $NAMESPACE"
```

Using variables makes scripts reusable across environments.

---

# Kubernetes Example

```bash
NAMESPACE=production

kubectl get pods -n $NAMESPACE
```

The same script can be used for development, staging, and production by changing only the namespace variable.

---

# Enterprise Best Practices

- Use meaningful variable names.
- Store repeated values in variables.
- Prefer environment variables for configurable settings.
- Use `$(...)` instead of backticks for command substitution.
- Quote variables to avoid unexpected behavior.
- Use `readonly` for constants.
- Avoid hardcoding environment-specific values.
- Validate user input before using it.

---

# Common Mistakes

- Adding spaces around `=`.
- Using unclear variable names.
- Forgetting the `$` while accessing variables.
- Hardcoding server names or credentials.
- Ignoring quotation marks around variables containing spaces.
- Using backticks instead of `$(...)`.
- Not checking user input before processing.

---

# Interview Questions

## Basic

1. What is a variable in shell scripting?
2. How do you declare and access variables?
3. What is the difference between local and environment variables?
4. What does the `read` command do?
5. What is command substitution?

## Intermediate

1. Explain the purpose of `export`.
2. Difference between `$@` and `$*`.
3. Explain positional parameters.
4. What is the difference between single and double quotes?
5. How do you perform arithmetic operations in Bash?

## Advanced

1. Design a reusable deployment script using variables, environment variables, user input, and command substitution for deploying applications across development, staging, and production environments.
2. Explain how variables and command substitution improve maintainability and reusability in enterprise automation scripts.
3. A CI/CD pipeline deploys applications to multiple Kubernetes namespaces. Design a shell scripting strategy using variables, environment variables, script arguments, and secure user input to support multiple environments while avoiding hardcoded configuration.

---

