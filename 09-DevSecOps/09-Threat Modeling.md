# 09 - Threat Modeling

## Introduction

Threat Modeling is a structured process used to identify, analyze, and mitigate potential security threats before an application is built. It helps development and security teams proactively identify vulnerabilities during the design phase instead of discovering them after deployment.

Threat Modeling is a key practice in Secure SDLC and DevSecOps because preventing vulnerabilities is far less expensive than fixing them in production.

---

## Why Do We Need Threat Modeling?

Every application handles valuable assets such as:

- Customer Data
- API Keys
- Payment Information
- Business Logic
- Cloud Infrastructure
- Source Code

Without Threat Modeling:

- Security risks are overlooked
- Design flaws remain unnoticed
- Applications become easier to attack
- Remediation costs increase
- Compliance becomes difficult

Threat Modeling helps identify security risks before development begins.

---

## What is Threat Modeling?

Threat Modeling is the process of understanding:

- What are we building?
- What can go wrong?
- How can attackers exploit it?
- What controls can prevent attacks?

It focuses on designing secure systems rather than fixing insecure ones later.

---

## When Should Threat Modeling Be Performed?

Threat Modeling should begin during the **Design Phase** of the SDLC.

```text
Requirements

↓

Architecture Design

↓

Threat Modeling

↓

Development

↓

Testing

↓

Deployment
```

It should also be revisited whenever:

- New features are added
- Architecture changes
- New services are introduced
- Cloud infrastructure changes
- Security incidents occur

---

# Threat Modeling Process

```text
Identify Assets

↓

Create Data Flow Diagram

↓

Identify Trust Boundaries

↓

Identify Threats

↓

Assess Risk

↓

Implement Security Controls

↓

Validate Controls
```

---

# What Are Assets?

Assets are anything valuable that must be protected.

Examples

- Customer Data
- Databases
- Kubernetes Cluster
- AWS Account
- Secrets
- Docker Images
- APIs
- Source Code

---

# Data Flow Diagram (DFD)

A Data Flow Diagram (DFD) shows how information moves through an application.

```text
+---------+
|  Client |
+---------+
      │
      ▼
+-------------+
| LoadBalancer|
+-------------+
      │
      ▼
+-------------+
| API Service |
+-------------+
      │
      ▼
+-------------+
|  Database   |
+-------------+
```

DFDs help identify where attackers may attempt to compromise the system.

---

# Trust Boundaries

Trust Boundaries separate trusted components from untrusted components.

```text
Internet
     │
     ▼
────────────── Trust Boundary ──────────────

Load Balancer

↓

Application

↓

Database
```

Whenever data crosses a trust boundary, security validation should occur.

---

# STRIDE Threat Model

Microsoft's STRIDE model is one of the most widely used threat modeling frameworks.

| Threat | Description |
|---------|-------------|
| Spoofing | Pretending to be another user |
| Tampering | Modifying data or software |
| Repudiation | Denying actions performed |
| Information Disclosure | Exposing sensitive information |
| Denial of Service | Making services unavailable |
| Elevation of Privilege | Gaining unauthorized permissions |

---

## Spoofing

An attacker impersonates another user or service.

Example

```text
Attacker

↓

Uses Stolen Credentials

↓

Logs In As Admin
```

Mitigation

- MFA
- Strong Authentication
- OAuth
- JWT Validation

---

## Tampering

Unauthorized modification of data.

Example

```text
Attacker

↓

Modify Payment Amount

↓

Database Updated
```

Mitigation

- Hashing
- Digital Signatures
- Input Validation

---

## Repudiation

Users deny performing an action.

Example

```text
User

↓

Deletes Record

↓

Claims "I Didn't Do It"
```

Mitigation

- Audit Logs
- Immutable Logging
- SIEM

---

## Information Disclosure

Sensitive information becomes exposed.

Example

```text
Application

↓

Returns Stack Trace

↓

Database Password Visible
```

Mitigation

- Encryption
- Secrets Management
- Secure Error Messages

---

## Denial of Service (DoS)

Attackers overwhelm a service.

Example

```text
Attacker

↓

Millions of Requests

↓

Application Crash
```

Mitigation

- Rate Limiting
- WAF
- Auto Scaling
- Load Balancing

---

## Elevation of Privilege

Attackers gain higher permissions.

Example

```text
Normal User

↓

Privilege Escalation

↓

Administrator Access
```

Mitigation

- Least Privilege
- RBAC
- IAM Policies

---

# Threat Modeling Workflow

```text
Business Requirement

↓

Application Design

↓

Identify Assets

↓

Create Data Flow Diagram

↓

Identify Trust Boundaries

↓

Identify Threats (STRIDE)

↓

Risk Assessment

↓

Security Controls

↓

Development

↓

Testing

↓

Deployment
```

---

## DevSecOps Pipeline Integration

```text
Business Requirements

↓

Threat Modeling

↓

Architecture Review

↓

Secure Design

↓

Development

↓

Git Commit

↓

Gitleaks

↓

SonarQube

↓

Dependency Scan

↓

Docker Build

↓

Trivy

↓

Checkov

↓

Deploy

↓

OWASP ZAP

↓

Production

↓

Falco
```

Threat Modeling happens before development begins, preventing many security issues from entering the pipeline.

---

## Production Workflow

```text
Business Requirement

↓

Threat Modeling Workshop

↓

Architecture Review

↓

Security Approval

↓

Development

↓

CI/CD Pipeline

↓

Production
```

---

## Production Example

A team plans to expose a Kubernetes API publicly.

During Threat Modeling:

```text
Architecture Review

↓

Identify Public API

↓

Potential Unauthorized Access

↓

Require Authentication

↓

Deploy Securely
```

The issue is identified during design instead of after deployment.

---

## Best Practices

- Perform Threat Modeling during design
- Include developers and security engineers
- Identify trust boundaries
- Protect sensitive assets
- Use STRIDE consistently
- Review architecture regularly
- Update models after major changes
- Document security decisions
- Validate implemented controls
- Revisit Threat Models after incidents

---

## Common Mistakes

- Skipping Threat Modeling
- Performing it after development
- Ignoring trust boundaries
- Not documenting threats
- Missing third-party integrations
- Assuming cloud services are secure by default
- Ignoring insider threats
- Never updating Threat Models

---

# Common Troubleshooting

## Issue 1

### New Feature Introduces Security Risk

**Cause**

Threat Modeling was not performed for the new feature.

**Resolution**

```text
Review Design

↓

Update Threat Model

↓

Identify Threats

↓

Implement Controls
```

---

## Issue 2

### Public API Exposes Sensitive Data

**Cause**

Missing authentication and authorization.

**Resolution**

```text
Identify Trust Boundary

↓

Implement Authentication

↓

Validate Access

↓

Deploy
```

---

## Issue 3

### Security Incident Not Considered During Design

**Cause**

Threat scenarios were not identified.

**Resolution**

```text
Review Architecture

↓

Perform STRIDE Analysis

↓

Update Controls

↓

Retest
```

---

## Issue 4

### Third-Party Service Becomes Attack Vector

**Cause**

External integrations were excluded from Threat Modeling.

**Resolution**

```text
Review Integrations

↓

Identify Risks

↓

Apply Security Controls

↓

Monitor Continuously
```

---

## Issue 5

### Architecture Changes Introduce New Threats

**Cause**

Threat Model was never updated.

**Resolution**

```text
Review Architecture

↓

Update Threat Model

↓

Assess Risks

↓

Implement Controls
```

---

# Production Interview Questions

## Question 1

### What is Threat Modeling?

Threat Modeling is a structured process used to identify potential security threats during the design phase of an application so appropriate security controls can be implemented before development begins.

---

## Question 2

### Why is Threat Modeling important?

Threat Modeling helps identify design flaws early, reduces remediation costs, improves application security, and prevents vulnerabilities from reaching production.

---

## Question 3

### When should Threat Modeling be performed?

It should be performed during the architecture and design phase before development starts and should be updated whenever major architectural changes occur.

---

## Question 4

### What are the main steps in Threat Modeling?

Identify assets, create Data Flow Diagrams, identify trust boundaries, analyze threats, assess risks, implement security controls, and validate those controls.

---

## Question 5

### What is STRIDE?

STRIDE is a threat modeling framework developed by Microsoft that categorizes threats into Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, and Elevation of Privilege.

---

## Question 6

### What is a Trust Boundary?

A Trust Boundary is a point where data moves between components with different trust levels, requiring authentication, authorization, or validation.

---

## Question 7

### Why are Data Flow Diagrams important in Threat Modeling?

They help visualize how data moves through the system, making it easier to identify attack surfaces and trust boundaries.

---

## Question 8

### Who should participate in a Threat Modeling session?

Developers, DevOps Engineers, DevSecOps Engineers, Security Engineers, Architects, Product Owners, and other stakeholders responsible for the application.

---

## Question 9

### Can Threat Modeling replace security testing?

No. Threat Modeling identifies potential risks during design, while security testing validates whether vulnerabilities actually exist during development and deployment.

---

## Question 10

### How is Threat Modeling integrated into DevSecOps?

Threat Modeling is performed before development begins. The identified threats drive secure architecture decisions and determine which security controls and automated security checks should be integrated into the CI/CD pipeline.

---

# Key Takeaways

- Threat Modeling is a proactive security practice performed during the design phase.
- It identifies potential threats before development begins.
- STRIDE is one of the most commonly used Threat Modeling frameworks.
- Data Flow Diagrams and Trust Boundaries help identify attack surfaces.
- Effective Threat Modeling reduces vulnerabilities, improves secure design, and strengthens the overall DevSecOps lifecycle.