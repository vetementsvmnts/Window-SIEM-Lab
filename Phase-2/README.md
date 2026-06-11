# Windows SIEM Lab - Phase 2 

This directory houses the technical visual assets documenting **Phase 2** of the Windows SIEM Lab setup. This phase marks the transition from baseline infrastructure deployment to active **Telemetry Generation and Audit Policy Configuration** on the Windows Server 2022 target system.

---

## Active Security Policy & Auditing Configuration

### 1. Local Security Policy Tuning
To ensure the Windows Event Log captures critical security data for ingestion into a SIEM pipeline, the Local Security Policy (`secpol.msc`) has been aggressively configured. Rather than relying on default settings, core advanced auditing policies have been explicitly explicitly set to track system activity.

* **Configured Audit Policies:**
  * **Audit account logon events:** Success, Failure *(Critical for detecting anomalous authentication patterns and brute force attempts)*
  * **Audit account management:** Success, Failure *(Tracks user creation, deletion, and group modifications)*
  * **Audit logon events:** Success, Failure
  * **Audit privilege use:** Success, Failure
  * **Audit process tracking:** Success, Failure *(Captures Event ID 4688 to track process creations, vital for detecting living-off-the-land techniques)*

![Local Security Policy Audit Configuration](Phase-2/Assets/Configuration.png)

---

## Telemetry Generation & Verification

### 2. Event Viewer (Security Log Validation)
Following the policy updates, the Windows Event Viewer (`eventvwr.msc`) was monitored to validate high-fidelity event generation. The security log successfully generated over 3,400 events, confirming that the logging pipeline is active.

* **Key Captured Event Identified:**
  * **Event ID 4720:** *A user account was created.* This directly validates that the user enumeration and provisioning steps from Phase 1 are being actively logged by the system's security auditing sub-system.

![Windows Security Event Log Validation](Phase-2/Assets/Event%20Viewer.png)

---

### 3. Registry Environment Mapping
The Windows Registry Editor (`regedit`) has been initialized to verify baseline system hives (`HKEY_LOCAL_MACHINE`, `HKEY_CURRENT_USER`). This forms the baseline structure needed for upcoming phases, which will involve tracking registry modifications (e.g., Run keys, persistent services) via Sysmon and auditing tools.

![Windows Registry Editor Baseline](Phase-2/Assets/Registry-Editor.png)

---

## File Directory Mapping

```text
Window-SIEM-Lab/
└── Phase-2/
    └── Assets/
        ├── Configuration.png
        ├── Event Viewer.png
        └── Registry-Editor.png
