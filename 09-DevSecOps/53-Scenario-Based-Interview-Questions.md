# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 1

## Jenkins pipeline completed successfully, ArgoCD synced successfully, but users receive HTTP 502 Bad Gateway immediately after deployment. How would you investigate?

### Expected Approach

Investigate from the user entry point:

- ALB Target Groups
- Ingress Controller
- Kubernetes Service
- Endpoints
- Pod Readiness
- Application Logs
- Backend Dependencies

Never assume a successful deployment means a healthy application.

---

# Scenario 2

## SonarQube Quality Gate passed, but the application crashes immediately after deployment. What could be the reason?

### Expected Approach

Remember that SonarQube checks code quality, not runtime behaviour.

Investigate:

- Application startup logs
- Environment variables
- ConfigMaps
- Secrets
- Database connectivity
- JVM/Runtime configuration
- Resource limits

---

# Scenario 3

## Trivy reports a Critical vulnerability in the Docker image just before production deployment. The business wants an urgent release. What would you do?

### Expected Approach

- Validate whether the vulnerability is exploitable.
- Check whether a fixed image exists.
- Assess business impact.
- Block deployment if the risk is unacceptable.
- Use the organization's risk acceptance process only when justified and approved.

---

# Scenario 4

## A developer accidentally commits AWS Access Keys into GitHub. The repository has already been pushed. What actions would you take?

### Expected Approach

Immediately:

- Revoke the exposed keys.
- Generate new credentials.
- Remove secrets from Git history.
- Scan repositories using Gitleaks.
- Review CloudTrail for unauthorized activity.
- Inform the security team.
- Replace hardcoded secrets with AWS Secrets Manager.

---

# Scenario 5

## ArgoCD shows "Synced", but the application version running in Kubernetes is still the old version. How would you troubleshoot?

### Expected Approach

Verify:

- Image tag
- ImagePullPolicy
- Deployment rollout
- ReplicaSet
- Pod status
- Git repository revision
- ArgoCD application history
- Container image digest

---

# Scenario 6

## After deploying a new version, Kubernetes Pods remain in Running state, but Readiness probes continue to fail. What would you investigate?

### Expected Approach

Check:

- Readiness endpoint
- Application startup logs
- Database availability
- External API connectivity
- Configuration
- Secrets
- Resource consumption
- Probe timeout configuration

---

# Scenario 7

## Terraform deployment suddenly fails even though the code hasn't changed. How would you investigate?

### Expected Approach

Review:

- Terraform state
- Backend availability
- Provider version
- Cloud credentials
- IAM permissions
- Resource drift
- API limits
- Recently modified infrastructure

---

# Scenario 8

## Your Amazon EKS application suddenly becomes very slow after a successful deployment. CPU usage is normal. Where would you investigate?

### Expected Approach

Check:

- Database latency
- Redis cache
- DNS resolution
- Network latency
- API dependencies
- Storage performance
- Application logs
- Thread pool utilisation

Performance issues are not always caused by CPU.

---

# Scenario 9

## Falco generates an alert stating that a shell was opened inside a production container. What actions would you take?

### Expected Approach

- Treat the alert as a potential security incident.
- Identify the affected pod.
- Verify who accessed the container.
- Review Kubernetes Audit Logs.
- Collect container logs.
- Isolate the workload if required.
- Investigate for compromise.
- Rotate affected credentials if necessary.

---

# Scenario 10

## A deployment completed successfully, but only users from one region are reporting application failures. How would you approach the investigation?

### Expected Approach

Investigate:

- Route53 routing
- Regional ALB health
- DNS propagation
- Network connectivity
- Regional infrastructure
- Kubernetes cluster health
- Backend service availability
- Recent regional changes

Avoid assuming the application itself is the root cause.

---

# Enterprise Investigation Flow

```text
Understand Problem

↓

Collect Evidence

↓

Check Recent Changes

↓

Infrastructure

↓

Platform

↓

Application

↓

Dependencies

↓

Root Cause

↓

Fix

↓

Validation

↓

Post-Incident Review
```

---

# Enterprise Best Practices

- Never troubleshoot based on assumptions.
- Always verify the scope and business impact first.
- Follow a structured investigation process from infrastructure to application.
- Correlate logs, metrics, events, and deployment history.
- Use GitOps for rollbacks instead of manual production changes.
- Preserve audit logs and evidence during security incidents.
- Document the root cause and preventive actions after every incident.
- Continuously improve monitoring, alerting, and automation based on production learnings.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 11

## A Jenkins pipeline suddenly starts failing at the Docker Build stage even though no code changes were made. How would you investigate?

### Expected Approach

Investigate:

- Jenkins agent availability
- Docker daemon status
- Disk space
- Build cache
- Base image availability
- Dockerfile changes
- Registry authentication
- Recent Jenkins plugin updates

---

# Scenario 12

## After deploying a new application version, Pods start showing `ImagePullBackOff`. The image exists in Amazon ECR. What would you check?

### Expected Approach

Verify:

- Image tag
- Repository name
- IAM Role for Service Account (IRSA)
- Node IAM permissions
- ECR authentication
- ImagePullSecrets
- Network connectivity
- Repository policy

---

# Scenario 13

## Your application works correctly inside the cluster, but users cannot access it through the Application Load Balancer. How would you troubleshoot?

### Expected Approach

Check:

- Route53 DNS
- ALB Listener Rules
- Target Group Health
- Ingress configuration
- Security Groups
- Kubernetes Service
- Service Endpoints
- Pod readiness

---

# Scenario 14

## A Terraform deployment attempts to recreate existing production resources instead of updating them. What could be the cause?

### Expected Approach

Investigate:

- Terraform state file
- Remote backend
- State locking
- Resource import status
- Workspace selection
- Drift between state and infrastructure
- Provider configuration

---

# Scenario 15

## After a successful deployment, users report extremely slow response times. Monitoring shows low CPU and memory utilisation. How would you investigate?

### Expected Approach

Check:

- Database query performance
- External API latency
- Redis cache
- Network latency
- DNS resolution
- Storage IOPS
- Thread pool exhaustion
- Connection pool utilisation

---

# Scenario 16

## Gitleaks detects hardcoded credentials in a Pull Request. The developer says they are test credentials. What would you do?

### Expected Approach

- Validate whether the credentials are active.
- Block the merge until verified.
- Remove secrets from Git history if necessary.
- Rotate credentials if exposed.
- Educate the developer.
- Store secrets in a secrets manager instead of Git.

---

# Scenario 17

## After updating Kubernetes Secrets, the application continues using the old credentials. Why?

### Expected Approach

Verify:

- Secret update completed
- Deployment rollout
- Pod restart
- Secret mount type
- Environment variable loading
- Application secret reload capability

Remember that many applications only read environment variables during startup.

---

# Scenario 18

## ArgoCD reports an application as `OutOfSync`. What would you investigate?

### Expected Approach

Review:

- Git repository
- Desired manifest
- Live Kubernetes resources
- Manual cluster changes
- Sync policy
- Failed sync operations
- Resource health
- Drift between Git and cluster

---

# Scenario 19

## Falco reports that a container attempted to modify a sensitive system file. How would you respond?

### Expected Approach

Treat it as a potential security incident.

Investigate:

- Container identity
- Namespace
- Pod owner
- Recent deployments
- Container logs
- Kubernetes Audit Logs
- User activity
- Possible privilege escalation

Contain the workload if malicious behaviour is confirmed.

---

# Scenario 20

## During a production deployment, SonarQube passes, Trivy passes, Checkov passes, and Jenkins succeeds. However, customers report missing data after deployment. How would you investigate?

### Expected Approach

Check:

- Database migration scripts
- Application version compatibility
- Rollback history
- Backup availability
- Deployment sequence
- API version changes
- Data validation
- Application logs

Remember that security scans validate security posture—they do not guarantee application functionality or data integrity.

---

# Enterprise Investigation Flow

```text
Incident Report

↓

Confirm Impact

↓

Review Recent Changes

↓

Infrastructure

↓

Platform

↓

Application

↓

Dependencies

↓

Root Cause

↓

Recovery

↓

Validation

↓

Lessons Learned
```

---

# Enterprise Best Practices

- Always validate the business impact before beginning technical troubleshooting.
- Compare recent deployments, infrastructure changes, and configuration updates.
- Correlate logs, metrics, traces, and audit events to identify the root cause.
- Avoid manual fixes in GitOps-managed environments unless required for emergency recovery.
- Preserve evidence during security incidents before making irreversible changes.
- Test rollback procedures regularly to ensure rapid recovery.
- Perform post-incident reviews and implement preventive controls.
- Continuously improve observability, automation, and deployment practices based on production experience.

---

