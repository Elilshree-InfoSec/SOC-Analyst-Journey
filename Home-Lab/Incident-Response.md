# Security Incident Report

## Suspected Brute Force, Process Execution & Lateral Movement Activity

**Target Hosts:** `LAPTOP-VM8UM21`, `WIN-DC-697`, `WIN-HOST-816`

---

## Incident Metadata

| Field                 | Value                                                           |
| --------------------- | --------------------------------------------------------------- |
| Incident ID           | INC-2026-0816A                                                  |
| Date of Investigation | 16 August 2026                                                  |
| Severity              | Medium                                                          |
| Classification        | False Positive — Legitimate Splunk Universal Forwarder Activity |
| Status                | Investigation Complete — No Compromise Observed                 |
| SIEM Platform         | Splunk                                                          |
| Data Source           | Windows Security Event Logs                                     |
| Sourcetype            | `WinEventLog:Security`                                          |
| Analyst Tier          | SOC Level 1                                                     |
| Target Account        | `reed.fernandez`                                                |

---

# 1. Executive Summary

On 16 August 2026, a Splunk investigation was conducted to analyze suspicious Windows Security events involving failed authentication attempts, process creation, and network share activity across three Windows hosts.

The initial event pattern appeared potentially consistent with a **brute-force attempt followed by process execution and possible lateral movement**. The investigation identified:

* 20 failed authentication events (`Event ID 4625`)
* 15 successful logon events (`Event ID 4624`)
* 71 process creation events (`Event ID 4688`)
* 72 process termination events (`Event ID 4689`)
* 28 NTLM credential validation events (`Event ID 4776`)
* 3 network share access events (`Event IDs 5140/5145`)

All 20 failed authentication events targeted the same account, `reed.fernandez`. No successful authentication for this account was identified within the investigated dataset.

Further analysis of the process creation events showed that the observed executables were associated with the **Splunk Universal Forwarder**, including `splunk-powershell.exe`, `splunk-netmon.exe`, and `splunk-admon.exe`, operating from the expected Splunk installation directory.

The three network share events were also temporally correlated with the observed Splunk activity. No evidence of malicious process execution, successful account compromise, or confirmed lateral movement was identified.

The incident is therefore classified as a **False Positive**, with the observed activity assessed as being associated with legitimate Splunk Universal Forwarder operations and/or an underlying local authentication or configuration issue.

This assessment is limited to the telemetry available within the investigated dataset.

---

# 2. Investigation Chronology and Search Queries

## Phase 1 — Windows Security Event Baseline

**Objective:** Establish the event distribution within the Windows Security dataset and identify event types requiring further investigation.

**Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security"
| stats count by EventCode
| sort - count
```

**Result:**

| Event ID    | Event Description          | Count |
| ----------- | -------------------------- | ----: |
| 4625        | Failed Account Logon       |    20 |
| 4624        | Successful Account Logon   |    15 |
| 4688        | New Process Created        |    71 |
| 4689        | Process Terminated         |    72 |
| 4776        | NTLM Credential Validation |    28 |
| 5140 / 5145 | Network Share Access       |     3 |

**Total Events Reviewed:** 228

**Analyst Note:** Event IDs 4625, 4688, and 5140/5145 were prioritized because their combination can indicate authentication attacks, suspicious execution, or lateral movement depending on the surrounding context.

---

# Phase 2 — Failed Authentication Analysis

**Objective:** Identify the account targeted by the failed authentication events and determine whether the activity was associated with a network source.

**Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security" EventCode=4625
| rex field=_raw "Account Name:\s+(?<TargetUser>\S+)"
| rex field=_raw "Source Network Address:\s+(?<Source_IP>\S+)"
| stats count by TargetUser, Source_IP
```

**Result:**

| Target Account   | Source Address | Failed Attempts |
| ---------------- | -------------- | --------------: |
| `reed.fernandez` | `-`            |              20 |

**Analyst Assessment:**

All 20 failed authentication events targeted the same account, `reed.fernandez`.

The source network address was recorded as `-`, meaning no conventional source IP address was available in the extracted event data.

The repeated targeting of a single account initially warranted investigation for possible brute-force activity. However, the absence of an identifiable external source IP prevented attribution to a remote attacker based on these events alone.

**Initial Classification:** Suspicious authentication activity — further investigation required.

---

# Phase 3 — Successful Authentication Verification

**Objective:** Determine whether the targeted account successfully authenticated following the observed failed attempts.

**Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security"
EventCode=4624 "reed.fernandez"
```

**Result:**

No matching Event ID 4624 events for `reed.fernandez` were identified within the investigated dataset.

**Compromise Assessment:**

No successful authentication for the targeted account was observed.

This indicates that the investigated failed authentication sequence did not result in an observed successful login within the available telemetry.

---

# Phase 4 — Successful Network Logon Review

**Objective:** Investigate the remaining Event ID 4624 events and determine whether they represented suspicious user activity.

**Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security"
EventCode=4624
```

**Result:**

15 successful logon events were identified.

Review of the event details showed network logon activity, including **Logon Type 3**, with some account fields represented as `-`.

**Analyst Assessment:**

Logon Type 3 represents a network logon and may be generated by legitimate system-to-system communication, services, applications, or administrative activity.

The observed events did not independently establish malicious authentication or successful attacker access.

Additional correlation with process activity was therefore performed before reaching a final classification.

---

# Phase 5 — Process Creation Analysis

**Objective:** Investigate the 71 Event ID 4688 process creation events for suspicious executables, scripts, command interpreters, or post-exploitation activity.

**Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security" EventCode=4688
| rex field=_raw "(?i)New Process Name:\s*(?<Process_Name>[^\r\n]+)"
| rex field=_raw "(?i)Process Command Line:\s*(?<Command_Line>[^\r\n]+)"
| table _time, ComputerName, Process_Name, Command_Line
```

**Result:**

The observed process creation events were associated with Splunk Universal Forwarder executables, including:

```text
C:\Program Files\SplunkUniversalForwarder\bin\splunk-powershell.exe
C:\Program Files\SplunkUniversalForwarder\bin\splunk-netmon.exe
C:\Program Files\SplunkUniversalForwarder\bin\splunk-admon.exe
```

The processes were associated with system-level execution and the affected Windows hosts.

**Analyst Assessment:**

The executable names and installation paths were consistent with legitimate Splunk Universal Forwarder components.

The presence of a PowerShell-related executable initially warranted investigation because PowerShell is commonly abused during post-exploitation activity. However, the identified executable was located within the expected Splunk Universal Forwarder installation directory.

No malicious executable outside the expected Splunk installation path was identified during this investigation.

**Finding:** Process creation activity was assessed as **legitimate monitoring activity** rather than confirmed malicious execution.

---

# Phase 6 — Network Share Access Analysis

**Objective:** Investigate the three network share events to determine whether they represented suspicious SMB activity or potential lateral movement.

**Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security"
(EventCode=5140 OR EventCode=5145)
```

**Result:**

Three network share access events were identified.

The events occurred at approximately:

```text
04:59:57 PM
```

The timestamp closely correlated with the observed Splunk Universal Forwarder process activity.

**Analyst Assessment:**

Network share access can be associated with both legitimate Windows operations and attacker lateral movement.

In this case, the available events did not provide sufficient evidence of:

* Unauthorized access to an administrative share
* Malware deployment
* File staging
* Remote execution
* Confirmed movement between systems

The temporal correlation with the legitimate Splunk monitoring activity supported a benign explanation.

**Finding:** Lateral movement was **not confirmed**.

---

# 3. Cross-Event Correlation

The investigation correlated authentication, process, and network activity rather than evaluating the individual events in isolation.

| Evidence              | Observation                                                      | Assessment                                     |
| --------------------- | ---------------------------------------------------------------- | ---------------------------------------------- |
| Event 4625            | 20 failed logons against `reed.fernandez`                        | Suspicious; required investigation             |
| Source Address        | `-`                                                              | No external source IP identified               |
| Event 4624            | No successful login for `reed.fernandez`                         | No observed successful targeted authentication |
| Event 4688            | 71 process creation events                                       | Required process validation                    |
| Process Paths         | Splunk Universal Forwarder directory                             | Consistent with legitimate software            |
| Process Names         | `splunk-powershell.exe`, `splunk-netmon.exe`, `splunk-admon.exe` | Consistent with Splunk components              |
| Event 5140/5145       | 3 network share events                                           | No confirmed malicious access                  |
| Timestamp Correlation | Share activity aligned with Splunk activity                      | Supports benign interpretation                 |

**Overall Finding:** The combined evidence did not demonstrate an attacker progressing from authentication attempts to successful access, malicious execution, or lateral movement.

---

# 4. Indicators and Relevant Artifacts

## Account Indicator

| Type           | Value            |
| -------------- | ---------------- |
| Target Account | `reed.fernandez` |

## Host Indicators

| Type | Value            |
| ---- | ---------------- |
| Host | `LAPTOP-VM8UM21` |
| Host | `WIN-DC-697`     |
| Host | `WIN-HOST-816`   |

## Process Indicators

| Process                 | Observed Path                                    |
| ----------------------- | ------------------------------------------------ |
| `splunk-powershell.exe` | `C:\Program Files\SplunkUniversalForwarder\bin\` |
| `splunk-netmon.exe`     | `C:\Program Files\SplunkUniversalForwarder\bin\` |
| `splunk-admon.exe`      | `C:\Program Files\SplunkUniversalForwarder\bin\` |

## Behavioral Indicators

| Indicator                 | Value                 |
| ------------------------- | --------------------- |
| Failed Authentication     | Event ID 4625         |
| Successful Authentication | Event ID 4624         |
| Process Creation          | Event ID 4688         |
| NTLM Validation           | Event ID 4776         |
| Network Share Access      | Event IDs 5140 / 5145 |
| Final Classification      | False Positive        |

---

# 5. MITRE ATT&CK Mapping

The following ATT&CK techniques were considered because the observed telemetry resembled behaviors commonly associated with these techniques.

Importantly, the mapping represents **investigative context and suspected behavior**, not confirmation that an attacker successfully performed the techniques.

| Field             | Mapping                                               |
| ----------------- | ----------------------------------------------------- |
| Tactic            | Credential Access (TA0006)                            |
| Technique         | T1110 — Brute Force                                   |
| Relevant Evidence | 20 failed authentication attempts against one account |
| Assessment        | Not confirmed as malicious                            |

| Field             | Mapping                                                       |
| ----------------- | ------------------------------------------------------------- |
| Tactic            | Execution (TA0002)                                            |
| Technique         | T1059 — Command and Scripting Interpreter                     |
| Relevant Evidence | `splunk-powershell.exe` process activity                      |
| Assessment        | Legitimate Splunk activity; malicious execution not confirmed |

| Field             | Mapping                                                         |
| ----------------- | --------------------------------------------------------------- |
| Tactic            | Lateral Movement (TA0008)                                       |
| Technique         | T1021.002 — SMB/Windows Admin Shares                            |
| Relevant Evidence | Event IDs 5140/5145                                             |
| Assessment        | Network share activity observed; lateral movement not confirmed |

| Field             | Mapping                                                                    |
| ----------------- | -------------------------------------------------------------------------- |
| Tactic            | Discovery (TA0007)                                                         |
| Technique         | T1016 — System Network Configuration Discovery                             |
| Relevant Evidence | `splunk-netmon.exe` activity                                               |
| Assessment        | Consistent with monitoring functionality; attacker discovery not confirmed |

### MITRE Assessment

The initial telemetry contained indicators that could resemble several MITRE ATT&CK techniques.

However, correlation with executable paths, execution context, timestamps, and authentication results indicated that the observed activity was associated with legitimate monitoring operations.

Therefore, **no malicious MITRE ATT&CK technique is confirmed for this incident**.

---

# 6. Recommended Actions

## 6.1 Account / Configuration Review

Review the `reed.fernandez` account and associated services or processes to determine the cause of the repeated authentication failures.

Areas to verify include:

* Account status
* Password or credential configuration
* Services using the account
* Scheduled tasks
* Splunk inputs or monitoring configurations
* Stored credentials associated with the account

---

## 6.2 Splunk Universal Forwarder Validation

Verify that the observed Splunk Universal Forwarder installation is authorized and expected on the affected hosts.

Recommended validation includes:

* Confirming the installation source
* Verifying the Splunk service configuration
* Reviewing the configured inputs
* Confirming the service account
* Checking for unexpected modifications to the Splunk installation directory

---

## 6.3 SIEM Detection Tuning

Review existing brute-force and suspicious process detections to determine whether legitimate Splunk-generated events are contributing to alert noise.

Detection tuning should **not** simply exclude all Splunk activity.

Instead, consider contextual conditions such as:

* Expected executable path
* Parent process
* Service account
* Host identity
* Source address
* Authentication type
* Process command line
* Frequency and timing

This reduces false positives while preserving visibility if Splunk-related processes are ever abused.

---

## 6.4 Continued Monitoring

Continue monitoring the affected hosts for:

* Successful authentication involving `reed.fernandez`
* Authentication from unexpected external IP addresses
* New or unusual process execution
* PowerShell execution outside approved Splunk paths
* Suspicious SMB activity
* Unexpected administrative share access
* Process execution from temporary or user-writable directories

Escalate to SOC Tier 2 / Incident Response if future telemetry demonstrates successful authentication, malicious process execution, credential abuse, or confirmed lateral movement.

---

# 7. Final SOC Assessment

| Item                                 | Result                                                            |
| ------------------------------------ | ----------------------------------------------------------------- |
| Incident Type                        | Suspected Brute Force / Process Execution / Lateral Movement      |
| Total Events Reviewed                | 228                                                               |
| Failed Authentication Events         | 20                                                                |
| Targeted Account                     | `reed.fernandez`                                                  |
| Successful Authentication for Target | None observed                                                     |
| Process Creation Events              | 71                                                                |
| Identified Process Family            | Splunk Universal Forwarder                                        |
| Network Share Events                 | 3                                                                 |
| Confirmed Lateral Movement           | No                                                                |
| Confirmed Malicious Execution        | No                                                                |
| Confirmed Account Compromise         | No                                                                |
| Final Classification                 | **False Positive**                                                |
| Investigation Status                 | **Complete**                                                      |
| Recommended Follow-Up                | Account/configuration review, Splunk validation, detection tuning |

---

# 8. Analyst Conclusion

The investigation identified a cluster of Windows Security events that initially resembled a brute-force attack followed by process execution and potential lateral movement.

A detailed review of the authentication, process creation, and network share telemetry found no evidence of successful authentication for the targeted account and no confirmed malicious process execution or lateral movement.

The 71 process creation events were associated with legitimate **Splunk Universal Forwarder** components operating from the expected installation directory. The network share events also correlated temporally with the observed monitoring activity.

The repeated authentication failures against `reed.fernandez` remain an operational issue requiring review; however, the available evidence does not demonstrate that they were caused by an external attacker.

The incident is therefore classified as:

> **FALSE POSITIVE — LEGITIMATE MONITORING ACTIVITY**

No active compromise was observed within the scope of the available telemetry.

---

# 9. Investigation Limitations

This assessment is based solely on the Windows Security events available within the investigated Splunk dataset.

The absence of malicious activity in the available logs does not independently prove that the affected systems were never compromised through another telemetry source, time period, or attack vector.

Where available, further validation could include:

* Endpoint Detection and Response (EDR) telemetry
* Windows Sysmon logs
* PowerShell operational logs
* Splunk Universal Forwarder service logs
* Windows Task Scheduler logs
* Windows service configuration
* Active Directory authentication logs
* Network traffic and firewall logs
* SMB/network share audit logs
* File integrity or endpoint telemetry

These additional sources would provide greater visibility into the root cause of the authentication failures and further validate the legitimacy of the observed Splunk processes.

---

# 10. MITRE ATT&CK Summary

```text
Initial Suspicious Activity
        |
        v
20 Failed Authentication Attempts
        |
        v
T1110 - Brute Force
        |
        v
No Successful Authentication Observed
        |
        +--------------------------+
        |                          |
        v                          v
71 Process Creations          3 Network Share Events
        |                          |
        v                          v
T1059 Considered              T1021.002 Considered
        |                          |
        v                          v
Splunk Universal Forwarder   No Confirmed Lateral Movement
        |
        v
Legitimate Monitoring Activity
        |
        v
FALSE POSITIVE
```

**Final Classification:**
**False Positive — No Confirmed Compromise**

**MITRE ATT&CK techniques considered during investigation:** `T1110`, `T1059`, `T1021.002`, `T1016`

**Confirmed malicious technique:** None
