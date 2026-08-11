# 🎣 Types of Phishing — Notes

> A structured reference on common phishing attack types, real-world examples, and prevention tips.

---

## 📌 What is Phishing?

**Phishing** is a cyberattack where criminals use deceptive tactics to trick individuals into divulging sensitive information — via fake emails, texts, calls, websites, or social media.

---

## 🔢 The 11 Types at a Glance

| # | Type | Delivery Method |
|---|---|---|
| 1 | Email Phishing | Email |
| 2 | Spear Phishing | Targeted email |
| 3 | Whaling | Targeted email (executives) |
| 4 | Smishing | SMS/text |
| 5 | Vishing | Phone call |
| 6 | Business Email Compromise (CEO Fraud) | Compromised email account |
| 7 | Clone Phishing | Duplicated/resent email |
| 8 | Evil Twin Phishing | Fake Wi-Fi network |
| 9 | Social Media Phishing | Social platforms/DMs |
| 10 | Search Engine Phishing | Fake indexed websites |
| 11 | Pharming | DNS redirection |

---

## 📧 1. Email Phishing

Impersonates trusted entities via email, often urging urgent action to steal credentials.

| Detail | Description |
|---|---|
| Tactic | Fake urgency (e.g. "account compromised") |
| Goal | Get victim to click a link → fake login page |
| Example | Fake "Gmail customer care" email (@Gm@il.com) leading to a spoofed login page |

---

## 🎯 2. Spear Phishing

Highly personalized emails targeting **specific individuals** within an organization.

| Detail | Description |
|---|---|
| Target | Chosen employees/companies (not mass emails) |
| Example | *Star Blizzard* (linked to Russian FSB) targeted U.S./U.K. government & defense orgs |
| Defense | Strong passwords, MFA, cautious link-checking, email scanning |

---

## 🐋 3. Whaling

Targets **high-profile executives** ("whales") like CEOs and CFOs.

| Detail | Description |
|---|---|
| Target | C-suite / high-level executives |
| Tactic | High-pressure scenarios (e.g. "company being sued") |
| Example | Fake "CFO" email to CEO requesting sensitive merger financial info |

---

## 📱 4. Smishing (SMS Phishing)

Uses text messages instead of email to lure victims.

| Detail | Description |
|---|---|
| Bait | Coupon codes, prize offers, delivery failure alerts |
| Example | **SNS Sender** tool used AWS to send fake USPS texts, harvesting names, addresses, card numbers |

---

## 📞 5. Vishing (Voice Phishing)

Uses **phone calls** to deceive victims into revealing info.

| Detail | Description |
|---|---|
| Impersonates | Banks, government agencies, tech support |
| Tactic | Fear/urgency to pressure disclosure |
| Example | Fake "bank fraud department" call requesting account verification details |

---

## 🏢 6. Business Email Compromise (CEO Fraud)

Attacker gains access to a **real executive email account** and uses it to defraud employees.

| Detail | Description |
|---|---|
| Method | Compromised legitimate executive account |
| Goal | Fraudulent wire transfers, fake invoices |
| Example | Finance employee wires funds after a fake "CEO" request |

---

## 👯 7. Clone Phishing

Recreates a **legitimate, previously received email** with malicious links/attachments swapped in.

| Detail | Description |
|---|---|
| Pretext | "Resending due to a broken link/attachment" |
| Example | Fake Blockworks/Etherscan sites tricked users into connecting crypto wallets, draining funds |

---

## 📶 8. Evil Twin Phishing

Sets up a **fake Wi-Fi network** mimicking a legitimate one.

| Detail | Description |
|---|---|
| Setup | Rogue hotspot (e.g. fake cafe Wi-Fi) |
| Risk | Intercepts traffic, redirects to phishing login pages |
| Example | Fake cafe Wi-Fi intercepts email login → fake "verify identity" page steals credentials |

---

## 📲 9. Social Media Phishing

Uses fake accounts/DMs on platforms like Facebook, Twitter, Instagram.

| Detail | Description |
|---|---|
| Tactic | Impersonates friends or brand support accounts |
| Example | Fake "friend" DM with a video/photo link → fake login page steals credentials |

---

## 🔍 10. Search Engine Phishing

Malicious sites get indexed on legitimate search engines with "too-good" deals.

| Detail | Description |
|---|---|
| Bait | Cheap products, incredible deals |
| Example | Fake "Am@zon.com" result mimicking Amazon's real site |

---

## 🌐 11. Pharming

Redirects users to fraudulent sites by exploiting **DNS servers**, without needing a clicked link.

| Detail | Description |
|---|---|
| Method | DNS server compromise or malware-tampered DNS settings |
| Example | Typing a real bank URL still redirects to a fake, identical-looking site |

---

## 🚩 Warning Signs of Phishing

| Sign | Why It Matters |
|---|---|
| Requests to "confirm" personal info | Legitimate orgs rarely ask this via email |
| Poor grammar/spelling | Common red flag of scam origin |
| High-pressure/urgent tone | Designed to bypass careful thinking |
| Unexpected links or attachments | Common malware delivery method |
| Too-good-to-be-true offers | Classic bait tactic |

---

## 🛡️ Prevention Tips

| Tip | Action |
|---|---|
| Be skeptical of unsolicited emails | Verify sender via a trusted, separate channel |
| Check URLs carefully | Hover to preview links before clicking |
| Use multi-factor authentication (MFA) | Adds protection even if credentials leak |
| Keep software updated | Patches known vulnerabilities |
| Educate & train | Build awareness for yourself, family, or team |
| Use antivirus software | Extra layer of defense against malicious links/files |
