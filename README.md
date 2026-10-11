# 🛠️ Programmatic Vulnerability Remediations

<img width="1000" alt="Programmatic remediations header image" src="https://github.com/user-attachments/assets/f5dea6a6-f7e9-4879-ad07-6a1a501a704a" />


> Turning vulnerability findings into repeatable fixes.

This directory contains the remediation commands and PowerShell scripts used throughout my **Vulnerability Management Portfolio Project**.

The goal was not simply to identify vulnerabilities with Tenable, but to take findings through the remediation lifecycle:

**Detect → Investigate → Remediate → Validate**

The commands below address operating system configuration issues, insecure authentication settings, vulnerable third-party software and other security findings identified during authenticated vulnerability assessments.

---

## 🔎 Remediation Overview

| # | Vulnerability / Configuration | Tenable Plugin ID | Remediation |
|---|---|---:|---|
| 01 | Windows Update Configuration | N/A | Restore automatic Windows Update configuration |
| 02 | Guest Account Hardening | N/A | Remove Guest from Administrators and disable the account |
| 03 | Wireshark / Ethereal Unsupported Version | `56710` | Remove vulnerable Wireshark / WinPcap installation |
| 04 | SMB Signing Not Required | `57608` | Require SMB server signing |
| 05 | EdgeCore / libcurl Vulnerabilities | `342615`, `342617` | Remove residual vulnerable EdgeCore components |
| 06 | RDP Network Level Authentication | `58453` | Require Network Level Authentication for RDP |
| 07 | LM / NTLMv1 Authentication Enabled | `63478` | Restore stronger LAN Manager authentication |

---

# 🪟 01 — Windows Update Remediation

### Finding

Windows Update settings had been intentionally weakened for the lab environment.

### 🛠️ Remediation

Run from an elevated Command Prompt:

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v ScheduledInstallDay /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v AUOptions /t REG_DWORD /d 4 /f
```

### 🔍 Verification

```cmd
reg query "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
```

The system was subsequently updated and rescanned with Tenable to validate the remediation.

---

# 👤 02 — Guest Account Hardening

### Finding

The built-in Guest account had been intentionally configured with excessive privileges during the lab.

### 🛠️ Remediation

Remove the Guest account from the local Administrators group:

```cmd
net localgroup Administrators Guest /delete
```

Disable the Guest account:

```cmd
net user Guest /active:no
```

### 🔍 Verification

```cmd
net user Guest
```

The account should report:

```text
Account active               No
```

Administrator group membership can also be reviewed with:

```cmd
net localgroup Administrators
```

---

# 🦈 03 — Vulnerable Wireshark / WinPcap Removal

### Tenable Finding

**Plugin 56710 — Wireshark / Ethereal Unsupported Version Detection**

An intentionally outdated installation of Wireshark and associated components generated multiple Critical, High and Medium vulnerability findings.

### 🛠️ Remediation

The vulnerable third-party software was removed from the system.

PowerShell can be used to identify installed Wireshark-related software:

```powershell
Get-CimInstance Win32_Product |
    Where-Object {
        $_.Name -match "Wireshark|WinPcap"
    } |
    Select-Object Name, Version
```

After removal, the host was rescanned to verify that the associated Wireshark findings were no longer detected.

### 💥 Result

This remediation produced the largest reduction in the vulnerability count during the project:

**24 → 7 reportable vulnerabilities**

---

# 🔐 04 — SMB Signing Remediation

### Tenable Finding

**Plugin 57608 — SMB Signing Not Required**

SMB signing helps protect SMB communication against tampering and certain man-in-the-middle attacks.

### 🛠️ Remediation

Run from an elevated PowerShell session:

```powershell
Set-SmbServerConfiguration `
    -RequireSecuritySignature $true `
    -EnableSecuritySignature $true `
    -Force
```

### 🔍 Verification

```powershell
Get-SmbServerConfiguration |
    Select-Object RequireSecuritySignature, EnableSecuritySignature
```

Expected result:

```text
RequireSecuritySignature EnableSecuritySignature
------------------------ -----------------------
True                     True
```

The subsequent authenticated Tenable scan confirmed that Plugin `57608` was no longer detected.

---

# 🧩 05 — EdgeCore / libcurl Remediation

### Tenable Findings

**Plugin 342615 — libcurl Negotiate Ambient User Connection Reuse**

**Plugin 342617 — libcurl HTTP/2 Server Push Use-After-Free**

Two Critical libcurl findings remained after the primary third-party software remediation.

Rather than assuming the findings were false positives, the affected filesystem paths were investigated to identify the source.

### 🔎 Investigation

The vulnerable library was traced to residual **EdgeCore / Microsoft Edge-related application files** remaining on the system.

This demonstrated an important vulnerability management lesson:

> A currently installed application can appear updated while an older vulnerable binary still exists elsewhere on disk.

### 🛠️ Remediation

The residual vulnerable components were removed after identifying the affected paths.

### 🔍 Validation

An authenticated Tenable rescan confirmed that both Critical libcurl findings were no longer detected.

### 💥 Result

```text
Critical vulnerabilities: 2 → 0
Total vulnerabilities:    6 → 4
```

This was the most investigation-heavy remediation in the project and demonstrated why scanner findings still require analyst validation rather than blind remediation.

---

# 🖥️ 06 — RDP Network Level Authentication

### Tenable Finding

**Plugin 58453 — Terminal Services Doesn't Use Network Level Authentication (NLA) Only**

Network Level Authentication requires authentication before a full Remote Desktop session is established, reducing unnecessary exposure of the RDP service.

### 🛠️ Remediation

Run from an elevated PowerShell session:

```powershell
Set-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
    -Name "UserAuthentication" `
    -Value 1 `
    -Force
```

### 🔍 Verification

```powershell
Get-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" `
    -Name "UserAuthentication"
```

Expected value:

```text
UserAuthentication : 1
```

The subsequent authenticated Tenable scan confirmed that Plugin `58453` was no longer detected.

---

# 🔑 07 — LAN Manager Authentication Hardening

### Tenable Finding

**Plugin 63478 — Microsoft Windows LM / NTLMv1 Authentication Enabled**

Weak LAN Manager authentication had been intentionally enabled as part of the lab environment.

### 🛠️ Remediation

Restore the stronger LAN Manager authentication configuration:

```powershell
Set-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "LmCompatibilityLevel" `
    -Value 5 `
    -Type DWord `
    -Force
```

### 🔍 Verification

```powershell
Get-ItemProperty `
    -Path "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" `
    -Name "LmCompatibilityLevel"
```

Expected value:

```text
LmCompatibilityLevel : 5
```

The final authenticated Tenable compliance assessment validated the configuration as:

**Send NTLMv2 response only. Refuse LM & NTLM**

The associated compliance audit passed following remediation.

---

# 📉 Remediation Impact

The remediation cycle reduced the host from:

| Severity | Initial | Final |
|---|---:|---:|
| 🔴 Critical | **4** | **0** |
| 🟠 High | **8** | **0** |
| 🟡 Medium | **11** | **2** |
| 🟢 Low | **1** | **1** |
| **Total** | **24** | **3** |

### 📊 **87.5% reduction in reportable vulnerabilities**

All **Critical and High severity findings were eliminated**.

The remaining findings consisted of two Medium certificate-related findings and one Low ICMP timestamp disclosure finding.

---

# 🧠 What This Demonstrated

This portion of the project moved beyond simply running vulnerability scans.

Each remediation required some combination of:

- 🔎 Interpreting scanner findings
- 🪟 Identifying affected Windows configurations or software
- ⚙️ Applying configuration changes through PowerShell or the command line
- 🧩 Investigating unexpected residual findings
- ✅ Validating system state after remediation
- 🔄 Performing authenticated Tenable rescans
- 📊 Comparing results across the remediation lifecycle

Most importantly, remediation was treated as a **closed-loop process**.

> **A vulnerability was not considered resolved simply because a command executed successfully.**
>
> **The rescan was the proof.**

---

## ⚠️ Lab Disclaimer

These commands were created and tested in an isolated cybersecurity lab environment for educational purposes.

Configuration changes should be tested and evaluated before being deployed to production systems. Security controls appropriate for one environment may introduce operational or compatibility issues in another.
