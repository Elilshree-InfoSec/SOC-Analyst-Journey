# ⚡ PowerShell Security — SOC Analyst Tier 1 Investigation Notes

---

# 🎯 Purpose

PowerShell is one of the most abused tools in modern attacks.

It is built into Windows, supports remote execution, can download and execute payloads, and can run code directly in memory without dropping a traditional malware file.

As a Tier 1 SOC Analyst, the goal is **not to treat all PowerShell activity as malicious**.

The goal is to investigate the context:

* Who executed it?
* How was it launched?
* What command/script was executed?
* Did it download or execute suspicious content?
* What happened after execution?

---

# 🧠 PowerShell Investigation Mindset

PowerShell itself is not malicious.

SOC analysts investigate the **behaviour around PowerShell execution**:

1. Who executed PowerShell?
2. What parent process launched it?
3. What command or script block ran?
4. Was it encoded or obfuscated?
5. Did it communicate externally?
6. Did it modify the system or access sensitive resources?

A legitimate administrator script and an attacker payload can look similar.

The difference is usually:

* User context
* Parent process
* Command arguments
* Execution timing
* Follow-up activity

---

# 📋 Important Events

| Event ID    | Source     | Purpose              | What It Tells You                                                    |
| ----------- | ---------- | -------------------- | -------------------------------------------------------------------- |
| 4103        | PowerShell | Module Logging       | Cmdlets and pipeline execution details                               |
| 4104        | PowerShell | Script Block Logging | **Actual script content executed** — including deobfuscated commands |
| 4105 / 4106 | PowerShell | Command Start/Stop   | Execution start and stop timeline                                    |
| 400 / 403   | PowerShell | Engine State         | PowerShell startup/shutdown and version information                  |
| 1           | Sysmon     | Process Creation     | PowerShell execution, parent process, command line                   |
| 3           | Sysmon     | Network Connection   | PowerShell network communication                                     |

⭐ **Event ID 4104 is the most important one to remember.**

Script Block Logging can reveal the actual PowerShell code being executed, even when attackers attempt to hide commands using encoding or obfuscation.

---

# 📍 Log Locations

## PowerShell Operational Logs

```
Applications and Services Logs
→ Microsoft
→ Windows
→ PowerShell
→ Operational
```

## Common Event Sources

```
Microsoft-Windows-PowerShell/Operational
```

---

# ✅ Normal Behaviour

Examples of legitimate PowerShell activity:

* Administrators running:

  * `Get-Process`
  * `Get-Service`
  * `Get-ADUser`

* Scheduled maintenance scripts running:

  * At expected times
  * Under known service accounts

* IT automation scripts:

  * Stored in trusted locations
  * Properly documented

* User manually opening PowerShell:

```
explorer.exe → powershell.exe
```

---

# 🚩 Suspicious Behaviour

## Obfuscation / Execution Evasion

* CommandLine containing:

```
-enc
-EncodedCommand
```

→ Base64 encoded PowerShell payload

---

* Suspicious execution flags:

```
-nop
-NoProfile

-w hidden
-WindowStyle Hidden

-exec bypass
-ExecutionPolicy Bypass
```

Especially suspicious when combined together.

---

* PowerShell downgrade:

```
-version 2
```

Why suspicious:

* PowerShell v2 lacks modern security features
* Can bypass logging and AMSI protections

---

## Download & Execute Behaviour

Suspicious examples:

```
Invoke-Expression (IEX)
Invoke-WebRequest
Net.WebClient
DownloadString
```

Especially:

```
Download → Execute
```

patterns.

---

## Suspicious Parent Processes

PowerShell launched by:

```
winword.exe
excel.exe
outlook.exe
mshta.exe
browser.exe
```

Common attacker chain:

```
Malicious document
        ↓
PowerShell execution
        ↓
Payload download
```

---

## Other Red Flags

* Very long, random-looking command lines
* Heavy use of symbols/variables (obfuscation)
* Reflection-based loading:

```
[Reflection.Assembly]
```

* AMSI bypass patterns
* PowerShell logging disabled or missing during suspicious activity
* PowerShell running from unusual locations:

```
%TEMP%
%AppData%
Users\Public
```

---

# ❓ Investigation Questions

When investigating PowerShell activity:

* What was the full command line?
* What was the Script Block content (Event ID 4104)?
* Who executed it?
* What parent process launched it?
* Was it encoded or obfuscated?
* Did it download anything?
* Did it communicate with an external IP/domain?
* Did it access sensitive processes such as `lsass.exe`?
* Is this normal behaviour for this user/system?

---

# 🔎 Splunk Investigation Examples

## Find PowerShell execution

```spl
index=sysmon EventCode=1 Image="*powershell.exe*"
```

---

## Find encoded PowerShell commands

```spl
index=sysmon EventCode=1 Image="*powershell.exe*" CommandLine="*-enc*"
```

---

## Find suspicious PowerShell flags

```spl
index=sysmon EventCode=1 Image="*powershell.exe*" 
(CommandLine="*-nop*" OR CommandLine="*-w hidden*" OR CommandLine="*-exec bypass*")
```

---

## Find PowerShell v2 downgrade attempts

```spl
index=sysmon EventCode=1 Image="*powershell.exe*" CommandLine="*-version 2*"
```

---

## Search Script Block content for download/execute patterns

```spl
index=wineventlog EventCode=4104
| search ScriptBlockText="*Invoke-Expression*" 
OR ScriptBlockText="*DownloadString*" 
OR ScriptBlockText="*IEX*"
```

---

## Find PowerShell launched by Office applications

```spl
index=sysmon EventCode=1 Image="*powershell.exe*"
(ParentImage="*winword.exe*" 
OR ParentImage="*excel.exe*" 
OR ParentImage="*outlook.exe*")
```

---

# 🧩 PowerShell Investigation Flow

```
4624
User Logon
        ↓
Sysmon Event ID 1
PowerShell Execution
        ↓
4104
Script Block Content
        ↓
Sysmon Event ID 22
DNS Query
        ↓
Sysmon Event ID 3
Network Connection
        ↓
Sysmon Event ID 11
File Creation
        ↓
Sysmon Event ID 10
Credential Access
```

A common attacker timeline:

**Execution → Script Analysis → Network Activity → Payload Drop → Credential Theft**

---

# 🧭 Related MITRE ATT&CK Techniques

| Technique     | Description                                   |
| ------------- | --------------------------------------------- |
| **T1059.001** | Command and Scripting Interpreter: PowerShell |
| **T1027**     | Obfuscated Files or Information               |
| **T1140**     | Deobfuscate/Decode Files or Information       |
| **T1105**     | Ingress Tool Transfer                         |
| **T1562.001** | Impair Defenses (AMSI/logging bypass)         |

---

# ⭐ Tier 1 Priority Checklist

When you see PowerShell:

```
1. Check the user
        ↓
2. Check parent process
        ↓
3. Check command line
        ↓
4. Check Event ID 4104 content
        ↓
5. Check DNS/network activity
        ↓
6. Check dropped files
        ↓
7. Check persistence or credential access
```

---

## Key Takeaway

PowerShell is not automatically malicious.

A SOC Analyst determines risk by analysing:

**Who + How + What + Where + What happened next**

The investigation is about the behaviour, not the tool.
