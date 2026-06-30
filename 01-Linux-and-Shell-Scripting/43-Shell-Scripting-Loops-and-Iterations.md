# Shell Scripting Loops and Iterations

## Introduction

Until now we have learned:

- Variables
- Data Types
- Conditions
- Functions
- Exit Status
- Redirections
- Logs
- Colors

The next important concept is:

```text
Loops
```

Loops help execute the same block of code multiple times without writing it repeatedly.

---

# What is a Loop?

A loop is a programming construct that repeatedly executes a set of statements until a condition is met.

Instead of writing:

```bash
echo 1
echo 2
echo 3
echo 4
echo 5
```

We can use a loop.

---

# Real World Example

Imagine a Xerox Machine.

For:

```text
3 Copies
```

You can manually press:

```text
Copy
Copy
Copy
```

But for:

```text
300 Copies
```

Manual work is impractical.

An automatic Xerox machine performs:

```text
Repeat Copy Operation
```

until all copies are printed.

This is exactly how loops work.

---

# Why Loops?

Without loops:

```text
More Code
More Effort
More Maintenance
```

With loops:

```text
Less Code
Less Duplication
Dynamic Execution
Easy Maintenance
```

---

# Benefits of Loops

## Reduce Code Duplication

Instead of:

```bash
echo 1
echo 2
echo 3
echo 4
echo 5
```

Use:

```bash
for i in 1 2 3 4 5
do
    echo $i
done
```

---

## Easy Maintenance

Change one loop.

Instead of modifying multiple lines.

---

## Dynamic Workloads

Example:

```text
10 Servers
100 Servers
1000 Servers
```

The same loop can handle all cases.

---

## Automation

Loops are heavily used in:

- Deployments
- Backups
- Monitoring
- Log Processing
- Server Management

---

# Iteration

Each execution of a loop is called:

```text
Iteration
```

Example:

```bash
for i in 1 2 3
```

Iterations:

```text
Iteration 1 → i=1
Iteration 2 → i=2
Iteration 3 → i=3
```

---

# For Loop

Most commonly used loop.

Used when the number of iterations is known.

---

# For Loop Syntax

```bash
for variable in values
do
    commands
done
```

---

# Example

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

# Variable Behavior

Example:

```bash
for NAME in Linux Docker Kubernetes
do
    echo $NAME
done
```

Output:

```text
Linux
Docker
Kubernetes
```

---

# Numeric Loop

Example:

```bash
for i in {1..10}
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
6
7
8
9
10
```

---

# Print Numbers 0 to 99

```bash
for i in {0..99}
do
    echo $i
done
```

---

# C Style For Loop

Similar to Java, C, and C++.

---

# Example

```bash
for (( i=0; i<10; i++ ))
do
    echo $i
done
```

Output:

```text
0
1
2
3
4
5
6
7
8
9
```

---

# Understanding C Style Loop

Initialization:

```bash
i=0
```

---

Condition:

```bash
i<10
```

---

Increment:

```bash
i++
```

---

# Comparison with Java

Java:

```java
for(int i=0;i<10;i++){
    System.out.println(i);
}
```

Shell:

```bash
for (( i=0; i<10; i++ ))
do
    echo $i
done
```

---

# While Loop

Used when the number of iterations is unknown.

Executes while a condition remains true.

---

# Syntax

```bash
while [ condition ]
do
    commands
done
```

---

# Example

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

# How While Loop Works

Step 1:

```text
Check Condition
```

Step 2:

```text
Execute Commands
```

Step 3:

```text
Update Variable
```

Step 4:

```text
Repeat
```

---

# Until Loop

Opposite of while loop.

Runs until condition becomes true.

---

# Syntax

```bash
until [ condition ]
do
    commands
done
```

---

# Example

```bash
COUNT=1

until [ $COUNT -gt 5 ]
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

# Infinite Loop

A loop that never ends.

Example:

```bash
while true
do
    echo "Running..."
done
```

---

# Stopping Infinite Loop

Press:

```text
Ctrl + C
```

---

# Loop Control Statements

## Break

Immediately exits loop.

---

# Example

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

# Continue

Skips current iteration.

---

# Example

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

# Real DevOps Example

Install Multiple Packages.

Without Loop:

```bash
dnf install nginx -y
dnf install mysql -y
dnf install git -y
dnf install docker -y
```

---

With Loop:

```bash
for PACKAGE in nginx mysql git docker
do
    dnf install $PACKAGE -y
done
```

---

# Server Management Example

```bash
for SERVER in web1 web2 web3
do
    echo "Connecting to $SERVER"
done
```

---

# Backup Example

```bash
for FILE in *.log
do
    cp $FILE /backup
done
```

---

# Monitoring Example

```bash
for SERVICE in nginx mysql docker
do
    systemctl status $SERVICE
done
```

---

# Common Mistakes

## Infinite Loop

```bash
COUNT=1

while [ $COUNT -le 5 ]
do
    echo $COUNT
done
```

Problem:

```text
COUNT never changes.
```

Infinite loop occurs.

---

## Wrong Condition

```bash
while [ $COUNT -gt 5 ]
```

May never execute.

---

# Performance Considerations

Loops do not automatically improve performance.

Benefits are:

```text
Less Effort
Less Code Duplication
Better Maintainability
Scalability
```

---

# Best Practices

## Use Meaningful Variable Names

Good:

```bash
PACKAGE
SERVER
SERVICE
```

Bad:

```bash
x
y
z
```

---

## Avoid Infinite Loops

Always ensure exit conditions exist.

---

## Keep Logic Simple

Complex nested loops are difficult to maintain.

---

## Combine with Functions

Loops and functions work very well together.

---

# Frequently Asked Interview Questions

## What is a Loop?

A mechanism used to execute a block of code repeatedly.

---

## What is Iteration?

A single execution of a loop.

---

## Types of Loops in Shell Scripting?

- For Loop
- While Loop
- Until Loop

---

## When Should You Use a For Loop?

When the number of iterations is known.

---

## When Should You Use a While Loop?

When execution depends on a condition.

---

## What is an Infinite Loop?

A loop that never terminates.

---

## What Does Break Do?

Exits the loop immediately.

---

## What Does Continue Do?

Skips the current iteration.

---

## Why Are Loops Important in DevOps?

They help automate repetitive tasks such as:

- Server Management
- Package Installation
- Monitoring
- Deployments

---

# Key Takeaways

- Loops automate repetitive tasks.
- Each execution of a loop is called an iteration.
- For loops are used when iterations are known.
- While loops execute while a condition is true.
- Until loops execute until a condition becomes true.
- Break exits a loop.
- Continue skips an iteration.
- Loops reduce code duplication.
- Loops improve maintainability and scalability.
- Loops are heavily used in DevOps automation scripts.