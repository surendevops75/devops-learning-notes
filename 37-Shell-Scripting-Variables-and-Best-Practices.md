# Shell Scripting Variables and Best Practices

## Introduction

Shell scripting is one of the most important skills for DevOps Engineers.

A script is a collection of Linux commands executed in a sequence.

Instead of manually executing commands one by one:

```bash
dnf install nginx -y
systemctl start nginx
systemctl enable nginx
```

We can automate them using a shell script.

---

# What is a Shell Script?

A shell script is a text file containing Linux commands.

Example:

```bash
#!/bin/bash

echo "Hello World"
```

File:

```text
hello.sh
```

Execution:

```bash
sh hello.sh
```

or

```bash
bash hello.sh
```

---

# Shebang

The first line of a shell script is called:

```text
Shebang
```

Example:

```bash
#!/bin/bash
```

---

# Purpose of Shebang

It tells Linux:

```text
Which Interpreter Should Execute This Script
```

Example:

```bash
#!/bin/bash
```

means:

```text
Execute Using Bash Shell
```

---

# Why Should Shebang Be First Line?

Linux reads the first line to determine:

```text
Interpreter
Execution Method
```

Therefore:

```bash
#!/bin/bash
```

must always be the first line.

---

# Shells in Linux

Linux supports multiple shells.

Examples:

- Bash Shell
- Korn Shell (ksh)
- C Shell (csh)
- Z Shell (zsh)

---

# Bash Shell

Most commonly used shell.

Example:

```bash
#!/bin/bash
```

---

# sh vs bash

In most Linux systems:

```text
sh
```

and

```text
bash
```

behave similarly.

Examples:

```bash
sh script.sh
```

```bash
bash script.sh
```

---

# Shell Script Extension

Common extension:

```text
.sh
```

Examples:

```text
deploy.sh
backup.sh
install.sh
```

---

# What is a Variable?

A variable is a container that stores a value.

Example:

```text
Box
```

can store:

```text
Books
Pens
Money
```

Similarly:

```text
Variable
```

stores data.

---

# Why Variables?

Without variables:

```bash
echo "catalogue"
systemctl restart catalogue
journalctl -u catalogue
```

Repeated value:

```text
catalogue
```

appears multiple times.

---

# Using Variables

```bash
COMPONENT=catalogue

echo $COMPONENT

systemctl restart $COMPONENT
```

Now only one place needs modification.

---

# Variable Syntax

```bash
VAR_NAME=VALUE
```

Example:

```bash
NAME=Ramesh
```

---

# Access Variable

```bash
echo $NAME
```

Output:

```text
Ramesh
```

---

# Multiple Variables

```bash
NAME=Ramesh
CITY=Hyderabad
ROLE=DevOps
```

---

# Display Variables

```bash
echo $NAME
echo $CITY
echo $ROLE
```

---

# Variable Naming Rules

Use:

```text
A-Z
a-z
0-9
_
```

---

## Good Examples

```bash
APP_NAME=catalogue

DB_HOST=mysql

ENVIRONMENT=dev
```

---

## Bad Examples

```bash
APP-NAME=catalogue

DB HOST=mysql
```

Spaces and special characters should be avoided.

---

# Why Variables Are Important

Variables improve:

- Reusability
- Flexibility
- Readability
- Maintainability

---

# Reusability

Example:

```bash
COMPONENT=user
```

Later:

```bash
COMPONENT=cart
```

Same script can be reused.

---

# Flexibility

Changing one variable updates behavior everywhere.

Example:

```bash
ENV=dev
```

Change to:

```bash
ENV=prod
```

without modifying entire script.

---

# Readability

Compare:

Without variables:

```bash
systemctl restart catalogue
journalctl -u catalogue
```

With variables:

```bash
COMPONENT=catalogue

systemctl restart $COMPONENT
journalctl -u $COMPONENT
```

Easier to understand.

---

# Maintainability

Future changes become simpler.

Instead of modifying multiple locations:

```bash
catalogue
catalogue
catalogue
catalogue
```

modify one variable.

---

# DRY Principle

DRY means:

```text
Don't Repeat Yourself
```

---

# Why DRY?

Avoid:

```text
Duplicate Logic
Duplicate Values
Duplicate Code
```

---

# Example Without DRY

```bash
echo "catalogue"

systemctl restart catalogue

journalctl -u catalogue
```

---

# Example With DRY

```bash
COMPONENT=catalogue

echo $COMPONENT

systemctl restart $COMPONENT

journalctl -u $COMPONENT
```

---

# Real World Example

Mathematics:

Given:

```text
x = 10
y = 20
```

Formula:

```text
x + y
```

Variables allow values to be substituted later.

---

# Common DevOps Variables

## Application Name

```bash
APP_NAME=frontend
```

---

## Database Host

```bash
DB_HOST=mysql.example.com
```

---

## Environment

```bash
ENV=production
```

---

## Port

```bash
PORT=8080
```

---

# Variable Expansion

Example:

```bash
NAME=Ramesh
```

Use:

```bash
echo Hello $NAME
```

Output:

```text
Hello Ramesh
```

---

# Using Curly Braces

Example:

```bash
NAME=Ramesh
```

```bash
echo ${NAME}
```

Output:

```text
Ramesh
```

---

# Command Substitution

Store command output.

Example:

```bash
DATE=$(date)
```

Display:

```bash
echo $DATE
```

---

# Read User Input

```bash
echo "Enter Name"

read NAME

echo "Welcome $NAME"
```

---

# Environment Variables

System-wide variables.

View:

```bash
env
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

Current Path:

```bash
echo $PATH
```

---

# Best Practices

## Use Meaningful Names

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

Bad:

```bash
mysql.example.com
```

Good:

```bash
DB_HOST=mysql.example.com
```

---

## Follow DRY Principle

Avoid repeating values.

---

## Use Uppercase for Constants

Example:

```bash
APP_NAME=frontend
```

---

## Add Comments

Example:

```bash
# Restart application
systemctl restart frontend
```

---

# Frequently Asked Interview Questions

## What is a Shell Script?

A file containing Linux commands executed sequentially.

---

## What is Shebang?

The first line of a script specifying the interpreter.

Example:

```bash
#!/bin/bash
```

---

## What is a Variable?

A container used to store data.

---

## What is Variable Syntax?

```bash
VAR_NAME=VALUE
```

---

## What is DRY?

Don't Repeat Yourself.

---

## Why Use Variables?

- Reusability
- Flexibility
- Readability
- Maintainability

---

## How Do You Display a Variable?

```bash
echo $VAR_NAME
```

---

## How Do You Read User Input?

```bash
read VARIABLE
```

---

## What Command Displays Environment Variables?

```bash
env
```

---

# Key Takeaways

- Shell scripts automate Linux operations.
- Every script should start with a shebang.
- Bash is the most commonly used shell.
- Variables store reusable values.
- Variables improve readability and maintainability.
- DRY means "Don't Repeat Yourself".
- Avoid hardcoding values in scripts.
- Use meaningful variable names.
- Environment variables provide system information.
- Variables are fundamental to writing reusable DevOps automation scripts.