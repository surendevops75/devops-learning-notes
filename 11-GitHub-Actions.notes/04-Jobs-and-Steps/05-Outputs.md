# GitHub Actions Job Outputs

Job outputs allow one job to pass values to another job.

This is useful when one job generates information that a later job needs.

Typical examples:

- Docker image tag
- Application version
- Commit SHA
- Artifact name
- Environment name
- Terraform output
- Deployment ID

Basic flow:

```text
Build Job
    |
    ↓
Generate Value
    |
    ↓
Job Output
    |
    ↓
Deploy Job
    |
    ↓
Use Output
```

---

# Step Output vs Job Output

There are two related concepts:

```text
Step Output
    |
    ↓
Available within the same job
```

and:

```text
Job Output
    |
    ↓
Available to dependent jobs
```

Example:

```text
Step
  |
  ↓
Step Output
  |
  ↓
Job Output
  |
  ↓
Another Job
```

A job output is normally created from a step output.

---

# Why Job Outputs Are Needed

Jobs run in separate runner environments.

For example:

```text
Build Job
    |
    └── Runner A

Deploy Job
    |
    └── Runner B
```

The second job cannot simply access variables created inside the first job.

Job outputs provide a controlled way to pass small pieces of information between jobs.

---

# Basic Job Output Example

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:

      - name: Generate Version
        id: version
        run: echo "version=v1.0.0" >> "$GITHUB_OUTPUT"

  deploy:

    needs: build

    runs-on: ubuntu-latest

    steps:

      - name: Display Version
        run: echo "${{ needs.build.outputs.version }}"
```

Flow:

```text
Build
  |
  └── Step Output
        |
        ↓
     Job Output
        |
        ↓
     Deploy Job
        |
        ↓
needs.build.outputs.version
```

---

# Step ID

The step generating the value needs an `id`.

Example:

```yaml
- name: Generate Version
  id: version
  run: echo "version=v1.0.0" >> "$GITHUB_OUTPUT"
```

Here:

```text
id: version
```

identifies the step.

The output is:

```text
steps.version.outputs.version
```

---

# $GITHUB_OUTPUT

GitHub Actions provides the `$GITHUB_OUTPUT` file for setting step outputs.

Example:

```yaml
- name: Generate Version
  id: version
  run: echo "version=v1.0.0" >> "$GITHUB_OUTPUT"
```

The format is:

```text
name=value
```

Example:

```text
version=v1.0.0
```

The value can then be referenced through:

```yaml
${{ steps.version.outputs.version }}
```

---

# Converting Step Output to Job Output

A job output references a step output.

Example:

```yaml
jobs:

  build:

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:

      - name: Generate Version
        id: version
        run: echo "version=v1.0.0" >> "$GITHUB_OUTPUT"
```

The relationship is:

```text
Step ID
   |
   ↓
version

Step Output
   |
   ↓
steps.version.outputs.version

Job Output
   |
   ↓
outputs.version
```

---

# Accessing Job Outputs

A downstream job must depend on the producing job.

Example:

```yaml
deploy:

  needs: build
```

Then:

```yaml
${{ needs.build.outputs.version }}
```

Complete example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.version.outputs.version }}

    steps:

      - name: Generate Version
        id: version
        run: echo "version=v1.2.3" >> "$GITHUB_OUTPUT"

  deploy:

    needs: build

    runs-on: ubuntu-latest

    steps:

      - name: Deploy Version
        run: echo "Deploying ${{ needs.build.outputs.version }}"
```

---

# Output Naming

Use clear output names.

Good:

```yaml
outputs:
  image_tag:
```

```yaml
outputs:
  version:
```

```yaml
outputs:
  artifact_name:
```

Avoid unclear names such as:

```yaml
outputs:
  x:
```

Meaningful names make large workflows easier to maintain.

---

# Docker Image Tag Example

One of the most useful production applications is generating a Docker image tag.

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.image.outputs.tag }}

    steps:

      - name: Generate Image Tag
        id: image
        run: |
          TAG="${GITHUB_SHA::7}"
          echo "tag=$TAG" >> "$GITHUB_OUTPUT"
```

The output is:

```text
image_tag
```

A deployment job can consume it:

```yaml
deploy:

  needs: build

  runs-on: ubuntu-latest

  steps:

    - name: Deploy Image
      run: |
        echo "Deploying image tag:"
        echo "${{ needs.build.outputs.image_tag }}"
```

Flow:

```text
Git Commit
    |
    ↓
SHA
    |
    ↓
Image Tag
    |
    ↓
Job Output
    |
    ↓
Deployment
```

---

# Production Image Flow

A production pipeline can use:

```text
Source Commit
     |
     ↓
Build
     |
     ↓
Docker Image
     |
     ↓
Tag
     |
     ↓
Security Scan
     |
     ↓
Approved Image
     |
     ↓
Deployment
```

The image tag can be passed through job outputs.

---

# Multiple Job Outputs

A job can expose multiple outputs.

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      version: ${{ steps.metadata.outputs.version }}
      image_tag: ${{ steps.metadata.outputs.image_tag }}
      artifact_name: ${{ steps.metadata.outputs.artifact_name }}

    steps:

      - name: Generate Metadata
        id: metadata
        run: |
          echo "version=v1.2.3" >> "$GITHUB_OUTPUT"
          echo "image_tag=a83f91c" >> "$GITHUB_OUTPUT"
          echo "artifact_name=application" >> "$GITHUB_OUTPUT"
```

Another job can access:

```yaml
${{ needs.build.outputs.version }}
```

```yaml
${{ needs.build.outputs.image_tag }}
```

```yaml
${{ needs.build.outputs.artifact_name }}
```

---

# Multiple Jobs Consuming the Same Output

A job output can be consumed by multiple downstream jobs.

Example:

```text
                 Build
                   |
              image_tag
                   |
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      Test       Security    Deploy
```

Example:

```yaml
test:
  needs: build
```

```yaml
security:
  needs: build
```

```yaml
deploy:
  needs:
    - build
    - test
    - security
```

Each can use:

```yaml
${{ needs.build.outputs.image_tag }}
```

---

# Outputs and needs

The relationship is:

```text
Job A
  |
  └── outputs.version
          |
          ↓
Job B
  |
  └── needs.jobA.outputs.version
```

Example:

```yaml
deploy:
  needs: build
  run-name: Deploy ${{ needs.build.outputs.version }}
```

The important requirement is that the consuming job must depend on the producing job.

---

# Outputs Without needs

Do not expect:

```yaml
${{ needs.build.outputs.version }}
```

to work correctly if the current job has not declared:

```yaml
needs: build
```

Correct:

```yaml
deploy:

  needs: build

  steps:
    - run: echo "${{ needs.build.outputs.version }}"
```

---

# Passing Commit SHA

GitHub already provides the commit SHA through:

```yaml
${{ github.sha }}
```

You may still expose a normalized or customized value as a job output.

Example:

```yaml
jobs:

  metadata:

    runs-on: ubuntu-latest

    outputs:
      commit_sha: ${{ steps.info.outputs.sha }}

    steps:

      - name: Get Commit SHA
        id: info
        run: echo "sha=${GITHUB_SHA}" >> "$GITHUB_OUTPUT"
```

Downstream:

```yaml
${{ needs.metadata.outputs.commit_sha }}
```

---

# Short Commit SHA

A common deployment identifier is a shortened SHA.

Example:

```yaml
- name: Generate Short SHA
  id: commit
  run: |
    SHA="${GITHUB_SHA::7}"
    echo "sha=$SHA" >> "$GITHUB_OUTPUT"
```

Job output:

```yaml
outputs:
  commit_sha: ${{ steps.commit.outputs.sha }}
```

This can be used for:

- Docker tags
- Helm values
- Deployment labels
- Artifact versions
- Release tracking

---

# Application Version

Example:

```yaml
- name: Read Application Version
  id: version
  run: |
    VERSION=$(grep '^version=' version.properties | cut -d'=' -f2)
    echo "version=$VERSION" >> "$GITHUB_OUTPUT"
```

Job output:

```yaml
outputs:
  version: ${{ steps.version.outputs.version }}
```

Deployment:

```yaml
- name: Deploy Version
  run: |
    echo "Deploying version ${{ needs.build.outputs.version }}"
```

---

# Artifact Name

Example:

```yaml
jobs:

  build:

    outputs:
      artifact: ${{ steps.artifact.outputs.name }}

    steps:

      - name: Set Artifact Name
        id: artifact
        run: echo "name=application-${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
```

The deployment job can reference:

```yaml
${{ needs.build.outputs.artifact }}
```

---

# Terraform Output

Job outputs can also be used to pass selected Terraform results.

Example:

```yaml
- name: Terraform Output
  id: tf
  run: |
    ENDPOINT=$(terraform output -raw alb_dns_name)
    echo "endpoint=$ENDPOINT" >> "$GITHUB_OUTPUT"
```

Job output:

```yaml
outputs:
  endpoint: ${{ steps.tf.outputs.endpoint }}
```

Another job:

```yaml
- name: Display Endpoint
  run: echo "${{ needs.infrastructure.outputs.endpoint }}"
```

Use care when exposing infrastructure information.

Do not expose sensitive Terraform outputs.

---

# Multiline Outputs

Outputs can contain multiline values, but special handling is required.

Example:

```yaml
- name: Generate Output
  id: data
  run: |
    {
      echo 'config<<EOF'
      echo 'line-one'
      echo 'line-two'
      echo 'EOF'
    } >> "$GITHUB_OUTPUT"
```

For production workflows, prefer simple outputs where possible.

For large data, use artifacts or an external storage mechanism instead of passing large values through outputs.

---

# Outputs Are Not for Large Data

Job outputs are best for small pieces of metadata.

Good:

```text
version
image_tag
artifact_name
commit_sha
deployment_id
```

Avoid using outputs for:

```text
Large files
Large logs
Large configuration files
Binary artifacts
Large JSON documents
```

Use artifacts or external storage for large data.

---

# Outputs and Artifacts

Use outputs for metadata:

```text
image_tag = a83f91c
```

Use artifacts for files:

```text
application.jar
```

Conceptually:

```text
Build
  |
  ├── Job Output
  |      └── image_tag
  |
  └── Artifact
         └── application.jar
```

They solve different problems.

---

# Outputs and Environment Variables

Environment variables are different from job outputs.

Environment variable:

```yaml
env:
  IMAGE_TAG: a83f91c
```

Job output:

```yaml
outputs:
  image_tag: ${{ steps.image.outputs.tag }}
```

Use job outputs when a value needs to cross a job boundary.

---

# Outputs and Secrets

Do not use job outputs to move secrets between jobs.

Bad design:

```text
Secret
  |
  ↓
Job Output
  |
  ↓
Another Job
```

Secrets should be accessed securely using:

```yaml
${{ secrets.SECRET_NAME }}
```

or through an approved secret-management system.

Do not expose secrets through outputs unnecessarily.

---

# Production Security Principle

Use outputs for:

```text
Metadata
Identifiers
Versions
Non-sensitive configuration
```

Avoid outputs for:

```text
Passwords
API tokens
Private keys
Cloud credentials
Sensitive customer data
```

---

# Job Output Dependency

Example:

```yaml
build:

  runs-on: ubuntu-latest

  outputs:
    image_tag: ${{ steps.image.outputs.tag }}

  steps:

    - name: Generate Tag
      id: image
      run: |
        echo "tag=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

deploy:

  needs: build

  runs-on: ubuntu-latest

  steps:

    - name: Deploy
      run: |
        helm upgrade --install catalogue ./helm/catalogue \
          --set image.tag="${{ needs.build.outputs.image_tag }}"
```

Architecture:

```text
Build
 |
 └── image_tag
       |
       ↓
Deploy
 |
 └── Helm
       |
       └── image.tag
```

---

# Production Kubernetes Example

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.image.outputs.tag }}

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Generate Image Tag
        id: image
        run: |
          TAG="${GITHUB_SHA::7}"
          echo "tag=$TAG" >> "$GITHUB_OUTPUT"

      - name: Build Docker Image
        run: |
          docker build \
            -t catalogue:${GITHUB_SHA::7} \
            .

  deploy:

    needs: build

    runs-on: ubuntu-latest

    environment:
      name: production

    steps:

      - name: Deploy with Helm
        run: |
          helm upgrade --install catalogue ./helm/catalogue \
            --namespace catalogue \
            --create-namespace \
            --set image.tag="${{ needs.build.outputs.image_tag }}"

      - name: Verify Rollout
        run: |
          kubectl rollout status \
            deployment/catalogue \
            -n catalogue \
            --timeout=5m
```

---

# Multiple Job Outputs in a Production Pipeline

Example:

```yaml
jobs:

  build:

    runs-on: ubuntu-latest

    outputs:
      image_tag: ${{ steps.metadata.outputs.image_tag }}
      version: ${{ steps.metadata.outputs.version }}
      artifact_name: ${{ steps.metadata.outputs.artifact_name }}

    steps:

      - name: Generate Metadata
        id: metadata
        run: |
          echo "image_tag=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
          echo "version=1.5.0" >> "$GITHUB_OUTPUT"
          echo "artifact_name=application-${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"
```

Downstream:

```yaml
deploy:

  needs: build

  steps:

    - name: Display Deployment Metadata
      run: |
        echo "Image: ${{ needs.build.outputs.image_tag }}"
        echo "Version: ${{ needs.build.outputs.version }}"
        echo "Artifact: ${{ needs.build.outputs.artifact_name }}"
```

---

# Outputs in a DevSecOps Pipeline

A pipeline can produce metadata once and reuse it throughout the pipeline.

```text
Build
  |
  ├── image_tag
  ├── version
  └── artifact_name
          |
          ↓
     ┌────┼────┐
     ↓    ↓    ↓
   Test Security UAT
     |    |     |
     └────┼─────┘
          ↓
       Production
```

This improves consistency because every stage can refer to the same image/version information.

---

# Build Once, Promote Many with Outputs

A strong deployment pattern is:

```text
Build
  |
  ↓
Generate Image Tag
  |
  ↓
Build Image
  |
  ↓
Push Image
  |
  ↓
Output Image Tag
  |
  ↓
QA
  |
  ↓
UAT
  |
  ↓
Production
```

The same image tag is promoted through environments.

Example:

```text
catalogue:a83f91c
```

should remain the same artifact identifier.

Avoid:

```text
catalogue:qa
catalogue:uat
catalogue:prod
```

if those tags represent different rebuilt images.

---

# Output Validation

Do not blindly trust generated values.

Example:

```yaml
- name: Validate Image Tag
  run: |
    if [[ -z "${{ needs.build.outputs.image_tag }}" ]]; then
      echo "Image tag is empty"
      exit 1
    fi
```

For production, validate values before using them in deployment commands.

---

# Output Injection Considerations

Outputs can originate from commands or external data.

Do not blindly place untrusted output into shell commands.

Risky pattern:

```yaml
run: ./deploy.sh "${{ needs.build.outputs.value }}"
```

If the value can contain unexpected shell characters, it may create command-injection risks depending on how the shell interprets it.

Prefer passing values through environment variables when appropriate:

```yaml
env:
  DEPLOY_VERSION: ${{ needs.build.outputs.version }}

run: ./deploy.sh "$DEPLOY_VERSION"
```

Still validate the value before using it.

---

# Safe Output Pattern

Example:

```yaml
- name: Generate Image Tag
  id: image
  run: |
    TAG="${GITHUB_SHA::7}"

    if [[ ! "$TAG" =~ ^[a-f0-9]{7}$ ]]; then
      echo "Invalid image tag"
      exit 1
    fi

    echo "tag=$TAG" >> "$GITHUB_OUTPUT"
```

This makes the expected format explicit.

---

# Job Output and Deployment Metadata

Outputs can carry:

```text
Application Version
Commit SHA
Docker Image
Image Tag
Artifact Name
Release Version
Deployment ID
```

This allows later jobs to maintain traceability.

Example:

```text
JIRA Change Request
       |
       ↓
Commit SHA
       |
       ↓
Image Tag
       |
       ↓
UAT
       |
       ↓
Production
```

---

# Deployment Traceability

A production deployment should ideally be traceable to:

```text
Change Request
       |
       ↓
Commit SHA
       |
       ↓
Build
       |
       ↓
Image Tag
       |
       ↓
Security Scan
       |
       ↓
UAT
       |
       ↓
Production
```

Job outputs can help carry identifiers through this process.

---

# Output From One Job to Multiple Jobs

Example:

```text
             Build
               |
               ↓
          image_tag
               |
       ┌───────┼────────┐
       ↓       ↓        ↓
      QA    Security    UAT
```

Example:

```yaml
qa:
  needs: build

security:
  needs: build

uat:
  needs: build
```

All can access:

```yaml
${{ needs.build.outputs.image_tag }}
```

---

# Output From Multiple Jobs

A downstream job can depend on multiple jobs and access outputs from each.

Example:

```yaml
deploy:

  needs:
    - build
    - security

  steps:

    - name: Display Information
      run: |
        echo "Image: ${{ needs.build.outputs.image_tag }}"
        echo "Security Result: ${{ needs.security.result }}"
```

Conceptual flow:

```text
Build
 |
 └── image_tag

Security
 |
 └── result

     ↓

Deploy
```

---

# Job Result vs Job Output

These are different.

Job result:

```yaml
${{ needs.build.result }}
```

Possible values:

```text
success
failure
cancelled
skipped
```

Job output:

```yaml
${{ needs.build.outputs.version }}
```

Example:

```text
Build
 |
 ├── result = success
 |
 └── output
       └── version = v1.5.0
```

---

# Outputs and Failure

If the producing job fails before generating the expected output, the downstream job may not have the expected value.

Example:

```text
Build
  |
  ↓
Generate Output
  |
  X
Failed
```

Then:

```text
Deploy
  |
  ↓
Expected image_tag
  |
  X
Unavailable / unusable
```

Required validation should happen before deployment.

---

# Output Debugging

When an expected output is missing, check:

```text
1. Step has an id
2. Step writes to $GITHUB_OUTPUT
3. Output name matches exactly
4. Job output maps to correct step output
5. Consumer declares needs
6. Consumer references correct Job ID
7. Consumer references correct output name
8. Producing job succeeded
```

Example chain:

```text
id: image

↓

steps.image.outputs.tag

↓

outputs.image_tag

↓

needs.build.outputs.image_tag
```

Check every link.

---

# Common Output Mistake

Incorrect:

```yaml
outputs:
  version: ${{ steps.version.outputs.version }}
```

but the step does not have:

```yaml
id: version
```

Correct:

```yaml
- name: Generate Version
  id: version
  run: echo "version=v1.0.0" >> "$GITHUB_OUTPUT"
```

---

# Common Output Mistake

Incorrect:

```yaml
echo "version=v1.0.0"
```

This only prints the value.

Correct:

```yaml
echo "version=v1.0.0" >> "$GITHUB_OUTPUT"
```

The value must be written to the output file.

---

# Common Output Mistake

Incorrect:

```yaml
deploy:
  run: echo "${{ needs.build.outputs.tag }}"
```

without:

```yaml
needs: build
```

Correct:

```yaml
deploy:
  needs: build

  steps:
    - run: echo "${{ needs.build.outputs.tag }}"
```

---

# Common Output Mistake

Confusing:

```yaml
steps.version.outputs.version
```

with:

```yaml
needs.build.outputs.version
```

Remember:

```text
steps
→ current job

needs
→ dependent job
```

---

# Production Best Practices

- Use job outputs for small pieces of metadata.
- Use `$GITHUB_OUTPUT` to create step outputs.
- Give output-producing steps clear IDs.
- Use descriptive output names.
- Declare `needs` before consuming another job's outputs.
- Validate important deployment values.
- Do not pass secrets through outputs.
- Use artifacts for large files.
- Use consistent image tags across environments.
- Prefer immutable identifiers such as commit SHA-based tags.
- Keep build metadata traceable.
- Use outputs to support build-once/promote-many strategies.
- Treat externally influenced output values as untrusted until validated.

---

# Common Mistakes

- Forgetting the step `id`.
- Forgetting `$GITHUB_OUTPUT`.
- Using the wrong output name.
- Forgetting `needs`.
- Confusing step outputs with job outputs.
- Passing large files through outputs.
- Passing secrets through outputs.
- Using unvalidated output values in shell commands.
- Rebuilding artifacts instead of promoting the same artifact.
- Losing traceability between commit, image, and deployment.

---

# Summary

Job outputs allow information to move between jobs.

The basic pattern is:

```text
Step
  |
  ↓
$GITHUB_OUTPUT
  |
  ↓
Job Output
  |
  ↓
needs.<job>.outputs.<output>
```

Example:

```yaml
jobs:

  build:

    outputs:
      image_tag: ${{ steps.image.outputs.tag }}

    steps:

      - id: image
        run: echo "tag=${GITHUB_SHA::7}" >> "$GITHUB_OUTPUT"

  deploy:

    needs: build

    steps:

      - run: echo "${{ needs.build.outputs.image_tag }}"
```

Use:

```text
Outputs
→ small metadata

Artifacts
→ files

Secrets
→ sensitive values

needs
→ job dependency
```

For production CI/CD:

```text
Build
  |
  ├── image_tag
  ├── version
  └── artifact_name
          |
          ↓
       Validation
          |
          ↓
          UAT
          |
          ↓
       Production
```

The key principle is:

```text
Use job outputs to pass small, controlled,
non-sensitive values between dependent jobs.
```

---

# Interview Questions

## Basic

1. What are job outputs in GitHub Actions?
2. Why are job outputs needed?
3. What is `$GITHUB_OUTPUT`?
4. What is a step output?
5. What is the difference between step output and job output?
6. How do you access a job output from another job?

## Intermediate

7. Why does the consuming job need `needs`?
8. How do you create multiple job outputs?
9. What is the difference between `steps.<id>.outputs.<name>` and `needs.<job>.outputs.<name>`?
10. When should you use outputs instead of artifacts?
11. How do you pass a Docker image tag between jobs?
12. How do you pass an application version between jobs?
13. Can multiple jobs consume the same job output?

## Advanced

14. Design a pipeline where the Build job generates an immutable Docker image tag and passes it to QA, Security, UAT, and Production jobs.
15. A downstream job receives an empty output. Explain how you would troubleshoot the complete step-output-to-job-output chain.
16. Explain why job outputs should not be used to transfer secrets.
17. Explain why large files should be transferred using artifacts rather than job outputs.
18. Design a production "build once, promote many" pipeline using job outputs and artifacts.
19. A deployment uses a job output inside a shell command. Explain the security risks if the output contains untrusted data and how you would make the command safer.
20. Design a deployment pipeline that maintains traceability from JIRA change request → commit SHA → Docker image tag → UAT → production using job outputs.
21. Explain the difference between `needs.<job>.result` and `needs.<job>.outputs.<name>`.
22. A Build job generates `image_tag`, but Production cannot access it. Explain every configuration point you would check.