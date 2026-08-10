# GitHub Actions - Java Microservices CI/CD Project

This project demonstrates how to design and implement a production-style
CI/CD pipeline for a Java microservices application using GitHub Actions.

The pipeline covers:

```
Source Code
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    ↓
Build
    |
    ↓
Unit Tests
    |
    ↓
Code Quality
    |
    ↓
Security Scanning
    |
    ↓
Docker Build
    |
    ↓
Container Scan
    |
    ↓
Amazon ECR
    |
    ↓
Deployment
    |
    ↓
Kubernetes / EKS
    |
    ↓
Validation
    |
    ↓
Monitoring
```

This project focuses specifically on a Java microservices CI/CD
implementation.

---

# 1. Project Overview

The application consists of multiple Java-based microservices.

Example services:

```
User Service
Product Service
Cart Service
Order Service
Payment Service
Inventory Service
Notification Service
```

Each service is independently buildable and deployable.

Example architecture:

```
GitHub Repository
      |
      +-------------------+
      |                   |
      ↓                   ↓
  User Service       Product Service
      |                   |
      ↓                   ↓
  Order Service      Inventory Service
      |                   |
      +---------+---------+
                |
                ↓
          Kubernetes
                |
                ↓
               EKS
```

---

# 2. Project Objective

The objective is to build a CI/CD pipeline that automatically:

```
1. Validates source code
2. Builds the Java application
3. Runs unit tests
4. Performs code-quality analysis
5. Performs security checks
6. Creates a Docker image
7. Scans the Docker image
8. Pushes the image to Amazon ECR
9. Deploys the application
10. Validates the deployment
11. Supports rollback
```

---

# 3. Technology Stack

## Application

```
Java
Spring Boot
Maven
REST APIs
Microservices
```

## Source Control

```
Git
GitHub
```

## CI/CD

```
GitHub Actions
```

## Code Quality

```
SonarQube
```

## Security

```
Trivy
Veracode
Dependency Scanning
Secret Detection
```

## Containerization

```
Docker
```

## Container Registry

```
Amazon ECR
```

## Infrastructure

```
Terraform
```

## Container Orchestration

```
Kubernetes
Amazon EKS
```

## Deployment

```
Helm
ArgoCD
```

## Monitoring

```
Prometheus
Grafana
ELK Stack
```

---

# 4. High-Level Architecture

The complete application delivery flow is:

```
Developer
    |
    ↓
GitHub
    |
    ↓
Pull Request
    |
    ↓
Code Review
    |
    ↓
GitHub Actions
    |
    +--- Checkout
    |
    +--- Java Setup
    |
    +--- Maven Build
    |
    +--- Unit Tests
    |
    +--- SonarQube
    |
    +--- Security Scan
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
Amazon ECR
    |
    ↓
Deployment
    |
    ↓
EKS
    |
    ↓
Health Validation
    |
    ↓
Monitoring
```

---

# 5. Repository Structure

A simple repository structure can look like:

```
java-microservices/
│
├── user-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── product-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── order-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── payment-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── inventory-service/
│   ├── src/
│   ├── pom.xml
│   └── Dockerfile
│
├── helm/
│   └── application/
│
└── .github/
    └── workflows/
        ├── ci.yml
        └── cd.yml
```

---

# 6. Microservice Structure

Each Java service follows a standard Maven project structure.

Example:

```
user-service/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   └── resources/
│   │
│   └── test/
│       └── java/
│
├── pom.xml
└── Dockerfile
```

---

# 7. Maven Project

The `pom.xml` manages:

```
Dependencies
    +
Plugins
    +
Build Configuration
    +
Test Configuration
    +
Packaging
```

Typical build command:

```
mvn clean package
```

For CI:

```
mvn clean test
```

or:

```
mvn clean package
```

---

# 8. CI/CD Flow

The pipeline can be divided into two major parts.

## CI

```
Checkout
    |
    ↓
Java Setup
    |
    ↓
Dependency Installation
    |
    ↓
Unit Tests
    |
    ↓
Maven Build
    |
    ↓
SonarQube
    |
    ↓
Security Scan
    |
    ↓
Docker Build
    |
    ↓
Trivy
```

## CD

```
ECR
    |
    ↓
Image Reference
    |
    ↓
Helm / GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Validation
```

---

# 9. GitHub Actions Workflow

The workflow is stored under:

```
.github/workflows/
```

Example:

```
.github/workflows/java-ci.yml
```

The workflow should be triggered when application code changes.

Typical triggers:

```
push
    +
pull_request
    +
workflow_dispatch
```

---

# 10. Pull Request Workflow

When a developer creates a pull request:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- Build
    +--- Test
    +--- SonarQube
    +--- Security
    |
    ↓
Required Checks
    |
    ↓
Code Review
    |
    ↓
Merge
```

The pull request should not be merged if mandatory checks fail.

---

# 11. Checkout Code

The first CI step is to retrieve the repository source code.

Flow:

```
GitHub
    |
    ↓
Runner
    |
    ↓
Repository Checkout
```

This allows subsequent steps to access:

```
Java Source
    +
pom.xml
    +
Tests
    +
Dockerfile
```

---

# 12. Configure Java

The runner needs the appropriate Java version.

For example:

```
Java 17
```

or another version required by the application.

Flow:

```
Runner
    |
    ↓
Configure JDK
    |
    ↓
Maven
```

The Java version should match the application's build requirements.

---

# 13. Maven Dependency Cache

Java builds frequently download dependencies.

Without caching:

```
Workflow
    |
    ↓
Download Maven Dependencies
    |
    ↓
Build
```

With caching:

```
Workflow
    |
    ↓
Maven Cache
    |
    ↓
Build
```

Caching reduces unnecessary network downloads.

---

# 14. Dependency Installation

Maven resolves dependencies from configured repositories.

Flow:

```
pom.xml
    |
    ↓
Maven
    |
    ↓
Dependency Resolution
    |
    ↓
Local Maven Repository
```

---

# 15. Unit Testing

Unit tests should run before packaging and deployment.

Flow:

```
Source Code
    |
    ↓
Maven
    |
    ↓
Unit Tests
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

A failing unit test should normally stop the CI pipeline.

---

# 16. Test Reports

Test results should be preserved for troubleshooting.

Example:

```
Test Execution
    |
    ↓
Test Reports
    |
    ↓
GitHub Actions
```

Useful information includes:

```
Passed Tests
    +
Failed Tests
    +
Error Messages
    +
Execution Time
```

---

# 17. Maven Build

After tests succeed:

```
mvn clean package
```

The build creates the application artifact.

For example:

```
target/application.jar
```

Flow:

```
Source
    |
    ↓
Maven
    |
    ↓
JAR
    |
    ↓
Docker
```

---

# 18. Build Failure

If Maven fails:

```
Source
    |
    ↓
Maven Build
    |
    ↓
Failure
    |
    X
Pipeline Stops
```

Possible causes:

```
Compilation Error
    +
Dependency Problem
    +
Test Failure
    +
Plugin Failure
    +
Configuration Error
```

The failure should be investigated before rerunning the pipeline.

---

# 19. Code Quality With SonarQube

SonarQube can analyze the Java source code.

Flow:

```
Source
    |
    ↓
Maven
    |
    ↓
SonarQube
    |
    ↓
Quality Gate
   / \
Pass  Fail
 |      |
 ↓      X
```

Continue  Stop

---

# 20. SonarQube Quality Gate

The quality gate can evaluate:

```
Bugs
    +
Vulnerabilities
    +
Code Smells
    +
Coverage
    +
Duplications
```

The organization defines which conditions should block the pipeline.

---

# 21. Security Scanning

Security should be integrated into CI.

Example:

```
Build
    |
    +--- SonarQube
    |
    +--- Dependency Scan
    |
    +--- Veracode
    |
    +--- Secret Detection
    |
    ↓
Security Gate
```

---

# 22. Dependency Security

Java applications depend on external libraries.

Example:

```
Application
    |
    +--- Spring
    +--- Database Driver
    +--- Logging Library
    +--- Other Dependencies
```

A vulnerable dependency can introduce application risk.

The pipeline should identify known vulnerable dependencies.

---

# 23. Vulnerable Dependency

If a critical vulnerability is detected:

```
Dependency Scan
    |
    ↓
Critical Vulnerability
    |
    X
Pipeline Blocked
```

Then:

```
Identify Version
    |
    ↓
Find Fixed Version
    |
    ↓
Update pom.xml
    |
    ↓
Test
    |
    ↓
Rescan
```

---

# 24. Veracode Integration

Veracode can be integrated into the security stage.

Flow:

```
Java Build
    |
    ↓
Security Analysis
    |
    ↓
Veracode
    |
    ↓
Security Result
    |
    ↓
Gate
```

The exact configuration depends on the organization's Veracode
implementation.

---

# 25. Secret Detection

The pipeline should detect accidental secrets.

Examples:

```
AWS Keys
    +
API Keys
    +
Passwords
    +
Tokens
```

If a secret is detected:

```
Pipeline
    |
    ↓
Secret Found
    |
    X
Pipeline Stops
```

If a real credential has already been exposed, it should be treated
as compromised and rotated.

---

# 26. Docker Build

After the Java application successfully builds:

```
Maven
    |
    ↓
application.jar
    |
    ↓
Docker Build
    |
    ↓
Docker Image
```

Example image:

```
java-user-service:<commit-sha>
```

---

# 27. Dockerfile

A Java service can use a Dockerfile conceptually like:

```
Base Image
    |
    ↓
Working Directory
    |
    ↓
Copy JAR
    |
    ↓
Configure User
    |
    ↓
Expose Application Port
    |
    ↓
Start Java Application
```

The image should avoid unnecessary packages and should not contain
secrets.

---

# 28. Multi-Stage Docker Build

A multi-stage Docker build can separate build and runtime concerns.

Architecture:

```
Build Stage
    |
    ↓
Maven
    |
    ↓
JAR
    |
    ↓
Runtime Stage
    |
    ↓
Java Application
```

Benefits:

```
Smaller Image
    +
Fewer Build Dependencies
    +
Reduced Attack Surface
```

---

# 29. Run Container as Non-Root

The application container should run as a non-root user whenever
possible.

Architecture:

```
Container
    |
    ↓
Non-Root User
    |
    ↓
Java Application
```

This reduces the impact of a potential container compromise.

---

# 30. Docker Image Tagging

Avoid relying on:

```
latest
```

for production deployments.

Use immutable identifiers such as:

```
Commit SHA
    +
Version
    +
Image Digest
```

Example:

```
user-service:<commit-sha>
```

This makes deployments traceable.

---

# 31. Trivy Container Scan

After building the image:

```
Docker Build
    |
    ↓
Trivy
    |
    ↓
Vulnerability Results
    |
    ↓
Security Gate
```

---

# 32. Trivy Critical Vulnerability

If Trivy finds a critical vulnerability:

```
Image
    |
    ↓
Trivy
    |
    ↓
Critical Finding
    |
    X
Pipeline Stops
```

Then:

```
Identify Vulnerable Package
    |
    ↓
Update Base Image / Dependency
    |
    ↓
Rebuild
    |
    ↓
Rescan
```

---

# 33. Push Image to ECR

After the image passes security validation:

```
Docker Image
    |
    ↓
AWS Authentication
    |
    ↓
ECR Login
    |
    ↓
ECR Repository
    |
    ↓
Push Image
```

---

# 34. AWS Authentication

Do not use long-lived AWS access keys when OIDC can be used.

Preferred architecture:

```
GitHub Actions
    |
    ↓
OIDC
    |
    ↓
AWS IAM
    |
    ↓
Temporary Credentials
    |
    ↓
ECR
```

---

# 35. ECR Repository

Each microservice can have its own repository.

Example:

```
ECR
│
├── user-service
├── product-service
├── cart-service
├── order-service
├── payment-service
└── inventory-service
```

This provides clear image separation.

---

# 36. ECR Image Lifecycle

The registry should have lifecycle management.

Possible policy:

```
Old Images
    |
    ↓
Lifecycle Policy
    |
    ↓
Cleanup
```

However, enough production versions should be retained to support
rollback and incident recovery.

---

# 37. Deployment Strategy

After the image is pushed:

```
ECR
    |
    ↓
Deployment Configuration
    |
    ↓
Kubernetes
    |
    ↓
EKS
```

There are two common approaches.

### Direct CI Deployment

```
GitHub Actions
    |
    ↓
kubectl / Helm
    |
    ↓
EKS
```

### GitOps Deployment

```
GitHub Actions
    |
    ↓
Update GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

For a GitOps-based architecture, the second approach is preferred.

---

# 38. Helm Deployment

Helm manages Kubernetes application configuration.

Example:

```
helm/
│
└── user-service/
    ├── Chart.yaml
    ├── values.yaml
    └── templates/
```

The image reference is supplied through Helm values.

---

# 39. Helm Values

Conceptually:

```
image:
  repository: ECR_REPOSITORY
  tag: COMMIT_SHA
```

This allows the same chart to deploy different versions.

---

# 40. GitOps Flow

The recommended flow is:

```
GitHub Actions
    |
    ↓
Build Image
    |
    ↓
Push ECR
    |
    ↓
Update Image Reference
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 41. ArgoCD Synchronization

ArgoCD continuously compares:

```
Git Desired State
    |
    vs
EKS Actual State
```

If they differ:

```
Drift
    |
    ↓
ArgoCD
    |
    ↓
Reconciliation
```

---

# 42. Production Deployment

Production deployment should have stronger controls.

Example:

```
CI
    |
    ↓
DEV
    |
    ↓
QA
    |
    ↓
Approval
    |
    ↓
PROD
    |
    ↓
EKS
```

---

# 43. Deployment Validation

After deployment, validate:

```
Pod Status
    +
Readiness
    +
Service
    +
ALB
    +
Application Health
    +
Logs
    +
Metrics
```

---

# 44. Kubernetes Health Checks

The Java application should expose a health endpoint appropriate
for the application's configuration.

Kubernetes can use:

```
Readiness Probe
    +
Liveness Probe
```

Readiness:

```
"Can this pod receive traffic?"
```

Liveness:

```
"Is this container still healthy enough to continue running?"
```

---

# 45. Readiness Failure

If readiness fails:

```
Pod
    |
    ↓
Not Ready
    |
    ↓
Traffic Removed
```

This can prevent unhealthy pods from receiving application traffic.

---

# 46. Liveness Failure

If the liveness check repeatedly fails:

```
Liveness Failure
    |
    ↓
Kubernetes
    |
    ↓
Container Restart
```

The exact behavior depends on the Kubernetes configuration.

---

# 47. Zero-Downtime Deployment

For a Java microservice:

```
Existing Pods
    |
    ↓
New Pods
    |
    ↓
Readiness Validation
    |
    ↓
Traffic Shift
    |
    ↓
Old Pods Terminated
```

Use:

```
Multiple Replicas
    +
Readiness Probes
    +
Rolling Deployment
```

---

# 48. Rollback Strategy

If the deployment causes problems:

```
New Version
    |
    ↓
Validation
    |
   / \
Pass  Fail
 |      |
 ↓      ↓
```

Continue Rollback
|
↓
Previous Version
|
↓
EKS
|
↓
Validation

---

# 49. Rollback With Immutable Images

Suppose:

```
Version A → Healthy
Version B → New Release
Version C → Current Release
```

If Version C fails:

```
Version C
    |
    X
    |
    ↓
Version B
    |
    ↓
Validate
```

Because each image is immutable, the rollback points to a known-good
artifact.

---

# 50. Failed Deployment Scenario

Question:

```
The Java application deployed successfully, but users receive
HTTP 503 errors. What do you check?
```

Answer:

I would check:

```
1. Pod Status
2. Readiness Probe
3. Service
4. Service Endpoints
5. ALB Target Health
6. Ingress Configuration
7. Container Port
8. Target Port
9. Application Logs
10. Recent Deployment Changes
```

---

# 51. CrashLoopBackOff Scenario

If a Java pod enters:

```
CrashLoopBackOff
```

Check:

```
kubectl describe pod
    |
    ↓
Events
```

Then:

```
kubectl logs <pod>
    |
    ↓
kubectl logs <pod> --previous
```

Investigate:

```
Application Crash
    +
Configuration
    +
Environment Variables
    +
Secrets
    +
Database Connectivity
    +
Memory
    +
Startup Failure
```

---

# 52. OOMKilled Scenario

If the Java application is:

```
OOMKilled
```

Investigate:

```
Memory Requests
    +
Memory Limits
    +
JVM Configuration
    +
Application Memory Usage
    +
Heap Configuration
```

Then determine whether to:

```
Optimize Application
    OR
Adjust JVM
    OR
Adjust Resource Configuration
```

---

# 53. Application Startup Failure

A Java application starts locally but fails in Kubernetes.

Check:

```
Environment Variables
    +
Secrets
    +
Configuration
    +
Database Connectivity
    +
Service Discovery
    +
Port Configuration
    +
JVM Settings
```

---

# 54. Database Connectivity Failure

If the Java service cannot connect to the database:

```
Application
    |
    ↓
Database Connection
    |
    X
Failure
```

Check:

```
DNS
    +
Network
    +
Security Groups
    +
Credentials
    +
Database Availability
    +
Connection Configuration
```

---

# 55. CI Failure Troubleshooting

If GitHub Actions fails during Maven build:

```
Workflow
    |
    ↓
Failed Step
    |
    ↓
Logs
    |
    ↓
Identify Error
    |
    ↓
Reproduce
    |
    ↓
Fix
    |
    ↓
Re-run
```

Do not simply rerun repeatedly without understanding the failure.

---

# 56. Maven Dependency Failure

Possible causes:

```
Repository Unavailable
    +
Incorrect Version
    +
Authentication
    +
Network
    +
Corrupt Cache
```

Approach:

```
Check Error
    |
    ↓
Validate Dependency
    |
    ↓
Check Repository
    |
    ↓
Validate Cache
    |
    ↓
Re-run
```

---

# 57. SonarQube Failure

If SonarQube fails:

```
Check Analysis Logs
    |
    ↓
Check Server Connectivity
    |
    ↓
Check Project Configuration
    |
    ↓
Review Findings
    |
    ↓
Validate Quality Gate
```

---

# 58. Docker Build Failure

If Docker build fails:

```
Check Dockerfile
    +
Build Context
    +
JAR Location
    +
Base Image
    +
Docker Build Logs
```

Typical flow:

```
Maven Build
    |
    ↓
Verify JAR
    |
    ↓
Docker Build
    |
    ↓
Image
```

---

# 59. ECR Push Failure

If ECR push fails:

```
Check AWS Authentication
    |
    ↓
Check OIDC
    |
    ↓
Check IAM Permissions
    |
    ↓
Check ECR Repository
    |
    ↓
Check Registry Login
    |
    ↓
Retry After Root Cause Is Fixed
```

---

# 60. ArgoCD Deployment Failure

If ArgoCD does not deploy the new image:

```
Check GitOps Commit
    |
    ↓
Check Repository
    |
    ↓
Check ArgoCD Application
    |
    ↓
Check Sync Status
    |
    ↓
Check Health
    |
    ↓
Check Kubernetes Resources
```

---

# 61. GitOps Drift

If someone manually changes the Kubernetes deployment:

```
Git
    |
    ↓
Desired State

EKS
    |
    ↓
Actual State
```

If different:

```
Drift
```

ArgoCD can detect and reconcile the difference according to its
configured policy.

---

# 62. Pipeline Security

The Java CI/CD pipeline should implement:

```
Branch Protection
    +
Pull Request Reviews
    +
CODEOWNERS
    +
OIDC
    +
Least Privilege
    +
Secret Management
    +
Security Scanning
    +
Protected Environments
```

---

# 63. Branch Protection

Production code should not be directly modified without the required
review process.

Flow:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
Required Checks
    |
    ↓
Review
    |
    ↓
Merge
```

---

# 64. CODEOWNERS

Sensitive files should have appropriate owners.

Examples:

```
.github/workflows/
terraform/
helm/
production/
```

Changes to these areas can require additional review.

---

# 65. GitHub Token Permissions

The workflow should request only required permissions.

Example principle:

```
Read
    |
    ↓
Only What Is Required
```

Avoid unnecessary write permissions.

---

# 66. Production Credentials

Production credentials should not be hardcoded in:

```
Source Code
    +
Dockerfile
    +
Helm Values
    +
GitHub Workflow
```

Use secure credential mechanisms instead.

---

# 67. OIDC Production Role

The production workflow can assume a restricted AWS role.

Architecture:

```
GitHub Actions
    |
    ↓
OIDC
    |
    ↓
Production IAM Role
    |
    ↓
AWS
```

The trust policy should restrict which repository and workflow
context can assume the role.

---

# 68. Environment Separation

Use separate environments:

```
DEV
    +
QA
    +
PROD
```

Each can have:

```
Different Permissions
    +
Different Configuration
    +
Different Deployment Controls
```

---

# 69. Production Approval

Production should require authorization.

Flow:

```
CI
    |
    ↓
QA
    |
    ↓
Production Environment
    |
    ↓
Approval
    |
    ↓
Deployment
```

---

# 70. Artifact Traceability

Every production deployment should be traceable.

Example:

```
Git Commit
    |
    ↓
GitHub Actions Run
    |
    ↓
Docker Image
    |
    ↓
ECR
    |
    ↓
GitOps Commit
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

This answers:

```
"Which source commit is running in production?"
```

---

# 71. Build Once, Deploy Many

A strong CI/CD principle is:

```
Build Once
    |
    ↓
Immutable Artifact
    |
    +--- DEV
    +--- QA
    +--- PROD
```

Do not rebuild the application separately for every environment.

---

# 72. Promotion Strategy

Example:

```
Build
    |
    ↓
Security
    |
    ↓
ECR
    |
    ↓
DEV
    |
    ↓
Validation
    |
    ↓
QA
    |
    ↓
Validation
    |
    ↓
PROD Approval
    |
    ↓
PROD
```

---

# 73. Deployment Strategies

For Java microservices, possible strategies include:

```
Rolling
    +
Canary
    +
Blue-Green
```

The appropriate strategy depends on:

```
Risk
    +
Traffic
    +
Architecture
    +
Business Requirements
```

---

# 74. Rolling Deployment

Architecture:

```
Old Pods
    |
    ↓
New Pod
    |
    ↓
Readiness
    |
    ↓
Old Pod Removed
    |
    ↓
Next New Pod
```

This provides gradual replacement.

---

# 75. Canary Deployment

Architecture:

```
Production Traffic
      |
      +---------+
      |         |
      ↓         ↓
  Old Version  New Version
                |
                ↓
            Small Traffic
                |
                ↓
             Validate
                |
                ↓
         Increase Traffic
```

---

# 76. Blue-Green Deployment

Architecture:

```
Production Traffic
      |
      ↓
   Blue
      |
      ↓
Switch Traffic
      |
      ↓
   Green
```

Rollback:

```
Green
  X
  |
  ↓
Blue
```

---

# 77. Observability

After deployment, monitor:

```
Application Metrics
    +
Kubernetes Metrics
    +
Logs
    +
Availability
    +
Error Rate
    +
Latency
```

Stack:

```
Prometheus
    +
Grafana
    +
ELK
```

---

# 78. Prometheus

Prometheus collects metrics.

Examples:

```
Request Rate
    +
Error Rate
    +
CPU
    +
Memory
    +
Application Metrics
```

---

# 79. Grafana

Grafana visualizes metrics.

Useful dashboards:

```
Application Health
    +
Kubernetes Health
    +
CPU
    +
Memory
    +
Request Rate
    +
Latency
    +
Error Rate
```

---

# 80. ELK Stack

ELK provides centralized logging.

Flow:

```
Java Application
    |
    ↓
Logs
    |
    ↓
ELK
    |
    ↓
Search / Analysis
```

Useful during:

```
Deployment Troubleshooting
    +
Incident Investigation
    +
Application Debugging
```

---

# 81. Deployment Verification

A deployment should not be considered successful merely because
Kubernetes accepted the manifest.

Validate:

```
Deployment Status
    +
Pod Readiness
    +
Application Health
    +
ALB Health
    +
Smoke Tests
```

---

# 82. Smoke Testing

After deployment:

```
Deploy
    |
    ↓
Health Check
    |
    ↓
API Smoke Test
    |
    ↓
Result
   / \
Pass  Fail
 |      |
 ↓      ↓
```

Success Rollback

---

# 83. Post-Deployment Validation

Validate:

```
HTTP Status
    +
Application Response
    +
Database Connectivity
    +
Dependency Connectivity
    +
Pod Health
    +
ALB Health
```

---

# 84. Failed Smoke Test

If smoke testing fails:

```
Deployment
    |
    ↓
Smoke Test
    |
    ↓
Failure
    |
    ↓
Stop Promotion
    |
    ↓
Investigate
    |
    ↓
Rollback / Fix
```

---

# 85. Production Incident

Scenario:

```
The new Java version was deployed successfully.
Five minutes later, API error rates increased.
```

Response:

```
Detect
    |
    ↓
Prometheus / Grafana
    |
    ↓
Correlate Deployment
    |
    ↓
ELK Logs
    |
    ↓
Identify Root Cause
    |
    ↓
Rollback / Fix
    |
    ↓
Validate
    |
    ↓
Monitor
```

---

# 86. Production Rollback

If rollback is required:

```
Current Version
    |
    ↓
Identify Known-Good Version
    |
    ↓
Update GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Validation
```

---

# 87. Rollback Decision

Rollback when:

```
New Release Causes Severe Regression
    +
Previous Version Is Known Good
    +
Rollback Is Safe
```

Fix forward when:

```
Rollback Is Unsafe
    +
Data Migration Prevents Rollback
    +
Fix Is Small And Fast
```

---

# 88. Database Migration Consideration

Microservice deployments may include database schema changes.

Before deployment:

```
Application
    |
    ↓
Database Migration
    |
    ↓
Compatibility
    |
    ↓
New Version
```

Migrations should be designed carefully so rollback remains
possible where required.

---

# 89. Backward-Compatible Database Changes

A safer migration pattern is:

```
Add New Structure
    |
    ↓
Deploy Compatible Application
    |
    ↓
Migrate Data
    |
    ↓
Remove Old Structure Later
```

This reduces deployment coupling.

---

# 90. Pipeline Failure Recovery

If the CI pipeline fails:

```
Identify Failed Job
    |
    ↓
Inspect Logs
    |
    ↓
Determine Root Cause
    |
    ↓
Fix
    |
    ↓
Re-run
```

If the failure is infrastructure-related, verify the runner and
external dependencies.

---

# 91. Self-Hosted Runner Scenario

If a self-hosted runner is required:

```
GitHub
    |
    ↓
Self-Hosted Runner
    |
    ↓
CI
    |
    ↓
Destroy / Clean
```

The runner should have:

```
Minimal Permissions
    +
Network Restrictions
    +
Clean Environment
    +
Controlled Access
```

Ephemeral runners are preferable where practical.

---

# 92. Pipeline Performance

A Java pipeline can become slow because of:

```
Maven Dependencies
    +
Unit Tests
    +
SonarQube
    +
Security Scans
    +
Docker Build
```

Optimization:

```
Maven Cache
    +
Parallel Jobs
    +
Docker Cache
    +
Test Parallelization
    +
Reusable Workflows
```

---

# 93. Parallel Security Checks

Independent checks can run in parallel.

Example:

```
Build
    |
    +--- SonarQube
    +--- Dependency Scan
    +--- Secret Scan
    |
    ↓
Security Gate
    |
    ↓
Docker Build
```

This reduces pipeline duration.

---

# 94. Matrix Strategy

If multiple microservices use the same CI logic:

```
Matrix
    |
    +--- user-service
    +--- product-service
    +--- order-service
    +--- payment-service
    +--- inventory-service
```

The same workflow logic can be reused for multiple services.

---

# 95. Service-Specific Builds

A microservice pipeline can identify which service changed.

Example:

```
Commit
    |
    ↓
Change Detection
    |
    +--- user-service → Build
    |
    +--- product-service → Skip
    |
    +--- order-service → Build
```

This is useful for larger repositories.

---

# 96. Monorepo vs Multi-Repo

## Monorepo

All services are stored in one repository.

Advantages:

```
Centralized Changes
    +
Shared Configuration
```

Challenges:

```
Larger Pipeline
    +
Change Detection Required
```

## Multi-Repo

Each service has its own repository.

Advantages:

```
Independent CI/CD
    +
Clear Ownership
```

Challenges:

```
More Repositories
    +
Workflow Standardization Required
```

---

# 97. Reusable Workflow

Common CI logic can be centralized.

Architecture:

```
Service A
    |
Service B
    |
Service C
    |
    ↓
Reusable Workflow
    |
    ↓
Standard CI
```

This reduces duplicated pipeline logic.

---

# 98. Java Microservices Standard CI

A standard CI flow can be:

```
Checkout
    |
    ↓
Java Setup
    |
    ↓
Maven Cache
    |
    ↓
Maven Test
    |
    ↓
Maven Package
    |
    ↓
SonarQube
    |
    ↓
Security Scan
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
ECR
```

---

# 99. Java Microservices Standard CD

A standard GitOps CD flow:

```
ECR
    |
    ↓
Image Tag
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Checks
    |
    ↓
Monitoring
```

---

# 100. Complete Project Flow

The complete project can be summarized as:

```
Developer
    |
    ↓
GitHub
    |
    ↓
Pull Request
    |
    ↓
Branch Protection
    |
    ↓
GitHub Actions
    |
    ↓
Checkout
    |
    ↓
Java Setup
    |
    ↓
Maven Cache
    |
    ↓
Unit Tests
    |
    ↓
Maven Package
    |
    ↓
SonarQube
    |
    ↓
Dependency Security
    |
    ↓
Veracode
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
ECR
    |
    ↓
Immutable Image
    |
    ↓
GitOps Repository
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Readiness Validation
    |
    ↓
Smoke Test
    |
    ↓
Prometheus
    +
Grafana
    +
ELK
    |
    ↓
Production
```

---

# 101. End-to-End Failure Handling

At every stage:

```
Failure
    |
    ↓
Stop Promotion
    |
    ↓
Investigate
    |
    ↓
Fix
    |
    ↓
Validate
    |
    ↓
Continue
```

Examples:

```
Maven Failure
    → Fix Code

SonarQube Failure
    → Fix Quality/Security Issue

Trivy Failure
    → Fix Vulnerability

ECR Failure
    → Fix Authentication/Registry Issue

ArgoCD Failure
    → Fix GitOps/Kubernetes Configuration

Health Check Failure
    → Investigate Deployment

Smoke Test Failure
    → Rollback / Fix
```

---

# 102. Production Security Architecture

The production path should use:

```
Developer
    |
    ↓
Pull Request
    |
    ↓
Review
    |
    ↓
CI Security
    |
    ↓
Immutable Artifact
    |
    ↓
Protected Environment
    |
    ↓
OIDC
    |
    ↓
Least Privilege
    |
    ↓
GitOps
    |
    ↓
EKS
```

---

# 103. Separation of Duties

The pipeline should avoid giving one identity complete control.

Example:

```
Developer
    |
    ↓
Code

Reviewer
    |
    ↓
Merge

CI
    |
    ↓
Build

Security
    |
    ↓
Security Validation

Release Approval
    |
    ↓
Production
```

This reduces the blast radius of compromised credentials or
unauthorized changes.

---

# 104. Auditability

Every production release should provide a traceable chain:

```
Developer
    |
    ↓
Commit
    |
    ↓
Pull Request
    |
    ↓
Review
    |
    ↓
GitHub Actions Run
    |
    ↓
Artifact
    |
    ↓
ECR
    |
    ↓
GitOps Commit
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

---

# 105. Real-World Interview Scenario

Question:

```
Explain a Java microservices GitHub Actions project you worked
on from source code to production.
```

### Strong Answer

I would explain the project in stages.

The application consisted of multiple Java-based microservices.

Developers pushed code to GitHub and created pull requests.

GitHub Actions handled the CI process.

The workflow first checked out the source code and configured the
required Java version. Maven dependencies were cached to reduce build
time.

Then the pipeline ran unit tests and built the Java application using
Maven.

After the build, I integrated code-quality and security validation
using tools such as SonarQube, dependency scanning, and Veracode.

Once the application passed the required quality and security gates,
I built the Docker image.

The image was scanned using Trivy to identify container
vulnerabilities.

After the image passed the security policy, GitHub Actions
authenticated to AWS using OIDC and pushed the immutable image to
Amazon ECR.

For Kubernetes deployment, the image reference was promoted through
the GitOps repository. ArgoCD detected the Git change and reconciled
the desired state into the EKS cluster.

The application was deployed using Kubernetes and Helm.

After deployment, I validated:

```
Pod Health
    +
Readiness
    +
Application Health
    +
ALB Health
    +
Smoke Tests
```

For observability, I used:

```
Prometheus
    +
Grafana
    +
ELK
```

If a release introduced a production issue, I would identify the
known-good image and use the GitOps rollback process to restore the
previous version.

The overall objective was to provide:

```
Automated CI/CD
    +
Security
    +
Immutable Artifacts
    +
GitOps
    +
Controlled Production Deployment
    +
Observability
    +
Reliable Rollback
```

---

# 106. Real-World Troubleshooting Scenario

Question:

```
A new Java microservice version was deployed through GitHub
Actions and ArgoCD. ArgoCD reports the application as healthy,
but users are receiving 503 errors.
```

### Answer

I would not assume the deployment itself is successful just because
ArgoCD reports synchronization.

I would investigate the complete traffic path:

```
User
    |
    ↓
ALB
    |
    ↓
Kubernetes Service
    |
    ↓
Pod
    |
    ↓
Java Application
```

First I would check pod status and readiness.

Then I would check:

```
Service
    +
Endpoints
    +
Target Port
    +
Container Port
    +
ALB Target Health
    +
Application Logs
```

I would use ELK to investigate application errors and Prometheus /
Grafana to check error rate, latency, CPU, and memory.

Once I identify the root cause, I would either fix forward or rollback
depending on severity and availability requirements.

---

# 107. Real-World Security Scenario

Question:

```
Trivy reports a critical vulnerability in the Java service image
immediately before production deployment.
```

### Answer

I would stop promotion if the production security policy blocks
critical vulnerabilities.

Then I would identify:

```
Vulnerable Package
    +
Installed Version
    +
Fixed Version
    +
Base Image
```

I would update the dependency or base image, rebuild the Docker image,
rerun Trivy, and only continue after the security gate passes.

If no fix exists, I would follow the organization's documented
security exception process rather than silently bypassing the scan.

---

# 108. Real-World Rollback Scenario

Question:

```
The deployment passed CI and QA, but production error rates
increased immediately after deployment.
```

### Answer

I would correlate the incident with the release using:

```
Prometheus
    +
Grafana
    +
ELK
    +
Deployment History
```

If the release is confirmed as the cause and the previous version is
known good:

```
Stop Promotion
    |
    ↓
Rollback
    |
    ↓
Known-Good Image
    |
    ↓
GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Validation
```

After recovery, I would perform root-cause analysis and add
preventive validation.

---

# 109. Project Best Practices

## Source Control

```
Use Pull Requests
    +
Protect Main Branch
    +
Require Reviews
    +
Use CODEOWNERS
```

## CI

```
Automated Tests
    +
Maven Build
    +
Code Quality
    +
Security Scanning
```

## Containers

```
Trusted Base Images
    +
Non-Root User
    +
Small Images
    +
Vulnerability Scanning
```

## AWS

```
OIDC
    +
IAM Least Privilege
    +
Private ECR
    +
Image Lifecycle Policies
```

## Kubernetes

```
RBAC
    +
Resource Limits
    +
Health Checks
    +
Multiple Replicas
```

## CD

```
GitOps
    +
ArgoCD
    +
Protected Production
    +
Immutable Artifacts
```

## Operations

```
Prometheus
    +
Grafana
    +
ELK
    +
Rollback
    +
Incident Response
```

---

# 110. Common Mistakes

## Mistake 1

Using `latest` for production images.

### Better

Use immutable image tags or digests.

---

## Mistake 2

Storing AWS access keys in GitHub Secrets when OIDC is available.

### Better

Use GitHub OIDC with IAM roles.

---

## Mistake 3

Building separately for DEV, QA, and PROD.

### Better

Build once and promote the same immutable artifact.

---

## Mistake 4

Skipping security checks to make the pipeline faster.

### Better

Optimize the checks using:

```
Caching
    +
Parallelization
    +
Efficient Scanning
```

---

## Mistake 5

Giving CI cluster-admin permissions.

### Better

Use least-privilege access.

---

## Mistake 6

Deploying without health validation.

### Better

Use:

```
Readiness
    +
Liveness
    +
Smoke Tests
```

---

## Mistake 7

No rollback strategy.

### Better

Maintain known-good immutable artifacts.

---

## Mistake 8

Treating ArgoCD synchronization as application health.

### Better

Validate:

```
Kubernetes
    +
Application
    +
Traffic
    +
Metrics
```

---

# 111. Project Summary

This Java microservices project demonstrates a complete CI/CD
implementation using GitHub Actions.

The final pipeline is:

```
GitHub
    |
    ↓
Pull Request
    |
    ↓
GitHub Actions
    |
    +--- Java Setup
    +--- Maven
    +--- Unit Tests
    +--- SonarQube
    +--- Security
    |
    ↓
Docker
    |
    ↓
Trivy
    |
    ↓
ECR
    |
    ↓
GitOps
    |
    ↓
ArgoCD
    |
    ↓
EKS
    |
    ↓
Health Checks
    |
    ↓
Smoke Tests
    |
    ↓
Prometheus
    +
Grafana
    +
ELK
```

Security is applied throughout:

```
Pull Request
    +
Code
    +
Dependencies
    +
Container
    +
AWS
    +
Kubernetes
    +
Production
```

---

# 112. Key Interview Takeaways

When explaining this project in an interview, remember these points:

```
1. GitHub Actions handled CI automation.

2. Maven handled Java builds and dependency management.

3. Unit tests ran before artifact promotion.

4. SonarQube handled code-quality and security analysis.

5. Dependency security checks identified vulnerable libraries.

6. Veracode was integrated into the security stage.

7. Docker packaged the Java application.

8. Trivy scanned container images.

9. GitHub Actions used OIDC for AWS authentication.

10. Amazon ECR stored immutable container images.

11. Helm packaged Kubernetes deployments.

12. ArgoCD implemented GitOps deployment.

13. EKS hosted the microservices.

14. Readiness and liveness checks protected application
    availability.

15. Prometheus and Grafana provided metrics and dashboards.

16. ELK provided centralized logging.

17. Production used controlled deployment and approval.

18. Rollback used a known-good immutable artifact.

19. Branch protection and CODEOWNERS protected critical changes.

20. Least privilege and separation of duties reduced security risk.
```

---

# 113. Final Architecture

```
┌───────────────────────┐
│      Developer        │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│        GitHub         │
│   Pull Request / Git  │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│   GitHub Actions      │
│                       │
│  Build                │
│  Unit Tests           │
│  SonarQube            │
│  Security             │
│  Veracode             │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│        Docker         │
│     Image Build       │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│        Trivy          │
│   Image Security      │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│      Amazon ECR       │
│  Immutable Container  │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│    GitOps Repository  │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│        ArgoCD         │
│   GitOps Controller   │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│        Amazon EKS     │
│                       │
│ Java Microservices    │
│ Kubernetes            │
│ Helm                  │
└───────────┬───────────┘
            │
            ↓
┌───────────────────────┐
│   ALB / Application   │
└───────────┬───────────┘
            │
            ↓
          Users

Monitoring:

┌───────────────┐
│  Prometheus   │
└───────┬───────┘
        │
        ↓
┌───────────────┐
│    Grafana    │
└───────────────┘

┌───────────────┐
│      ELK      │
│ Central Logs  │
└───────────────┘
```

---

# 114. Final Project Principle

The most important principle of this project is:

```
Build Once
    |
    ↓
Test
    |
    ↓
Secure
    |
    ↓
Package
    |
    ↓
Scan
    |
    ↓
Store
    |
    ↓
Promote
    |
    ↓
Deploy
    |
    ↓
Validate
    |
    ↓
Monitor
    |
    ↓
Rollback If Required
```

The goal is not simply to automate deployment.

The goal is to create a pipeline that is:

```
Automated
    +
Secure
    +
Repeatable
    +
Observable
    +
Auditable
    +
Reliable
    +
Recoverable
```

# END OF FILE - 01-Java-Microservices-CI-CD.md
