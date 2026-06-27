# Shell Scripting Data Types and Arrays

## Introduction

Programming languages use data types to represent different kinds of data.

Examples:

```text
Name        → String
Age         → Integer
Salary      → Decimal
Status      → Boolean
Grade       → Character
```

Most programming languages require explicit data type declarations.

Shell scripting works differently.

---

# What is a Data Type?

A data type defines:

```text
What kind of value can be stored
How much memory is allocated
What operations can be performed
```

---

# Common Data Types

## String

Stores text.

Examples:

```text
DevOps
Linux
Shell
AWS
Sivakumar Reddy
```

---

## Integer

Stores whole numbers.

Examples:

```text
10
100
5000
-25
```

---

## Float

Stores decimal values.

Examples:

```text
12.22
33.45
100.99
```

---

## Decimal

Stores highly precise decimal numbers.

Examples:

```text
33.836419859
58914710.59147590
```

Commonly used in:

```text
Banking
Financial Applications
Scientific Calculations
```

---

## Boolean

Represents:

```text
true
false
```

Examples:

```text
Application Running → true
Application Stopped → false
```

---

## Character

Stores a single character.

Examples:

```text
A
B
C
D
```

---

## Long

Stores very large numbers.

Examples:

```text
9873471328
158967547
```

---

## Complex Number

Contains real and imaginary parts.

Example:

```text
3 + 4i
```

---

## Logarithmic Values

Example:

```text
log34
```

Mostly used in mathematical calculations.

---

# Data Types in Programming Languages

Example:

```java
int i = 0;
int j = 0;
```

---

Another Example:

```java
float salary = 12500.50;
```

---

Example:

```java
long population = 9873471328;
```

---

# Explicit Data Types

Languages such as:

- Java
- C
- C++
- Go

require explicit data type declarations.

Example:

```java
int age = 25;
```

---

# Shell Scripting Behavior

Shell scripting does not have strict data types.

Everything is treated as:

```text
String
```

by default.

---

# Example

```bash
NAME=DevOps
```

Shell treats it as:

```text
String
```

---

Example:

```bash
AGE=25
```

Shell still stores it as:

```text
String
```

---

Example:

```bash
SALARY=50000.25
```

Shell still treats it as:

```text
String
```

---

# Why Does Shell Treat Everything as String?

Shell is primarily designed for:

```text
Command Execution
Automation
System Administration
```

rather than complex programming.

---

# Example

```bash
NUM1=10
NUM2=20
```

Stored internally as strings.

---

# Arithmetic Operations

Shell provides arithmetic expansion.

Example:

```bash
NUM1=10
NUM2=20

echo $((NUM1 + NUM2))
```

Output:

```text
30
```

---

# String Example

```bash
NAME="DevOps"

echo $NAME
```

Output:

```text
DevOps
```

---

# Arrays in Shell Scripting

Arrays allow storing multiple values in a single variable.

---

# Why Arrays?

Without arrays:

```bash
COURSE1=Linux
COURSE2=Shell
COURSE3=Docker
```

Many variables become difficult to manage.

---

Using arrays:

```bash
COURSES=("Linux" "Shell" "Docker")
```

Single variable stores multiple values.

---

# Array Declaration

Syntax:

```bash
ARRAY_NAME=("value1" "value2" "value3")
```

Example:

```bash
TECHNOLOGIES=("Linux" "Shell" "Docker")
```

---

# Access Entire Array

```bash
echo ${TECHNOLOGIES[@]}
```

Output:

```text
Linux Shell Docker
```

---

# Array Indexing

Array indexes start from:

```text
0
```

---

Example:

```bash
TECHNOLOGIES=("Linux" "Shell" "Docker")
```

---

Index Mapping

```text
0 → Linux
1 → Shell
2 → Docker
```

---

# Access First Element

```bash
echo ${TECHNOLOGIES[0]}
```

Output:

```text
Linux
```

---

# Access Second Element

```bash
echo ${TECHNOLOGIES[1]}
```

Output:

```text
Shell
```

---

# Access Third Element

```bash
echo ${TECHNOLOGIES[2]}
```

Output:

```text
Docker
```

---

# Array Example

```bash
#!/bin/bash

COURSES=("Linux" "Shell" "Docker")

echo ${COURSES[0]}
echo ${COURSES[1]}
echo ${COURSES[2]}
```

Output:

```text
Linux
Shell
Docker
```

---

# Array Length

Find total elements:

```bash
echo ${#COURSES[@]}
```

Output:

```text
3
```

---

# Add Element to Array

```bash
COURSES+=("Kubernetes")
```

---

Display:

```bash
echo ${COURSES[@]}
```

Output:

```text
Linux Shell Docker Kubernetes
```

---

# Loop Through Array

```bash
for COURSE in "${COURSES[@]}"
do
    echo $COURSE
done
```

Output:

```text
Linux
Shell
Docker
```

---

# Real World Example

Store server names:

```bash
SERVERS=("web1" "web2" "web3")
```

---

Loop through servers:

```bash
for SERVER in "${SERVERS[@]}"
do
    echo $SERVER
done
```

---

Output:

```text
web1
web2
web3
```

---

# Practical DevOps Examples

## Technologies List

```bash
TECHS=("Linux" "AWS" "Docker" "Kubernetes")
```

---

## Environment List

```bash
ENVS=("dev" "qa" "prod")
```

---

## Application Servers

```bash
APPS=("catalogue" "user" "cart")
```

---

# Common Mistakes

## Wrong Index

```bash
COURSES=("Linux" "Shell")

echo ${COURSES[5]}
```

Output:

```text
Blank
```

---

## Missing Quotes

Incorrect:

```bash
COURSES=(Linux Shell Docker)
```

Works in many cases but using quotes is safer.

Preferred:

```bash
COURSES=("Linux" "Shell" "Docker")
```

---

# Data Types vs Shell Variables

| Programming Languages | Shell Script |
|------------------------|-------------|
| int | String |
| float | String |
| boolean | String |
| char | String |
| long | String |

Shell converts values when required.

---

# Frequently Asked Interview Questions

## What Are Data Types?

Classifications of data such as strings, integers, booleans, and decimals.

---

## Does Shell Script Have Explicit Data Types?

No.

Shell treats everything as strings by default.

---

## What Is an Integer?

A whole number.

Example:

```text
10
20
100
```

---

## What Is a String?

Text data.

Example:

```text
Linux
AWS
DevOps
```

---

## What Is a Boolean?

A value representing:

```text
true
false
```

---

## What Is an Array?

A variable that stores multiple values.

---

## What Is the First Index of an Array?

```text
0
```

---

## How Do You Access the First Array Element?

```bash
${ARRAY[0]}
```

---

## How Do You Access All Array Elements?

```bash
${ARRAY[@]}
```

---

## How Do You Find Array Length?

```bash
${#ARRAY[@]}
```

---

# Key Takeaways

- Data types classify different kinds of values.
- Common data types include String, Integer, Float, Decimal, Boolean, Character, and Long.
- Shell scripting does not enforce explicit data types.
- Everything is treated as a string by default.
- Arrays allow storing multiple values in a single variable.
- Array indexing starts from 0.
- Arrays are commonly used for server lists, environments, and application names.
- Understanding arrays is important for DevOps automation and shell scripting.