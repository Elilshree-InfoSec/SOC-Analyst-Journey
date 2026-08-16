# Security Incident Report

## B2B Identity Threat: Coordinated Password Spray Activity

**Source IPs:** `73.15.72.101`, `66.176.252.11`

---

## Incident Metadata

| Field | Value |
|---|---|
| Incident ID | INC-2026-0812A |
| Date of Investigation | 12 August 2026 |
| Severity | Medium |
| Classification | Password Spraying (MITRE ATT&CK T1110.003) |
| Status | Investigation Complete — No Compromise Observed |
| SIEM Platform | Splunk |
| Data Source | BruteForce_json |
| Analyst Tier | SOC Level 1 |

---

## 1. Executive Summary

On 12 August 2026, a Splunk investigation was conducted to analyze authentication anomalies within the `BruteForce_json` dataset following an alert on repeated login failures.

The investigation identified 35 failed authentication events originating from two distinct source IP addresses:

| Source IP | Failed Attempts |
|---|---|
| 73.15.72.101 | 29 |
| 66.176.252.11 | 6 |

The activity targeted five distinct user accounts. All 35 observed authentication attempts resulted in failure.

Based on the observed pattern of authentication attempts distributed across multiple user accounts rather than repeated attempts against a single account, the activity was classified as Password Spraying, corresponding to MITRE ATT&CK technique T1110.003.

No successful authentication associated with the investigated source IP addresses was observed in the available dataset. Accordingly, no account compromise has been confirmed based on the available evidence.

---

## 2. Investigation Chronology and Search Queries

### Phase 1 — Environment Audit and Data Inventory

**Objective:** Establish the available data footprint prior to investigation, in accordance with SOC methodology for unfamiliar or undocumented environments.

**Query:**

```spl
| metadata type=sourcetypes
```

**Result:**

| Sourcetype | Event Count |
|---|---|
| BruteForce_json | 35 |
| fraud_detection.csv | 200 |

**Analyst Note:** Investigation scope was narrowed to `BruteForce_json`, as this sourcetype aligned with the reported authentication-anomaly alert. `fraud_detection.csv` was excluded as out of scope for this investigation.

---

### Phase 2 — Source IP Quantification

**Objective:** Identify and quantify the source IP addresses responsible for failed authentication events.

**Query:**

```spl
sourcetype="BruteForce_json" Operation="UserLoginFailed"
| stats count by ActorIpAddress
```

**Result:**

| Source IP | Failed Authentication Count |
|---|---|
| 73.15.72.101 | 29 |
| 66.176.252.11 | 6 |

**Total Failed Authentication Events:** 35

---

### Phase 3 — Attack Pattern Classification and Target Analysis

**Objective:** Determine whether the observed activity constitutes brute-force attempts against a single account or password-spraying attempts against multiple accounts.

**Query:**

```spl
sourcetype="BruteForce_json" Operation="UserLoginFailed"
| stats count dc(UserId) as unique_targets values(UserId) as targeted_users by ActorIpAddress
```

**Result:** Activity was directed against 5 unique user accounts:

- bpatel@rodsoto.onmicrosoft.com
- jhernan@rodsoto.onmicrosoft.com
- mhaag@rodsoto.onmicrosoft.com
- pbareiss@rodsoto.onmicrosoft.com
- rodsoto@rodsoto.onmicrosoft.com

**Classification:** Password Spraying — MITRE ATT&CK T1110.003

**Analyst Rationale:** The distribution of authentication attempts across multiple distinct accounts, rather than concentrated attempts against a single account, is consistent with password-spraying behavior as opposed to a traditional brute-force technique (T1110.001).

---

### Phase 4 — Compromise Verification

**Objective:** Determine whether any successful authentication occurred from the identified source IP addresses.

**Query:**

```spl
sourcetype="BruteForce_json" (ActorIpAddress="73.15.72.101" OR ActorIpAddress="66.176.252.11")
| stats count by ActorIpAddress, Operation
```

**Result:** All 35 authentication events associated with the two source IP addresses were logged as `UserLoginFailed`. No `UserLoginSucceeded` or equivalent success operation was observed for either source IP.

**Compromise Assessment:** No confirmed account compromise. This assessment is limited to the authentication events available within the analyzed dataset.

---

## 3. Indicators of Compromise (IOCs)

### Network Indicators

| Type | Value |
|---|---|
| Source IP | 73.15.72.101 |
| Source IP | 66.176.252.11 |

### Account Indicators

| Targeted Account |
|---|
| bpatel@rodsoto.onmicrosoft.com |
| jhernan@rodsoto.onmicrosoft.com |
| mhaag@rodsoto.onmicrosoft.com |
| pbareiss@rodsoto.onmicrosoft.com |
| rodsoto@rodsoto.onmicrosoft.com |

### Behavioral Indicators

| Indicator | Value |
|---|---|
| Operation | UserLoginFailed |
| MITRE ATT&CK Technique | T1110.003 — Password Spraying |

---

## 4. Recommended Containment and Remediation Actions

### 4.1 Network / Identity Controls

- Block or restrict the identified source IP addresses at the firewall, WAF, or conditional access policy layer.
  - 73.15.72.101
  - 66.176.252.11

### 4.2 Account Monitoring

Place the five targeted accounts under heightened monitoring for a minimum of 14 days, with specific attention to:

- Successful authentication events from any source
- Additional failed-login bursts
- Authentication attempts from anomalous or previously unseen source IP addresses
- Geographically improbable login activity (impossible travel)
- Unauthorized MFA method changes or registrations
- Password reset requests or completions
- Registration of new or unfamiliar authentication methods or devices

### 4.3 MFA Verification

- Confirm that Multi-Factor Authentication is enabled, enforced, and functioning correctly for all five targeted accounts.
- Where MFA is not enforced, escalate to the Identity/IAM team for immediate remediation.

### 4.4 Escalation Criteria

Escalate to SOC Tier 2 / Incident Response if any of the following are observed during the monitoring period:

- A successful authentication from either identified source IP
- A successful authentication to any of the five targeted accounts from an anomalous location or device
- Any MFA modification not initiated by the legitimate account holder

---

## 5. Final SOC Assessment

| Item | Result |
|---|---|
| Incident Type | Password Spraying |
| MITRE ATT&CK Technique | T1110.003 |
| Unique Attacking Source IPs | 2 |
| Total Failed Authentication Events | 35 |
| Targeted Corporate Accounts | 5 |
| Successful Authentication Observed | No |
| Confirmed Account Compromise | No |
| Investigation Status | Complete |
| Recommended Follow-Up | Continued monitoring, IP blocking, MFA verification |

---

## 6. Analyst Conclusion

A password-spraying attempt was identified against five user accounts belonging to the organization. The investigation identified 35 failed authentication events originating from two source IP addresses, with the majority of activity (29 of 35 events) originating from 73.15.72.101. No successful authentication associated with either investigated source IP was observed in the available dataset. The incident is therefore classified as an unsuccessful authentication attack with no confirmed compromise at this time. Containment and monitoring actions are recommended as a precautionary measure given the deliberate, multi-account targeting pattern observed.

---

## 7. Investigation Limitations

This assessment is based solely on the authentication events available within the `BruteForce_json` dataset. The absence of successful authentication within this dataset does not independently confirm that the accounts were not compromised through another source, time period, or attack vector outside the scope of this data.

Further investigation is recommended using the following additional telemetry, where available:

- Entra ID / Azure AD sign-in logs
- Windows Security event logs
- Endpoint Detection and Response (EDR) telemetry
- Network traffic / proxy logs
- MFA activity and registration logs
- Account modification / audit logs
- Additional cloud security and CASB logs

---

## 8. MITRE ATT&CK Mapping

| Field | Mapping |
|---|---|
| Tactic | Credential Access (TA0006) |
| Technique | T1110 — Brute Force |
| Sub-technique | T1110.003 — Password Spraying |
| Observed Evidence | 35 failed authentication attempts against 5 user accounts |
| Source IPs | 73.15.72.101, 66.176.252.11 |
| Successful Authentication | None observed |

**Attack Chain:**

```
Credential Access (TA0006)
    |
    v
T1110 - Brute Force
    |
    v
T1110.003 - Password Spraying
    |
    v
Multiple User Accounts Targeted (5)
    |
    v
35 Failed Authentication Attempts
    |
    v
No Successful Authentication Observed
```

**Final MITRE Classification:** T1110.003 — Password Spraying

---

## 9. Supporting Evidence

| Artifact | Description | Timestamp |
|---|---|---|
| <img width="1919" height="868" alt="Screenshot 2026-08-12 093625" src="https://github.com/user-attachments/assets/a7c886bd-3932-454d-b8c9-7c5fa6408024" /> | Splunk query output supporting investigation findings | 12 Aug 2026, 09:50 AM |


