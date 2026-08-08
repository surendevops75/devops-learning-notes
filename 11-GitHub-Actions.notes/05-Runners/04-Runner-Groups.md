# GitHub Actions Runner Groups

Runner groups allow organizations to organize self-hosted runners and control which repositories or workflows can access those runners.

They are especially useful in enterprise environments where runners are separated by:

- Environment
- Team
- Security boundary
- Network
- Business unit
- Workload type

Basic architecture:

```text
Organization
     |
     ├── CI Runner Group
     |
     ├── QA Runner Group
     |
     ├── UAT Runner Group
     |
     └── Production Runner Group
```

---

# Why Runner Groups?

Suppose an organization has:

```text
100 self-hosted runners
```

and wants:

```text
Development → Development runners
QA          → QA runners
UAT         → UAT runners
Production  → Production runners
```

Using only labels may not provide sufficient access governance.

Runner groups provide an additional access-control layer.

Conceptually:

```text
Repository
    |
    ↓
Runner Group
    |
    ↓
Eligible Runners
```

---

# Runner Group Architecture

```text
GitHub Organization
        |
        ↓
   Runner Groups
        |
   ┌────┼────┐
   ↓    ↓    ↓
  CI   QA   PROD
   |    |    |
   ↓    ↓    ↓
Runners Runners Runners
```

Each group can contain a set of self-hosted runners.

---

# Runner Groups vs Runner Labels

These are different concepts.

### Runner Labels

Labels answer:

```text
"What type of runner should execute this job?"
```

Example:

```text
linux
production
kubernetes
```

### Runner Groups

Groups answer:

```text
"Which runners are available to this repository/workflow?"
```

Example:

```text
Production Runner Group
```

A production workflow can use both.

---

# Example

Suppose:

```text
Production Runner Group

prod-01
  ├── self-hosted
  ├── linux
  └── production

prod-02
  ├── self-hosted
  ├── linux
  └── production
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

The runner group controls access to the runner pool.

The labels help select an appropriate runner within that pool.

---

# Runner Groups at Organization Level

A common enterprise design is:

```text
Organization
 |
 ├── Development
 │     ├── Runner 01
 │     └── Runner 02
 │
 ├── QA
 │     ├── Runner 01
 │     └── Runner 02
 │
 ├── UAT
 │     ├── Runner 01
 │     └── Runner 02
 │
 └── Production
       ├── Runner 01
       ├── Runner 02
       └── Runner 03
```

This separates runner pools by environment.

---

# Repository Access

Runner groups can be configured so that only selected repositories can use them.

Conceptually:

```text
Production Runner Group
       |
       ├── Repository A ✓
       ├── Repository B ✓
       ├── Repository C ✗
       └── Repository D ✗
```

This prevents every repository in the organization from automatically using sensitive production runners.

---

# Why This Matters

Imagine:

```text
Production Runner
       |
       ↓
AWS Production Account
       |
       ↓
EKS
```

If every repository can use that runner:

```text
Any Repository
       |
       ↓
Production Runner
       |
       ↓
Production Infrastructure
```

the blast radius becomes much larger.

Runner groups help restrict which repositories can access the runner pool.

---

# Production Runner Group

A production runner group could be:

```text
production-runners
```

Containing:

```text
prod-runner-01
prod-runner-02
prod-runner-03
```

The runners may have labels:

```text
self-hosted
linux
production
kubernetes
```

---

# Production Workflow

Conceptually:

```text
Protected Repository
        |
        ↓
Production Workflow
        |
        ↓
Production Runner Group
        |
        ↓
Production Runner
        |
        ↓
EKS
```

Additional controls should still be used.

---

# Runner Group Is Not the Same as Environment

Runner group:

```text
Controls runner access
```

Environment:

```text
Controls deployment environment protection
```

Example:

```text
Runner Group
      |
      ↓
Production Runner
      |
      ↓
Production Environment
      |
      ↓
Required Approval
      |
      ↓
Deployment
```

They provide different controls.

---

# Runner Group + Environment

Example:

```yaml
jobs:

  deploy:

    runs-on:
      - self-hosted
      - linux
      - production

    environment:
      name: production

    steps:

      - name: Deploy
        run: ./deploy.sh
```

Conceptually:

```text
Runner Group
    |
    ↓
Eligible Production Runner
    |
    ↓
Production Environment
    |
    ↓
Deployment Protection
```

---

# Runner Group + Labels

A strong production setup can use:

```text
Runner Group:
production-runners

Labels:
self-hosted
linux
production
kubernetes
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
  - kubernetes
```

This combines:

```text
Access control
+
Runner selection
```

---

# Runner Group + Branch Protection

A production deployment should generally come from a trusted workflow or protected branch.

Example:

```text
Feature Branch
      |
      ↓
Pull Request
      |
      ↓
Review
      |
      ↓
Protected Main Branch
      |
      ↓
Production Workflow
      |
      ↓
Production Runner Group
```

This reduces the risk of arbitrary branch code reaching privileged infrastructure.

---

# Runner Group + Pull Requests

Be especially careful with:

```text
pull_request
```

workflows.

A malicious pull request can potentially modify workflow-controlled code and execute commands.

Do not make sensitive production runner groups broadly accessible to untrusted pull request workflows.

---

# Security Boundary

A production runner group should be treated as a security boundary.

Example:

```text
Production Runner Group
       |
       ├── Private Network
       ├── Production IAM
       ├── Kubernetes Access
       └── Deployment Tools
```

Access should be limited.

---

# Principle of Least Privilege

Only give repositories access to runner groups they actually need.

Example:

```text
Repository A
   |
   └── CI Runner Group

Repository B
   |
   └── QA Runner Group

Production Repository
   |
   └── Production Runner Group
```

Avoid:

```text
All repositories
      |
      ↓
All runner groups
```

---

# Runner Groups for Teams

Runner groups can also separate teams.

Example:

```text
Organization
 |
 ├── Platform Team
 │      └── Platform Runners
 │
 ├── Application Team
 │      └── Application Runners
 │
 └── Security Team
        └── Security Runners
```

This can be useful when teams have different infrastructure or tooling requirements.

---

# Runner Groups for Network Zones

Another design:

```text
Public CI
   |
   └── CI Runner Group

Private Network
   |
   └── Private Runner Group

Production Network
   |
   └── Production Runner Group
```

This provides clearer network separation.

---

# Runner Groups for Workload Types

Example:

```text
General CI
   |
   └── general-runners

Docker Builds
   |
   └── docker-runners

GPU Workloads
   |
   └── gpu-runners

Production Deployment
   |
   └── production-runners
```

This can prevent specialized runners from being consumed by unrelated jobs.

---

# Runner Group Naming

Use clear names.

Examples:

```text
ci-runners
qa-runners
uat-runners
production-runners
platform-runners
security-runners
gpu-runners
```

Avoid ambiguous names such as:

```text
group1
group2
runnerpool
test123
```

---

# Naming Strategy

A useful convention:

```text
<environment>-<purpose>-runners
```

Examples:

```text
qa-ci-runners
uat-deployment-runners
production-deployment-runners
security-scan-runners
```

Keep naming consistent across the organization.

---

# Runner Group Inventory

Maintain documentation.

Example:

| Runner Group | Purpose | Environment | Access |
|---|---|---|---|
| ci-runners | Build/Test | CI | Application repos |
| qa-runners | QA deployment | QA | QA repos |
| uat-runners | UAT deployment | UAT | Release repos |
| production-runners | Production deployment | Production | Approved repos |

This makes governance easier.

---

# Production Runner Group Example

```text
production-runners
       |
       ├── prod-runner-01
       ├── prod-runner-02
       └── prod-runner-03

Labels:
       |
       ├── self-hosted
       ├── linux
       ├── production
       └── kubernetes
```

Workflow:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
  - kubernetes
```

---

# Runner Group and EKS

A production runner group may contain runners with network access to EKS.

```text
Production Runner Group
          |
          ↓
Production Runner
          |
          ↓
Private VPC
          |
          ↓
EKS
```

The runner's IAM and Kubernetes RBAC permissions should be limited to the required resources.

---

# Runner Group and AWS Accounts

A large organization may separate runners by AWS account.

Example:

```text
AWS Development Account
        |
        └── Dev Runner Group

AWS QA Account
        |
        └── QA Runner Group

AWS UAT Account
        |
        └── UAT Runner Group

AWS Production Account
        |
        └── Production Runner Group
```

This creates a strong infrastructure boundary.

---

# Multi-Account Architecture

```text
                 GitHub
                    |
          ┌─────────┼─────────┐
          ↓         ↓         ↓
       Dev Group  QA Group  Prod Group
          |         |          |
          ↓         ↓          ↓
       Dev AWS    QA AWS     Prod AWS
```

Each runner group can have different IAM permissions and network access.

---

# Runner Groups and IAM

Runner groups control:

```text
Who can use the runner
```

IAM controls:

```text
What the runner can access in AWS
```

Example:

```text
Repository
     |
     ↓
Runner Group
     |
     ↓
Runner
     |
     ↓
IAM Role
     |
     ↓
AWS Resources
```

All layers should follow least privilege.

---

# Runner Groups and Kubernetes RBAC

Similarly:

```text
Repository
     |
     ↓
Runner Group
     |
     ↓
Runner
     |
     ↓
Kubernetes Identity
     |
     ↓
RBAC
     |
     ↓
Namespace / Resources
```

Do not rely on runner groups alone.

---

# Runner Group and GitOps

If using ArgoCD:

```text
CI Runner Group
       |
       ↓
Build
       |
       ↓
Security Scan
       |
       ↓
Push Image
       |
       ↓
Update Git
       |
       ↓
ArgoCD
       |
       ↓
EKS
```

The CI runner may not require direct production cluster access.

This can simplify production runner privileges.

---

# Direct Deployment vs GitOps

### Direct deployment

```text
Production Runner
       |
       ↓
kubectl / Helm
       |
       ↓
EKS
```

Requires:

```text
Kubernetes connectivity
Kubernetes credentials
RBAC
```

### GitOps

```text
CI Runner
   |
   ↓
Git
   |
   ↓
ArgoCD
   |
   ↓
EKS
```

The CI runner can avoid direct cluster access.

---

# Runner Group Access Governance

For production runner groups, define:

```text
Approved repositories
Approved teams
Approved workflows
Approved environments
Approved administrators
```

Review access regularly.

---

# Runner Group Lifecycle

A runner group should have a lifecycle.

```text
Create Group
     |
     ↓
Add Runners
     |
     ↓
Configure Access
     |
     ↓
Assign Labels
     |
     ↓
Use in Workflows
     |
     ↓
Monitor
     |
     ↓
Audit
     |
     ↓
Remove / Replace
```

---

# Adding a New Production Runner

Example process:

```text
Provision VM
    |
    ↓
Apply Security Baseline
    |
    ↓
Install Required Tools
    |
    ↓
Install Runner
    |
    ↓
Register with Production Group
    |
    ↓
Apply Labels
    |
    ↓
Validate Connectivity
    |
    ↓
Enable
```

Do not manually add production runners without the required security checks.

---

# Removing a Runner

When a runner is retired:

```text
Drain / Stop Workloads
       |
       ↓
Remove Runner
       |
       ↓
Revoke Credentials
       |
       ↓
Remove Infrastructure
       |
       ↓
Update Inventory
```

Especially important for production runners.

---

# Runner Group Capacity

Suppose:

```text
Production Runner Group

Runner 01
Runner 02
Runner 03
```

and 20 production jobs arrive.

Some jobs may queue depending on available capacity and concurrency controls.

Scale the runner pool when required.

---

# Runner Group Scaling

```text
Production Runner Group
        |
        ├── Runner 01
        ├── Runner 02
        ├── Runner 03
        ├── Runner 04
        └── Runner 05
```

For dynamic scaling, consider an automated runner-management architecture.

Actions Runner Controller is covered in:

```text
05-Actions-Runner-Controller.md
```

---

# Runner Group Monitoring

Monitor:

```text
Runner online status
Job queue
Job duration
Runner utilization
Failure rate
Disk
CPU
Memory
Security events
```

For production runner groups, monitoring should be part of normal platform operations.

---

# Runner Group Failure

Suppose:

```text
Production Runner Group

Runner 01 → Offline
Runner 02 → Offline
Runner 03 → Offline
```

Then:

```text
Production Deployment
        |
        ↓
No Available Runner
        |
        ↓
Job Queued
```

Troubleshooting:

```text
Runner health
Network
Cloud infrastructure
Runner service
Group access
Labels
Capacity
```

---

# Runner Group vs Concurrency

These solve different problems.

### Runner Group

Controls:

```text
Which runners can execute the job
```

### Concurrency

Controls:

```text
How many workflow/job executions can run simultaneously
```

Example:

```yaml
concurrency:
  group: production
  cancel-in-progress: false
```

This can help prevent overlapping production deployments.

---

# Runner Group + Concurrency

Production architecture:

```text
Production Workflow
       |
       ↓
Concurrency Control
       |
       ↓
Production Runner Group
       |
       ↓
Production Runner
       |
       ↓
Production
```

This can help avoid two deployments modifying production simultaneously.

---

# Runner Group + Approval

A production deployment can use:

```text
1. Protected branch
2. Workflow validation
3. Runner group
4. Environment
5. Required reviewer
6. Deployment
```

Example:

```text
Main
 |
 ↓
Validation
 |
 ↓
Production Runner Group
 |
 ↓
Production Environment
 |
 ↓
Approval
 |
 ↓
Deploy
```

---

# Change Request Process

For enterprise deployments:

```text
JIRA Change Request
        |
        ↓
Required Approvals
        |
        ↓
Workflow Dispatch
        |
        ↓
Production Runner Group
        |
        ↓
Deployment
```

A runner group controls runner access; the change-management process should validate the business approval separately.

---

# Production Deployment Example

```yaml
name: Production Deploy

on:
  workflow_dispatch:
    inputs:
      jira_ticket:
        description: "Approved JIRA change request"
        required: true
        type: string

permissions:
  contents: read

concurrency:
  group: production
  cancel-in-progress: false

jobs:

  deploy:

    runs-on:
      - self-hosted
      - linux
      - production
      - kubernetes

    environment:
      name: production

    timeout-minutes: 30

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Validate Change Request
        run: |
          echo "Validating: ${{ inputs.jira_ticket }}"
          ./scripts/validate-change-request.sh \
            "${{ inputs.jira_ticket }}"

      - name: Deploy
        run: |
          helm upgrade --install catalogue ./helm/catalogue \
            --namespace catalogue \
            --create-namespace

      - name: Verify
        run: |
          kubectl rollout status \
            deployment/catalogue \
            -n catalogue \
            --timeout=5m
```

Production controls:

```text
workflow_dispatch
       |
       ↓
Change Request
       |
       ↓
Concurrency
       |
       ↓
Production Runner Group
       |
       ↓
Production Environment
       |
       ↓
Deployment
```

---

# Secure Production Runner Group

Recommended architecture:

```text
                         GitHub
                            |
                            ↓
                    Protected Repository
                            |
                            ↓
                     Production Workflow
                            |
                ┌───────────┴───────────┐
                ↓                       ↓
        Environment Rules        Runner Group
                |                       |
                ↓                       ↓
          Approval / Rules       Production Runners
                                        |
                                        ↓
                                  Private Network
                                        |
                                ┌───────┴───────┐
                                ↓               ↓
                               EKS             AWS
```

---

# Untrusted Code Protection

Bad:

```text
External PR
    |
    ↓
Production Runner Group
    |
    ↓
Production AWS
```

Better:

```text
External PR
    |
    ↓
GitHub-Hosted Runner
    |
    ↓
Build + Test
    |
    ↓
Review + Merge
    |
    ↓
Protected Production Workflow
    |
    ↓
Production Runner Group
```

---

# Runner Group Access Review

Periodically review:

```text
Which repositories can use production runners?

Which teams can modify workflows?

Which runners belong to production?

Which IAM roles can runners assume?

Which Kubernetes namespaces can runners access?
```

This helps detect excessive access.

---

# Runner Group Audit

Maintain an inventory:

```text
Group
   |
   ├── Repositories
   ├── Runners
   ├── Labels
   ├── Administrators
   ├── Network
   └── Permissions
```

Review changes regularly.

---

# Common Runner Group Mistake

Using one group:

```text
all-runners
```

for:

```text
Dev
QA
UAT
Production
```

This weakens separation.

Better:

```text
dev-runners
qa-runners
uat-runners
production-runners
```

---

# Common Runner Group Mistake

Giving every repository access to:

```text
production-runners
```

Even if most repositories do not deploy to production.

Better:

```text
Approved production repositories
        |
        ↓
production-runners
```

---

# Common Runner Group Mistake

Thinking:

```text
production-runners
```

automatically protects production.

It does not.

You still need:

```text
Branch protection
Workflow security
Environment protection
IAM
Kubernetes RBAC
Network controls
```

---

# Common Runner Group Mistake

Using labels without access controls.

Example:

```yaml
runs-on:
  - self-hosted
  - production
```

The label alone does not guarantee that only trusted workflows can access production infrastructure.

Use runner groups and appropriate workflow/environment controls.

---

# Common Runner Group Mistake

Putting CI and production deployment runners into the same pool.

Bad:

```text
General CI
    |
    ↓
Production Runner
```

Better:

```text
CI
 |
 └── CI Runner Group

Production
 |
 └── Production Runner Group
```

---

# Best Practices

- Separate runner groups by security boundary where appropriate.
- Create dedicated production runner groups.
- Restrict repository access to sensitive groups.
- Combine groups with runner labels.
- Use environment protection for production.
- Use least-privilege IAM.
- Use Kubernetes RBAC.
- Protect production workflows from untrusted code.
- Review runner group membership regularly.
- Maintain a runner inventory.
- Monitor runner capacity.
- Automate runner provisioning and replacement.
- Use ephemeral runners for sensitive workloads where appropriate.
- Keep CI and production deployment runners separated.
- Consider GitOps to reduce direct production runner privileges.
- Use concurrency controls for production deployments.

---

# Common Mistakes

- Using one runner group for everything.
- Giving every repository access to production runners.
- Treating runner groups as the only security control.
- Mixing CI and production runners.
- Allowing untrusted PRs to use privileged runner groups.
- Not reviewing group membership.
- Not monitoring runner capacity.
- Using unclear group names.
- Allowing stale runners to remain in production groups.
- Giving production runners excessive cloud or Kubernetes permissions.

---

# Summary

Runner groups organize self-hosted runners and provide an important access-control layer.

Conceptually:

```text
Organization
     |
     ↓
Runner Group
     |
     ↓
Eligible Runners
```

Runner labels help select the right runner:

```yaml
runs-on:
  - self-hosted
  - linux
  - production
```

Runner groups help control which repositories/workflows can access sensitive runner pools.

A strong production design is:

```text
Protected Repository
       |
       ↓
Production Workflow
       |
       ├── Environment Protection
       |
       ├── Runner Group
       |
       ├── Runner Labels
       |
       ├── IAM
       |
       └── Kubernetes RBAC
               |
               ↓
          Production
```

Remember:

```text
Runner Group
→ Access to runner pool

Runner Label
→ Runner selection

Environment
→ Deployment protection

IAM / RBAC
→ Resource authorization
```

The key principle is:

```text
Use runner groups to create clear trust boundaries,
especially around production infrastructure.
```

---

# Interview Questions

## Basic

1. What is a runner group?
2. Why are runner groups useful?
3. What is the difference between runner groups and runner labels?
4. Can runner groups contain multiple runners?
5. Why would you create a production runner group?
6. How do runner groups help organizations with many repositories?

## Intermediate

7. How would you organize runner groups for Dev, QA, UAT, and Production?
8. How do runner groups improve security?
9. How would you restrict production runners to specific repositories?
10. How do runner groups work with runner labels?
11. What is the difference between a runner group and a GitHub environment?
12. How would you troubleshoot a workflow that cannot access a runner group?
13. How would you manage runner group capacity?
14. Why should CI and production deployment runners be separated?

## Advanced / Production

15. Design a production runner architecture using runner groups, labels, environments, IAM, and Kubernetes RBAC.
16. A repository that should not deploy to production has access to the production runner group. How would you investigate and fix it?
17. How would you prevent untrusted pull requests from using a privileged production runner group?
18. Design runner groups for an organization with multiple AWS accounts and separate Dev, QA, UAT, and Production infrastructure.
19. Explain how runner groups and labels work together when selecting a runner.
20. Why are runner groups not sufficient as the only production security control?
21. How would you combine runner groups with GitHub environments and required approvals?
22. Design a production deployment workflow using a JIRA change request, protected branch, runner group, environment approval, and Kubernetes deployment.
23. How can GitOps with ArgoCD reduce the privileges required by a production runner group?
24. How would you audit production runner group membership?
25. How would you design runner groups for different teams such as Platform, Security, and Application teams?
26. A production runner group has three runners and all are offline. Walk through your troubleshooting process.
27. How would you prevent stale or compromised runners from remaining in a production runner group?
28. Explain the difference between runner-group access, runner selection, environment protection, IAM, and Kubernetes RBAC.
29. Design a secure enterprise runner architecture where CI uses general runners and production deployment uses isolated production runners.
30. How would you use Actions Runner Controller with runner groups for dynamic production runner capacity?