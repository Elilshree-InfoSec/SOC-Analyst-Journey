# Security Incident Report

## Suspected Brute Force & Lateral Movement Activity

**Target Hosts:** `LAPTOP-VM8UM21`, `WIN-DC-697`, `WIN-HOST-816`

---

## Incident Metadata

| Field                 | Value                                           |
| --------------------- | ----------------------------------------------- |
| Incident ID           | `INC-2026-0816A`                                |
| Date of Investigation | 16 August 2026                                  |
| Severity              | Medium                                          |
| Classification        | False Positive                                  |
| Status                | Investigation Complete — No Compromise Observed |
| SIEM Platform         | Splunk                                          |
| Data Source           | Windows Security Logs                           |
| Sourcetype            | `WinEventLog:Security`                          |
| Analyst Tier          | SOC Level 1                                     |

---

# 1. Executive Summary

An investigation was initiated after Windows Security logs showed repeated failed authentication attempts, process creation activity, and network share events.

### Key Findings

| Indicator                   |            Finding |
| --------------------------- | -----------------: |
| Failed Logons               |                 20 |
| Target Account              |   `reed.fernandez` |
| Successful Login for Target |      None observed |
| Process Creation            |          71 events |
| Network Share Events        |                  3 |
| Splunk Processes            |           Observed |
| Malicious Process           |    None identified |
| Lateral Movement            |      Not confirmed |
| Account Compromise          |       Not observed |
| Final Classification        | **False Positive** |

The observed activity was consistent with legitimate **Splunk Universal Forwarder** activity and/or an associated system configuration issue. No evidence of successful account compromise or confirmed lateral movement was identified within the available telemetry.

---

# 2. Investigation

## Phase 1 — Event Baseline

**Objective:** Identify the Windows Security events present in the dataset.

**SPL Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security"
| stats count by EventCode
```

**Result:**

| Event ID  | Description                | Count |
| --------- | -------------------------- | ----: |
| 4625      | Failed Logon               |    20 |
| 4624      | Successful Logon           |    15 |
| 4688      | Process Creation           |    71 |
| 4689      | Process Termination        |    72 |
| 4776      | NTLM Credential Validation |    28 |
| 5140/5145 | Network Share Access       |     3 |

**Total Events:** 228

**Assessment:**

Events 4625, 4688, and 5140/5145 were selected for further investigation.

---

## Phase 2 — Failed Authentication Analysis

**Objective:** Identify the targeted account, source address, and number of failed authentication attempts.

**SPL Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security" EventCode=4625
| rex field=_raw "Account Name:\s+(?<TargetUser>\S+)"
| rex field=_raw "Source Network Address:\s+(?<Source_IP>\S+)"
| stats count by TargetUser Source_IP
```

**Result:**

| Target Account   | Source Address | Failed Attempts |
| ---------------- | -------------- | --------------: |
| `reed.fernandez` | `-`            |              20 |

**Assessment:**

All 20 failed authentication events targeted `reed.fernandez`.

The source network address was recorded as `-`. No external source IP was identified in the available Event 4625 records.

The repeated failures initially indicated possible brute-force activity, requiring a successful authentication check.

---

## Phase 3 — Successful Authentication Check

**Objective:** Determine whether the targeted account successfully authenticated.

**SPL Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security" EventCode=4624 reed.fernandez
```

**Result:**

**0 events returned.**

**Assessment:**

No successful authentication for `reed.fernandez` was observed in the investigated dataset.

**Compromise:** Not observed.

---

## Phase 4 — Process Creation Analysis

**Objective:** Determine whether the 71 process creation events represented malicious execution.

**SPL Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security" EventCode=4688
| rex field=_raw "(?i)New Process Name:\s*(?<Process_Name>[^\r\n]+)"
| stats count by Process_Name
| sort - count
```

**Result:**

Observed processes included:

* `splunk-powershell.exe`
* `splunk-netmon.exe`
* `splunk-admon.exe`

Observed installation path:

```text
C:\Program Files\SplunkUniversalForwarder\bin\
```

**Assessment:**

The observed processes and installation path were consistent with Splunk Universal Forwarder components.

No suspicious executable outside the expected Splunk installation path was identified during the review.

**Finding:** No malicious process execution confirmed.

---

## Phase 5 — Network Share Analysis

**Objective:** Determine whether network share activity indicated possible lateral movement.

**SPL Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security" (EventCode=5140 OR EventCode=5145)
| stats count by EventCode ComputerName
```

**Result:**

* Network share events: **3**
* Activity occurred at approximately **04:59:57 PM**
* Events correlated with the period of Splunk process activity.

**Assessment:**

Network share activity was observed, but the available evidence did not demonstrate:

* Unauthorized administrative share access
* Remote execution
* Malware deployment
* File staging
* Confirmed movement between hosts

**Finding:** Lateral movement not confirmed.

---

# 3. Investigation Correlation

| Evidence         | Finding                                     | Assessment                          |
| ---------------- | ------------------------------------------- | ----------------------------------- |
| Event 4625       | 20 failed logons                            | Suspicious initially                |
| Target Account   | `reed.fernandez`                            | Single account                      |
| Source Address   | `-`                                         | No external IP identified           |
| Event 4624       | No successful login for target              | No compromise observed              |
| Event 4688       | 71 process creations                        | Investigated                        |
| Process Names    | Splunk components                           | Consistent with legitimate activity |
| Process Path     | Splunk installation directory               | Expected                            |
| Event 5140/5145  | 3 share events                              | No confirmed lateral movement       |
| Time Correlation | Share activity aligned with Splunk activity | Supports benign explanation         |

### Investigation Flow

```text
Failed Authentication
        ↓
Possible Brute Force
        ↓
Successful Login Check
        ↓
No Successful Login
        ↓
Process Investigation
        ↓
Legitimate Splunk Processes
        ↓
Network Share Investigation
        ↓
No Confirmed Lateral Movement
        ↓
FALSE POSITIVE
```

---

# 4. Indicators of Interest

### Account

| Type           | Value            |
| -------------- | ---------------- |
| Target Account | `reed.fernandez` |

### Hosts

| Host             |
| ---------------- |
| `LAPTOP-VM8UM21` |
| `WIN-DC-697`     |
| `WIN-HOST-816`   |

### Processes

| Process                 | Path                                             |
| ----------------------- | ------------------------------------------------ |
| `splunk-powershell.exe` | `C:\Program Files\SplunkUniversalForwarder\bin\` |
| `splunk-netmon.exe`     | `C:\Program Files\SplunkUniversalForwarder\bin\` |
| `splunk-admon.exe`      | `C:\Program Files\SplunkUniversalForwarder\bin\` |

---

# 5. Recommended Actions

### Account / System Review

* Review the `reed.fernandez` account configuration.
* Identify the service, scheduled task, or process generating the authentication failures.
* Verify whether the account is active and expected on the affected hosts.

### Splunk Validation

* Verify that Splunk Universal Forwarder is authorized on the affected hosts.
* Review the Forwarder configuration and service account.
* Confirm that the observed binaries are legitimate.

### SIEM Tuning

* Review the current brute-force detection logic.
* Determine whether legitimate Splunk activity is contributing to the alert.
* Use contextual conditions such as host, account, process path, and source address to reduce false positives.

### Monitoring

Continue monitoring for:

* Successful authentication involving `reed.fernandez`
* External source IP addresses
* Suspicious PowerShell activity
* Unexpected process execution
* Unusual SMB activity
* Administrative share access

Escalate to SOC Tier 2 if successful authentication, malicious execution, credential abuse, or confirmed lateral movement is observed.

---

# 6. Final SOC Assessment

| Item                         | Result                                   |
| ---------------------------- | ---------------------------------------- |
| Incident Type                | Suspected Brute Force / Lateral Movement |
| Failed Authentication Events | 20                                       |
| Targeted Account             | `reed.fernandez`                         |
| Successful Authentication    | **None observed**                        |
| Process Creation Events      | 71                                       |
| Splunk Processes Identified  | **Yes**                                  |
| Network Share Events         | 3                                        |
| Malicious Process Identified | **No**                                   |
| Lateral Movement Confirmed   | **No**                                   |
| Account Compromise Observed  | **No**                                   |
| Final Classification         | **False Positive**                       |
| Investigation Status         | **Complete**                             |

---

# 7. Analyst Conclusion

The investigation identified **20 failed authentication attempts** against `reed.fernandez`. No successful authentication for the account was observed.

Analysis of the 71 process creation events identified legitimate Splunk Universal Forwarder components operating from the expected installation directory. The three network share events did not provide sufficient evidence of malicious SMB activity or lateral movement.

Based on the available Windows Security telemetry, the activity is assessed as **benign Splunk-related activity rather than a confirmed security compromise**.

**Final Disposition: FALSE POSITIVE — NO CONFIRMED COMPROMISE**

---

# 8. Investigation Limitations

This assessment is based on the Windows Security telemetry available in the investigated Splunk dataset.

The dataset did not contain **Sysmon, PowerShell Script Block Logging, EDR, dedicated network telemetry, or additional Active Directory audit logs**. Therefore, process execution, PowerShell activity, network connections, and domain authentication activity could not be independently correlated beyond the available Windows Security events.

The absence of malicious activity in this dataset does not independently prove that no compromise occurred through another source, time period, or attack vector.

### Additional Telemetry for Future Investigation

The following data sources could provide greater visibility and enable deeper correlation:

* **Sysmon** — Detailed process, network, DNS, file, and process activity
* **PowerShell Logs** — Script Block Logging and command execution details
* **EDR Telemetry** — Endpoint process and behavioral activity
* **Active Directory Logs** — Domain authentication and Kerberos activity
* **Splunk Universal Forwarder Logs** — Endpoint collection and forwarding activity
* **Firewall / Network Logs** — Network connections and traffic patterns
* **SMB Audit Logs** — File and administrative share activity
* **Windows Service & Scheduled Task Logs** — Potential persistence mechanisms

> **Note:** These telemetry sources were not available in the current dataset and were therefore not used as evidence for the final incident classification.


---

# 9. MITRE ATT&CK Mapping

The following techniques were considered during triage based on the observed event patterns.

| Tactic            | Technique                                          | Evidence                             | Assessment                     |
| ----------------- | -------------------------------------------------- | ------------------------------------ | ------------------------------ |
| Credential Access | **T1110 — Brute Force**                            | 20 failed logons against one account | Not confirmed malicious        |
| Execution         | **T1059 — Command and Scripting Interpreter**      | `splunk-powershell.exe` observed     | Legitimate Splunk activity     |
| Lateral Movement  | **T1021.002 — SMB/Windows Admin Shares**           | Events 5140/5145                     | Lateral movement not confirmed |
| Discovery         | **T1016 — System Network Configuration Discovery** | `splunk-netmon.exe` observed         | Consistent with monitoring     |

### MITRE Assessment

The observed events initially resembled behaviors associated with several MITRE ATT&CK techniques. However, the investigation did not identify sufficient evidence to classify any of these techniques as **confirmed malicious activity**.

**Confirmed malicious ATT&CK technique: None.**

---

# Final SOC L1 Verdict

| Category           | Verdict                                                        |
| ------------------ | -------------------------------------------------------------- |
| Classification     | **FALSE POSITIVE**                                             |
| Compromise         | **Not observed**                                               |
| Brute Force        | **Not confirmed malicious**                                    |
| Lateral Movement   | **Not confirmed**                                              |
| Primary Finding    | **Legitimate Splunk Universal Forwarder activity**             |
| Recommended Action | **Close incident + review authentication/configuration issue** |

**Case Status: CLOSED — FALSE POSITIVE**

