# Cisco - Introduction To Splunk

## 1. What is Splunk?

Splunk is a unified platform that allows teams to work together or individually to ensure mission-critical digital systems stay secure and reliable.

### Core Capabilities

### 🔍 Searching with SPL
* **Cross-Source Analysis:** By entering a query into the Splunk search bar, you can find events that contain values across multiple data sources.
* **Statistical Analysis:** This allows you to analyze and run statistics on the events using **SPL (Search Processing Language)**.

### 📊 Reports & Dashboards
* **Saved Insights:** Search results can be saved to **Reports**, which provide ongoing insights into your data.
* **Visual Panels:** These saved reports can be directly used to power **Dashboard Panels** for real-time monitoring.

### 📐 Data Models & Pivot
* **Structured Datasets:** Knowledge can be organized into structured datasets called **Data Models**.
* **No-Code Visualization:** This allows users to quickly visualize data in **Pivot** without having to write custom searches.

### 🚨 Alerts
* **Custom Triggers:** Alerts allow you to set custom triggers so you are automatically notified of anomalies.
* **Incident Response:** This ensures you can respond to operational or security incidents as soon as they occur.

---

## 2. Splunk Web & Interface Notes

Comprehensive study notes covering Splunk Web components, user roles, application structure, and search fundamentals based on the Cisco Introduction to Splunk course material.

### 🧩 Splunk Apps Overview
* **Definition**: Apps are pre-configured environments sitting on top of a Splunk instance.
* **Purpose**: They extend built-in knowledge, visualizations, and capabilities.
* **Design**: Built specifically to solve a targeted use case or manage specific data sources.

### 👥 Splunk User Roles & Permissions
Roles determine exactly what a user can see, do, and interact with inside the platform.

| Role | Permissions & Capabilities |
| :--- | :--- |
| **Admin** | <ul><li>Most powerful system role</li><li>Can install apps and configurations</li><li>Can ingest and onboard new data</li><li>Can create global knowledge objects accessible to all users</li></ul> |
| **Power** | <ul><li>Can create and share knowledge objects with all users of a specific app</li><li>Can perform real-time data searches</li></ul> |
| **User** | <ul><li>Most restrictive default role</li><li>Can only see their own created knowledge objects</li><li>Can only see external knowledge objects explicitly shared with them</li></ul> |

> **Note**: Splunk also includes cloud-specific roles optimized for hosted environments.

### 🖥️ Splunk Web Home Interface
The Splunk Web Home screen serves as your primary launcher and management console.
* **Home Screen**: Used to launch, configure, and manage installed Splunk applications.
* **Documentation**: Built-in reference materials and guides accessible directly from the UI.
* **Customization**: Allows setting any custom dashboard as your default system landing page.
* **Data Ingestion**: Serve as a starting point to add new apps and browse/onboard new data sources.

### 🔍 The Search and Reporting App
The **Search and Reporting App** provides the default, core interface for querying, searching, and analyzing indexed data.

#### The 8 Main Components of the Interface
| Component | Description |
| :--- | :--- |
| **1. Splunk Bar** | Topmost bar to switch apps, edit account, check system messages, access settings, monitor jobs, and open help. |
| **2. App Bar** | Located below the Splunk bar. Used to navigate within the specific application currently open. |
| **3. Search Bar** | The central input field used to type and execute Search Processing Language (SPL) queries. |
| **4. Time Range Picker** | Dropdown menu used to constrain or retrieve events over a specific real-time or historical time period. |
| **5. Data Summary Button** | Provides an aggregate view of the data broken down by **Hosts**, **Sources**, and **Source Types**. |
| **6. Table Views** | A user-interface (UI)-driven way to explore, clean, and prepare data without needing to write SPL commands. |
| **7. Search History Menu** | Allows you to view, filter, and easily rerun past searches that you have previously executed. |
| **8. Search Terms / Elements** | Interactive elements within results to click and instantly narrow down active search events. |

---

## 3. Investigating Security Issues & Using Search

### 🛡️ Investigating a Security Use Case
Splunk allows analysts to run targeted queries to pinpoint security anomalies within network and system logs.
* **Example Application**: To verify if a malicious actor is targeting your server with bad credentials, type `failed` into the search bar.
* **Time Constraint**: Combine the keyword search with a strictly bounded selection from the **Time Range Picker** to isolate the exact attack window.

### ⚙️ Search Interface Elements

| UI Element | Functionality & Capabilities |
| :--- | :--- |
| **Save As Menu** | Allows saving search queries as reusable **Reports**, persistent **Alerts**, dashboards, or event types. |
| **Search Result Tabs** | Switches views between the raw **Events** tab, structural **Patterns**, and analytical **Statistics/Visualizations**. |
| **Search Action Buttons** | Quick-action inline controls used to **Pause**, **Stop**, **Share**, **Print**, or **Export** an active search job. |
| **Fields Sidebar** | Located on the left panel; displays automatically discovered and extracted event attributes for rapid filtering. |

### 📊 Tabs, Data Viewers, & Data Patterns
* **Patterns Tab**: Automatically parses incoming result segments to isolate recurring data clusters, aiding anomaly detection.
* **Statistics & Visualization Tab**: Converts unstructured search records into structured tables, charts, heatmaps, or graphs.
* **Transforming Commands**: Advanced query commands that process data into statistical tables or metrics required to build visualizations.
* **No Event Sampling**: Disabled by default to ensure searches look at 100% of the raw records rather than a statistical fraction.

### ⏱️ Search Job Lifecycle & Exporting

#### Job Durations
* **Default Lifespan**: An ad-hoc search job remains active and cached for **10 minutes**. After 10 minutes, Splunk must rerun the query from scratch.
* **Shared Lifespan**: Saved or shared search jobs remain active for **7 days**. Anyone with access can view the exact, immutable snapshot of data you captured.

#### Sharing & Exporting
* **Share Action**: Copies a direct link or bookmark to share search states with other team members.
* **Export Tool**: Allows downloading structured results directly into file formats like **CSV** or **JSON**.

### 🚥 Search Mode Selectors
Search modes balance the speed of execution against the depth of metadata discovery.

| Search Mode | Performance & Behavior |
| :--- | :--- |
| **Smart Mode** | Default setting. Automatically toggles discovery behaviors based on whether you run a search or a transforming command. |
| **Fast Mode** | Maximum performance. Disables dynamic field discovery to return rapid matching events. |
| **Verbose Mode** | Maximum visibility. Discovers, extracts, and populates all available fields, even if it impacts speed. |

### 📈 Timeline Component
The **Timeline** acts as an interactive, graphical representation of events mapped across time.
* **Granular Drilldown**: Users can select specific hourly or minute-by-minute bars in the chart to inspect spikes.
* **Zoom Scope**: Offers options to instantly **Zoom In** on crowded clusters or **Zoom Out** to inspect wider historical context.

---

## 4. Exploring Events

### 📋 Event List Behavior
Executing an SPL query generates a structured results stream containing your individual data records.
* **Visual Highlighting**: Any text matching your explicit search terms is automatically highlighted in yellow within the results.
* **Sorting Order**: Events display in **reverse chronological order** by default, placing the newest log entries at the very top.
* **Time Alignment**: The timestamp assigned to each log adapts dynamically to the custom **timezone** configured within your user account settings.

### 🏷️ Default Selected Fields
Every returned event automatically tracks and isolates three fundamental identity components in the sidebar by default:
1. **Host**: The physical or virtual machine generating the logs.
2. **Source**: The specific directory file name or network port source.
3. **Sourcetype**: The structural format of the data (e.g., `syslog`, `access_combined`).

### 🖱️ Interactive Text Context Actions
Analysts can refine active query parameters directly by mousing over the text fields within a raw log.
* **Hover State**: Rolling over raw text strings highlights the individual segment for selection.
* **Context Options**: Clicking text opens an inline utility menu allowing you to:
  * **Add**: Inject the token into the current search bar logic.
  * **New**: Wipe the current query and launch a new search utilizing that term.
  * **Exclude**: Automatically append a negative condition (`NOT`) to filter that value out.
* **Term Removal**: Clicking the highlighted text modifier a second time reverses the choice and deletes it from your search.

### ℹ️ Event Information Dropdown
* **Info Button**: Selecting the disclosure arrow (`>`) next to an event opens a dropdown viewer detailing the full metadata payload, field-value pairs, and structural breakdown of that specific event.

---

# 🔍 5. Using Search Terms & Logic

## ⚔️ Search Syntax Rules

| Rule | Summary | Example |
| :--- | :--- | :--- |
| **Wildcard (`*`)** | Matches any ending sequence. | `fail*` → `fail`, `failed`, `failure` |
| **Case Sensitivity** | Search terms are **not** case-sensitive. Boolean operators **are**. | `failed` = `FAILED` |
| **Exact Phrase** | Use double quotes for exact matches. | `"failed password"` |
| **Escape Quotes** | Use `\` before quotation marks. | `"user \"admin\" logged in"` |

## 🛠️ Boolean Operators

| Operator | Purpose | Example |
| :--- | :--- | :--- |
| **AND** | Both conditions must match | `failed AND login` |
| **OR** | Either condition matches | `failed OR blocked` |
| **NOT** | Exclude matching events | `failed AND NOT user=admin` |

> ⚠️ **Boolean operators must be UPPERCASE** (`AND`, `OR`, `NOT`).

## 📊 Boolean Evaluation Order

Splunk evaluates searches in this order:

```text
🥇 NOT
   ↓
🥈 OR
   ↓
🥉 AND
```

Use **parentheses `( )`** to override the default order.

Example:

```spl
(failed OR blocked) AND password
```
---

## 6. SPL Structure & Search Best Practices

### 📐 The 5 Components of an SPL 
Splunk Search Processing Language (SPL) queries are constructed using up to five distinct elements, separated by a **Pipe (`|`)** character which passes the results from one component to the next.

| SPL Component | Definition / Purpose | Example from Image |
| :--- | :--- | :--- |
| **Search Terms** | Filter criteria to pinpoint specific raw log events. | `index=network sourcetype=cisco_wsa_squid usage=Violation` |
| **Command** | Tells Splunk what specific action to perform on the data. | `stats` |
| **Function** | The calculation or analytical process run by the command. | `count()` |
| **Argument** | The specific field or variable the function acts upon. | `usage` |
| **Clause** | Defines how the final results are grouped, renamed, or structured. | `as Visits` |


### ⚙️ Case Sensitivity Rules

| Element Type | Case Sensitivity | Example |
| :--- | :--- | :--- |
| **SPL Framework** | **Not** case-sensitive (Commands, Functions, Clauses). | `stats` = `STATS`, `as` = `AS` |
| **Data Specifics** | **Is** case-sensitive (Specific field values). | `usage=Violation` (will not match *violation*) |


### 🛡️ Performance & Optimization Best Practices
To optimize search performance and reduce hardware load, follow these structural guidelines:

* **⏳ Limit by Time First:** Always use the time picker or a time modifier to limit events; processing less data yields faster results.
* **🏷️ Filter by Index-Time Fields:** After time constraints, filter by default fields (`index`, `host`, `source`, `sourcetype`). These are extracted during data onboarding and are highly indexed for speed.
* **🎯 Specificity Wins:** The more explicit details you provide to the search engine, the faster it can isolate relevant results.
* **➕ Inclusion over Exclusion:** Tell Splunk what to look for rather than what to ignore. Positive inclusion is computationally more efficient than exclusion.
* **🔀 Avoid Heavy Wildcards:** Use the `OR` or `IN` operators to specify multiple exact terms instead of broad wildcard tokens.
* **📉 Filter Early:** Apply filtering commands early in the query sequence to immediately minimize the volume of events passed down the pipe.

---

# What are Knowledge Objects?

## Why Knowledge Objects are Useful

* Can be created by one user and shared to other users
* Can be saved and reused by multiple people and apps
* Can be used in search
* Powerful tools for Splunk deployment

---

## Categories of Knowledge Objects

| Category | Included Objects | Description |
| --- | --- | --- |
| **Data Interpretation** | • Fields <br>

<br> • Field Extractions <br>

<br> • Calculated Fields | Perform calculations based on the values of existing fields. |
| **Data Classification** | • Event Types <br>

<br> • Transactions | Conceptually-related events that span time. |
| **Data Enrichment** | • Lookups <br>

<br> • Workflow Actions | Interact with external resources or narrow our search. |
| **Data Normalization** | • Tags <br>

<br> • Field Aliases | Field aliases give you a way to normalize data over multiple sources. |
| **Data Models** | • Hierarchically structured Datasets | Create data models for Pivot users. |

---

## Knowledge Manager Responsibilities

| Responsibility | Action |
| --- | --- |
| **Creation** | Oversee knowledge object creation |
| **Governance** | Implement naming conventions |
| **Data Prep** | Normalize Event Data |
| **Modeling** | Create Data Models and creating data models for Pivot users |

---


