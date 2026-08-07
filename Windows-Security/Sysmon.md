# 🛡️ Sysmon (System Monitor) — SOC Analyst Tier 1 Notes

## 📋 Sysmon Overview

| | |
|---|---|
| 📍 Log location | `Applications and Services Logs → Microsoft → Windows → Sysmon → Operational` |
| ⚙️ Installed as | A service + driver (Sysinternals / Microsoft) |
| 🎛️ Controlled by | A config file (XML) — defines what's included/excluded per event type |
| 🧠 Popular configs | SwiftOnSecurity config, Olaf Hartong's config |

**Why it exists:** Native Windows logs (like 4688) don't reliably capture command lines, don't log network connections, don't show DLL loads, and don't log DNS. Sysmon fills all of it.

| Event ID | Name | What It Tells You |
|---|---|---|
| 1️⃣ | Process Creation | What ran, full command line, parent process, hashes |
| 3️⃣ | Network Connection | Outbound/inbound connections — IP + port |
| 7️⃣ | Image Loaded | A DLL/module was loaded into a process |
| 🔟 | Process Access | One process opened a handle to another (credential theft) |
| 1️⃣1️⃣ | File Create | A file was written to disk |
| 2️⃣2️⃣ | DNS Query | A DNS lookup made by a process |

---

## 1️⃣ Event ID 1 — Process Creation

The single most-used Sysmon event. Most investigations start (and often end) here.

**🔍 Key fields:**

| Field | What to Check |
|---|---|
| Image | The executable that ran |
| ParentImage | What launched it |
| CommandLine | Full arguments — where `-enc`, `-w hidden`, obfuscation shows up |
| User | Account context |
| Hashes | SHA256/MD5 — for VT/threat intel lookups |
| CurrentDirectory | Ran from `%TEMP%`/`%AppData%`? Red flag |
| IntegrityLevel | Medium / High / System |

**🚩 Red Flags:**

| Pattern | Why It Matters |
|---|---|
| `winword.exe → powershell.exe`, `outlook.exe → cmd.exe`, `excel.exe → mshta.exe` | Classic malicious macro chain |
| `-enc`, `-nop`, `-w hidden`, `-exec bypass` in CommandLine | Obfuscated/malicious PowerShell pattern |
| Launched from `%TEMP%`, `%AppData%`, `\Public`, `\Downloads` | Common malware drop locations |
| `svchost.exe`/`lsass.exe` running from wrong path | Process masquerading |
| Long, obfuscated, Base64-heavy command line | Hiding true payload |

---

## 3️⃣ Event ID 3 — Network Connection

Shows outbound/inbound connections — something native Windows logs never give you.

**🔍 Key fields:**

| Field | What to Check |
|---|---|
| Image | Process that made the connection |
| DestinationIp / DestinationPort | Where it connected |
| SourceIp / SourcePort | Where it came from |
| Initiated | `true` = outbound connection started by this host |

**🚩 Red Flags:**

| Pattern | Why It Matters |
|---|---|
| `notepad.exe`/`calc.exe` making outbound connections | No legit reason to reach the internet |
| Unusual/high ports, no reputation/reverse DNS on dest IP | Possible C2 infrastructure |
| Same destination, regular time interval | Classic beaconing behavior |
| PowerShell/`cmd.exe` initiating direct outbound connection | Common in scripted C2/download-and-execute |

---

## 7️⃣ Event ID 7 — Image Loaded

Logs DLL/module loads. Noisier than most events — usually tuned/filtered in production.

**🔍 Key fields:**

| Field | What to Check |
|---|---|
| Image | Process loading the module |
| ImageLoaded | The DLL/module path |
| Signed / Signature | Is it signed, and by whom? |
| Hashes | For lookups |

**🚩 Red Flags:**

| Pattern | Why It Matters |
|---|---|
| Unsigned DLL in a normally fully-signed process | Tampering indicator |
| DLL loaded from `%TEMP%`/`%AppData%` instead of `System32` | Malicious module drop |
| Trusted process (`explorer.exe`, `svchost.exe`) loading an unusual DLL | Possible **DLL sideloading/hijacking** |

---

## 🔟 Event ID 10 — Process Access

One of the most important events — this is how you catch **credential dumping**.

**🔍 Key fields:**

| Field | What to Check |
|---|---|
| SourceImage | Process doing the accessing |
| TargetImage | Process being accessed |
| GrantedAccess | Access rights requested |
| CallTrace | Helps identify injection/dumping tool behavior |

**🚩 Red Flags:**

| Pattern | Why It Matters |
|---|---|
| ⚠️ Any unfamiliar process accessing `lsass.exe` | **The** textbook Mimikatz/credential-dumping pattern — escalate immediately |
| High `GrantedAccess` (broad memory read) from a non-standard tool | Likely dumping tool |
| `lsass.exe` accessed by a process running with unusual privilege | Suspicious even if the tool is unfamiliar |

---

## 1️⃣1️⃣ Event ID 11 — File Create

Logs when a file is created or overwritten on disk.

**🔍 Key fields:**

| Field | What to Check |
|---|---|
| Image | Process that created the file |
| TargetFilename | File path/name created |
| CreationUtcTime | Timestamp for timeline building |

**🚩 Red Flags:**

| Pattern | Why It Matters |
|---|---|
| `.exe`/`.dll`/`.ps1` dropped in `%TEMP%`, `%AppData%`, Startup, `\Public` | Common drop locations |
| Word/Outlook creating a new executable/script | Classic dropper behavior |
| Double extensions (`invoice.pdf.exe`) or mismatched file type | Disguised payload |
| Mass file creation/renaming across folders in a short window | Possible **ransomware** encryption |
| File dropped into Startup folder / matches Run key path | Persistence |

---

## 2️⃣2️⃣ Event ID 22 — DNS Query

Native Windows logging doesn't capture DNS at all — this is one of Sysmon's most valuable additions.

**🔍 Key fields:**

| Field | What to Check |
|---|---|
| Image | Process that made the query |
| QueryName | Domain being looked up |
| QueryResults | IP(s) returned |

**🚩 Red Flags:**

| Pattern | Why It Matters |
|---|---|
| Random/high-entropy domain names | Possible **DGA** malware |
| Very high volume/frequency to the same domain | Possible **DNS tunneling** |
| Process with no reason to need internet (local utility) making queries | Suspicious behavior |
| Domain has very recent registration / bad reputation | IOC match |
| PowerShell resolves a domain right before an Event ID 3 connection to it | Ties the attack story together |

---

## 🧩 Putting It Together — Investigation Flow

```
1️⃣  Process Creation      →  suspicious process + command line spotted
        ↓
2️⃣2️⃣  DNS Query            →  did it resolve anything suspicious?
        ↓
3️⃣  Network Connection     →  did it actually connect out, and where?
        ↓
1️⃣1️⃣  File Create           →  did it drop a payload / second stage?
        ↓
🔟  Process Access         →  did it (or anything downstream) touch lsass.exe?
```

This is the natural order most real Sysmon investigations follow: **execution → DNS → network → file drop → credential access.**

---

## ❓ Questions You Should Be Able to Answer

- ✅ What process ran, what was its full command line, and who launched it?
- ✅ Did it reach out to the network, and to where?
- ✅ Did it load anything unusual?
- ✅ Did it touch `lsass.exe` or another sensitive process?
- ✅ Did it write any new files to disk, and where?
- ✅ Did it resolve any suspicious domains?
