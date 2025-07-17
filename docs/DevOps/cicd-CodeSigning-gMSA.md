---
layout: page
title: DevOps - Code Signing with gMSA
parent: DevOps
---

# Code Signing with Certificates in CI/CD: A Developer's Guide

In modern CI/CD pipelines, signing executables and libraries is a critical step for security and integrity. This guide summarizes essential knowledge for developers setting up software signing with certificates, particularly in enterprise environments using gMSA accounts and Windows Server Core.

---

## 🔐 1. Signing Certificates in the Pipeline

If you're building software via Azure DevOps and wish to sign your output (e.g., `.exe`, `.dll`), you need:

- A valid Code Signing certificate (`.pfx`)
- The corresponding password
- A signing user context (e.g., gMSA account)
- Access to a timestamp server for trusted long-term signatures

### Where to install the certificate?
Certificates are installed into the `LocalMachine\My` store — this is the correct location for certificates used by services or system-wide tasks.

> The "My" store refers to "Personal" certificates in the context of the machine.

### What to sign?
To gain full benefits (e.g., SmartScreen, AV trust, preventing tampering), sign:

- All `.exe` files (client and server)
- All your own `.dll` libraries

**Avoid signing third-party libraries** (e.g., from NuGet) — you are not the publisher and should not modify them.

---

## 🕒 2. Timestamping: Why and How

### Purpose
A timestamp proves that the code was signed **when the certificate was valid**, even if the certificate expires later.

### Typical command (via SignTool)
```bash
signtool sign /fd sha256 /tr http://timestamp.digicert.com /td sha256 /f yourcert.pfx /p yourpassword yourapp.exe
```

- `/fd sha256`: Digest algorithm for the file
- `/td sha256`: Digest algorithm for the timestamp
- `/tr`: RFC 3161-compliant timestamp server

> Note: The Digicert URL must be accessible over the internet.

---

## 🏛️ 3. Using Certificates with gMSA Accounts

If your DevOps agent runs as a gMSA (Group Managed Service Account), you must ensure the certificate is:

- Installed in `LocalMachine\My`
- Its private key is accessible by the gMSA account

### Certificate location after import:
`%ProgramData%\Microsoft\Crypto\RSA\MachineKeys` — this is where private key material is stored.

> The key file permissions must allow access for the gMSA (e.g., `GMSA-DVOPS$`).

---

## ⚙️ 4. Automating Certificate Import

You can use PowerShell to automate:

- Loading configuration from a JSON file (`certConfig.json`)
- Reading the `.pfx` and installing it for machine use
- Granting key access to the gMSA

### Example Configuration
```json
{
  "certPassword": "YourSecurePassword",
  "certFileName": "signingCert.pfx",
  "signingServiceUser": "DOMAIN\\GMSA-DVOPS$"
}
```

Scripts:
- `Inspect-Cert.ps1`: inspects EKU and metadata before installation
- `Install-CodeSignCert.ps1`: installs the certificate securely


**`Inspect-Cert.ps1`**
```powershell
# =============================
# Inspect certificate before installation
# =============================

# Get script directory
$scriptDir = Split-Path -Parent $MyInvocation.MyCommand.Definition

# Load config from JSON
$configPath = Join-Path $scriptDir "certConfig.json"
if (!(Test-Path $configPath)) {
    Write-Error "Configuration file certConfig.json not found."
    exit 1
}

$config = Get-Content $configPath | ConvertFrom-Json
$pfxFile = Join-Path $scriptDir $config.certFile
$certPassword = $config.certPassword

# Validate file exists
if (!(Test-Path $pfxFile)) {
    Write-Error "PFX file '$pfxFile' not found."
    exit 1
}

# Load certificate from PFX without importing
try {
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
    $cert.Import($pfxFile, $certPassword, "Exportable,PersistKeySet")

    Write-Host "=== Certificate Information ==="
    Write-Host "Subject         : $($cert.Subject)"
    Write-Host "Issuer          : $($cert.Issuer)"
    Write-Host "Thumbprint      : $($cert.Thumbprint)"
    Write-Host "Valid From      : $($cert.NotBefore)"
    Write-Host "Valid Until     : $($cert.NotAfter)"
    Write-Host "Has Private Key : $($cert.HasPrivateKey)"

    $eku = $cert.Extensions | Where-Object { $_.Oid.FriendlyName -eq "Enhanced Key Usage" }
    if ($eku) {
        $decoded = New-Object System.Security.Cryptography.X509Certificates.X509EnhancedKeyUsageExtension $eku, $true
        Write-Host "Enhanced Key Usages:"
        foreach ($usage in $decoded.EnhancedKeyUsages) {
            Write-Host " - $($usage.FriendlyName) ($($usage.Value))"
        }
    } else {
        Write-Host "No Enhanced Key Usage extensions found."
    }

} catch {
    Write-Error "Failed to read the certificate: $_"
}

```

**`Install-CodeSignCert.ps1`**
```powershell
# =============================
# Install certificate for code signing from PFX
# =============================

# Get script directory
$scriptDir = Split-Path -Parent $MyInvocation.MyCommand.Definition

# Load config from JSON
$configPath = Join-Path $scriptDir "certConfig.json"
if (!(Test-Path $configPath)) {
    Write-Error "Configuration file certConfig.json not found."
    exit 1
}

$config = Get-Content $configPath | ConvertFrom-Json
$pfxFile = Join-Path $scriptDir $config.certFile
$certPassword = ConvertTo-SecureString -String $config.certPassword -AsPlainText -Force
$gmsaAccount = $config.signingServiceUser

# Validate file exists
if (!(Test-Path $pfxFile)) {
    Write-Error "PFX file '$pfxFile' not found."
    exit 1
}

# Import certificate into LocalMachine\My store
try {
    $cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
    $cert.Import($pfxFile, $config.certPassword, "Exportable,PersistKeySet")

    $store = New-Object System.Security.Cryptography.X509Certificates.X509Store("My", "LocalMachine")
    $store.Open("ReadWrite")
    $store.Add($cert)
    $store.Close()

    Write-Host "Certificate successfully installed in LocalMachine\My store."

    # Set private key permissions for gMSA account
    $thumbprint = $cert.Thumbprint
    $keyPath = "$env:ProgramData\Microsoft\Crypto\RSA\MachineKeys"
    $keyFile = Get-ChildItem $keyPath | Where-Object {
        (Get-Acl $_.FullName).Owner -like "*$($thumbprint.Substring(0,8))*"
    } | Select-Object -First 1

    if ($keyFile) {
        $acl = Get-Acl $keyFile.FullName
        $permission = "$gmsaAccount","Read","Allow"
        $accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule $permission
        $acl.AddAccessRule($accessRule)
        Set-Acl -Path $keyFile.FullName -AclObject $acl
        Write-Host "Permissions granted to $gmsaAccount on key file: $($keyFile.FullName)"
    } else {
        Write-Warning "Could not locate the private key file for the certificate."
    }

} catch {
    Write-Error "Failed to install certificate: $_"
}

```

---

## 🔍 5. EKU — Enhanced Key Usage: Required?

### Scenario: No EKU Present
Some certificates may not list any Enhanced Key Usage. According to [RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280), **a missing EKU means the certificate is valid for all usages**.

### But in practice:
Many tools **expect** an EKU for Code Signing:

- OID: `1.3.6.1.5.5.7.3.3` (Code Signing)
- Friendly name: `Code Signing`

#### Without EKU:
- Tools may show warnings
- SmartScreen may block the file
- Timestamping may fail
- EV policies not satisfied

### ✅ Recommendation
Use certificates with **explicit EKU** for code signing.

If your certificate lacks EKU, request a reissue from your CA with the correct settings.

---

## 🧪 6. How to Inspect a Certificate Before Installing

```powershell
$pfx = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$pfx.Import("signingCert.pfx", "yourpassword", "Exportable,PersistKeySet")
$pfx.Extensions | Where-Object { $_.Oid.FriendlyName -eq "Enhanced Key Usage" } | Format-List
```

Also helpful:
```powershell
$pfx.Issuer
$pfx.Subject
$pfx.NotBefore
$pfx.NotAfter
```

---

## 🧾 Summary

| Topic                       | Best Practice                                       |
|-----------------------------|-----------------------------------------------------|
| Certificate location        | LocalMachine\My                                     |
| Who needs access?           | gMSA account running the DevOps Agent               |
| Sign what?                  | Your own EXE and DLL files                          |
| Skip signing                | Third-party NuGet libraries                         |
| EKU required?               | ✅ Recommended (OID: 1.3.6.1.5.5.7.3.3)             |
| Timestamping                | ✅ Always use, e.g., Digicert /tr URL               |
| Tool                        | `signtool` with SHA256 and RFC3161 timestamping     |
| Key location                | %ProgramData%\Microsoft\Crypto\RSA\MachineKeys      |
