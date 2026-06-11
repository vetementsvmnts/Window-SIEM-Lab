 Windows SIEM Lab - Phase 4 

This directory contains the engineering documentation, configuration parameters, and telemetry validation assets for **Phase 4** of the Windows SIEM Lab. This phase focuses on extending detection visibility beyond baseline Windows Security Logs by deploying **Microsoft Sysinternals System Monitor (Sysmon)** and validating multi-source, high-fidelity log ingestion into the centralized Splunk SIEM control plane.

---

## 🛠️ Sysmon Deployment & Configuration Hardening

### 1. Hardened Rule Customization via Schema 4.91
To capture advanced execution context while mitigating log noise, Sysmon64 was provisioned using an enterprise-graded configuration schema modeled after the SwiftOnSecurity telemetry framework. 

* **Execution Directives:**
  * **Binary:** `Sysmon64.exe` (x64 Native Engine)
  * **Directives:** `-accepteula -i sysmonconfig-export.xml`
  * **Driver State:** Successful initialization and binding of the `SysmonDrv` minifilter driver to intercept low-level kernel abstractions.

![Sysmon 64 Engine Deployment](Phase-4/Assets/Sysmon-Configuration.png)

---

### 2. Operational Verification & Core Process Monitoring
Following deployment, daemon survival and operational integrity were validated via an administrative PowerShell context to confirm the active state of the background telemetry provider.

```powershell
Get-Service Sysmon64

```

Immediate verification of the `Microsoft-Windows-Sysmon/Operational` channel confirmed the generation of structural Event IDs (such as **Event ID 1: Process Creation**), successfully tracking target process lineage, command-line parameters, file hashes, and parent-child relationships.

---

##  Telemetry Engineering & Splunk Ingestion Mapping

### 3. Splunk Ingestion Pipeline Optimization

To ingest the newly exposed kernel-level telemetry, the Splunk Enterprise ingestion daemon (`splunkd`) was restarted via PowerShell to re-parse the application namespaces.

The ingestion pipeline was expanded to swallow data from the `Microsoft-Windows-Sysmon/Operational` EVTX path, normalizing it cleanly into the standardized **`XmlWinEventLog`** sourcetype format.

---

### 4. Enterprise Data Summary Validation

A post-ingestion query against the control plane validates data distribution across the network. The SIEM metrics confirm comprehensive collection across distinct system horizons:

* **Target Host Identity:** `WIN-9L46B0H60LL`
* **Total Aggregated Log Count:** >40,200 events successfully parsed into index buffers.

| Source Channel | Indexed Event Count | Data Type Horizon |
| --- | --- | --- |
| `XmlWinEventLog:Security` | 39,321 | Identity, Authentication & Audit Policy Telemetry |
| `XmlWinEventLog:Microsoft-Windows-Windows Defender/Operational` | 366 | Local Endpoint Anti-Malware Signal Trajectory |
| `XmlWinEventLog:System` | 524 | Core OS Infrastructure & Kernel State Shifts |
| `XmlWinEventLog:Application` | 9 | Userspace Software Context Events |

---

## 📂 File Directory Mapping

```text
Window-SIEM-Lab/
└── Phase-4/
    └── Assets/
        ├── Sysmon-Configuration.png
        ├── Sysmon-EventViewer.png
        ├── Sysmon-Powershell.png
        └── DataSummary-Eventlogs.png

```

> **Detection Engineering Note:** Deploying Sysmon with a modular, signature-driven XML configuration ensures visibility into sophisticated adversary actions like dynamic API resolution, process injection (`Event ID 8`), and raw disk access attempts (`Event ID 9`). This provides the necessary depth to construct robust threat hunting models using the MITRE ATT&CK framework.

```

```
