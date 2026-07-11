# ConfigMaps Advanced and Nginx Configuration

## Introduction

Earlier we learned:

```text
ConfigMaps Store

Non-Confidential Configuration
```

---

Most beginners use ConfigMaps only for:

```text
Environment Variables
```

---

Production environments commonly use ConfigMaps for:

```text
Application Configuration Files

Nginx Configuration

Prometheus Configuration

Grafana Configuration

Application Properties
```

---

# Real World Problem

Suppose we run:

```text
Nginx
```

inside Kubernetes.

---

Nginx requires:

```text
nginx.conf
```

---

Question

```text
Where Should We Store
nginx.conf ?
```

---

Bad Approach

```text
Build New Docker Image
Every Time Configuration Changes
```

---

Problems

```text
Slow

Difficult To Manage

Image Rebuild Required
```

---

Better Solution

```text
ConfigMap
```

---

# ConfigMap As File

ConfigMap can store:

```text
Entire File Content
```

---

Example

```text
nginx.conf
```

stored inside ConfigMap.

---

Application reads file directly.

---

Benefits

```text
No Image Rebuild

Easy Updates

Configuration Separation
```

---

# Basic ConfigMap Example

```yaml
apiVersion: v1

kind: ConfigMap

metadata:
  name: nginx-config

data:

  nginx.conf: |

    events {}

    http {

      server {

        listen 80;

        location / {

          return 200 "Welcome To Kubernetes";

        }

      }

    }
```

---

Notice

```text
Entire nginx.conf

Stored As Value
```

---

# Understanding Pipe Symbol

Example

```yaml
data:

  nginx.conf: |
```

---

Pipe Symbol

```text
|
```

means:

```text
Preserve Multi-Line Content
```

---

Everything below becomes:

```text
Single File Content
```

---

# Create ConfigMap

Apply

```bash
kubectl apply -f configmap.yaml
```

---

Verify

```bash
kubectl get configmaps
```

---

Output

```text
nginx-config
```

---

# View ConfigMap

Command

```bash
kubectl get configmap nginx-config -o yaml
```

---

Output

```text
nginx.conf Content
```

---

# Mount ConfigMap As Volume

ConfigMap itself does nothing.

---

Need to mount it inside Pod.

---

Example

```yaml
volumes:

- name: nginx-config

  configMap:

    name: nginx-config
```

---

Container

```yaml
volumeMounts:

- name: nginx-config

  mountPath: /etc/nginx/nginx.conf

  subPath: nginx.conf
```

---

# Why Use subPath?

Without subPath

```text
Entire Directory Mounted
```

---

With subPath

```text
Single File Mounted
```

---

Common Production Practice.

---

# Complete Example

ConfigMap

```yaml
apiVersion: v1

kind: ConfigMap

metadata:
  name: nginx-config

data:

  nginx.conf: |

    events {}

    http {

      server {

        listen 80;

        location / {

          return 200 "Hello Kubernetes";

        }

      }

    }
```

---

Pod

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: nginx

spec:

  containers:

  - name: nginx

    image: nginx

    volumeMounts:

    - name: nginx-config

      mountPath: /etc/nginx/nginx.conf

      subPath: nginx.conf

  volumes:

  - name: nginx-config

    configMap:

      name: nginx-config
```

---

Result

```text
Custom nginx.conf

Loaded Inside Container
```

---

# Verify Mounted File

Access Container

```bash
kubectl exec -it nginx -- bash
```

---

View File

```bash
cat /etc/nginx/nginx.conf
```

---

Output

```text
ConfigMap Content
```

---

# Updating Configuration

Modify ConfigMap

---

Apply

```bash
kubectl apply -f configmap.yaml
```

---

ConfigMap Updated.

---

Important

```text
Application May Need Restart
```

depending on application behavior.

---

# Production Use Cases

## Nginx

```text
nginx.conf
```

---

## Prometheus

```text
prometheus.yml
```

---

## Grafana

```text
datasource.yml
```

---

## Spring Boot

```text
application.properties
```

---

## NodeJS

```text
config.json
```

---

# ConfigMap As Environment Variables vs Files

## Environment Variables

Example

```yaml
envFrom:
- configMapRef:
    name: app-config
```

---

Best For

```text
Hostnames

Ports

URLs

Flags
```

---

## File Mounts

Example

```yaml
volumeMounts
```

---

Best For

```text
nginx.conf

prometheus.yml

application.properties
```

---

# Real Production Example

Roboshop Frontend

Uses

```text
Nginx
```

---

Nginx Configuration

```text
Routing Rules

Caching Rules

Headers

Reverse Proxy Configuration
```

---

Stored In

```text
ConfigMap
```

---

Mounted Into

```text
/etc/nginx/nginx.conf
```

---

Benefits

```text
No Image Rebuild

Easy Management

Version Controlled
```

---

# Common Mistakes

## Wrong ConfigMap Name

Volume

```yaml
configMap:
  name: nginx-config
```

---

ConfigMap

```yaml
metadata:
  name: nginx-config-prod
```

---

Result

```text
Pod Fails To Start
```

---

## Wrong File Name

ConfigMap

```yaml
data:
  nginx.conf
```

---

Mount

```yaml
subPath: config.conf
```

---

Result

```text
File Not Found
```

---

# Troubleshooting

View ConfigMap

```bash
kubectl get configmap nginx-config -o yaml
```

---

Describe Pod

```bash
kubectl describe pod nginx
```

---

Access Container

```bash
kubectl exec -it nginx -- bash
```

---

Verify File

```bash
cat /etc/nginx/nginx.conf
```

---

# Common Interview Questions

## Can ConfigMaps Store Files?

### Short Answer

Yes.

### Detailed Explanation

ConfigMaps can store complete file contents and mount them into containers.

### Production Example

nginx.conf stored inside ConfigMap.

---

## Why Use ConfigMaps For nginx.conf?

### Short Answer

To separate configuration from container images.

### Detailed Explanation

Configuration changes do not require image rebuilds.

### Production Example

Updating Nginx routing without building a new image.

---

## What is subPath?

### Short Answer

subPath mounts a single file from a volume.

### Production Example

Mounting nginx.conf into /etc/nginx/nginx.conf.

---

## How Do You Verify ConfigMap Mounts?

### Commands

```bash
kubectl exec -it pod-name -- bash

cat /etc/nginx/nginx.conf
```

---

# Key Takeaways

```text
ConfigMaps can store complete configuration files.

nginx.conf is commonly stored inside ConfigMaps.

ConfigMaps can be mounted as files using volumes.

subPath is used to mount a specific file.

Configuration changes can be managed separately from images.

Nginx, Prometheus, and Grafana commonly use ConfigMap-based configuration.

ConfigMaps improve maintainability and reduce image rebuilds.

Using ConfigMaps for configuration files is a common production pattern.
```