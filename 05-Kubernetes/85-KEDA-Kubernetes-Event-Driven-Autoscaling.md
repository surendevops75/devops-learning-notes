# KEDA (Kubernetes Event-Driven Autoscaling)

## Introduction

One of the biggest challenges in Kubernetes autoscaling is:

```text
Not Every Application Is CPU Intensive
```

---

Many applications process:

```text
SQS Messages

Kafka Messages

RabbitMQ Messages

Redis Queues

Azure Service Bus

Database Events
```

---

These applications may have:

```text
Low CPU

Low Memory

Huge Workload
```

---

Traditional Kubernetes autoscaling cannot handle this properly.

---

# Why KEDA Was Created?

Before KEDA, Kubernetes used:

```text
HPA

(Horizontal Pod Autoscaler)
```

---

HPA scales based on:

```text
CPU

Memory
```

---

Example

```text
CPU > 70%

↓

Scale Pods
```

---

This works well for:

```text
Web Applications

REST APIs

Frontend Applications
```

---

But fails for:

```text
Queue Based Applications
```

---

# Real Production Problem

Suppose we have:

```text
Order Processor
```

---

Architecture

```text
Customers

↓

Orders

↓

SQS Queue

↓

Order Processor Pods
```

---

Normal Load

```text
100 Messages
```

Pods

```text
2
```

---

Black Friday Sale

```text
50,000 Messages
```

Queue.

---

CPU Usage

```text
15%
```

---

HPA Sees

```text
CPU Healthy
```

---

Result

```text
No Scaling
```

---

But Queue Keeps Growing.

---

Orders Delayed.

---

Business Impact

```text
Slow Order Processing

Customer Complaints

Revenue Loss
```

---

# Problem Summary

HPA Understands

```text
CPU

Memory
```

---

Business Workload Depends On

```text
Queue Length
```

---

Mismatch.

---

# What Is KEDA?

KEDA stands for:

```text
Kubernetes Event Driven Autoscaling
```

---

KEDA allows Kubernetes to scale based on:

```text
Events

Messages

External Metrics
```

instead of only:

```text
CPU

Memory
```

---

# Main Goal

Convert

```text
Queue Length

Kafka Lag

Message Count
```

into

```text
Pod Scaling Decisions
```

---

# How KEDA Works

KEDA monitors:

```text
External Systems
```

such as:

```text
SQS

Kafka

RabbitMQ

Redis

Prometheus

Azure Service Bus
```

---

When workload increases:

```text
KEDA

↓

Creates HPA

↓

Scales Pods
```

---

# Architecture

```text
SQS Queue

↓

KEDA

↓

HPA

↓

Deployment

↓

Pods
```

---

# KEDA Components

## Scaler

Responsible For

```text
Reading External Metrics
```

---

Examples

```text
SQS Scaler

Kafka Scaler

RabbitMQ Scaler

Redis Scaler
```

---

## ScaledObject

Main Kubernetes Resource.

---

Defines

```text
What To Monitor

When To Scale
```

---

# Installation

Install KEDA

```bash
helm repo add kedacore https://kedacore.github.io/charts

helm repo update

helm install keda kedacore/keda \
  --namespace keda \
  --create-namespace
```

---

Verify

```bash
kubectl get pods -n keda
```

---

# Production Example

Order Processor Deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: order-processor

spec:

  replicas: 1

  selector:
    matchLabels:
      app: order-processor

  template:

    metadata:
      labels:
        app: order-processor

    spec:

      containers:

      - name: app

        image: order-processor:v1
```

---

# SQS Based Scaling

## Problem

Queue Length

```text
10

↓

100

↓

1000

↓

10000
```

---

Need More Pods.

---

# ScaledObject

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject

metadata:
  name: order-processor

spec:

  scaleTargetRef:
    name: order-processor

  minReplicaCount: 1

  maxReplicaCount: 50

  triggers:

  - type: aws-sqs-queue

    metadata:

      queueURL: https://sqs.us-east-1.amazonaws.com/123456789/orders

      queueLength: "10"

      awsRegion: us-east-1
```

---

# What Happens?

Queue

```text
0 Messages
```

---

Pods

```text
1
```

---

Queue

```text
100 Messages
```

---

Pods

```text
5
```

---

Queue

```text
1000 Messages
```

---

Pods

```text
25
```

---

Queue

```text
10000 Messages
```

---

Pods

```text
50
```

---

Automatically.

---

# Scale To Zero

One Huge Advantage.

---

Traditional HPA

```text
Minimum 1 Pod
```

---

KEDA Supports

```text
0 Pods
```

---

Flow

```text
No Messages

↓

0 Pods
```

---

Message Arrives

```text
1 Message

↓

1 Pod Starts
```

---

Cost Savings.

---

# Kafka Example

Architecture

```text
Kafka Topic

↓

Consumer Pods
```

---

Problem

```text
Kafka Lag Growing
```

---

KEDA Monitors

```text
Consumer Lag
```

---

Scales Consumers.

---

# RabbitMQ Example

Queue

```text
50000 Messages
```

---

KEDA

```text
Scale

2 Pods

↓

40 Pods
```

---

# Prometheus Example

Can Scale Using

```text
Custom Metrics
```

---

Example

```text
Requests Per Second

Active Users

Business Transactions
```

---

# Production Flow

```text
Black Friday Sale

↓

Orders Increase

↓

SQS Queue Grows

↓

KEDA Detects

↓

HPA Updated

↓

Pods Scale

↓

Orders Processed
```

---

# KEDA vs HPA

## HPA

Scales Using

```text
CPU

Memory
```

---

Best For

```text
Web Applications
```

---

## KEDA

Scales Using

```text
Queue Length

Kafka Lag

Events

External Metrics
```

---

Best For

```text
Event Driven Systems
```

---

# KEDA + HPA

Important

KEDA Does Not Replace HPA.

---

KEDA Creates

```text
HPA
```

behind the scenes.

---

Flow

```text
KEDA

↓

External Metric

↓

HPA

↓

Pods
```

---

# Real Production Use Cases

## Order Processing

```text
SQS

↓

Consumers
```

---

## Email Processing

```text
RabbitMQ

↓

Email Workers
```

---

## Log Processing

```text
Kafka

↓

Consumers
```

---

## Payment Processing

```text
Queue

↓

Workers
```

---

## ETL Systems

```text
Events

↓

Data Processing Pods
```

---

# Common Production Problems

## KEDA Not Scaling

Check

```bash
kubectl describe scaledobject
```

---

Usually

```text
Wrong Queue URL

Wrong Credentials

Missing Permissions
```

---

## Pods Not Scaling

Verify

```bash
kubectl get hpa
```

---

KEDA Creates HPA Automatically.

---

## AWS Access Errors

Verify

```text
IRSA

IAM Policies
```

---

# Troubleshooting Commands

Scaled Objects

```bash
kubectl get scaledobjects
```

---

Describe

```bash
kubectl describe scaledobject order-processor
```

---

HPA

```bash
kubectl get hpa
```

---

KEDA Pods

```bash
kubectl get pods -n keda
```

---

Logs

```bash
kubectl logs \
-n keda \
deployment/keda-operator
```

---

# Common Interview Questions

## Why Was KEDA Created?

### Short Answer

KEDA was created to scale applications using external events and metrics instead of only CPU and memory.

### Detailed Explanation

Traditional HPA works well for web applications but cannot understand queue length, Kafka lag, or business events. KEDA bridges this gap by monitoring external systems and converting those metrics into Kubernetes scaling actions.

### Production Example

```text
SQS Queue

50000 Messages

↓

KEDA Detects

↓

Scale 2 Pods

↓

50 Pods
```

### Follow-Up Questions

- Why can't HPA solve this problem?
- What metrics does KEDA support?
- Does KEDA replace HPA?
- Can KEDA scale to zero?

---

## Difference Between HPA And KEDA?

### Short Answer

HPA scales using CPU and memory while KEDA scales using external event sources and business metrics.

### Detailed Explanation

HPA depends on Kubernetes metrics, whereas KEDA integrates with external systems like Kafka, RabbitMQ, Redis, and SQS to make scaling decisions.

### Production Example

```text
CPU = 10%

Queue = 50000 Messages

HPA → No Scaling

KEDA → Scale Pods
```

### Follow-Up Questions

- Can KEDA work with HPA?
- Which workloads use KEDA?
- Which workloads use HPA?
- Can both run together?

---

## What Is Scale To Zero?

### Short Answer

KEDA can reduce replicas to zero when no workload exists.

### Detailed Explanation

Unlike traditional HPA, KEDA can completely stop worker pods when queues are empty and automatically restart them when work arrives.

### Production Example

```text
Queue Empty

↓

0 Pods

↓

Message Arrives

↓

1 Pod Starts
```

### Follow-Up Questions

- Why is scale-to-zero useful?
- Which workloads benefit from it?
- Does HPA support it?
- Any startup latency concerns?

---

## How Have You Used KEDA In Production?

### Short Answer

KEDA is commonly used for queue-based worker applications such as order processing, ETL jobs, event processing, and message consumers.

### Detailed Explanation

Instead of scaling workers using CPU, KEDA monitors queue depth and scales consumers based on actual workload demand.

### Production Example

```text
Order Queue

↓

50000 Messages

↓

KEDA

↓

50 Worker Pods

↓

Queue Drained
```

### Follow-Up Questions

- Which queue system did you use?
- How did you authenticate?
- How did you monitor scaling?
- What was the max replica count?

---

# Key Takeaways

```text
KEDA solves event-driven autoscaling problems.

HPA only understands CPU and memory.

KEDA understands queues, events, and external metrics.

KEDA works with SQS, Kafka, RabbitMQ, Redis, and Prometheus.

KEDA creates HPA resources automatically.

KEDA supports scale-to-zero.

KEDA is widely used for worker and consumer applications.

KEDA is ideal for queue-based and event-driven architectures.
```