# Kubernetes Jobs and CronJobs

## Introduction

Most Kubernetes workloads are:

```text
Long Running Applications
```

Examples:

```text
Frontend

Catalogue

User

Cart

Payment
```

---

These workloads use:

```text
Deployment

StatefulSet
```

because they should run continuously.

---

However some workloads are different.

Examples:

```text
Database Migration

Backup

Cleanup

Report Generation

Data Processing
```

---

These workloads:

```text
Run Once

Complete

Exit
```

---

For such workloads Kubernetes provides:

```text
Job

CronJob
```

---

# Why Deployments Are Not Suitable?

Deployment Goal

```text
Keep Pods Running Forever
```

---

Example

```text
Replicas = 3
```

---

If Pod Exits

```text
Deployment Creates New Pod
```

---

Good For

```text
Applications
```

---

Bad For

```text
Database Migration
```

because:

```text
Migration Should Run

Only Once
```

---

# What Is A Job?

A Job is a Kubernetes resource that ensures:

```text
Task Completes Successfully
```

---

Job Goal

```text
Run

Complete

Exit
```

---

Once Successful

```text
No New Pods Created
```

---

# Job Architecture

```text
Job

↓

Pod

↓

Task Executes

↓

Completed
```

---

# Production Example

Application Deployment

Before Application Starts

Need To Run:

```text
Database Migration
```

---

Flow

```text
Job

↓

Run SQL Script

↓

Complete

↓

Application Starts
```

---

# Example Job YAML

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: database-migration

spec:

  template:

    spec:

      containers:

      - name: migration

        image: mysql:8

        command:

        - sh

        - -c

        - echo Running Migration

      restartPolicy: Never
```

---

# Job Lifecycle

Create Job

↓

Pod Created

↓

Task Executes

↓

Success

↓

Completed

---

Status

```text
Completed
```

---

# Verify Job

```bash
kubectl get jobs
```

---

Output

```text
NAME

database-migration

COMPLETIONS 1/1
```

---

# Verify Pods

```bash
kubectl get pods
```

---

Status

```text
Completed
```

---

# Job Retries

What If Task Fails?

---

Kubernetes Retries

Automatically.

---

Controlled By

```yaml
backoffLimit: 3
```

---

Meaning

```text
Maximum

3 Retries
```

---

# Example

Migration Fails

↓

Retry 1

↓

Retry 2

↓

Retry 3

↓

Failed

---

# Parallel Jobs

Some workloads require:

```text
Multiple Parallel Tasks
```

---

Example

```text
Data Processing

Batch Processing
```

---

Configuration

```yaml
completions: 5

parallelism: 2
```

---

Meaning

```text
5 Successful Runs Needed

2 Pods Run Simultaneously
```

---

# Real Production Use Cases

## Database Migration

```text
Application Release

↓

Run Migration Job

↓

Update Schema

↓

Complete
```

---

## Data Import

```text
Import CSV

↓

Process Records

↓

Complete
```

---

## Backup

```text
Take Backup

↓

Store In S3

↓

Complete
```

---

## Cleanup

```text
Delete Old Logs

↓

Complete
```

---

# What Is A CronJob?

CronJob is:

```text
Scheduled Job
```

---

Think Of

```text
Linux Cron
```

---

Example

```text
Every Night

At 2 AM
```

Run Backup.

---

# CronJob Architecture

```text
Cron Schedule

↓

CronJob

↓

Job

↓

Pod

↓

Task

↓

Completed
```

---

# Production Examples

## Nightly Backup

```text
2 AM Every Day
```

---

## Log Cleanup

```text
Every Sunday
```

---

## Report Generation

```text
Every Morning
```

---

## Database Health Check

```text
Every Hour
```

---

# Cron Syntax

```text
* * * * *
│ │ │ │ │
│ │ │ │ └ Day Of Week
│ │ │ └── Month
│ │ └──── Day
│ └────── Hour
└──────── Minute
```

---

Example

```text
0 2 * * *
```

Meaning

```text
2 AM Daily
```

---

# CronJob Example YAML

```yaml
apiVersion: batch/v1
kind: CronJob

metadata:
  name: nightly-backup

spec:

  schedule: "0 2 * * *"

  jobTemplate:

    spec:

      template:

        spec:

          containers:

          - name: backup

            image: amazon/aws-cli

          restartPolicy: Never
```

---

# CronJob Flow

2 AM

↓

CronJob Triggered

↓

Job Created

↓

Pod Created

↓

Backup Executed

↓

Completed

---

# Verify CronJobs

```bash
kubectl get cronjobs
```

---

Output

```text
NAME

nightly-backup
```

---

# Verify Created Jobs

```bash
kubectl get jobs
```

---

Shows

```text
Backup Jobs
```

created by CronJob.

---

# Concurrency Policy

What If Previous Backup Is Still Running?

---

Options

```yaml
concurrencyPolicy: Allow
```

---

Meaning

```text
Run Both Jobs
```

---

Option

```yaml
concurrencyPolicy: Forbid
```

---

Meaning

```text
Skip New Job
```

---

Option

```yaml
concurrencyPolicy: Replace
```

---

Meaning

```text
Stop Old Job

Start New Job
```

---

# Production Backup Example

Application

```text
Roboshop
```

---

Requirement

```text
Backup MongoDB

Every Night
```

---

Flow

```text
CronJob

↓

Mongo Dump

↓

Upload To S3

↓

Completed
```

---

# Job vs CronJob

## Job

```text
Run Once
```

Examples

```text
Migration

Data Import

Cleanup
```

---

## CronJob

```text
Run Repeatedly
```

Examples

```text
Backup

Reports

Monitoring
```

---

# Common Production Problems

## Job Stuck

Symptoms

```text
Active
```

for long time.

---

Verify

```bash
kubectl logs
```

---

# Job Failing Repeatedly

Symptoms

```text
BackoffLimitExceeded
```

---

Cause

```text
Application Failure
```

---

# CronJob Not Running

Verify

```bash
kubectl describe cronjob
```

---

Check

```text
Schedule

Timezone

Events
```

---

# Too Many Completed Jobs

Problem

```text
Thousands Of Completed Jobs
```

---

Solution

```yaml
successfulJobsHistoryLimit: 3
```

---

# Troubleshooting Commands

Jobs

```bash
kubectl get jobs
```

---

Describe Job

```bash
kubectl describe job <job-name>
```

---

Logs

```bash
kubectl logs <pod-name>
```

---

CronJobs

```bash
kubectl get cronjobs
```

---

Describe CronJob

```bash
kubectl describe cronjob <cronjob-name>
```

---

Events

```bash
kubectl get events
```

---

# Production Architecture

```text
Cron Schedule

↓

CronJob

↓

Job

↓

Pod

↓

Execute Task

↓

Completed

↓

Result Stored
```

---

# Common Interview Questions

## What Is A Job?

### Short Answer

A Job is a Kubernetes resource that runs a task until successful completion.

### Detailed Explanation

Jobs are used for one-time tasks such as database migrations, data imports, and cleanup activities.

### Production Example

```text
Application Deployment

↓

Migration Job

↓

Completed
```

### Follow-Up Questions

- What happens if a Job fails?
- What is backoffLimit?
- How do you verify Job status?
- Can Jobs run in parallel?

---

## What Is A CronJob?

### Short Answer

A CronJob schedules Jobs to run at specified times.

### Detailed Explanation

CronJobs are Kubernetes equivalents of Linux cron and are commonly used for backups, reporting, and maintenance tasks.

### Production Example

```text
2 AM Daily

↓

Backup Job

↓

S3 Upload
```

### Follow-Up Questions

- What cron syntax is used?
- How do you prevent overlapping Jobs?
- What happens if a CronJob fails?
- How do you check CronJob history?

---

## Difference Between Job And CronJob?

### Short Answer

Job runs once, while CronJob runs repeatedly based on a schedule.

### Detailed Explanation

Jobs are designed for one-time execution, whereas CronJobs automatically create Jobs according to cron schedules.

### Production Example

Job

```text
Database Migration
```

CronJob

```text
Nightly Backup
```

### Follow-Up Questions

- Can CronJob create multiple Jobs?
- Which one is used for backups?
- Which one is used during deployments?
- Can a Job be manually triggered?

---

## Why Are Jobs Important In Production?

### Short Answer

They automate operational tasks without requiring continuously running applications.

### Detailed Explanation

Jobs and CronJobs handle maintenance, migration, backup, and reporting workloads efficiently.

### Production Example

```text
Deployment

↓

Migration Job

↓

Application Release
```

### Follow-Up Questions

- How do you retry failed Jobs?
- How do you monitor Jobs?
- What happens if Jobs overlap?
- How do you clean old Jobs?

---

# Key Takeaways

```text
Jobs are used for one-time tasks.

CronJobs are scheduled Jobs.

Deployments keep Pods running continuously.

Jobs run until completion.

CronJobs use cron schedules.

Jobs support retries using backoffLimit.

CronJobs support concurrency policies.

Database migrations commonly use Jobs.

Backups commonly use CronJobs.

Jobs and CronJobs are widely used in production Kubernetes environments.
```