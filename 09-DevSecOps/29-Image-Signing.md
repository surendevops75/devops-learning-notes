# Container Image Signing

## Introduction

Container Image Signing is the process of digitally signing container images to verify their authenticity, integrity, and publisher before deployment.

It ensures that only trusted container images are deployed into Kubernetes clusters, preventing image tampering and software supply chain attacks.

In DevSecOps, Image Signing is a critical security control that complements Container Scanning, SBOM, SLSA, and Admission Controllers.

---

## Why Do We Need Image Signing?

Container images stored in registries can be modified or replaced by attackers.

Without Image Signing:

- Fake container images may be deployed
- Image integrity cannot be verified
- CI/CD pipelines cannot validate trusted artifacts
- Supply chain attacks become easier
- Compliance requirements become difficult to meet

Image Signing ensures that only verified images are deployed.

---

## What is Image Signing?

Image Signing uses cryptographic signatures to prove that a container image:

- Was created by a trusted publisher
- Has not been modified
- Passed security validation
- Can be safely deployed

Image Signing commonly works with:

- Sigstore Cosign
- Docker Content Trust (Notary)
- Notation
- OCI Registries

---

## How It Works

```text
Docker Build

↓

Container Image

↓

Security Scan

↓

Generate SBOM

↓

Sign Image

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

Container Scan

↓

Generate SBOM

↓

Image Signing

↓

Container Registry

↓

Admission Controller

↓

Verify Signature

↓

Kubernetes Cluster
```

---

## Workflow

```text
Write Code

↓

Build Image

↓

Scan Image

↓

Generate SBOM

↓

Sign Image

↓

Push Registry

↓

Verify Signature

↓

Deploy
```

---

# Components of Image Signing

## Container Image

The artifact to be signed.

Examples

- Docker Image
- OCI Image

---

## Digital Signature

A cryptographic signature attached to the image.

```text
Container Image

↓

Generate Hash

↓

Private Key

↓

Digital Signature
```

---

## Registry

Stores:

- Container Images
- Signatures
- Attestations

Examples

- Amazon ECR
- Azure ACR
- Google Artifact Registry
- Harbor
- Docker Hub

---

## Verification

Before deployment:

```text
Container Image

↓

Retrieve Signature

↓

Verify

↓

Deploy
```

Only trusted images are allowed.

---

# Image Signing Workflow

```text
Docker Build

↓

Trivy Scan

↓

Generate SBOM

↓

Cosign Sign

↓

Push Registry

↓

Kyverno Verify

↓

Deploy
```

---

# Image Verification Flow

```text
Deployment Request

↓

Admission Controller

↓

Retrieve Signature

↓

Verify Signature

↓

Trusted?

↓

Yes

↓

Deploy

OR

No

↓

Reject
```

---

# Benefits of Image Signing

## Image Integrity

Ensures container images have not been modified.

---

## Publisher Authentication

Verifies the identity of the image publisher.

---

## Supply Chain Security

```text
Image

↓

Signed

↓

Verified

↓

Deploy
```

Prevents unauthorized images from entering production.

---

## Compliance

Supports:

- SLSA
- NIST SSDF
- PCI DSS
- ISO 27001
- Executive Order 14028

---

## Trust

Developers and deployment systems can verify every image before use.

---

# Common Image Signing Tools

| Tool | Purpose |
|------|---------|
| Sigstore Cosign | OCI image signing and verification |
| Docker Content Trust | Docker image signing |
| Notation | OCI artifact signing |
| Notary v2 | OCI trust framework |
| Cosign Verify | Image verification |

---

## Pipeline Integration

```text
Developer

↓

Git Commit

↓

SAST

↓

SCA

↓

Docker Build

↓

Container Scan

↓

Generate SBOM

↓

Image Signing

↓

Push Registry

↓

Admission Controller

↓

Verify Image Signature

↓

Deploy Production
```

Image verification should always occur before deployment.

---

## Production Workflow

```text
Developer

↓

Build Image

↓

Security Scan

↓

Generate SBOM

↓

Sign Image

↓

Push Registry

↓

Verify Signature

↓

Deploy Kubernetes
```

---

## Production Example

A developer builds a container image for a payment service.

```text
Docker Build

↓

Trivy Scan

↓

No Critical CVEs

↓

Cosign Sign

↓

Push Amazon ECR

↓

Kyverno VerifyImages

↓

Deploy Amazon EKS
```

When another user attempts to deploy an unsigned image:

```text
Unsigned Image

↓

Kyverno VerifyImages

↓

Signature Missing

↓

Deployment Rejected
```

Only trusted images are deployed.

---

## Best Practices

- Sign every production image
- Verify signatures before deployment
- Scan images before signing
- Generate an SBOM before signing
- Store images in trusted registries
- Rotate signing keys regularly
- Use keyless signing where possible
- Integrate image verification with Admission Controllers
- Audit image signing events
- Continuously monitor registry images

---

## Common Mistakes

- Deploying unsigned images
- Skipping signature verification
- Signing vulnerable images
- Using insecure private key storage
- Ignoring failed verification
- Allowing images from public registries without validation
- Forgetting to rotate signing keys
- Treating signing as optional

---

# Common Troubleshooting

## Issue 1

### Deployment Rejected Due to Missing Signature

**Cause**

The image was pushed without being signed.

**Resolution**

```text
Build Image

↓

Sign Image

↓

Push Registry

↓

Deploy Again
```

---

## Issue 2

### Signature Verification Failed

**Cause**

The image was modified after signing.

**Resolution**

```text
Rebuild Image

↓

Generate New Signature

↓

Push Registry

↓

Verify Again
```

---

## Issue 3

### Admission Controller Blocks Deployment

**Cause**

Image verification policy failed.

**Resolution**

```text
Review Policy

↓

Verify Image

↓

Update Signature

↓

Deploy Again
```

---

## Issue 4

### Signing Key Compromised

**Cause**

Private signing key was exposed.

**Resolution**

```text
Revoke Key

↓

Generate New Key

↓

Re-sign Images

↓

Update Verification Policies
```

---

## Issue 5

### Image Contains Critical Vulnerabilities After Signing

**Cause**

The image was signed before all vulnerabilities were resolved.

**Resolution**

```text
Update Dependencies

↓

Rebuild Image

↓

Run Container Scan

↓

Sign Updated Image

↓

Deploy
```

---

# Production Interview Questions

## Question 1

### What is Container Image Signing?

Container Image Signing is the process of digitally signing container images to verify their authenticity and integrity before deployment.

---

## Question 2

### Why is Image Signing important?

It prevents image tampering, ensures trusted deployments, and strengthens software supply chain security.

---

## Question 3

### Which tools are commonly used for Image Signing?

Sigstore Cosign, Docker Content Trust, Notation, and Notary v2.

---

## Question 4

### Why should images be scanned before signing?

Only secure, vulnerability-free images should be signed. Signing a vulnerable image makes it trusted but not secure.

---

## Question 5

### How does Kubernetes verify signed images?

Admission Controllers such as Kyverno VerifyImages or OPA Gatekeeper integrated with Cosign validate image signatures before allowing deployment.

---

## Question 6

### What happens if image signature verification fails?

The deployment should be rejected because the image cannot be verified as authentic or untampered.

---

## Question 7

### What is the relationship between Image Signing and SBOM?

The SBOM documents the components inside an image, while Image Signing guarantees the authenticity and integrity of that image.

---

## Question 8

### How does Image Signing support SLSA?

Image Signing verifies the integrity of build artifacts, while SLSA provides assurance about how those artifacts were produced. Together they strengthen the software supply chain.

---

## Question 9

### What is keyless image signing?

Keyless signing uses identity-based certificates (such as OIDC with Sigstore Fulcio) instead of long-lived private keys to sign images.

---

## Question 10

### How would you implement Image Signing in a production CI/CD pipeline?

Build the image, scan it for vulnerabilities, generate an SBOM, sign it with Cosign, push it to a trusted registry, and configure Kubernetes Admission Controllers to verify signatures before deployment.

---

# Key Takeaways

- Container Image Signing verifies image authenticity and integrity.
- Images should always be scanned before they are signed.
- Admission Controllers should verify signatures before deployment.
- Sigstore Cosign is the most widely adopted tool for OCI image signing.
- Image Signing is a core practice for securing Kubernetes deployments and modern software supply chains.