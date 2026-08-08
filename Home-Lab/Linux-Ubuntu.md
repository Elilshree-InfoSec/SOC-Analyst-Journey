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
 
---
 
### Command: `hostname`
 
**Purpose:**
Identify the machine being investigated.
 
**Output:**
```
elilshree-VirtualBox
```
 
**🛡️ SOC relevance:**
Helps identify which endpoint/server the investigation is being performed on.
 
---
 
### Command: `whoami`
 
**Purpose:**
Know which account you're currently using.
 
**Output:**
```
elilshree
```
 
**🛡️ SOC relevance:**
Confirms the identity/context of the session performing the investigation — important for chain-of-custody and knowing whether you're operating as a standard user or something more privileged.
 
---
 
### Command: `id`
 
**Purpose:**
Shows your UID, GIDs and group memberships. Useful for understanding privileges.
 
**Output:**
```
uid=1000(elilshree) gid=1000(elilshree) groups=1000(elilshree),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin)
```
 
**🛡️ SOC relevance:**
Membership in the `sudo` group means this account can escalate to root. Any investigation involving this account needs to account for that elevated privilege — both for what the account *could* do, and as a target for attackers looking to abuse existing sudo access rather than exploit a separate privilege escalation bug.
 
---
 
### Command: `hostnamectl`
 
**Purpose:**
System information — OS, hostname, kernel, architecture, etc.
 
**Output:**
 
![hostnamectl output](https://github.com/user-attachments/assets/05c928fe-05dc-4d17-abbd-a0b64cc4f1da)
 
**🛡️ SOC relevance:**
Gives a consolidated system fingerprint (OS version, kernel, virtualization platform) used to check for outdated/vulnerable software and to confirm this is a VM (VirtualBox), which affects how network and persistence findings should be interpreted.
 
---
 
### Command: `uname -a`
 
**Purpose:**
Kernel/system information.
 
**Output:**
```
Linux elilshree-VirtualBox 7.0.0-28-generic #28-24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Wed Jul 1 15:50:57 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

**🛡️ SOC relevance:**
Exact kernel version is used to cross-reference known kernel CVEs (e.g. privilege escalation exploits) relevant to this build of Ubuntu 24.04.1.
 
---
 
### Command: `uptime`
 
**Purpose:**
Shows how long the system has been running and system load. Useful when establishing a timeline.
 
**Output:**
```
08:41:38 up 9 min, 1 user, load average: 0.01, 0.00, 0.07
```
 
**🛡️ SOC relevance:**
A system freshly rebooted (9 minutes uptime here) is itself an investigative clue — could indicate a patch/update, a crash, or an attacker attempting to clear in-memory evidence. Load averages also help spot abnormal resource usage tied to malicious processes.
 
---
 
### Command: `date`
 
**Purpose:**
Know the current system time when interpreting timestamps.
 
**Output:**
```
Sat Aug 8 08:42:34 AM +08 2026
```
 
**🛡️ SOC relevance:**
Establishes the reference point (including timezone, `+08`) needed to correctly align log timestamps, file modification times, and login records into an accurate timeline later in the investigation.
 
---

## 02 — Know the Users

Now that the machine is fingerprinted, it's time to find out who has been using it — currently, historically, and unsuccessfully.

---

### Command: `who`

**Purpose:**
Shows currently logged-in users.

**Output:**
```
elilshree   seat0                             2026-08-08   08:33  (login screen)
elilshree   tty2                              2026-08-08   08:33  (tty2)
```

**🛡️ SOC relevance:**
Confirms exactly who is on the box right now and how they're connected (seat0 = local graphical session, tty2 = local terminal). Two sessions for the same user isn't unusual on a personal VM, but on a shared or production box, multiple concurrent sessions for one account is worth a second look.

---

### Command: `w`

**Purpose:**
Shows logged-in users plus what they're currently doing (idle time, CPU usage, current process).

**Output:**

![w output](https://github.com/user-attachments/assets/f9650637-ff4c-4a9e-be45-b3a2493e407a)

**🛡️ SOC relevance:**
Adds an activity layer on top of `who` — useful for catching a session that's technically "logged in" but has been idle for a suspiciously long time, or one running an unexpected command.

---

### Command: `users`

**Purpose:**
Quick, no-frills list of logged-in users.

**Output:**
```
elilshree elilshree
```

**🛡️ SOC relevance:**
Fast sanity check — good for scripting/automation when you just need names, not session detail.

---

### Command: `last`

**Purpose:**
Shows historical login sessions — one of the most important commands for a SOC analyst.

**Output:**

![last output](https://github.com/user-attachments/assets/5ea17c94-f56b-45a0-894f-a4d9a63ae89d)

**🛡️ SOC relevance:**
Look for:
- Unexpected usernames
- Unusual login times (odd hours, outside business hours)
- Unexpected source IPs / terminals
- Repeated or rapid sessions (possible automated/scripted access)

This builds the login history piece of your eventual timeline (step 10).

---

### Command: `lastlog`

**Purpose:**
Shows the last login time for every account on the system, including service/system accounts.

**Output:**
```
Username             Latest
root                 **Never logged in**
daemon               **Never logged in**
bin                  **Never logged in**
sys                  **Never logged in**
sync                 **Never logged in**
games                **Never logged in**
man                  **Never logged in**
lp                   **Never logged in**
mail                 **Never logged in**
news                 **Never logged in**
uucp                 **Never logged in**
proxy                **Never logged in**
www-data             **Never logged in**
backup               **Never logged in**
list                 **Never logged in**
irc                  **Never logged in**
_apt                 **Never logged in**
nobody               **Never logged in**
systemd-network      **Never logged in**
systemd-timesync     **Never logged in**
dhcpcd               **Never logged in**
messagebus           **Never logged in**
syslog               **Never logged in**
systemd-resolve      **Never logged in**
uuidd                **Never logged in**
usbmux               **Never logged in**
tss                  **Never logged in**
systemd-oom          **Never logged in**
kernoops             **Never logged in**
whoopsie             **Never logged in**
dnsmasq              **Never logged in**
avahi                **Never logged in**
tcpdump              **Never logged in**
sssd                 **Never logged in**
speech-dispatcher    **Never logged in**
cups-pk-helper       **Never logged in**
fwupd-refresh        **Never logged in**
saned                **Never logged in**
geoclue              **Never logged in**
cups-browsed         **Never logged in**
hplip                **Never logged in**
gnome-remote-desktop **Never logged in**
polkitd              **Never logged in**
rtkit                **Never logged in**
colord               **Never logged in**
gnome-initial-setup  **Never logged in**
gdm                  **Never logged in**
nm-openvpn           **Never logged in**
elilshree            **Never logged in**
splunk               **Never logged in**
```

**🛡️ SOC relevance:**
This is a full account inventory check. The key thing to flag: any system/service account showing an actual login timestamp (instead of "Never logged in") is highly suspicious — service accounts like `www-data`, `daemon`, or `sync` should never have interactive logins. Also worth noting: `elilshree` shows "Never logged in" here even though `who`/`last` show activity — this is because `lastlog` reads login records that don't always capture graphical/seat0 sessions the same way, which is a useful quirk to remember when cross-referencing tools rather than trusting one blindly.

---

### Command: `sudo lastb`

**Purpose:**
Shows failed login attempts — critical for spotting brute-force activity.

**Output:**
```
[Password prompt required — sudo]
btmp begins Mon Aug 3 11:13:48 2026
```

**🛡️ SOC relevance:**
`lastb` reads from `/var/log/btmp`, the failed-login log. A high volume of failed attempts, especially against multiple usernames or in rapid succession, is a classic brute-force indicator. The fact that `btmp` begins Aug 3 tells you the earliest point this log can show evidence from — anything before that date is outside this log's visibility.

---



