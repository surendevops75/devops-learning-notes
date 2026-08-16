# Production Incident Scenarios

> Monitoring, Observability & Logging — Production Incident Interview Preparation
>
> Focus: realistic production incidents, incident command, detection, triage, evidence, mitigation, recovery, validation, root-cause analysis, postmortems, SLO impact and senior DevOps/DevSecOps interview answers.

---

# 1. Purpose of This File

This file is different from generic troubleshooting.

Troubleshooting asks:

> "How would you investigate this technical problem?"

Production incident scenarios ask:

> "A real production system is failing right now. How do you lead the response, protect users, investigate, communicate, mitigate, recover and prevent recurrence?"

A strong incident response therefore covers:

    DETECT
       |
       v
    CONFIRM IMPACT
       |
       v
    DECLARE INCIDENT
       |
       v
    ASSIGN ROLES
       |
       v
    TRIAGE
       |
       v
    MITIGATE
       |
       v
    RECOVER
       |
       v
    VALIDATE
       |
       v
    ROOT CAUSE
       |
       v
    POSTMORTEM
       |
       v
    PREVENT RECURRENCE

---

# 2. Production Incident Response Principles

During an active incident:

1. Protect customers first.
2. Stop further damage.
3. Establish a shared timeline.
4. Prefer reversible mitigations.
5. Preserve evidence.
6. Communicate clearly.
7. Avoid random production changes.
8. Validate recovery using multiple signals.
9. Separate mitigation from root cause.
10. Convert lessons into engineering actions.

A senior engineer should be calm, systematic and evidence-driven.

---

# 3. Incident Roles

For a significant incident, separate responsibilities.

## Incident Commander

Owns:

- Incident coordination
- Priorities
- Decisions
- Escalation
- Communication

## Technical Lead

Owns:

- Technical investigation
- Hypotheses
- Mitigation
- Validation

## Communications Lead

Owns:

- Stakeholder updates
- Status messages
- Customer/business communication where appropriate

## Scribe

Records:

- Timeline
- Actions
- Findings
- Decisions

One engineer may perform multiple roles during smaller incidents.

---

# 4. Incident Lifecycle

    Detection
       |
       v
    Triage
       |
       v
    Classification
       |
       v
    Mitigation
       |
       v
    Recovery
       |
       v
    Validation
       |
       v
    Closure
       |
       v
    Postmortem

Do not treat "the alert stopped firing" as automatic proof that the incident is resolved.

---

# 5. Incident Severity

Example:

## SEV-1

- Major customer outage
- Large-scale payment failure
- Data integrity issue
- Critical security event

## SEV-2

- Significant degradation
- Important functionality unavailable
- Large customer segment affected

## SEV-3

- Limited impact
- Workaround available
- No major business disruption

Severity definitions should be organization-specific.

---

# 6. Incident Scenario: Complete API Outage

## Symptoms

    HTTP 5xx = 100%
    Success rate = 0%
    Users cannot access application

## Immediate response

1. Confirm the alert.
2. Check external availability.
3. Determine scope.
4. Check ALB.
5. Check Kubernetes.
6. Check recent deployment.
7. Check major dependencies.

Commands:

```bash
kubectl get pods -A
kubectl get svc -A
kubectl get ingress -A
kubectl get events --sort-by=.lastTimestamp
```

## Possible causes

- Bad deployment
- ALB failure
- Service selector problem
- All pods unavailable
- Database outage
- DNS issue
- Network failure

## Mitigation

If deployment correlation is strong:

```bash
kubectl rollout undo deployment/<deployment> -n <namespace>
```

Then validate:

- External HTTP status
- Error rate
- Latency
- Healthy targets
- Business transaction

---

# 7. Incident Scenario: 50% API Errors

Unlike a 100% outage, partial failures require segmentation.

Check:

    version
    region
    AZ
    pod
    node
    endpoint

Example:

```promql
sum(rate(http_requests_total{status=~"5.."}[5m]))
by (service, version, region)
```

If one version is responsible:

> Stop rollout or rollback.

If one region is responsible:

> Investigate regional infrastructure/dependency health.

---

# 8. Incident Scenario: Latency Suddenly Doubles

## Symptoms

    p50: 100ms -> 110ms
    p95: 400ms -> 900ms
    p99: 800ms -> 3s

Investigate:

1. Traffic
2. CPU
3. Memory
4. Database
5. External dependencies
6. Queue
7. Deployment
8. Network

Do not rely on average latency.

---

# 9. Incident Scenario: Checkout Failure

Architecture:

    User
      |
      v
    ALB
      |
      v
    Checkout Service
      |
      +--> Cart
      +--> Inventory
      +--> Payment
      +--> Database
      +--> Queue

Business symptom:

    Checkout success rate ↓

Investigate the complete transaction path.

A healthy checkout API process does not prove successful checkout.

---

# 10. Incident Scenario: Payment Provider Is Down

Symptoms:

    Payment errors ↑
    Checkout failures ↑

Immediate action:

- Confirm provider issue.
- Check whether fallback exists.
- Control retries.
- Prevent retry storm.
- Communicate business impact.

Potential mitigation:

    Circuit breaker
    Fallback
    Queue
    Controlled retry

Do not aggressively retry an unavailable provider.

---

# 11. Incident Scenario: Database Is Down

Pattern:

    Database unavailable
        |
        v
    Application errors
        |
        v
    Queue backlog
        |
        v
    User impact

Immediate priorities:

1. Confirm DB failure.
2. Check failover.
3. Prevent application overload.
4. Control retries.
5. Validate connections after recovery.

Do not restart every application pod unless there is a reason.

---

# 12. Incident Scenario: Database Connection Exhaustion

Symptoms:

    DB healthy
    Connections = maximum
    Application latency ↑

Investigate:

- Connection leaks
- Long-running queries
- Pool configuration
- Traffic
- Slow dependencies

Mitigation may include:

- Reduce traffic
- Roll back bad release
- Terminate clearly abnormal connections according to DB procedures
- Correct pool configuration

---

# 13. Incident Scenario: Database CPU Is 100%

Check:

- Query volume
- Slow queries
- Connection count
- Recent application deployment
- Missing indexes
- Batch jobs

Compare:

    Application request rate
    Database query rate

A database CPU spike may be caused by an application query regression.

---

# 14. Incident Scenario: RabbitMQ Queue Backlog

Symptoms:

    Queue depth ↑
    Queue age ↑
    Processing delay ↑

Investigate:

- Consumer count
- Consumer errors
- Worker CPU/memory
- DB latency
- External dependencies

Mitigation:

- Increase consumers if downstream capacity allows.
- Fix failed workers.
- Reduce producer pressure.
- Protect downstream systems.

Do not scale consumers blindly.

---

# 15. Incident Scenario: Retry Storm

Pattern:

    Dependency failure
       |
       v
    Retry
       |
       v
    More traffic
       |
       v
    Dependency overload
       |
       v
    More failures

Mitigation:

- Exponential backoff
- Jitter
- Maximum retry count
- Circuit breaker
- Request timeout

This is a common cascading-failure pattern.

---

# 16. Incident Scenario: Kubernetes Cluster Has Many Pending Pods

Symptoms:

    Pending pods ↑

Check:

```bash
kubectl get pods -A
kubectl get nodes
kubectl describe pod <pod> -n <namespace>
```

Investigate:

- CPU requests
- Memory requests
- Taints
- Tolerations
- Affinity
- Node capacity
- Quotas

If cluster capacity is genuinely insufficient, scale nodes.

---

# 17. Incident Scenario: Entire Node Becomes NotReady

Impact:

    Pods on node affected

Check:

```bash
kubectl describe node <node>
kubectl get pods -A -o wide
```

Investigate:

- Kubelet
- Runtime
- Disk
- Memory
- Network
- Node health

Then verify workloads have rescheduled successfully.

---

# 18. Incident Scenario: Multiple Nodes Become NotReady

This is higher severity.

Check whether nodes share:

- AZ
- Node group
- Instance type
- Network path
- Launch configuration

If multiple nodes fail simultaneously, investigate shared infrastructure.

---

# 19. Incident Scenario: Disk Full on Kubernetes Nodes

Symptoms:

    DiskPressure=True
    Pods evicted

Check:

```bash
df -h
df -i
du -xh /var | sort -h | tail
```

Look for:

- Container logs
- Images
- Temporary files
- Ephemeral storage

Mitigate according to approved cleanup procedures.

Then fix the source of growth.

---

# 20. Incident Scenario: Node Memory Pressure

Symptoms:

    MemoryPressure=True
    Evictions

Investigate:

- Top pods
- Requests/limits
- Memory leaks
- Node sizing
- System processes

Do not simply increase node size without checking workload behavior.

---

# 21. Incident Scenario: OOMKilled Production Pods

Check:

```bash
kubectl describe pod <pod>
kubectl logs <pod> --previous
kubectl top pod <pod>
```

Determine:

    Limit reached?
    Memory leak?
    Traffic increase?
    Large payload?
    Cache growth?

Mitigation:

- Roll back if release-related.
- Scale replicas if load-related.
- Adjust limits carefully.
- Fix application memory behavior.

---

# 22. Incident Scenario: CrashLoopBackOff Across Many Pods

If many pods crash at the same time, suspect shared causes:

- ConfigMap
- Secret
- Dependency
- Image
- Application release
- Network
- Common environment variable

Check:

```bash
kubectl get pods -A
kubectl get events --sort-by=.lastTimestamp
```

Shared failure is more likely than dozens of independent bugs.

---

# 23. Incident Scenario: Bad Deployment

Timeline:

    10:00 deployment starts
    10:02 errors increase
    10:03 latency increases
    10:04 alert fires

Compare:

    Old version
    New version

If new version is clearly unhealthy:

1. Pause rollout.
2. Roll back safely.
3. Validate.
4. Preserve evidence.
5. Investigate later.

---

# 24. Incident Scenario: Rollback Makes Things Worse

Possible causes:

- Database schema already changed
- Configuration changed independently
- External dependency changed
- Rollback image incompatible with current state

Do not repeatedly roll forward/backward without understanding dependencies.

---

# 25. Incident Scenario: Configuration Change Breaks Production

Symptoms:

    No code deployment
    Application suddenly fails

Investigate:

- ConfigMap
- Secret
- Environment variables
- Runtime configuration
- External configuration system

" No deployment occurred" does not mean no change occurred.

---

# 26. Incident Scenario: Secret Rotation Breaks Service

Timeline:

    Secret rotated
       |
       v
    Authentication failures
       |
       v
    API errors

Check:

- New credential
- Secret propagation
- Pod restart
- Application reload behavior
- External provider

Mitigate by restoring known-good credentials if safe.

Then fix rotation automation.

---

# 27. Incident Scenario: Certificate Expiry

Symptoms:

    TLS handshake failures
    HTTPS unavailable

Immediate:

- Confirm expiry.
- Renew/replace certificate.
- Validate chain.
- Test externally.

Prevention:

    Certificate expiry monitoring
    Automated renewal
    Renewal failure alerts

---

# 28. Incident Scenario: DNS Failure

Symptoms:

    Users cannot resolve application

Check:

- DNS records
- TTL
- Route configuration
- Resolver
- Regional behavior
- Recent DNS change

Use external probes to distinguish DNS failure from application failure.

---

# 29. Incident Scenario: ALB Targets Become Unhealthy

Trace:

    ALB
      |
      v
    Target group
      |
      v
    Service
      |
      v
    Endpoints
      |
      v
    Pods

Check:

- Health-check path
- Port
- Security groups
- Readiness
- Application listener

---

# 30. Incident Scenario: ALB Returns 502/503

Separate:

    Load-balancer-generated errors
    Application-generated errors

Investigate:

- Target health
- Connection failures
- Timeouts
- Service endpoints
- Application logs

---

# 31. Incident Scenario: DNS Works but Application Is Unreachable

If:

    DNS = healthy

Continue:

    TLS
      |
      v
    ALB
      |
      v
    Target
      |
      v
    Service
      |
      v
    Pod

Avoid stopping investigation at DNS.

---

# 32. Incident Scenario: CoreDNS Failure

Impact can be broad because many workloads depend on DNS.

Check:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system <coredns-pod>
kubectl top pods -n kube-system
```

Investigate:

- Pod availability
- CPU
- Memory
- Configuration
- Network

---

# 33. Incident Scenario: NetworkPolicy Accidentally Blocks Production

Symptoms:

    Pods healthy
    Connections timeout

Check:

- Recent NetworkPolicy changes
- Source labels
- Destination labels
- Ports

Rollback the policy change if clearly responsible and safe.

---

# 34. Incident Scenario: One AZ Has High Error Rate

Compare:

    AZ-A
    AZ-B
    AZ-C

If only one AZ is affected:

- Check node health.
- Check targets.
- Check network.
- Check infrastructure.
- Shift traffic if architecture supports it.

---

# 35. Incident Scenario: One Region Is Failing

Architecture:

    Global
      |
      +--> Region A
      +--> Region B
      +--> Region C

If Region A fails:

- Confirm regional scope.
- Compare regional health.
- Protect healthy regions.
- Shift traffic if designed for it.
- Validate capacity in surviving regions.

Do not shift traffic to a region without checking its capacity.

---

# 36. Incident Scenario: Regional Failover Causes Second Outage

Pattern:

    Region A fails
       |
       v
    Traffic -> Region B
       |
       v
    Region B overloaded
       |
       v
    Region B fails

This is a classic DR capacity problem.

DR architecture must include capacity planning for failover load.

---

# 37. Incident Scenario: Global Traffic Suddenly Drops

Check:

- DNS
- Routing
- ALB
- CDN where applicable
- Client behavior
- Upstream services

A traffic drop may be an outage even if application metrics show no errors.

---

# 38. Incident Scenario: Traffic Spikes 10x

Determine:

- Legitimate demand
- Marketing event
- Client bug
- Bot traffic
- Attack

Then evaluate:

- HPA
- Node scaling
- Database
- Cache
- Queue
- External dependencies

---

# 39. Incident Scenario: HPA Does Not Scale During Traffic Spike

Check:

```bash
kubectl get hpa
kubectl describe hpa <hpa>
kubectl top pods
```

Investigate:

- Metrics
- CPU requests
- Target
- Max replicas
- Metrics API
- Pod startup time

---

# 40. Incident Scenario: HPA Scales to Maximum but Errors Continue

This suggests scaling application replicas is not enough.

Check:

- Database
- External API
- Queue
- Network
- Node capacity
- Application bottleneck

Scaling callers can overload the dependency.

---

# 41. Incident Scenario: Prometheus Goes Down During an Application Outage

This is a monitoring resilience problem.

Questions:

- Are alerts still available?
- Is Alertmanager healthy?
- Is there external monitoring?
- Can logs detect the issue?
- Is there a second Prometheus?

Mitigation:

    Restore monitoring
    Maintain external detection
    Investigate HA gap

---

# 42. Incident Scenario: Prometheus Storage Fills

Symptoms:

    Prometheus unstable
    Queries fail
    Disk full

Check:

- Active series
- Ingestion rate
- Retention
- WAL
- Disk

Potential causes:

- Cardinality explosion
- New service
- New labels
- Scrape frequency

---

# 43. Incident Scenario: Cardinality Explosion

Timeline:

    Deployment
       |
       v
    New label
       |
       v
    Millions of series
       |
       v
    Prometheus memory ↑
       |
       v
    Monitoring degradation

Mitigate:

- Remove offending metric/label.
- Reduce scrape load.
- Protect Prometheus.
- Restore stable telemetry.

Prevention:

- Metric review
- Cardinality limits
- CI checks

---

# 44. Incident Scenario: Prometheus Query Overloads Monitoring

Symptoms:

    CPU ↑
    Memory ↑
    Queries slow

Find expensive dashboard/query.

Mitigation:

- Stop expensive query if possible.
- Reduce dashboard refresh.
- Use recording rules.
- Narrow query.

---

# 45. Incident Scenario: Grafana Is Unavailable During Major Incident

Use alternative paths:

- Prometheus
- Alertmanager
- Kibana
- CLI
- External monitoring

Then restore Grafana.

A dashboard outage should not become an application outage.

---

# 46. Incident Scenario: Grafana Shows Stale Data

Check:

- Datasource
- Query
- Refresh interval
- Time range
- Prometheus health
- Browser/cache behavior

Compare with raw datasource.

---

# 47. Incident Scenario: Alertmanager Sends No Notifications

Trace:

    Alert rule
       |
       v
    Alert firing
       |
       v
    Alertmanager
       |
       v
    Route
       |
       v
    Receiver
       |
       v
    Notification

Find the first failure.

---

# 48. Incident Scenario: Alert Storm During Database Outage

Primary failure:

    Database unavailable

Secondary:

    100 services fail

Use:

- Grouping
- Inhibition
- Dependency-aware alerting

The goal is to make the primary failure obvious.

---

# 49. Incident Scenario: Critical Alert Is Silenced

Check:

- Silence scope
- Owner
- Expiration
- Reason

If an inappropriate silence caused missed detection:

> Treat alert governance as part of the postmortem.

---

# 50. Incident Scenario: SLO Burn Rate Suddenly Spikes

Investigate:

- Error rate
- Traffic
- SLI calculation
- Recent deployment
- Dependency

High burn rate means reliability budget is being consumed quickly.

---

# 51. Incident Scenario: Error Budget Exhausted

This is not just a dashboard number.

Possible actions:

- Pause risky releases
- Prioritize reliability work
- Investigate recurring incidents
- Reduce operational risk

Use error budget as a decision mechanism.

---

# 52. Incident Scenario: ELK Stops Receiving Logs

Trace:

    Application
      |
      v
    Collector
      |
      v
    Logstash
      |
      v
    Elasticsearch
      |
      v
    Kibana

Find first broken stage.

---

# 53. Incident Scenario: Logstash Queue Grows Rapidly

Compare:

    Input rate
    Processing rate
    Output rate

If Elasticsearch is slow:

    Queue grows

If filters are expensive:

    Processing slows

Scale only after identifying the bottleneck.

---

# 54. Incident Scenario: Elasticsearch Cluster Turns Red

Treat as serious.

Investigate:

- Primary shard failures
- Node loss
- Disk
- Storage
- Corruption
- Allocation

Prioritize data availability and recovery.

---

# 55. Incident Scenario: Elasticsearch Disk Full

Immediate:

- Assess indexing impact.
- Check retention.
- Check large indices.
- Follow approved cleanup policy.

Long-term:

- Lifecycle management
- Capacity planning
- Storage tiers
- Better ingestion controls

---

# 56. Incident Scenario: Kibana Is Down

Application may still be healthy.

Use:

- Elasticsearch APIs
- Alternative dashboards
- Application logs
- Prometheus

Restore Kibana and investigate the failure.

---

# 57. Incident Scenario: Centralized Logging Is Down

Important question:

> Does application availability depend on centralized logging?

It should generally not.

Use asynchronous collection and buffering so application traffic can continue during temporary logging backend failures.

---

# 58. Incident Scenario: Logging Volume Increases 20x

Possible causes:

- DEBUG logging
- Exception loop
- Retry storm
- Traffic spike
- Duplicate logging

Immediate:

- Protect backend.
- Identify producer.
- Reduce unnecessary volume.
- Preserve important logs.

---

# 59. Incident Scenario: Observability Costs Double

Check:

- Metrics series
- Log ingestion
- Storage
- Retention
- Query volume

Find the change that correlates with cost increase.

---

# 60. Incident Scenario: One Service Produces Most Logs

Investigate:

- Error loop
- DEBUG level
- Request payload logging
- Duplicate logs

Fix producer rather than only scaling Elasticsearch.

---

# 61. Incident Scenario: Logs Contain Secrets

Immediate:

1. Stop further exposure.
2. Rotate affected secrets.
3. Restrict access.
4. Assess retention.
5. Fix logging.
6. Follow security response procedures.

Deleting an index may not remove backups or replicas.

---

# 62. Incident Scenario: Authentication System Fails

Impact:

    Login failures
    Token failures
    User-facing errors

Check:

- Identity provider
- DNS
- Certificates
- Token validation
- Application configuration
- External dependencies

This is a user-journey incident.

---

# 63. Incident Scenario: Cache Failure Causes Database Outage

Pattern:

    Cache unavailable
       |
       v
    Cache hit rate ↓
       |
       v
    DB traffic ↑
       |
       v
    DB saturation
       |
       v
    API errors

Mitigation:

- Restore cache
- Reduce load
- Protect DB
- Rate-limit if required

---

# 64. Incident Scenario: Queue Failure Causes Synchronous API Failures

If the API depends directly on the queue:

    Queue unavailable
       |
       v
    API request failure

Question:

> Is the dependency truly required synchronously?

Architectural improvement may be:

    API -> Durable acceptance
          |
          v
        Queue
          |
          v
        Worker

depending on business requirements.

---

# 65. Incident Scenario: External API Rate Limit

Symptoms:

    429 ↑

Check:

- Request rate
- Retry rate
- Client concurrency
- Provider limits

Mitigation:

- Backoff
- Queue
- Reduce concurrency
- Cache
- Request aggregation

---

# 66. Incident Scenario: External API Latency Causes Thread Exhaustion

Pattern:

    External API slow
       |
       v
    Requests wait
       |
       v
    Threads occupied
       |
       v
    New requests queue
       |
       v
    Timeouts

Mitigate:

- Timeout
- Circuit breaker
- Concurrency limit
- Fallback

---

# 67. Incident Scenario: Deployment Causes Database Load Spike

Timeline:

    New release
       |
       v
    Query count ↑
       |
       v
    DB CPU ↑
       |
       v
    API latency ↑

Likely application regression.

Rollback may be appropriate if safe.

---

# 68. Incident Scenario: New Feature Flag Causes Outage

Feature flags are production changes.

Check:

- Flag state
- Enablement percentage
- Affected version
- Region
- User segment

Mitigation:

    Disable flag

Then investigate.

---

# 69. Incident Scenario: Batch Job Overloads Production Database

Pattern:

    Batch starts
       |
       v
    DB CPU ↑
       |
       v
    API latency ↑

Mitigate:

- Stop/throttle batch
- Reschedule
- Rate-limit queries

Prevention:

- Capacity-aware scheduling
- Query optimization
- Resource isolation

---

# 70. Incident Scenario: CronJob Runs Twice

Possible causes:

- Duplicate schedules
- Controller behavior
- Job retries
- Misconfiguration

Check idempotency.

Production jobs should be designed so duplicate execution does not corrupt data.

---

# 71. Incident Scenario: Data Processing Is Delayed

Metrics:

    Queue age ↑
    Processing latency ↑
    Data freshness ↓

The user-facing SLI may be:

> Percentage of data processed within X minutes.

---

# 72. Incident Scenario: Monitoring Says Job Succeeded But Data Is Missing

Process success:

    exit code = 0

Business success:

    expected data exists

These are different signals.

Add business validation.

---

# 73. Incident Scenario: Node Failure Causes Application Outage

If one node failure causes outage:

Investigate:

- Replica count
- Pod distribution
- Anti-affinity
- Topology spread
- PDB
- Capacity

A highly available workload should tolerate expected node failure.

---

# 74. Incident Scenario: AZ Failure Causes Application Outage

Investigate:

- Replica distribution
- Node groups
- Load balancer
- Database topology
- Dependencies

HA must be designed across the actual failure domain.

---

# 75. Incident Scenario: PodDisruptionBudget Prevents Maintenance

A PDB may protect availability but also prevent voluntary disruptions.

Check:

- Desired replicas
- Available replicas
- PDB minAvailable/maxUnavailable

Balance availability with operational flexibility.

---

# 76. Incident Scenario: Cluster Upgrade Causes Unexpected Evictions

Investigate:

- PDB
- Node drain
- Pod topology
- Replica capacity
- Stateful workloads

Validate disruption behavior before upgrades.

---

# 77. Incident Scenario: Deployment Removes Too Many Replicas

Check:

- Rolling update strategy
- maxUnavailable
- maxSurge
- Readiness
- PDB

A deployment can technically progress while temporarily reducing capacity too much.

---

# 78. Incident Scenario: Readiness Probe Causes Traffic Drop

Symptoms:

    Pods Running
    Endpoints ↓

Investigate:

- Probe path
- Probe timeout
- Application startup
- Dependency behavior

A readiness failure may remove healthy-capable pods from traffic.

---

# 79. Incident Scenario: Liveness Probe Causes Outage

Pattern:

    Load ↑
       |
       v
    Response slower
       |
       v
    Liveness timeout
       |
       v
    Restart
       |
       v
    Capacity ↓
       |
       v
    More load

This can create a restart storm.

---

# 80. Incident Scenario: Image Registry Outage

Symptoms:

    New pods cannot start

Existing pods may remain healthy.

Mitigation:

- Avoid unnecessary restarts.
- Use cached images where possible.
- Restore registry access.
- Ensure image pull resilience.

---

# 81. Incident Scenario: ECR Access Fails

Check:

- IAM role
- Repository policy
- Network path
- ECR endpoint
- Image URI
- Token/authentication

Do not assume image is missing until registry access is verified.

---

# 82. Incident Scenario: Kubernetes Secret Mount Fails

Check:

```bash
kubectl describe pod <pod>
kubectl get secret <secret>
```

Investigate:

- Secret exists
- Namespace
- Key names
- Mount configuration
- Permissions

---

# 83. Incident Scenario: Service Discovery Breaks After Label Change

If Service selector no longer matches pods:

    Service
       |
       v
    No endpoints

Check:

```bash
kubectl get endpoints <service>
kubectl describe service <service>
kubectl get pods --show-labels
```

Fix selector/labels.

---

# 84. Incident Scenario: Service Has No Endpoints

Possible:

- Pods not ready
- Selector mismatch
- Namespace mismatch
- Deployment failure

This is a high-value Kubernetes troubleshooting scenario.

---

# 85. Incident Scenario: Network Latency Increases Between Services

Check:

- Node
- AZ
- Region
- Network
- Service mesh if applicable
- DNS
- Connection pools

Compare healthy and unhealthy paths.

---

# 86. Incident Scenario: DNS Latency Causes API Latency

Pattern:

    DNS latency ↑
       |
       v
    Dependency connection delay
       |
       v
    API latency ↑

CoreDNS is a dependency even when application code has not changed.

---

# 87. Incident Scenario: File Descriptor Exhaustion

Symptoms:

    Too many open files

Investigate:

- Socket count
- File handles
- Connection leaks
- Process limits

Fix leak first.

---

# 88. Incident Scenario: CPU Throttling Causes Latency

Check:

- CPU requests
- CPU limits
- Throttling metrics
- Traffic

Potential mitigation:

- Adjust resources
- Scale replicas
- Fix workload inefficiency

---

# 89. Incident Scenario: Memory Leak After Release

Timeline:

    Deploy
       |
       v
    Memory gradually ↑
       |
       v
    OOMKilled
       |
       v
    Restart
       |
       v
    Repeat

Pattern strongly suggests memory growth.

Rollback may be the fastest mitigation.

---

# 90. Incident Scenario: Noisy Neighbor

One workload consumes excessive:

- CPU
- Memory
- Disk
- Network

Impact appears in unrelated workloads.

Use:

- Requests/limits
- Quotas
- Priority
- Isolation
- Capacity planning

---

# 91. Incident Scenario: Observability Agent Becomes Noisy Neighbor

Monitoring agents themselves can consume significant resources.

Check:

- DaemonSet usage
- Scrape volume
- Log volume
- Processing load

Observability must have resource budgets.

---

# 92. Incident Scenario: Monitoring Data Is Lost During Collector Failure

Ask:

- Is local buffering available?
- Is data durable?
- How long can backend be unavailable?
- What telemetry is critical?

Define acceptable telemetry loss.

---

# 93. Incident Scenario: Monitoring Backend Fails During Application Failure

Use independent monitoring.

Example:

    Application
       |
       v
    Local monitoring
       |
       v
    Central monitoring

plus:

    External blackbox
       |
       v
    Critical alert

This avoids a shared single point of failure.

---

# 94. Incident Scenario: Alerting System Is Down

Potential impact:

    Application failure
       |
       X
    No page

Mitigate through:

- HA Alertmanager
- External monitoring
- Secondary notification path

Then restore primary alerting.

---

# 95. Incident Scenario: On-Call Receives Wrong Team Alerts

Check alert labels:

    team
    service
    environment
    severity

Fix routing.

Then add automated route tests.

---

# 96. Incident Scenario: Alert Storm During Region Failure

Use:

    region
    service
    dependency

Group alerts around primary regional incident.

Avoid hundreds of independent pages.

---

# 97. Incident Scenario: Customers Report Errors Before Monitoring

This indicates detection gap.

Improve:

- Synthetic monitoring
- Business SLI
- User-journey monitoring
- SLO burn-rate alerts

---

# 98. Incident Scenario: Monitoring Shows Healthy but Customer Journey Fails

Use blackbox:

    Login
      |
      v
    Search
      |
      v
    Checkout

Internal component health can remain green while the complete journey fails.

---

# 99. Incident Scenario: Incident Appears Resolved but Queue Is Still Growing

Application may accept requests while asynchronous processing remains unhealthy.

Continue monitoring:

- Queue depth
- Queue age
- Consumer rate
- Data freshness

Do not close until recovery criteria are met.

---

# 100. Incident Scenario: Incident Appears Resolved but Error Budget Keeps Burning

Historical SLO windows retain previous failures.

Check current burn rate separately from total window compliance.

---

# 101. Incident Scenario: Incident Recurs Every Day

Look for periodic patterns:

- Cron jobs
- Batch processing
- Traffic patterns
- Certificate processes
- Cache expiry
- Scheduled deployments

Plot incidents against time of day.

---

# 102. Incident Scenario: Incident Happens Every Monday

Investigate scheduled workloads:

- Reports
- Backups
- Batch jobs
- Traffic
- Maintenance

Recurring incidents often indicate predictable resource contention.

---

# 103. Incident Scenario: Incident Happens After Every Deployment

Investigate:

- Deployment process
- Probes
- Resource requests
- Database migrations
- Configuration
- Canary strategy

Do not accept deployment incidents as normal.

---

# 104. Incident Scenario: Incident Happens Only During Scale-Out

Possible:

- Startup configuration
- Image pull
- Readiness
- Dependency initialization
- Database connection surge

Scaling creates new load patterns.

---

# 105. Incident Scenario: New Pods Cannot Start During Outage

Check:

- Registry
- IAM
- Scheduling
- Node capacity
- Secrets
- ConfigMaps

Protect existing healthy pods from unnecessary restart.

---

# 106. Incident Scenario: Deployment During Dependency Outage

If dependency is already failing:

    New deployment
       |
       v
    More failures

Decision:

> Avoid introducing unrelated change during an active incident unless it is part of mitigation.

---

# 107. Incident Scenario: Security Incident Appears as Application Incident

Example:

    Traffic spike
    CPU ↑
    Error rate ↑

Could be:

- Attack
- Credential abuse
- Bot traffic

Coordinate with security response when indicators suggest malicious activity.

---

# 108. Incident Scenario: Logs Are Deleted During Investigation

Avoid destructive troubleshooting.

Preserve:

- Logs
- Metrics
- Events
- Deployment history
- Configuration changes

Incident evidence is valuable.

---

# 109. Incident Scenario: Engineer Wants to Restart Everything

Senior response:

> Before restarting, I would collect enough evidence to understand the failure. If restart is a safe mitigation, I would perform it in a controlled way while preserving logs and metrics.

Restart can remove evidence.

---

# 110. Incident Scenario: Engineer Wants to Increase Memory Immediately

Better response:

> I would first determine whether memory usage is caused by traffic, a leak, payload size, cache growth or an incorrect limit. Increasing memory may mitigate the symptom but can hide the root cause.

---

# 111. Incident Scenario: Engineer Wants to Delete Old Elasticsearch Indices

Better response:

> I would verify the retention policy and approved deletion process first. During an incident, deleting data blindly can create compliance or investigation problems.

---

# 112. Incident Scenario: Engineer Wants to Silence All Alerts

Better response:

> I would group and inhibit secondary alerts while preserving the primary incident signal. A blanket silence could hide important failures.

---

# 113. Incident Scenario: Incident Commander Asks for Root Cause Immediately

Correct response:

> I can provide the current leading hypothesis and supporting evidence, but I would distinguish that from confirmed root cause until the evidence is sufficient.

This is important senior behavior.

---

# 114. Incident Scenario: Multiple Teams Blame Each Other

Use evidence:

    Timeline
    Metrics
    Logs
    Changes
    Dependencies

Avoid:

> Team A caused it.

Prefer:

> The failure began after version X was deployed and affected only requests routed to that version.

Blameless investigation is more productive.

---

# 115. Incident Scenario: Stakeholders Ask "When Will It Be Fixed?"

Do not invent a time.

Communicate:

- Current impact
- Current mitigation
- What is being investigated
- Next update time
- Known uncertainty

Example:

> We have identified elevated checkout failures and paused the rollout. The team is validating rollback. We will provide the next update after validation.

---

# 116. Incident Scenario: Customer Impact Is Unknown

State uncertainty explicitly.

Investigate:

- Traffic
- Error rate
- Region
- User segment
- Business transactions

Avoid claiming zero impact simply because infrastructure is green.

---

# 117. Incident Scenario: Incident Has Multiple Simultaneous Symptoms

Create a hypothesis tree.

Example:

    API errors
       |
       +--> Database?
       |
       +--> Deployment?
       |
       +--> Network?
       |
       +--> External API?
       |
       +--> DNS?

Prioritize by:

    Impact
    Evidence
    Likelihood
    Reversibility

---

# 118. Incident Scenario: No Obvious Root Cause

Use elimination.

Check:

    Recent changes
    Dependencies
    Capacity
    Infrastructure
    Networking
    Application
    Data
    Security

Document ruled-out hypotheses.

Negative evidence is useful.

---

# 119. Incident Scenario: Two Problems Occur Simultaneously

Example:

    Deployment
    + Database slowdown

Do not force a single-cause explanation.

Determine whether:

- One caused the other
- They are independent
- They share a common cause

---

# 120. Incident Scenario: Monitoring Data Is Contradictory

Example:

    Grafana says healthy
    Logs show errors
    Users report failures

Validate:

- Time range
- Datasource
- Query
- Metric definition
- Log pipeline
- External checks

Trust evidence after validating the signal source.

---

# 121. Incident Scenario: Metrics Are Delayed

Determine:

    Event time
    Collection time
    Ingestion time
    Query time

A dashboard may look healthy simply because data is delayed.

---

# 122. Incident Scenario: Incident Is Limited to One Endpoint

Compare:

    /login
    /search
    /orders
    /payment

If only one endpoint fails:

- Endpoint-specific code
- Dependency
- Query
- Feature flag
- Route

This narrows the investigation.

---

# 123. Incident Scenario: Incident Is Limited to One Customer Segment

Check dimensions:

- Region
- Account tier
- Feature flag
- Client version
- Authentication method

Business segmentation can reveal hidden scope.

---

# 124. Incident Scenario: Mobile Clients Fail but Web Works

Investigate:

- API version
- TLS
- Authentication
- Headers
- CDN
- Client-specific endpoint

Do not assume backend is universally broken.

---

# 125. Incident Scenario: Only New Client Version Fails

Strong hypothesis:

    Client regression

Check:

- API compatibility
- Authentication
- Headers
- TLS
- Feature flags

Rollback client release if possible.

---

# 126. Incident Scenario: API Version Compatibility Breaks

Example:

    Client v1
       |
       X
    API v2

Prevention:

- Backward compatibility
- Contract tests
- Versioning
- Canary rollout

---

# 127. Incident Scenario: Database Migration Is Backward Incompatible

Danger:

    New schema
       |
       X
    Old application

Before rollback:

- Determine schema compatibility.
- Protect data.
- Follow migration rollback procedure.

Application rollback is not always sufficient.

---

# 128. Incident Scenario: Queue Messages Are Being Reprocessed

Symptoms:

    Duplicate processing
    Queue depth abnormal

Investigate:

- Ack behavior
- Consumer crashes
- Visibility timeout where applicable
- Idempotency

For critical workflows, consumers should be designed for safe retries.

---

# 129. Incident Scenario: Duplicate Orders Are Created

Possible:

- Retry
- Duplicate message
- Race condition
- Idempotency failure

Immediate concern:

> Data integrity.

Mitigation should prevent further duplicates before investigating historical records.

---

# 130. Incident Scenario: Data Corruption Suspected

Highest priorities:

1. Stop further corruption.
2. Preserve evidence.
3. Identify affected scope.
4. Protect backups.
5. Follow recovery procedure.
6. Involve appropriate data/security owners.

Do not run destructive cleanup commands casually.

---

# 131. Incident Scenario: Backup Failed Before Data Incident

Determine:

- Last successful backup
- Replication state
- Recovery options
- Data loss exposure

This directly affects RPO.

---

# 132. Incident Scenario: Restore Is Successful But Application Cannot Start

Possible:

- Schema version mismatch
- Configuration
- Secrets
- Network
- Dependencies

DR recovery must test the complete application path, not only storage restoration.

---

# 133. Incident Scenario: DR Region Has Insufficient Capacity

If failover causes overload:

    DR capacity planning failed

Improve:

- Capacity reservation
- Autoscaling
- Load testing
- Failover exercises

---

# 134. Incident Scenario: Monitoring Is Not Available in DR

This is a DR observability gap.

Ensure:

- Dashboards
- Alerts
- Collection
- Access
- Configuration

are available in the recovery environment.

---

# 135. Incident Scenario: Region Failover Works but Alerts Do Not

Check:

- Alert rules
- Region labels
- Routing
- External monitoring
- Notification dependencies

A service recovery without alerting recovery is incomplete.

---

# 136. Incident Scenario: Postmortem Reveals Missing Alert

Add:

- Specific SLI
- Alert rule
- Dashboard
- Runbook
- Test

Avoid generic action:

> Improve monitoring.

---

# 137. Incident Scenario: Postmortem Reveals Human Error

Do not stop at:

> Engineer made a mistake.

Ask:

- Why was the action allowed?
- Was there review?
- Was automation missing?
- Was the UI dangerous?
- Was rollback easy?
- Was documentation clear?

Improve the system.

---

# 138. Incident Scenario: Manual Production Changes Are Common

Risks:

- Drift
- No audit
- Inconsistent environments
- Difficult rollback

Improve:

    Git
      |
      v
    Review
      |
      v
    CI/CD
      |
      v
    Production

---

# 139. Incident Scenario: Configuration Drift Causes Incident

Compare:

    Desired state
    Actual state

GitOps helps detect drift.

For Kubernetes:

    Git
      |
      v
    ArgoCD
      |
      v
    Cluster

---

# 140. Incident Scenario: ArgoCD Shows Drift During Incident

Do not blindly sync.

First determine:

- Is drift intentional?
- Was emergency mitigation applied manually?
- Will sync undo mitigation?

During incidents, safety comes before reconciliation.

After recovery:

> Restore the desired state properly through Git.

---

# 141. Incident Scenario: Emergency Manual Fix Is Applied

Document:

- What changed
- Why
- Who
- When
- Expected effect

Then create a permanent code/configuration change.

Emergency changes should not become permanent undocumented drift.

---

# 142. Incident Scenario: Alerting Works in Staging but Not Production

Compare:

- Datasource
- Labels
- Secrets
- Routing
- Network
- Alert rules
- Notification configuration

Environment parity matters.

---

# 143. Incident Scenario: Dashboard Works in Staging but Not Production

Check:

- Datasource
- Metric labels
- Service names
- Namespace
- Environment variable
- Query

Do not assume dashboard portability without standardized metadata.

---

# 144. Incident Scenario: Production Metrics Are Missing After Deployment

Potential:

- Instrumentation removed
- Exporter disabled
- ServiceMonitor mismatch
- Label change
- Endpoint changed

Monitoring is part of release validation.

---

# 145. Incident Scenario: Deployment Removes Important Logs

Check:

- Log level
- Logger configuration
- stdout/stderr
- Collector filters
- Logstash filters

Avoid filtering out important errors.

---

# 146. Incident Scenario: Logging Pipeline Drops Events

Measure:

    Input
    Processed
    Output
    Dropped

If logs are dropped, determine:

- Where
- Why
- How much
- Which services

---

# 147. Incident Scenario: Monitoring Platform Has No Capacity Headroom

During application traffic spike:

    Application telemetry ↑
       |
       v
    Monitoring overload
       |
       v
    Visibility ↓

Capacity planning must include telemetry growth.

---

# 148. Incident Scenario: Observability Platform Becomes the Bottleneck

Symptoms:

- Application healthy
- Monitoring slow
- Dashboards unavailable
- Alerts delayed

This is a production incident for the platform itself.

---

# 149. Incident Scenario: Incident Detection Is Too Slow

Measure:

    Failure time
       |
       v
    Detection time

This is MTTD.

Improve:

- SLO alerts
- Synthetic checks
- Better thresholds
- Business signals

---

# 150. Incident Scenario: Recovery Is Too Slow

Measure:

    Detection
       |
       v
    Diagnosis
       |
       v
    Mitigation
       |
       v
    Validation

MTTR is not improved simply by adding more dashboards.

Improve the entire operational workflow.

---

# 151. Incident Scenario: Engineers Cannot Find Runbook

A runbook should be linked from:

- Alert
- Dashboard
- Service metadata

Example:

    Alert
      |
      v
    Runbook
      |
      v
    Dashboard
      |
      v
    Commands

---

# 152. Incident Scenario: Alert Has No Owner

Every production alert should identify:

    service
    team
    severity
    runbook

Ownership is part of alert quality.

---

# 153. Incident Scenario: Alert Is Too Generic

Bad:

> CPU high.

Better:

> Checkout service CPU saturation is causing request latency above the SLO threshold for 10 minutes.

Actionability matters.

---

# 154. Incident Scenario: Alert Has No Business Context

Add:

- Service
- SLO
- User impact
- Severity
- Runbook

This helps responders prioritize.

---

# 155. Incident Scenario: Incident Escalates Because Nobody Acknowledges

Define:

    Initial notification
       |
       v
    Timeout
       |
       v
    Escalation
       |
       v
    Secondary team

Escalation should be tested.

---

# 156. Incident Scenario: On-Call Engineer Is Unavailable

Have:

- Backup engineer
- Escalation policy
- Incident commander
- Service ownership

Avoid single-person dependency.

---

# 157. Incident Scenario: Multiple Incidents Occur Simultaneously

Prioritize by:

1. Customer impact
2. Business criticality
3. Data/security risk
4. Scope
5. Recovery difficulty

Do not treat all alerts equally.

---

# 158. Incident Scenario: Critical Service and Low-Priority Service Fail Together

Prioritize critical service.

Use incident severity and business impact.

This is why severity labels and service ownership matter.

---

# 159. Incident Scenario: Incident Crosses Multiple Teams

Create one incident coordination channel/process.

Assign:

- Incident commander
- Technical leads
- Communications

Avoid five teams independently making conflicting changes.

---

# 160. Incident Scenario: Incident Commander Is Not Technical

That is acceptable.

The IC coordinates.

Technical lead investigates.

Separating roles reduces cognitive overload.

---

# 161. Incident Scenario: Technical Lead Is Also IC

For small incidents this may be practical.

For major incidents, separate coordination from deep debugging when possible.

---

# 162. Incident Scenario: Stakeholder Communication Becomes Too Frequent

Provide predictable updates:

    Current impact
    Current mitigation
    Known facts
    Next action
    Next update

Avoid flooding responders with ad-hoc questions.

---

# 163. Incident Scenario: Incident Ends Without Validation

Before closure confirm:

- Error rate normal
- Latency normal
- Traffic normal
- Dependencies normal
- Queue recovered
- Business transactions healthy
- Alerts recovered
- No ongoing data loss

---

# 164. Incident Scenario: Recovery Is Partial

Example:

    API healthy
    Queue still growing

Incident is not fully resolved.

Continue until defined recovery criteria are satisfied.

---

# 165. Incident Scenario: Root Cause Is Known but Prevention Is Missing

Postmortem action:

    Root cause
       |
       v
    Engineering fix
       |
       v
    Test
       |
       v
    Monitoring
       |
       v
    Runbook

A postmortem without follow-through has limited value.

---

# 166. Incident Scenario: Same Incident Happens Again

Ask:

> Was the previous corrective action actually completed?

If yes:

> Why did it fail?

Possible issue:

- Wrong hypothesis
- Incomplete fix
- New failure mode
- Missing validation

---

# 167. Incident Scenario: Corrective Action Is "Add More Monitoring"

Ask:

> What specific failure signal was missing?

Then define:

    Metric
    SLI
    Alert
    Threshold
    Owner
    Runbook
    Test

---

# 168. Incident Scenario: Corrective Action Is "Increase Capacity"

Ask:

- Why did capacity become insufficient?
- Was demand expected?
- Is autoscaling correct?
- Is there a bottleneck?
- What is the cost?

Capacity is one possible fix, not always the best fix.

---

# 169. Incident Scenario: Incident Was Caused by Noisy Logs

Prevention:

- Structured logging
- Log levels
- Sampling where appropriate
- Rate limiting
- Volume monitoring

---

# 170. Incident Scenario: Incident Was Caused by Metric Cardinality

Prevention:

- Metric naming standards
- Label allowlists
- Cardinality review
- CI validation
- Ownership

---

# 171. Incident Scenario: Incident Was Caused by Alert Fatigue

Prevention:

- Remove non-actionable pages
- SLO alignment
- Grouping
- Inhibition
- Escalation
- Runbooks

---

# 172. Incident Scenario: Incident Was Caused by Missing Capacity Planning

Prevention:

- Load testing
- Forecasting
- Headroom
- Autoscaling
- Capacity alerts

---

# 173. Incident Scenario: Incident Was Caused by Dependency Failure

Prevention:

- Dependency SLO
- Timeout
- Circuit breaker
- Retry limits
- Fallback
- Dependency monitoring

---

# 174. Incident Scenario: Incident Was Caused by Deployment

Prevention:

- Automated tests
- Canary
- Progressive delivery
- SLO checks
- Automated rollback
- Observability validation

---

# 175. Incident Scenario: Incident Was Caused by Configuration Drift

Prevention:

- GitOps
- Drift detection
- Change review
- Automated deployment

---

# 176. Incident Scenario: Incident Was Caused by Human Error

Prevention:

- Automation
- Guardrails
- Approval
- Least privilege
- Safer defaults
- Runbooks

---

# 177. Incident Scenario: Incident Was Caused by Missing DR

Prevention:

- Backup
- Restore tests
- Multi-AZ
- Multi-region where justified
- Failover tests
- RPO/RTO validation

---

# 178. Incident Scenario: Incident Was Caused by Monitoring Failure

Prevention:

- Monitoring HA
- External checks
- Monitoring SLO
- Capacity planning
- Alert delivery tests

---

# 179. Incident Scenario: Incident Was Caused by Security Event

Response should involve appropriate security processes.

Priorities:

1. Contain.
2. Protect evidence.
3. Assess scope.
4. Rotate credentials where needed.
5. Restore secure operation.
6. Follow security/compliance procedures.

Do not treat a suspected security incident as only a performance problem.

---

# 180. Incident Scenario: Incident Involves Customer Data

Prioritize:

- Containment
- Data protection
- Access control
- Evidence preservation
- Appropriate escalation

Do not expose sensitive information in incident channels unnecessarily.

---

# 181. Incident Scenario: Monitoring Data Contains Customer Data

Review:

- Log fields
- Access permissions
- Retention
- Redaction
- Encryption

Observability data is production data and must be protected.

---

# 182. Incident Scenario: Postmortem Is Blaming an Engineer

Rewrite:

Bad:

> Engineer caused outage.

Better:

> The deployment process allowed an incompatible configuration to reach production without automated validation.

Focus on system improvement.

---

# 183. Blameless Postmortem Structure

## Summary

What happened?

## Impact

Who/what was affected?

## Timeline

What happened and when?

## Detection

How was it detected?

## Response

What actions were taken?

## Root cause

Why did it happen?

## Contributing factors

What made it possible?

## Recovery

How was service restored?

## Corrective actions

What prevents recurrence?

---

# 184. Example Postmortem

## Incident

Checkout errors increased after deployment.

## Impact

Checkout success rate dropped below SLO.

## Detection

Prometheus SLO alert fired.

## Timeline

    10:00 deployment
    10:02 error rate rises
    10:04 alert fires
    10:06 rollout paused
    10:08 rollback
    10:11 errors recover

## Root Cause

New version introduced an inefficient database query.

## Contributing Factors

- No query regression test
- No canary
- Missing database query alert

## Actions

- Add query regression test
- Add canary SLO gate
- Add DB query-rate dashboard

---

# 185. Incident Timeline Format

Use UTC consistently.

Example:

```text
10:00 UTC - Deployment started
10:02 UTC - Error rate increased
10:03 UTC - Customer reports received
10:04 UTC - Alert triggered
10:05 UTC - Incident declared
10:07 UTC - Rollout paused
10:09 UTC - Rollback started
10:11 UTC - Error rate normalized
10:15 UTC - Queue drained
10:20 UTC - Incident resolved
```

---

# 186. What to Preserve During an Incident

Preserve:

- Metrics
- Logs
- Events
- Deployment history
- Git commits
- Configuration versions
- Alert history
- Incident timeline
- Commands/actions taken

Avoid unnecessary destructive actions.

---

# 187. Production Incident Evidence Hierarchy

Strong evidence:

    Direct metrics
    Application logs
    Kubernetes events
    Deployment records
    Dependency telemetry

Supporting evidence:

    User reports
    Engineer observations
    Historical patterns

Hypotheses are not facts until supported.

---

# 188. Incident Hypothesis Tracking

Use:

| Hypothesis | Evidence For | Evidence Against | Status |
|---|---|---|---|
| New deployment | Errors began after release | Need version comparison | Investigating |
| DB failure | DB latency elevated | DB still available | Possible |
| Network | One AZ affected | Other paths healthy | Possible |

This prevents circular debugging.

---

# 189. Incident Command Questions

During interviews, explain:

> I would establish a clear incident owner, separate coordination from technical investigation where possible, maintain a timeline, communicate known facts and use reversible mitigations first.

This demonstrates senior operational maturity.

---

# 190. What Does "Resolved" Mean?

Resolved should mean:

- User impact stopped
- Service is stable
- Dependencies are healthy
- Backlogs recovered
- Alerts are normal
- Business transactions succeed
- No known ongoing data integrity issue

Root cause analysis can continue after incident resolution.

---

# 191. Incident Closure Criteria

Before closure:

1. Customer impact ended.
2. Monitoring confirms recovery.
3. Mitigation is stable.
4. Temporary changes are documented.
5. Follow-up owner is assigned.
6. Timeline is captured.

---

# 192. Production Incident Interview Question

> "Your production application is down. What do you do first?"

Strong answer:

> First I confirm the alert and determine customer impact and scope. I check external availability, then the main request path through DNS, ALB, Kubernetes services and pods. At the same time I establish the timeline and check recent deployments or configuration changes. I avoid making destructive changes before collecting enough evidence. If a recent deployment is clearly responsible and rollback is safe, I prioritize restoring service, then validate recovery using error rate, latency and business success metrics.

---

# 193. Production Incident Interview Question

> "What if you don't know the root cause?"

Answer:

> I would distinguish the leading hypothesis from confirmed root cause. During an active incident, my priority is safe mitigation and service restoration. I would preserve evidence, document hypotheses and continue root-cause analysis after recovery.

---

# 194. Production Incident Interview Question

> "Would you restart the pods?"

Answer:

> Only if restarting is a safe mitigation and I understand the risk. Before restarting, I would collect relevant logs, events and metrics because restart can destroy useful runtime evidence. If the pods are stuck or unhealthy and restart is known to restore service safely, I would use it deliberately.

---

# 195. Production Incident Interview Question

> "Would you roll back immediately?"

Answer:

> If the incident began immediately after a deployment, the new version clearly correlates with the failure and rollback is safe, I would pause the rollout and consider rollback as a mitigation. I would first verify database/schema compatibility and other dependencies because rollback is not always safe.

---

# 196. Production Incident Interview Question

> "How do you know the incident is resolved?"

Answer:

> I would not rely on one metric. I would validate error rate, latency, traffic, dependency health, queue/backlog recovery and business transaction success. I would also confirm that alerts have recovered and that there is no continuing data integrity problem.

---

# 197. Production Incident Interview Question

> "How do you reduce MTTR?"

Answer:

> I improve detection, alert quality, service ownership, dashboards, structured logs, correlation metadata, runbooks, rollback mechanisms and automation. I also analyze incidents to identify where time was spent in detection, diagnosis, mitigation and validation.

---

# 198. Production Incident Interview Question

> "How do you prevent alert fatigue?"

Answer:

> I make pages actionable and align them with user impact or SLO risk. I remove noisy alerts, use grouping and inhibition, tune thresholds and durations, assign ownership and link runbooks. Warning signals can remain visible without waking engineers unnecessarily.

---

# 199. Production Incident Interview Question

> "What if monitoring itself fails?"

Answer:

> The monitoring platform needs its own HA and external monitoring. I would use redundant Prometheus/alerting where appropriate and independent blackbox or platform-level checks so a monitoring failure does not create a complete detection blind spot.

---

# 200. Production Incident Interview Question

> "How do you handle incidents involving multiple teams?"

Answer:

> I establish one incident process and clear ownership. The incident commander coordinates priorities and communication while technical leads investigate individual components. We maintain a shared timeline and avoid conflicting changes.

---

# 201. Production Incident Interview Question

> "What makes a good postmortem?"

Answer:

> A good postmortem is factual, blameless and actionable. It explains impact, timeline, detection, response, root cause, contributing factors and concrete corrective actions with owners and priorities.

---

# 202. Production Incident Interview Question

> "What is the difference between mitigation and root-cause fix?"

Answer:

> Mitigation reduces current customer impact. Root-cause remediation removes or reduces the underlying failure mechanism. For example, restarting an OOMKilled application may restore service temporarily, while fixing the memory leak addresses the root cause.

---

# 203. Production Incident Interview Question

> "How do you troubleshoot a cascading failure?"

Answer:

> I map the dependency chain and identify the earliest abnormal signal. I look for retries, timeouts, connection-pool exhaustion, queue growth and dependency latency. Then I protect the failing dependency and reduce amplification through backoff, circuit breaking, rate limiting or controlled traffic.

---

# 204. Production Incident Interview Question

> "How do you troubleshoot a regional outage?"

Answer:

> I segment metrics by region and confirm whether the failure is isolated. I check traffic, application health, dependencies and infrastructure in each region. If a healthy region can safely accept the load, I can fail over traffic, but only after validating capacity. I then verify both customer recovery and the health of the surviving region.

---

# 205. Production Incident Interview Question

> "What is the role of observability during an incident?"

Answer:

> Metrics tell me what is changing, logs provide detailed event context, dashboards help establish scope and trends, alerts provide detection and routing, and SLOs tell me whether the issue represents meaningful reliability impact.

---

# 206. Production Incident Interview Question

> "What if metrics and logs disagree?"

Answer:

> I validate both signals. I check timestamps, query definitions, data freshness, collection paths and scope. I do not blindly trust a dashboard or a log entry until I understand how the signal was produced.

---

# 207. Production Incident Interview Question

> "How do you decide what to investigate first?"

Use:

    Customer impact
        +
    Scope
        +
    Recent change
        +
    Strong evidence
        +
    Reversibility

Prioritize actions that can safely restore service quickly.

---

# 208. Incident Response Anti-Patterns

Avoid:

- Blame
- Random restarts
- Uncontrolled rollbacks
- Blind scaling
- Permanent silences
- Destructive cleanup
- Untracked manual changes
- Guessing root cause
- Ignoring customers
- Closing incidents too early

---

# 209. Production Incident Command Checklist

## Detect

- [ ] Alert confirmed
- [ ] External health checked
- [ ] User impact identified

## Triage

- [ ] Scope identified
- [ ] Timeline established
- [ ] Recent changes checked
- [ ] Dependencies checked

## Mitigate

- [ ] Safe mitigation selected
- [ ] Change documented
- [ ] Rollback considered
- [ ] Blast radius controlled

## Recover

- [ ] Error rate normal
- [ ] Latency normal
- [ ] Dependencies healthy
- [ ] Backlog recovered
- [ ] Business success restored

## Close

- [ ] Timeline captured
- [ ] Temporary changes documented
- [ ] Root-cause work assigned
- [ ] Postmortem scheduled

---

# 210. Production Incident Scenario Matrix

| Incident | Primary Signals | First Investigation | Typical Mitigation |
|---|---|---|---|
| API outage | 5xx, availability | ALB/K8s/deployment | Rollback/fix |
| Latency | p95/p99 | Dependencies/resources | Reduce load/fix |
| OOM | memory/OOMKilled | Pod/application | Rollback/scale/fix |
| Node failure | NotReady | Node/workloads | Reschedule/replace |
| DB outage | errors/connections | DB health | Failover/protect |
| Queue backlog | depth/age | Consumers | Scale/fix consumers |
| DNS failure | synthetic/DNS | DNS/CoreDNS | Restore routing |
| ELK failure | ingestion lag | Collector/Logstash/ES | Restore pipeline |
| Prometheus failure | scrape/availability | Prometheus/storage | Restore HA |
| Alert storm | alert count | Primary dependency | Group/inhibit |
| SLO burn | burn rate | Error source | Mitigate impact |
| Region failure | regional SLO | Regional health | Failover |
| Security event | access/traffic | Security telemetry | Contain |

---

# 211. Advanced Incident Scenario: Full EKS Outage

Architecture:

    Route53
       |
       v
      ALB
       |
       v
      EKS
       |
       +--> Services
       +--> Database
       +--> Queue

Symptoms:

- API unavailable
- Pods failing
- Alerts firing

Response:

1. Confirm external failure.
2. Check ALB.
3. Check EKS cluster.
4. Check nodes.
5. Check control-plane-related symptoms.
6. Check dependencies.
7. Determine if another region is healthy.
8. Fail over only if safe.
9. Restore service.
10. Validate.

---

# 212. Advanced Incident Scenario: Complete Observability Failure

Symptoms:

    Grafana unavailable
    Prometheus unavailable
    ELK unavailable

Question:

> How do you know whether the application is healthy?

Use:

- External synthetic checks
- Load balancer checks
- Application endpoint tests
- Infrastructure/provider signals
- Direct logs if available
- Business reports

This scenario demonstrates why observability itself needs resilience.

---

# 213. Advanced Incident Scenario: Application and Monitoring Fail Together

Potential shared causes:

- Region failure
- Network failure
- Node failure
- Storage failure

Look for signals outside the affected environment.

Independent monitoring is critical.

---

# 214. Advanced Incident Scenario: Error Rate Is Normal but SLO Is Violated

Check:

- SLI definition
- Availability denominator
- Traffic
- SLO window
- Historical failures

A current error rate can be normal while a rolling SLO remains below target.

---

# 215. Advanced Incident Scenario: Error Rate Is High but SLO Is Not Violated

Possible:

- Low traffic
- Short-duration spike
- SLO window behavior
- SLI excludes affected requests

Verify the SLI definition.

This is a good interview trap.

---

# 216. Advanced Incident Scenario: One Dependency Is Slow but No Errors

Pattern:

    Dependency latency ↑
       |
       v
    Application latency ↑
       |
       v
    User experience degradation

Availability can remain 100% while latency SLO is violated.

---

# 217. Advanced Incident Scenario: One Dependency Fails but Application Remains Healthy

Possible:

    Fallback active

Monitor:

- Fallback rate
- Business success
- Dependency health

A green application can still be operating in degraded mode.

---

# 218. Advanced Incident Scenario: Fallback Becomes the Primary Path

If fallback usage remains high:

    Primary dependency failure
       |
       v
    Fallback
       |
       v
    Extended degradation

Track fallback usage as an operational signal.

---

# 219. Advanced Incident Scenario: Incident Is Caused by a Monitoring Blind Spot

Example:

    No metric for queue age

Queue backlog grows.

No alert.

Users eventually experience delay.

Corrective action:

- Add queue-age SLI.
- Add alert.
- Add dashboard.
- Add runbook.
- Test alert.

---

# 220. Advanced Incident Scenario: Incident Is Caused by Missing Ownership

Alert fires:

    service="orders"

No team assigned.

Response delayed.

Corrective action:

- Service catalog
- Owner metadata
- On-call mapping
- Runbook

---

# 221. Advanced Incident Scenario: Incident Is Caused by Poor Dashboard Design

Dashboard has:

    100 panels
    No service overview
    No deployment markers
    No SLO

Engineer spends 30 minutes locating the failure.

Corrective action:

    Service overview
       |
       v
    Golden signals
       |
       v
    Dependencies
       |
       v
    Logs

---

# 222. Advanced Incident Scenario: Incident Is Caused by Excessive Telemetry

Metrics:

    Millions of series

Logs:

    Huge ingestion

Queries:

    Slow

Monitoring platform becomes unstable.

Corrective action:

- Cardinality governance
- Log volume controls
- Retention
- Query optimization

---

# 223. Advanced Incident Scenario: Observability Cost Is Growing Faster Than Business

Track:

    Cost / service
    Cost / GB logs
    Cost / million samples
    Cost / cluster

Optimize telemetry based on business value.

---

# 224. Advanced Incident Scenario: Production Incident During Black Friday-Type Traffic

Priorities:

1. Confirm demand.
2. Check capacity.
3. Check autoscaling.
4. Protect database.
5. Protect dependencies.
6. Monitor SLO.
7. Control retries.
8. Scale safely.

Do not wait for every resource to hit 100% before acting.

---

# 225. Advanced Incident Scenario: Production Incident During Planned Maintenance

Check whether maintenance was expected.

Compare:

    Maintenance window
    Incident timeline

If correlated:

- Follow maintenance rollback/failover procedure.
- Communicate.
- Validate recovery.

---

# 226. Advanced Incident Scenario: Production Incident During Certificate Renewal

Potential:

    Renewal failure
       |
       v
    Expired cert
       |
       v
    TLS failure

Prevention:

- Renewal monitoring
- Expiry alert
- Automated validation
- Test renewal process

---

# 227. Advanced Incident Scenario: Production Incident After Infrastructure Change

Examples:

- Security group
- Route
- IAM
- Network policy
- Node group
- Load balancer

Check change history.

Infrastructure changes are production changes.

---

# 228. Advanced Incident Scenario: Production Incident After IAM Change

Symptoms:

- ECR pulls fail
- S3 access fails
- AWS API calls fail

Investigate:

- IAM policy
- Role
- Trust relationship
- Service account mapping
- Recent changes

Restore least-privilege access.

---

# 229. Advanced Incident Scenario: Production Incident After Security Group Change

Symptoms:

    Connection timeout

Check:

- Source
- Destination
- Port
- Protocol
- SG references

Network access should be validated after changes.

---

# 230. Advanced Incident Scenario: Production Incident After Node Group Upgrade

Check:

- Node readiness
- Runtime
- CNI
- DNS
- DaemonSets
- Pod scheduling
- Application health

Infrastructure upgrades can affect multiple platform layers.

---

# 231. Advanced Incident Scenario: Production Incident After Kubernetes Version Upgrade

Investigate:

- API compatibility
- Admission policies
- CRDs
- Controllers
- CNI
- Ingress
- Workload behavior

Use staged upgrades and rollback plans.

---

# 232. Advanced Incident Scenario: Production Incident During ArgoCD Sync

Check:

- What changed in Git?
- What resources changed?
- Is drift intentional?
- Did sync remove emergency configuration?
- Did a bad manifest deploy?

Do not blindly sync during an active incident.

---

# 233. Advanced Incident Scenario: Production Incident Caused by GitOps Drift

Desired:

    Git state

Actual:

    Cluster state

If drift is unexpected:

- Identify manual change.
- Determine whether it was emergency mitigation.
- Restore desired state safely.
- Prevent unauthorized manual changes.

---

# 234. Advanced Incident Scenario: Emergency Manual Change Is Required

Safe pattern:

    Assess
      |
      v
    Approve
      |
      v
    Change
      |
      v
    Validate
      |
      v
    Record
      |
      v
    Permanent fix

---

# 235. Advanced Incident Scenario: Incident Has No Clear End Time

Define recovery criteria.

Example:

> Incident ends when checkout success rate exceeds 99.9%, p95 latency returns below SLO threshold, payment dependency is healthy and queue age returns to baseline.

Concrete criteria prevent premature closure.

---

# 236. Advanced Incident Scenario: Incident Has Recovered but Risk Remains

Example:

    Service healthy
    Capacity = 95%

Do not declare operational risk gone.

Continue monitoring and reduce capacity risk.

---

# 237. Advanced Incident Scenario: Incident Causes Error Budget Exhaustion

After recovery:

- Freeze risky changes where policy requires.
- Prioritize reliability work.
- Review incident recurrence.
- Improve detection and prevention.

---

# 238. Advanced Incident Scenario: Incident Requires Customer Communication

Customer-facing status should focus on:

- Impact
- Start time
- Current status
- Workaround if available
- Recovery status

Avoid exposing speculative root cause.

---

# 239. Advanced Incident Scenario: Incident Requires Executive Communication

Executive summary:

    What happened?
    Business impact?
    Current status?
    Mitigation?
    Expected next action?

Avoid deep Kubernetes commands in executive communication.

---

# 240. Advanced Incident Scenario: Incident Requires Technical Handoff

Handoff should include:

- Current state
- What is known
- What is unknown
- Actions already taken
- Current hypothesis
- Next investigation
- Risks

This prevents repeated work.

---

# 241. Incident Handoff Template

```text
Incident:
Impact:
Start time:
Current status:

Known facts:
- 

Leading hypothesis:
- 

Actions completed:
- 

Actions pending:
- 

Risks:
- 

Dashboards:
- 

Logs:
- 

Next update:
-
```

---

# 242. Production Incident Mock Question Set

Practice answering these aloud:

1. Production API is returning 100% 5xx. What do you do?
2. API latency doubled but CPU is normal. Why?
3. EKS has 30 Pending pods. How do you investigate?
4. One node is NotReady. What is your response?
5. All pods are healthy but users cannot access the application.
6. ALB returns 503.
7. CoreDNS is down.
8. Database connections are exhausted.
9. RabbitMQ queue is growing.
10. Payment provider is returning 503.
11. Prometheus storage is full.
12. Prometheus is down during an outage.
13. Grafana is unavailable.
14. Elasticsearch is red.
15. Logstash queue is growing.
16. Alertmanager sends no notifications.
17. 500 alerts fire simultaneously.
18. SLO burn rate spikes.
19. One region is failing.
20. DR failover overloads the second region.
21. A deployment causes errors.
22. Rollback does not fix the issue.
23. Secret rotation breaks authentication.
24. Certificate expires.
25. A feature flag causes an outage.
26. Database migration is incompatible.
27. Cache failure overloads the database.
28. Retry storm begins.
29. Customers detect outage before monitoring.
30. Monitoring platform fails.
31. Observability costs double.
32. Logs contain credentials.
33. Node disk fills.
34. HPA does not scale.
35. HPA scales to maximum but errors continue.
36. A PDB blocks maintenance.
37. An emergency manual change is required.
38. ArgoCD wants to overwrite an emergency fix.
39. Metrics and logs disagree.
40. Root cause is unknown after recovery.

---

# 243. Model Answer Structure for Any Scenario

Use this structure in interviews:

## 1. Confirm

> I would first verify the alert and customer impact.

## 2. Scope

> I would determine whether the issue affects one service, version, region, AZ, namespace or the entire platform.

## 3. Timeline

> I would establish when it started and correlate it with recent changes.

## 4. Signals

> I would check traffic, errors, latency and saturation.

## 5. Dependencies

> I would inspect databases, queues, DNS, networking and external services.

## 6. Mitigate

> I would choose the safest reversible action that reduces customer impact.

## 7. Validate

> I would verify recovery through technical and business signals.

## 8. Prevent

> I would document root cause and implement targeted corrective actions.

---

# 244. What Interviewers Are Looking For

They are not only evaluating commands.

They are evaluating whether you:

- Think systematically
- Prioritize customers
- Understand dependencies
- Know Kubernetes
- Understand metrics
- Understand logs
- Understand alerting
- Can reason from evidence
- Can communicate
- Know when to rollback
- Understand risk
- Distinguish mitigation from root cause
- Think about prevention

---

# 245. Senior-Level Production Incident Mindset

Junior response:

> "I will restart the pod."

Intermediate response:

> "I will check logs and restart if necessary."

Senior response:

> "I will first establish customer impact, scope and timeline, correlate the issue with recent changes, inspect metrics/logs/dependencies, choose a safe reversible mitigation, validate recovery and then drive root-cause prevention."

That difference is important in DevOps interviews.

---

# 246. Final Production Incident Mental Model

    USER
      |
      v
    IMPACT
      |
      v
    DETECTION
      |
      v
    TRIAGE
      |
      v
    SCOPE
      |
      v
    TIMELINE
      |
      v
    EVIDENCE
      |
      +---- Metrics
      +---- Logs
      +---- Events
      +---- Changes
      +---- Dependencies
      |
      v
    MITIGATION
      |
      v
    RECOVERY
      |
      v
    VALIDATION
      |
      v
    ROOT CAUSE
      |
      v
    CORRECTIVE ACTION
      |
      v
    PREVENTION

---

# 247. Final Key Takeaways

> Production incident response is not the same as running troubleshooting commands.

> Start with customer impact.

> Establish scope before diving into details.

> Establish a timeline.

> Check recent changes, but do not assume causation.

> Use metrics to understand behavior.

> Use logs to understand events.

> Use events to understand infrastructure state.

> Check dependencies because microservices fail through dependency chains.

> Prefer reversible mitigations.

> Protect evidence.

> Do not restart everything blindly.

> Do not scale everything blindly.

> Do not silence everything blindly.

> Do not claim root cause without evidence.

> Rollback is a mitigation, not automatically a root-cause fix.

> Recovery must be validated using user-facing and technical signals.

> SLOs help distinguish minor infrastructure events from meaningful reliability incidents.

> Monitoring itself must be resilient.

> Centralized observability must not become a single point of failure.

> Every critical service needs ownership, alerts, dashboards, SLOs and runbooks.

> Every major incident should produce concrete engineering improvements.

> Blameless postmortems improve systems instead of merely assigning fault.

> The goal of incident response is not to prove who was wrong.

> The goal is to restore service safely, understand why it failed and make the system more resilient.

---

# 248. Completion

This completes:

    20-Interview-Preparation/
        07-Production-Incident-Scenarios.md

Next:

    08-Mock-Interview.md
