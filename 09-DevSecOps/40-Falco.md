# Falco

## Introduction

Falco is an open-source Cloud Native Computing Foundation (CNCF) runtime security tool that continuously monitors Linux hosts, containers, Kubernetes clusters, and cloud workloads for suspicious behaviour.

Unlike static security tools that scan code before deployment, Falco detects security threats while applications are running by monitoring Linux system calls (syscalls), Kubernetes audit events, and container runtime events.

Falco enables Security Operations (SecOps), DevSecOps, and Platform Engineering teams to detect attacks, policy violations, privilege escalation, container escapes, and suspicious processes in real time.

---

# Why Companies Use Falco

Even if an application passes SAST, SCA, IaC, Container, and DAST scans, it can still be compromised after deployment.

Falco provides runtime protection by continuously monitoring workloads and generating alerts when suspicious activity occurs.

## Benefits

- Runtime Threat Detection
- Container Security Monitoring
- Kubernetes Runtime Security
- Linux Host Monitoring
- Syscall Analysis
- Kubernetes Audit Monitoring
- Compliance Monitoring
- Real-time Alerting
- Policy-based Detection
- SIEM Integration

---

# Shift-Left vs Shift-Right Security

Modern DevSecOps requires both pre-deployment and runtime security.

| Stage | Security Tool |
|---------|---------------|
| Source Code | SonarQube |
| Dependencies | OWASP Dependency-Check |
| SAST | Veracode |
| Secrets | Gitleaks |
| Terraform | Checkov / TFSec |
| Containers | Trivy |
| Running Workloads | **Falco** |
| Monitoring | Prometheus + Grafana |

Falco complements earlier security stages by protecting running workloads.

---

# Where Falco Fits in DevSecOps

Falco runs after workloads have been deployed.

```text
Developer

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Pipeline

↓

Build

↓

Security Scans

↓

Docker Build

↓

Container Scan

↓

Deploy

↓

Amazon EKS

↓

Falco Runtime Monitoring

↓

Alerts

↓

SOC Team
```

Falco continuously monitors workloads after deployment.

---

# Enterprise Architecture

```text
                     Developers
                          │
                          ▼
                   Git Repository
                          │
                          ▼
               Jenkins / GitHub Actions
                          │
                          ▼
                  Build & Security
                          │
                          ▼
                     Amazon EKS
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
     Worker Node     Worker Node     Worker Node
          │               │               │
          ▼               ▼               ▼
       Falco Pod       Falco Pod       Falco Pod
          │               │               │
          └───────────────┼───────────────┘
                          ▼
                    Alert Manager
                          │
      ┌───────────┬────────────┬────────────┐
      ▼           ▼            ▼
   Slack       SIEM       Prometheus
```

---

# Production Architecture

```text
Developer

↓

GitHub Enterprise

↓

Jenkins

↓

Docker Build

↓

Amazon ECR

↓

ArgoCD

↓

Amazon EKS

↓

Falco DaemonSet

↓

Security Alerts

↓

SOC Team
```

Falco runs on every Kubernetes node as a DaemonSet to monitor all containers.

---

# Runtime Security vs Static Security

| Static Security | Runtime Security |
|-----------------|------------------|
| Before Deployment | After Deployment |
| Source Code | Running Processes |
| Docker Image | Live Containers |
| Terraform | Linux Syscalls |
| Build Pipeline | Runtime Behaviour |
| Preventive | Detective |

Both approaches are essential for enterprise security.

---

# What Can Falco Detect?

| Threat | Supported |
|----------|-----------|
| Container Escape | ✓ |
| Privilege Escalation | ✓ |
| Reverse Shell | ✓ |
| Crypto Mining | ✓ |
| Suspicious Shell | ✓ |
| Sensitive File Access | ✓ |
| Kubernetes Exec | ✓ |
| Network Connections | ✓ |
| File Modifications | ✓ |
| Unexpected Processes | ✓ |

---

# How Falco Works

Falco continuously observes kernel activity.

```text
Application

↓

Linux Kernel

↓

Syscalls

↓

Falco Engine

↓

Rules Engine

↓

Alert
```

Every syscall is evaluated against Falco detection rules.

---

# Prerequisites

| Component | Version |
|------------|----------|
| Kubernetes | 1.28+ |
| Helm | Latest |
| Docker | Latest |
| Linux | Supported |
| Amazon EKS | Supported |
| AKS | Supported |
| GKE | Supported |

---

# Installation Methods

Falco can be installed using:

- Helm
- Kubernetes Manifest
- Docker
- Linux Package
- Amazon EKS
- AKS
- GKE

Helm is the recommended installation method for production Kubernetes clusters.

---

# Add Helm Repository

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts

helm repo update
```

---

# Install Falco

```bash
helm install falco falcosecurity/falco \
--namespace falco \
--create-namespace
```

---

# Verify Installation

Check pods.

```bash
kubectl get pods -n falco
```

Expected output.

```text
falco-xxxxx

Running
```

Check DaemonSet.

```bash
kubectl get daemonset -n falco
```

Every worker node should have one Falco pod.

---

# Install on Amazon EKS

Deploy using Helm.

```bash
helm install falco falcosecurity/falco \
--namespace falco \
--create-namespace
```

Verify.

```bash
kubectl get pods -n falco
```

Falco automatically monitors workloads running on every EKS worker node.

---

# Install on AKS

```bash
helm install falco falcosecurity/falco \
--namespace falco \
--create-namespace
```

Verify installation.

```bash
kubectl get pods -n falco
```

---

# Install on Google Kubernetes Engine

```bash
helm install falco falcosecurity/falco \
--namespace falco \
--create-namespace
```

Verify.

```bash
kubectl get pods -n falco
```

---

# Verify Runtime Monitoring

Generate a test event.

```bash
kubectl exec -it nginx -- sh
```

Inside the container.

```bash
cat /etc/shadow
```

Falco detects the suspicious activity and generates an alert according to the active rules.

---

# First Alert Workflow

```text
Container

↓

Suspicious Activity

↓

Linux Syscall

↓

Falco Rule Match

↓

Security Alert

↓

SOC Team Notification
```

This confirms that Falco is actively monitoring runtime activity across the Kubernetes cluster.

---

# Configuration

Falco is highly configurable and allows organizations to customize runtime security policies based on their infrastructure and compliance requirements.

Configuration can include:

- Rules
- Custom Rules
- Outputs
- Priorities
- Kubernetes Audit Events
- Plugins
- Alert Destinations
- Exceptions

A centralized configuration ensures consistent runtime monitoring across all Kubernetes clusters.

---

# Configuration Files

The primary configuration files are:

```text
/etc/falco/

├── falco.yaml

├── falco_rules.yaml

├── falco_rules.local.yaml

└── config.d/
```

Production environments should store custom rules separately from the default rules.

---

# Main Configuration File

Example.

```bash
vi /etc/falco/falco.yaml
```

Example configuration.

```yaml
priority: notice

json_output: true

json_include_output_property: true

buffered_outputs: false
```

Restart Falco after configuration changes.

---

# Verify Configuration

Validate configuration.

```bash
falco --validate
```

Example output.

```text
Configuration Valid

↓

Falco Started
```

---

# Configuration Components

| Component | Purpose |
|-----------|----------|
| falco.yaml | Main configuration |
| Rules | Runtime detection policies |
| Outputs | Alert destinations |
| Plugins | Additional event sources |
| Priority | Alert severity |
| Exceptions | Ignore trusted activity |

---

# Rule Files

Default rules.

```text
/etc/falco/falco_rules.yaml
```

Custom rules.

```text
/etc/falco/falco_rules.local.yaml
```

Never modify the default rule file directly.

---

# Rule Structure

A Falco rule contains:

- Rule Name
- Description
- Condition
- Output
- Priority
- Tags

Example.

```text
Rule

↓

Condition

↓

Event Match

↓

Alert
```

---

# Example Custom Rule

```yaml
- rule: Shell Inside Container

  desc: Detect shell execution

  condition: container and shell_procs

  output: >

    Shell executed inside container

  priority: WARNING
```

Reload Falco after adding new rules.

---

# Rule Priority Levels

| Priority | Usage |
|-----------|-------|
| Debug | Development |
| Informational | Audit |
| Notice | Normal Events |
| Warning | Suspicious Activity |
| Error | High Risk |
| Critical | Immediate Response |
| Emergency | Severe Incident |

Production alerts usually begin at **Warning** level.

---

# JSON Output

Enable JSON output.

```yaml
json_output: true
```

Example.

```json
{

  "priority":"Warning",

  "rule":"Shell Inside Container",

  "container":"nginx",

  "user":"root"

}
```

JSON simplifies integration with SIEM platforms.

---

# Log Output

Store alerts locally.

```yaml
file_output:

  enabled: true

  filename: /var/log/falco.log
```

Verify.

```bash
tail -f /var/log/falco.log
```

---

# Standard Output

Enable console output.

```yaml
stdout_output:

  enabled: true
```

Useful during development and troubleshooting.

---

# Syslog Output

Forward alerts to Syslog.

```yaml
syslog_output:

  enabled: true
```

Suitable for enterprise logging infrastructure.

---

# Kubernetes Audit Events

Falco can monitor Kubernetes API activity.

Workflow.

```text
Kubernetes API

↓

Audit Event

↓

Falco

↓

Policy Evaluation

↓

Alert
```

Examples include:

- Pod creation
- Secret access
- RBAC changes
- Namespace deletion

---

# Plugins

Falco supports plugins for additional event sources.

Examples.

- Kubernetes Audit Logs
- CloudTrail
- JSON Events
- Custom Plugins

Plugins extend Falco beyond Linux syscalls.

---

# Alert Destinations

Falco supports multiple alert outputs.

```text
Falco

↓

Alert

├── File

├── Stdout

├── Syslog

├── Slack

├── Webhook

├── SIEM

└── Prometheus
```

Multiple outputs can be enabled simultaneously.

---

# Ignore Trusted Processes

Example exception.

```yaml
append: true

exceptions:

- name: trusted-shell

  fields:

    - proc.name

  comps:

    - =

  values:

    - bash
```

Exceptions should be approved by the security team.

---

# Rule Tags

Rules can be categorized.

Example.

```yaml
tags:

- container

- kubernetes

- mitre_execution
```

Tags simplify filtering and compliance reporting.

---

# Enable Metrics

Expose Prometheus metrics.

```yaml
metrics:

  enabled: true
```

Metrics help monitor Falco health and alert volumes.

---

# Enterprise Best Practices

- Keep custom rules in `falco_rules.local.yaml`.
- Do not modify default rule files.
- Enable JSON output for SIEM integration.
- Store alerts centrally.
- Monitor Kubernetes audit events.
- Review rule exceptions regularly.
- Enable Prometheus metrics.
- Test new rules before production rollout.
- Keep Falco updated with the latest rule set.
- Version-control all custom configurations.

---

# Understanding Falco Rules

Falco detects threats by evaluating runtime events against predefined rules.

Rule evaluation flow.

```text
Linux Syscall

↓

Falco Engine

↓

Rule Evaluation

↓

Condition Matched?

      │

 ┌────┴────┐

 ▼         ▼

Yes        No

 │          │

Alert    Continue
```

---

# Anatomy of a Falco Rule

A Falco rule contains several components.

| Component | Purpose |
|-----------|----------|
| Rule | Rule name |
| Description | Rule explanation |
| Condition | Detection logic |
| Output | Alert message |
| Priority | Alert severity |
| Tags | Classification |

Example.

```yaml
- rule: Write Below Binary Directory

  desc: Detect writes to system binaries

  condition: write_binary_dirs

  output: >

    Binary modified

  priority: ERROR

  tags:

    - filesystem

    - mitre_persistence
```

---

# Rule Conditions

Conditions define when a rule should trigger.

Example.

```text
Container

AND

Shell Process

AND

Root User

↓

Generate Alert
```

Example.

```yaml
condition:

container and

user.uid=0 and

shell_procs
```

---

# Monitoring Linux Processes

Falco continuously monitors process execution.

Workflow.

```text
Linux Process

↓

Execve Syscall

↓

Falco

↓

Rule Match

↓

Alert
```

Example detection.

- Reverse shell
- Bash execution
- Unexpected binaries
- Privilege escalation

---

# Monitoring File Access

Falco detects unauthorized access to sensitive files.

Examples.

```text
/etc/shadow

/etc/passwd

/root/.ssh

/etc/kubernetes

/var/lib/kubelet
```

Workflow.

```text
Sensitive File

↓

Read

↓

Falco

↓

Alert
```

---

# Monitoring Container Activity

Container runtime events are continuously monitored.

Examples.

```text
Container Start

Container Stop

Container Exec

Container Delete

Container Escape Attempt
```

Workflow.

```text
Container

↓

Runtime Event

↓

Falco

↓

Alert
```

---

# Detecting Shell Access

Unexpected shell access inside containers is a common attack indicator.

Workflow.

```text
Container

↓

/bin/bash

↓

Falco Rule

↓

Warning
```

Example.

```bash
kubectl exec -it nginx -- bash
```

An alert is generated when the rule conditions are satisfied.

---

# Detecting Privilege Escalation

Falco identifies attempts to gain elevated privileges.

Workflow.

```text
Container

↓

sudo

↓

Root Access

↓

Alert
```

Examples.

- sudo execution
- setuid binaries
- privilege escalation tools

---

# Detecting Container Escape

Container escape attempts represent critical security events.

Workflow.

```text
Container

↓

Host Namespace

↓

Kernel Access

↓

Falco

↓

Critical Alert
```

These alerts require immediate investigation.

---

# Detecting Reverse Shells

Reverse shells allow attackers to remotely control compromised containers.

Workflow.

```text
Container

↓

Bash

↓

Remote Connection

↓

Attacker

↓

Alert
```

Common processes.

- bash
- sh
- nc
- socat
- python
- perl

---

# Detecting Cryptocurrency Mining

Falco can detect suspicious mining processes.

Examples.

```text
xmrig

minerd

cpuminer
```

Workflow.

```text
Container

↓

Mining Process

↓

High CPU

↓

Falco Alert
```

---

# Detecting Kubernetes Exec

Unexpected `kubectl exec` operations may indicate unauthorized access.

Workflow.

```text
Administrator

↓

kubectl exec

↓

Running Pod

↓

Falco

↓

Alert
```

Production environments should monitor all interactive shell access.

---

# Detecting Secret Access

Falco monitors access to sensitive Kubernetes resources.

Examples.

```text
Secrets

ConfigMaps

Certificates

Service Account Tokens
```

Workflow.

```text
Application

↓

Secret Read

↓

Falco

↓

Alert
```

---

# Detecting Sensitive File Modification

Critical system files should rarely change during normal application execution.

Examples.

```text
/etc/passwd

/etc/group

/etc/shadow

/etc/sudoers
```

Workflow.

```text
File Modification

↓

Falco Rule

↓

Critical Alert
```

---

# Runtime Threat Detection Workflow

```text
Application

↓

Container

↓

Linux Kernel

↓

Syscalls

↓

Falco Rules

↓

Alert

↓

Security Team
```

---

# MITRE ATT&CK Mapping

Many built-in Falco rules align with MITRE ATT&CK techniques.

| MITRE Technique | Example |
|-----------------|----------|
| Initial Access | Suspicious Remote Connection |
| Execution | Shell Inside Container |
| Persistence | Binary Modification |
| Privilege Escalation | sudo Execution |
| Defense Evasion | Hidden Process |
| Credential Access | Secret File Access |
| Discovery | Network Enumeration |
| Lateral Movement | Remote Shell |
| Exfiltration | Suspicious File Copy |

This mapping helps security teams correlate runtime alerts with known attack techniques.

---

# Enterprise Best Practices

- Monitor all container execution events.
- Alert on interactive shell access.
- Detect privilege escalation attempts immediately.
- Monitor Kubernetes Secret access.
- Review file modification alerts.
- Enable Kubernetes audit monitoring.
- Map alerts to MITRE ATT&CK techniques.
- Tune rules to reduce false positives.
- Investigate all Critical alerts immediately.
- Regularly review and update custom detection rules.

---

