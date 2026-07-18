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
---

# Understanding Text Parameter

## What is a Text Parameter?

A **Text Parameter** allows users to enter **multiple lines of text** before starting the pipeline.

Unlike a String Parameter, which accepts only a single line, a Text Parameter is designed for larger inputs such as release notes, deployment instructions, change requests, or configuration snippets.

Trainer Example

```groovy
text(

    name: 'BIOGRAPHY',

    defaultValue: '',

    description: 'Enter some information about the person'

)
```

Although the trainer uses a biography as an example, in production environments this parameter is commonly used for deployment-related information.

---

# Why Do We Need Text Parameters?

Imagine a release engineer deploying version **3.2.0**.

The deployment must include release notes.

Example

```text
Release Notes

• Fixed payment gateway issue

• Added inventory caching

• Improved API response time

• Updated Docker base image
```

A String Parameter cannot handle this efficiently because it only accepts a single line.

A Text Parameter allows structured, multi-line input.

---

# Text Parameter Workflow

```text
User Clicks

Build with Parameters

        │

        ▼

Enter Multi-line Text

        │

        ▼

Pipeline Starts

        │

        ▼

params.BIOGRAPHY

        │

        ▼

Displayed

Stored

or Published
```

---

# Accessing Text Parameters

Inside Groovy

```groovy
echo "${params.BIOGRAPHY}"
```

Inside Shell

```groovy
sh """

echo "${params.BIOGRAPHY}"

"""
```

---

# Production Example

A deployment pipeline asks the release engineer to enter release notes.

```groovy
text(

    name: 'RELEASE_NOTES',

    defaultValue: '',

    description: 'Enter Release Notes'

)
```

Pipeline

```groovy
echo """

Release Notes

${params.RELEASE_NOTES}

"""
```

These notes can later be:

- Archived
- Sent to Slack
- Published to Confluence
- Attached to Release Documentation

---

# Best Practices

- Use Text Parameters only for multi-line inputs.
- Keep descriptions meaningful.
- Validate large inputs if required.
- Avoid storing passwords.
- Archive important release notes.

---

# Understanding Boolean Parameter

## What is a Boolean Parameter?

A Boolean Parameter creates a checkbox.

Possible values

```text
true

false
```

This parameter is commonly used to enable or disable optional pipeline actions.

Trainer Example

```groovy
booleanParam(

    name: 'DEPLOY',

    defaultValue: false,

    description: 'Toggle this value'

)
```

---

# Why Do We Need Boolean Parameters?

Not every pipeline execution should deploy the application.

Example

A developer pushes code.

The pipeline should:

- Compile
- Test
- Scan

But deployment should happen only when explicitly requested.

Boolean Parameters make this possible.

---

# Boolean Workflow

```text
Build with Parameters

        │

        ▼

Deploy?

☐ No

☑ Yes

        │

        ▼

Pipeline Starts

        │

        ▼

params.DEPLOY

        │

        ▼

Deploy Stage

Runs

or

Skipped
```

---

# Trainer Example

```groovy
echo "${params.DEPLOY}"
```

Output

```text
true
```

or

```text
false
```

depending on user selection.

---

# Production Example

```groovy
booleanParam(

    name: 'RUN_SECURITY_SCAN',

    defaultValue: true

)
```

Pipeline

```groovy
when {

    expression {

        params.RUN_SECURITY_SCAN

    }

}
```

Now the security scan runs only when enabled.

---

# Another Production Example

Deployment approval.

```groovy
booleanParam(

    name: 'DEPLOY',

    defaultValue: false

)
```

Pipeline

```groovy
when {

    expression {

        params.DEPLOY

    }

}
```

This prevents accidental deployments.

---

# Best Practices

- Use Boolean Parameters for optional stages.
- Keep default values safe.
- Production deployment should default to **false**.
- Use together with `when` conditions.
- Clearly describe what the checkbox controls.

---

# Understanding Choice Parameter

## What is a Choice Parameter?

A Choice Parameter displays a dropdown list.

Users can only select one predefined value.

Trainer Example

```groovy
choice(

    name: 'CHOICE',

    choices: ['One','Two','Three'],

    description: 'Pick something'

)
```

---

# Why Do We Need Choice Parameters?

Suppose developers type environments manually.

```text
dev

Dev

DEV

development

deev
```

All represent the same environment but introduce inconsistencies.

Choice Parameters eliminate these errors.

---

# Choice Workflow

```text
Build with Parameters

        │

        ▼

Dropdown

        │

        ▼

DEV

TEST

STAGE

PROD

        │

        ▼

Pipeline Reads

params.ENVIRONMENT
```

---

# Production Example

```groovy
choice(

    name: 'ENVIRONMENT',

    choices: [

        'dev',

        'test',

        'stage',

        'prod'

    ]

)
```

Pipeline

```groovy
echo "${params.ENVIRONMENT}"
```

User cannot enter invalid values.

---

# Another Production Example

AWS Region Selection

```groovy
choice(

    name: 'AWS_REGION',

    choices: [

        'ap-south-1',

        'us-east-1',

        'eu-west-1'

    ]

)
```

Now deployments occur only to approved AWS Regions.

---

# Best Practices

- Use Choice Parameters whenever possible.
- Limit values to approved options.
- Keep the list short.
- Arrange values logically.
- Use lowercase or uppercase consistently.

---

# Understanding Password Parameter

## What is a Password Parameter?

A Password Parameter hides user input while typing.

Trainer Example

```groovy
password(

    name: 'PASSWORD',

    defaultValue: 'SECRET',

    description: 'Enter a password'

)
```

The entered value is masked in the Jenkins UI.

---

# Important Limitation

A Password Parameter only hides the value from the user interface.

It is **not a secure secret management solution**.

For enterprise environments, Jenkins Credentials should always be preferred.

---

# Password Parameter vs Jenkins Credentials

| Password Parameter | Jenkins Credentials |
|--------------------|--------------------|
| User enters value each build | Stored securely in Jenkins |
| Temporary input | Long-term secret storage |
| Basic masking | Encrypted storage |
| Suitable for demos | Suitable for production |

---

# Production Recommendation

Instead of

```groovy
password(

    name: 'PASSWORD'

)
```

Use

```groovy
environment {

    AWS_ACCESS_KEY = credentials('aws-access-key')

}
```

or

```groovy
withCredentials(...)
```

This provides secure secret management.

---

# Accessing Parameters

Every parameter is available using the **params** object.

Example

```groovy
params.PERSON

params.BIOGRAPHY

params.DEPLOY

params.CHOICE

params.PASSWORD
```

Example

```groovy
echo "${params.PERSON}"

echo "${params.DEPLOY}"

echo "${params.CHOICE}"
```

Inside Shell

```groovy
sh """

echo ${params.PERSON}

echo ${params.CHOICE}

"""
```

---

# Complete Trainer Example

The trainer prints every parameter inside the Build stage.

```groovy
echo "Hello ${params.PERSON}"

echo "Biography: ${params.BIOGRAPHY}"

echo "Toggle: ${params.DEPLOY}"

echo "Choice: ${params.CHOICE}"

echo "Password: ${params.PASSWORD}"
```

This demonstrates how parameter values are accessed during pipeline execution.

---

# Parameter Comparison Table

| Parameter | Input Type | Production Use Case |
|------------|------------|--------------------|
| String | Single Line | Version, Branch, Image Tag |
| Text | Multi-line | Release Notes, Comments |
| Boolean | Checkbox | Deploy, Security Scan |
| Choice | Dropdown | Environment, Region |
| Password | Hidden Text | Temporary Sensitive Input |

---

# Which Parameter Should You Use?

| Requirement | Recommended Parameter |
|-------------|----------------------|
| Git Branch | String |
| Docker Version | String |
| Release Notes | Text |
| Enable Deployment | Boolean |
| Environment Selection | Choice |
| AWS Region | Choice |
| Temporary Password | Password |
| Production Secrets | Jenkins Credentials (Not Password Parameter) |