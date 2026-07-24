# Sigstore Cosign

## Introduction

Sigstore Cosign is an open-source tool used to digitally sign, verify, and attest container images and other software artifacts.

It helps ensure that only authentic and untampered artifacts are deployed to production, making it a key component of modern DevSecOps and Software Supply Chain Security.

---

## Why Do We Need Sigstore Cosign?

Container images stored in registries can be modified or replaced by attackers.

Without image signing:

- Fake container images may be deployed
- Artifact integrity cannot be verified
- CI/CD pipelines cannot validate trusted images
- Supply chain attacks become easier
- Compliance requirements become difficult to satisfy

Cosign ensures that only trusted artifacts are deployed.

---

## What is Sigstore Cosign?

Cosign is part of the Sigstore project.

It provides:

- Container Image Signing
- Signature Verification
- Artifact Attestations
- Keyless Signing
- SBOM Attestations
- Supply Chain Security

It supports OCI-compatible registries such as:

- Docker Hub
- Amazon ECR
- Azure ACR
- Google Artifact Registry
- Harbor
- JFrog Artifactory

---

## How It Works

```text
Build Container

↓

Generate Image

↓

Cosign Sign

↓

Store Signature

↓

Push Registry

↓

Verify Signature

↓

Deploy Kubernetes
```

---

## Architecture

```text
Developer

↓

Git Repository

↓

CI/CD Pipeline

↓

Docker Build

↓

Container Image

↓

Cosign Sign

↓

OCI Registry

↓

Admission Controller

↓

Verify Signature

↓

Deploy Production
```

---

## Workflow

```text
Developer Commit

↓

Build Image

↓

Scan Image

↓

Generate SBOM

↓

Cosign Sign

↓

Push Registry

↓

Verify Signature

↓

Deploy
```

---

# Key Features

## Image Signing

Cosign digitally signs container images.

```text
Docker Image

↓

Cosign Sign

↓

Digital Signature

↓

Registry
```

---

## Signature Verification

Before deployment:

```text
Container Image

↓

Cosign Verify

↓

Valid Signature

↓

Deploy
```

Invalid images are rejected.

---

## Keyless Signing

Cosign supports signing without manually managing cryptographic keys.

Uses:

- OIDC Identity
- Fulcio Certificate Authority
- Rekor Transparency Log

```text
Developer

↓

GitHub Identity

↓

Fulcio

↓

Temporary Certificate

↓

Cosign Sign
```

---

## Transparency Log

Every signing event is recorded.

```text
Artifact

↓

Sign

↓

Rekor Log

↓

Public Verification
```

This provides tamper-evident records.

---

## Attestations

Cosign can attach metadata to artifacts.

Examples

- SBOM
- SLSA Provenance
- Vulnerability Reports
- Build Metadata
- Compliance Reports

```text
Image

↓

SBOM

↓

Cosign Attestation

↓

Registry
```

---

# Cosign Components

## Fulcio

Issues short-lived signing certificates.

---

## Rekor

Stores immutable transparency logs.

---

## Cosign CLI

Signs and verifies artifacts.

---

## OCI Registry

Stores images, signatures, and attestations.

---

# Signing Workflow

```text
Docker Build

↓

Generate Image

↓

Cosign Sign

↓

Store Signature

↓

Push Registry

↓

Verify Signature

↓

Deploy
```

---

# Verification Workflow

```text
Deployment Request

↓

Retrieve Image

↓

Retrieve Signature

↓

Verify Signature

↓

Trusted

↓

Deploy
```

---

# Production Use Cases

## Secure Container Images

Ensure only trusted images reach Kubernetes.

---

## Secure CI/CD Pipelines

Automatically sign every production build.

---

## Kubernetes Admission Policies

```text
Deployment

↓

Verify Signature

↓

Valid

↓

Allow

OR

Invalid

↓

Reject
```

---

## Supply Chain Security

Cosign works with:

- SBOM
- SLSA
- in-toto
- Kubernetes Admission Controllers

---

# Common Cosign Commands

## Sign an Image

```bash
cosign sign <image>
```

---

## Verify an Image

```bash
cosign verify <image>
```

---

## Generate an Attestation

```bash
cosign attest <image>
```

---

## Verify an Attestation

```bash
cosign verify-attestation <image>
```

---

# Pipeline Integration

```text
Developer

↓

Git Commit

↓

SAST

↓

SCA

↓

Build Image

↓

Container Scan

↓

Generate SBOM

↓

Cosign Sign

↓

Push Registry

↓

Verify Signature

↓

Deploy Kubernetes
```

Unsigned images should never reach production.

---

## Production Workflow

```text
Developer

↓

Build Image

↓

Trivy Scan

↓

Generate SBOM

↓

Cosign Sign

↓

Push Registry

↓

Admission Controller

↓

Verify Signature

↓

Deploy Production
```

---

## Production Example

A team builds a Docker image using GitHub Actions.

```text
GitHub Actions

↓

Docker Build

↓

Trivy Scan

↓

Cosign Sign

↓

Push Amazon ECR

↓

Kyverno Verify Signature

↓

Deploy Amazon EKS
```

If the image signature verification fails, Kyverno blocks the deployment automatically.

---

## Best Practices

- Sign every production artifact
- Verify signatures before deployment
- Enable keyless signing when possible
- Store signatures in OCI registries
- Generate SBOM attestations
- Integrate Cosign into CI/CD pipelines
- Use Admission Controllers for signature verification
- Rotate signing credentials regularly
- Monitor transparency logs
- Audit image signing events

---

## Common Mistakes

- Deploying unsigned images
- Skipping signature verification
- Storing private keys insecurely
- Ignoring failed verification
- Signing only release builds manually
- Not generating attestations
- Using untrusted registries
- Forgetting to rotate signing credentials

---

# Common Troubleshooting

## Issue 1

### Signature Verification Failed

**Cause**

The container image was modified after signing.

**Resolution**

```text
Rebuild Image

↓

Sign Again

↓

Verify Signature

↓

Deploy
```

---

## Issue 2

### Image Has No Signature

**Cause**

The CI/CD pipeline skipped the signing stage.

**Resolution**

```text
Update Pipeline

↓

Add Cosign Sign

↓

Push Image

↓

Verify
```

---

## Issue 3

### Admission Controller Rejects Deployment

**Cause**

Image signature verification failed.

**Resolution**

```text
Verify Image

↓

Re-sign

↓

Push Registry

↓

Deploy Again
```

---

## Issue 4

### Fulcio Certificate Error

**Cause**

OIDC authentication failed during keyless signing.

**Resolution**

```text
Verify Identity

↓

Authenticate Again

↓

Retry Signing
```

---

## Issue 5

### Missing Attestation

**Cause**

SBOM or provenance was not generated before signing.

**Resolution**

```text
Generate SBOM

↓

Generate Provenance

↓

Create Attestation

↓

Verify
```

---

# Production Interview Questions

## Question 1

### What is Sigstore Cosign?

Sigstore Cosign is an open-source tool used to digitally sign, verify, and attest container images and software artifacts.

---

## Question 2

### Why is Cosign important in DevSecOps?

It ensures that only authentic and untampered artifacts are deployed, improving software supply chain security.

---

## Question 3

### What is keyless signing?

Keyless signing uses OIDC identities with Fulcio-issued short-lived certificates, eliminating the need to manually manage long-term signing keys.

---

## Question 4

### What is Rekor?

Rekor is Sigstore's transparency log that records signing events in an immutable, publicly verifiable ledger.

---

## Question 5

### What is Fulcio?

Fulcio is Sigstore's certificate authority that issues short-lived certificates for signing artifacts.

---

## Question 6

### Can Cosign sign artifacts other than container images?

Yes. Cosign can sign OCI artifacts, binaries, Helm charts, SBOMs, attestations, and other supported software artifacts.

---

## Question 7

### How is Cosign integrated into a CI/CD pipeline?

After building and scanning an artifact, the pipeline signs it with Cosign, pushes it to a registry, and verifies the signature before deployment.

---

## Question 8

### How do Kubernetes Admission Controllers work with Cosign?

Admission Controllers such as Kyverno or OPA Gatekeeper can verify Cosign signatures and reject unsigned or untrusted images.

---

## Question 9

### What are Cosign attestations?

Attestations are signed metadata attached to artifacts, such as SBOMs, SLSA provenance, vulnerability reports, and compliance evidence.

---

## Question 10

### How does Cosign strengthen software supply chain security?

By digitally signing artifacts, verifying authenticity, recording signing events in Rekor, and integrating with Kubernetes policy enforcement, Cosign helps ensure only trusted software is deployed.

---

# Key Takeaways

- Sigstore Cosign provides secure signing and verification for container images and software artifacts.
- It supports both key-based and keyless signing.
- Fulcio issues certificates, while Rekor stores immutable transparency logs.
- Cosign integrates seamlessly with SBOMs, SLSA, and Kubernetes Admission Controllers.
- Using Cosign in CI/CD pipelines significantly improves software supply chain security and artifact integrity.