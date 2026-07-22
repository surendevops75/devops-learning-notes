# Section 18 - Production Incident & Real-World Troubleshooting Scenarios

---

## Scenario 181

### Interview Question

A production deployment was completed successfully, but users immediately started reporting HTTP 500 errors. How would you troubleshoot it?

### Production-Level Answer

A successful deployment only confirms Kubernetes deployed the Pods.

Investigate the application, dependencies, and recent configuration changes before rolling back.

### Approach

Verify

- Deployment rollout
- Pod logs
- Application logs
- Database connectivity
- External APIs
- Recent ConfigMap or Secret changes

Check rollout history.

```bash
kubectl rollout history deployment <deployment-name>
```

### Common Root Causes

- Application bug
- Database unavailable
- Incorrect configuration
- Secret update
- API failure

### Best Practices

- Perform canary deployments
- Monitor application health
- Validate configuration before rollout
- Keep rollback ready

---

## Scenario 182

### Interview Question

Only one microservice in a production environment is failing, while all others are healthy. How would you investigate?

### Production-Level Answer

When only one service is affected, begin with that application's dependencies rather than the cluster infrastructure.

### Approach

Verify

- Pod health
- Service
- Ingress
- Database
- External APIs
- ConfigMaps
- Secrets

Review application logs.

### Common Root Causes

- Application bug
- Dependency failure
- Database issue
- Secret expiration
- Network policy

### Best Practices

- Isolate services
- Monitor service dependencies
- Implement health checks
- Enable distributed tracing

---

## Scenario 183

### Interview Question

Multiple applications across different namespaces suddenly become unavailable. Where do you begin?

### Production-Level Answer

A multi-namespace outage usually indicates shared infrastructure failure.

Start from the cluster components instead of individual applications.

### Approach

Verify

1. API Server
2. Worker Nodes
3. CoreDNS
4. Network
5. Storage
6. Ingress
7. Applications

Correlate Events and monitoring dashboards.

### Common Root Causes

- Worker Node failure
- DNS outage
- Network issue
- Storage issue
- Cloud provider outage

### Best Practices

- Follow structured troubleshooting
- Maintain incident runbooks
- Monitor shared infrastructure
- Correlate logs and metrics

---

## Scenario 184

### Interview Question

An application becomes unavailable immediately after updating a ConfigMap. How would you troubleshoot it?

### Production-Level Answer

Configuration updates can introduce application failures even when Kubernetes resources remain healthy.

Validate the configuration before restarting workloads.

### Approach

Verify

- ConfigMap changes
- Application logs
- Rollout history
- Mounted configuration
- Environment variables

Compare the new configuration with the previous version.

### Common Root Causes

- Invalid configuration
- Missing property
- Syntax error
- Application parsing failure

### Best Practices

- Version ConfigMaps
- Validate configuration
- Test changes in staging
- Use GitOps for configuration management

---

## Scenario 185

### Interview Question

A production incident starts immediately after a Secret rotation. What would you investigate?

### Production-Level Answer

Secret rotation is a common cause of authentication failures.

Verify that applications are consuming the updated credentials correctly.

### Approach

Check

- Secret update
- Pod restart
- Database credentials
- API credentials
- Application logs

Verify authentication manually.

### Common Root Causes

- Incorrect Secret
- Application cache
- Expired credentials
- Synchronization issue

### Best Practices

- Rotate Secrets gradually
- Validate credentials
- Automate Secret updates
- Test authentication after rotation

---

## Scenario 186

### Interview Question

A Kubernetes cluster appears healthy, but users report intermittent failures throughout the day. How would you investigate?

### Production-Level Answer

Intermittent failures require correlation across logs, metrics, and events because they may not be visible during investigation.

### Approach

Review

- Prometheus metrics
- Grafana dashboards
- Application logs
- Kubernetes Events
- Deployment history

Look for patterns matching user reports.

### Common Root Causes

- Resource spikes
- Network latency
- External dependency failures
- Autoscaling delays
- Application bugs

### Best Practices

- Correlate logs with metrics
- Monitor latency
- Define SLOs
- Capture incident timelines

---

## Scenario 187

### Interview Question

A production deployment succeeds, but only one replica continues serving the old application version. How would you troubleshoot it?

### Production-Level Answer

Mixed application versions usually indicate rollout or scheduling inconsistencies.

Verify ReplicaSets and Pod versions.

### Approach

Check

```bash
kubectl get rs
```

Verify Pod images.

```bash
kubectl get pods -o wide
```

Review rollout status.

### Common Root Causes

- Rollout interrupted
- Old ReplicaSet
- Image cache
- Manual Pod modification

### Best Practices

- Use immutable image tags
- Monitor rollouts
- Remove obsolete ReplicaSets
- Validate deployment completion

---

## Scenario 188

### Interview Question

An external API outage causes cascading failures across multiple microservices. How would you respond?

### Production-Level Answer

External dependencies should fail gracefully without affecting the entire platform.

Focus on reducing customer impact before restoring functionality.

### Approach

Verify

- External API availability
- Retry logic
- Circuit breaker
- Application logs
- Queue backlogs

Reduce dependency pressure where possible.

### Common Root Causes

- Third-party outage
- Aggressive retries
- Timeout configuration
- Missing circuit breaker

### Best Practices

- Implement circuit breakers
- Configure retries carefully
- Use exponential backoff
- Monitor third-party dependencies

---

## Scenario 189

### Interview Question

CPU, Memory, Network, and Storage all appear healthy, but application latency continues increasing. What would you investigate?

### Production-Level Answer

Infrastructure metrics alone cannot explain all performance issues.

Investigate application-level bottlenecks and downstream dependencies.

### Approach

Review

- Application response time
- Database queries
- API latency
- Thread utilization
- Garbage collection
- Connection pools

Compare performance with previous releases.

### Common Root Causes

- Slow SQL queries
- Lock contention
- Inefficient algorithms
- Connection pool exhaustion
- Application regression

### Best Practices

- Monitor application metrics
- Profile performance regularly
- Optimize database queries
- Benchmark new releases

---

## Scenario 190

### Interview Question

You are the primary on-call DevOps Engineer during a major production outage affecting customer-facing applications. What is your overall troubleshooting strategy?

### Production-Level Answer

During a major incident, avoid making assumptions or random changes.

Follow a structured incident response process to identify the root cause while minimizing business impact.

### Approach

1. Acknowledge the incident.
2. Assess customer impact.
3. Review recent deployments and infrastructure changes.
4. Verify Control Plane health.
5. Verify Worker Nodes.
6. Verify Networking.
7. Verify Storage.
8. Verify Application health.
9. Correlate logs, metrics, and Events.
10. Mitigate impact.
11. Perform Root Cause Analysis (RCA).
12. Document lessons learned.

### Common Root Causes

- Infrastructure failure
- Faulty deployment
- Configuration change
- External dependency outage
- Cloud provider issue
- Human error

### Best Practices

- Follow documented runbooks
- Communicate regularly with stakeholders
- Avoid unnecessary changes during incidents
- Perform blameless RCAs
- Continuously improve operational procedures

---

# End of Kubernetes Scenario-Based Interview Questions