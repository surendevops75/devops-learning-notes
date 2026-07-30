# Enterprise DevSecOps Interview Questions and Answers

## Introduction

Enterprise DevSecOps interviews focus on how security is integrated into every phase of the Software Development Life Cycle (SDLC). Interviewers expect practical knowledge of security automation, secure CI/CD pipelines, cloud security, Kubernetes security, Infrastructure as Code (IaC) security, container security, compliance, and incident response.

This guide contains production-oriented interview questions with concise answers based on enterprise DevSecOps practices.

---

# Enterprise DevSecOps Interview Flow

```text
Resume Discussion

↓

Current Project

↓

DevSecOps Fundamentals

↓

Secure SDLC

↓

CI/CD Security

↓

SAST

↓

SCA

↓

Secret Detection

↓

IaC Security

↓

Container Security

↓

Kubernetes Security

↓

Cloud Security

↓

Compliance

↓

Runtime Security

↓

Incident Response

↓

Scenario Based Questions

↓

Manager Round
```

---

# DevSecOps Fundamentals

---

# Question 1

## What is DevSecOps?

### Answer

DevSecOps is the practice of integrating security into every stage of the Software Development Life Cycle (SDLC). Instead of performing security checks only before production, security becomes an automated and continuous part of development, testing, deployment, and operations.

---

# DevSecOps Lifecycle

```text
Requirements

↓

Design

↓

Development

↓

Build

↓

SAST

↓

SCA

↓

Secrets Scan

↓

IaC Scan

↓

Container Scan

↓

Deploy

↓

Runtime Security

↓

Continuous Monitoring
```

---

# Question 2

## Why was DevSecOps introduced?

### Answer

Traditional security was performed at the end of software development, causing delayed releases and expensive fixes. DevSecOps moves security earlier in the lifecycle so vulnerabilities are detected and resolved before reaching production.

---

# Question 3

## What are the primary goals of DevSecOps?

### Answer

The objectives of DevSecOps include:

- Shift security left
- Automate security testing
- Reduce vulnerabilities
- Accelerate secure releases
- Improve compliance
- Strengthen software supply chain security
- Reduce security risks

---

# Question 4

## What is the difference between DevOps and DevSecOps?

| DevOps | DevSecOps |
|---------|------------|
| Focuses on speed and automation | Focuses on secure automation |
| Security often handled separately | Security integrated throughout SDLC |
| Limited automated security | Continuous automated security |
| Faster deployments | Secure and compliant deployments |

---

# Question 5

## Explain the "Shift Left" approach.

### Answer

Shift Left means performing security validation as early as possible during development instead of waiting until deployment or production.

Developers receive immediate feedback, reducing remediation cost and improving software quality.

---

# Shift Left Security

```text
Traditional

Development

↓

Testing

↓

Deployment

↓

Security

↓

Production


DevSecOps

Development

↓

Security

↓

Testing

↓

Build

↓

Deploy

↓

Production
```

---

# Question 6

## What is Shift Right Security?

### Answer

Shift Right focuses on monitoring and protecting applications after deployment using runtime security, intrusion detection, behavioural analysis, and continuous monitoring.

Examples include:

- Falco
- Runtime IDS
- SIEM
- Threat Detection
- Cloud Security Monitoring

---

# Question 7

## What are the core principles of DevSecOps?

### Answer

Enterprise DevSecOps follows these principles:

- Security as Code
- Automation
- Shift Left
- Continuous Security
- Least Privilege
- Immutable Infrastructure
- Continuous Monitoring
- Continuous Compliance
- Zero Trust
- Shared Responsibility

---

# Question 8

## What is Security as Code?

### Answer

Security as Code means defining security controls, policies, and compliance rules in version-controlled code so they can be automatically validated and enforced throughout the CI/CD pipeline.

Examples include:

- Checkov policies
- OPA Gatekeeper policies
- Kyverno policies
- Terraform policies
- GitHub branch protection rules

---

# Question 9

## What is Continuous Security?

### Answer

Continuous Security ensures that automated security validation occurs throughout the software lifecycle instead of relying on one-time assessments.

Security checks are integrated into:

- Source code
- Dependencies
- Infrastructure
- Containers
- Kubernetes
- Cloud resources
- Runtime environments

---

# Question 10

## What are the benefits of DevSecOps?

### Answer

Organizations adopt DevSecOps because it provides:

- Earlier vulnerability detection
- Faster remediation
- Reduced production risk
- Automated compliance
- Secure CI/CD pipelines
- Improved developer productivity
- Better audit readiness
- Lower remediation costs

---

# Enterprise DevSecOps Pipeline

```text
Developer

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Build

↓

Unit Tests

↓

SAST

↓

SCA

↓

Secret Scan

↓

IaC Scan

↓

Container Scan

↓

Policy Validation

↓

SBOM Generation

↓

Image Signing

↓

Artifact Repository

↓

GitOps

↓

Deployment

↓

Runtime Security

↓

Continuous Monitoring
```

---

# Question 11

## Why is automation important in DevSecOps?

### Answer

Manual security testing cannot keep pace with modern CI/CD pipelines. Automation enables consistent, repeatable, and fast security validation while reducing human error.

---

# Question 12

## What is a Security Gate in a CI/CD pipeline?

### Answer

A Security Gate is a checkpoint that validates predefined security policies before allowing the pipeline to continue.

Examples include:

- SonarQube Quality Gate
- Trivy Critical Vulnerability Check
- Checkov Policy Validation
- Veracode Policy Scan
- Gitleaks Secret Detection

If a critical policy fails, the deployment should stop until the issue is resolved or an approved exception is granted.

---

# Question 13

## What is Security Debt?

### Answer

Security Debt refers to unresolved vulnerabilities, insecure configurations, or security shortcuts that accumulate over time and increase future risk and remediation effort.

---

# Question 14

## What is the Shared Responsibility Model?

### Answer

In cloud environments, the cloud provider secures the underlying infrastructure, while customers are responsible for securing their applications, identities, configurations, operating systems (where applicable), and data.

---

# Question 15

## What skills are expected from a DevSecOps Engineer?

### Answer

A DevSecOps Engineer should have practical knowledge of:

- Linux
- Git
- CI/CD
- Secure SDLC
- AWS/Azure/GCP
- Docker
- Kubernetes
- Terraform
- Jenkins
- GitHub Actions
- GitLab CI
- SonarQube
- Trivy
- Veracode
- Checkov
- TFSec
- Gitleaks
- OWASP ZAP
- OPA Gatekeeper
- Kyverno
- Falco
- SIEM
- Secrets Management
- IAM
- Compliance Frameworks
- Incident Response

---

# Enterprise Best Practices

- Integrate security from the first stage of development.
- Automate every security validation possible.
- Treat security policies as version-controlled code.
- Block deployments when critical security gates fail.
- Apply the Principle of Least Privilege.
- Continuously monitor applications after deployment.
- Maintain compliance through automated policy enforcement.
- Perform regular security reviews and threat modelling.
- Generate SBOMs for software supply chain visibility.
- Adopt Zero Trust principles across infrastructure and applications.

---

# Secure SDLC Interview Questions

---

# Question 16

## What is Secure SDLC?

### Answer

Secure SDLC (Secure Software Development Life Cycle) integrates security activities into every phase of software development, from requirements gathering to production maintenance.

Instead of testing security only before release, Secure SDLC ensures security is built into the application from the beginning.

---

# Secure SDLC Lifecycle

```text
Requirements

↓

Security Requirements

↓

Architecture Design

↓

Threat Modeling

↓

Secure Coding

↓

Code Review

↓

SAST

↓

SCA

↓

Secret Scan

↓

Build

↓

Container Scan

↓

DAST

↓

Deployment

↓

Runtime Security

↓

Continuous Monitoring
```

---

# Question 17

## Why is Secure SDLC important?

### Answer

Secure SDLC helps organizations:

- Detect vulnerabilities early
- Reduce remediation costs
- Improve software quality
- Meet compliance requirements
- Reduce security incidents
- Protect customer data
- Minimize production risks

---

# Question 18

## What are the phases of Secure SDLC?

### Answer

A typical Secure SDLC consists of:

- Requirements
- Design
- Development
- Testing
- Deployment
- Operations
- Maintenance

Security activities are integrated into every phase.

---

# Question 19

## What security activities are performed during the Requirements phase?

### Answer

Typical activities include:

- Security requirements gathering
- Regulatory compliance identification
- Risk assessment
- Data classification
- Security acceptance criteria
- Privacy requirements

---

# Question 20

## What security activities are performed during the Design phase?

### Answer

Security design activities include:

- Threat modeling
- Security architecture review
- Trust boundary identification
- Authentication design
- Authorization design
- Encryption strategy
- Secure API design

---

# Question 21

## What security activities are performed during Development?

### Answer

During development:

- Secure coding practices
- Code reviews
- SAST
- Secret scanning
- Dependency management
- Secure library selection
- Peer review

---

# Question 22

## What security activities are performed during Testing?

### Answer

Testing includes:

- SAST validation
- DAST
- API security testing
- Penetration testing
- SCA validation
- Vulnerability assessment
- Security regression testing

---

# Question 23

## What security activities occur during Deployment?

### Answer

Before deployment:

- IaC scanning
- Container scanning
- Kubernetes policy validation
- Image signing
- SBOM generation
- Compliance validation
- Deployment approval

---

# Question 24

## What security activities occur after deployment?

### Answer

Production security includes:

- Runtime monitoring
- Threat detection
- Log analysis
- Security alerts
- Incident response
- Vulnerability management
- Patch management

---

# Question 25

## What is Threat Modeling?

### Answer

Threat Modeling is the process of identifying potential security threats, attack vectors, and weaknesses during the application design phase before implementation begins.

Its objective is to reduce security risks early in the SDLC.

---

# Threat Modeling Workflow

```text
Application Design

↓

Identify Assets

↓

Identify Threats

↓

Risk Analysis

↓

Mitigation

↓

Security Validation
```

---

# Question 26

## Why is Threat Modeling important?

### Answer

Threat Modeling enables organizations to:

- Discover security risks early
- Reduce design flaws
- Improve architecture
- Lower remediation costs
- Strengthen application security
- Meet compliance requirements

---

# Question 27

## What are Assets in Threat Modeling?

### Answer

Assets are valuable resources that require protection.

Examples include:

- Customer data
- Payment information
- Authentication tokens
- API keys
- Source code
- Databases
- Cloud resources
- Secrets
- Intellectual property

---

# Question 28

## What are Trust Boundaries?

### Answer

Trust Boundaries separate components with different security trust levels.

Whenever data crosses a trust boundary, additional validation and security controls should be applied.

---

# Trust Boundary Example

```text
Internet

↓

Load Balancer

========================
Trust Boundary
========================

Application

↓

Database
```

---

# Question 29

## What is STRIDE?

### Answer

STRIDE is Microsoft's threat modeling framework used to classify security threats.

| Threat | Meaning |
|---------|----------|
| S | Spoofing |
| T | Tampering |
| R | Repudiation |
| I | Information Disclosure |
| D | Denial of Service |
| E | Elevation of Privilege |

---

# Question 30

## Explain Spoofing.

### Answer

Spoofing occurs when an attacker pretends to be another user, system, or service to gain unauthorized access.

Examples include:

- Credential theft
- Token impersonation
- Session hijacking
- Fake identities

Mitigation:

- MFA
- Strong authentication
- Token validation
- Identity verification

---

# Question 31

## Explain Tampering.

### Answer

Tampering involves unauthorized modification of data, configuration files, requests, or application code.

Mitigation:

- Hashing
- Digital signatures
- Integrity validation
- Immutable infrastructure

---

# Question 32

## Explain Repudiation.

### Answer

Repudiation occurs when users deny performing an action because sufficient audit evidence is unavailable.

Mitigation:

- Audit logging
- Immutable logs
- Time synchronization
- Digital signatures

---

# Question 33

## Explain Information Disclosure.

### Answer

Information Disclosure occurs when confidential information becomes accessible to unauthorized users.

Examples:

- Database leaks
- API key exposure
- Sensitive logs
- Public storage buckets

Mitigation:

- Encryption
- Access control
- Data masking
- Least privilege

---

# Question 34

## Explain Denial of Service (DoS).

### Answer

Denial of Service attacks attempt to exhaust system resources, making applications unavailable to legitimate users.

Mitigation:

- Auto Scaling
- WAF
- Rate limiting
- Load balancing
- DDoS protection

---

# Question 35

## Explain Elevation of Privilege.

### Answer

Elevation of Privilege occurs when an attacker gains permissions beyond those originally granted.

Examples:

- Privilege escalation
- Kubernetes RBAC bypass
- IAM misconfiguration
- Container escape

Mitigation:

- Least Privilege
- RBAC
- IAM policies
- Security reviews
- Runtime protection

---

# Enterprise Best Practices

- Integrate Secure SDLC into every project.
- Perform Threat Modeling before development starts.
- Review security requirements during design.
- Identify trust boundaries early.
- Apply STRIDE during architecture reviews.
- Validate security controls throughout the SDLC.
- Automate security testing in CI/CD pipelines.
- Maintain immutable audit logs.
- Continuously review risks as applications evolve.
- Revisit threat models whenever significant architectural changes occur.

---

# SAST (Static Application Security Testing) Interview Questions

---

# Question 36

## What is SAST?

### Answer

Static Application Security Testing (SAST) is a security testing methodology that analyzes source code, bytecode, or binaries without executing the application. It helps identify security vulnerabilities early in the development lifecycle.

---

# SAST Workflow

```text
Developer

↓

Git Push

↓

CI Pipeline

↓

Source Code

↓

SAST Scan

↓

Security Report

↓

Developer Fix

↓

Re-Scan

↓

Build
```

---

# Question 37

## Why is SAST performed early in the SDLC?

### Answer

SAST is performed during development because vulnerabilities are easier, faster, and less expensive to fix before deployment.

Early detection reduces production risks and prevents insecure code from progressing through the CI/CD pipeline.

---

# Question 38

## What are the advantages of SAST?

### Answer

SAST provides:

- Early vulnerability detection
- Faster remediation
- Automated code analysis
- Integration with CI/CD
- Improved code quality
- Compliance support
- Reduced production risk

---

# Question 39

## What are the limitations of SAST?

### Answer

SAST cannot identify:

- Runtime vulnerabilities
- Business logic flaws
- Authentication bypass caused by deployment
- Server misconfigurations
- Infrastructure vulnerabilities
- Runtime attacks

SAST should be combined with DAST, SCA, and runtime security.

---

# Question 40

## What types of vulnerabilities can SAST detect?

### Answer

Common findings include:

- SQL Injection
- Cross-Site Scripting (XSS)
- Command Injection
- Hardcoded Credentials
- Weak Cryptography
- Buffer Overflow
- Null Pointer Issues
- Resource Leaks
- Insecure Coding Practices

---

# Question 41

## What is SonarQube?

### Answer

SonarQube is a Static Application Security Testing (SAST) and code quality platform that continuously analyzes source code for bugs, vulnerabilities, code smells, and maintainability issues.

It integrates with CI/CD pipelines to enforce quality and security gates before deployment.

---

# SonarQube Architecture

```text
Developer

↓

Git Repository

↓

CI Pipeline

↓

Sonar Scanner

↓

SonarQube Server

↓

Analysis Report

↓

Quality Gate

↓

Pipeline Decision
```

---

# Question 42

## What components make up SonarQube?

### Answer

The primary SonarQube components are:

- Sonar Scanner
- SonarQube Server
- Compute Engine
- Database
- Web Interface
- Quality Gates
- Quality Profiles

---

# SonarQube Architecture

```text
Developer

↓

Sonar Scanner

↓

Web Server

↓

Compute Engine

↓

Database

↓

Dashboard
```

---

# Question 43

## What is Sonar Scanner?

### Answer

Sonar Scanner is the client that analyzes source code and sends analysis results to the SonarQube server.

It can run from:

- Jenkins
- GitHub Actions
- GitLab CI
- Azure DevOps
- Local Developer Machines

---

# Question 44

## What is a Quality Gate?

### Answer

A Quality Gate is a set of predefined conditions that determine whether code meets organizational quality and security standards.

If the Quality Gate fails, the CI/CD pipeline should stop until the issues are resolved.

---

# Quality Gate Workflow

```text
Source Code

↓

Scan

↓

Quality Gate

├── PASS → Continue Pipeline

└── FAIL → Stop Pipeline
```

---

# Question 45

## What conditions are commonly included in a Quality Gate?

### Answer

Typical conditions include:

- No new Critical vulnerabilities
- No new Blocker vulnerabilities
- Minimum code coverage
- Maximum code duplication
- Security Rating
- Reliability Rating
- Maintainability Rating

---

# Question 46

## What is a Quality Profile?

### Answer

A Quality Profile is a collection of coding and security rules applied during code analysis.

Organizations typically create language-specific Quality Profiles for Java, Python, JavaScript, Go, C#, and other languages.

---

# Question 47

## What are Code Smells?

### Answer

Code Smells are maintainability issues that may not immediately break functionality but increase technical debt and make code more difficult to maintain.

Examples include:

- Duplicate code
- Long methods
- Unused variables
- Excessive complexity
- Dead code

---

# Question 48

## What is Technical Debt?

### Answer

Technical Debt represents the estimated effort required to fix maintainability issues identified during code analysis.

Reducing technical debt improves long-term software quality and maintainability.

---

# Question 49

## What is the difference between Bugs, Vulnerabilities, and Code Smells?

| Type | Description |
|------|-------------|
| Bugs | Defects that may cause incorrect application behaviour |
| Vulnerabilities | Security weaknesses that attackers can exploit |
| Code Smells | Maintainability issues that increase technical debt |

---

# Question 50

## What are Security Hotspots?

### Answer

Security Hotspots are pieces of code that require manual security review because they could become vulnerable depending on how they are implemented.

Unlike confirmed vulnerabilities, Security Hotspots require developer validation before remediation decisions are made.

---

# Question 51

## What is Branch Analysis in SonarQube?

### Answer

Branch Analysis allows separate analysis of feature branches, release branches, and the main branch.

This enables teams to identify security and quality issues before merging code into the primary branch.

---

# Question 52

## What is Pull Request Analysis?

### Answer

Pull Request Analysis scans only the changes introduced in a pull request and provides feedback directly to reviewers before the code is merged.

This prevents new security issues from entering the main branch.

---

# Pull Request Security Workflow

```text
Developer

↓

Feature Branch

↓

Pull Request

↓

Sonar Analysis

↓

Quality Gate

├── Pass

│      ↓

│    Merge

└── Fail

       ↓

Developer Fix

↓

Re-Scan

↓

Merge
```

---

# Question 53

## How do you integrate SonarQube into Jenkins?

### Answer

The typical workflow is:

```text
Git Checkout

↓

Build

↓

Unit Tests

↓

Sonar Scanner

↓

Quality Gate

↓

Continue or Stop Pipeline
```

The Jenkins pipeline waits for the Quality Gate result before proceeding to later stages such as container builds or deployments.

---

# Question 54

## Where should SonarQube be placed in a DevSecOps pipeline?

### Answer

SonarQube should run immediately after the application is built and unit tests complete, but before container images are created or deployments begin.

This ensures insecure code is identified before additional pipeline resources are consumed.

---

# Enterprise DevSecOps Pipeline with SonarQube

```text
Developer

↓

Git Push

↓

Checkout

↓

Build

↓

Unit Tests

↓

SonarQube Scan

↓

Quality Gate

↓

SCA

↓

Secret Scan

↓

IaC Scan

↓

Container Scan

↓

SBOM

↓

Image Signing

↓

Deploy
```

---

# Question 55

## How do you reduce false positives in SonarQube?

### Answer

False positives can be minimized by:

- Using appropriate Quality Profiles
- Updating rule sets regularly
- Reviewing Security Hotspots
- Customizing rules for organizational standards
- Suppressing findings only after documented review and approval

---

# Enterprise Best Practices

- Run SAST on every Pull Request.
- Block merges when Quality Gates fail.
- Customize Quality Profiles for each programming language.
- Treat Security Hotspots as mandatory review items.
- Review new vulnerabilities instead of only total vulnerabilities.
- Integrate SonarQube early in the CI/CD pipeline.
- Regularly update SonarQube and its rule sets.
- Combine SAST with SCA, DAST, IaC scanning, and container scanning.
- Continuously reduce technical debt rather than allowing it to accumulate.
- Monitor Quality Gate trends across all repositories.

---

# Software Composition Analysis (SCA) Interview Questions

---

# Question 56

## What is Software Composition Analysis (SCA)?

### Answer

Software Composition Analysis (SCA) is the process of identifying, analyzing, and managing third-party libraries, open-source dependencies, and their associated security vulnerabilities, license risks, and outdated versions.

---

# SCA Workflow

```text
Developer

↓

Build

↓

Dependency Resolution

↓

SCA Scan

↓

Vulnerability Database

↓

Risk Report

↓

Developer Fix

↓

Re-Scan

↓

Deploy
```

---

# Question 57

## Why is SCA important?

### Answer

Modern applications rely heavily on open-source components. A vulnerability in any dependency can compromise the entire application.

SCA helps organizations:

- Detect vulnerable libraries
- Reduce supply chain risks
- Ensure license compliance
- Maintain secure software
- Meet regulatory requirements

---

# Question 58

## What types of issues does SCA detect?

### Answer

SCA identifies:

- Known CVEs
- Vulnerable dependencies
- Outdated libraries
- License violations
- Transitive dependencies
- End-of-life packages
- Dependency conflicts

---

# Question 59

## What is a dependency?

### Answer

A dependency is an external library or package that an application requires to function.

Examples include:

- Maven packages
- npm packages
- Python pip modules
- Go modules
- NuGet packages

---

# Question 60

## What are direct and transitive dependencies?

### Answer

**Direct Dependency**

A library explicitly added by the developer.

**Transitive Dependency**

A library automatically installed because another dependency requires it.

---

# Dependency Example

```text
Application

↓

Spring Boot

↓

Logback

↓

SLF4J
```

In this example:

- Spring Boot → Direct Dependency
- Logback → Transitive Dependency
- SLF4J → Transitive Dependency

---

# Question 61

## What is a CVE?

### Answer

CVE (Common Vulnerabilities and Exposures) is a publicly assigned identifier for a known security vulnerability.

Example:

```text
CVE-2021-44228

(Log4Shell)
```

Every CVE has a unique identifier that enables consistent tracking across security tools and databases.

---

# Question 62

## What is CVSS?

### Answer

CVSS (Common Vulnerability Scoring System) is a standardized method for measuring the severity of security vulnerabilities.

---

# CVSS Severity Levels

| Score | Severity |
|--------|----------|
| 0.0 | None |
| 0.1–3.9 | Low |
| 4.0–6.9 | Medium |
| 7.0–8.9 | High |
| 9.0–10.0 | Critical |

---

# Question 63

## What is the National Vulnerability Database (NVD)?

### Answer

The National Vulnerability Database (NVD) is a publicly maintained repository of vulnerability information that includes CVEs, CVSS scores, affected products, and remediation guidance.

Many SCA tools use NVD data during analysis.

---

# Question 64

## What is a Vulnerability Database?

### Answer

A vulnerability database stores information about known software vulnerabilities.

Common sources include:

- NVD
- GitHub Security Advisories
- Vendor Security Advisories
- OS package repositories

SCA tools compare project dependencies against these databases.

---

# Question 65

## How does an SCA tool work?

### Answer

An SCA tool:

1. Detects project dependencies.
2. Builds the dependency tree.
3. Identifies versions.
4. Compares them with vulnerability databases.
5. Generates a vulnerability report.
6. Suggests remediation.

---

# SCA Detection Process

```text
Source Code

↓

Dependency Files

↓

Dependency Tree

↓

Version Detection

↓

Vulnerability Database

↓

Security Report
```

---

# Question 66

## What files are commonly scanned by SCA tools?

### Answer

Examples include:

| Technology | Dependency File |
|------------|-----------------|
| Maven | pom.xml |
| Gradle | build.gradle |
| npm | package.json |
| Python | requirements.txt |
| Go | go.mod |
| .NET | *.csproj |

---

# Question 67

## What is Dependency Confusion?

### Answer

Dependency Confusion is a supply chain attack where attackers publish malicious packages with the same name as internal packages to public repositories.

Build systems may accidentally download the malicious package.

---

# Dependency Confusion Attack

```text
Company

↓

Internal Package

↓

Build Server

↓

Public Repository

↓

Malicious Package

↓

Compromised Build
```

---

# Question 68

## How can Dependency Confusion be prevented?

### Answer

Organizations should:

- Use private package repositories.
- Configure repository priority.
- Reserve internal package names.
- Verify package sources.
- Review dependencies regularly.

---

# Question 69

## What are License Risks?

### Answer

Open-source software is distributed under different licenses.

Some licenses may impose restrictions on commercial distribution, modification, or redistribution.

SCA tools identify incompatible or restricted licenses.

---

# Common Open Source Licenses

| License | Characteristics |
|----------|-----------------|
| MIT | Permissive |
| Apache 2.0 | Permissive with patent protection |
| BSD | Permissive |
| GPL | Strong copyleft |
| LGPL | Limited copyleft |
| MPL | File-level copyleft |

---

# Question 70

## What is a False Positive in SCA?

### Answer

A false positive occurs when a tool reports a vulnerability that is not actually exploitable in the application's context.

False positives should be reviewed, documented, and approved before being ignored.

---

# Question 71

## What is Vulnerability Prioritization?

### Answer

Not every vulnerability requires immediate remediation.

Organizations prioritize based on:

- CVSS score
- Exploitability
- Internet exposure
- Business impact
- Availability of fixes
- Active exploitation

---

# Question 72

## What remediation options exist for vulnerable dependencies?

### Answer

Possible remediation approaches include:

- Upgrade the dependency.
- Replace the library.
- Apply vendor patches.
- Remove unused packages.
- Use temporary mitigations.
- Accept risk through documented approval.

---

# Question 73

## What is an SBOM?

### Answer

A Software Bill of Materials (SBOM) is a complete inventory of all software components, libraries, versions, and dependencies used to build an application.

SBOMs improve software supply chain visibility and support vulnerability management.

---

# SBOM Generation Workflow

```text
Application

↓

Dependencies

↓

SCA Tool

↓

SBOM

↓

Vulnerability Analysis

↓

Compliance Review
```

---

# Question 74

## How is SCA integrated into a DevSecOps pipeline?

### Answer

SCA is typically executed immediately after the build process and before container image creation.

Critical vulnerabilities should fail the pipeline according to organizational policy.

---

# Enterprise DevSecOps Pipeline

```text
Developer

↓

Git Push

↓

Build

↓

Unit Tests

↓

SonarQube

↓

SCA

↓

Secret Scan

↓

IaC Scan

↓

Container Scan

↓

SBOM

↓

Image Signing

↓

Deploy
```

---

# Question 75

## What are the best practices for Software Composition Analysis?

### Answer

Enterprise best practices include:

- Scan every build.
- Continuously monitor dependencies.
- Remove unused libraries.
- Upgrade dependencies regularly.
- Generate SBOMs for every release.
- Enforce vulnerability policies in CI/CD.
- Monitor transitive dependencies.
- Review license compliance.
- Prioritize remediation based on risk.
- Integrate SCA with vulnerability management and incident response processes.

---

# Enterprise Best Practices

- Never deploy applications with known critical vulnerabilities unless an approved exception exists.
- Maintain an inventory of all third-party components.
- Use trusted package repositories.
- Continuously monitor for newly disclosed CVEs affecting existing deployments.
- Combine SCA with SAST, DAST, IaC scanning, and container security.
- Regularly review dependency health and update strategies.
- Include SBOM generation as part of every production release.
- Implement automated policy gates based on vulnerability severity.
- Audit dependency licenses before introducing new libraries.
- Treat software supply chain security as a continuous process, not a one-time scan.

---

