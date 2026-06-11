

# Windows SIEM Lab Phase 5

A production-grade SIEM implementation designed to ingest, parse, and visualize Windows Security Log events (`WinEventLog:Security`) within a localized Splunk Enterprise environment. This project demonstrates active threat hunting, security monitoring, dashboard development, and alerting for critical lifecycle events including user management and authentication telemetry.

##  Architecture & Technical Stack

* **SIEM Platform:** Splunk Enterprise v10.4.0 (Hosted locally on `localhost:8000`)
* **Log Source:** Microsoft Windows Security Auditing (`LogName=Security`)
* **Target Endpoint:** Hostname: `Lenovo` | Domain: `LENOVO`
* **Ingestion Pipeline:** Splunk universal/local forwarder configured for structural WinEventLog data parsing.
* **Data Parsing Mode:** Smart Mode for dynamic field extraction (`Account_Name`, `Logon_Type`, `EventCode`, etc.).

---

## 🔍 Tracked Windows Event Identifiers

The environment explicitly tracks and parses the following Windows Security Log Event IDs (EIDs):

| Event ID | Operational Meaning | Category | Risk Profile |
| --- | --- | --- | --- |
| **4624** | An account was successfully logged on | Authentication | Audit / Baseline |
| **4625** | An account failed to log on | Authentication | Medium - Potential Brute Force / Spraying |
| **4720** | A user account was created | Account Management | High - Privileged Escalation / Persistence |

---

## 📊 Splunk Custom Dashboards & Alerting

### 1. `Successful-Login-Event-Dashboard`

* **Objective:** Visualizes structural success baselines across the ecosystem.
* **Key Fields Extracted:** `Time`, `Account`, `ComputerName`, `IpAddress`, `Logon_Type`, `Authentication_Package`.
* **Observed Baseline:** Captured service account logons (`LENOVO$ / SYSTEM`) utilizing **Logon Type 5** (Service Logon) via the **Negotiate** authentication package.

### 2. `Failed-Login-Dashboard`

* **Objective:** Monitors failed authentication spikes to identify misconfigured services, credential stuffing, or brute force activity.
* **Telemetry Specifics:** Captures sequential failed events with explicit Status Codes (e.g., `Status=0xC000006D` / `Sub Status=0xC0000038`, indicating explicit logon failures/bad passwords) running under **Logon Type 2** (Interactive / Local keyboard logon).

### 3. `Accout-Creation-Alerts` (Custom Alert Rule)

* **Core Query Logic:** Evaluates incoming security logs for `EventCode=4720`.
* **Trigger Condition:** Scheduled hourly at 15 minutes past the hour; triggers immediately when the number of results is $> 0$.
* **Downstream Action:** Generates an automated **Dashboard Studio Snapshot** for the security operations team.

---

## 🚨 Incident Log Analysis & Walkthrough

Based on historical data extracted from active indices (`index="wineventlogs"`), the following high-fidelity events were investigated:

### Local Account Creation Lineage (`EID 4720`)

* **Timestamp:** `2026-06-11 11:24:45.574 AM`
* **Subject (Actor):** `LENOVO\kitsa` (Logon ID: `0x7DEFF`) initiated the creation action.
* **Target (New Account):** A new local user account named **`tester`** was provisioned (`SAM Account Name: tester`).
* **Security Context:** Verified via both raw Splunk JSON/Key-Value payloads and Windows Event Viewer Event ID 4720 logs within `User Account Management`.

```text
Message: A user account was created.
Subject Security ID: S-1-5-21-900484236-3280652421-1342662532-1001
New Account SID:    S-1-5-21-900484236-3280652421-1342662532-1004

```

### Authentication Failure Anomalies (`EID 4625`)

* **Timestamp Cluster:** `2026-06-11 11:52:04 AM` to `11:52:09 AM`
* **Mechanics:** Rapid successive authentication failures logged sequentially within a 5-second window.
* **Logon Mechanism:** `Logon Type: 2` (Local Interactive), indicating manual entry attempts on the device console.
* **Status Codes:** `0xC000006D` (Logon failure), `0xC0000038` (Password expired or account details mismatch).

---

## ⚡ Deployment & Verification Testing

To test the data ingestion pipeline and confirm continuous monitoring capabilities, run the following optimized Splunk Search Processing Language (SPL) queries in the **Search & Reporting** app:

### Query Successful Logons:

```spl
index="wineventlogs" EventCode=4624 
| table _time, Account_Name, ComputerName, Logon_Type, Authentication_Package
| sort - _time

```

### Query Local User Creation Events:

```spl
index="wineventlogs" EventCode=4720
| rename Account_Name as Target_User
| table _time, ComputerName, Target_User, Security_ID

```
