# Jenkins User and RBAC Management

## Introduction

Jenkins is often accessed by multiple teams such as Developers, DevOps Engineers, QA Engineers, Release Managers, Security Teams, and Administrators.

If every user has administrator privileges, anyone can:

- Delete jobs
- Modify pipelines
- View secrets
- Deploy to production
- Install plugins
- Restart Jenkins

This introduces significant security and operational risks.

To prevent unauthorized access, Jenkins provides **Authentication** and **Authorization** mechanisms that control **who can log in** and **what actions they are allowed to perform**.

In enterprise environments, Jenkins security is typically integrated with centralized identity providers such as LDAP, Active Directory, or Single Sign-On (SSO), while permissions are managed using **Role-Based Access Control (RBAC)**.

---

# Why User Management?

Imagine a company with 250 engineers.

```text
                Jenkins

                    │

        ┌───────────┼────────────┐

        ▼           ▼            ▼

 Developers      QA Team      DevOps Team

        ▼           ▼            ▼

  Build Jobs   Test Jobs   Production Deployments
```

If every user receives Administrator access,

```text
Developer

↓

Delete Production Pipeline

↓

Production Deployment Stops
```

or

```text
QA Engineer

↓

Modify Jenkins Credentials

↓

Deployment Failure
```

or

```text
Intern

↓

Restart Jenkins

↓

All Running Builds Lost
```

Clearly,

every user should only receive the permissions required for their job.

This is called the **Principle of Least Privilege**.

---

# Authentication vs Authorization

These two concepts are frequently asked in interviews.

## Authentication

Authentication answers the question:

> **Who are you?**

Examples

- Username & Password
- LDAP Login
- Active Directory Login
- Google SSO
- GitHub OAuth
- SAML Login

Authentication only verifies identity.

Example

```text
User Login

↓

Username

↓

Password

↓

Verified

↓

Login Successful
```

---

## Authorization

Authorization answers the question:

> **What are you allowed to do?**

Examples

```text
Developer

↓

Can Build Jobs

↓

Cannot Delete Jobs

↓

Cannot Manage Jenkins
```

Another example

```text
Administrator

↓

Manage Jenkins

↓

Install Plugins

↓

Manage Credentials

↓

Restart Jenkins
```

---

## Authentication vs Authorization

```text
             User

              │

              ▼

      Authentication

      "Who are you?"

              │

              ▼

      Authorization

"What are you allowed to do?"

              │

              ▼

      Jenkins Dashboard
```

Authentication always happens **before** Authorization.

---

# Why RBAC?

RBAC stands for

**Role-Based Access Control**

Instead of assigning permissions to every individual user,

permissions are assigned to **Roles**.

Users receive Roles.

```text
Permission

↓

Role

↓

User
```

instead of

```text
Permission

↓

User

Permission

↓

User

Permission

↓

User
```

RBAC is much easier to manage.

---

# Jenkins Security Architecture

```text
                    Users

                        │

                        ▼

             Authentication Layer

      (LDAP / AD / Local / OAuth)

                        │

                        ▼

             Authorization Layer

            (RBAC / Matrix Security)

                        │

                        ▼

                 Jenkins Controller

                        │

        ┌───────────────┼───────────────┐

        ▼               ▼               ▼

     Pipelines      Credentials     Administration
```

---

# Enterprise Example

A software company has

- 120 Developers
- 20 QA Engineers
- 10 DevOps Engineers
- 5 Release Managers
- 2 Jenkins Administrators

Giving Administrator access to everyone would be dangerous.

Instead,

```text
Developer Role

↓

Read

Build

Workspace

Logs

---------------------

QA Role

↓

Read

Build

Test Reports

---------------------

DevOps Role

↓

Deploy

Credentials

Agents

---------------------

Administrator

↓

Full Control
```

This keeps Jenkins secure while allowing every team to perform its responsibilities.

---

# Benefits of RBAC

- Improved security
- Easier permission management
- Reduced accidental changes
- Better compliance
- Clear separation of duties
- Principle of Least Privilege
- Simplified onboarding and offboarding

---

# Production Security Flow

```text
Employee

↓

Login Request

↓

Authentication

↓

Identity Verified

↓

RBAC Evaluation

↓

Permission Granted

↓

Access Jenkins Resources
```

---

# Real Production Example

A developer pushes code.

```text
Developer

↓

GitHub

↓

Webhook

↓

Jenkins Pipeline

↓

Developer Can

✔ View Logs

✔ Retry Build

✔ Download Artifacts

✖ Delete Pipeline

✖ Modify Credentials

✖ Restart Jenkins
```

Meanwhile,

```text
DevOps Engineer

↓

Can

✔ Create Pipelines

✔ Configure Agents

✔ Manage Credentials

✔ Deploy Production

✔ Install Plugins
```

---

# Why Enterprises Prefer RBAC

Without RBAC

```text
300 Users

↓

300 Permission Sets
```

Very difficult to manage.

With RBAC

```text
300 Users

↓

5 Roles

↓

Easy Management
```
---

# Security Realm

## What is a Security Realm?

A **Security Realm** defines **how Jenkins authenticates users**.

It answers the question:

> **Who are you?**

When a user tries to log in, Jenkins checks the configured Security Realm to verify the user's identity.

Without a Security Realm,

Jenkins cannot determine whether a user is legitimate.

---

# Why Do We Need a Security Realm?

Imagine a Jenkins server accessible over the internet.

Without authentication,

```text
Internet

↓

Jenkins

↓

Everyone Gets Access
```

Anyone could:

- View pipelines
- Trigger builds
- Download artifacts
- Delete jobs
- Restart Jenkins

A Security Realm prevents unauthorized users from accessing Jenkins.

---

# Authentication Flow

```text
User

↓

Login Request

↓

Security Realm

↓

Verify Identity

↓

Authentication Successful

↓

Jenkins Dashboard
```

---

# Types of Security Realms

Jenkins supports multiple authentication providers.

```text
                    Jenkins

                       │

      ┌────────────────┼─────────────────┐

      ▼                ▼                 ▼

 Local Users       LDAP           Active Directory

      ▼                ▼                 ▼

 GitHub OAuth      SAML            OpenID Connect
```

---

# Local Jenkins User Database

The simplest authentication method.

Users are created directly inside Jenkins.

```text
Jenkins

↓

Create User

↓

Username

↓

Password

↓

Login
```

Suitable for

- Labs
- Small teams
- Learning environments

Not recommended for large enterprises.

---

# LDAP Authentication

Large organizations commonly use LDAP.

```text
User

↓

LDAP Login

↓

LDAP Server

↓

Authentication

↓

Jenkins Access
```

Benefits

- Centralized user management
- Password policies
- Easy onboarding
- Easy offboarding

---

# Active Directory Authentication

Many Windows-based organizations integrate Jenkins with Active Directory.

```text
Employee

↓

Windows Login

↓

Active Directory

↓

Jenkins
```

Benefits

- Single corporate identity
- Centralized authentication
- Group-based access

---

# OAuth Authentication

Popular for cloud-native environments.

Examples

- GitHub
- GitLab
- Google
- Microsoft Azure

```text
User

↓

GitHub Login

↓

OAuth

↓

Jenkins
```

Users log in using existing accounts instead of creating new Jenkins passwords.

---

# SAML Single Sign-On

Enterprise organizations often use SAML-based identity providers.

```text
Employee

↓

SSO Login

↓

Identity Provider

↓

Jenkins
```

Examples

- Okta
- Azure AD
- OneLogin
- Ping Identity

Users authenticate once and access multiple applications.

---

# Where to Configure?

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Security

↓

Security Realm
```

---

# Production Authentication Architecture

```text
                  Developers

                       │

                       ▼

                 Login Request

                       │

                       ▼

              Identity Provider

      (LDAP / AD / SAML / OAuth)

                       │

                Identity Verified

                       │

                       ▼

                    Jenkins
```

---

# Enterprise Example

A company has 500 engineers.

Instead of creating 500 Jenkins accounts,

the company integrates Jenkins with Active Directory.

```text
HR Creates Employee

↓

Employee Added to AD

↓

Employee Logs into Jenkins

↓

Authentication Successful
```

When an employee leaves,

```text
HR Disables AD Account

↓

Jenkins Access Removed
```

No manual Jenkins changes are required.

---

# Benefits of External Authentication

- Centralized identity management
- Password policies
- Multi-Factor Authentication
- Easy user provisioning
- Easy user removal
- Compliance requirements
- Single Sign-On support

---

# Authorization Strategy

Authentication verifies **who the user is**.

Authorization determines **what the user can do**.

```text
Authentication

↓

Identity Verified

↓

Authorization

↓

Permissions Assigned
```

Authentication alone is not enough.

---

# What is Authorization Strategy?

The **Authorization Strategy** controls the permissions granted to authenticated users.

It answers the question:

> **What are you allowed to do?**

---

# Authorization Flow

```text
User Login

↓

Authentication

↓

Authorization

↓

Permission Check

↓

Access Granted
```

---

# Types of Authorization Strategies

```text
Jenkins

↓

Authorization

↓

┌──────────────────────────────┐

Matrix Authorization

Role-Based Authorization (RBAC)

Logged-in Users

Anyone Can Do Anything

Legacy Mode

└──────────────────────────────┘
```

---

# Matrix-based Security

Permissions are assigned directly to users or groups.

Example

```text
User

↓

Read

Build

Workspace

View Logs

No Delete Permission
```

Suitable for

- Small teams
- Medium organizations

Can become difficult to manage as the number of users grows.

---

# Role-Based Authorization (RBAC)

Permissions are assigned to Roles.

Users receive Roles.

```text
Permissions

↓

Role

↓

User
```

Example

```text
Developer Role

↓

Read

Build

View Logs

-------------------

DevOps Role

↓

Deploy

Credentials

Agents

-------------------

Administrator

↓

Full Control
```

RBAC is the most common authorization model in enterprise Jenkins environments.

---

# Comparison

| Feature | Matrix Security | RBAC |
|----------|-----------------|------|
| Small Teams | Excellent | Good |
| Large Organizations | Difficult | Excellent |
| Easy Maintenance | Moderate | Excellent |
| Role Reuse | No | Yes |
| Enterprise Ready | Limited | Yes |

---

# Production Authentication & Authorization Flow

```text
Developer

↓

Login

↓

Active Directory

↓

Authentication Successful

↓

RBAC

↓

Developer Role

↓

Read

Build

View Logs

↓

Pipeline Access
```

---

# Best Practices

- Use LDAP, Active Directory, or SSO in enterprise environments.
- Avoid local Jenkins users except for emergency administrator accounts.
- Enable Multi-Factor Authentication through the identity provider whenever possible.
- Use RBAC instead of assigning permissions to individual users.
- Follow the Principle of Least Privilege.
- Periodically review user accounts and permissions.
- Disable inactive accounts immediately.

---

# Key Points

- Security Realm defines **how Jenkins authenticates users**.
- Authorization Strategy defines **what authenticated users are allowed to do**.
- Authentication always occurs before authorization.
- LDAP, Active Directory, OAuth, and SAML are commonly used in enterprise environments.
- RBAC is the preferred authorization model for production Jenkins environments because it is scalable and easier to manage.

---
# Matrix-Based Security

## What is Matrix-Based Security?

**Matrix-Based Security** is Jenkins' built-in authorization mechanism that allows administrators to assign **individual permissions** directly to users or groups.

Instead of assigning a role,

you assign permissions such as:

- Read
- Build
- Configure
- Delete
- Workspace
- Credentials
- Agent Management

to each user.

---

# Why Do We Need Matrix Security?

Consider a team with three users.

```text
Developer

↓

Can Build Jobs

Cannot Delete Jobs

-----------------------

QA Engineer

↓

Can View Jobs

Can View Test Reports

-----------------------

Administrator

↓

Full Access
```

Different users require different permissions.

Matrix Security makes this possible.

---

# Authentication vs Matrix Security

```text
User Login

↓

Authentication

↓

Identity Verified

↓

Matrix Security

↓

Permission Evaluation

↓

Jenkins Dashboard
```

Authentication identifies the user.

Matrix Security decides what that user can access.

---

# Where to Configure?

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Security

↓

Authorization

↓

Matrix-based Security
```

---

# Matrix Security Architecture

```text
                 Jenkins

                    │

             Authentication

                    │

                    ▼

          Matrix Authorization

                    │

      ┌─────────────┼──────────────┐

      ▼             ▼              ▼

 Developer        QA          Administrator

      │             │               │

 Read          Read          Full Control

 Build         Workspace

 Logs          Reports
```

---

# Permission Categories

Jenkins permissions are grouped into multiple categories.

```text
Overall

↓

Job

↓

Run

↓

View

↓

SCM

↓

Agent

↓

Credentials
```

Each category contains several permissions.

---

# Overall Permissions

These permissions affect Jenkins itself.

Examples

```text
Overall

↓

Read

Administer

Manage

System Read
```

Example

```text
Administrator

↓

Overall

↓

Administer
```

This grants full control over Jenkins.

---

# Job Permissions

Job permissions control access to pipelines.

Examples

```text
Job

↓

Read

Build

Configure

Delete

Discover

Workspace

Cancel
```

Developer Example

```text
Developer

↓

Read

Build

Workspace

No Delete
```

---

# Agent Permissions

These permissions control build agents.

```text
Agent

↓

Connect

Disconnect

Configure

Delete
```

Typically assigned only to DevOps engineers.

---

# Credentials Permissions

Credentials are highly sensitive.

```text
Credentials

↓

View

Create

Update

Delete

Manage Domains
```

Only administrators should have these permissions.

---

# Example Permission Matrix

```text
                 Read  Build Configure Delete Admin

Developer         ✔      ✔        ✖       ✖      ✖

QA                ✔      ✖        ✖       ✖      ✖

DevOps            ✔      ✔        ✔       ✖      ✖

Administrator     ✔      ✔        ✔       ✔      ✔
```

---

# Enterprise Example

A company has

- 150 Developers
- 30 QA Engineers
- 12 DevOps Engineers

Using Matrix Security

```text
Developer 1

↓

Assign Permissions

----------------------

Developer 2

↓

Assign Permissions

----------------------

Developer 3

↓

Assign Permissions

...
```

After several months,

the permission matrix becomes very large.

---

# Why Matrix Security Doesn't Scale

Imagine

```text
500 Users

↓

Assign Permissions Individually

↓

Thousands of Permission Entries
```

Managing permissions becomes difficult.

---

# Comparison

```text
Small Company

↓

20 Users

↓

Matrix Security

✔ Good Choice

-------------------------

Enterprise

↓

1000 Users

↓

Matrix Security

✖ Difficult to Maintain
```

Large organizations generally migrate to RBAC.

---

# Production Example

Developer Login

```text
Developer

↓

Authentication

↓

Matrix Security

↓

Read

Build

Workspace

↓

Run Pipeline
```

Administrator Login

```text
Administrator

↓

Authentication

↓

Matrix Security

↓

Administer

↓

Manage Jenkins
```

---

# Advantages

- Built into Jenkins
- No additional plugin required
- Easy for small teams
- Fine-grained permission control
- Simple initial setup

---

# Limitations

- Difficult to manage large numbers of users
- Permissions duplicated for every user
- High administrative overhead
- Increased chance of permission mistakes
- Not ideal for enterprise environments

---

# Why Enterprises Prefer RBAC

Matrix Security

```text
500 Users

↓

500 Permission Entries
```

RBAC

```text
500 Users

↓

Developer Role

QA Role

DevOps Role

Administrator Role
```

Much easier to maintain.

---

# Best Practices

- Use Matrix Security only for small and medium-sized teams.
- Follow the Principle of Least Privilege.
- Avoid granting **Overall → Administer** unless absolutely necessary.
- Regularly review assigned permissions.
- Transition to RBAC as the organization grows.
- Keep a separate emergency administrator account.

---

# Key Points

- Matrix-Based Security is Jenkins' built-in authorization mechanism.
- Permissions are assigned directly to individual users or groups.
- It provides fine-grained access control without requiring additional plugins.
- Matrix Security works well for small environments but becomes difficult to manage at enterprise scale.
- Most production organizations prefer RBAC because it simplifies permission management and improves scalability.

---

# Role-Based Access Control (RBAC)

## What is RBAC?

**Role-Based Access Control (RBAC)** is an authorization model where **permissions are assigned to roles**, and **users are assigned those roles**.

Instead of configuring permissions for every user individually,

you configure permissions once for each role.

---

# RBAC Workflow

```text
Permissions

↓

Role

↓

User
```

Example

```text
Read

Build

Workspace

↓

Developer Role

↓

50 Developers
```

One role can be assigned to hundreds of users.

---

# Why RBAC?

Consider an enterprise with

- 800 Developers
- 200 QA Engineers
- 50 DevOps Engineers

Without RBAC

```text
1050 Users

↓

Individual Permissions
```

With RBAC

```text
Developer Role

↓

QA Role

↓

DevOps Role

↓

Administrator Role

↓

Assign Users
```

Much simpler.

---

# Installing the Role Strategy Plugin

RBAC is provided by the **Role-based Authorization Strategy Plugin**.

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Plugins

↓

Available Plugins

↓

Role-based Authorization Strategy

↓

Install

↓

Restart Jenkins
```

---

# Enable RBAC

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Security

↓

Authorization

↓

Role-Based Strategy
```

After enabling,

the **Manage and Assign Roles** option becomes available.

---

# RBAC Architecture

```text
                Jenkins

                    │

            Authentication

                    │

            Role Strategy

                    │

      ┌─────────────┼─────────────┐

      ▼             ▼             ▼

Developer Role   QA Role   Administrator Role

      │             │             │

      ▼             ▼             ▼

 Developers       QA Users     Jenkins Admins
```

---

# Key Points

- RBAC assigns permissions to roles instead of individual users.
- The Role Strategy Plugin enables enterprise-grade authorization in Jenkins.
- RBAC reduces administrative effort and scales much better than Matrix Security.
- Roles can be reused across hundreds or thousands of users.
- RBAC is the recommended authorization model for production Jenkins environments.

---

# Creating Users

Before assigning permissions, users must exist in Jenkins or be imported from an external Identity Provider (LDAP, Active Directory, SSO).

In production environments, users are typically authenticated through an enterprise identity system rather than created manually.

---

# Creating a Local User

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Security

↓

Users

↓

Create User
```

Provide

- Username
- Password
- Full Name
- Email Address

Example

```text
Username : john.dev
Password : ********
Name     : John Smith
Email    : john@company.com
```

Once created, the user can authenticate but will not have any permissions until a role is assigned.

---

# Production Example

```text
New Developer Joins

↓

HR Creates Employee

↓

Active Directory Account Created

↓

Developer Logs into Jenkins

↓

RBAC Assigns Developer Role

↓

Ready to Build Pipelines
```

Notice that Jenkins user creation is automatic when integrated with LDAP/AD.

---

# Managing Users

User management includes:

- Creating users
- Updating user information
- Resetting passwords (for local users)
- Disabling access
- Removing users
- Reviewing last login

---

# User Lifecycle

```text
Create User

↓

Assign Role

↓

User Performs Work

↓

Role Updated

↓

User Leaves Company

↓

Disable Account

↓

Remove Access
```

---

# Enterprise Best Practice

Never delete users immediately.

Instead,

```text
Employee Leaves

↓

Disable Identity Account

↓

Audit Existing Builds

↓

Archive Logs

↓

Remove Access
```

This preserves build history and audit trails.

---

# Creating Roles

A **Role** is a collection of permissions.

Instead of assigning permissions individually,

you assign them once to the role.

---

# Typical Enterprise Roles

```text
Administrator

↓

Full Control

-------------------------

DevOps Engineer

↓

Configure Jobs

Deploy

Manage Agents

Manage Credentials

-------------------------

Developer

↓

Read

Build

Workspace

View Console

-------------------------

QA Engineer

↓

Read

View Reports

Run Test Pipelines

-------------------------

Release Manager

↓

Deploy Production

Approve Releases

View Pipelines
```

---

# Creating Roles

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Manage and Assign Roles

↓

Manage Roles
```

Create roles such as

```text
Developer

QA

DevOps

ReleaseManager

Administrator
```

---

# Assigning Permissions to Roles

Example

## Developer Role

```text
Overall

✔ Read

-------------------

Job

✔ Read

✔ Build

✔ Workspace

✔ Discover

-------------------

Run

✔ Replay

✔ Update
```

Developer cannot

```text
Delete Jobs

Manage Jenkins

Install Plugins

Manage Credentials
```

---

## QA Role

```text
Overall

✔ Read

-------------------

Job

✔ Read

✔ Build

-------------------

View

✔ Read
```

QA users can execute testing jobs but cannot modify pipelines.

---

## DevOps Role

```text
Overall

✔ Read

-------------------

Job

✔ Read

✔ Build

✔ Configure

✔ Delete

-------------------

Agent

✔ Configure

✔ Connect

✔ Disconnect

-------------------

Credentials

✔ View

✔ Update
```

---

## Administrator Role

```text
Overall

✔ Administer
```

The **Overall → Administer** permission automatically grants complete administrative access.

---

# Assigning Roles to Users

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Manage and Assign Roles

↓

Assign Roles
```

Example

```text
john.dev

↓

Developer Role

-------------------

alice.qa

↓

QA Role

-------------------

surendra

↓

DevOps Role

-------------------

admin

↓

Administrator Role
```

---

# RBAC Permission Flow

```text
User Login

↓

Authentication

↓

User Identified

↓

Assigned Role

↓

Permissions Loaded

↓

Access Granted
```

---

# Folder-Level Permissions

Large organizations separate projects into folders.

Example

```text
Jenkins

│

├── Payments

├── Banking

├── Insurance

└── Infrastructure
```

Each folder can have its own permissions.

---

# Folder-Level RBAC

```text
Payments Folder

↓

Developer Team A

↓

Build Only

----------------------

Infrastructure Folder

↓

DevOps Team

↓

Configure

Deploy

Credentials
```

Developers working on Payments cannot access Infrastructure jobs.

---

# Folder Permission Example

```text
Folder

↓

Payments

↓

Developer Role

↓

Read

Build

Workspace
```

```text
Folder

↓

Infrastructure

↓

DevOps Role

↓

Configure

Delete

Deploy
```

---

# Project-Level Permissions

RBAC can also restrict access to individual jobs.

Example

```text
Microservices

│

├── catalog-service

├── payment-service

├── shipping-service

└── production-deployment
```

Permissions

```text
Developers

↓

catalog-service

payment-service

shipping-service

-----------------------

Release Managers

↓

production-deployment
```

---

# Enterprise Example

```text
Developer

↓

Can Build

↓

Dev Environment

↓

QA Environment

------------------------

Cannot Deploy

↓

Production
```

Only Release Managers or DevOps Engineers can deploy to Production.

---

# Service Accounts

Not every Jenkins user is a human.

Many integrations use service accounts.

Examples

- GitHub Webhooks
- Terraform
- Argo CD
- SonarQube
- Docker Registry
- AWS CLI
- Kubernetes

---

# Service Account Architecture

```text
GitHub

↓

Webhook

↓

Jenkins Service Account

↓

Execute Pipeline
```

---

# Benefits of Service Accounts

- No shared human passwords
- Easier auditing
- Limited permissions
- Credential rotation
- Better security

---

# API Tokens

Instead of passwords,

Jenkins recommends using **API Tokens**.

Example

```text
CLI

↓

API Token

↓

Jenkins

↓

Authentication Successful
```

---

# Generate API Token

Navigate to

```text
User

↓

Configure

↓

API Token

↓

Generate Token
```

Example

```text
Token Name

↓

Terraform Automation

↓

Generate

↓

Copy

↓

Store Securely
```

---

# Best Practices for API Tokens

- Never commit tokens to Git.
- Rotate tokens regularly.
- Use separate tokens for different automation tools.
- Store tokens in Jenkins Credentials or a secrets manager.
- Delete unused tokens immediately.

---

# Key Points

- Users authenticate through Jenkins or an external Identity Provider.
- Roles group permissions together, simplifying administration.
- Assign permissions to roles—not individual users.
- Folder-level and project-level permissions provide fine-grained access control.
- Service accounts should be used for automation instead of personal accounts.
- API Tokens are more secure than passwords for automation and integrations.

---


# LDAP Integration

## What is LDAP?

**LDAP (Lightweight Directory Access Protocol)** is a centralized directory service used to authenticate users across an organization.

Instead of creating users inside Jenkins, Jenkins authenticates users against the organization's LDAP server.

---

# Why LDAP?

Without LDAP

```text
Developer

↓

Create Jenkins Account

↓

Create Git Account

↓

Create Nexus Account

↓

Create SonarQube Account

↓

Multiple Passwords
```

With LDAP

```text
Developer

↓

Corporate Login

↓

LDAP

↓

Access Jenkins

↓

Access Git

↓

Access SonarQube

↓

Access Nexus
```

One identity is used across all enterprise tools.

---

# LDAP Authentication Flow

```text
Developer

↓

Login

↓

Jenkins

↓

LDAP Server

↓

Identity Verified

↓

Jenkins Dashboard
```

---

# LDAP Production Architecture

```text
              Developers

                    │

                    ▼

                Jenkins

                    │

        LDAP Authentication Plugin

                    │

                    ▼

               LDAP Server

                    │

             User Database

                    │

             Authentication
```

---

# LDAP Configuration

Navigate to

```text
Dashboard

↓

Manage Jenkins

↓

Security

↓

Security Realm

↓

LDAP
```

Common configuration values

```text
LDAP Server URL

Manager DN

Manager Password

Root DN

User Search Base

Group Search Base
```

---

# Benefits of LDAP

- Centralized user management
- Strong password policies
- Easier onboarding
- Easier offboarding
- Group-based authentication
- Enterprise compliance

---

# Active Directory Integration

Many organizations use Microsoft Active Directory instead of a standalone LDAP server.

Jenkins integrates directly with AD.

---

# Active Directory Flow

```text
Employee

↓

Corporate Login

↓

Active Directory

↓

Authentication

↓

Jenkins Access
```

---

# Enterprise AD Example

```text
HR Creates Employee

↓

Active Directory

↓

Employee Logs into Jenkins

↓

Developer Role Assigned

↓

Pipeline Access
```

When an employee leaves

```text
HR Disables AD Account

↓

Jenkins Access Automatically Removed
```

---

# Active Directory Benefits

- Windows integration
- Group-based authorization
- Centralized identity
- Automatic account lifecycle
- Supports enterprise security policies

---

# Single Sign-On (SSO)

## What is SSO?

Single Sign-On allows users to authenticate **once** and access multiple enterprise applications.

Example

```text
Login Once

↓

GitHub

↓

Jenkins

↓

SonarQube

↓

Jira

↓

Confluence
```

No additional passwords are required.

---

# SSO Authentication Flow

```text
Developer

↓

Open Jenkins

↓

Redirect to Identity Provider

↓

Authentication

↓

Return to Jenkins

↓

Access Granted
```

---

# Common SSO Providers

```text
Azure AD

Okta

Google Workspace

Ping Identity

OneLogin

Keycloak
```

---

# SAML Authentication

SAML is one of the most common enterprise SSO protocols.

```text
Developer

↓

Jenkins

↓

SAML

↓

Identity Provider

↓

Authentication

↓

Jenkins
```

---

# OAuth Authentication

OAuth is commonly used with cloud platforms.

Examples

```text
GitHub

Google

GitLab

Microsoft
```

Flow

```text
Developer

↓

GitHub Login

↓

OAuth

↓

Jenkins
```

---

# Production Authentication Architecture

```text
                 Employees

                      │

                      ▼

             Identity Provider

       (AD / LDAP / SAML / OAuth)

                      │

               Authentication

                      │

                      ▼

                  Jenkins

                      │

             Role Strategy Plugin

                      │

          Developer / QA / DevOps

                      │

                      ▼

               Pipeline Access
```

---

# Production RBAC Workflow

```text
Developer Pushes Code

↓

GitHub

↓

Webhook

↓

Jenkins

↓

Authentication

↓

Developer Role

↓

Pipeline Starts

↓

Developer Can

✔ View Logs

✔ Retry Build

✔ Download Artifacts

✖ Configure Job

✖ Delete Job

✖ Deploy Production
```

---

# Enterprise Security Best Practices

## Authentication

- Use LDAP, Active Directory, or SSO instead of local users.
- Enable Multi-Factor Authentication through the Identity Provider.
- Keep only one emergency local administrator account.
- Disable anonymous access.
- Enforce strong password policies.

---

## Authorization

- Implement RBAC using the Role Strategy Plugin.
- Follow the Principle of Least Privilege.
- Avoid assigning **Overall → Administer** except to Jenkins administrators.
- Separate Developer, QA, DevOps, and Release Manager roles.
- Restrict production deployment permissions.

---

## Service Accounts

- Use dedicated service accounts for automation.
- Never use personal accounts for pipelines.
- Rotate API tokens regularly.
- Store secrets in Jenkins Credentials or an external secrets manager.
- Audit inactive service accounts.

---

## Audit & Compliance

- Review user permissions periodically.
- Remove inactive users promptly.
- Monitor authentication failures.
- Enable audit logging.
- Maintain documented access policies.

---

# Common Troubleshooting

## Issue 1: User Cannot Log In

Possible Causes

- Incorrect password
- LDAP/AD server unavailable
- User disabled
- Wrong Security Realm configuration

Resolution

```text
Check Authentication Logs

↓

Verify LDAP Connectivity

↓

Test User Credentials

↓

Restart Authentication Service (if required)
```

---

## Issue 2: User Can Log In but Cannot View Jobs

Cause

RBAC permissions are missing.

Resolution

```text
Manage and Assign Roles

↓

Verify Assigned Role

↓

Check Overall → Read

↓

Check Job → Read
```

---

## Issue 3: User Cannot Trigger Builds

Cause

Missing **Job → Build** permission.

Resolution

```text
Developer Role

↓

Enable

Job

↓

Build
```

---

## Issue 4: User Cannot Access Folder

Cause

Folder-level permissions are not configured.

Resolution

```text
Verify Folder Role

↓

Assign Folder Permissions

↓

Retest Access
```

---

## Issue 5: API Token Authentication Fails

Possible Causes

- Expired token
- Revoked token
- Wrong username
- Incorrect API endpoint

Resolution

```text
Generate New Token

↓

Update Automation

↓

Retest Authentication
```

---

# Production Interview Questions

## Question 1

### Production-Level Answer

What is the difference between Authentication and Authorization?

Authentication verifies **who the user is**, while Authorization determines **what the authenticated user is allowed to do**. Authentication always occurs before Authorization.

### Follow-Up Questions

- What is a Security Realm?
- What is an Authorization Strategy?
- Can Authorization happen before Authentication?

---

## Question 2

### Production-Level Answer

Why is RBAC preferred over Matrix Security?

RBAC assigns permissions to reusable roles, making it scalable and easier to manage for large organizations. Matrix Security assigns permissions directly to users, which becomes difficult to maintain as the number of users grows.

### Follow-Up Questions

- Which plugin enables RBAC?
- When would you use Matrix Security?
- What are the limitations of Matrix Security?

---

## Question 3

### Production-Level Answer

How would you secure a production Jenkins server?

- Integrate with LDAP/AD or SSO
- Enable RBAC
- Apply Least Privilege
- Use API Tokens instead of passwords
- Restrict administrator accounts
- Enable audit logging
- Regularly review permissions

### Follow-Up Questions

- Why disable anonymous access?
- Why keep an emergency admin account?
- How often should permissions be audited?

---

## Question 4

### Production-Level Answer

What are service accounts and why are they important?

Service accounts are non-human accounts used by automation tools like GitHub, Terraform, Kubernetes, or SonarQube. They provide dedicated identities with limited permissions, improving security and auditability.

### Follow-Up Questions

- Should service accounts use personal credentials?
- Where should API tokens be stored?
- How often should tokens be rotated?

---

## Question 5

### Production-Level Answer

How would you integrate Jenkins with Active Directory?

Install the Active Directory plugin, configure the AD server in the Security Realm, enable Role-Based Authorization, map AD users or groups to Jenkins roles, and verify authentication.

### Follow-Up Questions

- What if the AD server is unavailable?
- Can AD groups map directly to Jenkins roles?
- What are the benefits of AD integration?

---

## Question 6

### Production-Level Answer

What is the Principle of Least Privilege?

Users should receive only the minimum permissions required to perform their job, reducing the risk of accidental or malicious actions.

---

## Question 7

### Production-Level Answer

Why should API Tokens be preferred over passwords?

API Tokens are revocable, individually managed, auditable, and can be rotated without changing the user's password, making them more secure for automation.

---

## Question 8

### Production-Level Answer

What is Single Sign-On (SSO)?

SSO allows users to authenticate once with an Identity Provider and access multiple enterprise applications without logging in repeatedly.

---

## Question 9

### Production-Level Answer

How would you troubleshoot a user who can log in but cannot build jobs?

Verify the assigned RBAC role and ensure the **Job → Build** permission is enabled. Also check folder-level permissions if the job is inside a restricted folder.

---

## Question 10

### Production-Level Answer

What is the difference between LDAP and Active Directory?

LDAP is a protocol for accessing directory services, while Active Directory is Microsoft's directory service implementation that supports LDAP along with additional Windows-specific features like Group Policy and Kerberos authentication.

---

# Key Takeaways

- Security Realm controls **how users authenticate**.
- Authorization Strategy controls **what users can access**.
- Authentication always precedes authorization.
- RBAC is the recommended authorization model for enterprise Jenkins deployments.
- Matrix Security is suitable for small environments but becomes difficult to manage at scale.
- Integrate Jenkins with LDAP, Active Directory, or SSO for centralized identity management.
- Use service accounts and API tokens for automation instead of personal accounts.
- Apply the Principle of Least Privilege and review permissions regularly.
- Enable auditing and remove inactive accounts to maintain a secure Jenkins environment.