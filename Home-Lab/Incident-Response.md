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

An investigation was initiated after Windows Security logs showed repeated failed authentication attempts together with process creation and network share activity.

### Key Findings

| Indicator                   | Finding            |
| --------------------------- | ------------------ |
| Failed Logons               | 20                 |
| Target Account              | `reed.fernandez`   |
| Successful Login for Target | None observed      |
| Process Creation            | 71 events          |
| Network Share Events        | 3                  |
| Suspicious Processes        | None identified    |
| Splunk Processes            | Observed           |
| Confirmed Lateral Movement  | No                 |
| Confirmed Compromise        | No                 |
| Final Classification        | **False Positive** |

The activity was assessed as consistent with **legitimate Splunk Universal Forwarder activity and/or an associated system configuration issue**. No evidence of successful account compromise or confirmed lateral movement was identified within the available telemetry.

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
Event IDs 4625, 4688 and 5140/5145 were prioritized for further investigation.

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
All 20 failed authentication events targeted the same account, `reed.fernandez`. The source network address was recorded as `-`, with no external source IP identified in the available events.

The repeated attempts against a single account initially indicated possible brute-force activity. However, the available Event 4625 data did not provide evidence of a remote source.

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
```

**Result:**

The process events included:

* `splunk-powershell.exe`
* `splunk-netmon.exe`
* `splunk-admon.exe`

Observed installation path:

```text
C:\Program Files\SplunkUniversalForwarder\bin\
```

**Assessment:**
The processes and installation path were consistent with **Splunk Universal Forwarder** components.

No suspicious executable outside the expected Splunk directory was identified during the review.

The presence of `splunk-powershell.exe` was investigated because PowerShell-related execution can be abused by attackers. However, the identified binary was associated with the legitimate Splunk installation.

**Finding:** No malicious process execution confirmed.

---

## Phase 5 — Network Share Analysis

**Objective:** Determine whether the network share activity represented possible lateral movement.

**SPL Query:**

```spl
index="wineventlog" sourcetype="WinEventLog:Security" EventCode=5140 OR EventCode=5145
```

**Result:**

* Network share events: **3**
* Observed timestamp: **04:59:57 PM**
* Activity occurred during the same period as the Splunk process activity.

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

| Evidence         | Finding                                     | Assessment                    |
| ---------------- | ------------------------------------------- | ----------------------------- |
| 4625             | 20 failed logons                            | Suspicious initially          |
| Target           | `reed.fernandez`                            | Single account                |
| Source IP        | `-`                                         | No external IP identified     |
| 4624             | No successful login for target              | No compromise observed        |
| 4688             | 71 process creations                        | Investigated                  |
| Process Names    | Splunk components                           | Legitimate activity           |
| Process Path     | Splunk installation directory               | Expected                      |
| 5140/5145        | 3 share events                              | No confirmed lateral movement |
| Time Correlation | Share activity aligned with Splunk activity | Supports benign explanation   |

### Overall Assessment

The events initially resembled a potential attack chain:

```text
Failed Authentication
        ↓
Possible Brute Force
        ↓
Process Creation
        ↓
Possible Command Execution
        ↓
Network Share Activity
        ↓
Possible Lateral Movement
```

However, investigation did not identify the expected evidence of successful compromise.

Instead:

```text
Failed Authentication
        ↓
No Successful Login
        ↓
Legitimate Splunk Processes Identified
        ↓
Network Share Activity Correlated
        ↓
No Malicious Execution
        ↓
No Confirmed Lateral Movement
        ↓
FALSE POSITIVE
```

---

# 4. Indicators of Interest

## Account

| Type           | Value            |
| -------------- | ---------------- |
| Target Account | `reed.fernandez` |

## Hosts

| Host             |
| ---------------- |
| `LAPTOP-VM8UM21` |
| `WIN-DC-697`     |
| `WIN-HOST-816`   |

## Processes

| Process                 | Path                                             |
| ----------------------- | ------------------------------------------------ |
| `splunk-powershell.exe` | `C:\Program Files\SplunkUniversalForwarder\bin\` |
| `splunk-netmon.exe`     | `C:\Program Files\SplunkUniversalForwarder\bin\` |
| `splunk-admon.exe`      | `C:\Program Files\SplunkUniversalForwarder\bin\` |

---

# 5. Recommended Actions

### Account / System Review

* Review the `reed.fernandez` account configuration.
* Identify the service, task, or process generating the repeated authentication failures.
* Verify whether the account is active and expected on the affected hosts.

### Splunk Validation

* Verify that Splunk Universal Forwarder is authorized on all affected hosts.
* Review the Forwarder configuration and service account.
* Confirm that the observed binaries are legitimate.

### SIEM Tuning

* Review the current brute-force detection logic.
* Investigate whether legitimate Splunk activity is contributing to repeated false positives.
* Use contextual conditions such as host, process path, account, and source address rather than simply excluding all Splunk activity.

### Monitoring

Continue monitoring for:

* Successful authentication involving `reed.fernandez`
* External source IPs
* Suspicious PowerShell activity
* Unexpected process execution
* Unusual SMB activity
* Administrative share access

Escalate to SOC Tier 2 if evidence of successful authentication, malicious execution, credential abuse, or confirmed lateral movement appears.

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
| Account Compromise Confirmed | **No**                                   |
| Final Classification         | **False Positive**                       |
| Investigation Status         | **Complete**                             |

---

# 7. Analyst Conclusion

The investigation identified 20 failed authentication attempts against `reed.fernandez`, followed by process creation and network share activity.

No successful authentication for the targeted account was observed. The process investigation identified Splunk Universal Forwarder components operating from the expected installation directory, and the network share activity did not provide sufficient evidence of malicious lateral movement.

Based on the available telemetry, the activity is assessed as **legitimate Splunk-related activity rather than an active compromise**.

**Final Disposition: FALSE POSITIVE — NO CONFIRMED COMPROMISE**

---

# 8. Investigation Limitations

This assessment is based on the Windows Security telemetry available in the investigated Splunk dataset.

The absence of malicious activity in this dataset does not prove that no compromise occurred through another source, time period, or attack vector.

Additional telemetry that could be reviewed includes:

* Sysmon
* EDR
* PowerShell logs
* Active Directory authentication logs
* Splunk Universal Forwarder logs
* Windows service and scheduled task logs
* Firewall/network logs
* SMB audit logs

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

The observed events initially resembled behaviors associated with multiple ATT&CK techniques. However, the supporting telemetry did not demonstrate malicious execution or successful attacker activity.

**Confirmed malicious ATT&CK technique: None.**

---

## Final SOC L1 Verdict

**Classification:** False Positive
**Compromise:** Not observed
**Lateral Movement:** Not confirmed
**Primary Finding:** Legitimate Splunk Universal Forwarder activity
**Action:** Close incident and review the underlying authentication/configuration issue.
