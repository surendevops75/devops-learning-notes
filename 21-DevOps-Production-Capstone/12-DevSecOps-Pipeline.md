# DevSecOps Pipeline

> Deep production-oriented security notes for GitLab CI, AWS, ECR, EKS,
> Docker, Kubernetes, GitOps, Argo CD, and software supply-chain
> protection.

## Chapter Objective

This chapter builds the security layer around the CI/CD architecture.
The goal is not merely to list scanners. The goal is to design a
production delivery system where identity, source, dependencies, build
infrastructure, artifacts, deployment state, and runtime are protected
by independent controls.

## 1. DevSecOps Objective

DevSecOps integrates security into the software delivery lifecycle
rather than treating security as a final manual gate. In this capstone,
every important boundary is protected: source code, dependencies, CI
runners, build inputs, container images, registries, GitOps changes,
Kubernetes admission, runtime secrets, and production access. The
objective is not to produce zero scanner findings; it is to make
security risk visible, actionable, owned, and difficult to bypass.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 2. Security Pipeline Reference

A production sequence is: merge request -\> secret detection -\> SAST
-\> dependency/SCA analysis -\> IaC and Kubernetes policy checks -\>
tests -\> container build -\> SBOM -\> image vulnerability scan -\>
provenance/signing -\> registry publication -\> GitOps update -\>
admission verification -\> runtime monitoring. Jobs that do not require
credentials should run without cloud credentials. Protected jobs receive
only short-lived credentials for their exact purpose.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 3. Shift Left Without Shift Alone

Shift-left security means detecting defects early, but it does not mean
moving every security responsibility into the developer workstation.
Source scanning catches source-level issues, image scanning catches
image composition issues, admission catches deployment-policy
violations, and runtime controls catch behavior that static checks
cannot predict. Defense in depth is essential.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 4. Threat Model

Threats include malicious source changes, compromised developer
credentials, malicious dependencies, typosquatted packages, poisoned
build caches, compromised runners, leaked CI variables, vulnerable base
images, unauthorized ECR pushes, GitOps tampering, stolen cluster
credentials, and insecure Kubernetes workloads. Map each threat to
preventive, detective, and recovery controls.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 5. Security Ownership

Developers own secure application code and dependency hygiene. Platform
teams own runner security, CI templates, identity federation,
registries, and cluster controls. Security teams define policy,
vulnerability standards, and exception governance. Application owners
own remediation of vulnerabilities in their services. Shared ownership
must still have a named accountable party.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 6. SAST Fundamentals

Static Application Security Testing analyzes source without executing
the application. It can identify injection patterns, insecure
deserialization, path traversal, command execution, weak cryptography,
and dangerous API use. SAST is strongest when tuned to the application's
language and framework and when findings are triaged rather than blindly
treated as equal.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 7. SAST Pipeline Design

Run fast SAST checks on merge requests and deeper analysis on protected
branches or scheduled pipelines. Cache analyzers safely. Publish reports
as pipeline artifacts. Block according to policy-defined severities and
confidence rather than arbitrary scanner defaults.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 8. False Positives

Security scanners can report code that is technically suspicious but
safe in its actual context. Suppressions should be narrow, documented,
reviewed, and preferably tied to a rule and location. Avoid
repository-wide disabling of security rules merely to remove noise.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 9. SAST Remediation

For each finding, identify the vulnerable data flow, root cause,
exploitability, and correct fix. Prefer eliminating the dangerous
pattern over adding superficial filters. Add regression tests when
practical so the vulnerability does not return.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 10. Software Composition Analysis

SCA analyzes third-party dependencies and their known vulnerabilities.
Modern applications can contain hundreds or thousands of transitive
packages, so manual inventory is unreliable. Lock files, dependency
manifests, SBOMs, and vulnerability databases provide complementary
visibility.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 11. Direct vs Transitive Dependencies

A direct dependency is intentionally declared by the application. A
transitive dependency is introduced by another package. Remediation may
therefore require upgrading a parent package rather than directly
changing the vulnerable component. Do not assume a vulnerable package is
unused solely because it is transitive.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 12. Dependency Pinning

Use lock files and controlled version ranges. Pinning improves
reproducibility but must be combined with an update strategy. Completely
frozen dependencies can become a security liability if teams never
refresh them.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 13. Automated Dependency Updates

Automated update pull requests can continuously propose safe dependency
upgrades. They should execute the same test and security pipeline as
normal code changes. Never auto-merge dependency updates merely because
a version is newer.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 14. Known Vulnerability Policy

Define thresholds such as blocking critical vulnerabilities with a known
exploitable path, while allowing lower-risk findings to proceed with
tracked remediation. The exact threshold should reflect business risk,
exposure, exploitability, compensating controls, and regulatory
requirements.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 15. Vulnerability Prioritization

Severity alone is insufficient. Consider internet exposure, exploit
availability, active exploitation, asset criticality, affected component
reachability, privilege required, and whether the vulnerable code path
is actually used. A medium vulnerability in an internet-facing
authentication service may deserve more attention than a high finding in
unreachable test code.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 16. Secret Detection

Secret detection searches for API keys, tokens, private keys, passwords,
cloud credentials, and other high-entropy or known credential patterns.
Scan commits and the working tree. Prevention is better than cleanup:
use pre-commit controls where useful and server-side scanning as the
authoritative boundary.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 17. Secret Leak Response

If a real secret is exposed, assume compromise. Revoke or rotate it,
identify where it was used, inspect audit logs, remove the secret from
active source and history where required, and replace the credential
mechanism. Deleting the text from the latest commit does not make an
exposed credential safe.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 18. AWS OIDC

For GitLab-to-AWS authentication, use OIDC federation when supported.
The CI job obtains an identity token and exchanges it with AWS STS for
temporary credentials. The IAM trust policy should constrain issuer,
audience, project, branch or tag, and protected environment according to
the provider's token claims.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 19. IAM Trust Policy

The trust policy answers which external CI identities may assume the
role. Keep it narrowly scoped. Do not trust an entire organization or
all branches when only one protected release path requires access.
Validate the exact claims provided by the CI identity provider before
implementing the policy.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 20. IAM Permissions

The role permission policy answers what the assumed identity can do.
Separate build, publish, infrastructure, and deployment
responsibilities. A role that only pushes to one ECR repository should
not have unrestricted ECR or account-level permissions.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 21. Temporary Credentials

Temporary STS credentials reduce the lifetime of compromised credentials
and provide clearer audit events. They are still sensitive during their
lifetime, so do not print them, write them to artifacts, or expose them
to untrusted child processes.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 22. CI Variable Protection

Protected variables should be available only to protected branches or
environments. Mask values in logs where the platform supports masking.
Do not place secrets in command-line arguments if the runner may expose
process arguments or debug output.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 23. Runner Threat Model

A CI runner executes attacker-controlled code by design. Treat the
runner as a privileged security boundary. Use isolated runners for
trusted release jobs, restrict privileged Docker access, patch runner
hosts, control network egress, and prefer ephemeral execution where
practical.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 24. Docker Socket Risk

Mounting /var/run/docker.sock gives a container powerful control over
the host Docker daemon. A malicious build can potentially escape the
intended isolation boundary. Prefer daemonless or rootless build
approaches where feasible and isolate any runner that must use
privileged container builds.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 25. Build Isolation

Build environments should not contain production credentials. A build
should receive only the inputs necessary to compile and package the
application. Use ephemeral workspaces and clean sensitive temporary data
after execution.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 26. Dependency Supply Chain

Public package registries introduce upstream trust. Risks include
compromised maintainers, malicious packages, typosquatting, dependency
confusion, and abandoned packages. Approved registries, dependency
proxies, lock files, checksums, and review processes reduce exposure.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 27. Dependency Confusion

Dependency confusion occurs when a package manager resolves an
attacker-controlled public package instead of an intended private
package. Configure package-manager scopes and registries explicitly. Do
not rely on package-name uniqueness in a public namespace.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 28. Typosquatting

Attackers can publish packages with names similar to popular
dependencies. Review dependency changes carefully and use lock files,
trusted registries, and automated package reputation or policy controls
where available.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 29. Malicious Build Script

Package installation may execute lifecycle scripts. A compromised
dependency can therefore execute code inside CI. Keep CI credentials
minimal, isolate runners, restrict egress, and avoid giving build
processes unnecessary access to cloud or production systems.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 30. Infrastructure-as-Code Scanning

Scan Terraform and related infrastructure code for insecure cloud
configurations, public exposure, overly permissive IAM, unencrypted
storage, and weak network controls. IaC scanning complements Terraform
validation: valid syntax does not mean secure infrastructure.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 31. Terraform Security Gate

Run formatting and validation first, then security policy checks and
plan analysis. A production apply should require protected credentials
and appropriate approval. Never make security checks optional simply
because infrastructure changes are urgent.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 32. Kubernetes Manifest Scanning

Validate Kubernetes manifests for privileged containers, host
networking, hostPath use, dangerous capabilities, missing resource
limits, weak security contexts, and unsafe image references. Render Helm
charts before scanning so the scanner sees the effective manifests.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 33. Helm Security

Scan both chart source and rendered manifests. Check templates for
unsafe defaults, verify values files, avoid embedding credentials, and
make secure defaults the chart baseline. A secure template can still
become insecure if values override its controls.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 34. Pod Security

Use Kubernetes pod security standards or equivalent admission policy to
enforce restricted workload behavior where compatible. Controls should
cover privileged mode, host namespaces, capabilities, filesystem
permissions, and other high-risk settings.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 35. Network Security

Use Kubernetes NetworkPolicies to restrict pod-to-pod communication
where supported. Default-deny designs can reduce lateral movement. Then
explicitly permit required application, DNS, database, messaging, and
monitoring traffic.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 36. Image Provenance

Knowing an image digest identifies its contents, while provenance
explains how it was produced. Capture source commit, builder identity,
build process, dependencies, and timestamps. Provenance helps detect
artifacts produced outside the approved build system.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 37. SBOM Fundamentals

An SBOM inventories software components in an artifact. CycloneDX and
SPDX are common formats. Generate SBOMs during CI and associate them
with the exact image digest so vulnerability analysis remains tied to a
specific artifact.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 38. SBOM Lifecycle

Generate the SBOM at build time, store it in an approved artifact or
security system, and retain the relationship to the image digest. During
an incident, query the inventory to determine which deployed services
contain a vulnerable package.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 39. Container Image Scanning

Scan operating-system packages and application dependencies inside the
final runtime image. Scanning only the source repository misses
vulnerable base-image packages. Scanning only an intermediate build
stage can produce findings that are not present in the production image.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 40. Minimal Images

Smaller runtime images reduce attack surface, vulnerability inventory,
startup footprint, and unnecessary tools. Multi-stage builds can keep
compilers, package managers, shells, and build-only dependencies out of
production images.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 41. Non-Root Images

Run the application as a non-root user when possible. Set an explicit
USER, use appropriate filesystem permissions, and validate that the
application does not need privileged operations. Non-root execution
limits the impact of a container compromise.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 42. Image Signing

Sign the immutable image digest using an approved signing system. The
signature should identify the trusted build identity. Deployment
admission can then require signatures from approved producers before
allowing workloads into protected namespaces.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 43. Verification

Image verification must happen at a deployment boundary that cannot be
bypassed by changing a CI job alone. Admission policy can enforce that
only approved registries and signed artifacts are accepted.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 44. Key Management

Signing keys are high-value credentials. Prefer managed or
hardware-backed key protection where appropriate, restrict who can sign,
rotate keys under a defined process, and plan for key compromise and
trust-root updates.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 45. Container Registry Security

Restrict ECR repository permissions, enable appropriate scanning,
encrypt repositories, apply lifecycle policies, and audit pushes.
Production deployment identities should not have permission to rewrite
arbitrary repositories.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 46. ECR Repository Policy

Use repository policies only when cross-account or service-specific
access is required. Keep them narrow. Combine IAM identity policies with
resource policies carefully so the effective permission is understood.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 47. Cross-Account ECR

In multi-account architectures, build artifacts can be published into a
central registry account and consumed by workload accounts, or each
account can maintain its own registry. Cross-account access should be
explicit and least privileged. Do not solve account separation by
granting broad organization-wide ECR access.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 48. Artifact Immutability

Prefer digest-based deployment. If tag immutability is enabled in the
registry, use unique release tags and never depend on a moving latest
tag for production. Digest references provide the strongest guarantee of
exact content.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 49. GitOps Security

The GitOps repository is production control-plane data. Protect
branches, require reviews, validate manifests, restrict write access,
and audit changes. CI should update only the intended environment path
and should not have broad repository administration rights.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 50. GitOps Commit Security

A GitOps update should contain the service, environment, and new
immutable digest. Automated commits should be recognizable and
attributable to a controlled CI identity. Avoid scripts that rewrite
unrelated files, because accidental changes increase deployment risk.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 51. Argo CD Security Boundary

Argo CD should have only the Kubernetes permissions required for its
managed applications. Separate projects and namespaces where
appropriate. Do not expose the Argo CD API broadly, and protect
administrative access with strong authentication and authorization.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 52. Cluster Admission

Admission control is the last preventive gate before workload creation.
It can require trusted registries, signed images, restricted security
contexts, resource limits, and approved namespaces. This protects the
cluster even if a CI pipeline is accidentally misconfigured.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 53. Policy as Code

Represent security policies as code so changes are reviewable, testable,
and reproducible. Policy engines can validate Kubernetes manifests and
infrastructure configuration before deployment and at admission time.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 54. Security Gate Design

A security gate should have a defined input, rule, severity threshold,
output, owner, and exception path. Avoid gates that fail unpredictably
or provide no actionable report. Developers should understand why the
pipeline stopped and how to remediate it.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 55. Blocking vs Reporting

Not every security finding should block every pipeline. Block
high-confidence, high-impact issues at the appropriate boundary; report
lower-risk findings for remediation. Production-critical controls should
not be reduced to warnings simply to improve deployment frequency.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 56. Exception Management

A security exception must identify the finding, affected asset, business
justification, risk owner, compensating control, approval, remediation
plan, and expiration date. Exceptions should expire automatically or
require deliberate renewal.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 57. Emergency Exception

During a critical production incident, an emergency exception may be
necessary. It should be time-bound, approved by the correct authority,
recorded in the incident timeline, and followed by normal remediation.
Emergency does not mean invisible.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 58. Security Metrics

Useful metrics include critical vulnerabilities by age, mean time to
remediate, leaked-secret incidents, percentage of signed images,
percentage of workloads using approved base images, scanner coverage,
policy violations, and exception age.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 59. Security Logs

Centralize security-relevant CI and cloud audit events. Useful evidence
includes IAM role assumptions, ECR pushes, Git changes, Argo CD syncs,
Kubernetes admission decisions, and privileged administrative actions.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 60. Audit Trail

A complete release trail should connect developer change -\> commit -\>
pipeline -\> security reports -\> SBOM -\> image digest -\> registry -\>
GitOps commit -\> Argo CD sync -\> Kubernetes workload. This chain is
valuable for audits and incident response.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 61. Security Alerts

Alert on high-value events such as leaked secrets, unexpected privileged
role assumptions, unauthorized registry activity, failed admission
policies in protected environments, and suspicious GitOps changes. Avoid
alerting on every low-risk scanner finding or responders will ignore the
system.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 62. Runtime Security

CI security cannot guarantee runtime safety. Runtime controls include
least-privilege service accounts, network policies, read-only
filesystems where possible, resource limits, pod security, cloud IAM
boundaries, audit logging, and runtime threat detection.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 63. IRSA and Pod Identity

Workloads should obtain AWS permissions through workload identity
mechanisms rather than static credentials stored in Kubernetes Secrets.
Scope each service account to the AWS actions and resources it requires.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 64. Secrets Management

Do not bake secrets into images or Git. Use an approved secret manager
and integrate workloads through a controlled mechanism such as external
secret synchronization or CSI-based retrieval where appropriate. Runtime
access should be short-lived and least privileged when possible.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 65. Secret Rotation

Rotation should be designed before an incident. Know which services
consume a secret, how a new value is distributed, whether pods reload it
automatically, and how old credentials are revoked. Test rotation
regularly.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 66. Database Security

CI should never use unrestricted production database credentials for
ordinary tests. Use isolated databases, ephemeral environments,
synthetic data, and restricted test accounts. Production migrations
require separate approval and operational safeguards.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 67. Security Testing Environments

Keep security testing isolated from production. Dynamic tests should
target controlled environments. Do not point aggressive scanners or
exploit-validation tools at production without explicit authorization
and carefully defined scope.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 68. DAST

Dynamic Application Security Testing evaluates a running application. It
can identify issues that static analysis cannot, such as authentication
and authorization weaknesses, insecure headers, and runtime input
handling. Run it against a dedicated environment with synthetic data.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 69. API Security Testing

API-focused security tests should validate authentication,
authorization, input validation, rate limits, and object-level access
control. Test both successful and deliberately invalid requests while
avoiding real customer data.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 70. Container Runtime Controls

Use read-only root filesystems when feasible, drop unnecessary Linux
capabilities, disable privilege escalation where compatible, use seccomp
profiles, and apply resource controls. These controls reduce the impact
of application compromise.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 71. Network Segmentation

Separate CI, shared services, workloads, management, and data-plane
networks according to organizational architecture. Restrict east-west
and north-south flows rather than assuming that cluster membership
itself provides sufficient isolation.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 72. Egress Monitoring

Monitor outbound traffic from CI and production workloads where
appropriate. Unexpected connections can reveal compromised dependencies,
data exfiltration, command-and-control activity, or configuration
mistakes.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 73. Supply Chain Incident

If a dependency or base image is compromised, identify affected
artifacts using SBOM and registry metadata, stop promotion, rebuild from
trusted inputs, rotate potentially exposed credentials, and deploy a
verified replacement digest. Preserve evidence before deleting
compromised artifacts when incident policy requires it.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 74. Compromised Runner

Treat a suspected runner compromise as a credential and supply-chain
incident. Disable or isolate the runner, invalidate temporary or exposed
credentials as appropriate, inspect job history, rebuild the runner from
a trusted image, and verify artifacts produced during the exposure
window.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 75. Compromised GitOps Repository

If unauthorized production desired-state changes appear, preserve Git
history and audit evidence, revoke affected credentials, restore a
known-good commit, verify Argo CD and cluster state, and investigate the
access path before reopening automated promotion.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 76. Compromised Image

If an image is suspected to be malicious, identify all environments
using its digest, stop further promotion, isolate affected workloads if
necessary, deploy a trusted digest, and investigate build and registry
logs. Do not rely only on deleting the tag because deployed nodes may
still have the image.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 77. CVE Response

When a high-impact CVE is announced, query the SBOM inventory for
affected components, identify running image digests, determine
exploitability and exposure, patch or rebuild, scan the replacement
artifact, and promote it through the normal controlled path.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 78. Zero-Day Response

A zero-day requires rapid risk assessment. If exploitation is active,
apply temporary compensating controls such as WAF rules, network
restrictions, feature disablement, or workload isolation while building
and testing a permanent fix. Document every emergency control.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 79. Security in Multi-Cluster

Apply the same admission, image trust, identity, and GitOps policies
across clusters. Centralized policy definitions with cluster-specific
configuration reduce drift. Each cluster should still have its own
failure and access boundary.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 80. Security in Multi-Environment

Development can have more permissive debugging while staging and
production require stronger controls. However, security-sensitive
behavior should be representative enough that production-only failures
are minimized.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 81. Developer Experience

Security controls are more effective when remediation is easy. Provide
reusable CI templates, standard scanner configurations, clear error
messages, secure base images, approved dependency sources, and
documented exception processes.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 82. Security Pipeline Performance

Security checks can increase pipeline duration. Parallelize independent
scanners, cache safe dependency data, use incremental analysis where
reliable, and reserve expensive deep scans for appropriate branches or
schedules. Never optimize by removing critical security controls without
a documented risk decision.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 83. Scheduled Security Scans

Run scheduled scans against repositories and production artifact
inventories even when source code has not changed. New CVEs appear after
deployment, so event-driven CI alone cannot detect all emerging risk.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 84. Continuous Compliance

Compliance controls should be continuously testable: encrypted
registries, protected branches, signed images, approved base images,
least-privilege roles, and audit logging can be validated automatically.
Evidence should be generated from systems rather than maintained
manually when possible.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 85. Secure CI Template

A shared pipeline template should establish secure defaults: no secret
printing, protected release jobs, OIDC authentication, dependency
scanning, image scanning, SBOM generation, artifact retention, and
explicit deployment boundaries. Services should opt into capabilities
rather than recreating security logic from scratch.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 86. Secure Dockerfile Baseline

Use a maintained pinned base image, multi-stage builds, a minimal
runtime, non-root execution, no embedded credentials, explicit ports
where useful, predictable entrypoints, and a .dockerignore. Scan the
final image and record its digest.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 87. Secure Helm Baseline

Secure charts should provide non-root security contexts, dropped
capabilities, resource requests and limits, readiness and liveness
probes where appropriate, network policy integration, service-account
controls, and safe image references. Values should not contain plaintext
production secrets.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 88. Secure Terraform Baseline

Terraform should use remote state with encryption and locking
appropriate to the platform, least-privilege execution roles, encrypted
data stores, restricted security groups, private networking where
required, and policy checks before apply. Sensitive outputs must not be
exposed unnecessarily.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 89. Security Review Checklist

Before production promotion ask: Is the source reviewed? Did tests pass?
Did SAST and dependency checks pass? Were secrets scanned? Was IaC and
Kubernetes policy checked? Was the final image scanned? Is an SBOM
available? Is the image signed? Is the digest immutable? Are GitOps and
production approvals valid? Can we trace the artifact? Can we roll back?

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 90. Senior Interview: DevSecOps

A strong answer is: I embed security into CI with SAST, SCA, secret
detection, IaC and Kubernetes policy checks, container scanning, SBOM
generation, and artifact provenance. AWS authentication uses OIDC with
short-lived STS credentials and least-privilege IAM. Images are
immutable and preferably signed. CI updates GitOps with the digest
rather than directly deploying to EKS. Admission policies provide a
second enforcement boundary, while runtime identity, secrets management,
network policy, logging, and monitoring provide defense in depth.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 91. Senior Interview: Why OIDC

OIDC removes the need to store long-lived AWS access keys in CI. The
runner receives a short-lived identity token, AWS validates the trusted
issuer and claims, and STS returns temporary credentials for the
required role. If the job ends, the credentials naturally expire.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 92. Senior Interview: Why SBOM

An SBOM gives an inventory of components inside a specific artifact.
During a newly disclosed vulnerability, we can query which deployed
services contain the affected component rather than manually inspecting
every repository.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 93. Senior Interview: Why Sign Images

A digest proves which content is referenced, while a signature can
establish who or what trusted build process produced that content.
Admission verification can prevent unapproved artifacts from entering
protected clusters.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 94. Senior Interview: Why Scan Multiple Layers

SAST, SCA, IaC scanning, image scanning, admission, and runtime security
address different failure modes. No single scanner sees source
vulnerabilities, dependency vulnerabilities, cloud misconfiguration,
image composition, deployment policy, and runtime behavior
simultaneously.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## 95. Final Production Model

The complete security model is: secure source -\> secure identity -\>
isolated build -\> trusted dependencies -\> reproducible image -\> SBOM
-\> vulnerability gate -\> signature and provenance -\> restricted
registry -\> protected GitOps -\> Argo CD -\> admission policy -\>
least-privileged runtime -\> continuous monitoring -\> tested incident
response. This is the DevSecOps chain for the capstone.

### Production implementation checks

-   Identify the exact security boundary being protected.
-   Keep the control enforceable and auditable.
-   Give the job only the credentials it actually needs.
-   Produce machine-readable evidence where possible.
-   Define remediation and exception ownership.
-   Ensure the control cannot be silently bypassed by a normal release
    path.

### Operational questions

1.  What happens when this control fails?
2.  Who receives the failure and who owns remediation?
3.  What evidence proves the control actually ran?
4.  How is the control tested when the pipeline itself changes?
5.  What is the safe recovery path during an incident?

## Complete DevSecOps Flow

``` text
                    SOURCE
                      |
              Merge Request / Review
                      |
        +-------------+-------------+
        |             |             |
      SAST          SCA       Secret Detection
        |             |             |
        +-------------+-------------+
                      |
                Tests + IaC Scan
                      |
                      v
               Secure Build
                 BuildKit
                      |
          +-----------+-----------+
          |                       |
        SBOM                Image Scan
          |                       |
          +-----------+-----------+
                      |
               Quality / Risk Gate
                      |
             Sign + Provenance
                      |
                      v
                    ECR
                      |
             Immutable Digest
                      |
                      v
                 GitOps Repo
                      |
                Protected PR
                      |
                      v
                  Argo CD
                      |
                EKS Admission
                      |
                      v
              Kubernetes Runtime
                      |
       +--------------+--------------+
       |              |              |
   Workload ID   Network Policy   Runtime Logs
       |              |              |
       +--------------+--------------+
                      |
                Monitoring / SIEM
```

## Final DevSecOps Checklist

-   [ ] Protected source branches
-   [ ] Merge-request security checks
-   [ ] SAST
-   [ ] Dependency/SCA scanning
-   [ ] Secret detection
-   [ ] Terraform/IaC scanning
-   [ ] Kubernetes/Helm policy scanning
-   [ ] Isolated CI runners
-   [ ] OIDC federation to AWS
-   [ ] Short-lived STS credentials
-   [ ] Least-privilege IAM
-   [ ] Secure Docker build
-   [ ] Minimal non-root image
-   [ ] Final-image vulnerability scanning
-   [ ] SBOM generation
-   [ ] Artifact provenance
-   [ ] Image signing
-   [ ] ECR security
-   [ ] Immutable digest deployment
-   [ ] Protected GitOps repository
-   [ ] Argo CD access control
-   [ ] Kubernetes admission policy
-   [ ] Workload identity
-   [ ] Network policies
-   [ ] Runtime logging and monitoring
-   [ ] Vulnerability exception process
-   [ ] Secret rotation procedure
-   [ ] Supply-chain incident runbook
-   [ ] Tested rollback and recovery

## Capstone Principle

**DevSecOps is successful when security is enforced at multiple
independent boundaries, risk is visible before production, identities
are short-lived and least-privileged, artifacts are traceable and
immutable, and the organization can recover quickly when a control or
dependency is compromised.**

---