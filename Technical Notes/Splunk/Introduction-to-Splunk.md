# 🔍 Introduction to Splunk & Search Processing Language (SPL)

A practical reference guide covering core Splunk architecture, key concepts, and foundational Search Processing Language (SPL) commands for Security Operations Center (SOC) analysis.

---

## 📌 What is Splunk?

**Splunk** is a leading Security Information and Event Management (SIEM) and data analysis platform. It enables SOC analysts, threat hunters, and IT engineers to collect, index, search, analyze, and visualize machine-generated log data in real time.

> **Primary SOC Use Cases:**
> * **Log Aggregation:** Centralizing logs from firewalls, endpoints, servers, and cloud environments.
> * **Threat Detection & Incident Response:** Querying security logs to detect unauthorized access, malware activity, or policy violations.
> * **Compliance & Reporting:** Retaining audit trails and automating compliance documentation.

---

## 🏗️ Core Architecture & Deployment

Splunk processes log data across three primary architectural tiers:

| Component | Role | Description |
| :--- | :--- | :--- |
| **Forwarders** | Data Collection | Lightweight agents deployed on target endpoints/servers to collect and forward logs. |
| **Indexers** | Storage & Parsing | Parses, indexes, and securely stores raw machine data for fast retrieval. |
| **Search Heads** | Analytics & UI | Serves as the user interface where analysts execute SPL queries, build dashboards, and configure alerts. |

### Deployment Models
* **Splunk Enterprise:** Self-hosted deployment (on-premises or custom cloud instance).
* **Splunk Cloud Platform:** Fully managed SaaS offering maintained by Splunk.

---

## ⚡ Search Processing Language (SPL) Basics

Search Processing Language (SPL) is the query language used to search, filter, and transform log data within Splunk.

### 1. Basic Searching

Searches should always be scoped using specific metadata fields (`index`, `sourcetype`, or `host`) to optimize search execution time.

```spl
# Search all indexed events in the main index
index=main

# Search for failed login attempts within Linux authentication logs
index=security sourcetype=linux_secure "Failed password"
```

> 💡 **Performance Tip:** Scoping queries by `index` and `sourcetype` significantly reduces search runtime and memory overhead.

---

### 2. Transforming Data (`stats`)

The `stats` command calculates aggregate statistics over dataset results (similar to SQL `GROUP BY`).

#### **Count Events by Source IP**
```spl
index=main 
| stats count by src_ip
```
* **Purpose:** Groups matching events by the source IP address (`src_ip`) and displays total event counts.

#### **Identify Top Talkers (Sorted)**
```spl
index=main 
| stats count by src_ip 
| sort - count
```
* **Purpose:** Orders source IP addresses by event volume in descending order to identify scanning activity, volumetric attacks, or anomalous behavior.

---

## 🛠️ Quick Reference: Essential SPL Commands

| Command | Description | Example Query |
| :--- | :--- | :--- |
| `fields` | Selects or removes specific fields to optimize output. | `... \| fields src_ip, dest_ip, action` |
| `table` | Formats results into a clean tabular layout. | `... \| table _time, src_ip, dest_ip, status` |
| `where` | Filters search results using logical expressions. | `... \| where count > 100` |
| `rename` | Renames fields for clearer presentation in reports. | `... \| rename src_ip AS "Source IP"` |
| `timechart` | Generates statistical trend visualizations over time. | `... \| timechart count by action` |
