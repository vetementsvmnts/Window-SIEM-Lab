# Windows SIEM Lab - Phase 1 

This directory contains the visual assets, environment baselines, and configuration screenshots documenting **Phase 1** of the Windows SIEM Lab setup. These images track the foundational deployment of the target machines, local user account configurations, and core operating system services.

---

##  Lab Infrastructure 

### 1. VirtualBox Lab Environment
The virtualization baseline consists of multiple isolated instances deployed for security testing, target simulation, and telemetry ingestion. The core focus for this phase is the active configuration of the Windows Server 2022 instance.

* **Key Specifications:** * **Base Memory:** 8752 MB
  * **Processors:** 2 vCPUs
  * **Network Mode:** NAT (To be migrated to a Host-Only / Internal network interface for isolated telemetry ingestion)

![Lab Environment Baseline](Phase-1/Assets/Lab-Environment.png)

---

##  Target System Configuration

### 2. Windows Server 2022 Initial Provisioning
Initial boot and initialization of the `WinServer2022` target. This view captures the standard administrative initialization, background service verification (`Services.msc`), and the launching of the Server Manager dashboard to prep for Active Directory (ADDS) or Active Directory Certificate Services (ADCS) deployment.

![Windows Server Service Initialization](Phase-1/Assets/Windows-Server-Configuration.png)

---

### 3. Identity & Access Management (IAM) Baseline
To simulate real-world brute-force attacks, privilege escalation vectors, and log generation, distinct local test accounts have been provisioned on the target system.

* **Configured Local Accounts:**
  * `Administrator` (Built-in High-Privilege Account)
  * `tester` (Standard User Context)
  * `tester1` (Standard User Context)

![Target User Accounts](Phase-1/Assets/Different-User-Account.jpg)

---

## File Directory Mapping

```text
Window-SIEM-Lab/
└── Phase-1/
    └── Assets/
        ├── Lab-Environment.png
        ├── Windows-Server-Configuration.png
        └── Different-User-Account.jpg
