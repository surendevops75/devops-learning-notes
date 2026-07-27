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

# Kubernetes Runtime Monitoring

Falco continuously monitors Kubernetes workloads running across the cluster.

Architecture.

```text
Kubernetes Cluster

↓

Worker Nodes

↓

Falco DaemonSet

↓

Running Pods

↓

Runtime Events

↓

Alerts
```

Every node is protected by a Falco pod.

---

# Monitoring Pod Creation

Falco detects newly created pods.

Workflow.

```text
kubectl apply

↓

API Server

↓

Scheduler

↓

Pod Created

↓

Falco

↓

Event Logged
```

Unexpected pod creation can indicate unauthorized deployments.

---

# Monitoring Pod Deletion

Deleting production workloads unexpectedly may indicate malicious activity.

Workflow.

```text
Delete Pod

↓

API Server

↓

Falco

↓

Alert
```

Production clusters should audit all pod deletion events.

---

# Monitoring Privileged Containers

Privileged containers have elevated access to the host.

Workflow.

```text
Pod

↓

privileged=true

↓

Falco

↓

Critical Alert
```

Example.

```yaml
securityContext:

  privileged: true
```

Privileged containers should only be used when absolutely necessary.

---

# Monitoring HostPath Mounts

HostPath volumes provide direct access to the Kubernetes node.

Workflow.

```text
Container

↓

HostPath Volume

↓

Host Filesystem

↓

Falco

↓

Alert
```

Example.

```yaml
volumes:

- name: host

  hostPath:

    path: /
```

HostPath mounts should be carefully reviewed.

---

# Monitoring Host Network Usage

Pods using the host network bypass Kubernetes network isolation.

Example.

```yaml
hostNetwork: true
```

Workflow.

```text
Pod

↓

Host Network

↓

Falco

↓

Warning
```

---

# Monitoring Host PID Namespace

Sharing the host PID namespace increases security risk.

Example.

```yaml
hostPID: true
```

Workflow.

```text
Pod

↓

Host PID

↓

Falco

↓

Alert
```

---

# Monitoring Sensitive Volume Mounts

Examples.

```text
/var/run/docker.sock

/etc

/root

/var/lib/kubelet

/proc
```

Workflow.

```text
Sensitive Volume

↓

Container

↓

Falco

↓

Alert
```

---

# Monitoring Service Account Tokens

Falco detects unexpected access to Kubernetes service account tokens.

Workflow.

```text
Container

↓

Service Account Token

↓

Read

↓

Falco

↓

Alert
```

Unexpected token access should be investigated immediately.

---

# Monitoring Kubernetes Secrets

Example workflow.

```text
Application

↓

Read Secret

↓

API Server

↓

Falco

↓

Alert
```

Examples.

- Database passwords
- API tokens
- TLS certificates
- Cloud credentials

---

# Monitoring RBAC Changes

Unauthorized RBAC modifications can lead to privilege escalation.

Workflow.

```text
Role

↓

RoleBinding

↓

ClusterRole

↓

Falco

↓

Alert
```

Monitor all RBAC updates in production clusters.

---

# Monitoring Namespace Changes

Unexpected namespace creation or deletion may indicate unauthorized activity.

Workflow.

```text
Create Namespace

↓

API Server

↓

Falco

↓

Audit Event
```

---

# Monitoring Kubernetes Jobs

Jobs execute one-time workloads.

Workflow.

```text
Job

↓

Pod

↓

Execution

↓

Falco

↓

Alert
```

Unexpected Jobs should be reviewed by platform administrators.

---

# Monitoring CronJobs

CronJobs execute on a schedule.

Workflow.

```text
CronJob

↓

Scheduled Execution

↓

Pod

↓

Falco
```

Unexpected CronJobs may indicate persistence mechanisms.

---

# Monitoring Network Connections

Falco observes outbound network connections created by containers.

Workflow.

```text
Container

↓

Network Connection

↓

Destination IP

↓

Falco

↓

Alert
```

Examples include:

- Unknown external IPs
- Suspicious ports
- Unexpected destinations

---

# Monitoring DNS Requests

DNS activity can reveal malicious behaviour.

Workflow.

```text
Container

↓

DNS Query

↓

Unknown Domain

↓

Falco

↓

Alert
```

Unexpected DNS requests should be investigated.

---

# Monitoring File Downloads

Attackers often download additional tools after compromising a container.

Workflow.

```text
Container

↓

wget

↓

curl

↓

Download

↓

Falco
```

Common utilities.

- wget
- curl
- aria2

---

# Monitoring Package Installation

Production containers should be immutable.

Workflow.

```text
Container

↓

apt

↓

yum

↓

dnf

↓

apk

↓

Falco Alert
```

Installing software inside running containers is generally unexpected.

---

# Enterprise Runtime Security Workflow

```text
Container Starts

↓

Application Executes

↓

Linux Syscalls

↓

Kubernetes Events

↓

Falco Rules

↓

Alert Generated

↓

SIEM

↓

SOC Team

↓

Incident Response
```

Falco continuously evaluates runtime activity without requiring application changes.

---

# Enterprise Best Practices

- Monitor privileged containers.
- Alert on HostPath volume usage.
- Detect host namespace sharing.
- Monitor Kubernetes Secrets and RBAC changes.
- Review service account token access.
- Investigate unexpected outbound network connections.
- Treat package installation inside containers as suspicious.
- Forward alerts to a central SIEM.
- Correlate Falco alerts with Kubernetes audit logs.
- Continuously tune runtime detection rules to reduce false positives while maintaining security coverage.

---

# Jenkins Integration

Falco is a runtime security platform rather than a build-time scanner.

Instead of running inside the CI pipeline, Jenkins deploys Falco to Kubernetes and validates that runtime monitoring is operational.

Enterprise workflow.

```text
Developer

↓

Git Push

↓

Jenkins

↓

Build

↓

Docker Image

↓

Deploy to Amazon EKS

↓

Verify Falco

↓

Runtime Monitoring Enabled

↓

Production
```

---

# Production Jenkins Pipeline

```groovy
pipeline {

    agent any

    stages {

        stage('Checkout') {

            steps {

                checkout scm

            }

        }

        stage('Build') {

            steps {

                sh 'mvn clean package'

            }

        }

        stage('Deploy') {

            steps {

                sh './deploy.sh'

            }

        }

        stage('Verify Falco') {

            steps {

                sh '''

                kubectl get pods -n falco

                kubectl get daemonset -n falco

                '''

            }

        }

    }

}
```

---

# GitHub Actions Integration

Enterprise workflow.

```text
Git Push

↓

GitHub Actions

↓

Build

↓

Deploy

↓

Verify Falco

↓

Runtime Security Enabled
```

---

# Production GitHub Actions Workflow

```yaml
name: Verify-Falco

on:

  push:

    branches:

      - main

jobs:

  falco:

    runs-on: ubuntu-latest

    steps:

    - uses: actions/checkout@v4

    - name: Configure kubectl

      run: |

        kubectl version --client

    - name: Verify Falco

      run: |

        kubectl get pods -n falco

        kubectl get daemonset -n falco
```

---

# GitLab CI Integration

Example.

```yaml
stages:

  - deploy

  - runtime

verify-falco:

  stage: runtime

  script:

    - kubectl get pods -n falco

    - kubectl get daemonset -n falco
```

---

# Prometheus Integration

Falco exposes runtime metrics that can be collected by Prometheus.

Architecture.

```text
Falco

↓

Metrics Endpoint

↓

Prometheus

↓

Grafana

↓

Dashboard
```

Example metrics.

- Total alerts
- Rules triggered
- Events processed
- Dropped events
- Rule execution time

---

# Grafana Integration

Falco metrics can be visualized using Grafana dashboards.

```text
Falco

↓

Prometheus

↓

Grafana

↓

Security Dashboard
```

Typical dashboard panels.

- Alerts per hour
- Top triggered rules
- Alert severity
- Node activity
- Container activity

---

# SIEM Integration

Falco integrates with enterprise SIEM platforms.

Examples.

- Splunk
- IBM QRadar
- Microsoft Sentinel
- Elastic Security
- Google Chronicle

Workflow.

```text
Falco Alert

↓

Webhook

↓

SIEM

↓

Correlation

↓

SOC Team

↓

Incident Response
```

---

# Slack Integration

Security teams often receive alerts through Slack.

Workflow.

```text
Falco

↓

Webhook

↓

Slack

↓

Security Channel

↓

Investigation
```

Example notification.

```text
WARNING

Shell executed inside container

Namespace : production

Pod : payment-api
```

---

# Incident Response Workflow

Falco is commonly integrated into enterprise incident response processes.

```text
Falco Alert

↓

SOC Team

↓

Incident Validation

↓

Pod Investigation

↓

Containment

↓

Recovery

↓

Root Cause Analysis
```

---

# Runtime Security in Amazon EKS

Architecture.

```text
Amazon EKS

↓

Worker Nodes

↓

Falco DaemonSet

↓

Runtime Monitoring

↓

Security Alerts
```

Every EKS worker node should run exactly one Falco pod.

---

# Runtime Security in AKS

Architecture.

```text
Azure Kubernetes Service

↓

Worker Nodes

↓

Falco

↓

Runtime Events

↓

Alerts
```

---

# Runtime Security in Google Kubernetes Engine

Architecture.

```text
Google Kubernetes Engine

↓

Worker Nodes

↓

Falco

↓

Syscalls

↓

Security Alerts
```

---

# Runtime Security with ArgoCD

GitOps workflow.

```text
Git Repository

↓

ArgoCD

↓

Amazon EKS

↓

Falco Running

↓

Runtime Detection
```

GitOps ensures Falco deployments remain synchronized with Git.

---

# Enterprise Runtime Security Pipeline

```text
Developer

↓

Feature Branch

↓

Git Push

↓

Pull Request

↓

Code Review

↓

Merge

↓

CI Trigger

↓

Checkout

↓

Build

↓

Unit Tests

↓

Coverage

↓

SonarQube

↓

OWASP Dependency-Check

↓

Veracode

↓

Docker Build

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

ArgoCD

↓

Amazon EKS

↓

Falco Runtime Monitoring

↓

Prometheus

↓

Grafana

↓

SIEM

↓

SOC Team

↓

Production
```

Falco provides continuous runtime protection after workloads have been deployed.

---

# Common Mistakes

## Mistake 1

Installing Falco on only one node.

**Impact**

Some workloads remain unmonitored.

**Recommendation**

Deploy Falco as a DaemonSet across every worker node.

---

## Mistake 2

Modifying default rules directly.

**Impact**

Updates overwrite customizations.

**Recommendation**

Store custom rules in `falco_rules.local.yaml`.

---

## Mistake 3

Ignoring Falco alerts.

**Impact**

Runtime attacks may go undetected.

**Recommendation**

Forward alerts to a SIEM or incident management platform.

---

## Mistake 4

Generating too many false positives.

**Impact**

Security teams experience alert fatigue.

**Recommendation**

Tune rules and configure approved exceptions.

---

## Mistake 5

Not monitoring Kubernetes audit events.

**Impact**

Administrative attacks may not be detected.

**Recommendation**

Enable Kubernetes Audit Logs alongside syscall monitoring.

---

# Troubleshooting

## Scenario 1

### Falco Pods Are Not Running

**Cause**

DaemonSet deployment failed.

**Resolution**

```bash
kubectl get pods -n falco

kubectl describe daemonset falco -n falco
```

---

## Scenario 2

### No Alerts Generated

**Cause**

Rules are not matching or test events are missing.

**Resolution**

- Verify Falco is running.
- Check active rules.
- Generate a known test event.
- Review Falco logs.

---

## Scenario 3

### High CPU Usage

**Cause**

Large numbers of syscalls or excessive rule processing.

**Resolution**

- Optimize custom rules.
- Disable unnecessary plugins.
- Monitor resource utilization.

---

## Scenario 4

### Excessive False Positives

**Cause**

Rules are too broad.

**Resolution**

- Add rule exceptions.
- Tune conditions.
- Validate alerts before suppressing them.

---

## Scenario 5

### Missing Kubernetes Audit Events

**Cause**

Audit logging is not configured.

**Resolution**

- Enable Kubernetes Audit Logs.
- Configure the Falco audit plugin.
- Verify audit event collection.

---

# Production Interview Questions

## Question 1

### What is Falco?

**Answer**

Falco is a CNCF runtime security tool that detects suspicious activity in Linux hosts, containers, and Kubernetes clusters using system call and audit event monitoring.

---

## Question 2

### How is Falco different from Trivy?

**Answer**

Trivy scans images, filesystems, and Infrastructure as Code before deployment, whereas Falco monitors running workloads and detects suspicious runtime behaviour after deployment.

---

## Question 3

### Why is Falco deployed as a DaemonSet?

**Answer**

A DaemonSet ensures one Falco pod runs on every Kubernetes worker node, allowing runtime monitoring across the entire cluster.

---

## Question 4

### What types of events can Falco monitor?

**Answer**

Falco monitors Linux syscalls, Kubernetes audit events, container runtime events, file access, process execution, network activity, and plugin-based event sources.

---

## Question 5

### Can Falco detect container escapes?

**Answer**

Yes. Falco includes built-in rules that detect behaviours associated with container escape attempts and privilege escalation.

---

## Question 6

### Why shouldn't custom rules be added to `falco_rules.yaml`?

**Answer**

The default rule file is replaced during upgrades. Custom rules should be stored in `falco_rules.local.yaml` or separate rule files.

---

## Question 7

### How does Falco integrate with Kubernetes?

**Answer**

Falco runs as a DaemonSet, monitors container runtime events and Kubernetes audit logs, and generates alerts for suspicious activity across the cluster.

---

## Question 8

### How are Falco alerts typically consumed?

**Answer**

Alerts can be sent to files, stdout, syslog, Slack, webhooks, Prometheus, or enterprise SIEM platforms for investigation.

---

## Question 9

### Does Falco prevent attacks?

**Answer**

Falco primarily detects and alerts on suspicious runtime activity. It complements preventive controls by providing real-time visibility into running workloads.

---

## Question 10

### What are the enterprise best practices for Falco?

**Answer**

Deploy Falco on every Kubernetes node, maintain custom rules separately, integrate with SIEM and Prometheus, monitor Kubernetes audit events, tune rules to reduce false positives, and investigate Critical alerts immediately.

---

# Key Takeaways

- Falco provides runtime security for Kubernetes, containers, and Linux hosts.
- It detects suspicious behaviour using Linux syscalls and Kubernetes audit events.
- Deploy Falco as a DaemonSet to monitor every worker node.
- Keep custom rules separate from the default rule set.
- Integrate alerts with Prometheus, Grafana, Slack, and SIEM platforms.
- Monitor privileged containers, shell execution, secret access, and container escape attempts.
- Continuously tune rules to reduce false positives.
- Use Falco alongside SonarQube, OWASP Dependency-Check, Veracode, Gitleaks, Checkov, TFSec, Trivy, OWASP ZAP, GitOps, ArgoCD, and Amazon EKS to build a comprehensive enterprise DevSecOps platform.