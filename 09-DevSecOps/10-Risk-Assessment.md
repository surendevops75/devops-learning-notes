# Risk Assessment

## Introduction

Risk Assessment is the process of identifying, analyzing, evaluating, and prioritizing security risks that may impact an application, infrastructure, or business. It helps organizations understand which risks require immediate attention and which risks can be accepted, mitigated, transferred, or avoided.

In DevSecOps, Risk Assessment is a continuous activity performed throughout the Secure SDLC to reduce the likelihood and impact of security incidents.

---

## Why Do We Need Risk Assessment?

Modern applications face numerous security threats.

Without Risk Assessment:

- Critical vulnerabilities may remain unnoticed
- Security resources may be spent on low-priority issues
- Compliance requirements may not be met
- Business-critical assets may remain exposed
- Security decisions become reactive instead of proactive

Risk Assessment enables organizations to focus on the highest-risk areas first.

---

## What is Risk Assessment?

Risk Assessment evaluates security risks by answering five important questions.

- What are we protecting?
- What threats exist?
- What vulnerabilities can be exploited?
- How likely is the attack?
- What would be the business impact?

Based on these answers, organizations implement appropriate security controls.

---

# Risk Assessment Process

```text
Identify Assets

↓

Identify Threats

↓

Identify Vulnerabilities

↓

Determine Likelihood

↓

Determine Impact

↓

Calculate Risk

↓

Implement Security Controls

↓

Review & Monitor
```

---

# Key Components of Risk Assessment

## Assets

Assets are valuable resources that require protection.

Examples

- Customer Data
- Source Code
- Kubernetes Cluster
- AWS Resources
- Secrets
- Databases
- CI/CD Pipeline

---

## Threats

Threats are events or actors capable of causing damage.

Examples

- Hackers
- Insider Threats
- Malware
- Ransomware
- Supply Chain Attacks
- DDoS Attacks

---

## Vulnerabilities

Vulnerabilities are weaknesses that attackers can exploit.

Examples

- SQL Injection
- Weak Passwords
- Missing MFA
- Public S3 Buckets
- Vulnerable Dependencies
- Misconfigured Security Groups

---

## Likelihood

Likelihood represents the probability that a threat will exploit a vulnerability.

Typical Ratings

- Low
- Medium
- High

---

## Impact

Impact measures the damage caused if the attack succeeds.

Examples

- Financial Loss
- Data Breach
- Service Downtime
- Reputation Damage
- Regulatory Penalties

---

# Risk Formula

A commonly used formula is:

```text
Risk = Likelihood × Impact
```

Example

| Likelihood | Impact | Risk |
|------------|--------|------|
| Low | Low | Low |
| Low | High | Medium |
| Medium | Medium | Medium |
| Medium | High | High |
| High | High | Critical |

---

# Risk Matrix

```text
                    Impact

              Low   Medium   High

High          Med    High   Critical

Medium        Low    Medium   High

Low           Low     Low    Medium

             Likelihood
```

This matrix helps prioritize remediation activities.

---

# Types of Risks

## Business Risk

Risks affecting business operations.

Examples

- Revenue Loss
- Customer Trust
- Reputation Damage

---

## Technical Risk

Risks related to technology.

Examples

- Vulnerable Applications
- Misconfigured Infrastructure
- Outdated Software

---

## Operational Risk

Risks caused by operational failures.

Examples

- Human Error
- Backup Failure
- Deployment Failure

---

## Compliance Risk

Risks associated with regulatory requirements.

Examples

- GDPR
- HIPAA
- PCI-DSS
- ISO 27001

---

# Risk Treatment Strategies

```text
Risk Identified

↓

Choose Strategy

├── Accept
├── Mitigate
├── Transfer
└── Avoid
```

### Accept

The organization accepts the risk because the impact is minimal or the mitigation cost outweighs the benefit.

### Mitigate

Reduce the likelihood or impact by implementing security controls.

Examples

- MFA
- Encryption
- WAF
- IAM

### Transfer

Transfer the financial impact to another party.

Examples

- Cyber Insurance
- Managed Security Services

### Avoid

Eliminate the activity causing the risk.

Example

Disable an insecure public API instead of exposing it.

---

# Risk Assessment Workflow

```text
Business Requirements

↓

Identify Assets

↓

Threat Modeling

↓

Risk Assessment

↓

Security Controls

↓

Development

↓

Security Testing

↓

Deployment

↓

Continuous Monitoring
```

---

## Pipeline Integration

```text
Business Requirements

↓

Threat Modeling

↓

Risk Assessment

↓

Architecture Review

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

↓

Monitoring
```

Risk Assessment determines where security controls should be implemented throughout the pipeline.

---

## Production Workflow

```text
Business Requirement

↓

Architecture Design

↓

Threat Modeling

↓

Risk Assessment

↓

Security Controls

↓

CI/CD Pipeline

↓

Production

↓

Continuous Monitoring
```

---

## Production Example

An organization plans to expose an internal REST API publicly.

```text
Asset

↓

Public API

↓

Threat

Unauthorized Access

↓

Risk

High

↓

Mitigation

Authentication

↓

Rate Limiting

↓

Web Application Firewall

↓

Deploy Securely
```

The API is protected before being released to production.

---

## Best Practices

- Perform Risk Assessments early in the SDLC
- Prioritize business-critical assets
- Use standardized risk scoring
- Review risks periodically
- Automate vulnerability scanning
- Document all identified risks
- Update assessments after architectural changes
- Involve developers, security teams, and business stakeholders
- Monitor production continuously
- Review compliance requirements regularly

---

## Common Mistakes

- Performing Risk Assessment only once
- Ignoring business impact
- Treating every risk equally
- Ignoring third-party dependencies
- Failing to document risks
- Ignoring insider threats
- Delaying remediation of critical risks
- Not updating the Risk Register

---

# Common Troubleshooting

## Issue 1

### Critical Vulnerabilities Are Not Prioritized

**Cause**

Risk scoring was inaccurate.

**Resolution**

```text
Review Risk Matrix

↓

Recalculate Likelihood

↓

Recalculate Impact

↓

Prioritize Critical Risks
```

---

## Issue 2

### Security Controls Fail to Reduce Risk

**Cause**

The selected controls do not address the identified threat.

**Resolution**

```text
Review Threat

↓

Review Controls

↓

Implement Better Controls

↓

Validate Effectiveness
```

---

## Issue 3

### Compliance Audit Reports High-Risk Findings

**Cause**

Risk Assessments were outdated.

**Resolution**

```text
Update Risk Register

↓

Perform New Assessment

↓

Implement Missing Controls

↓

Revalidate Compliance
```

---

## Issue 4

### New Cloud Architecture Introduces Security Risks

**Cause**

Risk Assessment was not performed after the architecture changed.

**Resolution**

```text
Review Architecture

↓

Perform Assessment

↓

Update Controls

↓

Deploy Securely
```

---

## Issue 5

### Security Incident Occurs Despite Previous Assessment

**Cause**

New attack techniques emerged after deployment.

**Resolution**

```text
Investigate Incident

↓

Update Threat Model

↓

Update Risk Assessment

↓

Improve Security Controls
```

---

# Production Interview Questions

## Question 1

### What is Risk Assessment?

Risk Assessment is the process of identifying, analyzing, evaluating, and prioritizing security risks so organizations can implement appropriate security controls.

---

## Question 2

### Why is Risk Assessment important in DevSecOps?

It enables teams to focus on the highest-priority risks, improving security while optimizing development effort and resources.

---

## Question 3

### What are the key components of Risk Assessment?

Assets, Threats, Vulnerabilities, Likelihood, Impact, Risk Score, and Security Controls.

---

## Question 4

### What is the difference between a Threat and a Vulnerability?

A threat is something capable of causing damage, whereas a vulnerability is a weakness that allows the threat to succeed.

---

## Question 5

### How is Risk typically calculated?

Risk is commonly estimated by evaluating the likelihood of an attack and the potential business impact if it occurs.

---

## Question 6

### What are the four common Risk Treatment strategies?

Accept, Mitigate, Transfer, and Avoid.

---

## Question 7

### When should Risk Assessments be performed?

During planning, architecture design, before major releases, after infrastructure changes, after security incidents, and periodically throughout the application lifecycle.

---

## Question 8

### How does Threat Modeling support Risk Assessment?

Threat Modeling identifies potential threats, while Risk Assessment evaluates their likelihood and impact to determine remediation priorities.

---

## Question 9

### Can Risk Assessment eliminate all security risks?

No. Risk Assessment reduces and prioritizes risks, but continuous monitoring, vulnerability management, and periodic reassessment remain essential.

---

## Question 10

### How is Risk Assessment integrated into a DevSecOps pipeline?

Risk Assessment influences architecture decisions, security controls, CI/CD security checks, vulnerability scanning, deployment policies, and runtime monitoring to ensure risks are continuously managed throughout the software lifecycle.

---

# Key Takeaways

- Risk Assessment helps organizations identify and prioritize security risks.
- Assets, threats, vulnerabilities, likelihood, and impact form the foundation of risk analysis.
- Risk treatment strategies include Accept, Mitigate, Transfer, and Avoid.
- Risk Assessment should be a continuous activity throughout the Secure SDLC.
- Effective Risk Assessment enables secure, resilient, and compliant software delivery.