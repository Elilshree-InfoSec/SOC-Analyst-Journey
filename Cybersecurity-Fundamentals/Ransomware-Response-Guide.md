# 🛡️ Ransomware Response Guide — Notes

> A structured reference on ransomware basics, response steps, common mistakes, prevention, and incident planning for businesses.

---

## 📌 What is Ransomware?

**Ransomware** is malicious software that infects a system and restricts access (locks the system or encrypts files) until a ransom is paid.

| Aspect | Detail |
|---|---|
| Method | Locks screen or encrypts files, displays an on-screen ransom alert |
| Typical ransom | Often $200–$400 for individuals (varies widely for businesses); usually demanded in crypto (e.g. Bitcoin) |
| Paying ≠ guarantee | Payment doesn't guarantee file recovery, and doesn't remove the malware infection itself |

### Possible Impact

| Impact Area | Effect |
|---|---|
| Data | Temporary or permanent loss of sensitive/proprietary information |
| Operations | Disruption to regular business activities |
| Financial | Costs to restore systems and files |
| Reputation | Potential long-term damage to trust and brand |

---

## 🔎 Signs You May Be Infected

- Browser/desktop locked with a payment demand message
- A "ransom note" file (usually `.txt`) appears in file directories
- Files suddenly have unfamiliar new extensions appended (e.g. `.locked`, `.crypto`, `.encrypted`, or random 6–7 character extensions)

---

## ✅ Critical Steps After a Ransomware Attack

| # | Step | Why It Matters |
|---|---|---|
| 1 | Isolate affected systems | Prevents ransomware from spreading laterally across the network |
| 2 | Disconnect external devices | USBs, phones, cameras, external drives can also become compromised |
| 3 | Secure backups | Backups are often targeted for encryption/deletion by ransomware |
| 4 | Disable maintenance tasks | Preserves forensic evidence (logs, temp files) |
| 5 | Create backups of infected systems | Protects against data loss from faulty decryptors |
| 6 | Quarantine the malware | Enables analysis without destroying evidence |
| 7 | Identify the source (patient zero) | Reveals how attackers gained access |
| 8 | Assess the scale of infection | Maps full spread before remediation |
| 9 | Identify the ransomware strain | Determines if a free decryptor is available |
| 10 | Report the incident | Enables faster support and limits damage/cost |
| 11 | Decide on remediation path | Restore, decrypt, accept loss, or pay — last resort |

### 🔍 Step Details

| Step | Key Actions |
|---|---|
| **Isolate** | Disconnect Ethernet, Wi-Fi, Bluetooth, NFC — don't just pull cables |
| **Disconnect External Devices** | Immediately unplug USB drives, phones, cameras, external hard drives |
| **Secure Backups** | Disconnect backup storage from network; lock down access |
| **Disable Maintenance** | Stop temp file removal & log rotation — may hold forensic clues or encryption keys |
| **Backup Infected Systems** | Guards against buggy decryptors (e.g. Ryuk truncated files by 1 byte, corrupting VHD/database files) |
| **Quarantine** | Never delete/reformat — quarantine only; take memory dumps if malware is still active |
| **Patient Zero** | Can take weeks/months to trace; consider a professional forensics team if needed |
| **Assess Scale** | Check shared drives, network storage, external devices, and cloud storage services |
| **Identify Strain** | Use free identification tools; look up whether a public decryptor exists |
| **Report** | Report early to limit damage and recovery cost — internally and to relevant authorities |

---

## 🗂️ Remediation Options — Best to Worst Case

| Plan | Option | Notes |
|---|---|---|
| A | Restore from backup | Ideal path — verify backup integrity first, wipe infected systems fully before restoring |
| B | Decrypt the data | Requires correctly identifying the ransomware strain; a decryptor may not exist for newer strains |
| C | Accept the loss | Wipe and start fresh; keep encrypted files backed up in case a decryptor is released later |
| D | Pay the ransom | Last resort — no guarantee of recovery, funds criminal activity, encourages more attacks |

---

## 💰 Should You Pay the Ransom?

| Risk Factor | Detail |
|---|---|
| No guarantee of decryption | ~1 in 20 chance attackers take payment without providing a decryptor |
| Faulty decryptors | Attacker-provided tools may not work properly |
| Funds criminal activity | Payments may support trafficking, terrorism, etc. |
| Perpetuates the model | Paying validates and encourages more ransomware attacks |
| Money could be reused against you | Some payments have funded further attacks on the same victim |

💡 Larger, more "professional" gangs are statistically more likely to deliver a working decryptor than smaller operators. Note: for some sophisticated strains, data recovery without paying may not be possible at all — this decision should never be made lightly.

---

## 🚫 How NOT to Respond — Common Mistakes

| # | Mistake | Why to Avoid It |
|---|---|---|
| 1 | Restarting impacted devices | Some ransomware (e.g. Jigsaw) deletes files on reboot; also wipes memory forensics — hibernate instead |
| 2 | Connecting external storage to infected systems | Ransomware may still be active and will encrypt backups too |
| 3 | Paying the ransom immediately | Other options should be exhausted first |
| 4 | Communicating on the impacted network | Attackers may still be monitoring traffic — use out-of-band channels |
| 5 | Deleting files or ransom notes | May contain forensic clues or embedded decryption keys |
| 6 | Trusting ransomware authors | They are criminals with no obligation to honor agreements |

---

## 🔐 Preventative Measures — Reducing Infection Risk

| Measure | Purpose |
|---|---|
| Data backup & recovery plan | Isolate critical backups from the network; test them regularly |
| Patch & update software/OS | Closes exploitable entry points used by most attacks |
| Up-to-date antivirus | Scan all downloads before executing |
| Credential hygiene | Prevents brute-force attacks & credential theft |
| Principle of least privilege | Limits access/permissions to only what's necessary |
| Avoid enabling email macros | A top delivery method for malware payloads |
| Avoid unsolicited web links | Common phishing/ransomware entry point |
| Employee training | Builds awareness of phishing & social engineering |
| Multi-factor authentication (MFA) | Reduces unauthorized access risk |
| Review Active Directory | Closes backdoors like compromised service accounts |
| Network segregation | Contains incidents, limits spread |
| Secure remote access (RDP) | Popular attack vector — restrict via MFA-enabled VPN |
| Avoid BYOD | Personal devices are hard to secure/enforce policy on |
| Monitor/uninstall PowerShell | Common lateral movement tool for ransomware gangs |
| Cybersecurity insurance | Helps mitigate financial impact of an incident |

---

## 📊 Incident Severity Classification

Useful for prioritizing resources and escalation decisions.

| Level | Indicators |
|---|---|
| Critical | Most personnel/critical systems offline; high risk or confirmed breach of sensitive data; severe long-term reputational damage |
| High | ~50% of personnel affected; risk of breach of personal/sensitive data; potential serious reputational damage |
| Medium | ~20% of personnel affected; small number of non-critical systems affected; possible breach of small, non-sensitive data |
| Low | <10% of personnel affected temporarily; minimal impact; no data breach |

---

## 🔄 High-Level Incident Response Process

| Phase | Focus |
|---|---|
| 1. Prepare | Build response plans, playbooks, roles, and readiness before an incident occurs |
| 2. Detect, Investigate & Activate | Identify the incident, analyze scope, activate the response team |
| 3. Contain, Collect Evidence & Remediate | Stop the spread, preserve evidence, fix the root cause |
| 4. Recover & Report | Restore systems to normal operation, notify required parties |
| 5. Learn & Improve | Conduct post-incident review, update plans and defenses |

---

## 🗒️ Evidence Collection Basics

- Keep a detailed log of who collected each item, and when (with time zone)
- Record physical location, serial/model numbers, hostname, MAC/IP address, hash values
- Common evidence types: disk/memory images, network packet captures, log files, config files, screenshots, investigation notes

---

## 📝 Post-Incident Review

After recovery, hold a review to capture lessons learned:

| Question | Purpose |
|---|---|
| What were the root causes? | Understand how the incident happened |
| Could it have been prevented? How? | Identify gaps in controls |
| What worked well in the response? | Reinforce effective practices |
| How can the response improve next time? | Drive concrete action items |

- A **hot debrief** (right after recovery) captures fresh, immediate feedback
- A **formal debrief** (days–weeks later) allows deeper reflection and senior management involvement
- Document findings and assign owners to follow-up actions

---

## 🧪 Test Your Response Plan

> The worst time to figure out your ransomware response is **during** an actual attack.

- ✅ Regularly test incident response plans (tabletop or functional exercises)
- ✅ Ensure employees know their roles
- ✅ Identify and fix gaps in the response chain
- ✅ Review/update the plan after any incident, exercise, or major organizational change
