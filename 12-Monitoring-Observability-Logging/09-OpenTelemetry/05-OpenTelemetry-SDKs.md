# OpenTelemetry SDKs

## 1. Overview

OpenTelemetry SDKs provide the implementation layer that turns application instrumentation into telemetry.

The basic architecture is:

```text
Application
     ↓
OpenTelemetry API
     ↓
OpenTelemetry SDK
     ↓
Telemetry
     ↓
Exporter
     ↓
OpenTelemetry Collector
     ↓
Backend
```

The SDK is responsible for important tasks such as:

```text
Telemetry generation
Resource detection
Context management
Sampling
Processing
Exporting
Configuration
```

---

# 2. API vs SDK

The OpenTelemetry API defines the interfaces used by application code.

The SDK provides the actual implementation.

```text
Application
     ↓
OTel API
     ↓
OTel SDK
     ↓
Telemetry
```

Think of it as:

```text
API = What the application uses

SDK = How telemetry is implemented
```

---

# 3. SDK Responsibilities

An OpenTelemetry SDK typically handles:

```text
TracerProvider
MeterProvider
LoggerProvider
SpanProcessor
MetricReader
LogRecordProcessor
Exporters
Samplers
Resources
Context
```

Not every language exposes exactly the same class names or capabilities, but the underlying concepts are similar.

---

# 4. Supported Languages

OpenTelemetry provides SDKs and instrumentation ecosystems for many languages.

Common production languages include:

```text
Java
Python
JavaScript / Node.js
Go
.NET
C++
PHP
Ruby
Rust
```

Each language has its own SDK packages and installation mechanism.

---

# 5. General SDK Architecture

```text
                    Application
                         │
             ┌───────────┴───────────┐
             ↓                       ↓
        Manual Code          Auto Instrumentation
             │                       │
             └───────────┬───────────┘
                         ↓
                     OTel API
                         ↓
                     OTel SDK
                         │
            ┌────────────┼────────────┐
            ↓            ↓            ↓
         Traces        Metrics        Logs
            │            │            │
            └────────────┼────────────┘
                         ↓
                      Exporter
                         ↓
                     Collector
```

---

# 6. TracerProvider

The `TracerProvider` manages tracing functionality.

Conceptually:

```text
Application
     ↓
TracerProvider
     ↓
Tracer
     ↓
Span
```

The application obtains a tracer from the configured provider.

---

# 7. Tracer

A tracer creates spans.

Conceptually:

```text
Tracer
   ↓
Start Span
   ↓
Perform Operation
   ↓
End Span
```

Example:

```text
payment-service
      ↓
Tracer
      ↓
process-payment span
```

---

# 8. Span

A span represents an individual operation.

Example:

```text
Trace
│
├── HTTP Request
├── Database Query
├── Payment Processing
└── External API Call
```

Each operation can have its own span.

A span can contain:

```text
Span name
Start time
End time
Attributes
Events
Status
Links
Parent relationship
Trace ID
Span ID
```

---

# 9. Creating a Span

Conceptually:

```text
Tracer
  ↓
Start span
  ↓
Execute operation
  ↓
Set attributes
  ↓
Record error
  ↓
End span
```

Example pseudocode:

```text
span = tracer.startSpan("process-payment")

performPayment()

span.end()
```

The exact API depends on the programming language.

---

# 10. Manual Instrumentation

Manual instrumentation is useful when the application needs business-level visibility.

Example:

```text
process-order
validate-payment
reserve-inventory
send-notification
```

Automatic instrumentation may know that an HTTP request occurred, but manual instrumentation can provide business context.

---

# 11. Automatic Instrumentation

Automatic instrumentation can instrument supported libraries and frameworks.

Example:

```text
Java Application
       ↓
OTel Java Agent
       ↓
HTTP
JDBC
Kafka
Spring
       ↓
Telemetry
```

This significantly reduces application-code changes.

---

# 12. Manual + Automatic Instrumentation

Production applications often combine both.

```text
Application
    │
    ├── Auto Instrumentation
    │       ↓
    │   HTTP / DB / Messaging
    │
    └── Manual Instrumentation
            ↓
        Business Operations
```

This provides both technical and business-level observability.

---

# 13. MeterProvider

The `MeterProvider` manages metrics.

Conceptually:

```text
Application
     ↓
MeterProvider
     ↓
Meter
     ↓
Metric Instrument
```

Metrics can include:

```text
Counter
Histogram
Gauge / Observable Gauge
UpDownCounter
```

The exact available instruments depend on the language SDK and API version.

---

# 14. Meter

A meter creates metric instruments.

Conceptually:

```text
Meter
  │
  ├── Counter
  ├── Histogram
  └── Gauge
```

Example:

```text
payment_meter
      ↓
payment_requests_total
```

---

# 15. Counter

A counter represents a value that generally increases.

Example:

```text
HTTP requests
Payment attempts
Errors
Orders processed
```

Conceptually:

```text
Requests

100
 ↓
101
 ↓
102
 ↓
103
```

Example:

```text
payment_requests_total
```

---

# 16. Histogram

A histogram records a distribution of measurements.

Typical use cases:

```text
Request latency
Database latency
Response size
Processing duration
```

Example:

```text
HTTP request duration
      ↓
Histogram
      ↓
p50
p95
p99
```

Histograms are particularly useful for latency analysis.

---

# 17. Gauge

A gauge represents a value that can increase or decrease.

Examples:

```text
Active connections
Queue depth
Current temperature
Memory usage
Active requests
```

Conceptually:

```text
100
 ↓
80
 ↓
120
 ↓
95
```

---

# 18. Logs SDK

OpenTelemetry also provides APIs and SDK capabilities for logs.

Conceptually:

```text
Application
     ↓
LoggerProvider
     ↓
Logger
     ↓
Log Record
     ↓
Exporter
```

A log record can contain:

```text
Timestamp
Severity
Body
Attributes
Resource
Trace ID
Span ID
```

---

# 19. LoggerProvider

The `LoggerProvider` manages logging functionality.

Conceptually:

```text
LoggerProvider
      ↓
Logger
      ↓
Log Record
```

The implementation details vary by language.

---

# 20. Correlating Logs With Traces

One important capability is correlation.

Example:

```text
Log:
Payment failed

Trace ID:
abc123

Span ID:
def456
```

Then:

```text
Kibana Log
     ↓
Trace ID
     ↓
Trace Backend
```

This allows engineers to move from a log event to the corresponding distributed trace.

---

# 21. Resource

A resource describes the entity producing telemetry.

Example:

```text
service.name = payment
service.version = v2.1.0
deployment.environment = production
```

The SDK associates telemetry with the resource.

Architecture:

```text
Application
     ↓
Resource
     ↓
Telemetry
```

---

# 22. Resource Detection

SDKs can detect resource information automatically or receive it through configuration.

Possible information:

```text
Service
Host
Cloud provider
Cloud region
Container
Kubernetes
Process
Operating system
```

In Kubernetes:

```text
k8s.namespace.name
k8s.pod.name
k8s.container.name
k8s.node.name
```

---

# 23. Resource Detection in EKS

For an EKS workload:

```text
Application
     ↓
OTel SDK
     ↓
Resource Detection
     ↓
Kubernetes Metadata
     ↓
Telemetry
```

Useful attributes can identify:

```text
Cluster
Namespace
Pod
Container
Node
Service
```

This is valuable when troubleshooting production workloads.

---

# 24. Context

Context carries information associated with the current operation.

For distributed tracing, context can include:

```text
Trace ID
Span ID
Trace flags
Baggage
```

Architecture:

```text
Service A
   ↓
Context
   ↓
Service B
   ↓
Context
   ↓
Service C
```

---

# 25. Context Propagation

Context propagation connects spans across service boundaries.

Example:

```text
Frontend
   ↓
Orders
   ↓
Payment
```

Without propagation:

```text
Trace A
Trace B
Trace C
```

With propagation:

```text
Trace ABC
│
├── Frontend
├── Orders
└── Payment
```

---

# 26. W3C Trace Context

OpenTelemetry commonly uses W3C Trace Context.

Important headers include:

```text
traceparent
tracestate
```

Conceptually:

```text
HTTP Request
     │
     ├── traceparent
     └── tracestate
          ↓
       Service B
```

The receiving service extracts the context and continues the trace.

---

# 27. Baggage

Baggage allows additional contextual information to travel with requests.

Conceptually:

```text
Request
   ↓
Baggage
   ↓
Service A
   ↓
Service B
```

Baggage should be used carefully.

Do not put:

```text
Passwords
Access tokens
Sensitive personal information
Large payloads
```

into baggage.

---

# 28. Sampler

A sampler determines whether a trace should be recorded or exported.

Conceptually:

```text
Request
   ↓
Sampler
   ├── Record
   └── Drop
```

Sampling is important for high-volume environments.

---

# 29. Sampling Strategies

Common strategies include:

```text
AlwaysOn
AlwaysOff
TraceIdRatioBased
ParentBased
Tail Sampling
```

The exact available sampler options vary by SDK.

---

# 30. Always-On Sampling

Every trace is sampled.

```text
1000 requests
     ↓
1000 traces
```

Advantages:

```text
Complete visibility
Simple
```

Disadvantages:

```text
High storage
High network usage
Higher backend cost
```

This is generally unsuitable for very high-volume production environments unless carefully sized.

---

# 31. Always-Off Sampling

No traces are sampled.

```text
1000 requests
     ↓
0 traces
```

This can be useful for:

```text
Testing
Temporarily disabling tracing
```

It provides no tracing visibility.

---

# 32. Ratio-Based Sampling

A percentage of traces is sampled.

Example:

```text
1000 requests
     ↓
10% sampling
     ↓
100 traces
```

This reduces telemetry volume.

---

# 33. Parent-Based Sampling

A child service can respect the sampling decision made by the parent.

Architecture:

```text
Frontend
   ↓
Sampling Decision
   ↓
Orders
   ↓
Payment
```

This helps maintain consistent trace decisions across distributed services.

---

# 34. SpanProcessor

A span processor receives completed or starting spans and controls how they are handled before export.

Conceptually:

```text
Span
 ↓
SpanProcessor
 ↓
Exporter
```

Common concepts include:

```text
Simple processor
Batch processor
```

The exact APIs depend on the SDK language.

---

# 35. Simple Span Processor

A simple processor can export spans individually.

Conceptually:

```text
Span 1 → Export
Span 2 → Export
Span 3 → Export
```

This is easy to understand but can create significant overhead.

It is more appropriate for simple development/testing scenarios than high-volume production workloads.

---

# 36. Batch Span Processor

A batch processor groups spans before exporting.

```text
Span 1 ┐
Span 2 ├──→ Batch
Span 3 ┘
          ↓
       Exporter
```

Advantages:

```text
Better throughput
Lower network overhead
More efficient exports
```

Batch processing is commonly preferred for production.

---

# 37. Metric Reader

Metrics use a reader mechanism to collect and export metric data.

Conceptually:

```text
Metric Instrument
      ↓
Metric Reader
      ↓
Exporter
      ↓
Collector
```

The exact implementation differs between SDK languages.

---

# 38. Periodic Metric Export

Metrics may be exported periodically.

Conceptually:

```text
Application
     ↓
Metric
     ↓
Metric Reader
     ↓
Every N seconds
     ↓
Exporter
```

The interval should be chosen according to:

```text
Monitoring requirements
Telemetry volume
Backend capacity
Network overhead
```

---

# 39. SDK Exporters

SDK exporters send telemetry out of the application.

Common architecture:

```text
Application
     ↓
OTel SDK
     ↓
OTLP Exporter
     ↓
Collector
```

The Collector can then handle:

```text
Batching
Filtering
Sampling
Routing
Backend export
```

---

# 40. Direct-to-Backend vs Collector

Two possible architectures exist.

### Direct

```text
Application
    ↓
SDK
    ↓
Backend
```

### Collector

```text
Application
    ↓
SDK
    ↓
Collector
    ↓
Backend
```

The Collector architecture is often preferred in larger environments because it centralizes processing and backend integration.

---

# 41. Why Use a Collector

The Collector provides:

```text
Centralized processing
Backend abstraction
Filtering
Sampling
Batching
Routing
Retry
Security controls
```

Without a Collector, every application may need backend-specific configuration.

---

# 42. SDK Configuration Through Environment Variables

Example:

```bash
OTEL_SERVICE_NAME=payment
OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-gateway:4317
OTEL_RESOURCE_ATTRIBUTES=deployment.environment=production
```

The SDK reads these values according to the language's OpenTelemetry configuration support.

---

# 43. Java SDK Architecture

Java can use:

```text
OpenTelemetry API
OpenTelemetry SDK
OpenTelemetry Instrumentation
OpenTelemetry Java Agent
```

Architecture:

```text
Java Application
      ↓
Java Agent / SDK
      ↓
TracerProvider
MeterProvider
LoggerProvider
      ↓
OTLP
      ↓
Collector
```

---

# 44. Java Agent

For existing Java applications, the Java Agent is often useful.

```text
application.jar
      +
opentelemetry-javaagent.jar
      ↓
Automatic instrumentation
```

Startup concept:

```bash
java \
  -javaagent:/opt/opentelemetry/opentelemetry-javaagent.jar \
  -jar application.jar
```

---

# 45. Java Manual Instrumentation

Manual instrumentation can use the Java API.

Conceptually:

```java
Span span = tracer.spanBuilder("process-payment")
    .startSpan();

try {
    processPayment();
} finally {
    span.end();
}
```

This allows business-specific operations to be traced.

---

# 46. Node.js SDK Architecture

Node.js uses the OpenTelemetry JavaScript ecosystem.

Architecture:

```text
Node.js Application
       ↓
OTel API
       ↓
OTel SDK
       ↓
Instrumentation
       ↓
OTLP Exporter
       ↓
Collector
```

Instrumentation should be initialized before the modules requiring instrumentation are loaded.

---

# 47. Python SDK Architecture

Python:

```text
Python Application
       ↓
OTel API
       ↓
OTel SDK
       ↓
Instrumentation
       ↓
OTLP Exporter
       ↓
Collector
```

Python also supports automatic instrumentation for supported libraries.

---

# 48. Go SDK Architecture

Go applications can use the OpenTelemetry Go API and SDK.

Architecture:

```text
Go Application
      ↓
OTel API
      ↓
OTel SDK
      ↓
Tracer / Meter
      ↓
OTLP
      ↓
Collector
```

Go applications often use explicit initialization during application startup.

---

# 49. SDK Initialization

A typical application startup sequence:

```text
Process starts
    ↓
Load configuration
    ↓
Create Resource
    ↓
Create TracerProvider
    ↓
Create MeterProvider
    ↓
Configure exporters
    ↓
Register providers
    ↓
Initialize instrumentation
    ↓
Start application
```

The exact order and APIs depend on the language.

---

# 50. Graceful Shutdown

Applications should flush telemetry before terminating.

Conceptually:

```text
Application receives shutdown
          ↓
Stop accepting requests
          ↓
Flush telemetry
          ↓
Shutdown exporters
          ↓
Exit
```

Without graceful shutdown, telemetry still in buffers may be lost.

---

# 51. Kubernetes Pod Termination

In Kubernetes:

```text
Pod
 ↓
SIGTERM
 ↓
Application shutdown
 ↓
Flush telemetry
 ↓
Process exits
```

Configure an appropriate termination grace period when required.

Example:

```yaml
terminationGracePeriodSeconds: 30
```

The correct value depends on application shutdown and telemetry flush behavior.

---

# 52. SDK Resource Configuration

A production service should have consistent identity:

```text
service.name
service.version
deployment.environment
```

Example:

```text
service.name = orders
service.version = v3.2.1
deployment.environment = production
```

---

# 53. Kubernetes Metadata

A service can additionally identify its Kubernetes location:

```text
k8s.cluster.name
k8s.namespace.name
k8s.pod.name
k8s.container.name
```

Then telemetry can be queried by:

```text
service.name = payment
AND
k8s.namespace.name = production
```

---

# 54. Semantic Conventions

OpenTelemetry defines semantic conventions for common telemetry attributes.

Examples include attributes describing:

```text
HTTP
Database
Messaging
RPC
Cloud
Kubernetes
Services
```

Using standard attribute names improves consistency across services.

---

# 55. HTTP Instrumentation

An HTTP server request can produce:

```text
HTTP method
HTTP route
HTTP status
Request duration
Server address
```

Example:

```text
GET /api/orders
status = 200
duration = 125ms
```

Standardized conventions make telemetry easier to query.

---

# 56. Database Instrumentation

Database calls can generate spans.

Example:

```text
Orders
   ↓
Database Span
   ↓
SELECT orders
```

Useful information can include:

```text
Database system
Operation
Server
Duration
Status
```

Sensitive query data should be handled carefully.

---

# 57. Messaging Instrumentation

For asynchronous systems:

```text
Orders
   ↓
RabbitMQ / Kafka
   ↓
Payment
```

OpenTelemetry instrumentation can help trace messaging operations where supported.

Architecture:

```text
Producer Span
      ↓
Messaging Context
      ↓
Consumer Span
```

---

# 58. Trace Context Across Messaging

For asynchronous communication:

```text
Producer
   ↓
Message
   ↓
Queue
   ↓
Consumer
```

Trace context can be propagated through message metadata where supported.

This helps maintain trace relationships across asynchronous boundaries.

---

# 59. SDK and Microservices

A microservices platform might look like:

```text
                    ALB
                     ↓
                  Frontend
                     ↓
                  Orders
                     ↓
                  Payment
                     ↓
                 Inventory
```

Each service has:

```text
OTel SDK
   ↓
Service identity
   ↓
Instrumentation
   ↓
OTLP
```

All services send telemetry toward the Collector layer.

---

# 60. SDK Architecture in EKS

```text
EKS
│
├── Frontend
│    └── OTel SDK
│
├── Orders
│    └── OTel SDK
│
├── Payment
│    └── OTel SDK
│
└── Inventory
     └── OTel SDK
          │
          ↓
      OTel Agent
          ↓
      OTel Gateway
```

---

# 61. SDK Configuration in Deployment

Example:

```yaml
env:
  - name: OTEL_SERVICE_NAME
    value: "payment"

  - name: OTEL_RESOURCE_ATTRIBUTES
    value: "deployment.environment=production"

  - name: OTEL_EXPORTER_OTLP_ENDPOINT
    value: "http://otel-gateway.observability.svc.cluster.local:4317"
```

This allows the same application image to be deployed into different environments with environment-specific configuration.

---

# 62. Configuration by Environment

Development:

```text
service.name = payment
environment = development
```

Staging:

```text
service.name = payment
environment = staging
```

Production:

```text
service.name = payment
environment = production
```

The application artifact can remain unchanged.

---

# 63. SDK and Secrets

Avoid putting secrets into environment variables unless the platform's secret-management model is appropriate.

Never place:

```text
API keys
Passwords
Private keys
Access tokens
```

in source code.

Use:

```text
Kubernetes Secrets
External Secrets
Cloud secret management
```

as appropriate.

---

# 64. SDK Performance

Instrumentation introduces some overhead.

Potential overhead comes from:

```text
Span creation
Attribute processing
Context propagation
Metric recording
Exporting
Serialization
```

Production configuration should balance:

```text
Observability
Performance
Cost
```

---

# 65. Reducing SDK Overhead

Use:

```text
Sampling
Batching
Efficient instrumentation
Reasonable attribute counts
Selective instrumentation
Collector processing
```

Avoid generating telemetry that provides little operational value.

---

# 66. High-Cardinality Attributes

Be careful with attributes such as:

```text
user_id
request_id
session_id
full URL
random identifiers
```

High-cardinality attributes can increase backend resource usage significantly.

Before adding an attribute, ask:

```text
Will engineers actually query this?
```

---

# 67. SDK Sampling Strategy

For a high-volume production service:

```text
100,000 requests
      ↓
Sampling
      ↓
10,000 traces
```

But errors and important traces may need higher retention.

A more advanced architecture can use Collector tail sampling.

---

# 68. SDK Export Failure

If the Collector is unavailable:

```text
Application
     ↓
SDK Exporter
     X
Collector
```

The SDK should be configured according to the application's tolerance for:

```text
Retry
Queueing
Dropping telemetry
Export timeout
```

Telemetry should not be allowed to bring down the business application.

---

# 69. Observability Must Not Become a Dependency

A critical production principle:

```text
Application availability
        >
Observability availability
```

If the Collector is unavailable:

```text
Payment should continue processing
```

rather than:

```text
Payment fails because telemetry failed
```

Telemetry failures should be isolated from business traffic wherever possible.

---

# 70. SDK Timeouts

Exporters should use appropriate timeouts.

Conceptually:

```text
Application
   ↓
SDK
   ↓
Exporter timeout
   ↓
Collector
```

Avoid indefinite blocking of business operations because of telemetry export.

---

# 71. SDK Queueing

Some SDK implementations can queue telemetry before export.

Conceptually:

```text
Telemetry
   ↓
SDK Queue
   ↓
Exporter
   ↓
Collector
```

Queueing can help with temporary network interruptions but should be bounded.

---

# 72. Collector vs SDK Responsibilities

A useful separation:

```text
SDK
│
├── Instrumentation
├── Context
├── Resource
├── Sampling
└── Application-side export

Collector
│
├── Collection
├── Processing
├── Filtering
├── Routing
├── Batching
├── Centralized sampling
└── Backend export
```

Do not put every processing responsibility into the SDK.

---

# 73. Recommended Production Architecture

```text
Application
    │
    ↓
OTel SDK
    │
    ↓
OTLP
    │
    ↓
OTel Agent
    │
    ↓
OTel Gateway
    │
    ├──→ Prometheus
    │
    ├──→ Elasticsearch / ELK
    │
    └──→ Jaeger
```

This keeps applications relatively independent from observability backend implementations.

---

# 74. SDK Deployment Through GitOps

For a Kubernetes environment:

```text
GitHub
   ↓
Application Configuration
   ↓
Pull Request
   ↓
CI Validation
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Application + OTel Configuration
```

This provides consistent configuration across services.

---

# 75. SDK Version Management

Pin SDK and instrumentation versions where appropriate.

Example:

```text
Application
   ↓
OTel SDK vX
   ↓
Instrumentation vY
```

Do not automatically upgrade every observability dependency directly in production without validation.

Test upgrades in:

```text
Development
   ↓
Staging
   ↓
Production
```

---

# 76. SDK Upgrade Considerations

Before upgrading:

```text
Check compatibility
Review release notes
Test instrumentation
Test exporters
Check semantic changes
Run integration tests
Validate telemetry
```

Then deploy gradually.

---

# 77. SDK Troubleshooting

If traces are missing:

```text
Check SDK initialization
      ↓
Check instrumentation
      ↓
Check TracerProvider
      ↓
Check exporter
      ↓
Check endpoint
      ↓
Check Collector
```

If metrics are missing:

```text
Check MeterProvider
      ↓
Check instruments
      ↓
Check MetricReader
      ↓
Check exporter
      ↓
Check Collector
```

---

# 78. SDK Troubleshooting: No Logs

Check:

```text
LoggerProvider
Logging integration
Log exporter
Collector logs pipeline
Backend exporter
```

Then verify:

```text
Trace ID
Span ID
Service name
Severity
```

if log-trace correlation is expected.

---

# 79. SDK Troubleshooting: High Application CPU

Potential causes:

```text
Too much instrumentation
High span volume
Excessive attributes
High metric cardinality
Aggressive sampling/exporting
```

Approach:

```text
Measure
 ↓
Identify instrumentation overhead
 ↓
Reduce unnecessary telemetry
 ↓
Tune sampling
 ↓
Tune export
```

---

# 80. SDK Troubleshooting: High Application Memory

Potential causes:

```text
Large export queues
High telemetry volume
Large batches
Excessive attributes
Exporter problems
```

Approach:

```text
Check SDK queues
 ↓
Check Collector connectivity
 ↓
Reduce telemetry volume
 ↓
Tune batching
 ↓
Fix backend/export problems
```

---

# 81. SDK Troubleshooting: Broken Trace

If a distributed trace appears disconnected:

```text
Frontend
   ↓
Trace A

Orders
   ↓
Trace B

Payment
   ↓
Trace C
```

Check:

```text
Context propagation
traceparent
Instrumentation
HTTP middleware
Messaging propagation
```

The likely problem is context not being propagated correctly across service boundaries.

---

# 82. SDK Troubleshooting: Wrong Service Name

If all services appear under one service:

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
Instrumentation configuration
Deployment environment variables
```

---

# 83. SDK Troubleshooting: Duplicate Telemetry

Duplicate telemetry can occur if multiple instrumentation mechanisms are enabled.

Example:

```text
Auto Instrumentation
       +
Manual Instrumentation
       ↓
Same operation instrumented twice
```

Review instrumentation configuration and avoid duplicate instrumentation for the same library or operation.

---

# 84. SDK Troubleshooting: Telemetry Flood

Symptoms:

```text
CPU ↑
Memory ↑
Network ↑
Backend ingestion ↑
Storage ↑
```

Possible causes:

```text
Too many spans
High-cardinality metrics
Excessive logs
No sampling
No filtering
```

Actions:

```text
Reduce unnecessary telemetry
Tune sampling
Filter low-value events
Review instrumentation
```

---

# 85. SDK Production Checklist

```text
[ ] SDK version controlled
[ ] Service name configured
[ ] Service version configured
[ ] Environment configured
[ ] Resource attributes configured
[ ] Context propagation enabled
[ ] Appropriate instrumentation enabled
[ ] Sampling configured
[ ] Batch exporting configured
[ ] Export timeout configured
[ ] Collector endpoint configured
[ ] TLS configured where required
[ ] Sensitive data protected
[ ] High-cardinality attributes reviewed
[ ] Graceful shutdown configured
```

---

# 86. EKS SDK Checklist

```text
[ ] Application image contains required instrumentation
[ ] OTel endpoint uses Kubernetes Service DNS
[ ] Service name is unique
[ ] Namespace identified
[ ] Cluster metadata available
[ ] OTLP protocol matches Collector
[ ] NetworkPolicy allows telemetry traffic
[ ] TLS configured where required
[ ] Collector HA configured
[ ] Application does not depend on Collector availability
```

---

# 87. Complete SDK Architecture

```text
                              USER
                                │
                                ↓
                               ALB
                                │
                                ↓
                          Microservice
                                │
                       ┌────────┴────────┐
                       ↓                 ↓
                  OTel API        Auto Instrumentation
                       │                 │
                       └────────┬────────┘
                                ↓
                            OTel SDK
                                │
               ┌────────────────┼────────────────┐
               ↓                ↓                ↓
            Tracer           Meter            Logger
               ↓                ↓                ↓
            Spans            Metrics          Logs
               │                │                │
               └────────────────┼────────────────┘
                                ↓
                         OTLP Exporter
                                ↓
                          OTel Collector
                                ↓
                         OTel Gateway
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
          Prometheus            ELK              Jaeger
              ↓                 ↓                 ↓
           Grafana            Kibana            Trace UI
```

---

# 88. Production SDK Mental Model

Remember the SDK using this sequence:

```text
1. IDENTIFY
   Resource
   Service Name
   Version
   Environment

2. INSTRUMENT
   Automatic
   Manual

3. CREATE
   Spans
   Metrics
   Logs

4. CORRELATE
   Context
   Trace ID
   Span ID
   Baggage

5. CONTROL
   Sampling
   Filtering
   Attributes

6. EXPORT
   OTLP
   Collector

7. SHUTDOWN
   Flush
   Export
   Close
```

---

# 89. Final Key Concepts

The most important OpenTelemetry SDK concepts are:

```text
API
SDK
TracerProvider
Tracer
Span
MeterProvider
Meter
Metric Instruments
LoggerProvider
Resource
Context
Propagation
Sampler
SpanProcessor
MetricReader
Exporters
```

The production relationship is:

```text
API
 ↓
SDK
 ↓
Instrumentation
 ↓
Telemetry
 ↓
Processors / Sampling
 ↓
Exporter
 ↓
Collector
 ↓
Backend
```

The key principle is:

**The OpenTelemetry SDK is the application-side observability engine. It provides the APIs and implementations needed to create traces, metrics, and logs, attach consistent resource information, propagate trace context, apply sampling and processing, and export telemetry. In a production EKS microservices environment, the SDK should remain lightweight and focused on application instrumentation while the OpenTelemetry Collector handles centralized processing, batching, filtering, routing, security, and backend integration.**
