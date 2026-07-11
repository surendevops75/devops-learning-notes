# Liveness and Readiness Probes

## Introduction

Kubernetes can automatically determine:

```text
Is Application Running?

Is Application Ready To Serve Traffic?
```

---

This is achieved using:

```text
Liveness Probe

Readiness Probe
```

---

These probes are critical in production environments.

---

Without probes:

```text
Application May Be Down

But Pod Appears Healthy
```

---

Result

```text
Users Experience Failures
```

---

# Why Do We Need Probes?

Suppose:

```text
Java Application Started
```

---

Application takes:

```text
60 Seconds
```

to become available.

---

Pod Status

```text
Running
```

---

Application Status

```text
Not Ready
```

---

Without probes:

```text
Traffic Reaches Application

Application Fails
```

---

Kubernetes needs a mechanism to verify application health.

---

# What is a Liveness Probe?

Liveness Probe checks:

```text
Is Application Alive?
```

---

Purpose

```text
Detect Stuck Applications

Detect Hung Processes

Automatically Restart Containers
```

---

If liveness probe fails:

```text
Container Restarted
```

---

# Liveness Probe Flow

```text
Container Running
       ↓

Liveness Check
       ↓

Success
       ↓

Continue Running
```

---

If failed:

```text
Container Running
       ↓

Liveness Check
       ↓

Failure
       ↓

Restart Container
```

---

# What is a Readiness Probe?

Readiness Probe checks:

```text
Can Application Accept Traffic?
```

---

Purpose

```text
Control Traffic Flow
```

---

If readiness probe fails:

```text
Pod Removed
From Service Endpoints
```

---

Container continues running.

---

No restart occurs.

---

# Readiness Probe Flow

```text
Pod Started
      ↓

Readiness Check
      ↓

Ready
      ↓

Receive Traffic
```

---

Failure

```text
Readiness Check
      ↓

Failed
      ↓

No Traffic
```

---

Container still runs.

---

# Liveness vs Readiness

| Feature | Liveness | Readiness |
|----------|----------|----------|
| Purpose | Check Health | Check Availability |
| Restart Container | Yes | No |
| Controls Traffic | No | Yes |
| Failed Result | Restart | Remove From Service |
| Production Usage | Yes | Yes |

---

# Example Scenario

Application

```text
Catalogue Service
```

---

Situation

```text
Application Hung
```

---

Liveness Probe

```text
Fails
```

---

Result

```text
Container Restarted
```

---

Situation

```text
Database Unavailable
```

---

Application Running

But

```text
Cannot Serve Requests
```

---

Readiness Probe

```text
Fails
```

---

Result

```text
Traffic Stopped
```

---

# Types of Probes

Kubernetes supports:

```text
HTTP Probe

TCP Probe

Exec Probe
```

---

# HTTP Probe

Most common.

---

Example

```yaml
livenessProbe:

  httpGet:

    path: /health

    port: 8080
```

---

Kubernetes calls:

```text
http://pod-ip:8080/health
```

---

Expected

```text
HTTP 200
```

---

Success

```text
Healthy
```

---

# HTTP Readiness Probe

Example

```yaml
readinessProbe:

  httpGet:

    path: /ready

    port: 8080
```

---

Application must return:

```text
HTTP 200
```

---

Otherwise:

```text
Traffic Blocked
```

---

# TCP Probe

Checks:

```text
Port Reachability
```

---

Example

```yaml
livenessProbe:

  tcpSocket:

    port: 8080
```

---

Kubernetes verifies:

```text
Port Open?
```

---

Success

```text
Healthy
```

---

# Exec Probe

Runs Command Inside Container.

---

Example

```yaml
livenessProbe:

  exec:

    command:

    - cat

    - /tmp/healthy
```

---

Success

```text
Exit Code = 0
```

---

Failure

```text
Non-Zero Exit Code
```

---

# Probe Parameters

Common Settings

```yaml
initialDelaySeconds

periodSeconds

timeoutSeconds

failureThreshold
```

---

# initialDelaySeconds

Wait before first probe.

---

Example

```yaml
initialDelaySeconds: 30
```

---

Meaning

```text
Wait 30 Seconds
Before Checking
```

---

Useful for:

```text
Slow Starting Applications
```

---

# periodSeconds

How often probe runs.

---

Example

```yaml
periodSeconds: 10
```

---

Meaning

```text
Check Every 10 Seconds
```

---

# timeoutSeconds

Maximum wait time.

---

Example

```yaml
timeoutSeconds: 5
```

---

Meaning

```text
Fail If No Response
Within 5 Seconds
```

---

# failureThreshold

Number of failures before action.

---

Example

```yaml
failureThreshold: 3
```

---

Meaning

```text
Fail 3 Times
Then Take Action
```

---

# Complete Example

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:

  containers:

  - name: nginx

    image: nginx

    livenessProbe:

      httpGet:
        path: /
        port: 80

      initialDelaySeconds: 30

      periodSeconds: 10

    readinessProbe:

      httpGet:
        path: /
        port: 80

      initialDelaySeconds: 10

      periodSeconds: 5
```

---

# Services and Readiness

Suppose:

```text
3 Pods
```

behind Service.

---

Pod-1

```text
Ready
```

---

Pod-2

```text
Ready
```

---

Pod-3

```text
Not Ready
```

---

Service sends traffic only to:

```text
Pod-1

Pod-2
```

---

Pod-3 excluded automatically.

---

# Real Production Example

Frontend Application

---

Health Endpoint

```text
/health
```

---

Readiness Endpoint

```text
/ready
```

---

Load Balancer Traffic

```text
Only To Healthy Pods
```

---

Benefits

```text
Zero Downtime

Better Reliability

Safer Deployments
```

---

# Deployment and Readiness

During rolling updates:

```text
New Pod Created
```

---

Readiness Probe

```text
Must Pass
```

---

Only then:

```text
Old Pod Removed
```

---

This prevents:

```text
Downtime
```

---

# Common Problems

## Probe Path Wrong

Example

```yaml
path: /healthz
```

---

Application exposes

```yaml
path: /health
```

---

Result

```text
Probe Failure
```

---

# Application Starts Slowly

Example

```text
Java Application
```

---

Startup

```text
60 Seconds
```

---

Probe Starts

```text
10 Seconds
```

---

Result

```text
Continuous Restarts
```

---

Solution

```yaml
initialDelaySeconds: 60
```

---

# Troubleshooting Probes

Check Pod

```bash
kubectl describe pod pod-name
```

---

Events Section

```text
Liveness Probe Failed

Readiness Probe Failed
```

---

View Logs

```bash
kubectl logs pod-name
```

---

Test Endpoint

```bash
kubectl exec -it pod-name -- curl localhost:8080/health
```

---

# Common Interview Questions

## What is a Liveness Probe?

### Short Answer

A liveness probe checks whether an application is alive and should continue running.

### Detailed Explanation

If the probe fails repeatedly, Kubernetes restarts the container.

### Production Example

Restarting a hung Java application.

---

## What is a Readiness Probe?

### Short Answer

A readiness probe checks whether an application is ready to receive traffic.

### Detailed Explanation

If the probe fails, Kubernetes removes the Pod from Service endpoints.

### Production Example

Blocking traffic until database connections are established.

---

## Difference Between Liveness and Readiness?

### Short Answer

Liveness determines whether a container should be restarted.

Readiness determines whether a Pod should receive traffic.

### Production Example

Application alive but database unavailable.

---

## What Happens If Liveness Probe Fails?

### Answer

```text
Container Restarted
```

---

## What Happens If Readiness Probe Fails?

### Answer

```text
Pod Removed From Service Endpoints

Container Continues Running
```

---

# Key Takeaways

```text
Liveness checks if application is alive.

Readiness checks if application is ready.

Liveness failures restart containers.

Readiness failures stop traffic.

Readiness probes are critical for rolling updates.

HTTP probes are most commonly used.

TCP and Exec probes are also available.

Proper probe configuration improves reliability and availability.

Probes are essential in production Kubernetes environments.
```