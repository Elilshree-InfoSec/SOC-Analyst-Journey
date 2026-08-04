# 🔎 SPL Basics — Quick Cheat Sheet 

---

## 🧠 How SPL works

```
index=windows EventCode=4625 | stats count by user | sort -count
```

Read it left to right. `|` (pipe) = "then do this next." 👉

---

## 🔤 Search Basics

```spl
index=windowslogs                    " search everything in this index
index=windowslogs alice              " search for keyword "alice"
index=windowslogs UserName=alice     " search field UserName = alice
index=windowslogs UserName!=alice    " NOT alice
```

## ⚖️ Compare Values

| Symbol | Meaning |
|---|---|
| `=` | equals |
| `!=` | not equals |
| `>` | greater than |
| `<` | less than |
| `>=` `<=` | greater/less or equal |

## 🔗 AND / OR / NOT

```spl
user=david AND ip=10.10.10.10     " both true
user=david OR user=john           " either
user IN(david, john)              " shortcut for OR
NOT user=*                        " field doesn't exist at all
```

## 🌟 Wildcards

```spl
status=*fail*        " matches failed, failure, appfail...
ip=172.*              " matches anything starting 172.
ip=172.18.0.0/16       " subnet match
```

---

## 🧰 Basic Commands (memorize these first)

| Command | What it does | Example |
|---|---|---|
| `table` | show only these fields | `\| table _time user ip` |
| `fields` | keep/drop fields | `\| fields user ip` or `\| fields - ip` |
| `stats count by` | count how many, grouped | `\| stats count by user` |
| `sort` | order results | `\| sort -count` (- = descending) |
| `head` | first N results | `\| head 20` |
| `dedup` | remove duplicates | `\| dedup ip` |
| `rename` | rename a field | `\| rename user as Employee` |
| `where` | filter with conditions | `\| where count > 20` |
| `top` | most common values  | `\| top user limit=5` |
| `rare` | least common values  | `\| rare user limit=5` |

---

## 🌈 The Big 3 for Investigations

**1️⃣ `stats` — count/group things**
```spl
index=windows EventCode=4625
| stats count by user, src_ip
| sort -count
```
💬 "Show me failed logins, grouped by user and IP, worst first."

**2️⃣ `eval` — make a new field / calculate**
```spl
| eval is_admin = if(match(user, "admin"), "yes", "no")
```
💬 "If username has 'admin' in it, tag it."

**3️⃣ `rex` — pull a field out of messy text**
```spl
| rex field=_raw "src=(?<SrcIP>\d+\.\d+\.\d+\.\d+)"
```
💬 "Grab the IP address that comes after 'src=' in the raw log."

---

## 🕵️ Common Starter Searches

**🚪 Top failed logins:**
```spl
index=windows EventCode=4625
| stats count by user
| sort -count
```

**🔑 Failed SSH attempts by IP:**
```spl
sourcetype=linux_secure "Failed password for"
| stats count by src_ip
| sort -count
```

**🚨 Brute force (5+ fails from one IP):**
```spl
sourcetype=linux_secure "Failed password"
| stats count by src_ip
| where count >= 5
```

**🌍 Enrich IP with location:**
```spl
index=windows
| iplocation src_ip
| stats count by Country
```

**🛡️ Enrich with a lookup table (like threat intel):**
```spl
| lookup threat_intel_ips ip as src_ip OUTPUT threat_category
| where isnotnull(threat_category)
```

---

## ⏰ Time Range

Don't rely on the clock icon for real searches — type it:

```spl
earliest=-1h latest=now      " last 1 hour
earliest=-24h                " last 24 hours
```

---

## 💡 Quick Tips

-   Always filter by `index=` and `sourcetype=` first — it's faster.
-  `stats` = summary table (original events gone).
-  `eventstats` = same thing but keeps the original events too (more advanced, use once `stats` feels easy).
-  Put `|` (pipe) between every step — one action at a time.
-  Test small (`head 20`) before running big/wide searches.

