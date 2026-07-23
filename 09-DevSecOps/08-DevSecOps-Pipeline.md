# DevSecOps Pipeline

## Introduction

A DevSecOps Pipeline is a CI/CD pipeline that integrates automated security checks into every stage of software delivery. Instead of performing security only before production, security validation occurs continuously from code commit to runtime monitoring.

This approach enables organizations to deliver secure software rapidly without compromising quality or compliance.

---

## Why Do We Need a DevSecOps Pipeline?

Traditional CI/CD pipelines primarily focus on building, testing, and deploying applications.

Without security integration:

- Secrets may be committed to source code.
- Vulnerable dependencies can reach production.
- Misconfigured infrastructure may be deployed.
- Vulnerable container images can be released.
- Security issues are discovered too late.

A DevSecOps pipeline ensures security is automatically verified before every deployment.

---

## What is a DevSecOps Pipeline?

A DevSecOps Pipeline combines Continuous Integration, Continuous Delivery, and Continuous Security.

Instead of:

```text
Code

↓

Build

↓

Test

↓

Deploy

↓

Security Review
```

We follow:

```text
Code

↓

Security Scan

↓

Build

↓

Security Validation

↓

Deploy

↓

Runtime Monitoring
```

Security becomes an integral part of software delivery.

---

# Traditional CI/CD vs DevSecOps Pipeline

| Traditional CI/CD | DevSecOps Pipeline |
|-------------------|--------------------|
| Focus on delivery | Focus on secure delivery |
| Security at the end | Security throughout pipeline |
| Manual security reviews | Automated security validation |
| Late vulnerability detection | Early vulnerability detection |
| Reactive approach | Proactive approach |
| Higher remediation cost | Lower remediation cost |

---

# DevSecOps Pipeline Stages

## 1. Source Code

Developers write code and push it to the Git repository.

Example Tools

- GitHub
- GitLab
- Bitbucket

---

## 2. Secrets Scanning

Detect hardcoded secrets before the code enters the pipeline.

Purpose

- AWS Keys
- Azure Keys
- Passwords
- API Tokens
- SSH Keys

Example Tool

- Gitleaks

---

## 3. Build

Compile the application and package the source code.

Example Tools

- Maven
- Gradle
- npm

---

## 4. Static Application Security Testing (SAST)

Analyze source code without executing the application.

Detects

- SQL Injection
- XSS
- Hardcoded Credentials
- Code Smells
- Security Bugs

Example Tools

- SonarQube
- Veracode

---

## 5. Software Composition Analysis (SCA)

Analyze third-party libraries and dependencies.

Detects

- Vulnerable packages
- Known CVEs
- License issues

Example Tools

- OWASP Dependency-Check
- Snyk

---

## 6. Unit Testing

Validate application functionality.

Purpose

- Verify code correctness
- Prevent regressions

---

## 7. Container Build

Package the application into a Docker image.

Example Tool

- Docker

---

## 8. Container Image Scanning

Scan the container image for vulnerabilities.

Detects

- OS vulnerabilities
- Package vulnerabilities
- Misconfigurations

Example Tool

- Trivy

---

## 9. Infrastructure as Code (IaC) Scanning

Validate Terraform or Kubernetes manifests before deployment.

Detects

- Public S3 Buckets
- Open Security Groups
- Weak IAM Policies
- Kubernetes Misconfigurations

Example Tools

- Checkov
- tfsec

---

## 10. SBOM Generation

Generate a Software Bill of Materials (SBOM).

Benefits

- Dependency visibility
- Supply chain transparency
- Compliance support

---

## 11. Image Signing

Digitally sign container images before publishing.

Benefits

- Image authenticity
- Supply chain security
- Tamper detection

Example Tool

- Cosign

---

## 12. Artifact Repository

Store verified software artifacts.

Examples

- AWS ECR
- Azure ACR
- JFrog Artifactory

---

## 13. Deploy to Staging

Deploy the application to a staging environment for validation.

---

## 14. Dynamic Application Security Testing (DAST)

Analyze the running application.

Detects

- SQL Injection
- XSS
- SSRF
- Authentication Issues
- Security Headers

Example Tool

- OWASP ZAP

---

## 15. Policy Validation

Validate Kubernetes resources before deployment.

Checks

- Resource Limits
- Privileged Containers
- Security Context
- Image Policies

Example Tools

- OPA Gatekeeper
- Kyverno

---

## 16. Deploy to Production

Deploy only after all security gates pass.

---

## 17. Runtime Security

Continuously monitor workloads after deployment.

Detects

- Shell execution
- Privilege escalation
- Suspicious processes
- File modifications

Example Tool

- Falco

---

## 18. Monitoring & Feedback

Collect metrics, logs, and alerts.

Example Tools

- Prometheus
- Grafana
- ELK Stack

---

# Complete DevSecOps Pipeline Architecture

```text
Developer
     │
     ▼
Git Commit
     │
     ▼
Gitleaks
(Secrets Scan)
     │
     ▼
Build
     │
     ▼
SonarQube / Veracode
(SAST)
     │
     ▼
SCA Scan
(Dependency Check)
     │
     ▼
Unit Tests
     │
     ▼
Docker Build
     │
     ▼
Trivy
(Container Scan)
     │
     ▼
Checkov / tfsec
(IaC Scan)
     │
     ▼
SBOM Generation
     │
     ▼
Cosign
(Image Signing)
     │
     ▼
Artifact Registry
(ECR / ACR / Artifactory)
     │
     ▼
Deploy to Staging
     │
     ▼
OWASP ZAP
(DAST)
     │
     ▼
OPA / Kyverno
(Policy Validation)
     │
     ▼
Deploy to Production
     │
     ▼
Falco
(Runtime Security)
     │
     ▼
Prometheus + Grafana + ELK
Monitoring
```

---

## Pipeline Integration

```text
Developer

↓

Git Push

↓

Gitleaks

↓

Build

↓

SonarQube

↓

Dependency Scan

↓

Unit Tests

↓

Docker Build

↓

Trivy

↓

Checkov

↓

Cosign

↓

ECR

↓

Deploy to Staging

↓

OWASP ZAP

↓

OPA / Kyverno

↓

Deploy to Production

↓

Falco

↓

Prometheus

↓

Grafana

↓

ELK
```

Every stage acts as a security gate before software progresses to the next stage.

---

## Production Workflow

```text
Developer

↓

GitHub

↓

Jenkins

↓

Secrets Scan

↓

SAST

↓

SCA

↓

Docker Build

↓

Container Scan

↓

IaC Scan

↓

Image Signing

↓

ECR

↓

Deploy to EKS

↓

DAST

↓

Policy Validation

↓

Production

↓

Runtime Monitoring

↓

Alerting
```

---

## Production Example

A developer accidentally uses an outdated Log4j dependency.

```text
Developer

↓

Git Push

↓

Dependency Scan

↓

Critical CVE Detected

↓

Pipeline Failed

↓

Update Dependency

↓

Pipeline Re-run

↓

Deploy Successfully
```

The vulnerable dependency is removed before production.

---

## Best Practices

- Scan every Git commit
- Enforce Quality Gates
- Block deployments on HIGH and CRITICAL vulnerabilities
- Scan container images
- Scan Infrastructure as Code
- Generate SBOMs
- Sign container images
- Use policy validation
- Continuously monitor runtime workloads
- Keep security tools updated

---

## Common Mistakes

- Skipping secrets scanning
- Ignoring dependency vulnerabilities
- Deploying unsigned images
- Ignoring failed security gates
- Running outdated scanners
- Hardcoding credentials
- Missing runtime monitoring
- Ignoring compliance requirements

---

# Common Troubleshooting

## Issue 1

### Pipeline Stops During SAST

**Cause**

Critical security vulnerabilities exceeded the Quality Gate threshold.

**Resolution**

```text
Review SonarQube Report

↓

Fix Vulnerabilities

↓

Re-run Pipeline

↓

Continue Deployment
```

---

## Issue 2

### Dependency Scan Fails

**Cause**

Outdated third-party libraries contain HIGH or CRITICAL CVEs.

**Resolution**

```text
Update Dependencies

↓

Rebuild Application

↓

Re-run SCA

↓

Deploy
```

---

## Issue 3

### Trivy Reports Critical Image Vulnerabilities

**Cause**

Container image uses an outdated operating system or vulnerable packages.

**Resolution**

```text
Update Base Image

↓

Rebuild Image

↓

Re-run Trivy

↓

Push Secure Image
```

---

## Issue 4

### Deployment Blocked by OPA or Kyverno

**Cause**

Kubernetes manifests violate security policies.

**Resolution**

```text
Review Policy

↓

Update Manifest

↓

Validate Again

↓

Deploy
```

---

## Issue 5

### Falco Generates Runtime Alerts

**Cause**

Unexpected shell execution or suspicious container behavior.

**Resolution**

```text
Review Falco Alert

↓

Inspect Container Logs

↓

Contain Threat

↓

Perform Root Cause Analysis
```

---

# Production Interview Questions

## Question 1

### What is a DevSecOps Pipeline?

A DevSecOps Pipeline is a CI/CD pipeline that integrates automated security checks into every stage of software delivery, ensuring applications are secure before deployment.

---

## Question 2

### Why is a DevSecOps Pipeline important?

It identifies vulnerabilities early, automates security validation, reduces remediation costs, and prevents insecure software from reaching production.

---

## Question 3

### Which security scan should run immediately after a Git commit?

Secrets scanning using tools like Gitleaks should run immediately after a commit to detect exposed credentials before the build starts.

---

## Question 4

### What is the difference between SAST and DAST?

SAST analyzes source code without executing the application, while DAST tests a running application to identify runtime vulnerabilities.

---

## Question 5

### Why is Software Composition Analysis (SCA) required?

SCA identifies vulnerable third-party libraries, known CVEs, and license issues before software is deployed.

---

## Question 6

### Why should container images be scanned?

Container image scanning identifies operating system packages, application dependencies, and configuration vulnerabilities before images are published.

---

## Question 7

### Why is image signing important?

Image signing verifies the authenticity and integrity of container images, ensuring that only trusted artifacts are deployed.

---

## Question 8

### What is the purpose of policy validation in Kubernetes?

Policy validation ensures Kubernetes manifests comply with organizational security standards before deployment by enforcing rules such as non-root containers, required labels, and resource limits.

---

## Question 9

### What role does Falco play in a DevSecOps Pipeline?

Falco provides runtime security by monitoring running containers and Kubernetes workloads for suspicious activities such as privilege escalation, unauthorized shell access, and unexpected file modifications.

---

## Question 10

### Describe a complete production DevSecOps Pipeline.

A production DevSecOps Pipeline typically includes source code management, secrets scanning, SAST, SCA, unit testing, container image building, container image scanning, Infrastructure as Code scanning, SBOM generation, image signing, artifact storage, staging deployment, DAST, policy validation, production deployment, runtime security monitoring, and continuous observability using monitoring and logging tools.

---

# Key Takeaways

- A DevSecOps Pipeline integrates security into every stage of CI/CD.
- Security gates prevent vulnerable code, dependencies, infrastructure, and container images from reaching production.
- Automation reduces manual effort while improving security and compliance.
- Runtime monitoring complements pre-deployment security by detecting threats after deployment.
- A production-grade DevSecOps Pipeline combines preventive, detective, and monitoring controls to deliver secure and reliable software.