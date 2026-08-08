# 🛡️ SOC Analyst Tier 1 — Core Security Concepts

> CIA Triad, Threat Vectors, Attack Types, IOCs & MITRE ATT&CK

---

## 1. 🔺 CIA Triad

The foundation of everything in security.

| Pillar | Meaning | Broken By | Protected By |
|---|---|---|---|
| **Confidentiality** | Only authorized people access data | Data leaks, weak encryption | Encryption, access control |
| **Integrity** | Data stays accurate & untampered | Malware modifying files, MitM | Hashing, checksums, digital signatures |
| **Availability** | Systems accessible when needed | DDoS, ransomware, hardware failure | Backups, redundancy, failover |

> 💡 **Quick trick:** For any incident, ask *"Which side of the triangle got hit?"*

```
              Confidentiality
                   /\
                  /  \
                 /    \
                /  CIA \
               /________\
       Integrity      Availability
```

---

## 2. 🚪 Threat Vectors

The **entry point/path** an attacker uses — not the attack itself.

```
   Internet          Attacker
      |                  |
      v                  v
 +-----------------------------+
 |       THREAT VECTORS        |
 +-----------------------------+
 |  Email                  --> |
 |  Malicious websites     --> |
 |  USB / removable media  --> |
 |  Unpatched software     --> |
 |  Weak/stolen creds      --> |
 |  Social engineering     --> |
 |  Third-party/vendor     --> |
 +-----------------------------+
              |
              v
      YOUR ORGANIZATION
```

---

## 3. 🎯 Common Attack Types

| Attack | What It Is | Key Giveaway |
|---|---|---|
| **Phishing** | Fake email/message stealing creds or delivering malware | Spoofed sender, urgent tone |
| **Spear Phishing** | Phishing targeted at one specific person | Personalized details |
| **Whaling** | Phishing aimed at execs/high-value targets | Fake CEO/finance requests |
| **Ransomware** | Encrypts files, demands payment | Shadow copies deleted first |
| **Brute-force** | Guessing passwords repeatedly | Many failed logins, same source |
| **Credential Stuffing** | Reusing leaked user/pass pairs | Fails across multiple accounts |
| **Malware** | Viruses, worms, trojans, spyware | Odd processes, persistence |
| **DoS / DDoS** | Flooding system to knock it offline | Sudden traffic spike |
| **Man-in-the-Middle** | Intercepting traffic between two parties | ARP spoofing, cert mismatch |
| **SQL Injection** | Malicious SQL via input fields | Weird DB query patterns |
| **XSS** | Injected scripts run in victim's browser | Script tags in input |
| **Zero-day** | Exploiting an unpatched, unknown flaw | No signature/detection exists yet |

---

## 4. 🚨 Indicators of Compromise (IOCs)

Evidence a system **might already be compromised**.

```
IOC CATEGORIES

  File Hashes    --> MD5 / SHA1 / SHA256 -> check VirusTotal
  Malicious IPs  --> linked to C2 servers / botnets
  Bad Domains    --> typosquats (paypa1.com) / DGA domains
  Login Anomalies --> impossible travel, odd hours
  Network Traffic --> beaconing, weird outbound ports
  Odd Processes  --> Word/Excel spawning PowerShell
  File/Registry  --> new persistence, disabled AV
```

> 🧠 IOCs = **"what to look for"**
> MITRE ATT&CK = **"how it fits the bigger picture"**

---

## 5. 🗺️ MITRE ATT&CK Framework

A map of attacker behavior across the **full attack lifecycle**.

- **Tactic** = the *goal* (why)
- **Technique** = the *method* (how)

### The Attack Chain

```
Recon --> Resource Dev --> Initial Access --> Execution
                                                  |
                                                  v
Persistence <-- Priv. Escalation <-- Defense Evasion
      |
      v
Credential Access --> Discovery --> Lateral Movement
                                          |
                                          v
                  Collection --> Command & Control
                                          |
                                          v
                  Exfiltration --> Impact
```

| # | Tactic | Meaning |
|---|---|---|
| 1 | Reconnaissance | Gathering intel on target |
| 2 | Resource Development | Building attack infra |
| 3 | Initial Access | Getting in (phishing, exploit) |
| 4 | Execution | Running malicious code |
| 5 | Persistence | Staying in after reboot |
| 6 | Privilege Escalation | Gaining higher access |
| 7 | Defense Evasion | Avoiding detection |
| 8 | Credential Access | Stealing passwords/hashes |
| 9 | Discovery | Mapping the environment |
| 10 | Lateral Movement | Spreading to other systems |
| 11 | Collection | Gathering target data |
| 12 | Command & Control | Remote control channel |
| 13 | Exfiltration | Stealing data out |
| 14 | Impact | Final damage (encrypt/destroy) |

---

## 🔄 How It All Connects

```
Phishing Email (vector: email)
        |
        v
User clicks --> malware runs
        |
        v
Initial Access + Execution   (MITRE tactics)
        |
        v
IOC spotted: weird process / bad hash / C2 IP
        |
        v
Impact --> breaks Confidentiality or Availability (CIA)
```
