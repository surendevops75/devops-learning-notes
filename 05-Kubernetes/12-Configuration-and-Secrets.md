# Configuration and Secrets

## Introduction

Applications cannot run with code alone.

Every application needs:

```text
Configuration
```

---

Examples

```text
Database Host

Database Port

API URLs

Usernames

Passwords

Tokens

Certificates
```

---

Without configuration:

```text
Application Cannot Connect
To External Systems
```

---

Example

```text
Cart Service
      ↓

Needs Catalogue Service URL
```

---

Question

```text
Where Should We Store
Configuration?
```

---

Kubernetes provides:

```text
ConfigMaps

Secrets
```

---

# Types of Configuration

Application configuration can be divided into:

```text
Non-Confidential Data

Confidential Data
```

---

Non-Confidential

```text
Database Host

Application Port

Environment Name

Service URL
```

---

Confidential

```text
Passwords

API Keys

Tokens

Certificates
```

---

# Configuration Flow

Example

```text
Cart Service
      ↓

Needs Catalogue URL
```

---

Instead of hardcoding:

```text
http://catalogue:8080
```

---

Store configuration externally.

---

Benefits

```text
Flexibility

Security

Easy Updates
```

---

# What is a ConfigMap?

ConfigMap stores:

```text
Non-Confidential Configuration
```

---

Examples

```text
URLs

Ports

Environment Variables

Application Settings
```

---

Think of ConfigMap as:

```text
Configuration Storage
```

for Kubernetes applications.

---

# Why ConfigMaps?

Bad Practice

```yaml
env:
- name: CATALOGUE_URL
  value: http://catalogue:8080
```

inside every manifest.

---

Problem

```text
Hard To Change

Hard To Manage

Repeated Configuration
```

---

ConfigMaps solve this.

---

# Creating ConfigMap

Imperative Method

```bash
kubectl create configmap app-config \
--from-literal=CATALOGUE_HOST=catalogue
```

---

Verify

```bash
kubectl get configmaps
```

---

Output

```text
app-config
```

---

# ConfigMap YAML

Example

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: app-config

data:
  CATALOGUE_HOST: catalogue
  CATALOGUE_PORT: "8080"
```

---

Create

```bash
kubectl apply -f configmap.yaml
```

---

# View ConfigMap

Command

```bash
kubectl get configmap app-config -o yaml
```

---

Output

```yaml
data:
  CATALOGUE_HOST: catalogue
  CATALOGUE_PORT: "8080"
```

---

# Using ConfigMap as Environment Variables

ConfigMap

```yaml
data:
  CATALOGUE_HOST: catalogue
```

---

Pod

```yaml
env:
- name: CATALOGUE_HOST
  valueFrom:
    configMapKeyRef:
      name: app-config
      key: CATALOGUE_HOST
```

---

Result

```text
Environment Variable Created
```

---

# Using Entire ConfigMap

Example

```yaml
envFrom:
- configMapRef:
    name: app-config
```

---

Result

```text
All Keys Become
Environment Variables
```

---

# ConfigMap as Volume

ConfigMap can be mounted as files.

---

Example

```yaml
volumes:
- name: config-volume
  configMap:
    name: app-config
```

---

Container

```yaml
volumeMounts:
- name: config-volume
  mountPath: /app/config
```

---

Result

```text
Configuration Available
As Files
```

---

# Real Example

Cart Service

Needs:

```text
Catalogue Host

Catalogue Port
```

---

ConfigMap

```yaml
data:
  CATALOGUE_HOST: catalogue
  CATALOGUE_PORT: "8080"
```

---

Cart reads values dynamically.

---

# What are Secrets?

Secrets store:

```text
Sensitive Information
```

---

Examples

```text
Passwords

API Keys

Database Credentials

Certificates
```

---

Think of Secrets as:

```text
Secure Configuration Storage
```

---

# Why Not Use ConfigMaps?

ConfigMaps are:

```text
Plain Text
```

---

Not suitable for:

```text
Passwords

Tokens

Certificates
```

---

Use Secrets instead.

---

# Secret Example

Database Password

```text
roboshop123
```

---

Store as Secret.

---

# Creating Secret

Imperative Method

```bash
kubectl create secret generic mysql-secret \
--from-literal=password=roboshop123
```

---

Verify

```bash
kubectl get secrets
```

---

# Secret YAML

Example

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: mysql-secret

type: Opaque

data:
  password: cm9ib3Nob3AxMjM=
```

---

Notice

```text
Base64 Encoded
```

---

Not encrypted.

---

# Encode Secret

Example

```bash
echo -n "roboshop123" | base64
```

---

Output

```text
cm9ib3Nob3AxMjM=
```

---

# Decode Secret

```bash
echo cm9ib3Nob3AxMjM= | base64 -d
```

---

Output

```text
roboshop123
```

---

# Using Secret as Environment Variable

Example

```yaml
env:
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: password
```

---

Application receives:

```text
DB_PASSWORD
```

---

# Secret as Volume

Example

```yaml
volumes:
- name: secret-volume
  secret:
    secretName: mysql-secret
```

---

Mount

```yaml
volumeMounts:
- name: secret-volume
  mountPath: /secrets
```

---

Result

```text
Secret Available As File
```

---

# ConfigMap vs Secret

| Feature | ConfigMap | Secret |
|----------|----------|----------|
| Purpose | Configuration | Sensitive Data |
| Password Storage | No | Yes |
| API Keys | No | Yes |
| Environment Variables | Yes | Yes |
| Volume Mounts | Yes | Yes |
| Base64 Encoding | No | Yes |
| Non-Confidential Data | Yes | No |

---

# Production Example

Cart Service

Needs

```text
Catalogue URL

Redis Host

Database Password
```

---

ConfigMap

```yaml
data:
  CATALOGUE_HOST: catalogue
  REDIS_HOST: redis
```

---

Secret

```yaml
data:
  DB_PASSWORD: xxxxxx
```

---

Application

```text
Reads ConfigMap

Reads Secret
```

---

# Configuration Architecture

```text
Application
      │
      ├── ConfigMap
      │      │
      │      └── Non Confidential Data
      │
      └── Secret
             │
             └── Confidential Data
```

---

# Best Practices

## Use ConfigMaps For

```text
URLs

Ports

Hostnames

Feature Flags
```

---

## Use Secrets For

```text
Passwords

Tokens

Certificates
```

---

## Never Hardcode Credentials

Bad

```yaml
env:
- name: PASSWORD
  value: roboshop123
```

---

Good

```yaml
secretKeyRef
```

---

## Separate Configuration From Code

Benefits

```text
Reusability

Maintainability

Security
```

---

# Troubleshooting ConfigMaps

View ConfigMap

```bash
kubectl get configmap app-config -o yaml
```

---

Check Pod

```bash
kubectl describe pod pod-name
```

---

Verify Environment Variables.

---

# Troubleshooting Secrets

View Secret

```bash
kubectl get secret mysql-secret
```

---

Describe

```bash
kubectl describe secret mysql-secret
```

---

Verify

```text
Secret Exists

Correct Key Names
```

---

# Common Interview Questions

## What is a ConfigMap?

### Short Answer

A ConfigMap stores non-confidential configuration data.

### Detailed Explanation

It allows applications to externalize configuration and avoid hardcoding values.

### Production Example

Storing catalogue host and port information.

### Follow-Up Questions

- How can ConfigMaps be consumed?
- Can ConfigMaps be mounted as volumes?

---

## What is a Secret?

### Short Answer

A Secret stores sensitive information such as passwords and API keys.

### Detailed Explanation

Secrets allow applications to securely consume confidential data.

### Production Example

Database credentials for a payment service.

### Follow-Up Questions

- Are Secrets encrypted?
- How do you consume Secrets?

---

## Difference Between ConfigMap and Secret?

### Short Answer

ConfigMaps store non-confidential data, while Secrets store confidential data.

### Production Example

Catalogue URL in ConfigMap and MySQL password in Secret.

### Follow-Up Questions

- Can both be mounted as volumes?
- Can both be used as environment variables?

---

# Key Takeaways

```text
Applications require configuration.

ConfigMaps store non-confidential data.

Secrets store confidential data.

Configuration should never be hardcoded.

ConfigMaps can be consumed as environment variables or files.

Secrets can be consumed as environment variables or files.

Cart service may use ConfigMaps for catalogue details.

Database passwords should always be stored in Secrets.

Separating configuration from code improves maintainability and security.
```