# 🛡️ Sysmon -  SOC Analyst Tier 1 Investigation Notes
---

## 📋 Quick Reference

| Event ID | Name |
|---|---|
| 1️⃣ | Process Creation |
| 3️⃣ | Network Connection |
| 7️⃣ | Image Loaded |
| 🔟 | Process Access |
| 1️⃣1️⃣ | File Create |
| 2️⃣2️⃣ | DNS Query |

---

# 🧠 Sysmon Investigation Mindset

Sysmon does not tell you "this is malicious".

It provides visibility that helps analysts answer:

1. What executed?
2. Who executed it?
3. How was it launched?
4. Did it communicate externally?
5. Did it modify the system?
6. Did it attempt credential access?

A single event rarely proves compromise.
SOC analysts correlate multiple events to build the attack timeline.

---

# 1️⃣ Event ID 1 — Process Creation

## 🎯 Purpose
Tells you **what ran, who ran it, what launched it, and with what arguments.** This is the most-used Sysmon event — most investigations start (and often end) here.

## 🔍 Important Fields

| Field | What It Means |
|---|---|
| `ParentImage` | Full path of the process that launched this one |
| `Image` | Full path of the executable that actually ran |
| `CommandLine` | The full command used to launch it, including arguments/flags — this is where `-enc`, `-w hidden`, etc. show up |
| `User` | The account context the process ran under |
| `Hashes` | SHA256/MD5/IMPHASH of the executable — used for VirusTotal/threat intel lookups |
| `IntegrityLevel` | Windows privilege level of the process (Low/Medium/High/System) — elevated integrity on an unexpected process is a red flag |

## ✅ Normal Behaviour
- `explorer.exe → chrome.exe` (user opens a browser)
- `services.exe → svchost.exe` (normal service startup)
- `cmd.exe` launched by a user for routine admin tasks

## 🚩 Suspicious Behaviour
- `winword.exe → powershell.exe`, `outlook.exe → cmd.exe`, `excel.exe → mshta.exe`
- CommandLine containing `-enc`, `-nop`, `-w hidden`, `-exec bypass`
- Process launched from `%TEMP%`, `%AppData%`, `\Public`, `\Downloads`
- `svchost.exe`/`lsass.exe` running from a path other than `System32`

## ❓ Investigation Questions
- Who executed it?
- What launched it?
- What happened after execution?

## 🔎 Splunk Investigation Examples
```
Find PowerShell execution:
index=sysmon EventCode=1 Image="*powershell.exe*"
```
```
Find obfuscated/malicious PowerShell flags:
index=sysmon EventCode=1 CommandLine="*-enc*" OR CommandLine="*-nop*" OR CommandLine="*hidden*"
```
```
Find Office spawning a shell (macro pattern):
index=sysmon EventCode=1 ParentImage="*winword.exe*" Image="*powershell.exe*"
```

## 🧭 Related MITRE ATT&CK Techniques
- **T1059** — Command and Scripting Interpreter
- **T1105** — Ingress Tool Transfer
- **T1036** — Masquerading
- **T1204** — User Execution
- **T1547** — Boot or Logon Autostart Execution

## 🔗 Related Logs
- Windows Event ID 4688 → Process Creation
- PowerShell Event ID 4104 → Script execution
- Event ID 4624 → User logon context

---
---

# 3️⃣ Event ID 3 — Network Connection

## 🎯 Purpose
Tells you **which process made a network connection, and where to.** Native Windows logs provide limited network visibility.Sysmon adds process-level network connection visibility by showing which process communicated and where.

## 🔍 Important Fields

| Field | What It Means |
|---|---|
| `Image` | The process that made the connection |
| `DestinationIp` | The IP address it connected to |
| `DestinationPort` | The port used on the destination side |
| `SourceIp` / `SourcePort` | The local IP/port used for the connection |
| `Initiated` | `true` = outbound connection started by this host; `false` = inbound |

## ✅ Normal Behaviour
- `chrome.exe`/`outlook.exe` making outbound connections on 443
- `svchost.exe` reaching Windows Update endpoints

## 🚩 Suspicious Behaviour
- `notepad.exe`/`calc.exe` making outbound connections at all
- Connections to unusual/high ports or IPs with no reputation
- Same destination, regular time interval (beaconing)
- PowerShell/`cmd.exe` initiating a direct outbound connection

## ❓ Investigation Questions
- What process made the connection?
- Where did it connect, and on what port?
- Is there a repeating pattern (beaconing)?

## 🔎 Splunk Investigation Examples
```
Find suspicious outbound connections, grouped by destination:
index=sysmon EventCode=3
| stats count by DestinationIp, Image
```
```
Find PowerShell/cmd making direct outbound connections:
index=sysmon EventCode=3 (Image="*powershell.exe*" OR Image="*cmd.exe*")
```
```
Spot possible beaconing (regular, repeated connections to same IP):
index=sysmon EventCode=3
| stats count by DestinationIp, Image
| where count > 20
```

## 🧭 Related MITRE ATT&CK Techniques
- **T1071** — Application Layer Protocol
- **T1090** — Proxy
- **T1571** — Non-Standard Port
- **T1041** — Exfiltration Over C2 Channel

## 🔗 Related Logs
- Firewall logs
- DNS logs
- Proxy logs
- Event ID 22 → DNS Query

---
---

# 7️⃣ Event ID 7 — Image Loaded

## 🎯 Purpose
Tells you **when a process loads a DLL/module.** Noisier than most events, so it's usually filtered/tuned in production configs.

## 🔍 Important Fields

| Field | What It Means |
|---|---|
| `Image` | The process loading the module |
| `ImageLoaded` | Path of the DLL/module being loaded |
| `Signed` / `Signature` | Whether the module has a valid digital signature, and who signed it |
| `Hashes` | Hash of the loaded module — useful for lookups |

## ✅ Normal Behaviour
- Signed system DLLs loaded from `System32` by trusted processes

## 🚩 Suspicious Behaviour
- Unsigned DLL loaded into a normally fully-signed process
- DLL loaded from `%TEMP%`/`%AppData%` instead of `System32`
- Trusted process (`explorer.exe`, `svchost.exe`) loading an unusual DLL

## ❓ Investigation Questions
- What process loaded the module?
- Is the module signed, and by whom?
- Is this a DLL that process would normally load?

## 🔎 Splunk Investigation Examples
```
Find unsigned DLLs being loaded:
index=sysmon EventCode=7 Signed=false
```
```
Find DLLs loaded from suspicious paths:
index=sysmon EventCode=7 ImageLoaded="*\\AppData\\*" OR ImageLoaded="*\\Temp\\*"
```

## 🧭 Related MITRE ATT&CK Techniques
- **T1574.002** — Hijack Execution Flow: DLL Side-Loading
- **T1055** — Process Injection

---
---

# 🔟 Event ID 10 — Process Access

## 🎯 Purpose
Tells you **when one process opens a handle to another process.** This is one of the strongest detection points for credential dumping, especially attempts to access lsass.exe memory.

## 🔍 Important Fields

| Field | What It Means |
|---|---|
| `SourceImage` | The process performing the access |
| `TargetImage` | The process being accessed |
| `GrantedAccess` | The Windows access rights mask granted (a hex value) — certain values indicate broad memory-read access typical of credential dumping |
| `CallTrace` | Stack trace showing which DLLs/modules were involved in the access — useful for spotting injection/dumping tool behavior |

## ✅ Normal Behaviour
- Standard OS/security tooling briefly accessing other processes for legitimate monitoring

## 🚩 Suspicious Behaviour
- ⚠️ Any unfamiliar process accessing `lsass.exe` — the textbook Mimikatz/credential-dumping pattern
- High `GrantedAccess` (broad memory read) from a non-standard tool

## ❓ Investigation Questions
- What process accessed `lsass.exe` (or another sensitive process)?
- What access rights were granted?
- Is the source process signed/known-good?

## 🔎 Splunk Investigation Examples
```
Find LSASS access:
index=sysmon EventCode=10 TargetImage="*lsass.exe*"
```
```
Narrow to high-risk access rights (common dumping pattern):
index=sysmon EventCode=10 TargetImage="*lsass.exe*" GrantedAccess="0x1010" OR GrantedAccess="0x1410"
```

## 🧭 Related MITRE ATT&CK Techniques
- **T1003.001** — OS Credential Dumping: LSASS Memory
- **T1055** — Process Injection

## 🔗 Related Logs
- Security Event Logs
- EDR telemetry
- Malware analysis tools

---
---

# 1️⃣1️⃣ Event ID 11 — File Create

## 🎯 Purpose
Tells you **when a file is created or overwritten on disk** — useful for spotting dropped payloads and ransomware activity.

## 🔍 Important Fields

| Field | What It Means |
|---|---|
| `Image` | The process that created the file |
| `TargetFilename` | Full path of the file that was created |
| `CreationUtcTime` | Timestamp of creation, in UTC — useful for building a timeline |

## ✅ Normal Behaviour
- Browser downloads landing in `\Downloads`
- Applications writing their own config/temp files during normal use

## 🚩 Suspicious Behaviour
- `.exe`/`.dll`/`.ps1` dropped in `%TEMP%`, `%AppData%`, Startup, or `\Public`
- Word/Outlook creating a new executable or script
- Double extensions (`invoice.pdf.exe`) or mismatched file type
- Mass file creation/renaming across folders in a short window (ransomware)
- File dropped into Startup folder or matching a Registry Run key path

## ❓ Investigation Questions
- What process created the file?
- Where was it written?
- Does the file type match its extension?
- Is this tied to a persistence location?

## 🔎 Splunk Investigation Examples
```
Find executables dropped in suspicious folders:
index=sysmon EventCode=11 TargetFilename="*\\AppData\\*.exe" OR TargetFilename="*\\Temp\\*.exe"
```
```
Find mass file creation (possible ransomware):
index=sysmon EventCode=11
| stats count by Image
| where count > 100
```

## 🧭 Related MITRE ATT&CK Techniques
- **T1105** — Ingress Tool Transfer
- **T1486** — Data Encrypted for Impact
- **T1547** — Boot or Logon Autostart Execution

---
---

# 2️⃣2️⃣ Event ID 22 — DNS Query

## 🎯 Purpose
Tells you **what domain a process resolved.** Native Windows logging doesn't capture DNS at all — this is one of Sysmon's most valuable additions.

## 🔍 Important Fields

| Field | What It Means |
|---|---|
| `Image` | The process that made the DNS query |
| `QueryName` | The domain being looked up |
| `QueryResults` | The IP address(es) returned in the DNS response |

## ✅ Normal Behaviour
- Browsers/apps resolving well-known, reputable domains

## 🚩 Suspicious Behaviour
- Random/high-entropy domain names (possible DGA malware)
- Very high volume/frequency of queries to the same domain (DNS tunneling)
- A local utility with no reason to need internet making DNS queries
- Domain has very recent registration or known-bad reputation

## ❓ Investigation Questions
- What process made the query?
- Is the domain reputable, or newly registered/random-looking?
- Did this query lead to a network connection (Event ID 3) shortly after?

## 🔎 Splunk Investigation Examples
```
Find most-queried domains (spot outliers/DGA candidates):
index=sysmon EventCode=22
| stats count by QueryName
| sort -count
```
```
Find DNS queries from a specific suspicious process:
index=sysmon EventCode=22 Image="*powershell.exe*"
```

## 🧭 Related MITRE ATT&CK Techniques
- **T1071.004** — Application Layer Protocol: DNS
- **T1568.002** — Dynamic Resolution: Domain Generation Algorithms

---
---

## 🧩 Putting It Together — Investigation Flow

```
4624 User Logon
        ↓
Sysmon Event ID 1
(Process Execution)
        ↓
Sysmon Event ID 22
(DNS Resolution)
        ↓
Sysmon Event ID 3
(Network Connection)
        ↓
Sysmon Event ID 11
(File Drop)
        ↓
Sysmon Event ID 10
(Credential Access)
        ↓
Persistence Check
(Registry / Services / Scheduled Tasks)
```

This is the natural order most real Sysmon investigations follow: **execution → DNS → network → file drop → credential access.**

---

## ⭐ Priority During Triage

Most frequently investigated:

1. Event ID 1 - Process Creation ⭐⭐⭐⭐⭐
2. Event ID 3 - Network Connection ⭐⭐⭐⭐
3. Event ID 22 - DNS Query ⭐⭐⭐⭐
4. Event ID 11 - File Creation ⭐⭐⭐
5. Event ID 10 - Process Access ⭐⭐⭐
6. Event ID 7 - Image Loaded ⭐⭐
