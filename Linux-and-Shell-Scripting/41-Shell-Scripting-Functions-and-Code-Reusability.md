# Shell Scripting Functions and Code Reusability

## Introduction

As shell scripts grow larger, repeating the same code multiple times becomes difficult to maintain.

Example:

```text
Install MySQL
Check Status

Install Nginx
Check Status

Install MongoDB
Check Status
```

The same logic is repeated again and again.

Functions help solve this problem.

---

# What is a Function?

A function is a reusable block of code that performs a specific task.

Think of a function as:

```text
Input
  │
  ▼
Processing
  │
  ▼
Output
```

---

# Real World Example

Consider a calculator.

```text
Input:
10 + 20

Processing:
Addition

Output:
30
```

The addition logic is written once and reused many times.

A function works the same way.

---

# Why Functions?

Functions help achieve:

- Reusability
- Maintainability
- Readability
- DRY Principle
- Better Organization

---

# DRY Principle

DRY means:

```text
Don't Repeat Yourself
```

Avoid writing the same code repeatedly.

---

# Example Without Function

```bash
dnf install mysql -y

if [ $? -eq 0 ]; then
    echo "Installation Success"
else
    echo "Installation Failed"
fi

dnf install nginx -y

if [ $? -eq 0 ]; then
    echo "Installation Success"
else
    echo "Installation Failed"
fi
```

The same validation logic is duplicated.

---

# Example With Function

```bash
VALIDATE(){
    if [ $? -eq 0 ]; then
        echo "Success"
    else
        echo "Failed"
    fi
}
```

Now the function can be reused everywhere.

---

# Function Syntax

```bash
function_name(){

}
```

---

# Alternative Syntax

```bash
function function_name {

}
```

Both are valid.

---

# Simple Function Example

```bash
greet(){

    echo "Hello DevOps"

}
```

---

# Calling a Function

Defining a function does not execute it.

You must call it.

```bash
greet
```

Output:

```text
Hello DevOps
```

---

# Function Execution Flow

```text
Script
   │
   ▼
Call Function
   │
   ▼
Execute Function Code
   │
   ▼
Return to Script
```

---

# Multiple Function Calls

```bash
greet
greet
greet
```

Output:

```text
Hello DevOps
Hello DevOps
Hello DevOps
```

---

# Why Reusability Matters?

Without functions:

```text
Copy
Paste
Copy
Paste
Copy
Paste
```

Problems:

- Difficult Maintenance
- More Bugs
- Large Scripts

---

With functions:

```text
Write Once
Use Many Times
```

---

# Function Inputs

Functions can accept values.

Example:

```bash
greet(){

    echo "Hello $1"

}
```

Call:

```bash
greet DevOps
```

Output:

```text
Hello DevOps
```

---

# Multiple Inputs

```bash
greet(){

    echo "$1 and $2"

}
```

Call:

```bash
greet Linux Docker
```

Output:

```text
Linux and Docker
```

---

# Function Parameters

Inside functions:

| Variable | Meaning |
|-----------|----------|
| $1 | First Input |
| $2 | Second Input |
| $3 | Third Input |
| $# | Total Inputs |
| $@ | All Inputs |

---

# Example

```bash
details(){

    echo "Name: $1"
    echo "Role: $2"

}
```

Call:

```bash
details Ramesh DevOps
```

Output:

```text
Name: Ramesh
Role: DevOps
```

---

# Function Return Values

Functions can return an exit status.

Example:

```bash
my_function(){

    return 0

}
```

---

# Return Codes

```text
0     → Success

1-255 → Failure
```

---

# Example

```bash
check(){

    return 0

}
```

Call:

```bash
check

echo $?
```

Output:

```text
0
```

---

# Production Example

Package Installation.

Without function:

```bash
dnf install mysql -y

if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed"
fi

dnf install nginx -y

if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed"
fi
```

---

# Production Example With Function

```bash
VALIDATE(){

    if [ $1 -eq 0 ]; then
        echo "Success"
    else
        echo "Failed"
    fi

}
```

Usage:

```bash
dnf install mysql -y

VALIDATE $?
```

---

```bash
dnf install nginx -y

VALIDATE $?
```

---

# Common DevOps Function Example

```bash
CHECK_ROOT(){

    if [ $(id -u) -ne 0 ]; then
        echo "Run as root"
        exit 1
    fi

}
```

Call:

```bash
CHECK_ROOT
```

---

# Function Naming Best Practices

Use meaningful names.

Good:

```bash
INSTALL_PACKAGE

CHECK_ROOT

VALIDATE
```

---

Bad:

```bash
a

abc

x
```

---

# Function Organization

Typically functions are placed at the top of scripts.

Example:

```bash
#!/bin/bash

CHECK_ROOT(){

}

VALIDATE(){

}

main_logic
```

---

# Readability

Without functions:

```text
Huge Script
Repeated Logic
Hard to Understand
```

---

With functions:

```text
Small Blocks
Clear Purpose
Easy Maintenance
```

---

# Maintainability

Suppose success message changes.

Without functions:

```text
Change in 20 Locations
```

---

With functions:

```text
Change in One Location
```

---

# Real World DevOps Use Cases

## Package Installation

```bash
INSTALL_PACKAGE
```

---

## Service Validation

```bash
CHECK_SERVICE
```

---

## Root Access Validation

```bash
CHECK_ROOT
```

---

## Application Health Check

```bash
CHECK_HEALTH
```

---

## Database Connectivity

```bash
CHECK_DB
```

---

# Common Mistakes

## Forgetting to Call Function

Defined:

```bash
greet(){

    echo "Hello"

}
```

No output until:

```bash
greet
```

is executed.

---

## Poor Naming

Bad:

```bash
a(){

}
```

Good:

```bash
INSTALL_NGINX(){

}
```

---

## Duplicate Logic

Avoid repeating the same code.

Convert repeated logic into functions.

---

# Frequently Asked Interview Questions

## What is a Function?

A reusable block of code that performs a specific task.

---

## Why Use Functions?

- Reusability
- Maintainability
- Readability
- DRY Principle

---

## What is DRY?

```text
Don't Repeat Yourself
```

---

## How Do You Define a Function?

```bash
function_name(){

}
```

---

## How Do You Call a Function?

```bash
function_name
```

---

## Can Functions Accept Inputs?

Yes.

Using:

```bash
$1
$2
$3
```

---

## Can Functions Return Values?

Yes.

Using:

```bash
return
```

---

## What Does return 0 Mean?

```text
Success
```

---

## Why Are Functions Important in DevOps?

Functions reduce duplicate code and make automation scripts easier to manage.

---

# Key Takeaways

- Functions are reusable blocks of code.
- Functions improve readability and maintainability.
- DRY means "Don't Repeat Yourself".
- Functions help reduce duplicate logic.
- Functions can accept inputs using positional parameters.
- Functions can return exit codes.
- Meaningful function names improve script quality.
- Functions are heavily used in production-grade DevOps automation.
- Most large shell scripts are built using multiple reusable functions.