# Linux Monitoring

## 1. Overview

Linux monitoring is the process of continuously observing the health, performance, resource utilization, processes, services, filesystem, and network behavior of Linux servers.

For a DevOps engineer, Linux monitoring is fundamental because many production workloads run on:

```text
EC2
Kubernetes Nodes
Bastion Hosts
Application Servers
Database Servers
CI/CD Servers
Monitoring Servers
```

A Linux server can be reachable while still experiencing:

```text
High CPU
High memory usage
Disk exhaustion
Disk I/O bottlenecks
Network saturation
Process failures
Service failures
File descriptor exhaustion
Load average spikes
```

Therefore, Linux monitoring should cover:

```text
Linux Server
│
├── CPU
├── Memory
├── Load Average
├── Disk Space
├── Disk I/O
├── Network
├── Processes
├── Services
├── File Descriptors
└── System Health
```

---

# 2. Linux Monitoring Layers

A practical monitoring model is:

```text
                    Linux Server
                         │
       ┌─────────────────┼─────────────────┐
       ↓                 ↓                 ↓
    Compute           Memory            Storage
       │                 │                 │
     CPU/RAM          RAM/Swap         Disk/I/O
       │                 │                 │
       └─────────────────┼─────────────────┘
                         ↓
                     Processes
                         │
                         ↓
                      Services
                         │
                         ↓
                     Network
                         │
                         ↓
                    Applications
```

---

# 3. First Commands for Linux Health

When troubleshooting a Linux server, start with:

```bash
uptime
top
free -h
df -h
df -i
ps -ef
ss -lntp
```

These provide a quick overview of:

```text
System uptime
Load
CPU
Memory
Disk
Inodes
Processes
Listening ports
```

---

# 4. `uptime`

The `uptime` command provides:

```bash
uptime
```

Example:

```text
10:30:20 up 15 days, 4:22, 2 users, load average: 1.20, 1.10, 0.90
```

It shows:

```text
Current time
Uptime
Logged-in users
Load average
```

The three load values generally represent:

```text
1 minute
5 minutes
15 minutes
```

---

# 5. Load Average

Load average indicates the amount of work waiting for or using CPU resources, along with tasks in certain uninterruptible states.

Example:

```text
load average: 4.50, 3.20, 2.10
```

Interpret it together with CPU count.

For example:

```text
4 CPU cores
Load = 1
```

is very different from:

```text
1 CPU core
Load = 4
```

Do not interpret load average in isolation.

---

# 6. CPU Monitoring

Important CPU indicators include:

```text
CPU utilization
User CPU
System CPU
I/O wait
Idle CPU
Steal time
Load average
```

Useful commands:

```bash
top
mpstat
vmstat
```

---

# 7. `top`

`top` provides real-time process and system information.

Run:

```bash
top
```

Important fields include:

```text
%us
%sy
%id
%wa
%st
```

Conceptually:

```text
us → User CPU
sy → System CPU
id → Idle CPU
wa → I/O wait
st → Steal time
```

---

# 8. CPU User Time

High user CPU can indicate:

```text
Application workload
CPU-intensive processing
High request volume
Compression
Encryption
Data processing
```

Example:

```text
CPU
│
│          █████████
│        ███████████
│      █████████████
│____████████████████
└────────────────────→ Time
```

Investigate the process consuming CPU.

---

# 9. CPU System Time

System CPU represents CPU time spent executing kernel/system operations.

High system CPU may indicate:

```text
Heavy system calls
Network processing
Disk activity
Kernel operations
High process activity
```

Check:

```bash
top
vmstat
```

and identify which workload is generating the activity.

---

# 10. I/O Wait

I/O wait represents time associated with the CPU waiting for I/O operations to complete.

High I/O wait can occur due to:

```text
Slow disks
Heavy disk operations
Storage bottlenecks
Database I/O
Large file operations
```

Check:

```bash
top
iostat
vmstat
```

High I/O wait does not necessarily mean the CPU itself is the bottleneck.

---

# 11. CPU Steal Time

CPU steal time is especially important on virtualized environments.

It indicates CPU time that the virtual machine wanted but the underlying hypervisor allocated elsewhere.

In cloud environments:

```text
VM
 ↓
Hypervisor
 ↓
Physical resources
```

High steal time can indicate underlying CPU contention.

---

# 12. `nproc`

Check the number of available processing units:

```bash
nproc
```

Example:

```text
4
```

This is useful when interpreting:

```text
Load average
CPU utilization
Process concurrency
```

---

# 13. `lscpu`

For detailed CPU information:

```bash
lscpu
```

Useful information includes:

```text
CPU architecture
CPU(s)
Threads
Cores
Sockets
Virtualization
CPU model
```

---

# 14. Per-CPU Monitoring

Use:

```bash
mpstat -P ALL
```

This can show CPU utilization per processor.

Example concept:

```text
CPU0 → 20%
CPU1 → 25%
CPU2 → 90%
CPU3 → 15%
```

If only one CPU is saturated, investigate whether a workload is constrained to a single processing unit.

---

# 15. Memory Monitoring

Important memory indicators:

```text
Total memory
Used memory
Available memory
Free memory
Buffers
Cache
Swap
```

Use:

```bash
free -h
```

Example:

```text
               total   used   free   shared   buff/cache   available
Mem:            16Gi    8Gi    2Gi     500M       6Gi         7Gi
Swap:            4Gi    0Gi    4Gi
```

Focus particularly on **available memory**, not just the `used` value.

---

# 16. Linux Memory Model

Linux uses unused memory for useful purposes such as filesystem caching.

Conceptually:

```text
Physical RAM
│
├── Processes
├── Kernel
├── Buffers
└── Page Cache
```

Therefore:

```text
Used RAM ≠ Automatically Bad
```

A better indicator of memory pressure is whether available memory is becoming low and whether the system is actively using swap or experiencing reclaim pressure.

---

# 17. Swap Monitoring

Check:

```bash
free -h
```

Swap is disk-backed memory used when physical memory is under pressure.

Heavy swap activity can cause:

```text
Higher latency
Application slowdown
Disk I/O
Poor system responsiveness
```

Check swap activity using:

```bash
vmstat
```

---

# 18. `vmstat`

Run:

```bash
vmstat 1
```

It provides information about:

```text
Processes
Memory
Swap
I/O
System
CPU
```

Example:

```text
vmstat 1
```

The output can help correlate:

```text
CPU pressure
Memory pressure
Swap activity
I/O activity
```

---

# 19. OOM Conditions

When Linux cannot satisfy memory allocation requirements, the Out-Of-Memory mechanism may terminate processes.

Symptoms may include:

```text
Process killed
Service unavailable
Application restart
Kernel OOM messages
```

Check kernel logs:

```bash
dmesg | grep -i oom
```

or, on systemd-based systems:

```bash
journalctl -k | grep -i oom
```

---

# 20. Memory Troubleshooting

If memory usage is high:

```text
1. Check free -h
2. Check swap
3. Identify top memory processes
4. Check application behavior
5. Check process growth over time
6. Check kernel OOM events
7. Check recent deployments
```

Use:

```bash
ps -eo pid,ppid,cmd,%mem --sort=-%mem | head
```

---

# 21. Disk Space Monitoring

Check filesystem usage:

```bash
df -h
```

Example:

```text
Filesystem      Size  Used Avail Use%
/dev/xvda1       50G   42G    8G  84%
```

Monitor:

```text
Used space
Available space
Percentage used
Mount point
```

---

# 22. Disk Space Thresholds

Operational thresholds depend on workload, but a common approach is:

```text
Warning  → 70–80%
Critical → 85–90%+
```

The exact thresholds should be based on:

```text
Application behavior
Disk growth rate
Log volume
Recovery time
Filesystem characteristics
```

---

# 23. Finding Large Files

When disk usage is high:

```bash
du -sh /*
```

Then investigate specific directories:

```bash
du -sh /var/*
```

For example:

```bash
du -sh /var/log/*
```

This can identify directories consuming significant disk space.

---

# 24. Log Growth

A common cause of disk exhaustion is uncontrolled logs.

Example:

```text
Application
    ↓
Logs
    ↓
/var/log
    ↓
Disk usage ↑
```

Monitor:

```text
Log file size
Log growth rate
Log rotation
Retention
```

---

# 25. Inode Monitoring

Disk space can be available while the filesystem is unable to create new files because all inodes are exhausted.

Check:

```bash
df -i
```

Example:

```text
Filesystem      Inodes   IUsed   IFree IUse%
/dev/xvda1      3.2M     3.2M       0  100%
```

At 100% inode usage, new files may fail even if `df -h` shows free space.

---

# 26. Inode Exhaustion Causes

Common causes:

```text
Millions of small files
Temporary files
Application cache
Log fragments
Mail queues
Build artifacts
```

Troubleshooting should identify directories containing large numbers of files.

---

# 27. Disk I/O Monitoring

Disk capacity and disk performance are different.

A disk can have:

```text
50% free space
```

while still experiencing:

```text
High I/O latency
High IOPS
Throughput saturation
Queue depth
```

Monitor disk performance with:

```bash
iostat
```

---

# 28. `iostat`

Use:

```bash
iostat -xz 1
```

Important fields include:

```text
%util
await
r/s
w/s
rkB/s
wkB/s
```

Interpretation:

```text
r/s      → reads per second
w/s      → writes per second
await    → I/O wait time
%util    → device utilization
```

---

# 29. Disk I/O Troubleshooting

If application latency increases:

```text
Application latency ↑
        ↓
I/O wait ↑
        ↓
Disk latency ↑
        ↓
Investigate storage
```

Check:

```bash
iostat -xz 1
```

Then identify which processes generate the I/O.

---

# 30. `iotop`

`iotop` can show processes generating disk I/O.

Example:

```bash
sudo iotop
```

Useful when identifying:

```text
High-read processes
High-write processes
I/O-heavy applications
```

Availability depends on the Linux distribution and installed packages.

---

# 31. Process Monitoring

Processes are fundamental Linux resources.

Useful commands:

```bash
ps -ef
top
pgrep
pidof
```

Check:

```bash
ps -ef
```

to list running processes.

---

# 32. Finding a Process

Use:

```bash
pgrep nginx
```

or:

```bash
pidof nginx
```

Example:

```text
1234
```

Then inspect:

```bash
ps -p 1234 -f
```

---

# 33. Process CPU and Memory

Use:

```bash
ps -eo pid,ppid,cmd,%cpu,%mem --sort=-%cpu | head
```

This helps identify top CPU consumers.

For memory:

```bash
ps -eo pid,ppid,cmd,%mem --sort=-%mem | head
```

---

# 34. Process States

Linux processes can have different states.

Common states include:

```text
R → Running/Runnable
S → Sleeping
D → Uninterruptible sleep
T → Stopped
Z → Zombie
```

A large number of processes in `D` state can indicate I/O-related problems.

---

# 35. Zombie Processes

A zombie process has completed execution but still has an entry in the process table because its parent has not collected its exit status.

Find zombies:

```bash
ps -eo pid,ppid,state,cmd | awk '$3=="Z"'
```

A small number may not immediately indicate a serious problem, but persistent growth should be investigated.

---

# 36. Process Count

Monitor:

```bash
ps -e --no-headers | wc -l
```

A sudden increase may indicate:

```text
Process leak
Worker explosion
Forking problem
Runaway application
Job scheduler issue
```

---

# 37. File Descriptors

Linux processes use file descriptors for:

```text
Files
Sockets
Pipes
Network connections
```

A process can fail even when CPU and memory are healthy if it reaches its file-descriptor limit.

Check:

```bash
ulimit -n
```

---

# 38. Open File Descriptors

For a process:

```bash
ls /proc/<PID>/fd | wc -l
```

Example:

```bash
ls /proc/1234/fd | wc -l
```

This gives an approximate count of open descriptors for that process.

---

# 39. File Descriptor Exhaustion

Symptoms:

```text
Too many open files
New connections fail
Files cannot be opened
Application errors
Network requests fail
```

Investigate:

```text
Process limits
Open descriptors
Connection leaks
Application configuration
```

---

# 40. Network Monitoring

Important network indicators:

```text
Bandwidth
Packets
Errors
Drops
Connections
Listening ports
TCP states
Network latency
```

Useful commands:

```bash
ss
ip
sar
nload
```

---

# 41. `ss`

Check listening TCP ports:

```bash
ss -lntp
```

Example:

```text
LISTEN 0 128 0.0.0.0:80
LISTEN 0 128 0.0.0.0:443
```

This helps determine whether expected services are listening.

---

# 42. Established Connections

Use:

```bash
ss -ant
```

Filter established connections:

```bash
ss -ant state established
```

A sudden increase can indicate:

```text
Traffic spike
Connection leak
Slow clients
Backend dependency issues
Load imbalance
```

---

# 43. TCP States

Important TCP states include:

```text
LISTEN
ESTABLISHED
TIME-WAIT
CLOSE-WAIT
SYN-SENT
SYN-RECV
```

Monitoring TCP states can reveal network and application problems.

---

# 44. TIME_WAIT

A large number of `TIME_WAIT` connections can occur after many short-lived TCP connections.

Possible contributors:

```text
High request rate
Short-lived connections
Connection handling patterns
Client behavior
```

Do not immediately treat `TIME_WAIT` as a failure. Correlate it with:

```text
Ephemeral ports
Connection rate
Application behavior
Network errors
```

---

# 45. CLOSE_WAIT

A high or continuously increasing `CLOSE_WAIT` count can indicate that an application is not properly closing sockets.

Check:

```bash
ss -ant state close-wait
```

If the number grows continuously:

```text
Application
   ↓
Socket not closed
   ↓
CLOSE_WAIT ↑
   ↓
File descriptors ↑
```

This can eventually cause resource exhaustion.

---

# 46. Network Interface Statistics

Use:

```bash
ip -s link
```

Check:

```text
RX packets
TX packets
RX errors
TX errors
RX dropped
TX dropped
```

Unexpected errors or drops should be investigated.

---

# 47. Network Bandwidth

Monitor:

```text
Receive throughput
Transmit throughput
Packet rate
Errors
Drops
```

A network interface can become saturated even when CPU and memory are normal.

---

# 48. Network Troubleshooting Flow

```text
Application latency
       ↓
Network suspected
       ↓
Check connections
       ↓
Check packet errors
       ↓
Check drops
       ↓
Check bandwidth
       ↓
Check listening ports
       ↓
Check application
```

Useful commands:

```bash
ss -s
ip -s link
ss -lntp
```

---

# 49. Service Monitoring

Production Linux servers run services such as:

```text
nginx
docker
sshd
jenkins
node_exporter
application services
```

For systemd systems:

```bash
systemctl status <service>
```

Example:

```bash
systemctl status nginx
```

---

# 50. Service Health

Check whether a service is:

```text
Active
Inactive
Failed
Restarting
```

Example:

```bash
systemctl is-active nginx
```

A failed service should be investigated using logs.

---

# 51. Restarting a Service

If a service is confirmed to be unhealthy:

```bash
sudo systemctl restart nginx
```

But restarting should not be the first response to every failure.

First determine:

```text
Why did it fail?
Why is it unhealthy?
Will restarting cause impact?
```

---

# 52. System Logs

On systemd systems, use:

```bash
journalctl
```

Examples:

```bash
journalctl -u nginx
```

Last boot:

```bash
journalctl -b
```

Kernel messages:

```bash
journalctl -k
```

Follow logs:

```bash
journalctl -f
```

---

# 53. Service Failure Investigation

Example:

```text
nginx unavailable
```

Check:

```bash
systemctl status nginx
journalctl -u nginx
ss -lntp
```

Then investigate:

```text
Configuration
Port conflicts
Certificate issues
Permission problems
Resource exhaustion
Upstream failures
```

---

# 54. System Boot Monitoring

After a reboot:

```bash
uptime
journalctl -b
systemctl --failed
```

Check:

```text
Failed services
Boot errors
Mount failures
Network initialization
Application startup
```

---

# 55. Failed Services

List failed systemd units:

```bash
systemctl --failed
```

Example:

```text
UNIT            LOAD   ACTIVE SUB    DESCRIPTION
nginx.service   loaded failed failed nginx
```

This is useful during incident investigation.

---

# 56. Kernel Monitoring

Kernel problems can affect the entire server.

Useful commands:

```bash
dmesg
journalctl -k
```

Look for:

```text
OOM
Disk errors
Network errors
Filesystem errors
Hardware issues
Kernel warnings
```

---

# 57. Kernel OOM Investigation

Example:

```bash
journalctl -k | grep -i "out of memory"
```

Possible output:

```text
Out of memory: Killed process 1234
```

Then identify:

```text
Process
Memory usage
Application
Recent workload changes
```

---

# 58. System Load Troubleshooting

Suppose:

```text
Load average = 8
CPU cores = 4
```

Do not immediately conclude:

```text
CPU = 200%
```

Instead investigate:

```text
CPU utilization
I/O wait
Runnable processes
D-state processes
Memory pressure
```

Use:

```bash
top
vmstat 1
```

---

# 59. Linux Monitoring With Prometheus

A common production architecture uses Node Exporter.

```text
Linux Server
     ↓
Node Exporter
     ↓
Prometheus
     ↓
Grafana
```

Node Exporter exposes Linux host metrics to Prometheus.

---

# 60. Node Exporter

Node Exporter collects host-level metrics such as:

```text
CPU
Memory
Filesystem
Disk I/O
Network
Load
Processes
```

It is commonly deployed as a system service or container depending on the environment.

---

# 61. Node Exporter Architecture

```text
                    Linux Server
                         │
                 ┌───────┴───────┐
                 ↓               ↓
              Workloads      Node Exporter
                                 │
                              /metrics
                                 │
                                 ↓
                             Prometheus
                                 │
                                 ↓
                              Grafana
```

---

# 62. Prometheus Scraping

Prometheus periodically collects metrics from the exporter.

Conceptually:

```text
Prometheus
    │
    │ HTTP GET /metrics
    ↓
Node Exporter
```

The exporter exposes metrics such as:

```text
CPU
Memory
Filesystem
Network
```

---

# 63. Grafana Linux Dashboard

A useful dashboard contains:

```text
Linux Host
│
├── CPU Usage
├── Load Average
├── Memory Usage
├── Swap Usage
├── Disk Usage
├── Disk I/O
├── Network Traffic
├── Network Errors
├── Process Count
└── File Descriptors
```

---

# 64. CPU Alert

Example alert condition:

```text
CPU utilization > threshold
for several minutes
```

Before alerting, consider:

```text
Expected workload
Traffic patterns
Number of CPU cores
Deployment activity
Batch processing
```

A short CPU spike may not require an alert.

---

# 65. Memory Alert

A useful memory alert considers available memory rather than simply saying:

```text
Memory usage > 80%
```

Also consider:

```text
Swap activity
Memory pressure
OOM events
Application behavior
```

This reduces false positives.

---

# 66. Disk Alert

Example:

```text
Filesystem usage > 85%
```

But also monitor:

```text
Inodes
Disk growth rate
Critical mount points
```

For example:

```text
/
 /var
 /var/log
 /data
```

may require different thresholds.

---

# 67. Disk I/O Alert

Monitor:

```text
High I/O latency
High utilization
Abnormal throughput
Queue depth
```

Correlate with:

```text
Application latency
I/O wait
Database performance
```

---

# 68. Network Alert

Useful conditions include:

```text
High packet drops
High packet errors
Bandwidth saturation
Unexpected connection growth
```

Investigate whether the problem originates from:

```text
Host
Application
Network interface
Load balancer
Upstream dependency
```

---

# 69. Process Monitoring Alert

Alert on conditions such as:

```text
Critical process stopped
Unexpected process count
Repeated process restarts
Zombie process growth
```

For example:

```text
nginx process absent
```

may require immediate investigation on a standalone Linux server.

---

# 70. Service Availability Monitoring

A service should be monitored at multiple levels:

```text
Process
   ↓
Service
   ↓
Port
   ↓
Application
   ↓
User request
```

Example:

```text
nginx process running
        ↓
port 443 listening
        ↓
HTTP endpoint responds
        ↓
Application returns 200
```

A process being alive does not guarantee that the application is healthy.

---

# 71. Linux Monitoring Golden Signals

For infrastructure, monitor:

```text
Utilization
Saturation
Errors
Availability
```

Examples:

```text
CPU utilization
Disk saturation
Network errors
Service availability
```

---

# 72. Infrastructure Correlation

Suppose:

```text
Application latency ↑
```

Check:

```text
Application
     ↓
CPU
     ↓
Memory
     ↓
Disk I/O
     ↓
Network
     ↓
Dependency
```

This helps distinguish application problems from infrastructure problems.

---

# 73. Example Incident: High CPU

```text
Alert
  ↓
CPU = 95%
  ↓
Check top
  ↓
Identify process
  ↓
Check application logs
  ↓
Check traffic
  ↓
Check recent deployment
  ↓
Determine cause
```

Possible causes:

```text
Traffic spike
Infinite loop
CPU-intensive operation
Bad deployment
Background job
Insufficient capacity
```

---

# 74. Example Incident: High Memory

```text
Alert
  ↓
Available memory ↓
  ↓
Check free -h
  ↓
Check top processes
  ↓
Check swap
  ↓
Check OOM events
  ↓
Check application
  ↓
Identify memory growth
```

Possible causes:

```text
Memory leak
Large workload
Cache growth
Too many processes
Insufficient capacity
```

---

# 75. Example Incident: Disk Full

```text
Alert
  ↓
df -h
  ↓
Identify full filesystem
  ↓
du -sh
  ↓
Find large directory
  ↓
Find large files
  ↓
Check logs
  ↓
Check application data
  ↓
Clean/rotate/archive safely
  ↓
Validate
```

Do not blindly delete files from production systems.

---

# 76. Example Incident: Service Down

```text
Service unavailable
      ↓
systemctl status
      ↓
Check process
      ↓
Check port
      ↓
Check logs
      ↓
Check dependencies
      ↓
Check resources
      ↓
Remediate
      ↓
Validate endpoint
```

---

# 77. Linux Monitoring Runbook

```text
Alert
  ↓
Identify Host
  ↓
Check uptime/load
  ↓
Check CPU
  ↓
Check memory
  ↓
Check disk
  ↓
Check I/O
  ↓
Check network
  ↓
Check processes
  ↓
Check services
  ↓
Check system logs
  ↓
Check application logs
  ↓
Check recent changes
  ↓
Mitigate
  ↓
Validate Recovery
```

---

# 78. Production Monitoring Checklist

```text
Linux Host
├── CPU
├── Load Average
├── Memory
├── Swap
├── Disk Space
├── Inodes
├── Disk I/O
├── Network
├── Processes
├── File Descriptors
├── Services
├── Kernel
└── System Logs

Observability
├── Node Exporter
├── Prometheus
├── Grafana
└── Alerting
```

---

# 79. Commands Cheat Sheet

### CPU

```bash
top
lscpu
nproc
mpstat -P ALL
```

### Memory

```bash
free -h
vmstat 1
```

### Disk

```bash
df -h
df -i
du -sh /*
```

### Disk I/O

```bash
iostat -xz 1
iotop
```

### Processes

```bash
ps -ef
pgrep <process>
pidof <process>
```

### Network

```bash
ss -lntp
ss -ant
ss -s
ip -s link
```

### Services

```bash
systemctl status <service>
systemctl --failed
```

### Logs

```bash
journalctl
journalctl -u <service>
journalctl -k
journalctl -f
```

---

# 80. Interview Question

### What Linux metrics do you monitor in production?

**Answer:**

I monitor CPU utilization, load average, memory availability, swap activity, filesystem usage, inode usage, disk I/O latency and utilization, network throughput and errors, process count, file descriptors, service health, and kernel events. I correlate these host-level metrics with application metrics and logs to determine whether an incident is caused by infrastructure or the application.

---

# 81. Interview Question

### How would you troubleshoot a Linux server with high CPU?

**Answer:**

First I would confirm the CPU utilization and load average using `top` or `uptime`. Then I would identify the processes consuming CPU and determine whether the workload is expected. I would check traffic, recent deployments, background jobs, and application logs. I would also check per-CPU utilization and I/O wait to determine whether the issue is actual CPU saturation or an I/O-related problem.

---

# 82. Interview Question

### How would you troubleshoot high memory usage?

**Answer:**

I would check `free -h` and identify the available memory, swap usage, and top memory-consuming processes. Then I would investigate whether a specific process is continuously growing, check for OOM events, review application behavior, and compare the current state with historical usage. If required, I would profile the application or increase capacity only after understanding the root cause.

---

# 83. Interview Question

### How would you troubleshoot a server with high disk usage?

**Answer:**

I would first use `df -h` to identify the filesystem with high utilization. Then I would use `du` to find the directories consuming the most space. I would inspect logs, application data, temporary files, and old artifacts. I would also check inode usage using `df -i`. I would clean or rotate files safely and verify that the application continues operating normally.

---

# 84. Interview Question

### What is the difference between disk space and disk I/O?

**Answer:**

Disk space represents how much storage capacity is available, while disk I/O represents how actively and efficiently the storage device is processing read and write operations. A server can have plenty of free disk space but still experience high disk latency or I/O saturation. I would use `df` for capacity and `iostat` for I/O performance.

---

# 85. Interview Question

### What would you check if a Linux service is down?

**Answer:**

I would first check `systemctl status` to determine the service state. Then I would inspect the service logs using `journalctl`, check whether the expected port is listening with `ss`, and verify CPU, memory, disk, and network resources. I would also check configuration changes and dependencies. I would restart the service only after understanding the failure or when immediate recovery is necessary and safe.

---

# 86. Interview Question

### What is load average?

**Answer:**

Load average represents the amount of work competing for CPU and certain uninterruptible resources over 1, 5, and 15 minutes. I interpret it together with the number of CPU cores, CPU utilization, I/O wait, and process states. A high load does not automatically mean CPU saturation because I/O-bound tasks can also contribute to load.

---

# 87. Interview Question

### How do you monitor Linux servers using Prometheus?

**Answer:**

I deploy Node Exporter on the Linux hosts. Node Exporter exposes host-level metrics such as CPU, memory, filesystem, disk, network, and load metrics. Prometheus scrapes the exporter, and Grafana is used to visualize the metrics and create dashboards and alerts. I would also monitor the exporter and Prometheus themselves to make sure telemetry is being collected reliably.

---

# 88. Interview Question

### How would you investigate a server that is slow but CPU usage is low?

**Answer:**

I would not assume that low CPU means the server is healthy. I would check I/O wait, disk latency, memory pressure, swap activity, network latency, process states, database dependencies, and filesystem behavior. Commands such as `vmstat`, `iostat`, `free`, `ss`, and `top` can help identify whether the bottleneck is storage, memory, network, or a dependency.

---

# 89. Interview Question

### What is the difference between a process being alive and an application being healthy?

**Answer:**

A process being alive only means the process exists. It does not guarantee that the application can serve requests correctly. For example, an application process may be running while its database connection is broken or its HTTP listener is unavailable. Therefore production monitoring should validate the process, service, port, application endpoint, and critical dependencies.

---

# 90. Final Mental Model

```text
                         LINUX SERVER
                              │
       ┌──────────────────────┼──────────────────────┐
       ↓                      ↓                      ↓
      CPU                   MEMORY                 STORAGE
       │                      │                      │
       ├── Usage              ├── RAM                ├── Space
       ├── Load               ├── Available         ├── Inodes
       ├── I/O Wait           ├── Swap              ├── IOPS
       └── Steal              └── OOM               └── Latency
                              │
                              ↓
                         PROCESSES
                              │
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                 CPU/RAM    States     FDs
                              │
                              ↓
                           SERVICES
                              │
                              ↓
                           NETWORK
                              │
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                 Traffic   Errors    Connections
                              │
                              ↓
                       OBSERVABILITY
                              │
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
               Node Exporter Prometheus Grafana
                              │
                              ↓
                           ALERTS
                              │
                              ↓
                       TROUBLESHOOTING
                              │
                              ↓
                           ROOT CAUSE
                              │
                              ↓
                         REMEDIATION
```

**Key principle:** Linux monitoring is the foundation of infrastructure monitoring. Always correlate **CPU, load, memory, disk, I/O, network, processes, services, and system logs** rather than investigating a single metric in isolation. In a production DevOps environment, **Node Exporter + Prometheus + Grafana** provides the foundation for centralized Linux infrastructure monitoring.
