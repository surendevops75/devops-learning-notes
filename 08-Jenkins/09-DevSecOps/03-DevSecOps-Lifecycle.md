# DevSecOps Lifecycle

## Introduction

The DevSecOps Lifecycle integrates security into every phase of the Software Development Life Cycle (SDLC). Instead of performing security checks only before production, security becomes a continuous process from planning to monitoring.

Each phase includes automated security controls to detect vulnerabilities as early as possible.

---

## Why Do We Need the DevSecOps Lifecycle?

Modern applications are released multiple times a day.

Without a structured security lifecycle:

- Vulnerabilities reach production
- Secrets get exposed
- Insecure dependencies are deployed
- Infrastructure becomes misconfigured
- Compliance requirements are violated

A DevSecOps lifecycle ensures security is continuously enforced throughout software delivery.

---

## Traditional SDLC

```text
Planning

↓

Development

↓

Testing

↓

Deployment

↓

Security Review

↓

Production
```

Problems

- Security at the end
- Delayed releases
- High remediation cost
- Manual security testing

---

## DevSecOps Lifecycle

```text
Planning

↓

Code

↓

Build

↓

Test

↓

Package

↓

Release

↓

Deploy

↓

Operate

↓

Monitor
```

Security is integrated into every phase.

---

## DevSecOps Lifecycle Stages

### 1. Planning

Activities

- Security requirements
- Risk Assessment
- Threat Modeling
- Compliance Requirements

Security Goal

Build security requirements before writing code.

---

### 2. Development

Activities

- Secure Coding
- Code Reviews
- Secrets Management

Security Goal

Prevent insecure code from entering the repository.

---

### 3. Build

Activities

- Compile Application
- Dependency Resolution
- Package Creation

Security Goal

Verify dependencies and prevent vulnerable packages.

---

### 4. Testing

Activities

- Unit Tests
- Integration Tests
- SAST
- SCA

Security Goal

Identify vulnerabilities before packaging.

---

### 5. Package

Activities

- Docker Build
- Container Scan
- Image Signing
- SBOM Generation

Security Goal

Build trusted container images.

---

### 6. Release

Activities

- Artifact Verification
- Policy Validation
- Approval Gates

Security Goal

Release only verified software.

---

### 7. Deploy

Activities

- Kubernetes Deployment
- IaC Validation
- Admission Controllers

Security Goal

Deploy securely configured workloads.

---

### 8. Operate

Activities

- Runtime Monitoring
- Log Analysis
- Vulnerability Monitoring

Security Goal

Detect runtime attacks.

---

### 9. Monitor

Activities

- SIEM
- Alerting
- Incident Response
- Compliance Reports

Security Goal

Continuously improve security posture.

---

## Complete DevSecOps Lifecycle

```text
Planning
     │
     ▼
Development
     │
     ▼
Build
     │
     ▼
Testing
     │
     ▼
Package
     │
     ▼
Release
     │
     ▼
Deploy
     │
     ▼
Operate
     │
     ▼
Monitor
```

---

## Security Activities Across the Lifecycle

| Stage | Security Activity |
|---------|------------------|
| Planning | Threat Modeling |
| Development | Secure Coding |
| Build | Dependency Validation |
| Testing | SAST, SCA |
| Package | Image Scanning |
| Release | Image Signing |
| Deploy | IaC Validation |
| Operate | Runtime Security |
| Monitor | Incident Detection |

---

## Pipeline Integration

```text
Developer

↓

Git Push

↓

Gitleaks
(Secrets Scan)

↓

Build

↓

SonarQube
(SAST)

↓

Dependency Scan

↓

Docker Build

↓

Trivy Image Scan

↓

SBOM Generation

↓

Cosign Image Signing

↓

Checkov / tfsec
(IaC Scan)

↓

Deploy to Kubernetes

↓

OPA / Kyverno
(Policy Validation)

↓

OWASP ZAP
(DAST)

↓

Falco
(Runtime Monitoring)

↓

Production
```

Each lifecycle stage includes automated security validation before proceeding to the next stage.

---

## Production Workflow

```text
Developer

↓

Git Commit

↓

GitHub

↓

Jenkins

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

Deploy to EKS

↓

Runtime Monitoring

↓

SOC Dashboard
```

Security remains active throughout the application lifecycle.

---

## Production Example

A developer introduces a vulnerable library.

```text
Developer

↓

Git Push

↓

SCA Scan

↓

Critical CVE Found

↓

Pipeline Failed

↓

Dependency Updated

↓

Pipeline Re-executed

↓

Deployment Successful
```

The vulnerable package never reaches production.

---

## Best Practices

- Integrate security into every lifecycle stage
- Automate all security checks
- Scan every code commit
- Continuously monitor production
- Generate SBOMs
- Sign container images
- Enforce policy validation
- Rotate secrets regularly
- Keep dependencies updated
- Perform continuous compliance validation

---

## Common Mistakes

- Performing security only before deployment
- Ignoring dependency vulnerabilities
- Skipping runtime monitoring
- Deploying unsigned images
- Not validating Infrastructure as Code
- Hardcoding credentials
- Ignoring compliance failures
- Disabling security gates

---

# Common Troubleshooting

## Issue 1: Pipeline Stops During SAST

### Cause

Critical vulnerabilities exceeded the Quality Gate threshold.

### Resolution

```text
Review Findings

↓

Fix Vulnerabilities

↓

Re-run Scan

↓

Pipeline Continues
```

---

## Issue 2: Dependency Scan Reports Critical CVEs

### Cause

Application uses outdated third-party libraries.

### Resolution

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

## Issue 3: Container Image Scan Fails

### Cause

Docker image contains HIGH or CRITICAL vulnerabilities.

### Resolution

```text
Update Base Image

↓

Rebuild Image

↓

Re-run Trivy

↓

Push Image
```

---

## Issue 4: Deployment Blocked by Policy

### Cause

OPA or Kyverno rejected the deployment.

### Resolution

```text
Review Policy

↓

Fix Manifest

↓

Validate Again

↓

Deploy
```

---

## Issue 5: Runtime Security Generates Alerts

### Cause

Falco detected suspicious container activity.

### Resolution

```text
Review Falco Alert

↓

Investigate Container

↓

Contain Threat

↓

Perform RCA
```

---

# Production Interview Questions

## Question 1

### Production-Level Answer

What is the DevSecOps Lifecycle?

The DevSecOps Lifecycle integrates security into every phase of the SDLC, ensuring continuous security from planning through production monitoring.

---

## Question 2

### Production-Level Answer

Why is planning important in DevSecOps?

Planning identifies security requirements, compliance obligations, and potential threats before development begins.

---

## Question 3

### Production-Level Answer

What security activities occur during development?

Secure coding, code reviews, secrets management, and early security validation.

---

## Question 4

### Production-Level Answer

Why is dependency scanning included in the lifecycle?

Third-party libraries often introduce vulnerabilities. SCA tools identify known CVEs before deployment.

---

## Question 5

### Production-Level Answer

Why are container images scanned?

To detect operating system and package vulnerabilities before images are pushed to a registry.

---

## Question 6

### Production-Level Answer

Why should images be signed before deployment?

Image signing verifies authenticity and ensures only trusted artifacts are deployed.

---

## Question 7

### Production-Level Answer

What is runtime security?

Runtime security continuously monitors running workloads to detect suspicious behavior after deployment.

---

## Question 8

### Production-Level Answer

Why is continuous monitoring important?

New vulnerabilities and attacks can occur after deployment, making continuous monitoring essential.

---

## Question 9

### Production-Level Answer

Which lifecycle stage usually detects vulnerabilities earliest?

Development and Testing stages, where SAST, SCA, and secure coding practices identify issues before deployment.

---

## Question 10

### Production-Level Answer

What is the biggest advantage of following the complete DevSecOps Lifecycle?

It enables organizations to continuously deliver secure software while reducing vulnerabilities, compliance risks, and production incidents.

---

# Key Takeaways

- DevSecOps integrates security into every SDLC phase.
- Every lifecycle stage has specific security responsibilities.
- Security automation reduces manual effort and human error.
- Continuous monitoring protects workloads after deployment.
- A complete DevSecOps Lifecycle enables secure, reliable, and compliant software delivery.