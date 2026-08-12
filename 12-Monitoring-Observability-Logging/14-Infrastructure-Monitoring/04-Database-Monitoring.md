# Database Monitoring

## 1. Overview

Database monitoring is the process of continuously observing database health, performance, availability, connections, queries, storage, resources, replication, and errors.

For a DevOps engineer, database monitoring is critical because application performance is often directly dependent on database performance.

A typical production request path is:

```text
Client
   ↓
Load Balancer
   ↓
Application
   ↓
Database
   ↓
Storage
```

A database problem can appear to users as:

```text
High application latency
Timeouts
5xx errors
Connection failures
Slow API responses
Failed transactions
Application crashes
```

Therefore database monitoring should cover:

```text
Availability
Connections
CPU
Memory
Storage
Disk I/O
Latency
Query Performance
Locks
Transactions
Errors
Replication
Backups
Connection Pool
```

---

# 2. Database Monitoring Layers

A practical production database monitoring model is:

```text
                         DATABASE MONITORING
                                │
        ┌───────────────────────┼───────────────────────┐
        ↓                       ↓                       ↓
   Infrastructure          Database Engine          Application
        │                       │                       │
   ┌────┼────┐            ┌─────┼─────┐          ┌─────┼─────┐
   ↓    ↓    ↓            ↓     ↓     ↓          ↓     ↓     ↓
 CPU  RAM  Disk          Queries Locks Connections Latency Errors
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                ↓
                         Database Health
```

The objective is to answer:

```text
Is the database available?
Is the database accepting connections?
Is CPU under control?
Is memory sufficient?
Is storage sufficient?
Are queries slow?
Are connections exhausted?
Are locks increasing?
Are errors increasing?
Is replication healthy?
Are backups successful?
```

---

# 3. Important Database Metrics

Important production database metrics include:

```text
CPU Utilization
Memory Utilization
Storage Utilization
Disk IOPS
Disk Throughput
Disk Latency
Database Connections
Active Connections
Idle Connections
Query Latency
Query Throughput
Transaction Rate
Lock Waits
Deadlocks
Cache Hit Ratio
Replication Lag
Errors
Connection Failures
Backup Status
```

Metrics should always be correlated.

For example:

```text
CPU ↑
+
Query Latency ↑
+
Active Connections ↑
```

may indicate query or workload pressure.

---

# 4. Database Availability

Database availability means that applications can successfully connect to and use the database.

A database process running does not necessarily mean the database is healthy.

Example:

```text
Database Process
      ↓
Running
      ↓
Connections Exhausted
      ↓
Application Cannot Connect
```

Therefore monitor:

```text
Process availability
Connection availability
Query availability
Application-level health
```

---

# 5. Database Connectivity

Basic connectivity testing depends on the database engine.

For PostgreSQL:

```bash
pg_isready -h <host> -p 5432
```

For MySQL:

```bash
mysqladmin ping -h <host> -u <user> -p
```

For TCP-level testing:

```bash
nc -vz <host> <port>
```

The troubleshooting sequence is:

```text
DNS
 ↓
Network
 ↓
Port
 ↓
Database Listener
 ↓
Authentication
 ↓
Database
```

---

# 6. Database Port Monitoring

Common database ports include:

```text
PostgreSQL → 5432
MySQL      → 3306
MongoDB    → 27017
Redis      → 6379
```

Check connectivity:

```bash
nc -vz <database-host> <port>
```

Example:

```bash
nc -vz db.example.internal 5432
```

A successful TCP connection does not guarantee successful authentication.

---

# 7. CPU Monitoring

Database CPU utilization should be monitored continuously.

Example:

```text
CPU = 90%
```

Do not immediately increase the instance size.

Investigate:

```text
Traffic
Query rate
Slow queries
Expensive queries
Background jobs
Indexes
Connection count
Recent deployment
```

Typical relationship:

```text
Traffic ↑
   ↓
Queries ↑
   ↓
CPU ↑
   ↓
Query Latency ↑
   ↓
Application Latency ↑
```

---

# 8. CPU Troubleshooting

If database CPU suddenly increases:

```text
1. Check query rate.
2. Check active connections.
3. Identify expensive queries.
4. Check recent deployments.
5. Check missing or inefficient indexes.
6. Check database locks.
7. Check background jobs.
8. Check application traffic.
```

The goal is to identify why CPU increased before scaling resources.

---

# 9. Memory Monitoring

Memory is critical for database performance.

Monitor:

```text
Used Memory
Available Memory
Cache
Buffer Pool
Swap
Memory Pressure
```

High memory pressure can cause:

```text
Disk activity
Query latency
Cache misses
Performance degradation
OOM conditions
```

A database with sufficient memory can keep frequently accessed data in memory and reduce disk operations.

---

# 10. Database Cache

Databases use memory for caching data and indexes.

Conceptually:

```text
Application Query
      ↓
Database
      ↓
Cache
   ↙     ↘
Hit       Miss
 ↓         ↓
Return    Disk
            ↓
          Data
```

A high cache hit ratio generally indicates that many requested pages are served from memory.

Low cache performance may increase disk I/O.

---

# 11. Storage Monitoring

Monitor database storage continuously.

Important metrics:

```text
Storage Used
Storage Available
Storage Growth
Storage Utilization
```

Example:

```text
Database Storage = 500 GB

Used = 450 GB
Free = 50 GB
```

If storage continues increasing:

```text
450 GB
 ↓
460 GB
 ↓
470 GB
 ↓
480 GB
```

the team should investigate growth before the database reaches capacity.

---

# 12. Storage Exhaustion

Database storage exhaustion can cause:

```text
Insert failures
Transaction failures
Log failures
Database instability
Application errors
Database crash
```

Therefore alert before storage becomes critical.

Example:

```text
Warning  → 70%
Critical → 85%
```

Actual thresholds should be based on workload and growth rate.

---

# 13. Disk I/O Monitoring

Database performance is often heavily dependent on disk I/O.

Monitor:

```text
Read IOPS
Write IOPS
Read Throughput
Write Throughput
Disk Latency
Queue Depth
Utilization
```

High disk latency can cause:

```text
Slow queries
High transaction latency
Application latency
Timeouts
```

---

# 14. Disk Latency

Suppose:

```text
Database Query
      ↓
Disk Read
      ↓
High Latency
      ↓
Query Slows
```

If many queries are waiting on disk:

```text
Disk Latency ↑
      ↓
Query Latency ↑
      ↓
Connection Count ↑
      ↓
Application Latency ↑
```

This is why database and infrastructure metrics should be correlated.

---

# 15. Query Monitoring

Query monitoring is one of the most important parts of database monitoring.

Monitor:

```text
Query Rate
Query Latency
Slow Queries
Failed Queries
Queries per Second
Rows Read
Rows Returned
Execution Time
```

The goal is to identify queries that consume excessive resources.

---

# 16. Slow Query Monitoring

Slow queries can cause:

```text
High CPU
High Disk I/O
Connection Growth
Locking
Application Latency
Timeouts
```

Example:

```text
Normal Query = 20 ms

Problem Query = 5000 ms
```

If the application executes the problem query thousands of times, database performance can degrade rapidly.

---

# 17. Query Performance Investigation

When a query becomes slow, investigate:

```text
Execution Plan
Indexes
Rows Scanned
Rows Returned
Joins
Sorting
Filtering
Locks
Database Statistics
```

The important question is:

```text
Why did this query become expensive?
```

Do not immediately scale the database without understanding the workload.

---

# 18. Database Connections

Monitor:

```text
Maximum Connections
Active Connections
Idle Connections
Connection Usage
Connection Failures
```

Example:

```text
Maximum Connections = 500
Active Connections  = 480
```

The database is close to its connection limit.

If the application attempts additional connections:

```text
Connection Request
       ↓
Database
       ↓
Connection Limit Reached
       ↓
Connection Failure
```

---

# 19. Connection Pool Monitoring

Applications commonly use connection pools.

Architecture:

```text
Application
     ↓
Connection Pool
     ↓
Database
```

Monitor:

```text
Pool Size
Active Connections
Idle Connections
Waiting Requests
Connection Creation
Connection Timeout
```

A connection pool problem can appear as a database problem even when the database itself is healthy.

---

# 20. Connection Exhaustion

Scenario:

```text
Database Max Connections = 500
Active Connections = 500
```

New application requests may fail.

Possible causes:

```text
Traffic spike
Connection leak
Pool misconfiguration
Slow queries
Long-running transactions
Too many application instances
```

Investigate both:

```text
Application Connection Pool
+
Database Connection Limit
```

---

# 21. Connection Leak

A connection leak occurs when an application obtains a database connection but does not properly return or close it.

Possible flow:

```text
Request
  ↓
Connection Acquired
  ↓
Query
  ↓
Connection Not Released
  ↓
Another Request
  ↓
Another Connection
  ↓
Connections ↑
```

Eventually:

```text
Connection Limit Reached
```

Monitor connection growth over time.

---

# 22. Transactions

Monitor database transaction behavior.

Important metrics include:

```text
Transaction Rate
Committed Transactions
Rolled-back Transactions
Long-running Transactions
Transaction Latency
```

A sudden increase in rollbacks can indicate:

```text
Application errors
Constraint violations
Deadlocks
Timeouts
Database problems
```

---

# 23. Long-Running Transactions

Long-running transactions can hold:

```text
Locks
Connections
Resources
```

and prevent other operations from progressing.

Example:

```text
Transaction A
      ↓
Lock Held
      ↓
Transaction B
      ↓
Waiting
```

Monitor long-running transactions and investigate unexpected behavior.

---

# 24. Database Locks

Locks coordinate concurrent database operations.

However, excessive locking can cause performance problems.

Example:

```text
Transaction A
      ↓
Lock
      ↓
Row / Table
      ↑
      │
Transaction B
      ↓
Waiting
```

Monitor:

```text
Lock Count
Lock Wait Time
Blocked Queries
Blocking Queries
```

---

# 25. Lock Contention

Lock contention occurs when multiple transactions compete for the same resources.

Example:

```text
Transaction A ──┐
                ↓
              Row Lock
                ↑
Transaction B ──┘
```

Symptoms:

```text
Query latency ↑
Connections ↑
CPU may remain normal
Application timeout ↑
```

This is an important interview scenario because high latency does not always mean high CPU.

---

# 26. Deadlocks

A deadlock occurs when transactions wait for resources held by each other.

Example:

```text
Transaction A
   ↓
Lock Resource 1
   ↓
Waiting for Resource 2

Transaction B
   ↓
Lock Resource 2
   ↓
Waiting for Resource 1
```

Flow:

```text
A waits for B
B waits for A
```

The database must detect and resolve the deadlock, usually by aborting one transaction.

Monitor deadlock counts and errors.

---

# 27. Database Errors

Monitor:

```text
Connection Errors
Authentication Errors
Query Errors
Transaction Errors
Deadlocks
Timeouts
Replication Errors
Storage Errors
```

Errors should be correlated with:

```text
Application Logs
Database Logs
Metrics
Deployment Events
```

---

# 28. Database Logs

Database logs can reveal:

```text
Connection failures
Authentication failures
Slow queries
Deadlocks
Errors
Checkpoint activity
Replication problems
Startup failures
Shutdown events
```

Centralize logs where possible.

Architecture:

```text
Database
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

# 29. Replication Monitoring

For replicated databases:

```text
Primary
   │
   ├────────→ Replica 1
   │
   └────────→ Replica 2
```

Monitor:

```text
Replication Status
Replication Lag
Replica Health
Replication Errors
Replica Connections
```

---

# 30. Replication Lag

Replication lag means a replica is behind the primary.

Example:

```text
Primary
   ↓
Transaction committed
   ↓
Replica
   ↓
Transaction not yet applied
```

If lag increases:

```text
Replication Lag ↑
      ↓
Replica Data Becomes Stale
      ↓
Read Requests May See Older Data
```

Investigate:

```text
Network
Replica CPU
Replica Disk I/O
Replication throughput
Large transactions
```

---

# 31. Read Replica Monitoring

Read replicas are commonly used to distribute read workloads.

Architecture:

```text
                    Application
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
         Primary                  Read Replica
             │                       │
          Writes                   Reads
```

Monitor:

```text
Replica Health
Replication Lag
Read Traffic
CPU
Memory
Storage
Connections
```

Do not route critical reads to a replica without understanding replication consistency.

---

# 32. Database Backup Monitoring

Backups are part of database reliability.

Monitor:

```text
Backup Success
Backup Failure
Backup Duration
Backup Size
Backup Frequency
Retention
Recovery Point
```

A backup that has never been tested for restoration should not be assumed to be reliable.

---

# 33. Backup and Recovery

Production database monitoring should include recovery validation.

Conceptually:

```text
Database
   ↓
Backup
   ↓
Storage
   ↓
Restore Test
   ↓
Validation
```

Monitor both:

```text
Backup
+
Restore
```

A successful backup does not automatically guarantee successful recovery.

---

# 34. Database Monitoring With Prometheus

A common monitoring architecture is:

```text
Database
    ↓
Database Exporter
    ↓
Prometheus
    ↓
Grafana
```

The exporter exposes database metrics that Prometheus collects.

For example:

```text
Database
   ↓
postgres_exporter / mysql_exporter
   ↓
Prometheus
   ↓
Grafana
```

The exact exporter depends on the database engine.

---

# 35. Database Dashboard

A useful production dashboard can contain:

```text
┌────────────────────────────────────────────┐
│             DATABASE OVERVIEW              │
├────────────────────────────────────────────┤
│ CPU │ Memory │ Storage │ Connections       │
├────────────────────────────────────────────┤
│ Query Rate                                 │
├────────────────────────────────────────────┤
│ Query Latency                              │
├────────────────────────────────────────────┤
│ Slow Queries                               │
├────────────────────────────────────────────┤
│ Transactions                               │
├────────────────────────────────────────────┤
│ Locks / Deadlocks                          │
├────────────────────────────────────────────┤
│ Disk I/O / Disk Latency                    │
├────────────────────────────────────────────┤
│ Replication Lag                            │
├────────────────────────────────────────────┤
│ Errors                                     │
└────────────────────────────────────────────┘
```

---

# 36. Database Alerts

Useful alerts include:

```text
Database unavailable
High CPU
High memory pressure
Low storage
High disk latency
High connection usage
Connection failures
Slow queries
High query latency
Deadlocks
Lock waits
Replication lag
Backup failures
Database errors
```

Alerts should be actionable.

---

# 37. High Connection Alert

Example:

```text
Maximum Connections = 500
Active Connections  = 450
```

Connection usage:

```text
450 / 500 × 100 = 90%
```

At this point investigate:

```text
Traffic
Connection Pool
Slow Queries
Long Transactions
Connection Leaks
Application Instances
```

Do not simply increase the connection limit without understanding why connections are increasing.

---

# 38. High Query Latency Troubleshooting

Scenario:

```text
Query Latency ↑
```

Check:

```text
CPU
Disk I/O
Locks
Query Plan
Indexes
Connection Count
Database Memory
Storage
Recent Application Changes
```

Possible flow:

```text
Missing Index
      ↓
Full Table Scan
      ↓
Disk Reads ↑
      ↓
Query Latency ↑
      ↓
Application Latency ↑
```

---

# 39. High CPU Troubleshooting

Scenario:

```text
Database CPU = 95%
```

Investigate:

```text
Query Rate
Slow Queries
Expensive Queries
Execution Plans
Missing Indexes
Traffic
Background Jobs
Recent Deployment
```

Example:

```text
New Deployment
      ↓
New Query Pattern
      ↓
CPU ↑
      ↓
Query Latency ↑
      ↓
Application Latency ↑
```

---

# 40. High Disk Latency Troubleshooting

Scenario:

```text
Disk Latency ↑
```

Check:

```text
Read IOPS
Write IOPS
Throughput
Queue Depth
Database Queries
Storage Capacity
```

Possible causes:

```text
High query volume
Large scans
Heavy writes
Insufficient storage performance
Background maintenance
```

---

# 41. Database Monitoring During Deployment

Before deployment:

```text
CPU              → Normal
Connections      → Normal
Query Latency    → Normal
Errors           → Normal
Disk I/O         → Normal
```

After deployment:

```text
CPU ↑
Connections ↑
Query Latency ↑
Errors ↑
```

Investigate:

```text
Schema Changes
New Queries
Missing Indexes
Connection Pool Changes
Application Changes
Database Configuration
```

Database monitoring is essential during application deployments.

---

# 42. Database Monitoring During Traffic Spikes

When traffic increases:

```text
Traffic ↑
   ↓
Requests ↑
   ↓
Database Queries ↑
   ↓
Connections ↑
   ↓
CPU / Disk I/O ↑
   ↓
Query Latency ↑
```

Monitor:

```text
CPU
Connections
Query Rate
Query Latency
Disk I/O
Storage
Locks
Errors
```

---

# 43. Database Monitoring and Application Monitoring

Database monitoring should not be isolated.

Architecture:

```text
User
  ↓
Load Balancer
  ↓
Application
  ↓
Database
```

Monitor both:

```text
Application
├── Requests
├── Latency
├── Errors
└── Connections

Database
├── Queries
├── CPU
├── Memory
├── Disk
├── Connections
└── Locks
```

This helps identify whether application latency is caused by the database.

---

# 44. Example: Application Is Slow

Scenario:

```text
Application Latency ↑
```

Check:

```text
Application CPU
Application Memory
Database Latency
Database CPU
Database Connections
Slow Queries
External APIs
```

Suppose:

```text
Application CPU = 30%
Database CPU = 40%
Query Latency = 2 seconds
```

The database query path becomes a strong candidate.

---

# 45. Example: Database Connection Failures

Scenario:

```text
Application
     ↓
Database
     X
Connection Failed
```

Check:

```text
DNS
Network
Database Port
Security Group
Database Availability
Authentication
Connection Limit
Connection Pool
```

Commands:

```bash
nc -vz <database-host> <port>
```

Then use the database-specific health command.

---

# 46. Example: Database Storage Is Full

Scenario:

```text
Database Storage
      ↓
95% Used
```

Investigate:

```text
Database Growth
Large Tables
Logs
Temporary Files
Indexes
Old Data
Backup Files
```

Then determine whether to:

```text
Increase Storage
Archive Data
Remove Unnecessary Data
Optimize Retention
```

Do not delete production database files blindly.

---

# 47. Example: Database Replication Lag

Scenario:

```text
Primary
   ↓
Replica
   ↓
Replication Lag ↑
```

Check:

```text
Primary Write Rate
Replica CPU
Replica Disk I/O
Network
Replication Throughput
Large Transactions
Replica Health
```

Possible flow:

```text
Write Traffic ↑
      ↓
Primary Generates More Changes
      ↓
Replication Queue ↑
      ↓
Replica Lag ↑
```

---

# 48. Example: Deadlock Incident

Scenario:

```text
Application Errors ↑
Database Deadlocks ↑
```

Investigate:

```text
Blocking Transactions
Lock Order
Queries Involved
Transaction Duration
Recent Application Changes
```

Conceptually:

```text
Transaction A
    ↓
Lock A
    ↓
Waiting for B

Transaction B
    ↓
Lock B
    ↓
Waiting for A
```

This creates a deadlock.

---

# 49. Database Monitoring Troubleshooting Framework

Use this sequence during incidents:

```text
1. Is the database available?
        ↓
2. Can the application connect?
        ↓
3. Are connections within limits?
        ↓
4. Is CPU normal?
        ↓
5. Is memory normal?
        ↓
6. Is storage available?
        ↓
7. Is disk latency normal?
        ↓
8. Are queries slow?
        ↓
9. Are locks increasing?
        ↓
10. Are errors increasing?
        ↓
11. Is replication healthy?
        ↓
12. Are backups healthy?
```

This provides a structured troubleshooting approach.

---

# 50. Production Database Monitoring Architecture

```text
                         USERS
                           │
                           ↓
                    LOAD BALANCER
                           │
                           ↓
                      APPLICATION
                           │
                           ↓
                     ┌───────────┐
                     │ DATABASE  │
                     └─────┬─────┘
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
           Metrics        Logs        Traces
             │             │             │
             ↓             ↓             ↓
        DB Exporter       ELK       OpenTelemetry
             │             │             │
             ↓             ↓             ↓
        Prometheus      Kibana        Jaeger
             │
             ↓
          Grafana
             │
             ↓
           Alerts
```

---

# 51. Production Database Monitoring Best Practices

```text
1. Monitor database availability.
2. Monitor CPU.
3. Monitor memory.
4. Monitor storage.
5. Monitor disk I/O.
6. Monitor disk latency.
7. Monitor database connections.
8. Monitor connection pools.
9. Monitor query latency.
10. Monitor query throughput.
11. Monitor slow queries.
12. Monitor transactions.
13. Monitor locks.
14. Monitor deadlocks.
15. Monitor errors.
16. Monitor replication.
17. Monitor replication lag.
18. Monitor backups.
19. Monitor storage growth.
20. Correlate database metrics with application metrics.
21. Centralize database logs.
22. Use Prometheus and Grafana for metrics.
23. Use ELK for centralized logs.
24. Use tracing for distributed database calls.
25. Test backup restoration.
26. Create actionable alerts.
27. Monitor trends instead of only current values.
28. Review database performance after deployments.
29. Monitor connection pool behavior.
30. Investigate root causes before scaling resources.
```

---

# 52. Common Database Monitoring Mistakes

### Mistake 1: Monitoring Only CPU

```text
CPU = 30%
```

does not prove the database is healthy.

The database may have:

```text
High Disk Latency
High Connections
Lock Contention
Slow Queries
Replication Lag
```

---

### Mistake 2: Ignoring Connections

A database can have:

```text
CPU = 40%
```

but:

```text
Connections = 100%
```

and reject new requests.

---

### Mistake 3: Ignoring Query Performance

Increasing database resources may not solve:

```text
Missing Index
Bad Query
Full Table Scan
Inefficient Join
```

---

### Mistake 4: Ignoring Locks

Query latency can increase because queries are waiting for locks even when CPU is normal.

---

### Mistake 5: Ignoring Storage Growth

A database can suddenly become unavailable when storage reaches capacity.

---

### Mistake 6: Ignoring Replication Lag

A replica can be online but still serve stale data.

---

### Mistake 7: Monitoring Backups Without Testing Restore

A backup should be periodically restored and validated.

---

### Mistake 8: Restarting the Database Without Investigation

Restarting can temporarily hide:

```text
Connection leaks
Memory pressure
Slow queries
Lock problems
```

Always investigate the root cause.

---

# 53. Production Database Monitoring Checklist

```text
DATABASE
├── Availability
├── CPU
├── Memory
├── Storage
├── Disk I/O
├── Disk Latency
├── Query Rate
├── Query Latency
├── Slow Queries
├── Connections
├── Connection Pool
├── Transactions
├── Locks
├── Deadlocks
├── Errors
├── Replication
├── Replication Lag
└── Backups

APPLICATION
├── Requests
├── Latency
├── Errors
├── Database Connections
└── Dependency Calls

OBSERVABILITY
├── Database Exporter
├── Prometheus
├── Grafana
├── ELK
├── OpenTelemetry
└── Jaeger
```

---

# 54. Commands Cheat Sheet

## PostgreSQL

Check database readiness:

```bash
pg_isready -h <host> -p 5432
```

Connect:

```bash
psql -h <host> -U <user> -d <database>
```

---

## MySQL

Check database availability:

```bash
mysqladmin ping -h <host> -u <user> -p
```

Connect:

```bash
mysql -h <host> -u <user> -p
```

---

## Network Connectivity

```bash
nc -vz <database-host> <port>
```

---

## Linux Resource Monitoring

```bash
top
free -h
df -h
iostat -xz 1
vmstat 1
```

---

## Linux Connections

```bash
ss -s
ss -ant
```

---

# 55. Interview Question

## How do you monitor databases in production?

**Answer:**

I monitor the database at infrastructure, database-engine, and application levels.

At the infrastructure level I monitor CPU, memory, storage, disk I/O, and disk latency.

At the database level I monitor connections, query latency, slow queries, transactions, locks, deadlocks, errors, replication lag, and backups.

At the application level I monitor database connection pools, application latency, and database-related errors.

For centralized observability, I can use database exporters with Prometheus and Grafana for metrics, ELK for logs, and OpenTelemetry/Jaeger for tracing.

---

# 56. Interview Question

## How would you troubleshoot high database CPU?

**Answer:**

I would not immediately increase the database instance size.

First I would check:

```text
Query Rate
Slow Queries
Expensive Queries
Execution Plans
Indexes
Connections
Traffic
Recent Deployment
Background Jobs
```

Then I would identify which queries are consuming CPU.

If the workload is legitimate and resources are insufficient, I would consider scaling the database after understanding the workload.

---

# 57. Interview Question

## How would you troubleshoot database connection failures?

**Answer:**

I would troubleshoot from the network layer to the database layer.

First I would check:

```text
DNS
Network Connectivity
Route
Security Group
NACL
Database Port
Database Availability
Authentication
Connection Limit
Connection Pool
```

I would use:

```bash
nc -vz <host> <port>
```

and then use the database-specific health check.

---

# 58. Interview Question

## What would you do if database connections reach 100%?

**Answer:**

I would first determine why connections increased.

I would check:

```text
Application Traffic
Connection Pool Size
Active Connections
Idle Connections
Long-Running Queries
Long Transactions
Connection Leaks
Number of Application Instances
```

I would not blindly increase the database connection limit because the underlying issue could be an application connection leak or slow queries.

---

# 59. Interview Question

## How would you troubleshoot slow database queries?

**Answer:**

I would identify the slow query and investigate:

```text
Execution Plan
Indexes
Rows Scanned
Rows Returned
Joins
Sorting
Filtering
Locks
Database CPU
Disk I/O
```

I would determine whether the query is inefficient or whether the database infrastructure is under pressure.

---

# 60. Interview Question

## How would you troubleshoot database replication lag?

**Answer:**

I would check:

```text
Primary Write Rate
Replica CPU
Replica Disk I/O
Network
Replication Queue
Large Transactions
Replica Health
```

Then I would determine whether the replica cannot process changes fast enough or whether network/replication communication is delayed.

---

# 61. Interview Question

## What is database lock contention?

**Answer:**

Lock contention occurs when multiple transactions compete for the same database resources.

For example:

```text
Transaction A
      ↓
Holds Lock
      ↓
Transaction B
      ↓
Waits
```

This can increase query latency even when CPU utilization is normal.

I would monitor:

```text
Lock Wait Time
Blocked Queries
Blocking Queries
Long Transactions
```

---

# 62. Interview Question

## What is a database deadlock?

**Answer:**

A deadlock occurs when two or more transactions wait for resources held by each other.

Example:

```text
Transaction A
   ↓
Lock A
   ↓
Waiting for Lock B

Transaction B
   ↓
Lock B
   ↓
Waiting for Lock A
```

The database detects the deadlock and normally aborts one transaction.

I would investigate the queries and transaction order involved.

---

# 63. Interview Question

## How would you troubleshoot a database that is running out of storage?

**Answer:**

I would first determine what is consuming storage.

I would investigate:

```text
Large Tables
Indexes
Database Logs
Temporary Files
Old Data
Storage Growth
Backup Files
```

Then I would decide whether the correct action is:

```text
Increase Storage
Archive Data
Optimize Retention
Remove Unnecessary Data
```

I would avoid deleting database files manually.

---

# 64. Interview Question

## How do you correlate application latency with database performance?

**Answer:**

I compare application metrics with database metrics over the same time period.

For example:

```text
Application Latency ↑
        ↓
Database Query Latency ↑
        ↓
Database Disk Latency ↑
```

This indicates that database performance may be contributing to application latency.

I would then use logs and distributed tracing to identify the exact database operation responsible.

---

# 65. Production Incident Example

## Scenario

Users report:

```text
Application is very slow.
```

Application monitoring shows:

```text
Request Latency ↑
```

Database monitoring shows:

```text
Query Latency ↑
Disk Latency ↑
```

Further investigation:

```text
Large Query
    ↓
Full Table Scan
    ↓
Disk Reads ↑
    ↓
Query Latency ↑
    ↓
Application Latency ↑
```

Root cause:

```text
Inefficient query / missing index
```

The correct solution may be query or index optimization rather than simply increasing server size.

---

# 66. Production Incident Example

## Scenario

Applications suddenly report:

```text
Database connection failures
```

Investigation:

```text
Application
     ↓
Connection Pool
     ↓
Database
```

Metrics show:

```text
Database CPU = Normal
Database Memory = Normal
Connections = 100%
```

Further investigation shows:

```text
Application Deployment
      ↓
Connection Pool Size Increased
      ↓
More Connections
      ↓
Database Connection Limit Reached
      ↓
New Connections Fail
```

Root cause:

```text
Application connection-pool configuration
```

---

# 67. Production Incident Example

## Scenario

Read requests return stale data.

Architecture:

```text
Application
     │
     ├──── Write → Primary
     │
     └──── Read  → Replica
```

Monitoring shows:

```text
Replication Lag ↑
```

Therefore:

```text
Primary
   ↓
New Data
   ↓
Replication Delay
   ↓
Replica
   ↓
Stale Read
```

Root cause:

```text
Replication lag
```

The solution depends on the application's consistency requirements.

---

# 68. Production Incident Example

## Scenario

Database CPU suddenly reaches:

```text
95%
```

Investigation:

```text
CPU ↑
   ↓
Query Rate ↑
   ↓
New Deployment
   ↓
New Query Pattern
   ↓
Expensive Query
```

The correct response is to identify the expensive query and determine whether:

```text
Index
Query Optimization
Connection Pool
Application Change
Scaling
```

is the appropriate solution.

---

# 69. Database Monitoring Mental Model

```text
                           APPLICATION
                                │
                                ↓
                         CONNECTION POOL
                                │
                                ↓
                           DATABASE
                                │
        ┌───────────────┬───────┼────────┬───────────────┐
        ↓               ↓       ↓        ↓               ↓
      CPU            Memory   Storage   Queries       Connections
        │               │       │        │               │
        └───────────────┴───────┼────────┴───────────────┘
                                ↓
                         DATABASE HEALTH
                                │
                  ┌─────────────┼─────────────┐
                  ↓             ↓             ↓
                Locks        Replication    Backups
                  │             │             │
                  └─────────────┼─────────────┘
                                ↓
                         OBSERVABILITY
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
           Metrics             Logs             Traces
              ↓                 ↓                 ↓
         Prometheus             ELK        OpenTelemetry
              ↓                 ↓                 ↓
           Grafana            Kibana          Jaeger
              │
              ↓
            Alerts
              │
              ↓
         Root Cause
              │
              ↓
          Remediation
              │
              ↓
       Validate Recovery
```

---

# 70. Final Key Takeaway

Database monitoring is not simply checking whether the database process is running.

A production DevOps engineer should monitor the complete database path:

```text
Application
   ↓
Connection Pool
   ↓
Network
   ↓
Database
   ↓
Query
   ↓
Storage
```

And monitor:

```text
CPU
+
Memory
+
Storage
+
Disk I/O
+
Connections
+
Queries
+
Locks
+
Replication
+
Backups
```

The most important principle is:

```text
Database Metrics
+
Application Metrics
+
Logs
+
Traces
=
Faster Database Root Cause Analysis
```