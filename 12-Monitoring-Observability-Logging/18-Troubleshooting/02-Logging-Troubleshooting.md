# 02 - Logging Troubleshooting

> Production Logging Troubleshooting — Application Logs, Kubernetes Logs, ELK, Elasticsearch, Logstash, Kibana, Structured Logging, Log Ingestion, Parsing, Indexing, Storage, Retention, Performance, Security, Missing Logs, Duplicate Logs, Alert Correlation, Incident Response, Root Cause Analysis and Interview Preparation

---

# 1. Logging Troubleshooting Fundamentals

Logging troubleshooting is the process of identifying why logs are:

- Missing
- Delayed
- Duplicated
- Misformatted
- Incorrectly parsed
- Not searchable
- Not indexed
- Consuming excessive storage
- Causing application performance problems
- Exposing sensitive information
- Not reaching the centralized logging platform

A production logging pipeline commonly looks like:

    Application
        |
        v
    stdout / log file
        |
        v
    Log Collector
        |
        v
    Logstash / Pipeline
        |
        v
    Elasticsearch
        |
        v
      Kibana
        |
        v
    Engineer

When troubleshooting, follow the log from source to destination.

---

# 2. The Logging Troubleshooting Mindset

Do not start with Kibana just because the user cannot find a log.

The actual problem may be:

    Application
       |
       X
    No log generated

or:

    Application
       |
       v
    Collector
       |
       X
    Log dropped

or:

    Logstash
       |
       X
    Elasticsearch

or:

    Elasticsearch
       |
       v
    Log exists
       |
       X
    Kibana query/filter

Always determine:

> Where is the last place I can prove the log exists?

---

# 3. Logging Data Path

For Kubernetes:

    Application Pod
          |
          v
      stdout/stderr
          |
          v
    Container Runtime
          |
          v
    Node Log Files
          |
          v
    Log Collector
          |
          v
      Log Pipeline
          |
          v
     Elasticsearch
          |
          v
        Kibana

Depending on architecture, logs may be collected through:

- DaemonSet collectors
- Sidecars
- Node-level agents
- Direct application output
- Logstash
- Other ingestion systems

The exact architecture matters during troubleshooting.

---

# 4. First Question — Is the Application Generating the Log?

Suppose a developer says:

> "The payment error is not visible in Kibana."

First verify the application.

Check:

    Does the application produce the log?

If Kubernetes:

    kubectl logs <pod> -n <namespace>

If the log is not present there, the problem is before centralized logging.

Possible causes:

- Log statement not executed
- Wrong log level
- Application failure before logging
- Logging configuration
- Container writing somewhere unexpected
- Application logs suppressed

---

# 5. Check Current Pod Logs

Basic command:

    kubectl logs <pod> -n <namespace>

For multiple containers:

    kubectl logs <pod> -c <container> -n <namespace>

For previous crashed container:

    kubectl logs <pod> --previous -n <namespace>

Follow logs:

    kubectl logs -f <pod> -n <namespace>

Recent lines:

    kubectl logs --tail=100 <pod> -n <namespace>

Since a specific time:

    kubectl logs --since=1h <pod> -n <namespace>

Use the command that answers the investigation question.

---

# 6. Previous Container Logs

A common production problem:

    Pod crashes
       |
       v
    Container restarts
       |
       v
    Engineer checks current logs
       |
       v
    "Nothing useful"

The previous container may contain the actual failure.

Use:

    kubectl logs <pod> --previous -n <namespace>

This is especially important for:

- CrashLoopBackOff
- OOMKilled
- Application startup failures
- Dependency connection failures

---

# 7. Multiple Containers in a Pod

A pod may contain:

    Application Container
    Sidecar Container

If you run:

    kubectl logs <pod>

you may not be looking at the container you need.

Check:

    kubectl get pod <pod> -o jsonpath='{.spec.containers[*].name}'

Then:

    kubectl logs <pod> -c <container>

Always identify the correct container before concluding that logs are missing.

---

# 8. Init Container Logs

Init containers can fail before the application starts.

Check:

    kubectl logs <pod> -c <init-container>

Possible issues:

- Configuration generation
- Secret retrieval
- Database migration
- Permission setup
- Dependency checks

If the main container never starts, application logs may not exist.

---

# 9. stdout vs File Logging

Modern Kubernetes applications commonly log to:

    stdout
    stderr

The container runtime captures these logs.

Another application may write to:

    /var/log/application.log

These architectures require different collection strategies.

If the collector expects stdout but the application writes only to a file:

    Application
       |
       v
    /var/log/app.log

    Collector
       |
       X
    Does not see logs

The logging architecture must match the application behavior.

---

# 10. Why stdout Logging Is Common in Kubernetes

stdout/stderr logging works well with container orchestration because:

- Kubernetes can capture container output
- Collectors can read container logs
- Applications remain decoupled from host filesystem paths
- Containers remain easier to replace

Typical pattern:

    Application
        |
        v
    stdout
        |
        v
    Kubernetes runtime
        |
        v
    Collector

Avoid assuming every application automatically follows this pattern.

---

# 11. Application Log Level Troubleshooting

Suppose:

    ERROR logs appear
    DEBUG logs do not

Possible cause:

    Production log level = INFO

Then:

    DEBUG < INFO

so DEBUG entries are intentionally suppressed.

Check:

- Application configuration
- Environment variables
- ConfigMaps
- Logging framework configuration
- Runtime configuration

Do not immediately assume the collector is dropping logs.

---

# 12. Log Level Hierarchy

Common levels:

    TRACE
    DEBUG
    INFO
    WARN
    ERROR
    FATAL

If configured at:

    INFO

the application may emit:

    INFO
    WARN
    ERROR

but not:

    DEBUG
    TRACE

If logs appear inconsistent, confirm the configured level.

---

# 13. Structured Logging

Preferred production format:

    {
      "timestamp": "2026-08-15T10:30:00Z",
      "level": "ERROR",
      "service": "orders",
      "environment": "prod",
      "message": "Payment request failed",
      "error_code": "PAYMENT_TIMEOUT"
    }

Benefits:

- Predictable parsing
- Easier filtering
- Better aggregation
- Better dashboards
- Better alerting
- Better machine processing

Troubleshooting becomes harder when every application uses a different unstructured format.

---

# 14. Unstructured Log Problem

Example:

    2026-08-15 ERROR payment failed

Another service:

    ERROR | payment-service | timeout

Another:

    Payment failure at 10:30

A centralized system must parse all formats.

This increases:

- Grok complexity
- Parsing errors
- Maintenance
- Query complexity

Standardize logging formats where possible.

---

# 15. Timestamp Troubleshooting

Every production log should have a reliable timestamp.

Problems:

- Wrong timezone
- Application clock drift
- Collector timestamp replacing application timestamp
- Incorrect timestamp format
- Parsing failure

Example:

    Application says:
    10:30

    Elasticsearch says:
    04:30

This may simply be a timezone interpretation issue.

Always establish the timestamp standard used by the platform.

---

# 16. UTC Standardization

A common production practice is:

    Store timestamps in UTC.

Then dashboards can display them according to the user's timezone.

Benefits:

- Easier incident correlation
- Consistent cross-region operations
- Easier distributed system analysis
- Reduced timezone confusion

When investigating incidents, normalize timestamps.

---

# 17. Logging Incident Scope

First determine:

    One log missing?
    One pod?
    One service?
    One namespace?
    One node?
    Entire cluster?
    Entire logging platform?

Examples:

    One pod missing logs
        |
        v
    Application / container issue

    All pods on one node missing logs
        |
        v
    Collector/node issue

    All applications missing logs
        |
        v
    Central pipeline issue

Scope quickly narrows the search.

---

# 18. Logging Pipeline Troubleshooting Layers

Use this model:

    1. Application
    2. Container
    3. Node
    4. Collector
    5. Logstash
    6. Elasticsearch
    7. Kibana
    8. User Query

If the log is missing, identify the first broken layer.

---

# 19. Kubernetes Log File Architecture

Depending on runtime/configuration, container logs are represented through node-level log files.

Typical concept:

    Pod
      |
      v
    Container Runtime
      |
      v
    Node Log Files
      |
      v
    Log Collector

If a node has log problems:

- Disk may fill
- Collector may stop
- Files may rotate
- Logs may be lost depending on architecture

Node-level logging is therefore part of production infrastructure monitoring.

---

# 20. Log Collector as DaemonSet

A common architecture:

    Node-A -> Collector
    Node-B -> Collector
    Node-C -> Collector

The collector runs on each node and gathers local logs.

Advantages:

- Node-local collection
- Centralized forwarding
- Automatic coverage for new nodes

Troubleshoot:

    DaemonSet
    Pods
    Volumes
    Permissions
    Network
    Output destination

---

# 21. Collector DaemonSet Health

Check:

    kubectl get daemonset -n logging

Expected:

    DESIRED = CURRENT = READY

Then:

    kubectl get pods -n logging -o wide

Look for:

- Missing collector on a node
- CrashLoopBackOff
- Pending
- OOMKilled
- ImagePullBackOff
- Mount failures

If one node has no collector:

    Logs from that node may be missing.

---

# 22. Collector Volume Mount Problems

A node-level collector usually needs access to log locations.

If the volume is not mounted:

    Node Logs
       |
       X
    Collector

Check:

    kubectl describe pod <collector-pod> -n logging

Look for:

- FailedMount
- Permission denied
- Wrong hostPath
- Read-only filesystem

A collector can be Running while still failing to access the expected logs.

---

# 23. Collector Permission Problems

Possible causes:

- SecurityContext
- File permissions
- Host filesystem restrictions
- SELinux/AppArmor where applicable
- Incorrect user

Symptoms:

    Permission denied

The fix should follow least privilege.

Do not run the collector as privileged unless the architecture genuinely requires it.

---

# 24. Collector Cannot Reach Logstash

Flow:

    Collector
       |
       X
    Network
       |
       v
    Logstash

Check:

- DNS
- Port
- Service
- Endpoint
- NetworkPolicy
- Security Group
- Logstash listener

Test connectivity from the collector's network context.

---

# 25. Logstash Input Troubleshooting

Logstash may receive logs through:

- Beats
- TCP
- UDP
- HTTP
- Other supported inputs

Flow:

    Collector
       |
       v
    Logstash Input
       |
       v
    Filter
       |
       v
    Output

If no events enter Logstash, filters and Elasticsearch are not the first problem.

Check the input listener first.

---

# 26. Verify Logstash Listening Ports

On a host where appropriate:

    ss -lntp

or:

    netstat -lntp

Check the expected Logstash port.

If the port is not listening:

- Pipeline failed
- Configuration error
- Plugin issue
- Process startup issue

If it is listening but clients cannot connect:

- Network
- Firewall
- Security Group
- Service routing

---

# 27. Logstash Pipeline Configuration

A pipeline commonly contains:

    input {
      ...
    }

    filter {
      ...
    }

    output {
      ...
    }

Troubleshooting should isolate each stage.

Conceptually:

    Input
      |
      v
    Filter
      |
      v
    Output

If events enter but never reach Elasticsearch:

    Filter/output

is the next area to investigate.

---

# 28. Logstash Configuration Syntax Errors

A bad pipeline configuration can prevent Logstash from starting or cause a pipeline failure.

Check:

    kubectl logs <logstash-pod> -n logging

or system logs if installed directly on a host.

Look for:

- Syntax errors
- Plugin errors
- Invalid configuration
- Missing plugin
- Authentication failures

Validate configuration before production deployment.

---

# 29. Logstash Multiple Pipelines

Production Logstash may have:

    Pipeline-A -> Application logs
    Pipeline-B -> Security logs
    Pipeline-C -> Audit logs

A problem in one pipeline may not affect the others.

Identify the pipeline responsible for the missing data.

Do not restart every pipeline unnecessarily.

---

# 30. Logstash Persistent Queue

A persistent queue can buffer events when Elasticsearch is temporarily unavailable.

Architecture:

    Collector
        |
        v
    Logstash
        |
        v
    Persistent Queue
        |
        v
    Elasticsearch

If Elasticsearch goes down:

    Logstash
       |
       v
    Queue grows

After Elasticsearch recovers:

    Queue drains

Monitor queue size and disk capacity.

---

# 31. Logstash Queue Growth

Symptoms:

- Increasing queue size
- Increasing disk usage
- Delayed logs
- Elasticsearch output failures

Possible causes:

- Elasticsearch unavailable
- Elasticsearch slow
- Network latency
- Indexing rejection
- Log volume spike

A queue is a buffer, not an unlimited storage system.

---

# 32. Logstash Backpressure

Backpressure occurs when downstream processing cannot keep up with incoming events.

Example:

    Incoming Logs
         |
         v
       20k/s
         |
         v
      Logstash
         |
         v
      10k/s
         |
         v
    Elasticsearch

Queue grows.

Investigate:

- Event rate
- Filter cost
- Worker count
- Batch size
- Output performance
- Elasticsearch indexing capacity

---

# 33. Logstash CPU Saturation

Possible causes:

- Complex Grok
- Expensive filters
- Large JSON parsing
- High event volume
- Excessive enrichment

Check:

    CPU
    Memory
    Pipeline throughput
    Queue size

Optimization should target the expensive processing stage.

---

# 34. Logstash Memory Problems

Symptoms:

- OOMKilled
- Restarting
- Slow processing
- Queue behavior changes

Possible causes:

- High event volume
- Large messages
- Complex filters
- Insufficient heap

Check:

    kubectl describe pod <logstash-pod> -n logging

and:

    kubectl logs <logstash-pod> -n logging --previous

Do not only increase memory without identifying the workload driver.

---

# 35. Grok Parsing Troubleshooting

Suppose raw log:

    2026-08-15 10:30:00 ERROR payment failed

Grok pattern may extract:

    timestamp
    level
    message

If the pattern does not match:

- Fields may be missing
- Parsing may fail
- Original message may still exist
- Kibana filters may not work

Test patterns before production deployment.

---

# 36. Grok Failure

Common causes:

- Log format changed
- Regex incorrect
- Unexpected characters
- Optional fields not handled
- Multiple formats mixed together

Example:

Old:

    ERROR payment failed

New:

    ERROR payment failed for transaction 123

A strict parser may stop matching.

Design parsing rules to tolerate controlled variation.

---

# 37. JSON Parsing Troubleshooting

If application logs JSON:

    {"level":"ERROR","service":"orders"}

Logstash should parse JSON appropriately.

Possible issues:

- Invalid JSON
- Escaped JSON
- Nested JSON string
- Multiline JSON
- Incorrect field mapping

Always inspect the raw event before assuming Elasticsearch mapping is the problem.

---

# 38. Multiline Log Troubleshooting

Stack traces are a common example:

    Exception
      at line 1
      at line 2
      at line 3

If each line is treated as a separate event:

    Event 1 = Exception
    Event 2 = at line 1
    Event 3 = at line 2

This makes troubleshooting difficult.

The collector or ingestion pipeline should combine multiline records appropriately.

---

# 39. Java Stack Trace Problem

Java exception:

    java.lang.NullPointerException
        at com.example.OrderService.process(...)
        at com.example.OrderController.create(...)
        ...

If parsed as separate events, Kibana search becomes fragmented.

Correct design:

    One exception
       |
       v
    One log event
       |
       v
    Full stack trace field

This is important for production troubleshooting.

---

# 40. Log Parsing vs Application Logging

Do not use Logstash to fix every logging problem.

If application output is inconsistent:

    Application
       |
       v
    Bad format

it may be better to fix the application logger rather than building increasingly complex parsing rules.

Prefer:

    Standardized application logs

over:

    Extremely complex ingestion parsing.

---

# 41. Field Extraction Problem

Suppose Kibana cannot filter:

    service=orders

because the field was never extracted.

Raw log:

    service=orders level=ERROR

but Elasticsearch contains only:

    message="service=orders level=ERROR"

Then:

    service field = missing

The issue may be parsing.

---

# 42. Elasticsearch Indexing Troubleshooting

Flow:

    Logstash
       |
       v
    Elasticsearch
       |
       v
    Index

If Logstash receives events but Elasticsearch has no documents:

Check:

- Output configuration
- Elasticsearch connectivity
- Authentication
- TLS
- Index permissions
- Index health
- Rejections
- Disk watermarks

---

# 43. Elasticsearch Connectivity Failure

Symptoms:

- Logstash output errors
- Queue growth
- Missing logs
- Delayed ingestion

Possible causes:

- DNS
- Network
- Port
- TLS
- Credentials
- Elasticsearch unavailable

Follow:

    Logstash
       |
       v
    Network
       |
       v
    Elasticsearch

---

# 44. Elasticsearch Authentication Failure

Common errors:

    401 Unauthorized
    403 Forbidden

Possible causes:

- Wrong username/password
- Expired credentials
- Wrong API key
- Missing index permissions
- Security configuration change

Do not disable Elasticsearch security to solve an authentication issue.

Fix the credentials and permissions properly.

---

# 45. Elasticsearch TLS Failure

Possible causes:

- Certificate expired
- CA mismatch
- Hostname mismatch
- TLS protocol mismatch
- Certificate rotation problem

Check:

- Certificate validity
- Trust chain
- Hostname
- Logstash SSL settings
- Elasticsearch SSL settings

---

# 46. Elasticsearch Index Not Receiving Documents

If Logstash reports successful processing but Kibana shows nothing:

Check Elasticsearch directly.

Questions:

    Does the index exist?

    Is document count increasing?

    Is the correct index pattern used?

    Is the timestamp field correct?

    Is Kibana filtering the data?

Separate:

    Ingestion problem

from:

    Search problem

---

# 47. Index Naming Troubleshooting

Example:

Expected:

    logs-prod-*

Actual:

    application-logs-*

Kibana data view:

    logs-prod-*

Result:

    No logs

The data exists but Kibana is looking at the wrong index pattern.

Always verify actual index names.

---

# 48. Elasticsearch Index Template Problems

Templates can define:

- Mappings
- Settings
- Aliases

If a template is wrong:

- Fields may have wrong types
- Search may fail
- Sorting may behave unexpectedly
- Storage may increase

Changes to templates should be reviewed carefully.

---

# 49. Mapping Conflict

Example:

Day 1:

    response_time = 200

Day 2:

    response_time = "200ms"

Elasticsearch may encounter a type conflict depending on mapping.

Common result:

    Failed indexing

This is why field types should be standardized.

---

# 50. Dynamic Mapping Problems

Automatic mapping can be convenient but risky.

Example:

    status = "200"

may become a keyword/string.

Another application may send:

    status = 200

as a number.

Inconsistent schemas can create mapping problems.

Prefer controlled mappings for important production fields.

---

# 51. Elasticsearch Rejected Documents

Documents can be rejected because of:

- Mapping conflicts
- Invalid requests
- Cluster pressure
- Disk watermarks
- Too many requests
- Authentication
- Index blocks

Always inspect the exact Elasticsearch error.

Do not assume:

    "Logstash is running"

means:

    "All logs are indexed."

---

# 52. Elasticsearch Disk Watermark Problem

When disk usage becomes high, Elasticsearch protects itself.

Possible consequences:

- Shard allocation problems
- Index blocks
- Write failures
- Cluster health degradation

Check:

    Disk usage
    Cluster health
    Index health
    Allocation

For production systems, disk capacity must be monitored before watermarks are reached.

---

# 53. Elasticsearch Cluster Health

Typical health states:

    GREEN
    YELLOW
    RED

Conceptually:

    GREEN
      |
      v
    Healthy

    YELLOW
      |
      v
    Some replica issue

    RED
      |
      v
    Primary shard problem

A logging pipeline can still appear partially functional while cluster health is degraded.

---

# 54. Elasticsearch Shard Problems

If shards are unassigned:

Investigate:

- Node availability
- Disk watermarks
- Allocation rules
- Replica configuration
- Resource pressure

A single node failure can expose poor shard distribution.

---

# 55. Too Many Elasticsearch Shards

Symptoms:

- High heap
- Slow cluster operations
- Long recovery
- Increased overhead

Common cause:

    Too many small indices

Avoid creating excessive shards just because data is increasing.

Shard design should be based on:

- Data volume
- Query requirements
- Node capacity
- Retention

---

# 56. Elasticsearch Large Shards

Large shards can cause:

- Slow recovery
- Long relocation
- Increased operational risk

The correct shard size depends on architecture and workload.

Do not use a fixed shard-size number as a universal rule.

Measure actual cluster behavior.

---

# 57. Elasticsearch JVM / Heap Troubleshooting

Symptoms:

- High heap usage
- Long garbage collection
- Slow queries
- Node instability

Investigate:

- Heap pressure
- Query load
- Shard count
- Field data
- Aggregations
- Cluster size

Do not allocate all machine memory to JVM heap.

The OS also needs memory.

---

# 58. Elasticsearch Search Performance

If Kibana searches are slow:

Check:

    Query
       |
       v
    Index size
       |
       v
    Shards
       |
       v
    Cluster resources
       |
       v
    Disk
       |
       v
    Heap

Possible fixes:

- Narrow time range
- Improve query
- Use appropriate fields
- Optimize index strategy
- Review shard design
- Scale resources

---

# 59. Kibana Troubleshooting

If Kibana is unavailable:

    kubectl get pods -n logging

Then:

    kubectl logs <kibana-pod> -n logging

Check:

- Kibana process
- Elasticsearch connectivity
- Configuration
- Authentication
- TLS
- Memory
- Network

If Kibana is running but dashboards are empty, investigate the data source/index pattern instead.

---

# 60. Kibana Data View Problem

Kibana data views determine which indices are searched.

Example:

    Actual:
    logs-prod-2026.08.15

Data view:

    app-prod-*

Result:

    No results

Verify:

- Data view pattern
- Time field
- Index existence
- User permissions

---

# 61. Kibana Time Field Problem

If logs exist but time-based searches return nothing:

Possible cause:

- Wrong timestamp field
- Timestamp not parsed
- Incorrect timezone
- Invalid field type

Check the actual Elasticsearch document.

---

# 62. Kibana Query Troubleshooting

Start simple:

    service: orders

Then:

    service: orders AND level: ERROR

Then:

    service: orders AND level: ERROR
    AND environment: prod

Build queries incrementally.

If the simple query returns data but the complex query does not, identify which filter removes the results.

---

# 63. Kibana Permission Problem

A user may not see logs because of authorization.

Possible causes:

- Index permissions
- Role mapping
- Namespace/data access controls
- Security configuration

Distinguish:

    No data exists

from:

    User cannot access the data

---

# 64. Logs Missing for One Node

Situation:

    Node-A -> Logs OK
    Node-B -> Logs missing
    Node-C -> Logs OK

Strong suspicion:

    Node-B collector

Check:

    Collector Pod on Node-B
       |
       v
    Volume Mount
       |
       v
    Permissions
       |
       v
    Network
       |
       v
    Output

Do not troubleshoot the entire Elasticsearch cluster first.

---

# 65. Logs Missing for All Pods on One Node

This is a strong node/collector signal.

Possible causes:

- Collector failed
- Node disk problem
- Log directory inaccessible
- Collector volume mount issue
- Node network issue
- Node pressure

Compare the healthy node and affected node.

---

# 66. Logs Missing for One Application

If all other services are logging:

    Collector = likely healthy

Focus on:

- Application
- Pod
- Container
- Log level
- Log format
- Namespace filtering
- Collector filtering rules

This is a localized investigation.

---

# 67. Logs Missing After Pod Restart

Check:

    Current logs
    Previous logs
    Collector state
    Container runtime
    Pod lifecycle

Potential causes:

- Short-lived startup log
- Collector timing
- Log rotation
- Application logging change

Do not assume Kubernetes lost the logs without evidence.

---

# 68. Duplicate Logs

Symptoms:

    Same event appears multiple times.

Possible causes:

- Two collectors reading the same file
- Duplicate pipeline input
- Log forwarding loop
- Application writes duplicate events
- Retry behavior

Architecture:

    Application
       |
       +---- Collector A
       |
       +---- Collector B
              |
              v
          Duplicate

Check collection configuration.

---

# 69. Duplicate Logs After Deployment

A common scenario:

    Old collector
        +
    New collector

both read the same source.

Or:

    stdout
       +
    file collector

both capture the same application event.

After deployment, compare collector configuration and active agents.

---

# 70. Missing Logs Due to Filtering

Log collectors may intentionally drop records.

Examples:

- Namespace filter
- Container filter
- Level filter
- Regex filter
- Metadata filter

If logs disappear after a configuration change:

    Check filtering rules.

Be especially careful with broad drop patterns.

---

# 71. Logstash Conditional Drop

A pipeline might contain logic conceptually like:

    if condition {
       drop {}
    }

If the condition unexpectedly matches production logs:

    Event
      |
      v
    Filter
      |
      X
    Dropped

When logs disappear, inspect all drop conditions.

---

# 72. Sampling and Log Loss

Some systems intentionally sample logs.

Example:

    100,000 events
         |
         v
      Sampling
         |
         v
      10,000 events

This may be appropriate for high-volume debug/trace data.

But sampling must not silently remove:

- Security events
- Audit records
- Critical errors
- Compliance-required logs

---

# 73. Log Volume Spike

Symptoms:

- Elasticsearch CPU high
- Disk grows rapidly
- Logstash queue grows
- Kibana becomes slow

Possible causes:

- Application loop
- Exception storm
- DEBUG enabled
- Traffic spike
- Retry storm

Example:

    Application bug
        |
        v
    Exception loop
        |
        v
    100x log volume
        |
        v
    Logging platform overload

The application may be the root cause.

---

# 74. Logging Platform Backpressure

Architecture:

    Applications
         |
         v
      Collector
         |
         v
      Logstash
         |
         v
    Elasticsearch

If Elasticsearch slows:

    Collector
       |
       v
    Logstash queue ↑
       |
       v
    Disk ↑
       |
       v
    Potential collector backpressure

Monitor the entire pipeline.

---

# 75. Log Loss vs Log Delay

These are different.

Log loss:

    Event never arrives.

Log delay:

    Event arrives later.

Example:

    Application
       |
       v
    Logstash queue
       |
       v
    Elasticsearch

A growing queue causes delay, not necessarily loss.

This distinction matters during incident analysis.

---

# 76. Detecting Log Delay

Compare:

    Application timestamp

with:

    Ingestion timestamp

Example:

    Event created:
    10:00:00

    Indexed:
    10:05:00

Ingestion delay:

    5 minutes

Track this where operationally important.

---

# 77. Log Rotation Problems

If applications write files:

    /var/log/app.log

rotation may create:

    app.log
    app.log.1
    app.log.2

Collector behavior must correctly handle:

- File rotation
- Renames
- New files
- Truncation

Incorrect handling can cause:

- Duplicate logs
- Missing logs
- File descriptor problems

---

# 78. File Descriptor Problems

A collector can run into too many open files.

Symptoms:

    Too many open files

Possible causes:

- Huge number of files
- Log rotation
- File handle leak
- Too many watched files

Check:

    ulimit -n

and process/file descriptor usage where applicable.

---

# 79. Disk Full on Logging Node

If node disk reaches 100%:

    Logs
      |
      v
    Disk Full
      |
      +-- Application problems
      +-- Collector problems
      +-- Container runtime problems

Check:

    df -h

Then:

    du -sh

Find the largest consumers.

Do not delete logs blindly if they are required for incident or compliance purposes.

---

# 80. Inodes Exhausted

A filesystem can run out of inodes before disk bytes are full.

Symptoms:

    Disk usage appears reasonable
    But new files cannot be created

Check:

    df -i

This is particularly relevant when applications create many small log files.

---

# 81. Log Compression

Compression reduces storage usage.

Benefits:

- Lower disk cost
- Lower storage footprint

Tradeoff:

- CPU usage
- Processing complexity

Use compression where appropriate without creating unacceptable ingestion latency.

---

# 82. Log Retention Troubleshooting

If logs disappear earlier than expected, check:

- Index lifecycle policy
- Retention settings
- Deletion jobs
- Storage tiering
- Index naming
- Time-based policies

Do not assume:

    "Elasticsearch deleted my logs"

without checking lifecycle configuration.

---

# 83. Elasticsearch ILM / Retention

Index lifecycle management can automate:

    Hot
      |
      v
    Warm
      |
      v
    Cold
      |
      v
    Delete

If retention is incorrect, logs may be deleted too early or stored too long.

Review lifecycle policies as part of logging troubleshooting.

---

# 84. Log Storage Cost Incident

Situation:

    Elasticsearch storage grows rapidly.

Investigate:

- Daily log volume
- Retention
- Duplicate events
- DEBUG logging
- Large stack traces
- High-cardinality metadata
- Index/shard strategy

Do not solve only by increasing storage.

Find the volume driver.

---

# 85. Large Log Message Problem

Some applications may emit enormous payloads.

Examples:

- Full HTTP request body
- Large response body
- Serialized objects
- Stack traces repeated
- Debug dumps

Consequences:

- Network traffic
- CPU
- Elasticsearch storage
- Query performance

Prefer concise structured logs.

---

# 86. Sensitive Data in Logs

Never intentionally log:

- Passwords
- API keys
- Access tokens
- Private keys
- Session secrets
- Sensitive personal data

If discovered:

    1. Stop further exposure.
    2. Determine scope.
    3. Remove from logging.
    4. Rotate compromised credentials if necessary.
    5. Preserve appropriate security evidence.
    6. Review access.
    7. Improve redaction.

Logging incidents can become security incidents.

---

# 87. Log Redaction

Sensitive values should be redacted before storage when possible.

Example:

    Authorization: Bearer <redacted>

instead of:

    Authorization: Bearer eyJ...

Redaction should preferably happen as close to the source as practical.

Do not rely solely on Kibana hiding sensitive fields if the raw data is already stored.

---

# 88. Access Control for Logs

Logs may contain:

- User information
- Internal architecture
- Error details
- Security events

Use:

- RBAC
- Least privilege
- Index-level access
- Network restrictions
- Authentication
- Encryption

Not every developer needs unrestricted access to all production logs.

---

# 89. Audit Logging

Audit logs should be treated differently from normal application logs.

Audit events may require:

- Strong retention
- Restricted access
- Integrity controls
- Reliable timestamps
- Controlled deletion

Examples:

    User permission change
    Authentication event
    Administrative action

Do not apply aggressive retention policies to audit logs without checking requirements.

---

# 90. Correlation IDs

A correlation ID helps follow a request across services.

Example:

    User Request
        |
        v
    API Gateway / ALB
        |
        v
    Orders
        |
        v
    Payment
        |
        v
    Notification

Same correlation ID:

    request_id=abc123

Search:

    request_id: abc123

This dramatically improves distributed troubleshooting.

---

# 91. Correlation ID Troubleshooting

If logs cannot be correlated:

Check:

- ID generated
- ID propagated
- ID logged
- Collector preserved field
- Elasticsearch mapping preserved field

A common problem:

    Application generates ID
       |
       v
    Service B loses it

Then distributed troubleshooting becomes fragmented.

---

# 92. Trace ID vs Correlation ID

They are related but not necessarily identical.

A trace ID identifies a distributed trace.

A correlation ID may be an application-level identifier.

If both exist, include them in structured logs where appropriate.

Example:

    trace_id
    span_id
    request_id

This makes logs easier to correlate with other telemetry.

---

# 93. Log Enrichment

Useful metadata includes:

    service
    environment
    namespace
    pod
    container
    node
    region
    version

Example:

    service=orders
    environment=prod
    namespace=production
    pod=orders-7d9...
    version=v2

Enrichment helps answer:

> Which exact workload produced this event?

---

# 94. Kubernetes Metadata Enrichment

A collector may add:

    pod_name
    namespace
    container_name
    node_name
    labels

This is extremely useful for troubleshooting.

Example query:

    namespace=production
    service=orders
    level=ERROR

Without metadata, searching large centralized logs becomes difficult.

---

# 95. Log Schema Consistency

Define a standard schema.

Example:

    timestamp
    level
    service
    environment
    version
    namespace
    pod
    message
    error_code
    request_id

Benefits:

- Consistent dashboards
- Easier queries
- Easier alerting
- Easier parsing
- Better incident response

---

# 96. Schema Evolution

Applications change over time.

If field:

    error_code

becomes:

    errorCode

dashboards and searches may break.

Treat log schema changes as compatibility changes.

Use:

- Versioned schemas where necessary
- Backward compatibility
- CI validation
- Documentation

---

# 97. Logging Configuration Through GitOps

Keep:

    Logstash pipelines
    Collector configuration
    Elasticsearch templates
    Kibana objects
    Kubernetes manifests

in Git where practical.

Architecture:

    Git
      |
      v
    CI Validation
      |
      v
    ArgoCD
      |
      v
    EKS

Benefits:

- Review
- Audit
- Rollback
- Reproducibility

---

# 98. Logstash Configuration Validation

Before production:

    Syntax validation
       |
       v
    Pipeline test
       |
       v
    Sample logs
       |
       v
    Elasticsearch test index
       |
       v
    Production

Do not test a new parsing rule directly against the production pipeline without validation.

---

# 99. Elasticsearch Mapping Validation

Before sending new application fields:

Check:

- Field names
- Data types
- Nested structure
- Expected cardinality
- Index templates

A bad mapping can cause ingestion failures or long-term query problems.

---

# 100. Kibana Dashboard Validation

After logging changes validate:

    Logs arriving
        |
        v
    Fields extracted
        |
        v
    Queries work
        |
        v
    Dashboards work
        |
        v
    Alerts work

A pipeline change can succeed while dashboards silently become incorrect.

---

# 101. Logging Troubleshooting During Deployment

After an application deployment check:

    [ ] Application logs
    [ ] Log level
    [ ] Log format
    [ ] Timestamp
    [ ] Metadata
    [ ] Correlation ID
    [ ] Collector ingestion
    [ ] Elasticsearch indexing
    [ ] Kibana search
    [ ] Existing dashboards

Monitoring and logging validation should be part of deployment verification.

---

# 102. Logging Troubleshooting During Kubernetes Upgrade

Potential issues:

- Container runtime behavior
- Log path changes
- Collector compatibility
- Security context
- Volume mounts
- Metadata changes
- Node replacement

After upgrade:

    Check collector DaemonSet
       |
       v
    Check every node
       |
       v
    Check log ingestion
       |
       v
    Check Elasticsearch
       |
       v
    Check Kibana

---

# 103. Logging Troubleshooting During Node Replacement

When a new node joins:

    New Node
       |
       v
    Collector scheduled?
       |
       v
    Log path accessible?
       |
       v
    Logs collected?
       |
       v
    Elasticsearch receives?

A new node without a collector creates a logging blind spot.

---

# 104. Logging During Autoscaling

When pods scale:

    10 Pods
       |
       v
    50 Pods

Log volume may increase dramatically.

Monitor:

- Events/sec
- Log bytes/sec
- Collector CPU
- Logstash queue
- Elasticsearch ingestion
- Disk growth

Autoscaling the application can unintentionally overload the logging platform.

---

# 105. Logging During Incident Storms

Example:

    Database outage
       |
       v
    Application retries
       |
       v
    Thousands of errors
       |
       v
    Log volume spike
       |
       v
    Elasticsearch pressure

The logging system may become another victim.

Use:

- Appropriate log levels
- Rate limiting where appropriate
- Controlled retries
- Sampling for non-critical events
- Capacity planning

Never suppress critical security/audit information blindly.

---

# 106. Log Pipeline Observability

Monitor the logging system itself.

Key signals:

## Collector

    Events read
    Events sent
    Errors
    CPU
    Memory

## Logstash

    Events in
    Events out
    Queue size
    Pipeline errors
    CPU
    Memory

## Elasticsearch

    Documents indexed
    Rejections
    Cluster health
    Disk
    Heap

## Kibana

    Query errors
    Query latency
    Availability

---

# 107. Log Ingestion Rate

Measure:

    events/sec

and:

    bytes/sec

A sudden increase may indicate:

- Traffic spike
- Error storm
- DEBUG enabled
- Application loop

A sudden decrease may indicate:

- Application failure
- Collector failure
- Filtering
- Network failure

Changes in ingestion rate are valuable troubleshooting signals.

---

# 108. Log Drop Rate

If the collector supports drop/error metrics, monitor them.

Example:

    Events received = 100,000
    Events forwarded = 80,000

Potential issue:

    20,000 events lost/dropped

Investigate:

- Filters
- Parsing
- Output errors
- Buffer limits
- Backpressure

Do not look only at successful events.

---

# 109. Log Pipeline Latency

Track:

    Event Creation
        |
        v
    Collection
        |
        v
    Processing
        |
        v
    Indexing
        |
        v
    Search

If users see delayed logs, identify where latency is introduced.

---

# 110. Logging Platform High Availability

A production logging platform should avoid single points of failure.

Example:

    Collector
        |
        v
    Logstash
      /     \
     A       B
      \     /
       Elasticsearch
       /    |    \
      A     B     C
             |
             v
           Kibana

Actual architecture depends on scale and requirements.

Key principle:

> Critical logging components should be designed across appropriate failure domains.

---

# 111. Logging Disaster Recovery

Back up:

- Configuration
- Index templates
- Lifecycle policies
- Critical dashboards
- Security configuration
- Required log data according to business/compliance needs

For Elasticsearch, distinguish:

    Configuration recovery

from:

    Data recovery

A rebuildable logging platform still needs a documented recovery process.

---

# 112. Logging Recovery Testing

Do not assume backups work.

Test:

    Backup
      |
      v
    Restore
      |
      v
    Validate
      |
      v
    Search
      |
      v
    Application Incident Investigation

Validate that restored logs are actually searchable and usable.

---

# 113. Logging Security Incident

If credentials appear in logs:

    1. Identify exposed secret.
    2. Determine exposure window.
    3. Restrict access.
    4. Rotate/revoke credential.
    5. Remove future logging.
    6. Review stored copies and retention.
    7. Investigate access.
    8. Document incident.

This requires coordination between DevOps and security teams.

---

# 114. Logging Performance Troubleshooting

Symptoms:

- Application slower
- CPU higher
- Network higher
- Log volume high

Possible cause:

    Excessive logging

Example:

    DEBUG logging
       |
       v
    10x events
       |
       v
    CPU/network/storage ↑

Production logging should balance diagnostic value with system overhead.

---

# 115. Application Logging Can Cause Application Problems

Logging is not free.

Costs include:

- CPU
- Memory
- Disk
- Network
- Serialization
- Lock/contention in some frameworks

An application writing enormous logs can degrade itself.

Troubleshooting should consider:

    Application performance
         +
    Logging overhead

---

# 116. Synchronous Logging Concerns

If application threads perform expensive synchronous logging:

    Request
       |
       v
    Log Serialization
       |
       v
    I/O
       |
       v
    Request delayed

High-volume logging can increase request latency.

Use appropriate logging architecture for application requirements.

---

# 117. Logging and Microservices

In a microservices architecture:

    Service A
       |
       v
    Service B
       |
       v
    Service C
       |
       v
    Database

Centralized logs should allow filtering by:

    service
    version
    environment
    request_id
    trace_id
    pod
    namespace

Without consistent metadata, troubleshooting a request across services becomes difficult.

---

# 118. Distributed Incident Log Investigation

Suppose payment fails.

Search by:

    request_id=abc123

Find:

    API Gateway
       |
       v
    Orders
       |
       v
    Payment
       |
       v
    Database

Then correlate:

    Error log
       +
    Deployment
       +
    Metrics
       +
    Dependency logs

This turns centralized logging into an incident investigation tool rather than a simple log archive.

---

# 119. Log Correlation With Metrics

Example:

    Error Rate ↑
       |
       v
    Search logs
       |
       v
    Payment timeout
       |
       v
    Database latency ↑
       |
       v
    Database issue

Metrics provide the signal.

Logs provide context.

---

# 120. Log Correlation With Kubernetes

Example:

    Pod Restarts ↑
       |
       v
    Kubernetes Events
       |
       v
    Container Logs
       |
       v
    OOMKilled

This provides stronger evidence than any single data source.

---

# 121. Log Correlation With Deployment History

Example:

    10:00 Deployment v2
    10:02 Error logs increase
    10:03 Pod restarts increase
    10:04 Alert fires

This suggests the deployment should be investigated.

Use Git/ArgoCD history to verify exactly what changed.

---

# 122. Production Incident — No Logs in Kibana

Situation:

    Application is healthy
    kubectl logs shows events
    Kibana shows nothing

Investigation:

    Application
       |
       v
    kubectl logs = OK
       |
       v
    Collector = ?
       |
       v
    Logstash = ?
       |
       v
    Elasticsearch = ?
       |
       v
    Kibana = ?

If collector receives logs but Elasticsearch has no documents:

    Investigate Logstash/Elasticsearch.

If Elasticsearch contains documents but Kibana has no results:

    Investigate data view/time/query.

---

# 123. Production Incident — Logs Missing From One Node

Situation:

    Node-A logs = OK
    Node-B logs = missing
    Node-C logs = OK

Investigation:

    Node-B
      |
      v
    Collector Pod
      |
      v
    Volume
      |
      v
    Permission
      |
      v
    Network
      |
      v
    Output

Likely localized to Node-B.

---

# 124. Production Incident — Elasticsearch Is Full

Symptoms:

- Indexing failures
- Kibana incomplete
- Logstash queue growth
- Disk near 100%

Investigation:

    Disk
      |
      v
    Largest indices
      |
      v
    Retention
      |
      v
    Log volume
      |
      v
    Duplicate events
      |
      v
    DEBUG/error storm

Mitigation:

- Restore capacity
- Apply appropriate retention
- Stop unnecessary volume
- Protect critical logs
- Validate indexing recovery

Do not delete random indices.

---

# 125. Production Incident — Logstash Queue Full

Situation:

    Logstash input healthy
    Queue growing rapidly
    Elasticsearch slow

Root cause may be:

    Elasticsearch indexing bottleneck

Investigate:

- Elasticsearch health
- Disk
- Heap
- Rejections
- Network
- Indexing rate

Mitigation may include reducing non-critical log volume and restoring Elasticsearch capacity.

---

# 126. Production Incident — Duplicate Logs

Situation:

    Same event appears twice.

Investigation:

    1. Check application output.
    2. Check number of collectors.
    3. Check file/stdout collection overlap.
    4. Check Logstash pipelines.
    5. Check retry behavior.
    6. Check index duplication.

Do not simply delete duplicates from Elasticsearch without fixing the source.

---

# 127. Production Incident — Logs Have Wrong Timestamp

Situation:

    Application event occurred at 10:00
    Kibana shows 04:30

Investigate:

- Application timezone
- Timestamp field
- Parsing
- Elasticsearch date mapping
- Kibana timezone
- UTC conversion

First determine whether the stored timestamp is actually wrong or only displayed differently.

---

# 128. Production Incident — Java Stack Traces Broken

Situation:

Kibana shows:

    Exception
    at line 1
    at line 2
    at line 3

as separate events.

Root cause:

    Multiline parsing failure.

Fix:

    Configure appropriate multiline handling

Then validate:

    One exception
       |
       v
    One searchable event

---

# 129. Production Incident — Logs Suddenly Increase 100x

Investigation:

    Application deployment
       |
       v
    Log level
       |
       v
    Exception loop
       |
       v
    Retry loop
       |
       v
    Traffic increase

Immediate concern:

    Elasticsearch capacity

But root cause may be application behavior.

Use metrics to correlate:

    Request rate
    Error rate
    Log rate

---

# 130. Production Incident — Sensitive Data Found in Logs

Treat as a security incident.

Actions:

    Stop further exposure
          |
          v
    Identify secret/data
          |
          v
    Rotate credential if required
          |
          v
    Fix application logging
          |
          v
    Review access
          |
          v
    Assess retained copies
          |
          v
    Document and prevent recurrence

Never treat sensitive log exposure as a simple formatting problem.

---

# 131. Production Incident — Kibana Is Slow

Possible causes:

- Elasticsearch slow
- Large query
- Huge time range
- Too many aggregations
- Cluster pressure
- Shard issues
- High concurrent users

Troubleshooting:

    Kibana
       |
       v
    Query
       |
       v
    Elasticsearch
       |
       v
    Shards
       |
       v
    Disk/Heap/CPU

Narrow the query first to determine whether the problem is data volume or cluster performance.

---

# 132. Production Incident — Logs Delayed by 10 Minutes

This is a latency problem.

Trace:

    Application timestamp
       |
       v
    Collector timestamp
       |
       v
    Logstash receipt
       |
       v
    Elasticsearch index time
       |
       v
    Kibana search

Identify where the delay begins.

Possible causes:

- Logstash queue
- Elasticsearch indexing
- Network
- Collector buffering
- Resource pressure

---

# 133. Production Incident — Logs Missing After Collector Restart

Possible causes:

- Collector offset/checkpoint issue
- File rotation
- Container restart timing
- Buffer loss
- Log source behavior

Investigate:

- Collector state
- Input configuration
- Persistent state
- Source log files
- Restart timeline

Understand whether the architecture guarantees at-least-once, best-effort, or another delivery behavior before making claims about log loss.

---

# 134. Log Delivery Semantics

Logging systems may provide different delivery guarantees.

Possible models include:

- Best effort
- At-most-once
- At-least-once

At-least-once systems can produce duplicates during retries.

Best-effort systems may lose data during failures.

Production design should explicitly understand the delivery behavior.

---

# 135. Exactly-Once Assumption

Do not casually assume:

> "Every log will appear exactly once."

Distributed systems make exactly-once behavior difficult.

If duplicates matter:

- Use event identifiers
- Design idempotent processing
- Detect duplicates
- Understand retry behavior

For operational logs, the priority is often reliable useful visibility rather than perfect mathematical uniqueness.

---

# 136. Logging Troubleshooting With Git

When logging changes:

    Git commit
       |
       v
    CI validation
       |
       v
    ArgoCD sync
       |
       v
    Kubernetes
       |
       v
    Collector
       |
       v
    Logstash
       |
       v
    Elasticsearch

If logs fail immediately after a sync, inspect the Git diff first.

---

# 137. Logging Troubleshooting With Terraform

Terraform may affect:

- Security Groups
- IAM
- EKS nodes
- Storage
- Network
- Load Balancers

If centralized logging stops after Terraform changes:

    Terraform
       |
       v
    Identify changed resource
       |
       v
    Compare before/after
       |
       v
    Test logging path

Do not modify application configuration unless evidence points there.

---

# 138. Logging Troubleshooting Checklist — Application

    [ ] Application running
    [ ] Correct log level
    [ ] Correct output destination
    [ ] stdout/stderr available
    [ ] Log format valid
    [ ] Timestamp valid
    [ ] Correlation ID present
    [ ] No sensitive information
    [ ] No excessive logging
    [ ] Previous logs checked after restart

---

# 139. Logging Troubleshooting Checklist — Collector

    [ ] DaemonSet healthy
    [ ] Collector on every required node
    [ ] Volumes mounted
    [ ] Permissions correct
    [ ] Input paths correct
    [ ] Filters correct
    [ ] Multiline configuration correct
    [ ] Network connectivity
    [ ] Output healthy
    [ ] Queue/buffer healthy

---

# 140. Logging Troubleshooting Checklist — Logstash

    [ ] Pod/process healthy
    [ ] Input listening
    [ ] Pipeline loaded
    [ ] Filter valid
    [ ] No unexpected drop conditions
    [ ] Queue healthy
    [ ] CPU healthy
    [ ] Memory healthy
    [ ] Output connected
    [ ] Elasticsearch acknowledgements/errors checked

---

# 141. Logging Troubleshooting Checklist — Elasticsearch

    [ ] Cluster reachable
    [ ] Authentication works
    [ ] TLS works
    [ ] Cluster health acceptable
    [ ] Disk healthy
    [ ] Heap healthy
    [ ] Shards healthy
    [ ] Index exists
    [ ] Documents increasing
    [ ] No mapping conflicts
    [ ] No indexing rejection
    [ ] Retention policy correct

---

# 142. Logging Troubleshooting Checklist — Kibana

    [ ] Kibana healthy
    [ ] Elasticsearch reachable
    [ ] User authorized
    [ ] Correct data view
    [ ] Correct time field
    [ ] Correct timezone
    [ ] Query valid
    [ ] Filters correct
    [ ] Dashboard panels valid

---

# 143. Logging Troubleshooting Decision Tree

    Log Missing?
          |
          v
    Exists in kubectl logs?
       /           \
     No             Yes
     |               |
 Application      Collector
 problem          problem?
                     |
                     v
               Reaches Logstash?
                  /       \
                No         Yes
                |           |
             Network      Elasticsearch
             /input       indexing?
                           /      \
                         No        Yes
                         |          |
                     ES/output    Kibana
                     problem      query/view
                                   problem

This is the core troubleshooting flow.

---

# 144. Follow the Log

Always ask:

    Where was the log last confirmed?

Example:

    Application = YES
    Collector = YES
    Logstash = YES
    Elasticsearch = NO
    Kibana = NO

The investigation should focus on:

    Logstash -> Elasticsearch

not:

    Application

---

# 145. Compare Healthy and Unhealthy Paths

If one application works:

    App-A -> Logs OK

and another fails:

    App-B -> Logs missing

Compare:

    Log format
    Namespace
    Labels
    Collector filters
    Output
    Metadata
    Application config

This often reveals the difference faster than reviewing the whole logging system.

---

# 146. Preserve Evidence

Before changing the pipeline, collect:

- Pod logs
- Collector logs
- Logstash logs
- Elasticsearch errors
- Kubernetes events
- Current configuration
- Git commit
- ArgoCD sync
- Index health
- Disk usage
- Queue metrics

Evidence disappears after restarts or cleanup.

---

# 147. Safe Logging Troubleshooting

Avoid:

    Delete all indices
    Restart every collector
    Disable security
    Remove all filters
    Increase resources blindly

Prefer:

    Confirm
       |
       v
    Isolate
       |
       v
    Make one controlled change
       |
       v
    Validate
       |
       v
    Document

---

# 148. Logging Troubleshooting — Mitigation vs Root Cause

Example:

    Elasticsearch disk full

Mitigation:

    Increase storage

Root cause:

    Excessive DEBUG logs caused 5x daily volume.

Permanent prevention:

    Correct application log level
    + retention review
    + volume alerting
    + capacity planning

Always separate immediate recovery from permanent correction.

---

# 149. Logging Root Cause Analysis

RCA should answer:

    What failed?

    Where did the first failure occur?

    Why did it occur?

    Why did logging not detect it earlier?

    Why did the logging platform degrade?

    What prevented faster recovery?

    What will prevent recurrence?

Example:

    Root cause:
    Application entered retry loop.

    Contributing factor:
    DEBUG logging enabled.

    Impact:
    Elasticsearch indexing saturation.

    Detection gap:
    No alert on log ingestion latency.

    Prevention:
    Rate-aware logging + platform alerts.

---

# 150. Five Whys — Logging Incident

Why were logs unavailable?

    Elasticsearch rejected writes.

Why?

    Disk reached the flood-stage threshold.

Why?

    Log volume increased significantly.

Why?

    A new application release logged full request payloads.

Why?

    Logging change was not reviewed for production volume impact.

Root cause:

> Logging changes were not included in production capacity review.

---

# 151. Logging Platform Meta-Monitoring

Monitor the logging platform itself:

    Collector health
    Event rate
    Drop rate
    Queue size
    Processing latency
    Elasticsearch indexing rate
    Indexing errors
    Disk
    Heap
    Cluster health
    Kibana availability

Without this, logging can silently fail.

---

# 152. Logging SLO Concepts

Possible logging service objectives:

    Log ingestion availability
    Log ingestion latency
    Search availability
    Search latency
    Critical log delivery success

Example:

    99.9% of critical logs indexed
    within the required operational window.

Logging SLOs should reflect business and incident-response requirements.

---

# 153. Logging Troubleshooting During Security Events

During security incidents, preserve:

- Authentication logs
- Authorization events
- Administrative actions
- Network/security logs
- Relevant application errors

Avoid destructive cleanup that could remove forensic evidence.

Coordinate with security teams for retention and evidence handling.

---

# 154. Production Logging Architecture Review

Before production, verify:

    Applications
        |
        v
    Structured Logs
        |
        v
    Node Collector
        |
        v
    Logstash
        |
        v
    Elasticsearch
        |
        v
      Kibana

Also verify:

    HA
    Security
    Retention
    Backup
    Capacity
    Alerting
    Runbooks
    DR

---

# 155. Senior-Level Logging Troubleshooting Framework

Use:

    OBSERVE
       |
       v
    CONFIRM SOURCE
       |
       v
    TRACE PIPELINE
       |
       v
    ISOLATE FAILURE
       |
       v
    CHECK CAPACITY
       |
       v
    MITIGATE
       |
       v
    VALIDATE
       |
       v
    RCA
       |
       v
    PREVENT

The strongest troubleshooting answers are structured and evidence-driven.

---

# 156. Interview Question — Logs Are Missing in Kibana. What Do You Check?

Strong answer:

> I trace the logging path from the application to Kibana. First I verify that the application is generating the expected logs with `kubectl logs`. Then I check the collector on the node, its volume mounts and filters, Logstash input and pipeline health, Elasticsearch connectivity and indexing, and finally Kibana's data view, time field and query. I identify the last layer where the log is confirmed before making changes.

---

# 157. Interview Question — How Do You Troubleshoot Duplicate Logs?

Strong answer:

> I first verify whether the application itself generated duplicates. Then I check whether multiple collectors are reading the same source, whether stdout and file collection overlap, whether multiple Logstash pipelines process the same event, and whether retries are causing duplicate delivery. I fix the source of duplication rather than deleting duplicate documents manually.

---

# 158. Interview Question — How Do You Troubleshoot Elasticsearch Disk Full?

Strong answer:

> I first assess cluster health, disk usage and indexing impact. Then I identify which indices and workloads are consuming storage and check retention, duplicate events, log volume spikes and shard strategy. I restore safe capacity first if required, then address the root cause such as excessive logging or incorrect retention. I avoid deleting production indices blindly.

---

# 159. Interview Question — How Do You Troubleshoot Logstash Backpressure?

Strong answer:

> I compare input and output rates and check queue growth. Then I determine whether the bottleneck is Logstash processing, filters, network, or Elasticsearch indexing. I check CPU, memory, persistent queue, Elasticsearch health and rejected writes. The goal is to identify why downstream processing cannot keep up rather than simply increasing queue capacity.

---

# 160. Interview Question — Why Use Structured Logging?

Strong answer:

> Structured logging gives every event a predictable schema, making filtering, aggregation, alerting and incident investigation much easier. Fields such as service, environment, level, timestamp, request ID and error code can be queried directly instead of parsing free-form messages repeatedly.

---

# 161. Interview Question — How Do You Handle Sensitive Data in Logs?

Strong answer:

> I prevent sensitive information from being logged at the application level and use redaction where appropriate. If secrets are discovered in logs, I treat it as a security issue, determine the exposure, rotate compromised credentials when necessary, restrict access, review retained copies and fix the logging code to prevent recurrence.

---

# 162. Interview Question — How Do You Troubleshoot Multiline Logs?

Strong answer:

> I inspect the raw log source first. If a stack trace is being split into separate events, I configure multiline handling at the appropriate collection layer so one exception becomes one event. I then validate the resulting event in Elasticsearch and confirm Kibana searches return the complete stack trace.

---

# 163. Interview Question — How Do You Troubleshoot Logs From Only One Node Missing?

Strong answer:

> If logs from other nodes are healthy, I treat it as a localized node or collector issue. I check whether the collector DaemonSet pod is running on that node, verify its volume mounts and permissions, inspect collector logs, check the node's log files and disk, and test connectivity to the central logging pipeline.

---

# 164. Interview Question — How Do You Prevent Logging From Overloading Production?

Strong answer:

> I standardize log levels, avoid excessive DEBUG logging, prevent large payloads from being logged unnecessarily, monitor log ingestion volume, use appropriate filtering and sampling for non-critical data, apply retention policies, and capacity-plan the logging platform. I make sure security and audit requirements are preserved while reducing unnecessary volume.

---

# 165. Interview Question — How Do You Correlate Logs Across Microservices?

Strong answer:

> I use consistent metadata such as service, environment, version and namespace, along with a propagated request or correlation ID. Where tracing is available, I also include trace and span identifiers. This allows me to follow one request across multiple services and correlate logs with metrics and deployment events.

---

# 166. Interview Question — What Is Your Production Logging Troubleshooting Process?

Strong answer:

> I first confirm whether the application generated the event. Then I follow it through the collector, Logstash, Elasticsearch and Kibana. I check the last confirmed point, inspect errors and queues, correlate the issue with recent deployments or infrastructure changes, and determine whether the problem is loss, delay, duplication, parsing or search visibility. I mitigate safely, validate recovery, perform RCA and add preventive monitoring.

---

# 167. Advanced Scenario — All Logs From the Cluster Are Missing

Investigation:

    Applications
       |
       v
    kubectl logs = OK
       |
       v
    Collectors
       |
       X
    Collector deployment issue

Check:

    DaemonSet
    Pods
    Volumes
    RBAC
    Network

If collectors are healthy:

    Logstash
       |
       v
    Elasticsearch

Then investigate central pipeline health.

---

# 168. Advanced Scenario — Logs Exist in Elasticsearch but Kibana Shows None

This strongly suggests a search/UI layer problem.

Check:

    Index exists
       |
       v
    Document exists
       |
       v
    Data view
       |
       v
    Time field
       |
       v
    Time range
       |
       v
    Query filters
       |
       v
    Permissions

Do not modify Logstash if Elasticsearch already contains the expected documents.

---

# 169. Advanced Scenario — Logstash Receives Logs but Elasticsearch Rejects Them

Check exact rejection reason.

Possible causes:

    Mapping conflict
    Authentication
    Disk watermark
    Index block
    Cluster pressure
    Invalid request

Use the exact error message to choose the next layer.

Do not assume all Elasticsearch failures are storage problems.

---

# 170. Advanced Scenario — Logging Becomes Slow During Traffic Spike

Timeline:

    Traffic ↑
       |
       v
    Application logs ↑
       |
       v
    Collector CPU ↑
       |
       v
    Logstash queue ↑
       |
       v
    Elasticsearch indexing latency ↑
       |
       v
    Kibana delayed

This is a capacity problem across the pipeline.

Determine the first saturated component.

---

# 171. Advanced Scenario — Application Is Slow and Logs Increased

Possible relationship:

    Application bug
       |
       v
    Retry loop
       |
       v
    Error log storm
       |
       v
    CPU/network/storage pressure
       |
       v
    Application slows further

This is a feedback loop.

The logging platform may amplify an existing application problem.

---

# 172. Advanced Scenario — Logging Configuration Breaks After GitOps Deployment

Workflow:

    Git commit
       |
       v
    ArgoCD sync
       |
       v
    Collector configuration
       |
       v
    Logs disappear

Check:

    Git diff
    ArgoCD diff
    ConfigMap
    Pod configuration
    Collector logs

If the previous configuration worked:

    Revert
       |
       v
    Validate recovery
       |
       v
    Fix configuration
       |
       v
    Re-deploy safely

---

# 173. Advanced Scenario — New Application Version Changes Log Schema

Old:

    error_code

New:

    errorCode

Impact:

- Dashboards fail
- Searches fail
- Alerts fail
- Parsing may fail

Treat schema changes as compatibility changes.

Validate:

    Application
       |
       v
    Collector
       |
       v
    Elasticsearch mapping
       |
       v
    Kibana
       |
       v
    Alerts

---

# 174. Advanced Scenario — Elasticsearch Is Healthy but Log Ingestion Is Delayed

Possible causes:

- Logstash queue
- Collector buffer
- Network
- Filtering
- Processing bottleneck

Check timestamps at every stage.

If:

    Logstash queue ↑

then downstream processing is likely slower than input.

If:

    Queue normal

but indexing delay high:

    Investigate Elasticsearch.

---

# 175. Advanced Scenario — One Log Field Suddenly Cannot Be Searched

Example:

    error_code

was searchable yesterday but not today.

Check:

- Mapping
- Index template
- New index
- Field type
- Data view
- Query syntax

The problem may be a schema/mapping change rather than missing logs.

---

# 176. Logging Troubleshooting Best Practices

1. Start at the source.
2. Follow the log through every pipeline stage.
3. Identify the last confirmed point.
4. Separate loss from delay.
5. Separate ingestion from search.
6. Check scope before changing anything.
7. Check recent changes.
8. Preserve evidence.
9. Use structured logs.
10. Standardize timestamps.
11. Propagate correlation IDs.
12. Monitor collectors.
13. Monitor Logstash queues.
14. Monitor Elasticsearch disk and heap.
15. Validate mappings.
16. Control retention.
17. Protect sensitive data.
18. Test configuration changes.
19. Validate recovery.
20. Perform RCA and prevention.

---

# 177. Final Logging Troubleshooting Mental Model

When logs are missing:

    SOURCE
       |
       v
    GENERATION
       |
       v
    COLLECTION
       |
       v
    TRANSPORT
       |
       v
    PROCESSING
       |
       v
    INDEXING
       |
       v
    SEARCH
       |
       v
    VISUALIZATION

When logs are slow:

    GENERATION
       |
       v
    QUEUE
       |
       v
    PROCESSING
       |
       v
    INDEXING
       |
       v
    SEARCH

When logs are duplicated:

    SOURCE
       |
       +---- Collector A
       |
       +---- Collector B
       |
       +---- Retry
       |
       v
    Duplicate Event

When logs are incorrect:

    Application Format
          |
          v
       Parsing
          |
          v
       Mapping
          |
          v
       Search

---

# 178. Final Production Logging Troubleshooting Flow

                    LOGGING INCIDENT
                           |
                           v
                    CONFIRM SYMPTOM
                           |
                           v
                      DEFINE SCOPE
                           |
                           v
                  CHECK APPLICATION LOGS
                           |
                           v
                    CHECK COLLECTOR
                           |
                           v
                     CHECK LOGSTASH
                           |
                           v
                  CHECK ELASTICSEARCH
                           |
                           v
                       CHECK KIBANA
                           |
                           v
                   CHECK RECENT CHANGES
                           |
                           v
                      CHECK CAPACITY
                           |
                           v
                        MITIGATE
                           |
                           v
                       VALIDATE
                           |
                           v
                          RCA
                           |
                           v
                      PREVENTION

---

# 179. Final Production Principles

Remember:

> Start at the source, not at Kibana.

> Find the last place where the log is confirmed.

> A Running collector does not mean logs are being collected.

> A healthy Elasticsearch cluster does not mean every log was indexed.

> Logs existing in Elasticsearch does not mean Kibana is configured correctly.

> Log delay and log loss are different problems.

> Duplicate logs usually require fixing the collection/delivery path, not deleting documents.

> Structured logging makes production troubleshooting dramatically easier.

> Correlation IDs are essential for microservice incident investigation.

> Logging configuration is production configuration and should be version controlled.

> The logging platform itself must be monitored.

The complete production logging troubleshooting mindset is:

    CONFIRM
       +
    TRACE
       +
    ISOLATE
       +
    PRESERVE
       +
    MITIGATE
       +
    VALIDATE
       +
    RCA
       +
    PREVENT

This is the approach expected from a production DevOps / DevSecOps engineer when centralized logging fails during a real incident.
