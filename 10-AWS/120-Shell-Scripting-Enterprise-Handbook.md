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

# Chapter 5 - Functions, Script Modularity & Error Handling

As shell scripts grow larger, writing everything in a single file becomes difficult to maintain.

Functions help organize scripts into reusable, manageable blocks of code.

Enterprise shell scripts use functions extensively for:

- Code Reusability
- Readability
- Error Handling
- Logging
- Deployment Automation
- Monitoring
- Backup Operations

Functions are one of the most important concepts for writing production-ready shell scripts.

---

# What is a Function?

A function is a named block of code that performs a specific task.

Instead of writing the same commands repeatedly:

```text
Backup

Backup

Backup

Backup
```

Create one function:

```text
backup()

↓

Call Whenever Required
```

This reduces code duplication.

---

# Why Use Functions?

Without functions:

```text
Deploy App

↓

Restart Service

↓

Verify Service

↓

Repeat Same Code
```

With functions:

```text
deploy()

↓

restart()

↓

verify()
```

Each task is written once and reused throughout the script.

---

# Function Syntax

Basic syntax:

```bash
function_name() {
    commands
}
```

Example:

```bash
hello() {
    echo "Hello DevOps"
}
```

Call the function:

```bash
hello
```

Output:

```text
Hello DevOps
```

---

# Function with Parameters

Functions can accept arguments.

Example:

```bash
greet() {
    echo "Welcome $1"
}

greet Surendra
```

Output:

```text
Welcome Surendra
```

---

# Multiple Parameters

```bash
deploy() {
    echo "Application : $1"
    echo "Environment : $2"
}

deploy payment production
```

Output:

```text
Application : payment

Environment : production
```

---

# Returning Values

Functions return an exit status.

Example:

```bash
check_service() {

    systemctl is-active nginx >/dev/null

    return $?
}
```

The calling script can evaluate the returned exit code.

---

# Capturing Output

A function can also produce output.

Example:

```bash
current_date() {

    date
}

TODAY=$(current_date)

echo $TODAY
```

Output:

```text
Mon Aug 18 10:00:00 IST 2026
```

---

# Local Variables

Variables declared inside functions should remain local.

Example:

```bash
show_name() {

    local NAME="DevOps"

    echo $NAME
}
```

Using `local` prevents accidental modification of variables outside the function.

---

# Global Variables

Variables declared outside functions are global.

Example:

```bash
ENV=production

deploy() {

    echo $ENV
}
```

Global variables can be accessed by every function.

---

# Function Execution Flow

```text
Main Script

↓

Function Call

↓

Execute Function

↓

Return

↓

Continue Script
```

Functions always return control to the calling script.

---

# Organizing Enterprise Scripts

Instead of writing one large script:

```text
500 Lines

↓

Hard to Maintain
```

Break it into functions.

Example:

```text
validate()

↓

backup()

↓

deploy()

↓

verify()

↓

cleanup()
```

Each function performs one responsibility.

---

# Logging Function

Example:

```bash
log() {

    echo "$(date) : $1"
}

log "Deployment Started"
```

Output:

```text
Mon Aug 18 10:00:00 : Deployment Started
```

Centralized logging improves troubleshooting.

---

# Error Handling

Always validate command execution.

Example:

```bash
copy_file() {

    cp app.conf /backup/

    if [ $? -ne 0 ]
    then
        echo "Backup Failed"
        exit 1
    fi
}
```

Ignoring failures can cause serious production issues.

---

# Exit Statement

Terminate script execution.

Example:

```bash
exit 0
```

Successful completion.

Example:

```bash
exit 1
```

Failure.

Exit codes help CI/CD tools determine script status.

---

# set Options

Enable safer script execution.

Stop on first error:

```bash
set -e
```

Treat unset variables as errors:

```bash
set -u
```

Display executed commands:

```bash
set -x
```

Combine options:

```bash
set -eux
```

These options are widely used in production scripts.

---

# Trap

The `trap` command executes cleanup logic when a script exits.

Example:

```bash
cleanup() {

    echo "Cleaning temporary files..."
}

trap cleanup EXIT
```

Useful for:

- Temporary File Cleanup
- Log Collection
- Unlocking Resources
- Notification

---

# Modular Script Design

Enterprise scripts are usually divided into sections.

```text
Variables

↓

Functions

↓

Validation

↓

Main Logic

↓

Cleanup
```

This structure improves readability and maintenance.

---

# Example Script Structure

```text
#!/bin/bash

Variables

↓

Functions

↓

Input Validation

↓

Deployment

↓

Verification

↓

Cleanup
```

A consistent layout makes scripts easier to understand.

---

# Kubernetes Example

Deployment workflow:

```text
Validate Namespace

↓

Build Image

↓

Update Manifest

↓

Deploy

↓

Verify Pods

↓

Success
```

Each step should be implemented as a separate function.

---

# AWS Example

Infrastructure automation:

```text
Create EC2

↓

Attach Security Group

↓

Wait for Instance

↓

Configure Server

↓

Verify
```

Breaking each operation into a function simplifies maintenance.

---

# CI/CD Example

Pipeline execution:

```text
Checkout Code

↓

Build

↓

Test

↓

Docker Build

↓

Push Image

↓

Deploy

↓

Verify
```

Each stage can be represented by an individual function.

---

# Enterprise Example

Production deployment script:

```text
validate_environment()

↓

backup_database()

↓

deploy_application()

↓

verify_health()

↓

send_notification()
```

Each function performs one clearly defined task.

---

# Enterprise Best Practices

- Write small, reusable functions.
- Keep one responsibility per function.
- Use meaningful function names.
- Prefer local variables inside functions.
- Validate command success after critical operations.
- Use `set -euo pipefail` in production scripts.
- Implement centralized logging.
- Use `trap` for cleanup activities.

---

# Common Mistakes

- Writing the entire script without functions.
- Using global variables unnecessarily.
- Ignoring command failures.
- Not returning proper exit codes.
- Forgetting cleanup operations.
- Creating very large functions with multiple responsibilities.
- Using unclear function names.

---

# Interview Questions

## Basic

1. What is a function in shell scripting?
2. Why are functions used?
3. How do you pass parameters to a function?
4. What is the difference between local and global variables?
5. How do you call a function?

## Intermediate

1. How do functions improve script maintainability?
2. Explain the purpose of `set -e`, `set -u`, and `set -x`.
3. What is the `trap` command?
4. How do functions return values?
5. Why should production scripts use centralized logging?

## Advanced

1. Design a modular deployment script for Kubernetes using reusable functions for validation, deployment, verification, rollback, and notification.
2. Explain how functions, error handling, exit codes, and logging work together to build reliable enterprise automation scripts.
3. A CI/CD pipeline deploys applications across multiple Kubernetes clusters. Design a shell scripting framework using functions, local variables, `set -euo pipefail`, logging, and cleanup handlers to ensure reliable deployments and easy maintenance.

---

# Chapter 6 - Arrays, String Manipulation & Text Processing

Most enterprise automation scripts work with collections of data rather than single values.

Examples include:

- Server Lists
- Kubernetes Namespaces
- Docker Images
- AWS Instance IDs
- Log Files
- User Accounts

Arrays and string manipulation make it easy to process large amounts of information efficiently.

Text processing tools such as `grep`, `cut`, `awk`, and `sed` are among the most frequently used utilities by DevOps engineers.

---

# What is an Array?

An array is a collection of multiple values stored under a single variable.

Instead of creating multiple variables:

```bash
SERVER1=web01

SERVER2=web02

SERVER3=web03
```

Use one array:

```bash
SERVERS=("web01" "web02" "web03")
```

Arrays simplify automation tasks.

---

# Creating an Array

Example:

```bash
TOOLS=("Docker" "Kubernetes" "Terraform" "Jenkins")
```

This creates an array containing four values.

---

# Accessing Array Elements

Retrieve a specific element using its index.

```bash
echo ${TOOLS[0]}
```

Output:

```text
Docker
```

```bash
echo ${TOOLS[2]}
```

Output:

```text
Terraform
```

Array indexing starts at **0**.

---

# Display All Elements

Print every element in the array.

```bash
echo "${TOOLS[@]}"
```

Output:

```text
Docker Kubernetes Terraform Jenkins
```

---

# Array Length

Determine the number of elements.

```bash
echo ${#TOOLS[@]}
```

Output:

```text
4
```

Useful when processing dynamic datasets.

---

# Add Elements

Append a new value.

```bash
TOOLS+=("Ansible")
```

Updated array:

```text
Docker Kubernetes Terraform Jenkins Ansible
```

---

# Loop Through an Array

Example:

```bash
for TOOL in "${TOOLS[@]}"
do
    echo $TOOL
done
```

Output:

```text
Docker

Kubernetes

Terraform

Jenkins
```

---

# Associative Arrays

Bash also supports key-value pairs.

```bash
declare -A SERVERS

SERVERS[dev]="10.0.0.10"

SERVERS[prod]="10.0.1.20"
```

Access a value:

```bash
echo ${SERVERS[prod]}
```

Output:

```text
10.0.1.20
```

Associative arrays require Bash 4 or later.

---

# Strings

A string is a sequence of characters.

Example:

```bash
NAME="DevOps Engineer"
```

Strings are widely used for:

- File Names
- URLs
- Image Tags
- Branch Names
- Environment Names

---

# String Length

Determine string length.

```bash
NAME="Docker"

echo ${#NAME}
```

Output:

```text
6
```

---

# String Concatenation

Join multiple strings.

```bash
APP="payment"

ENV="production"

echo "$APP-$ENV"
```

Output:

```text
payment-production
```

---

# Substring

Extract part of a string.

```bash
TEXT="Kubernetes"

echo ${TEXT:0:4}
```

Output:

```text
Kube
```

---

# Replace Text

Replace part of a string.

```bash
APP="payment-service"

echo ${APP/service/api}
```

Output:

```text
payment-api
```

---

# Convert to Uppercase

```bash
NAME="docker"

echo ${NAME^^}
```

Output:

```text
DOCKER
```

---

# Convert to Lowercase

```bash
NAME="KUBERNETES"

echo ${NAME,,}
```

Output:

```text
kubernetes
```

---

# grep

`grep` searches for matching text.

Example:

```bash
grep "ERROR" application.log
```

Ignore case:

```bash
grep -i "failed" application.log
```

Show line numbers:

```bash
grep -n "timeout" application.log
```

Recursive search:

```bash
grep -r "database" .
```

`grep` is one of the most frequently used Linux commands.

---

# cut

Extract specific fields.

Example:

```bash
echo "Docker,Kubernetes,Jenkins" | cut -d',' -f2
```

Output:

```text
Kubernetes
```

---

# sort

Sort text.

```bash
sort servers.txt
```

Reverse order:

```bash
sort -r servers.txt
```

---

# uniq

Remove duplicate entries.

```bash
sort users.txt | uniq
```

Count duplicates:

```bash
sort users.txt | uniq -c
```

---

# wc

Count lines, words, or characters.

```bash
wc -l servers.txt
```

Count words:

```bash
wc -w notes.txt
```

---

# tr

Translate characters.

Convert lowercase to uppercase:

```bash
echo "devops" | tr 'a-z' 'A-Z'
```

Output:

```text
DEVOPS
```

---

# sed

`sed` is a stream editor used to modify text.

Replace text:

```bash
sed 's/dev/stage/' config.txt
```

Replace globally:

```bash
sed 's/dev/prod/g' config.txt
```

Edit a file directly:

```bash
sed -i 's/8080/9090/g' application.properties
```

`sed` is commonly used to update configuration files during deployments.

---

# awk

`awk` processes structured text.

Print the first column:

```bash
awk '{print $1}' users.txt
```

Print specific fields:

```bash
awk '{print $1,$3}' users.txt
```

`awk` is extremely powerful for parsing logs and reports.

---

# xargs

Convert input into command arguments.

Example:

```bash
cat files.txt | xargs rm
```

This removes every file listed in `files.txt`.

Always verify the input before executing destructive commands.

---

# Combining Commands

Linux commands are often combined using pipes.

Example:

```bash
cat application.log | grep ERROR | wc -l
```

Workflow:

```text
Read File

↓

Find Errors

↓

Count Matches
```

Pipelines are fundamental to shell scripting.

---

# Kubernetes Example

List all running pods.

```bash
kubectl get pods | grep Running
```

Extract pod names.

```bash
kubectl get pods | awk '{print $1}'
```

Restart multiple deployments.

```bash
kubectl get deploy | awk '{print $1}'
```

---

# AWS Example

Extract EC2 instance IDs.

```bash
aws ec2 describe-instances | grep InstanceId
```

Process the results using `awk`, `cut`, or loops.

---

# CI/CD Example

Update image tags.

```bash
sed -i 's/v1.0/v1.1/g' deployment.yaml
```

Commit updated manifests.

Deploy using GitOps.

---

# Enterprise Example

Find failed login attempts.

```bash
grep "Failed password" /var/log/secure
```

Count failures.

```bash
grep "Failed password" /var/log/secure | wc -l
```

This approach is frequently used for security investigations.

---

# Enterprise Best Practices

- Use arrays for collections of related values.
- Quote array expansions to handle spaces correctly.
- Prefer `grep`, `awk`, and `sed` over complex shell logic for text processing.
- Use pipelines to simplify scripts.
- Test text-processing commands before modifying production files.
- Keep string operations readable.
- Validate command output before further processing.
- Avoid unnecessary command chaining.

---

# Common Mistakes

- Forgetting that array indexing starts at zero.
- Omitting quotes around array variables.
- Using `sed -i` without testing the replacement.
- Parsing structured data with fragile string operations.
- Running `xargs rm` without verifying input.
- Ignoring whitespace in string processing.
- Writing overly complex pipelines that are difficult to debug.

---

# Interview Questions

## Basic

1. What is an array in Bash?
2. How do you access array elements?
3. What is the difference between indexed and associative arrays?
4. What does `grep` do?
5. What is the purpose of `sed`?

## Intermediate

1. Explain string manipulation in Bash.
2. Difference between `grep`, `sed`, and `awk`.
3. How does `cut` work?
4. Explain Linux pipelines.
5. How do you remove duplicate lines from a file?

## Advanced

1. Design a shell script that reads a list of Kubernetes namespaces from an array, updates deployment manifests using `sed`, validates them using `grep`, and deploys them automatically.
2. Explain how `grep`, `awk`, `sed`, `cut`, `sort`, `uniq`, and `xargs` work together in enterprise automation and log analysis.
3. A production Linux server generates multi-gigabyte log files every day. Design an efficient shell script using arrays, string manipulation, and Linux text-processing utilities to identify failed logins, summarize errors, extract key metrics, and generate a daily operational report.

---

# Chapter 7 - File Handling, Input/Output & Logging

Enterprise shell scripts constantly interact with files.

Typical operations include:

- Reading Configuration Files
- Processing CSV Files
- Parsing Log Files
- Creating Reports
- Backing Up Files
- Writing Audit Logs
- Generating Deployment Reports

A DevOps Engineer spends a significant amount of time writing scripts that read data from files and write results to new files.

---

# Why File Handling?

Consider a production environment with hundreds of servers.

Without file handling:

```text
Login Server 1

↓

Run Command

↓

Login Server 2

↓

Run Command

↓

Repeat
```

With file handling:

```text
servers.txt

↓

Shell Script

↓

Read Servers

↓

Execute Commands

↓

Generate Report
```

Automation becomes scalable and repeatable.

---

# Input and Output Streams

Every Linux process uses three standard streams.

| Stream | Description |
|----------|-------------|
| STDIN (0) | Standard Input |
| STDOUT (1) | Standard Output |
| STDERR (2) | Standard Error |

```text
Keyboard

↓

STDIN

↓

Program

↓

STDOUT

↓

Screen
```

Errors are written to **STDERR**.

---

# Reading a File

Display file contents.

```bash
cat servers.txt
```

Example:

```text
web01

web02

web03
```

Useful for small files.

---

# Reading File Line by Line

Recommended approach for large files.

```bash
while read SERVER
do
    echo $SERVER
done < servers.txt
```

This method is memory efficient and widely used in production scripts.

---

# Reading CSV Files

Example CSV:

```text
Name,Environment

Payment,Production

Order,Stage
```

Script:

```bash
while IFS=',' read APP ENV
do
    echo "$APP -> $ENV"
done < apps.csv
```

CSV processing is common in automation.

---

# Check if File Exists

```bash
if [ -f config.yaml ]
then
    echo "File Exists"
else
    echo "File Missing"
fi
```

Always validate files before processing them.

---

# Check Directory Exists

```bash
if [ -d /backup ]
then
    echo "Directory Found"
fi
```

---

# Create Files

```bash
touch report.txt
```

Create multiple files:

```bash
touch app.log deploy.log backup.log
```

---

# Create Directories

```bash
mkdir reports
```

Create nested directories:

```bash
mkdir -p reports/2026/august
```

---

# Copy Files

```bash
cp app.conf backup.conf
```

Copy directories:

```bash
cp -r configs backup/
```

---

# Move Files

```bash
mv report.txt archive/
```

Rename a file:

```bash
mv report.txt report_old.txt
```

---

# Delete Files

```bash
rm report.txt
```

Delete directory recursively:

```bash
rm -rf reports/
```

⚠️ Always verify paths before deleting files.

---

# File Redirection

Overwrite file contents:

```bash
echo "Deployment Successful" > report.txt
```

Append to a file:

```bash
echo "Deployment Completed" >> report.txt
```

Difference:

| Operator | Action |
|----------|--------|
| `>` | Overwrite |
| `>>` | Append |

---

# Redirect Errors

Write errors to a file.

```bash
command 2> errors.log
```

Append errors:

```bash
command 2>> errors.log
```

---

# Redirect Output and Errors

Store both standard output and errors.

```bash
command > output.log 2>&1
```

Or using modern Bash syntax:

```bash
command &> output.log
```

This is commonly used in automation scripts.

---

# Ignore Output

Suppress command output.

```bash
command > /dev/null
```

Suppress both output and errors.

```bash
command > /dev/null 2>&1
```

Useful for silent validation checks.

---

# Here Document (HEREDOC)

Write multiple lines into a file.

```bash
cat <<EOF > config.txt
Application=Payment
Environment=Production
Version=1.0
EOF
```

HEREDOCs simplify configuration generation.

---

# Temporary Files

Create a secure temporary file.

```bash
mktemp
```

Example output:

```text
/tmp/tmp.aBc123
```

Temporary files should be removed after use.

---

# Logging

Every production script should generate logs.

Example:

```bash
echo "$(date) : Deployment Started" >> deploy.log
```

Example log:

```text
2026-08-18 10:30 Deployment Started
```

Logs simplify troubleshooting.

---

# Centralized Logging Function

```bash
log() {

    echo "$(date '+%F %T') : $1" >> deploy.log
}
```

Usage:

```bash
log "Application Deployed"
```

This avoids repeated logging code.

---

# Backup Files

Simple backup:

```bash
cp app.conf app.conf.bak
```

Timestamped backup:

```bash
cp app.conf app.conf.$(date +%F)
```

This preserves previous versions.

---

# Process Large Files

Read only the first few lines:

```bash
head application.log
```

Read the last few lines:

```bash
tail application.log
```

Follow a growing log:

```bash
tail -f application.log
```

These commands are essential during production troubleshooting.

---

# Count Records

Count lines.

```bash
wc -l servers.txt
```

Count words.

```bash
wc -w notes.txt
```

Count characters.

```bash
wc -c file.txt
```

---

# Search Within Files

```bash
grep ERROR application.log
```

Search recursively.

```bash
grep -r "database" .
```

Count matches.

```bash
grep ERROR application.log | wc -l
```

---

# Kubernetes Example

Read deployment names from a file.

```bash
while read APP
do
    kubectl rollout restart deployment/$APP
done < deployments.txt
```

One script can restart hundreds of deployments.

---

# AWS Example

Read EC2 instance IDs.

```bash
while read ID
do
    aws ec2 stop-instances --instance-ids $ID
done < instances.txt
```

This approach is commonly used for infrastructure automation.

---

# CI/CD Example

Generate deployment reports.

```text
Pipeline

↓

Deployment

↓

Log File

↓

Report

↓

Notification
```

Logs provide traceability for every deployment.

---

# Enterprise Example

Daily backup workflow.

```text
Read Server List

↓

Connect

↓

Backup Database

↓

Write Log

↓

Generate Report
```

Each step records success or failure.

---

# Enterprise Best Practices

- Always verify file existence before reading.
- Use append (`>>`) for log files.
- Redirect errors to separate log files.
- Use timestamped backups.
- Remove temporary files after use.
- Store logs in a dedicated directory.
- Validate input files before processing.
- Generate audit logs for critical operations.

---

# Common Mistakes

- Overwriting log files using `>`.
- Ignoring error output.
- Using hardcoded file paths.
- Forgetting to close HEREDOCs correctly.
- Leaving temporary files behind.
- Deleting files without validation.
- Running destructive commands without backups.

---

# Interview Questions

## Basic

1. What are STDIN, STDOUT, and STDERR?
2. What is the difference between `>` and `>>`?
3. How do you check whether a file exists?
4. How do you read a file line by line?
5. What is `/dev/null`?

## Intermediate

1. Explain output redirection.
2. What is a HEREDOC?
3. How do you redirect both output and errors to a file?
4. Why should production scripts generate logs?
5. How do you process CSV files in Bash?

## Advanced

1. Design a shell script that reads a list of servers from a file, performs health checks, logs results, generates timestamped reports, and stores failures in a separate error log.
2. Explain how file handling, input/output redirection, logging, and temporary files contribute to reliable enterprise automation.
3. A CI/CD platform deploys applications to hundreds of servers every day. Design a shell scripting framework that reads deployment targets from configuration files, captures logs, redirects errors, creates backups, and generates deployment reports while ensuring reliability and auditability.

---

# Chapter 8 - Shell Scripting Automation for Linux, AWS, Docker & Kubernetes

One of the biggest advantages of shell scripting is its ability to automate repetitive administrative tasks.

Enterprise DevOps engineers use shell scripts to automate:

- Linux Administration
- AWS Operations
- Docker Management
- Kubernetes Administration
- CI/CD Pipelines
- Backup & Recovery
- Health Checks
- Monitoring

Automation improves consistency, reduces human error, and saves time.

---

# Automation Workflow

A typical automation workflow looks like this:

```text
Input

↓

Validation

↓

Execute Commands

↓

Collect Results

↓

Generate Logs

↓

Exit
```

Every production script should follow a structured workflow.

---

# Linux Administration Automation

Common Linux automation tasks include:

- Creating Users
- Managing Services
- Installing Packages
- Cleaning Log Files
- Disk Monitoring
- Backup Management
- System Updates

Example:

```bash
#!/bin/bash

systemctl restart nginx

systemctl status nginx
```

---

# User Creation Automation

Create multiple users automatically.

```bash
#!/bin/bash

for USER in dev1 dev2 dev3
do
    useradd $USER
    echo "$USER created successfully"
done
```

Automation removes repetitive manual work.

---

# Service Monitoring

Check if a service is running.

```bash
#!/bin/bash

if systemctl is-active --quiet nginx
then
    echo "Nginx is running"
else
    echo "Nginx is stopped"
fi
```

This script can be executed periodically using Cron.

---

# Disk Usage Monitoring

Monitor root filesystem usage.

```bash
#!/bin/bash

USAGE=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')

if [ "$USAGE" -gt 80 ]
then
    echo "Disk usage exceeds 80%"
fi
```

This helps prevent production outages caused by full disks.

---

# Backup Automation

Backup configuration files.

```bash
#!/bin/bash

cp /etc/nginx/nginx.conf \
/backup/nginx.conf.$(date +%F)
```

Timestamped backups simplify recovery.

---

# AWS CLI Automation

Shell scripts frequently automate AWS resources.

Examples:

- EC2
- S3
- IAM
- EKS
- CloudFormation
- Route53

---

# List EC2 Instances

```bash
#!/bin/bash

aws ec2 describe-instances
```

This command retrieves information about EC2 instances.

---

# Start an EC2 Instance

```bash
#!/bin/bash

INSTANCE=i-123456789

aws ec2 start-instances \
--instance-ids $INSTANCE
```

---

# Stop an EC2 Instance

```bash
#!/bin/bash

INSTANCE=i-123456789

aws ec2 stop-instances \
--instance-ids $INSTANCE
```

Useful for cost optimization in non-production environments.

---

# Upload Files to S3

```bash
#!/bin/bash

aws s3 cp backup.tar.gz \
s3://company-backups/
```

A common backup strategy is to upload archives to Amazon S3.

---

# Docker Automation

Shell scripts simplify Docker administration.

Common tasks:

- Build Images
- Run Containers
- Remove Containers
- Clean Images
- Push Images

---

# Build Docker Image

```bash
#!/bin/bash

docker build -t payment:v1 .
```

---

# Run Container

```bash
docker run -d \
--name payment \
payment:v1
```

---

# Remove Stopped Containers

```bash
docker container prune -f
```

Regular cleanup prevents storage exhaustion.

---

# Remove Unused Images

```bash
docker image prune -a -f
```

Useful on CI/CD build servers.

---

# Kubernetes Automation

Shell scripting is widely used with `kubectl`.

Common operations:

- Deploy Applications
- Restart Deployments
- Check Pods
- Verify Nodes
- Collect Logs

---

# Check Cluster Nodes

```bash
kubectl get nodes
```

Automated node verification is common in health-check scripts.

---

# Restart Deployment

```bash
kubectl rollout restart deployment/payment
```

---

# Verify Pod Status

```bash
kubectl get pods \
-n production
```

Scripts often validate that all pods reach the **Running** state.

---

# Wait for Deployment

```bash
kubectl rollout status \
deployment/payment
```

This ensures the deployment completes successfully.

---

# Collect Pod Logs

```bash
kubectl logs payment-abc123
```

Useful during automated troubleshooting.

---

# Health Check Script

Example workflow:

```text
Check CPU

↓

Check Memory

↓

Check Disk

↓

Check Service

↓

Generate Report
```

Health-check scripts provide a quick overview of system status.

---

# Cron Automation

Schedule recurring jobs.

Example:

```text
0 2 * * * backup.sh
```

This executes `backup.sh` every day at 2:00 AM.

Common Cron tasks:

- Backups
- Log Cleanup
- Monitoring
- Report Generation

---

# Deployment Automation

A typical deployment script performs:

```text
Pull Source Code

↓

Build

↓

Run Tests

↓

Build Docker Image

↓

Push Image

↓

Update Kubernetes

↓

Verify Deployment
```

Each stage can be automated using shell scripts.

---

# CI/CD Example

Pipeline workflow:

```text
GitHub

↓

Jenkins

↓

Shell Script

↓

Docker Build

↓

Push Image

↓

Update GitOps Repository

↓

ArgoCD

↓

Amazon EKS
```

Shell scripts orchestrate each stage of the pipeline.

---

# Enterprise Example

A nightly maintenance script:

```text
Clean Logs

↓

Backup Database

↓

Restart Services

↓

Generate Report

↓

Send Email
```

This reduces manual operational effort.

---

# Error Handling Example

Always validate command success.

```bash
docker build -t app:v1 .

if [ $? -ne 0 ]
then
    echo "Docker build failed"
    exit 1
fi
```

Failing early prevents incorrect deployments.

---

# Logging Automation

Example logging function:

```bash
log() {
    echo "$(date '+%F %T') : $1" >> automation.log
}

log "Backup Completed"
```

Centralized logging simplifies troubleshooting.

---

# Enterprise Best Practices

- Keep scripts modular and reusable.
- Validate prerequisites before execution.
- Log every critical operation.
- Use variables instead of hardcoded values.
- Test automation in non-production environments.
- Handle failures gracefully using exit codes.
- Store automation scripts in version control.
- Schedule recurring tasks using Cron.

---

# Common Mistakes

- Hardcoding resource names.
- Ignoring command failures.
- Running scripts without validation.
- Omitting logging.
- Running destructive commands without confirmation.
- Forgetting to clean temporary files.
- Executing production scripts without testing.

---

# Interview Questions

## Basic

1. Why is shell scripting used for automation?
2. How do you automate Linux administration tasks?
3. How do you automate Docker commands?
4. How do you automate Kubernetes operations?
5. What is Cron?

## Intermediate

1. Explain a health-check automation script.
2. How do you automate EC2 management using AWS CLI?
3. Why should production scripts generate logs?
4. How do shell scripts integrate with CI/CD pipelines?
5. Explain deployment automation using shell scripts.

## Advanced

1. Design an enterprise automation framework using shell scripts to manage Linux servers, AWS infrastructure, Docker containers, and Kubernetes clusters while ensuring proper logging, error handling, and reporting.
2. Explain how shell scripting integrates with Jenkins, GitHub Actions, ArgoCD, Docker, Terraform, and Kubernetes to automate enterprise deployments.
3. A financial organization manages hundreds of Linux servers and Kubernetes clusters across multiple AWS regions. Design a shell scripting solution to automate health checks, backups, deployments, monitoring, and scheduled maintenance while ensuring reliability, scalability, and auditability.

---

# Chapter 9 - Production Shell Scripting, Security, Scheduling & Enterprise Automation

Enterprise shell scripts are expected to do much more than execute commands.

A production-ready script should:

- Validate Input
- Handle Errors
- Generate Logs
- Schedule Tasks
- Send Notifications
- Protect Sensitive Data
- Support Rollback
- Be Easily Maintainable

A well-written script becomes a reliable operational tool rather than just a collection of commands.

---

# Production Automation Workflow

A typical production automation workflow follows these stages:

```text
Input

↓

Validation

↓

Execution

↓

Logging

↓

Verification

↓

Notification

↓

Cleanup

↓

Exit
```

Each stage improves reliability and traceability.

---

# Input Validation

Never assume user input is correct.

Example:

```bash
read -p "Enter Environment: " ENV

if [[ "$ENV" != "dev" && \
      "$ENV" != "stage" && \
      "$ENV" != "prod" ]]
then
    echo "Invalid Environment"
    exit 1
fi
```

Input validation prevents accidental production mistakes.

---

# Check Required Commands

Before executing automation, verify required tools are installed.

Example:

```bash
command -v kubectl >/dev/null 2>&1

if [ $? -ne 0 ]
then
    echo "kubectl is not installed"
    exit 1
fi
```

This prevents failures later in the script.

---

# Validate Required Files

Example:

```bash
if [ ! -f deployment.yaml ]
then
    echo "deployment.yaml not found"
    exit 1
fi
```

Always verify configuration files before deployment.

---

# Logging Standards

Every important action should be logged.

Example:

```bash
log() {

    echo "$(date '+%F %T') \
: $1" >> deployment.log
}
```

Example:

```bash
log "Deployment Started"

log "Docker Image Built"

log "Deployment Completed"
```

Logs simplify auditing and troubleshooting.

---

# Error Handling

Handle every critical command.

Example:

```bash
docker build -t payment:v2 .

if [ $? -ne 0 ]
then
    log "Docker Build Failed"
    exit 1
fi
```

Never continue after a critical failure.

---

# Rollback Strategy

Production scripts should support rollback.

Workflow:

```text
Backup

↓

Deploy

↓

Verify

↓

Failure?

↓

Rollback

↓

Restore
```

Automation should leave the environment in a consistent state.

---

# Notifications

Notify teams after important operations.

Examples:

- Email
- Slack
- Microsoft Teams
- Webhook

Workflow:

```text
Deployment

↓

Success / Failure

↓

Notification
```

This keeps stakeholders informed.

---

# Secure Credentials

Never hardcode credentials.

Avoid:

```bash
PASSWORD=Admin123
```

Prefer:

- Environment Variables
- AWS Secrets Manager
- HashiCorp Vault
- Kubernetes Secrets

Example:

```bash
DB_PASSWORD=$DATABASE_PASSWORD
```

---

# Temporary Files

Create temporary files securely.

```bash
TEMP_FILE=$(mktemp)
```

Remove them before exiting.

```bash
rm -f "$TEMP_FILE"
```

Or use:

```bash
trap 'rm -f "$TEMP_FILE"' EXIT
```

---

# Lock Files

Prevent multiple instances of the same script.

Example:

```bash
LOCK=/tmp/deploy.lock

if [ -f "$LOCK" ]
then
    echo "Another deployment is running."
    exit 1
fi

touch "$LOCK"

trap "rm -f $LOCK" EXIT
```

Lock files prevent duplicate executions.

---

# Scheduling with Cron

Recurring jobs are scheduled using Cron.

View current jobs:

```bash
crontab -l
```

Edit cron jobs:

```bash
crontab -e
```

Example:

```text
0 1 * * * /scripts/backup.sh
```

Runs every day at **1:00 AM**.

---

# Cron Schedule Examples

| Schedule | Description |
|----------|-------------|
| `* * * * *` | Every minute |
| `0 * * * *` | Every hour |
| `0 0 * * *` | Every day at midnight |
| `0 2 * * 0` | Every Sunday at 2 AM |
| `*/10 * * * *` | Every 10 minutes |

Cron is widely used for operational automation.

---

# Backup Automation

Example workflow:

```text
Stop Application

↓

Backup Database

↓

Backup Configuration

↓

Start Application

↓

Generate Report
```

Backups should always be validated.

---

# Log Rotation Script

Example:

```bash
find /var/log \
-name "*.log" \
-mtime +30 \
-delete
```

This removes log files older than 30 days.

Always verify retention policies before deletion.

---

# Health Check Script

A production health check might verify:

- CPU Usage
- Memory Usage
- Disk Usage
- Running Services
- Network Connectivity
- Kubernetes Cluster
- Database Connectivity

Generate a report after execution.

---

# Deployment Verification

After deployment, verify:

- Service Status
- Pod Status
- HTTP Endpoint
- Database Connectivity

Workflow:

```text
Deploy

↓

Health Check

↓

Success?

↓

Complete
```

Never assume deployment succeeded without validation.

---

# Scheduling Health Checks

Example:

```text
Every 5 Minutes

↓

Health Script

↓

Generate Report

↓

Alert if Failure
```

Frequent health checks reduce downtime.

---

# Enterprise Script Structure

A production script typically follows this structure:

```text
Variables

↓

Functions

↓

Input Validation

↓

Pre-Checks

↓

Execution

↓

Verification

↓

Logging

↓

Cleanup

↓

Exit
```

This structure improves maintainability.

---

# CI/CD Example

Deployment automation:

```text
GitHub

↓

Jenkins

↓

Shell Script

↓

Docker Build

↓

Push Image

↓

Update Manifest

↓

ArgoCD

↓

Kubernetes
```

Shell scripts coordinate multiple stages.

---

# Kubernetes Example

Production deployment:

```text
Check Namespace

↓

Build Image

↓

Deploy

↓

Verify Pods

↓

Verify Service

↓

Generate Report
```

Verification ensures reliable deployments.

---

# AWS Example

Infrastructure automation:

```text
Create EC2

↓

Configure Security Group

↓

Install Software

↓

Run Health Checks

↓

Tag Resources

↓

Generate Report
```

Automation reduces provisioning time.

---

# Enterprise Example

Nightly maintenance:

```text
Clean Logs

↓

Backup Files

↓

Restart Services

↓

Check Disk

↓

Email Report
```

This workflow keeps production servers healthy.

---

# Enterprise Best Practices

- Validate every input.
- Never hardcode secrets.
- Log every important action.
- Verify deployments before completion.
- Implement rollback procedures.
- Use lock files for critical scripts.
- Schedule recurring tasks using Cron.
- Keep scripts idempotent whenever possible.
- Test scripts thoroughly before production use.
- Store scripts in version control.

---

# Common Mistakes

- Running multiple instances of the same script.
- Hardcoding passwords or API keys.
- Ignoring deployment verification.
- Continuing execution after failures.
- Deleting files without backups.
- Running scheduled jobs without logging.
- Not cleaning temporary files.
- Forgetting rollback procedures.

---

# Interview Questions

## Basic

1. Why should production scripts generate logs?
2. What is Cron?
3. How do you schedule recurring jobs?
4. Why should shell scripts validate input?
5. What is the purpose of a lock file?

## Intermediate

1. Explain how `trap` is used for cleanup.
2. Why should scripts avoid hardcoded credentials?
3. How do you implement deployment verification?
4. Explain rollback automation.
5. How do you prevent multiple executions of the same script?

## Advanced

1. Design a production-ready shell scripting framework that includes input validation, logging, rollback, notifications, Cron scheduling, cleanup, and deployment verification.
2. Explain how production shell scripts integrate with Kubernetes, AWS, Jenkins, GitHub Actions, and ArgoCD while maintaining security, reliability, and auditability.
3. A financial organization runs thousands of scheduled automation jobs every day. Design a shell scripting platform that supports centralized logging, secure credential management, job scheduling, failure recovery, rollback, reporting, and operational monitoring across multiple environments.

---

