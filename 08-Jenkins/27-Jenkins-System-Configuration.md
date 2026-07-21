# Jenkins System Configuration

## Introduction

Jenkins System Configuration is the process of configuring global settings that control how the Jenkins controller behaves and interacts with users, build agents, external tools, cloud platforms, and CI/CD integrations.

Unlike job-level configuration, system configuration is shared across the entire Jenkins instance. Every pipeline, freestyle job, multibranch pipeline, and shared library can use these centralized settings.

A properly configured Jenkins server is easier to maintain, more secure, and provides a consistent experience for all development teams.

---

# Why Do We Need System Configuration?

Without proper system configuration, every pipeline would need to configure tools, integrations, and services individually.

This leads to:

- Duplicate configuration
- Human errors
- Difficult maintenance
- Inconsistent pipelines
- Security risks

Instead, Jenkins allows administrators to configure common settings once and reuse them everywhere.

---

# Without System Configuration

```text
Pipeline 1

↓

Configure Git

↓

Configure Maven

↓

Configure Docker

↓

Configure SonarQube

────────────────────────────

Pipeline 2

↓

Configure Git

↓

Configure Maven

↓

Configure Docker

↓

Configure SonarQube

────────────────────────────

Pipeline 3

↓

Same Configuration Again
```

Problems

- Duplicate configuration
- Hard to maintain
- High chance of mistakes
- Time consuming

---

# With System Configuration

```text
Administrator

↓

Manage Jenkins

↓

Configure Once

↓

Git

Maven

Docker

SonarQube

Slack

Email

↓

All Pipelines Reuse Configuration
```

Benefits

- Centralized management
- Easy maintenance
- Consistent configuration
- Faster pipeline development
- Better security

---

# Where is System Configuration?

Navigate to

```text
Dashboard

↓

Manage Jenkins
```

From here administrators can access

```text
Manage Jenkins

│

├── System

├── Tools

├── Credentials

├── Nodes

├── Clouds

├── Security

├── Plugins

└── Appearance
```

Most global settings are configured here.

---

# System Configuration Architecture

```text
                      Developers

                           │

                           ▼

                  Jenkins Controller

                           │

      ┌────────────────────┼────────────────────┐

      ▼                    ▼                    ▼

 Global Config         Global Tools        Credentials

      │                    │                    │

      ▼                    ▼                    ▼

 Git                 Maven              GitHub Token

 Docker              JDK                AWS Keys

 SonarQube           Terraform          SSH Keys

 Slack               NodeJS             Docker Login

 Email               Helm

      └────────────────────┼────────────────────┘

                           ▼

                  Production Pipelines
```

---

# Configuration Categories

Jenkins system configuration can be divided into several categories.

```text
Manage Jenkins

│

├── System Configuration

├── Tool Configuration

├── Credentials

├── Cloud Configuration

├── Security

├── Nodes

├── Plugins

└── Monitoring
```

Each category manages a different part of Jenkins.

---

# What Will We Configure?

In this chapter we'll configure:

- Jenkins URL
- System Message
- Global Environment Variables
- Number of Executors
- Quiet Period
- SCM Retry Count
- Usage Statistics
- Markup Formatter
- Global Tool Configuration
- Git Configuration
- Maven Configuration
- JDK Configuration
- Docker Configuration
- NodeJS Configuration
- Terraform Configuration
- SonarQube Configuration
- Email Configuration
- Slack Configuration
- Cloud Configuration
- Production Best Practices

---

# Production Example

Assume your organization has 500 Jenkins pipelines.

Without centralized configuration,

every Jenkinsfile would need to define:

- Git installation
- Maven installation
- Docker path
- SonarQube server
- SMTP server
- Slack configuration

This creates hundreds of duplicate configurations.

Instead,

Jenkins administrators configure them once.

```text
Administrator

↓

Configure Git

↓

Configure Maven

↓

Configure Docker

↓

Configure SonarQube

↓

Configure Slack

↓

Every Pipeline Uses Same Configuration
```

---

# Real Production Architecture

```text
                     Developers

                          │

                          ▼

                      GitHub

                          │

                          ▼

                 Jenkins Controller

                          │

      ┌───────────────────┼────────────────────┐

      ▼                   ▼                    ▼

 Global Tools       Credentials          System Config

      │                   │                    │

      ▼                   ▼                    ▼

 Git              AWS Credentials      Jenkins URL

 Maven            Docker Login         Email

 Docker           Sonar Token          Slack

 Helm             GitHub PAT           SonarQube

 Terraform         SSH Keys            Global Variables

      └───────────────────┼────────────────────┘

                          ▼

                 Production Pipelines

                          ▼

                     Amazon EKS
```

---

# Learning Objectives

After completing this chapter, you will be able to:

- Configure Jenkins for enterprise environments.
- Manage global settings.
- Configure development tools.
- Configure cloud integrations.
- Configure notifications.
- Configure SonarQube.
- Configure Docker.
- Configure Kubernetes Cloud.
- Understand production configuration practices.
- Troubleshoot common configuration issues.

---

# Chapter Flow

```text
Introduction

↓

System Configuration

↓

Global Tool Configuration

↓

Notification Configuration

↓

Cloud Configuration

↓

Production Architecture

↓

Best Practices

↓

Troubleshooting

↓

Interview Questions

↓

Key Takeaways
```
---

# Jenkins URL

## What is Jenkins URL?

The **Jenkins URL** is the globally configured address of the Jenkins controller.

It is used by Jenkins to generate links for:

- Build pages
- Console logs
- Email notifications
- Slack notifications
- REST API
- GitHub webhooks
- Plugin integrations

Every user and external system communicates with Jenkins using this URL.

---

# Why Do We Need Jenkins URL?

Imagine a developer receives a build failure email.

The email contains a link like:

```text
Open Build

↓

https://jenkins.company.com/job/catalog-service/152/
```

The developer clicks the link and immediately opens the failed build.

Without a proper Jenkins URL,

the email may contain

```text
http://localhost:8080/job/catalog-service/152/
```

Only the Jenkins server itself can access localhost.

Everyone else receives an error.

---

# Without Jenkins URL

```text
Build Finished

↓

Email Generated

↓

localhost Link

↓

Developer Clicks

↓

Cannot Open Build
```

Problems

- Broken email links
- Broken Slack links
- Failed webhook callbacks
- API clients cannot connect
- Plugins generate invalid URLs

---

# With Proper Jenkins URL

```text
Build Finished

↓

Notification Generated

↓

https://jenkins.company.com

↓

Developer Opens Build

↓

Issue Fixed Quickly
```

Benefits

- Working notifications
- Correct build URLs
- Reliable webhooks
- Successful integrations
- Better user experience

---

# Where to Configure?

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

System

↓

Jenkins Location

↓

Jenkins URL
```

---

# Example Configuration

Production

```text
https://jenkins.company.com/
```

Development

```text
http://dev-jenkins.company.local:8080/
```

---

# Good Configuration

```text
https://jenkins.company.com/
```

Uses

- HTTPS
- DNS
- SSL Certificate

---

# Bad Configuration

```text
http://localhost:8080
```

Reason

Only works from the Jenkins server.

---

Another bad example

```text
http://10.0.12.15:8080
```

Reason

Private IPs may not be reachable by developers or external services.

---

# Jenkins URL Architecture

```text
                 Developers

                      │

                      ▼

         https://jenkins.company.com

                      │

                      ▼

                Load Balancer

                      │

                      ▼

                Reverse Proxy

                      │

                      ▼

              Jenkins Controller
```

---

# Production Architecture

```text
Internet

↓

HTTPS

↓

NGINX

↓

Jenkins

↓

Port 8080
```

Users never directly access

```text
http://server-ip:8080
```

Instead they always use

```text
https://jenkins.company.com
```

---

# Why Reverse Proxy?

Instead of exposing Jenkins directly,

production environments use

```text
Browser

↓

HTTPS

↓

NGINX

↓

Jenkins
```

Advantages

- SSL termination
- Security
- Friendly URLs
- Better performance
- Load balancing
- Access control

---

# Jenkins URL Usage

Many Jenkins components depend on this URL.

```text
Jenkins URL

│

├── Email Notifications

├── Slack Notifications

├── GitHub Webhooks

├── REST API

├── Blue Ocean

├── Plugin Integrations

├── Console Links

└── Build URLs
```

---

# Example 1

Developer pushes code.

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Pipeline

↓

Build Success

↓

Email
```

Email contains

```text
https://jenkins.company.com/job/catalog-service/152/
```

---

# Example 2

Slack notification

```text
Pipeline

↓

Success

↓

Slack

↓

Open Build

↓

https://jenkins.company.com/job/catalog-service/152/
```

---

# Example 3

REST API

Applications interact using

```text
GET

https://jenkins.company.com/api/json
```

Example response

```json
{
  "jobs": [
    {
      "name": "catalog-service"
    }
  ]
}
```

---

# GitHub Webhook

GitHub sends events to Jenkins.

```text
GitHub

↓

Webhook

↓

https://jenkins.company.com/github-webhook/

↓

Jenkins

↓

Pipeline Triggered
```

If the URL is incorrect,

automatic builds stop working.

---

# Email Notification Example

```text
Subject

Build Failed

------------------------

Job

catalog-service

Build

#152

Console

https://jenkins.company.com/job/catalog-service/152/console
```

---

# Blue Ocean

Blue Ocean also depends on the configured Jenkins URL.

```text
Developer

↓

Blue Ocean

↓

Pipeline Visualization

↓

Open Stage Logs
```

---

# Plugins Using Jenkins URL

Common plugins

```text
GitHub Plugin

↓

Slack Plugin

↓

Email Extension Plugin

↓

Blue Ocean

↓

Bitbucket Plugin

↓

GitLab Plugin
```

All generate URLs using the global Jenkins URL.

---

# HTTPS Recommendation

Always prefer

```text
https://jenkins.company.com
```

Avoid

```text
http://jenkins.company.com
```

HTTPS protects

- Login credentials
- API Tokens
- Session cookies
- Build information

---

# SSL Architecture

```text
Developer

↓

HTTPS

↓

NGINX

↓

SSL Certificate

↓

Jenkins
```

---

# Common Mistakes

## Using localhost

```text
http://localhost:8080
```

Only works locally.

---

## Using Server IP

```text
http://192.168.1.25:8080
```

Not suitable for enterprise environments.

---

## Missing HTTPS

```text
http://jenkins.company.com
```

Traffic is unencrypted.

---

## Wrong DNS

```text
https://jenkins.internal
```

External services cannot resolve it.

---

## Missing Trailing Slash

Instead of

```text
https://jenkins.company.com
```

Prefer

```text
https://jenkins.company.com/
```

Some plugins generate cleaner URLs.

---

# Verification Checklist

After configuring Jenkins URL verify

```text
Login Page

✔

------------------------

Email Links

✔

------------------------

Slack Links

✔

------------------------

REST API

✔

------------------------

Webhook Trigger

✔

------------------------

Blue Ocean

✔
```

---

# Production CI/CD Example

```text
Developer Push

↓

GitHub

↓

Webhook

↓

https://jenkins.company.com

↓

Pipeline

↓

SonarQube

↓

Docker

↓

Amazon ECR

↓

Amazon EKS

↓

Slack Notification

↓

Developer Opens Build URL
```

---

# Best Practices

- Always use HTTPS.
- Use a DNS hostname instead of an IP address.
- Place Jenkins behind NGINX or Apache.
- Never configure localhost in production.
- Verify webhook callbacks after URL changes.
- Test email and Slack notifications.
- Renew SSL certificates before expiry.
- Use a consistent URL across all integrations.

---

# Key Points

- Jenkins URL is the global address of the Jenkins controller.
- It is required for notifications, APIs, webhooks, and plugins.
- Production environments should use HTTPS and a DNS hostname.
- Reverse proxies such as NGINX improve security and manage incoming traffic.
- Always verify integrations after modifying the Jenkins URL.

---
# System Message

## What is a System Message?

A **System Message** is a global message displayed on the Jenkins dashboard.

It is used by administrators to communicate important information to all Jenkins users.

Unlike email or Slack notifications, the System Message is visible whenever users log in to Jenkins.

---

# Why Do We Need a System Message?

In large organizations, hundreds of developers use the same Jenkins server.

Instead of sending emails every time,

administrators can display important announcements directly on the dashboard.

Examples include:

- Planned maintenance
- Upgrade notifications
- New policies
- Security announcements
- Contact information
- Environment details

---

# Without System Message

```text
Administrator

↓

Maintenance Planned

↓

No Notification

↓

Developers Start Builds

↓

Builds Interrupted
```

Problems

- Poor communication
- Unexpected build failures
- Developer confusion
- Increased support requests

---

# With System Message

```text
Administrator

↓

Configure System Message

↓

Users Login

↓

See Announcement

↓

Avoid Running Pipelines
```

Benefits

- Better communication
- Fewer failed builds
- Reduced support effort
- Improved user awareness

---

# Where to Configure?

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

System

↓

System Message
```

---

# Example System Message

```text
⚠ Jenkins Maintenance

Production Jenkins will undergo maintenance
on Saturday from 10:00 PM to 11:30 PM IST.

Please avoid starting long-running builds.
```

---

# Another Example

```text
Welcome to Production Jenkins

Supported Languages

• Java
• Python
• NodeJS

For support contact

devops@company.com
```

---

# System Message Architecture

```text
Administrator

↓

Configure Message

↓

Save

↓

Jenkins Dashboard

↓

Visible To All Users
```

---

# Production Example

Suppose Jenkins will be upgraded.

Instead of informing everyone individually,

display

```text
⚠ Jenkins Upgrade

Date:
25 July

Time:
09:00 PM

Expected Downtime:
30 Minutes
```

Every developer sees the announcement after login.

---

# Typical Enterprise Messages

## Maintenance Notice

```text
⚠ Scheduled Maintenance

Sunday

11 PM–12 AM

Avoid triggering builds.
```

---

## Security Notice

```text
Security Update

Rotate API Tokens before
30 September.
```

---

## Environment Information

```text
Production Jenkins

Region

Mumbai

Version

Jenkins LTS 2.516.x
```

---

## Contact Information

```text
DevOps Support

Email

devops@company.com

Slack

#devops-support
```

---

# Dashboard View

```text
+----------------------------------------------------+

             Production Jenkins

------------------------------------------------------

⚠ Maintenance Window

Saturday

10 PM–11 PM

Please avoid running deployments.

------------------------------------------------------

Jobs

Folders

Build Queue

Recent Builds

+----------------------------------------------------+
```

---

# When Should You Update It?

Update the message whenever there is:

- Planned maintenance
- Major upgrade
- Infrastructure migration
- Security advisory
- New organizational policy
- Temporary service disruption

---

# What Should Not Be Added?

Do **not** include:

- Passwords
- API Tokens
- AWS Keys
- Internal secrets
- Personal information

The System Message is visible to anyone with access to Jenkins.

---

# Best Practices

- Keep the message short and easy to read.
- Display only important announcements.
- Remove outdated messages after the event.
- Mention maintenance windows clearly.
- Include contact details when necessary.
- Avoid exposing confidential information.

---

# Common Mistakes

## Message Too Long

```text
Entire company policy document...
```

Avoid lengthy announcements.

---

## Outdated Message

```text
Maintenance

January 2025
```

Remove old messages after completion.

---

## Sensitive Information

```text
AWS Access Key

AKIA***********
```

Never display credentials.

---

# Enterprise Workflow

```text
Administrator

↓

Maintenance Planned

↓

Update System Message

↓

Developers Login

↓

Read Announcement

↓

Avoid Production Builds

↓

Maintenance Completed

↓

Remove Message
```

---

# Production Example

```text
DevOps Team

↓

Jenkins Upgrade Planned

↓

System Message Updated

↓

Developers Notified

↓

Upgrade Completed

↓

Message Removed
```

---

# Key Points

- A System Message is a global announcement displayed on the Jenkins dashboard.
- It helps communicate maintenance windows, upgrades, and important notices.
- Every logged-in user can see the message.
- Keep messages short, relevant, and up to date.
- Never include passwords, API tokens, or other sensitive information.
