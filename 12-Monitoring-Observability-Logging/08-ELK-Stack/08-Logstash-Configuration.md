# Logstash Configuration

## 1. Overview

After installing Logstash, the next step is configuring it for real-world log processing.

Logstash configuration controls:

```text
Input
Filter
Output
Pipelines
Workers
Batch processing
Persistent queues
Dead-letter queues
Logging
Monitoring API
Security
```

The core Logstash processing model is:

```text
                LOGSTASH
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      INPUT      FILTER     OUTPUT
        │          │          │
     Receive     Process      Send
     Events      Events      Events
```

In your production architecture:

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

---

# 2. Configuration Files

A package installation commonly uses:

```text
/etc/logstash/
│
├── logstash.yml
├── pipelines.yml
├── jvm.options
└── conf.d/
```

The responsibilities are different:

```text
logstash.yml
    ↓
Logstash-wide settings

pipelines.yml
    ↓
Pipeline definitions

*.conf
    ↓
Input / Filter / Output logic

jvm.options
    ↓
JVM settings
```

---

# 3. Configuration Architecture

```text
                    Logstash
                       │
             ┌─────────┴─────────┐
             ↓                   ↓
       logstash.yml        pipelines.yml
             │                   │
             │            ┌──────┴──────┐
             │            ↓             ↓
             │       Pipeline-A    Pipeline-B
             │            │             │
             │            ↓             ↓
             │         *.conf        *.conf
             │            │             │
             └────────────┴─────────────┘
                          ↓
                    Logstash Engine
```

---

# 4. `logstash.yml`

The main Logstash configuration file is commonly:

```text
/etc/logstash/logstash.yml
```

It controls Logstash-wide settings.

Typical areas include:

```text
Node identity
Pipeline management
API
Logging
Monitoring
Queueing
Dead-letter queue
Configuration reload
```

---

# 5. `pipelines.yml`

The file:

```text
/etc/logstash/pipelines.yml
```

defines one or more pipelines.

Conceptually:

```text
pipelines.yml
     │
     ├── application
     │
     ├── infrastructure
     │
     └── security
```

When Logstash starts normally without `-f` or `-e`, it reads the pipeline definitions from `pipelines.yml`.

---

# 6. Pipeline Configuration

A pipeline normally contains:

```ruby
input {
}

filter {
}

output {
}
```

This is the most important Logstash configuration structure.

---

# 7. Basic Pipeline

Example:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  mutate {
    add_field => {
      "environment" => "production"
    }
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

Flow:

```text
Beats
 ↓
Input
 ↓
Mutate
 ↓
Stdout
```

---

# 8. Input Configuration

Inputs define where events come from.

Common inputs:

```text
Beats
TCP
UDP
HTTP
File
Kafka
Syslog
```

Example:

```ruby
input {
  beats {
    port => 5044
  }
}
```

---

# 9. Beats Input

For a common logging architecture:

```text
Fluent Bit / Filebeat
          ↓
       TCP 5044
          ↓
       Logstash
```

Configuration:

```ruby
input {
  beats {
    port => 5044
  }
}
```

For production, configure TLS where required.

---

# 10. TCP Input

Example:

```ruby
input {
  tcp {
    port => 5000
  }
}
```

Architecture:

```text
Application
    ↓
TCP
    ↓
Logstash
```

Use this only when the application or collector is designed to send data over TCP.

---

# 11. UDP Input

Example:

```ruby
input {
  udp {
    port => 5000
  }
}
```

UDP does not provide TCP-style delivery guarantees.

Therefore, use it only for workloads where the protocol's delivery characteristics are acceptable.

---

# 12. HTTP Input

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

For production:

```text
TLS
Authentication
Network restriction
Rate limiting
```

should be considered.

---

# 13. File Input

Example:

```ruby
input {
  file {
    path => "/var/log/application.log"
    start_position => "beginning"
  }
}
```

For Kubernetes, however, node-level collectors such as Fluent Bit are generally a better fit for container log collection.

---

# 14. Kafka Input

For high-volume environments:

```text
Applications
     ↓
Kafka
     ↓
Logstash
     ↓
Elasticsearch
```

Kafka can provide durable buffering and replay.

---

# 15. Syslog Input

For infrastructure logging:

```text
Network Devices
Servers
Firewalls
Security Devices
      ↓
    Syslog
      ↓
   Logstash
```

This is useful for centralized infrastructure logging.

---

# 16. Filter Configuration

Filters transform events.

Common filters:

```text
Grok
Mutate
Date
JSON
Dissect
GeoIP
Translate
Drop
Ruby
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

# 17. Grok Configuration

Suppose the application sends:

```text
2026-08-11T10:30:15Z ERROR Payment failed
```

A Grok filter can extract:

```text
timestamp
level
message
```

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

---

# 18. Grok Best Practice

Do not use Grok unnecessarily.

Prefer:

```text
Structured JSON
     ↓
Minimal parsing
```

over:

```text
Unstructured text
     ↓
Complex regex
     ↓
Many fields
```

Complex Grok patterns can consume significant CPU.

---

# 19. Dissect Configuration

For predictable delimiter-based logs:

```text
payment|ERROR|500|Database timeout
```

Dissect can be more appropriate than Grok.

Conceptually:

```ruby
filter {
  dissect {
    mapping => {
      "message" => "%{service}|%{level}|%{status}|%{error_message}"
    }
  }
}
```

---

# 20. Mutate Configuration

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

Other operations include:

```text
rename
replace
update
remove_field
convert
uppercase
lowercase
```

---

# 21. Rename Fields

Example:

```ruby
filter {
  mutate {
    rename => {
      "app_name" => "service"
    }
  }
}
```

This is useful for normalizing different log sources.

---

# 22. Remove Fields

Example:

```ruby
filter {
  mutate {
    remove_field => [
      "unnecessary_field"
    ]
  }
}
```

Do not remove fields blindly.

Some metadata can be useful for:

```text
Troubleshooting
Security
Correlation
Kubernetes investigation
```

---

# 23. Convert Fields

Example:

```ruby
filter {
  mutate {
    convert => {
      "status_code" => "integer"
    }
  }
}
```

This allows Elasticsearch to treat the value as numeric.

---

# 24. Date Filter

A correct timestamp is critical.

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

Architecture:

```text
Application Timestamp
        ↓
     Logstash
        ↓
   @timestamp
        ↓
   Elasticsearch
        ↓
      Kibana
```

---

# 25. JSON Filter

If the `message` field contains JSON:

```json
{
  "service": "payment",
  "level": "ERROR",
  "message": "Payment failed"
}
```

Use:

```ruby
filter {
  json {
    source => "message"
  }
}
```

This produces structured fields.

---

# 26. Conditional Processing

Logstash supports conditional logic.

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

# 27. Multiple Conditions

Example:

```ruby
filter {
  if [environment] == "production" and [level] == "ERROR" {
    mutate {
      add_field => {
        "severity" => "critical"
      }
    }
  }
}
```

Use conditions carefully to avoid making pipelines unnecessarily complex.

---

# 28. Drop Events

Sometimes events should not be indexed.

Example:

```ruby
filter {
  if [healthcheck] == true {
    drop { }
  }
}
```

This can reduce unnecessary log volume.

Be careful because dropping events permanently removes them from the pipeline.

---

# 29. Sensitive Data Filtering

Sensitive fields should not be indexed.

Example:

```ruby
filter {
  mutate {
    remove_field => [
      "password",
      "access_token"
    ]
  }
}
```

Better still:

```text
Application
   ↓
Do not log secrets
   ↓
Collector
   ↓
Logstash
```

Preventing sensitive data at the source is preferable.

---

# 30. Output Configuration

Outputs determine where processed events go.

Common outputs:

```text
Elasticsearch
Kafka
HTTP
File
S3
Stdout
```

For ELK:

```ruby
output {
  elasticsearch {
    hosts => ["https://elasticsearch.example.internal:9200"]
  }
}
```

---

# 31. Elasticsearch Output

Example:

```ruby
output {
  elasticsearch {
    hosts => [
      "https://es-01.internal:9200",
      "https://es-02.internal:9200",
      "https://es-03.internal:9200"
    ]
  }
}
```

In production, configure appropriate authentication and TLS trust.

---

# 32. Index Naming

A simple strategy might be:

```ruby
output {
  elasticsearch {
    hosts => ["https://elasticsearch.internal:9200"]
    index => "application-logs-%{+YYYY.MM.dd}"
  }
}
```

However, index strategy should be designed together with:

```text
Retention
Lifecycle
Data streams
Volume
Shard strategy
Access control
```

Do not create excessive numbers of tiny indexes.

---

# 33. Data Streams

For modern Elasticsearch logging architectures, data streams can be appropriate for continuously generated time-series data.

Conceptually:

```text
Application Logs
       ↓
Logstash
       ↓
Data Stream
       ↓
Backing Indices
```

This should be evaluated alongside the organization's Elasticsearch version and index lifecycle strategy.

---

# 34. Stdout Output

For development:

```ruby
output {
  stdout {
    codec => rubydebug
  }
}
```

This allows you to inspect the complete processed event.

---

# 35. Multiple Outputs

A pipeline can send an event to more than one destination.

Example:

```ruby
output {
  elasticsearch {
    hosts => ["https://elasticsearch.internal:9200"]
  }

  stdout {
    codec => rubydebug
  }
}
```

Use stdout primarily for development/troubleshooting, not as a normal production sink.

---

# 36. Conditional Output

Example:

```ruby
output {
  if [log_type] == "security" {
    elasticsearch {
      hosts => ["https://elasticsearch.internal:9200"]
      index => "security-logs"
    }
  } else {
    elasticsearch {
      hosts => ["https://elasticsearch.internal:9200"]
      index => "application-logs"
    }
  }
}
```

This routes events based on metadata.

---

# 37. `pipelines.yml` Example

A production-style organization might look like:

```yaml
- pipeline.id: application
  path.config: "/etc/logstash/conf.d/application/*.conf"

- pipeline.id: infrastructure
  path.config: "/etc/logstash/conf.d/infrastructure/*.conf"

- pipeline.id: security
  path.config: "/etc/logstash/conf.d/security/*.conf"
```

Each pipeline has its own processing path.

---

# 38. Why Multiple Pipelines?

Instead of:

```text
One giant pipeline
        ↓
Application
Security
Infrastructure
Network
Audit
```

use:

```text
Application Pipeline
Security Pipeline
Infrastructure Pipeline
Network Pipeline
```

Advantages:

```text
Isolation
Maintainability
Independent tuning
Clear ownership
Reduced configuration complexity
```

---

# 39. Pipeline Workers

Logstash pipelines use worker threads to process events.

Example concept:

```text
Events
  ↓
Worker 1
Worker 2
Worker 3
Worker 4
  ↓
Output
```

The number of workers should be based on workload and CPU capacity.

Do not increase workers blindly.

---

# 40. `pipeline.workers`

A pipeline can specify:

```yaml
- pipeline.id: application
  path.config: "/etc/logstash/conf.d/application/*.conf"
  pipeline.workers: 4
```

The correct value depends on:

```text
CPU
Filter complexity
Event rate
Pipeline behavior
Output latency
```

---

# 41. Pipeline Batch Size

A pipeline can also control batch size:

```yaml
- pipeline.id: application
  path.config: "/etc/logstash/conf.d/application/*.conf"
  pipeline.batch.size: 125
```

Batch size influences:

```text
Throughput
Memory
Latency
Output behavior
```

Do not tune this without measuring.

---

# 42. Pipeline Batch Delay

Logstash can wait briefly to accumulate events into a batch.

Conceptually:

```text
Events
 ↓
Wait briefly
 ↓
Batch
 ↓
Workers
```

This can affect:

```text
Throughput
Latency
```

The default values should normally be retained until monitoring shows a reason to tune them.

---

# 43. Pipeline Output Workers

Output plugins can have their own worker configuration in supported cases.

Example concept:

```yaml
pipeline.output.workers: 1
```

Changing output concurrency can increase throughput but may also increase pressure on Elasticsearch.

Tune carefully.

---

# 44. Persistent Queue

For production logging, persistent queues can provide disk-backed buffering.

Configuration:

```yaml
queue.type: persisted
```

Conceptually:

```text
Input
 ↓
Persistent Queue
 ↓
Filter
 ↓
Output
 ↓
Elasticsearch
```

Persistent queues are useful when downstream systems experience temporary failures.

---

# 45. Persistent Queue Storage

A persistent queue commonly uses:

```text
path.data/queue
```

or a configured queue path.

For example:

```yaml
path.queue: /var/lib/logstash/queue
```

The queue should be placed on suitable persistent storage.

---

# 46. Persistent Queue Capacity

Example:

```yaml
queue.max_bytes: 10gb
```

This means the queue can buffer up to the configured byte capacity.

The disk must have enough capacity beyond the queue limit for the OS and other Logstash data.

If both `queue.max_bytes` and `queue.max_events` are configured, the first limit reached controls queue capacity.

---

# 47. Persistent Queue Page Size

Example:

```yaml
queue.page_capacity: 64mb
```

This controls the size of queue pages.

Do not change it just because a production tutorial uses a different value.

Page size is generally not a first-line performance tuning parameter.

---

# 48. Persistent Queue Design

Example:

```text
Logstash
   │
   ↓
Persistent Queue
   │
   ├── Disk
   │
   ↓
Output
   │
   ↓
Elasticsearch
```

If Elasticsearch is temporarily unavailable:

```text
Elasticsearch
      X
      ↑
Persistent Queue
      ↑
Logstash
```

Events can remain buffered until downstream processing resumes, subject to queue capacity and the failure scenario.

---

# 49. Persistent Queue Is Not Kafka

Important distinction:

```text
Persistent Queue
    ↓
Local Logstash durability
```

while:

```text
Kafka
    ↓
Distributed durable messaging
```

Persistent queues do not provide the same distributed replay and decoupling characteristics as Kafka.

---

# 50. Dead-Letter Queue

Logstash can use a dead-letter queue for certain events that cannot be successfully processed.

Conceptually:

```text
Event
 ↓
Pipeline
 ↓
Processing Problem
 ↓
DLQ
```

Enable when appropriate:

```yaml
dead_letter_queue.enable: true
```

---

# 51. Dead-Letter Queue Storage

The DLQ requires storage.

Conceptually:

```text
Logstash
 ├── Pipeline
 │
 └── Dead Letter Queue
          ↓
         Disk
```

Monitor DLQ growth.

A growing DLQ usually indicates a processing or output problem that requires investigation.

---

# 52. Logstash API

Logstash exposes an HTTP API.

A common configuration is:

```yaml
http.host: "127.0.0.1"
http.port: 9600
```

The API is intended for monitoring and operational information.

Do not expose an unauthenticated operational API publicly. Elastic guidance recommends avoiding publicly reachable binding where possible.

---

# 53. API Security

Architecture:

```text
Prometheus / Admin
       ↓
Private Network
       ↓
Logstash API
```

Avoid:

```text
Internet
   ↓
Logstash API
```

unless specifically protected by a secure architecture.

---

# 54. Log Level

Logstash logging levels include:

```text
fatal
error
warn
info
debug
trace
```

A typical production default is:

```yaml
log.level: info
```

Use:

```text
debug
trace
```

only when troubleshooting because they can generate significantly more logs.

---

# 55. Log Path

A package installation commonly uses:

```yaml
path.logs: /var/log/logstash
```

This separates Logstash's own operational logs from processed application logs.

---

# 56. Configuration Reload

Logstash can be configured to detect configuration changes automatically.

Conceptually:

```yaml
config.reload.automatic: true
```

Then:

```text
Configuration Change
       ↓
Logstash detects change
       ↓
Pipeline reload
```

Automatic reload can be useful, but production environments should still use controlled deployment and validation.

---

# 57. Configuration Reload Strategy

For your GitOps environment:

```text
Git
 ↓
Pull Request
 ↓
GitHub Actions
 ↓
Validation
 ↓
Merge
 ↓
ArgoCD
 ↓
Deployment
```

This is preferable to allowing uncontrolled manual configuration changes.

---

# 58. Environment Variables

Some settings can use environment variables.

Conceptually:

```ruby
output {
  elasticsearch {
    hosts => ["${ELASTICSEARCH_URL}"]
  }
}
```

This allows environment-specific configuration without embedding the endpoint directly in the pipeline.

Secrets still need proper secret management.

---

# 59. Secret Management

Never commit:

```text
Passwords
API keys
Private keys
Certificates
Tokens
```

into Git.

Use:

```text
AWS Secrets Manager
Kubernetes Secrets
External Secrets
Logstash Keystore
```

depending on the deployment architecture.

---

# 60. Logstash Keystore

The Logstash keystore can store sensitive values separately from ordinary configuration.

Conceptually:

```text
Secret
 ↓
Logstash Keystore
 ↓
Pipeline
 ↓
Elasticsearch
```

This is preferable to plaintext credentials in pipeline files.

---

# 61. Secure Elasticsearch Output

Conceptually:

```ruby
output {
  elasticsearch {
    hosts => ["https://es-prod.internal:9200"]
    api_key => "${ELASTIC_API_KEY}"
  }
}
```

The actual secret should be injected securely.

Use TLS and validate the Elasticsearch certificate chain.

---

# 62. TLS Architecture

Production:

```text
Fluent Bit
    ↓
   TLS
    ↓
Logstash
    ↓
   TLS
    ↓
Elasticsearch
```

This protects log data while it is in transit.

---

# 63. TLS Between Collector and Logstash

For a Beats-compatible input, configure TLS according to the plugin's supported SSL/TLS settings.

Conceptually:

```ruby
input {
  beats {
    port => 5044
    ssl_enabled => true
  }
}
```

The exact plugin syntax depends on the Logstash/plugin version, so verify the configuration against the version deployed.

---

# 64. TLS Between Logstash and Elasticsearch

Example architecture:

```text
Logstash
   ↓
HTTPS
   ↓
Elasticsearch
```

The Logstash Elasticsearch output must trust the Elasticsearch CA/certificate chain.

---

# 65. Certificate Management

Production certificate architecture:

```text
Certificate Authority
       │
 ┌─────┼─────┐
 ↓     ↓     ↓
LS-01 LS-02 Elasticsearch
```

Certificates should have:

```text
Correct identity
Valid expiration
Trusted CA
Secure private keys
```

---

# 66. Avoid Disabling Certificate Verification

Bad troubleshooting approach:

```text
TLS error
   ↓
Disable verification
```

Better:

```text
TLS error
 ↓
Check CA
 ↓
Check certificate
 ↓
Check hostname
 ↓
Check expiration
 ↓
Fix trust configuration
```

Never weaken TLS security simply to make a pipeline work.

---

# 67. Kubernetes ConfigMap

For EKS, non-secret Logstash configuration can be stored in a ConfigMap.

Conceptually:

```text
Git
 ↓
ConfigMap
 ↓
Logstash Pod
 ↓
Pipeline
```

Secrets should not be placed in a normal ConfigMap.

---

# 68. Kubernetes Secret

Sensitive values can be supplied through Kubernetes Secrets or an external secret-management solution.

Architecture:

```text
AWS Secrets Manager
       ↓
External Secret
       ↓
Kubernetes Secret
       ↓
Logstash
```

This is preferable to committing credentials to Git.

---

# 69. Logstash Deployment With Helm

A Helm-based deployment may separate:

```text
values.yaml
ConfigMap
Secret
Deployment
Service
PVC
RBAC
```

Example:

```text
logstash/
├── Chart.yaml
├── values.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── configmap.yaml
    └── secret.yaml
```

---

# 70. Logstash With ArgoCD

Your GitOps flow:

```text
GitHub
   ↓
Logstash Helm values/config
   ↓
Pull Request
   ↓
GitHub Actions
   ↓
Validation
   ↓
Security checks
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Logstash
```

This gives:

```text
Version control
Auditability
Repeatability
Rollback
Drift detection
```

---

# 71. Production Pipeline Organization

A clean structure:

```text
/etc/logstash/
└── conf.d/
    ├── application/
    │   ├── input.conf
    │   ├── filter.conf
    │   └── output.conf
    │
    ├── infrastructure/
    │   ├── input.conf
    │   ├── filter.conf
    │   └── output.conf
    │
    └── security/
        ├── input.conf
        ├── filter.conf
        └── output.conf
```

---

# 72. Application Pipeline

Example:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  if [service] == "payment" {
    mutate {
      add_field => {
        "team" => "payments"
      }
    }
  }
}

output {
  elasticsearch {
    hosts => ["https://es-prod.internal:9200"]
    index => "application-logs"
  }
}
```

---

# 73. Infrastructure Pipeline

Example:

```ruby
input {
  syslog {
    port => 5514
  }
}

filter {
  mutate {
    add_field => {
      "log_type" => "infrastructure"
    }
  }
}

output {
  elasticsearch {
    hosts => ["https://es-prod.internal:9200"]
    index => "infrastructure-logs"
  }
}
```

---

# 74. Security Pipeline

Example:

```ruby
input {
  tcp {
    port => 5515
  }
}

filter {
  mutate {
    add_field => {
      "log_type" => "security"
    }
  }
}

output {
  elasticsearch {
    hosts => ["https://es-prod.internal:9200"]
    index => "security-logs"
  }
}
```

These are learning examples; production input ports and security controls must match your architecture.

---

# 75. Structured Logging Pipeline

For applications producing JSON:

```text
Application
     ↓
JSON
     ↓
Fluent Bit
     ↓
Logstash
     ↓
Minimal transformation
     ↓
Elasticsearch
```

Example:

```json
{
  "timestamp": "2026-08-11T08:30:00Z",
  "service": "payment",
  "level": "ERROR",
  "message": "Database timeout",
  "status_code": 500
}
```

This reduces parsing complexity.

---

# 76. Kubernetes Metadata Pipeline

A Kubernetes event might contain:

```json
{
  "namespace": "production",
  "pod": "payment-7d9f8",
  "container": "payment",
  "service": "payment",
  "level": "ERROR"
}
```

Logstash can normalize and enrich this data before sending it to Elasticsearch.

---

# 77. Correlation Fields

For your observability architecture, preserve:

```text
trace.id
span.id
transaction.id
request.id
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

This allows:

```text
Metrics
   ↓
Logs
   ↓
Traces
```

to be correlated.

---

# 78. Logstash and Prometheus

Logstash operational metrics can be collected and visualized using your monitoring stack.

Architecture:

```text
Logstash
   ↓
Metrics API / Monitoring Integration
   ↓
Prometheus
   ↓
Grafana
```

Track:

```text
Events in
Events out
Queue
CPU
Memory
Pipeline health
Output failures
```

---

# 79. Important Logstash Metrics

At minimum monitor:

```text
Events in
Events out
Events duration
Pipeline throughput
Queue size
JVM heap
CPU
Memory
Elasticsearch output failures
```

---

# 80. Alerting

Useful alerts:

```text
Logstash down
Pipeline stopped
Events in suddenly zero
Events out suddenly zero
Queue growing
Persistent queue nearly full
Elasticsearch output failures
CPU saturation
JVM memory pressure
Disk nearly full
```

---

# 81. Backpressure

Example:

```text
Input
100,000 events/sec
        ↓
Logstash
        ↓
Elasticsearch
30,000 events/sec
```

Then:

```text
Queue ↑
Latency ↑
Disk ↑
```

This is backpressure.

Do not simply increase queue size indefinitely.

Find the bottleneck.

---

# 82. Backpressure Troubleshooting

Check:

```text
Input rate
Filter processing
Pipeline workers
Elasticsearch latency
Elasticsearch cluster health
Network
CPU
Memory
Queue
```

Architecture:

```text
Input
 ↓
Filter
 ↓
Output
 ↓
Bottleneck
```

The bottleneck determines overall throughput.

---

# 83. Pipeline Performance

Performance depends on:

```text
Event size
Events/sec
Grok complexity
JSON parsing
GeoIP
Pipeline workers
Batch size
Elasticsearch response time
Network latency
```

Measure before changing configuration.

---

# 84. Horizontal Scaling

If one Logstash node cannot process the workload:

```text
                 Load Balancer
                      │
             ┌────────┴────────┐
             ↓                 ↓
          LS-01              LS-02
             │                 │
             └────────┬────────┘
                      ↓
               Elasticsearch
```

This is generally preferable to endlessly increasing the size of one node.

---

# 85. Node Placement on EKS

For production:

```text
Worker-01
 └── Logstash-01

Worker-02
 └── Logstash-02

Worker-03
 └── Logstash-03
```

Use:

```text
Pod anti-affinity
Topology spread constraints
Node affinity
```

to improve resilience.

---

# 86. Persistent Queue on EKS

If enabled:

```text
Logstash Pod
     ↓
PVC
     ↓
Persistent Storage
```

Each replica should have appropriate independent storage.

Do not assume multiple Logstash Pods can safely share one writable persistent queue directory.

---

# 87. Production Configuration Example

A conceptual `logstash.yml`:

```yaml
node.name: logstash-prod-01

http.host: "127.0.0.1"
http.port: 9600

path.data: /var/lib/logstash
path.logs: /var/log/logstash

queue.type: persisted
queue.max_bytes: 10gb

log.level: info
```

This is a starting example, not a universal production configuration.

Queue capacity must be based on workload, outage duration, and available storage.

---

# 88. Production `pipelines.yml`

Example:

```yaml
- pipeline.id: application
  path.config: "/etc/logstash/conf.d/application/*.conf"
  pipeline.workers: 4
  pipeline.batch.size: 125

- pipeline.id: infrastructure
  path.config: "/etc/logstash/conf.d/infrastructure/*.conf"
  pipeline.workers: 2
  pipeline.batch.size: 125

- pipeline.id: security
  path.config: "/etc/logstash/conf.d/security/*.conf"
  pipeline.workers: 2
  pipeline.batch.size: 125
```

The worker and batch values must be tuned using actual workload measurements.

---

# 89. Production Pipeline

Example:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {

  if [message] =~ /^\{/ {
    json {
      source => "message"
    }
  }

  if [log_timestamp] {
    date {
      match => [
        "log_timestamp",
        "ISO8601"
      ]
    }
  }

  mutate {
    add_field => {
      "environment" => "production"
    }
  }

  mutate {
    remove_field => [
      "password",
      "access_token"
    ]
  }
}

output {
  elasticsearch {
    hosts => [
      "https://es-01.internal:9200",
      "https://es-02.internal:9200",
      "https://es-03.internal:9200"
    ]

    index => "application-logs"
  }
}
```

This demonstrates:

```text
Input
 ↓
JSON parsing
 ↓
Timestamp normalization
 ↓
Metadata
 ↓
Sensitive-field removal
 ↓
Elasticsearch
```

Actual authentication and TLS trust settings must be added according to the deployed Elasticsearch security configuration.

---

# 90. Configuration Validation

Before deployment:

```bash
sudo /usr/share/logstash/bin/logstash \
  --config.test_and_exit \
  -f /etc/logstash/conf.d/
```

Validate:

```text
Syntax
Plugins
Pipeline structure
Conditions
```

Then test the pipeline with real representative events.

---

# 91. Testing Strategy

Use:

```text
Development
    ↓
Staging
    ↓
Production
```

Test:

```text
Valid events
Malformed events
Large events
High event volume
Elasticsearch unavailable
Network failure
Authentication failure
TLS failure
Queue growth
Recovery
```

---

# 92. Configuration Rollback

Suppose a pipeline change causes:

```text
Events stop
```

With Git:

```text
Current configuration
       ↓
Bad change

Git history
       ↓
Previous version
```

Rollback:

```text
git revert
   ↓
GitHub Actions
   ↓
Validation
   ↓
ArgoCD
   ↓
Previous configuration
```

---

# 93. Configuration Drift

Bad:

```text
Git:
pipeline.workers: 4

Production:
pipeline.workers: 8
```

This is drift.

GitOps aims for:

```text
Desired State
      =
Actual State
```

---

# 94. Manual Change vs GitOps

Avoid:

```text
SSH
 ↓
Production
 ↓
vim /etc/logstash/...
```

Prefer:

```text
Git
 ↓
Pull Request
 ↓
Review
 ↓
GitHub Actions
 ↓
ArgoCD / Automation
 ↓
Production
```

This provides traceability.

---

# 95. Configuration Troubleshooting

When Logstash fails:

```text
1. Check service
2. Check Logstash logs
3. Validate configuration
4. Check pipeline ID
5. Check input
6. Check filter
7. Check output
8. Check Elasticsearch
9. Check TLS
10. Check credentials
11. Check queue
12. Roll back if required
```

---

# 96. Common Error: Pipeline Syntax

Example:

```text
Expected one of:
input
filter
output
```

Check:

```text
Braces
Quotes
Condition syntax
Plugin syntax
```

A single missing `}` can prevent an entire pipeline from starting.

---

# 97. Common Error: Wrong Pipeline Path

Example:

```yaml
- pipeline.id: application
  path.config: "/etc/logstash/conf.d/application/*.conf"
```

If the directory does not contain the expected files:

```text
Pipeline
   ↓
No configuration
```

Verify:

```bash
ls -la /etc/logstash/conf.d/application/
```

---

# 98. Common Error: Elasticsearch Authentication

Symptoms:

```text
401
403
```

Check:

```text
API key
Credentials
Index privileges
TLS
Elasticsearch security
```

Remember:

```text
401
 ↓
Authentication problem

403
 ↓
Authorization problem
```

---

# 99. Common Error: Elasticsearch TLS

Check:

```text
CA
Certificate
Hostname
Expiration
Trust configuration
HTTPS URL
```

Do not disable certificate verification as a permanent solution.

---

# 100. Common Error: Port Conflict

Check:

```bash
sudo ss -lntp | grep 5044
```

Then:

```bash
sudo lsof -i :5044
```

Identify the process using the port.

---

# 101. Common Error: Queue Full

Symptoms:

```text
Queue capacity reached
Events delayed
Input backpressure
```

Investigate:

```text
Elasticsearch
Output failures
Input rate
Pipeline performance
Disk capacity
```

Do not simply increase queue size without identifying why it is filling.

---

# 102. Common Error: High CPU

Investigate:

```text
Grok
Ruby
JSON
GeoIP
Event size
Workers
Input rate
```

Possible solutions:

```text
Structured logging
Simpler filters
Dissect
Horizontal scaling
Pipeline separation
```

---

# 103. Common Error: High Memory

Investigate:

```text
JVM heap
Batch size
Event size
Queue
Number of pipelines
Input rate
```

Do not allocate all available memory to the JVM.

---

# 104. Common Error: Logs Missing in Kibana

Trace the complete path:

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

Verify each stage rather than assuming the problem is Kibana.

---

# 105. Production Security Architecture

```text
                         PRIVATE VPC
                              │
                 ┌────────────┴────────────┐
                 ↓                         ↓
             Log Collectors             Kibana
                 │                         │
                 ↓                         ↓
              TLS                       TLS
                 │                         │
                 ↓                         ↓
            Logstash Cluster      Elasticsearch Cluster
                 │                         │
                 └────────────┬────────────┘
                              ↓
                         Secure Access
```

---

# 106. Complete Real-World Configuration Flow

```text
Application
     ↓
Structured JSON Logs
     ↓
Fluent Bit
     ↓
TLS
     ↓
Logstash
     ↓
Input
     ↓
Filter
     ├── JSON
     ├── Date
     ├── Mutate
     ├── Grok/Dissect
     └── Enrichment
     ↓
Output
     ↓
HTTPS/TLS
     ↓
Elasticsearch
     ↓
Kibana
```

---

# 107. Configuration Management Flow

For your DevOps environment:

```text
Developer
   ↓
GitHub
   ↓
Logstash Configuration
   ↓
Pull Request
   ↓
GitHub Actions
   ├── Syntax validation
   ├── Tests
   └── Security checks
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Logstash
```

---

# 108. Configuration Best Practices

```text
1. Keep configuration in Git.
2. Use multiple logical pipelines.
3. Prefer structured logs.
4. Minimize expensive Grok processing.
5. Use Dissect for predictable formats.
6. Parse timestamps correctly.
7. Remove sensitive data.
8. Use TLS.
9. Use least-privilege Elasticsearch credentials.
10. Protect the Logstash API.
11. Use persistent queues where justified.
12. Monitor queue growth.
13. Monitor events in/out.
14. Validate configuration before deployment.
15. Test in staging.
16. Use GitOps for Kubernetes.
17. Avoid manual production changes.
18. Size workers and batches based on measurements.
19. Plan for Elasticsearch failures.
20. Test recovery procedures.
```

---

# 109. Final Production Configuration Checklist

```text
Configuration
[ ] logstash.yml configured
[ ] pipelines.yml configured
[ ] Pipeline files organized
[ ] Inputs configured
[ ] Filters configured
[ ] Outputs configured

Performance
[ ] Workers sized
[ ] Batch size reviewed
[ ] CPU monitored
[ ] Memory monitored
[ ] Queue monitored

Reliability
[ ] Persistent queue evaluated
[ ] Queue capacity planned
[ ] DLQ evaluated
[ ] Elasticsearch failure tested
[ ] Backpressure tested

Security
[ ] TLS
[ ] Authentication
[ ] Least privilege
[ ] Secrets protected
[ ] API restricted
[ ] Sensitive fields removed

Operations
[ ] Prometheus monitoring
[ ] Grafana dashboard
[ ] Alerts
[ ] Logs
[ ] Git version control
[ ] CI validation
[ ] GitOps deployment
[ ] Rollback procedure
```

---

# 110. Final Mental Model

Remember Logstash configuration in five layers:

```text
                    LOGSTASH
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
        INPUT        FILTER       OUTPUT
          │            │            │
       Receive       Parse         Send
       Events        Transform     Events
                    Enrich
                       │
                       ↓
                  PIPELINE ENGINE
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Workers       Batch        Queue
          │            │            │
          └────────────┼────────────┘
                       ↓
                    Security
                       │
                  TLS + Auth
                       ↓
                 Elasticsearch
```

And for your real-world EKS environment:

```text
                  GitHub
                     ↓
              GitHub Actions
                     ↓
             Configuration Test
                     ↓
                  ArgoCD
                     ↓
                    EKS
                     ↓
              Logstash Cluster
             ┌───────┴───────┐
             ↓               ↓
          LS-01            LS-02
             │               │
             └───────┬───────┘
                     ↓
             Elasticsearch
             ┌───────┼───────┐
             ↓       ↓       ↓
            ES-01   ES-02   ES-03
                     ↓
                   Kibana
```

The key principle is:

**Treat Logstash configuration as production code. Separate pipelines logically, keep logs structured, process only what is necessary, protect secrets and traffic, design for backpressure and downstream failures, validate every configuration change, and deploy through your GitHub Actions + ArgoCD workflow.**
