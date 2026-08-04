# 📖 Windows Logging for SOC Analysts

## 🎯 Objective

Learn the most important Windows logs used by SOC analysts to detect, investigate, and respond to security incidents. This room focuses on authentication logs, user management, Sysmon, process monitoring, and PowerShell logging.

---

# 📝 Summary

Windows logs are one of the most valuable data sources inside a SIEM. Every login, process, file creation, registry modification, network connection, and PowerShell command can leave evidence behind.

For a SOC Analyst, understanding **what each Event ID means**, **where it is logged**, and **how events relate together** is far more important than memorizing every Windows Event ID.

---

# 🛡️ Why Logging Matters

| SOC Activity         | Why Logs Matter                                  |
| -------------------- | ------------------------------------------------ |
| 🚨 Incident Response | Identify when, how, and who performed the attack |
| 🔎 Threat Hunting    | Search for suspicious behaviour across endpoints |
| 📢 Alert Triage      | SIEM detections are built from logs              |
| 📊 Forensics         | Reconstruct the complete attack timeline         |

---

# 📂 Windows Event Logs

### 📍 Log Location

```
C:\Windows\System32\winevt\Logs
```

Windows logs are stored as **EVTX (binary)** files.

Use **Event Viewer (eventvwr)** to read them.

---

## 🖥️ Event Viewer Layout

| Component     | Purpose                                 |
| ------------- | --------------------------------------- |
| Log Sources   | Categories of Windows logs              |
| Event List    | Individual events                       |
| Event Details | Full event information (Plain Text/XML) |
| Filter/Search | Quickly locate relevant events          |

---

# ⭐ Important Event IDs

## 🔐 Authentication Logs

These are the **most important Windows Security logs** for SOC analysts.

| Event ID | Description      | Detects                        |
| -------- | ---------------- | ------------------------------ |
| **4624** | Successful Logon | Suspicious logins, RDP access  |
| **4625** | Failed Logon     | Brute force, password spraying |

---

## 🔍 Event ID 4624 (Successful Logon)

### Check these fields first

➡️ Username

➡️ Logon Type

➡️ Source IP

➡️ Source Hostname

➡️ Logon ID

---

### Common Use Cases

✅ Investigate suspicious RDP logins

✅ Find attacker entry point

✅ Correlate later events using **Logon ID**

---

## ❌ Event ID 4625 (Failed Logon)

### Detects

* Password spraying
* Brute-force attacks
* Login failures
* Account guessing

> ⚠️ Failed logon events can sometimes be noisy or misleading. Always investigate context before concluding an attack.

---

# 👤 User Management Events

Attackers frequently create or modify accounts for persistence.

| Event ID | Description           | Why It Matters            |
| -------- | --------------------- | ------------------------- |
| **4720** | User created          | Backdoor account          |
| **4722** | User enabled          | Reactivated account       |
| **4738** | User modified         | Account changes           |
| **4725** | User disabled         | Disable security accounts |
| **4726** | User deleted          | Covering tracks           |
| **4723** | User changed password | Credential modification   |
| **4724** | Password reset        | Privilege abuse           |
| **4732** | Added to group        | Privilege escalation      |
| **4733** | Removed from group    | Privilege changes         |

---

## 📝 Structure of User Management Events

Every event usually contains three sections.

### 👤 Subject

Who performed the action

↓

### 🎯 Object

Target user/account

↓

### ⚙️ Details

What exactly changed

---

### Example

```
Administrator
        │
        ▼
Created user: svc_backup
        │
        ▼
Added to Administrators group
```

---

# 🖥️ Process Monitoring

Most attacks eventually execute a process.

Process creation logs help identify:

* Malware execution
* LOLBins
* PowerShell
* Persistence
* Exploitation

---

# 4688 vs Sysmon Event ID 1

| Feature           | Event ID 4688 | Sysmon Event ID 1 |
| ----------------- | ------------- | ----------------- |
| Default Windows   | ❌ Disabled    | ❌ Requires Sysmon |
| Process Creation  | ✅             | ✅                 |
| Command Line      | ✅             | ✅                 |
| Parent Process    | ✅             | ✅                 |
| Process Hash      | ❌             | ✅                 |
| Digital Signature | ❌             | ✅                 |
| Recommended       | ⭐             | ⭐⭐⭐⭐⭐             |

> 💡 **Sysmon is the preferred process monitoring solution for SOC teams.**

---

# 🧩 Sysmon Event ID 1

### Important Fields

➡️ Process Name

➡️ Command Line

➡️ Process ID (PID)

➡️ Parent Process

➡️ User

➡️ Logon ID

➡️ Hash

➡️ Digital Signature

---

### Why It's Important

Sysmon provides enough context to:

* Build process trees
* Track malware execution
* Correlate attacker activity
* Reconstruct attack chains

---

# 📊 Useful Sysmon Event IDs

| Event ID | Purpose               | Detects               |
| -------- | --------------------- | --------------------- |
| **1**    | Process Creation      | Malware execution     |
| **3**    | Network Connection    | C2 communication      |
| **11**   | File Creation         | Malware dropped files |
| **13**   | Registry Modification | Persistence           |
| **22**   | DNS Query             | Suspicious domains    |

---

## 🔗 Correlation Tip

Most Sysmon events only show the **Process ID**.

To obtain full context:

```
Event ID 3
(Network Connection)
        │
        ▼
Use Process ID
        │
        ▼
Find Sysmon Event ID 1
        │
        ▼
View Process Name
Parent Process
Command Line
User
```

---

# ⚡ PowerShell Logging

Attackers love PowerShell because it can:

* Download malware
* Enumerate users
* Execute scripts
* Modify the system
* Exfiltrate data

Unfortunately...

Launching PowerShell creates only **one** process creation event.

All commands inside that session are **not visible** in Sysmon Event ID 1.

---

# 📄 PowerShell History File

```
C:\Users\<User>\AppData\Roaming\
Microsoft\Windows\PowerShell\
PSReadLine\
ConsoleHost_history.txt
```

---

## What It Records

✅ Every PowerShell command entered

✅ Survives reboots

✅ Separate history per user

---

## Limitations

❌ No command output

❌ Doesn't capture script contents (`script.ps1`)

❌ Can be manually deleted

---

# 🔗 Event Correlation (SOC Workflow)

A good SOC analyst **connects events together**, rather than analyzing them individually.

```
4624
Successful Login
        │
        ▼
Sysmon Event 1
Process Created
        │
        ▼
Sysmon Event 3
Network Connection
        │
        ▼
Sysmon Event 11
File Created
        │
        ▼
Sysmon Event 13
Registry Modified
```

Always correlate using:

* **Logon ID** → Track user activity
* **Process ID (PID)** → Track process activity

---

# 💡 SOC Analyst Tips

### Authentication

* Review unusual login times
* Check failed login spikes
* Investigate external RDP access
* Look for impossible travel or unfamiliar IPs

### User Accounts

* Watch for newly created accounts
* Monitor password resets
* Investigate users added to **Administrators**
* Verify disabled/deleted accounts

### Process Monitoring

* Always inspect parent-child relationships
* Review command-line arguments
* Verify process hashes and signatures
* Look for LOLBins (e.g., PowerShell, cmd, rundll32)

### Sysmon

* Install Sysmon whenever possible
* Correlate all events with **Event ID 1**
* Investigate unusual outbound connections
* Monitor dropped executables and registry changes

### PowerShell

* Check PowerShell history during investigations
* Hunt for:

  * `Invoke-WebRequest`
  * `Invoke-Expression (IEX)`
  * `DownloadString`
  * `Get-ChildItem`
  * `Get-Content`
  * Encoded commands

---

# ⭐ Key Takeaways

* Windows Event Viewer reads **EVTX** log files.
* **4624** = Successful login.
* **4625** = Failed login (brute force/password spraying).
* User management events reveal account creation, password resets, and privilege changes.
* **Sysmon Event ID 1** is one of the most valuable logs for SOC analysts.
* Additional Sysmon events (**3, 11, 13, 22**) provide network, file, registry, and DNS visibility.
* PowerShell commands are **not fully captured** by process creation logs; check the **PowerShell history file**.
* Correlate events using **Logon ID** and **Process ID** to reconstruct the attack chain.

---

# 🎯 Event IDs to Memorize

| Event ID        | Description                 |
| --------------- | --------------------------- |
| **4624**        | Successful Logon            |
| **4625**        | Failed Logon                |
| **4720**        | User Created                |
| **4722**        | User Enabled                |
| **4738**        | User Changed                |
| **4725**        | User Disabled               |
| **4726**        | User Deleted                |
| **4723**        | Password Changed            |
| **4724**        | Password Reset              |
| **4732**        | Added to Security Group     |
| **4733**        | Removed from Security Group |
| **1 (Sysmon)**  | Process Creation            |
| **3 (Sysmon)**  | Network Connection          |
| **11 (Sysmon)** | File Creation               |
| **13 (Sysmon)** | Registry Modification       |
| **22 (Sysmon)** | DNS Query                   |

---

# 💭 Personal Reflection

This room strengthened my understanding of the Windows logs that SOC analysts work with daily. I learned that successful investigations depend on correlating multiple logs—especially using **Logon ID** and **Process ID**—rather than analyzing single events in isolation. Sysmon stood out as one of the most valuable monitoring tools because it provides much richer context than default Windows logging. These logging fundamentals will be essential for SIEM investigations, threat hunting, and future SOC simulations.

---

# 📅 Progress

| Status         | Value                                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Room Completed | ✅                                                                                                                             |
| Difficulty     | Easy                                                                                                                            |
| Estimated Time | 1 hour                                                                                                                          |

