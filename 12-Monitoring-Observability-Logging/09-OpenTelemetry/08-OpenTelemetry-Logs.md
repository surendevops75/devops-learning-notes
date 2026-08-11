# OpenTelemetry Logs

## 1. Overview

OpenTelemetry Logs provides a standardized way to generate, collect, process, correlate, and export application logs.

The basic flow is:

```text
Application
     ↓
OpenTelemetry Logs API
     ↓
OpenTelemetry SDK
     ↓
Log Record
     ↓
Exporter
     ↓
OpenTelemetry Collector
     ↓
Log Backend
     ↓
Elasticsearch / Kibana
```

OpenTelemetry Logs becomes especially powerful when logs are correlated with traces and metrics.

```text
Metrics
   ↓
Problem detected
   ↓
Logs
   ↓
Detailed event information
   ↓
Trace
   ↓
Distributed request path
```

---

# 2. What Is a Log?

A log is a record describing an event that occurred in an application or system.

Examples:

```text
Application started
User authentication failed
Payment completed
Database connection failed
HTTP request returned 500
Order created
```

A traditional log might look like:

```text
2026-08-11 10:15:23 ERROR Payment failed
```

A structured OpenTelemetry log can contain much more information.

```text
Timestamp
Severity
Body
Attributes
Resource
Trace ID
Span ID
Instrumentation Scope
```

---

# 3. OpenTelemetry Log Architecture

```text
                    Application
                         │
                         ↓
                  Logging API / SDK
                         │
                         ↓
                    Log Record
                         │
                         ↓
                   Log Processor
                         │
                         ↓
                    Log Exporter
                         │
                         ↓
                  OTel Collector
                         │
                         ↓
                  Log Processing
                         │
                         ↓
                 Elasticsearch / ELK
                         │
                         ↓
                       Kibana
```

---

# 4. Why OpenTelemetry Logs?

Traditional applications often use different logging frameworks:

```text
Java       → Logback / Log4j
Node.js    → Winston / Pino
Python     → logging
Go         → standard logging / third-party libraries
```

OpenTelemetry provides a standardized observability model around logs.

The goal is to make logs easier to:

```text
Collect
Correlate
Process
Export
Analyze
```

---

# 5. Logs vs OpenTelemetry Logs

Traditional logging:

```text
Application
    ↓
Log File
    ↓
File Collector
    ↓
ELK
```

OpenTelemetry-based logging:

```text
Application
    ↓
OTel Logs
    ↓
OTLP
    ↓
Collector
    ↓
ELK
```

Both approaches can coexist during migration.

---

# 6. Log Record

A log record represents one log event.

Conceptually:

```text
Log Record
│
├── Timestamp
├── Observed Timestamp
├── Severity
├── Severity Text
├── Body
├── Attributes
├── Resource
├── Trace ID
├── Span ID
└── Instrumentation Scope
```

Example:

```text
Timestamp:
2026-08-11T10:15:23Z

Severity:
ERROR

Body:
Payment gateway timeout

service.name:
payment

trace_id:
abc123
```

---

# 7. Log Body

The body contains the primary log message.

Example:

```text
Payment gateway timeout
```

It may also contain structured data depending on the application and logging implementation.

For example:

```json
{
  "message": "Payment gateway timeout",
  "retryable": true
}
```

Structured log bodies are easier to process than unstructured text.

---

# 8. Severity

Severity indicates how important the event is.

Typical levels include:

```text
TRACE
DEBUG
INFO
WARN
ERROR
FATAL
```

The exact mapping depends on the application's logging framework and OpenTelemetry integration.

---

# 9. Severity Number

OpenTelemetry represents severity using a standardized numeric range.

The important concept is:

```text
Lower severity
      ↓
Higher severity
```

Applications can map framework-specific levels into OpenTelemetry severity values.

For example:

```text
DEBUG → Debug
INFO  → Info
WARN  → Warn
ERROR → Error
```

---

# 10. Severity Text

Severity text preserves the textual representation used by the application.

Example:

```text
severity_number = ERROR
severity_text   = ERROR
```

or:

```text
severity_text = WARN
```

This makes log filtering easier in backend systems.

---

# 11. Log Attributes

Attributes provide additional information about a log event.

Example:

```text
http.request.method = POST
http.response.status_code = 500
service.name = payment
environment = production
```

Architecture:

```text
Log Record
    │
    ├── Body
    ├── Severity
    └── Attributes
          ├── HTTP
          ├── Database
          └── Application
```

---

# 12. Resource Attributes

Resource attributes identify the entity producing the log.

Example:

```text
service.name = payment
service.version = v2.1.0
deployment.environment = production
```

In Kubernetes:

```text
k8s.namespace.name = production
k8s.pod.name = payment-7d9f8c
k8s.container.name = payment
```

This makes it possible to identify where a log originated.

---

# 13. Instrumentation Scope

A log record can contain information about the instrumentation scope that generated it.

Conceptually:

```text
Application
     ↓
Logging Instrumentation
     ↓
Instrumentation Scope
     ↓
Log Record
```

This is useful when multiple instrumentation libraries are involved.

---

# 14. Trace ID in Logs

One of the most important OpenTelemetry capabilities is trace correlation.

Example:

```text
Log:
Payment gateway timeout

trace_id:
abc123
```

The same trace ID exists in the distributed trace.

Therefore:

```text
Kibana
   ↓
Log
   ↓
trace_id
   ↓
Trace Backend
```

An engineer can move from a log to the complete request trace.

---

# 15. Span ID in Logs

A log can also contain a Span ID.

Example:

```text
trace_id = abc123
span_id  = def456
```

This identifies the specific operation that generated the log.

Architecture:

```text
Trace
│
├── HTTP Span
│
├── Database Span
│      └── Log
│
└── Payment Span
```

The log can point directly to the relevant span.

---

# 16. Log-Trace Correlation

Suppose a user reports:

```text
Payment failed
```

The engineer sees:

```text
Metric:
payment_errors_total ↑
```

Then:

```text
Log:
Payment gateway timeout
trace_id=abc123
```

Then:

```text
Trace:
Frontend
  ↓
Orders
  ↓
Payment
  ↓
Payment Gateway
```

This provides a complete troubleshooting workflow.

---

# 17. Context Propagation

Trace context must be propagated between services.

```text
Frontend
   ↓
traceparent
   ↓
Orders
   ↓
traceparent
   ↓
Payment
```

The same trace ID can then appear in:

```text
Spans
Logs
Metrics exemplars / related telemetry
```

where supported.

---

# 18. Structured Logging

Structured logging stores information as fields rather than embedding everything into a plain string.

Unstructured:

```text
Payment failed for order 123 because gateway timeout
```

Structured:

```json
{
  "event": "payment_failed",
  "order_id": "123",
  "reason": "gateway_timeout"
}
```

Structured logs are much easier to query and analyze.

---

# 19. Structured Logging With OpenTelemetry

A production log record might contain:

```text
severity = ERROR
event.name = payment_failed
order.status = failed
service.name = payment
deployment.environment = production
trace_id = abc123
span_id = def456
```

This allows Kibana queries such as:

```text
service.name = "payment"
AND severity = "ERROR"
```

---

# 20. Traditional Application Logging

An application may already have logging:

```text
Java
  ↓
Logback
  ↓
Console
```

OpenTelemetry does not necessarily require replacing the existing logging framework.

Instead, the application can integrate existing logs with OpenTelemetry.

---

# 21. Log Bridge Concept

A logging bridge can connect an existing framework to OpenTelemetry.

Conceptually:

```text
Application
     ↓
Existing Logging Framework
     ↓
OTel Log Integration
     ↓
OpenTelemetry Log Record
     ↓
Exporter
```

This is useful when migrating existing applications.

---

# 22. Application Logs in Kubernetes

A common Kubernetes logging model is:

```text
Application
     ↓
stdout / stderr
     ↓
Container Runtime
     ↓
Log Collection
     ↓
OTel Collector
     ↓
ELK
```

This avoids managing log files inside application containers.

---

# 23. Kubernetes Container Logs

Example:

```text
Pod
│
└── payment container
       ↓
    stdout
       ↓
Container log
       ↓
OTel Collector
```

The Collector can read container logs using an appropriate receiver such as `filelog`.

---

# 24. Filelog Receiver

The filelog receiver can collect logs from files.

Architecture:

```text
Container Logs
      ↓
Node filesystem
      ↓
Filelog Receiver
      ↓
OTel Collector
      ↓
Logs Pipeline
```

In Kubernetes, the Collector must have the appropriate filesystem access.

---

# 25. Kubernetes DaemonSet Logging

A common architecture is:

```text
Node-01
├── Application Pods
└── OTel Collector Agent

Node-02
├── Application Pods
└── OTel Collector Agent

Node-03
├── Application Pods
└── OTel Collector Agent
```

Each Collector can collect logs from its local node.

---

# 26. Kubernetes Log Collection Flow

```text
Application
     ↓
stdout / stderr
     ↓
Container Runtime
     ↓
Node Log Files
     ↓
OTel Collector DaemonSet
     ↓
OTel Gateway
     ↓
ELK
```

This is a scalable pattern for Kubernetes environments.

---

# 27. OTel Collector Logs Pipeline

A typical logs pipeline:

```text
Log Source
    ↓
Filelog / OTLP Receiver
    ↓
Memory Limiter
    ↓
Resource Processor
    ↓
Attributes / Transform
    ↓
Filter
    ↓
Batch
    ↓
Exporter
    ↓
ELK
```

---

# 28. OTLP Logs

Applications can send logs directly through OTLP.

```text
Application
     ↓
OpenTelemetry SDK
     ↓
OTLP
     ↓
OTel Collector
```

This provides a native OpenTelemetry logging path.

---

# 29. OTLP Logs Over gRPC

OTLP/gRPC commonly uses port:

```text
4317
```

Architecture:

```text
Application
     ↓
OTLP/gRPC
     ↓
Collector :4317
```

The application and Collector must use compatible protocol configuration.

---

# 30. OTLP Logs Over HTTP

OTLP/HTTP commonly uses:

```text
4318
```

Architecture:

```text
Application
     ↓
OTLP/HTTP
     ↓
Collector :4318
```

Use the protocol that best fits the environment.

---

# 31. Collector Logs Receiver

Example conceptual configuration:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
      http:

  filelog:
    include:
      - /var/log/pods/*/*/*.log
```

The exact Kubernetes paths depend on the node runtime and deployment configuration.

---

# 32. Logs Processor

Processors can modify logs before export.

Examples:

```text
Resource
Attributes
Filter
Transform
Batch
Memory Limiter
```

Architecture:

```text
Logs
 ↓
Processor
 ↓
Processed Logs
 ↓
Exporter
```

---

# 33. Log Filtering

Not every log needs to be stored.

Example:

```text
DEBUG logs
     ↓
Filter
     X
```

while:

```text
INFO
WARN
ERROR
     ↓
ELK
```

Filtering can reduce:

```text
Storage
Network
Elasticsearch ingestion
Cost
```

---

# 34. Filter by Environment

Example:

```text
deployment.environment = development
```

could have more verbose logs than:

```text
deployment.environment = production
```

Production should generally prioritize useful operational logs rather than unlimited debug output.

---

# 35. Filter Health Checks

Kubernetes and load balancers can generate large numbers of health-check logs.

Example:

```text
GET /health
GET /ready
GET /health
GET /ready
```

If these provide little operational value, they can be filtered or handled separately.

---

# 36. Log Enrichment

The Collector can add information to logs.

Example:

```text
Original:
Payment failed
```

After enrichment:

```text
Payment failed
service.name=payment
environment=production
namespace=production
cluster=prod-eks
```

This makes centralized logging much more useful.

---

# 37. Kubernetes Metadata Enrichment

A log can be enriched with:

```text
Cluster
Namespace
Pod
Container
Node
Deployment
```

Example:

```text
k8s.namespace.name = production
k8s.pod.name = payment-7d9f8c
k8s.container.name = payment
```

This makes it easy to identify the exact workload.

---

# 38. Log Parsing

Applications may produce unstructured logs.

Example:

```text
2026-08-11 10:15:23 ERROR Payment failed
```

A Collector or downstream processor may parse this into fields such as:

```text
timestamp
severity
message
```

However, structured logging at the application level is generally preferable when possible.

---

# 39. JSON Logs

A better application format:

```json
{
  "timestamp": "2026-08-11T10:15:23Z",
  "level": "ERROR",
  "message": "Payment failed",
  "service": "payment"
}
```

The Collector can process these fields and preserve useful structure.

---

# 40. Log Body vs Attributes

A useful design:

```text
Body:
Payment gateway timeout
```

Attributes:

```text
service.name = payment
payment.provider = stripe
error.type = timeout
http.status_code = 504
```

The body describes the event.

Attributes provide searchable dimensions.

---

# 41. Avoid High-Cardinality Log Fields

Logs can contain high-cardinality fields safely in some cases because logs are event-oriented, but uncontrolled data can still increase storage and indexing costs.

Examples requiring caution:

```text
Full request body
Large stack traces
Large URLs
Large payloads
Repeated unique identifiers
```

Store only information that provides operational value.

---

# 42. Sensitive Data

Never log:

```text
Passwords
Access tokens
Private keys
Credit card information
Session secrets
Authentication credentials
```

Example of a dangerous log:

```text
password=MyPassword123
```

This can create a serious security incident.

---

# 43. PII in Logs

Be careful with:

```text
Email addresses
Phone numbers
Addresses
Government identifiers
Personal information
```

Use masking or redaction when such information is genuinely required.

Prefer:

```text
customer_id = masked/controlled identifier
```

over storing unnecessary personal information.

---

# 44. Log Redaction

Example:

```text
Original:
Authorization: Bearer eyJ...

Redacted:
Authorization: [REDACTED]
```

Redaction can occur at:

```text
Application
Collector
Log processor
Backend ingestion
```

The earlier sensitive data is removed, the safer the system.

---

# 45. Log Severity Strategy

Production applications should have meaningful severity levels.

```text
DEBUG
 ↓
Detailed troubleshooting

INFO
 ↓
Normal important events

WARN
 ↓
Potential problem

ERROR
 ↓
Operation failed

FATAL
 ↓
Critical application failure
```

Do not classify every normal event as ERROR.

---

# 46. Good INFO Logs

Examples:

```text
Application started
Configuration loaded
Payment completed
Order created
Database connection established
```

These represent useful lifecycle or business events.

---

# 47. Good WARN Logs

Examples:

```text
Retrying payment request
Connection pool near capacity
External API response slow
Cache miss rate increasing
```

A warning should indicate something worth watching.

---

# 48. Good ERROR Logs

Examples:

```text
Payment gateway unavailable
Database query failed
Message processing failed
External API request failed
```

An ERROR should generally represent a failed operation or condition requiring investigation.

---

# 49. Exception Logging

An exception log should provide enough context to troubleshoot.

Useful fields:

```text
error.type
error.message
stack trace
service.name
operation
trace_id
span_id
```

Avoid logging sensitive request payloads alongside exceptions.

---

# 50. Stack Traces

Stack traces are useful for application debugging.

Example:

```text
NullPointerException
    at PaymentService.process()
    at OrderService.checkout()
```

But stack traces can be large.

Control:

```text
Log volume
Retention
Storage
Indexing
```

---

# 51. Log Correlation With Kubernetes

Example:

```text
Log
 ↓
service.name=payment
 ↓
k8s.namespace.name=production
 ↓
k8s.pod.name=payment-7d9f8c
 ↓
k8s.node.name=node-03
```

This lets engineers determine exactly where the event occurred.

---

# 52. Log Correlation With Deployment

Add:

```text
service.version
deployment.environment
```

Example:

```text
service.name=payment
service.version=v2.3.1
deployment.environment=production
```

This makes release-related incidents easier to investigate.

---

# 53. Release Troubleshooting

Suppose:

```text
Version v2.2.0
ERROR rate = 0.5%

Version v2.3.0
ERROR rate = 7%
```

Logs can identify:

```text
Payment timeout
Database exception
Configuration error
```

Then traces can show where the failure occurs.

---

# 54. Log Export to ELK

A common architecture:

```text
Application
     ↓
OTel SDK / Filelog
     ↓
OTel Collector
     ↓
Log Processing
     ↓
Logstash / Elasticsearch
     ↓
Elasticsearch
     ↓
Kibana
```

The exact topology depends on the existing ELK architecture.

---

# 55. Collector to Logstash

If Logstash is already part of the environment:

```text
Application
     ↓
OTel Collector
     ↓
Logstash
     ↓
Elasticsearch
     ↓
Kibana
```

The Collector provides:

```text
Collection
Enrichment
Filtering
Batching
Routing
```

Logstash can continue handling existing transformation or ingestion workflows.

---

# 56. Collector Directly to Elasticsearch

A supported Elasticsearch exporter can allow:

```text
Application
     ↓
OTel Collector
     ↓
Elasticsearch
     ↓
Kibana
```

This can simplify the architecture when Logstash is not required.

---

# 57. Choosing Collector vs Logstash

Use the Collector when you want:

```text
OpenTelemetry-native collection
Unified metrics/logs/traces
OTLP
Vendor-neutral telemetry processing
```

Keep Logstash when you need:

```text
Existing pipelines
Existing transformations
Existing integrations
Existing operational knowledge
```

Both can coexist.

---

# 58. Centralized Logging Architecture

For EKS:

```text
                    EKS
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
    Node-01       Node-02       Node-03
       │             │             │
    OTel Agent    OTel Agent    OTel Agent
       │             │             │
       └─────────────┼─────────────┘
                     ↓
                OTel Gateway
                     ↓
                  Logstash
                     ↓
               Elasticsearch
                     ↓
                   Kibana
```

---

# 59. Logs From Multiple Services

```text
Orders ──────┐
Payment ─────┤
Inventory ───┼→ OTel Collector → ELK
User ────────┤
Notification ┘
```

All services can use common fields:

```text
service.name
service.version
environment
trace_id
span_id
```

This makes centralized analysis much easier.

---

# 60. Multi-Environment Logging

Separate environments logically:

```text
development
staging
production
```

Use attributes such as:

```text
deployment.environment
```

Then Kibana queries can filter:

```text
deployment.environment = "production"
```

Avoid mixing production and development logs without clear environment metadata.

---

# 61. Multi-Cluster Logging

For multiple EKS clusters:

```text
prod-eks
   ↓
Collector
   ↓
Gateway
   ↓
Central ELK

staging-eks
   ↓
Collector
   ↓
Gateway
   ↓
Central ELK
```

Add:

```text
k8s.cluster.name
deployment.environment
```

to identify the source.

---

# 62. Multi-Region Logging

Example:

```text
ap-south-1
     ↓
Collector
     ↓
Regional Gateway
     ↓
Central ELK

ap-southeast-1
     ↓
Collector
     ↓
Regional Gateway
     ↓
Central ELK
```

Consider:

```text
Network cost
Latency
Data residency
Availability
Backend capacity
```

---

# 63. Log Retention

Not every log needs the same retention period.

Example:

```text
Debug logs
   ↓
Short retention

Application errors
   ↓
Longer retention

Security/audit logs
   ↓
Retention according to requirements
```

Retention should be determined by:

```text
Operational needs
Compliance
Security
Storage cost
Incident investigation
```

---

# 64. Log Storage Cost

Logging can become expensive.

Cost drivers include:

```text
Log volume
Retention
Indexing
Replication
Large messages
Stack traces
High-frequency events
```

Control costs using:

```text
Filtering
Sampling where appropriate
Retention policies
Compression
Tiered storage
```

Do not indiscriminately drop important error or security events.

---

# 65. Log Sampling

Logs are different from traces, so sampling should be applied carefully.

Potentially safe candidates:

```text
Repeated successful health checks
Very noisy low-value informational events
```

Avoid blindly sampling:

```text
Errors
Security events
Audit events
Critical business failures
```

---

# 66. Log Deduplication

Repeated identical logs can create excessive volume.

Example:

```text
Connection timeout
Connection timeout
Connection timeout
Connection timeout
...
```

A system can become noisy if one failure generates thousands of identical messages.

Use:

```text
Rate limiting
Aggregation
Filtering
Root-cause investigation
```

where appropriate.

---

# 67. Logging and Alerting

Logs can also trigger alerts.

Example:

```text
ERROR payment gateway unavailable
        ↓
Log pipeline
        ↓
Detection rule
        ↓
Alert
```

However, metrics are generally better for measuring rates and thresholds.

Use logs for detailed event context.

---

# 68. Metrics + Logs Alerting

Better architecture:

```text
Metric:
payment_error_rate > 5%
        ↓
Alert

Logs:
payment gateway timeout
        ↓
Investigation context
```

Metrics detect the problem.

Logs explain the individual events.

---

# 69. Logs and Traces During Incident Response

Incident workflow:

```text
1. Grafana
   ↓
High error rate

2. Kibana
   ↓
Payment timeout logs

3. Trace backend
   ↓
Database / gateway span slow

4. Kubernetes
   ↓
Check affected pods

5. Root cause
   ↓
Database connection exhaustion
```

This is the practical observability workflow.

---

# 70. Log Search Strategy

Good log searches usually start with:

```text
service.name
environment
severity
time range
```

Then narrow down using:

```text
error.type
http.route
status
trace_id
pod
version
```

Avoid searching the entire log corpus without filters.

---

# 71. Example Kibana Investigation

Start:

```text
deployment.environment = production
```

Then:

```text
service.name = payment
```

Then:

```text
severity = ERROR
```

Then:

```text
trace_id = abc123
```

This progressively narrows the incident.

---

# 72. Log Correlation Example

Suppose:

```text
Trace ID:
4bf92f3577b34da6a3ce929d0e0e4736
```

The same trace ID appears in:

```text
Orders log
Payment log
Database-related log
```

Kibana can search:

```text
trace_id = "4bf92f3577b34da6a3ce929d0e0e4736"
```

This reveals the logs associated with the distributed request.

---

# 73. Trace Context in Asynchronous Systems

For:

```text
Orders
   ↓
RabbitMQ
   ↓
Payment
```

trace context can be propagated through message metadata.

Then logs from the consumer can retain the same trace relationship.

```text
Producer
   ↓
Message
   ↓
Consumer
   ↓
Log
```

This is especially useful in microservices architectures.

---

# 74. Logging in a Java Application

Conceptually:

```text
Java Application
      ↓
Logback / Log4j
      ↓
OpenTelemetry integration
      ↓
OTel Log Record
      ↓
OTLP
      ↓
Collector
```

Existing logging frameworks can often remain in place while telemetry is standardized.

---

# 75. Logging in Node.js

A Node.js application may use:

```text
Pino
Winston
Console
```

Architecture:

```text
Node.js
   ↓
Logging Framework
   ↓
OTel Integration
   ↓
OTLP
   ↓
Collector
```

Structured logging is especially useful in Node.js microservices.

---

# 76. Logging in Python

Python applications commonly use:

```text
logging
```

Architecture:

```text
Python
   ↓
Python logging
   ↓
OTel integration
   ↓
Collector
```

The exact implementation depends on the chosen OpenTelemetry logging support and application architecture.

---

# 77. Logging in Kubernetes Containers

Prefer:

```text
Application
   ↓
stdout / stderr
```

instead of:

```text
Application
   ↓
Local application file
```

for many Kubernetes workloads.

Why?

```text
Container-native
Easier collection
Works well with DaemonSet collectors
No application log rotation responsibility
```

---

# 78. Log Rotation

When applications write directly to files, log rotation becomes important.

Without rotation:

```text
Application log
     ↓
10 GB
     ↓
50 GB
     ↓
Disk full
```

Container stdout logging can reduce this operational burden when the Kubernetes/container runtime logging model is properly configured.

---

# 79. Collector Log Rotation Considerations

The Collector itself also produces logs.

Monitor:

```text
Collector log volume
Error rate
Repeated warnings
Exporter failures
```

Do not allow diagnostic logging to become another source of excessive storage usage.

---

# 80. Collector Self-Logs

Collector logs can reveal:

```text
Exporter connection failures
Configuration errors
Pipeline errors
Receiver failures
Authentication failures
Backpressure
```

Example:

```text
exporter failed to send data
```

This indicates the problem may be downstream rather than in the application.

---

# 81. Collector Monitoring

Monitor the Collector with metrics such as:

```text
Received logs
Exported logs
Dropped logs
Exporter failures
Queue size
CPU
Memory
```

Architecture:

```text
Collector
   ↓
Collector Metrics
   ↓
Prometheus
   ↓
Grafana
```

The logging pipeline itself needs observability.

---

# 82. Log Pipeline Failure

If logs disappear:

```text
Application
   ↓
Log Source
   ↓
Receiver
   ↓
Processor
   ↓
Exporter
   ↓
Backend
```

Check every stage.

Do not immediately assume Elasticsearch is the problem.

---

# 83. Troubleshooting No Logs

Check:

```text
1. Is the application generating logs?
2. Are logs written to stdout/file?
3. Is the Collector receiving them?
4. Is the logs pipeline enabled?
5. Is a processor filtering them?
6. Is the exporter working?
7. Is the backend reachable?
8. Are logs indexed?
9. Is Kibana querying the correct index/data view?
```

---

# 84. Troubleshooting Collector Logs

Check:

```bash
kubectl get pods -n observability
```

Then:

```bash
kubectl logs <collector-pod> -n observability
```

Look for:

```text
receiver errors
exporter errors
authentication failures
connection refused
TLS errors
memory pressure
configuration errors
```

---

# 85. Troubleshooting Filelog Receiver

If Kubernetes logs are not collected, verify:

```text
File paths
Host filesystem mount
Container runtime format
File permissions
Include/exclude rules
Collector DaemonSet
```

A Collector running without access to the node's log files cannot collect those files.

---

# 86. Troubleshooting OTLP Logs

If applications send logs through OTLP:

```text
Application
     ↓
OTLP
     X
Collector
```

Check:

```text
Endpoint
Port
Protocol
TLS
Authentication
NetworkPolicy
Collector receiver
```

---

# 87. Troubleshooting Missing Trace IDs

If logs arrive but have no trace ID:

```text
Log
  X trace_id
```

Check:

```text
Context propagation
Active span
Logging integration
OTel instrumentation
Log bridge
```

The application must have the current tracing context available when the log is emitted.

---

# 88. Troubleshooting Wrong Service Name

If all logs appear as:

```text
service.name = application
```

instead of:

```text
orders
payment
inventory
```

check:

```text
OTEL_SERVICE_NAME
Resource configuration
Collector resource processor
Deployment environment
Instrumentation configuration
```

---

# 89. Troubleshooting Duplicate Logs

Possible causes:

```text
Application logging twice
Multiple collectors
Duplicate filelog receivers
Multiple exporters
Log forwarding plus OTLP
```

Architecture problem:

```text
Application
   ├──→ stdout → Collector
   └──→ OTLP → Collector
```

If both represent the same event, duplicate records may appear.

---

# 90. Troubleshooting High Log Volume

Symptoms:

```text
Elasticsearch storage ↑
Network ↑
Collector CPU ↑
Collector memory ↑
```

Investigate:

```text
DEBUG enabled
Health-check logs
Repeated errors
Large stack traces
Verbose libraries
Duplicate collection
```

Then filter or correct the source.

---

# 91. Logging During Deployments

During deployment, logs can identify:

```text
Application startup failures
Configuration errors
Dependency failures
Database connection problems
Readiness failures
Authentication errors
```

Combine:

```text
Kubernetes events
Metrics
Logs
Traces
```

for faster diagnosis.

---

# 92. Logging During Rollbacks

Suppose a new version starts producing:

```text
ERROR database schema mismatch
```

The deployment can be rolled back.

```text
Git
 ↓
Previous version
 ↓
ArgoCD
 ↓
EKS
```

Logs help confirm whether the rollback resolved the problem.

---

# 93. GitOps and Logging Configuration

Logging configuration should also be version controlled.

```text
GitHub
   ↓
Collector Config
   ↓
Pull Request
   ↓
Review
   ↓
CI
   ↓
ArgoCD
   ↓
EKS
```

This provides:

```text
Auditability
Review
Rollback
Consistency
Drift detection
```

---

# 94. Environment-Specific Logging

Development:

```text
DEBUG
INFO
WARN
ERROR
```

Production:

```text
INFO
WARN
ERROR
```

The exact levels depend on the application's requirements.

Avoid enabling excessive DEBUG logging in production unless temporarily required for troubleshooting.

---

# 95. Production Logging Strategy

A good production strategy:

```text
Normal operation
      ↓
INFO

Potential issue
      ↓
WARN

Failed operation
      ↓
ERROR

Critical failure
      ↓
FATAL
```

Each log should contain enough context to explain what happened.

---

# 96. Production Log Fields

Recommended fields include:

```text
timestamp
severity
message
service.name
service.version
deployment.environment
trace_id
span_id
error.type
error.message
k8s.namespace.name
k8s.pod.name
```

Only include fields that provide operational value.

---

# 97. Production Logging Architecture

```text
                         EKS
                          │
       ┌──────────────────┼──────────────────┐
       ↓                  ↓                  ↓
    Orders             Payment            Inventory
       │                  │                  │
    stdout             stdout             stdout
       │                  │                  │
       └──────────────────┼──────────────────┘
                          ↓
                    OTel Agent
                          ↓
                   Log Processing
                          ↓
                    OTel Gateway
                          ↓
                       Logstash
                          ↓
                    Elasticsearch
                          ↓
                       Kibana
```

---

# 98. Unified Observability Architecture

```text
                           EKS
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
          Metrics          Logs          Traces
             │              │              │
             └──────────────┼──────────────┘
                            ↓
                       OTel Agent
                            ↓
                       OTel Gateway
                            │
             ┌──────────────┼──────────────┐
             ↓              ↓              ↓
         Prometheus         ELK          Trace Backend
             ↓              ↓              ↓
          Grafana         Kibana       Trace UI
```

The common identifiers are:

```text
service.name
service.version
deployment.environment
trace_id
span_id
```

---

# 99. Production Logging Checklist

```text
[ ] Structured logging enabled
[ ] Appropriate severity levels
[ ] Service name configured
[ ] Service version configured
[ ] Environment configured
[ ] Kubernetes metadata available
[ ] Trace ID correlation enabled
[ ] Span ID correlation enabled
[ ] Sensitive data excluded
[ ] PII controlled
[ ] DEBUG volume controlled
[ ] Health-check noise controlled
[ ] Log rotation configured where needed
[ ] Collector logs pipeline configured
[ ] ELK integration verified
[ ] Retention configured
[ ] Log storage monitored
[ ] Collector monitored
[ ] Alerts configured
[ ] GitOps configuration management
```

---

# 100. EKS Production Checklist

```text
[ ] Application logs to stdout/stderr
[ ] OTel Agent deployed as required
[ ] Node log access configured
[ ] Filelog receiver configured if needed
[ ] OTLP receiver configured if required
[ ] Gateway deployed
[ ] Kubernetes metadata enrichment configured
[ ] Resource limits configured
[ ] Memory limiter configured
[ ] Filtering configured
[ ] Batch processor configured
[ ] TLS configured
[ ] RBAC reviewed
[ ] NetworkPolicies reviewed
[ ] ELK connectivity verified
[ ] Kibana queries verified
[ ] Trace correlation verified
[ ] Log volume monitored
[ ] High-cardinality fields reviewed
```

---

# 101. Complete Log Troubleshooting Flow

```text
Incident
   ↓
Grafana
   ↓
Metric indicates increased errors
   ↓
Kibana
   ↓
Search service + severity + time
   ↓
Find relevant log
   ↓
Extract trace_id
   ↓
Trace backend
   ↓
Follow distributed request
   ↓
Identify failing dependency
   ↓
Return to logs
   ↓
Confirm root cause
```

---

# 102. Final Mental Model

Remember OpenTelemetry Logs as:

```text
GENERATE
   ↓
Log Record
   ↓
ENRICH
   ↓
Resource + Attributes
   ↓
CORRELATE
   ↓
Trace ID + Span ID
   ↓
PROCESS
   ↓
Filter + Transform + Batch
   ↓
EXPORT
   ↓
Collector
   ↓
ELK
   ↓
Kibana
```

For a production EKS microservices platform:

```text
Application
     ↓
stdout / OTel Logs
     ↓
OTel Agent
     ↓
OTel Gateway
     ↓
Filtering + Enrichment + Batching
     ↓
Logstash / Elasticsearch
     ↓
Kibana
```

The key principle is:

**OpenTelemetry Logs standardizes application logging as part of a unified observability model. In a production Kubernetes environment, logs should be structured, enriched with service and Kubernetes metadata, correlated with trace context, protected from sensitive-data leakage, collected through scalable Collector agents and gateways, and delivered to ELK for centralized search and analysis. Metrics identify that something is wrong, logs provide detailed event context, and traces show how the request moved through the distributed system.**
