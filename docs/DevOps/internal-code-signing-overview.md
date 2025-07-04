---
layout: page
title: DevOps - Internal Code Signing - Overview
parent: DevOps
---

# Internal Code Signing - Overview

## Introduction to Internal Code Signing with Active Directory Certificate Services (AD CS)

In modern software development, code signing plays a crucial role in ensuring integrity, authenticity, and trust. Most developers associate code signing with expensive certificates from public Certificate Authorities (CAs) like DigiCert or GlobalSign. However, in enterprise environments, there's a powerful and cost-effective alternative: **internal code signing using your own PKI infrastructure, specifically with Active Directory Certificate Services (AD CS)**.

This article gives an overview of how internal code signing works, what role X.509 certificates play, and how to set up a trusted internal code signing process for your company.

---

## Why Sign Code?

Signed software ensures:
- **Integrity** – the code has not been tampered with.
- **Authenticity** – the code comes from a known and trusted source.
- **Trust** – end users and operating systems are more likely to execute code that is signed.

Unsigned executables often trigger warnings in Windows like “Unknown Publisher” or even block execution entirely through SmartScreen.

---

## What Is X.509 and Why Does It Matter?

X.509 is the industry-standard format for digital certificates. It is used in TLS/SSL, code signing, secure email, and many other areas. An X.509 certificate binds a public key to an identity (e.g., a person, server, or company) and is issued by a trusted Certificate Authority (CA).

A typical certificate chain includes:
- **Root CA** (trusted by the OS)
- **Intermediate CA** (issued by Root)
- **End Entity Certificate** (used for signing)

Public CAs restrict the creation of intermediate certificates to avoid abuse. For internal use, however, your company can issue and trust its own certificates using AD CS.

---

## Internal Code Signing: The Concept

Internal code signing involves:

1. Setting up your own enterprise CA (e.g., using AD CS)
2. Issuing code signing certificates for internal use
3. Signing your binaries in your CI/CD pipelines
4. Distributing your CA’s root certificate to all domain-joined machines via GPO (Group Policy)

This allows domain-joined clients to trust the signed binaries, eliminating UAC warnings or SmartScreen blocks.

---

## Step-by-Step Setup

### 1. Set Up Your CA with AD CS
- Install the **Enterprise Root CA** role on a Windows Server
- Configure certificate templates for **Code Signing**
- Enable auditing, access control, and expiration policies

### 2. Issue a Code Signing Certificate
- Use the MMC Certificate Snap-in or the CA web portal to request a new certificate based on the "Code Signing" template
- Export it as `.pfx` if needed for automated build use

### 3. Integrate into CI/CD
Use tools like `signtool.exe`:

```bash
signtool sign /fd SHA256 /f internal_codesign.pfx /p <password> /tr http://your-timestamp-server /td SHA256 myapp.exe
```

Make sure to include a timestamp server so that the signature remains valid even after the certificate expires.

### 4. Distribute Trust via Group Policy
Use GPO to deploy your internal CA’s **root certificate** to all client machines:

- Computer Configuration → Policies → Windows Settings → Security Settings → Public Key Policies → Trusted Root Certification Authorities

Once deployed, all signed binaries will be treated as trusted by Windows.

---

## Benefits of Internal Code Signing

| Benefit | Description |
|--------|-------------|
| Cost-effective | No need to buy certificates from public CAs |
| Trusted internally | Works seamlessly for domain-joined clients |
| Security control | You own the entire lifecycle (issuance, revocation, policies) |
| Automation-friendly | Can be integrated into your pipelines securely |

---

## Important Caveats

- **Not suitable for public distribution**: External users won’t trust your internal CA.
- **You must manage your PKI securely**: That includes CRL, key protection, and audits.
- **Certificate expiry still applies**: Use timestamping to avoid broken signatures.

---

## Conclusion

Internal code signing using Active Directory Certificate Services is a powerful and underused capability in enterprise environments. It allows companies to sign and trust their own software without relying on expensive third-party providers — as long as the scope is internal.

For public software, you'll still need a certificate from a globally trusted CA. But for internal tools, deployment packages, scripts, and installers, setting up internal code signing can vastly improve both trust and automation.
