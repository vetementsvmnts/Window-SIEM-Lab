# Windows SIEM Lab - Phase 3 

This directory contains the engineering and architectural documentation for **Phase 3** of the Windows SIEM Lab. This phase marks the completion of the local ingestion engine, transitioning the target machine from a standalone target into an active telemetry provider via a local instance of **Splunk Enterprise**.

---

## Splunk Enterprise Core Deployment

### 1. Instance Initialization & Authentication Baseline
To establish centralized log aggregation and parsing capabilities, Splunk Enterprise was successfully deployed on the target infrastructure. The web interface has been exposed locally, validating the integrity of the underlying splunkd daemon.

* **Deployment Details:**
  * **Listener Interface:** `localhost:8000` (Binding verification)
  * **Access Context:** Administrative credential initialization (`admin` context verification)

![Splunk Enterprise Authentication Interface](Phase-3/Assets/Splunk-OnLocalhost.jpg)

---

### 2. SIEM Control Plane Validation
Upon successful authentication, the Splunk web engine initialized into the default Search & Reporting launcher workspace. This environment functions as the central Security Operations Center (SOC) control plane for execution of SPL (Splunk Search Processing Language) hunting queries, alert thresholding, and live telemetry visualization.

![Splunk Enterprise Admin Control Plane](Phase-3/Assets/Splunk.png)

---

## Data Engineering & Data Ingestion Pipeline

### 3. High-Fidelity Event Log Ingestion Mapping
The primary objective of this phase was creating a concrete data ingestion channel mapped directly to the critical event producers tuned during Phase 2. The configuration inputs were meticulously reviewed and staged prior to submission to prevent index cluttering and schema mismatching.

* **Pipeline Ingestion Specifications:**
  * **Input Type:** `Windows Event Logs` (Structured WinEventLog data parsing)
  * **Target Channels:** `Security`, `System`, `Application`
  * **App Context:** `search` (Default processing namespace)
  * **Host Parameter:** `WIN-9L46B0H60LL` (Target machine identifier)
  * **Target Index Destination:** `wineventlogs` *(Explicitly segregated custom index optimized for analytical isolation and custom retention policies)*

![Splunk Ingestion Pipeline Review](Phase-3/Assets/Splunk-Configuration.png)

---

##  File Directory Mapping

```text
Window-SIEM-Lab/
└── Phase-3/
    └── Assets/
        ├── Splunk-OnLocalhost.jpg
        ├── Splunk.png
        └── Splunk-Configuration.png
