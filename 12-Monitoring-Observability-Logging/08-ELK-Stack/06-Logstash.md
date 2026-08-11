# Logstash

## 1. Overview

Logstash is a server-side data processing pipeline used to collect, process, transform, enrich, and forward data to one or more destinations.

In the ELK stack:

```text
Application / Infrastructure
            ↓
         Logstash
            ↓
      Elasticsearch
            ↓
          Kibana
```

Logstash is especially useful when raw logs need to be:

```text
Collected
Parsed
Filtered
Transformed
Enriched
Normalized
Routed
```

before they are indexed into Elasticsearch.

---

# 2. Logstash in the ELK Stack

The basic responsibility of each component is:

```text
Logstash
   ↓
Collect + Process + Transform

Elasticsearch
   ↓
Index + Store + Search

Kibana
   ↓
Visualize + Analyze
```

A typical production flow:

```text
Application
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

# 3. Why Logstash?

Applications generate logs in different formats.

For example:

```text
2026-08-11 10:30:15 ERROR Payment failed
```

Another application may generate:

```json
{
  "service": "payment",
  "level": "ERROR",
  "message": "Payment failed"
}
```

Another may generate:

```text
payment|ERROR|500|database timeout
```

Logstash can normalize these different formats into a consistent structure.

---

# 4. Logstash Mental Model

The most important concept is:

```text
INPUT
  ↓
FILTER
  ↓
OUTPUT
```

Example:

```text
Logs
 ↓
Input
 ↓
Grok
 ↓
Mutate
 ↓
Date
 ↓
JSON
 ↓
Elasticsearch
```

Remember:

**Input receives data, filters process data, and outputs send data somewhere.**

---

# 5. Logstash Pipeline

A Logstash pipeline consists of:

```text
Input
   ↓
Filter
   ↓
Output
```

Example:

```text
Application Logs
       ↓
   Beats Input
       ↓
      Grok
       ↓
     Mutate
       ↓
      Date
       ↓
Elasticsearch Output
```

---

# 6. Logstash Architecture

A production architecture can look like:

```text
                         EKS
                          │
                   Application Pods
                          │
                          ↓
                    Log Collector
                          │
                          ↓
                 ┌─────────────────┐
                 │    Logstash     │
                 │     Cluster     │
                 └─────────────────┘
                    │           │
                    ↓           ↓
                  LS-01       LS-02
                    │           │
                    └─────┬─────┘
                          ↓
              Elasticsearch Cluster
```

Logstash can run multiple pipelines and multiple instances.

---

# 7. Logstash Event

Logstash processes events.

An event can conceptually look like:

```json
{
  "message": "Payment failed",
  "service": "payment",
  "level": "ERROR"
}
```

Logstash can modify this event before sending it to Elasticsearch.

---

# 8. Logstash Event Lifecycle

Example:

```text
Raw Log
   ↓
Logstash Event
   ↓
Parse
   ↓
Normalize
   ↓
Enrich
   ↓
Output
```

For example:

```text
Raw:
"2026-08-11 ERROR payment failed"

After processing:

{
  "@timestamp": "...",
  "level": "ERROR",
  "service": "payment",
  "message": "payment failed"
}
```

---

# 9. Input Plugins

Inputs define where Logstash receives data.

Common inputs include:

```text
Beats
TCP
UDP
HTTP
File
Kafka
Syslog
```

For a Kubernetes logging architecture, a common pattern is:

```text
Fluent Bit / Filebeat
          ↓
       Logstash
```

---

# 10. Beats Input

A common Logstash input is the Beats input.

Example:

```ruby
input {
  beats {
    port => 5044
  }
}
```

This allows Beats-compatible shippers to send events to Logstash.

Architecture:

```text
Filebeat
   ↓
TCP 5044
   ↓
Logstash
```

---

# 11. TCP Input

Logstash can receive events over TCP.

Example:

```ruby
input {
  tcp {
    port => 5000
  }
}
```

This can be useful when an application or log collector sends data directly over TCP.

---

# 12. UDP Input

Logstash can also receive UDP traffic.

Example:

```ruby
input {
  udp {
    port => 5000
  }
}
```

UDP is connectionless and does not provide the same delivery guarantees as TCP.

Use it only when the application/protocol is appropriate for UDP.

---

# 13. HTTP Input

Logstash can receive HTTP requests.

Example:

```ruby
input {
  http {
    port => 8080
  }
}
```

Architecture:

```text
Application
     ↓
HTTP
     ↓
Logstash
```

Authentication, TLS, rate limiting, and network restrictions should be considered for production use.

---

# 14. File Input

Logstash can read local files.

Example:

```ruby
input {
  file {
    path => "/var/log/application.log"
    start_position => "beginning"
  }
}
```

However, in Kubernetes environments, a dedicated log collector is often a better fit for collecting container logs.

---

# 15. Kafka Input

For large environments, Kafka can be placed between producers and Logstash.

Architecture:

```text
Applications
     ↓
Kafka
     ↓
Logstash
     ↓
Elasticsearch
```

This introduces buffering and decouples producers from Logstash.

---

# 16. Syslog Input

Logstash can receive syslog events.

Architecture:

```text
Network Devices
Servers
Security Devices
       ↓
     Syslog
       ↓
    Logstash
```

This is useful for infrastructure and network logging.

---

# 17. Filter Plugins

Filters transform events.

Common filters include:

```text
Grok
Mutate
Date
JSON
Dissect
GeoIP
Drop
Translate
Ruby
```

The most commonly discussed filters for interview and real-world logging work are:

```text
Grok
Mutate
Date
JSON
Dissect
```

---

# 18. Grok

Grok extracts structured fields from unstructured text.

Input:

```text
2026-08-11 10:30:15 ERROR Payment failed
```

A Grok pattern can extract:

```text
timestamp
level
message
```

Conceptually:

```text
Raw Log
  ↓
Grok
  ↓
Structured Event
```

---

# 19. Grok Example

Example:

```ruby
filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:log_timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message_text}"
    }
  }
}
```

The resulting event can contain:

```text
log_timestamp
level
message_text
```

The exact pattern must match the application's actual log format.

---

# 20. Why Grok?

Without parsing:

```text
message =
"2026-08-11 10:30:15 ERROR Payment failed"
```

With parsing:

```text
timestamp = 2026-08-11 10:30:15
level     = ERROR
message   = Payment failed
```

Structured fields make Elasticsearch queries and Kibana visualizations much easier.

---

# 21. Mutate Filter

The mutate filter modifies fields.

Common operations include:

```text
rename
replace
update
add_field
remove_field
convert
lowercase
uppercase
```

Example:

```ruby
filter {
  mutate {
    add_field => {
      "environment" => "production"
    }
  }
}
```

---

# 22. Rename Fields

Example:

```ruby
filter {
  mutate {
    rename => {
      "host" => "server"
    }
  }
}
```

This can normalize fields from different sources.

---

# 23. Remove Fields

Unnecessary fields can be removed.

Example:

```ruby
filter {
  mutate {
    remove_field => [
      "agent",
      "input"
    ]
  }
}
```

Be careful when removing fields because some may be useful for troubleshooting.

---

# 24. Convert Fields

For example:

```ruby
filter {
  mutate {
    convert => {
      "status_code" => "integer"
    }
  }
}
```

This makes the field suitable for numeric operations and aggregations.

---

# 25. Date Filter

The date filter parses timestamps and sets the event timestamp.

Example:

```ruby
filter {
  date {
    match => [
      "log_timestamp",
      "ISO8601"
    ]
  }
}
```

This is important because Elasticsearch/Kibana rely heavily on correct timestamps.

---

# 26. Why Timestamp Parsing Matters

Suppose the actual event occurred at:

```text
10:30
```

but Logstash indexed it at:

```text
10:35
```

If the original timestamp is not preserved correctly, Kibana can show the event at the wrong time.

Therefore:

```text
Application Timestamp
        ↓
Logstash Date Filter
        ↓
@timestamp
```

---

# 27. JSON Filter

If the application already sends JSON inside the message field:

```json
{
  "service": "payment",
  "level": "ERROR",
  "message": "Payment failed"
}
```

Logstash can parse it.

Example:

```ruby
filter {
  json {
    source => "message"
  }
}
```

This converts JSON content into structured fields.

---

# 28. Dissect Filter

Dissect is useful when logs follow a predictable delimiter-based format.

Example:

```text
payment|ERROR|500|database timeout
```

A Dissect pattern can extract:

```text
service
level
status
message
```

Dissect is often simpler and faster than Grok for predictable formats.

---

# 29. Grok vs Dissect

| Feature             | Grok      | Dissect         |
| ------------------- | --------- | --------------- |
| Pattern matching    | Flexible  | Delimiter-based |
| Complex formats     | Excellent | Limited         |
| Predictable formats | Good      | Excellent       |
| Processing overhead | Higher    | Generally lower |
| Regex-based         | Yes       | No              |

Use the simplest parser that reliably handles the input.

---

# 30. Conditional Processing

Logstash can process different event types differently.

Example:

```ruby
filter {
  if [service] == "payment" {
    mutate {
      add_field => {
        "team" => "payments"
      }
    }
  }
}
```

This allows service-specific processing.

---

# 31. Conditional Routing

Events can also be routed to different outputs.

Example:

```ruby
output {
  if [environment] == "production" {
    elasticsearch {
      hosts => ["https://elasticsearch-prod:9200"]
    }
  } else {
    elasticsearch {
      hosts => ["https://elasticsearch-staging:9200"]
    }
  }
}
```

Use routing carefully to avoid sending sensitive production data to the wrong environment.

---

# 32. Output Plugins

Outputs define where events go.

Common outputs include:

```text
Elasticsearch
Kafka
File
HTTP
S3
Stdout
```

For ELK:

```ruby
output {
  elasticsearch {
    hosts => ["https://elasticsearch:9200"]
  }
}
```

---

# 33. Elasticsearch Output

Typical architecture:

```text
Logstash
   ↓
Elasticsearch Output
   ↓
Elasticsearch
```

Example:

```ruby
output {
  elasticsearch {
    hosts => ["https://es-prod:9200"]
    index => "logs-%{[service]}-%{+YYYY.MM.dd}"
  }
}
```

The exact index strategy should be designed carefully for the Elasticsearch version and lifecycle strategy.

---

# 34. Stdout Output

During development, stdout is extremely useful.

Example:

```ruby
output {
  stdout {
    codec => rubydebug
  }
}
```

This allows you to see how Logstash transformed the event.

---

# 35. Rubydebug

Example output:

```text
{
       "service" => "payment",
          "level" => "ERROR",
        "message" => "Payment failed"
}
```

This is useful when developing and troubleshooting pipelines.

Do not rely on verbose debug output as your normal production output.

---

# 36. Complete Basic Pipeline

Example:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => {
      "message" => "%{LOGLEVEL:level} %{GREEDYDATA:message_text}"
    }
  }

  mutate {
    add_field => {
      "environment" => "production"
    }
  }
}

output {
  elasticsearch {
    hosts => ["https://elasticsearch:9200"]
    index => "logs-production"
  }

  stdout {
    codec => rubydebug
  }
}
```

This demonstrates:

```text
Input
 ↓
Grok
 ↓
Mutate
 ↓
Elasticsearch
```

---

# 37. Pipeline Configuration

Logstash pipeline configuration is commonly managed through:

```text
pipelines.yml
```

A pipeline can reference one or more configuration files.

Conceptually:

```text
pipelines.yml
      ↓
Pipeline
      ↓
Input
Filter
Output
```

---

# 38. Multiple Pipelines

Large environments may have multiple pipelines.

Example:

```text
Pipeline 1
   ↓
Application Logs

Pipeline 2
   ↓
Security Logs

Pipeline 3
   ↓
Infrastructure Logs
```

This can isolate workloads.

---

# 39. Multi-Pipeline Architecture

```text
                 Logstash
                    │
       ┌────────────┼────────────┐
       ↓            ↓            ↓
   Pipeline A   Pipeline B   Pipeline C
       ↓            ↓            ↓
   Application   Security   Infrastructure
```

This is useful when different data streams require different processing.

---

# 40. Pipeline Isolation

Separate pipelines can provide:

```text
Independent configuration
Independent processing
Independent scaling considerations
Reduced configuration complexity
```

For large installations, this can be easier to operate than one massive pipeline.

---

# 41. Logstash Configuration Directory

Package installations commonly use:

```text
/etc/logstash/
```

Typical files/directories include:

```text
pipelines.yml
logstash.yml
jvm.options
conf.d/
```

The exact layout depends on the installation method and version.

---

# 42. `logstash.yml`

The main Logstash settings can be configured in:

```text
/etc/logstash/logstash.yml
```

Common configuration areas include:

```text
Node identity
Pipeline configuration
API settings
Logging
Monitoring
Queue behavior
```

---

# 43. `pipelines.yml`

Multiple pipelines can be defined in:

```text
/etc/logstash/pipelines.yml
```

Conceptual example:

```yaml
- pipeline.id: application
  path.config: "/etc/logstash/conf.d/application/*.conf"

- pipeline.id: security
  path.config: "/etc/logstash/conf.d/security/*.conf"
```

This is an example structure; exact paths should match your environment.

---

# 44. Configuration Directory Structure

A clean structure can be:

```text
/etc/logstash/
│
├── logstash.yml
├── pipelines.yml
│
└── conf.d/
    ├── application/
    │   ├── input.conf
    │   ├── filter.conf
    │   └── output.conf
    │
    └── security/
        ├── input.conf
        ├── filter.conf
        └── output.conf
```

This becomes easier to maintain than one huge configuration file.

---

# 45. Logstash Data Directory

Logstash may maintain local state and queue-related data depending on configuration.

Common package locations include:

```text
/var/lib/logstash/
```

Storage requirements depend on whether persistent queues are enabled.

---

# 46. Logstash Logs

Package installations commonly store Logstash logs under:

```text
/var/log/logstash/
```

These logs help troubleshoot:

```text
Pipeline errors
Plugin failures
Connection failures
Parsing errors
Elasticsearch output errors
```

---

# 47. Logstash API

Logstash provides an API that can expose operational information.

A commonly used endpoint is:

```text
http://localhost:9600
```

For example:

```bash
curl http://localhost:9600
```

The API can provide information about:

```text
Pipelines
Events
Plugins
Node information
```

Protect the API appropriately in production.

---

# 48. Logstash Monitoring

Monitor:

```text
Events in
Events out
Pipeline throughput
Pipeline failures
Queue size
CPU
Memory
JVM
Persistent queue
Elasticsearch output failures
```

Architecture:

```text
Logstash
   ↓
Metrics
   ↓
Prometheus-compatible monitoring / monitoring solution
   ↓
Grafana
```

The exact integration depends on your chosen monitoring approach.

---

# 49. Logstash Performance

Performance depends on:

```text
Input rate
Filter complexity
Grok patterns
Event size
Output latency
Elasticsearch capacity
CPU
Memory
Pipeline workers
Batch size
Persistent queue
```

Do not tune blindly.

Measure first.

---

# 50. Pipeline Workers

Logstash can process events using pipeline workers.

Conceptually:

```text
Events
 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

More workers can increase parallel processing when the workload supports it.

But excessive concurrency can increase CPU and downstream pressure.

---

# 51. Pipeline Batch Size

Logstash processes events in batches.

Conceptually:

```text
Events
 ↓
Batch
 ↓
Workers
 ↓
Output
```

Batch sizing affects:

```text
Throughput
Latency
Memory
Elasticsearch bulk behavior
```

Tune based on actual workload.

---

# 52. Persistent Queue

Persistent queues allow Logstash to retain events on disk when downstream systems are unavailable or when buffering is needed.

Architecture:

```text
Input
 ↓
Persistent Queue
 ↓
Processing
 ↓
Elasticsearch
```

Without sufficient buffering:

```text
Elasticsearch unavailable
        ↓
Logstash cannot deliver
        ↓
Events may be lost depending on the input/queue architecture
```

Persistent queues improve resilience, but they are not a replacement for an end-to-end durable messaging architecture.

---

# 53. Persistent Queue Architecture

```text
                 Logstash
                    │
             ┌──────┴──────┐
             ↓             ↓
           Input      Persistent Queue
                           │
                           ↓
                        Output
                           │
                           ↓
                    Elasticsearch
```

If Elasticsearch is temporarily unavailable, queued events can wait for recovery depending on queue capacity.

---

# 54. Persistent Queue Storage

Persistent queue data requires disk.

Therefore:

```text
Persistent Queue
       ↓
Disk
       ↓
Capacity Planning
```

Monitor queue disk usage.

A full persistent queue can eventually stop accepting more events.

---

# 55. Dead Letter Queue

Logstash can use a dead letter queue for events that cannot be processed under certain conditions.

Conceptually:

```text
Input
 ↓
Filter
 ↓
Processing Error
 ↓
Dead Letter Queue
```

This can help investigate problematic events without continuously failing the main pipeline.

---

# 56. Backpressure

Suppose:

```text
Input:
50,000 events/sec

Elasticsearch:
20,000 events/sec
```

Then:

```text
Incoming
   ↓
Logstash
   ↓
Queue grows
   ↓
Backpressure
```

If the mismatch continues, storage and memory can become exhausted.

---

# 57. Logstash and Elasticsearch Capacity

Logstash does not eliminate Elasticsearch bottlenecks.

Example:

```text
Applications
      ↓
Logstash
      ↓
Elasticsearch X
```

If Elasticsearch cannot keep up:

```text
Queue
  ↑
  grows
```

Therefore both systems must be capacity planned.

---

# 58. Logstash High Availability

A production architecture can use multiple Logstash nodes:

```text
              Log Collectors
                    │
          ┌─────────┴─────────┐
          ↓                   ↓
       Logstash-1          Logstash-2
          │                   │
          └─────────┬─────────┘
                    ↓
          Elasticsearch Cluster
```

This prevents a single Logstash instance from becoming a single point of failure.

---

# 59. Load Balancing Logstash

A collector can distribute events across Logstash nodes:

```text
Collector
   │
   ├────→ Logstash-1
   │
   └────→ Logstash-2
```

Alternatively, a load balancer can distribute traffic.

---

# 60. Logstash Persistent Queue in HA

Each Logstash node has its own local persistent queue.

Therefore:

```text
Logstash-1 → Local Queue
Logstash-2 → Local Queue
```

A persistent queue on one node does not automatically provide distributed durability.

For stronger durability and replay requirements, a distributed messaging system such as Kafka may be appropriate.

---

# 61. Logstash + Kafka

Large environments may use:

```text
Applications
     ↓
Collectors
     ↓
Kafka
     ↓
Logstash
     ↓
Elasticsearch
```

Kafka provides durable buffering and decoupling.

---

# 62. Why Kafka?

Kafka can help when:

```text
Log volume is high
Consumers need replay
Multiple consumers exist
Temporary downstream failures occur
Decoupling is required
```

Architecture:

```text
Producers
   ↓
Kafka
   ↓
Logstash
   ↓
Elasticsearch
```

---

# 63. Logstash + Kubernetes

A common EKS architecture:

```text
Kubernetes Pods
      ↓
Container Logs
      ↓
Fluent Bit
      ↓
Logstash
      ↓
Elasticsearch
```

Fluent Bit handles lightweight collection.

Logstash handles more complex transformation and routing.

---

# 64. Why Fluent Bit + Logstash?

Fluent Bit is lightweight and suitable for node-level collection.

Logstash is more powerful for complex processing.

Therefore:

```text
Fluent Bit
   ↓
Collect
Forward
Basic enrichment

Logstash
   ↓
Complex parsing
Transformation
Routing
Enrichment

Elasticsearch
   ↓
Store/Search
```

This separation works well for large Kubernetes environments.

---

# 65. Kubernetes Log Flow

```text
                    EKS
                     │
        ┌────────────┼────────────┐
        ↓            ↓            ↓
      Node 1       Node 2       Node 3
        │            │            │
     Fluent Bit  Fluent Bit  Fluent Bit
        │            │            │
        └────────────┼────────────┘
                     ↓
                 Logstash
                     │
                     ↓
              Elasticsearch
```

---

# 66. Kubernetes Metadata

Logstash can process metadata such as:

```text
Cluster
Namespace
Pod
Container
Node
Deployment
Service
```

Example:

```json
{
  "cluster": "prod-eks",
  "namespace": "production",
  "pod": "payment-7d89f",
  "container": "payment",
  "service": "payment"
}
```

This makes Kubernetes log investigation much easier.

---

# 67. Structured Logging

Prefer applications to emit structured logs.

Example:

```json
{
  "timestamp": "2026-08-11T10:30:15Z",
  "service": "payment",
  "level": "ERROR",
  "message": "Payment failed",
  "order_id": "ORD-1001",
  "status_code": 500
}
```

Then Logstash may need less parsing.

---

# 68. Structured Logs vs Grok

Bad:

```text
"2026-08-11 ERROR payment failed order=ORD-1001 status=500"
```

Better:

```json
{
  "level": "ERROR",
  "service": "payment",
  "order_id": "ORD-1001",
  "status_code": 500
}
```

The second format reduces parsing complexity.

---

# 69. Logstash as a Transformation Layer

Think of Logstash as:

```text
Raw Data
   ↓
Normalization
   ↓
Standard Schema
   ↓
Elasticsearch
```

For example:

```text
service_name
service
app
application
```

can be normalized to:

```text
service
```

---

# 70. Field Normalization

Example:

```ruby
filter {
  mutate {
    rename => {
      "app_name" => "[service][name]"
    }
  }
}
```

The exact schema depends on your logging standard.

The goal is consistency.

---

# 71. Enrichment

Logstash can enrich events with additional information.

Example:

```text
IP Address
   ↓
GeoIP
   ↓
Country
City
Coordinates
```

This can be useful for:

```text
Security logs
Web traffic
Network logs
```

Use enrichment selectively because it increases processing cost.

---

# 72. GeoIP

Conceptually:

```ruby
filter {
  geoip {
    source => "client.ip"
  }
}
```

This can add geographic metadata.

The exact fields depend on the plugin/version and available GeoIP database.

---

# 73. Sensitive Data

Never blindly index:

```text
Passwords
API keys
Access tokens
Private keys
Credit card data
Session secrets
```

Logstash can remove or mask sensitive fields.

Example:

```ruby
filter {
  mutate {
    remove_field => ["password", "token"]
  }
}
```

Application-level prevention is even better.

---

# 74. Data Masking

For sensitive information:

```text
Original:
password=Secret123

Processed:
password=[REDACTED]
```

Use filtering before data reaches Elasticsearch.

But do not rely only on Logstash for compliance; prevent sensitive data from being logged at the application source whenever possible.

---

# 75. Logstash Routing

Example:

```text
Security Logs
      ↓
Security Index

Application Logs
      ↓
Application Index

Infrastructure Logs
      ↓
Infrastructure Index
```

Conditional outputs can implement this routing.

---

# 76. Example Routing Pipeline

```ruby
output {
  if [log_type] == "security" {
    elasticsearch {
      hosts => ["https://es-prod:9200"]
      index => "security-logs"
    }
  } else if [log_type] == "application" {
    elasticsearch {
      hosts => ["https://es-prod:9200"]
      index => "application-logs"
    }
  }
}
```

Design index strategy carefully around retention and lifecycle management.

---

# 77. Logstash Error Handling

A production pipeline should consider:

```text
Parsing failures
Elasticsearch unavailable
Authentication failures
Malformed events
Queue exhaustion
Network failures
```

Monitor these conditions.

---

# 78. Elasticsearch Connection Failure

If Elasticsearch becomes unavailable:

```text
Logstash
   ↓
Elasticsearch X
```

Depending on the pipeline and queue configuration:

```text
Retry
 ↓
Queue
 ↓
Backpressure
```

Persistent queues can help absorb temporary failures.

---

# 79. Logstash Retry Behavior

The Elasticsearch output can retry failed delivery.

The exact behavior depends on the plugin and failure type.

Operationally:

```text
Elasticsearch unavailable
        ↓
Output failure
        ↓
Retry / queue
        ↓
Elasticsearch recovers
        ↓
Events delivered
```

Monitor repeated failures rather than assuming retries will solve a persistent problem.

---

# 80. Pipeline-to-Pipeline Communication

Logstash can route events between pipelines.

Conceptually:

```text
Input Pipeline
      ↓
Processing Pipeline
      ↓
Output Pipeline
```

This can help organize complex architectures.

---

# 81. Pipeline Architecture

Example:

```text
                 Logstash
                    │
           ┌────────┴────────┐
           ↓                 ↓
       Input Pipeline    Input Pipeline
           │                 │
           ↓                 ↓
     Application         Security
           │                 │
           └────────┬────────┘
                    ↓
                 Outputs
```

Use multiple pipelines when they genuinely improve isolation and maintainability.

---

# 82. Logstash Configuration Testing

Before production deployment:

```text
Configuration
      ↓
Syntax Validation
      ↓
Test Events
      ↓
Output Validation
      ↓
Performance Test
```

Do not deploy an untested Grok pattern directly to production.

---

# 83. Configuration Validation

Logstash provides command-line options for validating configuration.

A common pattern is:

```bash
bin/logstash --config.test_and_exit -f /path/to/pipeline.conf
```

The exact command may vary based on the installation and version.

This helps catch configuration errors before starting the pipeline.

---

# 84. Test Pipeline

A simple test pipeline:

```ruby
input {
  stdin {}
}

filter {
  mutate {
    add_field => {
      "environment" => "test"
    }
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

Start Logstash and enter:

```text
hello
```

You should see the processed event.

---

# 85. Development Testing

Test each stage separately:

```text
Input
 ↓
Does Logstash receive events?

Filter
 ↓
Are fields parsed correctly?

Output
 ↓
Does Elasticsearch receive events?

Kibana
 ↓
Can users search the events?
```

This makes debugging much easier.

---

# 86. Logstash Debugging Flow

When logs are missing:

```text
Application
   ↓
Collector
   ↓
Logstash Input
   ↓
Filter
   ↓
Output
   ↓
Elasticsearch
   ↓
Kibana
```

Check each boundary.

---

# 87. Input Troubleshooting

If Logstash receives nothing:

```text
Check:
Port
Network
Firewall
Collector
Input configuration
TLS
Authentication
```

Example:

```bash
ss -lntp | grep 5044
```

---

# 88. Filter Troubleshooting

If fields are missing:

```text
Check:
Grok pattern
Field name
JSON structure
Dissect pattern
Conditional logic
```

Use:

```ruby
stdout {
  codec => rubydebug
}
```

during development.

---

# 89. Output Troubleshooting

If Elasticsearch receives nothing:

```text
Check:
Elasticsearch URL
DNS
Port
TLS
Credentials
Permissions
Index configuration
Cluster health
```

Test connectivity independently.

---

# 90. Logstash Monitoring Metrics

Important operational metrics:

```text
Events in
Events out
Events filtered
Pipeline throughput
Queue size
CPU
Memory
JVM
Output failures
Elasticsearch response time
```

Track these over time.

---

# 91. Logstash Alerts

Useful alerts include:

```text
Logstash process down
Pipeline stopped
Events in = 0 unexpectedly
Events out drops
Queue grows rapidly
Persistent queue nearly full
Elasticsearch output failures
CPU saturation
Memory pressure
```

These alerts should be integrated into your observability system.

---

# 92. Prometheus + Logstash

Your monitoring architecture can be:

```text
Logstash
   ↓
Metrics
   ↓
Prometheus
   ↓
Grafana
```

Grafana can show:

```text
Events/sec
Queue size
CPU
Memory
Pipeline health
Output failures
```

The exact metrics collection mechanism depends on the Logstash monitoring integration you choose.

---

# 93. Logstash + Elasticsearch + Kibana

The complete logging flow:

```text
             APPLICATIONS
                  │
                  ↓
             LOG COLLECTOR
                  │
                  ↓
               LOGSTASH
          ┌───────┴───────┐
          ↓               ↓
       FILTERS          ROUTING
          │               │
          └───────┬───────┘
                  ↓
            ELASTICSEARCH
                  │
                  ↓
                KIBANA
```

---

# 94. Logstash Production Architecture

For your EKS environment:

```text
                         EKS
                          │
                ┌─────────┴─────────┐
                ↓                   ↓
             Services             Pods
                │                   │
                └─────────┬─────────┘
                          ↓
                     Fluent Bit
                          │
                          ↓
                ┌─────────────────┐
                │    Logstash     │
                │      HA         │
                └─────────────────┘
                   │           │
                   ↓           ↓
                 LS-01       LS-02
                   │           │
                   └─────┬─────┘
                         ↓
              Elasticsearch Cluster
                         │
                         ↓
                       Kibana
```

---

# 95. Logstash Security

Production Logstash should use:

```text
TLS
Authentication
Least privilege
Private networking
Protected API
Secret management
```

Avoid:

```text
Internet
   ↓
Logstash
```

unless there is a very specific and secured architecture.

---

# 96. TLS Between Logstash and Elasticsearch

Architecture:

```text
Logstash
   ↓
HTTPS/TLS
   ↓
Elasticsearch
```

Logstash needs to trust the Elasticsearch certificate chain.

Credentials should be stored securely rather than hard-coded into Git.

---

# 97. Logstash API Security

The Logstash API exposes operational information.

Do not leave administrative interfaces unnecessarily exposed.

Restrict access through:

```text
Private network
Firewall
Security groups
Network policies
Authentication where supported
```

---

# 98. Logstash Resource Planning

Consider:

```text
Events/sec
Average event size
Filter complexity
Grok complexity
Output latency
Queue requirements
CPU
Memory
Disk
```

Example:

```text
100,000 events/sec
```

requires very different architecture from:

```text
1,000 events/sec
```

Do not size based only on server count.

---

# 99. Horizontal Scaling

If one Logstash instance cannot keep up:

```text
                Load Balancer
                     │
             ┌───────┴───────┐
             ↓               ↓
         Logstash-1      Logstash-2
             │               │
             └───────┬───────┘
                     ↓
              Elasticsearch
```

Horizontal scaling is often preferable to simply making one node extremely large.

---

# 100. Vertical Scaling

You can also increase:

```text
CPU
Memory
Disk
```

for a Logstash node.

But first identify the bottleneck.

Example:

```text
CPU 100%
 ↓
Increase CPU / workers

Memory pressure
 ↓
Increase memory / review pipeline

Disk queue full
 ↓
Increase queue storage
```

---

# 101. Logstash and Backpressure

A healthy pipeline:

```text
Input rate
   ≈
Processing rate
   ≈
Output rate
```

Problem:

```text
Input rate
   >
Output rate
```

Then:

```text
Queue ↑
Latency ↑
Storage ↑
```

If this continues, the pipeline becomes unstable.

---

# 102. Logstash Pipeline Design Principles

Use:

```text
Simple pipelines
Structured logs
Efficient parsing
Minimal unnecessary transformations
Controlled routing
Persistent buffering where needed
Monitoring
```

Avoid:

```text
Huge monolithic pipeline
Unnecessary Grok
Repeated transformations
Uncontrolled field creation
Hard-coded secrets
```

---

# 103. Grok Performance

Complex Grok expressions can consume significant CPU.

Bad approach:

```text
One extremely complex Grok pattern
```

Better:

```text
Use structured logging
Use Dissect when format is predictable
Parse only required fields
Avoid unnecessary regex
```

---

# 104. Logstash and Structured Logging

Best architecture:

```text
Application
   ↓
JSON logs
   ↓
Fluent Bit
   ↓
Logstash
   ↓
Minimal transformation
   ↓
Elasticsearch
```

This reduces processing overhead.

---

# 105. Logstash and Trace IDs

For distributed tracing, preserve:

```text
trace.id
span.id
```

Example:

```json
{
  "service": "payment",
  "level": "ERROR",
  "trace.id": "abc123",
  "span.id": "def456"
}
```

Then:

```text
Kibana
  ↓
trace.id
  ↓
Jaeger
```

This connects logs with distributed traces.

---

# 106. Logstash and Metrics

Metrics and logs serve different purposes.

Prometheus:

```text
payment_5xx_rate
```

Logstash/Elasticsearch:

```text
Database connection timeout
```

Tracing:

```text
Order → Payment → Database
```

Together:

```text
Metrics
   +
Logs
   +
Traces
```

provide better incident investigation.

---

# 107. Real-World Incident Example

Suppose:

```text
Payment 5xx rate increases.
```

Prometheus:

```text
payment_5xx_rate ↑
```

Grafana:

```text
Alert triggered
```

Kibana:

```text
service:payment
level:ERROR
```

Logstash has parsed:

```text
error.type
database.host
trace.id
```

Then Jaeger:

```text
Order
 ↓
Payment
 ↓
Database
 ↓
Timeout
```

This creates an end-to-end observability workflow.

---

# 108. Common Logstash Mistakes

### Mistake 1: One giant pipeline

```text
Everything
   ↓
One massive configuration
```

This becomes difficult to maintain.

Prefer logical pipelines.

---

# 109. Common Logstash Mistakes

### Mistake 2: Excessive Grok

Parsing every field using expensive regex increases CPU usage.

Prefer structured logs where possible.

---

# 110. Common Logstash Mistakes

### Mistake 3: No buffering

If Elasticsearch becomes unavailable:

```text
Input
 ↓
Logstash
 ↓
Elasticsearch X
```

Without appropriate buffering, events may be lost depending on the architecture.

---

# 111. Common Logstash Mistakes

### Mistake 4: No monitoring

A Logstash pipeline can fail while applications continue producing logs.

Monitor:

```text
Events in
Events out
Queue
Errors
Output failures
```

---

# 112. Common Logstash Mistakes

### Mistake 5: Hard-coded credentials

Never commit:

```text
username=admin
password=Secret123
```

Use secure secret management.

---

# 113. Common Logstash Mistakes

### Mistake 6: Sending everything to one index

Avoid uncontrolled index growth.

Use a deliberate strategy based on:

```text
Data type
Retention
Access control
Volume
Lifecycle
```

---

# 114. Common Logstash Mistakes

### Mistake 7: Logging sensitive information

Prevent:

```text
Passwords
Tokens
Secrets
PII
```

from entering the pipeline.

The best solution is to prevent them at the application source.

---

# 115. Common Logstash Mistakes

### Mistake 8: No capacity planning

If:

```text
Input = 100k events/sec
```

but:

```text
Output = 30k events/sec
```

the queue will continuously grow.

Measure throughput before scaling.

---

# 116. Logstash Troubleshooting Flow

Use:

```text
Application
     ↓
Collector
     ↓
Logstash Input
     ↓
Logstash Filter
     ↓
Logstash Output
     ↓
Elasticsearch
     ↓
Kibana
```

At each stage ask:

```text
Is data entering?
Is it being processed?
Is it leaving?
Is the next component receiving it?
```

---

# 117. Logs Not Appearing in Kibana

Troubleshoot in this order:

```text
1. Application generated the log?
2. Collector received it?
3. Collector sent it?
4. Logstash input received it?
5. Filter parsed it?
6. Output sent it?
7. Elasticsearch indexed it?
8. Correct index?
9. Kibana data view correct?
10. Time range correct?
```

This avoids guessing.

---

# 118. Logstash CPU High

Check:

```text
Grok
Ruby filters
JSON parsing
GeoIP
Event size
Pipeline workers
Input rate
```

Potential solution:

```text
Simplify parsing
Use structured logs
Use Dissect
Scale horizontally
Tune workers
```

---

# 119. Logstash Queue Growing

Check:

```text
Input rate
Output rate
Elasticsearch latency
Elasticsearch health
Network
Pipeline workers
CPU
```

If:

```text
Input > Output
```

find the downstream bottleneck.

---

# 120. Logstash Cannot Connect to Elasticsearch

Check:

```text
DNS
Port 9200
TLS
Certificate
Credentials
API key
Permissions
Cluster health
```

Test connectivity from the Logstash host.

---

# 121. Logstash Parsing Failure

If a field is missing:

```text
Raw message
     ↓
Grok/JSON/Dissect
     ↓
Parsing failure
```

Inspect the original event.

Use:

```ruby
stdout {
  codec => rubydebug
}
```

in a controlled development environment.

---

# 122. Production Logstash Checklist

```text
[ ] Multiple instances where required
[ ] Private networking
[ ] TLS
[ ] Authentication
[ ] Least privilege
[ ] Secure secrets
[ ] Persistent queue where appropriate
[ ] Adequate disk
[ ] CPU/memory sizing
[ ] Pipeline monitoring
[ ] Error monitoring
[ ] Elasticsearch monitoring
[ ] Backup/replay strategy where required
[ ] Structured logs
[ ] Sensitive-data filtering
[ ] Configuration in Git
[ ] Staging validation
```

---

# 123. Real-World Deployment Flow

For your EKS environment:

```text
GitHub
   ↓
Logstash Configuration
   ↓
Pull Request
   ↓
GitHub Actions
   ↓
Validation
   ↓
Security Checks
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Logstash
```

This keeps the logging pipeline consistent with your DevOps/GitOps workflow.

---

# 124. Complete Production Logging Architecture

```text
                         USERS
                           │
                           ↓
                        Kibana
                           │
                           ↓
                Elasticsearch Cluster
              ┌────────────┼────────────┐
              ↓            ↓            ↓
             ES1          ES2          ES3
              ↑            ↑            ↑
              └────────────┼────────────┘
                           ↑
                  Logstash Cluster
                  ┌────────┴────────┐
                  ↓                 ↓
                LS-01             LS-02
                  ↑                 ↑
                  └────────┬────────┘
                           ↑
                     Fluent Bit
                  ┌────────┼────────┐
                  ↓        ↓        ↓
                 Node     Node     Node
                  │        │        │
                 Pods     Pods     Pods
```

---

# 125. Final Logstash Mental Model

Remember:

```text
                    LOGSTASH
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        INPUT        FILTER       OUTPUT
          │            │            │
       Receive       Parse        Send
       Events       Transform     Events
                    Enrich
                    Normalize
```

And in your environment:

```text
EKS
 ↓
Fluent Bit
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

With resilience:

```text
EKS
 ↓
Fluent Bit
 ↓
Logstash HA
 ↓
Persistent Queue / Kafka
 ↓
Elasticsearch HA
 ↓
Kibana HA
```

The key production principle is:

**Use Logstash as a controlled processing and routing layer between log producers and Elasticsearch. Keep application logs structured whenever possible, minimize expensive parsing, design for backpressure and failures, secure the pipeline, monitor its throughput, and manage its configuration through GitOps or configuration management rather than manual production changes.**
