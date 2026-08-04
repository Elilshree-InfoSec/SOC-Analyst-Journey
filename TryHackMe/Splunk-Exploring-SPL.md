# 🔍 TryHackMe — Splunk: Exploring SPL

## 🎯 Objectives
- Understand **Search Processing Language (SPL)**.
- Search, filter & transform log data.
- Visualize with charts/stats.
- Detect anomalies with SPL.

> 💡 Networks make **thousands of logs/min**. SPL lets you **filter → organize → summarize → visualize** so only relevant events surface.

---

## 🧠 1. The Pipeline Model (Most Important Concept)
Commands are chained with a **pipe `|`** — output of one becomes input of the next.

```spl
index=windowslogs               # 1. get raw events
| fields User SourceIp          # 2. keep only these columns
| dedup SourceIp                # 3. remove duplicate IPs
| table User SourceIp           # 4. display cleanly
```

---

## 🖥️ 2. Search & Reporting Interface

| Component | Purpose |
|-----------|---------|
| Search Head | Where you type SPL |
| Time Picker | Select timeframe (smaller = faster) |
| Search History | Reuse old queries |
| Data Summary | Lists hosts, sources, sourcetypes |
| Fields Sidebar | Click fields to build queries fast |

**Field Types:** `#` = numeric (can do math) · `α` = text/string · *Selected* = shown by default · *Interesting* = auto-detected (>20% of events).

---

## 🔎 3. Basic Searches

```spl
index=windowslogs               # everything in this index
index=windowslogs alice         # free-text: "alice" ANYWHERE in logs
```
> ⚠️ Always start with `index=` to avoid slow full-data scans.

---

## ⚔️ 4. Operators

### Relational
| Op | Meaning | Example |
|----|---------|---------|
| `=` / `!=` | equal / not equal | `User!=SYSTEM` |
| `>` `>=` | greater / or equal | `Count>=10` |
| `<` `<=` | less / or equal | `Age<20` |

### Logical
| Op | Meaning | Example |
|----|---------|---------|
| `AND` | both match | `User=James AND EventID=4624` |
| `OR` | either matches | `User=James OR User=John` |
| `NOT` | term/field absent | `NOT User=*` |
| `IN` | match a list | `User IN (James, John)` |

### Wildcards & CIDR
| Syntax | Purpose | Example |
|--------|---------|---------|
| `*` | partial text | `status=*fail*` |
| `*` | partial IP | `DestinationIp=172.*` |
| CIDR | whole subnet | `DestinationIp=172.18.0.0/16` |

```spl
"failed login"                  # exact phrase (order matters)
(alice AND bob) OR charlie      # () controls evaluation order
```

---

## 🎯 5. Filtering Commands (Trim the Noise)

| Command | Purpose |
|---------|---------|
| `fields` | include/exclude columns |
| `dedup` | remove duplicate values |
| `rename` | rename fields (readability / flatten JSON) |
| `regex` | filter by pattern (PCRE) |

```spl
index=windowslogs
| fields host User SourceIp     # keep only these 3
| fields - _raw                 # '-' EXCLUDES the bulky _raw field

index=windowslogs
| dedup SourceIp                # 7 unique IPs -> 7 events (cleans repeat logs, e.g. M365)

index=jsondata
| rename request.* as *         # flatten JSON: request.path -> path

index=windowslogs
| regex Image="\.exe$"          # \. = literal dot, $ = end of string -> only .exe
```

---

## 📑 6. Structuring Commands (Organize Output)

| Command | Purpose |
|---------|---------|
| `table` | clean column view (great for timelines) |
| `head N` | first (newest) N events |
| `tail N` | last (oldest) N events |
| `sort` | sort (use `sort -field` for descending) |
| `reverse` | flip event order |

```spl
index=windowslogs Hostname=Salena.Adam
| table _time Hostname EventID Category   # pick timeline columns
| reverse                                 # oldest -> newest = story order
```

---

## 🔗 7. Subsearch + `join` (Correlate 2 Sources)
**Why:** Sysmon (`EventID=1`) logs process creation but NOT `LogonType`. Security (`EventID=4624`) has it. Shared key = `LogonId`.

```spl
index=windowslogs EventID=1                       # MAIN: process creation events
| join LogonId                                    # link on LogonId
    [ search index=windowslogs EventID=4624       # SUBSEARCH runs FIRST
    | rename TargetLogonId as LogonId             # match Sysmon field name
    | fields LogonId LogonType IpAddress ]        # bring back these fields
| table _time Image User LogonType IpAddress      # unified result
```
> ⚠️ `join` is **slow on big data** — prefer `stats`/`eval` when possible. Use join to enrich source A with fields from source B.

---

## 📊 8. Transforming Commands (Summarize & Visualize)

| Command | Purpose |
|---------|---------|
| `top` / `rare` | most / least frequent values |
| `highlight` | mark keywords (switch view to Raw) |
| `stats` | aggregate math |
| `chart` | table ready for visualization |
| `timechart` | trends over time |

```spl
index=windowslogs | top User limit=5      # 5 MOST frequent users
index=windowslogs | rare User limit=5     # 5 LEAST frequent (often suspicious!)

index=windowslogs
| stats count by EventID                  # count events per EventID
| sort EventID

index=windowslogs | chart count by User   # visualize count per user

index=windowslogs Image!=""               # drop empty Image
| timechart span=30m count by Image limit=5   # 30-min buckets, top 5 processes
```

**`stats` functions:** `count` · `avg()` · `sum()` · `max()` · `min()`

| Command | X-axis | Best for |
|---------|--------|----------|
| `stats` | none (table) | raw numbers |
| `chart` | any field | comparing categories |
| `timechart` | always `_time` | time-based spikes |

---

## 🌍 9. Data Enrichment

| Command | Purpose |
|---------|---------|
| `iplocation` | add City/Region/Country from IP |
| `lookup` | enrich using a CSV/table |
| `eval` | create/modify fields, calculations |

```spl
index=windowslogs
| iplocation SourceIp                     # add geo fields
| stats count by Country                  # logins per country

index=windowslogs
| lookup user_roles Hostname OUTPUT UserRole   # map Hostname -> role from CSV
| stats count by Hostname UserRole

index=windowslogs
| eval LogonTypeDesc = case(LogonType==3,"Network Logon",
                            LogonType==5,"Service")   # code -> readable label
| stats count by LogonType LogonTypeDesc
```

---

## 🚨 10. Anomaly Detection
- **`eventstats`** = like `stats` but **keeps raw events** (adds a stat column to each row).
- **`where`** = like `search` but supports **expressions & field-to-field compares**.

### A. Outliers by Country (rare login location)
```spl
index=vpnlogs
| eventstats count as logins_by_user by user               # total logins per user
| eventstats count as logins_by_user_country by user src_country  # logins per user+country
| eval country_freq = logins_by_user_country / logins_by_user     # how rare is this country?
| where country_freq < 0.1                                 # threshold = keep rare ones
| table _time user src_ip src_country country_freq
```
> Result: kbrown logged in from Austria **once** (freq 0.005) out of 200 → possible VPN/breach.

### B. Outliers by Login Hour (Z-Score)
$$
zscore = \frac{|observed - average|}{standard\ deviation}
$$

> Higher Z-score = more anomalous. `zscore > 3` = way outside that user's normal window.

```spl
index=vpnlogs
| eval hour=tonumber(strftime(_time,"%H")) + tonumber(strftime(_time,"%M"))/60  # decimal hour
| eventstats avg(hour) as typical_hour stdev(hour) as stdev_hour by user        # per-user baseline
| eval zscore = abs(hour - typical_hour) / stdev_hour                           # how weird?
| where zscore > 3                                                              # keep outliers
| eval hour=round(hour,2), typical_hour=round(typical_hour,2)                   # tidy numbers
| eval stdev_hour=round(stdev_hour,2), zscore=round(zscore,2)
| table _time user src_ip src_country hour typical_hour stdev_hour zscore
| sort - zscore
```
> Result: jsmith normally logs in ~13:30 but was seen at 18:30 → investigate.

### C. ML & Impossible Travel
Built on the SAME basics: **SPL + math + threat context** (`iplocation` geo / `lookup` intel). Splunk's `fit`/`apply` add machine learning for future outliers.

---

## 📌 Quick Reference

| Category | Commands |
|----------|----------|
| Filtering | `search` `fields` `dedup` `rename` `regex` |
| Structuring | `table` `head` `tail` `sort` `reverse` |
| Correlation | `join` (subsearch) |
| Transforming | `top` `rare` `highlight` `stats` `chart` `timechart` |
| Enrichment | `iplocation` `lookup` `eval` |
| Anomaly | `eventstats` `where` |

---

## 🧠 Key Takeaways
- ✅ SPL = Splunk's language for searching/analyzing logs.
- 🔗 Pipe `|` chains commands like a data pipeline.
- 📋 `fields` + `table` = trim noise, clean output.
- 📊 `stats` `chart` `timechart` = summarize & visualize.
- 🛠️ `eval` + `case()` = map cryptic codes to readable labels.
- 🌍 `lookup` + `iplocation` = enrich with roles & geo.
- 🚨 `eventstats` + `where` = anomaly-detection backbone.
- 📈 Z-score = quantifies "how weird" an event is (foundation for ML).

---

## 📅 Progress
- Room Completed: ✅
- Difficulty: Intermediate
- Time Taken: 1 hour

