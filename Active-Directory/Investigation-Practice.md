# 🧪 AD Investigation Practice — Tier 1 Scenarios

> Realistic (fake) log snippets to practice the WHO/WHAT/WHERE/WHEN/HOW/NORMAL? framework.
> For each scenario: read the logs, work through the questions, decide **True Positive 🔴** or **False Positive 🟢** — then click the answer to check yourself.

---

## 🧭 How to Use This

For every scenario:
1. Identify the Event ID(s) involved
2. Walk through WHO / WHAT / WHERE / WHEN / HOW / NORMAL?
3. Decide: True Positive or False Positive — and why
4. Write one line as if escalating to Tier 2

---

## Scenario 1 — Multiple Failed Logins 🔑

```
EventID=4625  Account=jsmith        SourceIP=203.0.113.44  Time=02:14:03
EventID=4625  Account=jsmith        SourceIP=203.0.113.44  Time=02:14:05
EventID=4625  Account=jsmith        SourceIP=203.0.113.44  Time=02:14:07
EventID=4625  Account=jsmith        SourceIP=203.0.113.44  Time=02:14:09
EventID=4624  Account=jsmith        SourceIP=203.0.113.44  Time=02:14:12  LogonType=10
```

- Normal working hours for jsmith: 9am–6pm, logs in from internal IP only.
- Logon Type 10 = RemoteInteractive (RDP).

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Brute Force**

4 failed logins in 6 seconds followed by a success, at 2am, via RDP, from an external-looking source, for a user who never logs in at that hour or via RDP. Classic brute-force success pattern.

**Escalation note:** *"Possible successful brute-force against jsmith via RDP outside normal hours — recommend immediate password reset and session review."*

</details>

---

## Scenario 2 — Kerberos Ticket Requests 🎫

```
EventID=4769  Account=svc_backup   Service=SPN/FileServer01   Time=10:02:01
EventID=4769  Account=svc_backup   Service=SPN/SQLServer02    Time=10:02:03
EventID=4769  Account=svc_backup   Service=SPN/MailServer     Time=10:02:04
EventID=4769  Account=svc_backup   Service=SPN/AppServer03    Time=10:02:06
EventID=4769  Account=svc_backup   Service=SPN/WebServer01    Time=10:02:07
EventID=4769  Account=svc_backup   Service=SPN/DevServer      Time=10:02:09
EncryptionType=RC4
```

- `svc_backup` normally requests 1 ticket per night for a scheduled backup job to FileServer01 only, around 1am.
- All 6 requests came within 8 seconds, at 10am, encryption type RC4 (weaker/legacy).

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Kerberoasting**

6 service ticket requests for 6 different SPNs within 8 seconds, using weak RC4 encryption, completely outside the account's normal single-ticket nightly pattern. Textbook Kerberoasting.

**Escalation note:** *"svc_backup shows Kerberoasting pattern — 6 TGS requests in 8s, RC4 encryption, outside baseline behavior."*

</details>

---

## Scenario 3 — Password Spraying Pattern 🌊

```
EventID=4625  Account=user01  SourceIP=198.51.100.9  Time=03:00:01
EventID=4625  Account=user02  SourceIP=198.51.100.9  Time=03:00:04
EventID=4625  Account=user03  SourceIP=198.51.100.9  Time=03:00:07
EventID=4625  Account=user04  SourceIP=198.51.100.9  Time=03:00:10
EventID=4625  Account=user05  SourceIP=198.51.100.9  Time=03:00:13
... (47 more accounts, 1 attempt each, same source IP, 3-second intervals)
EventID=4624  Account=user23  SourceIP=198.51.100.9  Time=03:02:40  LogonType=3
```

- SourceIP `198.51.100.9` is an external IP, not part of the corporate network.
- `user23` is a regular helpdesk account.

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Password Spraying**

Many different accounts, one attempt each, same external source IP, evenly spaced — designed to avoid lockout thresholds. A success on the 23rd account confirms compromise.

**Escalation note:** *"Password spray from external IP 198.51.100.9 succeeded against user23 — recommend forced password reset for all targeted accounts and IP block."*

</details>

---

## Scenario 4 — Privileged Group Change 👑

```
EventID=4728  Group=Domain Admins   MemberAdded=t.wilson   ActorAccount=admin_helpdesk1   Time=15:41:22
```

- `admin_helpdesk1` is a helpdesk-tier admin account, normally only resets passwords and unlocks accounts — has no history of modifying Domain Admins group.
- `t.wilson` is a marketing department user, no prior admin history.
- Change occurred outside a documented change window.

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Privilege Escalation**

A helpdesk-tier account (which shouldn't be able to, or normally doesn't, touch Domain Admins) added a marketing user to Domain Admins, outside a change window, with no history to support it.

**Escalation note:** *"Unauthorized addition of t.wilson to Domain Admins by helpdesk-tier account admin_helpdesk1 — very high severity, escalate immediately."*

</details>

---

## Scenario 5 — Suspicious Replication Request 🕵️

```
EventID=4662  Operation="Replicating Directory Changes"  RequestingHost=MKT-LAPTOP07  Time=22:10:05
```

- `MKT-LAPTOP07` is a marketing department laptop — not a Domain Controller.
- Only Domain Controllers should ever legitimately request directory replication.
- No scheduled maintenance or IT ticket on file for this time.

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Possible DCSync**

A non-DC laptop requesting directory replication is one of the highest-severity patterns in AD — this is the signature of a DCSync attack.

**Escalation note:** *"Non-DC host MKT-LAPTOP07 requested directory replication — possible DCSync, treat as critical and isolate host."*

</details>

---

## Scenario 6 — AS-REP Roasting Indicator 🍞

```
EventID=4768  Account=c.nguyen  PreAuthType=None  Time=11:15:00
EventID=4768  Account=c.nguyen  PreAuthType=None  Time=11:15:02
EventID=4768  Account=c.nguyen  PreAuthType=None  Time=11:15:04
```

- `c.nguyen`'s account has "Do not require Kerberos preauthentication" enabled (a legacy/misconfigured setting).
- Requests came from a workstation `c.nguyen` has never used before.
- No corresponding helpdesk ticket for a new device.

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — AS-REP Roasting**

Repeated AS-REQ with no pre-authentication for an account with the vulnerable setting enabled, from an unfamiliar workstation, no supporting ticket.

**Escalation note:** *"c.nguyen shows AS-REP Roasting pattern from unrecognized host — recommend disabling 'no pre-auth' and investigating source workstation."*

</details>

---

## Scenario 7 — False Positive Check: Legitimate Admin Activity 🟢

```
EventID=4672  Account=it_admin_mreyes  SourceIP=10.10.4.22 (internal)  Time=09:32:00
EventID=4728  Group=Server Operators   MemberAdded=new_hire_dtan   ActorAccount=it_admin_mreyes  Time=09:35:10
```

- `it_admin_mreyes` is a Tier 3 IT admin with regular history of managing group memberships.
- Change ticket #INC00457 exists, approved, for onboarding `dtan` to Server Operators as part of new-hire setup.
- Occurred during business hours from a known internal admin workstation.

<details>
<summary>✅ Reveal Answer</summary>

**🟢 FALSE POSITIVE — Legitimate Activity**

Known Tier 3 admin, documented approved change ticket, business hours, internal known workstation, consistent with role.

**Escalation note:** *"Group change matches approved ticket INC00457 — closing as benign, no further action needed."*

</details>

---

## Scenario 8 — Odd Logon Type on a Service Account 🤖

```
EventID=4624  Account=svc_sql01  LogonType=10 (RemoteInteractive/RDP)  SourceIP=203.0.113.201 (external)  Time=01:47:00
```

- `svc_sql01` is a service account meant only for automated SQL Server processes — it should never interactively log in, let alone via RDP.
- Source IP is external, not a known VPN range.

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Service Account Misuse**

Service accounts should never RDP in interactively, especially not from an external IP at 1:47am. Strong indicator of compromise/misuse.

**Escalation note:** *"svc_sql01 logged in interactively via RDP from external IP — likely credential compromise, escalate as high priority."*

</details>

---

## Scenario 9 — High Volume of Account Creation 🏗️

```
EventID=4720  Account=temp_user1  CreatedBy=svc_provisioning  Time=14:00:00
EventID=4720  Account=temp_user2  CreatedBy=svc_provisioning  Time=14:00:03
EventID=4720  Account=temp_user3  CreatedBy=svc_provisioning  Time=14:00:05
EventID=4720  Account=temp_user4  CreatedBy=svc_provisioning  Time=14:00:07
```

- `svc_provisioning` is the HR automation account that creates accounts for new hires during batch onboarding, typically Mondays 9am.
- Today is Wednesday, 2pm — no onboarding batch scheduled.
- Usernames follow a generic `tempuserN` pattern, not the company's standard `firstname.lastname` convention.

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Possible Persistence**

Wrong day, wrong time, non-standard naming convention, all inconsistent with the known onboarding process. Could be an attacker creating backdoor accounts.

**Escalation note:** *"4 accounts created outside onboarding schedule with non-standard naming — possible persistence, verify with HR and investigate svc_provisioning for compromise."*

</details>

---

## Scenario 10 — NTLM Fallback ⬇️

```
EventID=4776  Account=r.patel  SourceHost=FIN-PC04  Time=10:05:00  AuthPackage=NTLM
```

- Domain policy requires Kerberos wherever possible; NTLM should only appear for legacy systems or specific known exceptions.
- `FIN-PC04` and `r.patel` have no history of NTLM usage — always Kerberos in past 90 days of logs.
- No legacy application on this host that would require NTLM fallback.

<details>
<summary>✅ Reveal Answer</summary>

**🔴 TRUE POSITIVE — Possible Pass-the-Hash / Downgrade**

A host/user with a clean 90-day Kerberos-only history suddenly using NTLM, with no legacy application to justify it, is a common sign of an attacker forcing an NTLM downgrade to enable Pass-the-Hash.

**Escalation note:** *"Unexpected NTLM fallback on FIN-PC04 with no legacy justification — possible downgrade attack, investigate host for compromise."*

</details>

