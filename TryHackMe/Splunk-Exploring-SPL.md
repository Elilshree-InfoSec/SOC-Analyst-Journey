# Splunk SPL Notes — TryHackMe: Exploring SPL

Study notes on Splunk's Search Processing Language (SPL) — how to search, filter, transform, and analyze log data for SOC investigations.

---

## Learning Objectives

- Understand how SPL processes and filters log data
- Chain and apply SPL commands effectively
- Visualize log data with charts and statistics
- Apply SPL skills to anomaly detection

**Prerequisites:** Introduction to SIEM, Splunk Basics

---

## 1. Search & Reporting App

The Search & Reporting app is Splunk's default interface for querying and analyzing ingested data.

| Component | Purpose |
|---|---|
| Search Head | Where SPL queries are entered |
| Time Picker | Sets the time range for a search (remember to set to **All time** when needed) |
| Search History | Stores previously run queries |
| Data Summary | Overview of available hosts, sources, and sourcetypes |

### Fields Sidebar

Appears on the left of the search results and shows which fields exist in the current results.

| Element | Meaning |
|---|---|
| Selected Fields | Fields shown by default in results |
| Interesting Fields | Auto-detected fields that may be useful (click → toggle "Selected" to add them) |
| `#` | Numeric field |
| `α` | Text (alpha-numeric) field |
| Count | Number of events containing that field |

> First search to try: `index=windowslogs` with time range set to **All time**. Both `index=windowslogs` and `index = windowslogs` are valid syntax.

---

## 2. Basic Searches

### Search an Index

An index is essentially a Splunk data container — searching it retrieves all events stored inside.

```spl
index=windowslogs
```

### Free Text Search

The simplest search type — Splunk looks for a keyword anywhere in the raw event, case-insensitive. Useful when you don't know field names yet or just want a quick hunt.

```spl
index=windowslogs alice
```

Returns all events containing the word "alice" anywhere in the log.

---

## 3. Search Operators

Operators are the building blocks of every SPL query — they filter, compare, and combine conditions. Operators only work reliably once events are parsed into fields.

### Relational Operators

| Operator | Meaning | Example | What It Does |
|---|---|---|---|
| `=` | Equal to | `UserName=Mark` | Field `UserName` equals `Mark` |
| `!=` | Not equal to | `UserName!=Mark` | Field `UserName` does not equal `Mark` |
| `<` | Less than | `Age<10` | `Age` is less than 10 |
| `<=` | Less than or equal | `Age<=10` | `Age` is 10 or below |
| `>` | Greater than | `Outbound_Traffic>50` | `Outbound_Traffic` exceeds 50 |
| `>=` | Greater than or equal | `Outbound_Traffic>=50` | `Outbound_Traffic` is 50 or more |

```spl
index=windowslogs AccountName!=SYSTEM
```
Filters out all events where `AccountName` is `SYSTEM`.

### Logical Operators

| Operator | Meaning | Example | Notes |
|---|---|---|---|
| `AND` | Both conditions true | `UserName=David AND IPAddress=10.10.10.10` | `AND` is implied between terms even if omitted |
| `OR` | Either condition true | `UserName=David OR UserName=John` | — |
| `NOT` | Field does not exist / exclude | `NOT UserName=*` | Different from `!=` — this checks field existence |
| `IN` | Match against a list | `UserName IN(David, John)` | Cleaner alternative to chained `OR` |

```spl
index=windowslogs AccountName!=SYSTEM AND AccountName=James
index=windowslogs AccountName!=SYSTEM AccountName=James   // same result, AND is implied
```

### Wildcards & CIDR

| Symbol | Example | What It Matches |
|---|---|---|
| `*` | `status=*fail*` | `failed`, `failure`, `appfail`, etc. |
| `*` | `DestinationIp=172.*` | Any IP starting with `172.` |
| CIDR notation | `DestinationIp=172.18.0.0/16` | Any IP inside that subnet |

---

## 4. Quotes, Parentheses & Order of Evaluation

### Quotes — Exact Phrase Matching

Quotes tell Splunk to treat the enclosed text as a single exact phrase (word order matters), and can also be used to escape reserved operator keywords.

| Query | Behavior |
|---|---|
| `index=windowslogs failed login` | Matches events containing both words, in any order |
| `index=windowslogs "failed login"` | Matches only the exact phrase, in that order |
| `index=windowslogs "TO BE OR NOT TO BE"` | Quotes force `OR`/`NOT` to be treated as plain text, not operators |

### Parentheses — Controlling Evaluation Order

`OR` has higher precedence than `AND` in SPL, so ambiguous queries can evaluate differently than expected. Parentheses remove the ambiguity.

| Query | How Splunk Reads It |
|---|---|
| `alice AND bob OR charlie` | `alice AND (bob OR charlie)` — likely **not** what you meant |
| `(alice AND bob) OR charlie` | Explicit grouping — matches (`alice` and `bob` together) **or** `charlie` alone |

---

## 5. The Pipe Operator

Commands are chained with the pipe symbol `|`. Each command's output becomes the input for the next command, letting you refine results step by step.

```spl
index=windowslogs
| fields User SourceIp
```

---

## 6. Filtering Commands

| Command | Purpose | Example |
|---|---|---|
| `fields` | Include/exclude specific fields from output | `\| fields User SourceIp` |
| `dedup` | Remove duplicate events based on a field | `\| dedup SourceIp` |
| `rename` | Rename a field for readability | `\| rename User as Employee` |
| `regex` | Filter using regular expressions (PCRE) | `\| regex Image="\.exe$"` |

### `fields` in Detail

Use `-` before a field name to exclude it; `+` explicitly includes one (default behavior, rarely needed).

```spl
index=windowslogs | fields host User SourceIp
```

### `dedup` in Detail

Keeps only one event per unique value of the chosen field — helpful for cleaning up noisy sources (e.g., some platforms log the same activity many times).

```spl
index=windowslogs
| fields EventID User Image Hostname SourceIp
| dedup SourceIp
```

### `rename` in Detail

Improves readability in reports and dashboards, and can flatten nested JSON/XML fields.

```spl
index=windowslogs
| fields EventID User Image Hostname SourceIp
| rename User as Employee
```

Flattening a nested field, e.g. turning `request.path` into `path`:

```spl
index=jsondata
| rename request.* as *
```

### `regex` in Detail

Matches field values against a pattern rather than an exact keyword — useful for structured formats (file extensions, ID formats, custom log parsing).

```spl
index=windowslogs | regex Image="\.exe$"
```
Returns only events where `Image` ends in `.exe` (`$` anchors the match to the end of the string).

---

## 7. Structuring Commands

| Command | Purpose | Example |
|---|---|---|
| `table` | Display chosen fields in a clean tabular format | `\| table _time EventID Hostname SourceName` |
| `head` | Return the first (newest) N events | `\| head 20` |
| `tail` | Return the last (oldest) N events | `\| tail 20` |
| `sort` | Sort results by a field (ascending) | `\| sort User` |
| `reverse` | Reverse the current event order | `\| reverse` |

`head`/`tail` are also handy for speeding up a search when you don't need the full result set.

### Building a Timeline with `table`

Combining `table` with a host filter and `reverse` reconstructs a chronological sequence of activity on a machine.

```spl
index=windowslogs Hostname=Salena.Adam
| table _time Hostname EventID Category
| reverse
```

---

## 8. Subsearches

A subsearch lets you correlate data across two different sources within a single query, using a shared field as the join key.

**Example scenario:** Sysmon process-creation events (`EventID=1`) don't include the logon type, but Windows Security logon events (`EventID=4624`) do. Both share a `LogonId` field, which can be used to join them.

```spl
index=windowslogs EventID=1
| join LogonId
    [ search index=windowslogs EventID=4624
    | rename TargetLogonId as LogonId
    | fields LogonId LogonType IpAddress ]
| table _time Image User LogonType IpAddress
```

**How it works:**

1. The subsearch (in `[ ]`) runs first — it searches `EventID=4624` events, renames `TargetLogonId` to `LogonId` so field names match, and keeps only `LogonId`, `LogonType`, `IpAddress`.
2. Splunk stores those results as a temporary lookup table.
3. The main search runs over `EventID=1` events, and for each one, checks if its `LogonId` matches an entry from the subsearch.
4. Matching `LogonType` and `IpAddress` values are added to the corresponding event.

> **Performance note:** subsearches don't scale well on large datasets. Many use cases can be replaced with `stats` + `eval` (see below), which is more efficient. Reach for subsearch + `join` mainly when you need to enrich data source A with fields from data source B.

---

## 9. Transforming Commands

Transforming commands turn raw events into aggregated summaries, statistics, and visualizations — instead of reading events one by one, you can spot patterns across thousands at once.

| Command | Purpose | Example |
|---|---|---|
| `top` | Most frequent values of a field | `\| top User limit=5` |
| `rare` | Least frequent values of a field | `\| rare User limit=5` |
| `highlight` | Visually highlight terms in raw event view | `\| highlight User EventID` |
| `stats` | Calculate statistics (count, sum, avg, etc.) | `\| stats count by EventID` |
| `chart` | Return results as a chart-ready table | `\| chart count by User` |
| `timechart` | Visualize values over time | `\| timechart span=30m count by Image` |

`limit=` on `top`/`rare` controls how many results are returned (default is 10).

### `highlight` in Detail

```spl
index=windowslogs | highlight User EventID Image "Process accessed"
```
Note: switch the results view from **List** to **Raw** to see the highlighting.

### `stats` Functions

| Function | Example | Description |
|---|---|---|
| `count` | `stats count by SourceIp` | Counts occurrences per value |
| `avg` | `stats avg(ProcessCount)` | Average of a numeric field |
| `sum` | `stats sum(Cost)` | Total of a numeric field |
| `max` | `stats max(Price)` | Maximum value |
| `min` | `stats min(UserAge)` | Minimum value |

```spl
index=windowslogs | stats count by EventID | sort EventID
```
Counts events per `EventID` and sorts the results ascending.

### `chart`

```spl
index=windowslogs | chart count by User
```
Same underlying logic as `stats`, formatted for direct use in Splunk's charting/visualization panel.

### `timechart`

```spl
index=windowslogs Image!="" | timechart span=30m count by Image limit=5
```
Excludes empty `Image` values, buckets events into 30-minute intervals, and charts the top 5 most common images — useful for spotting trends, spikes, or gaps over time.

---

## 10. Data Enrichment

| Command | Purpose | Example |
|---|---|---|
| `iplocation` | Adds geolocation fields (City, Region, Country) from an IP | `\| iplocation SourceIp` |
| `lookup` | Enriches events using an external CSV/lookup table | `\| lookup user_roles Hostname OUTPUT UserRole` |
| `eval` | Creates or modifies fields, including calculated ones | `\| eval LogonTypeDesc="Network Logon"` |

### `iplocation`

```spl
index=windowslogs | iplocation SourceIp | stats count by Country
```
Uses Splunk's built-in geolocation database — no external file needed.

### `lookup`

```spl
index=windowslogs
| lookup user_roles Hostname OUTPUT UserRole
| stats count by Hostname UserRole
```
Matches `Hostname` against a pre-loaded CSV (`user_roles`) to pull in a `UserRole` field.

### `eval`

One of the most versatile SPL commands — supports conditional logic, string manipulation, math, and more.

```spl
index=windowslogs
| eval LogonTypeDesc=case(LogonType==3, "Network Logon", LogonType==5, "Service")
| stats count by LogonType LogonTypeDesc
```
Assigns a human-readable label based on the numeric `LogonType` value.

---

## 11. Anomaly Detection

Goal: spot the events that look statistically different from the norm, even when nothing obvious stands out at first glance (e.g., in a dataset of thousands of VPN logins).

Key commands:

| Command | Purpose |
|---|---|
| `eventstats` | Like `stats`, but keeps the original individual events instead of collapsing them |
| `where` | Filters using logical/mathematical expressions (more powerful than plain `search`) |
| `eval` | Builds the calculated fields used in the analysis |

### Detecting Outliers by Country

**Scenario:** most users log in from their home country. A login from an unusual country for that specific user is worth investigating, even if it looks unremarkable in aggregate.

```spl
index=vpnlogs
| eventstats count as logins_by_user by user
| eventstats count as logins_by_user_country by user src_country
| eval country_freq=logins_by_user_country/logins_by_user
| where country_freq < 0.1
| table _time user src_ip src_country country_freq
```

| Step | What It Calculates |
|---|---|
| `logins_by_user` | Total logins for that user, across all countries |
| `logins_by_user_country` | Logins for that user from one specific country |
| `country_freq` | Ratio of country-specific logins to total logins |
| `where country_freq < 0.1` | Keeps only rare user–country combinations (threshold = 0.1, i.e. under 10%) |

A low `country_freq` (e.g., `0.005`, meaning that country accounts for only 0.5% of that user's logins) is a strong signal of either travel, VPN usage, or account compromise.

### Detecting Outliers by Hour

**Scenario:** some employees always log in during business hours; others have irregular schedules. A single "typical hour" threshold across all users doesn't work — instead, calculate what's normal *per user*, then measure how far a given login deviates from their own pattern.

```spl
index=vpnlogs
| eval hour=tonumber(strftime(_time, "%H")) + tonumber(strftime(_time, "%M"))/60
| eventstats avg(hour) as typical_hour stdev(hour) as stdev_hour by user
| eval zscore=abs(hour - typical_hour) / stdev_hour
| where zscore > 3
| eval hour=round(hour, 2), typical_hour=round(typical_hour, 2)
| eval stdev_hour=round(stdev_hour, 2), zscore=round(zscore, 2)
| table _time user src_ip src_country hour typical_hour stdev_hour zscore
```

| Field | Meaning |
|---|---|
| `hour` | Login time converted to a decimal hour (e.g., 13:30 → `13.5`) |
| `typical_hour` | That user's average login hour |
| `stdev_hour` | How spread out / predictable that user's login times are (lower = more consistent) |
| `zscore` | How many standard deviations the current login is from that user's typical hour |

#### Z-Score Formula

```
zscore = |observed - average| / standard_deviation
```

A higher z-score means a more anomalous event. `zscore > 3` is a common threshold for flagging outliers.

### Beyond the Basics: ML & Impossible Travel

More advanced detections — such as **Impossible Travel** (a user logging in from two geographically distant locations in a timeframe that would require faster-than-possible travel) — build on the same core commands (`eventstats`, `eval`, `where`), combined with:

- IP geolocation (`iplocation`)
- Threat intelligence lookups (`lookup`)
- Splunk's `fit` and `apply` commands, which apply machine learning models to detect outliers automatically

---

## Full Command Reference

| Command | Purpose |
|---|---|
| `search` | Base search / filter events |
| `fields` | Include or exclude fields |
| `table` | Display selected fields as a table |
| `dedup` | Remove duplicate events |
| `rename` | Rename a field |
| `regex` | Filter using regular expressions |
| `sort` | Sort results |
| `reverse` | Reverse event order |
| `head` | Return first N (newest) events |
| `tail` | Return last N (oldest) events |
| `join` | Correlate results from a subsearch |
| `top` | Most common field values |
| `rare` | Least common field values |
| `highlight` | Highlight terms in raw view |
| `stats` | Summarize/aggregate data |
| `chart` | Build a chart-ready table |
| `timechart` | Visualize data over time |
| `iplocation` | Add IP geolocation fields |
| `lookup` | Enrich data from an external table |
| `eval` | Create or modify fields |
| `eventstats` | Aggregate stats while keeping raw events |
| `where` | Advanced conditional filtering |

---

## Personal Reflection

This room gave me a much better understanding of how analysts use Splunk SPL to investigate logs efficiently. Instead of scrolling through thousands of raw events, SPL allows me to filter, organize, summarize, and visualize data to quickly identify suspicious activity. Learning commands such as `stats`, `eval`, `timechart`, and `eventstats` showed how powerful SPL is for real-world SOC investigations. As someone preparing for a SOC Analyst internship, this room strengthened my confidence in writing basic SPL queries and understanding how security analysts investigate incidents.

---

## 📅 Progress
- Room Completed: ✅
- Difficulty: Easy
- Time Taken: ~30 minutes
