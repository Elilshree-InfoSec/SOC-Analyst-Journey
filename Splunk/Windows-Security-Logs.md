# 🔐 Windows Security Logs 

---

## 📋 Core Event IDs

| Event ID | Purpose | Watch For |
|---|---|---|
| 4624 | Successful Logon | Wrong Logon Type for account, odd hour, new source IP/workstation |
| 4625 | Failed Logon | Sub-status code = real reason for failure |
| 4634 / 4647 | Logoff | Session duration, timeline building |
| 4648 | Explicit Credentials Used ("Run as") | Common in lateral movement |
| 4672 | Special Privileges Assigned | Pair with 4624 to spot privileged logons |
| 4688 | Process Creation | Command line + parent process = malware trace |
| 4698 | Scheduled Task Created | Common persistence mechanism |
| 4720 | User Account Created | Who created it? Expected? |
| 4722 | Account Enabled | — |
| 4723 | Password Change Attempt | — |
| 4724 | Password Reset | — |
| 4725 | Account Disabled | — |
| 4728 / 4732 / 4756 | Added to Security Group | Esp. Domain Admins/Enterprise Admins |
| 4738 | User Account Changed | — |
| 4768 | Kerberos TGT Request | Odd hours, RC4 encryption = downgrade |
| 4769 | Kerberos Service Ticket Request | Burst across many SPNs = Kerberoasting |
| 4771 | Kerberos Pre-Auth Failed | Brute force against AD accounts |

---

## 🎯 Logon Types (memorize these — shows up in 4624/4625)

| Type | Meaning | Notes |
|---|---|---|
| 2 | Interactive | Physical console logon |
| 3 | Network | SMB/shares — very common, mostly noise |
| 4 | Batch | Scheduled task execution |
| 5 | Service | Service account startup |
| 7 | Unlock | Workstation unlock |
| 8 | NetworkCleartext | Password sent in clear — investigate |
| 9 | NewCredentials | RunAs |
| 10 | RemoteInteractive | **RDP** — high investigative value |
| 11 | CachedInteractive | Logon using cached creds (no DC contact) |

---

## 🧩 Key Fields to Always Check

- **Account Name** — who
- **Logon Type** — how
- **Source Network Address** — from where
- **Workstation Name** — which device
- **Logon ID** — ties related events together (logon → logoff → process creation)
- **Sub-Status Code** (4625 only) — why it failed

---

## 🔑 4625 Sub-Status Codes

| Code | Meaning |
|---|---|
| 0xC000006A | Bad password |
| 0xC0000064 | User doesn't exist |
| 0xC0000234 | Account locked out |
| 0xC0000072 | Account disabled |
| 0xC0000071 | Password expired |

---

## 🚩 Red Flags to Investigate

- Many 4625s, **one account, many source IPs** → 🔴 brute force
- Many 4625s, **many accounts, one source IP** → 🔴 password spray
- 4625 immediately followed by successful 4624 → possible compromise after guessing
- 4648 (explicit creds) on a host the user doesn't normally access → lateral movement
- 4688 with abnormal parent-child: `winword.exe → powershell.exe`, `outlook.exe → cmd.exe` → malicious macro
- 4688 process launched from `%TEMP%`, `%AppData%`, `C:\Users\Public`
- New account (4720) added straight into Domain Admins (4728/4732) outside change window
- 4769 — burst of requests for many different SPNs from one account in a short window → **Kerberoasting**
- 4768/4769 using RC4 (`0x17`) instead of AES (`0x12`) → encryption downgrade attempt
- 4698 (scheduled task) created by an unexpected account or running as SYSTEM

---

## ❓ Questions You Should Be Able to Answer From These Logs

- Who logged in, and from where?
- Was the logon type appropriate for that account/role?
- Did it succeed or fail — and why (sub-status)?
- Was a new user created, and by whom?
- Was anyone added to a privileged group?
- What process was spawned, and by what parent?
- Is there Kerberos activity suggesting ticket abuse?

---

## ✅ Core Skills to Practice

1. Build a full **logon timeline** for a user using 4624/4625/4634.
2. Distinguish **brute force vs. password spray** from 4625 patterns.
3. Trace a **4688 process tree** to find the origin of a suspicious process.
4. Detect **Kerberoasting** from 4769 volume + encryption type.
5. Spot **unauthorized privileged group** membership changes.
6. Correlate **Logon ID** across events to build one coherent session story.

