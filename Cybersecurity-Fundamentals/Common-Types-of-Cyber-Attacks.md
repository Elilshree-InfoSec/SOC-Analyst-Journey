# 🔐 Common Types of Cyber Attacks — Notes

> A structured reference on the most common cyber attack types, how they work, and how to defend against them.

---

## 📌 What is a Cyber Attack?

A **cyber attack** is a deliberate, malicious attempt to breach the information system of an individual or organization.

- Usually economically motivated (ransom, financial gain)
- Some attacks aim purely at **data destruction**
- Other motives include political/activist purposes

---

## 🔟 Top 10 Common Cyber Attack Types

| # | Attack Type | One-line Summary |
|---|---|---|
| 1 | Malware | Malicious software that breaches, damages, or disables systems |
| 2 | Phishing | Fraudulent messages tricking users into revealing info or installing malware |
| 3 | Man-in-the-Middle (MitM) | Attacker secretly intercepts communication between two parties |
| 4 | Denial-of-Service (DoS/DDoS) | Floods systems with traffic to overwhelm and disable them |
| 5 | SQL Injection | Malicious code inserted via input fields to access protected data |
| 6 | Zero-day Exploit | Exploiting a vulnerability before a patch exists |
| 7 | Password Attack | Attempts to steal or guess credentials |
| 8 | Cross-site Scripting (XSS) | Malicious scripts injected into trusted websites |
| 9 | Rootkits | Hidden malware granting remote admin-level control |
| 10 | IoT Attacks | Exploiting weakly-secured internet-connected devices |

---

## 🦠 Malware — Subtypes

Malware is an umbrella term for malicious software. Common subtypes:

| Type | What It Does |
|---|---|
| Ransomware | Encrypts files, demands payment for decryption key |
| Fileless Malware | Runs in memory only — leaves no files, hard to detect |
| Spyware | Secretly collects user info (credentials, banking, personal data) |
| Adware | Displays unwanted ads, redirects to harmful sites |
| Trojans | Disguised as legitimate software; steals data or grants access |
| Worms | Self-replicating; spreads across networks, consumes bandwidth |
| Rootkits | Alters system behavior to hide malware and evade detection |
| Mobile Malware | Targets phones/tablets for data theft or fraud |
| Exploits | Takes advantage of software vulnerabilities |
| Scareware | Fake warnings that trick users into buying fake antivirus |
| Keylogger | Records keystrokes to steal credentials/card info |
| Botnet | Network of infected devices controlled for large-scale attacks |

---

## 🎣 Phishing — Variants

| Variant | Target/Method |
|---|---|
| Spear Phishing | Targeted at specific individuals or companies |
| Whaling | Targets senior executives/high-value stakeholders |
| Pharming | DNS cache poisoning → fake login pages |
| Vishing | Voice phishing (phone calls) |
| Smishing | SMS/text-based phishing |

💡 Attackers often use **social engineering** — gathering personal/work info to appear credible.

---

## 🕵️ Man-in-the-Middle (MitM) Attacks

- Attacker inserts themselves between two communicating parties
- Commonly exploits **unsecured public Wi-Fi**
- Hard to detect — victim believes traffic is going to a legitimate destination
- Often combined with phishing or malware to execute

---

## 🌊 Denial-of-Service (DoS) vs DDoS

| Aspect | DoS | DDoS |
|---|---|---|
| Source | Single system | Multiple infected host machines |
| Goal | Overload resources, block requests | Take system fully offline |
| Common Variants | TCP SYN flood, teardrop, smurf, ping-of-death | Botnet-driven floods |

---

## 🗄️ SQL Injection

- Malicious code inserted via forms, comment boxes, or search fields
- Forces the server to expose or manipulate protected data

**Prevention:**
- ✅ Use **prepared statements** with parameterized queries
- ✅ Treat parameters strictly as data, never as executable code

---

## ⏱️ Zero-Day Exploit

- Exploits a vulnerability **immediately after discovery**, before a patch exists
- Small window of opportunity for attackers before fixes are deployed

**Mitigation:**
- Continuous monitoring
- Proactive threat detection
- Agile patch management

---

## 🔑 Password Attacks

| Method | Description |
|---|---|
| Brute-force | Programmatically tries all possible password combinations |
| Dictionary Attack | Uses a list of common passwords |
| Social Engineering | Tricks users into revealing passwords |
| Database Breach | Attacker accesses leaked/stolen password databases |
| Network Sniffing | Captures unencrypted passwords over a network |

**Prevention:**
- 🔒 Account lockout after failed attempts
- 🔒 Two-factor authentication (2FA)

---

## 💻 Cross-Site Scripting (XSS)

- Malicious scripts (usually JavaScript, sometimes Flash/HTML) injected into trusted website content
- Executes in the victim's browser via dynamic content

---

## 🕳️ Rootkits

- Hide inside legitimate software
- Grant **remote, admin-level control** once installed
- Remain dormant until activated or triggered
- Commonly spread via email attachments or insecure downloads

---

## 📡 IoT Attacks

- Every connected device = a potential entry point
- IoT devices often have **low built-in security priority**
- 🐟 Real-world example: a casino was breached via an internet-connected **fish tank thermometer**

**Prevention:**
- Keep device OS updated
- Use strong, unique passwords per device
- Rotate passwords regularly

---

## 🛡️ General Mitigation Checklist

| Practice | Purpose |
|---|---|
| Secure coding practices | Prevent injection-based attacks |
| Keep systems/software updated | Close known vulnerabilities |
| Firewalls & threat management tools | Block/monitor malicious traffic |
| Antivirus software | Detect and remove malware |
| Access control & least privilege | Limit damage from compromised accounts |
| Regular backups | Recover from ransomware/data loss |
| Managed Detection & Response (MDR) | Proactively expose, isolate, and eliminate threats |
