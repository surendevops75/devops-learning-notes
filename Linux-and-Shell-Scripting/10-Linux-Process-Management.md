# Linux Process Management

## Introduction

A Process is a running instance of a program.

Whenever a command is executed in Linux, the operating system creates a process and assigns a unique Process ID (PID).

Example:

```bash
ls -l
```

Linux creates:

```text
Process
│
├── PID
└── Resources (CPU, Memory)
```

executes the command and returns the output.

---

# Real World Analogy

Consider an organization:

```text
Team Manager
│
├── Team Lead
│
├── Senior Engineer
│
├── Junior Engineer
│
└── Trainees
```

Suppose Team Lead assigns a task to a Senior Engineer.

```text
Team Lead
   │
   └── Senior Engineer
```

Team Lead becomes:

```text
Parent Process
```

Senior Engineer becomes:

```text
Child Process
```

Similarly in Linux:

```text
Parent Process
        │
        └── Child Process
```

---

# Process Terminology

## PID (Process ID)

Unique identifier assigned to every process.

Example:

```text
PID = 3456
```

---

## PPID (Parent Process ID)

The process that created another process.

Example:

```text
PPID = 1234
PID  = 3456
```

Meaning:

```text
Process 1234 created Process 3456
```

---

# Process Hierarchy

Example:

```text
systemd (PID 1)
│
├── sshd
│
├── nginx
│
├── docker
│
└── java
```

Every process originates from a parent process.

---

# Viewing Processes

## ps Command

Displays processes started by the current user.

### Syntax

```bash
ps
```

Example:

```bash
ps
```

Output:

```text
PID TTY          TIME CMD
123 pts/0    00:00:00 bash
456 pts/0    00:00:00 ps
```

---

# ps -ef Command

Displays all running processes.

### Syntax

```bash
ps -ef
```

Example:

```text
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 01:53 ?        00:00:00 /usr/lib/systemd/systemd
root           2       0  0 01:53 ?        00:00:00 [kthreadd]
root           3       2  0 01:53 ?        00:00:00 [rcu_gp]
root           4       2  0 01:53 ?        00:00:00 [rcu_par_gp]
```

---

# Understanding ps -ef Output

| Field | Meaning |
|---------|----------|
| UID | User owning process |
| PID | Process ID |
| PPID | Parent Process ID |
| C | CPU Usage |
| STIME | Start Time |
| TTY | Terminal |
| TIME | CPU Time Consumed |
| CMD | Command |

---

# Common ps Options

## ps -e

Show all processes.

```bash
ps -e
```

---

## ps -f

Full format listing.

```bash
ps -f
```

---

## ps -ef

All processes with detailed output.

```bash
ps -ef
```

---

## ps -u

Processes by specific user.

```bash
ps -u ec2-user
```

---

## ps aux

Detailed process information.

```bash
ps aux
```

---

# Searching Processes

## Find Nginx Process

```bash
ps -ef | grep nginx
```

Example:

```bash
ps -ef | grep nginx
```

Output:

```text
root 2345 1 0 nginx
```

---

## Find Java Process

```bash
ps -ef | grep java
```

---

## Find Docker Process

```bash
ps -ef | grep docker
```

---

# Foreground Process

A foreground process occupies the terminal.

Example:

```bash
sleep 100
```

Terminal becomes busy.

Cannot execute another command until completion.

```text
Current Terminal Blocked
```

---

# Background Process

Runs independently without blocking terminal.

### Example

```bash
sleep 10 &
```

Output:

```text
[1] 4567
```

Where:

```text
4567 = PID
```

Now terminal remains available.

---

# Job Control Commands

## View Background Jobs

```bash
jobs
```

---

## Move Job to Foreground

```bash
fg
```

---

## Move Job to Background

```bash
bg
```

---

# Monitoring Processes

## top Command

Displays real-time process information.

### Syntax

```bash
top
```

Displays:

- CPU Usage
- Memory Usage
- Running Processes
- Load Average

---

# top Output Overview

```text
PID
USER
CPU%
MEM%
TIME
COMMAND
```

---

# Why Monitor Processes?

## Scenario 1: High Traffic

Normal:

```text
10 Users
CPU  = 10%
RAM  = 25%
```

Increased Traffic:

```text
100 Users
CPU  = 20%
RAM  = 40%
```

Expected behavior.

---

## Scenario 2: Application Problem

Users:

```text
10
```

Resources:

```text
CPU  = 20%
RAM  = 40%
```

Unexpected increase.

Possible causes:

- Memory Leak
- Infinite Loop
- Application Bug
- Database Issue

---

# Memory Leak

A process consumes memory but does not release it.

Expected:

```text
Use Memory
Complete Task
Release Memory
```

Problem:

```text
Use Memory
Complete Task
Memory Not Released
```

Result:

```text
High RAM Usage
Server Slowdown
```

---

# Troubleshooting High Resource Usage

## Step 1

Check processes:

```bash
top
```

---

## Step 2

Identify high CPU process.

---

## Step 3

Check application logs.

Examples:

```text
Application Logs
Server Logs
Nginx Logs
Java Logs
```

---

## Step 4

Investigate root cause.

---

## Step 5

Restart service if necessary.

```bash
systemctl restart nginx
```

or

```bash
systemctl restart application
```

---

# Kill Process

Used to terminate a process.

## Syntax

```bash
kill PID
```

Example:

```bash
kill 1234
```

---

# How kill Works

```text
Request to terminate process
```

The process gets a chance to exit gracefully.

---

# Force Kill

## kill -9

### Syntax

```bash
kill -9 PID
```

Example:

```bash
kill -9 1234
```

Meaning:

```text
Immediate termination
```

No cleanup opportunity.

---

# Common Kill Signals

| Signal | Command | Purpose |
|----------|----------|----------|
| 15 | kill PID | Graceful termination |
| 9 | kill -9 PID | Force termination |
| 1 | kill -1 PID | Reload configuration |
| 19 | kill -19 PID | Pause process |
| 18 | kill -18 PID | Resume process |

---

# Kill All Processes by Name

## pkill

```bash
pkill nginx
```

---

## killall

```bash
killall nginx
```

---

# Process States

Linux processes can be in different states.

| State | Meaning |
|---------|---------|
| R | Running |
| S | Sleeping |
| D | Waiting |
| T | Stopped |
| Z | Zombie |

---

# Zombie Process

A process completed execution but still exists in process table.

Example:

```text
Parent Process
Child Process Completed
Parent Did Not Collect Status
```

Result:

```text
Zombie Process
```

---

# Orphan Process

Occurs when parent process dies before child process.

Example:

```text
Parent Dies
Child Continues Running
```

Linux reassigns it to:

```text
PID 1 (systemd)
```

---

# Useful Process Commands

## pgrep

Find process IDs.

```bash
pgrep nginx
```

---

## pidof

Get PID of process.

```bash
pidof nginx
```

---

## pstree

Display process hierarchy.

```bash
pstree
```

Example:

```text
systemd
 ├─sshd
 ├─nginx
 ├─docker
```

---

# Process and Service Relationship

Example:

```bash
systemctl start nginx
```

Creates:

```text
nginx process
```

Verify:

```bash
ps -ef | grep nginx
```

---

# Production Troubleshooting Example

Application inaccessible.

### Step 1

Check service.

```bash
systemctl status nginx
```

---

### Step 2

Check process.

```bash
ps -ef | grep nginx
```

---

### Step 3

Check resource usage.

```bash
top
```

---

### Step 4

Check logs.

```bash
journalctl -u nginx
```

---

### Step 5

Restart service.

```bash
systemctl restart nginx
```

---

# Frequently Asked Interview Questions

## What is a Process?

A running instance of a program.

---

## What is PID?

Process ID.

Unique identifier assigned to a process.

---

## What is PPID?

Parent Process ID.

Identifies the process that created another process.

---

## Difference Between Process and Program

| Program | Process |
|----------|----------|
| Static File | Running Program |
| Stored on Disk | Running in Memory |

---

## Difference Between Foreground and Background Process

| Foreground | Background |
|------------|------------|
| Blocks Terminal | Does Not Block Terminal |
| User Interaction | Runs Independently |

---

## Difference Between kill and kill -9

| Command | Behavior |
|----------|----------|
| kill PID | Graceful termination |
| kill -9 PID | Force termination |

---

## What is a Zombie Process?

Completed process waiting for parent acknowledgment.

---

## What is an Orphan Process?

A child process whose parent process has terminated.

---

## What is top Command?

Used for real-time monitoring of processes, CPU and memory usage.

---

# Key Takeaways

- Every running task is a process.
- Every process has a PID.
- Every process has a PPID.
- `ps` displays process information.
- `ps -ef` shows all running processes.
- `top` provides real-time monitoring.
- Background processes use `&`.
- `kill` sends termination request.
- `kill -9` forcefully terminates a process.
- Zombie and Orphan processes are important Linux interview topics.
- Process monitoring is a critical DevOps and Linux Administration skill.