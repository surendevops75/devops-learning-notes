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

# Chapter Overview

In this chapter, you will learn:

- Jenkins Security Model
- Security Realm
- Authorization Strategies
- Matrix-based Security
- Role-Based Access Control (RBAC)
- Creating Users
- Managing Users
- Creating Roles
- Assigning Roles
- Folder-Level Permissions
- Project-Level Permissions
- Service Accounts
- API Tokens
- LDAP Integration
- Active Directory Integration
- Single Sign-On (SSO)
- Production Security Architecture
- Best Practices
- Troubleshooting
- Interview Questions