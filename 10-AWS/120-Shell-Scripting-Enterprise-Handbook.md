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

# Chapter 3 - Conditional Statements & Decision Making

Conditional statements allow a shell script to make decisions based on specific conditions.

Instead of executing every command sequentially, a script can evaluate a condition and choose the appropriate action.

Conditional statements are heavily used in:

- Deployment Automation
- Health Checks
- Backup Scripts
- Monitoring Scripts
- CI/CD Pipelines
- Kubernetes Automation
- AWS Automation

Without conditional logic, automation scripts cannot respond intelligently to changing situations.

---

# Why Use Conditional Statements?

Consider a deployment script.

Without conditions:

```text
Deploy Application

↓

Restart Service

↓

Send Notification
```

Even if the deployment fails, the script continues executing.

With conditional logic:

```text
Deploy Application

↓

Success?

↓

Yes → Restart Service

↓

No → Exit & Notify Team
```

This makes automation safer and more reliable.

---

# The if Statement

The simplest decision-making statement is `if`.

Syntax:

```bash
if [ condition ]
then
    commands
fi
```

If the condition evaluates to **true**, the commands inside the block are executed.

---

# Example - Numeric Comparison

```bash
AGE=20

if [ $AGE -ge 18 ]
then
    echo "Eligible to Vote"
fi
```

Output:

```text
Eligible to Vote
```

---

# if...else Statement

Execute one block if the condition is true and another if it is false.

Syntax:

```bash
if [ condition ]
then
    commands
else
    commands
fi
```

Example:

```bash
STATUS="FAILED"

if [ "$STATUS" = "SUCCESS" ]
then
    echo "Deployment Successful"
else
    echo "Deployment Failed"
fi
```

---

# if...elif...else Statement

Use multiple conditions.

Syntax:

```bash
if [ condition ]
then
    commands
elif [ condition ]
then
    commands
else
    commands
fi
```

Example:

```bash
ENV="stage"

if [ "$ENV" = "dev" ]
then
    echo "Development"
elif [ "$ENV" = "stage" ]
then
    echo "Staging"
else
    echo "Production"
fi
```

---

# Test Command

Linux provides the `test` command to evaluate conditions.

Example:

```bash
test 10 -gt 5
```

Equivalent syntax:

```bash
[ 10 -gt 5 ]
```

Both forms produce the same result.

---

# Numeric Comparison Operators

| Operator | Meaning |
|----------|---------|
| `-eq` | Equal |
| `-ne` | Not Equal |
| `-gt` | Greater Than |
| `-lt` | Less Than |
| `-ge` | Greater Than or Equal |
| `-le` | Less Than or Equal |

Example:

```bash
if [ $A -gt $B ]
then
    echo "A is greater"
fi
```

---

# String Comparison Operators

| Operator | Meaning |
|----------|---------|
| `=` | Equal |
| `!=` | Not Equal |
| `-z` | Empty String |
| `-n` | Non-empty String |

Example:

```bash
NAME="DevOps"

if [ "$NAME" = "DevOps" ]
then
    echo "Matched"
fi
```

---

# File Test Operators

Shell scripts frequently check files before performing operations.

| Operator | Description |
|----------|-------------|
| `-f` | Regular file exists |
| `-d` | Directory exists |
| `-e` | File or directory exists |
| `-r` | Readable |
| `-w` | Writable |
| `-x` | Executable |
| `-s` | File is not empty |

Example:

```bash
if [ -f backup.tar ]
then
    echo "Backup exists"
fi
```

---

# Directory Check

Example:

```bash
if [ -d /var/log ]
then
    echo "Directory exists"
else
    echo "Directory missing"
fi
```

---

# Logical Operators

Combine multiple conditions.

| Operator | Meaning |
|----------|---------|
| `&&` | AND |
| `||` | OR |
| `!` | NOT |

Example:

```bash
if [ $AGE -ge 18 ] && [ $AGE -le 60 ]
then
    echo "Eligible"
fi
```

---

# Nested if Statements

Conditions can be placed inside other conditions.

Example:

```bash
if [ "$ENV" = "production" ]
then
    if [ "$STATUS" = "SUCCESS" ]
    then
        echo "Deployment Completed"
    fi
fi
```

Nested conditions are useful for complex workflows.

---

# case Statement

Use `case` when multiple values need to be matched.

Syntax:

```bash
case $VALUE in
    option1)
        commands
        ;;
    option2)
        commands
        ;;
    *)
        default
        ;;
esac
```

---

# Example - Environment Selection

```bash
ENV="prod"

case $ENV in
    dev)
        echo "Development"
        ;;
    test)
        echo "Testing"
        ;;
    prod)
        echo "Production"
        ;;
    *)
        echo "Unknown Environment"
        ;;
esac
```

`case` statements are cleaner than multiple `if...elif` blocks.

---

# Exit on Failure

Many enterprise scripts stop immediately when a critical command fails.

Example:

```bash
cp file.txt /backup

if [ $? -ne 0 ]
then
    echo "Backup Failed"
    exit 1
fi
```

This prevents later commands from executing after an error.

---

# Using Exit Codes in Conditions

Example:

```bash
systemctl status nginx

if [ $? -eq 0 ]
then
    echo "Nginx is Running"
else
    echo "Nginx is Down"
fi
```

Exit codes are widely used in automation scripts.

---

# Kubernetes Example

Check whether a namespace exists.

```bash
kubectl get namespace production >/dev/null 2>&1

if [ $? -eq 0 ]
then
    echo "Namespace Found"
else
    echo "Namespace Missing"
fi
```

---

# AWS Example

Verify whether an EC2 instance exists.

```bash
INSTANCE=$(aws ec2 describe-instances ...)

if [ -n "$INSTANCE" ]
then
    echo "Instance Found"
else
    echo "Instance Not Found"
fi
```

---

# CI/CD Example

A deployment pipeline should stop when tests fail.

```text
Build

↓

Run Tests

↓

Tests Passed?

↓

Yes → Deploy

↓

No → Exit Pipeline
```

Conditional logic protects production environments.

---

# Enterprise Best Practices

- Always validate user input.
- Quote string variables inside conditions.
- Check exit codes after critical commands.
- Use `case` for multiple fixed values.
- Keep conditions simple and readable.
- Exit immediately after critical failures.
- Use meaningful condition names.
- Test scripts with both success and failure scenarios.

---

# Common Mistakes

- Forgetting spaces inside `[ ]`.
- Comparing strings without quotes.
- Ignoring command exit codes.
- Writing deeply nested conditions that are difficult to read.
- Using multiple `if...elif` blocks where `case` is more appropriate.
- Continuing execution after critical failures.
- Hardcoding environment names.

---

# Interview Questions

## Basic

1. What is an `if` statement?
2. What is the difference between `if` and `if...else`?
3. What is the purpose of the `test` command?
4. Explain numeric comparison operators.
5. Explain string comparison operators.

## Intermediate

1. Difference between `if...elif` and `case`.
2. Explain file test operators.
3. How do you check whether a directory exists?
4. Explain logical operators in shell scripting.
5. Why should scripts check exit codes?

## Advanced

1. Design a deployment script that validates input, checks service status, verifies Kubernetes namespaces, and exits safely on failures using conditional statements.
2. Explain how conditional statements improve reliability in enterprise automation scripts used by Jenkins, GitHub Actions, and Kubernetes.
3. A CI/CD pipeline deploys applications to production. Design a shell script using `if`, `case`, file checks, logical operators, and exit codes to validate deployment prerequisites, handle failures gracefully, and prevent accidental production deployments.

---

# Chapter 4 - Loops, Iteration & Automation

One of the biggest advantages of shell scripting is the ability to perform repetitive tasks automatically.

Instead of executing the same command multiple times, loops allow a script to execute a block of code repeatedly.

Loops are widely used in:

- Server Automation
- User Management
- Backup Scripts
- Log Processing
- Kubernetes Administration
- AWS Automation
- CI/CD Pipelines

Understanding loops is essential for writing efficient automation scripts.

---

# Why Use Loops?

Without loops:

```text
Backup Server 1

Backup Server 2

Backup Server 3

Backup Server 4

Backup Server 5
```

Each command must be written manually.

With loops:

```text
Server List

↓

Loop

↓

Backup Every Server
```

The same logic is reused for every server.

---

# Types of Loops

Bash provides three primary looping mechanisms.

- for Loop
- while Loop
- until Loop

Each loop is suitable for different automation scenarios.

---

# The for Loop

The `for` loop executes a block of code for every item in a list.

Syntax:

```bash
for VARIABLE in LIST
do
    commands
done
```

---

# Example - Simple for Loop

```bash
for NAME in DevOps Docker Kubernetes
do
    echo $NAME
done
```

Output:

```text
DevOps

Docker

Kubernetes
```

---

# Numeric for Loop

Loop through a sequence of numbers.

```bash
for i in {1..5}
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

# Increment Values

Specify a step value.

```bash
for i in {10..50..10}
do
    echo $i
done
```

Output:

```text
10

20

30

40

50
```

---

# C-Style for Loop

Similar to programming languages like C or Java.

```bash
for ((i=1;i<=5;i++))
do
    echo $i
done
```

Useful when complex conditions are required.

---

# Loop Through Files

Process every file in a directory.

```bash
for FILE in *.log
do
    echo $FILE
done
```

This is commonly used for log management.

---

# Loop Through Directories

```bash
for DIR in /home/*
do
    echo $DIR
done
```

Useful for administration scripts.

---

# The while Loop

The `while` loop executes as long as a condition is true.

Syntax:

```bash
while [ condition ]
do
    commands
done
```

---

# Example - while Loop

```bash
COUNT=1

while [ $COUNT -le 5 ]
do
    echo $COUNT
    COUNT=$((COUNT+1))
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

# Read File Line by Line

One of the most common enterprise use cases.

```bash
while read SERVER
do
    echo $SERVER
done < servers.txt
```

Example `servers.txt`

```text
server1

server2

server3
```

This technique is widely used in infrastructure automation.

---

# Infinite Loop

```bash
while true
do
    echo "Running..."
    sleep 5
done
```

Useful for continuous monitoring scripts.

Always provide a way to terminate infinite loops safely.

---

# The until Loop

The `until` loop executes until a condition becomes true.

Syntax:

```bash
until [ condition ]
do
    commands
done
```

---

# Example - until Loop

```bash
COUNT=1

until [ $COUNT -gt 5 ]
do
    echo $COUNT
    COUNT=$((COUNT+1))
done
```

The loop stops when the condition evaluates to true.

---

# break Statement

The `break` statement immediately exits the loop.

Example:

```bash
for i in {1..10}
do
    if [ $i -eq 5 ]
    then
        break
    fi

    echo $i
done
```

Output:

```text
1

2

3

4
```

---

# continue Statement

The `continue` statement skips the current iteration.

Example:

```bash
for i in {1..5}
do
    if [ $i -eq 3 ]
    then
        continue
    fi

    echo $i
done
```

Output:

```text
1

2

4

5
```

---

# Nested Loops

Loops can be placed inside other loops.

Example:

```bash
for ENV in dev stage prod
do
    for APP in payment order inventory
    do
        echo "$APP -> $ENV"
    done
done
```

Useful for deploying multiple applications across environments.

---

# Processing Command Output

Loop through command results.

```bash
for USER in $(cut -d: -f1 /etc/passwd)
do
    echo $USER
done
```

This processes every user account on the system.

---

# Kubernetes Example

Restart multiple deployments.

```bash
for APP in payment order inventory
do
    kubectl rollout restart deployment/$APP
done
```

Instead of restarting each deployment manually, the loop automates the process.

---

# AWS Example

Stop multiple EC2 instances.

```bash
for ID in i-123 i-456 i-789
do
    aws ec2 stop-instances --instance-ids $ID
done
```

Loops simplify cloud administration.

---

# CI/CD Example

Deploy to multiple environments.

```text
Build

↓

for Environment

↓

Deploy

↓

Run Tests

↓

Next Environment
```

This approach reduces duplicate pipeline logic.

---

# Enterprise Example

Perform health checks on multiple servers.

```bash
while read SERVER
do
    ping -c 2 $SERVER
done < servers.txt
```

The script checks connectivity for every server listed in the input file.

---

# Performance Considerations

When working with large datasets:

- Avoid unnecessary nested loops.
- Read files efficiently.
- Minimize external command execution inside loops.
- Exit loops early when possible.
- Reuse variables where appropriate.

Efficient loops improve script performance.

---

# Enterprise Best Practices

- Use `for` loops for fixed lists.
- Use `while` loops for file processing.
- Use `break` to stop unnecessary processing.
- Use `continue` to skip invalid entries.
- Keep loop bodies simple and readable.
- Validate input before processing.
- Avoid infinite loops unless required.
- Add logging inside long-running loops.

---

# Common Mistakes

- Creating infinite loops accidentally.
- Forgetting to update loop variables.
- Running expensive commands repeatedly inside loops.
- Using nested loops unnecessarily.
- Not validating input files.
- Forgetting to quote variables.
- Processing empty input without checks.

---

# Interview Questions

## Basic

1. What is a loop in shell scripting?
2. What are the different types of loops?
3. Explain the `for` loop.
4. Explain the `while` loop.
5. What is the purpose of the `until` loop?

## Intermediate

1. Difference between `for` and `while`.
2. Explain the `break` statement.
3. Explain the `continue` statement.
4. How do you process a file line by line?
5. What are nested loops?

## Advanced

1. Design a shell script that reads a list of servers from a file, verifies connectivity, collects disk usage, and generates a health report using loops.
2. Explain how loops are used in enterprise automation for Kubernetes deployments, AWS resource management, and CI/CD pipelines.
3. A company manages hundreds of Kubernetes deployments across multiple environments. Design a scalable automation script using loops to perform rolling restarts, health checks, logging, and failure handling while minimizing execution time.

---

