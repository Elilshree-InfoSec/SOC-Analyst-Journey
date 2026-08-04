# 🔎 Introduction to Splunk — Revision Notes
*Based on: Cisco – Introduction to Splunk course*

---

## 1. 🧠 What is Splunk?

Splunk is a **unified platform** that helps teams keep mission-critical digital systems **secure and reliable** — by searching, analyzing, and visualizing machine data.

| Capability | What it does |
|---|---|
| 🔍 **Search (SPL)** | Query events across multiple data sources using Search Processing Language |
| 📊 **Reports & Dashboards** | Save searches as reports → power real-time dashboard panels |
| 📐 **Data Models & Pivot** | Organize data into structured models → visualize without writing SPL |
| 🚨 **Alerts** | Set custom triggers to get notified of anomalies automatically |

---

## 2. 🖥️ Splunk Web & Interface

### Apps
> Pre-configured environments on top of Splunk, built to solve a **specific use case** or manage specific data sources.

### 👥 User Roles

| Role | Can Do |
|---|---|
| **Admin** | Install apps/configs • Onboard new data • Create **global** knowledge objects |
| **Power** | Create & share knowledge objects (within an app) • Run real-time searches |
| **User** | Most restrictive • Sees only their own objects + objects explicitly shared with them |

> ☁️ Splunk also has **cloud-specific roles** for hosted environments.

### 🏠 Splunk Web Home
- Launch, configure & manage installed apps
- Built-in documentation
- Set a custom default landing dashboard
- Starting point to add apps / onboard data

### 🔍 Search & Reporting App — 8 Key Components

| # | Component | Purpose |
|---|---|---|
| 1 | **Splunk Bar** | Switch apps, account, settings, jobs, help |
| 2 | **App Bar** | Navigate within the current app |
| 3 | **Search Bar** | Type & run SPL queries |
| 4 | **Time Range Picker** | Set the time window for search |
| 5 | **Data Summary Button** | View data by Hosts / Sources / Source Types |
| 6 | **Table Views** | Explore/clean data with no SPL needed |
| 7 | **Search History** | View & rerun past searches |
| 8 | **Search Terms/Elements** | Click results to instantly refine search |

---

## 3. 🛡️ Searching & Investigating

**Example use case:** Searching `failed` + a bounded time range → spot failed login attempts / brute-force attacks.

### Search Interface Elements

| Element | Function |
|---|---|
| **Save As Menu** | Save as Report, Alert, Dashboard, or Event Type |
| **Result Tabs** | Events (raw) • Patterns (clusters) • Statistics/Visualizations |
| **Action Buttons** | Pause, Stop, Share, Print, Export |
| **Fields Sidebar** | Auto-discovered event fields for quick filtering |

### ⏱️ Job Lifecycle

| Type | Lifespan |
|---|---|
| Ad-hoc search | ⏳ 10 minutes (then must rerun) |
| Saved/shared search | 📅 7 days (fixed snapshot) |

📤 **Export formats:** CSV, JSON

### 🚥 Search Modes

| Mode | Behavior |
|---|---|
| **Smart** (default) | Auto-toggles based on search type |
| **Fast** | Max speed, no field discovery |
| **Verbose** | Max detail, all fields — slower |

> ℹ️ **No Event Sampling** is OFF by default — searches scan 100% of raw data.

---

## 4. 📋 Exploring Events

- Matched terms are **highlighted in yellow**
- Events shown in **reverse chronological order** (newest first)
- Timestamps follow your account's **timezone setting**

### Default Fields (always present)
- `Host` → machine that generated the log
- `Source` → file/port the data came from
- `Sourcetype` → data format (e.g. `syslog`, `access_combined`)

### 🖱️ Click-to-Refine Actions

| Action | Effect |
|---|---|
| **Add** | Adds term to current search |
| **New** | Clears search, starts fresh with that term |
| **Exclude** | Adds a `NOT` condition |

---

## 5. ⚔️ Search Syntax & Boolean Logic

| Rule | Example |
|---|---|
| Wildcard `*` | `fail*` → fail, failed, failure |
| Not case-sensitive (search terms) | `failed` = `FAILED` |
| Exact phrase → quotes | `"failed password"` |
| Escape quotes → `\` | `"user \"admin\" logged in"` |

### Boolean Operators (⚠️ must be UPPERCASE)

| Operator | Meaning | Example |
|---|---|---|
| `AND` | Both must match | `failed AND login` |
| `OR` | Either matches | `failed OR blocked` |
| `NOT` | Excludes | `failed AND NOT user=admin` |

**Evaluation order:** `NOT` → `OR` → `AND` (use `()` to override)
Example:

```spl
(failed OR blocked) AND password
```
---

## 6. 🧩 SPL Structure

SPL commands are chained with a **pipe `|`**, passing results from one stage to the next.

| Component | Purpose | Example |
|---|---|---|
| **Search Terms** | Filter raw events | `index=network sourcetype=cisco_wsa_squid usage=Violation` |
| **Command** | Action to perform | `stats` |
| **Function** | Calculation | `count()` |
| **Argument** | Field it acts on | `usage` |
| **Clause** | Grouping/renaming | `as Visits` |

**Case sensitivity:**
- SPL framework (commands/functions/clauses) → ❌ not case-sensitive
- Field **values** → ✅ case-sensitive (`Violation` ≠ `violation`)

### 🏆 Performance Best Practices
1. ⏳ Limit by **time** first
2. 🏷️ Filter by **index-time fields** (`index`, `host`, `source`, `sourcetype`)
3. 🎯 Be specific
4. ➕ Prefer inclusion over exclusion
5. 🔀 Avoid heavy wildcards — use `OR`/`IN`
6. 📉 Filter early in the pipeline

---

## 7. 🧱 Knowledge Objects

Reusable objects that can be **created once, shared, and reused** across searches, users, and apps.

| Category | Objects | Purpose |
|---|---|---|
| **Data Interpretation** | Fields, Field Extractions, Calculated Fields | Compute values from existing fields |
| **Data Classification** | Event Types, Transactions | Group related events over time |
| **Data Enrichment** | Lookups, Workflow Actions | Connect to external data/resources |
| **Data Normalization** | Tags, Field Aliases | Standardize fields across sources |
| **Data Models** | Hierarchical Datasets | Power Pivot (no-code) visualizations |

### 🧑‍💼 Knowledge Manager Role

| Responsibility | Action |
|---|---|
| Creation | Oversee knowledge object creation |
| Governance | Set naming conventions |
| Data Prep | Normalize event data |
| Modeling | Build data models for Pivot |

---

## 8. 💾 Reports

| Setting | Detail |
|---|---|
| **How to save** | Save As → Report |
| **Title** | Required (use a naming convention, e.g. `Type_Description`) |
| **Description** | Optional — explain intent/scope |
| **Content Type** | Table, Visualization, or both |
| **Time Picker** | Can be locked or left editable |

### 🔐 Report Permissions

| Role | Access |
|---|---|
| Standard User | Private by default (creator only) |
| Power User | Read & write, can edit shared reports |
| Admin | Can widen scope: Private → App → All Apps |

> ⚙️ **Run As:** Owner (creator's permissions) or User (viewer's permissions)

### ⏱️ Scheduling
- Run **hourly / daily / weekly / monthly**
- Lock to a fixed time range
- Trigger actions: email, script, webhook

---

## 9. 📊 Dashboards

| Feature | Options |
|---|---|
| Visual types | Charts, Tables, Trellis |
| Save to | New or existing dashboard |
| Framework | Classic Dashboards vs Dashboard Studio |
| Add panels | Edit → Add Panel |

### Ways to Add a Panel
1. **New Panel** — write a fresh SPL query
2. **New From Report** — reuse a saved report
3. **Clone** — duplicate a panel from another dashboard
4. **Add Prebuilt Panel** — reuse shared modules

### Editing Controls
- **UI mode** (drag & drop) vs **Source mode** (raw code)
- Theme toggle (e.g. Dark Theme)
- Input filters (text box, dropdown, time picker)
- Per-panel overrides (search, viz type, title, colors)
- **Drilldown Editor** — define click behavior (open link, filter, etc.)

---

## 10. 🎨 Dashboard Studio: Absolute vs Grid Layout

| Feature | Absolute Layout | Grid Layout |
|---|---|---|
| Charts | ✅ | ✅ |
| Custom background color | ✅ | ❌ |
| Custom canvas size | ✅ Free-form | ⚠️ Row height/width only |
| Visualization count | ✅ Unlimited | ⚠️ Depends on widths |
| Shapes (rect/line/ellipse) | ✅ | ❌ |
| Icons | ✅ | ❌ |
| Images (up to 16MB) | ✅ | ❌ |

- **Classic Dashboards** → backend **XML**
- **Dashboard Studio** → native **JSON** definition
- Toolbar: Add Visualizations/Inputs, Add Markdown, view configs, manage data sources
