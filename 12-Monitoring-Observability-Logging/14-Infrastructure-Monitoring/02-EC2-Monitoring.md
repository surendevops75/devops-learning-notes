# EC2 Monitoring

## 1. Overview

EC2 monitoring is the process of observing the health, performance, resource utilization, networking, storage, and application behavior of Amazon EC2 instances.

A production EC2 instance can be:

```text
Running
```

and still have serious problems such as:

```text
High CPU
Memory exhaustion
Disk full
High disk I/O
Network saturation
Application failure
Process failure
Connection exhaustion
```

Therefore EC2 monitoring should cover multiple layers:

```text
EC2 Instance
│
├── Instance Health
├── CPU
├── Memory
├── Disk
├── Network
├── Processes
├── Applications
├── System Logs
└── Dependencies
```

---

# 2. EC2 Monitoring Layers

A practical monitoring model is:

```text
                         EC2 INSTANCE
                              │
        ┌─────────────────────┼─────────────────────┐
        ↓                     ↓                     ↓
     AWS Layer            OS Layer            Application Layer
        │                     │                     │
   Instance State          CPU/RAM              Processes
   Status Checks            Disk                 Services
   Network                  Network              Application
   EBS                      Processes            Logs
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ↓
                         Observability
                              │
                     Prometheus / Grafana
```

---

# 3. EC2 Monitoring Categories

Monitor:

```text
Instance
├── Running state
├── Status checks
└── Availability

Compute
├── CPU
├── Load
└── CPU credit behavior

Memory
├── RAM
├── Swap
└── Memory pressure

Storage
├── EBS
├── Filesystem
├── Disk space
└── Disk I/O

Network
├── Bytes
├── Packets
├── Connections
└── Network errors

Application
├── Processes
├── Services
├── Logs
└── Health endpoints
```

---

# 4. EC2 Instance State

EC2 instances have lifecycle states such as:

```text
pending
running
stopping
stopped
shutting-down
terminated
```

For production monitoring, an unexpected transition from:

```text
running
```

to:

```text
stopped
```

should be investigated.

Possible causes include:

```text
Manual action
Automation
Scheduled shutdown
Instance failure
Operational mistake
```

---

# 5. EC2 Status Checks

EC2 provides status checks to identify instance and system-level problems.

Two important categories are:

```text
System status check
Instance status check
```

---

# 6. System Status Check

A system status check detects problems affecting the underlying AWS infrastructure supporting the instance.

Possible causes include:

```text
Underlying hardware issue
Network infrastructure problem
Host-level AWS infrastructure issue
```

Conceptually:

```text
AWS Infrastructure
       ↓
EC2 Host
       ↓
Instance
```

A system status check failure may indicate an issue below the guest operating system.

---

# 7. Instance Status Check

An instance status check focuses on the instance itself.

Potential causes include:

```text
Operating system problem
Boot problem
Network configuration issue
Resource-related problem
System initialization failure
```

Conceptually:

```text
EC2 Host
   ↓
EC2 Instance
   ↓
Guest OS
```

---

# 8. Status Check Troubleshooting

If a status check fails:

```text
1. Identify which check failed
2. Check instance state
3. Check system/application logs
4. Check recent changes
5. Check networking
6. Check OS health
7. Determine whether recovery/restart is appropriate
```

Do not immediately reboot without understanding the failure when the system is production-critical.

---

# 9. EC2 CPU Monitoring

CPU is one of the primary EC2 metrics.

Monitor:

```text
CPU utilization
Load average
CPU saturation
CPU credits for burstable instances
```

Example:

```text
CPU Utilization
│
│             █████████
│          ████████████
│       ███████████████
│____██████████████████
└──────────────────────→ Time
```

---

# 10. High CPU Causes

Common causes include:

```text
High application traffic
CPU-intensive application
Background jobs
Large batch processing
Encryption/compression
Runaway process
Infinite loop
Insufficient instance capacity
```

Troubleshooting:

```bash
top
ps -eo pid,ppid,cmd,%cpu --sort=-%cpu | head
```

---

# 11. EC2 CPU and Application Correlation

Suppose:

```text
CPU = 95%
```

Do not immediately resize the instance.

Check:

```text
Traffic
Application process
Recent deployment
Background jobs
Database activity
Network activity
```

Example:

```text
Traffic ↑
   ↓
Requests ↑
   ↓
Application CPU ↑
   ↓
Latency ↑
```

---

# 12. Burstable EC2 Instances

Some EC2 instance families use burstable CPU behavior.

Examples include:

```text
T-family instances
```

These instances use CPU credits.

Conceptually:

```text
Normal workload
      ↓
CPU Credits accumulate
      ↓
Traffic spike
      ↓
CPU burst
      ↓
Credits consumed
```

Therefore CPU utilization alone is not enough for monitoring burstable instances.

---

# 13. CPU Credit Monitoring

For burstable instances, monitor relevant CPU credit metrics.

Important concepts include:

```text
CPU credit balance
CPU credit usage
```

If credits become exhausted, sustained CPU performance may be affected depending on the instance configuration.

---

# 14. CPU Credit Troubleshooting

Suppose:

```text
CPU = 80%
CPU credits ↓ rapidly
```

Investigate:

```text
Traffic
Workload pattern
Application behavior
Instance type
CPU credit balance
```

If high CPU is normal for the workload, consider whether the instance family is appropriate.

---

# 15. EC2 Memory Monitoring

EC2 monitoring does not automatically provide all guest OS memory metrics through basic instance metrics.

For detailed memory monitoring, collect metrics from inside the instance.

For Linux:

```bash
free -h
vmstat 1
```

With Prometheus:

```text
EC2
 ↓
Node Exporter
 ↓
Prometheus
 ↓
Grafana
```

---

# 16. Why Memory Monitoring Matters

An EC2 instance may show:

```text
CPU = 30%
```

while memory is:

```text
Memory = 95%
```

The application can still fail.

Possible symptoms:

```text
Application crashes
OOM killer
Slow performance
Swap activity
Failed allocations
Service restarts
```

---

# 17. Memory Monitoring Architecture

```text
                 EC2
                  │
                  ↓
             Linux OS
                  │
           Node Exporter
                  │
                  ↓
             Prometheus
                  │
                  ↓
               Grafana
```

Useful metrics include:

```text
Available memory
Used memory
Swap
Memory pressure
```

---

# 18. EC2 Disk Monitoring

EC2 storage commonly uses:

```text
EBS volumes
```

Inside the Linux guest, monitor:

```text
Filesystem usage
Inodes
Disk I/O
Mount points
```

Commands:

```bash
df -h
df -i
```

---

# 19. EBS Monitoring

At the AWS storage layer, monitor:

```text
Volume performance
IOPS
Throughput
Queue behavior
Burst/credit behavior where applicable
```

The exact metrics depend on the EBS volume type.

---

# 20. EBS Capacity vs Performance

Two separate questions must be answered:

```text
Is the volume large enough?
```

and:

```text
Is the volume fast enough?
```

Example:

```text
Disk space:
30% used

But:

Disk I/O:
High latency
```

The volume has enough capacity but may still have a performance bottleneck.

---

# 21. Filesystem Monitoring

Inside EC2:

```bash
df -h
```

Example:

```text
Filesystem      Size  Used Avail Use%
/dev/xvda1       50G   45G    5G  90%
```

Monitor critical filesystems such as:

```text
/
 /var
 /var/log
 /data
```

depending on the application architecture.

---

# 22. Inode Monitoring

Check:

```bash
df -i
```

A filesystem can have:

```text
Disk space available
```

but:

```text
Inodes exhausted
```

This can prevent creation of new files.

Common causes:

```text
Millions of small files
Temporary files
Application cache
Log fragments
```

---

# 23. Disk I/O Monitoring

Inside EC2, use:

```bash
iostat -xz 1
```

Monitor:

```text
Read IOPS
Write IOPS
Latency
Utilization
Throughput
Queue behavior
```

High disk latency can affect:

```text
Application response time
Database performance
Log processing
Build jobs
```

---

# 24. EC2 Network Monitoring

Monitor network traffic at the instance level.

Important metrics include:

```text
Network bytes in
Network bytes out
Packets in
Packets out
```

Also monitor from inside Linux:

```bash
ss
ip -s link
```

---

# 25. Network Saturation

A network-heavy application may experience:

```text
High throughput
High packet rate
Connection growth
Network errors
```

Correlate network metrics with:

```text
Application latency
CPU
Connections
External dependencies
```

---

# 26. Network Connection Monitoring

Use:

```bash
ss -s
```

and:

```bash
ss -ant
```

Monitor:

```text
ESTABLISHED
TIME-WAIT
CLOSE-WAIT
SYN-SENT
SYN-RECV
```

A continuously increasing connection count may indicate:

```text
Traffic growth
Connection leak
Slow clients
Backend dependency issue
Application problem
```

---

# 27. EC2 Security Group and Monitoring

Security Groups control network traffic but are not themselves application health monitoring.

If an application is unreachable:

```text
Client
 ↓
Security Group
 ↓
Network
 ↓
EC2
 ↓
Application
```

Check each layer.

Do not assume that a running instance means the application is reachable.

---

# 28. EC2 Application Monitoring

Infrastructure monitoring should be combined with application monitoring.

Example:

```text
EC2
│
├── CPU
├── Memory
├── Disk
└── Network
       │
       ↓
Application
├── Requests
├── Latency
├── Errors
├── Processes
└── Health
```

This helps determine whether the problem is infrastructure or application-related.

---

# 29. Process Monitoring on EC2

Useful commands:

```bash
ps -ef
top
pgrep <process>
pidof <process>
```

For CPU:

```bash
ps -eo pid,cmd,%cpu --sort=-%cpu | head
```

For memory:

```bash
ps -eo pid,cmd,%mem --sort=-%mem | head
```

---

# 30. Service Monitoring

For systemd:

```bash
systemctl status <service>
```

Example:

```bash
systemctl status nginx
```

List failed services:

```bash
systemctl --failed
```

Monitor important production services such as:

```text
nginx
docker
sshd
jenkins
application services
node_exporter
```

---

# 31. EC2 System Logs

For Linux EC2 instances:

```bash
journalctl
```

Useful commands:

```bash
journalctl -k
journalctl -b
journalctl -u <service>
journalctl -f
```

Check for:

```text
OOM
Disk errors
Network failures
Service failures
Kernel problems
Boot problems
```

---

# 32. EC2 Application Logs

Application logs may be located in:

```text
/var/log/
/var/log/<application>/
```

or sent directly to:

```text
stdout/stderr
```

For centralized logging:

```text
EC2
 ↓
Log Collector
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

---

# 33. Prometheus Monitoring for EC2

A common DevOps architecture is:

```text
                  EC2
                   │
                   ↓
              Node Exporter
                   │
                   ↓
              Prometheus
                   │
                   ↓
                Grafana
```

Node Exporter collects host-level Linux metrics.

---

# 34. Node Exporter Metrics

Typical host-level metrics include:

```text
CPU
Memory
Filesystem
Disk I/O
Network
Load average
Processes
```

These provide deeper OS visibility than basic EC2 infrastructure metrics.

---

# 35. EC2 + Prometheus Architecture

For multiple instances:

```text
EC2-1 ── Node Exporter ──┐
EC2-2 ── Node Exporter ──┤
EC2-3 ── Node Exporter ──┼──→ Prometheus
EC2-4 ── Node Exporter ──┤
EC2-5 ── Node Exporter ──┘
                              ↓
                           Grafana
```

Prometheus can collect metrics from all instances.

---

# 36. EC2 Labels

For multiple EC2 instances, identify instances using labels such as:

```text
instance
environment
service
application
team
region
availability_zone
```

Example:

```text
environment="production"
service="payment"
```

This makes dashboards and alerts easier to organize.

---

# 37. EC2 Dashboard

A useful dashboard:

```text
┌────────────────────────────────────────────┐
│               EC2 OVERVIEW                 │
├────────────────────────────────────────────┤
│ CPU │ Memory │ Disk │ Network │ Load       │
├────────────────────────────────────────────┤
│ CPU Utilization                           │
├────────────────────────────────────────────┤
│ Load Average                              │
├────────────────────────────────────────────┤
│ Memory / Swap                             │
├────────────────────────────────────────────┤
│ Filesystem Usage                          │
├────────────────────────────────────────────┤
│ Disk I/O                                  │
├────────────────────────────────────────────┤
│ Network Traffic                            │
├────────────────────────────────────────────┤
│ Process Count                              │
├────────────────────────────────────────────┤
│ Instance Status                            │
└────────────────────────────────────────────┘
```

---

# 38. EC2 Monitoring at Two Levels

A production setup should distinguish:

```text
AWS Infrastructure Metrics
```

from:

```text
Guest OS Metrics
```

Conceptually:

```text
AWS Layer
├── Instance status
├── CPU
├── Network
└── EBS

Guest OS
├── Memory
├── Filesystem
├── Processes
├── Services
└── Load
```

Both layers are required for complete visibility.

---

# 39. Why One Monitoring Source Is Not Enough

Suppose:

```text
EC2 CPU = 30%
```

This does not tell us:

```text
Memory usage
Disk usage
Application health
Service status
File descriptor usage
```

Therefore combine:

```text
AWS metrics
+
Linux metrics
+
Application metrics
+
Logs
```

---

# 40. EC2 High CPU Troubleshooting

Scenario:

```text
EC2 CPU = 95%
```

Workflow:

```text
1. Confirm CPU metric
2. Check load average
3. Check top processes
4. Identify application
5. Check traffic
6. Check recent deployment
7. Check background jobs
8. Check CPU credits if burstable
9. Determine scaling/optimization action
10. Validate recovery
```

---

# 41. EC2 High Memory Troubleshooting

Scenario:

```text
Memory = 95%
```

Workflow:

```text
1. Check free -h
2. Check swap
3. Identify top processes
4. Check memory growth
5. Check OOM events
6. Check application logs
7. Check recent changes
8. Determine remediation
9. Validate recovery
```

---

# 42. EC2 Disk Full Troubleshooting

Scenario:

```text
Filesystem = 90%
```

Workflow:

```text
df -h
   ↓
Identify filesystem
   ↓
du -sh
   ↓
Find large directory
   ↓
Check logs/data
   ↓
Check inode usage
   ↓
Clean/archive/rotate safely
   ↓
Validate
```

---

# 43. EC2 Disk Performance Troubleshooting

Scenario:

```text
Application latency ↑
```

Check:

```text
CPU
Memory
I/O wait
iostat
EBS performance
Application logs
```

Possible flow:

```text
Application latency
       ↓
I/O wait ↑
       ↓
Disk latency ↑
       ↓
EBS/storage investigation
```

---

# 44. EC2 Network Troubleshooting

Scenario:

```text
Application unreachable
```

Check:

```text
1. Instance running?
2. Status checks healthy?
3. Security Group?
4. Network ACL?
5. Route?
6. Port listening?
7. Application running?
8. Network connections?
9. Application logs?
```

Inside the instance:

```bash
ss -lntp
ss -s
ip -s link
```

---

# 45. EC2 Service Down Troubleshooting

Scenario:

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
Port conflict
Certificate
Permissions
Disk
Memory
Upstream
Network
```

---

# 46. EC2 Instance Unreachable

A production troubleshooting sequence:

```text
Client
  ↓
DNS
  ↓
Load Balancer
  ↓
Security Group
  ↓
Network ACL
  ↓
Route
  ↓
EC2
  ↓
OS
  ↓
Service
  ↓
Application
```

Check the path systematically instead of jumping directly to the EC2 server.

---

# 47. EC2 Auto Scaling

For scalable workloads:

```text
Traffic ↑
   ↓
CPU / Request Metrics ↑
   ↓
Auto Scaling Group
   ↓
New EC2 Instances
   ↓
Capacity ↑
```

Monitor:

```text
Desired capacity
Current capacity
Healthy instances
Unhealthy instances
Scaling activities
```

---

# 48. Auto Scaling Monitoring

An unhealthy scaling system can cause:

```text
Traffic ↑
   ↓
Instances become overloaded
   ↓
Scaling fails
   ↓
Latency ↑
```

Monitor both:

```text
Application metrics
```

and:

```text
Auto Scaling behavior
```

---

# 49. EC2 Health During Deployment

Before deployment:

```text
CPU normal
Memory normal
Disk normal
Application healthy
```

During deployment:

```text
New version
   ↓
Process restart
   ↓
Health validation
   ↓
Traffic
```

Monitor:

```text
CPU
Memory
Errors
Latency
Process restarts
Application logs
```

---

# 50. EC2 Monitoring During Incident Response

During an incident:

```text
Alert
  ↓
Identify affected instance
  ↓
Check AWS status
  ↓
Check CPU
  ↓
Check memory
  ↓
Check disk
  ↓
Check network
  ↓
Check processes
  ↓
Check services
  ↓
Check logs
  ↓
Check application
  ↓
Mitigate
  ↓
Validate
```

---

# 51. EC2 Monitoring and Logging

Metrics:

```text
CPU
Memory
Disk
Network
```

Logs:

```text
Application errors
OS errors
Service failures
Kernel events
```

Together:

```text
Metrics
   ↓
Problem detected
   ↓
Logs
   ↓
Root cause evidence
```

---

# 52. EC2 Monitoring and Tracing

For microservices:

```text
EC2
 ↓
Application
 ↓
OpenTelemetry
 ↓
Trace
```

Tracing can show whether the EC2-hosted service is slow because of:

```text
Application
Database
External API
Network
```

---

# 53. Complete EC2 Observability

```text
                         EC2
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
   AWS Metrics        Linux Metrics      Application
       │                  │                  │
   CPU               CPU/RAM             Requests
   Network           Disk                Latency
   EBS               Network             Errors
   Status            Processes           Health
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                    Observability
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Metrics           Logs           Traces
          ↓               ↓               ↓
      Prometheus          ELK        OpenTelemetry
          ↓               ↓               ↓
       Grafana          Kibana           Jaeger
```

---

# 54. EC2 Production Alerts

Useful alerts include:

```text
Instance status check failure
High CPU
Low available memory
High swap activity
High filesystem usage
High inode usage
High disk I/O latency
Network errors
Critical service down
Unexpected process restart
High file descriptor usage
OOM events
Instance termination
Auto Scaling health problems
```

Alerts should be actionable and tuned to the workload.

---

# 55. Alert Example: CPU

```text
EC2 CPU > threshold
for several minutes
```

Before remediation:

```text
Check:
├── Traffic
├── Top processes
├── Load
├── CPU credits
└── Recent deployment
```

Then:

```text
Optimize
or
Scale
or
Resize
```

---

# 56. Alert Example: Disk

```text
Filesystem usage > threshold
```

Investigation:

```text
df -h
   ↓
df -i
   ↓
du
   ↓
Identify growth
   ↓
Log/data investigation
```

---

# 57. Alert Example: Status Check

```text
EC2 status check failed
```

Investigation:

```text
System status?
Instance status?
OS logs?
Network?
Boot?
AWS infrastructure issue?
```

Then determine whether:

```text
Recovery
Restart
Replacement
```

is appropriate.

---

# 58. EC2 Monitoring Best Practices

```text
1. Monitor AWS-level metrics.
2. Monitor guest OS metrics.
3. Monitor application metrics.
4. Monitor filesystem usage.
5. Monitor inode usage.
6. Monitor disk performance.
7. Monitor memory and swap.
8. Monitor CPU and load.
9. Monitor network connections.
10. Monitor critical services.
11. Monitor status checks.
12. Monitor burstable CPU credits where applicable.
13. Use Node Exporter for Linux metrics.
14. Use Prometheus for metric collection.
15. Use Grafana for dashboards.
16. Centralize application and system logs.
17. Correlate metrics with logs and traces.
18. Monitor Auto Scaling behavior.
19. Create actionable alerts.
20. Test monitoring and recovery procedures.
```

---

# 59. Common EC2 Monitoring Mistakes

### Mistake 1: Monitoring Only CPU

```text
CPU = 30%
```

does not prove the server is healthy.

### Mistake 2: Ignoring Memory

Memory exhaustion can cause application failures even with low CPU.

### Mistake 3: Ignoring Disk

Disk exhaustion can cause:

```text
Application failures
Log failures
Database failures
Boot problems
```

### Mistake 4: Ignoring Inodes

A filesystem can have free space but no available inodes.

### Mistake 5: Ignoring Guest OS Metrics

AWS-level metrics alone do not provide complete Linux visibility.

### Mistake 6: Restarting Without Investigation

Restarting can temporarily hide the actual problem.

---

# 60. Production EC2 Monitoring Checklist

```text
AWS Layer
├── Instance state
├── Status checks
├── CPU
├── Network
└── EBS

Linux Layer
├── CPU
├── Load
├── Memory
├── Swap
├── Filesystem
├── Inodes
├── Disk I/O
├── Processes
├── File descriptors
├── Services
└── Kernel

Application Layer
├── Requests
├── Latency
├── Errors
├── Health
└── Dependencies

Observability
├── Node Exporter
├── Prometheus
├── Grafana
├── ELK
└── OpenTelemetry/Jaeger
```

---

# 61. Interview Question

### How do you monitor EC2 instances in production?

**Answer:**

I monitor EC2 at three levels: AWS infrastructure, guest operating system, and application. At the AWS level I monitor instance status, CPU, network, and EBS-related metrics. Inside Linux I monitor memory, swap, filesystem, inode usage, disk I/O, processes, services, and network connections using tools such as `top`, `free`, `df`, `iostat`, and `ss`. For centralized monitoring, I can use Node Exporter with Prometheus and Grafana, while ELK and OpenTelemetry provide logging and tracing.

---

# 62. Interview Question

### What is the difference between EC2 monitoring and Linux monitoring?

**Answer:**

EC2 monitoring focuses on the AWS infrastructure and instance-level metrics such as CPU, network, EBS, and instance status checks. Linux monitoring focuses on the guest operating system, including memory, filesystems, processes, services, disk I/O, network connections, and kernel events. Both are required for complete visibility.

---

# 63. Interview Question

### How would you troubleshoot an EC2 instance with high CPU?

**Answer:**

First I would confirm the AWS CPU metric and check the load average inside Linux. Then I would use `top` or `ps` to identify the process consuming CPU. I would check application traffic, recent deployments, background jobs, and CPU credit behavior if it is a burstable instance. Based on the root cause, I would optimize the workload, scale horizontally, or resize the instance.

---

# 64. Interview Question

### EC2 CPU is normal, but the application is slow. What would you check?

**Answer:**

I would not assume the instance is healthy just because CPU is normal. I would check memory, swap, disk I/O latency, I/O wait, filesystem pressure, network connections, application processes, database latency, and external dependencies. I would also inspect application logs and traces to determine where the latency is occurring.

---

# 65. Interview Question

### How would you troubleshoot an EC2 disk-full issue?

**Answer:**

I would first run `df -h` to identify the full filesystem and `df -i` to check inode usage. Then I would use `du` to identify large directories and inspect logs, application data, temporary files, and artifacts. I would safely rotate, archive, or remove unnecessary data according to the application's requirements, then verify filesystem health.

---

# 66. Interview Question

### What is the difference between EC2 status checks and application health?

**Answer:**

EC2 status checks validate the underlying instance and system health at the AWS level. Application health is concerned with whether the actual service can process requests correctly. An EC2 instance can pass its status checks while the application is stopped or returning errors. Therefore application-level health checks are still required.

---

# 67. Interview Question

### How would you monitor memory on EC2?

**Answer:**

I would collect guest OS memory metrics using a mechanism such as Node Exporter and expose them to Prometheus. I would monitor available memory, memory utilization, swap activity, and OOM events. I would visualize these in Grafana and correlate memory issues with application processes and deployment changes.

---

# 68. Interview Question

### How would you monitor multiple EC2 instances?

**Answer:**

I would install or deploy Node Exporter on the Linux instances and configure Prometheus to scrape them. I would use labels such as environment, service, instance, region, and availability zone to organize the metrics. Grafana dashboards would provide fleet-level and instance-level visibility, and alerts would detect CPU, memory, disk, network, and availability problems.

---

# 69. Final Mental Model

```text
                           AWS
                            │
                       EC2 Instance
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
       Status              CPU             Network
       Checks               │                 │
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ↓
                         Linux OS
                            │
       ┌────────────────────┼────────────────────┐
       ↓                    ↓                    ↓
     Memory              Storage              Processes
       │                    │                    │
      RAM                  EBS                 Services
      Swap               Filesystem             │
       │                 Disk I/O                │
       └────────────────────┼────────────────────┘
                            ↓
                       Application
                            │
                 ┌──────────┼──────────┐
                 ↓          ↓          ↓
              Metrics      Logs      Traces
                 ↓          ↓          ↓
             Prometheus     ELK    OpenTelemetry
                 ↓          ↓          ↓
              Grafana     Kibana     Jaeger
                 │          │          │
                 └──────────┼──────────┘
                            ↓
                       Root Cause
                            ↓
                        Remediation
                            ↓
                    Validate Recovery
```

**Key principle:** EC2 monitoring should never depend on a single metric. A production EC2 instance must be monitored across **AWS instance health, CPU, memory, disk, EBS, network, processes, services, and applications**, with **Prometheus/Grafana, centralized logging, and tracing** providing deeper observability.
