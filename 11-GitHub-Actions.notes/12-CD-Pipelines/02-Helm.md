# Helm

Helm is a package manager for Kubernetes that helps define, install, upgrade, configure, and manage Kubernetes applications.

Instead of maintaining many repeated Kubernetes YAML files, Helm allows teams to create reusable templates and provide environment-specific values.

A typical CD flow is:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        ↓
    Build / Test / Security
        |
        ↓
    Container Image
        |
        ↓
    ECR
        |
        ↓
    Helm
        |
        ↓
    Kubernetes
        |
        ↓
    Application

---

# Why Helm Is Used

Without Helm, a Kubernetes application may require many YAML files:

    deployment.yaml
    service.yaml
    configmap.yaml
    ingress.yaml
    hpa.yaml
    secret.yaml

For multiple environments, this can become difficult to maintain.

Example:

    DEV
      |
      +-- deployment.yaml
      +-- service.yaml
      +-- ingress.yaml

    QA
      |
      +-- deployment.yaml
      +-- service.yaml
      +-- ingress.yaml

    PROD
      |
      +-- deployment.yaml
      +-- service.yaml
      +-- ingress.yaml

Helm helps reduce duplication by using reusable templates and values.

---

# Helm Mental Model

Remember:

    Helm Chart
        |
        +-- Templates
        +-- Values
        +-- Chart Metadata
        |
        ↓
    Helm
        |
        ↓
    Kubernetes Manifests
        |
        ↓
    Kubernetes API
        |
        ↓
    Application

---

# Helm Chart

A Helm Chart is a package containing the files required to define a Kubernetes application.

A typical chart structure is:

    myapp/
    |
    ├── Chart.yaml
    ├── values.yaml
    |
    └── templates/
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        └── configmap.yaml

---

# Chart.yaml

`Chart.yaml` contains metadata about the Helm chart.

Example:

    apiVersion: v2
    name: myapp
    description: My application Helm chart
    type: application
    version: 1.0.0
    appVersion: "1.4.7"

Important fields include:

    apiVersion
    name
    description
    type
    version
    appVersion

---

# Chart Version

The chart `version` represents the version of the Helm chart itself.

Example:

    version: 1.0.0

If the Helm templates or chart configuration changes, the chart version may change.

---

# App Version

`appVersion` represents the version of the application being deployed.

Example:

    appVersion: "1.4.7"

The application version and chart version can be different.

Example:

    Chart Version:
        2.3.0

    Application Version:
        1.4.7

---

# values.yaml

`values.yaml` contains configurable values used by the chart templates.

Example:

    replicaCount: 3

    image:
      repository: myapp
      tag: "1.4.7"

    service:
      type: ClusterIP
      port: 80

The values can be overridden during deployment.

---

# Helm Templates

Templates contain Kubernetes YAML with dynamic values.

Example concept:

    replicas: {{ .Values.replicaCount }}

The value comes from:

    values.yaml

For example:

    replicaCount: 3

The rendered Kubernetes manifest becomes:

    replicas: 3

---

# Helm Template Flow

The process is:

    values.yaml
        |
        ↓
    templates/
        |
        ↓
    Helm Template Engine
        |
        ↓
    Kubernetes YAML
        |
        ↓
    Kubernetes

---

# Helm Values

Values allow the same chart to be configured differently.

Example:

    values-dev.yaml
    values-qa.yaml
    values-uat.yaml
    values-prod.yaml

The chart remains reusable.

---

# Environment-Specific Values

Example:

    DEV

    replicaCount: 1

    QA

    replicaCount: 2

    PROD

    replicaCount: 5

Same chart:

    myapp

Different configuration:

    DEV → 1 replica
    QA  → 2 replicas
    PROD → 5 replicas

---

# Helm Chart Structure

A common production structure is:

    myapp/
    |
    ├── Chart.yaml
    ├── values.yaml
    ├── .helmignore
    |
    └── templates/
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        ├── configmap.yaml
        ├── secret.yaml
        ├── hpa.yaml
        ├── serviceaccount.yaml
        └── _helpers.tpl

---

# templates Directory

The `templates` directory contains Kubernetes resource templates.

Common templates:

    deployment.yaml
    service.yaml
    ingress.yaml
    configmap.yaml
    secret.yaml
    hpa.yaml

Helm renders these templates using values.

---

# _helpers.tpl

`_helpers.tpl` is commonly used to define reusable template helpers.

Examples of reusable logic include:

    Application Name
    Full Resource Name
    Labels
    Selector Labels

This helps keep templates consistent.

---

# .helmignore

`.helmignore` specifies files that should not be included when packaging the chart.

It works similarly in concept to:

    .gitignore

It helps keep unnecessary files out of the packaged chart.

---

# Helm CLI

The main command-line tool is:

    helm

Common commands include:

    helm version

    helm create

    helm lint

    helm template

    helm install

    helm upgrade

    helm upgrade --install

    helm list

    helm status

    helm history

    helm rollback

    helm uninstall

    helm package

    helm repo

---

# Check Helm Version

Command:

    helm version

This displays the installed Helm version.

It is useful for verifying that Helm is installed and available in the deployment runner.

---

# Create a Helm Chart

Command:

    helm create myapp

This creates a starter chart structure.

Result:

    myapp/
    |
    ├── Chart.yaml
    ├── values.yaml
    ├── charts/
    ├── templates/
    └── .helmignore

The generated structure can then be customized for the application.

---

# Helm Lint

Command:

    helm lint ./myapp

`helm lint` checks a chart for potential problems.

It is useful as an early validation step in CI.

Flow:

    Helm Chart
        |
        ↓
    helm lint
        |
        +-- PASS
        |
        +-- FAIL

---

# Helm Template

Command:

    helm template myapp ./myapp

This renders the Helm templates locally without installing the release into Kubernetes.

Flow:

    Chart
      |
      ↓
    helm template
      |
      ↓
    Rendered Kubernetes YAML

This is useful for inspecting what Helm will generate.

---

# Helm Template With Values

Example:

    helm template myapp ./myapp \
      -f values-prod.yaml

The command renders the chart using production-specific values.

---

# Helm Install

Command:

    helm install myapp ./myapp

This installs a Helm release.

Flow:

    Helm Chart
        |
        ↓
    helm install
        |
        ↓
    Kubernetes
        |
        ↓
    Resources

---

# Helm Release

A release is an installed instance of a Helm chart.

Example:

    Chart:
        myapp

    Release:
        myapp

The same chart can be installed multiple times using different release names or namespaces.

---

# Release Name

Example:

    helm install myapp ./myapp

Here:

    myapp

is the release name.

You can inspect it with:

    helm list

---

# Helm List

Command:

    helm list

It shows Helm releases in the current namespace.

For all namespaces:

    helm list --all-namespaces

This is useful for deployment verification.

---

# Helm Status

Command:

    helm status myapp

This provides information about the current Helm release.

It can help determine whether a release was successfully deployed.

---

# Helm Upgrade

Command:

    helm upgrade myapp ./myapp

This updates an existing Helm release.

Typical flow:

    Existing Release
        |
        ↓
    New Chart / Values
        |
        ↓
    helm upgrade
        |
        ↓
    Kubernetes
        |
        ↓
    New Application Version

---

# Helm Upgrade With New Image

Example:

    helm upgrade myapp ./myapp \
      --set image.tag=1.4.7

The deployment uses the new application image version.

---

# Helm Install or Upgrade

A common CD command is:

    helm upgrade --install myapp ./myapp

Meaning:

    Release Exists
        |
        +-- YES → Upgrade
        |
        +-- NO → Install

This makes the command useful for automated deployment pipelines.

---

# Helm Upgrade With Values

Example:

    helm upgrade --install myapp ./myapp \
      -f values-prod.yaml

This deploys the chart using production-specific values.

---

# Helm Upgrade With Multiple Values Files

Values can be layered.

Example:

    helm upgrade --install myapp ./myapp \
      -f values.yaml \
      -f values-prod.yaml

The later values file can override earlier values.

---

# Helm Set

Values can also be overridden using:

    --set

Example:

    helm upgrade --install myapp ./myapp \
      --set image.tag=1.4.7

This is useful for dynamically changing values during CI/CD.

---

# Values Precedence

A simplified precedence model is:

    Chart Default Values
          |
          ↓
    values.yaml
          |
          ↓
    Environment Values
          |
          ↓
    --set
          |
          ↓
    Final Value

The exact behavior depends on how values are supplied and merged.

---

# Environment Values Example

Common structure:

    helm/
    |
    ├── Chart.yaml
    ├── values.yaml
    ├── values-dev.yaml
    ├── values-qa.yaml
    ├── values-uat.yaml
    ├── values-prod.yaml
    |
    └── templates/

The same chart can be used across environments.

---

# DEV Deployment

Example:

    helm upgrade --install myapp ./myapp \
      -f values-dev.yaml

Flow:

    Chart
      |
      ↓
    DEV Values
      |
      ↓
    Helm
      |
      ↓
    DEV Kubernetes

---

# QA Deployment

Example:

    helm upgrade --install myapp ./myapp \
      -f values-qa.yaml

Flow:

    Chart
      |
      ↓
    QA Values
      |
      ↓
    Helm
      |
      ↓
    QA Kubernetes

---

# UAT Deployment

Example:

    helm upgrade --install myapp ./myapp \
      -f values-uat.yaml

---

# Production Deployment

Example:

    helm upgrade --install myapp ./myapp \
      -f values-prod.yaml

Production should normally have additional deployment protection and approval controls.

---

# Helm Namespace

A Helm release can be installed into a specific namespace.

Example:

    helm upgrade --install myapp ./myapp \
      --namespace production \
      --create-namespace

The namespace provides logical separation.

---

# Multi-Environment Namespaces

Example:

    Kubernetes Cluster
        |
        +-- dev
        |    |
        |    +-- myapp
        |
        +-- qa
        |    |
        |    +-- myapp
        |
        +-- uat
        |    |
        |    +-- myapp
        |
        +-- production
             |
             +-- myapp

The same Helm chart can be deployed into different namespaces.

---

# Helm and Kubernetes

Helm does not replace Kubernetes.

Helm provides packaging, templating, release management, and deployment capabilities.

The relationship is:

    Helm
      |
      ↓
    Kubernetes API
      |
      ↓
    Kubernetes Resources

---

# Helm and kubectl

`kubectl` communicates directly with Kubernetes.

Example:

    kubectl apply -f deployment.yaml

Helm generates and manages Kubernetes resources using chart templates.

Example:

    helm upgrade --install myapp ./myapp

Both ultimately interact with Kubernetes resources.

---

# Helm vs Raw YAML

Raw YAML:

    deployment.yaml
    service.yaml
    ingress.yaml

Helm:

    Chart
      |
      +-- Templates
      +-- Values
      +-- Release Management

Helm becomes especially useful when the same application must be deployed repeatedly with different configurations.

---

# Helm Benefits

Helm provides:

- Reusable charts
- Templating
- Versioned releases
- Environment-specific configuration
- Upgrade support
- Rollback support
- Packaging
- Dependency management
- Release history
- Standardized deployments

---

# Helm Dependencies

A Helm chart can depend on other charts.

Example:

    Application Chart
        |
        +-- Redis
        +-- PostgreSQL
        +-- Other Dependency

Dependencies can be defined in chart metadata.

This is useful when applications require additional packaged components.

---

# Helm Dependency Commands

Common commands include:

    helm dependency update

    helm dependency build

    helm dependency list

These help manage chart dependencies.

---

# Helm Repository

Helm repositories store and distribute charts.

Common concepts:

    Repository
        |
        ↓
    Helm Chart
        |
        ↓
    helm install

A team can use an internal chart repository for enterprise applications.

---

# Helm Repo Add

Command:

    helm repo add <repo-name> <repo-url>

This adds a chart repository to the local Helm configuration.

The repository URL should come from the trusted chart provider or organization's internal platform.

---

# Helm Repo Update

Command:

    helm repo update

This refreshes information about available charts from configured repositories.

---

# Helm Search

Example:

    helm search repo <keyword>

This searches configured Helm repositories.

---

# Helm Package

Command:

    helm package ./myapp

This packages the chart into a `.tgz` archive.

Example:

    myapp-1.0.0.tgz

The package can then be stored or distributed through a chart repository.

---

# Helm Chart Versioning

Helm charts can be versioned.

Example:

    myapp-1.0.0
    myapp-1.1.0
    myapp-2.0.0

Chart versions should be managed consistently.

---

# Application Version vs Chart Version

Example:

    Chart:
        version: 2.1.0

    Application:
        appVersion: "1.5.2"

The chart version identifies the chart package.

The application version identifies the application.

---

# Helm Release History

Helm maintains release history.

Command:

    helm history myapp

This can show previous revisions.

Example concept:

    Revision 1 → v1.4.5
    Revision 2 → v1.4.6
    Revision 3 → v1.4.7

This is useful for troubleshooting and rollback.

---

# Helm Rollback

Command:

    helm rollback myapp <revision>

Example:

    helm rollback myapp 2

This restores a previous Helm release revision.

Flow:

    v1.4.7
       |
       ↓
      FAIL
       |
       ↓
    Helm Rollback
       |
       ↓
    Previous Revision
       |
       ↓
    v1.4.6

---

# Helm Rollback vs Kubernetes Rollback

Kubernetes:

    kubectl rollout undo deployment/myapp

Helm:

    helm rollback myapp <revision>

When Helm manages the release, Helm rollback can restore the release to a previous Helm revision.

The appropriate rollback method depends on how the application is deployed and managed.

---

# Helm Uninstall

Command:

    helm uninstall myapp

This removes the Helm release and the resources managed by that release according to Helm's release behavior.

Use carefully, especially in production.

---

# Helm Dry Run

Helm can simulate an operation without actually applying it.

Example:

    helm upgrade --install myapp ./myapp \
      --dry-run

This is useful for checking rendered resources before deployment.

---

# Helm Debug

The `--debug` option provides additional output.

Example:

    helm template myapp ./myapp \
      --debug

This can help troubleshoot template rendering problems.

---

# Helm Lint in CI

A good CI pipeline can include:

    Checkout
        |
        ↓
    Helm Lint
        |
        ↓
    Helm Template
        |
        ↓
    Security / Policy Checks
        |
        ↓
    Package
        |
        ↓
    Publish

Helm validation should happen before production deployment.

---

# Helm CD Pipeline

A typical Helm CD flow is:

    GitHub
        |
        ↓
    GitHub Actions
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Push Image
        |
        ↓
    ECR
        |
        ↓
    Helm Upgrade
        |
        ↓
    Kubernetes
        |
        ↓
    Rollout
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

---

# Helm With Docker

Typical flow:

    Source Code
        |
        ↓
    Docker Build
        |
        ↓
    Image
        |
        ↓
    Trivy
        |
        ↓
    ECR
        |
        ↓
    Helm
        |
        ↓
    Kubernetes

Helm does not build the Docker image.

Helm deploys the Kubernetes resources that reference the image.

---

# Helm Image Configuration

Example values:

    image:
      repository: 123456789012.dkr.ecr.region.amazonaws.com/myapp
      tag: "1.4.7"
      pullPolicy: IfNotPresent

The Deployment template can reference these values.

---

# Helm Deployment Template

Conceptually:

    containers:
      - name: myapp
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"

Values:

    image:
      repository: myapp
      tag: "1.4.7"

Helm renders the final Kubernetes manifest.

---

# Helm Service Template

Conceptually:

    apiVersion: v1

    kind: Service

    metadata:
      name: {{ include "myapp.fullname" . }}

    spec:
      selector:
        app.kubernetes.io/name: {{ include "myapp.name" . }}

      ports:
        - port: {{ .Values.service.port }}
          targetPort: {{ .Values.service.targetPort }}

The values control the generated Service configuration.

---

# Helm Ingress Template

Ingress can also be templated.

Environment-specific values may control:

    Host
    Paths
    TLS
    Service Name
    Service Port

Example concept:

    ingress:
      enabled: true
      host: app.example.com

Different environments can use different hosts.

---

# Helm ConfigMap Template

A ConfigMap can be templated.

Example concept:

    data:
      APP_ENV: {{ .Values.app.environment }}
      LOG_LEVEL: {{ .Values.app.logLevel }}

DEV:

    environment: development

PROD:

    environment: production

---

# Helm Secret Template

Secrets can also be templated, but sensitive values should not be casually placed into Git-managed values files.

Better approaches include:

    External Secret Management
    Cloud Secret Manager
    Sealed Secrets
    External Secrets
    Appropriate Kubernetes Secret Handling

The selected solution depends on the organization's architecture and security requirements.

---

# Helm and AWS Secrets

For AWS workloads, secrets can be managed through appropriate AWS services and injected into Kubernetes using a supported integration.

Conceptually:

    AWS Secret Manager
          |
          ↓
    Kubernetes Integration
          |
          ↓
    Kubernetes Secret
          |
          ↓
    Application

Do not commit production passwords or credentials into `values-prod.yaml`.

---

# Helm Environment Separation

Example:

    values.yaml
        |
        ↓
    Common Configuration

    values-dev.yaml
        |
        ↓
    DEV Overrides

    values-qa.yaml
        |
        ↓
    QA Overrides

    values-prod.yaml
        |
        ↓
    PROD Overrides

This allows the chart to remain reusable.

---

# Common Values

Common configuration can be stored in:

    values.yaml

Example:

    service:
      port: 80

    image:
      repository: myapp

Environment-specific values can override only what differs.

---

# Production Values

Example:

    replicaCount: 5

    image:
      tag: "1.4.7"

    resources:
      requests:
        cpu: "500m"
        memory: "512Mi"

      limits:
        cpu: "1"
        memory: "1Gi"

Production values should be reviewed and managed carefully.

---

# Helm and GitOps

Helm works very well with GitOps tools such as ArgoCD.

Flow:

    Git
      |
      ↓
    Helm Chart
      |
      ↓
    ArgoCD
      |
      ↓
    Kubernetes

ArgoCD can render and deploy Helm charts to Kubernetes.

---

# Helm + ArgoCD Architecture

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        ↓
    Build / Test / Scan
        |
        ↓
    ECR
        |
        ↓
    GitOps Repository
        |
        ↓
    Helm
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Application

---

# Helm With ArgoCD Environments

Example:

    GitOps Repository
        |
        +-- dev
        |    |
        |    +-- Helm Values
        |
        +-- qa
        |    |
        |    +-- Helm Values
        |
        +-- uat
        |    |
        |    +-- Helm Values
        |
        +-- prod
             |
             +-- Helm Values

The same Helm chart can be used with environment-specific values.

---

# Helm Release Naming

Choose predictable release names.

Example:

    myapp-dev
    myapp-qa
    myapp-uat
    myapp-prod

Or use the same release name in separate namespaces:

    Namespace: dev
        Release: myapp

    Namespace: qa
        Release: myapp

    Namespace: prod
        Release: myapp

Consistency is important.

---

# Helm Namespace Strategy

One possible approach:

    Cluster
       |
       +-- dev
       |    |
       |    +-- Helm Release
       |
       +-- qa
       |    |
       |    +-- Helm Release
       |
       +-- prod
            |
            +-- Helm Release

This provides logical environment separation.

---

# Helm Upgrade Strategy

A production deployment can follow:

    New Image
        |
        ↓
    Update Image Tag
        |
        ↓
    Helm Lint
        |
        ↓
    Helm Template
        |
        ↓
    Approval
        |
        ↓
    Helm Upgrade
        |
        ↓
    Rollout
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

---

# Helm Atomic Deployment

Helm supports atomic upgrade behavior.

Conceptually:

    helm upgrade --install myapp ./myapp \
      --atomic

With `--atomic`, Helm waits for the operation and can roll back when the upgrade fails according to Helm's behavior.

This can be useful for controlled releases, but the timeout and application health behavior must be configured appropriately.

---

# Helm Timeout

Helm operations can have a timeout.

Example:

    helm upgrade --install myapp ./myapp \
      --timeout 10m

The timeout should reflect realistic application startup and rollout behavior.

---

# Helm Wait

Helm can wait for resources to become ready.

Example:

    helm upgrade --install myapp ./myapp \
      --wait

This can make the CD pipeline wait for Kubernetes resources to reach the expected state.

For production deployments, combine this with appropriate health checks and rollout validation.

---

# Helm Atomic + Wait

A common controlled deployment pattern can use:

    helm upgrade --install myapp ./myapp \
      --atomic \
      --wait \
      --timeout 10m

This tells Helm to wait for the release and handle failed upgrades according to atomic behavior.

The exact settings should be tested with the application.

---

# Helm Deployment Validation

After Helm deployment:

    helm status myapp
        |
        ↓
    kubectl get pods
        |
        ↓
    kubectl rollout status
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

A successful Helm command alone does not always prove that the application is fully healthy from the user's perspective.

---

# Helm Troubleshooting

When a Helm deployment fails, check:

    helm status myapp

    helm history myapp

    helm get values myapp

    helm get manifest myapp

    kubectl get pods

    kubectl describe pod <pod-name>

    kubectl logs <pod-name>

    kubectl get events

These commands help identify whether the problem is in Helm rendering, Kubernetes resources, configuration, or application startup.

---

# helm get values

Command:

    helm get values myapp

This shows the values associated with the release.

For all values:

    helm get values myapp --all

This can help troubleshoot unexpected configuration.

---

# helm get manifest

Command:

    helm get manifest myapp

This shows the Kubernetes manifests associated with the release.

This is useful when investigating what Helm actually deployed.

---

# Helm Template Troubleshooting

If a template fails:

    helm template myapp ./myapp

Inspect:

    YAML Syntax
    Template Syntax
    Values
    Indentation
    Missing Variables
    Incorrect Functions

Run:

    helm lint ./myapp

before deployment.

---

# Helm YAML Indentation

YAML indentation is significant.

Incorrect indentation can produce invalid Kubernetes manifests.

Therefore:

    helm lint

and:

    helm template

should be part of validation.

---

# Helm Template Functions

Helm uses Go template syntax and provides template functions.

Examples include functions for:

    String Processing
    Default Values
    Formatting
    Encoding
    Conditional Logic

Example concept:

    {{ .Values.image.tag }}

Helm templates can become complex, so templates should be kept readable.

---

# Helm Conditional Logic

Templates can conditionally create resources.

Example concept:

    {{- if .Values.ingress.enabled }}

    Ingress Resource

    {{- end }}

If:

    ingress:
      enabled: false

the Ingress resource can be omitted from the rendered output.

---

# Helm Loops

Helm templates can iterate over lists.

This is useful for repeated configuration.

Example concept:

    {{- range .Values.hosts }}

    {{ . }}

    {{- end }}

Use loops carefully so generated manifests remain understandable.

---

# Helm Default Values

A template can provide default behavior.

Example concept:

    {{ .Values.service.port | default 80 }}

This can prevent missing optional values from causing unexpected behavior.

---

# Helm Naming Helpers

Using helper templates can standardize resource names.

Example concept:

    {{ include "myapp.fullname" . }}

This helps avoid repeating naming logic across:

    Deployment
    Service
    ConfigMap
    Ingress

---

# Helm Labels

Consistent labels help with:

    Selection
    Monitoring
    Troubleshooting
    Organization

Common labels include:

    app.kubernetes.io/name
    app.kubernetes.io/instance
    app.kubernetes.io/version
    app.kubernetes.io/managed-by

---

# Helm and Kubernetes Labels

Example relationship:

    Deployment
       |
       +-- labels
       |
       ↓
    Pod Template
       |
       +-- labels
       |
       ↓
    Service Selector

Labels must be consistent where selectors depend on them.

---

# Helm CD Security

Important practices:

- Do not commit secrets into values files
- Use least-privilege Kubernetes permissions
- Protect production environments
- Validate charts
- Scan images
- Review changes
- Restrict chart repositories
- Use trusted dependencies
- Pin dependency versions where appropriate
- Audit production releases
- Use immutable image versions
- Restrict Helm access in production

---

# Helm Dependency Security

Third-party charts should be evaluated before use.

Check:

    Chart Source
    Maintainer
    Version
    Dependencies
    Security History
    Configuration
    Permissions

Do not blindly install unknown charts into production clusters.

---

# Helm Chart Testing

A chart should be tested before deployment.

Possible checks:

    helm lint
        |
        ↓
    helm template
        |
        ↓
    Kubernetes Validation
        |
        ↓
    Security / Policy Checks
        |
        ↓
    Deployment

Additional tests can use Kubernetes test environments.

---

# Helm CI Pipeline

A Helm-focused CI pipeline can be:

    Checkout
        |
        ↓
    Helm Lint
        |
        ↓
    Helm Template
        |
        ↓
    Manifest Validation
        |
        ↓
    Security / Policy Scan
        |
        ↓
    Package Chart
        |
        ↓
    Publish Chart

---

# Helm CD Pipeline

A Helm-focused CD pipeline can be:

    Select Environment
        |
        ↓
    Select Image Version
        |
        ↓
    Load Environment Values
        |
        ↓
    Helm Upgrade
        |
        ↓
    Wait
        |
        ↓
    Rollout Validation
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Deployment Success

---

# Helm With GitHub Actions

A typical pipeline structure is:

    GitHub Actions
        |
        +-- Checkout
        |
        +-- Helm Lint
        |
        +-- Helm Template
        |
        +-- AWS Authentication
        |
        +-- ECR Authentication
        |
        +-- Helm Upgrade
        |
        +-- Deployment Validation

---

# GitHub Actions Helm Deployment Example

Conceptual workflow:

    name: Helm Deployment

    on:
      workflow_dispatch:

    jobs:

      deploy:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Helm Lint
            run: |
              helm lint ./helm/myapp

          - name: Render Templates
            run: |
              helm template myapp ./helm/myapp \
                -f ./helm/myapp/values-prod.yaml

          - name: Configure AWS Credentials
            uses: aws-actions/configure-aws-credentials@v6

          - name: Update kubeconfig
            run: |
              aws eks update-kubeconfig \
                --region <region> \
                --name <cluster>

          - name: Deploy
            run: |
              helm upgrade --install myapp ./helm/myapp \
                --namespace production \
                --create-namespace \
                -f ./helm/myapp/values-prod.yaml \
                --wait \
                --timeout 10m

          - name: Verify
            run: |
              helm status myapp \
                --namespace production

---

# Important Point About the Example

The workflow demonstrates the deployment structure.

In a real environment, configure:

    AWS OIDC
    IAM Role
    EKS Permissions
    Production Environment Protection
    Image Repository
    Image Tag
    Namespace
    Values Files

according to the organization's architecture.

---

# Helm Deployment With Dynamic Image Tag

A common CD pattern is:

    CI
      |
      ↓
    Build Image
      |
      ↓
    Push ECR
      |
      ↓
    Image Tag
      |
      ↓
    Helm Deployment

Example:

    helm upgrade --install myapp ./helm/myapp \
      -f values-prod.yaml \
      --set image.tag=1.4.7

This allows the pipeline to deploy the newly built image.

---

# Helm With Commit SHA

Instead of a release version, the pipeline can use the Git commit SHA.

Example:

    image:
      tag: abc123def456

Flow:

    Git Commit
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    Helm
        |
        ↓
    Kubernetes

This creates a direct relationship between source and deployed image.

---

# Helm and Artifact Promotion

Build once:

    Docker Build
        |
        ↓
    Image: myapp:1.4.7
        |
        ↓
    ECR

Promote:

    myapp:1.4.7
        |
        +-- DEV
        +-- QA
        +-- UAT
        +-- PROD

Helm changes the deployment configuration, not the application artifact itself.

---

# Helm Environment Promotion

Example:

    Build Image
        |
        ↓
    ECR
        |
        ↓
    Helm DEV
        |
        ↓
    Validation
        |
        ↓
    Helm QA
        |
        ↓
    Validation
        |
        ↓
    Helm UAT
        |
        ↓
    Approval
        |
        ↓
    Helm PROD

---

# Helm Rollback Strategy

A production deployment should have a rollback plan.

Example:

    v1.4.6
       |
       ↓
    Production

    v1.4.7
       |
       ↓
    Helm Upgrade
       |
       ↓
      FAIL
       |
       ↓
    Helm Rollback
       |
       ↓
    v1.4.6

Helm history makes release revisions easier to identify.

---

# Helm Rollback Command

Example:

    helm history myapp

Then:

    helm rollback myapp 2

After rollback:

    helm status myapp

Then validate:

    kubectl get pods

    kubectl rollout status deployment/myapp

    Health Check

---

# Helm and Zero Downtime

Helm itself does not automatically guarantee zero downtime.

Zero-downtime behavior depends on Kubernetes configuration such as:

    Replica Count
    RollingUpdate
    Readiness Probes
    maxUnavailable
    maxSurge
    Application Startup
    Service Configuration

Helm provides the mechanism to configure these Kubernetes resources.

---

# Helm and Rolling Updates

Typical flow:

    Helm Upgrade
        |
        ↓
    Kubernetes Deployment
        |
        ↓
    New Pods
        |
        ↓
    Readiness Checks
        |
        ↓
    Traffic
        |
        ↓
    Old Pods Removed

This creates a controlled application update.

---

# Helm and Health Checks

A deployment should not be considered successful only because Helm returned success.

Use:

    Helm Status
        |
        ↓
    Pod Readiness
        |
        ↓
    Service Validation
        |
        ↓
    Application Health
        |
        ↓
    Smoke Tests

This provides stronger validation.

---

# Helm Production Deployment Checklist

Before production deployment:

    [ ] Chart lint passes
    [ ] Templates render successfully
    [ ] Image exists
    [ ] Image tag is immutable
    [ ] Security scan passed
    [ ] Environment values reviewed
    [ ] Secrets are not hardcoded
    [ ] Namespace is correct
    [ ] Kubernetes access is correct
    [ ] Production approval completed
    [ ] Helm upgrade executed
    [ ] Rollout completed
    [ ] Pods are Ready
    [ ] Health checks pass
    [ ] Smoke tests pass
    [ ] Rollback plan available
    [ ] Deployment recorded

---

# Helm Troubleshooting Checklist

When a Helm deployment fails:

    [ ] helm status
    [ ] helm history
    [ ] helm get values
    [ ] helm get manifest
    [ ] helm lint
    [ ] helm template
    [ ] kubectl get pods
    [ ] kubectl describe pod
    [ ] kubectl logs
    [ ] kubectl get events
    [ ] Check image
    [ ] Check ConfigMap
    [ ] Check Secret
    [ ] Check Service
    [ ] Check Ingress
    [ ] Check readiness probe
    [ ] Check resource limits
    [ ] Check namespace

---

# Common Helm Errors

Common problems include:

    Template Rendering Failure
    Missing Value
    Invalid YAML
    Incorrect Indentation
    Wrong Namespace
    Wrong Image Tag
    ImagePullBackOff
    CrashLoopBackOff
    Failed Readiness Probe
    Invalid Service Selector
    Missing Secret
    Invalid Ingress Configuration
    Timeout
    Insufficient Kubernetes Resources

---

# Helm Scenario

## Helm Upgrade Failed. What Would You Do?

I would first determine whether the failure occurred during:

    Template Rendering
    Helm Operation
    Kubernetes Resource Creation
    Pod Startup
    Health Checks

Commands:

    helm status myapp

    helm history myapp

    helm get manifest myapp

    kubectl get pods

    kubectl describe pod <pod-name>

    kubectl logs <pod-name>

    kubectl get events

If the release is unhealthy and rollback is required:

    helm rollback myapp <revision>

Then validate the previous version.

---

# Helm Scenario

## How Would You Deploy the Same Application to DEV, QA, and PROD?

I would create one reusable Helm chart and separate environment-specific values.

Example:

    Chart
      |
      +-- values-dev.yaml
      +-- values-qa.yaml
      +-- values-prod.yaml

Deployment:

    DEV:
      helm upgrade --install ... -f values-dev.yaml

    QA:
      helm upgrade --install ... -f values-qa.yaml

    PROD:
      helm upgrade --install ... -f values-prod.yaml

The application image can remain the same while environment-specific configuration changes.

---

# Helm Scenario

## How Would You Deploy a New Docker Image?

First:

    Build Image
        |
        ↓
    Scan Image
        |
        ↓
    Push ECR

Then:

    Helm Upgrade
        |
        ↓
    --set image.tag=<new-version>
        |
        ↓
    Kubernetes
        |
        ↓
    Rollout

Finally:

    Health Check
        |
        ↓
    Smoke Test

---

# Helm Scenario

## Why Use Helm Instead of Many Kubernetes YAML Files?

I would use Helm when I need:

    Reusable Templates
    Environment-Specific Values
    Release Management
    Versioning
    Upgrade Support
    Rollback
    Packaging
    Dependency Management

Instead of maintaining separate complete manifests for every environment, I can maintain one reusable chart and provide environment-specific values.

---

# Helm Scenario

## What Is the Difference Between Helm and Kubernetes?

Kubernetes is the container orchestration platform.

Helm is a package manager and templating/release tool for Kubernetes.

Relationship:

    Helm
      |
      ↓
    Kubernetes API
      |
      ↓
    Kubernetes Resources

Helm does not replace Kubernetes.

---

# Helm Scenario

## What Is the Difference Between Chart Version and App Version?

Chart version:

    version: 2.0.0

represents the Helm chart version.

Application version:

    appVersion: "1.4.7"

represents the application version.

They can change independently.

---

# Helm Scenario

## What Is a Helm Release?

A Helm release is an installed instance of a Helm chart.

Example:

    Chart:
        myapp

    Release:
        myapp-prod

Helm tracks the release and its revisions.

---

# Helm Scenario

## How Does Helm Support Rollback?

Helm maintains release history.

Example:

    Revision 1 → v1.4.5
    Revision 2 → v1.4.6
    Revision 3 → v1.4.7

If revision 3 fails:

    helm rollback myapp 2

This restores the previous release configuration.

---

# Helm Scenario

## How Would You Secure Helm in Production?

I would:

    1. Restrict production access
    2. Use least-privilege RBAC
    3. Protect GitHub production environments
    4. Require approvals where appropriate
    5. Avoid hardcoded secrets
    6. Scan container images
    7. Validate Helm charts
    8. Use trusted chart dependencies
    9. Use immutable image versions
    10. Maintain release audit history

---

# Helm Scenario

## How Would You Integrate Helm With GitHub Actions?

The flow would be:

    GitHub
       |
       ↓
    GitHub Actions
       |
       +-- Checkout
       +-- Helm Lint
       +-- Helm Template
       +-- Build / Test
       +-- Security Scan
       |
       ↓
    ECR
       |
       ↓
    AWS Authentication
       |
       ↓
    EKS
       |
       ↓
    Helm Upgrade
       |
       ↓
    Rollout Validation
       |
       ↓
    Health Check

---

# Helm Scenario

## How Would You Integrate Helm With ArgoCD?

I would store the Helm chart and environment configuration in Git.

Example:

    GitOps Repository
        |
        +-- Helm Chart
        |
        +-- DEV Values
        +-- QA Values
        +-- UAT Values
        +-- PROD Values
        |
        ↓
      ArgoCD
        |
        ↓
      EKS

ArgoCD continuously compares the desired Git state with the Kubernetes state.

---

# Helm Scenario

## What Would You Do If the Same Chart Works in DEV but Fails in PROD?

I would compare:

    Chart Version
    Values
    Image Version
    Secrets
    ConfigMaps
    Resources
    Namespace
    Kubernetes Version
    Ingress
    Service
    Environment Variables

First I would render the production configuration:

    helm template myapp ./myapp \
      -f values-prod.yaml

Then compare it with the working DEV rendering.

I would also inspect the deployed Kubernetes resources and Pod events.

---

# Helm Scenario

## How Would You Prevent a Wrong Image From Being Deployed?

I would:

    1. Use immutable image tags
    2. Pass the exact image version to Helm
    3. Validate the image exists
    4. Record commit SHA
    5. Scan the image before promotion
    6. Avoid `latest`
    7. Validate the deployed image after rollout

Example:

    CI:
      myapp:1.4.7

    Helm:
      image.tag=1.4.7

    Kubernetes:
      myapp:1.4.7

---

# Helm Scenario

## How Would You Validate a Helm Deployment?

I would use multiple levels:

    Helm Lint
        |
        ↓
    Helm Template
        |
        ↓
    Helm Upgrade
        |
        ↓
    Helm Status
        |
        ↓
    Kubernetes Rollout
        |
        ↓
    Pod Readiness
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test

---

# Helm Scenario

## What Is Build Once, Deploy Many With Helm?

The application is built once:

    Source
      |
      ↓
    Docker Build
      |
      ↓
    Image: 1.4.7
      |
      ↓
    ECR

Then Helm deploys the same image:

    DEV
      |
      ↓
    QA
      |
      ↓
    UAT
      |
      ↓
    PROD

Helm values can change environment-specific configuration without rebuilding the application.

---

# Helm Best Practices

- Keep charts reusable
- Keep templates readable
- Use meaningful values
- Use environment-specific values files
- Use immutable image versions
- Avoid `latest`
- Run `helm lint`
- Run `helm template`
- Validate generated manifests
- Protect production deployments
- Use least-privilege RBAC
- Keep secrets out of Git
- Use trusted dependencies
- Pin dependency versions where appropriate
- Use release history
- Test rollback
- Use health checks
- Use deployment timeouts
- Use appropriate rollout settings
- Keep charts version controlled
- Keep application and chart versions clear
- Use GitOps where appropriate

---

# Helm Anti-Patterns

## Anti-Pattern 1: Hardcoded Environment Configuration

Bad:

    deployment.yaml
        |
        +-- DEV Database
        +-- QA Database
        +-- PROD Database

Better:

    Helm Chart
        |
        +-- values-dev.yaml
        +-- values-qa.yaml
        +-- values-prod.yaml

---

# Anti-Pattern 2: Secrets in Git

Bad:

    values-prod.yaml

    databasePassword: "MyProductionPassword"

Better:

    External Secret Management
        |
        ↓
    Kubernetes

Sensitive production credentials should not be committed as plain text.

---

# Anti-Pattern 3: Using latest

Bad:

    image:
      tag: latest

Better:

    image:
      tag: "1.4.7"

---

# Anti-Pattern 4: No Helm Validation

Bad:

    helm upgrade

Better:

    helm lint
        |
        ↓
    helm template
        |
        ↓
    helm upgrade
        |
        ↓
    Validation

---

# Anti-Pattern 5: No Rollback Plan

Bad:

    Production
       |
       ↓
    Helm Upgrade
       |
       ↓
      FAIL
       |
       ↓
    No Recovery Plan

Better:

    Production
       |
       ↓
    Helm Upgrade
       |
       ↓
      FAIL
       |
       ↓
    Helm Rollback
       |
       ↓
    Health Check

---

# Anti-Pattern 6: Excessive Template Complexity

Helm templates should remain understandable.

Avoid putting excessive application logic into templates.

Better:

    Simple Templates
        +
    Clear Values
        +
    Reusable Helpers

This makes maintenance easier.

---

# Helm Interview Questions

## Basic

1. What is Helm?
2. Why is Helm used with Kubernetes?
3. What is a Helm Chart?
4. What is Chart.yaml?
5. What is values.yaml?
6. What is a Helm template?
7. What is a Helm release?
8. What is `helm install`?
9. What is `helm upgrade`?
10. What is `helm rollback`?
11. What is `helm lint`?
12. What is `helm template`?
13. What is `helm package`?
14. What is `helm list`?
15. What is `helm status`?

---

# Intermediate Interview Questions

16. How do you deploy an application using Helm?

17. How do you manage DEV, QA, and PROD values?

18. How do you pass an image tag to Helm?

19. How do you perform a Helm rollback?

20. What is the difference between `helm install` and `helm upgrade`?

21. What does `helm upgrade --install` do?

22. What is the difference between `version` and `appVersion`?

23. How do you manage Helm dependencies?

24. How do you validate a Helm chart?

25. How do you troubleshoot Helm deployment failures?

26. How do you use Helm with GitHub Actions?

27. How do you use Helm with ArgoCD?

28. How do you manage secrets with Helm?

29. How do you deploy the same chart to multiple environments?

30. How do you implement Helm rollback?

---

# Advanced Interview Questions

31. Design a Helm-based CD pipeline for EKS.

32. How would you design reusable Helm charts for microservices?

33. How would you manage environment-specific values at enterprise scale?

34. How would you secure Helm deployments?

35. How would you integrate Helm with ArgoCD?

36. How would you implement Helm-based rollback?

37. How would you ensure the same image is promoted across environments?

38. How would you troubleshoot a Helm release that is stuck?

39. How would you design Helm chart versioning?

40. How would you manage chart dependencies securely?

---

# Real-World DevOps Example

Suppose we have a microservices platform:

    User
    Product
    Cart
    Orders
    Payment
    Inventory
    Notification

Each service has a Docker image.

Example:

    user:1.4.7
    product:1.4.7
    cart:1.4.7
    orders:1.4.7
    payment:1.4.7
    inventory:1.4.7
    notification:1.4.7

Helm can provide reusable deployment configuration for these services.

---

# Microservices Helm Architecture

    Helm
      |
      +-- user
      +-- product
      +-- cart
      +-- orders
      +-- payment
      +-- inventory
      +-- notification
      |
      ↓
    Kubernetes
      |
      +-- Deployments
      +-- Services
      +-- ConfigMaps
      +-- Ingress

Each service can have its own release or be managed through a larger application chart, depending on the architecture.

---

# Microservices Environment Promotion

Example:

    Build
      |
      ↓
    Scan
      |
      ↓
    ECR
      |
      ↓
    Helm DEV
      |
      ↓
    Test
      |
      ↓
    Helm QA
      |
      ↓
    Test
      |
      ↓
    Helm UAT
      |
      ↓
    Approval
      |
      ↓
    Helm PROD

The same image versions should be promoted according to the release strategy.

---

# Helm With EKS and ALB

A common AWS architecture is:

    Internet
        |
        ↓
      ALB
        |
        ↓
    Ingress
        |
        ↓
    Kubernetes Service
        |
        ↓
    Helm-managed Deployment
        |
        ↓
       Pods

Helm manages the Kubernetes resources, while AWS Load Balancer Controller can manage the AWS Application Load Balancer from Kubernetes configuration.

---

# Helm Deployment Mental Model

Remember:

    Chart
      |
      ↓
    Values
      |
      ↓
    Templates
      |
      ↓
    Rendered Kubernetes YAML
      |
      ↓
    Helm Release
      |
      ↓
    Kubernetes
      |
      ↓
    Pods
      |
      ↓
    Application

---

# Final Helm CD Mental Model

The complete CD process is:

    Developer
        |
        ↓
    GitHub
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Test
        +-- SonarQube
        +-- Trivy
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    Helm Chart
        |
        ↓
    Environment Values
        |
        ↓
    Helm Upgrade
        |
        ↓
    Kubernetes
        |
        ↓
    Deployment
        |
        ↓
    Rolling Update
        |
        ↓
    Readiness
        |
        ↓
    Service
        |
        ↓
    Ingress / ALB
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Production Application

---

# Final Concept

Helm provides a standardized way to package, configure, deploy, upgrade, and roll back Kubernetes applications.

The most important concepts to remember are:

    Chart
        ↓
    values.yaml
        ↓
    Templates
        ↓
    Rendered Kubernetes Manifests
        ↓
    Helm Release
        ↓
    Kubernetes

For multi-environment deployments:

    One Helm Chart
        |
        +-- DEV Values
        +-- QA Values
        +-- UAT Values
        +-- PROD Values
        |
        ↓
    Same Application Image
        |
        ↓
    Environment-Specific Deployment

For CI/CD:

    Build Once
        |
        ↓
    Scan
        |
        ↓
    Push Image
        |
        ↓
    Helm Deploy
        |
        ↓
    Validate
        |
        ↓
    Promote
        |
        ↓
    Rollback if Required

Helm is therefore an important component of Kubernetes-based CD pipelines, especially when applications need reusable deployment templates, environment-specific configuration, release management, upgrades, and reliable rollback capabilities.