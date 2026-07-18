# Jenkins Parameters

# Introduction

In real-world software development, the same Jenkins Pipeline is executed multiple times for different purposes. Although the pipeline logic remains the same, the input values often change.

For example:

- Deploy to Development
- Deploy to Testing
- Deploy to Staging
- Deploy to Production

Similarly, developers may want to:

- Build different Git branches
- Build different application versions
- Enable or disable deployment
- Select different AWS Regions
- Provide release notes
- Trigger rollback

Creating separate Jenkins pipelines for each of these scenarios would be difficult to maintain.

Instead, Jenkins provides **Build Parameters**, allowing users to supply values when triggering a pipeline.

Parameters make Jenkins Pipelines:

- Dynamic
- Reusable
- Interactive
- Flexible
- Production Ready

The trainer's Jenkinsfile demonstrates the most commonly used parameter types.

```groovy
parameters {

    string(
        name: 'PERSON',
        defaultValue: 'Mr Jenkins',
        description: 'Who should I say hello to?'
    )

    text(
        name: 'BIOGRAPHY',
        defaultValue: '',
        description: 'Enter some information about the person'
    )

    booleanParam(
        name: 'DEPLOY',
        defaultValue: false,
        description: 'Toggle this value'
    )

    choice(
        name: 'CHOICE',
        choices: ['One', 'Two', 'Three'],
        description: 'Pick something'
    )

    password(
        name: 'PASSWORD',
        defaultValue: 'SECRET',
        description: 'Enter a password'
    )

}
```

When a user clicks **Build with Parameters**, Jenkins displays these fields and collects input before the pipeline starts.

---

# Why Do We Need Parameters?

Imagine an application that needs to be deployed to four different environments.

Without Parameters

```text
Pipeline-DEV

Pipeline-TEST

Pipeline-STAGE

Pipeline-PROD
```

Problems

- Four pipelines to maintain
- Duplicate code
- Difficult updates
- Increased maintenance effort
- Higher chances of mistakes

Instead, Jenkins allows one pipeline to handle every environment.

Using Parameters

```text
Build With Parameters

        │

        ▼

Choose Environment

 DEV

 TEST

 STAGE

 PROD

        │

        ▼

Same Pipeline Executes
```

Only the input changes.

The pipeline remains exactly the same.

---

# What are Jenkins Parameters?

Jenkins Parameters are user inputs collected before the pipeline starts.

These values become available during pipeline execution through the **params** object.

Think of parameters as command-line arguments passed to a program.

Example

Instead of writing

```bash
./deploy.sh dev
```

Jenkins automatically provides

```groovy
params.ENVIRONMENT
```

inside the pipeline.

---

# Parameter Workflow

The following diagram shows how parameters are used during execution.

```text
User Clicks

Build with Parameters

        │

        ▼

Jenkins Displays Form

        │

        ▼

User Enters Values

        │

        ▼

Pipeline Starts

        │

        ▼

Parameters Stored

        │

        ▼

Pipeline Reads

params

        │

        ▼

Conditional Execution
```

Every parameter becomes available before the first stage executes.

---

# Parameter Block

All parameters are defined inside the **parameters** block.

General Syntax

```groovy
parameters {

    parameter_type(
        name: 'PARAMETER_NAME',
        defaultValue: 'value',
        description: 'Description'
    )

}
```

After the pipeline starts,

Jenkins automatically creates

```groovy
params.PARAMETER_NAME
```

which can be accessed anywhere inside the pipeline.

---

# Types of Jenkins Parameters

Jenkins supports multiple parameter types.

The trainer demonstrated five commonly used types.

| Parameter | Purpose |
|------------|----------|
| String | Single line input |
| Text | Multi-line input |
| Boolean | Checkbox (True/False) |
| Choice | Dropdown List |
| Password | Hidden Input |

Each serves a different purpose.

---

# Understanding String Parameter

## What is a String Parameter?

A String Parameter accepts a single line of text from the user.

It is the most commonly used Jenkins parameter.

Typical use cases include:

- Version Number
- Git Branch
- Docker Image Tag
- Username
- Environment Name
- AWS Account ID

Trainer Example

```groovy
string(

    name: 'PERSON',

    defaultValue: 'Mr Jenkins',

    description: 'Who should I say hello to?'

)
```

---

# String Parameter Workflow

```text
User Clicks

Build with Parameters

        │

        ▼

Enter Name

        │

        ▼

Pipeline Starts

        │

        ▼

params.PERSON

        │

        ▼

Used Inside Pipeline
```

---

# Accessing String Parameters

Inside Groovy

```groovy
echo "${params.PERSON}"
```

Inside Shell

```groovy
sh """

echo ${params.PERSON}

"""
```

Output

```text
Mr Jenkins
```

---

# Production Example

Suppose developers want to build different application versions.

Instead of modifying the Jenkinsfile every time,

create a String Parameter.

```groovy
string(

    name: 'VERSION',

    defaultValue: '1.0.0',

    description: 'Application Version'

)
```

Pipeline

```groovy
sh """

docker build -t catalogue:${params.VERSION} .

"""
```

If the user enters

```text
2.5.1
```

Jenkins executes

```bash
docker build -t catalogue:2.5.1 .
```

without changing the Jenkinsfile.

---

# Another Production Example

Instead of hardcoding the Git branch

```bash
git checkout main
```

create

```groovy
string(

    name: 'BRANCH',

    defaultValue: 'main'

)
```

Pipeline

```groovy
git branch: params.BRANCH,
    url: 'https://github.com/company/catalogue.git'
```

Now developers can build

```text
main

feature/login

feature/payment

release-v2
```

using the same pipeline.

---

# Best Practices for String Parameters

- Use meaningful parameter names.
- Keep default values sensible.
- Validate input before using it.
- Avoid using String Parameters for passwords.
- Use lowercase for branch names.
- Use versioning standards.
- Keep descriptions clear.
- Do not overload one parameter with multiple values.

---

# Common Mistakes

## Using String Instead of Choice

Bad Example

```text
dev

deev

development

DEV

Dev
```

Different users may enter different values.

Better Solution

Use a Choice Parameter.

---

## Hardcoding Values

Instead of

```bash
docker build -t catalogue:1.0.0 .
```

Use

```groovy
params.VERSION
```

This improves flexibility.

---

# Real Production Scenario

An organization releases software every week.

Every release has a different version.

Without parameters

Developers edit the Jenkinsfile before every release.

With String Parameters

```text
Build with Parameters

↓

Version

↓

3.4.7

↓

Pipeline Executes

↓

Docker Image

catalogue:3.4.7
```

The Jenkinsfile never changes.

Only the input changes.

---

# Interview Tip

One of the most common Jenkins interview questions is:

> **Why would you use a String Parameter instead of creating multiple pipelines?**

Remember:

A single parameterized pipeline is easier to maintain, easier to review, reusable, and follows the DRY (Don't Repeat Yourself) principle.