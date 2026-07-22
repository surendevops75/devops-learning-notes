# Introduction to DevSecOps

## What is DevSecOps?

DevSecOps (Development, Security, and Operations) is the practice of integrating security into every phase of the Software Development Life Cycle (SDLC). Instead of treating security as a final step before deployment, DevSecOps makes security a shared responsibility across developers, security teams, and operations.

The primary goal is to build, test, deploy, and operate applications securely without slowing down software delivery.

---

## Traditional SDLC

```
        Development
             │
             ▼
          Build/Test
             │
             ▼
         Operations
             │
             ▼
         Security Audit
             │
             ▼
         Production
```

Security happens at the end.

Problems:

• Vulnerabilities are discovered late.

• Fixing issues becomes expensive.

• Releases are delayed.

• Developers receive security feedback after development is complete.

---

## DevSecOps SDLC

```
Developer
    │
    ▼
Code Commit
    │
    ▼
Static Security Scan (SAST)
    │
    ▼
Build
    │
    ▼
Dependency Scan (SCA)
    │
    ▼
Container Scan
    │
    ▼
Infrastructure Scan
    │
    ▼
Deploy
    │
    ▼
Dynamic Security Scan (DAST)
    │
    ▼
Production Monitoring
```

Security is integrated into every stage of the pipeline.

---

## Core Principle

Instead of saying:

```
Build First
Secure Later
```

DevSecOps follows:

```
Build Securely
Deploy Securely
Operate Securely
```

---

## Shared Responsibility

```
        +------------------+
        |   Developers     |
        +------------------+
                 │
                 │
        +------------------+
        | Security Team    |
        +------------------+
                 │
                 │
        +------------------+
        | Operations Team  |
        +------------------+
```

Everyone is responsible for security.

---

## DevSecOps Goals

• Detect vulnerabilities early

• Automate security testing

• Reduce security risks

• Improve compliance

• Secure CI/CD pipelines

• Protect infrastructure

• Reduce manual security reviews

• Enable continuous delivery with security

---

## Security Throughout the Pipeline

```
Code
 │
 ▼
SAST
 │
 ▼
Build
 │
 ▼
SCA
 │
 ▼
Docker Image Scan
 │
 ▼
IaC Scan
 │
 ▼
Deploy
 │
 ▼
DAST
 │
 ▼
Runtime Security
```

Every stage contains automated security validation.

---

## Common DevSecOps Tools

| Category | Example Tools |
|-----------|---------------|
| Source Code | GitHub, GitLab |
| CI/CD | Jenkins, GitHub Actions |
| SAST | SonarQube, Veracode |
| SCA | Trivy, Snyk |
| IaC Scan | Checkov, tfsec |
| Container Scan | Trivy, Clair |
| DAST | OWASP ZAP |
| Secrets Scan | Gitleaks |
| Policy | OPA Gatekeeper, Kyverno |
| Runtime Security | Falco |
| Kubernetes Security | Kubescape, Trivy |

---

## Production Example

A developer pushes code to GitHub.

Pipeline executes automatically:

```
Developer
      │
      ▼
Git Push
      │
      ▼
Jenkins
      │
      ▼
SonarQube
      │
      ▼
Trivy
      │
      ▼
Checkov
      │
      ▼
Docker Build
      │
      ▼
Image Scan
      │
      ▼
Deploy to Kubernetes
      │
      ▼
OWASP ZAP
      │
      ▼
Production
```

The application is deployed only if all security checks pass.

---

## Benefits of DevSecOps

• Faster vulnerability detection

• Lower remediation cost

• Automated security

• Secure software delivery

• Faster compliance

• Continuous monitoring

• Reduced production incidents

• Stronger software supply chain security

---

## Best Practices

• Shift security left

• Automate security scans

• Never hardcode secrets

• Use least privilege access

• Scan every Docker image

• Scan Infrastructure as Code

• Continuously monitor workloads

• Automate compliance checks

• Keep dependencies updated

• Integrate security into CI/CD

---
---

## Interview Questions

### Basic

**1. What is DevSecOps?**

DevSecOps is the practice of integrating security into every stage of the Software Development Life Cycle (SDLC) by automating security testing and making security a shared responsibility among Development, Security, and Operations teams.

---

**2. What is the main goal of DevSecOps?**

To identify and fix security vulnerabilities as early as possible while maintaining fast and reliable software delivery.

---

**3. What does "Shift Left" mean in DevSecOps?**

Shift Left means moving security testing to the early stages of software development instead of performing it only before production deployment.

---

**4. How is DevSecOps different from DevOps?**

DevOps focuses on faster software delivery through collaboration and automation, whereas DevSecOps integrates security checks and controls throughout the CI/CD pipeline without sacrificing delivery speed.

---

**5. Why is security integrated into every pipeline stage?**

Because vulnerabilities discovered early are easier, faster, and less expensive to fix than vulnerabilities found after deployment.

---

### Intermediate

**6. Why is DevSecOps considered a cultural change rather than just a set of tools?**

Because security becomes the responsibility of everyone involved in software delivery, not just the security team.

---

**7. Name some common DevSecOps tools.**

- SonarQube
- Veracode
- Trivy
- Checkov
- tfsec
- OWASP ZAP
- Gitleaks
- Falco
- OPA Gatekeeper
- Kyverno

---

**8. Where are security scans typically performed in a CI/CD pipeline?**

- Source Code
- Dependencies
- Docker Images
- Infrastructure as Code
- Kubernetes Manifests
- Runtime Environment

---

**9. Why is automation important in DevSecOps?**

Automation ensures every code change is consistently scanned for vulnerabilities without relying on manual reviews.

---

**10. What are the biggest benefits of implementing DevSecOps?**

- Faster vulnerability detection
- Lower remediation cost
- Secure CI/CD pipelines
- Continuous compliance
- Improved software quality
- Reduced production incidents

---

## Key Takeaways

• DevSecOps integrates security into the entire SDLC.

• Security is everyone's responsibility.

• Security testing is automated.

• Every code change is validated before deployment.

• Secure software is delivered continuously without slowing development.