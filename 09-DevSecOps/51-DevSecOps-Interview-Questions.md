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

# Trivy Interview Questions

---

# Question 76

## What is Trivy?

### Answer

Trivy is an open-source security scanner developed by Aqua Security that scans container images, file systems, Git repositories, Kubernetes clusters, Infrastructure as Code (IaC), and Software Bill of Materials (SBOM) for vulnerabilities, secrets, and misconfigurations.

---

# Trivy Architecture

```text
Application

↓

Container Image

↓

Trivy Scanner

↓

Vulnerability Database

↓

Security Report

↓

CI/CD Decision
```

---

# Question 77

## Why is Trivy widely used in DevSecOps?

### Answer

Trivy is popular because it:

- Is easy to install
- Supports multiple scan types
- Has fast scan times
- Integrates with CI/CD
- Detects vulnerabilities and misconfigurations
- Generates SBOMs
- Supports multiple output formats

---

# Question 78

## What can Trivy scan?

### Answer

Trivy supports scanning:

- Container Images
- File Systems
- Git Repositories
- Kubernetes Clusters
- Infrastructure as Code
- SBOM Files
- VM Images
- Root File Systems

---

# Question 79

## What security issues can Trivy detect?

### Answer

Trivy detects:

- OS package vulnerabilities
- Application dependency vulnerabilities
- Secrets
- IaC misconfigurations
- Kubernetes misconfigurations
- Dockerfile issues
- Sensitive files
- License information (SBOM workflows)

---

# Question 80

## Explain the Trivy scanning workflow.

### Answer

```text
Source Code

↓

Build

↓

Container Image

↓

Trivy Scan

↓

Vulnerability Database

↓

Security Report

↓

Fix Issues

↓

Re-Scan

↓

Deploy
```

---

# Question 81

## What vulnerability databases does Trivy use?

### Answer

Trivy downloads vulnerability information from trusted sources, including:

- National Vulnerability Database (NVD)
- GitHub Security Advisories
- Distribution-specific advisories
- Vendor security advisories

The database is updated regularly to detect newly disclosed vulnerabilities.

---

# Question 82

## What is Image Scanning?

### Answer

Image Scanning analyzes container images for known vulnerabilities in:

- Base operating system packages
- Installed libraries
- Application dependencies

This helps identify security issues before images are deployed.

---

# Question 83

## How do you scan a Docker image using Trivy?

### Answer

```bash
trivy image nginx:latest
```

Trivy analyzes the image layers and reports detected vulnerabilities.

---

# Question 84

## What is File System Scanning?

### Answer

File System Scanning analyzes local directories before container images are built.

It detects:

- Vulnerable dependencies
- Secrets
- Configuration issues

---

# Example

```bash
trivy fs .
```

---

# Question 85

## What is Repository Scanning?

### Answer

Repository Scanning analyzes source code repositories directly for vulnerabilities, secrets, and misconfigurations before the build process begins.

---

# Example

```bash
trivy repo https://github.com/example/project
```

---

# Question 86

## What is Kubernetes Scanning?

### Answer

Kubernetes Scanning reviews cluster resources for security issues such as:

- Privileged containers
- Missing security contexts
- Insecure RBAC
- Misconfigured workloads
- Exposed services

---

# Kubernetes Scan Workflow

```text
Cluster

↓

Pods

↓

Deployments

↓

RBAC

↓

Network Policies

↓

Security Analysis

↓

Report
```

---

# Question 87

## What is Infrastructure as Code (IaC) Scanning in Trivy?

### Answer

Trivy scans Infrastructure as Code templates to identify security misconfigurations before infrastructure is provisioned.

Supported technologies include:

- Terraform
- Kubernetes YAML
- Helm Charts
- Dockerfiles

---

# Question 88

## What is Secret Scanning?

### Answer

Secret Scanning identifies sensitive information accidentally stored in source code or configuration files.

Examples include:

- AWS Access Keys
- API Keys
- Database Passwords
- SSH Private Keys
- Tokens
- Certificates

---

# Secret Scan Workflow

```text
Repository

↓

Files

↓

Secret Detection

↓

Security Report

↓

Developer

↓

Remove Secret

↓

Rotate Credentials
```

---

# Question 89

## What are Misconfigurations?

### Answer

Misconfigurations are insecure settings that increase security risk even if no software vulnerabilities exist.

Examples include:

- Privileged containers
- Public S3 buckets
- Root containers
- Weak IAM permissions
- Missing encryption
- Public databases

---

# Question 90

## What severity levels does Trivy use?

### Answer

Trivy classifies findings into:

| Severity | Description |
|----------|-------------|
| Unknown | Severity not available |
| Low | Low risk |
| Medium | Moderate risk |
| High | Significant risk |
| Critical | Immediate attention required |

---

# Question 91

## How should organizations prioritize Trivy findings?

### Answer

Prioritization should consider:

- Severity
- Exploitability
- Internet exposure
- Business impact
- Availability of patches
- Active exploitation

Critical findings affecting production-facing systems should receive the highest priority.

---

# Question 92

## Can Trivy fail a CI/CD pipeline?

### Answer

Yes.

Organizations commonly configure Trivy to fail the pipeline when vulnerabilities exceed predefined severity thresholds.

Example policy:

```text
Critical Vulnerability

↓

Pipeline Failed

↓

Developer Fix

↓

Re-Scan

↓

Pipeline Continues
```

---

# Question 93

## Where should Trivy be placed in a DevSecOps pipeline?

### Answer

Trivy should execute after the application or container image is built but before deployment.

Typical placement:

```text
Build

↓

SonarQube

↓

SCA

↓

Trivy

↓

SBOM

↓

Image Signing

↓

Artifact Repository

↓

Deployment
```

---

# Question 94

## What output formats does Trivy support?

### Answer

Trivy supports multiple output formats including:

- Table
- JSON
- SARIF
- CycloneDX
- SPDX
- Template-based reports

These formats support CI/CD systems, dashboards, and compliance workflows.

---

# Question 95

## How does Trivy integrate with Jenkins, GitHub Actions, and GitLab CI?

### Answer

Trivy can be executed as a pipeline stage after the build process.

If the scan detects vulnerabilities exceeding the configured policy threshold, the pipeline can stop automatically before deployment.

---

# Enterprise Trivy Pipeline

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

Trivy Image Scan

↓

Secret Scan

↓

SBOM

↓

Cosign

↓

Artifact Repository

↓

GitOps

↓

Production
```

---

# Enterprise Best Practices

- Scan every container image before pushing it to the registry.
- Scan source code repositories for secrets before building artifacts.
- Scan Infrastructure as Code before provisioning resources.
- Keep Trivy's vulnerability database updated.
- Block deployments containing critical vulnerabilities unless an approved exception exists.
- Generate SBOMs for every production release.
- Prioritize remediation based on exploitability and business impact.
- Integrate Trivy with Jenkins, GitHub Actions, and GitLab CI.
- Combine Trivy with SonarQube, SCA, Checkov, Gitleaks, and runtime security tools for comprehensive coverage.
- Continuously monitor deployed workloads for newly disclosed vulnerabilities.

---

# OWASP Dependency-Check Interview Questions

---

# Question 116

## What is OWASP Dependency-Check?

### Answer

OWASP Dependency-Check is an open-source Software Composition Analysis (SCA) tool that identifies publicly disclosed vulnerabilities in project dependencies by comparing them against vulnerability databases such as the National Vulnerability Database (NVD).

---

# Dependency-Check Workflow

```text
Application

↓

Dependency Files

↓

Dependency Analysis

↓

NVD Database

↓

CVE Matching

↓

Security Report

↓

Developer Fix
```

---

# Question 117

## Why do organizations use OWASP Dependency-Check?

### Answer

Organizations use OWASP Dependency-Check to:

- Detect vulnerable libraries
- Reduce software supply chain risks
- Automate dependency analysis
- Improve compliance
- Prevent vulnerable software from reaching production

---

# Question 118

## Which dependency files can OWASP Dependency-Check analyze?

### Answer

It supports many package managers including:

| Technology | Dependency File |
|------------|-----------------|
| Maven | pom.xml |
| Gradle | build.gradle |
| npm | package-lock.json |
| Yarn | yarn.lock |
| Python | requirements.txt |
| Go | go.mod |
| .NET | packages.config |

---

# Question 119

## What security issues does OWASP Dependency-Check identify?

### Answer

It identifies:

- Known CVEs
- Vulnerable dependencies
- Outdated libraries
- Transitive dependency risks
- Vulnerability severity
- Remediation recommendations

---

# Question 120

## How does OWASP Dependency-Check work?

### Answer

The tool:

1. Reads dependency manifests.
2. Builds a dependency inventory.
3. Matches package versions with known CVEs.
4. Calculates risk based on CVSS scores.
5. Generates detailed vulnerability reports.

---

# Detection Workflow

```text
Dependency Files

↓

Package Identification

↓

Version Detection

↓

NVD Lookup

↓

CVE Match

↓

Risk Report
```

---

# Question 121

## Does OWASP Dependency-Check perform SAST?

### Answer

No.

Dependency-Check does not analyze application source code. It performs Software Composition Analysis (SCA) by evaluating third-party libraries and dependencies.

---

# Question 122

## Does OWASP Dependency-Check perform DAST?

### Answer

No.

It does not test running applications. Runtime security testing requires DAST tools such as OWASP ZAP.

---

# Question 123

## What is the difference between Dependency-Check and Trivy?

| OWASP Dependency-Check | Trivy |
|-------------------------|--------|
| Primarily SCA | Multi-purpose security scanner |
| Focuses on dependencies | Scans images, IaC, Kubernetes, secrets and dependencies |
| No container image scanning | Supports container image scanning |
| No Kubernetes scanning | Supports Kubernetes scanning |
| Open-source | Open-source |

---

# Question 124

## What is the difference between Dependency-Check and Veracode SCA?

### Answer

Both identify vulnerable dependencies.

Dependency-Check is an open-source tool primarily focused on dependency vulnerability detection, while Veracode SCA provides enterprise features such as centralized management, policy enforcement, compliance reporting, and broader application security integration.

---

# Question 125

## Which vulnerability databases are used?

### Answer

Common sources include:

- National Vulnerability Database (NVD)
- CISA advisories
- Vendor security advisories
- CVE database

These databases are updated regularly with newly disclosed vulnerabilities.

---

# Question 126

## What is CVE matching?

### Answer

Dependency-Check compares detected package names and versions against known CVE records to determine whether a dependency is affected by published vulnerabilities.

---

# CVE Matching

```text
Dependency

↓

Package Version

↓

NVD Search

↓

Matching CVE

↓

Risk Score

↓

Security Report
```

---

# Question 127

## What reports can OWASP Dependency-Check generate?

### Answer

It supports multiple report formats including:

- HTML
- JSON
- XML
- JUnit
- SARIF

These reports can be integrated with CI/CD pipelines and security dashboards.

---

# Question 128

## Can OWASP Dependency-Check fail a CI/CD pipeline?

### Answer

Yes.

Organizations commonly configure pipelines to fail when vulnerabilities exceed approved severity thresholds.

Example:

```text
Critical CVE Found

↓

Pipeline Failed

↓

Developer Fix

↓

Re-Scan

↓

Continue Pipeline
```

---

# Question 129

## Where should Dependency-Check run in a DevSecOps pipeline?

### Answer

Dependency-Check should execute immediately after the application build and before container image creation or deployment.

This prevents vulnerable dependencies from progressing through later pipeline stages.

---

# Enterprise Pipeline

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

Dependency-Check

↓

Trivy

↓

Secrets Scan

↓

IaC Scan

↓

SBOM

↓

Deploy
```

---

# Question 130

## What are false positives in Dependency-Check?

### Answer

False positives occur when a reported CVE does not actually affect the application because the vulnerable code path is not used or the package identification is incorrect.

Each finding should be reviewed before suppression.

---

# Question 131

## How should vulnerabilities be prioritized?

### Answer

Organizations should prioritize based on:

- CVSS severity
- Business impact
- Internet exposure
- Exploit availability
- Criticality of the application
- Availability of patches

Critical exploitable vulnerabilities should be remediated first.

---

# Question 132

## What are suppression files?

### Answer

Suppression files allow organizations to exclude known false positives after proper security review and documentation.

Suppressions should be reviewed regularly and removed when no longer required.

---

# Question 133

## How do you integrate Dependency-Check with Jenkins?

### Answer

A common workflow is:

```text
Checkout

↓

Build

↓

Dependency-Check

↓

Generate Report

↓

Evaluate Severity

├── Pass

│      ↓

│ Continue

└── Fail

       ↓

Developer Fix
```

The generated reports can be archived and used for audit and compliance purposes.

---

# Question 134

## How does Dependency-Check support compliance?

### Answer

Dependency-Check helps organizations demonstrate that open-source dependencies are regularly scanned for known vulnerabilities.

Its reports support compliance initiatives such as:

- PCI DSS
- ISO 27001
- SOC 2
- NIST Secure Software Development Framework (SSDF)

---

# Question 135

## What are enterprise best practices for OWASP Dependency-Check?

### Answer

Organizations should:

- Scan every build.
- Keep vulnerability databases updated.
- Fail builds on critical vulnerabilities.
- Review suppression files periodically.
- Monitor transitive dependencies.
- Integrate reports into vulnerability management.
- Combine Dependency-Check with SAST, DAST, IaC scanning, and container scanning.
- Continuously monitor dependencies after deployment.
- Generate reports for compliance audits.
- Re-scan applications whenever dependencies are upgraded.

---

# Enterprise Best Practices

- Scan every application build automatically.
- Update vulnerability databases before scanning.
- Treat critical CVEs as deployment blockers unless formally approved.
- Review and document every suppression rule.
- Monitor both direct and transitive dependencies.
- Combine Dependency-Check with SonarQube, Trivy, Veracode, Checkov, and Gitleaks for layered security.
- Maintain an inventory of all third-party components.
- Integrate reports into enterprise SIEM and vulnerability management platforms.
- Continuously assess dependencies as new CVEs are published.
- Make dependency security an ongoing process rather than a one-time activity.

---

# Checkov Interview Questions

---

# Question 136

## What is Checkov?

### Answer

Checkov is an open-source Infrastructure as Code (IaC) security scanner developed by Bridgecrew (now part of Palo Alto Networks). It scans infrastructure code for security misconfigurations before resources are deployed.

---

# Checkov Workflow

```text
Terraform

↓

CloudFormation

↓

Kubernetes YAML

↓

Dockerfile

↓

Checkov Scan

↓

Policy Engine

↓

Security Report

↓

Developer Fix

↓

Deploy
```

---

# Question 137

## Why is Checkov important in DevSecOps?

### Answer

Checkov helps organizations detect security misconfigurations during development instead of after infrastructure is deployed.

This supports the Shift Left security approach by preventing insecure infrastructure from reaching production.

---

# Question 138

## What Infrastructure as Code technologies does Checkov support?

### Answer

Checkov supports scanning:

- Terraform
- Terraform Plan
- CloudFormation
- Kubernetes YAML
- Helm Charts
- Dockerfiles
- GitHub Actions
- GitLab CI
- Azure ARM Templates
- Bicep

---

# Question 139

## What types of security issues can Checkov detect?

### Answer

Checkov identifies:

- Public cloud resources
- Weak IAM policies
- Missing encryption
- Security group misconfigurations
- Unrestricted network access
- Missing logging
- Kubernetes security issues
- CI/CD pipeline misconfigurations

---

# Question 140

## How does Checkov work?

### Answer

Checkov parses Infrastructure as Code files, evaluates them against predefined or custom security policies, and generates a report highlighting policy violations.

---

# Checkov Scan Process

```text
IaC Files

↓

Parser

↓

Policy Evaluation

↓

Passed Checks

↓

Failed Checks

↓

Security Report
```

---

# Question 141

## Which cloud platforms are supported by Checkov?

### Answer

Checkov supports scanning infrastructure for:

- AWS
- Azure
- Google Cloud Platform (GCP)
- Kubernetes
- Multi-cloud environments

---

# Question 142

## What are Policies in Checkov?

### Answer

Policies are predefined security rules used to evaluate Infrastructure as Code.

Each policy checks whether infrastructure follows security best practices.

Example:

- S3 bucket encryption enabled
- Security groups not open to the internet
- EBS volumes encrypted
- IAM least privilege

---

# Question 143

## What is Policy-as-Code?

### Answer

Policy-as-Code is the practice of defining security and compliance requirements as version-controlled code.

This enables automated policy validation within CI/CD pipelines.

---

# Policy-as-Code Workflow

```text
Infrastructure Code

↓

Security Policies

↓

Policy Engine

↓

Pass

or

Fail
```

---

# Question 144

## Can organizations create custom policies in Checkov?

### Answer

Yes.

Organizations can create custom Python or YAML-based policies to enforce internal security standards and compliance requirements beyond the built-in checks.

---

# Question 145

## What are Check IDs?

### Answer

Each Checkov security policy has a unique identifier called a Check ID.

Example:

```text
CKV_AWS_20

CKV_K8S_21

CKV_DOCKER_2
```

These identifiers help teams reference and manage specific policy violations.

---

# Question 146

## What severity levels are used in Checkov?

### Answer

Checkov integrates with policy metadata to classify findings by severity, typically including:

- Low
- Medium
- High
- Critical

Organizations often use these severity levels to determine whether a pipeline should continue.

---

# Question 147

## What is Terraform Plan Scanning?

### Answer

Terraform Plan Scanning analyzes the execution plan generated by Terraform before infrastructure changes are applied.

This helps identify security issues based on the actual planned infrastructure rather than only the source code.

---

# Terraform Plan Workflow

```text
Terraform Code

↓

terraform plan

↓

Plan File

↓

Checkov

↓

Policy Validation

↓

Apply
```

---

# Question 148

## What Kubernetes resources can Checkov scan?

### Answer

Checkov can analyze Kubernetes manifests such as:

- Pods
- Deployments
- Services
- StatefulSets
- DaemonSets
- Ingress
- Network Policies
- RBAC Resources

---

# Question 149

## What Kubernetes misconfigurations can Checkov detect?

### Answer

Examples include:

- Privileged containers
- Missing resource limits
- Missing security contexts
- Running as root
- HostPath usage
- Missing Network Policies
- Privilege escalation enabled
- Insecure capabilities

---

# Question 150

## Can Checkov scan Dockerfiles?

### Answer

Yes.

Checkov evaluates Dockerfiles for insecure practices such as:

- Running containers as root
- Using latest tags
- Missing HEALTHCHECK instructions
- Excessive privileges
- Insecure base images

---

# Question 151

## Can Checkov scan CI/CD pipelines?

### Answer

Yes.

Checkov supports scanning CI/CD configuration files including GitHub Actions and GitLab CI to detect insecure pipeline configurations before execution.

---

# Question 152

## How is Checkov integrated into a CI/CD pipeline?

### Answer

A typical workflow is:

```text
Git Push

↓

Build

↓

SonarQube

↓

Dependency Scan

↓

Checkov

↓

Trivy

↓

Deploy
```

If policy violations exceed organizational thresholds, the pipeline should stop until remediation is complete.

---

# Question 153

## What output formats does Checkov support?

### Answer

Common output formats include:

- CLI
- JSON
- JUnit XML
- SARIF
- CycloneDX
- GitHub-compatible reports

These formats integrate with CI/CD platforms and security dashboards.

---

# Question 154

## How should Checkov findings be prioritized?

### Answer

Organizations should prioritize findings based on:

- Severity
- Internet exposure
- Business impact
- Compliance requirements
- Resource criticality
- Exploitability

Critical issues affecting production infrastructure should be addressed first.

---

# Question 155

## What are enterprise best practices for Checkov?

### Answer

Organizations should:

- Scan every Infrastructure as Code change.
- Scan Pull Requests before merging.
- Block deployments for critical policy violations.
- Use custom policies for organizational standards.
- Scan Terraform Plans in addition to Terraform code.
- Integrate Checkov with Jenkins, GitHub Actions, and GitLab CI.
- Review policy exceptions regularly.
- Combine Checkov with Terraform, Trivy, Gitleaks, and runtime security.
- Maintain version-controlled security policies.
- Continuously improve security baselines.

---

# Enterprise DevSecOps Pipeline with Checkov

```text
Developer

↓

Git Push

↓

Pull Request

↓

Build

↓

SonarQube

↓

Dependency-Check

↓

Checkov

↓

Terraform Plan

↓

Trivy

↓

SBOM

↓

Image Signing

↓

Artifact Repository

↓

GitOps

↓

Amazon EKS

↓

Production
```

---

# Enterprise Best Practices

- Shift infrastructure security to the earliest stages of development.
- Scan every Pull Request containing Infrastructure as Code.
- Maintain custom security policies for enterprise compliance.
- Enforce Policy-as-Code across all cloud platforms.
- Fail CI/CD pipelines for critical infrastructure misconfigurations.
- Continuously update Checkov to receive new security policies.
- Combine Checkov with TFSec, Trivy, Gitleaks, SonarQube, and Veracode for comprehensive DevSecOps coverage.
- Review exceptions through formal security approval processes.
- Store scan reports for auditing and compliance evidence.
- Re-scan infrastructure whenever templates or policies are updated.

---

# TFSec Interview Questions

---

# Question 156

## What is TFSec?

### Answer

TFSec is an open-source Infrastructure as Code (IaC) security scanner specifically designed for Terraform. It analyzes Terraform code to identify security misconfigurations before infrastructure is provisioned.

---

# TFSec Workflow

```text
Terraform Code

↓

TFSec

↓

Security Policies

↓

Misconfiguration Detection

↓

Security Report

↓

Developer Fix

↓

Deploy
```

---

# Question 157

## Why is TFSec used in DevSecOps?

### Answer

TFSec enables organizations to identify infrastructure security issues during development instead of after deployment. This reduces security risks and supports Shift Left security practices.

---

# Question 158

## What types of Terraform resources can TFSec scan?

### Answer

TFSec scans almost every Terraform resource, including:

- AWS Resources
- Azure Resources
- Google Cloud Resources
- Kubernetes Resources
- Network Resources
- IAM Resources
- Storage Resources
- Databases
- Compute Resources

---

# Question 159

## What security issues can TFSec detect?

### Answer

TFSec identifies:

- Public S3 buckets
- Unencrypted storage
- Weak IAM permissions
- Public databases
- Open Security Groups
- Missing logging
- Disabled encryption
- Insecure Kubernetes configurations

---

# Question 160

## How does TFSec work?

### Answer

TFSec parses Terraform configuration files, evaluates them against built-in security policies, and reports policy violations before infrastructure is deployed.

---

# TFSec Scan Process

```text
Terraform Files

↓

Terraform Parser

↓

Security Policies

↓

Policy Evaluation

↓

Passed Checks

↓

Failed Checks

↓

Security Report
```

---

# Question 161

## Which cloud providers are supported by TFSec?

### Answer

TFSec supports Terraform resources for:

- AWS
- Microsoft Azure
- Google Cloud Platform (GCP)
- Kubernetes

Support depends on the Terraform providers and available security rules.

---

# Question 162

## What are built-in security checks in TFSec?

### Answer

Built-in security checks validate common cloud security best practices.

Examples include:

- Encryption enabled
- Logging enabled
- Public access disabled
- Secure IAM policies
- Secure networking
- Least privilege
- Resource protection

---

# Question 163

## Can TFSec detect compliance violations?

### Answer

Yes.

TFSec helps identify infrastructure configurations that may violate internal security policies and industry compliance standards by detecting insecure Terraform configurations.

---

# Question 164

## What are false positives in TFSec?

### Answer

False positives occur when TFSec reports a potential security issue that is acceptable in the application's context or is mitigated through other security controls.

Every suppression should be reviewed and documented.

---

# Question 165

## Can TFSec scan Terraform modules?

### Answer

Yes.

TFSec evaluates both root Terraform configurations and reusable modules to identify security issues throughout the infrastructure codebase.

---

# Module Scan Workflow

```text
Terraform Root Module

↓

Child Modules

↓

TFSec

↓

Security Analysis

↓

Report
```

---

# Question 166

## Can TFSec scan Terraform Plans?

### Answer

TFSec primarily analyzes Terraform configuration files before execution.

For evaluating the planned infrastructure state, many organizations combine TFSec with Terraform Plan analysis and tools such as Checkov.

---

# Question 167

## What are common AWS security checks performed by TFSec?

### Answer

Examples include:

- S3 encryption enabled
- S3 public access blocked
- Security Groups restricted
- EBS encryption enabled
- RDS encryption enabled
- IAM least privilege
- CloudTrail enabled
- KMS encryption configured

---

# Question 168

## What are common Kubernetes security checks?

### Answer

TFSec can identify Terraform-managed Kubernetes issues such as:

- Privileged containers
- Containers running as root
- Missing Network Policies
- Missing resource limits
- Weak RBAC configurations
- Insecure security contexts

---

# Question 169

## Can TFSec be integrated into CI/CD?

### Answer

Yes.

TFSec integrates with:

- Jenkins
- GitHub Actions
- GitLab CI
- Azure DevOps
- Bitbucket Pipelines

Security checks execute automatically before infrastructure deployment.

---

# Question 170

## Where should TFSec run in a DevSecOps pipeline?

### Answer

TFSec should run after source code checkout and before Terraform Plan or Terraform Apply.

This prevents insecure infrastructure from being provisioned.

---

# Enterprise Pipeline

```text
Developer

↓

Git Push

↓

Pull Request

↓

Build

↓

SonarQube

↓

Dependency Scan

↓

Checkov

↓

TFSec

↓

Terraform Plan

↓

Terraform Apply

↓

Deploy
```

---

# Question 171

## What output formats does TFSec support?

### Answer

TFSec supports multiple output formats, including:

- CLI
- JSON
- SARIF
- JUnit
- CSV
- HTML

These reports integrate with security dashboards and CI/CD platforms.

---

# Question 172

## How should TFSec findings be prioritized?

### Answer

Organizations should prioritize findings based on:

- Severity
- Resource exposure
- Business impact
- Compliance requirements
- Environment (Development, Test, Production)
- Ease of exploitation

Critical production issues should be addressed immediately.

---

# Question 173

## What is the difference between TFSec and Checkov?

| TFSec | Checkov |
|-------|----------|
| Focuses primarily on Terraform | Supports multiple IaC technologies |
| Terraform-specific policies | Multi-cloud and multi-framework support |
| Lightweight Terraform scanner | Broader Policy-as-Code platform |
| Strong Terraform security checks | Terraform, Kubernetes, Dockerfile, CI/CD and more |

---

# Question 174

## Can TFSec fail a CI/CD pipeline?

### Answer

Yes.

Organizations commonly configure TFSec to fail builds when high or critical infrastructure security issues are detected.

Example:

```text
Critical Finding

↓

Pipeline Failed

↓

Developer Fix

↓

Re-Scan

↓

Continue Pipeline
```

---

# Question 175

## What are enterprise best practices for TFSec?

### Answer

Organizations should:

- Scan every Terraform change.
- Scan Pull Requests before merging.
- Prevent Terraform Apply when critical findings exist.
- Combine TFSec with Checkov for broader IaC coverage.
- Integrate TFSec into all CI/CD pipelines.
- Review exceptions through formal approval.
- Keep security policies updated.
- Store reports for compliance evidence.
- Continuously improve Terraform security baselines.
- Re-scan infrastructure after every code change.

---

# Enterprise DevSecOps Pipeline with TFSec

```text
Developer

↓

Git Push

↓

Pull Request

↓

Build

↓

SonarQube

↓

Dependency-Check

↓

Checkov

↓

TFSec

↓

Terraform Plan

↓

Terraform Apply

↓

Trivy

↓

SBOM

↓

Image Signing

↓

Artifact Repository

↓

GitOps

↓

Amazon EKS

↓

Production
```

---

# Enterprise Best Practices

- Run TFSec during every Infrastructure as Code pipeline.
- Block infrastructure deployments containing critical security issues.
- Combine TFSec with Checkov to improve Terraform security coverage.
- Scan reusable Terraform modules as well as root modules.
- Integrate TFSec findings into enterprise vulnerability management.
- Use version-controlled security policies and review them regularly.
- Continuously update TFSec to receive new security checks.
- Store scan reports for audit and compliance purposes.
- Validate all remediations with re-scans before deployment.
- Treat Infrastructure as Code security as a mandatory stage in every DevSecOps pipeline.

---

