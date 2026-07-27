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

