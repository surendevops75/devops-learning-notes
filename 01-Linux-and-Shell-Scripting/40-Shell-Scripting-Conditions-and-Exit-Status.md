# Shell Scripting Conditions and Exit Status

## Introduction

Conditions help a script make decisions.

In real life, we make decisions every day:

```text
If it is raining
    Take an umbrella
Else
    Take sunglasses
```

The same concept is used in shell scripting.

A script evaluates a condition and executes different actions based on the result.

---

# Why Conditions?

Without conditions:

```text
Execute Everything
```

With conditions:

```text
Make Decisions
Execute Appropriate Actions
```

---

# Algorithm Before Coding

Before writing code:

```text
Think First
Code Later
```

A good programmer spends more time thinking about the solution than writing code.

---

# Example: Umbrella or Sunglasses

Algorithm:

```text
If it is raining
    Take umbrella
Else
    Take sunglasses
```

---

# Example: Server Health

Algorithm:

```text
Check Service Status

If Service Running
    Continue
Else
    Restart Service
```

---

# If Statement

Used when a block should execute only when a condition is true.

Syntax:

```bash
if [ condition ]; then
    commands
fi
```

---

# Example

```bash
NUMBER=5

if [ $NUMBER -lt 10 ]; then
    echo "Number is less than 10"
fi
```

Output:

```text
Number is less than 10
```

---

# If-Else Statement

Used when two possible outcomes exist.

Syntax:

```bash
if [ condition ]; then
    commands
else
    commands
fi
```

---

# Example

```bash
NUMBER=20

if [ $NUMBER -lt 10 ]; then
    echo "Less than 10"
else
    echo "Greater than or equal to 10"
fi
```

Output:

```text
Greater than or equal to 10
```

---

# If-Elif-Else Statement

Used when multiple conditions need to be checked.

Syntax:

```bash
if [ condition1 ]; then
    commands
elif [ condition2 ]; then
    commands
else
    commands
fi
```

---

# Example

```bash
MARKS=75

if [ $MARKS -ge 90 ]; then
    echo "Grade A"
elif [ $MARKS -ge 70 ]; then
    echo "Grade B"
else
    echo "Grade C"
fi
```

---

# Comparison Operators

## Numeric Comparisons

| Operator | Meaning |
|-----------|----------|
| -eq | Equal |
| -ne | Not Equal |
| -gt | Greater Than |
| -lt | Less Than |
| -ge | Greater Than or Equal |
| -le | Less Than or Equal |

---

# Examples

Equal:

```bash
[ 10 -eq 10 ]
```

---

Not Equal:

```bash
[ 10 -ne 5 ]
```

---

Greater Than:

```bash
[ 20 -gt 10 ]
```

---

Less Than:

```bash
[ 5 -lt 10 ]
```

---

# String Comparisons

Equal:

```bash
[ "$NAME" = "DevOps" ]
```

---

Not Equal:

```bash
[ "$NAME" != "AWS" ]
```

---

# Negation Operator

Symbol:

```text
!
```

Meaning:

```text
NOT
```

---

# Example

```bash
if ! systemctl is-active nginx >/dev/null; then
    echo "Nginx is not running"
fi
```

---

# Even or Odd Number

Problem:

```text
Determine whether a number is even or odd.
```

---

# Algorithm

```text
Take Number

Divide by 2

Get Remainder

If Remainder = 0
    Even
Else
    Odd
```

---

# Example

```text
14 / 2

Remainder = 0

Even
```

---

```text
15 / 2

Remainder = 1

Odd
```

---

# Shell Script Example

```bash
NUMBER=15

if [ $((NUMBER % 2)) -eq 0 ]; then
    echo "Even Number"
else
    echo "Odd Number"
fi
```

---

# Prime Number Logic

A prime number is divisible only by:

```text
1
and
itself
```

Examples:

```text
2
3
5
7
11
13
```

---

# Prime Number Algorithm

```text
Take Number

Check divisibility from 2 to Number-1

If divisible by any number
    Not Prime

Otherwise
    Prime
```

---

# Real World DevOps Example

Before installing software:

```text
Check Root Access

If Root
    Proceed

Else
    Stop
```

---

# Exit Status

Every Linux command returns an exit status.

Example:

```bash
dnf install mysql -y
```

After execution:

```bash
echo $?
```

---

# What is $?

```bash
$?
```

stores:

```text
Previous Command Exit Status
```

---

# Exit Status Values

| Exit Code | Meaning |
|------------|----------|
| 0 | Success |
| 1-127 | Failure |

---

# Example

Successful Command:

```bash
pwd
```

Check Status:

```bash
echo $?
```

Output:

```text
0
```

---

# Failed Command

```bash
abcd
```

Check Status:

```bash
echo $?
```

Output:

```text
127
```

---

# Why Exit Status Matters?

Automation should know:

```text
Did the command succeed?
Or fail?
```

---

# Example Logic

```text
Install MySQL

If Installation Successful
    Continue

Else
    Stop
```

---

# Installation Validation Example

```bash
dnf install mysql -y

if [ $? -eq 0 ]; then
    echo "Installation Successful"
else
    echo "Installation Failed"
fi
```

---

# Root User Validation

Administrative tasks usually require root privileges.

Examples:

```bash
dnf install nginx -y
```

```bash
useradd devops
```

```bash
systemctl restart nginx
```

---

# Check Current User ID

```bash
id -u
```

---

# Understanding Output

Root User:

```text
0
```

---

Normal User:

```text
1000
1001
1002
```

---

# Root Validation Algorithm

```text
Check User ID

If User ID = 0
    Proceed

Else
    Stop Script
```

---

# Example Script

```bash
if [ $(id -u) -ne 0 ]; then
    echo "Please run with root access"
    exit 1
fi
```

---

# Why Stop the Script?

If root access is unavailable:

```text
Package Installation Fails
User Creation Fails
Service Management Fails
```

Stopping early saves time.

---

# Common DevOps Example

Install Multiple Packages:

```bash
dnf install mysql -y
dnf install nginx -y
dnf install mongodb-mongosh -y
```

Before running:

```text
Validate Root Access
```

---

# Good Scripting Practices

## Think Before Coding

Focus on:

```text
Problem Solving
Algorithm
Logic
```

before writing code.

---

## Keep Scripts Readable

Bad:

```bash
x=10
```

Good:

```bash
SERVER_COUNT=10
```

---

## Avoid Unnecessary Code

Smaller scripts are easier to maintain.

---

## Validate Before Proceeding

Check:

- User Access
- Dependencies
- Command Success

---

# Frequently Asked Interview Questions

## What is a Condition?

A mechanism used to make decisions in scripts.

---

## What is an If Statement?

Executes commands only when a condition is true.

---

## Difference Between If and If-Else?

If:

```text
One Possible Path
```

If-Else:

```text
Two Possible Paths
```

---

## What is Exit Status?

Return code of a command.

---

## What Does Exit Code 0 Mean?

```text
Success
```

---

## What Does Exit Code 127 Mean?

```text
Command Not Found
```

---

## How Do You Check Previous Command Status?

```bash
echo $?
```

---

## How Do You Verify Root Access?

```bash
id -u
```

---

## What Does id -u Return for Root User?

```text
0
```

---

## Why Are Conditions Important?

They help scripts make decisions and automate workflows safely.

---

# Key Takeaways

- Conditions are used to make decisions in shell scripts.
- Always think about the algorithm before coding.
- If, If-Else, and If-Elif are the primary conditional statements.
- Comparison operators help evaluate conditions.
- The `!` operator represents logical NOT.
- Every Linux command returns an exit status.
- Exit code `0` means success.
- Exit codes `1-127` generally indicate failure.
- `$?` stores the previous command's exit status.
- Root validation using `id -u` is a common DevOps scripting practice.
- Conditions and exit status checks are essential for reliable automation.