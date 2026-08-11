# 🛡️ Ransomware Response Guide

> A structured reference on how to respond to a ransomware attack, common mistakes to avoid, and prevention strategies for businesses.

---

## 📌 Overview

An effective **ransomware response plan** can be the difference between:
- A contained incident vs. a company-wide infection
- Swift remediation vs. permanent business closure

⚠️ **Key principle:** Failing to prepare = preparing to fail.

---

## ✅ 8 Critical Steps After a Ransomware Attack

| # | Step | Why It Matters |
|---|---|---|
| 1 | Isolate affected systems | Prevents ransomware from spreading laterally across the network |
| 2 | Secure backups | Backups are often targeted for encryption/deletion by ransomware |
| 3 | Disable maintenance tasks | Preserves forensic evidence (logs, temp files) |
| 4 | Create backups of infected systems | Protects against data loss from faulty decryptors |
| 5 | Quarantine the malware | Enables analysis without destroying evidence |
| 6 | Identify & investigate patient zero | Reveals how attackers gained access |
| 7 | Identify the ransomware strain | Determines if a free decryptor is available |
| 8 | Decide whether to pay the ransom | Last resort — weigh risks carefully |

---

### 🔍 Step Details

| Step | Key Actions |
|---|---|
| **1. Isolate** | Remove infected systems from the network immediately |
| **2. Secure Backups** | Disconnect backup storage from network; lock down access |
| **3. Disable Maintenance** | Stop temp file removal & log rotation — may hold forensic clues or encryption keys |
| **4. Backup Infected Systems** | Guards against buggy decryptors (e.g. Ryuk truncated files by 1 byte, corrupting VHD/database files) |
| **5. Quarantine** | Never delete/reformat — quarantine only; take memory dumps if malware is still active |
| **6. Patient Zero** | Can take weeks/months to trace; consider a professional forensics team if needed |
| **7. Identify Strain** | Use free tools like Emsisoft's ID tool or ID Ransomware |
| **8. Ransom Decision** | Only after all other options are exhausted |

---

## 💰 Should You Pay the Ransom?

| Risk Factor | Detail |
|---|---|
| No guarantee of decryption | ~1 in 20 chance attackers take payment without providing a decryptor |
| Faulty decryptors | Attacker-provided tools may not work properly |
| Funds criminal activity | Payments may support trafficking, terrorism, etc. |
| Perpetuates the model | Paying validates and encourages more ransomware attacks |

💡 Larger, more "professional" gangs are statistically more likely to deliver a working decryptor than smaller operators (e.g. Dharma, Phobos).

---

## 🚫 How NOT to Respond — Common Mistakes

| # | Mistake | Why to Avoid It |
|---|---|---|
| 1 | Restarting impacted devices | Some ransomware (e.g. Jigsaw) deletes files on reboot; also wipes memory forensics |
| 2 | Connecting external storage to infected systems | Ransomware may still be active and will encrypt backups too |
| 3 | Paying the ransom immediately | Other options should be exhausted first |
| 4 | Communicating on the impacted network | Attackers may still be monitoring traffic — use out-of-band channels |
| 5 | Deleting files or ransom notes | May contain forensic clues or embedded decryption keys (e.g. DoppelPaymer, BitPaymer) |
| 6 | Trusting ransomware authors | They are criminals with no obligation to honor agreements |

---

## 🔐 Preventative Measures — Reducing Infection Risk

| Measure | Purpose |
|---|---|
| Credential hygiene | Prevents brute-force attacks & credential theft |
| Principle of least privilege | Limits access to only what's necessary |
| Employee training | Builds awareness of phishing & social engineering |
| Multi-factor authentication (MFA) | Reduces unauthorized access risk |
| Review Active Directory | Closes backdoors like compromised service accounts |
| Network segregation | Contains incidents, limits spread |
| Secure remote access (RDP) | Popular attack vector — restrict via MFA-enabled VPN |
| Avoid BYOD | Personal devices are hard to secure/enforce policy on |
| Monitor/uninstall PowerShell | Common lateral movement tool for ransomware gangs |
| Cybersecurity insurance | Helps mitigate financial impact of an incident |

---

## 🧪 Test Your Response Plan

> The worst time to figure out your ransomware response is **during** an actual attack.

- ✅ Regularly test incident response plans
- ✅ Ensure employees know their roles
- ✅ Identify and fix gaps in the response chain

## 📚 Source

Adapted from: *"8 Critical Steps to Take After a Ransomware Attack"* — Jareth, Emsisoft, September 17, 2020.
