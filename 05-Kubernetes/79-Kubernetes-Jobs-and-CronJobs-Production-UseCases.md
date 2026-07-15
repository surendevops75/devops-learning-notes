# Kubernetes Jobs and CronJobs - Production Use Cases

## Introduction

The previous file explained:

```text
What Is A Job?

What Is A CronJob?

How They Work?
```

---

This file focuses on:

```text
How Real Companies Use Jobs And CronJobs
In Production
```

---

# Production Use Cases

Most Kubernetes Jobs and CronJobs are used for:

```text
Database Operations

Backups

Data Processing

Maintenance Tasks

Security Tasks

Cluster Operations

Reporting
```

---

# 1. Database Migration Job

## Production Scenario

Developers added a new feature.

Application Version

```text
v1
```

Table

```sql
users
```

---

New Version

```text
v2
```

Requires

```sql
ALTER TABLE users
ADD COLUMN customer_phone VARCHAR(20);
```

---

Before deploying v2:

```text
Schema Must Be Updated
```

---

# Flow

```text
CI/CD Pipeline

↓

Migration Job

↓

Schema Updated

↓

Application Deployment
```

---

# YAML Example

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: mysql-migration

spec:
  template:
    spec:
      containers:
      - name: migration

        image: mysql:8

        env:
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: password

        command:
        - sh
        - -c
        - |
          mysql \
          -h mysql.roboshop.svc.cluster.local \
          -u root \
          -p$MYSQL_PASSWORD \
          < /scripts/migration.sql

      restartPolicy: Never
```

---

# Why Job?

Because migration should:

```text
Run Once

Complete

Exit
```

---

# 2. MongoDB Backup CronJob

## Production Scenario

Requirement

```text
Backup MongoDB

Every Day

At 2 AM
```

---

# Flow

```text
CronJob

↓

mongodump

↓

Backup File

↓

S3 Upload

↓

Completed
```

---

# YAML Example

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: mongodb-backup

spec:
  schedule: "0 2 * * *"

  jobTemplate:
    spec:
      template:
        spec:

          containers:

          - name: backup

            image: mongo

            command:
            - sh
            - -c
            - |
              mongodump \
              --host mongodb \
              --out /backup

              aws s3 cp \
              /backup \
              s3://roboshop-backups/ \
              --recursive

          restartPolicy: Never
```

---

# Why CronJob?

Because backup is:

```text
Recurring

Scheduled

Automated
```

---

# 3. MongoDB Restore Job

## Production Scenario

Developer Accidentally Deleted Data

Need To Restore

```text
Yesterday's Backup
```

---

# Flow

```text
Download Backup

↓

Restore Job

↓

Database Restored

↓

Validation
```

---

# YAML Example

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: mongodb-restore

spec:
  template:
    spec:

      containers:

      - name: restore

        image: mongo

        command:
        - sh
        - -c
        - |
          mongorestore \
          --host mongodb \
          /backup

      restartPolicy: Never
```

---

# 4. Log Cleanup CronJob

## Production Scenario

Application Logs Growing Daily

Need To Delete Logs Older Than:

```text
30 Days
```

---

# Flow

```text
1 AM

↓

Cleanup Job

↓

Delete Old Logs

↓

Disk Space Recovered
```

---

# YAML Example

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: log-cleanup

spec:
  schedule: "0 1 * * *"

  jobTemplate:
    spec:
      template:
        spec:

          containers:

          - name: cleanup

            image: busybox

            command:
            - sh
            - -c
            - |
              find /logs -mtime +30 -delete

          restartPolicy: Never
```

---

# 5. EFS Backup To S3 CronJob

## Production Scenario

Shared Files Stored In:

```text
EFS
```

---

Need Daily Backup To:

```text
S3
```

---

# Flow

```text
CronJob

↓

tar

↓

gzip

↓

S3 Upload

↓

Completed
```

---

# Example Command

```bash
tar -czf backup.tar.gz /efs-data

aws s3 cp backup.tar.gz \
s3://roboshop-backups/
```

---

# 6. Product Import Job

## Production Scenario

Business Uploads

```text
products.csv
```

Contains

```text
500,000 Products
```

---

Need To Load Into Database.

---

# Flow

```text
CSV

↓

Import Job

↓

Database

↓

Completed
```

---

# Why Job?

Because import should:

```text
Run Once

Complete

Exit
```

---

# 7. Customer Data Export Job

## Production Scenario

Analytics Team Requests

```text
Customer Data
```

---

# Flow

```text
Database

↓

CSV

↓

S3

↓

Analytics Team
```

---

# Example

```bash
mysqldump

↓

CSV

↓

S3
```

---

# 8. Cache Warmup Job

## Production Scenario

Deployment Completed.

Need To Populate Redis Cache.

---

# Flow

```text
Deployment

↓

Cache Warmup Job

↓

Redis Populated

↓

Traffic Enabled
```

---

# Example

```text
Database

↓

Redis

↓

Application Faster
```

---

# 9. Cache Refresh CronJob

## Production Scenario

Product Catalog Updates Frequently.

Need To Refresh Redis Daily.

---

# Flow

```text
Database

↓

CronJob

↓

Redis

↓

Updated Cache
```

---

# Schedule

```text
12 AM Daily
```

---

# 10. Secret Rotation CronJob

## Production Scenario

Security Team Requirement

```text
Rotate Passwords

Every 90 Days
```

---

# Flow

```text
Generate Password

↓

Update Secrets Manager

↓

Update Kubernetes Secret

↓

Restart Pods
```

---

# Example Services

```text
MySQL

MongoDB

Redis
```

---

# 11. SSL Certificate Validation CronJob

## Production Scenario

Need Alert Before Certificate Expires.

---

# Flow

```text
CronJob

↓

Check ACM

↓

Expiry < 30 Days?

↓

Send Slack Alert
```

---

# Example

```bash
aws acm list-certificates
```

---

# 12. Cluster Health Check CronJob

## Production Scenario

Platform Team Wants Hourly Validation.

---

Checks

```text
Pods

Nodes

PVCs

Services

Ingress
```

---

# Flow

```text
CronJob

↓

kubectl Commands

↓

Generate Report

↓

Slack
```

---

# Example Commands

```bash
kubectl get nodes

kubectl get pods -A

kubectl get pvc -A
```

---

# 13. EKS Upgrade Validation Job

## Production Scenario

Cluster Upgraded

```text
1.32

↓

1.33
```

---

Need Validation.

---

Checks

```text
DNS

Database

Secrets

External APIs
```

---

# Flow

```text
Upgrade

↓

Validation Job

↓

All Checks Passed

↓

Production Ready
```

---

# 14. Security Scan CronJob

## Production Scenario

Scan Container Images Daily.

---

Tools

```text
Trivy

Grype
```

---

# Flow

```text
CronJob

↓

Image Scan

↓

Vulnerability Report

↓

Slack Alert
```

---

# Example

```bash
trivy image catalogue:v10
```

---

# 15. ECR Cleanup CronJob

## Production Scenario

ECR Contains

```text
Thousands Of Images
```

---

Need To Keep

```text
Latest 20
```

---

Delete Remaining.

---

# Flow

```text
CronJob

↓

List Images

↓

Delete Old Images

↓

Save Storage Cost
```

---

# 16. Monthly AWS Cost Report

## Production Scenario

Finance Team Needs Cost Report.

---

Schedule

```text
1st Day Of Every Month
```

---

# Flow

```text
Cost Explorer

↓

CSV

↓

Email

↓

Finance Team
```

---

# 17. ETL Processing Job

## Production Scenario

Move Data To Data Warehouse.

---

# Flow

```text
Extract

↓

Transform

↓

Load

↓

Completed
```

---

# Example

```text
MySQL

↓

S3

↓

Redshift
```

---

# 18. Batch Processing Job

## Production Scenario

Need To Process

```text
1 Million Orders
```

---

# Job Spec

```yaml
parallelism: 10

completions: 100
```

---

# Flow

```text
100 Tasks

↓

10 Parallel Pods

↓

Completed
```

---

# 19. Disaster Recovery Validation CronJob

## Production Scenario

Weekly DR Validation.

---

Checks

```text
Backup Exists

Restore Works

Database Healthy
```

---

# Flow

```text
CronJob

↓

Restore Test

↓

Validation

↓

Report
```

---

# 20. Kubernetes Resource Cleanup CronJob

## Production Scenario

Remove

```text
Completed Jobs

Failed Pods

Unused Resources
```

---

# Example

```bash
kubectl delete pod \
--field-selector=status.phase=Failed
```

---

# 21. Namespace Audit CronJob

## Production Scenario

Find

```text
Unused Namespaces

Unused PVCs

Unused Services
```

---

Generate Report Weekly.

---

# 22. Compliance Validation CronJob

## Production Scenario

Security Team Requires Validation.

---

Checks

```text
Privileged Pods

Root Containers

Missing Resource Limits

Missing Labels
```

---

# Flow

```text
CronJob

↓

Security Audit

↓

Report

↓

Slack
```

---

# Job vs CronJob In Production

## Job

Used For

```text
Migration

Restore

Import

Validation

ETL

Batch Processing
```

---

## CronJob

Used For

```text
Backups

Cleanup

Health Checks

Reports

Security Scans

Secret Rotation
```

---

# Common Interview Questions

## How Are Jobs Used In Your Production Environment?

### Short Answer

Jobs are used for one-time operational tasks such as database migrations, data imports, EKS upgrade validation, cache warmups, and disaster recovery testing.

### Detailed Explanation

Jobs are preferred whenever a task should run once, complete successfully, and exit. Unlike Deployments, Jobs are not expected to run continuously.

### Production Example

```text
Application Release

↓

Database Migration Job

↓

Schema Updated

↓

Application Deployment
```
---

# Key Takeaways

```text
Jobs are used for one-time operational tasks.

CronJobs are used for recurring scheduled tasks.

Database migrations are the most common Job use case.

Backups are the most common CronJob use case.

Security scans, health checks, reporting, and secret rotation are commonly automated using CronJobs.

Jobs and CronJobs are heavily used by Platform, DevOps, SRE, Security, and Data Engineering teams.

Many production operational activities can be automated using Jobs and CronJobs.
```