

# Phase 6 – Create Splunk Dashboards

## Objective

To synthesize collected Windows Security Event Logs into structured, actionable visual intelligence. By building dedicated Splunk dashboards, this phase establishes clear visibility into systemic authentication trends, critical administrative changes, and active process executions across monitored endpoints.

---

## Required Dashboards & Implementation Details

### Dashboard 1 – Authentication Activity

This dashboard tracks user access trends, highlighting standard patterns and potential credential abuse or brute-force vectors.

* **Successful Logins**
* *SPL Filter:* `index="wineventlogs" EventCode=4624`
* *Visual Component:* Time-series line chart or a real-time event counter isolating valid authentications.


* **Failed Logins**
* *SPL Filter:* `index="wineventlogs" EventCode=4625`
* *Visual Component:* Single-value gauge and data table logging targeted accounts, failure reasons, and source host names.


* **Locked Accounts**
* *SPL Filter:* `index="wineventlogs" EventCode=4740`
* *Visual Component:* High-priority alert table identifying accounts frozen due to consecutive failed authentication attempts.



#### Interface Preview

The following snapshot captures the baseline telemetry validation for authentication behaviors, capturing critical error states (`EventCode=4625`) mapping back to host endpoints:

---

### Dashboard 2 – Administrative Activity

This dashboard monitors privilege health and directory modifications, helping security teams identify potential insider threats or unauthorized persistence mechanisms.

* **New Accounts**
* *SPL Filter:* ```spl
index="wineventlogs" EventCode=4720
| eval Time=strftime(_time, "%Y-%m-%d %H:%M:%S")
| eval "New User"=mvindex(Account_Name, 1)
| eval "Created By"=mvindex(Account_Name, 0)
| eval Domain=coalesce(Account_Domain, "-")
| table Time, "New User", "Created By", Domain, ComputerName
| sort - Time
```

```




* **Admin Privilege Assignments**
* *SPL Filter:* `index="wineventlogs" EventCode=4672`
* *Visual Component:* Audit table tracking security-sensitive rights assignments issued to system and user accounts.


* **Account Deletions**
* *SPL Filter:* `index="wineventlogs" EventCode=4726`
* *Visual Component:* List of removed directory objects to maintain user lifecycle oversight.



#### Core Administrative Metrics

Below is the structural breakdown of new account generation logs, isolating the creator identity (`kitsa`) and the newly provisioned entity (`testeradmin`):

#### Live Dashboard Deployments

The corresponding deployed dashboards translate raw event queries into polished, executive-ready auditing interfaces:

| New Accounts Overview | Admin Privilege Assignments |
| --- | --- |
|  |  |

---

### Dashboard 3 – Process Activity

Focused on endpoint behaviors, this interface watches for execution vectors, abnormal script invocations, or signs of living-off-the-land attacks.

* **PowerShell Execution & Command Prompt Usage**
* *SPL Filter:* ```spl
index="wineventlogs" EventCode=4688 (Image="*powershell.exe" OR Image="*cmd.exe")
```

```


* *Visual Component:* Dedicated process tracking list detailing the parent processes, specific command-line strings, and execution timestamps.


* **Suspicious Processes**
* *SPL Filter:* Filters tracking executions out of atypical paths (e.g., `AppData\Local\Temp*`, `\Users\Public\*`) or monitoring binary renames and anomalous child processes under web servers.



#### Command & Script Auditing

The underlying execution tracking panels rely on continuous monitoring of process creation events (`EventCode=4688`) to flag CLI usage trends over time:

#### Live Dashboard Deployments

The operational telemetry is split into clear, readable streams for live threat hunting:

| PowerShell Execution Activity | Command Prompt Usage Dashboard |
| --- | --- |
|  |  |

---

## Verifications & Key Observations

1. **Data Normalization:** Using Splunk commands like `mvindex()` ensures that multi-value fields in Windows security logs (like target users vs. responsible actors) are accurately parsed into distinct columns.
2. **Time Synchronization:** Time charts clearly map out when event spikes happen, allowing analysts to correlate bulk administrative additions directly with simultaneous process spikes.
3. **Audit Completeness:** Keeping track of both `SYSTEM` and user-generated privilege assignments helps confirm that background OS tasks and human admin activities are properly separated.
