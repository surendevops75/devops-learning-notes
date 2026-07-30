# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Introduction

Scenario-based interviews are designed to evaluate how you think, investigate, troubleshoot, communicate, and make technical decisions under real production conditions.

Unlike theoretical interviews, there is usually **no single correct answer**. Interviewers want to understand your investigation process, technical reasoning, security mindset, and ability to restore production safely.

In enterprise interviews, you are expected to think across the entire DevSecOps platform rather than focusing on a single tool.

---

# How to Answer Every Scenario

Always use a structured approach.

```text
Understand the Problem

↓

Collect Information

↓

Identify the Scope

↓

Analyze Possible Causes

↓

Verify the Root Cause

↓

Implement the Fix

↓

Validate the Solution

↓

Prevent Future Occurrences
```

---

# Enterprise DevSecOps Pipeline

```text
Developer

↓

Feature Branch

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Trigger

↓

Checkout

↓

Build

↓

Unit Tests

↓

Coverage

↓

SonarQube

↓

Policy / Quality Gate

↓

Docker Build

↓

Container Scan

↓

SBOM

↓

Image Signing

↓

Artifact Repository

↓

GitOps

↓

ArgoCD

↓

Amazon EKS

↓

Smoke Test

↓

Production
```

---

# Scenario 1

## Your company follows GitOps using Jenkins, SonarQube, Trivy, ArgoCD, and Amazon EKS.

A developer reports that the Jenkins pipeline completed successfully.

- SonarQube Quality Gate passed.
- Trivy found no Critical vulnerabilities.
- Docker image was pushed successfully.
- ArgoCD synchronized successfully.
- Kubernetes deployment completed without errors.

However, users suddenly receive **HTTP 502 Bad Gateway** after deployment.

As the DevSecOps Engineer, how would you investigate this production issue?

---

## Answer

A successful CI/CD pipeline does **not** guarantee a healthy application.

I would troubleshoot from the **user entry point toward the application**, validating every layer before making changes.

---

## Step 1 — Confirm the Incident

First, determine whether the issue affects:

- One user
- One application
- One environment
- One Availability Zone
- Entire production

Questions to answer:

- Is every request failing?
- Did the problem begin immediately after deployment?
- Are previous application versions working?
- Are all services affected?

---

## Step 2 — Check Recent Changes

Before assuming infrastructure failure, verify what changed.

Review:

- Git commits
- Pull Request
- Jenkins build
- Deployment manifest
- Helm values
- ArgoCD sync history
- Image tag
- Recent infrastructure changes

---

## Change Timeline

```text
Git Commit

↓

Pull Request

↓

Merge

↓

Pipeline

↓

Deployment

↓

502 Errors
```

---

## Step 3 — Verify Load Balancer

Since users receive **502 Bad Gateway**, the first production component to investigate is the Load Balancer.

Verify:

- Target Group Health
- Listener Rules
- Target Registration
- Health Check Status

Possible issues include:

- Unhealthy targets
- Failed health checks
- Incorrect listener configuration
- Wrong backend port

---

## Step 4 — Verify Kubernetes Ingress

Check whether the Ingress routes traffic correctly.

Validate:

- Hostname
- Paths
- Backend Service
- TLS configuration
- Ingress events

An incorrect backend service or port can result in 502 responses even when the deployment succeeds.

---

## Request Flow

```text
User

↓

Route53

↓

Application Load Balancer

↓

Ingress

↓

Kubernetes Service

↓

Pods

↓

Application
```

---

## Step 5 — Verify Kubernetes Service

Check:

- Service type
- Port
- TargetPort
- Selector labels
- Endpoints

If selectors do not match Pod labels, the Service will have no healthy endpoints.

---

## Step 6 — Verify Pod Health

Inspect:

- Running status
- Restart count
- Readiness probe
- Liveness probe
- Startup probe

A pod may be running but still unavailable because the readiness probe is failing.

---

## Step 7 — Check Application Logs

Review:

- Current logs
- Previous logs
- Exception stack traces
- Database connection errors
- Configuration errors
- Startup failures

Never assume Kubernetes is the problem before checking the application itself.

---

## Step 8 — Verify Configurations

Confirm that:

- ConfigMaps are correct
- Secrets are mounted
- Environment variables are available
- Certificates are valid
- External endpoints are reachable

Many production failures occur because configuration values differ between environments.

---

## Step 9 — Validate Dependencies

Check connectivity to:

- Database
- Redis
- Message Queue
- Third-party APIs
- Internal microservices

A healthy application may still return 502 errors if required dependencies are unavailable.

---

## Step 10 — Roll Back if Necessary

If the new release is confirmed as the root cause:

- Roll back through GitOps.
- Allow ArgoCD to synchronize the previous version.
- Validate application health.
- Continue investigating in a safe environment.

Avoid making manual changes directly in the cluster.

---

## Investigation Flow

```text
User Reports 502

↓

Confirm Incident

↓

Review Recent Changes

↓

ALB Health

↓

Ingress

↓

Service

↓

Endpoints

↓

Pods

↓

Application Logs

↓

Configuration

↓

Dependencies

↓

Root Cause

↓

Rollback (if required)

↓

Restore Service
```

---

## What the Interviewer is Evaluating

The interviewer wants to assess whether you:

- Troubleshoot methodically instead of guessing.
- Understand the complete request flow.
- Know the relationship between AWS and Kubernetes.
- Validate infrastructure before changing configurations.
- Consider application issues as well as platform issues.
- Use GitOps correctly instead of making manual production fixes.
- Prioritize service restoration while investigating the root cause.

---

# Enterprise Best Practices

- Start troubleshooting from the user-facing layer and move inward.
- Verify recent changes before assuming infrastructure failure.
- Use logs, events, metrics, and health checks to identify the root cause.
- Avoid manual production changes in GitOps-managed environments.
- Roll back safely when customer impact is high.
- Validate every layer of the request path before concluding the investigation.
- Document the incident timeline, root cause, resolution, and preventive actions.
- Treat production incidents as opportunities to improve automation, monitoring, and deployment reliability.