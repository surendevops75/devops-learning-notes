# 02-Process-Management

## Linux Process Management for DevOps Engineers

Process management is one of the most important Linux fundamentals for a DevOps engineer.

When a production application is slow, consuming high CPU, using excessive memory, hanging, restarting, or refusing to stop, you need to understand:

```text
Process
PID
PPID
Process state
CPU
Memory
Threads
File descriptors
Signals
Priority
Scheduling
Services
cgroups
Containers
```

A strong troubleshooting mindset is:

```text
Observe
  ↓
Identify
  ↓
Measure
  ↓
Understand
  ↓
Act
  ↓
Verify
```

Do not kill processes simply because they consume resources. First determine whether the behavior is expected.

---

# 1. What Is a Process?

A process is a running instance of a program.

Example:

```text
nginx binary
    ↓
running nginx process
    ↓
PID 1234
```

A process has resources such as:

```text
PID
PPID
memory
CPU time
file descriptors
environment
working directory
user/group
signals
threads
scheduling priority
```

---

# 2. Program vs Process

A **program** is executable code stored on disk.

A **process** is that program while it is executing.

Example:

```text
/usr/bin/python3
```

is a program.

When you run:

```bash
python3 app.py
```

Linux creates a process.

---

# 3. PID

PID means:

```text
Process ID
```

Every running process has a PID.

Example:

```text
PID
1234
```

Find the current shell:

```bash
echo $$
```

`$$` represents the current shell's PID.

---

# 4. PPID

PPID means:

```text
Parent Process ID
```

Processes are generally created by another process.

Example:

```text
systemd
   |
   +-- sshd
        |
        +-- bash
             |
             +-- python3 app.py
```

Each child has a parent.

---

# 5. PID 1

On a normal modern Linux system, PID 1 is the first userspace process.

Often:

```text
systemd
```

Check:

```bash
ps -p 1 -o pid,comm,args
```

PID 1 has special responsibilities, including adopting orphaned processes and managing services on systemd systems.

Inside containers, PID 1 may instead be the container's application or an init process.

---

# 6. Process Tree

Use:

```bash
pstree
```

Or:

```bash
pstree -p
```

Example:

```text
systemd(1)
 ├─sshd(900)
 │  └─bash(1200)
 │     └─python3(1500)
 └─nginx(700)
    ├─nginx(701)
    └─nginx(702)
```

Process trees are extremely useful during incident investigation.

---

# 7. `ps`

The classic process inspection command:

```bash
ps
```

Current shell processes:

```bash
ps
```

All processes:

```bash
ps -ef
```

Another common format:

```bash
ps aux
```

Important point:

```text
ps -ef
```

and

```text
ps aux
```

use different output formats, but both are commonly used to inspect processes.

---

# 8. `ps -ef`

Example:

```bash
ps -ef
```

Typical columns:

```text
UID
PID
PPID
C
STIME
TTY
TIME
CMD
```

Useful for understanding:

```text
who owns process
PID
parent process
CPU time
start time
command
```

---

# 9. `ps aux`

```bash
ps aux
```

Typical columns include:

```text
USER
PID
%CPU
%MEM
VSZ
RSS
TTY
STAT
START
TIME
COMMAND
```

Useful for identifying resource-heavy processes.

---

# 10. Find a Specific Process

```bash
ps -ef | grep nginx
```

But this can also match the `grep` command itself.

Better:

```bash
pgrep -a nginx
```

Or:

```bash
pidof nginx
```

---

# 11. `pgrep`

Find processes:

```bash
pgrep nginx
```

Show PID and command:

```bash
pgrep -a nginx
```

Find by user:

```bash
pgrep -u nginx
```

This is usually cleaner than:

```bash
ps -ef | grep ...
```

---

# 12. `pkill`

`pkill` sends a signal to matching processes.

Example:

```bash
pkill -TERM nginx
```

Be careful with broad patterns.

Before using `pkill`, inspect:

```bash
pgrep -a nginx
```

Do not use broad process matching blindly in production.

---

# 13. `pidof`

```bash
pidof nginx
```

Output may look like:

```text
702 701
```

This indicates multiple matching processes.

---

# 14. Process State

`ps` can show process states.

Example:

```bash
ps -eo pid,ppid,stat,comm
```

Common state codes:

```text
R = Running/runnable
S = Interruptible sleep
D = Uninterruptible sleep
T = Stopped
Z = Zombie
I = Idle kernel thread
```

Additional flags can appear beside the main state.

---

# 15. Running State

`R` means the process is running or runnable.

Example:

```text
PID   STAT   COMMAND
1200  R      python3
```

A process can be runnable without currently executing on a CPU because another runnable task may be scheduled first.

---

# 16. Sleeping State

`S` generally means interruptible sleep.

Many normal applications spend most of their time sleeping while waiting for:

```text
I/O
network
timers
events
locks
```

A sleeping process is not necessarily unhealthy.

---

# 17. Uninterruptible Sleep

`D` commonly indicates uninterruptible sleep, often associated with kernel I/O waits.

Potential causes include:

```text
disk I/O
network filesystem
storage problems
kernel-level waits
```

If many processes remain in `D` state, investigate the underlying I/O system rather than repeatedly killing applications.

---

# 18. Zombie Process

A zombie is a process that has exited but whose parent has not yet collected its exit status.

Example:

```text
Parent
  |
  +-- Zombie
```

A zombie does not continue executing application code.

---

# 19. Find Zombies

```bash
ps -eo pid,ppid,stat,comm | awk '$3 ~ /Z/'
```

Or:

```bash
ps aux | awk '$8 ~ /Z/'
```

Investigate the parent process.

---

# 20. How to Fix Zombies

Do not try to kill a zombie directly.

The process has already exited.

Investigate its parent:

```bash
ps -o pid,ppid,stat,comm -p <PID>
```

Then:

```text
understand parent behavior
       ↓
ensure parent reaps children
       ↓
restart/fix parent if appropriate
```

In some cases, terminating the parent causes PID 1 or another subreaper to adopt and reap the zombie.

---

# 21. Orphan Process

An orphan is a process whose original parent has exited.

Linux reassigns orphaned processes to an appropriate reaper, traditionally PID 1.

Example:

```text
Parent
  |
  +-- Child

Parent exits

Child
  |
  v
adopted by reaper
```

Orphans are not automatically bad.

---

# 22. Process Tree Troubleshooting

When a process behaves unexpectedly, inspect:

```bash
pstree -aps <PID>
```

This can reveal:

```text
who started it
which service launched it
shell wrappers
supervisors
container runtimes
```

---

# 23. CPU Usage

Check:

```bash
ps aux --sort=-%cpu | head
```

This shows processes consuming high CPU.

Another useful command:

```bash
top
```

---

# 24. Memory Usage

Check:

```bash
ps aux --sort=-%mem | head
```

Useful columns:

```text
%MEM
VSZ
RSS
```

Do not interpret VSZ as actual physical memory consumption.

RSS is generally much more useful when investigating resident memory.

---

# 25. RSS vs VSZ

### VSZ

Virtual memory size.

It can include:

```text
mapped files
shared libraries
reserved virtual address space
```

### RSS

Resident Set Size.

Memory pages currently resident in physical memory.

When troubleshooting memory pressure, look at RSS alongside:

```text
shared memory
cgroups
container limits
swap
```

---

# 26. `top`

Run:

```bash
top
```

Useful information:

```text
load average
CPU usage
memory
swap
processes
PID
CPU
memory
state
```

Common interactive commands:

```text
P = CPU sort
M = memory sort
k = kill
r = renice
q = quit
```

Use carefully in production.

---

# 27. `top` CPU Meaning

The CPU section can show:

```text
us = user space
sy = kernel/system
ni = nice
id = idle
wa = I/O wait
hi = hardware interrupts
si = software interrupts
st = steal time
```

High CPU usage does not always mean the application itself is inefficient.

---

# 28. I/O Wait

High:

```text
wa
```

can indicate significant CPU time waiting for I/O.

Investigate:

```text
storage
filesystem
network filesystem
database
application I/O
```

Do not automatically blame CPU-intensive application code.

---

# 29. Load Average

Linux load average represents the number of tasks that are runnable or waiting in certain uninterruptible states.

Check:

```bash
uptime
```

or:

```bash
cat /proc/loadavg
```

Example:

```text
1.20 0.90 0.70
```

represents approximately:

```text
1 minute
5 minutes
15 minutes
```

---

# 30. Interpret Load with CPU Count

Check CPU count:

```bash
nproc
```

Suppose:

```text
CPU cores = 8
load = 2
```

That is generally less concerning than:

```text
CPU cores = 2
load = 8
```

But load must be interpreted with workload characteristics and historical baseline.

---

# 31. Process CPU Time

```bash
ps -eo pid,etime,time,comm
```

This shows elapsed time and accumulated CPU time.

Difference matters:

```text
long-running process
!=
CPU-intensive process
```

A process can run for weeks while consuming very little CPU.

---

# 32. Threads

A process may contain multiple threads.

Inspect:

```bash
ps -eLf
```

For one process:

```bash
ps -T -p <PID>
```

Threads share much of the process's address space and resources.

---

# 33. Thread Troubleshooting

High CPU from one application can be caused by one hot thread.

Inspect:

```bash
top -H -p <PID>
```

This can show individual threads.

For deeper debugging, correlate:

```text
thread ID
application stack
logs
profiling
```

---

# 34. Process Niceness

Linux scheduling priority can be influenced by niceness.

View:

```bash
ps -eo pid,ni,pri,comm
```

Typical nice range:

```text
-20 = higher priority
  0 = default
+19 = lower priority
```

Lower nice values require appropriate privileges.

---

# 35. `nice`

Start a process with a lower priority:

```bash
nice -n 10 command
```

Example:

```bash
nice -n 10 python3 batch_job.py
```

Useful for background workloads that should not compete aggressively with production workloads.

---

# 36. `renice`

Change priority of an existing process:

```bash
renice 10 -p <PID>
```

Be careful. Scheduling changes can affect application behavior.

---

# 37. Signals

Signals are asynchronous notifications sent to processes.

Common signals:

```text
SIGTERM = 15
SIGKILL = 9
SIGINT  = 2
SIGHUP  = 1
SIGSTOP = 19
SIGCONT = 18
```

Check:

```bash
kill -l
```

---

# 38. SIGTERM

`SIGTERM` requests graceful termination.

Example:

```bash
kill -TERM <PID>
```

or:

```bash
kill <PID>
```

because `kill PID` normally sends SIGTERM.

A well-designed application should handle SIGTERM and perform cleanup.

---

# 39. SIGKILL

```bash
kill -KILL <PID>
```

or:

```bash
kill -9 <PID>
```

SIGKILL cannot be caught or ignored by the target process.

Use it only when graceful termination fails or when policy requires it.

It prevents the application from performing normal cleanup.

---

# 40. Why `kill -9` Is Not the First Choice

Using:

```bash
kill -9
```

immediately can cause:

```text
unfinished writes
unclean shutdown
lost in-memory state
temporary files
connection termination
corruption risk for poorly designed applications
```

Preferred sequence:

```text
SIGTERM
   ↓
wait
   ↓
verify
   ↓
SIGKILL only if necessary
```

---

# 41. SIGINT

`SIGINT` is commonly generated by:

```text
Ctrl+C
```

Example:

```bash
python3 app.py
```

Press:

```text
Ctrl+C
```

The terminal sends SIGINT to the foreground process group.

---

# 42. SIGHUP

Historically associated with terminal hangup.

Applications may also use SIGHUP to request configuration reload.

Behavior is application-specific.

Never assume every service reloads safely on SIGHUP.

---

# 43. SIGSTOP and SIGCONT

Stop:

```bash
kill -STOP <PID>
```

Continue:

```bash
kill -CONT <PID>
```

SIGSTOP cannot be caught or ignored.

These are useful for controlled diagnostics but should be used carefully in production.

---

# 44. Sending Signals from Python

```python
import os
import signal

os.kill(
    pid,
    signal.SIGTERM,
)
```

For subprocesses you own:

```python
process.terminate()
```

and:

```python
process.kill()
```

have different semantics.

---

# 45. Python Subprocess Lifecycle

```python
import subprocess

process = subprocess.Popen(
    ["sleep", "30"]
)

print(process.pid)

process.terminate()

process.wait(
    timeout=5
)
```

If the child does not terminate:

```python
process.kill()
process.wait()
```

Always consider reaping child processes.

---

# 46. Process Exit Codes

A process can return an exit status.

Python:

```python
result = subprocess.run(
    ["true"]
)

print(result.returncode)
```

Successful command:

```text
0
```

Non-zero:

```text
failure or special condition
```

Interpret exit codes according to the specific program.

---

# 47. Exit Status Is Not the Same as Signal

A process can terminate because of a signal.

For subprocesses, Python exposes return information through `returncode`.

A negative return code can indicate signal termination on Unix.

Example:

```python
result = subprocess.run(
    ["sleep", "100"]
)

if result.returncode < 0:
    print(
        "Terminated by signal:",
        -result.returncode,
    )
```

---

# 48. Foreground vs Background

Foreground process:

```bash
python3 app.py
```

The shell waits for it.

Background:

```bash
python3 app.py &
```

The shell can continue.

Check jobs:

```bash
jobs
```

---

# 49. `&`

Example:

```bash
sleep 100 &
```

The shell returns immediately.

But backgrounding is not a production service manager.

For production services, prefer:

```text
systemd
Kubernetes
supervisor
approved service manager
```

---

# 50. `nohup`

```bash
nohup python3 app.py > app.log 2>&1 &
```

Useful for simple long-running interactive tasks.

But for production services, use a proper supervisor.

---

# 51. Job Control

Common commands:

```text
Ctrl+C = terminate foreground job with SIGINT
Ctrl+Z = stop foreground job with SIGTSTP
bg     = continue stopped job in background
fg     = bring job to foreground
jobs   = list shell jobs
```

---

# 52. `disown`

Shell-specific command:

```bash
disown
```

It removes a job from the shell's job table.

This can be useful for interactive sessions but is not a replacement for service management.

---

# 53. `screen` and `tmux`

For interactive sessions:

```text
tmux
screen
```

can keep terminal sessions alive after disconnecting.

They are useful for administration but should not replace systemd or Kubernetes for production application lifecycle management.

---

# 54. Process Groups

Processes can belong to process groups.

This matters when terminating:

```text
shell pipelines
job-control groups
application process trees
```

Killing only one PID may leave child processes running.

---

# 55. Session vs Process Group

Conceptually:

```text
Session
  |
  +-- Process Group
        |
        +-- Process
        +-- Process
```

This becomes important when managing subprocess trees.

---

# 56. Kill a Process Tree

Be careful with:

```bash
kill <PID>
```

It may only signal one process.

For a service, let the service manager handle the process group when possible.

For custom subprocess management, design explicit parent/child lifecycle handling.

---

# 57. Process Environment

Inspect:

```bash
tr ' ' '
' < /proc/<PID>/environ
```

This may contain:

```text
environment variables
credentials
tokens
configuration
```

Therefore access should be treated as sensitive.

Never paste `/proc/<PID>/environ` into public logs.

---

# 58. Process Command Line

```bash
tr ' ' ' ' < /proc/<PID>/cmdline
```

or:

```bash
ps -p <PID> -o args=
```

Command-line arguments can contain secrets.

Avoid passing secrets as CLI arguments when safer mechanisms exist.

---

# 59. Process Working Directory

```bash
readlink -f /proc/<PID>/cwd
```

This can reveal where an application is running from.

Useful when a service behaves differently between:

```text
interactive shell
systemd
cron
container
```

---

# 60. Process Executable

```bash
readlink -f /proc/<PID>/exe
```

This can reveal the actual executable path.

Useful when:

```text
multiple versions installed
symlink points somewhere unexpected
wrong binary running
```

---

# 61. Process Root

```bash
readlink -f /proc/<PID>/root
```

Inside containers or chroots, this can help understand the process's filesystem view.

---

# 62. File Descriptors

Processes use file descriptors for:

```text
stdin = 0
stdout = 1
stderr = 2
files
sockets
pipes
devices
```

Inspect:

```bash
ls -l /proc/<PID>/fd
```

---

# 63. Count File Descriptors

```bash
ls /proc/<PID>/fd | wc -l
```

Or with Python:

```python
from pathlib import Path

fd_count = len(
    list(Path(
        f"/proc/{pid}/fd"
    ).iterdir())
)

print(fd_count)
```

High FD usage can indicate:

```text
connection leak
file leak
socket leak
```

---

# 64. `lsof`

`lsof` means:

```text
List Open Files
```

Examples:

```bash
lsof -p <PID>
```

Find process using a port:

```bash
lsof -i :8080
```

Find deleted files still open:

```bash
lsof +L1
```

---

# 65. Deleted but Open Files

A process can keep a deleted file open.

Then:

```text
directory entry deleted
      |
      v
process still holds FD
      |
      v
disk space remains used
```

This explains some cases where:

```bash
df -h
```

shows high usage while visible files do not account for it.

Investigate with:

```bash
lsof +L1
```

---

# 66. `/proc/<PID>/status`

Useful process information:

```bash
cat /proc/<PID>/status
```

It can show:

```text
Name
State
Pid
PPid
Uid
Gid
Threads
VmSize
VmRSS
FDSize
```

This is excellent for Linux-level diagnostics.

---

# 67. `/proc/<PID>/stat`

```bash
cat /proc/<PID>/stat
```

Contains low-level process statistics.

It is machine-oriented and less convenient for manual reading.

Use established tools or libraries when possible.

---

# 68. Process Limits

```bash
cat /proc/<PID>/limits
```

This shows limits such as:

```text
Max open files
Max processes
Stack size
Core file size
```

Very useful during resource-limit incidents.

---

# 69. `ulimit`

Shell:

```bash
ulimit -a
```

Specific:

```bash
ulimit -n
```

The latter represents the soft open-file limit for the shell/processes it launches.

Service managers may apply different limits.

---

# 70. systemd Resource Limits

For a service:

```bash
systemctl show nginx
```

You can inspect properties such as:

```text
LimitNOFILE
LimitNPROC
MemoryMax
CPUQuota
```

Exact availability depends on systemd version and configuration.

---

# 71. cgroups

Control groups provide resource control and accounting.

They are heavily used by:

```text
systemd
Docker
Kubernetes
container runtimes
```

They can limit:

```text
CPU
memory
PIDs
I/O
```

---

# 72. Why cgroups Matter

A process can look healthy from the host perspective but be constrained by its cgroup.

Example:

```text
Host memory = 32 GB
Container limit = 1 GB
Application uses = 1.1 GB
```

The host may have plenty of free memory while the container is OOM-killed.

Always check the relevant resource boundary.

---

# 73. Containers and Processes

Containers are not virtual machines.

At the Linux level, containers are implemented using mechanisms including:

```text
namespaces
cgroups
capabilities
filesystem isolation
security policies
```

The application still runs as Linux processes.

---

# 74. Container PID View

Inside a container:

```bash
ps
```

may show only the processes visible inside its PID namespace.

On the host, the same workload has host-level PIDs.

This explains why:

```text
container PID
!=
host PID
```

---

# 75. PID 1 Inside Containers

The container's PID 1 has special responsibilities.

It may need to:

```text
receive signals
reap child processes
handle shutdown
```

A poorly designed PID 1 can cause:

```text
zombies
bad signal handling
slow shutdown
```

Use an appropriate init process where necessary.

---

# 76. Docker Process Inspection

Useful commands:

```bash
docker top <container>
```

and:

```bash
docker stats
```

These provide container-level process/resource information.

---

# 77. Kubernetes and Processes

A Kubernetes Pod can contain one or more containers.

Conceptually:

```text
Node
 |
 +-- Pod
      |
      +-- Container
            |
            +-- Application process
```

For troubleshooting:

```bash
kubectl top pod
kubectl exec
kubectl describe pod
kubectl logs
```

Use node-level tools when investigating host resource pressure.

---

# 78. OOMKilled

A Kubernetes container can show:

```text
Reason: OOMKilled
```

This does not necessarily mean the entire node ran out of memory.

Investigate:

```text
container memory limit
container working set
node memory
cgroup events
recent deployment
application behavior
```

---

# 79. CPU Throttling

A container can have enough CPU allocated but still experience throttling because of CPU limits.

Investigate:

```text
CPU requests
CPU limits
cgroup CPU metrics
application latency
node capacity
```

Do not assume high latency means the application code is inherently slow.

---

# 80. Process Priority in Production

Use priority changes cautiously.

Changing niceness may affect:

```text
latency
throughput
batch jobs
CPU fairness
```

Do not use `renice` as a permanent solution to a capacity problem.

---

# 81. `kill` vs `systemctl stop`

For a managed service:

Prefer:

```bash
systemctl stop nginx
```

rather than:

```bash
kill <PID>
```

Why?

The service manager understands:

```text
unit
dependencies
process groups
restart policy
state
```

Directly killing a managed process can cause systemd to restart it.

---

# 82. `systemctl restart` vs `kill`

A proper restart:

```bash
systemctl restart nginx
```

can provide:

```text
controlled stop
start
logging
dependencies
service state
```

Use direct signals mainly for processes that are not appropriately managed through a service manager or for controlled troubleshooting.

---

# 83. Restart Loops

A service may repeatedly restart.

Check:

```bash
systemctl status <service>
journalctl -u <service>
```

Also inspect:

```text
Restart=
StartLimit*
exit code
signal
configuration
dependency
```

A restart loop can turn one failure into resource exhaustion.

---

# 84. `systemd` Restart Policy

Example:

```ini
[Service]
Restart=on-failure
RestartSec=5
```

This can improve resilience, but it should not hide a persistent application failure.

---

# 85. Detect Restart Loops

Look for:

```text
started
failed
started
failed
started
failed
```

If this repeats, stop treating it as a simple transient failure.

Find the root cause.

---

# 86. Process Metrics

Useful metrics include:

```text
CPU
RSS
virtual memory
threads
file descriptors
context switches
I/O
page faults
process count
```

Do not rely on CPU and memory alone.

---

# 87. Context Switches

Frequent context switching can be associated with:

```text
many runnable threads
high concurrency
synchronization
small work units
```

Tools such as:

```bash
pidstat
vmstat
```

can help investigate system behavior.

---

# 88. `pidstat`

Example:

```bash
pidstat 1
```

Useful for per-process statistics over time.

CPU:

```bash
pidstat -u 1
```

Memory:

```bash
pidstat -r 1
```

I/O:

```bash
pidstat -d 1
```

---

# 89. `vmstat`

```bash
vmstat 1
```

Useful for observing:

```text
processes
memory
swap
I/O
system
CPU
```

It provides a broader system view than `ps`.

---

# 90. `iostat`

If installed:

```bash
iostat -xz 1
```

Useful for investigating storage performance.

Look at:

```text
utilization
await
queue
throughput
```

Exact interpretation depends on storage architecture.

---

# 91. `sar`

If sysstat is installed:

```bash
sar
```

It can provide historical system statistics depending on configuration.

Useful for answering:

```text
Was CPU already high before the incident?
```

rather than only looking at the current state.

---

# 92. Process Monitoring Strategy

For an incident:

```text
top
  ↓
identify PID
  ↓
ps details
  ↓
pstree
  ↓
/proc
  ↓
lsof
  ↓
pidstat
  ↓
application logs
```

Use multiple sources instead of relying on one command.

---

# 93. CPU Incident Workflow

```text
CPU high?
    |
    v
identify top processes
    |
    v
is CPU user or system?
    |
    v
check threads
    |
    v
check load
    |
    v
check recent deployment
    |
    v
inspect application
```

---

# 94. Memory Incident Workflow

```text
Memory high?
    |
    v
host or cgroup?
    |
    v
identify process
    |
    v
RSS / threads / mappings
    |
    v
OOM events?
    |
    v
recent deployment?
    |
    v
leak / traffic / configuration?
```

---

# 95. Process Stuck Workflow

```text
Process stuck?
    |
    v
check STAT
    |
    +--> D -> investigate I/O
    |
    +--> S -> inspect wait/dependency
    |
    +--> R -> inspect CPU
    |
    +--> T -> inspect stop signal
    |
    +--> Z -> inspect parent
```

This is much better than immediately using:

```bash
kill -9
```

---

# 96. High Process Count

Check:

```bash
ps -e --no-headers | wc -l
```

Then inspect:

```text
process distribution
thread count
service behavior
forking
container workload
PID limits
```

---

# 97. PID Exhaustion

A system can run out of available process IDs or hit process limits.

Check:

```text
process count
ulimit
systemd TasksMax
cgroup pids.max
application process creation
```

In containers/Kubernetes, PID limits can be especially relevant.

---

# 98. Fork Bomb Concept

A fork bomb repeatedly creates processes until resources are exhausted.

Conceptually:

```text
process
  ↓
two processes
  ↓
four
  ↓
eight
  ↓
...
```

Do not execute fork-bomb commands on real systems.

The important DevOps lesson is:

```text
process creation must be controlled
```

---

# 99. Process Security

A process runs with an identity.

Check:

```bash
ps -eo user,group,pid,comm
```

Principle:

```text
application
   ↓
dedicated user
   ↓
minimum permissions
```

Avoid running normal applications as root.

---

# 100. Capabilities

Linux capabilities can grant specific privileged operations without full root privileges.

Examples include:

```text
CAP_NET_BIND_SERVICE
CAP_NET_ADMIN
```

Container security commonly uses capabilities to reduce privileges.

Inspect where appropriate with:

```bash
getcap /path/to/binary
```

---

# 101. `sudo`

Use:

```bash
sudo command
```

only when needed.

For automation, prefer:

```text
specific sudo policy
dedicated service account
limited command set
```

rather than unrestricted sudo.

---

# 102. Environment-Based Secrets

A process environment can contain secrets.

Therefore:

```text
environment variables
```

are not automatically secure.

Depending on the environment, other processes or privileged users may be able to inspect them.

Prefer approved secret-management mechanisms.

---

# 103. Process Command-Line Secrets

Avoid:

```bash
myapp --password=secret
```

Command arguments can be visible through process inspection.

Prefer:

```text
secret manager
protected configuration
file descriptor
stdin where appropriate
```

depending on the application.

---

# 104. Process Core Dumps

Core dumps can contain sensitive memory.

Before enabling or collecting them in production, consider:

```text
secrets
PII
credentials
memory size
storage
retention
access control
```

Core dumps should be governed by security policy.

---

# 105. `strace`

`strace` traces system calls.

Example:

```bash
strace -p <PID>
```

Useful for diagnosing:

```text
file access
network calls
process waits
system calls
```

Use cautiously in production because tracing can affect performance and generate large amounts of output.

---

# 106. `strace` Examples

Trace file operations:

```bash
strace -e trace=file -p <PID>
```

Trace network-related calls:

```bash
strace -e trace=network -p <PID>
```

Trace process-related calls:

```bash
strace -e trace=process -p <PID>
```

Use a narrow scope.

---

# 107. `lsof` + `strace`

If a process appears stuck:

```text
lsof
  ↓
identify files/sockets
  ↓
strace
  ↓
understand system call
```

This can distinguish:

```text
waiting on file
waiting on network
polling
sleeping
retrying
```

---

# 108. `gdb`

For native applications, `gdb` can inspect process state.

Example:

```bash
gdb -p <PID>
```

This requires appropriate permissions and should be used carefully in production.

---

# 109. Profiling

For performance problems, consider:

```text
application profiler
perf
py-spy
language-specific tooling
```

Do not rely on `top` alone for detailed performance analysis.

---

# 110. Python Process Profiling

For Python applications, useful approaches can include:

```text
cProfile
py-spy
sampling profilers
application metrics
```

The right tool depends on whether the problem is:

```text
CPU
I/O
lock contention
memory
external dependency
```

---

# 111. Process vs Thread

Process:

```text
separate virtual address space
```

Thread:

```text
shares process address space
```

Processes generally provide stronger isolation.

Threads can provide efficient concurrency but require careful synchronization.

---

# 112. Thread Safety

Shared memory can create:

```text
race conditions
deadlocks
lock contention
data corruption
```

For production troubleshooting, distinguish:

```text
high CPU
high thread count
blocked thread
deadlock
I/O wait
```

---

# 113. Deadlock

A deadlock can occur when:

```text
Thread A holds Lock 1
and waits for Lock 2

Thread B holds Lock 2
and waits for Lock 1
```

Result:

```text
A waits for B
B waits for A
```

System-level tools can show the process is alive even though useful work has stopped.

---

# 114. Process Health Is Multi-Dimensional

A process can be:

```text
running but unhealthy
```

or:

```text
sleeping but healthy
```

or:

```text
active but CPU-starved
```

or:

```text
alive but blocked on I/O
```

Always combine:

```text
state
resources
logs
dependencies
application health
```

---

# 115. Monitoring vs Troubleshooting

Monitoring asks:

```text
Is something wrong?
```

Troubleshooting asks:

```text
Why is it wrong?
```

Prometheus/Grafana can reveal:

```text
CPU spike
memory growth
latency
restarts
```

Linux process tools help identify:

```text
which process
which thread
which resource
which system interaction
```

---

# 116. Production Process Monitoring Stack

A realistic stack may be:

```text
Application
    |
    +--> Process
    |
    +--> Logs --> ELK
    |
    +--> Metrics --> Prometheus
                     |
                     v
                   Grafana
```

Linux process tools are still essential during incidents.

---

# 117. Alert Example — High CPU

Bad alert:

```text
CPU > 80% for 10 seconds
```

Better approach:

```text
sustained CPU saturation
+
application impact
+
historical baseline
```

Alert thresholds should reflect workload behavior.

---

# 118. Alert Example — Process Down

A process-level alert can be misleading if systemd or Kubernetes intentionally manages restarts.

Better monitoring may check:

```text
service health
application endpoint
error rate
latency
availability
```

rather than only whether a PID exists.

---

# 119. PID Monitoring Is Fragile

Avoid treating:

```text
PID 1234 exists
```

as proof that:

```text
application is healthy
```

PIDs are reused.

A stronger check is:

```text
service manager state
application health
process metrics
```

---

# 120. Process Restart and PID Changes

After:

```bash
systemctl restart nginx
```

the PID may change.

Therefore scripts should not permanently store a PID unless they have a robust reason and lifecycle model.

Use service identity where possible.

---

# 121. PID Files

Some applications use:

```text
/var/run/app.pid
```

or:

```text
/run/app.pid
```

A stale PID file can point to:

```text
nonexistent process
different process
reused PID
```

Always validate the PID's command and ownership.

---

# 122. Safe PID File Check

Conceptually:

```text
read PID
   ↓
validate integer
   ↓
check process exists
   ↓
verify expected command/user
   ↓
act
```

Do not blindly kill the PID from a stale file.

---

# 123. Process Lock vs PID File

PID files identify a process.

Lock files prevent duplicate work.

They are not interchangeable.

For scheduled jobs, a real locking mechanism is preferable to simply checking whether a PID file exists.

---

# 124. Process Supervisors

Common process/service managers include:

```text
systemd
supervisord
Kubernetes
container runtimes
```

A supervisor generally provides:

```text
start
stop
restart
health
logs
resource policy
```

Use one instead of custom infinite restart loops.

---

# 125. Infinite Restart Loop Anti-Pattern

Bad:

```bash
while true
do
    ./app
done
```

If the application fails instantly:

```text
start
fail
start
fail
start
fail
...
```

This can consume CPU and flood logs.

Use controlled backoff and proper service supervision.

---

# 126. systemd vs Custom Watchdog

Prefer systemd's built-in mechanisms when the service is systemd-managed.

For example:

```ini
Restart=on-failure
RestartSec=5
```

Then monitor:

```text
restart frequency
failure reason
application health
```

---

# 127. Watchdog Concept

A watchdog can detect:

```text
application stopped responding
```

and trigger recovery.

But watchdogs should avoid masking root causes.

A good design includes:

```text
health signal
timeout
bounded recovery
alerting
diagnostic collection
```

---

# 128. Process Lifecycle

Typical lifecycle:

```text
Created
  ↓
Running
  ↓
Sleeping / waiting
  ↓
Running
  ↓
Terminating
  ↓
Exited
```

Possible special states:

```text
Stopped
Zombie
```

---

# 129. `fork` Concept

Historically, Unix processes can be created using:

```text
fork()
```

The child initially inherits much of the parent's process context.

Modern applications and runtimes may use different process-creation mechanisms internally.

As a DevOps engineer, understand the conceptual parent/child relationship.

---

# 130. `exec` Concept

`exec` replaces the current process image with another program.

Conceptually:

```text
shell process
   |
   +--> exec -> nginx
```

The PID can remain the same while the executable changes.

This is important when interpreting process trees.

---

# 131. Why `exec` Matters in Containers

Container entrypoints often use:

```text
exec application
```

so the application becomes PID 1 and receives signals directly.

Without correct signal forwarding:

```text
SIGTERM
   |
   X
application
```

can result in poor shutdown behavior.

---

# 132. Shell Wrapper Problem

Example:

```bash
#!/bin/sh

python3 app.py
```

The shell may remain as the main process.

Using:

```bash
exec python3 app.py
```

replaces the shell with Python.

This can improve signal handling in container entrypoints.

---

# 133. Kubernetes Graceful Shutdown

Kubernetes generally sends termination signals during Pod termination.

Application should:

```text
receive SIGTERM
   ↓
stop accepting work
   ↓
finish safe operations
   ↓
exit
```

If shutdown exceeds the configured grace period, Kubernetes can force termination.

---

# 134. Process Signals in Kubernetes

When a Pod is terminated, understand:

```text
preStop hook
termination signal
grace period
container runtime
PID 1
application shutdown
```

Poor signal handling can cause:

```text
dropped requests
slow deployments
failed rollouts
```

---

# 135. Graceful Shutdown Design

Application should support:

```text
SIGTERM
```

and stop cleanly.

For HTTP services:

```text
stop accepting new requests
drain active requests
close connections
flush important state
exit
```

Do not make shutdown infinitely long.

---

# 136. Process Resource Boundaries

When investigating resources, identify:

```text
host
VM
container
Pod
process
thread
```

Example:

```text
Host: 16 GB
Pod limit: 1 GB
Process RSS: 900 MB
```

The relevant memory boundary for the process may be the Pod/container limit, not the host's total RAM.

---

# 137. Process Troubleshooting Decision Tree

```text
Application slow
      |
      +--> CPU high?
      |      |
      |      +--> yes -> process/thread CPU
      |
      +--> Memory high?
      |      |
      |      +--> yes -> RSS/cgroup/OOM
      |
      +--> I/O wait?
      |      |
      |      +--> yes -> storage/network FS
      |
      +--> Network?
      |      |
      |      +--> yes -> sockets/DNS/TCP
      |
      +--> Blocked?
             |
             +--> inspect state/syscalls
```

---

# 138. Practical Command Cheat Sheet

## List processes

```bash
ps -ef
ps aux
```

## Sort CPU

```bash
ps aux --sort=-%cpu | head
```

## Sort memory

```bash
ps aux --sort=-%mem | head
```

## Find process

```bash
pgrep -a nginx
```

## Process tree

```bash
pstree -p
```

## Interactive monitoring

```bash
top
```

## Threads

```bash
ps -T -p <PID>
top -H -p <PID>
```

## Open files

```bash
lsof -p <PID>
```

## Process details

```bash
cat /proc/<PID>/status
```

## Limits

```bash
cat /proc/<PID>/limits
```

## File descriptors

```bash
ls -l /proc/<PID>/fd
```

## Signals

```bash
kill -TERM <PID>
kill -KILL <PID>
```

## Service

```bash
systemctl status <service>
```

---

# 139. Practical Command Cheat Sheet — Resource Investigation

CPU:

```bash
top
ps aux --sort=-%cpu | head
pidstat -u 1
```

Memory:

```bash
free -h
ps aux --sort=-%mem | head
pidstat -r 1
```

Disk/I/O:

```bash
df -h
df -i
iostat -xz 1
pidstat -d 1
```

Load:

```bash
uptime
cat /proc/loadavg
```

Processes:

```bash
ps -ef
pstree -p
```

---

# 140. Practical Command Cheat Sheet — Services

Status:

```bash
systemctl status nginx
```

Active check:

```bash
systemctl is-active nginx
```

Start:

```bash
systemctl start nginx
```

Stop:

```bash
systemctl stop nginx
```

Restart:

```bash
systemctl restart nginx
```

Enable at boot:

```bash
systemctl enable nginx
```

Logs:

```bash
journalctl -u nginx
```

---

# 141. Practical Command Cheat Sheet — Signals

Graceful:

```bash
kill -TERM <PID>
```

Interrupt:

```bash
kill -INT <PID>
```

Force:

```bash
kill -KILL <PID>
```

Stop:

```bash
kill -STOP <PID>
```

Continue:

```bash
kill -CONT <PID>
```

List signals:

```bash
kill -l
```

---

# 142. Production Rule: Observe Before Acting

When you see:

```text
CPU = 100%
```

Do not immediately:

```bash
kill -9
```

Instead:

```text
identify process
   ↓
check threads
   ↓
check logs
   ↓
check recent changes
   ↓
check workload
   ↓
decide remediation
   ↓
verify
```

---

# 143. Production Rule: Resource Symptoms Are Not Root Causes

Examples:

```text
High CPU
```

could be:

```text
traffic
bad loop
compression
encryption
GC
query processing
retry storm
```

High memory:

```text
traffic
cache
memory leak
large workload
container limit
```

High I/O:

```text
logs
database
backup
storage issue
```

Find the underlying cause.

---

# 144. Production Rule: Check Recent Changes

During an incident ask:

```text
What changed?
```

Look at:

```text
deployment
configuration
traffic
dependency
database
kernel
package
infrastructure
scaling
```

A recent change is evidence, not automatically the root cause.

---

# 145. Production Rule: Compare with Baseline

Current:

```text
CPU 80%
```

means little without context.

Compare:

```text
normal CPU
traffic
historical behavior
time of day
deployment window
```

Use metrics and logs to establish baseline.

---

# 146. Production Rule: Correlate Signals

Strong diagnosis often combines:

```text
CPU metrics
+
process stats
+
application logs
+
network metrics
+
deployment events
```

Example:

```text
Traffic increased
      +
CPU increased
      +
worker count unchanged
      +
request latency increased
```

This provides a stronger hypothesis than CPU alone.

---

# 147. Production Rule: Verify Remediation

After an action:

```text
restart
configuration change
resource adjustment
rollback
```

verify:

```text
process state
service state
health endpoint
logs
metrics
user impact
```

Never assume the action solved the issue.

---

# 148. Production Rule: Preserve Evidence

Before killing a problematic process, when practical collect:

```text
PID
PPID
command
CPU
memory
state
threads
open files
logs
environment metadata
service state
```

This can make post-incident analysis much easier.

Be careful with sensitive information.

---

# 149. Production Rule: Avoid Evidence Destruction

An immediate:

```bash
kill -9
```

can remove the opportunity to inspect:

```text
process state
open resources
stack
temporary state
```

If the process is causing imminent harm, containment takes priority. Otherwise collect useful evidence first.

---

# 150. Production Rule: Use the Right Abstraction

If systemd manages the service:

```text
systemctl
```

If Kubernetes manages the workload:

```text
kubectl
```

If a container runtime manages it:

```text
docker/container runtime
```

If Terraform manages infrastructure:

```text
terraform
```

Use the appropriate control plane instead of bypassing it unnecessarily.

---

# 151. Interview Question — What Is a Process?

**Answer:**

> A process is a running instance of a program. It has a PID and resources such as memory, CPU time, file descriptors, environment, and threads. Processes can have parent-child relationships, receive signals, and be managed by service supervisors such as systemd.

---

# 152. Interview Question — Difference Between PID and PPID?

**Answer:**

> PID identifies the current process, while PPID identifies its parent process. I use PPID and process trees during troubleshooting to understand how an application was started and which supervisor or parent controls it.

---

# 153. Interview Question — What Is a Zombie Process?

**Answer:**

> A zombie is a process that has already exited but whose parent has not collected its exit status. I don't try to kill the zombie itself; I investigate the parent process and why it isn't reaping its children.

---

# 154. Interview Question — What Is an Orphan Process?

**Answer:**

> An orphan is a child process whose parent has exited. Linux reassigns it to an appropriate reaper, traditionally PID 1. An orphan is not necessarily a problem.

---

# 155. Interview Question — Difference Between SIGTERM and SIGKILL?

**Answer:**

> SIGTERM requests graceful termination and can be handled by the application. SIGKILL forces termination and cannot be caught or ignored. I normally use SIGTERM first, wait and verify, and use SIGKILL only when necessary.

---

# 156. Interview Question — How Do You Troubleshoot High CPU?

**Answer:**

> I identify the top CPU processes using `top` or `ps`, determine whether the CPU is user or system time, inspect threads with `top -H` or `ps -T`, check load average, recent deployments, traffic, and application logs. Then I identify the root cause before taking remediation.

---

# 157. Interview Question — How Do You Troubleshoot High Memory?

**Answer:**

> I identify the process with high RSS, determine whether the issue is host-level or cgroup/container-level, check memory trends and OOM events, inspect recent deployments and application behavior, and correlate the process usage with container limits and workload.

---

# 158. Interview Question — Process Is in D State. What Do You Do?

**Answer:**

> D usually indicates uninterruptible sleep, commonly associated with I/O. I investigate storage, filesystem, network filesystem, kernel logs, and I/O metrics. I don't assume that killing the process will solve the underlying problem.

---

# 159. Interview Question — How Do You Troubleshoot a Zombie?

**Answer:**

> I identify the zombie's PPID, inspect the parent process, and determine why the parent isn't collecting child exit status. The zombie has already exited, so sending SIGKILL to it is not the solution.

---

# 160. Interview Question — How Do You Find Which Process Uses a Port?

**Answer:**

> I can use `ss -lntp`, `lsof -i :PORT`, or `netstat` on systems where it is available. I prefer `ss` or `lsof` on modern Linux systems.

---

# 161. Interview Question — What Is the Difference Between Process and Thread?

**Answer:**

> Processes generally have separate address spaces, while threads within a process share the process's address space and many resources. Threads can provide efficient concurrency but introduce synchronization concerns such as races and deadlocks.

---

# 162. Interview Question — Why Is PID Monitoring Not Enough?

**Answer:**

> A PID only tells me that a process exists. It doesn't prove the application is healthy, and PIDs can be reused. For production monitoring I prefer service state, health endpoints, application metrics, logs, and resource metrics.

---

# 163. Interview Question — How Do Containers Affect Process Troubleshooting?

**Answer:**

> Containers use Linux namespaces and cgroups. A process has a PID inside its container namespace and another PID on the host. Resource limits are also applied through cgroups, so I check the container or Pod limits rather than only host-level resources.

---

# 164. Interview Question — Why Does PID 1 Matter in Containers?

**Answer:**

> PID 1 has special signal and child-reaping responsibilities. If the container's PID 1 doesn't handle SIGTERM or reap child processes correctly, graceful shutdown and zombie handling can fail.

---

# 165. Interview Question — systemctl Stop vs kill

**Answer:**

> If systemd manages the application, I prefer `systemctl stop` because systemd understands the service unit, dependencies, process tracking, and restart policy. Directly killing a PID can cause systemd to restart the process or leave related processes behind.

---

# 166. Interview Question — How Do You Handle a Restart Loop?

**Answer:**

> I inspect `systemctl status` and `journalctl` to identify the actual failure. I also check restart policy, start limits, configuration, dependencies, and recent changes. I don't keep restarting the application without understanding why it exits.

---

# 167. Interview Question — What Is cgroup?

**Answer:**

> A cgroup is a Linux mechanism for organizing processes and controlling or accounting for resources such as CPU, memory, I/O, and process counts. Containers, Kubernetes, and systemd rely heavily on cgroups.

---

# 168. Interview Question — Host Has Free Memory but Container Is OOMKilled. Why?

**Answer:**

> The container may have a memory limit enforced by its cgroup. The host can still have free memory while the container exceeds its configured limit. I would inspect container memory usage, limit, working set, and OOM events.

---

# 169. Interview Question — How Do You Debug a Hanging Process?

**Answer:**

> First I inspect its process state, CPU, memory, open files, and parent. If it is in D state I investigate I/O. If needed, I use tools such as `strace` to understand system calls. I also inspect application logs and dependencies before terminating it.

---

# 170. Interview Question — Why Not Always Use kill -9?

**Answer:**

> SIGKILL prevents graceful cleanup. It can interrupt application shutdown, leave temporary state, and remove valuable diagnostic evidence. I use SIGTERM first and only escalate to SIGKILL when appropriate.

---

# 171. Interview Scenario — Nginx Is Down

**Question:**

Nginx is reported down. What do you do?

**Answer:**

```text
systemctl status nginx
        ↓
journalctl -u nginx
        ↓
configuration validation
        ↓
ss -lntp
        ↓
HTTP health check
        ↓
recent deployment/config change
```

If configuration is invalid, fix or restore the known-good configuration rather than repeatedly restarting.

---

# 172. Interview Scenario — CPU 100%

**Question:**

A production EC2 instance shows 100% CPU. What do you do?

**Answer:**

> I first identify the top processes and determine whether the CPU is user, system, or I/O-related. Then I inspect threads, load average, recent deployments, traffic, and application logs. I correlate the process metrics with application behavior before deciding whether to scale, restart, rollback, or fix the workload.

---

# 173. Interview Scenario — Memory 95%

**Question:**

The server has 95% memory usage.

**Answer:**

> I identify which processes consume memory and check available memory, swap, RSS, memory trends, and OOM events. If the application runs inside a container, I also check cgroup memory limits. I distinguish normal caching from actual memory pressure before taking action.

---

# 174. Interview Scenario — Process in D State

**Question:**

A process is stuck in D state.

**Answer:**

> I investigate the I/O path using `iostat`, `vmstat`, `lsof`, kernel logs, and filesystem or storage metrics. D state usually indicates an uninterruptible wait, so killing the process may not resolve the underlying storage problem.

---

# 175. Interview Scenario — Zombie Processes Increasing

**Question:**

You see thousands of zombie processes.

**Answer:**

> I identify their common PPID and inspect the parent application. The parent is likely failing to reap child processes. I investigate the application's process-management behavior and remediate the parent rather than trying to kill already-exited zombies.

---

# 176. Interview Scenario — Service Keeps Restarting

**Answer:**

```bash
systemctl status app
journalctl -u app
systemctl show app
```

Then inspect:

```text
exit code
signal
configuration
dependencies
permissions
environment
restart policy
recent changes
```

Fix the cause before increasing restart frequency.

---

# 177. Interview Scenario — Application Works Manually but Not Through systemd

Possible differences:

```text
PATH
working directory
environment
user
permissions
Python virtual environment
configuration
secret access
```

Inspect the unit:

```bash
systemctl cat app
```

Then compare the service environment and execution context with the interactive shell.

---

# 178. Interview Scenario — App Uses 100% CPU After Deployment

Approach:

```text
deployment timestamp
       |
       v
CPU increase timestamp
       |
       v
application logs
       |
       v
thread-level CPU
       |
       v
code/config change
       |
       v
rollback or fix
```

Use evidence rather than assuming the deployment caused the problem.

---

# 179. Interview Scenario — Disk Is Full but Files Look Normal

Check:

```bash
df -h
df -i
lsof +L1
```

If deleted-open files exist:

```text
process still holds FD
      ↓
space remains allocated
      ↓
identify process
      ↓
restart/close FD according to application policy
```

---

# 180. Interview Scenario — Port Already in Use

Check:

```bash
ss -lntp
```

or:

```bash
lsof -i :8080
```

Then determine:

```text
which process
who owns it
which service started it
whether it is expected
```

Do not simply kill the process without understanding the service architecture.

---

# 181. Interview Scenario — Too Many Open Files

Check:

```bash
ulimit -n
cat /proc/<PID>/limits
ls /proc/<PID>/fd | wc -l
lsof -p <PID>
```

Then investigate:

```text
FD leak
connection pool
file access
service limits
traffic
```

Increasing the limit can be useful, but it may only hide an underlying leak.

---

# 182. Interview Scenario — Process Consumes High CPU but `top` Looks Normal

Possible causes:

```text
short CPU spikes
multiple threads
sampling interval
I/O wait
another process
container limits
```

Use:

```bash
pidstat
top -H
sar
```

and application metrics.

---

# 183. Interview Scenario — Container Restarts Frequently

Check:

```bash
kubectl get pod
kubectl describe pod
kubectl logs --previous
```

Then inspect:

```text
exit code
OOMKilled
liveness probe
readiness probe
resource limits
application startup
dependencies
```

Process management concepts remain important even though Kubernetes manages the lifecycle.

---

# 184. Interview Scenario — Kubernetes Pod Has CPU Throttling

Check:

```text
CPU requests
CPU limits
cgroup CPU metrics
node capacity
application latency
```

Then determine whether:

```text
limit is too low
workload is unexpectedly CPU-heavy
node is overloaded
```

---

# 185. Interview Scenario — Graceful Deployment Shutdown

A production application should:

```text
receive SIGTERM
     ↓
stop new requests
     ↓
drain active requests
     ↓
close connections
     ↓
flush necessary state
     ↓
exit
```

This reduces dropped requests during deployments.

---

# 186. Interview Scenario — Why Did systemd Restart My Process?

Possible reason:

```ini
Restart=on-failure
```

Check:

```bash
systemctl cat app
systemctl show app
journalctl -u app
```

If you manually kill the process, systemd may interpret it as a failure and restart it.

---

# 187. Interview Scenario — Child Process Survives Parent

Possible causes:

```text
orphan adoption
double-fork
daemonization
service manager
process group
```

Inspect:

```bash
pstree -p
ps -o pid,ppid,pgid,sid,comm
```

---

# 188. Interview Scenario — Python Script Leaves Child Processes

Use:

```python
subprocess.Popen()
```

carefully.

Make sure:

```text
children are waited for
timeouts exist
signals are propagated
process groups are managed when necessary
```

A production automation script should not accumulate orphaned children.

---

# 189. Process Management Best Practices

```text
1. Observe before acting.
2. Prefer service managers for services.
3. Use SIGTERM before SIGKILL.
4. Understand process trees.
5. Check resource boundaries.
6. Inspect cgroups in containers.
7. Monitor threads when needed.
8. Investigate D state as an I/O problem.
9. Investigate zombies through their parents.
10. Verify remediation.
11. Preserve useful evidence.
12. Use least privilege.
13. Avoid secrets in arguments/environment/logs.
14. Bound retries and restarts.
15. Use health checks rather than PID existence.
```

---

# 190. Production Troubleshooting Framework

When receiving an alert:

```text
ALERT
  ↓
SCOPE
  ↓
IMPACT
  ↓
OBSERVE
  ↓
IDENTIFY PROCESS
  ↓
CHECK RESOURCES
  ↓
CHECK DEPENDENCIES
  ↓
CHECK RECENT CHANGES
  ↓
FORM HYPOTHESIS
  ↓
TEST SAFELY
  ↓
REMEDIATE
  ↓
VERIFY
  ↓
DOCUMENT
```

---

# 191. Example: Full CPU Investigation

Start:

```bash
uptime
```

Then:

```bash
ps aux --sort=-%cpu | head -10
```

Identify PID:

```bash
ps -p <PID> -o pid,ppid,stat,%cpu,%mem,etime,cmd
```

Inspect threads:

```bash
top -H -p <PID>
```

Check tree:

```bash
pstree -aps <PID>
```

Inspect logs:

```bash
journalctl -u <service>
```

Then determine root cause.

---

# 192. Example: Full Memory Investigation

Start:

```bash
free -h
```

Then:

```bash
ps aux --sort=-%mem | head -10
```

Inspect:

```bash
cat /proc/<PID>/status
cat /proc/<PID>/limits
```

If containerized:

```text
check cgroup/container memory limit
```

Then:

```text
check OOM events
check application logs
check memory trend
```

---

# 193. Example: Full Process Investigation

Given PID:

```text
12345
```

Run:

```bash
ps -p 12345 -o pid,ppid,user,stat,%cpu,%mem,etime,args
```

Then:

```bash
pstree -aps 12345
```

Then:

```bash
readlink -f /proc/12345/exe
readlink -f /proc/12345/cwd
cat /proc/12345/status
cat /proc/12345/limits
```

Then:

```bash
lsof -p 12345
```

This provides a strong process-level snapshot.

---

# 194. Python Process Inspection

Python can use `psutil`:

```python
import psutil

process = psutil.Process(pid)

print("Name:", process.name())
print("Status:", process.status())
print("CPU:", process.cpu_percent())
print("Memory:", process.memory_info().rss)
print("Threads:", process.num_threads())
```

Access permissions can restrict some process information.

---

# 195. Python Process Tree

```python
import psutil

process = psutil.Process(pid)

print("Parent:", process.ppid())

for child in process.children(
    recursive=True
):
    print(
        child.pid,
        child.name()
    )
```

This is useful for custom diagnostic tools.

---

# 196. Python Process Termination

```python
import psutil

process = psutil.Process(pid)

process.terminate()

try:
    process.wait(timeout=5)
except psutil.TimeoutExpired:
    process.kill()
    process.wait()
```

This implements:

```text
graceful termination
      ↓
wait
      ↓
force only if necessary
```

---

# 197. Python Process Resource Snapshot

```python
import psutil

process = psutil.Process(pid)

with process.oneshot():
    info = {
        "pid": process.pid,
        "name": process.name(),
        "status": process.status(),
        "cpu_percent": process.cpu_percent(),
        "memory_rss": process.memory_info().rss,
        "threads": process.num_threads(),
    }

print(info)
```

`oneshot()` can reduce repeated process information lookups.

---

# 198. Python Find High CPU Processes

```python
import psutil

for process in psutil.process_iter(
    ["pid", "name", "cpu_percent"]
):
    try:
        if process.info["cpu_percent"] > 80:
            print(process.info)
    except (
        psutil.NoSuchProcess,
        psutil.AccessDenied,
    ):
        pass
```

Process lists can change while you inspect them, so handle races and permission errors.

---

# 199. Python Find High Memory Processes

```python
import psutil

for process in psutil.process_iter(
    ["pid", "name", "memory_info"]
):
    try:
        rss = process.info[
            "memory_info"
        ].rss

        if rss > 500 * 1024 * 1024:
            print(
                process.pid,
                process.info["name"],
                rss,
            )

    except (
        psutil.NoSuchProcess,
        psutil.AccessDenied,
    ):
        pass
```

Thresholds should be based on workload.

---

# 200. Python Process Health Tool

A production-oriented tool can output:

```json
{
  "hostname": "server01",
  "process": {
    "pid": 1234,
    "name": "python3",
    "status": "running",
    "cpu_percent": 23.4,
    "memory_rss_mb": 412,
    "threads": 12
  },
  "service": "myapp",
  "healthy": true
}
```

This can feed:

```text
CI/CD
monitoring
incident tooling
automation
```

---

# 201. Process Management + Ansible

Ansible can manage services declaratively:

```yaml
- name: Ensure nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
```

Python can provide:

```text
custom diagnostics
validation
data processing
API integration
```

Prefer Ansible for fleet-level configuration management when that is its intended role.

---

# 202. Process Management + Terraform

Terraform should provision:

```text
EC2
security groups
IAM
EKS
network
load balancers
```

Python can perform:

```text
post-provision validation
custom health checks
integration
```

Do not use Python to secretly mutate Terraform-managed infrastructure because that creates drift.

---

# 203. Process Management + Jenkins

A Jenkins pipeline can:

```text
build
test
scan
deploy
verify
```

Python can execute:

```text
preflight
post-deployment validation
Linux diagnostics
artifact checks
```

A failed Python check should return a non-zero exit code so Jenkins can fail the stage.

---

# 204. Process Management + GitHub Actions

Example:

```yaml
- name: Run Linux health check
  run: |
    python3 scripts/health.py
```

For reproducibility, use a defined Python version and dependencies.

---

# 205. Process Management + GitLab CI/CD

A job can run:

```bash
python3 scripts/preflight.py
```

Then:

```text
exit 0 -> continue
exit non-zero -> stop
```

This makes Linux validation part of the delivery pipeline.

---

# 206. Process Management + Prometheus

Prometheus should provide continuous metrics.

Python can provide:

```text
specialized checks
```

Do not repeatedly spawn expensive commands every few seconds if an exporter already provides the metric.

---

# 207. Process Management + Grafana

Grafana can visualize:

```text
CPU
memory
process count
restarts
latency
resource saturation
```

During incidents:

```text
Grafana trend
    +
Linux process inspection
    +
application logs
```

provides stronger diagnosis.

---

# 208. Process Management + ELK

ELK can reveal:

```text
application exceptions
restart messages
OOM events
connection errors
configuration failures
```

Linux process tools provide the live state.

Together:

```text
Logs = what happened
Metrics = how much/when
Process tools = what is happening now
```

---

# 209. Observability Mental Model

```text
Metrics
  ↓
detect symptom

Logs
  ↓
understand event

Process/system tools
  ↓
identify resource/process

Application diagnostics
  ↓
find root cause

Remediation
  ↓
verify
```

---

# 210. Common Mistakes

Avoid:

```text
kill -9 immediately
restart without diagnosis
monitor PID only
ignore PPID
ignore cgroup limits
assume high memory = leak
assume high CPU = application bug
ignore D state
ignore deleted-open files
run services as root
expose secrets through ps
create infinite restart loops
ignore process limits
```

---

# 211. Final Process Management Checklist

```text
[ ] Understand PID/PPID
[ ] Read process state
[ ] Use ps/top/pgrep/pstree
[ ] Understand CPU and memory
[ ] Understand RSS vs VSZ
[ ] Understand threads
[ ] Understand signals
[ ] Prefer SIGTERM before SIGKILL
[ ] Understand zombies/orphans
[ ] Inspect file descriptors
[ ] Know /proc
[ ] Understand systemd
[ ] Understand cgroups
[ ] Understand container PIDs
[ ] Understand resource limits
[ ] Use lsof/strace when appropriate
[ ] Preserve incident evidence
[ ] Verify remediation
[ ] Use least privilege
```

---

# 212. Final Mental Model

When you troubleshoot a Linux process, think:

```text
WHO?
  |
  +--> PID
  +--> PPID
  +--> USER
  +--> SERVICE

WHAT?
  |
  +--> COMMAND
  +--> EXECUTABLE
  +--> WORKING DIRECTORY
  +--> ENVIRONMENT

STATE?
  |
  +--> R
  +--> S
  +--> D
  +--> T
  +--> Z

RESOURCES?
  |
  +--> CPU
  +--> MEMORY
  +--> THREADS
  +--> FILE DESCRIPTORS
  +--> I/O

BOUNDARY?
  |
  +--> HOST
  +--> VM
  +--> CONTAINER
  +--> POD
  +--> CGROUP

EVIDENCE?
  |
  +--> LOGS
  +--> METRICS
  +--> /proc
  +--> lsof
  +--> strace
  +--> recent changes

ACTION?
  |
  +--> SIGTERM
  +--> SERVICE MANAGER
  +--> ROLLBACK
  +--> SCALE
  +--> FIX

VERIFY
```

> **A DevOps engineer should not just know how to kill a process. The real skill is knowing why the process is unhealthy, what owns it, what resource is affected, what the safest remediation is, and how to verify that the system recovered.**
