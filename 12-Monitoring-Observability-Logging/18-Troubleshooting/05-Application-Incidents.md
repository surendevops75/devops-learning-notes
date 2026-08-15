# 05 - Application Incidents

> Production Application Incident Troubleshooting — Java, Node.js, Python, Microservices, APIs, Databases, Caches, Queues, Memory, CPU, Threads, Connection Pools, Timeouts, Retries, HTTP Errors, Configuration, Deployments, Kubernetes/EKS, CI/CD, Observability, Incident Response, RCA and DevOps Interview Preparation

---

# 1. Application Incident Fundamentals

An application incident occurs when a running application fails to meet expected behavior.

Examples:

    HTTP 5xx errors
    High latency
    Timeouts
    Application crashes
    Memory growth
    CPU saturation
    Database failures
    Queue backlog
    Dependency failures
    Incorrect responses
    Authentication failures
    Partial service degradation

In a production microservices platform:

    Client
      |
      v
    ALB / Ingress
      |
      v
    API Service
      |
      +---- User Service
      |
      +---- Product Service
      |
      +---- Cart Service
      |
      +---- Order Service
      |
      +---- Payment Service
      |
      +---- Inventory Service
      |
      +---- Notification Service
      |
      +---- Database / Cache / Queue

An incident in one dependency can create symptoms across many services.

---

# 2. Application Incident Mindset

Do not start with:

> "Restart the application."

Start with:

    What is the user impact?
    What changed?
    What is failing?
    Is the failure total or partial?
    Is it application code or infrastructure?
    Is a dependency failing?
    Is the application overloaded?
    Is the problem reproducible?

Use:

    Detect
      |
      v
    Scope
      |
      v
    Investigate
      |
      v
    Mitigate
      |
      v
    Validate
      |
      v
    RCA
      |
      v
    Prevent

---

# 3. Define Application Impact

Determine:

    All users?
    Some users?
    One API?
    One endpoint?
    One region?
    One availability zone?
    One application version?
    One tenant?
    One dependency?

Example:

    /orders = failing
    /products = healthy
    /users = healthy

This indicates a narrower problem than:

    Entire platform unavailable.

---

# 4. Application Incident Evidence

Collect:

- Request/error rate
- Latency
- Application logs
- Trace IDs
- Deployment version
- Pod status
- Restart count
- CPU
- Memory
- Thread count
- Connection pools
- Database health
- Cache health
- Queue depth
- External dependency status
- Recent configuration changes

Preserve evidence before restarting or deleting workloads.

---

# 5. Golden Signals

For application incidents, monitor:

    Latency
    Traffic
    Errors
    Saturation

Example:

    Traffic ↑
       |
       v
    CPU ↑
       |
       v
    Latency ↑
       |
       v
    Errors ↑

This can indicate capacity saturation.

---

# 6. RED Method

For request-driven applications:

    Rate
    Errors
    Duration

Example:

    Requests/sec
    HTTP 5xx rate
    p95 latency

RED is especially useful for APIs and microservices.

---

# 7. USE Method

For infrastructure/resources:

    Utilization
    Saturation
    Errors

Example:

    CPU utilization
    CPU run queue
    Disk errors

Combine RED and USE.

Application:

    RED

Infrastructure:

    USE

---

# 8. Application Troubleshooting Layers

Use this model:

    User
      |
      v
    DNS
      |
      v
    Load Balancer
      |
      v
    Application
      |
      v
    Runtime
      |
      v
    Dependency
      |
      v
    Database / Cache / Queue

Determine the first failing layer.

---

# 9. HTTP 4xx vs 5xx

## 4xx

Usually indicates a client/request/authentication problem.

Examples:

    400 Bad Request
    401 Unauthorized
    403 Forbidden
    404 Not Found
    409 Conflict
    429 Too Many Requests

## 5xx

Usually indicates server-side failure.

Examples:

    500 Internal Server Error
    502 Bad Gateway
    503 Service Unavailable
    504 Gateway Timeout

Do not assume every 4xx is harmless or every 5xx is an application bug.

---

# 10. HTTP 400 Bad Request

Possible causes:

- Invalid JSON
- Missing required field
- Incorrect parameter
- Invalid content type
- Request validation failure

Troubleshoot:

    Request
      |
      v
    API validation
      |
      v
    Error response

Check application logs and request payload structure.

---

# 11. HTTP 401 Unauthorized

Possible causes:

- Missing token
- Expired token
- Invalid credentials
- Wrong authentication configuration

Check:

- Authentication service
- Token validation
- Secret configuration
- Clock synchronization if token expiration is involved

Do not expose credentials in logs.

---

# 12. HTTP 403 Forbidden

Authentication may succeed but authorization fails.

Possible causes:

- Missing role
- RBAC issue
- Policy change
- Incorrect user permissions
- Service identity issue

Differentiate:

    401 = authentication problem

    403 = authorization problem

---

# 13. HTTP 404 Not Found

Possible causes:

- Wrong endpoint
- Route missing
- Ingress path mismatch
- API version mismatch
- Resource does not exist
- Incorrect Service routing

Check:

    Client
      |
      v
    ALB / Ingress
      |
      v
    Application route

A 404 does not always mean the application is down.

---

# 14. HTTP 409 Conflict

Common causes:

- Duplicate resource
- Optimistic locking
- Concurrent update
- State transition conflict

Example:

    Two requests
        |
        v
    Create same order
        |
        v
    Conflict

Check business logic and database constraints.

---

# 15. HTTP 429 Too Many Requests

Possible causes:

- Rate limiting
- API gateway limit
- Application protection
- Dependency throttling

Investigate:

    Request rate
    Rate-limit configuration
    Client retry behavior

A poorly designed client can make 429 incidents worse by retrying aggressively.

---

# 16. HTTP 500 Internal Server Error

Start with:

    Application logs
       |
       v
    Exception
       |
       v
    Stack trace
       |
       v
    Root cause

Possible causes:

- Null pointer
- Database failure
- Configuration error
- Dependency exception
- Code bug
- Serialization failure

Use trace IDs to connect the failed request to logs.

---

# 17. HTTP 502 Bad Gateway

Often means a proxy/load balancer could not successfully communicate with the upstream.

Possible causes:

- Backend connection failure
- Incorrect target port
- Application crashed
- Upstream reset
- Proxy configuration

Check:

    ALB / Ingress
       |
       v
    Service
       |
       v
    Pod
       |
       v
    Application

---

# 18. HTTP 503 Service Unavailable

Possible causes:

- No healthy backend
- No service endpoints
- Application unavailable
- Load balancer target unhealthy
- Maintenance
- Capacity issue

Check:

    Health
    Endpoints
    Pods
    Target health

---

# 19. HTTP 504 Gateway Timeout

A 504 usually means the upstream did not respond within the allowed timeout.

Possible causes:

- Slow database
- Slow external API
- Thread pool exhaustion
- Connection pool exhaustion
- Application overload
- Network delay

Trace the request path.

---

# 20. Latency Incident

Latency should be analyzed by percentile.

Example:

    p50 = 100ms
    p95 = 800ms
    p99 = 4s

If only p99 is high:

    A small subset of requests may be slow.

If p50, p95 and p99 all increase:

    Broader systemic degradation is likely.

---

# 21. Average Latency Can Mislead

Example:

    99 requests = 100ms
    1 request = 10 seconds

Average may hide the slow request.

Use:

    p50
    p90
    p95
    p99

for production latency analysis.

---

# 22. Latency Breakdown

Trace:

    API = 5s
       |
       +---- DB = 4.5s
       |
       +---- Cache = 50ms
       |
       +---- External API = 200ms

Root bottleneck:

    Database

Do not optimize the API framework when the database consumes most of the latency.

---

# 23. Database Slow Query Incident

Symptoms:

    API latency ↑
    DB CPU ↑
    Query duration ↑

Investigate:

- Slow query
- Missing index
- Lock contention
- Connection pool
- Data growth
- Query plan

A database incident can appear as an application latency incident.

---

# 24. Database Connection Pool Exhaustion

Example:

    Application
       |
       v
    Connection Pool
       |
       X
    No free connections
       |
       v
    Requests wait
       |
       v
    Latency ↑
       |
       v
    Timeout

Check:

    Active connections
    Idle connections
    Pool max
    Connection wait time

---

# 25. Connection Leak

If connections continuously increase:

    Connections ↑
       |
       v
    Never returned
       |
       v
    Pool exhausted

Possible causes:

- Missing close
- Transaction not released
- Exception path leak
- Application bug

A restart may temporarily fix it while hiding the root cause.

---

# 26. Database Connection Storm

If many pods restart simultaneously:

    20 pods
       |
       v
    20 connection pools
       |
       v
    Database connection spike

Scaling an application can unexpectedly overload the database.

Capacity planning must consider:

    Pods × connections/pod

---

# 27. Cache Incident

Example:

    Application
       |
       v
    Redis
       X
    Failure

Possible effects:

- Higher database load
- Higher latency
- Increased CPU
- Increased errors

Cache failures can create secondary database incidents.

---

# 28. Cache Stampede

If many requests miss the cache simultaneously:

    Cache miss
       |
       v
    1000 requests
       |
       v
    Database
       |
       v
    Database overload

Mitigation strategies may include:

- Request coalescing
- TTL strategy
- Cache warming
- Backoff
- Rate limiting

---

# 29. Queue Backlog Incident

Example:

    Producers
       |
       v
    RabbitMQ
       |
       v
    Consumers

If:

    Producer rate > Consumer rate

then:

    Queue depth ↑

Check:

- Consumer count
- Consumer errors
- Processing latency
- Message retry rate
- Dead-letter queue
- Dependency health

---

# 30. Consumer Lag

Consumer lag means messages are arriving faster than they are processed.

Possible causes:

- Consumer CPU
- Slow database
- External API
- Consumer crash
- Insufficient replicas
- Processing bug

Scale consumers only if the bottleneck is actually consumer capacity.

---

# 31. Poison Message

A message repeatedly fails processing.

Flow:

    Message
       |
       v
    Consumer
       |
       X
    Processing error
       |
       v
    Retry
       |
       X
    Same error
       |
       v
    Retry loop

Use:

    Dead-letter queue
    Retry limits
    Error classification

---

# 32. Retry Storm

Example:

    Database unavailable
       |
       v
    Application retries
       |
       v
    More DB requests
       |
       v
    DB overload
       |
       v
    More failures
       |
       v
    More retries

Retries should use:

    Exponential backoff
    Jitter
    Maximum attempts

---

# 33. Timeout Configuration

Timeouts exist at multiple layers:

    Client timeout
    ALB timeout
    Ingress timeout
    Application timeout
    Database timeout
    External API timeout

If configured incorrectly:

    Lower layer = 30s
    Upper layer = 5s

The upper layer may fail before the lower layer completes.

---

# 34. Timeout Chain

A useful design:

    Client
      |
    10s
      |
    API
      |
    8s
      |
    Service
      |
    5s
      |
    Dependency

Timeouts should be designed deliberately.

Avoid unlimited waits.

---

# 35. Thread Pool Exhaustion

Application:

    200 request threads

All waiting on:

    Slow database

New requests:

    Cannot obtain thread

Result:

    Queue ↑
    Latency ↑
    Timeout ↑

Check:

- Active threads
- Waiting threads
- Thread pool size
- Blocking calls

---

# 36. Java Thread Pool Incident

Typical symptoms:

- High latency
- Thread count high
- Request queue growing
- Database waits
- Timeouts

Take thread dumps during the incident where operationally appropriate.

Look for:

    BLOCKED
    WAITING
    TIMED_WAITING

---

# 37. Java Heap Incident

Symptoms:

    Heap ↑
    GC ↑
    Latency ↑
    OOM

Investigate:

- Heap usage
- GC pauses
- Allocation rate
- Heap dump where appropriate
- Recent code changes

Do not increase heap indefinitely.

---

# 38. Java Garbage Collection Incident

Example:

    Traffic ↑
       |
       v
    Allocation ↑
       |
       v
    GC frequency ↑
       |
       v
    CPU ↑
       |
       v
    Latency ↑

Check:

- GC pause duration
- GC frequency
- Heap occupancy
- Allocation rate

---

# 39. Java Deadlock

Symptoms:

- Requests hang
- CPU may be normal
- Thread pool appears occupied
- Application does not progress

Use thread dumps to identify threads waiting on locks.

Typical pattern:

    Thread A holds Lock 1
       |
       v
    waits for Lock 2

    Thread B holds Lock 2
       |
       v
    waits for Lock 1

Deadlock requires application-level investigation.

---

# 40. Node.js Event Loop Blocking

Node.js uses an event-driven model.

A CPU-heavy synchronous operation can block:

    Event Loop
       |
       X
    Requests wait
       |
       v
    Latency ↑

Symptoms:

- CPU high
- Event-loop delay high
- Requests slow

Look for synchronous/blocking operations.

---

# 41. Node.js Memory Incident

Possible causes:

- Memory leak
- Large objects
- Unbounded cache
- Large request bodies
- Excessive concurrency

Monitor:

    Heap
    RSS
    Event loop
    GC

Restarting may hide the leak temporarily.

---

# 42. Python Worker Exhaustion

A Python application may run multiple workers.

Example:

    Gunicorn
       |
       +---- Worker 1
       +---- Worker 2
       +---- Worker 3
       +---- Worker 4

If all workers are blocked:

    New requests wait.

Check:

- Worker count
- Worker state
- Request duration
- CPU
- Memory
- External dependencies

---

# 43. Python Memory Growth

Investigate:

- Object accumulation
- Cache growth
- Large datasets
- Worker recycling
- Application code changes

Compare memory over time rather than only checking one point.

---

# 44. CPU Saturation Incident

Symptoms:

    CPU ↑
    Latency ↑
    Throughput ↓

Possible causes:

- Traffic increase
- Infinite loop
- Expensive query
- Serialization
- Compression
- Encryption
- Excessive logging
- Garbage collection

Find what consumes CPU.

---

# 45. CPU Throttling in Kubernetes

Application:

    CPU request = 500m
    CPU limit = 500m

Actual demand:

    1 CPU

Container can be throttled.

Symptoms:

    CPU throttling
    Latency increase

Check both:

    Application CPU demand

and:

    Kubernetes CPU limits.

---

# 46. Memory Saturation Incident

Symptoms:

    Memory ↑
    OOMKilled
    GC ↑
    Evictions
    Swap/pressure where applicable

Determine:

    Container-level

or:

    Node-level

failure.

---

# 47. File Descriptor Exhaustion

Applications use file descriptors for:

- Sockets
- Files
- Pipes

If exhausted:

    Too many open files

Symptoms:

- Connection failures
- File errors
- Network failures

Check process file descriptor limits and open descriptor count.

---

# 48. Socket Exhaustion

Possible causes:

- Connection leak
- High traffic
- TIME_WAIT accumulation
- Short-lived connections

Investigate:

    Active connections
    TIME_WAIT
    Connection pool
    Keep-alive configuration

Do not change kernel parameters before identifying the application behavior.

---

# 49. DNS Failure in Application

Application:

    payment-service

DNS:

    payment-service.namespace.svc

If DNS fails:

    Requests timeout/fail.

Check:

- CoreDNS
- Search domains
- DNS configuration
- Service name
- Namespace
- Network path

---

# 50. DNS Latency Incident

DNS resolution itself may become slow.

Symptoms:

    Application latency ↑

while:

    Backend healthy

Investigate:

- DNS query rate
- CoreDNS resources
- Network latency
- External DNS
- Application DNS caching

---

# 51. External API Failure

Example:

    Application
       |
       v
    Payment Provider
       X
    Timeout

Possible impact:

    Thread pool occupied
    Connections occupied
    Retries increase
    Latency increases

External dependencies must have:

    Timeout
    Retry policy
    Circuit breaker where appropriate
    Fallback strategy where appropriate

---

# 52. Circuit Breaker

Circuit breaker states:

    CLOSED
       |
       v
    Failure threshold
       |
       v
    OPEN
       |
       v
    Recovery test
       |
       v
    HALF-OPEN
       |
       v
    CLOSED

Circuit breakers prevent continuous calls to a failing dependency.

---

# 53. Circuit Breaker Incident

If circuit opens unexpectedly:

Check:

- Failure threshold
- Timeout
- Dependency health
- Network errors
- Recent release

Do not simply increase the threshold without understanding the dependency failure.

---

# 54. Authentication Service Failure

Example:

    API
      |
      v
    Auth Service
      X
    Failure

Effects:

- 401/5xx
- Login failures
- Service-to-service failures

Check:

- Auth service health
- Token validation
- Certificates
- Secrets
- Network
- Clock synchronization

---

# 55. Certificate Expiration Incident

Symptoms:

    TLS handshake failure

Possible causes:

- Expired certificate
- Wrong certificate
- Trust chain problem
- Hostname mismatch

Check:

    Certificate expiry
    Issuer
    SAN
    Trust chain

Automate certificate monitoring where possible.

---

# 56. Clock Skew Incident

Authentication and TLS systems can depend on correct time.

If:

    Server clock differs significantly

possible symptoms:

- Token rejected
- TLS problems
- Signature validation failure

In cloud environments, verify time synchronization and node health.

---

# 57. Configuration Incident

Application configuration may come from:

    Environment variables
    ConfigMaps
    Secrets
    Helm values
    External configuration systems

A single wrong value can cause application failure.

Compare:

    Working version
    Failing version

---

# 58. Configuration Drift

Example:

    Staging:
    DATABASE_URL=correct

    Production:
    DATABASE_URL=old

The application code is identical but behavior differs.

Configuration must be versioned and controlled.

---

# 59. Environment Variable Incident

Check:

    kubectl describe pod <pod>

Avoid exposing secret values.

Verify:

    Variable exists
    Correct key
    Correct source
    Correct environment

---

# 60. Feature Flag Incident

A feature flag can activate new behavior without a new binary.

Example:

    new-payment-flow=true

Suddenly:

    Error rate ↑

Investigate:

- Flag change
- Rollout scope
- Affected users
- Dependency requirements

Feature flags are production changes and should be audited.

---

# 61. Bad Deployment Incident

Typical timeline:

    Deployment
       |
       v
    New pods
       |
       v
    Error rate ↑
       |
       v
    Latency ↑

Immediate action:

    Roll back if justified.

Then compare:

    Code
    Config
    Dependencies
    Runtime
    Resource requirements

---

# 62. Canary Deployment Incident

Canary:

    95% old
     5% new

If:

    Canary error rate ↑

stop rollout.

Do not allow a known-bad version to expand.

---

# 63. Blue-Green Deployment Incident

Traffic:

    Blue = current
    Green = new

If Green fails:

    Keep Blue serving traffic.

Then investigate Green independently.

This reduces production exposure.

---

# 64. Database Schema Migration Incident

Application version may expect:

    New column

Database migration may not have completed.

Result:

    Application errors

Safe migration strategy often uses backward-compatible changes:

    Add new field
       |
       v
    Deploy compatible application
       |
       v
    Start using field
       |
       v
    Remove old field later

---

# 65. Backward Compatibility

During rolling deployments:

    Old version
       +
    New version

may run simultaneously.

Database/API changes must support both where required.

Otherwise:

    New code works
    Old code fails

or:

    Old code works
    New code fails

---

# 66. API Versioning Incident

Example:

    Client expects /v1
    Service changed behavior

Potential causes:

- Breaking change
- Missing compatibility
- Incorrect routing
- Version mismatch

Use explicit API contracts and compatibility testing.

---

# 67. Serialization Failure

Possible causes:

- JSON schema mismatch
- Null field
- Type change
- Version incompatibility
- Unexpected payload

Symptoms:

    400
    500
    Consumer errors

Inspect the exact payload and schema without exposing sensitive data.

---

# 68. Microservice Contract Failure

Example:

    Orders expects:
    payment.status

Payment now returns:

    payment.state

Result:

    Orders fails.

Contract testing and backward-compatible API evolution reduce this risk.

---

# 69. Queue Contract Failure

Producer sends:

    version=2

Consumer only understands:

    version=1

Possible result:

    Message processing failures
    DLQ growth

Monitor message schema compatibility.

---

# 70. Application Logging Incident

If application behavior is failing but logs are missing:

Check:

    Application logger
       |
       v
    stdout/stderr
       |
       v
    Collector
       |
       v
    Log pipeline
       |
       v
    Elasticsearch/Kibana

Do not assume missing logs mean the application did not execute.

---

# 71. Logging Too Much

Excessive logs can cause:

    CPU ↑
    Disk/network ↑
    Elasticsearch load ↑
    Storage cost ↑

Debug logging should be controlled in production.

---

# 72. Logging Too Little

If only:

    "Request failed"

is logged, troubleshooting is difficult.

Useful structured fields include:

    timestamp
    level
    service
    environment
    request_id
    trace_id
    endpoint
    status
    duration

Avoid sensitive values.

---

# 73. Correlation ID

A request ID can follow a request across services.

Example:

    request_id=abc

This helps correlate:

    API log
    Orders log
    Payment log

Trace ID provides stronger distributed tracing correlation where tracing is implemented.

---

# 74. Trace-Based Application Troubleshooting

For a slow request:

    Trace
       |
       +---- API 5s
       |
       +---- DB 4.2s
       |
       +---- Cache 100ms

Then:

    Logs
       |
       v
    DB timeout / slow query

Then:

    Metrics
       |
       v
    DB CPU ↑

This is a strong observability-driven diagnosis.

---

# 75. Application Health Endpoints

Typical endpoints:

    /health
    /ready
    /live

Health should be designed intentionally.

Example:

    Liveness:
    Is process alive?

    Readiness:
    Can application serve traffic?

Do not make health checks excessively expensive.

---

# 76. Dependency Health Checks

A readiness check may verify critical dependencies, but be careful.

If every dependency is included:

    Database down
       |
       v
    All pods NotReady
       |
       v
    Entire service unavailable

Sometimes the application should remain available for partial functionality.

Health design must match business behavior.

---

# 77. Application Startup Failure

Startup can fail because:

- Config missing
- Secret missing
- Database unavailable
- Migration failed
- Port occupied
- Dependency unavailable
- Permission issue

Check:

    Startup logs
    Exit code
    Environment
    Mounted files
    Dependency connectivity

---

# 78. Graceful Shutdown

During deployment:

    SIGTERM
       |
       v
    Stop accepting new requests
       |
       v
    Complete active requests
       |
       v
    Close connections
       |
       v
    Exit

If graceful shutdown is not implemented:

- Requests may be dropped
- Connections may be reset
- Queue messages may be duplicated

---

# 79. Kubernetes Termination Grace Period

Application shutdown should fit within:

    terminationGracePeriodSeconds

If application needs 60 seconds but Kubernetes allows 30 seconds:

    Process may be killed before graceful shutdown completes.

Tune based on actual application behavior.

---

# 80. Connection Draining

During pod termination:

    Pod
       |
       v
    Readiness = false
       |
       v
    Stop receiving new traffic
       |
       v
    Existing requests finish
       |
       v
    Shutdown

This reduces dropped requests during rolling deployments.

---

# 81. Application Concurrency

Too much concurrency can cause:

    CPU saturation
    Memory pressure
    DB connection exhaustion
    External API throttling

More concurrency does not always mean more throughput.

Measure:

    Throughput
    Latency
    Errors
    Resource usage

---

# 82. Backpressure

Backpressure prevents producers from overwhelming consumers.

Example:

    API
       |
       v
    Queue
       |
       v
    Worker

If workers slow down:

    Queue grows

The system should have controlled behavior instead of unlimited memory growth.

---

# 83. Queue Backpressure Incident

Bad design:

    Unlimited in-memory queue

Result:

    Memory ↑
       |
       v
    OOM

Better:

    Bounded queue
       |
       v
    Backpressure
       |
       v
    Controlled degradation

---

# 84. Rate Limiting

Rate limiting protects services from overload.

Example:

    10,000 req/s
       |
       v
    Rate limiter
       |
       v
    2,000 req/s allowed

Excess traffic receives:

    429

Rate limits should be designed with client retry behavior.

---

# 85. Thundering Herd

Many clients retry simultaneously after a failure.

Example:

    Dependency recovers
       |
       v
    10,000 clients retry
       |
       v
    Dependency overloaded again

Use:

    Exponential backoff
    Jitter
    Rate limiting
    Caching

---

# 86. Dependency Failure Isolation

For critical dependencies:

    Timeout
    Retry limit
    Circuit breaker
    Bulkhead
    Fallback where appropriate

These patterns prevent one failure from consuming all application resources.

---

# 87. Bulkhead Pattern

Separate resources for different workloads.

Example:

    API A -> Thread Pool A
    API B -> Thread Pool B

If API A becomes slow:

    Pool A exhausted

but:

    Pool B remains available.

This limits blast radius.

---

# 88. Cascading Application Failure

Example:

    Payment API slow
       |
       v
    Orders threads blocked
       |
       v
    Orders latency ↑
       |
       v
    API retries ↑
       |
       v
    CPU ↑
       |
       v
    Pods scale
       |
       v
    Database connections ↑
       |
       v
    Database overloaded

The original dependency problem creates a system-wide incident.

---

# 89. Partial Failure

A microservice platform may continue serving some functions.

Example:

    Products = healthy
    Users = healthy
    Orders = degraded
    Payments = unavailable

Do not declare the entire platform down.

Identify degraded functionality and business impact.

---

# 90. Graceful Degradation

Example:

    Recommendation Service
       X
    Failure

Main product browsing may still work.

Instead of:

    Entire page = 500

application may:

    Show product page without recommendations.

Design for partial failure where business requirements allow it.

---

# 91. Application Capacity Incident

Capacity dimensions include:

    CPU
    Memory
    Threads
    Connections
    File descriptors
    Network
    Database
    Queue
    External API quota

A system can have low CPU while being fully saturated on database connections.

---

# 92. Saturation Is Not Just CPU

Example:

    CPU = 40%
    Memory = 50%

But:

    DB connections = 100%

Application is saturated.

Always monitor resource-specific bottlenecks.

---

# 93. File Descriptor Capacity

If:

    Open FDs -> limit

application may fail to:

- Open files
- Create sockets
- Accept connections

Symptoms may appear unrelated.

Check OS/process limits where access is available.

---

# 94. Thread Leak

If thread count grows continuously:

    Threads ↑
       |
       v
    Memory ↑
       |
       v
    Context switching ↑
       |
       v
    Performance ↓

Investigate thread creation and lifecycle.

---

# 95. Connection Leak

Same pattern:

    Connections ↑
       |
       v
    Pool exhausted
       |
       v
    Requests wait
       |
       v
    Timeouts

Monitor pool utilization and connection lifetime.

---

# 96. Memory Leak

Typical pattern:

    Memory
      |
      |       /
      |      /
      |     /
      |____/________ time

If memory does not return after workload decreases:

    Possible leak.

Compare:

    Traffic
    Heap
    RSS
    GC
    Restarts

---

# 97. Memory Leak Mitigation

Temporary:

    Restart affected instances

But permanent solution:

    Identify allocation source
    Fix code
    Test
    Deploy
    Monitor

A restart should not be documented as the root cause.

---

# 98. CPU Spike Investigation

When CPU suddenly increases:

Check:

    Traffic
    Request type
    Application version
    Thread activity
    GC
    Database/query behavior
    Logging
    Background jobs

Compare:

    Healthy pod
    Affected pod

---

# 99. Error Rate Spike Investigation

Example:

    5xx = 0.2%
       |
       v
    Suddenly 15%

Correlate with:

    Deployment
    Traffic
    Dependency
    Resource usage
    Logs
    Traces

The timestamp relationship is critical.

---

# 100. Latency Spike Investigation

Start:

    p95/p99 ↑

Then:

    Which endpoint?
       |
       v
    Which service?
       |
       v
    Which dependency?
       |
       v
    Which operation?

Tracing is particularly useful here.

---

# 101. Traffic Spike Investigation

If request rate increases:

Check:

- Legitimate traffic
- Marketing event
- Client bug
- Retry storm
- Bot traffic
- Attack
- Internal loop

Traffic increase is not automatically a reason to scale blindly.

---

# 102. Retry Loop Detection

Example:

    Service A -> B
       |
       v
    B fails
       |
       v
    A retries
       |
       v
    B fails
       |
       v
    A retries

If multiple services retry:

    A -> B -> C

the amplification can become severe.

---

# 103. Dependency Timeout Budget

Suppose API timeout:

    5 seconds

and it calls:

    Service B = 4 seconds
    Service C = 4 seconds

Sequential calls cannot both consume the full budget.

Design dependency timeouts based on the total request budget.

---

# 104. Sequential vs Parallel Calls

Sequential:

    API
      |
      v
    B 2s
      |
      v
    C 2s
      |
      v
    Total ~4s

Parallel:

    API
      |
      +---- B 2s
      |
      +---- C 2s

Total can approach:

    ~2s

But parallelism increases resource demand and dependency load.

---

# 105. Database Lock Contention

Symptoms:

- Queries waiting
- Latency increases
- CPU may be normal
- Connections occupied

Investigate:

- Blocking transactions
- Lock duration
- Long-running transactions
- Query patterns

Database lock issues often appear as application timeout incidents.

---

# 106. Deadlock at Database Layer

Example:

    Transaction A locks Row 1
       |
       v
    waits for Row 2

    Transaction B locks Row 2
       |
       v
    waits for Row 1

Database may detect and abort one transaction.

Application should handle the resulting error appropriately.

---

# 107. Transaction Too Long

Long transactions can:

- Hold locks
- Increase connection usage
- Delay other requests
- Increase database pressure

Measure transaction duration.

Keep transaction scope as small as practical.

---

# 108. N+1 Query Problem

Example:

    1 query for orders
       +
    100 queries for customers

Total:

    101 queries

This can create severe latency and database load.

Trace/database instrumentation can reveal repeated query patterns.

---

# 109. Connection Pool Sizing

Too small:

    Requests wait.

Too large:

    Database overwhelmed.

Pool size must consider:

    Application replicas
    Database connection limit
    Query duration
    Traffic
    Workload type

Example:

    20 pods × 50 connections
    = potential 1000 DB connections

---

# 110. Cache TTL Incident

TTL too short:

    Cache miss rate ↑
    Database load ↑

TTL too long:

    Stale data

Tune based on:

    Data freshness
    Read/write pattern
    Database capacity

---

# 111. Cache Eviction Incident

If cache memory fills:

    Entries evicted
       |
       v
    Cache hit rate ↓
       |
       v
    Database load ↑

Monitor:

    Hit rate
    Memory
    Evictions
    Latency

---

# 112. Queue Consumer Failure

If consumers crash:

    Queue depth ↑

Check:

- Consumer logs
- Restart count
- Dependency failures
- Resource usage
- Message format

Do not simply increase consumers if every consumer fails on the same poison message.

---

# 113. Dead-Letter Queue

A DLQ isolates messages that cannot be processed.

Flow:

    Main Queue
       |
       v
    Consumer
       |
       X
    Failure
       |
       v
    Retry limit
       |
       v
    DLQ

Monitor DLQ size and alert on unexpected growth.

---

# 114. Application Deployment Validation

After deployment verify:

    Pods
    Readiness
    Error rate
    Latency
    Logs
    Metrics
    Traces
    Dependency connectivity
    Business transactions

A deployment is not successful merely because:

    kubectl rollout status = success.

---

# 115. Smoke Testing

After deployment test:

    Health
    Login
    Core API
    Database operation
    Critical business workflow

For a microservices platform:

    Create user
       |
       v
    Browse product
       |
       v
    Add cart
       |
       v
    Create order
       |
       v
    Payment
       |
       v
    Inventory

Validate the critical path.

---

# 116. Synthetic Monitoring

A synthetic transaction periodically executes a known workflow.

Example:

    Login
      |
      v
    Product
      |
      v
    Cart
      |
      v
    Checkout

It detects issues before users report them.

---

# 117. Canary Monitoring

During canary rollout compare:

    Old version
    vs
    New version

Metrics:

    Error rate
    Latency
    CPU
    Memory
    Business success rate

If new version is worse:

    Stop rollout.

---

# 118. Business Metrics During Application Incidents

Technical health may be green while business functionality fails.

Example:

    HTTP 200
    but payment transaction fails.

Monitor business KPIs where possible:

    Successful orders
    Payment success
    Login success
    Checkout completion

Observability should include business outcomes.

---

# 119. Application Incident Severity

Example:

## Sev-1

Critical business function unavailable.

## Sev-2

Major degradation affecting significant users.

## Sev-3

Limited impact with workaround.

## Sev-4

Minor issue.

Severity definitions should match organizational incident policy.

---

# 120. Incident Mitigation Options

Depending on evidence:

    Rollback
    Scale
    Disable feature flag
    Fail over
    Reduce traffic
    Stop retry storm
    Restart unhealthy workload
    Restore dependency
    Apply configuration correction

Choose the safest reversible action first.

---

# 121. Rollback Decision

Rollback is appropriate when:

    Strong correlation with recent release
    Known previous-good version
    Rollback is safe
    User impact is significant

Before rollback consider:

    Database migration compatibility
    State changes
    Queue schema
    External side effects

---

# 122. Feature Flag Mitigation

If a new feature causes failures:

    Disable feature
       |
       v
    Existing path restored

This can be safer than a full deployment rollback.

Feature flags should have emergency controls and ownership.

---

# 123. Scaling Mitigation

If capacity is clearly the bottleneck:

    Increase replicas
    Increase node capacity
    Adjust autoscaling

But first determine:

    Is dependency capacity sufficient?

Scaling an application against a saturated database can worsen the incident.

---

# 124. Traffic Shedding

When overloaded:

    Incoming traffic
       |
       v
    Rate limit
       |
       v
    Protect core functionality

Prefer:

    Graceful degradation

over:

    Total outage.

---

# 125. Dependency Failover

If primary dependency fails:

    Primary
       X
       |
       v
    Secondary

Failover must be tested before production incidents.

Do not introduce an untested failover during a critical incident without understanding the risk.

---

# 126. Incident Validation

After mitigation verify:

    Error rate ↓
    Latency ↓
    Traffic stable
    Pods stable
    Dependency healthy
    Queue draining
    Database connections normal
    Logs normal
    Traces normal
    Business transactions succeeding

Recovery must be measurable.

---

# 127. Application RCA

Example:

    Incident:
    Orders API latency increased.

    Root cause:
    Database query became slow after data growth.

    Contributing factor:
    Missing index.

    Impact:
    p99 latency increased to 8 seconds.

    Mitigation:
    Reduced traffic and applied approved database optimization.

    Prevention:
    Query performance testing and database monitoring.

---

# 128. Five Whys — High Latency

Why was API latency high?

    Database query slow.

Why?

    Full table scan.

Why?

    Required index missing.

Why?

    Data volume increased after production growth.

Why?

    Query performance was not tested against production-scale data.

Prevention:

    Production-scale performance testing.

---

# 129. Five Whys — 5xx Spike

Why did 5xx increase?

    Application exception.

Why?

    New code expected a configuration field.

Why?

    Production ConfigMap lacked the field.

Why?

    Configuration was not included in deployment validation.

Why?

    CI pipeline validated the image but not runtime configuration.

Prevention:

    Runtime configuration validation.

---

# 130. Five Whys — Database Connection Exhaustion

Why were requests timing out?

    No free DB connections.

Why?

    Connection pool exhausted.

Why?

    Connections were not released.

Why?

    Error path skipped cleanup.

Why?

    Resource lifecycle was not tested under failure conditions.

Prevention:

    Connection leak testing + application instrumentation.

---

# 131. Five Whys — Queue Backlog

Why did queue depth increase?

    Consumers processed slowly.

Why?

    Database operations slowed.

Why?

    Database CPU reached saturation.

Why?

    New application release increased query volume.

Why?

    N+1 query pattern introduced.

Prevention:

    Query analysis + load testing + code review.

---

# 132. Application Incident Runbook

A good runbook contains:

    Symptoms
    Impact
    Metrics
    Logs
    Traces
    Initial commands
    Dependency checks
    Mitigation
    Validation
    Escalation
    Rollback
    RCA

It should explain:

> What to check next based on what you find.

---

# 133. Application Incident Commands — Kubernetes

Useful commands:

    kubectl get pods -n <namespace>

    kubectl describe pod <pod> -n <namespace>

    kubectl logs <pod> -n <namespace>

    kubectl logs <pod> --previous -n <namespace>

    kubectl get svc -n <namespace>

    kubectl get endpoints -n <namespace>

    kubectl get events -n <namespace> --sort-by=.lastTimestamp

    kubectl top pods -n <namespace>

    kubectl top nodes

Use commands to collect evidence, not simply to execute random fixes.

---

# 134. Application Incident Commands — Linux

Useful commands where node/process access is available:

    ps -ef

    top

    free -m

    df -h

    du -sh

    ss -lntp

    ss -s

    uptime

    vmstat

    iostat

    journalctl

Use the command that answers a specific question.

---

# 135. Application Incident — CPU

Flow:

    CPU high?
       |
       v
    Which process?
       |
       v
    Which thread?
       |
       v
    What operation?
       |
       v
    Recent change?
       |
       v
    Mitigate / fix

Do not stop at:

    CPU = 100%.

---

# 136. Application Incident — Memory

Flow:

    Memory high?
       |
       v
    Container or node?
       |
       v
    Heap/RSS?
       |
       v
    Leak or workload increase?
       |
       v
    OOM?
       |
       v
    Mitigate / fix

---

# 137. Application Incident — Network

Flow:

    Request fails?
       |
       v
    DNS?
       |
       v
    TCP?
       |
       v
    TLS?
       |
       v
    HTTP?
       |
       v
    Application?
       |
       v
    Dependency?

Separate network layers.

---

# 138. Application Incident — Database

Flow:

    API slow?
       |
       v
    DB calls slow?
       |
       v
    Connections?
       |
       v
    Locks?
       |
       v
    CPU?
       |
       v
    Query plan?
       |
       v
    Storage?

Avoid assuming "database is slow" without evidence.

---

# 139. Application Incident — Queue

Flow:

    Queue depth ↑
       |
       v
    Consumer healthy?
       |
       v
    Processing time?
       |
       v
    Dependency slow?
       |
       v
    Consumer capacity?
       |
       v
    Poison message?
       |
       v
    DLQ?

---

# 140. Observability-Driven Application Troubleshooting

Use:

    Metrics
       |
       v
    Detect abnormality

    Traces
       |
       v
    Locate bottleneck

    Logs
       |
       v
    Explain failure

    Kubernetes
       |
       v
    Validate platform state

    Dependency metrics
       |
       v
    Confirm root cause

This prevents tunnel vision.

---

# 141. Incident Correlation Example

Scenario:

    p95 latency ↑
    5xx ↑
    DB CPU ↑

Trace:

    API
      |
      v
    DB query = 4s

Logs:

    Query timeout

Kubernetes:

    Pods healthy

Conclusion:

    Application infrastructure is healthy;
    database/query layer is the bottleneck.

---

# 142. Incident Correlation — Memory

Scenario:

    Memory ↑
    GC ↑
    Latency ↑
    Restarts ↑

Logs:

    OutOfMemoryError

Kubernetes:

    OOMKilled

Likely conclusion:

    Application memory pressure.

Investigate code/runtime configuration rather than only increasing replicas.

---

# 143. Incident Correlation — Queue

Scenario:

    Queue depth ↑
    Consumer CPU = 20%
    DB latency = 5s

Conclusion:

    Consumer is waiting on database.

Scaling consumers may increase DB pressure.

Fix the database bottleneck first.

---

# 144. Incident Correlation — External API

Scenario:

    API latency ↑
    Threads ↑
    External API latency ↑
    CPU normal

Conclusion:

    Threads are blocked waiting for dependency.

Potential mitigation:

    Timeout
    Circuit breaker
    Reduce traffic
    Dependency failover

---

# 145. Production Scenario — API Suddenly Returns 500

Investigation:

    Error rate ↑
       |
       v
    Deployment 2 min ago
       |
       v
    Logs show missing configuration
       |
       v
    ConfigMap changed
       |
       v
    New application expects new key

Mitigation:

    Roll back or restore compatible configuration.

Prevention:

    Configuration validation in CI/CD.

---

# 146. Production Scenario — API Returns 504

Investigation:

    504
       |
       v
    Trace = 8s
       |
       v
    DB span = 7s
       |
       v
    DB CPU = 95%
       |
       v
    Slow query detected

Root cause:

    Database saturation/query performance.

Do not increase ALB timeout as the first fix.

---

# 147. Production Scenario — Application Restarts Every Few Hours

Investigation:

    Restart pattern periodic
       |
       v
    Memory increases continuously
       |
       v
    OOMKilled
       |
       v
    Traffic-independent memory growth

Likely:

    Memory leak.

Restarting provides temporary mitigation only.

---

# 148. Production Scenario — Application Works After Scaling

Investigation:

    2 pods = slow
    6 pods = healthy

But:

    DB CPU increases

Possible interpretation:

    Application was capacity constrained,
    but database may become the next bottleneck.

Scaling must be validated across the complete dependency chain.

---

# 149. Production Scenario — Only One Endpoint Is Slow

Example:

    /users = 100ms
    /products = 120ms
    /orders = 5s

Trace orders:

    DB = 4.5s

Focus on order dependency path.

Do not increase cluster capacity for the entire platform without evidence.

---

# 150. Production Scenario — Only New Pods Fail

Old pods:

    healthy

New pods:

    errors

Compare:

    Image
    Environment
    ConfigMap
    Secrets
    ServiceAccount
    Resource limits
    Dependencies

This strongly suggests a deployment/configuration/runtime difference.

---

# 151. Production Scenario — All Pods Fail After Secret Rotation

Investigation:

    Secret changed
       |
       v
    Authentication errors
       |
       v
    Application cannot connect

Check:

- Secret key
- Credential validity
- Application reload behavior
- External service configuration

Restore known-good credentials if approved, then correct rotation.

---

# 152. Production Scenario — Login Works but Checkout Fails

This is a partial application failure.

Check:

    Login -> healthy
    Products -> healthy
    Cart -> healthy
    Checkout -> failing

Focus on:

    Payment
    Inventory
    Order
    Database

Do not restart unrelated services.

---

# 153. Production Scenario — Error Rate Increases Only During Traffic Spikes

Possible causes:

- Capacity limit
- Thread pool
- Connection pool
- CPU
- Database
- Rate limiting
- External API quota

Compare:

    Traffic
    Capacity
    Errors

Look for the saturation threshold.

---

# 154. Production Scenario — No Errors but Users Report Slowness

HTTP status may remain:

    200

while:

    p99 latency = 10s

This is why monitoring only errors is insufficient.

Check:

    Latency
    Traces
    Dependency performance
    Queueing
    Resource saturation

---

# 155. Production Scenario — Application Logs Show Nothing

Possible causes:

- Logging level
- Logger failure
- Application crash before logging
- stdout/stderr pipeline
- Collector failure
- Log routing problem

Check whether:

    Application logs absent

or:

    Logs generated but not collected.

---

# 156. Production Scenario — Trace Exists but Application Looks Healthy

A trace may reveal:

    API = 3s
    External API = 2.8s

Application CPU:

    30%

This shows:

> Low CPU does not mean low latency.

The application may be waiting on external I/O.

---

# 157. Production Scenario — CPU High but Traffic Normal

Possible causes:

- Infinite loop
- Memory/GC behavior
- Background job
- Logging
- New inefficient code
- Retry loop
- Query serialization

Compare application version and process/thread activity.

---

# 158. Production Scenario — Memory High After Deployment

Timeline:

    Deployment
       |
       v
    Memory growth
       |
       v
    OOMKilled

Strong correlation exists.

Compare:

    Old image
    New image

If rollback restores normal memory behavior, investigate the release.

---

# 159. Production Scenario — Database Healthy but API Slow

Check:

    DB health = normal

Then:

    Connection pool
    Thread pool
    Network
    Serialization
    Cache
    External API

A healthy database does not prove the application's database interaction is healthy.

---

# 160. Production Scenario — Queue Backlog After Deployment

Compare:

    Consumer version
    Processing time
    Error rate
    DB usage
    Message schema

If processing time increased after deployment:

    New consumer version likely changed throughput.

---

# 161. Application Incident Response Checklist

## Detection

    [ ] Error rate
    [ ] Latency
    [ ] Traffic
    [ ] Saturation

## Scope

    [ ] Endpoint
    [ ] Service
    [ ] Version
    [ ] Region/AZ
    [ ] User segment

## Investigation

    [ ] Logs
    [ ] Traces
    [ ] Metrics
    [ ] Pods
    [ ] Dependencies
    [ ] Database
    [ ] Cache
    [ ] Queue

## Mitigation

    [ ] Rollback
    [ ] Scale
    [ ] Feature flag
    [ ] Traffic control
    [ ] Dependency failover

## Recovery

    [ ] Error rate normal
    [ ] Latency normal
    [ ] Business transaction works
    [ ] Dependencies healthy

---

# 162. Application Incident RCA Checklist

    [ ] Incident summary
    [ ] User impact
    [ ] Start/end time
    [ ] Detection method
    [ ] Timeline
    [ ] Root cause
    [ ] Contributing factors
    [ ] Mitigation
    [ ] Recovery
    [ ] Corrective action
    [ ] Preventive action
    [ ] Monitoring improvement
    [ ] Runbook update

---

# 163. Interview Question — How Do You Troubleshoot High API Latency?

Strong answer:

> I first identify which endpoints and percentiles are affected, then correlate latency with traffic, errors and resource saturation. I use distributed tracing to break the latency into downstream calls and inspect application logs for the failing or slow operation. I then check database, cache, queue and external API performance before deciding on mitigation.

---

# 164. Interview Question — How Do You Troubleshoot HTTP 500?

Strong answer:

> I check the application logs using the request or trace ID, identify the exception and determine whether it originated in application code or a dependency. I correlate the error with recent deployments, configuration changes and dependency health. I use rollback or another safe mitigation if the incident is release-related and then perform RCA.

---

# 165. Interview Question — How Do You Troubleshoot 504?

Strong answer:

> I determine which timeout layer returned the 504, then trace the request through the application and dependencies. I check database latency, external API latency, thread and connection pools and application resource saturation. I avoid simply increasing the timeout because that can hide the underlying bottleneck.

---

# 166. Interview Question — How Do You Troubleshoot Memory Leaks?

Strong answer:

> I compare memory growth with traffic and workload over time, determine whether the issue is heap or overall process memory, and correlate it with garbage collection and restarts. I compare application versions and use runtime profiling or heap analysis where appropriate. Restarting can be a temporary mitigation but does not solve the leak.

---

# 167. Interview Question — How Do You Troubleshoot CPU Spikes?

Strong answer:

> I first establish whether traffic increased. If not, I identify the process or workload consuming CPU and investigate expensive operations, background jobs, garbage collection, logging, retries or recent code changes. I compare healthy and affected instances and use application profiling when necessary.

---

# 168. Interview Question — How Do You Troubleshoot Database Connection Pool Exhaustion?

Strong answer:

> I check active, idle and waiting connections, pool limits and database-side connection counts. I determine whether traffic increased, queries became slower or connections are leaking. I also calculate total potential connections across all replicas because scaling application pods can multiply database connection demand.

---

# 169. Interview Question — How Do You Troubleshoot Queue Backlog?

Strong answer:

> I check producer rate versus consumer processing rate, consumer health, processing latency, dependency latency and error/retry rates. I determine whether additional consumers would actually help or simply overload a downstream dependency. I also inspect the DLQ for poison messages.

---

# 170. Interview Question — How Do You Prevent Cascading Failures?

Strong answer:

> I use timeouts, bounded retries with exponential backoff and jitter, circuit breakers, bulkheads, rate limiting, backpressure and graceful degradation. I also monitor dependencies and define clear capacity limits so that one failing service does not consume all application resources.

---

# 171. Interview Question — Why Can Scaling Make an Incident Worse?

Strong answer:

> Scaling increases application capacity but also increases pressure on dependencies. For example, if every pod opens 50 database connections, doubling pods can double database connections. If the database is already saturated, scaling the application can make the incident worse.

---

# 172. Interview Question — How Do You Troubleshoot an Incident After Deployment?

Strong answer:

> I compare the incident timeline with the deployment, check the new image and runtime configuration, compare old and new pod behavior, and inspect logs, metrics and traces. If there is strong evidence that the release caused the issue and rollback is safe, I roll back to restore service, then reproduce and fix the underlying problem.

---

# 173. Interview Question — How Do You Troubleshoot a Microservices Incident?

Strong answer:

> I start with the user-facing symptom and trace the request through the service dependency chain. I use metrics to identify abnormal services, traces to locate latency or failure boundaries, and logs to explain the error. I then verify Kubernetes health and external dependencies before applying the smallest safe mitigation.

---

# 174. Interview Question — What Is the Difference Between a Symptom and Root Cause?

Example:

    Symptom:
    CrashLoopBackOff

    Root cause:
    Database authentication failure

Another:

    Symptom:
    504

    Root cause:
    Slow database query

Another:

    Symptom:
    High CPU

    Root cause:
    Retry storm

A strong DevOps engineer always asks:

> What caused the observed symptom?

---

# 175. Senior-Level Application Troubleshooting Framework

Use:

    USER IMPACT
         |
         v
    SERVICE SCOPE
         |
         v
    TIME CORRELATION
         |
         v
    METRICS
         |
         v
    TRACES
         |
         v
    LOGS
         |
         v
    DEPENDENCIES
         |
         v
    RESOURCE SATURATION
         |
         v
    RECENT CHANGES
         |
         v
    MITIGATION
         |
         v
    VALIDATION
         |
         v
    RCA
         |
         v
    PREVENTION

---

# 176. Application Incident Decision Tree

                    INCIDENT
                       |
                       v
                 USER IMPACT?
                       |
                       v
                 ERROR OR LATENCY?
                  /           \
              ERROR          LATENCY
                |               |
                v               v
             LOGS            TRACES
                |               |
                v               v
           EXCEPTION?       BOTTLENECK?
                |               |
                v               v
           DEPENDENCY?       DB/API/CPU
                |               |
                +-------+-------+
                        |
                        v
                  RECENT CHANGE?
                        |
                        v
                    MITIGATE
                        |
                        v
                    VALIDATE
                        |
                        v
                       RCA

---

# 177. Application Troubleshooting Matrix

| Symptom | First Check | Common Root Cause |
|---|---|---|
| 400 | Request/logs | Invalid input |
| 401 | Auth logs | Token/credential |
| 403 | Authorization | Permission |
| 404 | Route/Ingress | Wrong path |
| 409 | Application/DB | Conflict |
| 429 | Rate limiter | Traffic/throttling |
| 500 | Application logs | Code/dependency |
| 502 | Upstream health | Backend connection |
| 503 | Endpoints/health | No healthy backend |
| 504 | Trace/latency | Slow dependency |
| High CPU | Process/profile | Expensive operation |
| High memory | Heap/RSS | Leak/workload |
| OOMKilled | Container state | Memory limit |
| DB timeout | Pool/query | DB saturation |
| Queue backlog | Consumers | Processing bottleneck |
| Cache miss spike | Cache metrics | TTL/eviction |
| TLS failure | Certificate | Expiry/trust |
| Connection refused | Port/listener | App/service |
| Timeout | Network/dependency | Connectivity/slow backend |

---

# 178. Production Application Troubleshooting Checklist

## Request Layer

    [ ] DNS
    [ ] Load balancer
    [ ] HTTP status
    [ ] Request rate
    [ ] Latency

## Application Layer

    [ ] Logs
    [ ] Trace
    [ ] Runtime
    [ ] Threads/workers
    [ ] CPU
    [ ] Memory
    [ ] File descriptors

## Dependency Layer

    [ ] Database
    [ ] Cache
    [ ] Queue
    [ ] External APIs
    [ ] Authentication

## Kubernetes Layer

    [ ] Pod
    [ ] Readiness
    [ ] Restarts
    [ ] Resources
    [ ] Service
    [ ] Endpoints
    [ ] Ingress

## Change Layer

    [ ] Deployment
    [ ] ConfigMap
    [ ] Secret
    [ ] Feature flag
    [ ] Database migration
    [ ] Infrastructure change

---

# 179. Production Application Reliability Principles

Remember:

> Low CPU does not mean the application is healthy.

> A 200 response does not guarantee successful business behavior.

> A pod being Running does not mean it is serving traffic.

> A database being healthy does not mean application queries are healthy.

> Scaling the application can overload the database.

> Retries without limits can turn a dependency failure into a platform-wide outage.

> Timeouts must be designed across the complete request chain.

> Restarting a leaking application only provides temporary relief.

> Queue backlog must be analyzed from producer rate versus consumer processing rate.

> Health checks should represent the intended failure behavior.

> Application observability must combine metrics, logs and traces.

> Recent deployments and configuration changes should always be correlated with incident timelines.

> Restore service safely first, then identify and remove the root cause.

The production application incident mindset is:

    DETECT
       +
    SCOPE
       +
    CORRELATE
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

This is the approach expected from a production DevOps / DevSecOps engineer troubleshooting Java, Node.js, Python and microservices workloads running on Kubernetes/EKS.
