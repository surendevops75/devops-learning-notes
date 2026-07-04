# Zero Downtime vs Planned Downtime Deployments

## Introduction

Applications require regular deployments for:

```text
New Features

Bug Fixes

Security Updates

Performance Improvements
```

The biggest question during deployment is:

```text
Should Users Continue Accessing The Application?

OR

Can We Stop The Application Temporarily?
```

This leads to two deployment approaches:

```text
Zero Downtime Deployment

Planned Downtime Deployment
```

---

# What is Planned Downtime?

Planned Downtime means:

```text
Application Is Temporarily Unavailable
```

during deployment.

Users are informed in advance.

---

# Example

```text
Maintenance Window

12:00 AM - 06:00 AM
```

Announcement:

```text
System Maintenance In Progress

Application Will Be Unavailable
```

---

# Planned Downtime Deployment Flow

```text
Announce Maintenance Window
          ↓
Stop Application
          ↓
Deploy New Version
          ↓
Validate Application
          ↓
Start Application
          ↓
End Maintenance Window
```

---

# Production Example

Banking Application:

```text
12:00 AM - 06:00 AM
```

During this period:

```text
Net Banking Disabled

Transactions Blocked

Updates Performed
```

---

# Advantages

```text
Simple Deployment

Less Infrastructure Complexity

Easy Troubleshooting
```

---

# Disadvantages

```text
Application Unavailable

Poor User Experience

Business Impact
```

---

# What is Zero Downtime Deployment?

Zero Downtime Deployment means:

```text
Application Remains Available

Throughout Deployment
```

Users continue accessing services while updates occur.

---

# Goal

```text
Deploy New Version

Without Service Interruption
```

---

# Common Techniques

```text
Rolling Updates

Blue-Green Deployment

Canary Deployment

ASG Instance Refresh
```

---

# Rolling Update Approach

Current:

```text
Catalogue-v1
```

Running On:

```text
Catalogue-1

Catalogue-2

Catalogue-3

Catalogue-4
```

---

New Release:

```text
Catalogue-v2
```

---

# Deployment Flow

```text
Create New Instance
         ↓
Health Check
         ↓
Delete One Old Instance
         ↓
Repeat
```

---

# Example

Step 1

```text
Create Catalogue-5
```

using:

```text
catalogue-v2
```

---

Step 2

```text
Health Check Passed
```

---

Step 3

```text
Terminate Catalogue-1
```

---

Repeat until all servers use:

```text
catalogue-v2
```

---

# HR Analogy

Current Team:

```text
10 Employees
```

Need to replace them.

Wrong Approach:

```text
Fire All 10

Hire New 10
```

Result:

```text
No Work Happens
```

---

Correct Approach:

```text
Recruit One Person

Give KT

Release One Old Employee

Repeat
```

Work continues smoothly.

---

# Same Concept in AWS

```text
Launch New Instance

Verify Health

Remove Old Instance
```

---

# During Deployment

Both versions may run simultaneously.

Example:

```text
Catalogue-v1

Catalogue-v2
```

---

# Why Is This Safe?

Because:

```text
Traffic Continues

Capacity Maintained

No Outage
```

---

# Role of Load Balancer

Traffic Flow:

```text
User
   ↓
ALB
   ↓
Target Group
   ↓
Healthy Instances
```

---

# Health Check Validation

Before removing an old server:

AWS verifies:

```text
New Server Healthy
```

---

Example:

```bash
curl http://catalogue.backend-alb-dev.daws86s.fun/health
```

---

Expected Response:

```text
Healthy
```

---

# If Validation Fails

AWS:

```text
Stops Deployment
```

and continues serving traffic through healthy servers.

---

# Planned Downtime Example

Deployment Process:

```text
terraform apply -auto-approve
```

---

Actions:

```text
Stop Existing Application

Deploy New Version

Start Application
```

---

Result:

```text
Users Cannot Access Application
```

during deployment.

---

# Zero Downtime Example

Deployment Process:

```text
terraform apply -auto-approve
```

---

Actions:

```text
Create New AMI

Update Launch Template

ASG Refresh

Rolling Update
```

---

Result:

```text
Application Remains Available
```

---

# Comparison

## Planned Downtime

```text
Application Stopped

Users Affected

Simple Process

Lower Infrastructure Cost
```

---

## Zero Downtime

```text
Application Available

Users Not Affected

More Complex

Higher Reliability
```

---

# Production Considerations

Critical Systems:

```text
Banking

E-Commerce

Healthcare

Payment Systems
```

usually require:

```text
Zero Downtime Deployments
```

---

Internal Systems:

```text
Development

Testing

Non-Critical Applications
```

may use:

```text
Planned Downtime
```

---

# Complete Zero Downtime Architecture

```text
New Code
    ↓
Terraform Apply
    ↓
Create New AMI
    ↓
Launch Template Version
    ↓
ASG Refresh
    ↓
Rolling Update
    ↓
ALB Health Checks
    ↓
Production Traffic
```

---

# Benefits of Zero Downtime Deployments

```text
No Service Interruption

Better User Experience

Reduced Business Impact

Safer Rollouts

Improved Availability
```

---

# Common Interview Questions

## What is Zero Downtime Deployment?

### Short Answer

A deployment strategy where the application remains available during updates.

### Detailed Explanation

New application instances are deployed and validated before old instances are removed, ensuring uninterrupted service.

### Production Example

ASG Instance Refresh replacing Catalogue-v1 with Catalogue-v2.

### Follow-Up Questions

- What AWS services help achieve zero downtime?
- What deployment strategies support zero downtime?

---

## What is Planned Downtime?

### Short Answer

A deployment approach where the application is intentionally stopped during updates.

### Detailed Explanation

Users are informed about a maintenance window while upgrades are performed.

### Production Example

Banking application maintenance scheduled from 12:00 AM to 06:00 AM.

### Follow-Up Questions

- Why would organizations choose planned downtime?
- What are its disadvantages?

---

## Why is Rolling Update Considered Zero Downtime?

### Short Answer

Because old servers remain available while new servers are deployed.

### Detailed Explanation

New instances are validated before old instances are terminated, ensuring continuous service.

### Production Example

Replacing Catalogue-v1 servers one at a time with Catalogue-v2 servers.

### Follow-Up Questions

- Can old and new versions run simultaneously?
- What role do health checks play?

---

## Which Deployment Strategy Would You Recommend in Production?

### Short Answer

For customer-facing systems, Zero Downtime Deployments are preferred.

### Detailed Explanation

They minimize user impact, improve availability, and reduce business risk.

### Production Example

Using ASG Refresh and Rolling Updates for production microservices running behind ALBs.

### Follow-Up Questions

- What are alternatives to Rolling Updates?
- What is Blue-Green Deployment?

---

# Key Takeaways

```text
Applications require regular deployments.

Planned Downtime makes applications temporarily unavailable.

Maintenance windows are used for planned downtime.

Zero Downtime Deployments keep applications available during updates.

Rolling Updates are a common zero downtime deployment strategy.

ASG Instance Refresh automates rolling updates.

Health Checks validate new servers before removing old servers.

Old and new application versions may run simultaneously.

Critical production systems typically require zero downtime deployments.

Zero downtime deployments improve reliability and user experience.
```
