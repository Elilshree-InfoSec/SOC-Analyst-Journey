# 🔍 TryHackMe - Splunk: Exploring SPL

## 🎯 Learning Objectives
- Understand **Search Processing Language (SPL)**.
- Search, filter, and transform log data.
- Visualize data using charts and statistics.
- Detect anomalies using SPL.

---

# 🖥️ 1. Splunk Search & Reporting

## Main Components

| Component | Purpose |
|------------|----------|
| Search Head | Enter SPL queries |
| Time Picker | Select search timeframe |
| Search History | View previous searches |
| Data Summary | Shows hosts, sources & sourcetypes |
| Fields Sidebar | View extracted fields |

## Field Types

| Type | Meaning |
|------|---------|
| Selected Fields | Default extracted fields |
| Interesting Fields | Automatically detected useful fields |
| # | Numeric field |
| α | Text field |

---

# 🔎 2. Basic Searches

## Search an Index

```spl
index=windowslogs
```

## Free Text Search

```spl
index=windowslogs alice
```

Searches for the keyword **alice** anywhere in the logs.

---

# ⚔️ 3. Search Operators

## Relational Operators

| Operator | Description | Example |
|----------|-------------|---------|
| = | Equal to | `User=Mark` |
| != | Not equal to | `User!=SYSTEM` |
| > | Greater than | `Count>10` |
| >= | Greater than or equal | `Count>=10` |
| < | Less than | `Age<20` |
| <= | Less than or equal | `Age<=20` |

Example:

```spl
index=windowslogs AccountName!=SYSTEM
```

---

## Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| AND | Both conditions must match | `User=James AND EventID=4624` |
| OR | Either condition matches | `User=James OR User=John` |
| NOT | Field does not exist | `NOT User=*` |
| IN | Multiple values | `User IN (James, John)` |

---

## 🌐 Wildcards & CIDR

| Syntax | Purpose | Example |
|--------|---------|---------|
| `*` | Partial text match | `status=*fail*` |
| `*` | Partial IP match | `DestinationIp=172.*` |
| CIDR | Search subnet | `DestinationIp=172.18.0.0/16` |

---

# 📝 4. Quotes & Parentheses

## Exact Phrase

```spl
"failed login"
```

## Group Conditions

```spl
(alice AND bob) OR charlie
```

💡 Use parentheses to control the order of evaluation.

---

# 🔗 5. Pipe Operator (`|`)

Commands are chained using:

```spl
|
```

Example:

```spl
index=windowslogs
| fields User SourceIp
```

➡️ Each command passes its output to the next command.

---

# 🎯 6. Filtering Commands

| Command | Purpose | Example |
|---------|----------|---------|
| `fields` | Show selected fields | `| fields User SourceIp` |
| `dedup` | Remove duplicate values | `| dedup SourceIp` |
| `rename` | Rename fields | `| rename User as Employee` |
| `regex` | Filter using Regular Expressions | `| regex Image="\.exe$"` |

---

# 📑 7. Structuring Commands

| Command | Purpose | Example |
|---------|----------|---------|
| `table` | Display selected fields | `| table _time EventID Hostname` |
| `head` | Show first/newest events | `| head 20` |
| `tail` | Show last/oldest events | `| tail 20` |
| `sort` | Sort results | `| sort User` |
| `reverse` | Reverse event order | `| reverse` |

---

# 🔄 8. Subsearch

Used to correlate data from multiple searches.

```spl
| join LogonId
    [ search ... ]
```

💡 Useful for combining multiple log sources into one investigation.

---

# 📊 9. Transforming Commands

| Command | Purpose | Example |
|---------|----------|---------|
| `top` | Most common values | `| top User` |
| `rare` | Least common values | `| rare User` |
| `highlight` | Highlight keywords | `| highlight User EventID` |
| `stats` | Generate statistics | `| stats count by EventID` |
| `chart` | Create charts | `| chart count by User` |
| `timechart` | Time-based visualization | `| timechart span=30m count by Image` |

## Common `stats` Functions

| Function | Purpose |
|----------|---------|
| count | Count events |
| avg | Average value |
| sum | Total values |
| max | Maximum value |
| min | Minimum value |

---

# 🌍 10. Data Enrichment

| Command | Purpose | Example |
|---------|----------|---------|
| `iplocation` | Add IP geolocation | `| iplocation SourceIp` |
| `lookup` | Enrich data using CSV lookup | `| lookup user_roles Hostname OUTPUT UserRole` |
| `eval` | Create or modify fields | `| eval LogonTypeDesc="Network Logon"` |

---

# 🚨 11. Anomaly Detection

## Useful Commands

| Command | Purpose |
|---------|----------|
| `eventstats` | Calculate statistics while preserving events |
| `where` | Filter using expressions |
| `eval` | Create calculated fields |

Example:

```spl
| where country_freq < 0.1
```

Detects users logging in from unusual countries.

### 📈 Z-Score Formula

```text
zscore = |observed - average| / standard deviation
```

➡️ Higher Z-score = More anomalous.

---

# 📌 Common SPL Commands

| Command | Purpose |
|---------|----------|
| search | Search events |
| fields | Select fields |
| table | Display fields |
| stats | Summarize data |
| chart | Create chart |
| timechart | Time visualization |
| dedup | Remove duplicates |
| rename | Rename fields |
| regex | Regex filtering |
| sort | Sort results |
| reverse | Reverse order |
| head | First events |
| tail | Last events |
| top | Most common values |
| rare | Least common values |
| highlight | Highlight keywords |
| lookup | Data enrichment |
| iplocation | IP geolocation |
| eval | Create/modify fields |
| eventstats | Statistics while keeping events |
| where | Advanced filtering |
| join | Combine searches |

---

# 🧠 Key Takeaways

- ✅ SPL is Splunk's query language for searching and analyzing logs.
- 🔗 Commands are chained using the **pipe (`|`)** operator.
- 📋 `fields` and `table` help simplify investigation results.
- 📊 `stats`, `chart`, and `timechart` summarize and visualize data.
- 🛠️ `eval` creates calculated fields for better analysis.
- 🌍 `lookup` and `iplocation` enrich log data with additional context.
- 🚨 `eventstats` + `where` are useful for anomaly detection.
- 🔄 `join` allows correlation between multiple log sources.

---

# 💭 Personal Reflection

This room gave me a much better understanding of how analysts use **Splunk SPL** to investigate logs efficiently. Instead of scrolling through thousands of raw events, SPL allows me to filter, organize, summarize, and visualize data to quickly identify suspicious activity. Learning commands such as **stats**, **eval**, **timechart**, and **eventstats** showed me how powerful SPL is for real-world SOC investigations. As someone preparing for a **SOC Analyst internship**, this room strengthened my confidence in writing basic SPL queries and understanding how security analysts investigate incidents.

---

# 📅 Progress

- Room Completed: ✅ 
- Difficulty: Intermediate 
- Time Taken: 1 hour
