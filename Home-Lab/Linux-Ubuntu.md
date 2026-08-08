# 🐧 SOC Analyst Tier 1 — Linux Home Lab Practice

## Workflow Used

```
1. Know the system
       ↓
2. Know the users
       ↓
3. Check authentication
       ↓
4. Check running processes
       ↓
5. Check services
       ↓
6. Check network connections
       ↓
7. Check files
       ↓
8. Check persistence
       ↓
9. Search logs
       ↓
10. Build a timeline
       ↓
11. Investigate suspicious activity
       ↓
12. Document findings
```

---

## 01 — Know Your Machine

Before investigating anything, establish your baseline.

### Command: `hostname`

**Purpose:**
Identify the machine being investigated.

**Output:**
```
elilshree-VirtualBox
```

**SOC relevance:**
Helps identify which endpoint/server the investigation is being performed on.

---

### Command: `whoami`

**Purpose:**
Know which account you're currently using.

**Output:**
```
elilshree
```

**SOC relevance:**
Confirms the identity/context of the session performing the investigation — important for chain-of-custody and knowing whether you're operating as a standard user or something more privileged.

---

### Command: `id`

**Purpose:**
Shows your UID, GIDs and group memberships. Useful for understanding privileges.

**Output:**
```
uid=1000(elilshree) gid=1000(elilshree) groups=1000(elilshree),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin)
```

**SOC relevance:**
Membership in the `sudo` group means this account can escalate to root. Any investigation involving this account needs to account for that elevated privilege — both for what the account *could* do, and as a target for attackers looking to abuse existing sudo access rather than exploit a separate privilege escalation bug.

---

### Command: `hostnamectl`

**Purpose:**
System information — OS, hostname, kernel, architecture, etc.

**Output:**
<img width="525" height="310" alt="Screenshot 2026-08-08 083912" src="https://github.com/user-attachments/assets/11cce20a-12c0-45a2-8192-f886d2392706" />

**SOC relevance:**
Gives a consolidated system fingerprint (OS version, kernel, virtualization platform) used to check for outdated/vulnerable software and to confirm this is a VM (VirtualBox), which affects how network and persistence findings should be interpreted.

---

### Command: `uname -a`

**Purpose:**
Kernel/system information.

**Output:**
```
Linux elilshree-VirtualBox 7.0.0-28-generic #28-24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Wed Jul 1 15:50:57 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

**SOC relevance:**
Exact kernel version is used to cross-reference known kernel CVEs (e.g. privilege escalation exploits) relevant to this build of Ubuntu 24.04.1.

---

### Command: `uptime`

**Purpose:**
Shows how long the system has been running and system load. Useful when establishing a timeline.

**Output:**
```
08:41:38 up 9 min, 1 user, load average: 0.01, 0.00, 0.07
```

**SOC relevance:**
A system freshly rebooted (9 minutes uptime here) is itself an investigative clue — could indicate a patch/update, a crash, or an attacker attempting to clear in-memory evidence. Load averages also help spot abnormal resource usage tied to malicious processes.

---

### Command: `date`

**Purpose:**
Know the current system time when interpreting timestamps.

**Output:**
```
Sat Aug 8 08:42:34 AM +08 2026
```

**SOC relevance:**
Establishes the reference point (including timezone, `+08`) needed to correctly align log timestamps, file modification times, and login records into an accurate timeline later in the investigation.

---

## Notes for Next Steps

- Account `elilshree` is a standard user with `sudo` privileges — relevant for steps 3 (authentication) and 11 (privilege escalation checks).
- Kernel `7.0.0-28-generic` / Ubuntu 24.04.1 — note for later CVE cross-referencing.
- System uptime was ~9 minutes at time of baseline capture — worth revisiting once logs are reviewed to confirm reason for last boot.<img width="525" height="310" alt="Screenshot 2026-08-08 083912" src="https://github.com/user-attachments/assets/5370691d-315c-4568-bb2b-f132c932abd7" />


