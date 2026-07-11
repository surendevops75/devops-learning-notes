# Kubectx and Kubens

## Introduction

As Kubernetes usage grows, engineers often work with:

```text
Multiple Clusters

Multiple Namespaces

Development Environment

QA Environment

Production Environment
```

---

Switching contexts manually becomes tedious.

---

Kubernetes administrators commonly use:

```text
kubectx

kubens
```

to simplify cluster and namespace switching.

---

# What is kubectx?

kubectx is a utility used to:

```text
Switch Between Kubernetes Clusters

View Available Contexts

Manage Multiple Clusters
```

---

Without kubectx:

```bash
kubectl config use-context cluster-name
```

---

With kubectx:

```bash
kubectx cluster-name
```

---

Much simpler.

---

# What is a Context?

A Kubernetes context contains:

```text
Cluster

User

Namespace
```

information.

---

View Contexts

```bash
kubectl config get-contexts
```

---

Example

```text
dev-cluster

qa-cluster

prod-cluster
```

---

Current Context

```bash
kubectl config current-context
```

---

# Installing kubectx

Clone Repository

```bash
sudo git clone \
https://github.com/ahmetb/kubectx \
/opt/kubectx
```

---

Create Symbolic Link

```bash
sudo ln -s \
/opt/kubectx/kubectx \
/usr/local/bin/kubectx
```

---

Verify

```bash
kubectx
```

---

# Listing Contexts

Command

```bash
kubectx
```

---

Example Output

```text
dev-cluster

qa-cluster

prod-cluster
```

---

Current Context

```text
Marked With *
```

---

# Switching Contexts

Example

```bash
kubectx prod-cluster
```

---

Result

```text
Switched To Production Cluster
```

---

Verify

```bash
kubectl config current-context
```

---

# Why kubectx is Useful?

Without kubectx

```bash
kubectl config use-context prod-cluster
```

---

With kubectx

```bash
kubectx prod-cluster
```

---

Faster and easier.

---

# What is kubens?

kubens is a utility used to:

```text
Switch Namespaces Quickly
```

---

Without kubens

```bash
kubectl config set-context \
--current \
--namespace=roboshop
```

---

With kubens

```bash
kubens roboshop
```

---

Much simpler.

---

# Installing kubens

Create Symbolic Link

```bash
sudo ln -s \
/opt/kubectx/kubens \
/usr/local/bin/kubens
```

---

Verify

```bash
kubens
```

---

# Listing Namespaces

Command

```bash
kubens
```

---

Example Output

```text
default

roboshop

monitoring

argocd
```

---

Current Namespace

```text
Marked With *
```

---

# Switching Namespaces

Example

```bash
kubens roboshop
```

---

Verify

```bash
kubectl get pods
```

---

Now commands automatically use:

```text
roboshop Namespace
```

---

# Production Example

Suppose you manage:

```text
Dev Cluster

QA Cluster

Production Cluster
```

---

Contexts

```text
roboshop-dev

roboshop-qa

roboshop-prod
```

---

Switch To Production

```bash
kubectx roboshop-prod
```

---

Switch Namespace

```bash
kubens roboshop
```

---

Now troubleshooting occurs in:

```text
Production Cluster

Roboshop Namespace
```

---

# Common Mistakes

## Wrong Cluster

Issue

```text
Accidentally Running Commands
In Production
```

---

Verify

```bash
kubectl config current-context
```

---

Before making changes.

---

## Wrong Namespace

Issue

```text
Pod Not Found
```

---

Check

```bash
kubens
```

---

Verify correct namespace.

---

# Troubleshooting

View Current Context

```bash
kubectl config current-context
```

---

View Contexts

```bash
kubectl config get-contexts
```

---

View Namespace

```bash
kubectl config view --minify \
-o jsonpath='{..namespace}'
```

---

# Common Interview Questions

## What is kubectx?

### Short Answer

A utility used to switch Kubernetes contexts quickly.

### Production Example

Switching between development and production clusters.

---

## What is kubens?

### Short Answer

A utility used to switch namespaces quickly.

### Production Example

Switching between roboshop and monitoring namespaces.

---

## Why Use kubectx and kubens?

### Short Answer

They simplify cluster and namespace management.

### Detailed Explanation

They reduce typing, improve productivity, and minimize operational mistakes.

---

# Key Takeaways

```text
kubectx switches Kubernetes contexts.

kubens switches namespaces.

Contexts represent cluster access configurations.

Namespaces organize resources within clusters.

kubectx and kubens improve productivity.

Always verify context before production changes.

These tools are widely used by Kubernetes administrators and DevOps engineers.
```