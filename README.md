<div align="center">

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║    ██╗    ██╗██╗███╗   ██╗██████╗  ██████╗ ██╗    ██╗                       ║
║    ██║    ██║██║████╗  ██║██╔══██╗██╔═══██╗██║    ██║                       ║
║    ██║ █╗ ██║██║██╔██╗ ██║██║  ██║██║   ██║██║ █╗ ██║                       ║
║    ██║███╗██║██║██║╚██╗██║██║  ██║██║   ██║██║███╗██║                       ║
║    ╚███╔███╔╝██║██║ ╚████║██████╔╝╚██████╔╝╚███╔███╔╝                       ║
║     ╚══╝╚══╝ ╚═╝╚═╝  ╚═══╝╚═════╝  ╚═════╝  ╚══╝╚══╝                       ║
║                                                                              ║
║           S I E M  ·  L A B  —  S P L U N K  +  S Y S M O N               ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### `windows-siem-lab`

**Enterprise-grade SIEM pipeline on Windows Server — Sysmon telemetry, Splunk ingestion,
and detection engineering aligned to MITRE ATT&CK.**

---

![Platform](https://img.shields.io/badge/platform-Windows%20Server%202022-0078D4?style=flat-square&logo=windows&logoColor=white)
![Splunk](https://img.shields.io/badge/Splunk-Enterprise%209.x-FF6600?style=flat-square&logo=splunk&logoColor=white)
![Sysmon](https://img.shields.io/badge/Sysmon-v15.x-00ADEF?style=flat-square&logo=microsoft&logoColor=white)
![MITRE ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-mapped-E8001A?style=flat-square)
![Status](https://img.shields.io/badge/status-operational-00C853?style=flat-square)
![Author](https://img.shields.io/badge/author-Kitsana%20Thuekoh-7B61FF?style=flat-square)

</div>

---

## `>_ Overview`

This repository documents the end-to-end deployment of a Windows-based SIEM environment built for **detection engineering**, **threat hunting**, and **incident response simulation**. The pipeline captures enriched host telemetry via Sysmon, ships it through the Splunk Universal Forwarder, and surfaces detection logic across a custom Splunk dashboard — all within an isolated lab network.

Built to enterprise standards. Documented to production quality.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         L A B   A R C H I T E C T U R E                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│   ┌──────────────────────┐        ┌──────────────────────────────────┐  │
│   │   Windows Server     │        │      Splunk Enterprise           │  │
│   │   2022 — WIN-DC01    │        │      Linux VM — SIEM-01          │  │
│   │                      │        │                                  │  │
│   │  ┌────────────────┐  │        │  ┌────────────┐  ┌───────────┐  │  │
│   │  │    Sysmon      │  │  9997  │  │  Indexer   │  │  Search   │  │  │
│   │  │  (telemetry)   │──┼───────►│  │  :9997     │  │  Head     │  │  │
│   │  └────────────────┘  │        │  └────────────┘  └───────────┘  │  │
│   │  ┌────────────────┐  │        │         │               │        │  │
│   │  │  UF (Forwarder)│  │        │  ┌──────▼───────────────▼────┐  │  │
│   │  │  inputs.conf   │  │        │  │   Dashboard / Alerts      │  │  │
│   │  └────────────────┘  │        │  └───────────────────────────┘  │  │
│   │                      │        │                                  │  │
│   │  AD DS · DNS · GPO   │        │  :8000  Web UI                  │  │
│   └──────────────────────┘        └──────────────────────────────────┘  │
│                                                                         │
│   Network: Host-Only  ·  192.168.10.0/24  ·  Domain: lab.local         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## `>_ Tech Stack`

| Layer | Technology | Version |
|-------|-----------|---------|
| Host OS | Windows Server Standard (Desktop Experience) | 2022 |
| Directory Services | Active Directory Domain Services | — |
| Telemetry Agent | Sysinternals Sysmon | v15.x |
| Sysmon Config | SwiftOnSecurity `sysmonconfig.xml` | latest |
| Log Forwarder | Splunk Universal Forwarder | 9.x |
| SIEM Platform | Splunk Enterprise | 9.x |
| Hypervisor | VMware Workstation Pro / VirtualBox | — |

---

## `>_ Detection Coverage`

Events collected, indexed, and mapped to ATT&CK tactics.

| Event ID | Source | Description | ATT&CK Tactic |
|----------|--------|-------------|---------------|
| `4624` | Security | Successful logon | Initial Access |
| `4625` | Security | Failed logon | Credential Access |
| `4648` | Security | Logon with explicit credentials | Lateral Movement |
| `4688` + CLI | Security | Process creation with command line | Execution |
| `4720` | Security | User account created | Persistence |
| `4722` | Security | User account enabled | Persistence |
| `4740` | Security | Account lockout | Credential Access |
| `4719` | Security | Audit policy changed | Defence Evasion |
| `1` | Sysmon | Process creation | Execution |
| `3` | Sysmon | Network connection | C2 / Exfiltration |
| `11` | Sysmon | FileCreate | Persistence |
| `13` | Sysmon | Registry value set | Persistence |
| `22` | Sysmon | DNS query | Discovery |

---

## `>_ Quick Start`

### Prerequisites

- VMware Workstation Pro or VirtualBox
- Windows Server 2022 evaluation ISO
- Splunk Enterprise installer (Linux, `.deb`)
- Splunk Universal Forwarder installer (Windows, `.msi`)
- [Sysmon v15+](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [SwiftOnSecurity sysmonconfig.xml](https://github.com/SwiftOnSecurity/sysmon-config)

---

### 1 · Deploy Windows Server

```powershell
# Post-install: set static IP and hostname
Rename-Computer -NewName "WIN-DC01" -Restart

# Install AD DS role
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSForest `
  -DomainName "lab.local" `
  -DomainNetbiosName "LAB" `
  -ForestMode "WinThreshold" `
  -DomainMode "WinThreshold" `
  -InstallDns `
  -Force
```

### 2 · Deploy Sysmon

```powershell
# Download SwiftOnSecurity config (or supply your own)
Invoke-WebRequest `
  -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" `
  -OutFile "sysmonconfig.xml"

# Install with config
.\sysmon64.exe -accepteula -i sysmonconfig.xml

# Verify
Get-Service Sysmon64
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

### 3 · Configure Audit Policy

```powershell
# Enable command-line logging in process creation events
Set-ItemProperty `
  -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit" `
  -Name "ProcessCreationIncludeCmdLine_Enabled" `
  -Value 1

# Apply advanced audit policy via auditpol
auditpol /set /subcategory:"Logon" /success:enable /failure:enable
auditpol /set /subcategory:"Credential Validation" /success:enable /failure:enable
auditpol /set /subcategory:"Process Creation" /success:enable
auditpol /set /subcategory:"User Account Management" /success:enable /failure:enable
```

### 4 · Configure Splunk Universal Forwarder

```ini
# %SPLUNK_HOME%\etc\system\local\inputs.conf

[WinEventLog://Security]
index = windows_security
disabled = false

[WinEventLog://System]
index = windows_system
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = sysmon
disabled = false
renderXml = true
```

```ini
# %SPLUNK_HOME%\etc\system\local\outputs.conf

[tcpout]
defaultGroup = splunk_indexer

[tcpout:splunk_indexer]
server = 192.168.10.20:9997
```

```powershell
# Restart forwarder to apply
Restart-Service SplunkForwarder
```

### 5 · Validate in Splunk

```spl
# Confirm data is flowing
index=windows_security | stats count by sourcetype

# Failed logon summary
index=windows_security EventCode=4625
| stats count by Account_Name, Source_Network_Address
| sort - count

# Process creation timeline (Sysmon)
index=sysmon EventCode=1
| table _time, User, Image, CommandLine, ParentImage
| sort - _time

# Account creation audit
index=windows_security EventCode=4720
| table _time, Account_Name, SubjectUserName, src_ip
```

---

## `>_ Repository Structure`

```
windows-siem-lab/
├── docs/
│   ├── Windows_SIEM_Lab_Report.docx    # Full technical lab report
│   └── architecture.png                # Lab network diagram
├── configs/
│   ├── sysmonconfig.xml                # Sysmon ruleset (SwiftOnSecurity base)
│   ├── inputs.conf                     # Splunk UF input configuration
│   └── outputs.conf                    # Splunk UF output configuration
├── splunk/
│   ├── dashboards/
│   │   └── siem_overview.xml           # Splunk dashboard XML
│   └── searches/
│       └── detection_queries.spl       # SPL detection library
├── scripts/
│   ├── deploy_sysmon.ps1               # Sysmon install + config script
│   ├── audit_policy.ps1                # Audit policy hardening script
│   └── create_lab_users.ps1            # AD user provisioning script
└── README.md
```

---

## `>_ SPL Detection Library`

A selection of production-ready SPL searches included in this lab.

```spl
──────────────────────────────────────────────
  BRUTE FORCE DETECTION — EID 4625
──────────────────────────────────────────────
index=windows_security EventCode=4625
| bucket _time span=5m
| stats count by _time, Account_Name, Source_Network_Address
| where count > 5
| eval alert="Potential brute force: " + Account_Name

──────────────────────────────────────────────
  LOLBAS EXECUTION — SYSMON EID 1
──────────────────────────────────────────────
index=sysmon EventCode=1
| eval lolbas=if(match(Image,"(?i)(certutil|bitsadmin|mshta|wscript|cscript|regsvr32|rundll32|msiexec|wmic)"),1,0)
| where lolbas=1
| table _time, User, Image, CommandLine, ParentImage

──────────────────────────────────────────────
  PERSISTENCE VIA RUN KEY — SYSMON EID 13
──────────────────────────────────────────────
index=sysmon EventCode=13
| where match(TargetObject,"(?i)\\\\Run\\\\|\\\\RunOnce\\\\")
| table _time, User, TargetObject, Details, Image
```

---

## `>_ Author`

<div align="center">

**Kitsana Thuekoh**
Penetration Tester · Security Researcher

[![CPTS](https://img.shields.io/badge/HTB-CPTS-9FEF00?style=flat-square&logo=hackthebox&logoColor=black)](https://www.hackthebox.com)
[![PenTest+](https://img.shields.io/badge/CompTIA-PenTest%2B-C8202F?style=flat-square&logo=comptia&logoColor=white)](https://www.comptia.org)
[![Security+](https://img.shields.io/badge/CompTIA-Security%2B-C8202F?style=flat-square&logo=comptia&logoColor=white)](https://www.comptia.org)
[![NASA VDP](https://img.shields.io/badge/NASA-VDP%20Recognition-0B3D91?style=flat-square&logo=nasa&logoColor=white)](https://www.nasa.gov)

</div>

---

<div align="center">

```
[ MITRE ATT&CK ]  ·  [ NIST SP 800-53 ]  ·  [ CIS Benchmarks ]
```

*Built for education, detection engineering, and security operations research.*

</div>
