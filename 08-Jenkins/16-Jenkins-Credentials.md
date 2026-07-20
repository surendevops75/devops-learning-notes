# Jenkins Credentials

## Introduction

Production CI/CD pipelines interact with multiple external systems such as GitHub, Docker Hub, AWS, Kubernetes, SonarQube, and Artifactory.

These systems require authentication. Instead of hardcoding usernames, passwords, or API keys inside the Jenkinsfile, Jenkins provides a secure Credentials Management system.

Jenkins Credentials allow pipelines to securely access external resources without exposing sensitive information.

---

Without Credentials

```text
Jenkinsfile

↓

Username = admin

Password = admin123

↓

Security Risk
```

---

With Jenkins Credentials

```text
Jenkins

↓

Credentials Store

↓

Secure Authentication

↓

Pipeline Uses Credentials
```

---

# Why Do We Need Jenkins Credentials?

Imagine a production pipeline.

```text
GitHub

↓

Clone Repository

↓

Docker Hub

↓

Push Image

↓

AWS

↓

Deploy to EKS

↓

SonarQube

↓

Upload Reports
```

Every system requires authentication.

Instead of writing passwords inside Jenkinsfiles,

Jenkins retrieves them securely from its Credentials Store.

---

Benefits

```text
Secure

Centralized

Reusable

Encrypted

Easy to Manage
```

---

# Where Are Credentials Used?

Almost every enterprise pipeline uses credentials.

Example

```text
GitHub PAT

↓

Clone Repository

---------------------

Docker Hub

↓

Push Images

---------------------

AWS Access Keys

↓

Deploy Infrastructure

---------------------

Kubernetes Config

↓

Deploy Applications

---------------------

SonarQube Token

↓

Code Analysis
```

---

# Jenkins Credentials Architecture

```text
Developer

↓

Jenkins Pipeline

↓

Credentials Store

↓

Authentication

↓

External Service
```

---

# Types of Jenkins Credentials

Jenkins supports multiple credential types.

```text
Username & Password

Secret Text

SSH Username with Private Key

Secret File

Certificate
```

---

# Username with Password

Used when authentication requires both a username and password.

Examples

```text
GitHub

Docker Hub

JFrog Artifactory

Nexus Repository
```

---

Example

```text
Username

↓

surendra

Password

↓

********
```

---

# Secret Text

Stores a single secret value.

Examples

```text
GitHub PAT

SonarQube Token

Slack Token

API Keys
```

---

Example

```text
Token

↓

ghp_xxxxxxxxxxxxx
```

---

# SSH Username with Private Key

Used for Git repositories accessed using SSH.

```text
GitHub

↓

SSH Key

↓

Clone Repository
```

---

Examples

```text
GitHub SSH

GitLab SSH

Bitbucket SSH
```

---

# Secret File

Stores files securely.

Examples

```text
Kubeconfig

AWS Credentials File

JSON Keys

License Files
```

---

Example

```text
kubeconfig

↓

Deploy to Kubernetes
```

---

# Certificate

Stores certificates required for secure communication.

Examples

```text
SSL Certificates

PKCS12

Java Keystore
```

---

# Credential Scope

Jenkins supports two scopes.

```text
Global

System
```

---

Global

```text
Accessible

↓

All Pipelines
```

---

System

```text
Only Jenkins Internal Usage
```

---

# Adding Credentials

Navigate to

```text
Manage Jenkins

↓

Credentials

↓

Global

↓

Add Credentials
```

---

Required Information

```text
Type

ID

Username

Password / Secret

Description
```

---

# Credential ID

Each credential has a unique identifier.

Example

```text
github-creds

aws-creds

dockerhub-creds

sonarqube-token
```

---

Pipelines reference the Credential ID instead of passwords.

```groovy
credentialsId: "github-creds"
```

---

# Using Credentials in Pipeline

Example

```groovy
git branch: "main",
    credentialsId: "github-creds",
    url: "https://github.com/company/cart.git"
```

---

Jenkins automatically retrieves the stored credentials.

The password is never exposed in the Jenkinsfile.

---

# Environment Variables with Credentials

Jenkins can expose credentials as environment variables.

Example

```groovy
environment {

    SONAR_TOKEN = credentials('sonarqube-token')

}
```

---

Pipeline

↓

Credential Store

↓

Environment Variable

↓

Pipeline Uses Secret

---

# Using Secret Text

Example

```groovy
withCredentials([string(credentialsId: 'sonarqube-token',
variable: 'TOKEN')]) {

    sh "mvn sonar:sonar -Dsonar.login=$TOKEN"

}
```

---

# Using Username and Password

Example

```groovy
withCredentials([usernamePassword(

credentialsId: 'dockerhub-creds',

usernameVariable: 'USER',

passwordVariable: 'PASS'

)]) {

    sh "docker login -u $USER -p $PASS"

}
```

---

# Using SSH Credentials

Example

```groovy
git branch: "main",

credentialsId: "github-ssh",

url: "git@github.com:company/cart.git"
```

---

# Credential Flow

```text
Pipeline

↓

Credential ID

↓

Credentials Store

↓

Authentication

↓

External Service
```

---