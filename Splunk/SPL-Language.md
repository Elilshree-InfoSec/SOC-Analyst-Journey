# SPL (Search Processing Language) Cheat Sheet for SOC Analysts

A quick-reference guide to Splunk's Search Processing Language, organized for day-to-day SOC investigation work.

---

## Table of Contents

1. [Core Concepts](#core-concepts)
2. [Search Operators](#search-operators)
3. [Quotes & Parentheses](#quotes--parentheses)
4. [Filtering & Structuring Commands](#filtering--structuring-commands)
5. [Transforming Commands](#transforming-commands)
6. [Field Extraction & Enrichment](#field-extraction--enrichment)
7. [Subsearches & Joins](#subsearches--joins)
8. [Statistical Commands: stats vs eventstats vs streamstats](#statistical-commands-stats-vs-eventstats-vs-streamstats)
9. [tstats & Data Model Acceleration](#tstats--data-model-acceleration)
10. [Splunk Enterprise Security: Correlation Searches & RBA](#splunk-enterprise-security-correlation-searches--rba)
11. [Common Investigation Queries](#common-investigation-queries)
12. [Anomaly Detection Patterns](#anomaly-detection-patterns)
13. [Performance Optimization](#performance-optimization)
14. [Search Modes & Time Ranges](#search-modes--time-ranges)

---

## Core Concepts

Every SPL search is a **pipeline**. Events flow left to right through commands separated by the pipe character `|`. Each command transforms the result set handed to it by the previous command.

```spl
index=windows sourcetype=WinEventLog EventCode=4625
| stats count by Account, src_ip
| sort -count
| head 20
```

- `index=windows sourcetype=WinEventLog EventCode=4625` — the **base search** (runs against the indexer).
- `| stats ...` — aggregates events.
- `| sort -count` — orders results descending by count.
- `| head 20` — returns only the first 20 rows.

**Rule of thumb:** put `index`, `sourcetype`, and exact field=value filters as early as possible — this is what makes a search fast.

---

## Search Operators

### Relational Operators

| Operator | Example | Explanation |
|---|---|---|
| `=` | `UserName=Mark` | Field equals value |
| `!=` | `UserName!=Mark` | Field does not equal value |
| `<` | `Age<10` | Less than |
| `<=` | `Age<=10` | Less than or equal to |
| `>` | `Outbound_Traffic>50` | Greater than |
| `>=` | `Outbound_Traffic>=50` | Greater than or equal to |

### Logical Operators

| Operator | Example | Explanation |
|---|---|---|
| `NOT` | `NOT UserName=*` | Returns events where the field does **not exist** (different from `!=`) |
| `AND` | `UserName=David AND IPAddress=10.10.10.10` | Both conditions must be true (AND is implicit between terms) |
| `OR` | `UserName=David OR UserName=John` | Either condition is true |
| `IN` | `UserName IN(David, John)` | Cleaner alternative to chained `OR` for long lists |

> `AND` is implied between adjacent terms, so `AccountName!=SYSTEM AccountName=James` behaves the same as `AccountName!=SYSTEM AND AccountName=James`.

### Wildcards & CIDR

| Symbol | Example | Explanation |
|---|---|---|
| `*` | `status=*fail*` | Matches `failed`, `failure`, `appfail`, etc. |
| `*` | `DestinationIp=172.*` | Matches any IP starting with `172.` |
| CIDR | `DestinationIp=172.18.0.0/16` | Matches any IP within the subnet |

> **Performance warning:** leading wildcards (`*value`) or `NOT` combined with wildcards force a full scan and bypass the index — avoid in scheduled/production searches.

---

## Quotes & Parentheses

**Quotes** define an exact phrase (word order matters) and can be used to escape search operators:

```spl
index=windowslogs failed login          " both keywords, any order
index=windowslogs "failed login"        " exact phrase, in that order
index=windowslogs "TO BE OR NOT TO BE"  " treated as a literal phrase, not operators
```

**Parentheses** control operator precedence. `OR` binds tighter than you might expect, so group explicitly:

```spl
" Implicit / ambiguous — Splunk evaluates AND before checking your intent
index=windowslogs alice AND bob OR charlie
" -> evaluated as: alice AND (bob OR charlie)

" Explicit and correct if you wanted (alice AND bob) OR charlie
index=windowslogs (alice AND bob) OR charlie
```

---

## Filtering & Structuring Commands

| Command | Example | Explanation |
|---|---|---|
| `table` | `\| table _time EventID Hostname SourceName` | Displays only the specified fields, in order, as a clean table |
| `fields` | `\| fields host User SourceIp` | Includes fields; use `-` to exclude (`\| fields - SourceIp`) |
| `dedup` | `\| dedup SourceIp` | Removes duplicate events based on the specified field(s) |
| `rename` | `\| rename User as Employee` | Renames a field for readability; also useful to flatten nested JSON/XML fields (`\| rename request.* as *`) |
| `head` | `\| head 20` | Returns the first 20 events (fast — stops scanning once satisfied) |
| `tail` | `\| tail 20` | Returns the last (oldest) 20 events |
| `sort` | `\| sort User` / `\| sort -count` | Sorts ascending by default; prefix field with `-` for descending |
| `reverse` | `\| reverse` | Reverses the current result order |
| `where` | `\| where FailedCount > 20` | Filters using `eval`-style boolean expressions (works on calculated/aggregated fields, unlike `search`) |
| `regex` | `\| regex Image = "\.exe$"` | Filters using PCRE regular expressions against a field's raw value |

---

## Transforming Commands

Commands that turn raw events into summaries, statistics, or visualizations. Searches using them are called **transforming searches**.

| Command | Example | Explanation |
|---|---|---|
| `top` | `\| top User limit=5` | Most frequent values of a field |
| `rare` | `\| rare User limit=5` | Least frequent values of a field |
| `chart` | `\| chart count by User` | Tabular output shaped for visualization |
| `timechart` | `\| timechart span=30m count by Image limit=5` | Time-bucketed statistics — ideal for trend/spike detection |
| `highlight` | `\| highlight User EventID "Process accessed"` | Visually highlights terms/fields in raw event view (List/Raw view only) |

### `stats` Functions

| Function | Example | Explanation |
|---|---|---|
| `count` | `stats count by SourceIp` | Number of occurrences |
| `dc()` | `dc(src_ip)` | Distinct (unique) count |
| `values()` | `values(EventCode)` | Collects all unique values into a multivalue field |
| `avg()` | `avg(ProcessCount)` | Average |
| `sum()` | `sum(Cost)` | Sum |
| `min()` / `max()` | `min(UserAge)` / `max(Price)` | Minimum / maximum |
| `earliest()` / `latest()` | `earliest(_time)` / `latest(_time)` | Min/max time value |
| `stdev()` | `stdev(hour)` | Standard deviation — useful for beaconing/regularity detection |

```spl
| stats count as EventCount,
        dc(src_ip) as UniqueSourceIPs,
        values(EventCode) as EventCodes,
        earliest(_time) as FirstSeen,
        latest(_time) as LastSeen
  by Account
```

---

## Field Extraction & Enrichment

### `eval` — compute new fields

```spl
| eval
    IsAdmin = if(match(Account, "admin|svc_|SYSTEM"), "true", "false"),
    Duration = strptime(EndTime, "%Y-%m-%d %H:%M:%S") - strptime(StartTime, "%Y-%m-%d %H:%M:%S"),
    Category = case(
        count > 100, "High Volume",
        count > 20, "Medium Volume",
        true(), "Low Volume"
    )
```

### `rex` — extract fields with regex (named capture groups)

```spl
| rex field=_raw "src=(?<SrcIP>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})\s+dst=(?<DstIP>\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3})"

" sed-style substitution mode
| rex field=CommandLine mode=sed "s/temp/REDACTED/g"
```

### `lookup` — enrich events from a CSV/KV Store table

```spl
| lookup dnslookup clientip as src_ip OUTPUT clienthost
| lookup threat_intel_ips ip as src_ip OUTPUT threat_category, threat_confidence
| where isnotnull(threat_category)
```

### `inputlookup` — read a lookup table as a data source

```spl
| inputlookup threat_intel_ips.csv
| where threat_confidence > 80
```

### `iplocation` — enrich with geo data

```spl
| iplocation SourceIp
| stats count by Country
```

### `transaction` — group related events into sessions

```spl
index=firewall action=* src_ip=192.168.1.100
| transaction src_ip maxspan=1h maxpause=5m
| eval Duration = duration
| where eventcount > 10
```

---

## Subsearches & Joins

**Subsearch** — results of one search feed into another as filter values, wrapped in `[ ... ]`:

```spl
index=proxy dest_category=malware
| return 20 dest
| format
```

**join** — correlate two data sources on a shared field (e.g., linking a Sysmon process-creation event to its Windows logon context via `LogonId`):

```spl
index=windowslogs EventID=1
| join LogonId
    [ search index=windowslogs EventID=4624
    | rename TargetLogonId as LogonId
    | fields LogonId LogonType IpAddress]
| table _time Image User LogonType IpAddress
```

> ⚠️ Subsearches are capped at 10,000 results by default and **do not scale well** on large datasets. Prefer `stats` + `eval`, or `lookup`, wherever possible. Reserve `join`/subsearch for cases where you genuinely need to enrich data source A with fields from data source B.

---

## Statistical Commands: stats vs eventstats vs streamstats

| Command | Behavior |
|---|---|
| `stats` | Aggregates events into a **summary table** — one row per unique `by` value; original events are discarded. |
| `eventstats` | Computes the same aggregations but **adds them as new fields on every original event**, preserving the raw events. Useful for enriching events with a group-level metric before filtering (e.g., flag events belonging to a user whose total login count exceeds a threshold). |
| `streamstats` | Computes a **running aggregation in time order** as events flow through the pipeline (e.g., `streamstats count by src_ip` adds a running per-source event count). |

---

## tstats & Data Model Acceleration

`tstats` queries **pre-computed, accelerated data model summaries** instead of raw events — dramatically faster for large time windows (commonly cited as 10–100x faster than `stats` over raw events, when the relevant fields are in the CIM data model).

```spl
| tstats summariesonly=true
    count as EventCount,
    dc(Authentication.src) as UniqueSources
  from datamodel=Authentication.Authentication
  where Authentication.action=failure
  by Authentication.user, _time span=1h
| rename Authentication.user as user
| where EventCount > 30 AND UniqueSources > 1
```

- `summariesonly=true` restricts results to the accelerated summary data only (fast, but only as current as the last acceleration run).
- Field names inherit the data model prefix (e.g., `Authentication.user`) — `rename Authentication.* as *` strips it for readability.
- The **CIM (Common Information Model)** normalizes fields across different sourcetypes/vendors, so a data model search (e.g., `datamodel=Authentication`) queries all mapped authentication sources regardless of underlying index, as long as the sourcetypes are CIM-mapped via their Technology Add-ons.

---

## Splunk Enterprise Security: Correlation Searches & RBA

**Correlation search** — a saved search that runs on a schedule and (traditionally) generates a **notable event** when conditions match.

**Risk-Based Alerting (RBA)** — instead of alerting on every individual low-confidence signal, each correlation search writes a **risk event** (with a risk score) to the Risk Index. A separate "Risk Notables" rule watches the Risk Index and creates a notable only when an entity's **accumulated** risk score crosses a threshold (commonly 100) within a rolling window. This is intended to reduce alert volume, since isolated low-confidence signals stop generating individual incidents on their own.

```spl
| tstats summariesonly=true count from datamodel=Authentication.Authentication
  where Authentication.action=failure by Authentication.user, Authentication.src
| rename Authentication.* as *
| where count > 20
| eval risk_score = case(
    count > 100, 75,
    count > 50, 50,
    count > 20, 25
  ),
  risk_object = user,
  risk_object_type = "user",
  risk_message = "Brute force detected: " . count . " failures from " . src
| table risk_object, risk_object_type, risk_score, risk_message, src, count
```

**Risk object types:** `"user"` (account-based), `"system"` (computer-based), `"other"` (IPs/domains that don't map cleanly to a user or system). Correct typing ensures risk events for the same entity aggregate together on the Risk Analysis dashboard.

---

## Common Investigation Queries

> ⚠️ Field names below (`Account`, `Source_Network_Address`, `TargetUserName`, etc.) assume Windows Security Event Log field extractions. Adjust to match your environment's actual field mappings/CIM normalization before use.

### Authentication & Identity

**Brute force — failed logins per account per source IP:**
```spl
index=windows sourcetype=WinEventLog EventCode=4625
earliest=-1h
| stats count as FailedCount,
        values(Source_Network_Address) as SourceIPs,
        dc(Source_Network_Address) as UniqueSourceIPs
  by Account
| where FailedCount > 20
| sort -FailedCount
```

**Credential stuffing — failures followed by a success from the same source:**
```spl
index=windows sourcetype=WinEventLog (EventCode=4624 OR EventCode=4625)
earliest=-2h
| eval Action = if(EventCode==4624, "Success", "Failure")
| stats values(Action) as Actions,
        count(eval(Action="Failure")) as Failures,
        count(eval(Action="Success")) as Successes
  by Account, Source_Network_Address
| where Failures > 5 AND Successes > 0
```

**Privileged group membership changes** (EventCodes 4728, 4732, 4756 — additions to security-enabled global/local/universal groups):
```spl
index=windows sourcetype=WinEventLog EventCode IN (4728, 4732, 4756)
| rex field=_raw "Member:\s+Security ID:\s+\S+\s+Account Name:\s+(?<MemberName>[^\n]+)"
| rex field=_raw "Group:\s+Security ID:\s+\S+\s+Group Name:\s+(?<GroupName>[^\n]+)"
| eval PrivilegedGroup = if(match(GroupName, "Domain Admins|Enterprise Admins|Schema Admins|Administrators|Backup Operators"), "true", "false")
| where PrivilegedGroup="true"
| table _time, SubjectUserName, MemberName, GroupName, ComputerName
```

**Account lockouts (EventCode 4740):**
```spl
index=windows sourcetype=WinEventLog EventCode=4740
earliest=-24h
| stats count as LockoutCount,
        values(Source_Workstation) as SourceWorkstations,
        values(Caller_Computer_Name) as CallerComputers
  by TargetUserName
| where LockoutCount > 2
| sort -LockoutCount
```

### Network

**DNS exfiltration — high unique-subdomain volume per domain:**
```spl
index=dns sourcetype=dns_logs record_type=A OR record_type=TXT
earliest=-1h
| rex field=query "(?<subdomain>[^\.]+)\.(?<domain>[^\.]+\.[^\.]+)$"
| stats dc(subdomain) as UniqueSubdomains, count as QueryCount
  by src_ip, domain
| where UniqueSubdomains > 50
| lookup alexa_top1m domain OUTPUT rank
| where isnull(rank)
| sort -UniqueSubdomains
```

**Beaconing — regular-interval, small-payload outbound connections:**
```spl
index=proxy OR index=netflow sourcetype=bluecoat OR sourcetype=palo_alto_traffic
earliest=-24h
| stats count as ConnectionCount,
        avg(eval(tonumber(bytes_out))) as AvgBytesOut,
        stdev(eval(tonumber(_time))) as TimeStdev
  by src_ip, dest_ip, dest_port
| where ConnectionCount > 10
| where TimeStdev < 300
| where AvgBytesOut < 5000
| sort TimeStdev
```

**Large outbound transfers (possible exfiltration):**
```spl
index=proxy sourcetype=bluecoat action=TCP_TUNNEL OR action=TCP_MISS
earliest=-24h
| eval BytesMB = bytes_out / 1048576
| stats sum(BytesMB) as TotalMBOut, count as RequestCount
  by src_ip, dest_host
| where TotalMBOut > 500
| lookup alexa_top1m domain as dest_host OUTPUT rank
| where isnull(rank)
| sort -TotalMBOut
```

**Port scanning (firewall deny logs):**
```spl
index=firewall sourcetype=palo_alto_traffic action=deny
earliest=-15m
| stats dc(dest_port) as UniquePortsProbed, count as DenyCount
  by src_ip, dest_ip
| where UniquePortsProbed > 20
| sort -UniquePortsProbed
```

### Endpoint

**Suspicious/encoded PowerShell execution:**
```spl
index=windows sourcetype=WinEventLog EventCode=4688 OR (sourcetype=xmlwineventlog EventCode=4688)
| where match(Process_Command_Line, "-enc\\s+|EncodedCommand|FromBase64String|Invoke-Expression|IEX")
| rex field=Process_Command_Line "-enc\\s+(?<B64Payload>[A-Za-z0-9\+/=]+)"
| eval DecodedPayload = if(isnotnull(B64Payload), replace(urldecode(B64Payload),"%2B","+"), null)
| table _time, ComputerName, Creator_Process_Name, Process_Command_Line, B64Payload
| sort -_time
```

**New scheduled task creation (EventCode 4698), excluding built-in MS/Windows tasks:**
```spl
index=windows sourcetype=WinEventLog EventCode=4698
| rex field=_raw "Task Name:\s+(?<TaskName>[^\r\n]+)"
| rex field=_raw "Task Content:\s+(?<TaskContent>[\s\S]+?)</Task>"
| where NOT match(TaskName, "^\\\\Microsoft\\\\|^\\\\Windows\\\\")
| stats count by _time, SubjectUserName, ComputerName, TaskName
| sort -_time
```

**Lateral movement via PsExec (EventCode 7045 — new service installation):**
```spl
index=windows sourcetype=WinEventLog EventCode=7045
| where match(ServiceFileName, "\\\\ADMIN\$\\\\|PSEXESVC|paexec")
| stats count by _time, ComputerName, ServiceName, ServiceFileName, AccountName
| sort -_time
```

**Living-off-the-land binary (LOLBin) abuse — MITRE ATT&CK T1218:**
```spl
index=windows sourcetype=WinEventLog EventCode=4688
| where match(New_Process_Name, "(?i)certutil\.exe|bitsadmin\.exe|mshta\.exe|regsvr32\.exe|rundll32\.exe|wscript\.exe|cscript\.exe")
| where match(Process_Command_Line, "(?i)http|ftp|download|decode|urlcache")
| table _time, ComputerName, Creator_Process_Name, New_Process_Name, Process_Command_Line
| sort -_time
```

---

## Anomaly Detection Patterns

**Outlier detection by country (frequency-based, per user):**
```spl
index=vpnlogs
| eventstats count as logins_by_user by user
| eventstats count as logins_by_user_country by user src_country
| eval country_freq=logins_by_user_country/logins_by_user
| where country_freq < 0.1
| table _time user src_ip src_country country_freq
```
`eventstats` preserves the raw events while attaching group-level counts, letting you compute `country_freq` (how rare this user+country pairing is) per event, then filter to rare pairings.

**Outlier detection by login hour (z-score against each user's own baseline):**
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
A `zscore` of 3 means the observed login hour is 3 standard deviations from that user's typical login time — a strong anomaly signal even without a fixed "business hours" rule.

> More advanced ML-based detections (e.g., **Impossible Travel**) build on these same primitives — `iplocation` for geo context, `lookup` for threat intel, and Splunk's `fit`/`apply` commands for applying trained ML models.

---

## Performance Optimization

1. **Maximize base-search efficiency.** Everything before the first `|` runs against the indexer's inverted index. Exact `field=value` matches (`EventCode=4625`) are fast. Mid-string or trailing wildcards (`EventCode=46*`) are slower. `NOT` combined with wildcards (`NOT sourcetype=*windows*`) forces a full scan.
2. **Use `tstats` for large time-window aggregations** (typically >4 hours across high-volume sourcetypes) when the fields are in a CIM data model — 10–100x faster than `stats` over raw events.
3. **Reduce event count before expensive operations.** Apply `stats`/`where` filters *before* `join`, `transaction`, or large lookups.
4. **Prefer indexed fields in base searches** (`source`, `sourcetype`, `host`, `index`, and any field explicitly marked `indexed=true` in `transforms.conf`) over fields that require full raw-event extraction.
5. **Match scheduled search frequency to actual detection latency needs** — don't run a monthly-trend search every 5 minutes; it wastes scheduler resources and can delay other jobs.
6. **Use the Job Inspector** (magnifying glass icon on the search results page) to see execution time per command, events scanned vs. returned at each stage, and cache hit rate — this tells you exactly where the bottleneck is.

---

## Search Modes & Time Ranges

### Search Modes

| Mode | Behavior |
|---|---|
| **Fast** | Extracts only the fields required by the search commands; no field discovery or event decoration. Best for high-performance/known-field searches. Auto-applied when all referenced fields are indexed fields. |
| **Smart** (default) | Uses fast mode for transforming searches (those producing statistical tables) and verbose mode for non-transforming searches. |
| **Verbose** | Full field extraction, event decoration, and timeline generation for every event. Most expensive — best for open-ended exploration. |

> For production scheduled searches / correlation rules, explicitly set `dispatch.search_mode = fast` in `savedsearches.conf`.

### Time Range Best Practice

Always set explicit `earliest`/`latest` rather than relying on the UI time picker for scheduled content:

```spl
earliest=-60m@m latest=now
```

For a correlation search running every 5 minutes with a 1-hour lookback, use a small overlap buffer to avoid missing events due to indexing latency:

```spl
earliest=-65m@m latest=-5m@m
```

> Indexing latency is typically under 30 seconds for Universal Forwarder-delivered events, but can stretch to several minutes during high-volume ingestion spikes — the overlap buffer protects against gaps.

---

## Quick Reference: Essential Six Commands

If you only learn six commands first, learn these — they cover the large majority of real SOC investigation/detection work:

| Command | Purpose |
|---|---|
| `stats` | Aggregate/count events by field values |
| `eval` | Compute new fields, booleans, conditionals |
| `rex` | Extract fields from unstructured text via regex |
| `transaction` | Group related events into sessions |
| `lookup` | Enrich events with external CSV/KV Store data |
| `where` | Filter using eval-style boolean expressions (vs. `search`'s keyword syntax) |

`table` (select output columns) and `sort` (order results) round out the essential toolkit.
