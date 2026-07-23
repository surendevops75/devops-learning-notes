# Risk Assessment

## Introduction

Risk Assessment is the process of identifying, analyzing, and prioritizing security risks that could impact an application, infrastructure, or business. It helps organizations understand which threats pose the greatest risk and where security efforts should be focused.

In DevSecOps, Risk Assessment is performed continuously throughout the Software Development Life Cycle (SDLC) to reduce the likelihood and impact of security incidents.

---

## Why Do We Need Risk Assessment?

Every application faces security threats, but not all threats carry the same level of risk.

Without Risk Assessment:

- Critical vulnerabilities may be ignored.
- Resources may be wasted on low-priority issues.
- Business-critical assets remain exposed.
- Compliance requirements become difficult to meet.
- Security investments may be ineffective.

Risk Assessment helps organizations prioritize security efforts based on business impact.

---

## What is Risk Assessment?

Risk Assessment evaluates potential threats by considering:

- What asset is being protected?
- What threats exist?
- What vulnerabilities can be exploited?
- What is the likelihood of an attack?
- What would be the business impact?
- What controls can reduce the risk?

---

# Risk Assessment Process

```text
Identify Assets

↓

Identify Threats

↓

Identify Vulnerabilities

↓

Analyze Likelihood

↓

Analyze Impact

↓

Calculate Risk

↓

Implement Controls

↓

Review & Monitor
```

---

# Key Components of Risk Assessment

## Asset

Anything valuable to the organization.

Examples

- Customer Data
- Kubernetes Cluster
- AWS Account
- Source Code
- Secrets
- Databases

---

## Threat

Anything capable of causing harm.

Examples

- Hackers
- Malware
- Insider Threats
- DDoS Attacks
- Supply Chain Attacks

---

## Vulnerability

A weakness that can be exploited.

Examples

- Weak Passwords
- Public S3 Bucket
- SQL Injection
- Missing MFA
- Vulnerable Dependencies

---

## Likelihood

The probability that a threat will exploit a vulnerability.

Examples

- Low
- Medium
- High

---

## Impact

The damage caused if the threat succeeds.

Examples

- Financial Loss
- Reputation Damage
- Data Breach
- Service Downtime
- Compliance Violations

---

## Risk Formula

A commonly used approach is:

```text
Risk = Likelihood × Impact
```

Example

| Likelihood | Impact | Risk |
|------------|--------|------|
| Low | Low | Low |
| Medium | High | High |
| High | High | Critical |

---

# Risk Matrix

```text
                Impact

           Low  Med  High

High       Med  High Critical

Medium     Low  Med  High

Low        Low  Low  Med

          Likelihood
```

This matrix helps prioritize remediation activities.

---

# Types of Risks

## Business Risk

Risks affecting business operations.

Examples

- Revenue loss
- Compliance penalties
- Reputation damage

---

## Technical Risk

Risks related to technology.

Examples

- Unpatched software
- Vulnerable dependencies
- Weak authentication

---

## Operational Risk

Risks caused by operational failures.

Examples

- Human error
- Misconfiguration
- Backup failure

---

## Compliance Risk

Risks related to regulatory requirements.

Examples

- GDPR
- HIPAA
- PCI-DSS
- ISO 27001

---

# Risk Treatment Options

```text
Risk Identified

↓

Choose Strategy

↓

Accept

OR

Mitigate

OR

Transfer

OR

Avoid
```

### Accept

The organization accepts the risk.

### Mitigate

Implement security controls to reduce risk.

### Transfer

Transfer the risk through insurance or third-party services.

### Avoid

Remove the activity that introduces the risk.

---

# Risk Assessment Workflow

```text
Business Requirement

↓

Identify Assets

↓

Threat Modeling

↓

Risk Assessment

↓

Define Security Controls

↓

Development

↓

Testing

↓

Deployment

↓

Continuous Monitoring
```

---

## DevSecOps Pipeline Integration

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

Falco Monitoring
```

Risk Assessment determines where security controls should be applied within the pipeline.

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

A team plans to expose an internal API to the internet.

```text
Identify Asset

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

Rate Limiting

WAF

↓

Deploy
```

The identified risks are reduced before deployment.

---

## Best Practices

- Perform Risk Assessments early
- Review risks regularly
- Prioritize critical assets
- Document identified risks
- Update assessments after architecture changes
- Use a standardized risk scoring model
- Automate vulnerability scanning
- Continuously monitor production
- Involve business and technical teams
- Review compliance requirements

---

## Common Mistakes

- Ignoring business impact
- Assessing risks only once
- Prioritizing all risks equally
- Missing third-party risks
- Ignoring insider threats
- Poor documentation
- Not updating risk assessments
- Delaying remediation of critical risks

---

# Common Troubleshooting

## Issue 1

### Critical Vulnerability Not Prioritized

**Cause**

Improper risk scoring.

**Resolution**

```text
Review Likelihood

↓

Review Impact

↓

Recalculate Risk

↓

Prioritize Remediation
```

---

## Issue 2

### Security Controls Do Not Reduce Risk

**Cause**

Controls do not address the identified threat.

**Resolution**

```text
Review Threat

↓

Select Appropriate Control

↓

Validate Effectiveness

↓

Deploy
```

---

## Issue 3

### Compliance Audit Finds Unaddressed Risks

**Cause**

Risk Register was not updated.

**Resolution**

```text
Review Assets

↓

Update Risk Register

↓

Implement Controls

↓

Reassess
```

---

## Issue 4

### New Architecture Introduces Unknown Risks

**Cause**

Risk Assessment was not repeated after design changes.

**Resolution**

```text
Review Architecture

↓

Perform New Assessment

↓

Update Controls

↓

Deploy
```

---

## Issue 5

### Production Incident Occurs Despite Previous Assessment

**Cause**

Threat landscape changed after deployment.

**Resolution**

```text
Investigate Incident

↓

Update Risk Assessment

↓

Improve Controls

↓

Continuous Monitoring
```

---

# Production Interview Questions

## Question 1

### What is Risk Assessment?

Risk Assessment is the process of identifying, analyzing, evaluating, and prioritizing security risks so appropriate mitigation strategies can be implemented.

---

## Question 2

### Why is Risk Assessment important in DevSecOps?

It helps organizations focus on the highest-priority risks, enabling secure software delivery while optimizing security resources.

---

## Question 3

### What are the main components of Risk Assessment?

Assets, Threats, Vulnerabilities, Likelihood, Impact, Risk Score, and Security Controls.

---

## Question 4

### What is the difference between a Threat and a Vulnerability?

A threat is something capable of causing harm, while a vulnerability is a weakness that a threat can exploit.

---

## Question 5

### How is Risk commonly calculated?

Risk is commonly estimated using the relationship between Likelihood and Impact, often expressed as Risk = Likelihood × Impact.

---

## Question 6

### What are the common risk treatment strategies?

Accept, Mitigate, Transfer, and Avoid.

---

## Question 7

### When should Risk Assessments be performed?

During planning and design, before major releases, after architectural changes, following security incidents, and periodically throughout the application lifecycle.

---

## Question 8

### How does Risk Assessment support Threat Modeling?

Threat Modeling identifies potential threats, while Risk Assessment evaluates their likelihood and impact to determine which risks require mitigation.

---

## Question 9

### Can Risk Assessment eliminate all security risks?

No. It helps identify and prioritize risks, but continuous monitoring and periodic reassessment are required because the threat landscape constantly evolves.

---

## Question 10

### How is Risk Assessment integrated into DevSecOps?

Risk Assessment is performed before development begins and guides architecture decisions, security controls, automated security testing, and monitoring throughout the DevSecOps pipeline.

---

# Key Takeaways

- Risk Assessment helps prioritize security efforts based on business impact.
- Assets, threats, vulnerabilities, likelihood, and impact are the foundation of risk analysis.
- Risk treatment strategies include Accept, Mitigate, Transfer, and Avoid.
- Continuous Risk Assessment is essential because applications and threats evolve over time.
- Effective Risk Assessment enables organizations to build secure, resilient, and compliant software.