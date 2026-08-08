# 🗂️ Active Directory — SOC Analyst Notes

> Full reference notes — AD architecture, authentication, key attacks, detection & event IDs

---

## 1. 🏛️ What Is Active Directory?

Active Directory (AD) is Microsoft's **directory service** — a centralized database that stores and manages information about users, computers, groups, and permissions across a network (a "domain").

```
                +---------------------------+
                |     ACTIVE DIRECTORY       |
                |  (identity & access hub)   |
                +---------------------------+
                 /        |         |        \
                v         v         v         v
            Users     Computers   Groups    Policies
```

It's the reason a user can log into any company laptop with one set of credentials, and the reason IT can push settings to thousands of machines at once. It is also the **#1 target** in most modern network intrusions — control AD, control the network.

---

## 2. 🧱 AD Structure — Key Building Blocks

| Term | Definition |
|---|---|
| **Forest** | The top-level container; one or more domains sharing a common schema/config |
| **Domain** | Logical group of users/computers/resources managed under one security boundary |
| **Tree** | A group of domains sharing a contiguous namespace (e.g. `corp.com`, `sales.corp.com`) |
| **OU (Organizational Unit)** | Folder-like container used to organize objects and apply Group Policy |
| **Domain Controller (DC)** | Server that holds a copy of AD and handles authentication requests |
| **Schema** | Defines what object types and attributes can exist in AD |
| **Global Catalog (GC)** | A DC that stores a searchable partial copy of *every* object in the forest |
| **Trust** | A relationship allowing users in one domain to access resources in another |

```
FOREST
 └── corp.com (Domain / Tree root)
       ├── OU: Finance
       │     ├── User: alice
       │     └── Computer: FIN-PC01
       ├── OU: IT
       │     ├── User: bob (Domain Admin)
       │     └── Computer: DC01  <-- Domain Controller
       └── OU: Sales
             └── sales.corp.com (Child Domain)
```

---

## 3. 👤 AD Objects

| Object Type | Description |
|---|---|
| **User** | An account representing a person (or service) |
| **Computer** | An account representing a machine joined to the domain |
| **Group** | A collection of users/computers for permission assignment |
| **GPO (Group Policy Object)** | A set of rules applied to users/computers (password policy, software restrictions, etc.) |
| **Service Account** | Non-human account used to run services — often over-privileged, high attack value |

**Group types worth knowing:**

| Group | Privilege Level |
|---|---|
| Domain Users | Standard access |
| Domain Admins | Full control of the domain — **highest priority to monitor** |
| Enterprise Admins | Full control across the entire forest |
| Backup Operators | Can bypass file permissions for backup — often abused |
| Server Operators | Can manage servers — privilege escalation risk |

---

## 4. 🔑 Authentication Protocols

### Kerberos (default, modern AD auth)

Ticket-based system — passwords are never sent across the network directly.

```
  Client                 KDC (Domain Controller)              Service
    |                           |                                  |
    | 1. Request TGT            |                                  |
    |-------------------------->|                                  |
    |     (AS-REQ / AS-REP)     |                                  |
    | 2. Receive TGT            |                                  |
    |<--------------------------|                                  |
    | 3. Request Service Ticket |                                  |
    |-------------------------->|                                  |
    |     (TGS-REQ / TGS-REP)   |                                  |
    | 4. Receive TGS            |                                  |
    |<--------------------------|                                  |
    | 5. Present TGS to Service ---------------------------------->|
    |                                                              |
    | 6. Access Granted <------------------------------------------|
```

- **KDC** (Key Distribution Center) = runs on every DC
- **TGT** (Ticket Granting Ticket) = proves you authenticated
- **TGS** (Ticket Granting Service ticket) = proves you can access a specific service
- **KRBTGT account** = special account that signs/encrypts all Kerberos tickets — if its hash is stolen, attacker can forge unlimited tickets (Golden Ticket)

### NTLM (legacy, still around for backward compatibility)

Challenge-response protocol, weaker than Kerberos, still common in older environments — a common downgrade target for attackers.

### LDAP

Protocol used to **query and modify** AD (e.g. "find all users in Finance OU"). Heavily used for legitimate admin tasks *and* attacker recon.

---

## 5. 📜 Group Policy (GPO)

- Applied at **Site → Domain → OU** level (in that processing order)
- Controls password policies, software deployment, login scripts, firewall rules, etc.
- Attackers abuse GPOs for:
  - **Persistence** — pushing malicious scripts/scheduled tasks to all machines
  - **Lateral movement** — deploying malware domain-wide in one shot

```
Site Policy --> Domain Policy --> OU Policy --> Final Applied Policy
   (applied first)                          (applied last, wins conflicts)
```

---

## 6. 🔄 Replication & FSMO Roles

DCs replicate data between each other to stay in sync. Certain roles can only exist on **one** DC at a time (FSMO = Flexible Single Master Operations):

| FSMO Role | Purpose |
|---|---|
| Schema Master | Controls changes to the AD schema |
| Domain Naming Master | Controls adding/removing domains in the forest |
| RID Master | Allocates unique security IDs for new objects |
| PDC Emulator | Handles time sync, password changes, account lockouts |
| Infrastructure Master | Keeps cross-domain object references up to date |

> As Tier 1 you won't manage these, but know that **DCSync attacks abuse the replication process** to steal password hashes.

---

## 7. 🎯 Major AD Attacks

| Attack | How It Works | What You'd See |
|---|---|---|
| **Password Spraying** | Try 1 common password across many accounts to avoid lockout | Many accounts, same password, low attempts each |
| **Kerberoasting** | Request service tickets for accounts with SPNs, crack offline | Spike in TGS requests (Event ID 4769), especially for service accounts |
| **AS-REP Roasting** | Target accounts with "no pre-auth required" set, crack ticket offline | AS-REQ without prior authentication (Event ID 4768) |
| **Pass-the-Hash (PtH)** | Authenticate using a stolen NTLM hash, no plaintext password needed | NTLM logins with no matching interactive logon pattern |
| **Pass-the-Ticket (PtT)** | Reuse a stolen Kerberos ticket to impersonate a user | Ticket reused across different hosts/sessions |
| **Golden Ticket** | Forge a TGT using a stolen KRBTGT hash — near-total forest control | Tickets with abnormal lifetimes; logons with no matching AS-REQ |
| **Silver Ticket** | Forge a TGS for a specific service (doesn't need KRBTGT) | Access to a service with no corresponding TGT request |
| **DCSync** | Impersonate a DC to request password hash replication | Replication requests from a non-DC IP/host |
| **Privilege Escalation via Groups** | Attacker adds an account to Domain Admins / similar | Sudden group membership change (Event ID 4728/4732) |
| **GPO Abuse** | Modify a GPO to push malicious script/task domain-wide | Unexpected GPO edits, especially by non-admin accounts |

### Simplified Attack Path

```
Phishing / Initial Access
        |
        v
Recon via LDAP (enumerate users/groups)
        |
        v
Credential Access (Kerberoasting / PtH / Password Spray)
        |
        v
Privilege Escalation (add to Domain Admins)
        |
        v
Lateral Movement (PtT across machines)
        |
        v
Domain Dominance (DCSync / Golden Ticket)
```

---

## 8. 📋 Key Windows Event IDs (Your Bread & Butter)

| Event ID | Meaning | Why It Matters |
|---|---|---|
| **4624** | Successful logon | Baseline — check logon type & source |
| **4625** | Failed logon | Brute-force / spraying indicator |
| **4634** | Logoff | Session tracking |
| **4672** | Admin logon (special privileges assigned) | High-priority — track who's logging in with admin rights |
| **4720** | Account created | Could indicate attacker persistence |
| **4726** | Account deleted | Anti-forensics / cleanup activity |
| **4728 / 4732 / 4756** | Member added to security-enabled group | Privilege escalation red flag |
| **4738** | User account changed | Could indicate tampering (e.g. password never expires set) |
| **4768** | Kerberos TGT requested (AS-REQ) | Watch for AS-REP Roasting patterns |
| **4769** | Kerberos service ticket requested (TGS-REQ) | Watch for Kerberoasting spikes |
| **4771** | Kerberos pre-auth failed | Related to brute-force / spraying |
| **4776** | NTLM authentication attempt | Legacy auth usage, PtH indicator |
| **5136** | Directory object modified | GPO/AD object tampering |

**Logon Types (appears alongside 4624/4625) worth knowing:**

| Type | Meaning |
|---|---|
| 2 | Interactive (physical console login) |
| 3 | Network (e.g. accessing a file share) |
| 4 | Batch |
| 5 | Service |
| 10 | RemoteInteractive (RDP) |

---

## 9. 🛠️ Tools You'll Encounter

| Tool | Use |
|---|---|
| **Event Viewer** | Reviewing local/DC security logs |
| **PowerShell (Get-ADUser, Get-ADGroupMember, etc.)** | Querying AD directly |
| **Sysmon** | Enhanced endpoint logging (process creation, network conns) |
| **BloodHound** | Attackers (and defenders) map AD attack paths / privilege relationships |
| **Mimikatz** | Attacker tool for credential dumping (PtH, Golden Ticket) — know it by name, you'll see it in alerts |
| **SIEM correlation rules** | Pre-built detections for the attacks above (e.g. "Kerberoasting detected") |

---

## 10. 🔎 AD Investigation Skills

This is the part that actually matters day-to-day. Theory tells you *what* an attack is — investigation skill is being able to look at a raw event and figure out **is this bad, and why**.

For any authentication/AD event, work through these six questions:

| Question | What You're Checking |
|---|---|
| **WHO** | Username — privileged? service account? disabled/deleted account? |
| **WHAT** | Successful or failed? Kerberos or NTLM? |
| **WHERE** | Source IP, source workstation, destination host, which DC was involved? |
| **WHEN** | Exact timestamp, frequency, timeline before/after |
| **HOW** | Logon Type, auth package, ticket activity, NTLM usage |
| **NORMAL?** | Normal user, workstation, time, source, and attempt count for this account? |

> If you can't answer most of these from the raw log, that's your cue to pivot and pull more context (source host history, related events, asset owner) before deciding severity.

### General Investigation Flow

```
        SUSPICIOUS EVENT
               |
               v
        Identify Event ID
               |
               v
        WHO was targeted?
               |
               v
      Source IP / Host / DC
               |
               v
      How many attempts / how often?
               |
               v
     One account or many accounts?
               |
               v
   Any successful logon (4624) after?
               |
               v
   Related events (4771 / 4776 / 4769)?
               |
               v
        Is this behavior normal?
               |
               v
   TRUE POSITIVE  <--------->  FALSE POSITIVE
               |
               v
        Document findings
```

### Attack → Evidence Mapping

**Password Spraying**

```
Many users --> failed auth --> same source --> few attempts per user
                        |
                        v
                 Look at: 4625 / 4771
                        |
                        v
     Investigate: source IP --> targeted accounts --> timestamps
                   --> any successful login afterward?
```

**Kerberoasting**

```
One user --> requests many service tickets --> targets SPNs/service accounts
                        |
                        v
                 Look at: 4769 (TGS-REQ)
                        |
                        v
       Investigate: ticket encryption type (RC4 = red flag)
                   --> volume of requests in short window
```

**DCSync**

```
Non-DC machine --> requests directory replication data
                        |
                        v
             Look at: replication-related events
                        |
                        v
        Investigate: source host — is it actually a DC?
                   --> treat as high severity if not
```

> These three patterns cover a large share of what you'll actually be asked to triage as Tier 1 — worth practicing until they're second nature.

---

## 11. ✅ Tier 1 Priority Checklist

- Know the **key Event IDs** above cold (4624, 4625, 4672, 4768, 4769, 4728/4732)
- Always treat **Domain Admin / Enterprise Admin** activity as high priority
- Recognize **Kerberoasting** (TGS request spikes) and **password spraying** (many accounts, one password) patterns
- Understand that **DC-related alerts = automatically high severity**
- Know that **group membership changes** are a classic privilege escalation signal
- Recognize tool names like **Mimikatz** and **BloodHound** in alerts/IOCs
- Understand the basic **attack path**: Recon → Credential Access → Priv Esc → Lateral Movement → Domain Dominance

---

*Notes for personal SOC study — Active Directory fundamentals*
