# Code Signing

## Introduction

Code Signing is the process of digitally signing software, scripts, binaries, and container images to verify their authenticity and integrity.

It ensures that software has not been modified after it was signed and confirms that it came from a trusted publisher.

In DevSecOps, Code Signing is a critical control for securing the software supply chain.

---

## Why Do We Need Code Signing?

Software can be modified by attackers during development, storage, or distribution.

Without Code Signing:

- Malware can replace legitimate software
- Attackers can tamper with artifacts
- Users cannot verify software authenticity
- Supply chain attacks become easier
- Organizations lose trust in software releases

Code Signing helps prevent unauthorized modifications and ensures software integrity.

---

## What is Code Signing?

Code Signing uses cryptographic techniques to digitally sign software artifacts.

A digital signature proves:

- Who created the software
- Whether the software has been modified
- Whether the publisher is trusted

Common artifacts that can be signed include:

- Source Code Releases
- Executables
- JAR Files
- Docker Images
- Helm Charts
- Kubernetes Manifests
- Terraform Modules

---

## How It Works

```text
Build Artifact

↓

Generate Hash

↓

Encrypt Hash

↓

Digital Signature

↓

Attach Signature

↓

Distribute Software

↓

Verify Signature

↓

Install Software
```

---

## Architecture

```text
Developer

↓

CI/CD Pipeline

↓

Build Artifact

↓

Generate Hash

↓

Private Key

↓

Digital Signature

↓

Artifact Repository

↓

Deployment

↓

Verify Using Public Key
```

---

## Workflow

```text
Developer Writes Code

↓

Build Application

↓

Generate Artifact

↓

Sign Artifact

↓

Store Artifact

↓

Verify Signature

↓

Deploy
```

---

# Components of Code Signing

## Artifact

The software package that will be signed.

Examples:

- Binary
- Container Image
- Helm Chart
- ZIP Package

---

## Hash Function

A cryptographic hash uniquely represents the artifact.

Common algorithms:

- SHA-256
- SHA-384
- SHA-512

```text
Artifact

↓

SHA-256

↓

Unique Hash
```

---

## Private Key

Used to create the digital signature.

```text
Private Key

↓

Encrypt Hash

↓

Signature
```

The private key must always remain secure.

---

## Public Key

Used to verify the signature.

```text
Artifact

↓

Signature

↓

Public Key

↓

Verified
```

Anyone can use the public key to verify authenticity.

---

# Code Signing Process

```text
Build Artifact

↓

Calculate Hash

↓

Encrypt Hash

↓

Create Signature

↓

Publish Artifact

↓

Verify Signature

↓

Deploy
```

---

# Benefits of Code Signing

## Software Integrity

Ensures artifacts are not modified.

---

## Publisher Authentication

Confirms software comes from a trusted source.

---

## Supply Chain Protection

```text
Artifact

↓

Signed

↓

Verified

↓

Deploy
```

Unsigned or modified artifacts are rejected.

---

## Compliance

Supports standards such as:

- NIST
- SLSA
- PCI DSS
- ISO 27001

---

## Trust

Users and deployment systems can verify software authenticity before execution.

---

# Common Code Signing Tools

| Tool | Purpose |
|------|---------|
| Sigstore Cosign | Container and artifact signing |
| GPG | File and package signing |
| OpenSSL | Certificate and signature generation |
| Jarsigner | Java JAR signing |
| Notary | Docker image signing |

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

Build

↓

Generate Artifact

↓

Code Signing

↓

Verify Signature

↓

Push Registry

↓

Deploy
```

Only signed artifacts proceed to deployment.

---

## Production Workflow

```text
Developer

↓

Build Artifact

↓

Generate Signature

↓

Store Artifact

↓

Verify Signature

↓

Deploy Production
```

---

## Production Example

A CI/CD pipeline builds a Docker image.

```text
Docker Build

↓

Cosign Sign Image

↓

Push Registry

↓

Deployment Request

↓

Verify Signature

↓

Deployment Approved
```

If the signature verification fails, deployment is blocked.

---

## Best Practices

- Sign every production artifact
- Protect private signing keys
- Store keys in secure vaults or HSMs
- Verify signatures before deployment
- Rotate signing keys regularly
- Automate signing in CI/CD
- Use strong cryptographic algorithms
- Monitor signing activities
- Keep certificates up to date
- Reject unsigned artifacts

---

## Common Mistakes

- Storing private keys in source code
- Sharing signing keys among multiple users
- Deploying unsigned artifacts
- Ignoring signature verification
- Using weak cryptographic algorithms
- Not rotating signing keys
- Signing only release builds manually
- Forgetting to verify downloaded artifacts

---

# Common Troubleshooting

## Issue 1

### Signature Verification Failed

**Cause**

Artifact was modified after signing.

**Resolution**

```text
Verify Artifact

↓

Rebuild

↓

Sign Again

↓

Redeploy
```

---

## Issue 2

### Private Key Lost

**Cause**

Signing key was deleted or corrupted.

**Resolution**

```text
Recover Backup

↓

Generate New Key Pair

↓

Update CI/CD

↓

Re-sign Artifacts
```

---

## Issue 3

### Expired Certificate

**Cause**

The signing certificate has expired.

**Resolution**

```text
Renew Certificate

↓

Update Pipeline

↓

Sign Again
```

---

## Issue 4

### Unsigned Artifact Deployed

**Cause**

Signature verification step was skipped.

**Resolution**

```text
Add Verification Stage

↓

Fail Unsigned Builds

↓

Redeploy
```

---

## Issue 5

### Key Compromise

**Cause**

Private signing key was exposed.

**Resolution**

```text
Revoke Certificate

↓

Generate New Keys

↓

Re-sign Software

↓

Audit Systems
```

---

# Production Interview Questions

## Question 1

### What is Code Signing?

Code Signing is the process of digitally signing software artifacts to verify their authenticity and integrity.

---

## Question 2

### Why is Code Signing important?

It prevents tampering, verifies software authenticity, and strengthens software supply chain security.

---

## Question 3

### What is the role of a private key in Code Signing?

The private key creates the digital signature and must remain confidential.

---

## Question 4

### What is the role of a public key?

The public key verifies the digital signature and confirms the artifact has not been modified.

---

## Question 5

### What types of artifacts can be signed?

Executables, container images, Helm charts, JAR files, Terraform modules, Kubernetes manifests, and software packages.

---

## Question 6

### Where should signing keys be stored?

Signing keys should be stored in secure systems such as Hardware Security Modules (HSMs), cloud key management services, or secrets management platforms like HashiCorp Vault.

---

## Question 7

### What happens if signature verification fails?

The artifact should be rejected because its authenticity or integrity cannot be trusted.

---

## Question 8

### Which tools are commonly used for Code Signing?

Sigstore Cosign, GPG, OpenSSL, Jarsigner, and Docker Notary.

---

## Question 9

### How is Code Signing integrated into a DevSecOps pipeline?

Artifacts are automatically signed after the build stage, verified before deployment, and rejected if verification fails.

---

## Question 10

### How does Code Signing improve software supply chain security?

It ensures only trusted, untampered software is deployed, reducing the risk of malicious modifications and supply chain attacks.

---

# Key Takeaways

- Code Signing verifies software authenticity and integrity.
- Digital signatures protect software from tampering.
- Private keys create signatures, while public keys verify them.
- Integrate Code Signing into every CI/CD pipeline.
- Code Signing is a foundational practice for secure software supply chain management and DevSecOps.