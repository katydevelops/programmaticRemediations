# 🛠️ Programmatic Vulnerability Remediations

> Turning vulnerability findings into repeatable fixes.

This directory contains the remediation commands and PowerShell scripts used throughout my **Vulnerability Management Portfolio Project**.

The goal was not simply to identify vulnerabilities with Tenable, but to take each finding through the remediation lifecycle:

**Detect → Investigate → Remediate → Validate**

The scripts and commands below address operating system configuration issues, insecure authentication settings, vulnerable third-party software and other security findings identified during authenticated vulnerability assessments.

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

### Remediation

Run from an elevated Command Prompt:

```cmd
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v ScheduledInstallDay /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v AUOptions /t REG_DWORD /d 4 /f
