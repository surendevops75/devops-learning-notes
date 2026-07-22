# Jenkins Upgrade and Maintenance

## Introduction

A Jenkins server is never a "set it and forget it" application.

New releases introduce:

- Security patches
- Bug fixes
- Performance improvements
- Plugin compatibility updates
- New features

Without regular maintenance, Jenkins becomes vulnerable, unstable, and difficult to support.

A production Jenkins administrator must plan upgrades carefully to minimize downtime while ensuring compatibility.

---

# Why Upgrade Jenkins?

Imagine a Jenkins server that hasn't been updated for two years.

```text
Jenkins

↓

Old Version

↓

Outdated Plugins

↓

Security Vulnerabilities

↓

Build Failures

↓

Production Risk
```

Eventually,

- Plugins stop working.
- Security vulnerabilities remain unpatched.
- Java versions become unsupported.
- New integrations fail.

---

# Why Maintenance is Important?

Even without upgrades,

Jenkins requires regular maintenance.

Examples

- Disk cleanup
- Workspace cleanup
- Plugin review
- User cleanup
- Backup verification
- JVM tuning
- Log rotation

Without maintenance,

```text
Controller

↓

Disk Full

↓

Queue Grows

↓

Slow Builds

↓

Developers Affected
```

---

# Upgrade vs Maintenance

| Upgrade | Maintenance |
|----------|-------------|
| Install newer version | Keep existing installation healthy |
| New features | Stability |
| Security patches | Performance |
| Version change | Regular operational tasks |

---

# Jenkins Release Types

Jenkins provides two release channels.

```text
Jenkins Releases

│

├── LTS (Long-Term Support)

└── Weekly Releases
```

---

# LTS (Long-Term Support)

LTS releases are recommended for production.

Characteristics

- Stable
- Thoroughly tested
- Security patches
- Bug fixes
- Long-term support

Example

```text
Production

↓

Jenkins LTS

↓

Reliable CI/CD
```

---

# Weekly Releases

Weekly releases provide

- Latest features
- Faster bug fixes
- Experimental improvements

Example

```text
Development Lab

↓

Weekly Version

↓

Testing New Features
```

Not generally recommended for enterprise production environments.

---

# Production Recommendation

```text
Development

↓

Weekly (Optional)

----------------------

Production

↓

LTS
```

---

# Upgrade Architecture

```text
                 Production Jenkins

                         │

                  Backup Created

                         │

                  Test Environment

                         │

               Upgrade Validation

                         │

                  Production Upgrade

                         │

                 Health Verification
```

Never upgrade production directly without testing.

---

# Upgrade Lifecycle

```text
Check Release Notes

↓

Verify Compatibility

↓

Create Backup

↓

Test Upgrade

↓

Schedule Maintenance

↓

Upgrade Production

↓

Health Checks

↓

Monitor
```

---

# Pre-Upgrade Checklist

Before every upgrade,

verify

- Backup completed
- Restore tested
- Plugin compatibility
- Java compatibility
- Available disk space
- Maintenance window approved
- Rollback plan documented

---

# Production Upgrade Workflow

```text
Current Version

↓

Backup

↓

Upgrade Test Environment

↓

Run Pipelines

↓

Validate Plugins

↓

Approve

↓

Upgrade Production
```

---

# Checking Current Version

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

System Information
```

or

```text
Manage Jenkins

↓

About Jenkins
```

---

# Checking Plugin Compatibility

Before upgrading,

review

```text
Installed Plugins

↓

Available Updates

↓

Compatibility

↓

Known Issues
```

Avoid upgrading incompatible plugins.

---

# Plugin Dependency Check

Example

```text
Docker Plugin

↓

Requires

↓

Credentials Plugin

↓

Requires

↓

Pipeline Plugin
```

Upgrading one plugin may require upgrading several dependencies.

---

# Java Compatibility

Every Jenkins version supports specific Java versions.

Example

```text
Jenkins

↓

Java 17

↓

Supported

---------------------

Jenkins

↓

Java 8

↓

Unsupported
```

Always verify Java compatibility before upgrading.

---

# Upgrade Environment

Never test upgrades directly in production.

```text
Production

↓

Backup

↓

Clone

↓

Testing Server

↓

Upgrade

↓

Validate

↓

Production Upgrade
```

---

# Maintenance Window

Production upgrades should occur during approved maintenance windows.

Example

```text
Sunday

02:00 AM

↓

Maintenance

↓

Upgrade

↓

Verification

↓

Service Restored
```

---

# Enterprise Example

```text
Friday

↓

Backup

↓

Saturday

↓

Testing

↓

Sunday

↓

Production Upgrade

↓

Monday

↓

Developers Start Work
```

This minimizes business impact.

---

# Upgrade Flow Diagram

```text
Download New Version

↓

Backup Jenkins

↓

Stop Jenkins

↓

Install New Version

↓

Start Jenkins

↓

Verify Health

↓

Run Test Pipeline

↓

Upgrade Complete
```

---

# Health Checks After Upgrade

Immediately verify

- Jenkins starts successfully
- Plugins load correctly
- Agents reconnect
- Pipelines execute
- Credentials available
- Web UI accessible
- Authentication working

---

# Best Practices

- Always use LTS releases for production.
- Never upgrade without a verified backup.
- Test upgrades in a staging environment.
- Read release notes before upgrading.
- Upgrade during scheduled maintenance windows.
- Verify plugin and Java compatibility before installation.

---

# Key Points

- Production environments should use Jenkins LTS releases.
- Every upgrade should begin with a verified backup.
- Test upgrades before applying them to production.
- Plugin and Java compatibility must be validated.
- Post-upgrade health checks are mandatory before returning Jenkins to users.

---

# 32 - Jenkins Upgrade and Maintenance

# Introduction

A Jenkins server is never a "set it and forget it" application.

New releases introduce:

- Security patches
- Bug fixes
- Performance improvements
- Plugin compatibility updates
- New features

Without regular maintenance, Jenkins becomes vulnerable, unstable, and difficult to support.

A production Jenkins administrator must plan upgrades carefully to minimize downtime while ensuring compatibility.

---

# Why Upgrade Jenkins?

Imagine a Jenkins server that hasn't been updated for two years.

```text
Jenkins

↓

Old Version

↓

Outdated Plugins

↓

Security Vulnerabilities

↓

Build Failures

↓

Production Risk
```

Eventually,

- Plugins stop working.
- Security vulnerabilities remain unpatched.
- Java versions become unsupported.
- New integrations fail.

---

# Why Maintenance is Important?

Even without upgrades,

Jenkins requires regular maintenance.

Examples

- Disk cleanup
- Workspace cleanup
- Plugin review
- User cleanup
- Backup verification
- JVM tuning
- Log rotation

Without maintenance,

```text
Controller

↓

Disk Full

↓

Queue Grows

↓

Slow Builds

↓

Developers Affected
```

---

# Upgrade vs Maintenance

| Upgrade | Maintenance |
|----------|-------------|
| Install newer version | Keep existing installation healthy |
| New features | Stability |
| Security patches | Performance |
| Version change | Regular operational tasks |

---

# Jenkins Release Types

Jenkins provides two release channels.

```text
Jenkins Releases

│

├── LTS (Long-Term Support)

└── Weekly Releases
```

---

# LTS (Long-Term Support)

LTS releases are recommended for production.

Characteristics

- Stable
- Thoroughly tested
- Security patches
- Bug fixes
- Long-term support

Example

```text
Production

↓

Jenkins LTS

↓

Reliable CI/CD
```

---

# Weekly Releases

Weekly releases provide

- Latest features
- Faster bug fixes
- Experimental improvements

Example

```text
Development Lab

↓

Weekly Version

↓

Testing New Features
```

Not generally recommended for enterprise production environments.

---

# Production Recommendation

```text
Development

↓

Weekly (Optional)

----------------------

Production

↓

LTS
```

---

# Upgrade Architecture

```text
                 Production Jenkins

                         │

                  Backup Created

                         │

                  Test Environment

                         │

               Upgrade Validation

                         │

                  Production Upgrade

                         │

                 Health Verification
```

Never upgrade production directly without testing.

---

# Upgrade Lifecycle

```text
Check Release Notes

↓

Verify Compatibility

↓

Create Backup

↓

Test Upgrade

↓

Schedule Maintenance

↓

Upgrade Production

↓

Health Checks

↓

Monitor
```

---

# Pre-Upgrade Checklist

Before every upgrade,

verify

- Backup completed
- Restore tested
- Plugin compatibility
- Java compatibility
- Available disk space
- Maintenance window approved
- Rollback plan documented

---

# Production Upgrade Workflow

```text
Current Version

↓

Backup

↓

Upgrade Test Environment

↓

Run Pipelines

↓

Validate Plugins

↓

Approve

↓

Upgrade Production
```

---

# Checking Current Version

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

System Information
```

or

```text
Manage Jenkins

↓

About Jenkins
```

---

# Checking Plugin Compatibility

Before upgrading,

review

```text
Installed Plugins

↓

Available Updates

↓

Compatibility

↓

Known Issues
```

Avoid upgrading incompatible plugins.

---

# Plugin Dependency Check

Example

```text
Docker Plugin

↓

Requires

↓

Credentials Plugin

↓

Requires

↓

Pipeline Plugin
```

Upgrading one plugin may require upgrading several dependencies.

---

# Java Compatibility

Every Jenkins version supports specific Java versions.

Example

```text
Jenkins

↓

Java 17

↓

Supported

---------------------

Jenkins

↓

Java 8

↓

Unsupported
```

Always verify Java compatibility before upgrading.

---

# Upgrade Environment

Never test upgrades directly in production.

```text
Production

↓

Backup

↓

Clone

↓

Testing Server

↓

Upgrade

↓

Validate

↓

Production Upgrade
```

---

# Maintenance Window

Production upgrades should occur during approved maintenance windows.

Example

```text
Sunday

02:00 AM

↓

Maintenance

↓

Upgrade

↓

Verification

↓

Service Restored
```

---

# Enterprise Example

```text
Friday

↓

Backup

↓

Saturday

↓

Testing

↓

Sunday

↓

Production Upgrade

↓

Monday

↓

Developers Start Work
```

This minimizes business impact.

---

# Upgrade Flow Diagram

```text
Download New Version

↓

Backup Jenkins

↓

Stop Jenkins

↓

Install New Version

↓

Start Jenkins

↓

Verify Health

↓

Run Test Pipeline

↓

Upgrade Complete
```

---

# Health Checks After Upgrade

Immediately verify

- Jenkins starts successfully
- Plugins load correctly
- Agents reconnect
- Pipelines execute
- Credentials available
- Web UI accessible
- Authentication working

---

# Best Practices

- Always use LTS releases for production.
- Never upgrade without a verified backup.
- Test upgrades in a staging environment.
- Read release notes before upgrading.
- Upgrade during scheduled maintenance windows.
- Verify plugin and Java compatibility before installation.

---

# Key Points

- Production environments should use Jenkins LTS releases.
- Every upgrade should begin with a verified backup.
- Test upgrades before applying them to production.
- Plugin and Java compatibility must be validated.
- Post-upgrade health checks are mandatory before returning Jenkins to users.

---

# Upgrading Jenkins

There are multiple ways to upgrade Jenkins depending on the installation method.

Common installation methods

- Native Package (RPM/DEB)
- WAR File
- Docker
- Kubernetes
- Helm Chart

---

# Linux Package Upgrade

For Debian/Ubuntu

```bash
sudo apt update
sudo apt install jenkins
```

For RHEL/CentOS

```bash
sudo yum update jenkins
```

or

```bash
sudo dnf update jenkins
```

After upgrading

```bash
sudo systemctl restart jenkins
```

---

# WAR File Upgrade

If Jenkins is deployed using a WAR file

```text
Download Latest WAR

↓

Replace Existing WAR

↓

Restart Jenkins
```

Example

```bash
java -jar jenkins.war
```

---

# Docker Upgrade

Docker deployments are upgraded by replacing the container.

```text
Pull New Image

↓

Stop Old Container

↓

Start New Container

↓

Reuse Existing JENKINS_HOME
```

Example

```bash
docker pull jenkins/jenkins:lts
```

---

# Kubernetes Upgrade

Production Kubernetes environments typically use Helm.

Upgrade flow

```text
Update Helm Values

↓

helm upgrade

↓

New Controller Pod

↓

Mount Existing PVC

↓

Jenkins Starts
```

Example

```bash
helm upgrade jenkins jenkinsci/jenkins
```

---

# Plugin Upgrades

Plugins should be upgraded carefully.

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Plugins

↓

Updates
```

---

# Plugin Upgrade Workflow

```text
Available Updates

↓

Read Release Notes

↓

Check Dependencies

↓

Upgrade

↓

Restart Jenkins

↓

Verify Pipelines
```

---

# Plugin Upgrade Best Practices

Avoid updating every plugin at once.

Recommended approach

```text
Critical Plugins

↓

Upgrade

↓

Validate

↓

Next Plugin Group
```

This makes troubleshooting much easier.

---

# Plugin Compatibility

Example

```text
Git Plugin

↓

Compatible

↓

Pipeline Plugin

↓

Compatible

↓

Docker Plugin

↓

Compatible
```

Always verify compatibility with your Jenkins version.

---

# Restart Strategies

Some upgrades require a restart.

```text
Install Plugin

↓

Safe Restart

↓

Running Builds Finish

↓

Restart Jenkins
```

Safe Restart is preferred because active builds are allowed to complete.

---

# Safe Restart

Navigate to

```text
Manage Jenkins

↓

Prepare for Shutdown
```

Behavior

```text
No New Builds

↓

Current Builds Finish

↓

Restart

↓

Ready
```

---

# Restart vs Safe Restart

| Restart | Safe Restart |
|----------|--------------|
| Immediate | Waits for builds |
| Running jobs interrupted | Running jobs complete |
| Higher risk | Recommended |

---

# Maintenance Tasks

A Jenkins administrator should perform regular maintenance.

Typical activities

- Review plugins
- Remove unused jobs
- Clean workspaces
- Archive artifacts
- Rotate logs
- Verify backups
- Review security
- Monitor resources

---

# Workspace Cleanup

Old workspaces consume disk space.

Example Pipeline

```groovy
post {
    always {
        cleanWs()
    }
}
```

Benefits

- Reduced storage
- Faster builds
- Cleaner agents

---

# Build Cleanup

Configure build retention.

Example

```text
Keep

Last 30 Builds

↓

Delete Older Builds
```

This prevents uncontrolled storage growth.

---

# Plugin Cleanup

Review plugins regularly.

```text
Installed Plugins

↓

Unused Plugins

↓

Remove

↓

Restart
```

Unused plugins

- Increase attack surface
- Consume memory
- Slow startup

---

# User Cleanup

Review inactive users.

```text
Users

↓

Inactive

↓

Disable

↓

Remove Access
```

This improves security.

---

# Credential Review

Regularly review

- Expired credentials
- Unused credentials
- Duplicate credentials
- API Tokens

Rotate sensitive credentials periodically.

---

# System Health Checks

Regularly verify

```text
Controller

↓

Healthy

----------------

Agents

↓

Connected

----------------

Pipelines

↓

Passing

----------------

Disk

↓

Healthy

----------------

Backups

↓

Successful
```

---

# Configuration as Code (JCasC)

## What is JCasC?

Jenkins Configuration as Code (JCasC) stores Jenkins configuration in YAML files.

Instead of configuring Jenkins manually,

everything becomes version-controlled.

---

# JCasC Architecture

```text
Git Repository

↓

JCasC YAML

↓

Jenkins Startup

↓

Configuration Applied
```

---

# Benefits of JCasC

- Version control
- Easy rollback
- Repeatable deployments
- Faster disaster recovery
- Infrastructure as Code

---

# Example

```yaml
jenkins:
  systemMessage: "Production Jenkins"
```

Instead of clicking through the UI,

configuration is stored in Git.

---

# Rolling Upgrade Strategy

Large organizations often use a staged approach.

```text
Development

↓

QA

↓

Staging

↓

Production
```

Each environment validates the upgrade before moving to the next.

---

# Rollback Strategy

Every upgrade should have a rollback plan.

```text
Upgrade

↓

Problem Found

↓

Restore Backup

↓

Rollback Version

↓

Resume Service
```

Never perform upgrades without a tested rollback procedure.

---

# Production Upgrade Architecture

```text
               GitHub

                  │

                  ▼

         Production Jenkins

                  │

          Backup Created

                  │

        Upgrade Controller

                  │

      Validate Plugins

                  │

      Validate Agents

                  │

       Execute Test Pipeline

                  │

         Service Available
```

---

# Production Maintenance Schedule

Example

```text
Daily

Workspace Cleanup

Backup Verification

----------------------

Weekly

Plugin Review

Disk Cleanup

Log Review

----------------------

Monthly

Security Review

Java Updates

Plugin Audit

DR Test

----------------------

Quarterly

Jenkins Upgrade

Capacity Planning
```

---

# Best Practices

## Upgrades

- Always use LTS releases in production.
- Upgrade plugins in small batches.
- Test upgrades in staging first.
- Use Safe Restart instead of immediate restart.
- Read release notes before upgrading.

---

## Maintenance

- Remove unused plugins.
- Rotate logs.
- Clean workspaces.
- Archive old builds.
- Monitor JVM health.
- Verify backups regularly.
- Audit users and credentials.

---

# Common Troubleshooting

## Issue 1: Jenkins Fails After Upgrade

Possible Causes

- Plugin incompatibility
- Unsupported Java version
- Corrupted installation

Resolution

```text
Review Logs

↓

Disable Problem Plugin

↓

Restore Backup

↓

Retry
```

---

## Issue 2: Plugins Disabled

Cause

Dependency mismatch.

Resolution

```text
Plugin Manager

↓

Install Required Dependency

↓

Restart Jenkins
```

---

## Issue 3: Agents Cannot Connect

Possible Causes

- Java version mismatch
- Agent version mismatch
- Network issue

Resolution

```text
Verify Java

↓

Reconnect Agent

↓

Restart Agent Service
```

---

## Issue 4: Slow Startup

Possible Causes

- Too many plugins
- Plugin initialization
- Low memory

Resolution

```text
Review Startup Logs

↓

Remove Unused Plugins

↓

Increase JVM Heap
```

---

## Issue 5: Pipelines Fail After Upgrade

Possible Causes

- Plugin API changes
- Deprecated pipeline syntax
- Tool version mismatch

Resolution

```text
Review Console Output

↓

Update Pipeline

↓

Retest
```

---

# Production Interview Questions

## Question 1

### Production-Level Answer

Why is Jenkins LTS recommended for production?

LTS releases receive long-term support, security patches, and stability fixes, making them more suitable for enterprise environments than weekly releases.

### Follow-Up Questions

- When would you use weekly releases?
- How often should LTS be upgraded?
- What risks exist when using outdated versions?

---

## Question 2

### Production-Level Answer

What should you do before upgrading Jenkins?

Create and verify a backup, review release notes, check plugin and Java compatibility, test the upgrade in a staging environment, schedule a maintenance window, and prepare a rollback plan.

---

## Question 3

### Production-Level Answer

What is Safe Restart?

Safe Restart prevents new builds from starting while allowing running builds to finish before restarting Jenkins, minimizing disruption.

---

## Question 4

### Production-Level Answer

Why should plugins be upgraded in small batches?

Upgrading a few plugins at a time makes it easier to identify compatibility issues and reduces the risk of widespread failures.

---

## Question 5

### Production-Level Answer

What is Jenkins Configuration as Code (JCasC)?

JCasC stores Jenkins configuration in YAML files, enabling version control, repeatable deployments, and easier disaster recovery.

---

## Question 6

### Production-Level Answer

How would you roll back a failed Jenkins upgrade?

Restore the verified backup of JENKINS_HOME, reinstall the previous Jenkins version if necessary, validate plugins, reconnect agents, and confirm pipelines are functioning.

---

## Question 7

### Production-Level Answer

Why should unused plugins be removed?

Unused plugins increase memory usage, startup time, maintenance effort, and the overall security attack surface.

---

## Question 8

### Production-Level Answer

How often should maintenance be performed?

Routine tasks such as workspace cleanup and backup verification should be performed daily, while plugin reviews, security audits, and upgrades should follow scheduled weekly, monthly, or quarterly maintenance windows.

---

## Question 9

### Production-Level Answer

Why is a staging environment important for upgrades?

It allows teams to validate compatibility, detect issues, and test pipelines before applying changes to production, reducing operational risk.

---

## Question 10

### Production-Level Answer

What are the most important post-upgrade checks?

Verify Jenkins startup, plugin loading, agent connectivity, authentication, credentials, pipeline execution, and overall system health before returning the service to users.

---

# Key Takeaways

- Use **LTS releases** for production Jenkins environments.
- Always back up and test upgrades before applying them.
- Upgrade plugins carefully and validate dependencies.
- Use **Safe Restart** to minimize build interruptions.
- Automate configuration with **Jenkins Configuration as Code (JCasC)** where possible.
- Perform regular maintenance to keep Jenkins secure, stable, and performant.
- Every upgrade should include a tested rollback strategy.

