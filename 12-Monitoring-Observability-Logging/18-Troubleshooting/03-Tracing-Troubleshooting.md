# 03 - Tracing Troubleshooting

> Production Distributed Tracing Troubleshooting — Trace/Span Concepts, Trace Context, Instrumentation, Sampling, Propagation, Jaeger, OpenTelemetry, Kubernetes, EKS, Microservices, Missing Traces, Partial Traces, Broken Context, High Cardinality, Performance, Storage, Security, Incident Response, Root Cause Analysis and Interview Preparation

---

# 1. Tracing Troubleshooting Fundamentals

Distributed tracing helps answer:

> What happened to one request as it moved through multiple services?

Typical architecture:

    Client
      |
      v
    ALB / Ingress
      |
      v
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

A trace represents the complete request journey.

A span represents one operation within that journey.

Example:

    Trace: abc123

        API Request
             |
             +---- Orders Service
             |
             +---- Payment Service
             |
             +---- Database Query

If tracing is broken, the engineer may see only one part of the request.

---

# 2. The Tracing Troubleshooting Mindset

Do not immediately assume:

> "Jaeger is broken."

Tracing can fail at multiple layers:

    Application
       |
       v
    Instrumentation
       |
       v
    Context Propagation
       |
       v
    Collector
       |
       v
    Backend
       |
       v
    UI / Query

The first question should be:

> Where is the last point where the trace is confirmed?

---

# 3. Distributed Trace Data Path

A common OpenTelemetry architecture:

    Application
        |
        v
    OTel SDK / Auto Instrumentation
        |
        v
    OTel Collector
        |
        v
    Jaeger
        |
        v
    Jaeger UI

Another architecture may send directly to the tracing backend:

    Application
        |
        v
    Tracing Backend

The troubleshooting method remains the same:

    Generate
       |
       v
    Propagate
       |
       v
    Collect
       |
       v
    Process
       |
       v
    Store
       |
       v
    Query

---

# 4. Trace vs Span

A trace is the complete distributed request.

A span is one operation.

Example:

    Trace ID: abc123

    Span 1: API request
       |
       +---- Span 2: Orders service
                  |
                  +---- Span 3: Payment service
                             |
                             +---- Span 4: Database query

If one downstream span is missing, the trace may still exist but be incomplete.

---

# 5. Parent and Child Spans

Example:

    API Span
       |
       +---- Database Span
       |
       +---- Payment Span

The API span is the parent.

Payment and database operations may be children.

If parent-child relationships are wrong:

- Trace topology becomes incorrect
- Service dependency maps become misleading
- Latency analysis becomes difficult

---

# 6. Trace ID

The trace ID identifies the overall distributed request.

Example:

    trace_id=4bf92f3577b34da6a3ce929d0e0e4736

Every related span should carry the same trace ID.

If:

    Service A -> trace_id=AAA
    Service B -> trace_id=BBB

then context propagation has probably failed.

---

# 7. Span ID

Each span has its own identifier.

Example:

    Trace ID = AAA

    API Span = 111
    Payment Span = 222
    DB Span = 333

All belong to:

    Trace ID = AAA

The span ID identifies an individual operation.

---

# 8. Trace Context

Trace context allows one service to tell another:

> This request belongs to this trace.

Common context includes:

    trace ID
    span ID
    trace flags
    trace state

HTTP requests commonly propagate context through headers.

A modern tracing implementation often uses W3C Trace Context.

---

# 9. Context Propagation Failure

Example:

    Service A
       |
       | trace_id=AAA
       v
    Service B
       |
       | trace_id=BBB
       v
    Service C

The trace is fragmented.

Symptoms:

- Multiple traces for one request
- Missing parent-child relationships
- Service map looks wrong
- Downstream latency cannot be correlated

---

# 10. Propagation Troubleshooting

Check:

- HTTP headers
- Messaging metadata
- Framework instrumentation
- Client libraries
- Service mesh behavior
- Custom middleware
- Async processing

Do not assume HTTP propagation automatically covers asynchronous messaging.

---

# 11. HTTP Trace Context

Typical concept:

    Request
       |
       v
    traceparent header
       |
       v
    Service B

If an intermediate proxy strips the header:

    Service A
       |
       v
    Proxy
       |
       X
    Trace Context
       |
       v
    Service B

Service B may create a new trace.

---

# 12. ALB / Ingress and Tracing

In an EKS environment:

    Client
      |
      v
    AWS ALB
      |
      v
    Ingress
      |
      v
    Kubernetes Service
      |
      v
    Pod

Do not assume the ALB itself provides complete application tracing.

Application instrumentation and context propagation still need to be verified.

For troubleshooting, establish:

    Client request
        |
        v
    ALB
        |
        v
    Application

and determine where the trace context starts.

---

# 13. Async Messaging and Trace Context

Example:

    Orders Service
        |
        v
    RabbitMQ
        |
        v
    Payment Worker

The trace context must be propagated through message metadata.

If not:

    Orders trace
         |
         v
      RabbitMQ
         |
         X
      Payment trace

The worker may start a new unrelated trace.

---

# 14. Trace Propagation Through RabbitMQ

A practical approach is to propagate trace context in message headers.

Conceptually:

    Message
      |
      +-- trace context
      |
      +-- business payload

Consumer extracts the context and creates a child span.

Troubleshoot:

- Producer instrumentation
- Message headers
- Consumer extraction
- Framework instrumentation
- Custom messaging code

---

# 15. Database Tracing

A database operation may appear as:

    Service
       |
       +---- DB Span

If DB spans are missing:

Check:

- Database client instrumentation
- Driver support
- Auto instrumentation
- Manual instrumentation
- Connection pool behavior

Do not assume every database call is automatically traced.

---

# 16. External API Tracing

Example:

    Orders
       |
       v
    Payment Provider

If outbound HTTP instrumentation is enabled:

    Orders Span
       |
       +---- HTTP Client Span

If missing:

- Check HTTP client instrumentation
- Check context propagation
- Check library compatibility
- Check sampling

---

# 17. Application Instrumentation

Tracing requires instrumentation.

Possible approaches:

- Auto instrumentation
- SDK instrumentation
- Manual instrumentation

Auto instrumentation is convenient.

Manual instrumentation is useful when:

- Business operations need explicit spans
- Custom protocols are used
- Important internal operations are not automatically captured

---

# 18. Instrumentation Failure

Symptoms:

- Application works
- Metrics work
- Logs work
- No traces

Possible causes:

- SDK not initialized
- Agent not loaded
- Wrong endpoint
- Sampling configuration
- Exporter failure
- Unsupported framework

First verify instrumentation itself.

---

# 19. Java Application Tracing

For Java:

    Application
       |
       v
    Java Agent / SDK
       |
       v
    OTel Collector
       |
       v
    Tracing Backend

Check:

- Agent loaded
- Java startup arguments
- OTEL environment variables
- Exporter endpoint
- Application framework compatibility

If the application starts without the tracing agent, traces may not exist.

---

# 20. Node.js Application Tracing

Typical areas:

- Node.js SDK
- Auto instrumentation
- HTTP client
- Express / framework instrumentation
- Database instrumentation

Check initialization order.

Instrumentation usually needs to be initialized before relevant modules are loaded.

---

# 21. Python Application Tracing

Check:

- OTel package installation
- Instrumentation package
- Application startup
- Exporter
- Endpoint
- Environment variables

For frameworks such as Flask or FastAPI, verify that framework instrumentation is actually enabled.

---

# 22. Trace Exporter Troubleshooting

Application may generate spans but fail to export them.

Architecture:

    Application
       |
       v
    Span created
       |
       X
    Exporter
       |
       v
    Collector

Check:

- Endpoint
- Port
- Protocol
- TLS
- Authentication
- Network
- Retry behavior

---

# 23. OTLP Endpoint Troubleshooting

If using OpenTelemetry:

    Application
       |
       v
    OTLP
       |
       v
    Collector

A wrong endpoint can cause:

    Connection refused
    Timeout
    DNS failure
    TLS error

Verify the exact endpoint and protocol.

---

# 24. OTLP Protocol Confusion

Common OTLP transport choices include:

- OTLP/gRPC
- OTLP/HTTP

A configuration expecting one protocol may fail against an endpoint configured for another.

When troubleshooting, verify:

    Protocol
    Host
    Port
    TLS
    Path where applicable

Do not troubleshoot only by hostname.

---

# 25. DNS Troubleshooting

In Kubernetes:

    Application
       |
       v
    otel-collector.logging.svc
       |
       v
    Service
       |
       v
    Collector

If DNS fails:

    Collector cannot be reached.

Check:

    kubectl get svc -n logging

and:

    kubectl get endpoints -n logging

Also verify DNS resolution from the application pod where appropriate.

---

# 26. Kubernetes Service With No Endpoints

Example:

    OTel Service
       |
       X
    No endpoints

The application can resolve the service name but there is nowhere to send the traffic.

Check:

    kubectl get endpoints <service> -n <namespace>

Then:

    kubectl get pods -n <namespace> -o wide

Check selector mismatch.

---

# 27. Collector Pod Not Ready

Check:

    kubectl get pods -n observability

Look for:

- CrashLoopBackOff
- Pending
- OOMKilled
- Readiness failure
- ImagePullBackOff

Then:

    kubectl logs <collector> -n observability

and:

    kubectl describe pod <collector> -n observability

---

# 28. OTel Collector Configuration

A simplified Collector pipeline:

    receivers
        |
        v
    processors
        |
        v
    exporters

Example concept:

    OTLP Receiver
        |
        v
    Batch Processor
        |
        v
    Jaeger / OTLP Exporter

If traces enter but do not leave, inspect processors and exporters.

---

# 29. Collector Receiver Troubleshooting

If no traces arrive:

Check:

- Receiver enabled
- Correct protocol
- Correct port
- Service endpoint
- Network policy
- Application exporter configuration

A healthy collector process does not guarantee the receiver is configured correctly.

---

# 30. Collector Processor Troubleshooting

Processors can:

- Batch
- Filter
- Transform
- Enrich
- Sample

An incorrect processor configuration can change or drop telemetry.

When traces disappear:

    Receiver = YES
    Processor = ?
    Exporter = ?

Trace the pipeline stage by stage.

---

# 31. Collector Exporter Troubleshooting

If receiver metrics show spans but backend has none:

Investigate exporter.

Possible causes:

- Backend unreachable
- Wrong endpoint
- Authentication
- TLS
- Queue overflow
- Export failures

Check collector logs and telemetry metrics.

---

# 32. Collector Batch Processing

Batching improves efficiency.

Architecture:

    Spans
      |
      v
    Batch
      |
      v
    Export

If batch configuration is too aggressive:

- Traces may be delayed
- Shutdown may lose buffered data depending on configuration

If batching is too small:

- More network calls
- Higher overhead

Troubleshoot latency versus throughput.

---

# 33. Sampling

Sampling determines which traces are retained.

Common concepts:

- Always sample
- Never sample
- Probabilistic sampling
- Parent-based sampling
- Tail sampling

If traces are missing, sampling may be intentional rather than a failure.

---

# 34. Head Sampling

Decision is made early.

Example:

    1000 requests
       |
       v
    10% sampling
       |
       v
    ~100 traces

Advantages:

- Lower cost
- Lower overhead

Tradeoff:

- Important traces may be discarded before their outcome is known.

---

# 35. Tail Sampling

Decision can be made after observing the trace.

Example:

    Keep:
    ERROR traces
    Slow traces

    Drop:
    Normal fast traces

This is useful for production observability but requires buffering and more infrastructure.

---

# 36. Sampling Troubleshooting

If users report:

> "The error happened but I cannot find the trace."

Check:

- Sampling percentage
- Parent-based sampling
- Tail sampling rules
- Error sampling policy
- Trace retention

Do not immediately blame instrumentation.

---

# 37. Error Traces Should Be Prioritized

A production strategy may retain more:

- Errors
- High latency requests
- Important transactions
- Security events

while sampling normal traffic.

This balances:

    Observability
    +
    Cost

---

# 38. Trace Not Found in Jaeger

Possible causes:

1. Request was not sampled.
2. Instrumentation did not create spans.
3. Context propagation failed.
4. Exporter failed.
5. Collector failed.
6. Backend rejected data.
7. Retention removed the trace.
8. Search query/time range is wrong.

Use this list systematically.

---

# 39. Jaeger UI Troubleshooting

If Jaeger is reachable but no traces appear:

Check:

- Service name filter
- Time range
- Tags
- Trace ID
- Backend connectivity
- Sampling
- Ingestion

Start with a broad time range and service selection.

---

# 40. Trace Search by Trace ID

If you know:

    trace_id=abc123

search directly.

This is better than starting with a broad service search when investigating a specific request.

If the trace ID cannot be found:

    Check whether it was exported
    Check retention
    Check backend

---

# 41. Trace Search by Service

Start with:

    Service = orders

Then narrow:

    Error = true

Then:

    Duration > threshold

Then:

    Specific endpoint

Build queries progressively.

---

# 42. Trace Search by Duration

Slow request:

    Duration = 8 seconds

Find trace.

Then inspect:

    API span = 8 sec
       |
       +---- DB = 7.5 sec

The trace immediately identifies the bottleneck.

---

# 43. Trace Search by Error

Example:

    Error = true

Then inspect:

    Exception type
    Error message
    Service
    Endpoint
    Child spans

This is useful during incident response.

---

# 44. Missing Child Span

Trace:

    API
       |
       +---- Payment
       |
       +---- Database

Expected:

    Payment child span

Actual:

    API
       |
       +---- Database

Possible causes:

- Payment instrumentation missing
- Context not propagated
- Sampling differences
- Async boundary
- Export failure

Determine whether the child span was never created or was created but not exported.

---

# 45. Partial Trace

A partial trace may look like:

    Service A
       |
       v
    Service B
       |
       X
    Service C missing

Possible causes:

- Service C not instrumented
- Context lost
- Service C not exporting
- Sampling
- Backend ingestion issue

Partial traces are common in mixed-instrumentation environments.

---

# 46. Broken Parent-Child Relationship

Example:

    Trace AAA
      |
      +---- Service A
      |
      +---- Service B

but Service B should be a child of Service A.

If Service B has:

    parent = none

context propagation likely failed.

Check HTTP/message context.

---

# 47. Wrong Service Name

If every service appears as:

    unknown_service

or:

    service

troubleshooting becomes difficult.

Set stable service identity.

Useful attributes:

    service.name
    service.version
    deployment.environment

Do not create random service names on every deployment.

---

# 48. Service Name Changes

Example:

    orders

becomes:

    orders-service

Dashboards and service dependency queries may appear broken.

When changing service names:

- Update dashboards
- Update alerts
- Update runbooks
- Update queries
- Communicate the change

---

# 49. Trace Attributes

Useful attributes:

    http.method
    http.route
    http.status_code
    service.name
    deployment.environment
    cloud.region
    k8s.namespace.name
    k8s.pod.name

These help locate the exact workload.

Avoid uncontrolled high-cardinality attributes.

---

# 50. High Cardinality in Tracing

Bad attribute:

    user_id

if millions of unique users are indexed aggressively.

Other examples:

    session_id
    random request payload
    unique URL query strings

High cardinality can increase:

- Storage
- Query cost
- Backend memory
- Index pressure

Choose attributes carefully.

---

# 51. URL Cardinality Problem

Bad:

    /users/12345
    /users/12346
    /users/12347

Better:

    /users/{id}

Framework instrumentation should capture route templates where supported.

This keeps metrics and trace dimensions manageable.

---

# 52. Sensitive Data in Traces

Do not blindly record:

- Passwords
- Tokens
- Authorization headers
- Full request bodies
- Credit card data
- Secrets

Tracing attributes can become a security risk.

Use redaction and controlled attribute collection.

---

# 53. Trace Sampling and Security

Do not assume sampling makes sensitive data safe.

A sampled trace can still contain sensitive information.

Security controls must exist regardless of sampling.

---

# 54. Trace Storage Retention

If an old trace cannot be found:

Check:

- Retention period
- Storage policy
- Index lifecycle
- Backend cleanup
- Time range

Trace backends are not necessarily permanent archives.

---

# 55. Jaeger Storage Troubleshooting

Depending on architecture, Jaeger may use storage such as:

- Elasticsearch
- Other supported storage backends

If storage is unhealthy:

    Jaeger Collector
        |
        v
    Storage
        |
        X
    Trace unavailable

Check storage health before blaming Jaeger UI.

---

# 56. Jaeger Collector Troubleshooting

Typical path:

    Application
       |
       v
    Collector
       |
       v
    Storage

Check:

- Collector pods
- Receiver ports
- Export/storage connection
- Authentication
- Resource usage
- Errors

---

# 57. Jaeger Query Troubleshooting

Path:

    Jaeger UI
       |
       v
    Query Service
       |
       v
    Storage

If traces are stored but search fails:

    Query layer

may be the problem.

Separate:

    Ingestion

from:

    Query

---

# 58. OpenTelemetry Collector vs Jaeger

In a modern architecture:

    Application
       |
       v
    OTel Collector
       |
       +---- Metrics
       +---- Logs
       +---- Traces
       |
       v
    Backends

Jaeger may be only the trace backend/UI.

If traces fail, determine whether the issue is:

    Application
    OTel Collector
    Jaeger
    Storage
    Query

---

# 59. Kubernetes OTel Collector Deployment Models

Common models:

    DaemonSet
    Deployment
    Gateway

DaemonSet is useful for node-local collection.

Gateway is useful for centralized processing.

Architecture may be:

    Pods
      |
      v
    OTel Collector DaemonSet
      |
      v
    OTel Gateway
      |
      v
    Backend

Troubleshoot each layer separately.

---

# 60. Collector DaemonSet Troubleshooting

Check:

    kubectl get daemonset -n observability

Then:

    kubectl get pods -n observability -o wide

Ensure all required nodes have collectors.

A missing collector creates an observability gap.

---

# 61. Collector Gateway Troubleshooting

If node collectors are healthy but traces are missing centrally:

    DaemonSet
       |
       v
    Gateway
       |
       X
    Backend

Check:

- Gateway service
- Endpoints
- Network
- Resource usage
- Pipeline configuration
- Exporter

---

# 62. OOMKilled Collector

If collector restarts:

    kubectl describe pod <collector> -n observability

Check:

    Last State
    Reason: OOMKilled

Possible causes:

- High trace volume
- Large batches
- Tail sampling
- Too many attributes
- Insufficient memory

Do not simply increase memory without checking telemetry volume.

---

# 63. Collector CPU Saturation

Possible causes:

- High span volume
- Expensive processors
- Attribute transformations
- Tail sampling
- Export bottleneck

Check:

    CPU
    throughput
    queue
    dropped spans

Scale or optimize the actual bottleneck.

---

# 64. NetworkPolicy Troubleshooting

A NetworkPolicy may block:

    Application -> Collector

or:

    Collector -> Backend

Symptoms:

    Connection timeout

Check:

- Namespace
- Pod selectors
- Ports
- Egress rules
- Ingress rules

Always inspect network controls before changing application code.

---

# 65. EKS Security Group Troubleshooting

If tracing crosses network boundaries:

    Pod
       |
       v
    Node / ENI
       |
       v
    Backend

Security Groups or network routing can block traffic.

Check:

- Security Group rules
- Route tables
- DNS
- VPC connectivity
- Endpoint accessibility

---

# 66. Tracing in Private EKS Subnets

If applications and collectors are private:

    Private Pod
       |
       v
    Private Collector
       |
       v
    Private Backend

No public Internet path is necessarily required.

Verify routing according to the actual deployment architecture.

---

# 67. Trace Export TLS Troubleshooting

Symptoms:

    TLS handshake failure
    Certificate error
    Unknown CA

Check:

- Certificate validity
- CA bundle
- Hostname
- TLS settings
- Secret mounted correctly

Do not disable TLS verification as a permanent fix.

---

# 68. Trace Export Authentication

Possible failures:

    401
    403

Check:

- Credentials
- API key
- Secret
- IAM integration where applicable
- Permissions

Use least privilege.

---

# 69. Trace Data Loss During Restart

Buffered spans may be lost depending on configuration.

Consider:

- Batch processor
- Queue
- Persistent buffering
- Shutdown behavior

Graceful shutdown helps flush in-flight telemetry.

---

# 70. Collector Graceful Shutdown

During deployment:

    Collector
       |
       v
    Stop accepting
       |
       v
    Flush buffered spans
       |
       v
    Exit

This reduces telemetry loss during planned changes.

---

# 71. Tracing During Autoscaling

Example:

    Pods: 10 -> 50

Trace volume increases.

Potential effects:

- Collector CPU ↑
- Network ↑
- Backend ingestion ↑
- Storage ↑

Autoscaling the application should be considered in tracing capacity planning.

---

# 72. Trace Volume Spike

Possible causes:

- Traffic spike
- Instrumentation duplication
- Sampling changed
- Retry storm
- Error storm

Check:

    Requests/sec
    Spans/sec
    Bytes/sec

Compare trace volume with application traffic.

---

# 73. Duplicate Spans

Possible causes:

- Multiple instrumentation mechanisms
- Auto + manual instrumentation
- Duplicate SDK initialization
- Collector duplication
- Retries

Example:

    Auto instrumentation
          +
    Manual instrumentation
          |
          v
    Duplicate spans

Avoid instrumenting the same operation twice unintentionally.

---

# 74. Double Instrumentation

Example:

    HTTP client auto instrumentation
          +
    Manual HTTP span

Result:

    Parent HTTP span
       |
       +---- Manual HTTP span
       |
       +---- Auto HTTP span

This may be legitimate in some designs but can also create noisy traces.

Review instrumentation boundaries.

---

# 75. Trace Overhead

Tracing adds overhead through:

- Span creation
- Context propagation
- Attribute processing
- Export
- Network
- Storage

High sampling rates can increase application and infrastructure overhead.

Production tracing requires controlled sampling.

---

# 76. Debugging Instrumentation Without Production Risk

Prefer:

    Development
       |
       v
    Staging
       |
       v
    Small production scope
       |
       v
    Full rollout

Validate:

- Trace creation
- Context propagation
- Export
- Backend
- Performance

Do not enable maximum tracing globally as the first troubleshooting step.

---

# 77. Trace Context and Retries

Suppose:

    Service A
       |
       v
    Service B
       |
       X timeout
       |
       v
    Retry

The retry may be represented as:

- Separate span
- Child attempt span
- New operation depending on instrumentation

Understand your instrumentation semantics before interpreting trace latency.

---

# 78. Trace Context and Async Workers

Example:

    API
      |
      v
    Queue
      |
      v
    Worker

There may be a time gap between producer and consumer.

Trace design should preserve useful relationship information while respecting asynchronous semantics.

A broken implementation can produce unrelated traces.

---

# 79. Trace Context Through Proxies

Proxies may:

- Preserve headers
- Modify headers
- Remove headers
- Create new context

If tracing breaks after introducing a proxy:

    Compare request headers before/after proxy.

This is especially important with:

- Ingress
- Service mesh
- API proxies
- Load balancers

---

# 80. Service Mesh Tracing

A service mesh can generate telemetry at the proxy layer.

Architecture:

    Application
       |
       v
    Sidecar Proxy
       |
       v
    Sidecar Proxy
       |
       v
    Application

This can provide network-level visibility.

But application-level instrumentation may still be needed for business operations.

---

# 81. Application vs Infrastructure Tracing

Infrastructure tracing can show:

    HTTP request
    Network request
    Service-to-service latency

Application tracing can show:

    Checkout
    Validate payment
    Reserve inventory
    Create order

Both have value.

Do not confuse proxy-level spans with complete business-level tracing.

---

# 82. Tracing and Logs Correlation

A strong observability setup can connect:

    Trace
       |
       +---- Logs
       |
       +---- Metrics

For example:

    Trace ID
       |
       v
    Application logs
       |
       v
    Error details

This dramatically improves incident investigation.

---

# 83. Trace and Log Correlation Troubleshooting

If trace exists but logs cannot be correlated:

Check:

- Trace ID injection into logs
- Log format
- Collector fields
- Elasticsearch mapping
- Kibana query

If logs contain:

    trace_id=abc

you can search for all events associated with that trace.

---

# 84. Trace and Metrics Correlation

Metrics tell you:

    Error rate ↑
    Latency ↑

Tracing tells you:

    Which request?
    Which service?
    Which operation?

Use:

    Metrics -> detect
    Traces -> isolate
    Logs -> explain

This is a useful incident-response model.

---

# 85. Production Incident — No Traces at All

Symptoms:

- Application healthy
- Logs healthy
- Metrics healthy
- Traces absent

Investigation:

    Instrumentation
       |
       v
    Exporter
       |
       v
    Collector
       |
       v
    Backend
       |
       v
    Sampling

Check each layer.

---

# 86. Production Incident — Only Some Services Have Traces

Example:

    API -> Orders -> Payment -> Inventory

Traces show:

    API
    Orders

but not:

    Payment
    Inventory

Possible causes:

- Partial instrumentation
- Context propagation
- Exporter configuration
- Sampling
- Network

Compare configuration across services.

---

# 87. Production Incident — Each Service Has a Separate Trace

Symptoms:

    Trace A = API
    Trace B = Orders
    Trace C = Payment

Likely cause:

    Trace context not propagated.

Check:

- HTTP headers
- Messaging metadata
- Client instrumentation
- Proxy behavior

---

# 88. Production Incident — Traces Appear but Are Missing Database Spans

Check:

- Database driver instrumentation
- SDK initialization
- Supported version
- Connection pool instrumentation
- Sampling

Do not assume the database itself is invisible.

Instrumentation must exist around the database operation.

---

# 89. Production Incident — Traces Suddenly Disappear After Deployment

Timeline:

    Deployment
       |
       v
    Traces disappear

Check:

- Environment variables
- Agent configuration
- Collector endpoint
- Service name
- Sampling
- Startup arguments
- NetworkPolicy
- Secret changes

Compare the previous and new deployment manifests.

---

# 90. Production Incident — Traces Delayed

Symptoms:

    Request completed
    Trace appears several minutes later

Possible causes:

- Collector queue
- Batch processor
- Backend ingestion
- Storage pressure
- Network delay

Compare event timestamp and ingestion timestamp.

---

# 91. Production Incident — Trace Backend Storage Full

Symptoms:

- New traces unavailable
- Collector export errors
- Backend health degraded

Investigate:

- Storage utilization
- Retention
- Trace volume
- Sampling
- Index/shard strategy where applicable

Restore safe capacity first, then fix the volume or retention problem.

---

# 92. Production Incident — Collector OOMKilled

Timeline:

    Traffic ↑
       |
       v
    Trace volume ↑
       |
       v
    Collector memory ↑
       |
       v
    OOMKilled
       |
       v
    Trace gaps

Investigate:

- Sampling
- Batch size
- Queue
- Attributes
- Tail sampling
- Collector replicas

---

# 93. Production Incident — Duplicate Traces

Possible causes:

- Duplicate instrumentation
- Duplicate collectors
- Multiple exporters
- Retries
- Same telemetry sent through two paths

Trace the complete path.

Do not delete backend data before understanding the source.

---

# 94. Production Incident — Wrong Service Map

Symptoms:

    Expected:
    API -> Orders -> Payment

Actual:

    API
    Payment
    Orders

Possible causes:

- Context propagation
- Incorrect service names
- Missing parent relationships
- Sampling
- Proxy instrumentation

Service maps are only as accurate as telemetry context.

---

# 95. Production Incident — Trace Exists but UI Search Cannot Find It

Check:

- Trace retention
- Search time range
- Service filter
- Backend indexing
- Query service
- Trace ID
- UI/backend connectivity

If direct backend query can find it but UI cannot:

    UI/query layer

is the likely problem.

---

# 96. Production Incident — High Trace Storage Cost

Investigate:

- Sampling rate
- Span count per trace
- Large attributes
- Long retention
- High traffic
- Duplicate spans
- Full payload capture

Do not reduce sampling blindly.

Protect traces needed for:

- Errors
- High latency
- Critical workflows

---

# 97. Production Incident — Sensitive Data in Trace Attributes

Actions:

    Identify field
       |
       v
    Stop collection
       |
       v
    Redact application
       |
       v
    Rotate secret if required
       |
       v
    Restrict access
       |
       v
    Assess retention
       |
       v
    Prevent recurrence

Treat this as a security issue.

---

# 98. Tracing Troubleshooting Checklist — Application

    [ ] SDK/agent loaded
    [ ] Instrumentation enabled
    [ ] Service name correct
    [ ] Sampling expected
    [ ] Exporter configured
    [ ] Endpoint correct
    [ ] Protocol correct
    [ ] TLS correct
    [ ] Context propagated
    [ ] No duplicate instrumentation
    [ ] No sensitive attributes

---

# 99. Tracing Troubleshooting Checklist — Collector

    [ ] Collector running
    [ ] Receiver enabled
    [ ] Receiver port reachable
    [ ] Processor valid
    [ ] Exporter valid
    [ ] Queue healthy
    [ ] CPU healthy
    [ ] Memory healthy
    [ ] No dropped spans
    [ ] Backend reachable

---

# 100. Tracing Troubleshooting Checklist — Backend

    [ ] Backend healthy
    [ ] Storage available
    [ ] Ingestion working
    [ ] Query working
    [ ] Retention correct
    [ ] No indexing/storage failures
    [ ] UI connected
    [ ] Service names visible

---

# 101. Tracing Troubleshooting Decision Tree

    Trace Missing?
          |
          v
    Was span created?
       /       \
     No         Yes
     |           |
Instrumentation  Exporter
problem          |
                v
             Collector
                |
                v
             Backend
                |
                v
             Query/UI

If only downstream services are missing:

    Check context propagation first.

---

# 102. Trace Troubleshooting by Symptom

## No traces

Check:

    Instrumentation
    Sampling
    Exporter
    Collector
    Backend

## Partial traces

Check:

    Propagation
    Service instrumentation
    Export

## Wrong service map

Check:

    Service names
    Parent relationships
    Context

## Slow traces

Check:

    Backend
    Query
    Sampling
    Storage

## High cost

Check:

    Sampling
    Span volume
    Attributes
    Retention

---

# 103. Evidence Collection During Tracing Incidents

Collect:

- Application logs
- Application configuration
- OTel environment variables
- Collector logs
- Collector configuration
- Backend health
- Trace IDs
- Deployment version
- Kubernetes events
- Network errors
- Sampling configuration

Preserve the evidence before making major changes.

---

# 104. Tracing Troubleshooting With Kubernetes

Useful commands:

    kubectl get pods -n observability

    kubectl describe pod <pod> -n observability

    kubectl logs <pod> -n observability

    kubectl get svc -n observability

    kubectl get endpoints -n observability

    kubectl get daemonset -n observability

Use them to establish:

    Pod health
    Service health
    Endpoint health
    Collector placement

---

# 105. Tracing Troubleshooting With EKS

Check:

    EKS nodes
    Pod networking
    Security Groups
    NetworkPolicies
    DNS
    Collector placement
    IAM where applicable
    Private subnet routing

A trace export failure may be infrastructure-related even when the application is healthy.

---

# 106. Tracing Troubleshooting After Terraform Changes

If tracing fails after Terraform:

    Identify changed resources

Potential areas:

- Security Groups
- IAM
- EKS node configuration
- Load Balancers
- DNS
- Networking

Compare:

    Before
       |
       v
    Terraform plan
       |
       v
    After

Do not change tracing code until infrastructure changes are ruled out.

---

# 107. Tracing Troubleshooting After ArgoCD Deployment

If traces disappear after GitOps sync:

Check:

    Git diff
       |
       v
    ArgoCD diff
       |
       v
    Deployment environment
       |
       v
    OTel configuration
       |
       v
    Collector endpoint

Rollback if necessary, then identify the configuration regression.

---

# 108. Tracing Configuration as Code

Store:

- Collector configuration
- Kubernetes manifests
- Helm values
- Instrumentation configuration
- Sampling rules
- Backend configuration

in Git where practical.

Benefits:

- Review
- Audit
- Reproducibility
- Rollback

---

# 109. Tracing Change Validation

Before production:

    Build
       |
       v
    Deploy staging
       |
       v
    Generate test request
       |
       v
    Verify trace
       |
       v
    Verify child spans
       |
       v
    Verify context
       |
       v
    Verify backend
       |
       v
    Verify dashboards

Tracing should be tested as part of deployment validation.

---

# 110. Synthetic Trace Test

A useful operational test:

    Send known request
        |
        v
    Generate unique request ID
        |
        v
    Search trace backend
        |
        v
    Verify expected services
        |
        v
    Verify latency
        |
        v
    Verify errors

This can detect broken tracing before a real incident.

---

# 111. Trace Health Monitoring

Monitor:

    Spans received
    Spans exported
    Spans dropped
    Export errors
    Collector queue
    Collector CPU
    Collector memory
    Backend ingestion
    Query latency

Tracing should be observable itself.

---

# 112. Trace Ingestion Rate

Measure:

    spans/sec

Compare with:

    requests/sec

If requests remain stable but spans/sec suddenly becomes 10x:

Possible causes:

- Duplicate instrumentation
- More spans per request
- Sampling change
- Instrumentation bug

---

# 113. Span Count Per Trace

One request may produce:

    5 spans

Another:

    500 spans

A sudden increase can cause:

- Storage growth
- CPU increase
- Collector pressure

Investigate which instrumentation created the extra spans.

---

# 114. Span Explosion

Example:

    One request
       |
       +---- API
       +---- DB
       +---- Cache
       +---- HTTP
       +---- Internal method
       +---- Internal method
       +---- ...
       |
       v
    500 spans

Manual instrumentation may be too granular.

Trace useful business operations rather than every trivial function.

---

# 115. Sampling Strategy Troubleshooting

A good production strategy may be:

    Normal requests
       |
       v
    Low sampling

    Errors
       |
       v
    Higher sampling

    Slow requests
       |
       v
    Higher sampling

This gives more useful traces without storing every request.

---

# 116. Tail Sampling Troubleshooting

If tail sampling is used:

Check:

- Decision rules
- Memory
- Trace buffering
- Timeout
- Error criteria
- Latency criteria

Incorrect tail sampling can drop the traces you actually need.

---

# 117. Trace Retention Strategy

Retention should reflect:

- Incident investigation needs
- Cost
- Compliance
- Traffic
- Storage capacity

Do not retain everything indefinitely.

Do not retain so little that production incidents cannot be investigated.

---

# 118. Tracing Security Best Practices

1. Encrypt telemetry in transit.
2. Restrict backend access.
3. Use least privilege.
4. Avoid sensitive attributes.
5. Redact secrets.
6. Protect collector endpoints.
7. Secure credentials.
8. Monitor access.
9. Review retention.
10. Separate environments where required.

---

# 119. Tracing Performance Best Practices

1. Use controlled sampling.
2. Avoid excessive span creation.
3. Avoid huge attributes.
4. Batch exports.
5. Monitor collector resources.
6. Scale collectors appropriately.
7. Avoid unnecessary synchronous export work.
8. Test instrumentation overhead.
9. Monitor backend capacity.
10. Optimize high-volume services first.

---

# 120. Tracing Cost Optimization

Major cost drivers:

    Request volume
    Sampling rate
    Spans/request
    Attribute size
    Retention
    Storage backend
    Query volume

Optimization:

    Keep useful traces
       |
       v
    Reduce unnecessary spans
       |
       v
    Sample intelligently
       |
       v
    Control retention

---

# 121. Root Cause Analysis — Missing Traces

Example:

    Root cause:
    OTel exporter pointed to an outdated collector service.

    Impact:
    Application traces were not exported.

    Detection gap:
    No alert on exporter failure.

    Contributing factor:
    Configuration changed without synthetic trace validation.

    Prevention:
    CI configuration validation + trace health monitoring.

---

# 122. Root Cause Analysis — Broken Trace Context

Example:

    Root cause:
    Custom HTTP client did not propagate W3C trace context.

    Impact:
    Downstream service created independent traces.

    Detection gap:
    No cross-service trace validation.

    Prevention:
    Standard HTTP client instrumentation and integration testing.

---

# 123. Root Cause Analysis — Collector OOM

Example:

    Root cause:
    Trace volume increased after instrumentation change.

    Contributing factors:
    High span count and insufficient collector memory.

    Impact:
    Collector restarts and partial trace loss.

    Prevention:
    Capacity testing + sampling + collector scaling + memory alerts.

---

# 124. Five Whys — Missing Trace

Why was the trace missing?

    Collector did not receive it.

Why?

    Application exporter endpoint was incorrect.

Why?

    Collector service name changed.

Why?

    Kubernetes service was renamed during deployment.

Why?

    Tracing endpoint was not validated after the deployment.

Root prevention:

> Add automated trace connectivity validation to deployment checks.

---

# 125. Five Whys — Trace Backend Full

Why are new traces unavailable?

    Backend storage is full.

Why?

    Trace volume increased.

Why?

    Sampling changed from 10% to 100%.

Why?

    Debug configuration was enabled globally.

Why?

    No production guardrail prevented high-volume sampling.

Prevention:

    Configuration validation + sampling guardrails.

---

# 126. Senior-Level Tracing Troubleshooting Framework

Use:

    DETECT
       |
       v
    IDENTIFY TRACE SCOPE
       |
       v
    VERIFY INSTRUMENTATION
       |
       v
    VERIFY CONTEXT
       |
       v
    VERIFY EXPORT
       |
       v
    VERIFY COLLECTOR
       |
       v
    VERIFY BACKEND
       |
       v
    VERIFY QUERY
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

---

# 127. Interview Question — No Traces Are Appearing. What Do You Check?

Strong answer:

> I first verify whether the application is actually creating spans. Then I check sampling and exporter configuration, endpoint and protocol, collector receiver and pipeline health, backend ingestion and finally the UI/query layer. I determine the last confirmed point before changing anything.

---

# 128. Interview Question — Why Are Traces Fragmented Across Services?

Strong answer:

> The most likely causes are broken trace context propagation, unsupported or incorrectly initialized instrumentation, proxy/header manipulation, or asynchronous messaging without context propagation. I would inspect the trace IDs at each service boundary and verify the propagation headers or message metadata.

---

# 129. Interview Question — How Do You Troubleshoot Missing Child Spans?

Strong answer:

> I first determine whether the child operation was instrumented and whether the span was created. If it was created, I check export and collector behavior. If it was never created, I check the relevant framework or client instrumentation. I also verify sampling and context propagation.

---

# 130. Interview Question — How Do You Reduce Tracing Cost?

Strong answer:

> I control sampling, prioritize error and slow traces, reduce unnecessary span creation, avoid large/high-cardinality attributes, batch telemetry, optimize collector capacity and apply appropriate retention. I avoid simply disabling tracing because that removes valuable production visibility.

---

# 131. Interview Question — What Is the Difference Between Head and Tail Sampling?

Strong answer:

> Head sampling makes the decision early, so it is efficient but cannot know the final outcome of the trace. Tail sampling makes the decision after observing the trace, which allows policies such as retaining errors and slow requests, but requires buffering and more resources.

---

# 132. Interview Question — How Do You Troubleshoot Trace Context Propagation?

Strong answer:

> I capture the trace ID at the source and compare it at each service boundary. For HTTP I inspect the propagation headers, and for asynchronous messaging I inspect message metadata. If the downstream service starts a new trace, I determine where the context was lost.

---

# 133. Interview Question — How Do You Correlate Traces With Logs?

Strong answer:

> I include trace ID and, where useful, span ID in structured application logs. During an incident I can start from a slow or failed trace and search the centralized logs using the same trace ID. This lets me combine the timing information from tracing with detailed error information from logs.

---

# 134. Interview Question — How Do You Troubleshoot OTel Collector OOMKilled?

Strong answer:

> I check span volume, batch size, queue configuration, tail sampling, attribute size and collector CPU/memory. I determine whether the workload increased or configuration became more expensive. Then I mitigate through scaling, sampling or configuration optimization rather than only increasing memory.

---

# 135. Interview Question — How Do You Troubleshoot Tracing in EKS?

Strong answer:

> I check application instrumentation and exporter configuration first, then Kubernetes service discovery, collector pods, receiver ports, endpoints, NetworkPolicies, DNS and network connectivity. In EKS I also consider node placement, security groups, VPC routing and private networking depending on the architecture.

---

# 136. Interview Question — What Would You Monitor for the Tracing Platform?

Strong answer:

> I would monitor spans received and exported, dropped spans, exporter errors, collector CPU and memory, queue depth, backend ingestion rate, backend storage, query latency and tracing availability. The tracing system itself needs observability.

---

# 137. Interview Question — Why Are Duplicate Spans Appearing?

Strong answer:

> I would check for auto-instrumentation plus manual instrumentation of the same operation, duplicate SDK initialization, multiple collectors, multiple export paths and retry behavior. I would identify the exact point where duplication begins before changing the backend.

---

# 138. Interview Question — What Makes a Good Production Trace?

A useful trace should provide:

    Trace ID
    Service name
    Operation
    Start/end time
    Duration
    Parent relationship
    Error information
    Relevant attributes
    Environment
    Version

It should answer:

> Where did the request spend its time and where did it fail?

---

# 139. Production Tracing Troubleshooting Checklist

## Application

    [ ] Instrumentation loaded
    [ ] Correct service name
    [ ] Correct endpoint
    [ ] Correct protocol
    [ ] Sampling expected
    [ ] Context propagated
    [ ] No duplicate instrumentation
    [ ] Sensitive attributes removed

## Collector

    [ ] Receiver healthy
    [ ] Processor healthy
    [ ] Exporter healthy
    [ ] Queue healthy
    [ ] CPU healthy
    [ ] Memory healthy
    [ ] No dropped spans

## Backend

    [ ] Ingestion healthy
    [ ] Storage healthy
    [ ] Retention correct
    [ ] Query healthy
    [ ] UI healthy

---

# 140. Final Tracing Troubleshooting Mental Model

When no trace exists:

    INSTRUMENT
        |
        v
    SAMPLE
        |
        v
    EXPORT
        |
        v
    COLLECT
        |
        v
    STORE
        |
        v
    QUERY

When a trace is incomplete:

    PROPAGATION
        |
        v
    DOWNSTREAM INSTRUMENTATION
        |
        v
    EXPORT

When tracing is expensive:

    REQUEST RATE
        |
        v
    SPANS / REQUEST
        |
        v
    SAMPLING
        |
        v
    ATTRIBUTE SIZE
        |
        v
    RETENTION

---

# 141. Final Production Tracing Principles

Remember:

> A trace can exist while still being incomplete.

> Missing downstream spans often point to context propagation or instrumentation problems.

> Always distinguish "span was never created" from "span was created but not exported."

> Sampling can intentionally make a trace unavailable.

> A healthy collector process does not prove that its receiver and exporter are working.

> A healthy tracing backend does not prove that the application is exporting telemetry.

> High-cardinality and large attributes can create major tracing costs.

> Trace context must survive service and asynchronous boundaries.

> Tracing should be correlated with logs and metrics.

> The tracing platform itself must be monitored.

> Never disable TLS or security controls as a permanent troubleshooting solution.

The complete production tracing troubleshooting mindset is:

    DETECT
       +
    TRACE THE TRACE
       +
    ISOLATE
       +
    MITIGATE
       +
    VALIDATE
       +
    RCA
       +
    PREVENT

This is the approach expected from a production DevOps / DevSecOps engineer when distributed tracing fails during a real microservices or EKS incident.
