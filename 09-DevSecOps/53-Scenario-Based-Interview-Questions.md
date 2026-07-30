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

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 21

## A developer reports that Jenkins fails during the SonarQube Quality Gate stage, but the SonarQube dashboard shows "Passed." How would you investigate?

### Expected Approach

Check:

- Jenkins SonarQube plugin configuration
- SonarQube webhook delivery
- Project Key
- Authentication token
- Network connectivity
- Quality Gate timeout
- Jenkins logs
- SonarQube server logs

---

# Scenario 22

## After a successful deployment, only one microservice is unable to communicate with another service inside the Kubernetes cluster. How would you troubleshoot?

### Expected Approach

Verify:

- Service DNS resolution
- Kubernetes Service configuration
- Endpoints
- Network Policies
- Pod labels
- Namespace configuration
- Application logs
- Target service availability

---

# Scenario 23

## Your AWS infrastructure deployment fails with an "Access Denied" error, even though Terraform code has not changed. What would you investigate?

### Expected Approach

Review:

- IAM Role permissions
- IAM policy changes
- Service Control Policies (SCP)
- Temporary credential expiration
- AWS account changes
- Terraform backend access
- CloudTrail logs

---

# Scenario 24

## After a production deployment, CPU usage suddenly reaches 95% across multiple Pods. How would you investigate?

### Expected Approach

Check:

- Recent application changes
- Infinite loops
- Traffic spikes
- HPA behaviour
- Resource requests and limits
- JVM or runtime tuning
- Application profiling
- Database response time

---

# Scenario 25

## ArgoCD continuously reports "OutOfSync" even after every successful synchronization. What could be the reason?

### Expected Approach

Investigate:

- Manual cluster changes
- Mutating admission webhooks
- Helm-generated values
- Resource annotations
- Custom Resource Definitions
- Sync options
- IgnoreDifferences configuration

---

# Scenario 26

## A production Pod is repeatedly getting OOMKilled. Increasing memory fixes the issue temporarily, but it happens again after a few hours. How would you investigate?

### Expected Approach

Verify:

- Memory leaks
- Heap dumps
- Application profiling
- Garbage Collection behaviour
- Resource limits
- Traffic patterns
- Large object allocation
- Recent code changes

---

# Scenario 27

## Your Docker image size suddenly increases from 350 MB to 2.5 GB. How would you determine the cause?

### Expected Approach

Review:

- Dockerfile changes
- Base image updates
- Installed packages
- Multi-stage build usage
- Large dependencies
- Build artifacts
- Layer history
- `.dockerignore` configuration

---

# Scenario 28

## A security team reports that an application is using a vulnerable open-source library even though Dependency Scanning passed. How would you investigate?

### Expected Approach

Check:

- Dependency database updates
- Scanner version
- Transitive dependencies
- SBOM contents
- Package lock files
- Manual dependency additions
- False negatives
- Scanner configuration

---

# Scenario 29

## A Kubernetes Deployment remains in the "Progressing" state for a long time and never completes. What would you check?

### Expected Approach

Verify:

- ReplicaSet status
- Pod readiness
- Image availability
- Scheduling events
- Resource availability
- Readiness probe failures
- Persistent Volume issues
- Deployment strategy

---

# Scenario 30

## During a routine security review, you discover that several developers have AdministratorAccess in the AWS Production account. How would you address this?

### Expected Approach

Take a structured approach:

- Review business requirements
- Identify unused permissions
- Apply Least Privilege
- Use IAM Roles instead of users
- Enable MFA
- Review CloudTrail activity
- Implement periodic access reviews
- Document and approve privileged access

---

# Enterprise Investigation Flow

```text
Production Issue

↓

Identify Scope

↓

Review Recent Changes

↓

Infrastructure

↓

Cloud Services

↓

Kubernetes

↓

Application

↓

Security

↓

Root Cause

↓

Recovery

↓

Validation

↓

Prevent Recurrence
```

---

# Enterprise Best Practices

- Verify identity, access, and configuration changes before assuming application defects.
- Correlate cloud logs, Kubernetes events, and CI/CD history during investigations.
- Follow the Principle of Least Privilege for all production access.
- Keep Terraform state, infrastructure, and Git repositories synchronized.
- Continuously monitor resource utilisation and application behaviour after deployments.
- Investigate recurring issues to identify root causes instead of applying temporary fixes.
- Maintain an up-to-date SBOM and regularly update vulnerability databases.
- Conduct post-incident reviews and convert findings into preventive controls and automation.

---

# Enterprise DevSecOps Scenario-Based Interview Questions

---

# Scenario 31

## A deployment completed successfully, but ArgoCD reports the application as Healthy while customers still receive HTTP 503 Service Unavailable. How would you investigate?

### Expected Approach

Check:

- ALB Target Group health
- Kubernetes Service endpoints
- Pod readiness status
- Ingress configuration
- Backend application logs
- Health check path
- Recent deployment changes
- Network connectivity

---

# Scenario 32

## Your Jenkins pipeline suddenly fails during the Trivy scan because of a database update error. How would you troubleshoot?

### Expected Approach

Verify:

- Internet connectivity
- Trivy database cache
- Trivy version
- Proxy configuration
- Offline database mirror
- Runner disk space
- Registry connectivity
- Retry strategy

---

# Scenario 33

## A developer accidentally deletes the Terraform remote state file stored in Amazon S3. What actions would you take?

### Expected Approach

Investigate:

- S3 Versioning
- Backup availability
- State recovery options
- DynamoDB state lock status
- Cloud resources already provisioned
- Terraform plan before applying
- Team communication
- Recovery validation

---

# Scenario 34

## Falco generates repeated alerts indicating that multiple containers are attempting privilege escalation. How would you respond?

### Expected Approach

Check:

- Affected namespaces
- Container images
- Deployment history
- Kubernetes Audit Logs
- RBAC permissions
- Host activity
- Recent image changes
- Isolate affected workloads if malicious behaviour is confirmed

---

# Scenario 35

## During deployment, Kubernetes Pods cannot connect to Amazon RDS even though the database is healthy. What would you investigate?

### Expected Approach

Verify:

- Security Groups
- Network ACLs
- VPC routing
- RDS endpoint
- DNS resolution
- IAM authentication (if used)
- Secrets
- Database firewall rules

---

# Scenario 36

## After merging a Pull Request, Jenkins builds an older commit instead of the latest code. How would you troubleshoot?

### Expected Approach

Review:

- Git branch configuration
- Jenkins webhook
- SCM polling
- Repository cache
- Branch references
- Detached HEAD state
- Pipeline configuration
- Build parameters

---

# Scenario 37

## A newly deployed application suddenly starts consuming all available database connections. How would you investigate?

### Expected Approach

Check:

- Connection pool configuration
- Connection leaks
- Application logs
- Database monitoring
- Recent code changes
- Query execution time
- Idle connections
- Maximum connection limits

---

# Scenario 38

## Amazon EKS nodes are healthy, but new Pods remain in Pending state. Existing Pods continue running normally. What could be the issue?

### Expected Approach

Verify:

- Available CPU and memory
- Node taints
- Tolerations
- Node selectors
- Affinity rules
- Persistent Volume Claims
- Scheduler events
- Cluster Autoscaler status

---

# Scenario 39

## During a production release, users report intermittent failures while the application appears healthy. How would you approach this issue?

### Expected Approach

Investigate:

- Load Balancer health
- Session persistence
- Multiple application replicas
- Pod restarts
- API latency
- Network packet loss
- DNS resolution
- Backend dependency health

---

# Scenario 40

## A security audit reveals that production Kubernetes Secrets are stored as plain YAML files in the Git repository. How would you remediate this?

### Expected Approach

Take immediate action:

- Remove secrets from Git history
- Rotate all exposed credentials
- Move secrets to AWS Secrets Manager or Kubernetes External Secrets
- Encrypt sensitive configuration
- Enable secret scanning
- Review repository access
- Audit credential usage
- Update deployment process

---

# Enterprise Investigation Flow

```text
Incident Detected

↓

Validate Impact

↓

Review Recent Changes

↓

CI/CD Pipeline

↓

Cloud Infrastructure

↓

Kubernetes

↓

Application

↓

Security Controls

↓

Root Cause

↓

Recovery

↓

Validation

↓

Continuous Improvement
```

---

# Enterprise Best Practices

- Validate infrastructure and application health independently during troubleshooting.
- Protect Terraform state with versioning, backups, and state locking.
- Use automated secret management instead of storing sensitive values in Git.
- Investigate runtime security alerts immediately and preserve evidence.
- Verify cloud networking before assuming application failures.
- Ensure CI/CD pipelines always build the intended source revision.
- Monitor database connection pools and application resource consumption.
- Regularly review Kubernetes scheduling constraints and cluster capacity.
- Implement automated health checks, alerting, and rollback strategies.
- Document every production incident and use the findings to strengthen security and operational resilience.

---

