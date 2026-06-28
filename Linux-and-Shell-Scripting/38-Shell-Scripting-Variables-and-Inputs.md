# Shell Scripting Variables and Inputs

## Introduction

Variables are one of the most important concepts in Shell Scripting.

Variables help us:

- Avoid Hardcoding
- Improve Reusability
- Improve Maintainability
- Improve Readability
- Follow DRY (Don't Repeat Yourself)

A variable acts as a container that stores a value.

---

# Git Workflow Recap

Before writing automation scripts, it is important to store them in Git repositories.

Typical Workflow:

```text
Clone Repository
        │
        ▼
Make Changes
        │
        ▼
Add to Staging Area
        │
        ▼
Commit to Local Repository
        │
        ▼
Push to Remote Repository
```

---

# Clone Repository

Used for first-time download.

```bash
git clone <repo-url>
```

Example:

```bash
git clone https://github.com/user/project.git
```

---

# Add to Staging Area

Stage a single file:

```bash
git add file-name
```

Example:

```bash
git add script.sh
```

---

Stage all files:

```bash
git add .
```

---

# Commit to Local Repository

```bash
git commit -m "Added variable examples"
```

Commit stores:

- Who changed
- When changed
- Why changed

---

# View Commit History

```bash
git log
```

Displays:

- Commit ID
- Author
- Date
- Commit Message

---

# Push to Central Repository

```bash
git push origin main
```

Uploads commits to GitHub/GitLab/Bitbucket.

---

# Git Configuration

Configure username:

```bash
git config --global user.name "Your Name"
```

Example:

```bash
git config --global user.name "Sivakumar"
```

---

Configure email:

```bash
git config --global user.email "your-email@example.com"
```

Example:

```bash
git config --global user.email "siva@gmail.com"
```

---

# Why Variables?

Without variables:

```bash
echo "catalogue"

systemctl restart catalogue

journalctl -u catalogue
```

Value repeated multiple times.

---

Using variables:

```bash
COMPONENT=catalogue

echo $COMPONENT

systemctl restart $COMPONENT

journalctl -u $COMPONENT
```

---

# Variable Declaration

Syntax:

```bash
VAR_NAME=VALUE
```

Example:

```bash
NAME=DevOps
```

---

# Access Variable

Using Dollar Symbol:

```bash
echo $NAME
```

Output:

```text
DevOps
```

---

# Alternative Syntax

Using Curly Braces:

```bash
echo ${NAME}
```

Output:

```text
DevOps
```

---

# Why Use Curly Braces?

Useful when combining text.

Example:

```bash
NAME=DevOps

echo "${NAME}Engineer"
```

Output:

```text
DevOpsEngineer
```

---

# Ways to Pass Values to Scripts

There are multiple ways to provide values to a shell script.

```text
1. Declare Inside Script
2. Command Line Arguments
3. Read User Input
4. Environment Variables
5. Dynamic Command Execution
```

---

# Method 1: Declare Inside Script

Example:

```bash
#!/bin/bash

NAME=DevOps

echo $NAME
```

Output:

```text
DevOps
```

---

# Method 2: Command Line Arguments

Pass values while running the script.

Example:

```bash
sh 04-variables.sh Trump Putin
```

---

# Positional Parameters

Shell automatically stores arguments.

```text
$1 → Trump
$2 → Putin
```

---

Example Script

```bash
#!/bin/bash

echo $1
echo $2
```

Run:

```bash
sh 04-variables.sh Trump Putin
```

Output:

```text
Trump
Putin
```

---

# Common Positional Parameters

| Variable | Meaning |
|-----------|----------|
| $0 | Script Name |
| $1 | First Argument |
| $2 | Second Argument |
| $3 | Third Argument |
| $# | Number of Arguments |
| $@ | All Arguments |
| $* | All Arguments |

---

# Example

Script:

```bash
#!/bin/bash

echo "Script Name: $0"
echo "First Arg : $1"
echo "Second Arg: $2"
```

Run:

```bash
sh demo.sh Linux Docker
```

Output:

```text
Script Name: demo.sh
First Arg : Linux
Second Arg: Docker
```

---

# Method 3: Read User Input

Example:

```bash
echo "Enter Name"

read NAME

echo $NAME
```

---

Execution:

```text
Enter Name

Ramesh
```

Output:

```text
Ramesh
```

---

# Method 4: Environment Variables

Environment Variables are variables available to the current process and its child processes.

---

# Export Variable

Example:

```bash
export COURSE="DevSecOps with AWS"
```

Verify:

```bash
echo $COURSE
```

Output:

```text
DevSecOps with AWS
```

---

# Environment Variable Scope

Environment Variable exists while:

```text
Current Process Is Alive
```

and is available to:

```text
Child Processes
```

---

# View Environment Variables

```bash
env
```

---

# Persistent Environment Variables

To make variables available permanently:

Edit:

```bash
~/.bashrc
```

---

Example:

```bash
export COURSE="DevSecOps with AWS"
```

---

Reload Configuration

```bash
source ~/.bashrc
```

or

```bash
source .bashrc
```

---

# Why Source Command?

Without source:

```text
Changes Not Loaded
```

With source:

```text
Current Shell Reloaded
```

---

# Method 5: Dynamic Command Execution

Sometimes variable values come from command output.

Syntax:

```bash
VAR_NAME=$(command)
```

This is called:

```text
Command Substitution
```

---

# Example

Store Current Date:

```bash
DATE=$(date)
```

Display:

```bash
echo $DATE
```

---

# Example

Store Current User:

```bash
USER_NAME=$(whoami)
```

Display:

```bash
echo $USER_NAME
```

---

# Real World Example

Capture Script Start Time:

```bash
START_TIME=$(date +%s)
```

---

Capture Script End Time:

```bash
END_TIME=$(date +%s)
```

---

Calculate Execution Time:

```bash
TOTAL_TIME=$((END_TIME - START_TIME))
```

---

Display:

```bash
echo $TOTAL_TIME
```

Output:

```text
Execution Time in Seconds
```

---

# Script Timing Example

```bash
#!/bin/bash

START_TIME=$(date +%s)

sleep 5

END_TIME=$(date +%s)

TOTAL_TIME=$((END_TIME - START_TIME))

echo "Execution Time: $TOTAL_TIME"
```

Output:

```text
Execution Time: 5
```

---

# Common Environment Variables

Current User:

```bash
echo $USER
```

---

Home Directory:

```bash
echo $HOME
```

---

Current Shell:

```bash
echo $SHELL
```

---

Current Working Directory:

```bash
echo $PWD
```

---

# Best Practices

## Use Meaningful Variable Names

Good:

```bash
DB_HOST
```

Bad:

```bash
X
```

---

## Avoid Hardcoding

Instead of:

```bash
mysql.example.com
```

Use:

```bash
DB_HOST=mysql.example.com
```

---

## Use Environment Variables for Configuration

Example:

```bash
DB_HOST
DB_USER
DB_PASS
```

---

## Use Command Substitution

Capture dynamic values automatically.

---

# Frequently Asked Interview Questions

## What is a Variable?

A container used to store data.

---

## How Do You Declare a Variable?

```bash
VAR_NAME=VALUE
```

---

## Difference Between $VAR and ${VAR}?

Both access variable values, but `${VAR}` is safer when combining strings.

---

## What are Positional Parameters?

Command-line arguments passed to a script.

Example:

```text
$1
$2
$3
```

---

## What is an Environment Variable?

A variable available to the current process and child processes.

---

## How Do You Create an Environment Variable?

```bash
export VAR_NAME=VALUE
```

---

## What Does source ~/.bashrc Do?

Reloads shell configuration without opening a new terminal.

---

## What is Command Substitution?

Executing a command and storing its output.

Example:

```bash
VAR=$(command)
```

---

# Key Takeaways

- Variables improve reusability and maintainability.
- Variables can be declared inside scripts.
- Values can be passed using command-line arguments.
- User input can be captured using `read`.
- Environment variables are created using `export`.
- Permanent variables are commonly stored in `.bashrc`.
- Command substitution stores command output into variables.
- Positional parameters `$1`, `$2`, `$3` are commonly used in shell scripts.
- Variables are fundamental to DevOps automation and scripting.