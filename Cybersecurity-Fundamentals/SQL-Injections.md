# 🗄️ SQL Injection (SQLi) — SOC Analyst Notes

> Defensive-focused reference: what SQLi looks like in logs/alerts, detection signatures, and triage. Offensive/exploitation techniques are summarized only at a conceptual level — deep-dive on payload crafting is for later, on the pentesting side.

---

## 📌 What is SQL Injection?

**SQL injection (SQLi)** is a web security vulnerability where an attacker manipulates the queries an application sends to its database — potentially exposing, modifying, or deleting data, and in some cases compromising the backend server itself.

**MITRE ATT&CK:** T1190 (Exploit Public-Facing Application)

---

## 💥 Impact of a Successful Attack

| Impact | Detail |
|---|---|
| Unauthorized data access | Passwords, credit card details, personal user information |
| Data tampering | Modify or delete data → persistent changes to app behavior |
| Backend compromise | Can escalate to compromise the underlying server/infrastructure |
| Denial-of-service | Attackers can disrupt availability |
| Persistent backdoor | Long-term, unnoticed compromise of systems |
| Business impact | Reputational damage, regulatory fines (seen in many high-profile breaches) |

---

## 🔍 Where SQLi Typically Shows Up in Logs

| Query Type | Injection Point | Log Source |
|---|---|---|
| SELECT | WHERE clause (most common) | WAF, web access logs |
| SELECT | Table/column name, ORDER BY clause | WAF, DB audit logs |
| UPDATE | Updated values or WHERE clause | DB audit logs |
| INSERT | Inserted values | DB audit logs |

---

## 🚩 Detection Signatures — What to Alert On

| Pattern | What It Suggests |
|---|---|
| `' OR 1=1--`, `' OR '1'='1` | Logic bypass / hidden data retrieval attempt |
| `UNION SELECT`, `UNION ALL SELECT` | Attempt to pull data from other tables |
| `information_schema`, `all_tables`, `all_tab_columns`, `sysobjects` | Database structure reconnaissance |
| `--`, `#`, `/* */` in input fields | Comment injection to truncate queries |
| `WAITFOR DELAY`, `SLEEP(`, `pg_sleep(`, `dbms_pipe.receive_message` | Time-based blind SQLi probing |
| `xp_dirtree`, `xp_cmdshell`, `UTL_INADDR.get_host_address`, `LOAD_FILE` | Out-of-band (OAST) exfiltration attempts via DNS/file access |
| `CAST(`, `CONVERT(`, `EXTRACTVALUE(` | Error-based SQLi attempting to leak data via error messages |
| Repeated single-quote (`'`) submissions to same field | Manual fuzzing / vulnerability probing |
| Encoded payloads (URL-encoded, XML entity-encoded keywords) | WAF filter evasion attempt |

**High-fidelity alert combo:** DB error burst + abnormal response time variance + repeated requests from the same source IP/session = strong SQLi indicator, not just noise.

---

## 🧪 Attack Categories (Conceptual Overview)

| Type | What It Does | SOC Relevance |
|---|---|---|
| Retrieving hidden data | Modifies a query to return additional/unreleased results | Look for comment sequences (`--`) in parameters |
| Subverting application logic | Bypasses logic checks (e.g. login without a password) | Watch for authentication anomalies tied to malformed input |
| UNION attacks | Retrieves data from other database tables | Look for `UNION SELECT` in logs, spikes in returned data volume |
| Blind SQL injection | Exploits without seeing query results directly | Detected via timing anomalies, DNS beacon attempts, or conditional error patterns |
| Second-order (stored) SQLi | Malicious input stored safely, then unsafely reused later | Harder to detect — requires tracing data flow across requests/time |

💡 **Blind SQLi note:** Since there's no visible output, attackers rely on timing, error, or out-of-band (DNS) signals — meaning **DNS logs and response-time anomalies** are just as important as WAF logs for catching it.

---

## 🧬 Recon Activity to Watch For

Attackers fingerprint the database before/during exploitation — these queries in logs are a red flag even if the "attack" itself hasn't succeeded yet:

| Info Sought | Example Query Pattern |
|---|---|
| DB version | `@@version`, `v$version`, `version()` |
| Table names | `information_schema.tables`, `all_tables` |
| Column names | `information_schema.columns`, `all_tab_columns` |

---

## 🧾 Don't Overlook Non-URL Injection Points

- SQLi isn't limited to URL query strings — check **JSON and XML body payloads** too
- Attackers may encode/escape characters (e.g. XML entity encoding of `SELECT`) specifically to **bypass WAF keyword filters** — raw request body inspection matters more than surface-level parameter checks

---

## 🛡️ Verifying Prevention Controls (What "Good" Looks Like)

| Defense | What to Verify Is in Place |
|---|---|
| **Parameterized queries / prepared statements** | The primary defense — confirm dev teams use these, not string concatenation |
| Least privilege DB accounts | App's DB user shouldn't have admin/DDL rights it doesn't need |
| WAF rules tuned for SQLi | Confirm signatures cover UNION, time-based, and encoded payload patterns |
| DB activity monitoring | Ensure query auditing is enabled for sensitive tables |
| Input validation / allow-listing | For values that can't be parameterized (table/column names, ORDER BY) |

**Vulnerable pattern to flag in code review:**
```java
String query = "SELECT * FROM products WHERE category = '" + input + "'";
```

**Safe pattern:**
```java
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
```

---

## 🛡️ SOC Triage Checklist for a SQLi Alert

| Step | Action |
|---|---|
| 1 | Confirm the payload pattern in the raw request (not just alert summary) |
| 2 | Check WAF action — was it blocked or did it pass through? |
| 3 | If passed through, check DB audit logs for corresponding query execution |
| 4 | Check for data returned / anomalous response size or timing |
| 5 | Identify source IP — check reputation, prior activity, geolocation |
| 6 | Check for recon patterns (information_schema queries) preceding the attempt |
| 7 | If successful, scope what data/tables may have been accessed |
| 8 | Escalate per severity — confirmed data access = high/critical |
| 9 | Recommend/verify WAF rule or code-level fix to close the gap |

---

## 🧠 Quick Recap

| Concept | Key Takeaway |
|---|---|
| Root cause | Untrusted input treated as part of SQL code instead of data |
| Hardest variant to catch | Out-of-band blind SQLi — rely on DNS logs + timing, not just WAF alerts |
| Best defense to verify | Parameterized queries / prepared statements |
| Common SOC mistake | Only monitoring WAF logs and missing DB audit logs / DNS-based exfiltration |
